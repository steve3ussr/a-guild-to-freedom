# A Guild to Freedom

价格, 设备数量 , 流量, 方便程度, 金融和数据安全, 售后服务: 这些要素不可能全部最优化. 

建议直接购买现成的服务, 但如果你满足以下几个要素: 

- 能负担得起每年300-500元的相关支出, 或者有人可以分摊成本
- 有较多的设备数量, 而不是仅仅使用一部手机
- 现成的服务中流量无法满足要求
- 由于经历过跑路, 不佳的售后服务或使用体验而不信任其他服务提供商
- 愿意自己动手解决任何问题, 自己来做售前和售后
- 有空闲时间
- 已经拥有, 或者愿意查阅相关的专业知识
- 担心数据泄露

那么这篇文章中的流程值得一试. 

## 0 Preparation



> List of service providers

- Xshell, XFtp: **software** for SSH and SFTP login to server

- Vultr: buy a VPS via **website**

- Cloudflare: provide a bunch of net services, access to it by **website**

- godaddy: buy and manage domain name through **website**

  

> [optional]
>
> - DNSPod, 现在归于腾讯云下, 提供国内的DNS加速
> - cloudflareST@github, 一个优化连接速度的项目仓库

- 在开始之前, 首先阅读文章最后一个章节, 最好是能阅读全文



## 1 Principle

一句话: 通过代理服务器来访问受限资源或者加速访问, 借助海外服务器来转发请求和资源. 

## 2 Deploy Server and Install with Primary Configure

> 可以不使用一键脚本, 带来方便的同时也会有一些麻烦

- [ ] 访问Vultr官网, 注册账号, 不必填写实名信息, 通过支付宝等方式充值
- [ ] [可选项: 启用2FA, 否则有被盗号的风险]
- [ ] 购买服务器(Deploy Server). 可以根据实际需要来选择, 目前有丰富的服务器形式可供选择; 其中最便宜的是`Cloud Compute -> Intel Regular Performance -> New York (NJ)`, 这是唯一一个提供 $3.5 per month 的地点: 前提是选择合适的操作系统, 推荐 Debian 11 **警告: 不应该购买仅支持IPv6的服务器**
- [ ] 服务器部署完成后, 在控制面板可以看到**IPv4地址, 系统账号密码**. 如果安装 Debian 的话默认用户名是 root
- [ ] 安装 XShell , 根据以上的三个信息SSH登录到服务器, 通常SSH是不会被限制的; 用个人电脑 ping 一下 IP, 能通即可, 丢包正常
- [ ] 如果上一步出现问题, 说明分配的IPv4地址在黑名单中, 需要**先部署新服务器, 再销毁旧服务器; 反顺序操作的话会分配相同的IPv4地址**
- [ ] 使用别人提供的一键Shell脚本安装: `bash <(curl -s -L https://git.io/v2ray.sh)`安装脚本写的很直白
  - [ ] 伪装协议部分可暂时选择TCP或者mKCP_BT下载等
  - [ ] 不建议选择动态端口
  - [ ] Shadowsocks可以不配置
- [ ] 获取服务端 url (`vmess://`)

## 3 Install Client

> `varayN`是由`2dust@github`开发的适用于 Windows 的客户端程序, 他还开发了适用于 Android 的 `v2rayNG`. 当然除此之外还有很多其他替代品, 以下流程仅适用于`varayN, v2rayNG`

- [ ] 右键添加`Vmess`服务器
- [ ] 在托盘将系统代理改为自动配置
- [ ] 尝试访问受限资源, 但**必须浅尝辄止**, 知道可以正常访问就足够了



## 4 Enhance Your Disguise

> 伪装成BT下载或者其它形式并没有用, 在强大的检测能力面前不匹配的协议头会被标记为可疑对象, 因此需要更安全的协议. 以下流程使用 Godaddy + Cloudflare

- [ ] 注册 godaddy 来购买域名, 也可在其他海外平台购买域名
- [ ] 选择喜欢的二级域名搭配便宜的一级域名(如`.site`). *提示: godaddy可能需要将币种和语言改为 English 才能正常使用*
- [ ] 购买两个不同的域名, 其中一个域名(不对外展示, 较为随意)通过 NS 接入 Cloudflare , 另一个域名(会对外展示, 不建议带有姓名等特征)通过 CNAME 接入. 以下将用这些名称来指代这两个域名: 
  - 非公开域名 / `protevted.com`
  - 公开域名     / `public.com` 
