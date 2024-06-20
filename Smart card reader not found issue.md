Although the `lsusb` command listed the smart card reader device but the middle-ware application that have to mediate between the browser plugin and the hardware device didn't find it. 

To resolve the issue follow these steps:

**Install required packages:**
Ensure you have all the necessary packages installed for smart card reader support.
```sh
sudo apt-get install pcscd pcsc-tools libccid libpcsclite1
```

**Start / Restart `pcscd` service:**
The `pcscd` service is responsible for communicating with smart card readers. Restart this service to ensure it's running properly.
```sh
sudo systemctl restart pcscd
sudo systemctl status pcscd
```

**Check Permissions:**
Ensure that your user has the necessary permissions to access the smart card reader. Add your user to the `scard` group (if available).
```sh
sudo usermod -aG scard $USER
```

**Verify Smart Card Reader Detection:** 
Use `pcsc_scan` to check if the smart card reader is detected properly.
```sh
pcsc_scan
```
