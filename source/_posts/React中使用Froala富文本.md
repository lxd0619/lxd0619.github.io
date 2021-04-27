---
title: React中使用Froala富文本
comments: true
date: 2021-04-21 10:24:40
tags: 
    - 富文本
categories:
    - front_end
---

Froala富文本有不同语言的各种版本，此为React版。

<!-- more -->

#### 安装

+ npm	   `npm install react-froala-wysiwyg --save`
+ or yarn   `yarn add react-froala-wysiwyg`

#### 使用

```react
// 引入 CSS 文件.
import 'froala-editor/css/froala_style.min.css';
import 'froala-editor/css/froala_editor.pkgd.min.css';

// 如需要图片视频上传功能请引入此插件
import "froala-editor/js/plugins/image.min.js";
import "froala-editor/js/plugins/video.min.js";

import FroalaEditorComponent from 'react-froala-wysiwyg';

            <FroalaEditorComponent
                ref="froalaEditor"
                tag="textarea"
                config={{
                  placeholderText: "请输入!",
                  charCounterCount: false,
                  height: 300,			//编辑器高度
                  language: "zh_cn",
                  toolbarButtons: {},	//工具栏按钮
                  //图片上传按钮
                  imageInsertButtons: [
                    "imageUpload",	//本地上传
                    "imageByURL"	//URL上传
                  ],
                  //视频上传按钮
                  videoInsertButtons: [
                    "videoByURL",	//URL上传
                    "videoEmbed",	//嵌入代码
                    "videoUpload"	//本地上传
                  ],
                  // 图片上传地址，视频类似
                  imageUploadURL: uploadURL,
                  // 图片上传附带参数，视频类似
                  imageUploadParams: {parmars},
				          //请求头
                  requestHeaders: { 
                    utoken: Storage.get("token")
                  },
                  events: {
                  	//初始化
                    initialized: function() {
                      var editor = this;
                      console.log("富文本初始化", editor);
                      //初始化禁用富文本，如果在低版本此方法可能不生效
                      //editor.edit.off();
                    },
                    //这是图片的相关方案，视频的类似
                    "image.beforeUpload": async images => {
                      // 可以在此做格式校验等
                      // 如果想取消上传请返回 return false;
                      console.log("image.beforeUpload", images);
                    },
                    // 图片被上传到服务器
                    "image.uploaded": function(response) {
                      console.log("image.uploaded", response);
                    },
                    // 图片被插入到编辑器.
                    "image.inserted": function(img, response) {
                      console.log("image.inserted", img, response);
                    },
                    // 图像在编辑器中被替换.
                    "image.replaced": function(img, response) {
                      console.log("image.replaced", img, response);
                    },
                    //上传报错
                    "image.error": function(error, response) {
                      console.log("image.error", error, response);
                    },
                    ...
                  }
                }}
                model={materialHtml} //相当于value
                //相当于onChange
                onModelChange={value => console.log("富文本内容", value)} 
              />
```

可以通过refs设置富文本的启用/禁用

```js
this.refs.froalaEditor.editor.edit.on();	//启用
this.refs.froalaEditor.editor.edit.off(); 	//禁用
```

{% label @注：后台保存图片需要返回给前台图片的路径，而且格式必须是这样的，否则会报错 %}

```
{link:"http://xxxxx.jpg"}
```

##### 版权问题

因为编辑器有版权问题，会在编辑器内出现一行红色背景的字，提醒你版权问题，影响用户体验，可以通过样式去除，（仅限个人学习使用，如需商用请购买版权）。

```less
:global(.fr-wrapper > div:first-child) {
  position: absolute;
  top: -10000px;
  opacity: 0;
}
:global(.fr-element.fr-view) {
  position: absolute;
  top: 0;
}
:global(.fr-placeholder) {
  margin-top: 0 !important;
}
:global(.fr-second-toolbar) {
  display: none;
}

```

此为最终效果
![image-20210421170138414](/hexo_blog/images/image-20210421170138414.png)

#### 参考

详细API请参考官方说明文档[froala官方文档](https://froala.com/wysiwyg-editor/docs/)。

其他参考：

+ [H5中使用froala](https://blog.csdn.net/lianzhang861/article/details/83590084)
+ [npmjs——react-froala-wysiwyg](https://www.npmjs.com/package/react-froala-wysiwyg)