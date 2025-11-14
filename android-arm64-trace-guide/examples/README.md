# QBDI Android ARM64 示例代码

本目录包含各种使用 QBDI 在 Android ARM64 设备上进行 trace 的示例代码。

## 📁 目录结构（规划）

```
examples/
├── frida/                      # Frida + QBDI 示例
│   ├── basic_trace.js         # 基础 trace
│   ├── trace_libc.js          # trace libc 函数
│   ├── trace_crypto.js        # trace 加密函数
│   ├── memory_access.js       # 内存访问监控
│   ├── function_calls.js      # 函数调用追踪
│   └── code_coverage.js       # 代码覆盖率
│
├── pyqbdi/                     # PyQBDI 示例
│   ├── basic_trace.py         # 基础 trace
│   ├── trace_native.py        # trace native 函数
│   └── memory_monitor.py      # 内存监控
│
└── native/                     # Native C/C++ 示例
    ├── trace_demo.c           # C 语言示例
    ├── trace_demo.cpp         # C++ 示例
    └── CMakeLists.txt         # 编译配置
```

## 🚀 快速开始

所有示例的详细说明和使用方法，请参考主文档：

- [快速开始-使用预编译包.md](../快速开始-使用预编译包.md)

## 📝 示例说明

### Frida 示例

这些示例展示了如何使用 Frida + QBDI 进行动态 trace。

**特点:**
- 无需修改应用
- 动态注入
- JavaScript 编写
- 快速迭代

### PyQBDI 示例

这些示例展示了如何使用 Python API 进行 trace。

**特点:**
- Python 语法
- 易于自动化
- 便于数据分析

### Native 示例

这些示例展示了如何使用 C/C++ API 进行 trace。

**特点:**
- 高性能
- 底层控制
- 可集成到现有项目

## 💡 使用建议

1. **新手**: 从 Frida 的 `basic_trace.js` 开始
2. **Python 用户**: 从 PyQBDI 的 `basic_trace.py` 开始
3. **C/C++ 开发者**: 从 Native 的 `trace_demo.c` 开始

## 📚 相关文档

- [INDEX.md](../INDEX.md) - 文档导航
- [快速开始-使用预编译包.md](../快速开始-使用预编译包.md) - 快速开始指南
- [README.md](../README.md) - 完整使用指南
- [文件说明.md](../文件说明.md) - 文件详细说明

## 🔗 外部资源

- **QBDI 官方示例**: https://github.com/QBDI/QBDI/tree/master/examples
- **QBDI 文档**: https://qbdi.readthedocs.io/
- **Frida 示例**: https://frida.re/docs/examples/

---

**注意**: 本目录目前为规划状态。完整的示例代码已包含在各个主文档中。
你可以从主文档中复制代码并在此处创建文件，或参考 QBDI 官方仓库的 examples 目录。

