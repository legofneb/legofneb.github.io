---
layout: post
title:  "Npm Install in CI Build"
date:   2016-02-16
categories: javascript
---

I had some difficulty running `npm install` on the server. Mainly, it could take anywhere from 2 minutes - 10 minutes to run depending on the time of day. Since we use gated check-ins, this just won't do.

I tried several solutions but ran into errors on each of them:

* Checking in node_modules (files that begin with $ cannot be checked into TFS, core-js was a culprit of this)
* Use local-npm (reduced the time by about 50%, but 1-5 minutes for a build is still too long and it requires configuration on the build server)
* Globally installed modules (yeah... this was never a good idea, ran into way too many modules being installed globally as I moved more tasks to the build server)

Here is the solution that actually worked well. It takes inspiration from the `pac` npm module, which tarballs the node_modules directory and then extracts it on the server. I couldn't use `pac` because it does not work with npm 3. Instead, I created 2 powershell scripts. The first packs a copy of the node_modules directory, while the second unpacks it on the server.

Here is the pack script:

```
powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::CreateFromDirectory('node_modules', 'nodeModulesBackup.zip', 'NoCompression', $false); }"
```

Pretty simple. All it does is takes your local node_modules and packs them up. Now we can check in 'nodeModulesBackup.zip' and on the server we run the unpack command.

```
powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::ExtractToDirectory('nodeModulesBackup.zip', 'node_modules'); }"
```

This has now taken our javascript build times from 2-10 minutes to a predictable 45 seconds. I can live with this. Just be sure to add `<Exec Command='.\upack-command.cmd' />` in your build target. You may also have to run `npm rebuild` if the build server runs a different system architecture than your local machine. My Windows 7 node_modules were compatible with a Server 2012 build server.

I could optimize this further. The zip command is relatively slow for unpacking the file. For comparison, unpacking a .tar of the node_modules directory took ~5 seconds. I didn't include this since it would require an additional executable, but may do it in the future to save 40 seconds.