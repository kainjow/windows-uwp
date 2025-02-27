---
description: C#/WinRT is a tool set that provides WinRT projection support for C# code.
title: C#/WinRT
ms.date: 05/19/2020
ms.topic: article
keywords: windows 10, uwp, standard, c#, winrt, cswinrt, projection
ms.localizationpriority: medium
---

# C#/WinRT

C#/WinRT is a NuGet-packaged toolkit that provides Windows Runtime (WinRT) projection support for the C# language. A projection assembly is an interop assembly, which enables programming WinRT APIs in a natural and familiar way for the target language. The C#/WinRT projection hides the details of interop between C# and WinRT interfaces, and provides mappings of many [WinRT types to appropriate .NET equivalents](net-mappings-of-winrt-types.md), such as strings, URIs, common value types, and generic collections.

C#/WinRT currently provides support for consuming WinRT APIs through the use of [Target Framework Monikers](/windows/apps/desktop/modernize/desktop-to-uwp-enhance#net-5-use-the-target-framework-moniker-option) (TFMs) in .NET 5+. Setting the TFM with a specific Windows SDK version adds references to the Windows SDK projection and runtime assemblies generated by C#/WinRT.

The latest [C#/WinRT NuGet package](https://www.nuget.org/packages/Microsoft.Windows.CsWinRT/) enables you to [create](#create-and-distribute-an-interop-assembly) and [reference](#reference-an-interop-assembly) your own WinRT interop assemblies for .NET 5+ consumers. The latest C#/WinRT version also includes a [preview of authoring](create-windows-runtime-component-cswinrt.md) WinRT types in C#.

For additional information, see the [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo).

## Motivation for C#/WinRT

[.NET Core](/dotnet/core/) is the focus for the .NET platform, and .[NET 5](/dotnet/core/dotnet-five) is the latest major release. It is an open-source, cross-platform runtime that can be used to build device, cloud, and IoT applications.

Previous versions of .NET Framework and .NET Core had built-in knowledge of WinRT, a Windows-specific technology. To support the portability and efficiency goals of .NET 5, we [lifted the WinRT projection support out of the .NET compiler and runtime](/dotnet/core/compatibility/interop/5.0/built-in-support-for-winrt-removed) and moved it into the C#/WinRT toolkit. The goal of C#/WinRT is to provide parity with the built-in WinRT support provided by earlier versions of the C# compiler and .NET runtime. For details, see [.NET mappings of Windows Runtime types](net-mappings-of-winrt-types.md).

C#/WinRT also supports WinUI 3. This release of WinUI lifts native Microsoft UI controls and features out of the operating system. This enables app developers to use the latest controls and visuals on Windows 10, version 1803, and later releases.

Finally, C#/WinRT is a general toolkit and is intended to support other scenarios where built-in support for WinRT is not available in the C# compiler or .NET runtime.

## Create and distribute an interop assembly

WinRT APIs are defined in Windows Metadata (WinMD) files. The C#/WinRT NuGet package ([Microsoft.Windows.CsWinRT](https://www.nuget.org/packages/Microsoft.Windows.CsWinRT/)) includes the C#/WinRT compiler, **cswinrt.exe**, which you can use to process WinMD files and generate .NET 5 C# code. C#/WinRT compiles these source files into an interop assembly, similar to how [C++/WinRT](../cpp-and-winrt-apis/index.md) generates headers for the C++ language projection. You can then distribute the C#/WinRT interop assemblies for .NET 5 applications to reference.

To invoke **cswinrt.exe** from a project, install the latest [C#/WinRT NuGet package](https://www.nuget.org/packages/Microsoft.Windows.CsWinRT/) in a C# class library project. In this project you can reference the WinMD files you want to generate a projection assembly for, either through a NuGet package reference, project reference, or direct file reference. By default, the **Windows** and **Microsoft** namespaces are not projected, as C#/WinRT already generates these projections through support for the TFM and WinUI 3. For a full list of C#/WinRT project properties, refer to the [C#/WinRT NuGet documentation](https://github.com/microsoft/CsWinRT/blob/master/nuget/readme.md).

The following project fragment demonstrates a simple invocation of **cswinrt** to generate projection sources for types in the Contoso namespace. These sources are then included in the project build.

```xml
<PropertyGroup>
  <CsWinRTIncludes>Contoso</CsWinRTIncludes>
</PropertyGroup>
```

An interop assembly is typically distributed as a NuGet package along with the implementation assembly for .NET 5 applications to reference. For more details on how to create and distribute an interop assembly, see [Make a C# projection from a C++/WinRT component, distribute as a NuGet for .NET 5+ apps](net-projection-from-cppwinrt-component.md).

## Reference an interop assembly

Typically, C#/WinRT interop assemblies are referenced by application projects. But they may also be referenced in turn by intermediate interop assemblies. For example, the WinUI interop assembly would reference the Windows SDK interop assembly.

To consume projected C#/WinRT types, add a reference to the appropriate C#/WinRT interop NuGet package to your project. 

If you distribute a third-party WinRT component without an official interop assembly, an application project may follow the procedure for [creating an interop assembly](#create-and-distribute-an-interop-assembly) to generate its own private projection sources. We don't recommend this approach, because it can produce conflicting projections of the same type within a process. NuGet packaging, following the [Semantic Versioning](https://semver.org) scheme, is designed to prevent this. An official third-party interop assembly is preferred.

### WinRT type activation

C#/WinRT supports activation of WinRT types hosted by the operating system, as well as third-party components such as [Win2D](https://www.nuget.org/packages/Win2D.uwp/). Support for third-party component activation in a desktop application is enabled with [registration free WinRT activation](https://blogs.windows.com/windowsdeveloper/2019/04/30/enhancing-non-packaged-desktop-apps-using-windows-runtime-components/), available in Windows 10 version 1903 and later. This may also require use of the [VCRT Forwarders](https://www.nuget.org/packages/Microsoft.VCRTForwarders.140/) package, if the component was built to target UWP apps.

C#/WinRT also provides an activation fallback path if Windows fails to activate the type as described above. In this case, C#/WinRT attempts to locate a native implementation DLL based on the fully qualified type name, progressively removing elements. For example, the fallback logic would attempt to activate the Contoso.Controls.Widget type from the following modules, in sequence:

1. Contoso.Controls.Widget.dll
2. Contoso.Controls.dll
3. Contoso.dll

C#/WinRT uses the [LoadLibrary alternate search order](/windows/win32/dlls/dynamic-link-library-search-order#alternate-search-order-for-desktop-applications) to locate an implementation DLL. An app relying on this fallback behavior should package the implementation DLL alongside the app module.

## Common errors and troubleshooting

- Error: "Windows Metadata not provided or detected."

  You can specify Windows Metadata by using the `<CsWinRTWindowsMetadata>` project property, for example:

  ```xml
  <CsWinRTWindowsMetadata>10.0.19041.0</CsWinRTWindowsMetadata>
  ```

  In C#/WinRT version 1.2.1 and later, this property defaults to `TargetPlatformVersion`, which is derived from the Windows SDK version specified in the `TargetFramework` property.
  
- Error	CS0246: The type or namespace name 'Windows' could not be found (are you missing a using directive or an assembly reference?)

  To address this error, edit your `<TargetFramework>` property to target a specific Windows version, for example:
  ```xml
  <TargetFramework>net5.0-windows10.0.19041.0</TargetFramework>
  ```
  Refer to the docs on [Calling Windows Runtime APIs](/windows/apps/desktop/modernize/desktop-to-uwp-enhance) for more details on specifying the `<TargetFramework>` property.


### .NET SDK versioning errors

You may encounter the following errors or warnings in a project that is built with an earlier .NET SDK version than any of its dependencies.

| Error or warning message | Reason |
|--------------------------|--------|
| Warning MSB3277: Found conflicts between different versions of WinRT.Runtime or Microsoft.Windows.SDK.NET that could not be resolved. | This build warning occurs when referencing a library that exposes Windows SDK types on its API surface. |
| [Error CS1705](/dotnet/csharp/language-reference/compiler-messages/cs1705): Assembly 'AssemblyName1' uses 'TypeName' which has a higher version than referenced assembly 'AssemblyName2' | This build compiler error occurs when referencing and consuming exposed Windows SDK types in a library. |
| System.IO.FileLoadException | This runtime error may occur when calling certain APIs in a library that does not expose Windows SDK types. |

To fix these errors, update your .NET SDK to the latest version. Doing so will ensure that the runtime and Windows SDK assembly versions used by your application are compatible with all dependencies. These errors may occur with early servicing/feature updates to the .NET 5 SDK, because runtime fixes may require updates to our assembly versions.

## Known issues

Known issues and breaking changes are noted in the [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo).

If you encounter any functional issues with the C#/WinRT NuGet package, the cswinrt.exe compiler, or the generated projection sources, submit issues to us via the [C#/WinRT issues page](https://github.com/microsoft/CsWinRT/issues).

## Additional resources

* [C#/WinRT GitHub repo](https://aka.ms/cswinrt/repo)