- [ ] *提示: 在 Godaddy 购买的域名默认接入 Godaddy 自己的 DNS 服务*
- [ ] 注册 Cloudflare 账号
- [ ] 将非公开域名通过NS方式接入Cloudflare, 需要按照提示将NS更换为Cloudflare提供的 *提示: 更换需要时间, 更换完成后会有邮件提醒, 可以不用一直查询状态*
- [ ] 管理**非公开域名**的DNS记录, 增加一条A记录, 指向服务器 IPv4地址, 如`int.protected.com: 200.200.200.200`, 代理模式为转发流量(橘色云朵)
- [ ] *提示: 在大多数时候主机名填写`int`即可, 如果填写`int.protected.com`则会变成`int.protected.com.protected.com`*
- [ ] 配置服务器 
  - [ ] 管理**公开域名**的DNS记录, 增加一条A记录, 指向服务器IPv4地址, 如`tmp.public.com`, 代理模式为仅DNS(如果在Cloudflare NS接入则应该选择灰色云朵) *提示: 这步操作没有必要, 只是一键脚本带来的麻烦*
  - [ ] SSH登录到服务器, 更改服务配置为`ws+tls`, 这一步强制要求将伪装域名(`tmp.public.com`)解析到服务器IPv4地址, 其他选项可以根据情况自行选择
  - [ ] 如果选择接入Cloudflare则应该在配置完成后切换为橙色云朵
- [ ]  管理**非公开域名**的SSL/TLS, 选择完全模式(不能严格), 并确认边缘证书中显示已经发放
- [ ] 管理**非公开域名**的SSL/TLS中的自定义主机名. 尽管Cloudflare的免费计划不提供CNAME接入, 但在这里可提供另一种形式的CNAME接入. 
  - [ ] 之前的`int.protected.com`在这里是回退源地址
  - [ ] 为回源地址添加一个主机名: 之前使用的`tmp.public.com`. 此时需要按照提示在**公开域名**的DNS管理处添加TXT记录
  - [ ] **耐心等待一段时间**, 如果刷新Cloudflare控制台页面可能导致证书验证值刷新

## 5 [Optional] Optimize Net Connection

> Cloudflare在中国没有服务器, 默认分配的Cloudflare IP可能性能不佳; 通过自选IP可最大程度地提升访问速度, 但也会带来困扰

- [ ] 将**公开域名**接入到国内支持多源解析的DNS服务提供商, 如DNSPod
- [ ] 如果不能成功同步原有的DNS记录, 则需要手动添加
- [ ] 利用`CloudflareST@github`提供的工具, 在不同网络环境(移动/联通/电信/教育网)下测试最佳服务器地址
- [ ] 添加多个A记录, 如对于电信选择`tmp.public.com -> 201.201.201.201`, 对于移动选择`tmp.public.com -> 202.202.202.202`

> 多源解析带来的问题是如果切换ISP(网络服务提供商), 比如从移动切换为联通, 由于DNS缓存未刷新可能导致无法连接. 对于 Android 可尝试: 
>
> - 重启设备
> - 在Chrome Dev中访问`chrome://net-internals/#DNS`清除DNS缓存
>
> 由于我没有所有类型的设备, 因此不清楚其他设备是否会有相同问题, 以及如何解决

## 6 Notes

- 如果不能正常访问, 我也不可能知道是哪一步的问题. 如果可以的话, 推荐先使用其他付费服务在受限资源上查询, 也许能获得更多帮助

- 单个域名的价格通常在$2左右

- Vultr服务器最便宜为$\$3.5\times12=\$42$, 有500GB/mo的流量

- 支付时的汇率价格通常比网上查到的贵一些. 国内而言支付宝比其他方式 (银联, 国区PayPal) 都要便宜

- 可能的话, 和其他人分摊成本

- 记得给服务器续费

- CPU, 内存和硬盘不太重要, 流量比较重要

- Vultr的每月似乎只有28天

- 目前Vultr的流量方式为每日增加以避免单日大流量, 例如每月2800GB流量并非月初一次性提供, 而是按照每日增加100GB的方式提供

- Cloudlfare的免费计划曾经是限流100GB/mo, 现在似乎是取消了限制, 但是薅羊毛也要按照基本的法去薅, 不建议过度薅羊毛

- 一键脚本中默认屏蔽了某教相关网站

- 对VPS供应商来说, 流量和带宽是大头. 国外的价格比国内好很多, 300元/年左右的国内小型VPS往往只有1-6Mbps的带宽, 相比之下国外的用网成本小很多

- 事物总是在变化的, 过去Vultr的$3.5服务器是1000GB/mo流量, Cloudflare可以通过Partner直接CNAME接入, Cloudflare免费计划流量限制为100GB/mo: 但现在不一样了

  
