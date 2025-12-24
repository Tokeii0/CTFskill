# 🎵 音频隐写分析模块

## 适用文件类型
- WAV / MP3 / FLAC / OGG / M4A / AAC

## 检查清单

```yaml
基础检查:
  - [ ] 文件类型识别（file / mediainfo）
  - [ ] 元数据/ID3 标签（exiftool / ffprobe）
  - [ ] 文件尾部追加数据（binwalk）
  - [ ] 字符串搜索（strings | grep）

频谱图分析:
  - [ ] 时域频谱图（Spectrogram）
  - [ ] 频域瀑布图（Waterfall）
  - [ ] 3D 频谱图
  - [ ] 特定频率范围分析

音频隐写:
  - [ ] LSB 隐写（音频样本 LSB）
  - [ ] 回声隐写（Echo hiding）
  - [ ] 相位编码（Phase coding）
  - [ ] 扩频隐写（Spread spectrum）

特殊编码:
  - [ ] SSTV（慢扫描电视）
  - [ ] 摩尔斯电码（Morse code）
  - [ ] DTMF 拨号音
  - [ ] AFSK（音频移频键控）

音频特性:
  - [ ] 双声道分离（左右声道）
  - [ ] 逆向播放（Reverse）
  - [ ] 变速播放（Speed change）
  - [ ] 音频拼接检测
  - [ ] 静音段分析
  - [ ] 波形异常检测

常用工具:
  - Audacity (频谱图可视化)
  - Sonic Visualiser (专业音频分析)
  - DeepSound (音频隐写工具)
  - RX-SSTV / qsstv (SSTV 解码)
  - multimon-ng (摩尔斯/DTMF)
  - ffmpeg, sox (音频处理)
  - scipy.io.wavfile (Python)
```

## 分析流程

### Step 1: 基础信息收集

```bash
# 文件信息
file audio.wav
mediainfo audio.wav

# 元数据（重点关注 Comment/Title/Artist）
exiftool audio.wav
exiftool -a -G1 audio.wav

# ID3 标签（MP3）
ffprobe audio.mp3 2>&1 | grep -iE "title|artist|comment|album"
id3v2 -l audio.mp3

# 嵌入文件检测
binwalk audio.wav
foremost audio.wav

# 字符串搜索
strings audio.wav | grep -iE "flag|ctf|key"
```

### Step 2: 频谱图分析（最常见）

```bash
# 方法 1: 使用脚本生成频谱图
bash scripts/spectrogram.sh audio.wav

# 方法 2: 使用 sox
sox audio.wav -n spectrogram -o spectrogram.png -x 2000 -y 500 -z 80

# 方法 3: 使用 ffmpeg
ffmpeg -i audio.wav -lavfi showspectrumpic=s=1920x1080:mode=separate spectrogram.png

# 方法 4: 使用 Audacity（GUI）
# 1. 打开音频文件
# 2. 选择 Spectrogram 视图
# 3. 调整参数: 频率范围、窗口大小
# 4. 查看是否有隐藏的图像/文字
```

### Step 3: LSB 隐写检测

```python
#!/usr/bin/env python3
"""WAV 音频 LSB 提取"""
import wave
import numpy as np

def extract_lsb(wav_file):
    with wave.open(wav_file, 'rb') as wav:
        frames = wav.readframes(wav.getnframes())
        audio_data = np.frombuffer(frames, dtype=np.int16)
        
        # 提取 LSB
        lsb = audio_data & 1
        
        # 转换为字节
        # 每 8 个 bit 组成一个字节
        bytes_list = []
        for i in range(0, len(lsb), 8):
            if i + 8 <= len(lsb):
                byte = 0
                for j in range(8):
                    byte = (byte << 1) | lsb[i + j]
                bytes_list.append(byte)
        
        data = bytes(bytes_list)
        
        # 尝试解码
        try:
            text = data.decode('utf-8', errors='ignore')
            print(text)
        except:
            # 保存为文件
            with open('extracted.bin', 'wb') as f:
                f.write(data)
            print("Saved to extracted.bin")

if __name__ == '__main__':
    import sys
    extract_lsb(sys.argv[1])
```

### Step 4: SSTV 解码

```bash
# SSTV（慢扫描电视）- 音频中编码的图像

# 方法 1: 使用 RX-SSTV（Windows）
# 播放音频，软件会自动解码

# 方法 2: 使用 qsstv（Linux）
qsstv
# 然后播放音频文件

# 方法 3: 使用 Python（sstv 库）
pip3 install sstv
python3 << EOF
from sstv import SSTV
from PIL import Image
sstv = SSTV.from_file('audio.wav')
image = sstv.decode()
image.save('sstv_decoded.png')
EOF
```

