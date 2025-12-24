# CTF Misc Solver Skill

> 专业的 CTF Misc 方向自动化解题 Skill，适用于 各类 AI 框架

## 🎯 这是什么？

一个**系统化的 CTF Misc 解题框架**，能够：

- ✅ 自动识别题目类型（图片/音频/压缩包/流量/内存/编码）
- ✅ 提供标准化分析流程
- ✅ 生成可直接运行的自动化脚本
- ✅ 给出多条备选解题路径

## 📁 目录结构

```
.agent/skills/ctf-misc-skill/
├── SKILL.md              # 🧠 主控文件（调度大脑）
├── README.md             # 📖 本文件
├── modules/              # 📚 分析模块（扩展参考）
│   ├── image.md         # 图片隐写
│   ├── audio.md         # 音频隐写
│   ├── archive.md       # 压缩包分析
│   ├── network.md       # 流量分析
│   ├── memory.md        # 内存取证
│   └── encoding.md      # 编码加密
├── scripts/              # 💻 脚本模板库
│   ├── decode_multilayer.py
│   ├── png_height_fix.py
│   ├── zip_fake_encrypt.py
│   ├── spectrogram.sh
│   ├── usb_keyboard.py
│   ├── volatility_auto.py
│   ├── memory_flag_search.py
│   └── vol_extract.sh
└── docs/                 # 📄 文档
    ├── QUICKREF.md      # 快速参考
    ├── CHANGELOG.md     # 更新日志
    └── TOOLS.md         # 工具安装指南
```

## 🚀 快速开始

### 1. 安装 Skill

将整个 `.agent` 目录放置在你的项目根目录下，AI 会自动识别。

### 2. 触发 Skill

只需说：

```
"帮我分析这个 png，找一下 flag"
"这是一道 CTF Misc 题"
"这个内存镜像怎么分析？"
```

### 3. Skill 会自动

1. 识别文件类型
2. 选择对应分析流程
3. 生成自动化脚本
4. 提供多条解题路径

## 🎯 支持的题型

| 类型 | 覆盖范围 | 核心工具 |
|------|---------|---------|
| 🖼️ **图片** | LSB/EXIF/像素/尺寸 | zsteg, stegsolve, PIL |
| 🎵 **音频** | 频谱/LSB/SSTV/摩尔斯 | Audacity, sox, ffmpeg |
| 📦 **压缩包** | 伪加密/明文攻击/CRC | 7z, fcrackzip, bkcrack |
| 📡 **流量** | HTTP还原/USB解析 | Wireshark, tshark |
| 🧠 **内存** | 进程/文件/哈希 | Volatility 3 |
| 🔠 **编码** | 多层Base/ROT/古典 | CyberChef, Python |

## 📋 核心特性

### 1. 智能调度系统

```yaml
SKILL.md (主控大脑)
  ↓
识别文件类型
  ↓
调用对应 module
  ↓
生成自动化脚本
  ↓
输出解题方案
```

### 2. 模块化设计

- **modules/** - 详细的工具使用方法和检查清单
- **scripts/** - 8 个常用自动化脚本模板
- **docs/** - 快速参考和工具安装指南

### 3. 脚本使用约定

```yaml
Scripts 定位:
  - 参考模板，可根据题目改造
  - 优先生成一键可运行版本
  - 包含完整的错误处理

可用脚本:
  - decode_multilayer.py    # 多层编码爆破
  - png_height_fix.py       # PNG 高度修复
  - zip_fake_encrypt.py     # ZIP 伪加密修复
  - volatility_auto.py      # Volatility 快速分析
  - memory_flag_search.py   # 内存 Flag 搜索
  - ... 等 8 个脚本
```

## 🛠️ 环境配置

### 必装工具

```bash
# 文件分析
apt install binwalk foremost exiftool file

# 图片隐写
gem install zsteg
apt install steghide

# 流量分析
apt install wireshark tshark

# 内存取证
pip3 install volatility3

# Python 库
pip3 install pillow pycryptodome
```

详细安装指南: `docs/TOOLS.md`

## 📖 使用文档

- **SKILL.md** - 主控文件，包含完整的解题流程和核心技术
- **docs/QUICKREF.md** - 快速参考卡片
- **docs/TOOLS.md** - 工具安装指南
- **modules/*.md** - 各类型题目的详细分析方法

## 🎓 设计理念

### SKILL.md = 调度大脑

- ✅ 包含完整的触发条件和场景识别
- ✅ 定义文件类型 → modules 的调度规则
- ✅ 提供核心技术要点（不只是索引）
- ✅ 明确 scripts 使用约定和边界

### modules = 扩展参考

- 详细的工具使用方法
- 完整的检查清单
- 具体的命令示例

### scripts = 模板库

- 可直接运行的参考脚本
- 允许根据题目改造
- 包含使用说明和边界约定

## 📊 版本信息

- **当前版本**: v1.1
- **最后更新**: 2025-12-24
- **包含模块**: 6 个分析模块 + 8 个脚本模板

## 📝 License

MIT License - 仅用于 CTF 比赛和安全学习研究

---

**开始使用**: 直接在 Claude 中说 "这是一道 CTF Misc 题" 即可触发 Skill！
