qiniu-js-sdk
============

基于七牛API及Plupload开发的前端JavaScript SDK

示例网站：[七牛JavaScript SDK 示例网站 - 配合七牛nodejs SDK ](http://upqiniu.duapp.com/)
## 概述

本SDK适用于IE8+、chrome、Firefox 等浏览器，基于 七牛云存储官方API 构建，其中上传功能基于 Plupload 插件封装。开发者使用本 SDK 可以方便的从浏览器端上传文件至七牛云存储，并对上传成功后的图片进行丰富的数据处理操作。本 SDK 可使开发者忽略上传底层实现细节，而更多的关注 UI 层的展现。

### 功能简介

* 上传
 * html5模式大于4M时可分块上传，小于4M时直传
 * Flash、html4模式直接上传
 * 继承了Plupload的功能，可筛选文件上传、拖曳上传等
* 下载(公开资源)
* 数据处理（图片）
 * imageView2（缩略图）
 * imageMogr2（高级处理，包含缩放、裁剪、旋转等）
 * imageInfo （获取基本信息）
 * exif      （获取图片EXIF信息）
 * watermark （文字、图片水印）
 * pipeline  （管道，可对imageView2、imageMogr2、watermark进行链式处理）

### SDK构成介绍
* Plupload
* qiniu.js，SDK主体文件，上传功能\数据处理实现


## 安装和运行程序
* 服务端准备

    本SDK依赖服务端颁发upToken，可以通过以下二种方式实现：
    *   利用[七牛服务端SDK](http://developer.qiniu.com/docs/v6/sdk/)构建后端服务
    *   利用七牛底层API构建服务，详见七牛[上传策略](http://developer.qiniu.com/docs/v6/api/reference/security/put-policy.html)和[上传凭证](http://developer.qiniu.com/docs/v6/api/reference/security/upload-token.html)


    后端服务应提供一个URL地址，供SDK初始化使用，前端通过Ajax请求该地址后获得upToken。Ajax请求成功后，服务端返回应以下类似的json：

    ```
        {
            "uptoken": "0MLvWPnyya1WtPnXFy9KLyGHyFPNdZceomL..."
        }
    ```
* 引入Plupload

    *   [Plupload下载](http://plupload.com/download)

    *   引入`plupload.full.min.js`（产品环境）或 引入`plupload.dev.js`和`moxie.js`（开发调试）

* 引入qiu.js，初始化SDK
    *   获取SDK源码 `git clone https://github.com/SunLn/qiniu-js-sdk.git`，`qiu.js`位于`src`目录内

    *   初始化SDK

    ```

        var Q = new Qiniu({
            runtimes: 'html5,flash,html4',    //上传模式,依次退化
            browse_button: 'pickfiles',       //上传选择的点选按钮，**必需**
            uptoken_url: '/token',            //Ajax请求upToken的Url，**必需**（服务端提供）
            domain: 'http://qiniu-plupload.qiniudn.com/',   //bucket 域名，下载资源时用到，**必需**
            container: 'container',           //上传区域DOM ID，默认是browser_button的父元素，
            max_file_size: '100mb',           //最大文件体积限制
            flash_swf_url: 'js/plupload/Moxie.swf',  //引入flash,相对路径
            max_retries: 3,                   //上传失败最大重试次数
            dragdrop: true,                   //开启可拖曳上传
            drop_element: 'container',        //拖曳上传区域元素的ID，拖曳文件或文件夹后可触发上传
            chunk_size: '4mb',                //分块上传时，每片的体积
            auto_start: true,                 //选择文件后自动上传，若关闭需要自己绑定事件触发上传
            init: {
                'FilesAdded': function(up, files) {
                    plupload.each(files, function(file) {
                        //文件添加进队列后,处理相关的事情
                    });
                },
                'BeforeUpload': function(up, file) {
                       //每个文件上传前,处理相关的事情
                },
                'UploadProgress': function(up, file) {
                       //每个文件上传时,处理相关的事情
                },
                'FileUploaded': function(up, file, info) {
                       //每个文件上传成功后,处理相关的事情
                       //其中 info 是文件上传成功后，服务端返回的json，形式如
                       // {
                       //    "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
                       //    "key": "gogopher.jpg"
                       //  }
                       // 参考http://developer.qiniu.com/docs/v6/api/overview/up/response/simple-response.html

                       // var domain = up.getOption('domain');
                       // var res = parseJSON(info);
                       // var sourceLink = domain + res.key; 获取上传成功后的文件的Url
                },
                'Error': function(up, err, errTip) {
                       //上传出错时,处理相关的事情
                },
                'UploadComplete': function() {
                       //队列文件处理完毕后,处理相关的事情
                }
            }
        });

        // domain 为七牛空间（bucket)对应的域名，选择某个空间后，可通过"空间设置->基本设置->域名设置"查看获取
    ```

* 运行网站，通过点击`pickfiles`元素，选择文件后上传

* 对上传成功的图片进行数据处理

    *  watermark（水印）

    ```

        // Q 对象为初始化SDK时声明的对象，下同
        // key 为每个文件上传成功后，服务端返回的json字段，即资源的最终名字，下同
        // key 可在每个文件'FileUploaded'事件被触发时获得

        var imgLink = Q.watermark({
             mode: 1,  // 图片水印
             image: 'http://www.b1.qiniudn.com/images/logo-2.png', // 图片水印的Url，mode = 1 时 **必需**
             dissolve: 50,          // 透明度，取值范围1-100，非必需，下同
             gravity: 'SouthWest',  // 水印位置，为以下参数[NorthWest、North、NorthEast、West、Center、East、SouthWest、South、SouthEast]之一
             dx: 100,  // 横轴边距，单位:像素(px)
             dy: 100   // 纵轴边距，单位:像素(px)
         }, key);

         // imgLink 可以赋值给 html 的 img 元素的 src 属性，下同

    ```

    或

    ```

        var imgLink = Q.watermark({
             mode: 2,  // 文字水印
             text: 'hello world !', // 水印文字，mode = 2 时 **必需**
             dissolve: 50,          // 透明度，取值范围1-100，非必需，下同
             gravity: 'SouthWest',  // 水印位置，同上
             fontsize: 500,         // 字体大小，单位: 缇
             font : '黑体',          // 水印文字字体
             dx: 100,  // 横轴边距，单位:像素(px)
             dy: 100,  // 纵轴边距，单位:像素(px)
             fill: '#FFF000'        // 水印文字颜色，RGB格式，可以是颜色名称
         }, key);

    ```
    具体水印参数解释见[水印（watermark）](http://developer.qiniu.com/docs/v6/api/reference/fop/image/watermark.html)
    *  imageView2

    ```

        var imgLink = Q.imageView2({
           mode: 3,  // 缩略模式，共6种[0-5]
           w: 100,   // 具体含义由缩略模式决定
           h: 100,   // 具体含义由缩略模式决定
           q: 100,   // 新图的图像质量，取值范围：1-100
           format: 'png'  // 新图的输出格式，取值范围：jpg，gif，png，webp等
         }, key);

    ```
    具体缩略参数解释见[图片处理（imageView2）](http://developer.qiniu.com/docs/v6/api/reference/fop/image/imageview2.html)
    *  imageMogr2

    ```

        var imgLink = Q.imageMogr2({
           auto-orient: true,  // 布尔值，是否根据原图EXIF信息自动旋正，便于后续处理，建议放在首位。
           strip: true,   // 布尔值，是否去除图片中的元信息
           thumbnail: '1000x1000'   // 缩放操作参数
           crop: '!300x400a10a10',  // 裁剪操作参数
           gravity: 'NorthWest',    // 裁剪锚点参数
           quality: 40,  // 图片质量，取值范围1-100
           rotate: 20,   // 旋转角度，取值范围1-360，缺省为不旋转。
           format: 'png',// 新图的输出格式，取值范围：jpg，gif，png，webp等
           blur:'3x5'    // 高斯模糊参数
         }, key);

    ```

    具体高级图像处理参数解释见[高级图像处理（imageMogr2）](http://developer.qiniu.com/docs/v6/api/reference/fop/image/imagemogr2.html)
    *  imageInfo

    ```
        var imageInfoObj = Q.imageInfo(key);
    ```
    具体 imageInfo 解释见[图片基本信息（imageInfo）](http://developer.qiniu.com/docs/v6/api/reference/fop/image/imageinfo.html)
    *  exif

    ```
        var exifOjb = Q.exif(key);
    ```

    具体 exif 解释见[图片EXIF信息（exif）](http://developer.qiniu.com/docs/v6/api/reference/fop/image/exif.html)
    *  pipeline(管道操作）

    ```

        var fopArr = [{
            fop: 'watermark', // 指定watermark操作
            mode: 2, // 此参数同watermark函数的参数，下同。
            text: 'hello world !',
            dissolve: 50,
            gravity: 'SouthWest',
            fontsize: 500,
            font : '黑体',
            dx: 100,
            dy: 100,
            fill: '#FFF000'
        },{
            fop: 'imageView2',  // 指定imageView2操作
            mode: 3,  // 此参数同imageView2函数的参数，下同
            w: 100,
            h: 100,
            q: 100,
            format: 'png'
        }，{
            fop: 'imageMogr2',  // 指定imageMogr2操作
            auto-orient: true,  // 此参数同imageMogr2函数的参数，下同。
            strip: true,
            thumbnail: '1000x1000'
            crop: '!300x400a10a10',
            gravity: 'NorthWest',
            quality: 40,
            rotate: 20,
            format: 'png',
            blur:'3x5'
        }];
        // fopArr 可以为三种类型'watermark'、'imageMogr2'、'imageView2'中的任意1-3个
        // 例如只对'watermark'、'imageMogr2'进行管道操作，则如下即可
        // var fopArr = [{
        //    fop: 'watermark', // 指定watermark操作
        //    mode: 2, // 此参数同watermark函数的参数，下同。
        //    text: 'hello world !',
        //    dissolve: 50,
        //     gravity: 'SouthWest',
        //     fontsize: 500,
        //     font : '黑体',
        //     dx: 100,
        //     dy: 100,
        //     fill: '#FFF000'
        // },{
        //    fop: 'imageMogr2',  // 指定imageMogr2操作
        //    auto-orient: true,  // 此参数同imageMogr2函数的参数，下同。
        //    strip: true,
        //    thumbnail: '1000x1000'
        //    crop: '!300x400a10a10',
        //    gravity: 'NorthWest',
        //    quality: 40,
        //    rotate: 20,
        //    format: 'png',
        //    blur:'3x5'
        // }];


        var imgLink = Q.pipeline(fopArr, key));

    ```
    具体管道操作解释见[管道操作](http://developer.qiniu.com/docs/v6/api/overview/fop/pipeline.html)

## 运行示例

直接运行本SDK示例网站的服务

*  [安装nodejs](http://nodejs.org/download/)

*  获取源代码：
    `git clone https://github.com/SunLn/qiniu-js-sdk.git`
*  进入`example`目录,修改`server.js`，`Access Key`和`Secret Key` 按如下方式获取

    * [开通七牛开发者帐号](https://portal.qiniu.com/signup)
    * [登录七牛开发者自助平台，查看 AccessKey 和 SecretKey](https://portal.qiniu.com/setting/key) 。

        ```

            qiniu.conf.ACCESS_KEY = '<Your Access Key>';

            qiniu.conf.SECRET_KEY = '<Your Secret Key>';

            var uptoken = new qiniu.rs.PutPolicy('<Your Bucket Name>');

        ```

*  在`example`目录运行`node server.js` 或者 在根目录运行`make`启动
*  访问`http://127.0.0.1:18080/`或`http://localhost:18080/`

## 说明

1. 本SDK依赖Plupload，初始化之前请引入Plupload插件

2. 本SDK依赖服务端颁发uptoken，必须构建后端服务

3. 如果您想了解更多七牛的上传策略，建议您仔细阅读 [七牛官方文档-上传](http://developer.qiniu.com/docs/v6/api/reference/up/)

4. 如果您想了解更多七牛的图片处理，建议您仔细阅读 [七牛官方文档-图片处理](http://developer.qiniu.com/docs/v6/api/reference/fop/image/)

5. 本SDK示例生成upToken时，指定的`Bucket Name`为公开空间，所以可以公开访问上传成功后的资源。若您生成upToken时，指定的`Bucket Name`为私有空间，那您还需要在服务端进行额外的处理才能访问您上传的资源。具体参见[下载凭证](http://developer.qiniu.com/docs/v6/api/reference/security/download-token.html)。本SDK数据处理部分功能不适用于私有空间。

## 贡献代码

1. 登录 https://github.com

2. Fork https://github.com/SunLn/qiniu-js-sdk.git

3. 创建您的特性分支 (git checkout -b new-feature)

4. 提交您的改动 (git commit -am 'Added some features or fixed a bug')

5. 将您的改动记录提交到远程 git 仓库 (git push origin new-feature)

6. 然后到 github 网站的该 git 远程仓库的 new-feature 分支下发起 Pull Request


## 许可证

> Copyright (c) 2014 qiniu.com

## 基于 MIT 协议发布:

> [www.opensource.org/licenses/MIT](http://www.opensource.org/licenses/MIT)
