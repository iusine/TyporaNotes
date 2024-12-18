分享之前分别感谢：

[**@Kuingsmile**](https://github.com/Kuingsmile)大佬的开源项目[**PicList**](https://github.com/Kuingsmile/PicList/)图像上传。

[**@GitHub**](https://github.com/)提供图像存储。

[**@Cloudflare**](https://www.jsdelivr.com/) 提供的免费开源公共 CDN JSDelivr。



### 1、下载PicList。

#### Windows

- [https://release.piclist.cn/latest/PicList-Setup-2.9.3-ia32.exe](https://release.piclist.cn/latest/PicList-Setup-2.9.3-ia32.exe)

- [https://release.piclist.cn/latest/PicList-Setup-2.9.3-x64.exe](https://release.piclist.cn/latest/PicList-Setup-2.9.3-x64.exe)

- [https://release.piclist.cn/latest/PicList-Setup-2.9.3.exe](https://release.piclist.cn/latest/PicList-Setup-2.9.3.exe)

#### Mac

- [https://release.piclist.cn/latest/PicList-2.9.3-arm64.dmg](https://release.piclist.cn/latest/PicList-2.9.3-arm64.dmg)

- [https://release.piclist.cn/latest/PicList-2.9.3-x64.dmg](https://release.piclist.cn/latest/PicList-2.9.3-x64.dmg)

- [https://release.piclist.cn/latest/PicList-2.9.3-universal.dmg](https://release.piclist.cn/latest/PicList-2.9.3-universal.dmg)

### 2、配置GitHub设置

**一、在设置显示github作为图床。**

![3-1.jpg](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-1.jpg)

**二、配置比对和说明**

创建github仓库获取Token请移步[教程 - Picture Hosting (piclist.vimio.top)](http://piclist.vimio.top/#/pictureHosting/help)

<img src="https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-2.png" alt="3-2.png"  />

![3-3.png](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-3.png)

**三、图片上传**

![3-4.png](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-4.png)

![3-5.png](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-5.png)

**四、图片检验**

如果加速源无法使用可以更改加速源，将github的设定自定义域名处的<font  color="#00c3ff"><u>cdn.jsdmirror.com</u></font >改为其他加速源

<font  color="#00c3ff"><u>jsd.onmicrosoft.cn</u></font >

<font  color="#00c3ff"><u>gcore.jsdelivr.net</u></font >

<font  color="#00c3ff"><u>testingcf.jsdelivr.net</u></font >

<img src="https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-7.png" style="zoom: 80%;" />

**五、Typora便捷操作**

打开Typora，点击左上角的->偏好设置点击图像，找到<font  color="#00c3ff"><u>上传服务设定</u></font >选中PiGo(Piclist是基于PiGo开发的，所以Piclist是完全兼容PiGo的)，在Piclist的包文件路径找到Piclist.exe选中,插入图片时选着<font  color="#00c3ff"><u>图片上传</u></font >不勾选本地位置，将网络位置勾选，实现复制图片到Typora图片自动上传到Piclist。

![](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-8.png)

### 3、Picture Hosting + GitHub + JSDelivr

在此追加感谢：

[**@Naccl**](https://github.com/Naccl)大佬的开源项目[**PictureHosting**](https://github.com/Naccl/PictureHosting)图像上传

1、PictureHosting项目部署：

虚拟主机/服务器 部署都可以。

加速源不可用时，查询保存过的图片，可以一键更改保存过的CDN加速源。

[**项目开源地址https://github.com/vi-mio/PictureHosting**](https://github.com/vi-mio/PictureHosting)

[**教程及效果：https://piclist.vimio.top**](https://piclist.vimio.top)

![](https://jsd.onmicrosoft.cn/gh/vi-mio/resources/BlogImage/3-10.png)