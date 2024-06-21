This method ensures that the USB device is automatically mounted at boot time or when the device is plugged in. Here's a step-by-step guide to set this up:

1. Find out the `UUID` of the device:
```bash
blkid

# Output
/dev/sdc1: UUID="58a6cda7-90cf-447e-8583-e0ae6eb0dc3e" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="ext-media" PARTUUID="a9f0e4a5-8d55-4805-be02-b919b384f04e"
```

2. Create a directory for the mount point
```bash
sudo mkdir /mnt/extmedia
```

3. Create `systemd` mount unit file
```bash
cat << EOF > /etc/systemd/system/mnt-extmedia.mount
[Unit]
Description=Mount external media contents

[Mount]
What=/dev/disk/by-uuid/58a6cda7-90cf-447e-8583-e0ae6eb0dc3e
Where=/mnt/extmedia
Type=ext4
Options=defaults,noatime

[Install]
WantedBy=multi-user.target
EOF
```
- **What**: The device identifier. Using the UUID is recommended. Replace `1234-5678` with the actual UUID of your device.
- **Where**: The mount point.
- **Type**: The filesystem type (replace with the correct type for your USB device, e.g., `ext4`, `ntfs`, etc.).
- **Options**: Mount options (`defaults,noatime` is a common choice).

`The unit file name must reflect the mount point path, replacing slashes (`/`) with dashes (`-`). For `/mnt/usb`, the unit file should be named `mnt-usb.mount.`

4. Reload `systemd`, enable and start the mount
```bash
sudo systemctl daemon-reload
sudo systemctl enable mnt-usb.mount
sudo systemctl start mnt-usb.mount
```

5. Check if the device is mounted correctly
```bash
df -h
```
