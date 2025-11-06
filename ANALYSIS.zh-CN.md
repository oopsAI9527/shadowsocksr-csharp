# ShadowsocksR C# 项目功能与目录架构分析（中文）

本文为非技术背景读者准备，尽量用通俗语言解释本项目（ShadowsocksR for Windows）的功能、运行原理和代码目录结构，方便快速上手与定位问题。

## 一、项目概览（它是做什么的）

- 这是一个 Windows 客户端，提供系统托盘界面（右下角小图标），用于连接 ShadowsocksR（简称 SSR）服务器，把你的网络请求按规则“走代理”或“直连”。
- 支持多种“代理模式”：保持不改、直连、PAC 模式（自动按规则选择是否走代理）、全局模式（全部走代理）。
- 在本地开启一个“代理服务”（像一个小型转运站）：浏览器或系统把网络流量先送到这里，再由它加密并转发到远端 SSR 服务器。
- 提供 PAC 文件与服务：自动为浏览器/系统生成合适的代理规则。
- 支持多种加密与混淆（提升安全与抗干扰能力），并有服务器订阅、负载均衡、日志等功能。

把它想象成“网络快递转运站”：
- 你在电脑上把包裹（网络请求）交给它；
- 它根据“名单”（PAC 规则）决定哪些包裹需要走代理路线；
- 走代理的包裹会先加密、再通过远端服务器转运到目的地；
- 不需要代理的包裹就直接投递（直连）。

## 二、核心功能列表（用户可见与后台机制）

- 系统托盘界面与菜单（`View/`、`MenuViewController.cs`）
  - 切换代理模式：不修改、直连、PAC、全局
  - 服务器管理：添加/编辑/选择服务器，支持订阅更新
  - 负载均衡与随机选择：在多服务器之间智能分配
  - 启动时自动运行、允许局域网访问、日志查看

- 本地代理服务（`Controller/Local.cs`、`HttpProxyRunner.cs`）
  - 在本地开启 SOCKS5/HTTP 代理端口（默认 1080，可在设置里改）
  - 接收浏览器/系统的请求，按配置进行加密、混淆、转发
  - 支持 UDP（部分应用需要），可通过 SocksCap/ProxyCap 助力

- 系统代理设置与 PAC（`Controller/SystemProxy.cs`、`Controller/PACServer.cs`）
  - 自动写入 Windows 的代理设置（注册表与 WinINet API）
  - 生成并提供 PAC 规则服务，动态返回“走代理/直连”的脚本
  - PAC URL 带 `auth` 防护参数，浏览器/系统通过该 URL 获取规则

- 加密与混淆（`Encryption/EncryptorFactory.cs`、`Obfs/ObfsFactory.cs`）
  - 加密：如 `MbedTLSEncryptor`、`LibcryptoEncryptor`、`SodiumEncryptor` 等
  - 混淆/协议：如 `AuthChain_*`、`HttpSimpleObfs`、`TlsTicketAuthObfs` 等
  - 以“工厂模式”注册与创建，便于扩展与组合

- 配置与订阅（`Model/Configuration.cs`、`Model/Server.cs`）
  - 维护服务器列表、当前索引、系统代理模式、端口、DNS、日志等
  - 支持服务器订阅（从远端拉取节点），白名单/规则模式切换

- 日志与工具（`Util/`、`View/LogForm.cs`）
  - 记录运行异常、连接状态、速度测试指标，便于排查问题
  - 常用工具函数（时间戳、DNS 查询、防护校验等）

## 三、运行原理（数据是怎么“走起来”的）

1. 启动阶段（`Program.cs`）
   - 读取配置（端口、服务器、模式等），初始化控制器 `ShadowsocksController`
   - 挂载系统托盘菜单 `MenuViewController`，根据命令行处理如“开机启动”设置
   - 启动本地代理与 PAC 服务

2. 代理工作流（请求转运）
   - 浏览器/系统把请求发到本地端口（如 `127.0.0.1:1080`）
   - `Local.cs` 解析请求，选择当前服务器，应用加密与混淆，转发到远端
   - 远端服务器解密后访问目标网站，再把结果回传到本地，再到浏览器
   - 如处于 PAC 模式：浏览器会按 PAC 脚本判断某域名是否需要走代理

3. 系统代理与 PAC（`SystemProxy.cs`、`PACServer.cs`）
   - 写入注册表 `ProxyEnable`、`ProxyServer`、`AutoConfigURL`（或使用 WinINet）
   - PAC URL 形如：`http://127.0.0.1:<本地端口>/pac?auth=<密码>&t=<时间戳>`
   - 刷新 IE/系统代理（`NotifyIE()`），并同步 LAN 配置，确保设置生效
   - PAC 服务返回内容中会替换占位符：`__DIRECT__` 与 `__PROXY__`，决定走向

## 四、启动流程（从入口到托盘界面）

- `Program.cs` 是入口：创建 `ShadowsocksController` 与 `MenuViewController`
- 加载配置、初始化日志、处理命令行（例如 `--setautorun`）
- 控制器启动：本地代理监听端口、PAC 服务就绪、系统代理更新
- 托盘菜单出现：可切换模式、管理服务器、查看日志与设置

## 五、目录结构（从外到内）

- 仓库根目录
  - `README.md`：使用说明与下载提示
  - `AGENTS.md`、`CONTRIBUTING.md`、`LICENSE`：贡献与许可信息
  - `shadowsocks-csharp.sln`：解决方案文件（用于 Visual Studio）
  - `shadowsocks-csharp/`：核心代码目录（详见下方）
  - `test/`：测试工程与用例

