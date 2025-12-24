# 📡 流量分析模块

## 适用文件类型
- PCAP / PCAPNG

## 检查清单

```yaml
检查项:
  - [ ] HTTP 文件传输还原
  - [ ] FTP/SMB 凭据提取
  - [ ] DNS 隧道 / ICMP 隧道
  - [ ] TCP 流重组
  - [ ] USB 键盘/鼠标流量解析
  - [ ] TLS 解密（如有私钥）
  - [ ] 无线抓包（802.11）
  - [ ] WebSocket / HTTP/2 流量
  - [ ] 异常协议识别

常用工具:
  - Wireshark (GUI 分析)
  - tshark (命令行)
  - tcpdump
  - scapy (Python)
  - NetworkMiner (文件提取)
  - foremost
```

## 分析流程

### Step 1: 基础信息
```bash
# 查看流量包统计
capinfos traffic.pcap

# 协议分布
tshark -r traffic.pcap -q -z io,phs

# 会话统计
tshark -r traffic.pcap -q -z conv,tcp
```

### Step 2: HTTP 分析
```bash
# 提取 HTTP 对象
tshark -r traffic.pcap --export-objects http,./http_objects

# 查看 HTTP 请求
tshark -r traffic.pcap -Y "http.request" -T fields -e http.request.uri

# 查看 HTTP 响应
tshark -r traffic.pcap -Y "http.response" -T fields -e http.file_data
```

### Step 3: FTP/SMB 凭据
```bash
# FTP 用户名密码
tshark -r traffic.pcap -Y "ftp.request.command == USER || ftp.request.command == PASS" -T fields -e ftp.request.arg

# SMB 文件传输
tshark -r traffic.pcap --export-objects smb,./smb_objects
```

### Step 4: DNS 隧道检测
```bash
# 查看 DNS 查询
tshark -r traffic.pcap -Y "dns.qry.name" -T fields -e dns.qry.name

# 提取 DNS 数据（可能是 Base64 编码）
tshark -r traffic.pcap -Y "dns.qry.name" -T fields -e dns.qry.name | cut -d'.' -f1 | base64 -d
```

### Step 5: USB 键盘流量
```bash
# 提取 USB 数据
tshark -r usb.pcap -T fields -e usb.capdata > usb_data.txt

# 使用脚本解析
python3 scripts/usb_keyboard.py usb_data.txt
```

### Step 6: TCP 流追踪
```bash
# 追踪特定 TCP 流
tshark -r traffic.pcap -q -z follow,tcp,ascii,0

# 或使用 Wireshark GUI
# 右键数据包 → Follow → TCP Stream
```

## 常见出题套路

1. **HTTP 文件传输** → `--export-objects` 提取
2. **USB 键盘** → `scripts/usb_keyboard.py` 解析
3. **DNS 隧道** → 提取域名前缀解码
4. **FTP 明文传输** → 直接提取用户名密码
5. **TCP 流重组** → Follow TCP Stream
6. **ICMP 隧道** → 提取 ICMP data 字段

## 脚本参考

详见 `scripts/usb_keyboard.py`

## Wireshark 过滤器速查

```
# HTTP
http.request.method == "POST"
http contains "flag"

# DNS
dns.qry.name contains "ctf"

# FTP
ftp.request.command == "PASS"

# USB
usb.capdata

# TCP 端口
tcp.port == 8080

# 包含特定字符串
frame contains "flag"
```
