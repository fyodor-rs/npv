本文是搭建V2Ray服务器（ws+tls+web）的完整教程。除了搭建V2Ray+ws+tls+web的过程，还包括配置CDN隐藏IP，打开BBR加速，以及简单配置防火墙防探测。最近（2020年初开始）防火墙加大了封杀VPN的力度，很多SS，SSR，纯VMess都开始间歇掉线，有些代理甚至直接被封IP。如果你打算自己搭建翻墙服务，强烈推荐V2Ray+ws+tls+web（CDN可选）一步到位。

本文面向小白，你只要会购买虚拟主机，会使用SSH连接服务器，那么看懂本教程就毫无压力。当然如果有搭建SS/SSR的经验，就更容易看懂了。建议能看懂本教程的，尽量购买VPS自建翻墙服务。按照本文搭建实在有困难的用户，可以考虑迷雾通或其它外资VPN。

不推荐使用一键脚本，很多一键脚本都存在安全隐患，轻则屏蔽掉几个网站，重则把你的服务器变成“肉鸡”（即黑客攻击别人电脑的跳板）。另外，就算用脚本搭建服务器，本文中的大部分操作，比如购买域名，配置域名解析等等，脚本无法自动完成。你恐怕还要亲自购买域名，亲自配置域名解析，亲自登录VPS执行脚本。本文中的方法只比一键脚本多出几步，但是可以大大降低安全风险。

自建V2Ray服务器首先要购买VPS（虚拟主机），为避免广告嫌疑，正文中不推荐VPS，我会在评论中补充一些常见的外资VPS。一般而言，自建服务的成本远远低于机场，大多数VPS每个月花费20-30元左右，流量1TB/月，有些外资VPS价格低到10-20元，甚至每月不到10元。如果愿意折腾，还可以用免费的谷歌云。此外，自建服务没有客户端数量限制，如果多人分摊成本，价格就更便宜了。

