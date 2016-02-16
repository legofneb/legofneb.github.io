---
layout: post
title:  "Javascript Compiling in MSBuild for Continuous Integration"
date:   2016-02-16
categories: javascript
---

Javascript build tools have become so easy to use between grunt, gulp, or even npm. However; it is not so easy to implement javascript build tools with msbuild. Here is how I added the build tools with msbuild.

Add a Target which runs your scripts *after* your Imports.

```
<Target Name="NpmInstall" Condition="'$(ConfigurationName)' != 'Debug'" AfterTargets="BeforeBuild">
    <Exec Command=".\node_modules\.bin\grunt compile" />
    <Exec Command="npm test" />
</Target>
```

Be sure to attach to the BeforeBuild Target rather than the AfterBuild. I found that MSDeploy picked up files before the AfterBuild event occured. I also used the `'$(ConfigurationName)' != 'Debug'` condition to ensure that the `grunt compile` and `npm test` commands only run on the build server, rather than locally.

You may have noticed that I omitted the `npm install` command. I intentionally did this for performance and posted my solution in [this post]({% post_url 2016-02-16-npm-install-in-ci-build %}).

So we have compiled the files, now we need include our files in our deploy process.

```
<Target Name="Include Generated Files" AfterTargets="BeforeBuild">
    <ItemGroup>
      <Content Include="Scripts\dist\**\*.js" Condition="'$(ConfigurationName)' != 'Debug'" />
      <Content Include="Content\stylesheets\**\*.css" Condition="'$(ConfigurationName)' != 'Debug'" />
    </ItemGroup>
</Target>
```

The key is wrapping the ItemGroup within a target, otherwise everytime the solution is reloaded locally, visual studio would attempt to add every valid file. This way, the files are only added in our CI build. Additionally, our compiled files are now a part of the solution so we can use msdeploy or powershell scripts and this will work.