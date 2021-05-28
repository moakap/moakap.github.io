---
title: Expo中的Assets资源使用
date: 2021-05-24 11:52:51
tags:
  - expo
  - assets
---


图片、字体、视频、音频和其它你的App里用到的不是JavaScript的文件都被认为是资源(asset). 就比如说在在互联网上， 所有你通过http传输获取的东西叫资源。这一点跟移动app不同，在app里资源一般跟app本身的二进制文件打包在一起。

然而，在Expo中通过`require`引用的资源（例如`<Image source={require('./assets/images/example.png')} />`在编译的时候已经在本地文件系统存在了）跟在web中通过URL引用的资源（例如`<Image source={{uri: 'http://yourwebsite.com/logo.png'}} />`）是有区别的。Expo不保证通过web URL引用的这些资源的可用性，因为这些资源不在Expo的管理范围。另外，Expo本身无法获取任意Web URL的全部信息。当你的资源存在于本地文件系统的时候，打包器能够读取资源的基本元数据metadata，比如宽度、高度，然后将这些信息传递给你的App，所以你不需要明确指明某个图片的宽度和高度。然而当你使用远程web URL引用资源的时候，你就必须明确得指明宽度、高度这些信息，否则系统会默认使用0x0。 最后，我们也会看到两者的缓存机制也不同。

接下来是对`require`类型资源得详细解释，这些资源必须在编译的时候就已经在文件系统存在。而在通过远程URL引用图片资源的例子中，系统默认你知道怎么将一个图片上传到网络，并确保能够被web和移动App访问。

## 资源的位置

### 开发环境

当你工作在项目的本地开发环境时，所有资源被存放在本地文件系统，并且集成到JavaScript的模块中。所以如果想要包含某个图片，你可以直接在javascript中使用`require`引用，像这样`require('./assets/images/example.png')`。这里唯一的不同是，你必须指定资源的文件后缀；没有后缀的话，系统会默认这是一个javascript文件。下边这个语句会在编译的时候把资源连同资源的元数据一起赋值给一个对象，然后整个对象传递给`Image`组件做渲染。

`<Image source={require('./assets/images/example.png')} />`

### 生产环境

每次你通过Expo发布App的时候，Expo都会把所有需要的资源上传到Amazon CloudFront，一个快速的CDN。而且整个上传过程会非常智能，没有变化的资源会被跳过。整个过程都由Expo自动完成，不需要开发者任何操作。



## 自定义支持的资源扩展

有时候你需要用到的文件类型不在Expo默认的支持范围。例如，如果你想要在App中导入数据库，就可能会用到.db文件。参考[自定义Metro](https://docs.expo.io/guides/customizing-metro/)了解如何通过Metro自定义iOS和Android资源扩展，参考[自定义Webpack](https://docs.expo.io/guides/customizing-webpack/)了解如何通过Webpack自定义Web资源扩展。



## 优化

### 图片

图片一般是Expo项目中占用空间最多的资源。优化图片可以减少用户设备上的空间占用，从而降低显示前的加载时间和使用带宽。可以通过执行`npx expo-optimize`命令来压缩PNG和JPEG图片。也可以设置下边的选项：

- `--save`: 备份所有图片（以`.orig`文件的形式保存）
- `--quality=N`: 设置文件压缩比率，N可以是1-100之间的整数（包含1和100，默认为60）
- `--include="[pattern]"`: 设置仅优化满足通配符模式的资源（默认使用app.json中的`assetBundlePatterns`字段）
- `--exclude="[pattern]"`: 排除满足通配符模式的资源

注意：所有通配符得匹配范围都是相对于项目的根目录，跟执行命令所在的目录无关。

### 字体

字体是启动App必不可缺的资源。web应用中的字体加载问题分为FOUT (Flash of Unstyled Text), FOIT (Flash of Invisible Text), FOFT (Flash of Faux Text)这几种，详细可以参考[这里](https://css-tricks.com/fout-foit-foft/)。

icon-font-powered的默认行为是第一次加载的时候使用FOIT，然后在后续的加载中字体会自动被缓存。相比较于传统Web，在用户对移动设备有更高的标准要求，所以你可能需要在初始屏幕加载的过程中预加载和缓存字体和一些重要的图片。

### 缓存

参考[这里]([Asset Caching - Expo Documentation](https://docs.expo.io/guides/preloading-and-caching-assets/))了解更多资源缓存的话题。







