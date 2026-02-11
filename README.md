# KVM NVIDIA GPU Passthrough

The following setup is for a host with Intel integrated graphics and a dedicated GPU which is only assigned to a guest virtual machine.

Host:
- PC: Dell Optiplex XE4
- CPU: Intel Core i5-12500 vPro (18 MB cache, 6 cores, 12 threads, 3.00 GHz to 4.60 GHz, 65 W)
- iGPU: Intel UHD Graphics 770
- RAM: 2x 16 GB DDR4 3200 MHz
- dGPU: PNY NVIDIA RTX A1000 (8 GB GDDR6, 2304 CUDA cores, 72 Tensor cores, 18 RT cores, 50 W)
- NVMe: 3x 500GB WD Blue SN580
- dUSB: PCIe Insignia NS-PCIEC8 Renesas uPD720202 USB 3.0 Host Controller

## Linux Host Setup Steps

1. Give the NVIDIA dGPU to the VFIO driver instead of the host's default Noveau driver. Enter the PCIe IDs for both the video and audio portion of your GPU. Give the dUSB controller to the VFIO driver also, append PCIe ID of the USB controller. Add softdep lines to prevent the Linux kernel to claim the devices first.
````
$ cat /etc/modprobe.d/vfio.conf 
softdep nouveau pre: vfio-pci
softdep xhci_pci_renesas pre: vfio-pci
options vfio-pci ids=10de:25b0,10de:2291,1912:0015
````
2. Add vfio drivers to be loaded at startup.
```
$ cat /etc/modules-load.d/vfio-pci.conf
vfio
vfio_iommu_type1
vfio_pci
```
3. Add the following options to your `GRUB_CMDLINE_LINUX_DEFAULT`. Note that the `pcie_aspm=off` resolves USB controller reset issue after guest machine shutdown.
```
$ cat /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_aspm=off"
GRUB_CMDLINE_LINUX=""
```
4. Regenerate initramfs and GRUB.
```
$ sudo update-initramfs -u -k all
$ sudo update-grub
```

## Set up automatic monitor switchover based on whether a USB hub is present in the host or removed
Use a USB switch device to connect a monitor USB hub into the host USB controller or into the guests dUSB controller. The following udev rule allows the host to monitor whether the hub is present and therefore follow the USB switch state.

1. Create a new udev rule as shown below.
```
$ cat /etc/udev/rules.d/99-monitor-hub-switch.rules
ACTION=="add", KERNEL=="1-7.1", SUBSYSTEM=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="5409", RUN+="/usr/bin/ddcutil setvcp 0x60 0x11"
ACTION=="remove", KERNEL=="1-7.1", SUBSYSTEM=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="5409", RUN+="/usr/bin/ddcutil setvcp 0x60 0x0f"
```
2. Reload udev rules and re-trigger.
```
$ sudo udevadm control --reload-rules
$ sudo udevadm trigger
```

## Set up Moonlight Logitech Extreme 3D joystick passhtrough on Moonlight

1. Find where your `gamecontrollerdb.txt` is stored. On Mac OS, it is in `~/Library/Caches/Moonlight Game Streaming Project/Moonlight/gamecontrollerdb.txt`. The following exerpt will map the joystick axes and buttons to a virtual Xbox Controller which Moonlight can forward. You can then assign the virtual Xbox Controller in your flight simulator settings.
```
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Windows,
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Mac OS X,
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Linux,
```
