[TOC]

## 业务需求

**移动端**用户查看/**预览**在**其他服务器**上的**PDF格式**的业务协议，该协议有可能**更新变动**。

## 先说结论

方案大都来源于网络，咱们肯定也都有尝试过，我只是做了一个搬运、试错、筛选的工作。个人建议是使用pdf插件，然后解决那个跨域问题。

## 解决方案

### 一、直接跳转或使用iframe

遇到的问题：PC和IOS正常，Android会直接进下载页面，需要下载后才能打开。

这样的话依赖用户操作，体验不好，貌似[Android系统是不支持阅读PDF](<https://www.jianshu.com/p/33d454933a98>)，需要使用插件。

### 二、使用pdf.js插件

这里遇到**跨域问题**，目前可以尝试的方案：

1. 允许跨域。服务端配置CORS允许跨域，并修改/注释掉viewer.js中的跨域错误提示。

   ```js
   // 在目标服务器header头部中设置允许跨域访问
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Methods: GET, POST, PUT, OPTIONS
   Access-Control-Expose-Headers: Accept-Ranges, Content-Encoding, Content-Length, Content-Range
   ```

   缺点：可行性取决于目标服务器，这样配置不一定被允许

2. 消除跨域。把pdf放在项目中，通过定时任务，定时更新资源。
   缺点：时效性和占用服务器资源

3. 使用代理。不清楚咱们目前web服务是怎么部署的，如果是nginx那直接：

   ```js
   // 示例配置，具体字段以实际路径为准
   location /files {
     proxy_pass http://download.zybank.com.cn/;
   }
   ```

4. 文件流。需要后端接口配合，未能测试。
   参考资料<https://www.cnblogs.com/linxingyun/p/9293455.html>

### ~~三、pdf转html或图片后再加载~~

可用插件：pdf2htmlEX

问题：咱们资源是访问目标服务器动态获取的，不适合该业务场景。如果js在客户端转化的话有性能问题也没有找到合适的插件，舍近求远了。

### ~~四、使用pdfobject插件~~

遇到的问题：该插件基于**embed**标签，兼容性不好，经测试，iphone7和荣耀10均不支持

测试地址：<http://zhaohaipeng.com/dist/pdfobject.html>

兼容一览表：<https://en.wikipedia.org/wiki/Comparison_of_web_browsers#Image_format_support>

## 参考资料

1. <https://www.xiejiahe.com/detail/5be97f71df53a14006035e2a>
2. https://blog.csdn.net/qappleh/article/details/80250492
3. https://juejin.im/post/5a7badf26fb9a063353198a1
4. https://github.com/SimonZhangITer/MyBlog/issues/8
5. https://www.cnblogs.com/linxingyun/p/9293455.html

