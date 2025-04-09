# wyse

### QMX

aw@solar:~$ udevadm info -a -n /dev/ttyACM0 | grep -E "idVendor|idProduct|KERNELS"
    KERNELS=="1-3:1.0"
    KERNELS=="1-3"
    ATTRS{idProduct}=="a34c"
    ATTRS{idVendor}=="0483"
    KERNELS=="usb1"
    ATTRS{idProduct}=="0002"
    ATTRS{idVendor}=="1d6b"
    KERNELS=="0000:00:15.0"
    ATTRS{dbc_idProduct}=="0010"
    ATTRS{dbc_idVendor}=="1d6b"
    KERNELS=="pci0000:00"
    
### Winkeyer Large

aw@solar:~$ udevadm info -a -n /dev/ttyUSB0 | grep -E "idVendor|idProduct|KERNELS"
    KERNELS=="ttyUSB0"
    KERNELS=="1-6.2:1.0"
    KERNELS=="1-6.2"
    ATTRS{idProduct}=="7523"
    ATTRS{idVendor}=="1a86"
    KERNELS=="1-6"
    ATTRS{idProduct}=="5415"
    ATTRS{idVendor}=="0bda"
    KERNELS=="usb1"
    ATTRS{idProduct}=="0002"
    ATTRS{idVendor}=="1d6b"
    KERNELS=="0000:00:15.0"
    ATTRS{dbc_idProduct}=="0010"
    ATTRS{dbc_idVendor}=="1d6b"
    KERNELS=="pci0000:00"



## **1. Identify Devices and Ports**

### **Step 1: Find `idVendor`, `idProduct`, and Physical Port Path**

For each device:

1. Plug the device into the desired USB port.
2. Run:

```bash
udevadm info -a -n /dev/ttyACM0 | grep -E "idVendor|idProduct|KERNELS"
```

Example output:

```
ATTRS{idVendor}=="0483"
ATTRS{idProduct}=="a34c"
KERNELS=="1-3"  # Physical port path (e.g., bus 1, port 3)
```


Repeat for all devices, noting their `idVendor`, `idProduct`, and `KERNELS` (port path).

---

## **2. Create Persistent Udev Rules**

### **Step 2: Define udev Rules for Each Device**

Create a file for each device in `/etc/udev/rules.d/` (e.g., `99-radio.rules`, `99-arduino.rules`):

```bash
sudo nano /etc/udev/rules.d/99-radio.rules
```

Add rules like this:

```bash
# QMX Radio (Port 1-3)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="1-3", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="a34c", SYMLINK+="ttyQMX", MODE="0666", GROUP="dialout"

# Arduino (Port 1-4)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="1-4", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", SYMLINK+="ttyArduino", MODE="0666", GROUP="dialout"

# Pi Pico (Port 2-1)
ACTION=="add", SUBSYSTEM=="tty", KERNELS=="2-1", ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="000a", SYMLINK+="ttyPico", MODE="0666", GROUP="dialout"
```


### **Step 3: Reload Udev Rules**

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## **3. Create Systemd Services**

### **Step 4: Define Templated Systemd Service Files**

Create a template service file (e.g., `/etc/systemd/system/rigctld@.service`):

```bash
sudo nano /etc/systemd/system/rigctld@.service
```

Add:

```ini
[Unit]
Description=Rigctld for %i
BindsTo=dev-%i.device
After=dev-%i.device

[Service]
ExecStart=/usr/local/bin/rigctld -m 2057 -r /dev/%i -t 4533 -s 115200
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


### **Step 5: Enable Services for Each Device**

```bash
sudo systemctl enable rigctld@ttyQMX
sudo systemctl enable rigctld@ttyArduino
sudo systemctl enable rigctld@ttyPico
```

---

## **4. Verify and Troubleshoot**

### **Step 6: Check Symlinks and Services**

- Confirm symlinks exist:

```bash
ls -l /dev/ttyQMX /dev/ttyArduino /dev/ttyPico
```

- Check service status:

```bash
systemctl status rigctld@ttyQMX
journalctl -u rigctld@ttyQMX
```


### **Step 7: Test Communication**

Use `minicom` or `screen` to verify device access:

```bash
sudo minicom -D /dev/ttyQMX -b 115200
```

---

## **5. Advanced Configuration**

### **Handling Multiple Instances**

- **Unique Ports**: Assign each device to a dedicated physical USB port.
- **Service Ports**: Use different TCP ports (e.g., `4533`, `4534`) for each `rigctld` instance.


### **Auto-Start on Plug-In**

Use `udev` rules to trigger services dynamically:

```bash
ACTION=="add", SUBSYSTEM=="tty", ..., TAG+="systemd", ENV{SYSTEMD_WANTS}="rigctld@%k.service"
```

---

## **Final Notes**

- **Physical Port Binding**: Use `KERNELS` in udev rules to tie devices to specific ports, avoiding conflicts.
- **Permissions**: Ensure `MODE="0666"` and `GROUP="dialout"` for non-root access.
- **Scalability**: Use systemd templates to manage multiple devices with minimal configuration.

By following this workflow, you’ll have a robust setup where each device is persistently mapped to its designated port and managed by dedicated services. Let me know if you need further refinements!

<div>⁂</div>

[^1]: https://acroname.com/blog/how-find-vendor-id-and-product-id-your-usb-device

[^2]: https://unix.stackexchange.com/questions/566682/get-the-device-path-of-usb-given-device-id

[^3]: https://wiki.domoticz.com/PersistentUSBDevices

[^4]: https://www.derekdemuro.com/2020/06/06/finding-usb-devices-with-lsusb-idvendor-idproduct/

[^5]: https://itsfoss.com/list-usb-devices-linux/

[^6]: https://unix.stackexchange.com/questions/346259/how-to-get-a-usb-device-path-by-its-idvendor-and-idproduct

[^7]: https://askubuntu.com/questions/349995/find-usb-devices-directory-sys-bus-usb-devices-using-idvendor-idproduct

[^8]: https://stackoverflow.com/questions/70513926/how-to-get-the-vendor-and-product-id-of-a-usb-device-from-sys-bus-devices-usb-d

[^9]: https://askubuntu.com/questions/987966/how-do-i-get-the-usb-drive-path-and-label-by-vendor-id-and-product-id

[^10]: https://askubuntu.com/questions/56938/getting-information-on-my-usb-devices

[^11]: https://www.baeldung.com/linux/check-for-usb-devices

[^12]: https://stackoverflow.com/questions/3279800/how-to-get-usb-vendor-and-product-info-programmatically-on-linux

[^13]: https://stackoverflow.com/questions/5565848/how-to-find-the-base-address-of-usb-to-parallel-port-device-in-linux

[^14]: https://www.reddit.com/r/linuxquestions/comments/4m6l1j/multiple_devices_with_same_product_id_vendor_id/

[^15]: https://unix.stackexchange.com/questions/747084/does-every-usb-device-have-a-vendor-id-and-product-id

[^16]: https://www.linuxquestions.org/questions/linux-newbie-8/how-to-detect-usb-devices-like-mp3-players-571705/

[^17]: https://superuser.com/questions/983399/how-to-uniquely-identify-a-usb-device-in-linux

[^18]: https://superuser.com/questions/645459/does-xubuntu-keep-a-log-of-all-usb-devices-ever-connected-to-the-system

[^19]: https://wiki.debian.org/HowToIdentifyADevice/USB

[^20]: https://itsfoss.com/find-mac-address-linux/

