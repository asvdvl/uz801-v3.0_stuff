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
  
  ```bash
  edl wf ~/flash.bin \
  edl reset
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
  <details>
    <summary>edl</summary>
    
    edl w boot handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-boot.img
    edl w system handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-system.img


  </details>
  <details>
    <summary>fastboot</summary>
    this is the only block of code with fastboot, I will show the following only as edl but it is also possible with fastboot
    
    fastboot flash boot handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-boot.img
    fastboot flash system handsomemod-msm89xx-msm8916-Handsome_handsome-openstick-uz801v3-ext4-system.img


  </details>
  next steps i took from <a href="https://wiki.postmarketos.org/wiki/Zhihe_series_LTE_dongles_(generic-zhihe)">this wiki</a>
  
  flash sbl1 and tz
  
  ```bash
  mkdir dragon
  cd dragon
  wget https://releases.linaro.org/96boards/dragonboard410c/linaro/rescue/17.09/dragonboard410c_bootloader_emmc_android-88.zip
  unzip dragonboard410c_bootloader_emmc_android-88.zip
  edl w sbl1 sbl1.mbn
  edl w tz tz.mbn
  ```

  flash hyp
  
  ```bash
  cd ..
  git clone https://github.com/msm8916-mainline/qhypstub.git
  cd qhypstub; git clone https://github.com/msm8916-mainline/qtestsign.git
  make CROSS_COMPILE=aarch64-linux-gnu-
  edl w hyp qhypstub-test-signed.mbn
  ```

  flash aboot

  ```bash
  cd ..; git clone https://github.com/msm8916-mainline/lk2nd.git
  cd lk2nd; make LK1ST_DTB=msm8916-512mb-mtp LK1ST_COMPATIBLE=zhihe,uz801-v3 TOOLCHAIN_PREFIX=arm-none-eabi- lk1st-msm8916
  ../qhypstub/qtestsign/qtestsign.py aboot build-lk1st-msm8916/emmc_appsboot.mbn
  edl w aboot build-lk1st-msm8916/emmc_appsboot-test-signed.mbn
  ```
</details>
and I get a bootloop, look at the dmesg file for details, I got it from a tx contact