V2Ray+ws+tls+web是目前最稳定的翻墙技术之一，即使在六四、十一也稳如泰山。和SSR的流量混淆不同，V2Ray+ws+tls用真正的https流量翻墙，没有必要做任何混淆。在防火长城看来，你的流量和不计其数的https流量没有任何区别。但是，如果有好事者主动访问你的代理服务器，就会发现一些不对劲：
[![https://i.imgur.com/avZQgib.png](https://i.imgur.com/avZQgib.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9hdlpRZ2liLnBuZw)

【fig0.1】


尽管流量没有任何明显的特征，但是如果墙主动访问代理服务器，会发现流量的目的地没有真正的网站，从而识破https流量的目的。使用https流量不做掩护，反而增大了IP被墙的概率。因此你需要在V2Ray服务外面加一层真网站做掩护。
[![https://i.imgur.com/raSjDmM.png](https://i.imgur.com/raSjDmM.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9yYVNqRG1NLnBuZw)

【fig0.2】


这就是V2Ray+ws+tls再加上web的原理。在配置了真实网站之后，只有你自己知道是个代理，在别人看来是个网站（包括墙）。反过来说，V2Ray+ws+tls+web也可以看作是自建了一个网站，然后利用这个网站来翻墙。因此，如果你读完本文，不仅能学会V2Ray翻墙，还能学到一点建网站的流程。

上面“https流量”的正确叫法是“tls流量”，这东西就是你平时浏览网页发出的流量。这里为了方便新手理解，叫做“https流量”。

**安全提示：**

> 如果服务器之前运行过SS，SSR，V2Ray（非TLS）等服务，请确保先停止原来的代理服务，再安装V2Ray。如果不知道怎么停止，请重装VPS（在网页控制面板上点reinstall，不到一分钟就搞定）。翻墙的隐蔽性取决于最薄弱的一环，如果服务器上同时运行其它代理软件，这些代理软件依然会被墙探测到，这种情况下V2Ray+ws+tls+web并不能保证隐蔽性。



自建网站看上去很复杂，其实很简单，只要按照以下步骤：



- 购买域名&配置域名解析：V2Ray需要域名伪装成真正的网站，因此你需要购买一个域名，并把域名绑定到服务器的IP地址上。
- 一键填写配置文件：本文已经写好了配置文件，只要把域名，UUID等信息填进去就行了。
- 一键安装v2ray+nginx
- 上传配置文件&一键运行
- 配置客户端


成功翻墙以后，还可以做以下事情进一步强化：

- 可选1. 加固服务器
- 可选2. 配置CDN隐藏IP
- 可选3. 使用BBR加速
- 可选4. 自行编译Nginx


下面按照以上顺序讲解配置。



**1. 购买域名&配置域名解析**


这里从域名注册商GoDaddy购买域名，用Cloudflare提供域名解析。GoDaddy本身也提供域名解析服务，这里用Cloudflare是为了配置CDN方便。如果遇到墙加高，或者IP被墙等极端情况，只要简单的配置就可以切换成v2ray+ws+tls+web+cdn。CDN翻墙速度较慢，但是稳定性极高。

可能不少新手没听说过GoDaddy和Cloudflare，这里介绍一下，GoDaddy是世界上最大的域名提供商，占据市场30%的份额。Cloudflare是世界上最大的CDN提供商，全球半数的网站都在使用Cloudflare。注册不用担心隐私泄露或钓鱼风险。

注册GoDaddy和CloudFlare需要邮箱。特别注意，购买域名之后，域名服务商会公开邮箱地址。建议至少使用gmail注册，如果对隐私有较高要求，可以用Protonmail等匿名邮箱注册。



**1.1 注册GoDaddy**


点击这里进入[GoDaddy的新加坡官网](https://pincong.rocks/url/link/aHR0cHM6Ly9zZy5nb2RhZGR5LmNvbS96aA)，全过程都有中文界面。出于安全考虑，选择邮箱注册。
[![https://i.imgur.com/bu3N2ZK.png](https://i.imgur.com/bu3N2ZK.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9idTNOMlpLLnBuZw)

【fig1.1.1】





**1.2 选择一个域名**


GoDaddy和淘宝的用法完全一样。注册完成以后，回到GoDaddy首页。点击搜索框，输入一个你想要的域名，查询价格。GoDaddy会根据域名包含的词汇定价，为了降低成本，域名尽量选得随机一些，其中不要包含任何单词，我这里头滚键盘输入hrw1rdzqa7c5a8u3ibkn。
[![https://i.imgur.com/IxWvPW8.png](https://i.imgur.com/IxWvPW8.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9JeFd2UFc4LnBuZw)

【fig1.2.1】


可以看到不同后缀的域名价格差别很大，通常.com .net这类域名比较昂贵.website，.site，.rocks，.xyz价格较低，选一个最便宜的。这里我们选www.hrw1rdzqa7c5a8u3ibkn.website，这个域名一年不到7块钱。如果你购买时发现价格贵一些也是正常的，域名首年的价格通常不到10块钱。
[![https://i.imgur.com/ywKsYoB.png](https://i.imgur.com/ywKsYoB.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS95d0tzWW9CLnBuZw)

【fig1.2.2】


接下来进入购物车，隐私保护不用选。
[![https://i.imgur.com/8fM0rJP.png](https://i.imgur.com/8fM0rJP.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS84Zk0wckpQLnBuZw)

【fig1.2.3】


点进入购物车，进入结算页面。GoDaddy默认选购买2年，我们的域名只用来翻墙，选1年就可以了，到时候再换。
GoDaddy支持支付宝或信用卡付款。第一次购买域名，GoDaddy会要求你填写个人信息，这里姓名和手机号随便填一个假的就行。
[![https://i.imgur.com/NACRyMf.png](https://i.imgur.com/NACRyMf.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9OQUNSeU1mLnBuZw)

【fig1.2.4】


购买完成，刚买到的域名不会马上在域名列表里出现，一般会有一两分钟的延迟。
接下来注册Cloudflare。GoDaddy的页面暂时不要关，一会还要回来配置域名服务器。



**1.3 注册CloudFlare。**


打开Cloudflare官网，用邮箱注册，如图。注册页面入口[https://dash.cloudflare.com/sign-up](https://pincong.rocks/url/link/aHR0cHM6Ly9kYXNoLmNsb3VkZmxhcmUuY29tL3NpZ24tdXA)
[![https://i.imgur.com/S2epVPg.png](https://i.imgur.com/S2epVPg.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9TMmVwVlBnLnBuZw)

【fig1.3.1】


接下来输入刚才购买的域名，注意这里输入的是【二级域名】。所谓二级域名，可以理解为网址去掉www。
比如我的网站的网址是www.hrw1rdzqa7c5a8u3ibkn.website，那么这里应该输入hrw1rdzqa7c5a8u3ibkn.website，如图：
[![https://i.imgur.com/Q8Q5TA9.png](https://i.imgur.com/Q8Q5TA9.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9ROFE1VEE5LnBuZw)

【fig1.3.2】


点【Add Site】，把域名交给Cloudflare托管。

接下来选择套餐，这里选择FREE套餐。
[![https://i.imgur.com/brAXbxG.png](https://i.imgur.com/brAXbxG.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9ickFYYnhHLnBuZw)

【fig1.3.3】


点【Confirm Plan】，进入管理页面，不要关掉页面，接下来配置域名解析。



**1.4 配置域名解析**


配置域名解析分两步：

- 配置域名服务器记录（也叫name server，NS记录）
- 配置地址解析记录（也叫address，A记录）


NS记录用来表明由哪台服务器对域名进行解析。从GoDaddy买到域名后，域名是由Godaddy的服务器进行解析的。我们这里把Godaddy的服务器换成Cloudflare的服务器。

如图是Cloudflare的管理界面，如果你的域名之前有配置域名解析，管理界面会显示之前的记录。暂时不用管这些。
点Continue，修改域名服务器。



【fig1.4.1】


点【Continue with default】
[![https://i.imgur.com/YiZuOAP.png](https://i.imgur.com/YiZuOAP.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9ZaVp1T0FQLnBuZw)

【fig1.4.2】


接下来Cloudflare会提示你变更域名服务器，并给出了方法。
[![https://i.imgur.com/HTUNjSb.png](https://i.imgur.com/HTUNjSb.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9IVFVOalNiLnBuZw)

【fig1.4.3】


画红框的部分就是Cloudflare提供的两个域名服务器，我这里是ns90.domaincontrol.com和ns91.domaincontrol.com，
你看到的的可能和我不一样。

回到[GoDaddy](https://pincong.rocks/url/link/aHR0cHM6Ly9zZy5nb2RhZGR5LmNvbS96aA)，点击屏幕右上角的用户名，选择【我的产品】。
[![https://i.imgur.com/H7iSxUX.png](https://i.imgur.com/H7iSxUX.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9IN2lTeFVYLnBuZw)

【fig1.4.4】


这里可以看到你拥有的域名，点击域名旁边的【DNS】，进入DNS管理页面。
[![https://i.imgur.com/he6OqIE.png](https://i.imgur.com/he6OqIE.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9oZTZPcUlFLnBuZw)

【fig1.4.5】


在DNS管理界面向下拉，找到域名服务器。如图所示，这里可以看到GoDaddy提供的两个域名服务器，点击【更改】。
[![https://i.imgur.com/LBZ5vNd.png](https://i.imgur.com/LBZ5vNd.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9MQlo1dk5kLnBuZw)

【fig1.4.6】


选择【输入我自己的域名服务器】
[![https://i.imgur.com/93ic5KS.png](https://i.imgur.com/93ic5KS.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS85M2ljNUtTLnBuZw)

【fig1.4.7】


输入刚才Cloudflare提供的两个域名服务器，我这里是fccp.ns.cloudflare.com和xjp.ns.cloudflare.com，
点击【保存】。
[![https://i.imgur.com/LUl7HAd.png](https://i.imgur.com/LUl7HAd.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9MVWw3SEFkLnBuZw)

【fig1.4.8】


注意不要有多余的域名服务器，不是CloudFlare提供的就要删除，否则可能会出问题。

接下来转移域名服务器可能需要几分钟，转移完成后会收到Cloudflare的邮件，可以先等一阵子。

配置地址解析（A记录）
转移域名服务器完成后，进入[cloudflare的首页](https://pincong.rocks/url/link/aHR0cHM6Ly93d3cuY2xvdWRmbGFyZS5jb20)，点击右上角的【log in】，进入你的账户，如图：
[![https://i.imgur.com/rdFD2Ke.png](https://i.imgur.com/rdFD2Ke.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9yZEZEMktlLnBuZw)

【fig1.4.9】


点击买来的域名，进入下一步，如图：
[![https://i.imgur.com/dMSA0xq.png](https://i.imgur.com/dMSA0xq.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9kTVNBMHhxLnBuZw)

【fig1.4.10】


点击【DNS】按钮，进入Cloudflare的DNS管理页面，如下图：
[![https://i.imgur.com/BCeOWfm.png](https://i.imgur.com/BCeOWfm.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9CQ2VPV2ZtLnBuZw)

【fig1.4.11】


点击【Add record】，一次可以添加一条解析记录.

这里简单讲解一下，如果不想了解原理，可以跳过这一部分。

每条域名解析有四个部分：Type，Name，Address，TTL

Type是域名解析的类型，常见的几种有

- A记录：即地址（Address）记录，用来指定域名的IPv4地址。如果需要将域名指向一个IP地址，就需要添加A记录。举个例子，我们要把域名www.hrw1rdzqa7c5a8u3ibkn.website指向VPS的IP地址218.30.118.6，就要添加A记录。
- AAAA记录，指定域名的IPv6地址。
- CNAME：即规范名字（Canonical Name）记录，俗称“别名”。如果需要把域名指向另一个域名，就要添加CNAME记录。
- NS：域名服务器记录，如果要把域名交给其他DNS服务器解析，就需要添加NS记录。我们刚才修改的就是NS记录。


接下来说明每个选项应该填什么，以及为什么这么填：

**Name**
对于A记录，这里介绍三种填法

- www：表示解析带www的域名，即www.hrw1rdzqa7c5a8u3ibkn.website
- @：直接解析裸域名，即hrw1rdzqa7c5a8u3ibkn.website
- *：表示泛解析，即匹配其他所有域名 *.hrw1rdzqa7c5a8u3ibkn.website


**Address**
这里填VPS的IP地址，我这里是218.30.118.6

**TTL**
即 Time To Live，缓存的生存时间。指本地dns缓存解析记录的时间，缓存失效后会再次获取记录。在Cloudflare里，如果配置了CDN，则这里填Auto即可；如果没有配置CDN，可以选择Auto，也可以选择一个大于1小时的值。

理解了上面的内容以后，接下来添加两条A记录（如果VPS只有IPv6地址则添加AAAA记录）

点一下云朵，确保云是灰色的（DNS only）。橘色云朵表示此解析记录使用CDN，灰色云朵表示不使用 CDN，点击云朵可以切换。这里不要使用CDN，否则接下来的配置会出问题。

- 第一条A记录，name填www，address填服务器IP地址，TTL选择1小时。（表示解析带www的地址）
- 第二条A记录，name填@，address填服务器IP地址，TTL选择1小时。（表示解析不带www的裸域名）


填好之后的正确结果如图：
[![https://i.imgur.com/qDKfuRy.png](https://i.imgur.com/qDKfuRy.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9xREtmdVJ5LnBuZw)

【fig1.4.12】


注意云一定是灰色的。


**检查是否配置成功：**
配置完成后点【save】就大功告成了，可以打开windows的powershell，输入

```
ping www.hrw1rdzqa7c5a8u3ibkn.website
```


(替换成你的域名)

Ping一下你的域名，如果能Ping通，就说明域名解析没问题。



**2. 配置文件**


V2Ray和Nginx的配置文件这里已经写好。你需要做的就是填表格，把配置中标出的地方换成自己的内容。
编辑配置文件可以用Windows记事本，不过推荐使用Notepad++。
下载地址：
[网页链接](https://pincong.rocks/url/link/aHR0cHM6Ly9ub3RlcGFkLXBsdXMtcGx1cy5vcmcvZG93bmxvYWRzLw)，[直接下载链接(7.8.4)](https://pincong.rocks/url/link/aHR0cHM6Ly9naXRodWIuY29tL25vdGVwYWQtcGx1cy1wbHVzL25vdGVwYWQtcGx1cy1wbHVzL3JlbGVhc2VzL2Rvd25sb2FkL3Y3LjguNC9ucHAuNy44LjQuSW5zdGFsbGVyLng2NC5leGU)



**2.1 V2Ray配置文件**


V2Ray配置文件如下：

```
{
"inbound": {
    "protocol": "vmess",
    "listen": "127.0.0.1",
 "port": 8964,
 "settings": {"clients": [
        {"id": "◆◆◆◆◆◆◆◆◆◆◆◆"}
    ]},
 "streamSettings": {
 "network": "ws",
 "wsSettings": {"path": "/★★★★★★★★★★★★"}
    }
},

"outbound": {"protocol": "freedom"}
}
```


是不是很短？接下来把标了符号的地方换成你自己的信息。

**(1)**
◆◆◆◆◆◆◆◆◆◆◆◆：标“◆”的地方填写UUID。

UUID可以从这个网站生成：[https://www.uuidgenerator.net/](https://pincong.rocks/url/link/aHR0cHM6Ly93d3cudXVpZGdlbmVyYXRvci5uZXQv)。只要打开或者刷新这个网页就可以得到一个UUID。
举个例子，我生成的UUID是：



> 63c0042a-4a85-4d03-a488-3ba3aa002461



**(2)**
★★★★★★★★★★★★：标“★”的地方填写一个随机字符串。注意不要删掉前面的斜杠。

“随机字符串”就是你在键盘上胡乱敲打出来的东西，比如dsfhsdjfhref。[推荐用这个网站生成一个](https://pincong.rocks/url/link/aHR0cHM6Ly93d3cucmFuZG9tLm9yZy9zdHJpbmdzLz9udW09MSZsZW49NyZkaWdpdHM9b24mdXBwZXJhbHBoYT1vbiZsb3dlcmFscGhhPW9uJnVuaXF1ZT1vZmYmZm9ybWF0PWh0bWwmcm5kPW5ldw)，只要打开或刷新网页就可以得到一个随机字符串。

我用这个网站随机生成的字符串是mL7Gg8K

这个随机字符串就是WebSocket路径，**不要抄我这里的例子，去自己生成一个！**否则会被墙探测出来。建议WebSocket路径取得长一些（5个字符以上），过于简单，过于常见的路径（比如/ray，/v2，/v2ray之类的名称），很容易被墙探测出来。

填好之后的配置如下图：
[![https://i.imgur.com/K1FEnrw.png](https://i.imgur.com/K1FEnrw.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9LMUZFbnJ3LnBuZw)

【fig2.1.1】


最后，把V2Ray的配置文件另存为config.json



**2.2 Nginx配置文件**


Nginx配置文件如下：

```
server {
    ### 1:
    server_name ●●●●●●●●●●●●;

    listen 80 reuseport fastopen=10;
    rewrite ^(.*) https://$server_name$1 permanent;
    if ($request_method  !~ ^(POST|GET)$) { return  501; }
    autoindex off;
    server_tokens off;
}

server {
    ### 2:
    ssl_certificate /etc/letsencrypt/live/●●●●●●●●●●●●/fullchain.pem;

    ### 3:
    ssl_certificate_key /etc/letsencrypt/live/●●●●●●●●●●●●/privkey.pem;

    ### 4:
    location /★★★★★★★★★★★★
    {
        proxy_pass http://127.0.0.1:8964;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_requests 10000;
        keepalive_timeout 2h;
        proxy_buffering off;
    }

    listen 443 ssl reuseport fastopen=10;
    server_name $server_name;
    charset utf-8;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_requests 10000;
    keepalive_timeout 2h;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers off;

    ssl_session_cache shared:SSL:60m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 10s;

    if ($request_method  !~ ^(POST|GET)$) { return 501; }
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options nosniff;
    add_header Strict-Transport-Security max-age=31536000 always;
    autoindex off;
    server_tokens off;

    index index.html index.htm  index.php;
    location ~ .*\.(js|jpg|JPG|jpeg|JPEG|css|bmp|gif|GIF|png)$ { access_log off; }
    location / { index index.html; }
}
```


看上去很长，实际上只有四处需要填写，配置文件里用#1，#2，#3，#4标出了位置，把标符号的地方换成你自己的信息。

●●●●●●●●●●●●：标注“●”的地方填写域名，**注意这里的域名带www**

这个域名就是前面购买的域名，本文中是www.hrw1rdzqa7c5a8u3ibkn.website，一共有三处需要填，都以“●”标出：

**(1)** server_name ●●●●●●●●●●●●; 注意域名和前面的server_name保持一个空格，后面的分号“;”不要删掉。
填好之后：（注意带www，以下皆相同）

```
server_name www.hrw1rdzqa7c5a8u3ibkn.website;
```


**(2)** ssl_certificate /etc/letsencrypt/live/●●●●●●●●●●●●/fullchain.pem; 两边的斜杠“/”不要删掉。
填好之后：

```
ssl_certificate /etc/letsencrypt/live/www.hrw1rdzqa7c5a8u3ibkn.website/fullchain.pem;
```


**(3)** ssl_certificate /etc/letsencrypt/live/●●●●●●●●●●●●/fullchain.pem; 一样，两边的斜杠“/”不要删掉
填好之后：

```
ssl_certificate /etc/letsencrypt/live/www.hrw1rdzqa7c5a8u3ibkn.website/fullchain.pem;
```


**(4)** ★★★★★★★★★★★★：标注“★”的地方填写一个随机字符串，这个随机字符串必须和V2Ray配置中的一样，不然无法工作。注意不要删掉前面的斜杠。
这个例子里，此处填mL7Gg8K。

填好之后的配置如下图：
[![https://i.imgur.com/Vc5LhXw.png](https://i.imgur.com/Vc5LhXw.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9WYzVMaFh3LnBuZw)

【fig2.1.2】


最后，把Nginx的配置文件另存为default.conf（注意扩展名就是.conf）



**3. 上传配置 & 运行**



**3.1 连接到服务器，安装v2ray+Nginx**


很多新手在买到VPS之后不知所措，其实VPS和游戏账号是一样的，买游戏账号付款之后，店家会私信告诉你两件东西：

> 用户名，密码



买VPS付款之后，VPS提供商会给你发邮件，告诉你四件东西：

> IP地址，密码，登录账号，端口



IP地址和密码一定会有，登录账号如果没说，默认是root。端口如果没说，默认填22.

拿到登录信息之后，就可以登录服务器了。这里推荐Bitvise SSH，轻量级，但是功能强大。
下载链接：[https://www.bitvise.com/ssh-client-download](https://pincong.rocks/url/link/aHR0cHM6Ly93d3cuYml0dmlzZS5jb20vc3NoLWNsaWVudC1kb3dubG9hZA)
直接下载链接：[https://dl.bitvise.com/BvSshClient-Inst.exe](https://pincong.rocks/url/link/aHR0cHM6Ly9kbC5iaXR2aXNlLmNvbS9CdlNzaENsaWVudC1JbnN0LmV4ZQ)
安装好之后的界面如下图，点红框圈起的下拉菜单，【Initial method】下拉菜单里面选【password】，在【Store encrypted password in profile】选项上打勾。
[![https://i.imgur.com/FliqoSK.png](https://i.imgur.com/FliqoSK.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9GbGlxb1NLLnBuZw)

【fig3.1.1】


这里假定我们的IP地址是218.30.118.6，密码是12345，登录账号和密码分别是root和22. 如图：

- 【Host】：这里填IP地址
- 【Port】：这里填端口，如果没说就是22
- 【Username】：这里填用户名，如果没说就是root
- 【Password】：登录密码。

填好之后如下图，点【Log in】就可以登录了。
[![https://i.imgur.com/mWm5Oxq.png](https://i.imgur.com/mWm5Oxq.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9tV201T3hxLnBuZw)

【fig3.1.2】


第一次登录服务器会弹出窗口，问是否要保存密钥，点【Accept and save】继续。

成功登录之后会弹出两个窗口。
[![https://i.imgur.com/DvucZdx.png](https://i.imgur.com/DvucZdx.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9EdnVjWmR4LnBuZw)

【fig3.1.3】


[![https://i.imgur.com/UNzAKOO.png](https://i.imgur.com/UNzAKOO.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9VTnpBS09PLnBuZw)

【fig3.1.4】


一个是文件浏览窗口，使用方法和Windows资源管理器一样，这里可以浏览服务器上的文件。把电脑中的文件拖拽到这里，就可以把文件上传到服务器。
另一个黑色的窗口是命令窗口。在电脑上复制一段文字，在这个窗口上右键，就可以把文字粘贴到命令窗口。



**3.2 配置SSL证书**


为了用真正的https流量翻墙，我们的网站必须有合法的SSL证书。可以用自动化工具Certbot申请证书，只要把以下命令复制到命令窗口，依次执行即可。

这里说的“证书”，实际指的是“数字证书”。当然申请完全是免费的，申请时需要邮箱地址。如有必要，可以使用匿名邮箱。

**(1) 安装Certbot：**

```
yum install -y python36 && pip3 install certbot
```



运行这条命令后，如果显示：

> Successfully installed xxxx, xxxx, xxxx (各种软件包名字)


就表示成功。

**(2) 停止防火墙**

```
systemctl stop firewalld && systemctl disable firewalld
```


注意，在CentOS7版本以上，默认开启防火墙，**不关闭防火墙将无法申请证书**。某些系统上没有安装firewalld防火墙，执行这一步命令会报错，但是不影响后面的操作。

运行这条命令后，如果显示：

> Removed /etc/systemd/system/multi-user.target.wants/firewalld.service.
> Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.


就表示成功

**(3) 申请SSL证书**
这一步做个填空题，把这条命令里的域名和邮箱，换成你自己的信息。

```
certbot certonly --standalone --agree-tos -n -d www.●●●●●● -d ●●●●●● -m ▲▲▲@▲▲▲.▲▲▲
```


第一个-d加一个带www的域名，第二个-d加一个不带www的域名，-m后面加你的电子邮箱。
注意前后要带空格。

例子：（域名：www.hrw1rdzqa7c5a8u3ibkn.website，邮箱：xijinping@protonmail.com）

```
certbot certonly --standalone --agree-tos -n -d www.hrw1rdzqa7c5a8u3ibkn.website -d hrw1rdzqa7c5a8u3ibkn.website -m xijinping@protonmail.com
```


运行这条命令后，如果显示：

> IMPORTANT NOTES:
> \- Congratulations! Your certificate and chain have been saved at:
>  /etc/letsencrypt/live/www.hrw1rdzqa7c5a8u3ibkn.website/fullchain.pem
>  Your key file has been saved at:
>  /etc/letsencrypt/live/www.hrw1rdzqa7c5a8u3ibkn.website/privkey.pem
>  Your cert will expire on 2020-06-04. To obtain a new or tweaked
>  version of this certificate in the future, simply run certbot
>  again. To non-interactively renew *all* of your certificates, run
>  "certbot renew"
> \- Your account credentials have been saved in your Certbot
>  configuration directory at /etc/letsencrypt. You should make a
>  secure backup of this folder now. This configuration directory will
>  also contain certificates and private keys obtained by Certbot so
>  making regular backups of this folder is ideal.
> \- If you like Certbot, please consider supporting our work by:
>
>  Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
>  Donating to EFF:          https://eff.org



就表示成功。

注意：这一步比较容易出错，常见的问题有：

- 其它代理占用了80，443端口。解决方法：停止其它代理软件，或重装VPS。
- 没有正确配置域名解析。解决方法：ping一下域名，看看能不能正确解析到IP。注意不要打开CDN（云朵点灰）。
- 没有关闭防火墙。解决方法：回到（2），关闭防火墙。


**(4) 配置证书自动更新**

```
echo "0 0 1 */2 * service nginx stop; certbot renew; service nginx start;" | crontab
```


运行这条命令后，如果没有任何信息输出，就表示成功。

我们申请的证书只有三个月期限，上面的命令表示每隔两个月，证书就自动续命一次，从而保证可以一直用下去。

**3.3 安装V2Ray和Nginx**
**(1) 一键安装**
V2Ray和Nginx可以一键安装，把下列命令复制粘贴到控制台，运行即可。



```
yum install -y nginx && yum install -y curl && bash -c "$(curl -L -s https://install.direct/go.sh)"
```


运行这条命令后，如果最后一行显示：
V2Ray v4.x.x is installed.
就表示成功。（如果V2Ray安装成功，那么Nginx也一定安装成功）

**(2) 关闭SELinux**
在某些系统上，需要关闭SELinux，否则Nginx无法正常将流量转发给V2Ray，输入

```
setsebool -P httpd_can_network_connect 1 && setenforce 0
```


关闭SELinux，没有提示就表示成功。



**4. 上传配置文件&运行**


安装好V2Ray和Nginx后，我们终于来到了最后一步。在启动V2Ray之前，需要把之前的配置文件上传



**4.1 上传配置文件**


这一步把第（2）步编辑好的配置文件上传就可以了。
首先上传V2Ray的配置文件，V2Ray的配置文件存储在

```
/etc/v2ray
```


目录下，把上面这个路径，复制到文件管理器的路径栏，回车，即可跳转到该目录下。如图：
[![https://i.imgur.com/Cj92ghh.png](https://i.imgur.com/Cj92ghh.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9DajkyZ2hoLnBuZw)

【fig4.1.1】


可以看到这里已经有一个config.json文件了，这是V2Ray安装时自动生成的。接下来的操作和Windows一样，把你编辑好的config.json拖拽到这里，就可以上传了。

文件管理器会提示你存在同名文件，选择【Overwrite】覆盖原来的文件即可。
[![https://i.imgur.com/n0Uqnj7.png](https://i.imgur.com/n0Uqnj7.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9uMFVxbmo3LnBuZw)

【fig4.1.2】


上传完成好以后，最好再验证一下，输入

```
/usr/bin/v2ray/v2ray -test -config=/etc/v2ray/config.json
```


如果显示：

> V2Ray 4.x.x (V2Fly, a community-driven edition of V2Ray.)
> A unified platform for anti-censorship.
> Configuration OK.



说明配置没有问题。

然后按同样的步骤上传nginx配置文件，Nginx的配置文件存储在

```
/etc/nginx/conf.d
```


目录下，转到这个目录，拖拽上传你编辑的default.conf文件即可。

再验证一下Nginx配置是否正确，输入：

```
nginx -t
```


如果显示：

> nginx: the configuration file /etc/nginx/ngin短网址nf syntax is ok
> nginx: configuration file /etc/nginx/ngin短网址nf test is successful



说明配置没有问题。



**4.2 启动**


V2Ray和Nginx都是守护进程，可以认为是Windows上的“后台服务”。把V2Ray和Nginx配置成守护进程以后，这两个程序就可以在服务器上持续运行了。Linux服务器的稳定性非常高，可以连续不重启运行一年，甚至更长。我们接下来就让V2Ray和Nginx在服务器上运行一年。

在Linux上启动一个守护进程很简单，输入以下两条命令就可以启动V2Ray和Nginx：



```
service v2ray start
service nginx start
```


有启动，也有其它操作，这里列出所有有用的命令，方便管理后台：

启动V2Ray:

```
service v2ray start
```


重启V2Ray:

```
service v2ray restart
```


注：这一条是常用命令，每次修改配置文件后，都要重启一下V2Ray。

查看V2Ray状态：

```
service v2ray status
```


停止V2Ray：

```
service v2ray stop
```


查看V2Ray版本:

```
/usr/bin/v2ray/v2ray -version
```


测试V2Ray配置文件:

```
/usr/bin/v2ray/v2ray -test -config=/etc/v2ray/config.json
```


注：常用命令，每次修改配置文件后，最好检查一下配置文件是否正确。
配置文件位置：

```
/etc/v2ray/config.json 
```


Nginx：
启动Nginx:

```
service nginx start
```


重启Nginx:

```
service nginx restart
```


查看Nginx状态：

```
service nginx status
```


停止Nginx：

```
service nginx stop
```


测试Nginx配置文件:

```
nginx -t
```


配置文件位置：

```
/etc/nginx/conf.d/default.conf
```


配置完成后，可以在浏览器里输入网址，如果显示Nginx的红色欢迎页面，就说明网址配置成功了！
接下来要做的是上传一个网页模板，这样别人访问你的服务器就会看到一个真的网站。
[![https://i.imgur.com/Npk1rOW.png](https://i.imgur.com/Npk1rOW.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9OcGsxck9XLnBuZw)

【fig4.2.1】





**4.3 上传网页模板**


去Google上搜“website template”可以找到很多提供网页模板的网站，这里随便找一家，例如[https://colorlib.com/wp/templates/](https://pincong.rocks/url/link/aHR0cHM6Ly9jb2xvcmxpYi5jb20vd3AvdGVtcGxhdGVzLw)

网页模板强烈建议用纯英文模板，其中不要包含任何中文内容，否则（可能）会增加网站被墙的概率。

下载好以后，解压压缩文件，一路点开，如图，可以看到里面有一个index.html文件(有些是index.htm或index.php)
[![https://i.imgur.com/8UXO1Ul.png](https://i.imgur.com/8UXO1Ul.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS84VVhPMVVsLnBuZw)

【fig4.3.1】


把这个文件夹里的所有东西，包括index.html，blog.html，以及css，fonts，img，js几个文件夹，全部上传到
/usr/share/nginx/html/
目录下面。上传方法前面已有介绍，打开Bitvise SSH，拖动到文件管理窗口即可上传。

接下来打开网址，这时候可以看到一个**真正的网站**。(提示：不同系统的Nginx欢迎页面可能不同，只要这里可以显示网页，就说明Nginx工作正常)



**5. 客户端配置**



**5.1 Windows客户端**


Windows客户端推荐V2RayN，V2RayN是开源软件，下载地址：
[https://github.com/2dust/v2rayN/releases](https://pincong.rocks/url/link/aHR0cHM6Ly9naXRodWIuY29tLzJkdXN0L3YycmF5Ti9yZWxlYXNlcw)

可以看到有一个v2rayN-Core.zip和一个v2rayN.zip，这里下载v2rayN-Core.zip（GUI界面+V2Ray内核）。

安装好V2RayN之后，如图
[![https://i.imgur.com/UP27H1F.png](https://i.imgur.com/UP27H1F.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9VUDI3SDFGLnBuZw)

【fig5.1.1】


点【服务器】按钮，选择【添加VMess】服务器。

- 地址：你的VPS的IP地址，这里我的IP是218.30.118.6。
- 端口：443
- 用户ID：就是2.1节中，V2Ray配置文件里的UUID，本文中是63c0042a-4a85-4d03-a488-3ba3aa002461
- 额外ID：0（保持默认值）
- 加密方式：随便选。
- 传输协议：选ws，即WebSocket
- 别名：随便填。
- 伪装类型：none（保持默认值）
- 伪装域名：绑定到服务器的域名，我的是www.hrw1rdzqa7c5a8u3ibkn.website
- 路径：即前面的随机字符串，注意前面必须要加上斜杠“/”，这里的例子是/mL7Gg8K
- 底层传输安全：选tls

配置完成后如下图:
[![https://i.imgur.com/rYmRD6S.png](https://i.imgur.com/rYmRD6S.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9yWW1SRDZTLnBuZw)

【fig5.1.2】





**5.2 Android客户端**


安卓客户端推荐V2RayNG，配置和V2RayN可以互通。下载地址：
[https://github.com/2dust/v2rayNG/releases](https://pincong.rocks/url/link/aHR0cHM6Ly9naXRodWIuY29tLzJkdXN0L3YycmF5TkcvcmVsZWFzZXM)

分享配置很简单，在Windows的V2RayN客户端里，点击服务器列表，勾选右边的“显示分享内容”，可以显示配置的二维码。安卓端选择“扫描二维码”导入配置即可。



**5.3 iOS客户端**


iOS客户端全部要收费，常用有Shadowrocket（小火箭）等。配置方法略。

其它平台客户端（Mac OS，Linux）可以查看V2Ray官网的客户端列表：[神一样的工具们](https://pincong.rocks/url/link/aHR0cHM6Ly92MnJheS5jb20vYXdlc29tZS90b29scy5odG1s)
**
到这一步结束，整个V2Ray翻墙的搭建就结束了。接下来是一些可选配置，可以加强你的服务器的隐蔽性和安全性。**



**可选配置1：使用CDN隐藏IP**


CDN相当于在服务器前又加了一层代理，墙只知道你的域名和CDN的IP，无法得知代理服务器的真实IP。如果伪装网站开启了DoH+ESNI，甚至连域名都可以隐藏。因此v2ray+ws+tls+web+CDN相当于事实上的双重代理，它的隐蔽性和安全性非常高。缺点是Cloudflare 会让访问延迟变高一些。除非遇到IP被墙，或者六四前后等墙加高等极端情况，如果平时翻墙很稳定，就没有必要打开CDN。

因为前面已经注册了Cloudflare解析，所以使用CDN非常简单，只要两步即可。

**(1)** 登录Cloudflare账号，点击【DNS】按钮，进入Cloudflare的管理页面，如图：
[![https://i.imgur.com/ihzyMBe.png](https://i.imgur.com/ihzyMBe.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS9paHp5TUJlLnBuZw)

【figb.1】


点一下灰色的云，让颜色变成橙色即可。

点击切换后，域名不会马上解析到CDN的地址，一般会有几分钟的延迟。可以ping一下你的服务器的域名，如果返回地址是CDN的IP，就说明切换完成。

**(2.1) 使用自动选择的CDN IP地址**
接下来配置客户端。客户端切换成CDN很简单，配置的其它地方不用改动，只要把地址一栏换成域名即可，如图
[![https://i.imgur.com/1MsBKA0.png](https://i.imgur.com/1MsBKA0.png)](https://pincong.rocks/url/img/aHR0cHM6Ly9pLmltZ3VyLmNvbS8xTXNCS0EwLnBuZw)

【figb.2】


手机端配置方法类似，把IP换成网址即可。

**(2.2) 手动指定CDN IP地址**
(2.1)节介绍的方法可以满足大多数人的需要，事实上要连接的CDN节点也可以手动指定。手动指定CDN节点地址，可以免费使用不同运行商的线路和不同国家的入口节点。如果自动分配的CDN节点速度不理想，可以尝试手动指定入口节点。

配置方法也非常简单：在第(1)节把云朵点成橙色之后，在V2RayN的【地址】一栏里，不填入网址，而是填入CDN的IP地址。
举例如下：

- 电信线路+美国旧金山节点：172.64.0.8
- 联通线路+日本节点：104.20.157.84
- 移动线路+新加坡节点：104.28.14.8

上述IP是属于Cloudflare的公共地址，所以不会被封。Cloudflare提供了非常多的地区+线路组合，具体可参加网友@marxist 的贴子：
《[关于国产输入法的隐私问题以及如何选择合适的cloudflare IP地址？](https://pincong.rocks/question/25479)》
原贴见：[https://ofvps.com/201907510](https://pincong.rocks/url/link/aHR0cHM6Ly9vZnZwcy5jb20vMjAxOTA3NTEw)



**可选配置2：加固服务器，配置防火墙**


如果VPS上没有其它服务，建议打开防火墙。服务器对外只暴露80，443，SSH端口，可以降低代理服务器被探测的风险。

前面的步骤中禁用了防火墙firewalld，不是所有的机器都安装了firewalld，我们这里使用ufw防火墙作为替代。

安装ufw：

```
yum install -y epel-release && yum install -y ufw
```


打开SSH，HTTP，HTTPS端口，运行：

```
ufw disable && ufw allow ssh && ufw allow http && ufw allow https && ufw enable
```


如果ssh端口不是22，那么需要将ssh改为端口号。例如ssh端口为14320，则：

```
ufw disable && ufw allow 14320 && ufw allow http && ufw allow https && ufw enable
```


ufw和firewalld的底层实现都是一样的，都调用了linux iptables，本质并无太大区别。



**可选配置3：使用BBR加速**


BBR是谷歌开发的拥塞控制算法，可以降低延迟，加快访问速度。启用BBR需要4.10以上版本Linux内核，现在大多数VPS都满足这一条件，输入uname(空格)-a可以查看内核版本.
如果内核版本大于4.10就可以用BBR了，把以下三条命令复制到命令窗口执行：

```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```


然后运行以下命令，查看BBR是否启动成功：

```
sysctl net.ipv4.tcp_congestion_control
```


如果提示

```
net.ipv4.tcp_congestion_control = bbr
```


就表示成功启动了BBR加速。



**可选配置4：编译Nginx**


***\*本节内容需要有一定Linux基础\****

某些系统上，通过yum安装的Nginx不支持TLS1.3，需要自行编译。启用TLS1.3可以明显降低VMess+WS+TLS的延迟（握手1-RTT，恢复会话0-RTT）。此外，TLS1.3第一个RTT之后的握手包均被加密，（可能）会降低TLS协议的指纹特征。

Caddy（另一个HTTP反向代理软件）也支持TLS1.3，但自行配置和编译的Nginx可以通过调整多种参数，达到更高的性能。自行编译Nginx也可以启用一些其它反向代理中的特征，例如HTTP/2等。

Nginx编译安装步骤：

更新所有软件及系统内核（用时较长，可选）：

```
yum -y update
```


安装依赖软件和库：

```
yum -y install wget gcc make perl pcre pcre-devel zlib zlib-devel
```


下载OpenSSL 1.1.1g（截至2020年4月21日的最新版）

```
wget https://github.com/openssl/openssl/archive/OpenSSL_1_1_1g.zip
unzip OpenSSL_1_1_1g.zip
rm OpenSSL_1_1_1g.zip && mv openssl-OpenSSL_1_1_1g openssl
```


下载Nginx 1.18.0（截至2020年4月21日的最新版）

```
wget https://nginx.org/download/nginx-1.18.0.tar.gz
tar -xzvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
```


配置编译选项

```
./configure --with-openssl=../openssl --with-openssl-opt='enable-tls1_3' --with-http_v2_module --with-http_ssl_module --with-http_gzip_static_module
```


这一步是Nginx启用TLS1.3的关键，--with-openssl-opt='enable-tls1_3'表示启用TLS1.3，--with-http_v2_module表示启用HTTP/2

编译&安装

```
make && make install
```


编译完成的Nginx二进制文件位置在/usr/local/nginx/sbin/nginx，可用以下命令进行测试：

```
/usr/local/nginx/sbin/nginx -V
```


与此对应的，Nginx配置文件目录和网页文件目录分别在：

```
/usr/local/nginx/conf
/usr/local/nginx/html
```


为了把Nginx配置成系统服务，还需要配置systemd文件：

```
[Unit]
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```


最后把上述文件命名为nginx.service，放在/etc/systemd/system下，就完成了Nginx的编译安装。

\--
到这里，整个V2Ray翻墙教程就结束了，过程总结：

1. 购买域名 & 配置域名解析
2. 安装Nginx和V2Ray
3. 上传配置文件


可选步骤：

- CDN隐藏IP
- 打开防火墙
- BBR加速
- 编译Nginx