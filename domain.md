# 域名注册

要想实现一个web应用，域名可以说是必不可少的，人们通过域名来访问你的页面，应用之间通过域名来相互调用接口、资源等  
目前很有多平台都可以注册域名，我的这些服务基本都在[阿里云](https://www.aliyun.com/)完成的

## 域名注册
打开阿里云，你能看到它提供了纷繁复杂的服务内容，甚至连域名注册的入口都水容易发现，好在机智如我:)，在左上角的展开菜单里找到了入口：产品 > 精选 > 域名注册   
进入域名注册页面就可以在这里查找你心仪的域名了：
![域名注册](./images/domain/01.png)

如果我想注册*mimei*这个关键词相关的域名，可以在搜索框里直接输入这个关键词进行搜索，其结果会告诉你这个关键词相关的域名是否可以注册：
![域名注册](./images/domain/02.png)
如果显示“已注册”，说明这些域名已经被人注册了，你们就有办法再注册~ （可以尝试通过域名市场联系购买，这是另外的话题了）

现在假设我们将要注册的域名还是可注册状态，点击“加入清单”，这时会提交我们云登录，如果已经注册了帐户，直接登录就可以了。如果没有注册过阿里云，由于都是阿里系，你可以直接用taobao帐户或者支付宝帐户登录，这里不再赘诉，假设已经都完全了登录的过程，并且将想要的域名加入到了清单里，这时页面右侧的清单就会有我们所选的域名（当然你可以继续添加），点击“立即结算”，进入付款页面，按要求完成付款后，就算已经成功购买了。


## 域名解析
假设已经成功购买了域名，这里登录控制台后，通过左上角 -> 域名，进入域名管理页面，就可以看到我们刚刚购买的域名。  
通过域名对应的“解析”链接，进入该域名的解析页面，更详细的信息见阿里云域名解析[官方文档](https://help.aliyun.com/document_detail/29716.html)  
如果你对域名解析还没有明确概念，可以看一下知乎这个[帖子](https://www.zhihu.com/question/20266795)，应当会有帮助。

## 域名的备案
由于近年对网络管理更趋严格，域名不备案，服务器已经不能为你提供解析服务，阿里云的一会[通知](https://help.aliyun.com/noticelist/articleid/20722497.html?spm=5176.8087400.4.3.68b415c9ssCxHQ)里就有如下内容：  
“如您未能通过核验的，根据通知要求，阿里云将不能为您提供接入服务”  
所以要想使域名正常使用，备案要放在第一步（当然你可以去国外的服务商购买域名、服务器，跳过国内监管...），备案分公司和个人两种类型，由于备案需要提交各种证件，公司类型的还要提供营业执照、法人信息等内容，最终信息还要提交工信部审核，所以周期较长（3-4周不等），需提前预留好时间。  
备案过程中，可能会需要较多问题，好在阿里云提交了比较完整的[文档](https://help.aliyun.com/document_detail/61819.html?spm=5176.11065259.1996646101.searchclickresult.61f227a0tbGmtS)，而且有问题也可以电话客服，所以基本上都能解决。

## https证书申请及部署
什么是https，来自维基百科的解释：  
超文本传输安全协议（英语：Hypertext Transfer Protocol Secure，缩写：HTTPS，常称为HTTP over TLS，HTTP over SSL或HTTP Secure）是一种透过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。这个协议由网景公司（Netscape）在1994年首次提出，随后扩展到互联网上。  
为什么要用https呢？主要是“保护所有类型网站上的网页真实性，保护账户和保持用户通信，身份和网络浏览的私密性。”  
保护账户和保持用户通信，这个很好理解，我们交易时的敏感信息当然不希望明文在网络上传输，太容易遭到劫持和篡改。什么是“保护所有类型网站上的网页真实性”呢？举个我自身遇到的例子，想必就很容易理解了，在家的时候浏览我所在公司的网站，惊奇的发现页面上多了一个广告区域，做为一个前端开发人员，我清楚的知道这个区域肯定不是我加的，联系公司运维，排查后发现是由于我们小区的宽带运营商，对网络流量进行了劫持和篡改，将网络广告植入到页面中，由此获利。  
试想这是一个资讯类网站，没什么敏感信息，如果是一个交易类网站，等于是在网络上裸奔...，所以https对于很多网站来说至关重要。  

要实现https，就需要SSL证书，证书的价格对于个人来说相对较高，好在阿里云提供了单域名的免费证书，对于我们个人测试来说是够用的，具体访问是：左上角 -> 产品与服务 -> 安全 -> SSL证书，就可以进入到证书页面。  
点击右上角“购买证书”，品牌选Symantec，**第3项**保护类型 '1个域名'，这里第2项会有‘免费型DV SSL’选项，点击购买，去支付就好。  
免费证书只要按要求配置验证文件正确，系统就可自动完成签发。所以需要按提示，配置相应的TXT记录。

至此https的证书基本已经完全了，当取得证书后，下一步就看如何在服务器上部署证书。