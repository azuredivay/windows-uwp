---
description: C#/WinRT is a tool set that provides WinRT projection support for C# code.
title: C#/WinRT
ms.date: 05/19/2020
ms.topic: article
keywords: windows 10, uwp, standard, c#, winrt, cswinrt, projection
ms.localizationpriority: medium
---

# C#/WinRT

> [!IMPORTANT]
> C#/WinRT is in public preview and may be substantially modified before final release. Microsoft makes no warranties, express or implied, with respect to the information provided here.

C#/WinRT is a NuGet-packaged toolkit that provides Windows Runtime (WinRT) projection support for the C# language. A *projection* is a translation layer, such as an interop assembly, that enables programming WinRT APIs in a natural and familiar way for the target language. For example, the C#/WinRT projection hides the details of interop between C# and WinRT interfaces, and provides mappings of many WinRT types to appropriate .NET equivalents, such as strings, URIs, common value types, and generic collections.

C#/WinRT currently provides support for consuming WinRT types, and the current preview enables you to [create](#create-an-interop-assembly) and [reference](#reference-an-interop-assembly) WinRT interop assemblies. Future releases of C#/WinRT will add support for authoring WinRT types in C#.

For additional information about C#/WinRT, see the [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo).

## Motivation for C#/WinRT

[.NET Core](/dotnet/core/) is the focus for the .NET platform, and .NET 5 is the next major release. It is an open-source, cross-platform runtime that can be used to build device, cloud, and IoT applications.

Previous versions of .NET Framework and .NET Core had built-in knowledge of WinRT, a Windows-specific technology. To support the portability and efficiency goals of .NET 5, we lifted the WinRT projection support out of the .NET compiler and runtime and moved it into the C#/WinRT toolkit. The goal of C#/WinRT is to provide parity with the built-in WinRT support provided by earlier versions of the C# compiler and .NET runtime. For details, see [.NET mappings of Windows Runtime types](../winrt-components/net-framework-mappings-of-windows-runtime-types.md).

C#/WinRT also supports WinUI 3.0. This release of WinUI lifts native Microsoft UI controls and features out of the operating system. This enables app developers to use the latest controls and visuals on Windows 10, version 1803, and later releases.

Finally, C#/WinRT is a general toolkit and is intended to support other scenarios where built-in support for WinRT is not available in the C# compiler or .NET runtime. C#/WinRT supports versions of the .NET runtime compatible down to .NET Standard 2.0, such as Mono 5.4.

## Create an interop assembly

WinRT APIs are defined in Windows Metadata (*.winmd) files. The C#/WinRT NuGet package ([Microsoft.Windows.CsWinRT](https://www.nuget.org/packages/Microsoft.Windows.CsWinRT/)) includes the C#/WinRT compiler, **cswinrt**, which you can use to process Windows Metadata files and generate .NET 5.0 C# code. You can compile these source files to interop assemblies, similar to how [C++/WinRT](../cpp-and-winrt-apis/index.md) generates headers for C++ language projection. You can then distribute the C#/WinRT interop assemblies for applications to reference, along with the C#/WinRT runtime assembly.

For a walkthrough that demonstrates how to create and distribute an interop assembly as a NuGet package, see [Walkthrough: Generate a .NET 5 projection from a C++/WinRT component and update the NuGet](net-projection-from-cppwinrt-component.md).

### Invoke cswinrt.exe

To invoke cswinrt.exe from a project, install the latest [C#/WinRT NuGet package](https://www.nuget.org/packages/Microsoft.Windows.CsWinRT/). You can then set C#/WinRT-specific project properties in a **C# Library** project to generate an interop assembly. The following project fragment demonstrates a simple invocation of **cswinrt** to generate projection sources for types in the Contoso namespace. These sources are then included in the project build.

```xml
<PropertyGroup>
  <CsWinRTIncludes>Contoso</CsWinRTIncludes>
</PropertyGroup>
```

In this project you would also need to reference the CsWinRT NuGet package and the project-specific .winmd files you want to project, whether through a NuGet package, project reference or direct reference. By default, the **Windows** and **Microsoft** namespaces are not projected. For a full list of CsWinRT project properties, refer to the [CsWinRT NuGet documentation](https://github.com/microsoft/CsWinRT/blob/master/nuget/readme.md).

### Distribute the interop assembly

An interop assembly is typically distributed as a NuGet package, along with a dependency on the C#/WinRT NuGet package for the required C#/WinRT runtime assembly, **winrt.runtime.dll**.

To ensure that the correct version of the C#/WinRT runtime is deployed for .NET 5.0 applications, include a `targetFramework` condition in the .nuspec file with a dependency on the C#/WinRT NuGet package.

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
  <metadata>
    <dependencies>
      <group targetFramework="net5.0">
        <dependency id="Microsoft.Windows.CsWinRT" version="0.9.0" />
      </group>
    </dependencies>
  </metadata>
</package>
```

> [!NOTE]
> The target framework moniker for .NET 5.0 is moving from ".NETCoreApp5.0" to "net5.0". 

## Reference an interop assembly

Typically, C#/WinRT interop assemblies are referenced by application projects. But they may also be referenced in turn by intermediate interop assemblies. For example, the WinUI interop assembly would reference the Windows SDK interop assembly.

To consume projected C#/WinRT types, add a reference to the appropriate C#/WinRT interop NuGet package to your project. This causes both the interop assembly and the C#/WinRT runtime assembly to be added to the project.

If you distribute a third-party WinRT component without an official interop assembly, an application project may follow the procedure for [creating an interop assembly](#create-an-interop-assembly) to generate its own private projection sources. We don't recommend this approach, because it can produce conflicting projections of the same type within a process. NuGet packaging, following the [Semantic Versioning](https://semver.org) scheme, is designed to prevent this. An official third-party interop assembly is preferred.

### WinRT type activation

C#/WinRT supports activation of WinRT types hosted by the operating system, as well as third-party components such as [Win2D](https://www.nuget.org/packages/Win2D.uwp/). Support for third-party component activation in a desktop application is enabled with [registration free WinRT activation](https://blogs.windows.com/windowsdeveloper/2019/04/30/enhancing-non-packaged-desktop-apps-using-windows-runtime-components/), available in Windows 10 version 1903 and later. This may also require use of the [VCRT Forwarders](https://www.nuget.org/packages/Microsoft.VCRTForwarders.140/) package, if the component was built to target UWP apps.

C#/WinRT also provides an activation fallback path if Windows fails to activate the type as described above. In this case, C#/WinRT attempts to locate a native implementation DLL based on the fully qualified type name, progressively removing elements. For example, the fallback logic would attempt to activate the Contoso.Controls.Widget type from the following modules, in sequence:

1. Contoso.Controls.Widget.dll
2. Contoso.Controls.dll
3. Contoso.dll

C#/WinRT uses the [LoadLibrary alternate search order](/windows/win32/dlls/dynamic-link-library-search-order#alternate-search-order-for-desktop-applications) to locate an implementation DLL. An app relying on this fallback behavior should package the implementation DLL alongside the app module.

## Known issues

There are some known interop-related performance issues in the current preview of C#/WinRT. These will be addressed prior to the final release in late 2020. Other known issues and breaking changes will be noted in the [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo).

If you encounter any functional issues with the C#/WinRT NuGet package, the cswinrt.exe compiler, or the generated projection sources, submit issues to us via the [C#/WinRT issues page](https://github.com/microsoft/CsWinRT/issues).

## Additional resources

* [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo)
