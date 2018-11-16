@[toc]
## 写在前面
刚入门前端的时候写过很多的微信H5页面，时隔多年感觉应该是手到擒来，不曾想竟很是费了一些功夫。现在把本次开发过程中遇到的问题以及我是如何解决的，做个记录。防止自己以后再去解决解决过的问题。   
**github**：https://github.com/qq9694526/vue-wxh5
## 一、微信网页授权
网页授权流程分为四步，这里只说前端需要做的，其中的第一步：跳转授权页面获取code。
这里分享下我的授权逻辑（下图），它有两个优点：   
1. 授权跳转在dom渲染之前，体验会好一些；   
2. 本地存储了openId，前后端均不用频繁的与微信服务器交互。  

 ![Alt](http://i1.bvimg.com/510196/2d6c8d38fbd0364a.png)
## 二、微信jssdk授权
如果你页面中有用到分享、上传图片、微信支付等功能，那么需要先进行js-sdk授权。我这边封装成了2个方法：initConfig和setShare，方便在路由/页面切换的时候重复调用。
```javascript
//main.js
import wxsdk from './config/wxsdk.js' //该模块提供initConfig和setShare方法，具体代码太长见github
Vue.prototype.wxsdk = wxsdk;//挂载到全局

//使用
 created() {
   this.wxsdk.initConfig(location.href.split("#")[0], () => {
     this.wxsdk.setShare(this.user.openId);
    });
 }
```
## 三、webpack-dev-server解决跨域
讲真的所有跨域解决方案都必须有服务端的参与，诚然这个问题是浏览器抛出的，但让前端去解决真的很冤。下面两个配置让你永远告别跨域烦恼。本地开发用webpack-dev-server，测试生产环境用nginx。
```javascript
//接口根路径http://47.105.59.***:9090/zt-wx
//以vue-cli搭建的项目config举例 config/index.js
  dev: {
    proxyTable: {
      '/zt-wx': {
        target: 'http://47.105.59.***:9090',  //目标接口域名
        changeOrigin: true  //是否跨域
      }
    },
  }
//实际发起请求时的url
this.http.get(`/zt-wx/api/wx/info`).then(); //http是我自己对axios的再封装  

//nginx代理配置
server {
    location /zt-wx {
		proxy_pass http://47.105.59.***:9090;
    }
}        
```
## 四、iso初次加载白屏、跳转白屏
**问题现象**： ios页面初次加载白屏，刷新后正常，但切换到其他页面再后退，又会白屏。   
**问题原因**：在ios机器上使用webview开发Vue项目时候，go history(-1)，无法将body的高度拉掉，使得内容被遮住了。   
**解决办法**：html，body都是100%，#app撑起了父元素的告诉，但是浏览器默认的滚动scroll并不是#app，而是body,某些因素，造成返回history 后，无法复原（ios 的锅），为此，我们将#app进行了绝对定位，并让它重新成为 scroll 的对象，从而解决问题。   
```css
#app {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}
```
参考资料：https://blog.csdn.net/guxuehua/article/details/79026655
## 五、ios路由/跳转页面后分享失效
**问题现象**：ios微信路由到另一个页面选择图片OK，但分享失效，刷新这个页面分享就正常了。   
已累计尝试解决超过8小时，至今未果。   
参考资料：https://www.zhihu.com/question/59388458/answer/340351305
## 六、上传图片报错：处理异常
这个报错甚是诡异，因为前端和后端代码均没有“处理异常”这4个字。本来想甩锅给微信不管了的，但随后在做限制上传图片大小功能的时候阴差阳错的给解决了。   
**问题原因**：后端tomcat服务默认设置表单提交数据大小上限为2M，大于2M就会报错。   
**解决办法**：后端大神把server.xml中maxPostSize的值改为-1后解决。   
参考资料：https://blog.csdn.net/w18756901575/article/details/79374621
## 七、正确导出图片格式
这个项目首页基本是由图片堆砌成的，一开始切出来的图(默认.png)压缩后在400k-1.3M之间。一开始还以为PSD素材有问题。直到项目最后才闪回，想起图片格式的知识点，改导出成.jpg格式后压缩出来的图片基本控制在100K以内了。具体的.png.jpg这些图片格式的知识有兴趣的自己查。
## 八、vuex使用之同步用户信息
讲道理小项目是不应该用vuex的，但是用着确实爽，即简单又省心省力。由于我总是忘记它的方法名，所以在这里贴下代码，方便以后随时cv。  
```javascript
//config/store.js
const store = new Vuex.Store({
    state: {
        user: {}
    },
    mutations：{
        updateUser(state, data){
            state.user = data;
        }
    }
})
//在组件中使用
computed: {
    user() {
        return this.$store.state.user;
    }
}
//在需要的时候更新数据
this.$store.commit("updateUser", user);
```
## 九、使用html2canvas生成的海报不显示图片
**问题原因**：引入的图片资源路径跨域造成的。   
**解决办法**：我先是按照官方给的那个php的方案弄的，未能解决。最后舔着脸让后端大佬把图片资源目录挪到我web服务目录下给解决的。   
```javascript
//安装
npm install --save html2canvas
//引入
import html2canvas from "html2canvas";
//使用
const myPosterWrap = document.getElementById("posterWrap");
html2canvas(myPosterWrap).then(canvas => {
    this.posterSrc = canvas.toDataURL("image/png");
    this.uploadPosterImg(this.posterSrc);
});
```
## 十、css黑科技之放置指定比例的图片
就是把不定宽图片按指定比例显示，直接上码（1：1.25）。
```html
//html
<div class="poster-img-wrap">
    <div class="poster-img-place"></div>
    <img class="poster-img" :src="user.picAddress" alt="">
</div>
```
```css
//less
.poster-img-wrap {
    position: absolute;
    top: 28%;
    left: 0;
    right: 0;
    width: 80%;
    margin: 0 auto;
    .poster-img-place {
        width: 100%;
        padding-top: 125%;
    }
    .poster-img {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }
}
```
## 十一、ios页面加载不全不能滚动
**问题描述** ：ios从首页进入，跳转其他页面再后退到首页，首页只显示一屏内容且无法滚动。
**问题原因**：在于ios浏览器内核的别致渲染逻辑：它会预先找到相应的overflow: scroll元素，如果子元素高度高于父元素，则建立原生的scrollView实现滚动。问题就出现在这个“预先”上，它预先获取的高度并不是子元素渲染后的真实高度。
**解决办法**：给设置了滚动的#app元素下的子元素p-index设置min-height: calc(100% + 1px);
```css
#app {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}
.p-index{
   min-height: calc(100% + 1px);
}
```
其实能解决全靠这篇博文，这里就不赘述了。https://blog.csdn.net/nongweiyilady/article/details/83039868   
神奇的是，我能看到这篇文章全来赖于地铁上无聊打开了很久没打开的CSDNapp，切到前端分类，“不滚动”三个字立马映入眼帘，点进去的第一眼 ，就感觉是对的人了。迷茫的时候就看书，古人诚不我欺！
## 一些忠告
> - 能小程序就别网页开发；
> - 不意淫不揣摩待定的需求；
> - 坚持看图作业的优良传统；
> - 迷茫的时候看书，焦虑的时候学习；
