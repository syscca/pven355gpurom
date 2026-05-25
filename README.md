# pven355gpurom

## 硬酷R2 MAX N355 PVE WINDOWS 11 核显GPU直通ROM带音频输出.

1. 检查核显设备
运行以下命令，确认核显 PCI 地址（通常是 00:02.0）：
```
lspci | grep VGA
```
输出示例：
```
00:02.0 VGA compatible controller: Intel Corporation Device <ID>
```
3. 修改内核参数
编辑 GRUB 配置文件：
```
nano /etc/default/grub
```
在 GRUB_CMDLINE_LINUX_DEFAULT 中添加以下参数：
```
intel_iommu=on iommu=pt video=efifb:off
```
完整示例：
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt video=efifb:off"
```
保存并更新 GRUB：
```
update-grub
```
```
reboot
```
4. 加载必要模块
编辑模块配置文件：
```
nano /etc/modules
```
添加以下内容：
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
5. 禁用模块：
```
nano /etc/modprobe.d/pve-blacklist.conf
```
```
blacklist i915
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
```
6. 绑定vfio-pci

运行以下命令，确认 PCI 地址
```
lspci -nn | grep VGA
lspci -nn | grep Audio
```
```
00:1f.3 Audio device: Intel Corporation Alder Lake-N PCH High Definition Audio Controller
```

```
nano /etc/modprobe.d/vfio.conf
```
```
options vfio-pci ids=8086:46d3,8086:54c8
```
8. 启用iommu_unsafe_interrupts
```
nano /etc/modprobe.d/iommu_unsafe_interrupts.conf
```
```
options vfio_iommu_type1 allow_unsafe_interrupts=1
```
9. HDMI audio crackling/broken
```
nano /etc/modprobe.d/snd-hda-intel.conf
```
```
options snd-hda-intel enable_msi=1
```
运行以下命令以刷新配置：
```
update-initramfs -u
```
```
reboot
```
上传ROM到目录
```
ls /usr/share/kvm/Intel.rom
```
加权限
```
chmod 644 /usr/share/kvm/Intel.rom
```
步骤 3: 配置虚拟机
1. 添加核显直通配置
编辑虚拟机的配置文件：
```
nano /etc/pve/qemu-server/<VMID>.conf
```
添加或修改以下内容：
```
100.conf
```

自行生成命令，用UBU提取BIOS生成IntelGopDriver.efi，IgdAssignmentDxe.efi和PlatformGOPPolicy.efi用https://github.com/tomitamoeko/VfioIgdPkg编译。
```
.\EfiRom.exe -e .\IntelGopDriver.efi .\IgdAssignmentDxe.efi .\PlatformGOPPolicy.efi -f 0x8086 -i 0x46d3 -o n355_igd.rom
```

下面是AI说明
好的，针对你的需求，我将提供一份**只编译 `IgdAssignmentDxe.efi` 和 `PlatformGOPPolicy.efi` 两个模块**的精简指南。你不需要编译整个 DSC 文件，只需单独指定每个模块的 INF 路径即可。

---

## ✅ 在 Debian 13 上单独编译 IgdAssignmentDxe 和 PlatformGOPPolicy

### 1️⃣ 准备环境（同前，但可跳过完整包编译依赖）

```bash
sudo apt update
sudo apt install -y build-essential git python3 python3-pip python3-venv uuid-dev nasm iasl gcc-multilib
mkdir -p ~/UEFI && cd ~/UEFI
git clone https://github.com/tianocore/edk2.git
cd edk2
git submodule update --init --recursive
make -C BaseTools
python3 -m venv .venv
source .venv/bin/activate
pip install -r pip-requirements.txt
```

### 2️⃣ 获取 VfioIgdPkg 源码

```bash
cd ~/UEFI
git clone https://github.com/tomitamoeko/VfioIgdPkg.git
mv VfioIgdPkg edk2/
cd edk2
source edksetup.sh
```

### 3️⃣ 分别编译两个模块（不编译整个包）

#### 编译 IgdAssignmentDxe.efi
```bash
build -p VfioIgdPkg/VfioIgdPkg.dsc \
      -m VfioIgdPkg/IgdAssignmentDxe/IgdAssignment.inf \
      -a X64 -b RELEASE -t GCC
```

#### 编译 PlatformGOPPolicy.efi
```bash
build -p VfioIgdPkg/VfioIgdPkg.dsc \
      -m VfioIgdPkg/PlatformGopPolicy/PlatformGopPolicy.inf \
      -a X64 -b RELEASE -t GCC
```

> **注意**：`-m` 参数指定要编译的模块 INF 文件路径，`-p` 依然需要包 DSC 文件来解析依赖，但只会编译你指定的那一个模块。

### 4️⃣ 查找生成的文件

编译成功后，文件位于：
```bash
~/UEFI/edk2/Build/VfioIgdPkg/RELEASE_GCC5/X64/
```

你可以用以下命令确认：
```bash
ls -lh ~/UEFI/edk2/Build/VfioIgdPkg/RELEASE_GCC5/X64/*.efi
```

应该会看到：
- `IgdAssignmentDxe.efi`
- `PlatformGOPPolicy.efi`

---

## 🧩 后续：合并 ROM 文件

将这两个文件和你从 BIOS 提取的 `IntelGopDriver.efi` 放在一起，使用 `EfiRom` 合并（建议在 Windows 下执行，Linux 版也可用）：

```bash
EfiRom -e IntelGopDriver.efi IgdAssignmentDxe.efi PlatformGOPPolicy.efi \
       -f 0x8086 -i 0x46d3 -o n355_igd.rom
```

验证 ROM 头部：
```bash
hexdump -C n355_igd.rom | head -1
```
期望输出以 `55 aa` 开头。

---

## ⚠️ 常见问题

| 错误 | 解决方法 |
|------|----------|
| `build: command not found` | 确保在 edk2 目录下执行了 `source edksetup.sh` |
| `Tool chain [GCC] is not defined` | 尝试改用 `-t GCC5` 或检查 `gcc --version` |
| `fatal error: gnu/stubs-32.h: No such file` | `sudo apt install gcc-multilib` |
| `IgdAssignment.inf` 找不到 | 确认路径 `VfioIgdPkg/IgdAssignmentDxe/IgdAssignment.inf` 存在 |

如果编译 `PlatformGOPPolicy.inf` 提示缺少依赖，可以忽略它（该模块是可选组件），只编译 `IgdAssignmentDxe` 也可以合并 ROM。

---

按照上述步骤，你就能获得所需的两个 EFI 驱动文件，且无需编译整个庞大的 `VfioIgdPkg` 包。如果编译中遇到具体错误，请提供输出信息，我会帮你分析。

PlatformGOPPolicy.efi

🧩
