# RackNerd 配置无从下手？从套餐选购到服务器环境搭建全流程，含防火墙、BBR 加速、LNMP 部署保姆级教程

上周朋友想搞个自己的小站，买了一台 RackNerd 的特价 VPS，结果开通之后对着那个黑乎乎的终端发呆，连 root 密码都不知道在哪改。其实 RackNerd 配置这件事拆开来看一点都不难，难的是网上教程要么只讲买、要么只讲搭，中间那一段"开通之后到底先干嘛"反而没人说清楚。这篇就把从选套餐到把环境跑起来的全过程一次讲完，看完直接能动手。

## RackNerd 配置前先想清楚：你这台 VPS 要干什么

说句实在话，很多人买 RackNerd 是被 $10 出头的年付特价吸引进来的，买完才想"我拿它干嘛"。配置方向不同，套餐选法和后续操作差很多。RackNerd 是一家主打低价高配的 KVM VPS 商家，全 SSD RAID-10 存储、1Gbps 端口、20 个机房可选，开通过去基本是分钟级的，定位就是"花小钱拿到一台完全自己说了算的 Linux 服务器"。

先分一下场景：

- 只是想跑个代理或者做点轻量转发，1GB RAM 那档就够
- 要搭 WordPress 或一个小站，2GB 起步比较踏实
- 跑多个服务、数据库一起上，4GB 以上再考虑
- 纯练手、学习 Linux，最便宜那档随便玩

把自己对号入座之后再往下看，别一上来就买最大那档，用不上就是浪费。

## RackNerd 套餐怎么选：全档位配置一览

下面这张表是 RackNerd 官网 KVM VPS 全部在售套餐，配置和价格直接照搬官网定价页，没做任何加工。年付的 512MB 那档是 RackNerd 长期的招牌低价款，月付套餐则适合不想一次掏太多、想先用用看的人。

| 套餐 | CPU | 内存 | SSD 存储 | 月流量 | IPv4 | 价格 | 计费周期 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 512MB RAM | 1 vCore | 512 MB | 30 GB RAID-10 | 500 GB @ 1Gbps | 1 个 | $26.99 | 年付 |
| 1GB RAM | 2 vCore | 1 GB | 50 GB RAID-10 | 1 TB @ 1Gbps | 1 个 | $17.99 | 月付 |
| 2GB RAM | 3 vCore | 2 GB | 75 GB RAID-10 | 2 TB @ 1Gbps | 1 个 | $20.59 | 月付 |
| 4GB RAM | 4 vCore | 4 GB | 130 GB RAID-10 | 3 TB @ 1Gbps | 1 个 | $24.59 | 月付 |
| 6GB RAM | 5 vCore | 6 GB | 170 GB RAID-10 | 4 TB @ 1Gbps | 1 个 | $27.59 | 月付 |
| 8GB RAM | 6 vCore | 8 GB | 220 GB RAID-10 | 5 TB @ 1Gbps | 1 个 | $36.59 | 月付 |
| 12GB RAM | 7 vCore | 12 GB | 300 GB RAID-10 | 6 TB @ 1Gbps | 1 个 | $55.99 | 月付 |

讲真，从 1GB 到 4GB 这一档每个月只多几美元，配置差距却很实在——多出来的内存和 CPU 在跑数据库或者并发稍微一上来的时候就顶得住。我自己用下来最顺手的是 2GB 那档，搭个小站外加跑点脚本，资源占用常年不到一半。

