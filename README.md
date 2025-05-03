# 🛠️ Enabling Monitor Mode on Raspberry Pi Zero 2 W with Pwnagotchi Firmware (With packet injection working)

This guide walks you through extracting working Wi-Fi firmware from a Pwnagotchi image and using it to enable monitor mode on Kali Linux running on a Raspberry Pi Zero 2 W.

# 📥 1. Download Kali Linux for Raspberry Pi Zero 2 W:
Go to the official Kali ARM images page:

👉 https://www.kali.org/get-kali/#kali-arm

Scroll down and download the image for Raspberry Pi Zero 2 W (Pi-Tail)

![image](https://github.com/user-attachments/assets/4efd2671-11e8-4b1e-8cf1-66f75f73f722)

# 💾 2. Flash the SD Card:
Flash the Kali image to your SD card using your preferred tool. I recommend using Raspberry Pi Imager for simplicity.

![image](https://github.com/user-attachments/assets/661c759c-7c6c-4dee-b1a2-fc96ec63da07)

# 🤖 3. Download a Working Pwnagotchi Image:
Download the working Pwnagotchi image that supports the Raspberry Pi Zero 2 W.
The one used in this guide is:

👉 https://github.com/jayofelony/pwnagotchi/releases/download/v2.9.5.3/pwnagotchi-2.9.5.3-64bit.img.xz

![image](https://github.com/user-attachments/assets/8ce802dd-327f-4bae-bd65-762102a37194)

# 🗂️ 4. Mount the Pwnagotchi .img on Linux
After extracting the .img.xz file, you can mount the image using loop devices:

### Create a temporary directory
```
mkdir /mnt/pwnagotchi
```
### Use 'fdisk' or 'partx' to find partition offsets
```
fdisk -l pwnagotchi-2.9.5.3-64bit.img
```
### Example mount (replace offsets accordingly)
```
sudo losetup -P /dev/loop0 pwnagotchi-2.9.5.3-64bit.img
sudo mount /dev/loop0p2 /mnt/pwnagotchi
```
![image](https://github.com/user-attachments/assets/2340d1da-ea61-4206-8aac-54632c1225ba)


# 📦 5. Compress the Required Firmware Folders
Navigate to the mounted image and compress the following folders:

/lib/firmware/brcm/

/lib/firmware/cypress/
```
cd /mnt/pwnagotchi/lib/firmware/
sudo tar czvf firmware_folders.tar.gz brcm cypress
```
![image](https://github.com/user-attachments/assets/f74c6aba-9222-479c-bdab-69ee6eec422d)

![image](https://github.com/user-attachments/assets/633d5d89-188a-495f-81fe-2cd296d7c0d9)

# 📁 6. Save the Archive
Move the compressed file to a safe location for transfer later:
```
mv firmware_folders.tar.gz ~/Desktop/
```
![image](https://github.com/user-attachments/assets/4acb1663-2264-4669-8e5d-e87565d8df1e)

# 🔌 7. Connect Raspberry Pi via Ethernet-over-USB
Connect your Raspberry Pi Zero 2 W to your computer via Ethernet over USB and SSH into it. This may not be enabled by default, but you can google it or ask chatGPT how to enable it. It's quite simple, but I won't go into details because this tutorial isn't about how to enable Ethernet over USB. (Credits to [rudyrdx](https://github.com/rudyrdx/Raspberry-Pi-Zero-2w/blob/main/Monitor%20Mode.md) for this image)

![image](https://github.com/user-attachments/assets/163e67f1-4a15-4b96-90db-999a780b59a4)

# 📤 8. Transfer the Firmware to Your Raspberry Pi
Use FileZilla or any SCP/SFTP client of your choice to transfer the firmware_folders.tar.gz archive to your Raspberry Pi.

# ⚙️ 9. Replace and Extract Firmware on the Pi
SSH into your Pi and run:
```
sudo su
rm -rf /lib/firmware/brcm/
```
Now extract the firmware archive:
```
cd /lib/firmware/
tar xzvf /path/to/firmware_folders.tar.gz
```
This will recreate both brcm/ and cypress/ directories in /lib/firmware.

# 🔗 10. Create Required Symlinks

Ensure that any symbolic links needed by the system are recreated (if applicable).
You can check this running inside /mnt/pwnagotchi/lib/firmware/brcm/ the next command:
```
ls -l
```

![image](https://github.com/user-attachments/assets/231f52b4-eab7-44a3-9296-e672e0b1df85)

and also you can use this code to generate the symlinks on your PI:

```
#!/bin/bash

# Path to the directory where the symlinks should go
DEST_DIR="/lib/firmware/brcm"

# List of symlinks in the format: link -> target
declare -A symlinks=(
    ["BCM-0a5c-6410.hcd"]="BCM-0bb4-0306.hcd"
    ["BCM43430A1.raspberrypi,3-model-b.hcd"]="BCM43430A1.hcd"
    ["BCM43430A1.raspberrypi,model-zero-2-w.hcd"]="../synaptics/SYN43430A1.hcd"
    ["BCM43430A1.raspberrypi,model-zero-w.hcd"]="BCM43430A1.hcd"
    ["BCM43430B0.raspberrypi,model-zero-2-w.hcd"]="../synaptics/SYN43430B0.hcd"
    ["BCM4345C0.raspberrypi,3-model-a-plus.hcd"]="BCM4345C0.hcd"
    ["BCM4345C0.raspberrypi,3-model-b-plus.hcd"]="BCM4345C0.hcd"
    ["BCM4345C0.raspberrypi,4-compute-module.hcd"]="BCM4345C0.hcd"
    ["BCM4345C0.raspberrypi,4-model-b.hcd"]="BCM4345C0.hcd"
    ["BCM4345C0.raspberrypi,5-model-b.hcd"]="BCM4345C0.hcd"
    ["BCM4345C5.raspberrypi,400.hcd"]="BCM4345C5.hcd"
    ["BCM4345C5.raspberrypi,4-compute-module.hcd"]="BCM4345C5.hcd"
    ["brcmfmac43012-sdio.bin"]="../cypress/cyfmac43012-sdio.bin"
    ["brcmfmac43012-sdio.clm_blob"]="../cypress/cyfmac43012-sdio.clm_blob"
    ["brcmfmac43340-sdio.bin"]="../cypress/cyfmac43340-sdio.bin"
    ["brcmfmac43362-sdio.bin"]="../cypress/cyfmac43362-sdio.bin"
    ["brcmfmac43362-sdio.lemaker,bananapro.txt"]="brcmfmac43362-sdio.cubietech,cubietruck.txt"
    ["brcmfmac4339-sdio.bin"]="../cypress/cyfmac4339-sdio.bin"
    ["brcmfmac43430-sdio.bin"]="../cypress/cyfmac43430-sdio.bin"
    ["brcmfmac43430-sdio.raspberrypi,3-model-b.bin"]="../cypress/cyfmac43430-sdio.bin"
    ["brcmfmac43430-sdio.raspberrypi,3-model-b.txt"]="brcmfmac43430-sdio.txt"
    ["brcmfmac43430-sdio.raspberrypi,model-zero-w.bin"]="../cypress/cyfmac43430-sdio.bin"
    ["brcmfmac43430-sdio.raspberrypi,model-zero-w.txt"]="brcmfmac43430-sdio.txt"
    ["brcmfmac43430-sdio.raspberrypi,model-zero-2-w.bin"]="brcmfmac43436s-sdio.bin"
    ["brcmfmac43430-sdio.raspberrypi,model-zero-2-w.txt"]="brcmfmac43436s-sdio.txt"
    ["brcmfmac43430b0-sdio.raspberrypi,model-zero-2-w.bin"]="brcmfmac43436-sdio.bin"
    ["brcmfmac43430b0-sdio.raspberrypi,model-zero-2-w.clm_blob"]="brcmfmac43436-sdio.clm_blob"
    ["brcmfmac43430b0-sdio.raspberrypi,model-zero-2-w.txt"]="brcmfmac43436-sdio.txt"
)

echo "Creating symlinks in $DEST_DIR..."
cd "$DEST_DIR" || exit 1

for link in "${!symlinks[@]}"; do
    target="${symlinks[$link]}"
    echo "ln -sf $target $link"
    ln -sf "$target" "$link"
done

echo "All symlinks were created."
```

# ✅ 11. Test Monitor Mode
Reboot your Raspberry Pi with the new firmware installed. Make sure it's powered via its normal power supply (not Ethernet over USB).

Once booted:
```
sudo airmon-ng start wlan0
```

If successful, this will create the wlan0mon interface in monitor mode.

![image](https://github.com/user-attachments/assets/d0ccd453-513b-4c20-b38b-bb5588f2c040)


# 🎉 Done!
You should now have full monitor mode support on your Kali Linux running on a Raspberry Pi Zero 2 W using firmware extracted from a Pwnagotchi image.

