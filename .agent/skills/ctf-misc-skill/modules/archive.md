# 📦 压缩包分析模块

## 适用文件类型
- ZIP / RAR / 7Z / TAR / GZ

## 检查清单

```yaml
检查项:
  - [ ] 伪加密（ZIP flag 位修改）
  - [ ] 明文攻击（已知部分明文）
  - [ ] CRC32 碰撞爆破（小文件内容）
  - [ ] 多层套娃（递归解压）
  - [ ] 注释字段隐藏信息
  - [ ] 密码本爆破
  - [ ] NTFS 交换数据流（Windows）
  - [ ] 损坏的压缩包修复
  - [ ] 分卷压缩包合并

常用工具:
  - unzip, 7z, rar
  - fcrackzip (ZIP 密码爆破)
  - john, hashcat
  - bkcrack (明文攻击)
  - zipdetails (ZIP 结构分析)
```

## 分析流程

### Step 1: 基础检查
```bash
# 查看压缩包内容（不解压）
unzip -l archive.zip
7z l archive.7z
rar l archive.rar

# 查看注释
unzip -z archive.zip

# 查看详细结构
zipdetails archive.zip
```

### Step 2: 伪加密检测
```bash
# 使用脚本自动修复
python3 scripts/zip_fake_encrypt.py archive.zip

# 手动检查（十六进制编辑器）
# 查看 0x06-0x07 位置的加密标志位
```

### Step 3: 密码爆破
```bash
# fcrackzip（快速）
fcrackzip -u -D -p wordlist.txt archive.zip

# John the Ripper
zip2john archive.zip > hash.txt
john --wordlist=wordlist.txt hash.txt

# Hashcat（GPU 加速）
hashcat -m 13600 -a 0 hash.txt wordlist.txt
```

### Step 4: CRC32 爆破（小文件）
```python
import zipfile
import binascii

def crack_crc32(target_crc, max_length=6):
    """爆破小文件内容（纯数字/字母）"""
    import itertools
    import string
    
    chars = string.ascii_letters + string.digits
    for length in range(1, max_length + 1):
        for attempt in itertools.product(chars, repeat=length):
            data = ''.join(attempt).encode()
            if binascii.crc32(data) & 0xffffffff == target_crc:
                return data.decode()
    return None

# 从 ZIP 中获取 CRC32
with zipfile.ZipFile('archive.zip') as zf:
    for info in zf.infolist():
        print(f"{info.filename}: CRC32 = {info.CRC:08x}")
        if info.file_size < 10:  # 小文件
            result = crack_crc32(info.CRC)
            if result:
                print(f"Found: {result}")
```

### Step 5: 明文攻击
```bash
# 使用 bkcrack（需要已知部分明文）
# 1. 创建包含已知明文的 ZIP
zip plain.zip known_file.txt

# 2. 执行明文攻击
bkcrack -C encrypted.zip -c target.txt -P plain.zip -p known_file.txt

# 3. 使用恢复的密钥解密
bkcrack -C encrypted.zip -k <keys> -D decrypted.zip
```

## 常见出题套路

1. **伪加密** → `scripts/zip_fake_encrypt.py` 一键修复
2. **弱密码** → fcrackzip + rockyou.txt
3. **CRC32 碰撞** → 小文件爆破脚本
4. **明文攻击** → bkcrack（需要部分已知文件）
5. **多层套娃** → 递归解压脚本
6. **注释隐藏** → `unzip -z` 查看

## 脚本参考

详见 `scripts/zip_fake_encrypt.py`
