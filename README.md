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
  
  ```bash
  edl rf ~/backup.bin \

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

</details>
