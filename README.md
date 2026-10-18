ESP32_AdBlocker

ESP32_AdBlocker 充当 DNS Sinkhole（类似 Pi-Hole），对拦截列表中的任何域名返回 0.0.0.0，否则使用外部 DNS 服务器解析 IP 地址。这可防止从被拦截域名获取内容或向其发送内容。它提供了一个 Web 服务器用于控制该服务并监控其运行。

版本 3.7 包括

· 用户 @dateno1 对 DNS 查询处理和 LED 状态的增强
· 时区修复
· 来自用户 @blueheart001 的修复

要求

ESP32_AdBlocker 是一个 Arduino sketch。ESP32 模块需要 PSRAM：

· 带有 8MB PSRAM 的 ESP32-S3 可容纳当前规模的拦截列表。拦截列表检查耗时 <50 微秒。
· 带有 4MB PSRAM 的 ESP32 可能会截断当前规模的拦截列表。拦截列表检查耗时 <100 微秒。

运行

上电后，将下载默认拦截列表。ESP32_AdBlocker 在处理并排序数据后需要几分钟才能就绪。可在网页上监控进度。后续重新加载同一文件会快得多，因为只需处理更新。ESP32-S3 的速度约为 ESP32 的两倍。由于只能下载一个文件，应使用合并后的拦截列表。理想情况下选择小于 PSRAM 大小的文件。文件格式应为 HOSTS 格式或 Adblock 格式（仅处理域名条目）。例如，以下站点提供了合适文件的列表：https://github.com/StevenBlack/hosts。

随后，ESP32_AdBlocker 将每天在给定时间下载所选文件以保持拦截列表更新。用户还可以单独添加自己要拦截或放行的站点，这些站点存储在本地自定义拦截列表中。

ESP32_AdBlocker 网页上的条目包括：

· 允许的域名：自重启以来已放行的域名请求数
· 拦截的域名：自重启以来已拦截的域名请求数
· 拦截列表文件的当前 URL：正在使用的拦截列表的 URL
· 输入拦截列表或域名的新 URL：
· 输入拦截列表的新 URL 后，按 Reload 按钮下载，或留空以重新加载当前拦截列表。
· 输入要拦截的额外域名 URL 后，按 AddDomain 按钮。如果重复或无法解析则不会添加。弹窗消息将显示结果。
· 输入要从拦截列表中移除的现有域名 URL 后，按 DelDomain 按钮。弹窗消息将显示结果。
· 输入域名 URL 以检查是否在拦截列表中后，按 CheckDomain 按钮。弹窗消息将显示结果。
· 停止拦截列表加载：按 StopLoad 按钮停止当前正在下载的拦截列表。
· 清除自定义拦截列表：清除用户手动添加或移除的自定义条目
· 启用 AdBlocker：切换广告拦截的开或关

要将 ESP32_AdBlocker 设为你的首选 DNS 服务器，请在路由器/设备中以其 IPv4 地址替换当前 DNS 服务器 IP。ESP32_AdBlocker 没有 IPv6 地址，但某些设备默认使用 IPv6，因此请在设备/路由器上禁用 IPv6 DNS 以强制其使用 IPv4 DNS。
例如，对于 Windows PC 网络适配器，要使用 IP 地址为 192.168.1.168 的 AdBlocker 作为 DNS 服务器，请在 Windows 命令提示符中输入：netsh interface ip set dns "Wi-Fi" static 192.168.1.168
要切换回常用 DNS 服务器，例如 Google，请输入：netsh interface ip set dns "Wi-Fi" static 8.8.8.8

浏览器必须禁用“使用安全 DNS”，因为这会覆盖适配器和路由器的 DNS 设置。

安装

将 github 文件下载到 Arduino IDE sketch 文件夹，从应用程序文件夹名称中删除 -main。

使用启用了 PSRAM 的 esp32 arduino core 最低 v3.1.1 以及以下分区方案进行编译：

· ESP32-S3 -8M with spiffs (...)
· ESP32 -Minimal SPIFFS (...)

首次安装时，应用程序将以 wifi AP 模式启动——连接到 SSID：ESP32_AdBlocker_...，以允许通过 192.168.4.1 上的网页输入路由器和密码详细信息。配置文件（密码除外）会自动创建，并且当有互联网连接可用时，应用程序网页会自动从 GitHub 下载到 SD 卡的 /data 文件夹。

后续对应用程序或 /data 文件夹文件的更新，可使用 OTA Upload 选项卡进行。/data 文件夹也可使用 Edit Config 选项卡上的 Reload /data 按钮，或使用 WebDAV 客户端从 GitHub 重新加载。

配置

更多配置详细信息可通过 Edit Config 选项卡访问，其中显示更多按钮：

网络：额外的网络和 Web 服务器设置。特别是设置：

· 路由器 IP 地址以获取静态 IP
· 静态 IP 地址，用作 AdBlocker DNS 服务器 IP

设置：影响拦截列表运行的环境设置。

以太网：选择所需的网络。要配置以太网，请定义用于连接外部以太网控制器的 SPI 引脚编号。按 Save 使更改持久化。

日志记录

可在网页选项卡 Show Log 上监控应用程序日志消息。

Verbose 按钮将显示每个被拦截或接受连接的额外日志记录。

网络选择

默认网络接口是 Wifi，但也可以使用带有内置以太网的开发板，或通过连接外部以太网控制器来改用以太网。该功能仅针对连接到 ESP32S3 开发板的 W5500 以太网控制器进行了测试。所选网络接口在配置并重启后可用。

选项：

· WiFi：默认，如果以太网无法连接也作为回退
· Eth+AP：以太网加 ESP 接入点。请勿在每个网络上同时打开网页。
· 以太网：仅以太网，无 Wifi

LED 状态

板载或外部 LED 可用于指示应用程序状态，具体取决于可用 LED 的类型：

简单 LED：

· AP 模式：慢速闪烁
· 活动：常亮
· 离线：极慢闪烁
· 故障：快速闪烁
· 下载 hosts：中速闪烁

W2812 RGB LED：

· AP 模式：慢速闪烁黄色
· 活动：常亮绿色
· 离线：极慢闪烁青色
· 故障：快速闪烁红色
· 下载 hosts：中速闪烁蓝色
· 尝试在没有互联网的情况下下载 hosts：闪烁紫色

LED 使用网页选项卡 Edit Config 中的 Settings 按钮进行配置。
将 LED Pin 值设为 0 可禁用 LED。