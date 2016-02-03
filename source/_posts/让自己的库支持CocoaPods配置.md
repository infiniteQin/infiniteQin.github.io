---
title: 让自己的库支持CocoaPods配置
date: 2016-02-03 17:22:17
tags: cocoapods pods
---

#前情提要
公司项目一直在用CocoaPods管理第三方包，所以就写了个开源图片选择库（仿微信），随带学学CocoaPods私有库的配置以及如何让自己的开源库支持了pod install。

#准备工作

github创建私有库 如testSpecs.git

pod repo add testSpecs https://github.com/qgg/testSpecs.git

github创建工具库 如QGGImagePicker 注意创建的时候勾选开源协议
命令行创建模版工程 
pod lib create QGGImagePicker

根据提示创建完成

```
XXXXXX$ pod lib create QGGImagePicker
Cloning `https://github.com/CocoaPods/pod-template.git` into `QGGImagePicker`.
Configuring QGGImagePicker template.
------------------------------
To get you started we need to ask a few questions, this should only take a minute.
If this is your first time we recommend running through with the guide: 
 - http://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and double click links to open in a browser. )
What language do you want to use?? [ ObjC / Swift ]
 > ObjC
Would you like to include a demo application with your library? [ Yes / No ]
 > Yes
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None
Would you like to do view based testing? [ Yes / No ]
 > No
What is your class prefix?
 > QGG
```

完成后打开工程目录如下:工具库主要在红框内容Classes目录下进行开发
![1.png](http://upload-images.jianshu.io/upload_images/233856-b2f0519f1bf92e33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)
修改工程下的.podspec文件，如

```

# Be sure to run `pod lib lint QGGImagePicker.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = "QGGImagePicker"
  s.version          = "0.0.1"
  s.summary          = "QGGImagePicker."

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!  
  s.description      = <<-DESC
                        A ImagePicker Like WeChat's ImagePicker
                       DESC

  s.homepage         = "https://github.com/infiniteQin/QGGImagePicker.git"
  # s.screenshots     = "www.example.com/screenshots_1", "www.example.com/screenshots_2"
  s.license          = 'MIT'
  s.author           = { "changqin" => "changqin@ixiaopu.com" }
  s.source           = { :git => "https://github.com/infiniteQin/QGGImagePicker.git", :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.platform     = :ios, '7.0'
  s.requires_arc = true

  s.source_files = 'Pod/Classes/**/*','Pod/Classes/**/**/*'
  #s.resource_bundles = {
  #  'QGGImagePicker' => ['Pod/Assets/*.png']
  #}
  #s.resources = "Pod/*.xcassets"

  # s.public_header_files = 'Pod/Classes/**/*.h'
  s.frameworks = "UIKit", "AssetsLibrary"
  s.dependency 'Masonry', '~> 0.6.3'
end
```

#本地验证

```
pod lib lint 
```
验证成功后推送工程到github给如QGGImagePicker工具库打tag（和podspec中的版本保持一致）
#验证远程库

```
pod spec lint
```

看到输出如下内容就成功了

QGGImagePicker.podspec passed validation.

#私用库中添加工具库

```
pod repo push testSpecs QGGImgePicker.podspec
```

#使用

pod search QGGImagePicker
Podfile文件添加 pod 'QGGImagePicker', '~> 0.0.1'
pod update --verbose --no-repo-update
注意这时候会报错，解决办法
pod spec lint --sources=‘https://github.com/qgg/testSpecs.git,https://github.com/CocoaPods/Specs'
或者
将~/.cocoapods/repos/testSpecs/下的内容copy到~/.cocoapods/repos/master/Specs下

#支持****CocoaPods****公开库

### 方法一
到https://github.com/CocoaPods/Specs.git 下fork一份并clone到本地
将私有库中的.podspec文件转成.json格式添加到工程中， 再到github上的fork工程中发起 [New pull request]

```
$ pod ipc spec QGGImagePicker.podspec >> QGGImagePicker.podspec.json
```
将内容（目录结构如下）添加到CocoaPods/Specs提交
待作者审核后即可
![2.png](http://upload-images.jianshu.io/upload_images/233856-03a1c1eba1c07447.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

### 方法二 （推荐）
注册trunk

```
pod trunk regist qgg@163.com 'qgg' --verbose
```
去邮箱验证trunk注册，用下面的命令查询自己的trunk注册信息

```
pod trunk me
```
将.podspec推送到CocoaPods的repo

```
pod trunk push QGGImagePicker.podspec
```
这个过程会验证podspec文件是否合法－－上传podspec文件到turnk服务器（省去了fork和pull request的操作）－－将podspec文件转化成json格式的文件。


#最后
分享几篇我参考的文章
https://cocoapods.org/
http://www.cocoachina.com/ios/20150508/11785.html
http://www.cocoachina.com/ios/20150228/11206.html


