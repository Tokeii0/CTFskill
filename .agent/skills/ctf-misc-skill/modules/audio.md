# 🎵 音频隐写分析模块

## 适用文件类型
- WAV / MP3 / FLAC / OGG

## 检查清单

```yaml
检查项:
  - [ ] 频谱图（Spectrogram）隐藏图像/文字
  - [ ] 音频 LSB 隐写
  - [ ] SSTV（慢扫描电视）
  - [ ] 摩尔斯电码 / DTMF 拨号音
  - [ ] 隐藏的额外音轨
  - [ ] 元数据/ID3 标签
  - [ ] 音频拼接 / 逆向播放
  - [ ] 波形图异常
  - [ ] 音频隐写工具（DeepSound / SilentEye）

常用工具:
  - Audacity (频谱图可视化)
  - Sonic Visualiser
  - DeepSound
  - RX-SSTV (SSTV 解码)
  - multimon-ng (摩尔斯/DTMF)
  - ffmpeg, sox
  - scipy.io.wavfile (Python)
```

## 分析流程

### Step 1: 基础信息
```bash
# 文件信息
file audio.wav
mediainfo audio.wav

# 元数据
exiftool audio.wav

# ID3 标签（MP3）
ffprobe audio.mp3
```

### Step 2: 频谱图分析（最常见）
```bash
# 使用脚本生成频谱图
bash scripts/spectrogram.sh audio.wav

# 或使用 Audacity
# 1. 打开音频文件
# 2. 选择 Spectrogram 视图
# 3. 调整参数查看隐藏内容
```

### Step 3: LSB 隐写检测
```python
import wave
import numpy as np

with wave.open('audio.wav', 'rb') as wav:
    frames = wav.readframes(wav.getnframes())
    audio_data = np.frombuffer(frames, dtype=np.int16)
    
    # 提取 LSB
    lsb = audio_data & 1
    # 转换为字节
    bytes_data = np.packbits(lsb)
    print(bytes_data.tobytes())
```

### Step 4: SSTV 解码
```bash
# 使用 RX-SSTV 或 qsstv
# 播放音频，软件会自动解码图像
```

### Step 5: 摩尔斯电码
```bash
# 使用 multimon-ng
multimon-ng -t wav -a MORSE audio.wav
```

## 常见出题套路

1. **频谱图隐藏** → Audacity 查看频谱
2. **SSTV 图像** → RX-SSTV 解码
3. **摩尔斯电码** → 听音频或用 multimon-ng
4. **逆向播放** → Audacity 反转效果
5. **双声道隐写** → 分离左右声道分析
6. **DeepSound 加密** → 需要密码提取

## 脚本参考

详见 `scripts/spectrogram.sh`
