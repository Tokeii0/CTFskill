# 🧠 内存取证分析模块

## 适用文件类型
- RAW / VMEM / DMP / LIME / VMSS

## 检查清单

```yaml
检查项:
  - [ ] 内存镜像格式识别（raw / vmem / dmp / lime）
  - [ ] 操作系统识别（Windows / Linux / macOS）
  - [ ] 进程列表分析（可疑进程 / 隐藏进程）
  - [ ] 网络连接提取（IP / 端口 / 域名）
  - [ ] 命令行历史（cmdline / bash_history）
  - [ ] 文件提取（filescan / dumpfiles）
  - [ ] 注册表分析（hivelist / printkey）
  - [ ] 密码哈希提取（hashdump / lsadump）
  - [ ] 剪贴板内容（clipboard）
  - [ ] 屏幕截图恢复（screenshot）
  - [ ] 恶意代码注入检测（malfind / hollowfind）
  - [ ] 浏览器历史/缓存（iehistory / chromehistory）
  - [ ] 环境变量 / 用户信息

常用工具:
  - Volatility 2 / Volatility 3
  - MemProcFS
  - Rekall
  - strings + grep（快速搜索）
  - bulk_extractor（批量提取）
```

## 分析流程

### Step 1: 快速搜索（优先）
```bash
# 直接搜索 flag（最快）
strings -e l memory.raw | grep -iE "flag|ctf"
strings -e b memory.raw | grep -iE "flag|ctf"

# 使用脚本搜索
python3 scripts/memory_flag_search.py memory.raw
```

### Step 2: 系统识别
```bash
# Volatility 3 识别 OS
vol -f memory.raw windows.info
vol -f memory.raw linux.banner.Banners

# 或使用自动化脚本
python3 scripts/volatility_auto.py memory.raw
```

### Step 3: 进程分析
```bash
# 进程列表
vol -f memory.raw windows.pslist
vol -f memory.raw windows.pstree

# 查看命令行
vol -f memory.raw windows.cmdline

# 网络连接
vol -f memory.raw windows.netscan
```

### Step 4: 敏感数据提取
```bash
# 剪贴板
vol -f memory.raw windows.clipboard

# 环境变量
vol -f memory.raw windows.envars | grep -iE "flag|password|key"

# 屏幕截图
vol -f memory.raw windows.screenshot --dump

# 密码哈希
vol -f memory.raw windows.hashdump
```

### Step 5: 文件提取
```bash
# 扫描文件
vol -f memory.raw windows.filescan > filescan.txt

# 搜索可疑文件
grep -iE "flag|Desktop|Documents" filescan.txt

# 提取文件
vol -f memory.raw windows.dumpfiles --virtaddr 0x... --dump-dir ./output

# 或使用批量提取脚本
bash scripts/vol_extract.sh memory.raw
```

## Volatility 3 核心插件

### Windows
```bash
windows.info              # 系统信息
windows.pslist            # 进程列表
windows.pstree            # 进程树
windows.netscan           # 网络连接
windows.cmdline           # 命令行
windows.filescan          # 文件扫描
windows.dumpfiles         # 文件提取
windows.registry.hivelist # 注册表
windows.hashdump          # 密码哈希
windows.clipboard         # 剪贴板
windows.screenshot        # 屏幕截图
windows.envars            # 环境变量
```

### Linux
```bash
linux.banner.Banners      # 系统信息
linux.pslist.PsList       # 进程列表
linux.pstree.PsTree       # 进程树
linux.netstat.NetStat     # 网络连接
linux.bash.Bash           # Bash 历史
```

## 常见出题套路

1. **剪贴板隐藏** → `windows.clipboard`
2. **命令行历史** → `windows.cmdline` 查看执行过的命令
3. **桌面文件** → `filescan` + 提取 Desktop 路径文件
4. **环境变量** → `envars` 查找 FLAG 变量
5. **屏幕截图** → `screenshot` 恢复屏幕画面
6. **浏览器历史** → 提取浏览器进程内存

## 脚本参考

- `scripts/volatility_auto.py` - 自动化分析
- `scripts/memory_flag_search.py` - Flag 搜索
- `scripts/vol_extract.sh` - 批量文件提取
