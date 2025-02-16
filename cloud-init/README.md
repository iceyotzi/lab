# cloud-init
Create ISO images with preferred `cloud-init` profile baked into it.

reference: [Automating Custom ISO with `Cloud-Init`](https://dev.to/otomato_io/automating-custom-iso-with-cloud-init-2lc2)

## prerequisites
```bash
// install required packages
sudo apt update
sudo apt install p7zip-full p7zip-rar genisoimage fakeroot xorriso isolinux binutils squashfs-tools
// download and unzip ubuntu 24.04 server iso
curl -X GET -OL https://releases.ubuntu.com/noble/ubuntu-24.04.1-live-server-amd64.iso
7z x -y ubuntu-24.04.1-live-server-amd64.iso -oiso
``` 

```bash
sed -i -e 's/---/ autoinstall  ---/g' iso/boot/grub/grub.cfg
sed -i -e 's/---/ autoinstall  ---/g' iso/boot/grub/loopback.cfg
sed -i -e 's,---, ds=nocloud\\\\\\;s=/cdrom/nocloud/  ---,g'  iso/boot/grub/grub.cfg
sed -i -e 's,---, ds=nocloud\\\\\\;s=/cdrom/nocloud/  ---,g' iso/boot/grub/loopback.cfg
```

I created the required files
```
mkdir -p iso/nocloud
touch iso/nocloud/{meta-data,user-data}
```

## user-data
```bash
xorriso -as mkisofs \
  -V "Ubuntu Install CD" \
  --modification-date='2024042312460900' \
  --grub2-mbr --interval:local_fs:0s-15s:zero_mbrpt,zero_gpt:"$UBUNTU_ISO" \
  --protective-msdos-label \
  -partition_cyl_align off \
  -partition_offset 16 \
  --mbr-force-bootable \
  -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b --interval:local_fs:5370020d-5380159d::"$UBUNTU_ISO" \
  -appended_part_as_gpt \
  -iso_mbr_part_type a2a0d0ebe5b9334487c068b6b72699c7 \
  -c '/boot.catalog' \
  -b '/boot/grub/i386-pc/eltorito.img' \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot-info-table \
  --grub2-boot-info \
  -eltorito-alt-boot \
  -e '--interval:appended_partition_2_start_1342505s_size_10140d:all::' \
  -no-emul-boot \
  -boot-load-size 10140 \
  -o "$out_iso" \
  /contents
```
