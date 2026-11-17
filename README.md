# QJ 控制器 WSL 编译环境

适用于同代际的 **32 位 xeno-x86 控制器工程**，支持 `ETHERCAT_X86` 和 `VIRTUAL_SERVO`。不是 ARM 编译环境，也不是实时控制器运行系统。

## 下载

- [Release 页面](https://github.com/zizhuoruzi/qj-wsl-env/releases/tag/wsl-env-v1)
- [加密环境包](https://github.com/zizhuoruzi/qj-wsl-env/releases/download/wsl-env-v1/qj-wsl-env.7z)
- [下载文件校验值](https://github.com/zizhuoruzi/qj-wsl-env/releases/download/wsl-env-v1/SHA256SUMS.txt)

下载公开；**解压密码由发布者授权提供**，本仓库、Release 和 agent 提示词均不包含密码。

## 内容

- Ubuntu 12.04 i686 的 GCC/G++ 4.6.3、glibc 2.15、make 3.81。
- Xenomai 2.6.4 和目标内核 `3.14.17-xenomai` 的构建树。
- 原系统的补充库与开发头文件。
- `prepare-rootfs.sh`、`build-controller.sh`、版本清单及 SHA-256 清单。

这是环境包，不包含控制器项目源码、现成控制器升级包、现场配置或账户私钥。工程自己的 `src`、`lib` 和额外 SDK 仍由工程提供；“通用”仅限上述平台，不能保证所有工程的附加依赖都齐全。

## 在 Windows 上使用

需要 Windows x64、WSL 2、已安装的 Ubuntu 发行版以及 7-Zip。若已经能编译，先检查已有环境，保留它，避免重复恢复。启用 WSL 可能需要管理员权限、重启和 CPU 虚拟化；已有 VMware/虚拟机软件应先确认兼容性。

PowerShell 下载：

```powershell
$dir = Join-Path $env:USERPROFILE 'Downloads\qj-wsl-env'
New-Item -ItemType Directory -Force -Path $dir | Out-Null
curl.exe --fail --location --output "$dir\qj-wsl-env.7z" 'https://github.com/zizhuoruzi/qj-wsl-env/releases/download/wsl-env-v1/qj-wsl-env.7z'
curl.exe --fail --location --output "$dir\SHA256SUMS.txt" 'https://github.com/zizhuoruzi/qj-wsl-env/releases/download/wsl-env-v1/SHA256SUMS.txt'
Get-FileHash -Algorithm SHA256 -LiteralPath "$dir\qj-wsl-env.7z"
```

把结果与 `SHA256SUMS.txt` 比对。向发布者取得解压授权后执行下面命令，7-Zip 会在终端询问密码：

```powershell
& "$env:ProgramFiles\7-Zip\7z.exe" x "$dir\qj-wsl-env.7z" "-o$dir"
if ($LASTEXITCODE -ne 0) { throw '解压失败，停止恢复' }
```

恢复脚本自动使用自身所在目录，不需要编辑路径；可直接传 Windows 路径作为第一个参数：

```powershell
$linuxDir = (wsl.exe -d Ubuntu --exec wslpath -u $dir).Trim()
if ($LASTEXITCODE -ne 0) { throw 'Windows 目录转换失败' }
wsl.exe -d Ubuntu -u root --exec bash "$linuxDir/prepare-rootfs.sh"
if ($LASTEXITCODE -ne 0) { throw '环境恢复未完成，保留日志' }
wsl.exe -d Ubuntu -u root --exec install -m 755 "$linuxDir/build-controller.sh" /opt/qjar-build/build-controller.sh
if ($LASTEXITCODE -ne 0) { throw '构建入口安装失败' }
```

恢复会先验证内部校验清单，在独立目录解压，编译并运行一个 32 位 C++ 程序，成功后才迁入 `/opt/qjar-build/ubuntu12-i686`。若目标已存在，脚本拒绝覆盖。

编译与打包示例（将工程目录替换成真实路径）：

```powershell
wsl.exe -d Ubuntu -u root --exec bash /opt/qjar-build/build-controller.sh 'D:\控制器工程\cx-v3.0' --package
```

产物位于工程自己的 `build-wsl/controller.<编号>/control_update.tar.gz`，同时保存编译日志、源码校验结果和 `result.json`。构建不执行 `make clean/install/up`，不连接或部署到控制器。

## 边界和来源

chroot 用于固定旧工具链路径，**不是安全沙箱**；仅构建可信的控制器源码。WSL 2 使用宿主提供的 Linux 内核，目标 Xenomai 内核文件只用于编译，编译通过不等于已验证驱动加载或实时运动。

版本记录见 `manifest.json`。GNU/Linux、Ubuntu 软件包、Xenomai 等各自保留原许可证；本项目的说明不替代各软件许可证。Ubuntu 历史包源见 <https://old-releases.ubuntu.com/ubuntu/>，GNU 软件源见 <https://ftp.gnu.org/gnu/>，Xenomai 源见 <https://source.denx.de/Xenomai/xenomai>。目标内核源码及其许可证随内核构建树保留。
