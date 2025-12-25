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

## 无工具替代方案

当没有 Wireshark/tshark 时：

### 纯 Python 替代 (使用 scapy)

```python
#!/usr/bin/env python3
"""使用 scapy 分析 pcap（pip install scapy）"""

from scapy.all import *

# 读取 pcap
packets = rdpcap('traffic.pcap')

# 1. 基础统计
print(f"Total packets: {len(packets)}")

# 2. 提取 HTTP 数据
for pkt in packets:
    if pkt.haslayer('Raw'):
        payload = pkt['Raw'].load
        if b'HTTP' in payload or b'GET' in payload or b'POST' in payload:
            print(payload.decode('utf-8', errors='ignore'))

# 3. 提取所有字符串
for pkt in packets:
    if pkt.haslayer('Raw'):
        data = pkt['Raw'].load
        # 搜索 flag
        if b'flag' in data.lower():
            print(f"[+] Found in packet: {data}")

# 4. TCP 流重组
sessions = packets.sessions()
for session, pkts in sessions.items():
    data = b''.join(bytes(p['Raw'].load) for p in pkts if p.haslayer('Raw'))
    if b'flag' in data.lower():
        print(f"[+] Session {session}: {data}")
```

### 无依赖 Python (手动解析)

```python
#!/usr/bin/env python3
"""无依赖 pcap 解析（仅适用于简单情况）"""

def parse_pcap_simple(filename):
    """简单提取 pcap 中的字符串"""
    with open(filename, 'rb') as f:
        data = f.read()
    
    # 提取可打印字符串
    result = []
    current = []
    for byte in data:
        if 32 <= byte < 127:
            current.append(chr(byte))
        else:
            if len(current) >= 10:
                s = ''.join(current)
                if 'flag' in s.lower() or 'http' in s.lower():
                    result.append(s)
            current = []
    return result

# 使用
strings = parse_pcap_simple('traffic.pcap')
for s in strings:
    print(s)
```

### 在线工具替代

```yaml
PCAP 分析:
  - https://apackets.com/ - 在线 PCAP 分析
  - https://www.cloudshark.org/ - 云端 Wireshark
  - https://packettotal.com/ - 流量分析

文件提取:
  - 将 pcap 上传到在线分析工具
  - 使用 NetworkMiner (Windows GUI)
```

### 系统自带命令

```bash
# tcpdump (通常预装)
tcpdump -r traffic.pcap -A | grep -i flag
tcpdump -r traffic.pcap -X | head -100

# 字符串搜索
strings traffic.pcap | grep -iE "flag|password|user"

# hexdump 查看
hexdump -C traffic.pcap | head -50
```

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
