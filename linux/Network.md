#linux
```table-of-contents
```
# OSI 七層設定

![[Screenshot 2025-12-22 at 5.27.48 PM.png]]

防火牆主要是在第 2、3、4 層過濾封包，第二層包含了 MAC address，第三層包含了 IP，而第四層則包含了 port 號碼。

# TCP/IP

| Layer |     OSI      |    TCP/IP    |
|:-----:|:------------:|:------------:|
|   7   | Application  | Applicatyion |
|   5   | Presentation |      ^       |
|   5   |   Session    |      ^       |
|   4   |  Transport   |  Transport   |
|   3   |   Network    |   Network    |
|   2   |  Data-Link   |     Link     |
|   1   |   Physical   |      ^       |

# Layer 2

* 處理 frame
* 包含來源 MAC 及目的 MAC
	![[Screenshot 2025-12-25 at 7.47.43 PM.png]]

## MTU
Frame 所能傳送最大的資料量可達 1500 bytes，又稱為 MTU (Maximum Transmission Unit) 

# Layer 3
