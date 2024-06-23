To mount an USB device (eg. external HDD) follow the steps described here: [[Permanent mount USB device with systemd]]

To redirect the device to a KVM VM if it is starting follow the next points:

### Mount the USB device to the host machine with `systemd`
#### Find the device vendor and product ID's
```bash
lsusb -v -s <bus>:<device>
```
Replace `<bus>` and `<device>` with the bus and device numbers you noted earlier. Look for lines starting with `idVendor` and `idProduct`.
#### Create the XML configuration
```xml
<hostdev mode='subsystem' type='usb' managed='yes'>
  <source>
    <vendor id='0x<vendor_id>'/>
    <product id='0x<product_id>'/>
  </source>
</hostdev>
```
Then run
```bash
sudo virsh attach-device <vm-name> --file <path of the config file> --config
```
The command above runs with the `--config` option which apply the changes after next boot. Use `--live` if you want to apply the changes on the running domain.
##### Check inside the VM
- Log in to the VM.
- Check if the USB device is recognized by running commands like `lsusb`, `dmesg`, or `fdisk -l` depending on the type of device.

#### Following steps
For the rest follow the steps described in the [[Permanent mount USB device with systemd]] article inside the VM.