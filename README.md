## 为何要自建梯子

我试过好几个机场提供商，都有网络慢的情况，不怎么稳定，通过自建的梯子就比较稳定。

## 重要提醒

请确保符合当地法律，仅供学习、研究用途，不要发表不当言论。
## 选择 VPS 提供商

创建梯子首先你要有海外的服务器，选择合适的 VPS 提供商非常重要，它决定了网络的好坏。尽量选择大厂的提供商，或者稳定已经经营数年的提供商，小厂容易跑路，你可以选择 BandwagonHost、Vultr 这些提供商（不过听说 Vultr 这些提供商因为国人扎堆多，很多 IP 被封了），我选择的是荧光云，用的是在美国硅谷的服务器，线路是双向 CN2 GIA，我用起来速度还不错，看视频还是可以的，价格一个月49 。  

选择服务器时有几个要注意的地方：
- **服务器地区**：香港、日本、新加坡这些地方虽然离国内近，但是服务器价格高，CN2 线路一个月就要上百，如果你不在意延迟可以选择美国的服务器，价格便宜，几十块搞定，看视频是没问题的，油管 4k 60挺流畅，因为延迟网站访问速度会慢一点，对于我来说影响不大。
- **线路**：如果你追求网络的稳定和速度，线路一定要是 CN2 GIA，CN2 线路是中国电信推出的高质量互联网专线网络，相比于普通国际线路网络更稳定，如果你选择的是普通的国际线路，在高峰时段可能会出现网络拥堵，访问慢的问题，当然 CN2 线路价格会比较高
- **服务器测试**：在购买之前一定要选择可以测试延迟的服务器，看服务器访问网站的延迟如何，如果延迟过高或者丢包严重不要买
- **可更换IP**：尽量选择可更换公网 IP 的 VPS 提供商，最好可以无限次免费更换，因为自建梯子 IP 被封是可能存在的
- **按小时付费**：推荐找有这种模式的提供商，如果发现不好用可以退订
- **网络带宽**：这个要看清楚，可不要选到只有 5M 的带宽
- **服务器配置**：选最低的来，作为网络中转足够了

## 服务器部署

购买服务器后，一定要先测试服务器的公网 ip 是否能正常访问，如果访问不了说明被墙了，需要更换 ip。
服务器的系统选择 linux 即可，我选择的是 ubuntu

### Xray 部署

要使用服务器进行网络转发，则需要在服务器部署 xray，它可以在你的电脑的客户端上连接上服务器，进行网络转发。  
可以部署 x-ui，使用一键脚本进行安装，在服务器运行：