- `shadowsocks-csharp/` 关键子目录
  - `Controller/`：运行控制层（本地代理、系统代理、PAC、HTTP 代理等）
    - `Local.cs`：本地 SOCKS/HTTP 代理核心，连接远端、加密、传输
    - `SystemProxy.cs`：写系统代理与 PAC URL、刷新系统设置
    - `PACServer.cs`：生成并返回 PAC 内容，决定直连/代理
    - `HttpProxyRunner.cs`、`HttpProxy.cs`：HTTP 代理逻辑
    - `ShadowsocksController.cs`：整体协调、模式切换、配置保存
  - `Model/`：数据模型
    - `Configuration.cs`：全局配置（端口、模式、DNS、订阅、日志等）
    - `Server.cs`：单个服务器配置（地址、端口、加密、混淆）
  - `Encryption/`：加密算法工厂与实现
    - `EncryptorFactory.cs`：注册/创建加密器（如 MbedTLS/Libcrypto/Sodium）
  - `Obfs/`：混淆与协议实现
    - `ObfsFactory.cs`：注册/创建混淆器（如 AuthChain、HTTP/TLS 混淆）
  - `View/` 与 `Properties/`、`Resources/`：界面窗体与资源（托盘菜单、设置窗体、日志窗口、语言资源等）
  - `Util/`：工具方法（日志、DNS、时间、校验等）
  - `Program.cs`：程序入口
  - `*.csproj`：不同目标框架的工程文件（如 dotnet2.0 / 4.0）

- `test/`
  - 单元测试示例（`ServerTest.cs`、`UnitTest.cs`），验证基础逻辑

## 六、配置与常用参数（`Configuration.cs` 摘要）

- `sysProxyMode`：系统代理模式（0 保持、1 直连、2 PAC、3 全局）
- `localPort`：本地代理端口（默认 1080）
- `configs` 与 `index`：服务器列表与当前选择
- `random`、`balanceAlgorithm`：随机选择与负载均衡策略
- `proxyEnable`、`proxyType`、`proxyHost`、`proxyPort`：上游代理（如再套一层 HTTP/SOCKS）
- `localAuthPassword`：PAC URL 中的认证参数，用于保护 PAC 服务
- `dns`、`local_dns_servers`：DNS 相关设置
- `reconnectTimes`、`connect_timeout`：连接重试与超时策略
- `logging`：日志开关与级别
- `serverSubscribe`：服务器订阅相关配置

提示：配置会被读取并自动“纠错”（Fix），保证重要字段存在默认值。

## 七、加密与混淆支持（可选组合）

- 加密器（示例）：`MbedTLSEncryptor`、`LibcryptoEncryptor`、`SodiumEncryptor`、`NoneEncryptor`（不加密）
- 混淆/协议（示例）：`AuthChain_a-f`、`AuthAkarin`、`HttpSimpleObfs`、`TlsTicketAuthObfs`、`VerifyDeflateObfs` 等
- 通过工厂类统一管理，便于在服务器与客户端配置中选择匹配的算法与参数。

## 八、构建与运行（开发者视角）

- 推荐 `Visual Studio Community 2017`
- 打开 `shadowsocks-csharp.sln`，选择合适的工程（如 `shadowsocks-csharp.csproj` 或 `shadowsocks-csharp4.0.csproj`）进行构建
- 也可通过命令行工具（如 `MSBuild`）在对应 .NET Framework 环境下构建

## 九、使用要点（用户视角）

- 托盘图标右键菜单：
  - 选择“Enable System Proxy（启用系统代理）”后，浏览器应设置为“使用系统代理”
  - PAC 模式：自动按名单决定是否走代理；全局模式：所有请求走代理
  - 不启用系统代理时，浏览器可手动设置代理到 `127.0.0.1:<localPort>`（SOCKS5/HTTP）
- PAC 文件可更新：支持从 GFWList 更新，也支持用户自定义规则文件
- UDP 流量：需要配合 SocksCap/ProxyCap 等工具强制指定走代理

## 十、常见问题与排查

- 代理不生效：
  - 检查系统代理模式是否为 PAC/全局
  - 浏览器是否“使用系统代理”；或手动设置到 `127.0.0.1:<localPort>`
  - 端口是否被占用（改用其他端口并在设置中更新）

- PAC 无法访问：
  - 确认 PAC URL 中的 `auth` 参数（由 `localAuthPassword` 提供）
  - 防火墙是否拦截本地 PAC 服务访问

- 连接慢或失败：
  - 查看日志窗口，关注错误码与超时统计
  - 切换服务器或加密/混淆方案；调整超时与重试次数

## 十一、可改进建议（面向后续开发）

- 增强测试覆盖度：为 PAC 服务、系统代理设置、订阅更新等关键路径补充单元测试
- 提供更友好的引导界面：加入“首次运行向导”，自动检测端口占用与网络连通
- 优化日志与诊断：支持一键导出诊断包，便于用户反馈问题
- 文档化配置项：在 UI 中对每个选项给出简短说明和推荐值

## 十二、本次分析记录（变更与说明）

- 依据仓库 `README.md` 与核心代码（`Program.cs`、`ShadowsocksController.cs`、`Local.cs`、`SystemProxy.cs`、`PACServer.cs`、工厂类等）整理成本文档
- 文件已存放于仓库根目录，供非技术读者快速理解项目

—— 完 ——