# nuget

## CLI

[CLI Reference](https://docs.microsoft.com/en-us/nuget/reference/nuget-exe-cli-reference)

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
- use 'dotnet pack' for .net standard and .net core (first: convert csproj to sdk-style and Migrate from packages.config to PackageReference)

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

## Tipps and Tricks

### Clear all local caches

In case of any nuget issues, clear all local nuget folders:

`dotnet nuget locals all --clear`
`nuget locals all -clear`

### nupkg contains source files instead of bin

Wrong argument: give csproj-file instead of nuspec-file