```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

接下来它会自动安装，它会让你设置 x-ui 的用户名和密码，提示要安装 HTTP 证书时，选择使用 IP ，输入服务器的公网 IP 地址。  
安装完之后会启动服务，你可以在控制台的输出中找到 x-ui 系统的访问地址

### 配置入站

用你的电脑的浏览器访问 x-ui 的地址，登录后在入站列表中添加入站，入站配置如下：
- 端口：⚠️ 一定要选择 443，选择其它端口可能会被 GFW 干扰
- 协议：选择 vless
- 传输：选择 TCP
- 安全：选择 Reality ，这个很重要，它可以把你的访问流量伪装成特定网站的流量，从而避免你的梯子被封，它是现在最有效的防封手段。公钥和密钥可以通过下面的按钮生成
其它地方不用管了，可以点创建了。 

### 配置 BBR

服务器一定要配置 BBR，否则你使用梯子的网速可能会非常的慢，没有 BBR，网络出现丢包时就会降低网速，导致你的带宽跑不满，BBR 可以在丢包发生时也能最大化利用带宽。  
在服务器依次执行：

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

## 客户端安装

你需要在你的电脑上安装客户端，才可以连接到服务器，可以使用 clash verge。
github 链接：[clash-verge-rev/clash-verge-rev: A modern GUI client based on Tauri, designed to run in Windows, macOS and Linux for tailored proxy experience](https://github.com/clash-verge-rev/clash-verge-rev)

如果你想使用其它客户端，这个仓库列出了所有的 clash 客户端：
[clashbk/clash: Clash官网各版本Clash下载地址及备份下载地址](https://github.com/clashbk/clash)

安装完成之后，在 clash verge 的订阅中导入订阅，订阅链接可以在 x-ui 的入站中获取，在入站列表中，在菜单选择导出链接-订阅设置，复制里面的链接使用浏览器打开，如果页面没有加载需耐心等待，这个页面会显示二维码，选择右边的 Clash 二维码，点击二维码就能复制里面的链接，使用此链接作为订阅地址导入到 Clash Verge 中。

![348](./files/Pasted%20image%2020260506184533.png)

到这里你已经全部完成了，你可以在客户端打开系统代理，测试网站是否能正常访问
## 非 443 端口被 GFW 干扰的风险

REALITY 协议虽然是目前抗检测能力最强的方案，但需要注意端口选择：

- **443 端口**：REALITY 伪装效果最好，因为流量看起来就是普通的 HTTPS，GFW 难以区分
- **非 443 端口**（如 11847、11848）：GFW 会重点检测非标准端口上的 TLS 流量，一旦识别出代理特征就会主动干扰，表现为**连接间歇性超时、握手失败**

即使换了端口号，只要服务器 IP 已被 GFW 标记，代理仍然会被持续干扰。`nc -zv` 测试能通（TCP 三层连通），但 `xray` 的 REALITY 握手会被干扰导致失败——因为 TCP 空连接不触发 DPI 检测，而有协议数据的 TLS 握手才会触发。

## 方案二：VLESS + TCP + TLS

当 REALITY 被干扰后，可改用 VLESS over TCP + TLS，使用标准 Let's Encrypt 证书：

### 服务端配置

在服务器创建一个独立的 xray 配置（不经过 x-ui 管理）：

```json
{
  "log": {"loglevel": "warning"},
  "inbounds": [{
    "port": 8443,
    "listen": "0.0.0.0",
    "protocol": "vless",
    "settings": {
      "clients": [{"id": "你的UUID", "email": "用户标识"}],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "tls",
      "tlsSettings": {
        "certificates": [{
          "certificateFile": "/etc/letsencrypt/live/你的域名/fullchain.pem",
          "keyFile": "/etc/letsencrypt/live/你的域名/privkey.pem"
        }]
      }
    },
    "sniffing": {"enabled": true, "destOverride": ["http", "tls"]}
  }],
  "outbounds": [
    {"protocol": "freedom", "tag": "direct"},
    {"protocol": "blackhole", "tag": "blocked"}
  ]
}
```

启动（后台运行）：
```bash
nohup xray -c xray-tls-tcp.json > xray.log 2>&1 &
```

### 客户端配置（Clash Verge）

在 Clash Verge 客户端打开设置页面，找到 Verge 高级设置，选择打开配置目录，打开 `profiles.yaml` 文件通过 `current` 属性找到当前的配置文件的名称，根据此名称在  `profiles`  目录内找到配置文件，例如 `profiles/[配置文件名称].yaml`，在配置文件手动添加节点：

```yaml
- name: 中转-TLS
  type: vless
  server: 服务器IP          # 不要用域名！DNS 可能解析到错误 IP
  port: 8443
  uuid: 你的UUID
  network: tcp
  tls: true
  servername: 你的域名       # SNI 用于 TLS 证书验证
  udp: true
```

**注意**：`server` 字段应直接填 IP 地址而非域名，避免 DNS 污染/劫持导致连接到错误的服务器。

## 443 端口 + Nginx SNI 分流

如果当前服务器有网站占用了 443 端口，要让网站和代理共用同一个 443 端口，需配置 nginx stream 模块根据 SNI 分流。

### 方案原理

```
用户请求 →
  SNI = xxx.com     → nginx HTTP 处理 →网站后端
  SNI = 其他或 IP 直连   → nginx stream    → xray:8443
```

### Nginx 配置

在 `/etc/nginx/nginx.conf` 中添加 stream 块（在 http 块之外）：

```nginx
stream {
    map $ssl_preread_server_name $backend {
        yaolilin.com        web;
        www.yaolilin.com    web;
        default             xray;
    }

    upstream web {
        server 127.0.0.1:443;  # 转发回 nginx 自己的 HTTP 处理
    }

    upstream xray {
        server 127.0.0.1:8443;
    }

    server {
        listen 443;
        ssl_preread on;
        proxy_pass $backend;
    }
}
```

HTTP 服务器块保持原有 `listen 443 ssl` 配置不变。

此方案的优势：
- 所有代理流量都走 443 端口，GFW 无法区分
- 网站正常访问不受影响
- 无需额外开放端口

