# CTF Misc Skill - 内存取证模块更新

## ✅ 更新内容

### 1. YAML Frontmatter 更新
- 在 `description` 中添加了内存镜像（raw/vmem/dmp）相关触发场景
- 添加了"内存取证"、"Volatility"等关键词

### 2. Phase 2 分类处理 - 新增内存取证类别

新增 **🧠 内存取证类（Memory Dump / RAW / VMEM）** 模块，包含：

#### 检查项（13 项）
- 内存镜像格式识别（raw / vmem / dmp / lime）
- 操作系统识别（Windows / Linux / macOS）
- 进程列表分析（可疑进程 / 隐藏进程）
- 网络连接提取（IP / 端口 / 域名）
- 命令行历史（cmdline / bash_history）
- 文件提取（filescan / dumpfiles）
- 注册表分析（hivelist / printkey）
- 密码哈希提取（hashdump / lsadump）
- 剪贴板内容（clipboard）
- 屏幕截图恢复（screenshot）
- 恶意代码注入检测（malfind / hollowfind）
- 浏览器历史/缓存（iehistory / chromehistory）
- 环境变量 / 用户信息

#### 常用工具
- Volatility 2 / Volatility 3
- MemProcFS
- Rekall
- strings + grep（快速搜索）
- bulk_extractor（批量提取）

#### Volatility 3 常用插件列表
提供了 11 个最常用的 Volatility 3 插件及其用途说明

### 3. Common Techniques - 新增 3 个内存取证脚本

#### 脚本 6: 内存取证快速分析（Volatility 3）
- **功能**: 自动化执行 7 个关键分析步骤
- **特点**: 
  - 自动识别操作系统（Windows/Linux）
  - 依次执行：系统信息、进程列表、网络连接、命令行、文件扫描、剪贴板、环境变量
  - 每个步骤都有清晰的输出标记
  - 提供下一步建议

#### 脚本 7: 内存镜像中搜索 Flag
- **功能**: 使用多种正则模式在内存中搜索 flag
- **特点**:
  - 6 种 flag 模式匹配（包括 MD5 格式）
  - 使用 mmap 提高大文件读取效率
  - 自动去重和验证可打印性
  - 失败时提供备选方案

#### 脚本 8: Volatility 文件提取自动化（Bash）
- **功能**: 批量提取内存中的可疑文件
- **特点**:
  - 自动提取桌面文件
  - 自动提取 flag 相关文件
  - 自动提取常见文档类型
  - 提取后自动搜索 flag

### 4. Trigger Examples 更新
新增 5 个内存取证相关触发示例：
```
"帮我分析这个内存镜像"
"这是一个 memory dump，怎么找 flag？"
"Volatility 应该用哪些插件？"
"内存取证题，给了一个 .raw 文件"
"vmem 文件怎么分析？"
```

### 5. Required Tools Reference 更新
在推荐工具部分添加：
- Volatility 2 / Volatility 3 (内存取证)
- MemProcFS (内存取证)
- bulk_extractor (批量提取)

### 6. README.md 同步更新
- 适用场景中添加"内存取证"
- 触发示例中添加内存取证示例
- 推荐工具链中添加 Volatility 和 MemProcFS

## 📊 统计信息

- **新增检查项**: 13 项
- **新增工具**: 5 个
- **新增脚本**: 3 个（Python x2, Bash x1）
- **新增触发示例**: 5 个
- **代码行数**: 约 200+ 行

## 🎯 覆盖的内存取证场景

1. **Windows 内存取证**
   - 进程分析、网络连接、注册表、密码哈希
   - 剪贴板、屏幕截图、文件提取

2. **Linux 内存取证**
   - 进程树、Bash 历史、网络状态
   - 环境变量、用户信息

3. **通用分析**
   - 字符串搜索、文件签名识别
   - 批量数据提取

## 🔧 使用示例

当用户提供内存镜像文件时，Skill 会：

1. 自动识别操作系统类型
2. 执行标准化分析流程
3. 提供可直接运行的 Volatility 命令
4. 生成自动化分析脚本
5. 在内存中搜索 flag 模式
6. 提取可疑文件并分析

## ✨ 特色功能

- **自动化程度高**: 一键执行多个 Volatility 插件
- **容错性强**: 自动适配 Windows/Linux
- **效率优化**: 使用 mmap 处理大文件
- **结果清晰**: 每步都有明确的输出标记
- **可扩展**: 易于添加新的分析插件

---

**更新完成时间**: 2025-12-24
**版本**: v1.1 (添加内存取证模块)
