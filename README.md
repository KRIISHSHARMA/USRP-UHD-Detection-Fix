# **Issue and Solution: USRP B210 Not Detected by UHD**

## **Issue Overview**
When using a USRP B210 with UHD, the following error messages may appear during execution of tools like `uhd_usrp_probe` or GNU Radio:

```
[WARNING] [B200] EnvironmentError: IOError: Could not find path for image: usrp_b200_fw.hex

Using images directory: <no images directory located>

Set the environment variable 'UHD_IMAGES_DIR' appropriately or follow the below instructions to download the images package.

Please run:

"/lib/x86_64-linux-gnu/uhd/utils/uhd_images_downloader.py"
[ERROR] [UHD] Device discovery error: send_to: Required key not available
RuntimeError: LookupError: KeyError: No devices found for ----->
Empty Device Address
```

Despite the firmware file being present on the system (e.g., `/usr/local/share/uhd/images/usrp_b200_fw.hex`), UHD cannot locate the required files or recognize the device.

---

## **Root Cause**
1. UHD cannot find the firmware images because the `UHD_IMAGES_DIR` environment variable is not set or points to an incorrect directory.
2. USB permissions may not be correctly configured, preventing device access.

---

## **Solution Steps**

### **1. Verify Firmware Location**
Run the following command to locate the firmware files:
```bash
locate usrp_b200_fw.hex
```
Ensure the firmware files are in a directory like:
```
/usr/local/share/uhd/images/usrp_b200_fw.hex
```

### **2. Set `UHD_IMAGES_DIR` Environment Variable**
Inform UHD of the correct images directory:
```bash
export UHD_IMAGES_DIR=/usr/local/share/uhd/images
```
To make this change permanent, add the export command to your shell configuration file (e.g., `.bashrc` or `.zshrc`):
```bash
echo 'export UHD_IMAGES_DIR=/usr/local/share/uhd/images' >> ~/.bashrc
```
Reload your shell configuration:
```bash
source ~/.bashrc
```

### **3. Download UHD Images (Optional)**
If the images directory is missing files or outdated, re-download the UHD firmware package:
```bash
sudo /lib/x86_64-linux-gnu/uhd/utils/uhd_images_downloader.py
```

### **4. Verify UHD Configuration**
Check if UHD recognizes the USRP device:
```bash
uhd_find_devices
```
The output should list your USRP B210 device without errors.

### **5. Grant USB Permissions**
Ensure USB permissions are correctly configured:
1. Create a udev rule for Ettus devices:
   ```bash
   echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="2500", ATTR{idProduct}=="0020", MODE="0666"' | sudo tee /etc/udev/rules.d/10-usrp.rules
   ```
2. Reload the udev rules:
   ```bash
   sudo udevadm control --reload-rules && sudo udevadm trigger
   ```
3. Reconnect your USRP B210 device.

### **6. Test with `uhd_usrp_probe`**
Run the following command to verify that the device is detected:
```bash
uhd_usrp_probe
```
You should now see detailed information about your USRP B210 device.

---

## **Summary of Commands**
```bash
# Verify firmware location
locate usrp_b200_fw.hex

# Set UHD_IMAGES_DIR environment variable
export UHD_IMAGES_DIR=/usr/local/share/uhd/images

# Make the change permanent
echo 'export UHD_IMAGES_DIR=/usr/local/share/uhd/images' >> ~/.bashrc
source ~/.bashrc

# Download UHD images (if necessary)
sudo /lib/x86_64-linux-gnu/uhd/utils/uhd_images_downloader.py

# Grant USB permissions
sudo bash -c 'echo "SUBSYSTEM==\"usb\", ATTR{idVendor}==\"2500\", ATTR{idProduct}==\"0020\", MODE=\"0666\"" > /etc/udev/rules.d/10-usrp.rules'
sudo udevadm control --reload-rules && sudo udevadm trigger

# Test device detection
uhd_find_devices
uhd_usrp_probe
```

---

---

With this setup, UHD should successfully detect and initialize your USRP B210 device. If further issues arise, consult the [Ettus Knowledge Base](https://kb.ettus.com/) or UHD documentation.

