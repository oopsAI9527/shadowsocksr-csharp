ShadowsocksR for Windows
=======================

[![Build Status]][Appveyor]    <b> <=== \[click this to download nightly version\] </b>

#### Download

You will need to download and install [7-Zip](http://www.7-zip.org/) in order 
to extract the ShadowsocksR archive.

Download the [latest release] for ShadowsocksR for Windows.

_Optionally_, right-click on the downloaded 7z file and select 
**CRC SHA** > **SHA-256**. Verify that the SHA-256 checksum displayed 
matches the expected checksum which was shown on the releases page.

Right-click on the downloaded 7z file and do **7-Zip** > **Extract Here** 
or extract to a new folder.

_Optionally_, download and install [Gpg4win](https://www.gpg4win.org/). 
From the Windows start menu, launch program **Kleopatra**. 
Do **File** > **New Certificate** to create a personal OpenPGP key pair. 
Save the signing key from
[Akkariiin/pubkey](https://github.com/Akkariiin/pubkey) as a text file. 
Then do **File** > **Import Certificates** to import the signing key text file.
After import, select the signing key and do 
**Certificates** > **Certify Certificates**. 
You will need to enter the passphrase for your own key. 
Finally, do **File** > **Decrypt/Verify Files** for the executable 
you propose to use (see below). A message confirming successful verification 
of the signature appears against a green background. 
Close program **Kleopatra**.

For >= Windows 8 or with .Net 4.0, using ShadowsocksR-dotnet4.0.exe.

For <= Windows 7 or with .Net 2.0, using ShadowsocksR-dotnet2.0.exe.

#### Usage

1. Find ShadowsocksR icon in the system tray.
2. You can add multiple servers in servers menu.
3. Select Enable System Proxy menu to enable system proxy. Please disable other
proxy addons in your browser, or set them to use system proxy.
4. You can also configure your browser proxy manually if you don't want to enable
system proxy. Set Socks5 or HTTP proxy to 127.0.0.1:1080. You can change this
port in Global settings.
5. You can change PAC rules by editing the PAC file. When you save the PAC file
with any editor, ShadowsocksR will notify browsers about the change automatically.
6. You can also update the PAC file from GFWList. Note your modifications to the PAC
file will be lost. However you can put your rules in the user rule file for GFWList.
Don't forget to update from GFWList again after you've edited the user rule.
7. For UDP, you need to use SocksCap or ProxyCap to force programs you want
to proxy to tunnel over ShadowsocksR.

### Develop

Visual Studio Community 2017 is recommended.

#### License

GPLv3

Copyright © Akkariiin 2019. Forked from ShadowsocksR by BreakWa11

[Appveyor]:       https://ci.appveyor.com/project/Akkariiin/shadowsocksr-csharp
[Build Status]:   https://ci.appveyor.com/api/projects/status/ey901turnakim5nv/branch/master?svg=true
[latest release]: https://github.com/shadowsocksrr/shadowsocksr-csharp/releases

### 项目分析文档（中文）

为便于非技术读者快速了解本项目的功能与架构，可查看仓库根目录中的 `ANALYSIS.zh-CN.md`。

### CI 构建与发布（GitHub Actions）

以下为使用 GitHub Actions 自动打包并发布 Release 的简要指南：

- 触发条件：推送以 `v` 开头的标签（例如：`v2025.11.06`）。
- 运行环境：使用 `windows-2019` 运行器以确保 .NET Framework 4.0 参考程序集可用。
- 构建产物：
  - `artifacts/ShadowsocksR-dotnet2.0-{version}.zip`
  - `artifacts/ShadowsocksR-dotnet4.0-{version}.zip`
- 发布流程：工作流会在构建完成后自动创建 GitHub Release，并将上述 ZIP 作为附件上传。
- 常见问题与解决：
  - MSB3644（缺少 .NETFramework,Version=v4.0 的参考程序集）：
    - 解决方案一：使用 `windows-2019` 运行器；
    - 解决方案二：安装 .NET Framework 4.0 Developer Pack（Targeting Pack）。
- 本地触发示例：
  - `git tag v2025.11.06 && git push origin v2025.11.06`
  - 或在 GitHub Actions 界面手动 Dispatch（手动触发）。
