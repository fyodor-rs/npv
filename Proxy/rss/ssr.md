1.tool

2.系统centos7 64

```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

加速原版bbr

```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
sysctl net.ipv4.tcp_congestion_control //检测
```

常用命令行

```
启动SSR：
/etc/init.d/shadowsocks-r start
退出SSR：
/etc/init.d/shadowsocks-r stop
重启SSR：
/etc/init.d/shadowsocks-r restart
SSR状态：
/etc/init.d/shadowsocks-r status
卸载SSR：
./shadowsocks-all.sh uninstall
```

```

Your Server IP        :  144.202.19.239
Your Server Port      :  18062
Your Password         :  3531520
Your Protocol         :  auth_aes128_sha1
Your obfs             :  http_simple
Your Encryption Method:  aes-256-cfb


```

