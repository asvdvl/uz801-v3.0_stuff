# uz801-v3.0_stuff
this repository was created because I could not find the original firmware for the specified modem. so I created this repository similar to [this repository](https://github.com/Mio-sha512/openstick-stuff). As I know, firmware from version 2.1 is not quite suitable for it, so I will post everything that works in [releases](https://github.com/asvdvl/uz801-v3.0_stuff/releases).

# edl pins + tx uart(1.8v) [source](https://wiki.postmarketos.org/wiki/Zhihe_series_LTE_dongles_(generic-zhihe))
<details>
  <summary>Image</summary>
  https://wiki.postmarketos.org/wiki/File:Uz801_board.jpg
  <img src="https://wiki.postmarketos.org/images/thumb/0/00/Uz801_board.jpg/800px-Uz801_board.jpg">
</details>

# openwrt firmware status
<details>
  <summary>steps to reproduce</summary>
  to begin with, I mean that you have a stock image and you have a backup of it (in my case, the boot and system are corrupted)
  also need to save fsc, fsg, modemst1, modemst2 partitions.
  however, there are problems with wrt, regarding compilation, I still couldn’t compile wireguard, so I’m trying debian for now
  
  ```bash
  edl rf backup.bin \

  edl r fsc,fsg,modemst1,modemst2 fsc.bin,fsg.bin,modemst1.bin,modemst2.bin
  ```
  compile openwrt, the steps are taken from <a href="https://github.com/chautruongthinh/OpenWrt_UZ801V3">this repository</a> (github actions) as well as the config
  
  ```bash
  git clone https://github.com/chautruongthinh/HandsomeMod ./hsm_uzv3
  cd ./hsm_uzv3
  ./scripts/feeds update -a
  ./scripts/feeds install -a
  make download -j8
  make -j$(nproc)
  cd ./bin/targets/msm89xx/msm8916/
  ```

  next, I took the image from version 2.1 and uploaded it as is to the modem
  after which I restored the partitions and uploaded openwrt
  ```bash
  #download and extract https://github.com/Mio-sha512/openstick-stuff/releases/download/Firmware/uz801_v2.1_openwrt.zip  (or my image from releases)
  #edl wf uz801_v2.1_openwrt.bin
  #edl reset
    
  edl w fsc fsc.bin
  edl w fsg fsg.bin
  edl w modemst1 modemst1.bin
  edl w modemst2 modemst2.bin
    
  edl w boot handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-boot.img
  simg2img handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-system.img rootfs.img
  edl w rootfs rootfs.img
  ```
  however, there are problems with wrt, regarding compilation, I still couldn’t compile wireguard, so I’m trying debian for now
</details>

# debian firmware status
<details>
  <summary>steps to reproduce</summary>
  first of all, thanks to these guys from this issue https://github.com/OpenStick/OpenStick/issues/46 for the port
  
  sources of instructions: 
  - https://github.com/OpenStick/OpenStick/issues/46#issuecomment-1641749343 (custom aboot, and I haven't checked the one that comes with base-generic.zip)
  - https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/README (basic reinstallation instructions)
  
  you should download (and unzip some) these files:
  - [base.zip (unpack)](https://github.com/OpenStick/OpenStick/releases/download/v1/base.zip)  I used base-generic.zip (by accident) and it works, but you need to use the file from the link, if that doesn't work, you can try this one
  - [debian.zip (unpack)](https://github.com/OpenStick/OpenStick/releases/download/v1/debian.zip)
  - [boot.img without hddled](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/kernel/boot.img-5.15.0-orbital+) !OR! [boot.img with hddled](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/kernel/boot.img-5.15.0-orbital+-hddled)
  - [linux-image](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/kernel/linux-image-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb)
  - [linux-headers](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/kernel/linux-headers-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb)
  - [wifi stuff](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/modem_wifi/WCNSS_qcom_wlan_nv.bin)
  - [modem stuff](https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/modem_wifi/uz801_v30_modem.bin)
  
  if you cannot enter fastboot (like me) then you need to follow the steps from the 1st section of the base/flash.sh file but for edl instead of fastboot
```
edl rf backup.bin #this is not in the script but it is useful to have
```

```
wget https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/boot/aboot-uz801.bin
edl e boot
edl w aboot aboot-uz801.bin
edl reset
```
  Now the modem should go into fastboot mode, check with the command below, one device should be displayed (if more, then unplug all unnecessary ones)
```
fastboot devices
```
  now run
```
cd ./base
./flash.sh
cd ../debian
./flash.sh
```
  now push this files 
  - boot.img-5.15.0-orbital+(or boot.img-5.15.0-orbital+-hddled)
  - linux-image-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb
  - linux-headers-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb
  - uz801_v30_modem.bin
  - WCNSS_qcom_wlan_nv.bin

  to modem via sftp(login/password below) or adb push to home folder(/home/user)
  log in via ssh or adb
```
ssh user@192.168.68.1 #password: 1
```
  or
```
adb shell
```
  now run
```
sudo -i
dpkg -i linux-image-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb 
dpkg -i linux-headers-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb
dd if=boot.img of=/dev/mmcblk0p12
reboot
```
Install firmware for Modem and WiFi
```
cp WCNSS_qcom_wlan_nv.bin /lib/firmware/wlan/prima/
mount -o loop -t vfat uz801_v30_modem.bin /mnt
cd /mnt/image/
sudo cp mba.mbn /lib/firmware/
sudo cp modem.* /lib/firmware/
sudo cp wcnss.* /lib/firmware/
# In modem_pr/mcfg/configs/mcfg_sw/generic/, look for the appropriate mcfg_sw.mbn for your region (and carrier if applicable). Copy it to /lib/firmware
# in my case
cp modem_pr/mcfg/configs/mcfg_sw/generic/eu/ee/commerci/mcfg_sw.mbn /lib/firmware
reboot
```
You can fix missing dnsmasq configuration after upgrading, [with instructions from @xiv3r](https://github.com/asvdvl/uz801-v3.0_stuff/issues/1#issue-3067616139)
```
nano /etc/dnsmasq.conf

dhcp-range=interface:wlan0,192.168.100.100,192.168.100.150,12h
dhcp-range=interface:usb0,192.168.200.100,192.168.200.150,12h

ssh user@192.168.200.1 - RNDIS
ssh user@192.168.100.1 - AP
```
That’s all, then you can install the software either by connecting to wifi or using a mobile network(may not work)
```
sudo apt update
sudo apt install nano zsh git curl iproute2
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
omz plugin enable debian systemd history
```
</details>
