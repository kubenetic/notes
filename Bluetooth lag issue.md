Bluetooth input devices and headphone has started to lag after PopOS! installed. The solution found in this post: https://askubuntu.com/a/1418151

The issue is not related to the Bluetooth timeout but more likely the USB auto suspend feature built into the kernel. This is how i went about fixing it:

1. Run command to find out the id of your Bluetooth module
```bash
$ lsusb -vt

/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/6p, 10000M
    ID 1d6b:0003 Linux Foundation 3.0 root hub
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/12p, 480M
    ID 1d6b:0002 Linux Foundation 2.0 root hub
    |__ Port 5: Dev 2, If 0, Class=Vendor Specific Class, Driver=, 12M
        ID 27c6:538d Shenzhen Goodix Technology Co.,Ltd. 
    |__ Port 6: Dev 3, If 0, Class=Video, Driver=uvcvideo, 480M
        ID 0bda:565a Realtek Semiconductor Corp. 
    |__ Port 6: Dev 3, If 1, Class=Video, Driver=uvcvideo, 480M
        ID 0bda:565a Realtek Semiconductor Corp. 
    |__ Port 10: Dev 4, If 0, Class=Wireless, Driver=btusb, 12M
        ID 8087:0aaa Intel Corp. Bluetooth 9460/9560 Jefferson Peak (JfP)
    |__ Port 10: Dev 4, If 1, Class=Wireless, Driver=btusb, 12M
        ID 8087:0aaa Intel Corp. Bluetooth 9460/9560 Jefferson Peak (JfP)
```

The id of my Bluetooth module is **8087:0aaa**

2. Create or update a udev rule to disable auto suspend for the module.
```bash
$ echo 'ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="8087", ATTR{idProduct}=="0aaa", ATTR{power/autosuspend}="-1"' >> /etc/udev/rules.d/50-usb_power_save.rules
```

After this reboot your PC and the lag should go away.

Note that **idVendor** was set to **8087** and **idProduct** was set to **0aaa** to reflect my bluetooth settings

More about the issue: [USB Auto-suspend](https://wiki.archlinux.org/title/Power_management#USB_autosuspend) 