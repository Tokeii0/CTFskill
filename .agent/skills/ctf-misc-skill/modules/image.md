# 🖼️ 图片隐写分析模块

## 适用文件类型
- PNG / JPG / BMP / GIF / WEBP

## 检查清单

```yaml
检查项:
  - [ ] 文件尾部是否追加数据（binwalk / foremost）
  - [ ] LSB 隐写（zsteg / stegsolve / PIL 手动提取）
  - [ ] EXIF 注释/GPS/作者字段
  - [ ] PNG IDAT chunk 异常 / CRC 校验错误
  - [ ] 图片尺寸与实际像素不符（修改高度恢复）
  - [ ] 双图片拼接 / APNG 多帧
  - [ ] 特定通道/bit plane 隐藏信息
  - [ ] 二维码/条形码识别
  - [ ] 图片异或/拼图还原
  - [ ] 色板（palette）隐写

常用工具:
  - zsteg, stegsolve, pngcheck
  - gimp, PIL/Pillow
  - pyzbar (二维码)
  - exiftool
```

## 分析流程

### Step 1: 快速扫描
```bash
# 文件类型
file image.png

# 查看文件头
xxd image.png | head -20

# EXIF 信息
exiftool image.png

# 嵌入文件检测
binwalk image.png
foremost image.png

# 字符串搜索
strings image.png | grep -iE "flag|ctf|key"
```

### Step 2: PNG 专项检查
```bash
# LSB 全面扫描
zsteg -a image.png

# PNG 结构检查
pngcheck -v image.png

# CRC 修复（如果报错）
python3 scripts/png_height_fix.py image.png
```

### Step 3: JPG 专项检查
```bash
# Steghide 检测
steghide info image.jpg

# 密码爆破
stegseek image.jpg wordlist.txt

# JPEG 注释
exiftool -Comment image.jpg
```

### Step 4: 高级分析
```python
# 使用 PIL 手动提取 LSB
from PIL import Image
import numpy as np

img = Image.open('image.png')
pixels = np.array(img)

# 提取 R 通道 LSB
lsb = pixels[:,:,0] & 1
# 可视化
Image.fromarray(lsb * 255).show()
```

## 常见出题套路

1. **PNG 高度被修改** → 使用 `scripts/png_height_fix.py` 爆破
2. **LSB 隐写** → zsteg 一键扫描
3. **EXIF 隐藏** → exiftool 查看所有字段
4. **文件拼接** → binwalk 提取，foremost 分离
5. **通道隐写** → stegsolve 查看各个 bit plane
6. **二维码** → 截图后用 pyzbar 识别

## 脚本参考

详见 `scripts/png_height_fix.py`
