# QBDI Android ARM64 Trace 指南 - 目录索引

欢迎使用 QBDI Android ARM64 Trace 完整指南！

## 📚 文档导航

### 🚀 新手入门（推荐从这里开始）

1. **[快速开始-使用预编译包.md](./快速开始-使用预编译包.md)** ⭐
   - 无需编译，直接使用
   - 详细的下载和部署步骤
   - 包含完整的示例代码
   - **推荐所有用户先阅读此文档**

2. **[文件说明.md](./文件说明.md)**
   - 预编译包内容详解
   - 文件用途说明
   - 下载命令速查
   - 不同使用方式的文件需求

### 📖 完整文档（进阶用户）

3. **[README.md](./README.md)**
   - 完整的使用指南
   - 包含从源码编译的方法
   - 三种使用方式的详细说明
   - 高级技巧和进阶示例

### 📂 示例代码

4. **[examples/](./examples/)**
   - Frida 脚本示例
   - Python 脚本示例
   - C/C++ 代码示例

---

## 🎯 根据你的需求选择阅读路径

### 路径 1: 我只想快速开始使用

```
快速开始-使用预编译包.md → 开始 trace!
```

**适合人群:** 
- 第一次使用 QBDI
- 只想快速体验功能
- 不需要修改 QBDI 源码

**预计时间:** 30 分钟

---

### 路径 2: 我想深入了解

```
快速开始-使用预编译包.md → README.md → 文件说明.md
```

**适合人群:**
- 需要了解完整功能
- 想学习高级用法
- 可能需要自定义开发

**预计时间:** 2 小时

---

### 路径 3: 我需要从源码编译

```
README.md (编译部分) → 使用对应的方式章节
```

**适合人群:**
- 需要修改 QBDI 源码
- 需要特定的编译选项
- 开发 QBDI 相关工具

**预计时间:** 4 小时+

---

## 📋 快速参考

### 下载链接

- **QBDI Releases**: https://github.com/QBDI/QBDI/releases/tag/v0.12.0
- **主包**: QBDI-0.12.0-android-AARCH64.tar.gz (~20 MB)
- **PyQBDI**: pyqbdi-0.12.0-*.whl (~4.7 MB)

### 使用方式对比

| 方式 | 难度 | 灵活性 | 性能 | 推荐场景 |
|------|------|--------|------|----------|
| Frida + QBDI | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 动态分析、逆向工程 |
| PyQBDI | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ | 自动化分析、脚本化 |
| Native C/C++ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 深度集成、性能优先 |

### 环境要求速查

**Android 设备:**
- ✅ ARM64 (arm64-v8a) 架构
- ✅ Android 7.0+ (API 24+)
- ✅ Root 权限（推荐）
- ✅ USB 调试已启用

**开发机器:**
- ✅ Python 3.x
- ✅ ADB 工具
- ✅ (Frida) Node.js + npm
- ✅ (编译) Android NDK + CMake

---

## 🎓 学习路线建议

### 第一天：基础入门

1. 阅读 [快速开始-使用预编译包.md](./快速开始-使用预编译包.md)
2. 下载并部署 QBDI
3. 运行第一个 trace 示例（trace libc strlen）
4. 理解基本概念：VM、回调、指令分析

**学习目标:**
- [ ] 成功部署 QBDI 到 Android 设备
- [ ] 运行简单的 trace 脚本
- [ ] 看到指令级别的输出
- [ ] 理解回调函数的作用

### 第二天：实战应用

1. 选择一个真实的 Android 应用
2. 找到感兴趣的函数（使用 Frida 枚举导出）
3. 编写 trace 脚本
4. 分析输出结果

**学习目标:**
- [ ] 能够定位目标函数
- [ ] 编写自定义回调函数
- [ ] 过滤和格式化输出
- [ ] 理解内存访问 trace

### 第三天：高级技巧

1. 阅读 [README.md](./README.md) 的进阶技巧部分
2. 实现条件断点
3. 实现函数调用追踪
4. 实现代码覆盖率统计

**学习目标:**
- [ ] 使用条件过滤
- [ ] 修改寄存器值
- [ ] 构建调用栈
- [ ] 统计代码覆盖

---

## 💡 实用技巧

### 快速查找目标函数

```javascript
// 列出所有模块
Process.enumerateModules().forEach(function(m) {
    console.log(m.name);
});

// 列出模块的导出函数
var mod = Process.getModuleByName("libnative.so");
mod.enumerateExports().forEach(function(exp) {
    console.log(exp.name + " @ " + exp.address);
});
```

### 快速测试设备是否准备好

```bash
# 一键检查脚本
cat > check.sh << 'EOF'
#!/bin/bash
echo "[*] Checking device..."
adb devices -l

echo "[*] Checking architecture..."
adb shell getprop ro.product.cpu.abi

echo "[*] Checking SELinux..."
adb shell getenforce

echo "[*] Checking frida-server..."
adb shell ps | grep frida-server

echo "[*] Checking libQBDI.so..."
adb shell ls -l /data/local/tmp/libQBDI.so
EOF

chmod +x check.sh
./check.sh
```

### 快速部署脚本

```bash
# deploy.sh
#!/bin/bash
set -e

echo "[*] Deploying QBDI to device..."

# 推送文件
adb push libQBDI.so /data/local/tmp/
adb shell chmod 644 /data/local/tmp/libQBDI.so

# 检查 frida-server
if ! adb shell ps | grep -q frida-server; then
    echo "[*] Starting frida-server..."
    adb shell su -c "/data/local/tmp/frida-server &"
    sleep 2
fi

# 关闭 SELinux
adb shell su -c "setenforce 0" 2>/dev/null || true

echo "[*] Deployment complete!"
echo "[*] Device ready for tracing"
```

---

## 🔧 故障排查

### 常见问题速查

| 问题 | 解决方案 | 文档位置 |
|------|----------|----------|
| libQBDI.so 加载失败 | 检查路径和权限 | 快速开始-使用预编译包.md § 常见问题 |
| Frida 连接失败 | 检查版本匹配 | 快速开始-使用预编译包.md § 常见问题 |
| SELinux 阻止 | setenforce 0 | 快速开始-使用预编译包.md § 环境准备 |
| 找不到目标函数 | 使用枚举 API | README.md § 常见问题 |
| trace 输出为空 | 检查监控范围 | README.md § 常见问题 |
| 性能问题 | 使用条件过滤 | README.md § 进阶技巧 |

---

## 📞 获取帮助

### 官方资源

- **文档**: https://qbdi.readthedocs.io/
- **GitHub**: https://github.com/QBDI/QBDI
- **Issues**: https://github.com/QBDI/QBDI/issues

### 社区资源

- **Frida 文档**: https://frida.re/docs/
- **Android NDK**: https://developer.android.com/ndk/

### 提问建议

在提问时，请提供:
1. QBDI 版本
2. Android 版本和架构
3. 使用的方式（Frida/PyQBDI/Native）
4. 错误信息或日志
5. 最小可复现示例

---

## 🎉 开始你的 QBDI 之旅！

推荐从这里开始：

👉 **[快速开始-使用预编译包.md](./快速开始-使用预编译包.md)**

祝你使用愉快！Happy Tracing! 🚀

---

最后更新: 2025-11-14

