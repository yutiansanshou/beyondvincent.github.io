---
layout: post
title: "iOS 7中的一些小修改"
date: 2013-09-20 11:45
comments: true
categories: iOS翻译
published: ture

---

{% img /images/2013/09/4.png %}


## **小引**

大家都知道iOS 7做了很大的调整，当然也有一些轻微的修改，我们来稍微看一下吧。

<!--more-->

注：本文译自[`iOS 7 Additions: OMG Finally!`](http://www.doubleencore.com/2013/09/ios-7-additions-omg-finally/)

###**目录**
* Message UI Framework(在消息中添加文件)
* Media Player Framework(MPVolumeView - 检测airplay和当前airplay的route)
* AVFoundation(条码扫描)
* 更多

###** Message UI Framework(在消息中添加文件)**

在iOS 7之前我们可以使用`MFMessageComposeViewController`来撰写文本消息，但是要想添加文件是不可能做到的，不过在iOS 7中我们可以使用这个方法就能添加文件了：`- (BOOL)addAttachmentData:(NSData *)attachmentData typeIdentifier:(NSString *)uti filename:(NSString *)filename;`。如下代码示例所示：

```
if ([MFMessageComposeViewController canSendText] && [MFMessageComposeViewController canSendAttachments] && [MFMessageComposeViewController isSupportedAttachmentUTI:(NSString *)kUTTypePNG]) {
    MFMessageComposeViewController *vc = [[MFMessageComposeViewController alloc] init];
    vc.messageComposeDelegate = self;
    vc.recipients = @[@"Yawkey"];
    UIImage *myImage = [UIImage imageNamed:@"Yawkey_business_dog.png"];
    BOOL attached = [vc addAttachmentData:UIImagePNGRepresentation(myImage) typeIdentifier:(NSString*)kUTTypePNG filename:@"Yawkey_business_dog.png"];
    if (attached) {
        NSLog(@"Attached (:");
    }
    else {
        NSLog(@"Not attached ):");
    }
    [self presentViewController:vc animated:YES completion:nil];
}
```
{% img /images/2013/09/5.png %}


###** Media Player Framework(MPVolumeView - 检测airplay和当前airplay的route)**

`MPVolumeView`可以帮助我们与AirPlay系统进行交互。不过，一直以来都是很难获得用户选择操作的信息。现在通过新增的两个属性和通知，我们可以更加深入的了解AirPlay系统了。
```
@property areWirelessRoutesAvailable;
@property isWirelessRouteActive;
```
这两个属性可以告诉我们是否有可用的AirPlay，以及是否以及被选中了。下面两个是通知：
```
NSString *const MPVolumeViewWirelessRoutesAvailableDidChangeNotification;
NSString *const MPVolumeViewWirelessRouteActiveDidChangeNotification;
```
通过这两个通知我们可以知道可用AirPlay发生了改变，以及用户修改了当前正在使用的AirPlay route。


###**AVFoundation(条码扫描)**

`AVFoundation`中现在已经内置支持一维和二维码的扫描。之前要想在iOS程序中读取条形码和QR码，则需要使用第三方库，例如ZXing和ZBar。在iOS 7中默认支持4中机器条码，需要做的就是将`AVCaptureMetadataOutput` hook up到`AVCaptureSession`。另外可以对`AVCaptureMetadataOutput`进行配置以检测如下这些任意机器可读的条码类型：
```
AVMetadataObjectTypeUPCECode
AVMetadataObjectTypeCode39Code
AVMetadataObjectTypeCode39Mod43Code
AVMetadataObjectTypeEAN13Code
AVMetadataObjectTypeEAN8Code
AVMetadataObjectTypeCode93Code
AVMetadataObjectTypeCode128Code
AVMetadataObjectTypePDF417Code
AVMetadataObjectTypeQRCode
AVMetadataObjectTypeAztecCode
```

当配置好`AVCaptureMetadataOutputObjectsDelegate`，就可以响应`- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection;`检测到的所有条码。


###**更多**

上面只是列出了少许新内容，你可以通过苹果提供的文档[What’s New in iOS](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS7.html) 查阅更多相关内容。

注：本文是iOS 7开发者指南中的11篇中的第1篇。你可以在[这里](http://www.doubleencore.com/2013/09/essential-ios-7-developers-guide)看到指南的全部内容。

