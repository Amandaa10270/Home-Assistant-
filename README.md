# 🏠 我的智能家居控制系统

基于 [Home Assistant](https://www.home-assistant.io/)（免费、开源、本地运行），统一控制 **扫地机器人、监控摄像头、暖气**，在手机上随时随地查看和操作。

| 暖气 | 扫地机器人 |
|---|---|
| <img src="docs/heating.png" width="260"> | <img src="docs/vacuum.png" width="260"> |

## 功能一览

**手机仪表盘（5 个页面）**
- **首页**：家庭模式、暖气温度、扫地机状态、书桌摄像头画面、快捷按钮
- **监控**：实时画面、移动侦测记录、报警开关
- **暖气**：温控旋钮、舒适/睡眠/节能温度设置、24 小时温度曲线
- **扫地机**：开始 / 暂停 / 停止 / 回充 / 寻找、上次清扫时间、清扫记录
- **设置**：起床 / 睡觉时间、各项自动化总开关

**自动化（不用手动操作）**

| 场景 | 自动执行 |
|---|---|
| 全家都出门了 | 切换到「离家」→ 暖气降到节能温度 → 扫地机开始清扫 → 监控开始报警 |
| 有人回家 | 切换到「在家」→ 暖气恢复舒适温度 → 正在清扫的扫地机自动回充 |
| 回家路上（距家 5 公里内） | 暖气提前预热，并推送通知 |
| 到睡觉时间 | 「睡眠」模式 → 暖气调到睡眠温度，监控开启夜间报警 |
| 离家 / 睡眠时摄像头检测到移动 | 截图并推送到手机，点开可看实时画面 |
| 扫地机清扫完成 / 被卡住 | 推送手机通知 |
| 室温低于 12°C | 推送防冻提醒 |
| 每天凌晨 | 自动清理 7 天前的监控截图 |

> 以上逻辑已在 Home Assistant 2026.2 上实际运行测试通过。

---

## 安装步骤

### 第 1 步：用闲置电脑当主机
你的旧电脑就可以当 Home Assistant 主机，24 小时开着，不用另买设备。**笔记本最好**：省电，自带电池，停电也不会立刻关机。

**两种装法，选一种：**

| | 方式 A：整机装 Home Assistant OS（推荐） | 方式 B：保留 Windows，用虚拟机 |
|---|---|---|
| 适合 | 这台电脑以后专门用来跑 HA | 电脑里还有文件或程序要保留 |
| 优点 | 最稳定，加载项、备份、升级都最省心 | 不用清空电脑 |
| 缺点 | ⚠️ **会清空整个硬盘** | 更占资源；Windows 自动更新重启时 HA 会停 |

> ⚠️ 选方式 A 之前，**先把电脑里要留的照片和文件备份到 U 盘或网盘**。

### 第 2 步：安装 Home Assistant

**方式 A：整机安装（官方教程：[Generic x86-64](https://www.home-assistant.io/installation/generic-x86-64)）**
1. 准备一个 **8GB 以上的 U 盘**
2. 在另一台电脑上，用 [Balena Etcher](https://etcher.balena.io/) 把 **Ubuntu Live** 写进 U 盘
3. 旧电脑插上 U 盘开机，按 F12、F2 或 Esc（不同品牌按键不同）选择从 U 盘启动，进入 Ubuntu 的「试用」模式
4. 在 Ubuntu 里打开浏览器，下载 **haos_generic-x86-64** 镜像，再用自带的「磁盘」工具把它恢复到电脑硬盘
5. 关机，拔掉 U 盘，**插上网线**（第一次安装建议用网线），开机
6. 几分钟后，在手机或其他电脑的浏览器里打开 `http://homeassistant.local:8123`

> 如果电脑开机后找不到系统：进 BIOS，打开 **UEFI** 启动，关闭 **Secure Boot**。

**方式 B：Windows 虚拟机（官方教程：[Windows](https://www.home-assistant.io/installation/windows)）**
1. 安装 [VirtualBox](https://www.virtualbox.org/)
2. 下载官方的 **VirtualBox 镜像（.vdi）**，新建虚拟机：类型选 Linux / Other 64-bit，2GB 以上内存，打开 **EFI**
3. 网卡设成 **桥接网卡**，否则找不到家里的设备
4. 把 Windows 电源设置改成**永不睡眠**，并让虚拟机随开机自动启动

**装好之后，把本仓库的配置放进去：**
1. 在 HA 里打开 **设置 → 加载项**，安装 **Samba share** 或 **Studio Code Server**
2. 把本仓库 `config/` 里的文件复制到 HA 的 `config` 文件夹，原来的同名文件直接覆盖
3. **开发者工具 → YAML → 检查配置**，没问题就重启

> 电脑设置：在 BIOS 里打开「**通电自动开机**」（Restore on AC Power Loss），停电恢复后会自动开机。笔记本可以合上盖子用，但要设置「合盖不睡眠」（方式 B）。

### 第 3 步：接入你的设备

#### 🧹 Narwal Freo X Ultra 扫地机
Freo X Ultra **没有局域网接口**，只能通过云鲸的云端连接。可用的是社区集成 [Behike/narwal-integration](https://github.com/Behike/narwal-integration)，作者就是在 Freo X Ultra 上开发测试的。
1. 先装 **HACS**（HA 的社区插件商店）：按 [HACS 官方教程](https://hacs.xyz/docs/use/) 安装并重启
2. HACS → 右上角 ⋮ → **自定义存储库** → 填 `https://github.com/Behike/narwal-integration`，类别选 **Integration**
3. 在 HACS 里搜索 **Narwal**，安装后重启 HA
4. **设置 → 设备与服务 → 添加集成 → Narwal**，填云鲸 App 的**邮箱、密码**，地区选 **US**
5. 如果要求填 **Product Key**，Freo X Ultra 的是 `EHf6cRNRGT`

接入后，把实体 ID 改成 **`vacuum.narwal`**（扫地机）和 **`sensor.narwal_battery`**（电量）。你在云鲸 App 里给扫地机起名叫 "Home assistant"，所以默认 ID 可能是 `vacuum.home_assistant` 之类，记得改。

> ⚠️ **说实话，这一步可能会卡住**：
> - 这个集成还在开发中，用的人很少，云鲸更新固件或云端后可能失效
> - 有的版本（如 [quittung 的分支](https://github.com/quittung/narwal-integration)）要求填 32 位的「Device Name」，这个只能从 App 的网络数据里抓，普通用户很难拿到。如果遇到这一步，把界面截图发给我
> - 本地集成（Narwal Local、Narwal Flow 等）**不支持 Freo X Ultra**，不用试
>
> 就算扫地机暂时接不上，暖气和监控也照常工作，手机页面里扫地机那一块会显示「不可用」。

#### 🔥 Google Nest 温控器
用 HA 官方的 **Google Nest** 集成。Google 要求先完成一次性设置，大约 20 分钟：
1. 在 [Google Device Access](https://console.nest.google.com/device-access) 注册，需要一次性支付 **5 美元**
2. 在 Google Cloud 创建 OAuth 凭据
3. 在 HA 里添加 **Google Nest** 集成，跟着向导走（每一步都有说明）

详细图文步骤见 [官方文档](https://www.home-assistant.io/integrations/nest/)。

接入后，把温控器的实体 ID 改成 **`climate.heating`**（方法见第 4 步）。

> 夏天 Nest 处于制冷或自动冷暖模式时，自动调温会自己跳过，不会误开暖气。

#### 📹 CareCamPro 摄像头
CareCamPro 是白牌摄像头，没有专门的集成，但大多支持 **ONVIF / RTSP** 标准协议：
1. 在 CareCamPro App 里查看摄像头的 **IP 地址**，在路由器里给它固定 IP
2. HA 添加 **ONVIF** 集成，填：
   - 主机：摄像头 IP
   - 端口：先试 **6688**，不行再试 **8899**、**80**
   - 用户名：`admin`
   - 密码：你在 App 里设置的设备密码。网上流传的出厂默认密码是 `admin123456`
3. 成功后会生成摄像头画面；如果摄像头支持，还会生成**移动侦测**传感器
4. ONVIF 连不上的话，改用 **Generic Camera** 集成，填 RTSP 地址试试（不同批次路径不一样）：
   - `rtsp://admin:密码@IP:554/onvif1`
   - `rtsp://admin:密码@IP:8554/profile0`

   可以先用电脑上的 VLC「打开网络串流」验证哪个能播放

> 🔒 **安全提醒**：这类便宜摄像头默认密码是公开的，一定要改掉。

> 以上端口和路径来自网友经验，不同批次的摄像头可能不同。卡住了把报错发给我。

#### 🔌 Kasa EP10 智能插座
1. 先用 **Kasa App** 把插座连上家里 Wi-Fi。注意它**只支持 2.4GHz**，如果 Spectrum 路由器 2.4G 和 5G 用的是同一个名字，一般也能连上
2. HA 通常会在 **设置 → 设备与服务 → 已发现** 里自动找到 **TP-Link Smart Home**，点「配置」即可。较新的固件会要求填 Kasa / TP-Link 账号密码
3. 把两个插座的实体 ID 改成 `switch.plug_1`、`switch.plug_2`

> EP10 没有电量统计功能。想看耗电量，可以选带电量统计的型号，比如 Kasa EP25。

### 第 4 步：统一实体 ID ⭐
进入 **设置 → 设备与服务 → 实体**，点开对应设备 → ⚙️ → 修改「实体 ID」：

| 改成这个 ID | 是什么 |
|---|---|
| `vacuum.narwal` | Narwal 扫地机 |
| `sensor.narwal_battery` | 扫地机电量 |
| `climate.heating` | Nest 温控器 |
| `camera.desk` | 书桌摄像头的画面 |
| `binary_sensor.desk_motion` | 书桌摄像头的移动侦测 |
| `switch.plug_1` / `switch.plug_2` | 两个 Kasa 插座 |

> 以后加摄像头？告诉我，我帮你改 `cameras.yaml` 和手机页面。
> 摄像头没有移动侦测传感器的话，实时画面照常能看，只是没有「有人移动」推送。

### 第 4.5 步：闲置平板 → 墙上中控屏 📱
本仓库自带一个横屏中控页面，一屏就能看到家庭模式、天气、暖气、摄像头、扫地机和插座：

<img src="docs/tablet.png" width="600">

**iPad：**
1. 安装 **Home Assistant** App，登录后在侧边栏打开 **「中控屏」**
2. 太旧的 iPad 装不了 App 的话，用 Safari 打开 `http://homeassistant.local:8123/home-tablet`，点「分享 → 添加到主屏幕」
3. **设置 → 辅助功能 → 引导式访问**：开启后，在中控屏页面连按三下侧边键，平板就锁定在这个页面
4. **设置 → 显示与亮度 → 自动锁定 → 永不**，平板一直插着电

**安卓平板：**
1. 安装 **Home Assistant** App，或者用 **Fully Kiosk Browser**（专门做墙上屏幕的浏览器：全屏显示、有人走近时自动亮屏）
2. 打开 `http://homeassistant.local:8123/home-tablet`
3. 开发者选项里打开「充电时保持唤醒」

> 平板长期插电，电池可能会鼓包。很多安卓平板可以设置「充到 80% 停止」，记得打开。

### 第 5 步：安装手机 App
1. 在 App Store / Google Play 搜索 **Home Assistant** 并安装
2. 登录你的 Home Assistant，**允许定位和通知权限**
3. 到 **设置 → 设备与服务 → Mobile App** 查看你手机的名字，然后把通知服务改成 `notify.mobile_app_phone`：
   最简单的办法是在 App 里 **设置 → 伴侣应用 → 设备名称** 改成 `phone`
4. 在 App 侧边栏打开 **「我的家」** 仪表盘，可设为默认首页

### 第 6 步：开启「回家预热」（可选）
**设置 → 设备与服务 → 添加集成 → Proximity（邻近）**，区域选「家」，跟踪你的手机，名字填 `home`。会生成 `sensor.home_nearest_distance` 等实体，预热自动化会自动生效。

### 第 7 步：在外面也能用
- **最简单**：订阅 [Nabu Casa](https://www.nabucasa.com/)（官方远程访问，同时支持官方开发）
- **免费**：用 [Tailscale](https://tailscale.com/) 组建虚拟局域网，手机装 Tailscale 即可访问
- 不建议直接在路由器上做端口映射暴露 8123

### 第 8 步：重启并检查
**开发者工具 → YAML → 检查配置**，没有报错后点 **重启**。

---

## 目录结构

```
config/
├── configuration.yaml        主配置
├── packages/                 按设备拆分的功能
│   ├── home_mode.yaml        家庭模式（在家/离家/睡眠/度假）
│   ├── vacuum.yaml           扫地机器人
│   ├── heating.yaml          暖气
│   ├── cameras.yaml          监控摄像头
│   └── plugs.yaml            智能插座
├── dashboards/mobile.yaml    手机仪表盘
├── dashboards/tablet.yaml    平板中控屏
└── secrets.yaml.example      密码模板（复制成 secrets.yaml）
docker-compose.yml            Docker 部署（备用，Linux / NAS 用）
```

## 常见调整

- **修改温度**：直接在手机「暖气」页拖动滑块即可。重启后会恢复为 `heating.yaml` 里的 `initial` 值，想永久修改请改那里。
- **修改起床 / 睡觉时间**：手机「设置」页。
- **不想离家自动扫地 / 不想收到报警**：手机「设置」页关掉对应开关。
- **度假模式**：在首页手动选「度假」，暖气保持 12°C 防冻，监控全天报警，回家后自动恢复。

---

## 🏡 全屋智能升级路线

已有：扫地机 ✅ 暖气 ✅ 摄像头 ✅ 智能插座 ✅ 主机电脑 ✅ 中控平板 ✅

按「性价比 + 对生活改善」从高到低排。以下都**适合租房**：插上就用，不用改电线。

| 优先级 | 买什么 | 能实现什么 |
|---|---|---|
| ✅ 已有 | 闲置电脑当主机，闲置平板当中控屏 | 不用另买 |
| ② 强烈推荐 | **Zigbee 网关**：Home Assistant Connect ZBT-2（USB，插在主机电脑上） | 接入下面这些便宜、省电、不占 Wi-Fi 的传感器 |
| ③ | **门窗传感器**（Aqara、Third Reality 等 Zigbee 款） | 开门推送到手机；开窗时暖气自动暂停，省暖气费 |
| ④ | **人体传感器** | 晚上起夜自动开灯；没人自动关灯 |
| ⑤ | **温湿度传感器**（每个房间一个） | 卧室温度偏低时，暖气按卧室温度调节；冬天太干燥时提醒你 |
| ⑥ | **智能灯泡**（Kasa / Tapo / Philips Hue） | 回家自动开灯、睡觉一键关全屋、早上渐亮叫醒 |
| ⑦ | **漏水传感器**（放在洗衣机、水槽下） | 漏水立刻报警，避免泡坏地板被房东扣押金 |
| ⑧ 可选 | 智能门锁 / 门铃 | 远程开门、访客提醒（租房要先问房东） |

**买传感器时认准 Zigbee 协议**：便宜、电池能用 1 到 2 年，而且不依赖厂家 App 和云端。
避开只能用自家 App、Home Assistant 接不了的小众品牌。

买了新设备告诉我型号，我帮你接进系统、加到手机页面、写好联动。
