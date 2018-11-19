# l-imgupload

> 一个（简陋的）图片上传组件，实现了图片随意截取

## 引用库
[Vue.js][1]

## 安装步骤

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```
## 参数及方法
```
参数名：inputWidth
说明：组件宽度
类型：Number
默认值：200px

参数名：inputHeight
说明：组件高度
类型：Number
默认值：200px

参数名：cuttingRatio
说明：裁剪比例，限定比例为宽/高，为空时没有比例限制
类型：Number
默认值：0

事件名：getImageData
说明：框选完成后鼠标抬起时触发，返回选定区域的图像数据
参数：blobData
参数格式：Blob对象

事件名：getImageDataURL
说明：鼠标拖动的每一帧触发，返回选定区域的图像数据，可用于预览区域展示
参数：dataURL
参数格式：dataURL
```
## 使用截图
最终效果搞成了这个样子
![使用截图][2]
##实现过程
记录了一些（无聊的）思路和开发流程

[实践是检验程序员的唯一标准01：用户不想跟你说话并向你扔出一张图片 - 图片上传组件开发【思路篇】][3]

[实践是检验程序员的唯一标准02：用户不想跟你说话并向你扔出一张图片 - 图片上传组件开发【开发篇】][4]


## 待完成功能
###181119
* 截取功能的开关 - 我万一就想上传整张图呢
* 标记功能 - 我想标个终点
* 打码功能 - 别想歪了，毕竟金融行业怕泄漏客户信息！

## 写在后面
这是我在github留下的第1个印记，感谢(dear)[包包][5]的鼓励和督促！

[1]:https://github.com/vuejs/vue
[2]:https://segmentfault.com/img/bVbip2p
[3]:https://segmentfault.com/a/1190000013038300?_ea=4843335
[4]:https://segmentfault.com/a/1190000016743925?_ea=4855622
[5]:https://github.com/iris-official
