#!/bin/bash

# Create TMP folder

# Download latest Linux Mint MATE ISO
curl XXXXX > LatestMint.iso

# Extract all files from ISO to dir
bsdtar -C ./$destination -xf ./LatestMint.iso

# Copy preseed files to $destination
chmod +w -R ./isofiles/preseed
cp ./newCTA.seed ./isofiles/preseed/CTA.seed
chmod -w -R ./isofiles/preseed

# Copy isolinux.cfg to $destination
chmod +w -R ./isofiles/isolinux
cp ./isolinux.cfg ./isofiles/preseed/CTA.seed
chmod -w -R ./isofiles/isolinux

# Unsquash FS
cp ./isofile/casper
sudo unsquashfs ./filesystem.squashfs

# Copy skel to FS
cp -R ../../skel/* ./squashfs-root/etc/skel/

# Delete old squashFS
sudo rm ./filesystem.squashfs

# Generate new one
sudo mksquashfs ./squashfs-root ./filesystem.squashfs -comp xz

# Generate new .size file
chmod +w ./filesystem.size
printf $(sudo du -sx --block-size=1 ./squashfs-root) > ./filesystem.size
chmod -w ./filesystem.size

# Delete uncompressed FS
sudo rm -r ./squashfs-root

# Update MD5s of new ones
cd ../

chmod +w ./MD5SUMS
find -type f -print0 | sudo xargs -0 md5sum | grep -v isolinux/boot.cat > MD5SUMS
chmod -w ./MD5SUMS

# Make iso
cd ..
genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o CTA-Linux-Mint.iso isofiles





TODO (Ubiquity installer?)
Autostart
Set language English
Set keyboard to UK
Multimedia codecs: ubiquity/use_nonfree
AutomateDisk (LVM + crypt)
Set location / timezone
Disable help / guide prompt
Run apt update / upgrade

