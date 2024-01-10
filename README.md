这里有一个细节的操作：

```
https://i007it.com/2022/06/14/Linux%E4%BD%BF%E7%94%A8v2ray/


# 配置文件
/usr/local/etc/v2ray/config.json


# 启动V2ray
sudo systemctl start v2ray

# 检查V2ray状态
sudo systemctl status v2ray

# 设置V2ray开机自启动
sudo systemctl enable v2ray



```





因为在CentOS的服务器上装东西的，用到了github，直接访问不了。本地windows系统用的v2rayN的客户端，想到v2ray在Linux上也可以用，就装上试一下。

v2ray本身是不区分服务端和客户端的，只要配置好相关文件，反正都可正常使用。（就是配置文件的区别）

1.下载 v2ray-linux-64.zip
v2ray的Github地址：
https://github.com/v2ray/v2ray-core/releases/

目前最新的版本是v4.31.0，下面有Download页面：
https://github.com/v2fly/v2ray-core/releases/tag/v4.31.0

在页面中找到 v2ray-linux-64.zip 文件下载（我的是64位的CentOS系统）。

下载后解压出来是一个 v2ray-linux-64 目录，用ftp工具上传到linux的服务器上。

当然，也可以直接把解压包上传后，再用unzip命令解压。

2.把文件复制到对应的目录中
用复制(cp命令)或移动(mv命令)都可以。这里用cp举例。

首先，进入 v2ray-linux-64 目录，可以用 ls -l查看目录下的文件。
目录中的几个文件需要修改下权限，需要添加下可执行的权限。

cd v2ray-linux-64

chmod 755 v2ray
chmod 755 v2ctl
chmod 755 systemd/system/v2ray.service
chmod 755 systemd/system/v2ray@.service
然后复制目录中的文件到指定位置：

cp v2ray /usr/local/bin/
cp v2ctl /usr/local/bin/

cp systemd/system/v2ray.service /etc/systemd/system/
cp systemd/system/v2ray@.service /etc/systemd/system/

mkdir /usr/local/share/v2ray/
cp geoip.dat /usr/local/share/v2ray/
cp geosite.dat /usr/local/share/v2ray/

mkdir /var/log/v2ray/
cp access.log /var/log/v2ray/
cp error.log /var/log/v2ray/
两个日志文件没有的话，自己新建一个就行，要保证所有人都有读写权限。
反正配置文件中不用的话，其实也无所谓，就先建着扔着。

还有一个config.json配置文件，等配置完了再复制。

3.config.json配置文件
原生的V2ray并不支持订阅，反正我本来就在windows下用的，直接在v2rayN的客户端，服务器列表中中右键->【导出所选服务器为客户端配置】，保存成config.json文件。

然后把这个config.json文件也上传到 v2ray-linux-64 目录中，再来复制。

mkdir /usr/local/etc/v2ray/
cp config.json /usr/local/etc/v2ray/config.json
以下配置文件仅为参考（需将outbounds处settings中改成自己的）：

{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "tag": "socks",
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "allowTransparent": false
      }
    },
    {
      "tag": "http",
      "port": 1081,
      "listen": "127.0.0.1",
      "protocol": "http",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "udp": false,
        "allowTransparent": false
      }
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "abc.abc.net",
            "port": 30000,
            "users": [
              {
                "id": "effffff6-fffffb-4aaa-8888-aaaaaaaaaa",
                "alterId": 1,
                "email": "t@t.tt",
                "security": "chacha20-poly1305"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp"
      },
      "mux": {
        "enabled": true,
        "concurrency": 8
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "block",
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      }
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "domainMatcher": "linear",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "enabled": true
      },
      {
        "type": "field",
        "outboundTag": "proxy",
        "domain": [
          "geosite:google"
        ],
        "enabled": true
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "domain": [
          "domain:example-example.com",
          "domain:example-example2.com"
        ],
        "enabled": true
      },
      {
        "type": "field",
        "outboundTag": "block",
        "domain": [
          "geosite:category-ads-all"
        ],
        "enabled": true
      }
    ]
  }
}
我这里就没把log文件配置写进去，需要的话再写上：

"log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
...
记得这两个文件看一下，要有读写权限。

4.启动v2ray
# 启动V2ray
sudo systemctl start v2ray

# 检查V2ray状态
sudo systemctl status v2ray

# 设置V2ray开机自启动
sudo systemctl enable v2ray
V2ray状态：


5.检验代码是否生效
curl -x socks5://127.0.0.1:1080 https://www.google.com -v
如果能返回google.com的源代码，即表示配置成功。

本文标题：Linux使用v2ray
本文作者：HDUZN
创建时间：2022-06-14 21:10:35
本文链接：http://hduzn.cn/2022/06/14/Linux使用v2ray/
版权声明：本博客所有文章除特别声明外，均采用 BY-NC-SA 许可协议。转载请注明出处！


# fhs-install-v2ray

> 欲查阅以简体中文撰写的介绍，请访问：[README.zh-Hans-CN.md](README.zh-Hans-CN.md)

> Bash script for installing V2Ray in operating systems such as Debian / CentOS / Fedora / openSUSE that support systemd

該腳本安裝的文件符合 [Filesystem Hierarchy Standard (FHS)](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)：

```
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

## 重要提示

**不推薦在 docker 中使用本專案安裝 v2ray，請直接使用 [官方映象](https://github.com/v2fly/docker)。**  
如果官方映象不能滿足您自定義安裝的需要，請以**復刻並修改上游 dockerfile 的方式來實現**。  

本專案**不會為您自動生成配置檔案**；**只解決使用者安裝階段遇到的問題**。其他問題在這裡是無法得到幫助的。  
請在安裝完成後參閱 [文件](https://www.v2fly.org/) 瞭解配置檔案語法，並自己完成適合自己的配置檔案。過程中可參閱社群貢獻的 [配置檔案模板](https://github.com/v2fly/v2ray-examples)  
（**提請您注意這些模板複製下來以後是需要您自己修改調整的，不能直接使用**）

## 使用

* 該腳本在執行時會提供 `info` 和 `error` 等信息，請仔細閱讀。

### 安裝和更新 V2Ray

```
// 安裝執行檔和 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

### 安裝最新發行的 geoip.dat 和 geosite.dat

```
// 只更新 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 移除 V2Ray

```
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

### 解決問題

* 「[不安裝或更新 geoip.dat 和 geosite.dat](https://github.com/v2fly/fhs-install-v2ray/wiki/Do-not-install-or-update-geoip.dat-and-geosite.dat)」。
* 「[使用證書時權限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates)」。
* 「[從舊腳本遷移至此](https://github.com/v2fly/fhs-install-v2ray/wiki/Migrate-from-the-old-script-to-this)」。
* 「[將 .dat 文檔由 lib 目錄移動到 share 目錄](https://github.com/v2fly/fhs-install-v2ray/wiki/Move-.dat-files-from-lib-directory-to-share-directory)」。
* 「[使用 VLESS 協議](https://github.com/v2fly/fhs-install-v2ray/wiki/To-use-the-VLESS-protocol)」。

> 若您的問題沒有在上方列出，歡迎在 Issue 區提出。

**提問前請先閱讀 [Issue #63](https://github.com/v2fly/fhs-install-v2ray/issues/63)，否則可能無法得到解答並被鎖定。**

## 貢獻

請於 [develop](https://github.com/v2fly/fhs-install-v2ray/tree/develop) 分支進行，以避免對主分支造成破壞。

待確定無誤後，兩分支將進行合併。
