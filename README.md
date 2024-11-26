# nuget

## CLI

[CLI Reference](https://docs.microsoft.com/en-us/nuget/reference/nuget-exe-cli-reference)

## Warning - outdated nugets

`dotnet list package --include-transitive --deprecated`

<https://learn.microsoft.com/en-us/nuget/reference/errors-and-warnings/nu1901-nu1904>
<[https://devblogs.microsoft.com/nuget/nugetaudit-2-0-elevating-security-and-trust-in-package-management/](https://devblogs.microsoft.com/nuget/nugetaudit-2-0-elevating-security-and-trust-in-package-management/#dotnet-nuget-audit-fix)>

`dotnet nuget audit fix`

## Install nupkg to Local Nuget Store

1) create a folder c:\temp\nuget
2) Install nupkg in this folder:
`nuget add mypackage.0.0.0.nupkg -source c:\temp\nuget`
3) In Visual Studio (2019), add this folder to the avialable nuget servers

Source: <https://medium.com/@churi.vibhav/creating-and-using-a-local-nuget-package-repository-9f19475d6af8>

## Get nuget from protected server

`nuget install My.Cool.Nuget -source https://some.where.azure.com/company/_packaging/product/nuget/v3/index.json`

## Config available Nuget Servers in Solution

Add this a config file: <MySolution>\.nuget\NuGet.Config in the folder where <MySolution>.sln sits

Example config file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <solution>
        <add key="disableSourceControlIntegration" value="true" />
    </solution>
    <packageRestore>
        <add key="enabled" value="True" />
    </packageRestore>
  <packageSources>
    <add key="server1" value="https://tfs.myserver.org/tfs/ABC/_packaging/DEF/nuget/v3/index.json" />
    <add key="server2" value="https://tfs.myserver.org/tfs/ABC/_packaging/GHI/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

## Create .NET Standard nuget

<details>
  <summary>Example for nuget config in csproj:</summary>

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>
  
 <PropertyGroup>
    <!-- where should the nuget package be created at -->
    <PackageOutputPath>./nupkg</PackageOutputPath>
    
    <!-- nuget related properties -->
    <Authors>Sayed Ibrahim Hashimi</Authors>
    <Description>Sample library showing how to create a .NET library.</Description>
    <Version>1.0.0</Version>
    <Copyright>Copyright 2020 © Sayed Ibrahim Hashimi. All rights reserved.</Copyright>
    <PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>
    <RepositoryUrl>https://github.com/sayedihashimi/sayedha.samplelibrary</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <PackageIconUrl>https://raw.githubusercontent.com/sayedihashimi/sayedha.samplelibrary/master/assets/icon-120x120.png</PackageIconUrl>
    <PackageIcon>icon-120x120.png</PackageIcon>
  </PropertyGroup>
  <ItemGroup>
    <None Include="icon-120x120.png" Pack="true" PackagePath="\"/>
  </ItemGroup>
</Project>
```
</details>

https://devblogs.microsoft.com/visualstudio/creating-and-packaging-net-standard-library/

Cross Platform Targeting: <https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting>

## nuget pack

- use 'nuget pack' for .net framework
- use 'dotnet pack' for .net Standard and .net core (first: convert csproj to sdk-style and Migrate from packages.config to PackageReference)

### examples

`nuget.exe pack .\myApp\myProj.csproj -NonInteractive -OutputDirectory c:\temp\nuget -Properties Configuration=Release -IncludeReferencedProjects -Verbosity normal`

"The -IncludeReferencedProject flag only really makes sense when your project is managing dependencies via packages.config file." According https://github.com/NuGet/Home/issues/5720

`dotnet pack .\myApp\myProj.csproj --output c:\temp\nuget --configuration Release --verbosity normal`

### Error NU5014 in `nuget pack`

running this statement: 

>nuget.exe pack .\myApp\myProj.csproj -NonInteractive -OutputDirectory c:\temp\nuget -Properties Configuration=release -IncludeReferencedProjects -Verbosity normal

Referencing a .net Standard project gives this error:

>Error NU5014: Error occurred when processing file 'C:\Work.proj\myApp\myProj.csproj': Unable to find 'bin\release\myApp\bin\release\'. Make sure the project has been built.

Options to solve this issue:

- refrence the .net Standard package by nuget, not by project reference
- in my case, the b.dll had to refrence c.dll as nuget package reference, not as project reference: a.dll -> b.dll -> c.dll
	if nuspec ist there, make sure the following depencency gets added to your nuspec in the nuget. If not, add this lines to nuspec (not needed: had been generated in my case)
	```xml
	<dependencies>
      <dependency id="myProj" version="2.0.0" />
    </dependencies>
	```
- remove argument -IncludeReferencedProjects (usually not recommended, because we usually want the references included)
	-  "The -IncludeReferencedProject flag only really makes sense when your project is managing dependencies via packages.config file."
		- according https://github.com/NuGet/Home/issues/5720
- check open issue: https://github.com/NuGet/Home/issues/3891 (closed, but not fixed: https://github.com/dotnet/sdk/issues/6688)

## Create your own nuget server
	
https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server

## Tipps and Tricks

### Reinstall packages

After updating framework or installing or updating nugets, it is reccommended to reinstall all packages:

```ps
Update-Package -Reinstall
Update-Package -Reinstall -ProjectName Project.Name.Here
Update-Package Package.Name.Here -Reinstall -ProjectName Project.Name.Here
```

### Binding

> Binding redirects are added if your app or its components reference more than one version of the same assembly.

<https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/how-to-enable-and-disable-automatic-binding-redirection?redirectedfrom=MSDN>

### Clear all local caches

In case of any nuget issues, clear all local nuget folders:

- `dotnet nuget locals all --clear`
- `nuget locals all -clear`

More details about managing folders: https://docs.microsoft.com/en-us/nuget/consume-packages/managing-the-global-packages-and-cache-folders

### install/restore error: Cycle detected

Do not use the same name for folder and package (example: dotnet add package IdentityServer4 in Folder \IdentityServer4)

```cmd
c:\Work.proj\IdentityServer4>dotnet add package IdentityServer4
  Determining projects to restore...
  Writing C:\Users\bop\AppData\Local\Temp\tmp4852.tmp
info : Adding PackageReference for package 'IdentityServer4' into project 'c:\Work.proj\IdentityServer4\IdentityServer4.csproj'.
info : Restoring packages for c:\Work.proj\IdentityServer4\IdentityServer4.csproj...
error: Cycle detected.
error:   IdentityServer4 -> IdentityServer4 (>= 0.0.0).
info : Package 'IdentityServer4' is compatible with all the specified frameworks in project 'c:\Work.proj\IdentityServer4\IdentityServer4.csproj'.
error: Value cannot be null. (Parameter 'path1')
```

### nuget pack error

- Use the latest version of nuget.exe (Azure DevOps: use "Nuget Tool Installer")
- for .net Standard, use `dotnet pack`

### nupkg contains source files instead of bin

Wrong argument: give csproj-file instead of nuspec-file

### Error 401 Unauthorized

Login Visual Studio with target account (try to disable vpn first)

### Linux

Slow? performance issues? check this:
Starting with .NET 8 SDK, verification is enabled by default. To opt out, set the environment variable DOTNET_NUGET_SIGNATURE_VERIFICATION to false.

<https://learn.microsoft.com/en-us/dotnet/core/tools/nuget-signed-package-verification#linux>
