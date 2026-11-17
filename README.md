# 🏠 我的智能家居控制系统

基于 [Home Assistant](https://www.home-assistant.io/)（免费、开源、本地运行），统一控制 **扫地机器人、监控摄像头、暖气**，在手机上随时随地查看和操作。

| 暖气 | 扫地机器人 |
|---|---|
| <img src="docs/heating.png" width="260"> | <img src="docs/vacuum.png" width="260"> |

## 功能一览

**手机仪表盘（5 个页面）**
- **首页**：家庭模式、暖气温度、扫地机状态、所有摄像头画面、快捷按钮
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

### 第 1 步：准备一台常开的设备
任选一种：树莓派 4/5、NAS（群晖/威联通）、旧电脑或迷你主机。需要和扫地机、摄像头在**同一个 Wi-Fi / 局域网**。

### 第 2 步：安装 Home Assistant
**方式 A：Docker（推荐，NAS / 电脑）**——注意这台设备要和扫地机、摄像头连同一个 Spectrum Wi-Fi
```bash
git clone <这个仓库地址> home-assistant
cd home-assistant
docker compose up -d
```

**方式 B：树莓派**：按 [官方教程](https://www.home-assistant.io/installation/raspberrypi) 安装 Home Assistant OS，然后把本仓库 `config/` 目录里的文件复制到它的 `/config` 目录（可用 *File editor* 或 *Samba* 加载项）。

启动后用浏览器打开 `http://设备IP:8123`，按提示创建账号。

### 第 3 步：接入你的三类设备

#### 🧹 Narwal 云鲸扫地机
Narwal 没有官方 Home Assistant 集成，用社区开发的 **Narwal Local**（[cvette/narwal-ha](https://github.com/cvette/narwal-ha)）。它走局域网直连，不需要云鲸账号。
1. 先装 **HACS**（HA 的社区插件商店）：按 [HACS 官方教程](https://hacs.xyz/docs/use/) 安装并重启
2. HACS → 右上角 ⋮ → **自定义存储库** → 填 `https://github.com/cvette/narwal-ha`，类别选 **Integration**
3. 搜索安装 **Narwal Local**，重启 HA
4. 扫地机一般会出现在 **设置 → 设备与服务 → 已发现**，点「配置」即可。没有出现就手动添加 **Narwal Local**，填扫地机的 IP 地址
5. 建议在路由器（Spectrum）里给扫地机**固定 IP**（DHCP 保留）

会自动生成 `vacuum.narwal`、`sensor.narwal_battery`，配置里已经用这两个名字，**不用改名**。

> ⚠️ 这个集成目前还是早期版本，作者主要用 J4 测试，云鲸固件更新后可能失效。如果你的型号用不了，告诉我型号，我帮你换别的集成。

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

### 第 4 步：统一实体 ID ⭐
进入 **设置 → 设备与服务 → 实体**，点开对应设备 → ⚙️ → 修改「实体 ID」：

| 改成这个 ID | 是什么 |
|---|---|
| `climate.heating` | Nest 温控器 |
| `camera.front_door` | 第 1 个摄像头（门口） |
| `camera.living_room` | 第 2 个摄像头（客厅） |
| `binary_sensor.front_door_motion` | 第 1 个摄像头的移动侦测 |
| `binary_sensor.living_room_motion` | 第 2 个摄像头的移动侦测 |

Narwal 扫地机不用改。

> 摄像头数量不是 2 个，或者装在别的位置？告诉我，我帮你改 `cameras.yaml` 和手机页面。
> 摄像头没有移动侦测传感器的话，实时画面照常能看，只是没有「有人移动」推送。

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
│   └── cameras.yaml          监控摄像头
├── dashboards/mobile.yaml    手机仪表盘
└── secrets.yaml.example      密码模板（复制成 secrets.yaml）
docker-compose.yml            Docker 一键部署
```

## 常见调整

- **修改温度**：直接在手机「暖气」页拖动滑块即可。重启后会恢复为 `heating.yaml` 里的 `initial` 值，想永久修改请改那里。
- **修改起床 / 睡觉时间**：手机「设置」页。
- **不想离家自动扫地 / 不想收到报警**：手机「设置」页关掉对应开关。
- **度假模式**：在首页手动选「度假」，暖气保持 12°C 防冻，监控全天报警，回家后自动恢复。
