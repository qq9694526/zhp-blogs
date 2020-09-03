
### 步骤流程
1. 浏览器代理服务器设置和fiddler一致，端口默认8888
2. 打开目标网页，在fiddler列表中找到目标文件
3. 在AutoResponder 勾选Enable result 与unmatched requests passthrough
4. 点击add Rule按钮， 在右侧下方rule Editor填写要目标文件和替换文件 然后save
5. 页面重新访问 ，可通过log,看是否替换成功

### 参考链接
[fiddler抓包工具-- 本地资源替换线上文件](https://www.cnblogs.com/shichangchun/p/10731297.html)
