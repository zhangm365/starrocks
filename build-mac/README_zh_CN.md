# StarRocks macOS 构建系统

本目录包含在 macOS（特别是 Apple Silicon/ARM64）上构建 StarRocks 的脚本和配置。

## 快速开始

### 构建第三方依赖

```bash
cd build-mac
./build_thirdparty.sh
```

### 构建选项

```bash
# 显示帮助
./build_thirdparty.sh --help

# 清理后重新构建（删除所有之前的构建）
./build_thirdparty.sh --clean

# 使用指定数量的并行任务构建
./build_thirdparty.sh --parallel 8

# 仅安装 Homebrew 依赖
./build_thirdparty.sh --homebrew-only

# 仅构建源码依赖（跳过 Homebrew）
./build_thirdparty.sh --source-only
```

## 清理构建产物

### 完全清理（所有第三方库）

要清理**所有**第三方构建产物并重新开始：

```bash
./build_thirdparty.sh --clean
```

这将删除：
- `../thirdparty/build/` - 所有构建目录
- `../thirdparty/installed/` - 所有已安装的库

### 部分清理（特定库）

如果只需要重新构建特定的库（例如 brpc），可以手动删除其构建目录：

#### 清理 brpc

```bash
# 删除 brpc 构建目录
rm -rf ../thirdparty/build/brpc

# 删除 brpc 已安装的文件（可选，用于完全清理）
rm -f ../thirdparty/installed/lib/libbrpc.a
rm -f ../thirdparty/installed/lib64/libbrpc.a
rm -rf ../thirdparty/installed/include/brpc
```

清理后重新构建：

```bash
./build_thirdparty.sh
```

#### 清理其他库

同样的模式适用于其他库。构建目录位于：
- `../thirdparty/build/<库名>/`

例如：
- `../thirdparty/build/protobuf/`
- `../thirdparty/build/glog/`
- `../thirdparty/build/rocksdb/`

## macOS 上的常见构建问题

### 1. 无效的 Mach-O 文件错误

如果看到类似以下的错误：
```
file is not of required architecture
ld: warning: ignoring file libXXX.a, file was built for unsupported file format
```

**解决方案**：清理并重新构建有问题的库
```bash
# 以 brpc 为例
rm -rf ../thirdparty/build/brpc
./build_thirdparty.sh
```

### 2. 多个库的链接器错误

如果多个库显示 Mach-O 错误，通常最好进行完全清理：

```bash
./build_thirdparty.sh --clean
```

### 3. 版本冲突

如果通过 Homebrew 安装的库与构建冲突：

```bash
# 仅从源码重新构建，跳过 Homebrew
./build_thirdparty.sh --clean --source-only
```

## 目录结构

```
build-mac/
├── build_thirdparty.sh    # 第三方库的主构建脚本
├── build_be.sh            # StarRocks Backend 构建脚本
├── env_macos.sh           # macOS 环境配置
├── CMakeLists.txt         # macOS 的 CMake 配置
├── patch/                 # 第三方库的补丁
└── shims/                 # 兼容性 shims

../thirdparty/
├── build/                 # 构建目录（自动生成）
│   ├── brpc/             # brpc 构建产物
│   ├── protobuf/         # protobuf 构建产物
│   └── ...               # 其他库
├── installed/            # 已安装的库（自动生成）
│   ├── lib/              # 已安装的库
│   ├── include/          # 已安装的头文件
│   └── bin/              # 已安装的二进制文件
├── src/                  # 已下载的源码压缩包（自动生成）
└── patches/              # 第三方库的补丁
```

## 常见问题

### 问：清理 brpc 是指删除 thirdparty/build/brpc 目录吗？

**答：是的。** 要专门清理 brpc，请删除 `../thirdparty/build/brpc` 目录。构建脚本会在下次运行时自动重新构建它。

如需完全清理 brpc（包括已安装的文件）：
```bash
rm -rf ../thirdparty/build/brpc
rm -f ../thirdparty/installed/lib/libbrpc.a
rm -f ../thirdparty/installed/lib64/libbrpc.a
rm -rf ../thirdparty/installed/include/brpc
```

### 问：build/ 和 installed/ 目录有什么区别？

- `build/` - 包含构建产物、目标文件和编译期间创建的临时文件
- `installed/` - 包含 StarRocks 使用的最终编译库、头文件和二进制文件

### 问：我可以删除 src/ 目录吗？

`src/` 目录包含已下载的源码压缩包。您可以删除它，但脚本会在下次运行时重新下载。通常保留它是安全的。

### 问：如何强制重新构建特定的库？

1. 删除库的构建目录：`rm -rf ../thirdparty/build/<库名>`
2. 可选：删除已安装的文件：`rm -f ../thirdparty/installed/lib/lib<库名>.a`
3. 运行构建脚本：`./build_thirdparty.sh`

## 环境变量

- `STARROCKS_THIRDPARTY` - 覆盖第三方目录位置
- `PARALLEL_JOBS` - 通过 `--parallel N` 参数设置

## 另请参阅

- 主第三方构建脚本：`../thirdparty/build-thirdparty.sh`（用于 Linux）
- 主 README：`../README.md`
- 贡献指南：`../CONTRIBUTING.md`