### Step 5: 摩尔斯电码解码

```bash
# 方法 1: 使用 multimon-ng
multimon-ng -t wav -a MORSE audio.wav

# 方法 2: 手动听音频
# 短音 = .（dit）
# 长音 = -（dah）
# 然后在线解码: https://morsecode.world/international/decoder/audio-decoder-adaptive.html

# 方法 3: 使用 Python
pip3 install morse-audio-decoder
python3 << EOF
from morse_audio_decoder import MorseDecoder
decoder = MorseDecoder('audio.wav')
print(decoder.decode())
EOF
```

### Step 6: DTMF 拨号音解码

```bash
# DTMF（双音多频）- 电话拨号音

# 使用 multimon-ng
multimon-ng -t wav -a DTMF audio.wav

# 使用 Python
pip3 install dtmf-decoder
python3 << EOF
from dtmf_decoder import DTMFDecoder
decoder = DTMFDecoder('audio.wav')
print(decoder.decode())
EOF
```

### Step 7: 双声道分析

```bash
# 分离左右声道
ffmpeg -i audio.wav -map_channel 0.0.0 left.wav
ffmpeg -i audio.wav -map_channel 0.0.1 right.wav

# 或使用 sox
sox audio.wav left.wav remix 1
sox audio.wav right.wav remix 2

# 分别分析两个声道
```

### Step 8: 高级分析

```python
# 1. 逆向播放
from pydub import AudioSegment
audio = AudioSegment.from_wav('audio.wav')
reversed_audio = audio.reverse()
reversed_audio.export('reversed.wav', format='wav')

# 2. 变速播放（不改变音调）
from pydub import AudioSegment
from pydub.playback import play
audio = AudioSegment.from_wav('audio.wav')
# 加速 1.5 倍
faster = audio.speedup(playback_speed=1.5)
faster.export('faster.wav', format='wav')

# 3. 波形异常检测
import wave
import numpy as np
import matplotlib.pyplot as plt

with wave.open('audio.wav', 'rb') as wav:
    frames = wav.readframes(wav.getnframes())
    audio_data = np.frombuffer(frames, dtype=np.int16)
    
    # 绘制波形
    plt.figure(figsize=(15, 5))
    plt.plot(audio_data)
    plt.savefig('waveform.png')
    
    # 查找异常值
    mean = np.mean(audio_data)
    std = np.std(audio_data)
    anomalies = np.where(np.abs(audio_data - mean) > 3 * std)[0]
    print(f"Found {len(anomalies)} anomalies")

# 4. 静音段分析
from pydub import AudioSegment
from pydub.silence import detect_silence

audio = AudioSegment.from_wav('audio.wav')
silences = detect_silence(audio, min_silence_len=500, silence_thresh=-40)
print(f"Silent segments: {silences}")
```

## 常见出题套路与解法

### 套路 1: 频谱图隐藏文字/图像

**特征**: 音频听起来正常，但频谱图中有内容

**解法**:
```bash
# 生成频谱图
sox audio.wav -n spectrogram -o spec.png

# 在 Audacity 中查看
# 调整频率范围（通常在 0-10kHz）
# 调整窗口大小（Window size）
```

**常见变体**:
- 文字在高频段（需要调整频率范围）
- 图像需要旋转 90 度
- 需要调整对比度才能看清

### 套路 2: SSTV 图像传输

**特征**: 音频中有类似调制解调器的声音

**解法**:
```bash
# 使用 qsstv 或 RX-SSTV 解码
# 常见模式: Martin M1, Scottie S1, Robot 36
```

### 套路 3: 摩尔斯电码

**特征**: 音频中有规律的短音和长音

**解法**:
```bash
# 自动解码
multimon-ng -t wav -a MORSE audio.wav

# 手动解码
# . = E, - = T, .. = I, etc.
```

### 套路 4: LSB 隐写

**特征**: 音频样本的最低位被修改

**解法**:
```python
# 使用上面的 LSB 提取脚本
python3 extract_lsb.py audio.wav
```

### 套路 5: 逆向播放

**特征**: 音频倒放后有内容

**解法**:
```python
from pydub import AudioSegment
audio = AudioSegment.from_wav('audio.wav')
reversed_audio = audio.reverse()
reversed_audio.export('reversed.wav', format='wav')
```

### 套路 6: 双声道隐写

**特征**: 左右声道不同，或需要异或

