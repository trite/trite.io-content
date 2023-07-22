---
title: Weird build error in dotnet project file extension not recognized
created: 2023-04-01
---

Ran into this error today in a F# project:

FSC : error FS0226: The file extension of 'C:\Users\username\AppData\Local\Programs\Microsoft VS Code\GitHub Copilot' is not recognized. Source files must have extension .fs, .fsi, .fsx, .fsscript, .ml or .mli.

This took a minute to figure out the cause, and I have no idea how it happened in the first place. The issue was that this `Compile` line somehow made it into my .fsproj file:

```xml
  <ItemGroup>
    ...
    <Compile Include="..\..\..\..\..\..\Users\username\AppData\Local\Programs\Microsoft VS Code\GitHub Copilot" />
    ...
  </ItemGroup>
```

Removing this fixed it.
