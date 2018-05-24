title: "给博客加了小绿锁"
date:  2016-03-05 12:00:00
tags:
- Blogging
---
晚上偶然发现[轮神的新域名][1]的证书挂了，于是顺便去 [SSL.DO][2] 看了一眼 SSL 证书的价格，发现一个单域名的只要几美刀就好，就毫不犹豫买了一个。之前用的是 Hexo，全是静态页面，所以不要小绿锁也无所谓；现在改成动态网站了，该负的责还是要负起来的。

第一次接触 SSL，生成 CSR 时找错教程直接生成了个证书……后来是照着 DigitalOcean 的一篇教程来做的，[《How To Install an SSL Certificate from a Commercial Certificate Authority》][3]。提交后发现需要指定一个「证书审批邮箱」，除了 WhoisGuard 的邮箱之外全都是以「@kyouko.net」结尾的，于是手忙脚乱地去绑定了个域名邮箱然后才顺利接到邮件完成审核。

审核通过后证书会自动发到邮箱，用 scp 丢到 VPS 上放到正确的位置配置一下就好了。Chain certificates 的话 COMODO 发了三个 `.crt` 过来，可以 cat 成一个 `.ca-bundle`。重启 Apache，用 https 打头的 URL 访问已经可以顺利出现小绿锁了，开心。不过还是有两个小问题，一个是 Typecho 使用的 normalize.css 是放在一个没有 HTTPS 访问的 CDN 上的，在 cdnjs 找到相同的版本替换一下 URL 就好；另一个问题是访问 http://kyouko.net 不会自动跳转到 https。这个问题可以用 Apache 的 Rewrite 来实现但是 Apache 官方强烈反对，而是建议用 Redirect 功能：在域名的 conf 里像这样加上一行：

```
<VirtualHost *:80>
    ServerName kyouko.net
    Redirect / https://kyouko.net
    ...
</VirtualHost>
```

（后来又发现腾讯微博和新浪微博的 favicon 是 HTTP 连接，所以在文章页小绿锁消失了，不高兴。特别是新浪，居然把 HTTPS 重定向到 HTTP。暂时把这两个分享图标拿掉了，反正没有人会分享，没什么影响。等有时间了正儿八经写个插件。）

收工。

[1]: https://unstart.zone/
[2]: https://ssl.do/
[3]: https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority
