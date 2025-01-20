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
intel_iommu=on iommu=pt
```
完整示例：
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
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
lspci -nn | grep VGA Audio
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
