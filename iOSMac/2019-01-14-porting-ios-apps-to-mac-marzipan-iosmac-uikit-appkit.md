# Marzipan - 移植iOS到mac

伴随着masOS Mojave的到来，Apple正在增加对在macOS上运行UIKit应用程序的支持，而无需在AppKit中重写UI。虽然还没正式开始对第三方开发人员支持，但让我们来探讨2019年会发生什么以及在今天尝试一下。



这片文章基于我在try! Swift New York大会上的“Hacking Marzipan”。[幻灯片在这里](https://speakerdeck.com/steipete/hacking-marzipan).



[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/2OuQarA0a7I/0.jpg)](https://youtu.be/2OuQarA0a7I)	



*⚠️*警告: 深挖iOSMac平台需要破解Mac，会弱化Mac的安全性，因此您应该使用其他单独的机器来探索这项新技术。暂时不要分发iOSMac应用程序 - 底层框架仍然是私有的，可能会发生变化。



## 什么是Marzipan？

Marzipan是Apple面向Mac的新iOS应用层。它合理的使用UIKit和大多数相关iOS框架编写的应用程序。你已经可以在Mac上运行iOS应用程序，并且你已经通过模拟器执行了多年。



Marzipan在某种程度上是模拟器的演变。但是它不使用模拟器的架构；它集成得更深入，并且包括支持在窗口添加按钮，菜单栏，焦点提示等等。还有拖放操作。



有趣的是：Marzipan拥有size classes和缩放因子为0.77 - 所有的绘制变得更小来适应Mac。



## 历史及时间轴

时至今日，UIKit并不是第一次运行在不是为它设计的平台上。Facebook有一个闭源的内部项目叫[OSMeta](https://twitter.com/steipete/status/931639019179528192),它几乎重写了整个iOS平台。微软拥有一个叫islandwood的项目，也叫做[Windows Bridge for iOS](https://developer.microsoft.com/en-us/windows/bridges/ios).在Github上你也可以找到一些像[Chameleon Project](http://chameleonproject.org/)的尝试者。然后，UIKit就像头巨大的野兽，实际上也只有Apple可以完整移植它。



当Mark Gurman于2017年末在[Bloomberg](https://www.bloomberg.com/news/articles/2017-12-20/apple-is-said-to-have-plan-to-combine-iphone-ipad-and-mac-apps)泄露出关于Marzipan的消息时，几乎没有人相信Apple真的会沿着这条路走下去。甚至他们自己的工程师也很惊讶当Craig Federighi选择在2018年WWDC上展示了一段相关内容[[sneak peek at WWDC 2018](https://9to5mac.com/2018/06/04/apple-gives-a-sneak-peek-at-multi-year-project-to-bring-uikit-ios-apps-to-the-mac/)](https://9to5mac.com/2018/06/04/apple-gives-a-sneak-peek-at-multi-year-project-to-bring-uikit-ios-apps-to-the-mac/)，因为直到2019年末macOS 10.15才会有第三方开发人员的官方SDK。



## iOSMac的架构

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/marzipan-features-wwdc2018-251359d9.png)



运行iOSMac应用程序将生成整个进程列表。有一个AppKit shell显示UIKit应用程序，然后是UIKitSystem.app，这是Mac的FrontBoard版本：

- /Applications/**VoiceMemos.app**
- /System/iOSSupport/System/Library/PrivateFrameworks/ VoiceMemos.framework/Support/**voicememod**
- /System/Library/CoreServices/**UIKitSystem.app**
- /System/Library/PrivateFrameworks/UIKitHostAppServices.framework/ Versions/A/XPCServices/**UIKitHostApp.xpc** (disguised)



如果你想知道得更多，看看Adam Demasi写的一篇[关于iOSMac架构内部](https://kirb.me/2018/06/07/iosmac-research.html)的文章。



## 当前破解的概况

Apple正在分阶段测试这个新平台。 macOS Mojave在iOSMac上运行新的Apple提供的应用程序，提供第1阶段。通过第2阶段，开发人员可以访问该平台。但是，有各种方法可以解决Apple目前对Marzipan的限制。



### [marzipan_hook](https://github.com/justMaku/marzipan_hook)

Michał Kałużny的破解是第一个同时也很值得提及的其中之一。他利用🍎的iOSMac应用，注入你的代码到一个应用里将它变成你自己的，就像病毒侵入了细胞，让细胞失去其原来的作用。这不是一种可扩展的方式，但是Kałużny极具创意。



## [MarzipanPlatter](https://github.com/biscuitehh/MarzipanPlatter)

早期，MarzipanPlatter是将iOS应用程序转换为macOS的最佳选择，我成功地将Mojave Beta 1上的PDF Viewer移植到Mac上。然而，我的方法混合了UIKit和AppKit，这是有缺陷的(但是[上了头条](https://9to5mac.com/2018/06/13/marzipan-in-mojave-porting-ios-apps-to-macos/))

> Got [@pdfviewerapp](https://twitter.com/pdfviewerapp?ref_src=twsrc%5Etfw) running on Marzipan 🤯
>
> Featuring  inline video, page curl, popovers, scrolling, text selection, inline  forms, adding text, color inspector, search, editing documents - almost  everything works. Took me half a day. Project is 1MLOC ObjC/C++.
>
> Major props to Apple! [pic.twitter.com/1ofpe5AqyM](https://t.co/1ofpe5AqyM)
>
> — Peter Steinberger (@steipete) 
>
> June 11, 2018



## [marzipanify](https://github.com/steventroughtonsmith/marzipanify)

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/marzipanify-70dc4ad3.jpg)



[Steven Troughton-Smith ](https://twitter.com/stroughtonsmith)提供最完整的转化工具 [marzipanify](https://github.com/steventroughtonsmith/marzipanify)。这是目前移植应用的最佳方式。它基本上执行以下操作：

- 添加 “Marzipan Glue”
- 贴个 Info.plist
- 修改Mach头文件
- 添加私用的iOSMac entitlements



我们将在本文中使用marzipanify。大力宣扬Steven的这个神奇的工具。如果你发现它很有用，可以考虑在Patreon [supporting him on Patreon](https://www.patreon.com/steventroughtonsmith)上支持他。



# 关掉你的安全防护 🙀

🍎早已锁了很多功能，所以你不能开箱即用的制作iOSMac应用 - 至少你要破解一下。到这里我们所谈及的破解就是我们需要关掉🍎系统内置的保护机制。这不是你日常使用中需要涉及的地方，我之前已经提及过，你最好使用另一台机器来体验它。

```shell
sudo csrutil disable
sudo nvram boot-args=“amfi_get_out_of_my_way=0x1”
```

这里所需的破解，你也是需要禁用沙盒。🍎控制了哪些应用可以获得iOSMac的entitlement，如果你没有那魔术般的amfi_get_out_of_my_way 启动参数，你的体验将提前终止。（amfi 是 AppleMobileFileIntegrity的缩写）。



## 虚拟机并不是总能行

iOSMac 平台需要GPU加速器，并在其不可用时简单的退出。

**更新**：苹果似乎正在积极研究这一缺陷。从Mojave Beta 11开始，一些应用（包括用于Mac的Home和PDF Viewer）现在在VM内运行，而News这个应用仍然无法加载。向[Michael Thomas](https://twitter.com/NSBiscuit)致敬，告诉我这个变化！

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/mojave-vm-1c70d9d6.png)

## 移植PDF Viewer 到Mac

[PDF Viewer](http://pdfviewer.io)是一个相当复杂的iOS应用程序，用C，C ++，Objective-C和Swift编写。它基于[PSPDFKit](https://pspdfkit.com/)，它本身有超过一百万行代码，自2010年以来一直在开发中。这使得一个非常引人注目的测试用例。



### 步骤一：最小的deployment target = iOS 12

您需要确保项目的最小部署目标，并且所有相关框架都设置为iOS 12.仅当设置了iOS 12时，链接器才会发出LC_BUILD_VERSION标志而不是早期的LC_BUILD_VERSION_MIN_MACOS，这是iOSMac加载正确依赖项所必需的。 marzipanify会尝试解决这个问题，但它会造成很多麻烦，你最终可能会得到一个无法启动的二进制文件。这是使用一个xcconfig文件有价值的地方：

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/deployment-targets-78602a79.png)



### 步骤二： 移除框架

删除使用已弃用的代码。 iOSMac是一个新平台，不包括已弃用的类。最有可能出问题的是UIWebView，尽管现在WKWebView已使用多年，但它仍然无处不在。大多数框架都可用，除非将它们移植到Mac是没有意义的，如下所示：

- `SafariServices.framework` (`SFSafariViewController` does not make sense on the Mac; use openURL)
- `CoreTelephony.framework` (`CTTelephonyNetworkInfo` especially)
- `Social.framework` (`SLComposeViewController`)
- `MessageUI.framework` (`MFMailComposeViewController`)
- `OpenGLES.framework` (rewritten to Metal)
- `AddressBook.framework` (too new?)



如果存在框架，则无法保证所有symbols都可用。例如，我们使用的是UIMarkupTextPrintFormatter，它自iOS 4.2以来一直是UIKit的一部分，但不在iOSMac中。启动时，您将很快看到缺少哪些symbols：

```shell
./PDFViewerMac

dyld: Symbol not found: _OBJC_CLASS_$_UIMarkupTextPrintFormatter
  Referenced from: /marzipanify/PDFViewer.app/
                   Contents/MacOS/./PDFViewer (which was built for Mac OS X 12.0)

  Expected in: /System/iOSSupport/System/Library/Frameworks/UIKit.framework/UIKit
 in /marzipanify/PDFViewer.app/Contents/MacOS/./PDFViewer
```

以下是不完整的symbols列表:

- `UIImpactFeedbackGenerator` (your Mac has no way to generate haptic feedback, but this should be a NOP instead)
- `UIPrintInfo` (probably related to `UIWebView` missing)
- `[UIViewController setNeedsUpdateOfHomeIndicatorAutoHidden:]`(Mojave’s UIKit branch might have been cut before the iPhone X branch was merged?)
- `UIDocumentBrowser` (the document browser concept makes no sense on the Mac)
- `PHCachingImageManager` (`Photos.framework` is a problem in and of itself)



我们选择了一种混合方法来解决这些问题，方法是在一个小的胶水式的文件中添加一些缺失的symbols：

```objective-c
@interface UIViewController (MarzipanSupport)
- (void)setNeedsUpdateOfHomeIndicatorAutoHidden;
@end

@implementation UIViewController (MarzipanSupport)
- (void)setNeedsUpdateOfHomeIndicatorAutoHidden {}
@end
```



### 步骤三：转换和自动化

由于使用marzipanify需要额外的步骤，我的建议是自动化转换过程。我没有尝试将自动化脚本添加为Xcode构建设置，因为将其作为一个小脚本运行似乎已经足够了。 LLDB准备就绪后，键入run：

```shell
#!/bin/bash
rm -rf PDFViewer.app
cp -r /Users/steipete/Builds/PDFViewer-.../Build/Products/Debug-iphonesimulator/PDFViewer.app .
./marzipanify PDFViewer.app
rm Entitlements*
lldb PDFViewer.app
```



### 步骤四：白名单中的Swift

这个花了我很长时间才明白。系统抱怨Swift支持库在那里但不是为iOSMac构建的。所以我只是从/ System / Library / PrivateFrameworks / Swift手动复制它们。然而，一旦我这样做，几乎不起作用，并且出现不断崩溃的情况与奇怪的过度释放问题。我在这里使用Swift 4.2，但/ System / Library / PrivateFrameworks / Swift中的Swift支持库是旧版本，并且在最近的某个时候，[调用约定已被更改](https://www.jessesquires.com/blog/swifts-new-calling-convention/)。因此，虽然对Objective-C桥接类的基本调用仍然有效，但是一旦我执行了更复杂的调用，就会调用Bundle.main.bundleURL：

```shell
: Library not loaded: @rpath/libswiftAVFoundation.dylib
  Referenced from: /marzipanify/PDFViewerMac.app/Contents/MacOS/PDFViewerMac

  Reason: no suitable image found.  Did find:
  /marzipanify/PDFViewerMac.app/Contents/MacOS/../Frameworks/libswiftAVFoundation.dylib: mach-o, but not built for iOSMac

Process 78797 stopped
* thread #1, stop reason = signal SIGABRT
    frame #0: 0x000000010523a162 dyld`__abort_with_payload + 10
dyld`__abort_with_payload:
->  0x10523a162 <+10>: jae    0x10523a16c               ; <+20>
Target 0: (PDFViewerMac) stopped.
```



编辑 /System/iOSSupport/dyld/macOS-whitelist.txt` and append `/Applications/PDFViewerMac.app/Contents/MacOS （替换调你的binary的地址）。

如果任何依赖项将AppKit加载到您的进程中，iOSMac将拒绝加载，除非您通过LLDB运行它。但是，有一个很好的理由将AppKit列入黑名单：它修改/与许多UIKit内部冲突，你的应用程序可能会在启动时崩溃或只是不渲染任何字体。 （如果你在 [UILabel ns_widgetType] 上遇到崩溃，你会知道原因。）

```shell
*** Terminating app due to uncaught exception 'NSInternalInconsistencyException', 
    reason: 'AppKit is getting loaded into a disallowed context'

*** First throw call stack:
(
  0   CoreFoundation                      0x00007fff4b49343d __exceptionPreprocess + 256
  1   libobjc.A.dylib                     0x00007fff772e1720 objc_exception_throw + 48
  2   CoreFoundation                      0x00007fff4b4ae08e +[NSException raise:format:arguments:] + 98
  3   Foundation                          0x00007fff4d82955d -[NSAssertionHandler 
                                          handleFailureInMethod:object:file:lineNumber:description:] + 194
  4   AppKit                              0x00007fff489175cb +[NSApplication load] + 672
```

在我们的例子中，Photos.framework导致AppKit被加载，我找到的唯一修复是删除照片及其在应用程序中的任何引用。 （我们使用它来为图像注释选择图像。但是，这是一个次要功能，没有它，应用将运行良好。）

```shell
otool -L /System/Library/Frameworks/Photos.framework/Photos

/System/Library/Frameworks/Photos.framework/Photos:
  /System/Library/Frameworks/Photos.framework/Versions/A/Photos
  /System/Library/Frameworks/AVFoundation.framework/Versions/A/AVFoundation
  /System/Library/Frameworks/Foundation.framework/Versions/C/Foundation
  /System/Library/Frameworks/AppKit.framework/Versions/C/AppKit
  (output shortened)
```

以下是PDF Viewer在Mac上的展示。文件选择器概念不是我们将在macOS上提供的东西，但是对于黑客来说它是可以的，并且它确实很好用，包括添加新文件时的目录观察器：



<video src="https://pspdfkit.com/images/blog/2018/marzipan-iosmac/PDFViewer-iOSMac-f9005ebc.mp4" width="100%" autoplay="" playsinline="" muted=""></video>



## 做个更好的Mac公民

看看工具栏，我们在这里做得还不够。要成为一名优秀的Mac公民意味着我们使用原生工具栏，就像Apple的Home应用。为了了解这是如何完成的，我通常会使用IDA或Hopper进行深度挖掘 - 两者都非常适合在Apple的应用程序中进行拆解和窥视。通过在Hopper中打开Home应用程序并搜索“工具栏”，可以轻松找到负责的类：

```swift
class iOSMacToolbarController {
    init() {
        print("iOSMac Marzipan extensions initializing.")

        guard (NSClassFromString("_UIWindowToolbarButtonItem") != nil) else {
            return }

        let titleButton = _UIWindowToolbarButtonItem(identifier: "com.pspdfkit.viewer.test")
        titleButton?.title = "A button"

        guard let toolbarController = UIApplication.shared.keyWindow!.
              value(forKey: "_windowToolbarController") as? _UIWindowToolbarController else {
            return
        }

        toolbarController.itemIdentifiers = ["com.pspdfkit.viewer.test"]
        toolbarController.templateItems = [titleButton]

        print("iOSMac extension initialized.")
    }
}
```

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/UIKitCore-linkerError-093c79e4.png)



好的，所以这并不奇怪。链接器不喜欢我们在这里声明类但不提供实现，这显然不是iOS UIKit的一部分。这个花了我一段时间，但当然每个问题都有解决方案。许多年前，Apple为类添加了弱连接，整个框架可能很弱。现在，我可以将它提取到一个框架中并完成它，但我们希望在这里快速得到一些东西，所以我们只是每个类的弱链接。 这指示编译器弱链接指定的类。现在它链接，所以让我们运行吧！

![img](https://pspdfkit.com/images/blog/2018/marzipan-iosmac/weak-link-class-96970ac8.png)



## 到这里不容易啊

运行时似乎并不高兴这里没有symbol。我们告诉链接器可能没有可用的东西，但是我们还没有告诉运行时。我们来解决这个问题！ 这指示编译器弱链接指定的类。现在它链接，所以让我们运行吧！

```shell
dyld: Symbol not found: _OBJC_CLASS_$__UIWindowToolbarButtonItem

  Referenced from: /Users/steipete/Library/Developer/CoreSimulator/Devices/43869E4F-C148-4D80-9E85-82466FAA8FED/data/Containers/Bundle/Application/6AFDFACD-0CD8-46FA-9F91-75B67B7A78F9/ViewerMac.app/ViewerMac

  Expected in: flat namespace
```



## 弱链接头

使用weak-import属性，我们可以告诉运行时方法可能不可用，在这种情况下，它们被解析为nil：

```shell
__attribute__((weak_import)) @interface _UIWindowToolbarItem : NSObject
```

看起来是这样的:



<video src="https://pspdfkit.com/images/blog/2018/marzipan-iosmac/toolbar-button-action-2dcdf1c9.mp4" width="100%" autoplay="" playsinline="" muted=""></video>



## 福利：查看视图层级

虽然我没有办法触发Xcode的视图调试器，但[Vlas Voloshin](https://twitter.com/argentumko)的这次[演讲](https://www.youtube.com/watch?v=EpUnke2yDug)提到了Reveal的工作原理。虽然后来某些测试版中对Mojave的更改阻止了加载Reveal服务器，但Itty Bitty Apps的优秀团队更加努力，并使他们的v18版本与iOSMac平台兼容。调试器控制台中的简单显示 **load -a** 将起到作用。



##  总结

将iOS应用程序移植到Mac是多么令人兴奋的事，而且使用🍎的新iOSMac平台比以往更容易。只需几个技巧，就可以在不到一天的时间内移植整个大型复杂的应用程序。如何为您的下一个'实验周五'或Hackathon项目选择此项目？在Twitter上ping [@steipete](https://twitter.com/steipete)并向我们展示你的结果 - 让[Marzipandemic](https://twitter.com/rgriff/status/1004405013462933504)开始吧！