qm importdisk 100 /var/lib/vz/template/iso/openwrt.img local-lvm

ls /dev/disk/by-id

qm set 101 -sata2 /dev/disk/by-id/disk id

qm importdisk 101 /var/lib/vz/template/iso/rr.img local-lvm

一键优化脚本
wget -q -O /root/pve_source.tar.gz 'https://bbs.x86pi.cn/file/topic/2024-01-06/file/24f723efc6ab4913b1f99c97a1d1a472b2.gz' && tar zxvf /root/pve_source.tar.gz && /root/./pve_source







天钡wtr n100 nas pve8核显直通win10只需要以下几个步骤，比较简单！

pve8系统安装我这里直接忽略，直接进入主题。

1、执行命令：

nano /etc/default/grub
在里面加入以下内容

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
2、执行命令：

update-grub
3、执行命令：

nano /etc/modprobe.d/pve-blacklist.conf
在里面加入

blacklist nvidiafb
blacklist amdgpu
blacklist i915
blacklist snd_hda_intel
options vfio_iommu_type1 allow_unsafe_interrupts=1
解释：屏蔽三大显卡驱动，屏蔽hdmi声音驱动；

    options vfio_iommu_type1 allow_unsafe_interrupts=1允许不安全的设备中断

4、执行命令：

update-initramfs -u -k all
解释：更新initramfs。

5、执行命令：

reboot
6、直通虚拟机环境设置

在创建win10虚拟机里面请选择默认 ovmf+i440fx 7.2版本或者最新以上机型（不能选q35！），cpu选host，光驱选择ide，硬盘选择sata硬盘，其他保持默认设置。

添加核显pcie 00.02.0，显示设置为无 none。

并同时添加负责声音的pcie设备00.1f.3

intel核显编号和声卡pci编号都很固定！从6-13代我确认看了都是这两个编号，不像amd核显编号会有变化！

手动修改虚拟机参数。

执行命令

nano /etc/pve/qemu-server/100.conf
100是你需要直通的windows虚拟机

增加一行 

args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on
修改这一行为这样

hostpci0: 0000:00:02.0,legacy-igd=1,romfile=n100.rom
声卡是下面这个

hostpci1: 0000:00:1f.3
解释一下参数含义：

①addr=02.0 00:02.0 的 pci bus 位置，如果 IGD 被分配到了这一位置，那么会激活 Legacy 直通模式，因为 Legacy 模式下要求 IGD 的 pci bus 位置需要使用 00:02.0

②x-igd-opregion=on 

The x-igd-opregion property exposes opregion (VBT included) to guest driver so that the guest driver could parse display connector information from. This property mandatory for the Windows VM to enable display output.

③x-igd-gms=0x2

The x-igd-gms property sets a value multiplied by 32 as the amount of pre-allocated memory (in units of MB) to support IGD in VGA modes.

32M为基数0x2=64M 0x8=256M 0x10=512M ，这个值要大于等于物理机中bios核显设置的大小。比如物理机核显设置64M，这里gms你就要必须0x2及以上。

④legacy-igd=1 使用传统igd直通模式



常见问题：解决11-14代pve核显直通win10 win11闪屏黑屏问题
1.设置物理机bios核显为64m
2.设置虚拟机类型为linux
3.参数x-igd-gms=0x2 改为0x8，最大可改为0x10，32M为基数0x2=64M 0x8=256M 0x10=512M