**解法**:
```bash
# 分离声道
ffmpeg -i audio.wav -map_channel 0.0.0 left.wav
ffmpeg -i audio.wav -map_channel 0.0.1 right.wav

# 异或两个声道
python3 << EOF
import wave
import numpy as np

with wave.open('left.wav', 'rb') as l, wave.open('right.wav', 'rb') as r:
    left = np.frombuffer(l.readframes(l.getnframes()), dtype=np.int16)
    right = np.frombuffer(r.readframes(r.getnframes()), dtype=np.int16)
    xor = left ^ right
    
    # 保存结果
    with wave.open('xor.wav', 'wb') as out:
        out.setparams(l.getparams())
        out.writeframes(xor.tobytes())
EOF
```

### 套路 7: DeepSound 加密

**特征**: 使用 DeepSound 工具隐藏文件

**解法**:
```
1. 下载 DeepSound
2. 打开音频文件
3. 输入密码（可能在题目中）
4. 提取隐藏文件
```

### 套路 8: 音频拼接

**特征**: 多段音频需要拼接

**解法**:
```python
from pydub import AudioSegment

# 拼接音频
audio1 = AudioSegment.from_wav('part1.wav')
audio2 = AudioSegment.from_wav('part2.wav')
combined = audio1 + audio2
combined.export('combined.wav', format='wav')
```

### 套路 9: 变速隐藏

**特征**: 音频需要加速/减速才能听清

**解法**:
```python
from pydub import AudioSegment

audio = AudioSegment.from_wav('audio.wav')

# 尝试不同速度
for speed in [0.5, 0.75, 1.5, 2.0]:
    modified = audio.speedup(playback_speed=speed)
    modified.export(f'speed_{speed}.wav', format='wav')
```

### 套路 10: 元数据隐藏

**特征**: ID3 标签或注释中藏有 flag

**解法**:
```bash
exiftool -a -G1 audio.mp3
id3v2 -l audio.mp3
ffprobe audio.mp3 2>&1 | grep -i comment
```

## 实战技巧

### 技巧 1: 快速判断音频类型

```bash
#!/bin/bash
FILE=$1
echo "[*] Analyzing: $FILE"

# 基础信息
file $FILE
mediainfo $FILE | grep -iE "duration|sample rate|channels"

# 元数据
exiftool $FILE | grep -iE "comment|title|artist"

# 生成频谱图
sox $FILE -n spectrogram -o quick_spec.png -x 1000 -y 300
echo "[+] Spectrogram saved to quick_spec.png"

# 检查嵌入文件
binwalk $FILE | grep -v "^DECIMAL"
```

### 技巧 2: 批量频谱图生成

```bash
#!/bin/bash
for file in *.wav; do
    sox "$file" -n spectrogram -o "${file%.wav}_spec.png"
done
```

### 技巧 3: 音频格式转换

```bash
# WAV to MP3
ffmpeg -i audio.wav -codec:a libmp3lame audio.mp3

# MP3 to WAV
ffmpeg -i audio.mp3 audio.wav

# 任意格式转 WAV（推荐用于分析）
ffmpeg -i audio.* -ar 44100 -ac 2 output.wav
```

### 技巧 4: 频率范围扫描

```python
# 扫描特定频率范围
import numpy as np
from scipy.io import wavfile
from scipy import signal
import matplotlib.pyplot as plt

rate, data = wavfile.read('audio.wav')

# 短时傅里叶变换
f, t, Sxx = signal.spectrogram(data, rate)

# 查找特定频率范围的能量
freq_range = (1000, 5000)  # 1-5 kHz
freq_indices = np.where((f >= freq_range[0]) & (f <= freq_range[1]))[0]
energy = np.sum(Sxx[freq_indices, :], axis=0)

plt.plot(t, energy)
plt.savefig('freq_energy.png')
```

## 工具速查

```bash
# 基础工具
file audio.wav                  # 文件类型
mediainfo audio.wav             # 详细信息
exiftool audio.wav              # 元数据
binwalk audio.wav               # 嵌入文件

# 频谱图
sox audio.wav -n spectrogram -o spec.png
ffmpeg -i audio.wav -lavfi showspectrumpic=s=1920x1080 spec.png
# Audacity GUI

# 特殊编码
multimon-ng -t wav -a MORSE audio.wav    # 摩尔斯
multimon-ng -t wav -a DTMF audio.wav     # DTMF
qsstv                                     # SSTV

# 音频处理
ffmpeg -i audio.wav -map_channel 0.0.0 left.wav  # 分离声道
sox audio.wav reversed.wav reverse                # 逆向
```

## Python 库推荐

```bash
pip3 install pydub scipy numpy matplotlib wave
pip3 install sstv morse-audio-decoder dtmf-decoder
```

## 脚本参考

详见 `scripts/spectrogram.sh`
