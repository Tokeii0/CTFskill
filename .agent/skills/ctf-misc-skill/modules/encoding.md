# 🔠 编码与加密分析模块

## 适用场景
- 纯文本编码字符串
- 多层嵌套编码
- 古典密码

## 检查清单

```yaml
编码识别优先级:
  1. Base64 / Base32 / Base58 / Base85
  2. Hex / Binary / Octal
  3. URL Encoding / HTML Entities
  4. ROT13 / ROT47 / Caesar 全枚举
  5. Morse / Bacon / 培根密码
  6. 栅栏密码 / 维吉尼亚（需要 key 或频率分析）
  7. 多层嵌套编码（递归解码）

识别技巧:
  - Base64: [A-Za-z0-9+/=] 且长度 %4==0
  - Base32: [A-Z2-7=] 大写为主
  - Hex: [0-9A-Fa-f] 且长度为偶数
  - Binary: 只有 0 和 1
  - Morse: 只有 . - 和空格
  - 如果解码结果仍像编码，继续递归
```

## 分析流程

### Step 1: 自动识别
```bash
# 使用 CyberChef 自动识别（推荐）
# https://gchq.github.io/CyberChef/

# 或使用脚本递归解码
python3 scripts/decode_multilayer.py encoded.txt
```

### Step 2: Base 系列
```python
import base64

# Base64
base64.b64decode(data)

# Base32
base64.b32decode(data)

# Base58 (需要 base58 库)
import base58
base58.b58decode(data)

# Base85
base64.b85decode(data)
```

### Step 3: 进制转换
```python
# Hex to bytes
bytes.fromhex(hex_string)

# Binary to bytes
int(binary_string, 2).to_bytes(length, 'big')

# Octal to bytes
int(octal_string, 8).to_bytes(length, 'big')
```

### Step 4: ROT / Caesar
```python
import codecs

# ROT13
codecs.decode(text, 'rot_13')

# Caesar 全枚举
for shift in range(26):
    result = ''.join(chr((ord(c) - 65 + shift) % 26 + 65) if c.isupper() 
                     else chr((ord(c) - 97 + shift) % 26 + 97) if c.islower() 
                     else c for c in text)
    print(f"Shift {shift}: {result}")
```

### Step 5: 古典密码
```bash
# 使用在线工具
# https://www.dcode.fr/

# 摩尔斯电码
# . = dit, - = dah
# 空格分隔字母，/ 分隔单词

# 培根密码
# A/B 两种字符，每 5 个一组
```

## 常见出题套路

1. **多层 Base64** → 递归解码直到出现可读文本
2. **Base64 + Hex** → 先 Base64 再 Hex
3. **ROT13 变体** → 尝试所有 shift
4. **URL 编码** → `urllib.parse.unquote()`
5. **HTML 实体** → `html.unescape()`
6. **混合编码** → CyberChef Magic 自动识别

## 在线工具推荐

- **CyberChef** - https://gchq.github.io/CyberChef/
  - Magic 功能可自动识别编码
  - 支持链式操作
  
- **dcode.fr** - https://www.dcode.fr/
  - 古典密码专家
  - 支持频率分析

## 脚本参考

详见 `scripts/decode_multilayer.py`

## 快速命令

```bash
# Base64 解码
echo "SGVsbG8=" | base64 -d

# Hex 解码
echo "48656c6c6f" | xxd -r -p

# URL 解码
python3 -c "import urllib.parse; print(urllib.parse.unquote('%48%65%6c%6c%6f'))"

# ROT13
echo "Uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
