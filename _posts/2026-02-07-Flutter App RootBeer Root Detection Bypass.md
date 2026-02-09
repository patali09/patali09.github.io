---
title: Flutter App RootBeer Root Detection Bypass
date: 2026-02-08 14:10:00 +0800
categories: [Android]
tags: [Root Detection Bypass, Frida Hooking,Android App Security, App Security]
---
# RootBeer Root Detection Bypass in Flutter Application

Target Package: [https://pub.dev/packages/root_checker_plus](https://pub.dev/packages/root_checker_plus)

We had a target application `com.mysafebank.package` (redacted). We opened our target application into `jadx-gui` . Inside `lib` , there was `libapp.so` and `libflutter.so,` which made us clear that application is developed with flutter framework.

![image.png](../images%2FFlutter%20App%20RootBeer%20Root%20Detection%20Bypass%2Fimage.png)

Looking futher we found RootBeer. 

![2.png](../%2Fimages%2FFlutter%20App%20RootBeer%20Root%20Detection%20Bypass%2F2.png)

Analysing further, we found out that it was using the flutter package: [https://pub.dev/packages/root_checker_plus](https://pub.dev/packages/root_checker_plus)

At very first, we decompiled our target application with apktool.

```jsx
apktool d com.mysafebank.package.apk
```

Then, looking inside `~/com.mysafebank.package.apk/lib/arm64-v8a/` ,

![image.png](../images%2FFlutter%20App%20RootBeer%20Root%20Detection%20Bypass%2Fimage%201.png)

We disassemble the `libapp.so` using blutter and searched for `root_checker_plus.dart`.

```jsx
// lib: root_checker_plus, url: package:root_checker_plus/root_checker_plus.dart

// class id: 1049842, size: 0x8
class :: {
}

// class id: 499, size: 0x8, field offset: 0x8
abstract class RootCheckerPlus extends Object {

  static _ isRootChecker(/* No info */) async {
    // ** addr: 0x639f70, size: 0x70
    // 0x639f70: EnterFrame
    //     0x639f70: stp             fp, lr, [SP, #-0x10]!
    //     0x639f74: mov             fp, SP
    // 0x639f78: AllocStack(0x28)
    //     0x639f78: sub             SP, SP, #0x28
    // 0x639f7c: SetupParameters()
    //     0x639f7c: stur            NULL, [fp, #-8]
    // 0x639f80: CheckStackOverflow
    //     0x639f80: ldr             x16, [THR, #0x48]  ; THR::stack_limit
    //     0x639f84: cmp             SP, x16
    //     0x639f88: b.ls            #0x639fd8
    // 0x639f8c: InitAsync() -> Future<bool?>
    //     0x639f8c: add             x0, PP, #8, lsl #12  ; [pp+0x8f10] TypeArguments: <bool?>
    //     0x639f90: ldr             x0, [x0, #0xf10]
    //     0x639f94: bl              #0x3baea8  ; InitAsyncStub
    // 0x639f98: r16 = <bool>
    //     0x639f98: ldr             x16, [PP, #0x1840]  ; [pp+0x1840] TypeArguments: <bool>
    // 0x639f9c: r30 = Instance_MethodChannel
    //     0x639f9c: add             lr, PP, #0x1c, lsl #12  ; [pp+0x1c340] Obj!MethodChannel@937d01
    //     0x639fa0: ldr             lr, [lr, #0x340]
    // 0x639fa4: stp             lr, x16, [SP, #8]
    // 0x639fa8: r16 = "isRootChecker"
    //     0x639fa8: add             x16, PP, #0x1c, lsl #12  ; [pp+0x1c348] "isRootChecker"
    //     0x639fac: ldr             x16, [x16, #0x348]
    // 0x639fb0: str             x16, [SP]
    // 0x639fb4: r4 = const [0x1, 0x2, 0x2, 0x2, null]
    //     0x639fb4: ldr             x4, [PP, #0x48]  ; [pp+0x48] List(5) [0x1, 0x2, 0x2, 0x2, Null]
    // 0x639fb8: r0 = invokeMethod()
    //     0x639fb8: bl              #0x86c040  ; [package:flutter/src/services/platform_channel.dart] MethodChannel::invokeMethod
    // 0x639fbc: mov             x1, x0
    // 0x639fc0: stur            x1, [fp, #-0x10]
    // 0x639fc4: r0 = Await()
    //     0x639fc4: bl              #0x3bac68  ; AwaitStub
    // 0x639fc8: cmp             w0, NULL
    // 0x639fcc: b.ne            #0x639fd4
    // 0x639fd0: r0 = false
    //     0x639fd0: add             x0, NULL, #0x30  ; false
    // 0x639fd4: r0 = ReturnAsync()
    //     0x639fd4: b               #0x3ef968  ; ReturnAsyncStub
    // 0x639fd8: r0 = StackOverflowSharedWithoutFPURegs()
    //     0x639fd8: bl              #0x8d5080  ; StackOverflowSharedWithoutFPURegsStub
    // 0x639fdc: b               #0x639f8c
  }
}

```

On analyzing the script, 

```jsx
 0x639fc8: cmp       w0, NULL
 0x639fcc: b.ne      #0x639fd4
```

If we update the `b.ne` to perform `NOP instruction`, we might can bypass the root detection.

We use frida for patching in runtime. We wrote simple script and started the application by hooking with frida and successfully bypassed the root detection.

```jsx
Process.attachModuleObserver({
    onAdded: function(module) {
        if (module.name === "libapp.so") {
            const branch_offset = 0x639fcc;
            const branch_addr = module.base.add(branch_offset);
            console.log("Changing memory permissions");
            Memory.protect(branch_addr, 4, 'rwx');
            console.log("Overwriting instruction");
            branch_addr.writeByteArray([0x1F, 0x20, 0x03, 0xD5]);
            }
        },
    onRemoved: function(module) {}
})
```

We can run this script  `frida -U -f 'com.mysafebank.package' -l rootBypassScript.js`