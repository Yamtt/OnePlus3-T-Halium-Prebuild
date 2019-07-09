# OnePlus3(T)-Halium-Prebuild

## what works what don't work?
  * [x] boot
  * [x] graphics
  * [ ] calls
  * [x] 3/4G
  * [x] SMS (in/out)
  * [x] sound
  * [ ] gps (only the frontend)
  * [x] wifi
  * [ ] Bluetooth
  * [ ] Camera
  * [x] Anbox
  * [x] Rotation


### For Ubuntu Touch

before the installation, you'll need to format your userdata and your cache partition to ext4. Be careful, it'll erase all your personal files (like pictures, etc...)

just use the prebuild image just as  a normal compiled image.
install it with the JBB's halium-install script [here](https://github.com/JBBgameich/halium-install)
and get the ubports rootfs from [here](https://ci.ubports.com/job/xenial-rootfs-armhf/lastSuccessfulBuild/artifact/out/ubports-touch.rootfs-xenial-armhf.tar.gz)
```./halium-install -p ut ubports-touch.rootfs-xenial-armhf.tar.gz system.img```
```sudo fastboot flash boot halium-boot.img```

then while in TWRP
```adb shell 'touch /data/.writable_image; mkdir /a; mount /data/rootfs.img /a; echo manual | tee /a/etc/init/rsyslog.override;  touch /a/.writable_device_image; umount /a; sync'```


some command are needed in order to get a fully working UT device (run as root).
```
chmod 666 /dev/kgsl-3d0
adduser --force-badname --system --home /nonexistent --no-create-home --quiet _apt

sudo apt install pulseaudio-modules-droid-24
```

#### Anbox
```
sudo apt update
sudo apt install anbox-ubuntu-touch
```
reboot the phone and then
```
anbox-tool install
```
and finally, reboot your phone twice (why twice? no idea)

### Known Issues
* Sometimes it'll not boot (it provide only telnet), the work-around initialy was to go to twrp then reboot -> system. it seems to not work anymore
* A little freeze (1-2 sec) occur when trying to wakeup the phone (already fixed but not yet published).
* The phone heat a lot while using Anbox.

### how to compile

follow http://docs.halium.org/en/latest/porting/first-steps.html

and before ```setup oneplus3```,
replace halium/devices/manifests/oneplus_oneplus3.xml
by https://gist.github.com/Vince1171/e1aa5fda8dda213f909f71ef70de0792

(I haven't pushed my work to the halium repo as it seems that I've break plasma mobile)

```
cd halium/libhybris/ && git remote add ubports https://github.com/ubports/libhybris.git && git fetch ubports && git checkout ubports/hybris-compat-layer-fixes
cd halium/ && git clone https://github.com/ubports/platform-api.git && cd platform-api && git checkout xenial
```

in ```build/core/main.mk```
change
```
# Specific projects for Halium
subdirs += \
        halium/hybris-boot \
        halium/droidmedia \
        halium/halium-boot
```
into
```
# Specific projects for Halium
subdirs += \
    halium/hybris-boot \
    external/droidmedia \
    halium/halium-boot \
    external/audioflingerglue \
    halium/platform-api \
    halium/libhybris
```
finally go to ```external``` directory
```git clone https://github.com/mer-hybris/audioflingerglue.git```

```source build/envsetup.sh && breakfast halium_oneplus3-userdebug && mka halium-boot systemimage hybris-boot```
