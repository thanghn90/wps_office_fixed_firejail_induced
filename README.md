# WPS Office with fixes (roll back to a working version) and run under firejail
This guide is for those who want to rebuild a .deb package of a working WPS office version for Linux, with firejail induced into the executables. The problem is, as I'm writing this guide, the official debian package (.deb) of WPS office for Linux on WPS website, version 11.1.0.10920.XA_amd64, does NOT work with Unikey for Vietnamese writing, including both ibus-unikey and fcitx-unikey (or any kind of different input methods for different languages). There is a slightly older build version on SNAP that does work with Unikey input methods (https://snapcraft.io/wps-2019-snap), but it's a SNAP package. If you have no issue installing and using a SNAP package, or have no issue with the official deb package, skip this guide. However, if you want to fix the official deb package by modifying it with the content of the working SNAP version (and add firejail integration to prevent the software from connecting to the internet and send your private data to baidu), continue to read on.

# Step-by-step Instruction
1. Download the official deb package from WPS Office website, version 11.1.0.10920.XA_amd64: https://wdl1.pcfg.cache.wpscdn.com/wpsdl/wpsoffice/download/linux/10920/wps-office_11.1.0.10920.XA_amd64.deb

2. Extract the deb package using the following command:

<code>cd ~/Downloads</code>

<code>mkdir tmp</code>

<code>sudo dpkg-deb -R wps-office_11.1.0.10920.XA_amd64.deb ./tmp</code>

<code>sudo rm -rf ~/Downloads/tmp/opt/</code>

This will extract the content of the .deb package into your ~/Downloads/tmp directory, assuming that you downloaded deb file is located in ~/Downloads, and then remove the opt directory of that deb package (which is the broken version, by the way).

Source: https://unix.stackexchange.com/questions/138188/easily-unpack-deb-edit-postinst-and-repack-deb

3. Download the wps-2019-snap package using the following command:

<code>snap download wps-2019-snap</code>

This will download the snap package to your current directory. All hail to @cyrpaut: https://github.com/cyrpaut/wps-2019-snap for hosting an actually working version of WPS Office.

4. Make a directory and mount the snap package as a squashfs partition:

<code>mkdir snap_wps</code>

<code>sudo mount -t squashfs -o ro wps-2019-snap.snap ./snap_wps</code> (note: your snap package name may vary)

This will mount the snap package in ./snap_wps directory.

Source: https://askubuntu.com/questions/1162798/how-do-i-view-the-contents-of-a-snap-file

5. Now, simply copy the opt directory under the mounted snap package into the tmp directory where the deb package was extracted, of course, as root (to preserve file permissions):

<code>sudo cp -r ./snap_wps/opt ~/Downloads/tmp/</code>

6. Rebuild the deb package using the following command:

<code>sudo dpkg-deb -b ~/Downloads/tmp wps-office-9125.deb</code>

This will take a while (~15min on my puny ASUS X202E 2013 minilaptop), and will make your CPU runs quite intensively due to compression, so be patient.

7. Install the fixed deb package:

<code>sudo dpkg -i wps-office-9125.deb</code>

8. Install WPS Office fonts by downloading from https://github.com/BannedPatriot/ttf-wps-fonts (credit: @BannedPatriot) and run the install.sh as root:

<code>sudo ./install.sh</code>

9. Install Microsoft fonts (Arial, Tahoma, Times New Roman, etc...):

<code>sudo apt install ttf-mscorefonts-installer</code>

10. (Extra) Install firejail and reboot

<code>sudo apt install firejail</code>

<code>sudo reboot</code>

11. (Still extra) Download the four firejail-induced executables of WPS Office from this git repository and overwrite them to your /usr/bin/, then change permission of those four files to 755 of those files:

<code>chmod 755 et wps wpp wpspdf</code>

This will ensure the WPS softwares cannot connect to the internet. Credit to a reddit folk: https://www.reddit.com/r/linux/comments/lhb8pd/using_wps_office_or_any_app_you_want_with_blocked/


12. (for Vietnamese typing) Use fcitx-unikey instead of ibus-unikey or ibus-bamboo if you want to type Vietnamese in WPS Office. ibus cause significant slowdown during WPS Office startup (I suspect it's due to gtk2 implementation of ibus or some kind of qt-gtk incompatibility within ibus module. At once, I thought I was able to fix it by installing appmenu-gtk2-module (https://askubuntu.com/questions/1237747/ubuntu-20-04-applications-take-too-long-to-start-up), but after a reboot, it's slow again, and removing+reinstalling appmenu-gtk2-module does not work anymore). I'll write a separate guide just to instruct you how to install and config fcitx-unikey.
