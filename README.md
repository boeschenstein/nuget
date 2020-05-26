# nuget

## Install nupkg to Local Nuget Store

1) create a folder c:\temp\nuget
2) Install nupkg in this folder:
`nuget add mypackage.0.0.0.nupkg -source c:\temp\nuget`
3) In Visual Studio (2019), add this folder to the avialable nuget servers

Source: <https://medium.com/@churi.vibhav/creating-and-using-a-local-nuget-package-repository-9f19475d6af8>

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
