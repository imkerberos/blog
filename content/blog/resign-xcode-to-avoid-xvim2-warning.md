+++
title = "对 Xcode 重新签名解决 Xvim2 加载警告"
author = ["Evilee"]
date = 2021-03-04
lastmod = 2021-03-04T15:44:58+08:00
tags = ["Xcode"]
categories = ["计算机"]
draft = false
creator = "Emacs 27.1 (Org mode 9.5 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

自从 Xcode 禁止非官方插件后, 重新签名 Xcode 可以重新开启插件加载(Xvim2), 但是在使用命令行的 xcodebuild 时, 会出现如下的错误:
<!--more-->

```bash
021-03-04 15:33:29.144 xcodebuild[29907:2618716] WARNING: Failed to load plugin at path: "/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin", skipping. Error: Error Domain=NSCocoaErrorDomain Code=3587 "dlopen_preflight(/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2): no suitable image found.  Did find:
        /Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2: code signature in (/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2) not valid for use in process using Library Validation: mapped file has no Team ID and is not a platform binary (signed with custom identity or adhoc?)
        /Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2: stat() failed with errno=1" UserInfo={NSLocalizedFailureReason=The bundle is damaged or missing necessary resources., NSLocalizedRecoverySuggestion=Try reinstalling the bundle., NSFilePath=/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2, NSDebugDescription=dlopen_preflight(/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2): no suitable image found.  Did find:
        /Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2: code signature in (/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2) not valid for use in process using Library Validation: mapped file has no Team ID and is not a platform binary (signed with custom identity or adhoc?)
        /Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin/Contents/MacOS/XVim2: stat() failed with errno=1, NSBundlePath=/Users/kerberos/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XVim2.xcplugin, NSLocalizedDescription=The bundle “XVim2” couldn’t be loaded because it is damaged or missing necessary resources.}
```

xcodebuild 命令显示出如上的错误, 可以用以下的命令验证:

```bash
xcodebuild -list > /dev/null
```

重新签名:

```bash
codesign -dvv /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild
codesign -v -s XcodeSigner -f --timestamp=none /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild
codesign -dvv /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild
xcodebuild -list > /dev/null
```

没有告警了, 万事 OK.