如果你看完还是拿不准，👉 [对比 RackNerd 全部套餐，挑最适合你的那一档](https://bit.ly/RacKnerd) 直接进官方套餐页对着配置选，比纠结半天强。

顺带提一嘴，RackNerd 不定期会放出年付特价款，价格比常规月付划算不少，碰到合适的可以直接下手，不用等所谓的"最低价"——这种低价商家的促销节奏没什么规律，等来等去可能就没了。

## 从零开始配置 RackNerd：完整操作流程

下面这一段是整篇文章的核心，按顺序走一遍，开通到能用大概半小时。

### 第一步：下单并选机房

进 👉 [RackNerd 套餐选购页](https://bit.ly/RacKnerd) 选好套餐，点 Order Now 进配置页。这里有几个选项要选：

- **机房位置**：亚洲用户首选洛杉矶或圣何塞，延迟相对低；欧洲用户选阿姆斯特丹、伦敦、都柏林；美东选纽约、阿什本、亚特兰大
- **操作系统**：新手选 Ubuntu 22.04 或 Debian 12，资料最多；要跑老软件可以选 CentOS 替代品 AlmaLinux；想省心用宝塔面板就选 CentOS 7.9 或 Debian
- **计费周期**：年付单价最低，月付灵活适合试用

选完点 Continue 进结账页，填账号信息、选支付方式（支持 PayPal、信用卡、加密货币、银行转账），同意条款提交即可。

开通基本是秒级的。订单提交后通常 5 到 15 分钟内你会收到两封邮件：一封订单确认，一封标题是 **"KVM VPS Login Information"**，里面有 IP、root 密码、SolusVM 控制面板地址和登录账号。这封邮件要存好，后面要用。

### 第二步：首次 SSH 登录

拿到 IP 和 root 密码之后，Mac/Linux 直接开终端，Windows 用 Bitvise、Xshell、PuTTY 都行，或者新版的 Windows Terminal 也自带 OpenSSH。

bash
ssh root@你的服务器IP


第一次连接会问 yes/no，输 yes。然后粘密码——这里有个坑，粘贴的时候终端不会显示任何字符，不是没粘上，粘完直接回车就行。

进系统第一件事不是装东西，是改密码。

bash
passwd


输入两次新密码。这一步很多人跳过，但其实 root 密码是邮件明文发的，改掉心里踏实。

### 第三步：更新系统

新开的 VPS 系统包都是出厂状态，先把所有软件更新到最新。

Ubuntu/Debian 系：

bash
apt update && apt upgrade -y


CentOS/AlmaLinux 系：

bash
dnf update -y


更新完重启一次，让新内核生效。

bash
reboot


### 第四步：开启 BBR 提升网络性能

BBR 是 Google 开发的 TCP 拥塞控制算法，对跨境网络尤其有用，开启之后实际下载速度和稳定性都会有明显改善。RackNerd 默认装的系统内核比较老，多数情况需要手动开。

检查当前是否已经开了 BBR：

bash
sysctl net.ipv4.tcp_congestion_control


输出里如果已经有 `bbr` 就不用动了。没有的话，编辑 sysctl 配置：

bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p


再验证一次：

bash
sysctl net.ipv4.tcp_congestion_control


看到 `bbr` 就成了。这一步我自己在洛杉矶机房实测下来，从国内拉文件速度从原来的十几 MB/s 稳到 30MB/s 上下，不算玄学。

### 第五步：配置防火墙

这一步是新手最容易翻车的地方——装好 Nginx 发现外网访问不了，十有八九就是防火墙没放行。

**Ubuntu 用 ufw：**

bash
ufw allow 22/tcp      # SSH
ufw allow 80/tcp      # HTTP
ufw allow 443/tcp     # HTTPS
ufw enable
ufw status


**CentOS/AlmaLinux 用 firewalld：**

bash
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
firewall-cmd --list-all


顺手把 SSH 端口改一下，能挡掉一大半自动扫描的爆破脚本。编辑 `/etc/ssh/sshd_config`，找到 `#Port 22` 改成比如 `Port 22022`，保存后重启 sshd，记得防火墙也要放行新端口。

bash
systemctl restart sshd


改完先别关当前 SSH 会话，新开一个窗口用新端口连一下确认能上，再关老的，不然就把自己锁外面了。

### 第六步：搭建 Web 环境

到这一步服务器已经是个干净的、能用的、网络也优化过的状态。接下来看你跑什么。

**纯命令行党用 LNMP 一键包：**

bash
wget http://soft.vpser.net/lnmp/lnmp1.9.tar.gz -cO lnmp1.9.tar.gz
tar zxf lnmp1.9.tar.gz
cd lnmp1.9
./install.sh lnmp


按提示选 Nginx、MySQL、PHP 版本，等大概 20 到 40 分钟装完。装完默认首页访问 `http://你的IP` 就能看到 LNMP 探针页。

**想省心用宝塔面板：**

bash
curl -sSO http://download.bt.cn/install/install_panel.sh
bash install_panel.sh


装完会给你面板地址、用户名、密码，浏览器进去点点点就行，建站、装数据库、配 SSL 全图形化。宝塔对新手特别友好，代价是面板本身会吃一点资源，1GB 内存那档勉强能跑，2GB 以上才舒服。

## RackNerd 配置中的几个常见坑

讲真，RackNerd 这家在同价位里算稳的，但用的人多了坑也集中。提前知道能少踩几个。

**80 端口访问不了**：九成是防火墙没开。按第五步的方法放行 80 和 443 就行。还有一成是 Web 服务没起来，`systemctl status nginx` 看一下状态。

**SSH 连不上**：要么 IP 被你本地网络屏蔽了，要么端口改了忘了放行防火墙，要么服务器被你 reboot 卡住了。RackNerd 后台有个 VNC 控制面板能直接进系统，紧急情况下用那个。

**流量跑超**：RackNerd 流量是计费的不是限速的，超了会按 GB 收费。低配套餐一个月 500GB 对小站够用，跑下载站或者大流量代理的话提前算清楚。

**机房选错**：亚洲用户选了纽约机房，延迟直接 200ms 起步。买之前对着自己主要用户群的位置选，洛杉矶和圣何塞对国内相对友好。

**续费涨价**：特价款续费多半会涨，买之前看清楚是首年价还是常态价。RackNerd 续费不会涨得离谱，但也不会一直是首发那个价。

## 关于 RackNerd 这家值不值得入

我自己用过 RackNerd 几年，最直观的几个感受：

开通快，基本是分钟级，没遇到过等半天的情况。工单响应在同价位里算快的，半夜发的工单二十几分钟有回复，没碰到过推脱。SolusVM 控制面板能直接重启、重装、装系统、看流量，自助操作覆盖面挺广，IP 被封还能自助换 IP，这点很多家没有。退款方面官方有 3 天退款政策，不满意可以退，不过特价款的具体条款下单前看一眼服务协议更稳妥。

配置给的算大方，同价位对比下来 SSD 容量、流量、端口速度都不算抠门。20 个机房可选也是它的一个优势，想换地方比一些只有两三个机房的商家灵活太多。

如果你是第一次自己折腾 VPS，RackNerd 是个不错的练手对象——便宜，错了也不心疼，配置过程能让你把 Linux 服务器那套流程走通一遍，后面换别家也都会了。

👉 [前往 RackNerd 查看当前所有套餐与最新特价](https://bit.ly/RacKnerd)

## 套餐快速选购指引

下面这一段是给赶时间的人准备的，对着需求直接挑。

- **学生党练手 / 跑轻量脚本 / 单一代理服务**：512MB 年付那档，$26.99 一年，性价比天花板
  👉 [选择 512MB 年付方案](https://my.racknerd.com/cart.php?a=add&pid=1&aff=11397)
- **个人小站 / WordPress / 一个小项目**：2GB 月付那档，$20.59/月，跑 WP 不卡
  👉 [选择 2GB 月付方案](https://my.racknerd.com/cart.php?a=add&pid=21&aff=11397)
- **多站点 / 跑数据库 / 中等并发**：4GB 月付那档，$24.59/月，留足余量
  👉 [选择 4GB 月付方案](https://my.racknerd.com/cart.php?a=add&pid=22&aff=11397)
- **重负载应用 / 多用户服务**：8GB 月付那档，$36.59/月，跑得动大部分东西
  👉 [选择 8GB 月付方案](https://my.racknerd.com/cart.php?a=add&pid=24&aff=11397)

中间档位的 1GB、6GB、12GB 没列在上面不是不好，是这两个边界档之间大多人选这两头。要全档位对比回上面那张表。

## RackNerd 配置常见问题

**Q：RackNerd 支持支付宝付款吗？**

官方支付方式是 PayPal、信用卡、加密货币、银行转账，没列支付宝。但 PayPal 可以绑国内信用卡或者用余额付，算是最接近的方案。一些第三方代付也能走，但那不是 RackNerd 官方渠道，自己掂量。

**Q：开通之后能换机房吗？**

不能直接一键换机房。RackNerd 的机房在订单时选定，之后想换得提工单让客服协助，多数情况是要新开一台再迁移数据，不是免费的。下单前想清楚机房选哪个。

**Q：VPS 装完系统能再换吗？**

能。SolusVM 控制面板里有 Reinstall 选项，选好新系统点一下就行，几分钟重装完，原来数据全没。想换发行版（比如从 Ubuntu 换 Debian）也是这个路子。

**Q：BBR 开了之后网速真的会快吗？**

要看你的使用场景。跨境传输、长距离连接提升明显，本地同机房基本无感。我自己在洛杉矶机房从国内拉文件，开了 BBR 之后稳定性改善比绝对速度改善更明显——以前偶尔掉到几 MB/s，开了之后基本稳在 20MB/s 以上。

**Q：特价套餐和常规套餐有什么区别？**

特价套餐通常是年付锁价，配置相对固定，价格低但续费会涨回常规价。常规月付套餐配置档位更全、续费价格稳定，灵活度高。练手用特价，长期用选常规。

**Q：RackNerd 适合跑什么，不适合跑什么？**

适合：个人站、博客、轻量应用、代理、学习练手、小型数据库。不适合：高并发大流量业务、对 SLA 要求极高的生产环境、需要 GPU 的场景。它的定位就是便宜够用，别指望它干专业服务器干的活。

## 最后几句

RackNerd 配置这件事，看完上面的流程其实就那么几步：选套餐、下单、SSH 登录、改密码、更新系统、开 BBR、配防火墙、装环境。每一步单独看都不难，串起来半小时能搞定。难的是第一次做的时候每一步都要查，所以这篇把每一步的命令和坑都写在一起，照着敲就行。

如果你已经有明确需求，直接对着上面的选购指引选套餐走流程；如果还在犹豫，先从最便宜的 512MB 年付那档起步，$26.99 一年试错成本极低，用不顺手退了也不亏。

👉 [前往 RackNerd 开始你的第一台 VPS 配置](https://bit.ly/RacKnerd)
