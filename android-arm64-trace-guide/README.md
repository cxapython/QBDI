# QBDI Android ARM64 Trace 使用指南

本指南详细介绍如何在 ARM64 架构的 Android 设备上使用 QBDI 进行指令级别的动态跟踪（trace）。

## ⚡ 快速开始

**🎉 不想编译？直接使用预编译包!**

👉 **[点击查看：快速开始指南（使用预编译包）](./快速开始-使用预编译包.md)**

推荐大多数用户使用预编译包，省时省力！本文档提供从源码编译的完整说明。

---

## 目录

1. [使用预编译包（推荐）](#使用预编译包推荐)
2. [环境准备](#环境准备)
3. [编译 QBDI for Android ARM64](#编译-qbdi-for-android-arm64)
4. [三种使用方式](#三种使用方式)
   - [方式一：Frida + QBDI (推荐)](#方式一frida--qbdi-推荐)
   - [方式二：PyQBDI](#方式二pyqbdi)
   - [方式三：Native C/C++](#方式三native-cc)
4. [示例代码](#示例代码)
5. [常见问题](#常见问题)

---

## 使用预编译包（推荐）

### 📦 下载地址

**最新版本**: QBDI 0.12.0 (2024-10-14)

**GitHub Releases**: https://github.com/QBDI/QBDI/releases/tag/v0.12.0

### 需要下载的文件

#### Android ARM64 主包

**QBDI-0.12.0-android-AARCH64.tar.gz**
- 大小: ~20 MB
- 包含: libQBDI.so、frida-qbdi.js、头文件、文档

**下载命令:**
```bash
wget https://github.com/QBDI/QBDI/releases/download/v0.12.0/QBDI-0.12.0-android-AARCH64.tar.gz
tar -xzf QBDI-0.12.0-android-AARCH64.tar.gz
cd QBDI-0.12.0-android-AARCH64
```

#### PyQBDI Wheel 包（可选）

如果使用 Python 方式，下载对应的 wheel:

- **Python 3.10**: `pyqbdi-0.12.0-cp310-cp310-linux_aarch64.whl`
- **Python 3.11**: `pyqbdi-0.12.0-cp311-cp311-linux_aarch64.whl`
- **Python 3.12**: `pyqbdi-0.12.0-cp312-cp312-linux_aarch64.whl`
- **Python 3.13**: `pyqbdi-0.12.0-cp313-cp313-linux_aarch64.whl`

**安装命令:**
```bash
pip3 install pyqbdi-0.12.0-cp311-cp311-linux_aarch64.whl
```

### 快速部署

```bash
# 1. 推送 QBDI 库到设备
adb push lib/libQBDI.so /data/local/tmp/
adb shell chmod 644 /data/local/tmp/libQBDI.so

# 2. 复制 Frida 绑定到工作目录
cp share/qbdi/frida-qbdi.js ./

# 3. 完成！现在可以使用了
```

**详细使用说明**: 请参考 [快速开始指南（使用预编译包）](./快速开始-使用预编译包.md)

---

## 环境准备

### 1. 开发机器环境

**必需软件：**
- CMake >= 3.12
- Ninja 或 Make
- C++17 工具链（gcc/clang）
- Android NDK（推荐 r25 或更高版本）
- ADB (Android Debug Bridge)
- Python 3.x（如果使用 PyQBDI）
- Node.js 和 npm（如果使用 Frida）

**安装 Android NDK：**

```bash
# 下载 NDK
wget https://dl.google.com/android/repository/android-ndk-r25c-darwin.dmg  # macOS
# 或
wget https://dl.google.com/android/repository/android-ndk-r25c-linux.zip   # Linux

# 解压并设置环境变量
export NDK_PATH=/path/to/android-ndk-r25c
```

### 2. Android 设备准备

**设备要求：**
- ARM64 (AArch64) 架构
- Android 7.0 (API 24) 或更高版本
- 已 root（推荐，某些功能需要）
- 启用 USB 调试

**设备配置：**

```bash
# 连接设备
adb devices

# 如果使用 Frida，需要关闭 SELinux（需要 root）
adb shell su -c "setenforce 0"

# 检查架构
adb shell getprop ro.product.cpu.abi
# 输出应该是: arm64-v8a
```

---

## 编译 QBDI for Android ARM64

### 步骤 1: 配置编译环境

```bash
# 进入 QBDI 项目目录
cd /Users/chennan/fridaproject/QBDI

# 创建编译目录
mkdir build-android-arm64
cd build-android-arm64
```

### 步骤 2: 运行配置脚本

```bash
# 设置 NDK 路径
export NDK_PATH=/path/to/android-ndk-r25c

# 运行配置脚本
../cmake/config/config-android-aarch64.sh
```

**配置说明：**
- `QBDI_PLATFORM=android`: 目标平台为 Android
- `QBDI_ARCH=AARCH64`: 目标架构为 ARM64
- `ANDROID_ABI=arm64-v8a`: Android ABI 为 ARM64
- `ANDROID_PLATFORM=24`: 最低支持 Android API 24 (Android 7.0)

### 步骤 3: 编译

```bash
# 使用 Ninja 编译
ninja

# 编译完成后，关键文件位于：
# - build-android-arm64/libQBDI.so          # 主库
# - build-android-arm64/tools/frida-qbdi.js # Frida 绑定
```

### 步骤 4: 推送到设备

```bash
# 推送 QBDI 库到设备
adb push libQBDI.so /data/local/tmp/

# 设置可执行权限
adb shell chmod 755 /data/local/tmp/libQBDI.so

# 如果使用 Frida
adb push tools/frida-qbdi.js /data/local/tmp/
```

---

## 三种使用方式

### 方式一：Frida + QBDI (推荐)

**优点：**
- 无需重新编译目标应用
- 动态注入，灵活性高
- JavaScript 脚本开发，快速迭代
- 支持 hook 系统库和应用代码

#### 1. 安装 Frida

```bash
# 在开发机器上安装 Frida
pip install frida-tools

# 下载对应架构的 frida-server
# 访问: https://github.com/frida/frida/releases
# 下载: frida-server-x.x.x-android-arm64.xz

# 解压并推送到设备
xz -d frida-server-*-android-arm64.xz
adb push frida-server-*-android-arm64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server

# 在设备上运行 frida-server（需要 root）
adb shell su -c "/data/local/tmp/frida-server &"
```

#### 2. 编写 Frida 脚本

创建文件 `trace_app.js`：

```javascript
import { VM, InstPosition, VMAction } from "./frida-qbdi.js";

// 初始化 QBDI VM
var vm = new VM();
console.log("[*] QBDI version: " + vm.version.string);

// 目标函数地址（可以通过符号名获取）
var targetFunction = Module.findExportByName("libnative-lib.so", "Java_com_example_myapp_MainActivity_stringFromJNI");

if (targetFunction) {
    console.log("[*] Target function found at: " + targetFunction);
    
    // 添加要监控的模块
    vm.addInstrumentedModuleFromAddr(targetFunction);
    
    // 定义指令回调函数
    var instCallback = vm.newInstCallback(function(vm, gpr, fpr, data) {
        var inst = vm.getInstAnalysis();
        console.log("0x" + inst.address.toString(16) + ": " + inst.disassembly);
        return VMAction.CONTINUE;
    });
    
    // 添加回调
    vm.addCodeCB(InstPosition.PREINST, instCallback, null);
    
    // Hook 目标函数
    Interceptor.attach(targetFunction, {
        onEnter: function(args) {
            console.log("[*] Entering function, starting trace...");
            
            // 获取寄存器状态
            var ctx = this.context;
            var state = vm.getGPRState();
            
            // 设置寄存器（ARM64）
            state.x0 = ptr(ctx.x0);
            state.x1 = ptr(ctx.x1);
            state.x2 = ptr(ctx.x2);
            state.x3 = ptr(ctx.x3);
            state.sp = ptr(ctx.sp);
            state.pc = ptr(targetFunction);
            
            vm.setGPRState(state);
            
            // 运行 VM 进行 trace
            vm.run(targetFunction, ptr(0xffffffffffffffff));
        },
        onLeave: function(retval) {
            console.log("[*] Function returned");
        }
    });
} else {
    console.error("[!] Target function not found!");
}
```

#### 3. 编译和运行脚本

```bash
# 安装 frida-compile
npm install -g frida-compile

# 将 frida-qbdi.js 复制到当前目录
cp /Users/chennan/fridaproject/QBDI/build-android-arm64/tools/frida-qbdi.js .

# 编译脚本
frida-compile trace_app.js -o trace_app_compiled.js

# 注入到正在运行的应用
frida -U -n com.example.myapp -l trace_app_compiled.js

# 或者启动应用并注入
frida -U -f com.example.myapp -l trace_app_compiled.js --no-pause
```

---

### 方式二：PyQBDI

**优点：**
- Python 语法，开发方便
- 适合自动化分析
- 可以结合其他 Python 分析工具

#### 1. 安装 PyQBDI

```bash
# 方法 1: 从 PyPI 安装（如果支持你的平台）
pip install PyQBDI

# 方法 2: 从源码编译
cd /Users/chennan/fridaproject/QBDI
python -m pip install --upgrade pip setuptools wheel build
python -m build -w
pip install dist/PyQBDI-*.whl
```

#### 2. 编写 Python 脚本

创建文件 `trace_native.py`：

```python
#!/usr/bin/env python3

import sys
import pyqbdi
import ctypes

def instruction_callback(vm, gpr, fpr, data):
    """每条指令执行前的回调"""
    inst = vm.getInstAnalysis()
    print(f"0x{inst.address:x}: {inst.disassembly}")
    return pyqbdi.CONTINUE

def memory_callback(vm, access, addr, size, value, data):
    """内存访问回调"""
    access_type = "READ" if access == pyqbdi.MEMORY_READ else "WRITE"
    print(f"[MEM {access_type}] 0x{addr:x} ({size} bytes) = 0x{value:x}")
    return pyqbdi.CONTINUE

def trace_native_function(lib_path, func_name):
    """Trace 本地库函数"""
    
    # 加载目标库
    lib = ctypes.CDLL(lib_path)
    func = getattr(lib, func_name)
    func_ptr = ctypes.cast(func, ctypes.c_void_p).value
    
    print(f"[*] Function {func_name} at 0x{func_ptr:x}")
    
    # 初始化 QBDI VM
    vm = pyqbdi.VM()
    
    # 创建栈空间
    state = vm.getGPRState()
    stack = pyqbdi.allocateVirtualStack(state, 0x100000)
    
    # 添加要监控的模块
    vm.addInstrumentedModuleFromAddr(func_ptr)
    
    # 注册回调
    vm.addCodeCB(pyqbdi.PREINST, instruction_callback, None)
    vm.addMemAccessCB(pyqbdi.MEMORY_READ_WRITE, memory_callback, None)
    
    # 记录内存访问
    vm.recordMemoryAccess(pyqbdi.MEMORY_READ_WRITE)
    
    # 模拟函数调用
    pyqbdi.simulateCall(state, 0xdeadbeef)
    
    # 运行 VM
    success = vm.run(func_ptr, 0xdeadbeef)
    
    print(f"[*] Trace {'succeeded' if success else 'failed'}")
    
    # 清理
    pyqbdi.alignedFree(stack)

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <library_path> <function_name>")
        sys.exit(1)
    
    trace_native_function(sys.argv[1], sys.argv[2])
```

#### 3. 在 Android 上运行

```bash
# 推送脚本到设备
adb push trace_native.py /data/local/tmp/

# 在设备上运行（需要安装 Python for Android）
adb shell "cd /data/local/tmp && python3 trace_native.py /system/lib64/libc.so strlen"
```

---

### 方式三：Native C/C++

**优点：**
- 性能最优
- 可以编译成独立的可执行文件
- 适合集成到现有 native 项目

#### 1. 编写 C 代码

创建文件 `trace_demo.c`：

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include "QBDI.h"

// 指令回调函数
VMAction instruction_callback(VMInstanceRef vm, GPRState *gprState, 
                              FPRState *fprState, void *data) {
    const InstAnalysis *inst = qbdi_getInstAnalysis(
        vm, QBDI_ANALYSIS_INSTRUCTION | QBDI_ANALYSIS_DISASSEMBLY);
    
    printf("0x%lx: %s\n", inst->address, inst->disassembly);
    
    return QBDI_CONTINUE;
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        printf("Usage: %s <library> <function>\n", argv[0]);
        return 1;
    }
    
    // 加载目标库
    void *handle = dlopen(argv[1], RTLD_LAZY);
    if (!handle) {
        printf("Error: Cannot load library %s\n", argv[1]);
        return 1;
    }
    
    // 获取函数地址
    void *func_ptr = dlsym(handle, argv[2]);
    if (!func_ptr) {
        printf("Error: Cannot find function %s\n", argv[2]);
        dlclose(handle);
        return 1;
    }
    
    printf("[*] Function %s at %p\n", argv[2], func_ptr);
    
    // 初始化 QBDI
    VMInstanceRef vm;
    qbdi_initVM(&vm, NULL, NULL, 0);
    
    // 获取寄存器状态
    GPRState *state = qbdi_getGPRState(vm);
    
    // 分配栈空间
    uint8_t *stack;
    qbdi_allocateVirtualStack(state, 0x100000, &stack);
    
    // 添加监控范围
    qbdi_addInstrumentedModuleFromAddr(vm, (rword)func_ptr);
    
    // 添加回调
    qbdi_addCodeCB(vm, QBDI_PREINST, instruction_callback, NULL, 0);
    
    // 记录内存访问
    qbdi_recordMemoryAccess(vm, QBDI_MEMORY_READ_WRITE);
    
    // 模拟调用
    qbdi_simulateCall(state, 0xdeadbeef, 0);
    
    // 运行 VM
    printf("[*] Starting trace...\n");
    bool success = qbdi_run(vm, (rword)func_ptr, 0xdeadbeef);
    printf("[*] Trace %s\n", success ? "succeeded" : "failed");
    
    // 清理
    qbdi_alignedFree(stack);
    qbdi_terminateVM(vm);
    dlclose(handle);
    
    return 0;
}
```

#### 2. 编译

创建 `CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.12)
project(QBDITraceDemo)

# 设置 QBDI 路径
set(QBDI_ROOT "/Users/chennan/fridaproject/QBDI/build-android-arm64")

# 添加 QBDI 头文件目录
include_directories("${QBDI_ROOT}/../include")

# 添加可执行文件
add_executable(trace_demo trace_demo.c)

# 链接 QBDI 库
target_link_libraries(trace_demo "${QBDI_ROOT}/libQBDI.so" dl)
```

编译：

```bash
# 创建编译目录
mkdir build-demo
cd build-demo

# 使用 Android NDK 编译
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK_PATH/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=arm64-v8a \
      -DANDROID_PLATFORM=24 \
      -DCMAKE_BUILD_TYPE=Release \
      -GNinja \
      ..

ninja

# 推送到设备
adb push trace_demo /data/local/tmp/
adb shell chmod 755 /data/local/tmp/trace_demo
```

#### 3. 运行

```bash
# 在设备上执行
adb shell "cd /data/local/tmp && LD_LIBRARY_PATH=. ./trace_demo /system/lib64/libc.so strlen"
```

---

## 示例代码

### 完整的 Frida trace 示例

详见 `examples/frida_trace_app.js`

### 完整的 PyQBDI 示例

详见 `examples/pyqbdi_trace.py`

### 完整的 Native C 示例

详见 `examples/native_trace_demo.c`

---

## 常见问题

### 1. libQBDI.so 加载失败

**问题：** `dlopen failed: library "libQBDI.so" not found`

**解决：**
```bash
# 确保库在正确位置
adb push libQBDI.so /data/local/tmp/

# 使用 LD_LIBRARY_PATH
adb shell "LD_LIBRARY_PATH=/data/local/tmp /data/local/tmp/your_app"

# 或者推送到系统库目录（需要 root）
adb shell su -c "mount -o remount,rw /system"
adb shell su -c "cp /data/local/tmp/libQBDI.so /system/lib64/"
```

### 2. SELinux 权限问题

**问题：** 注入失败或权限被拒绝

**解决：**
```bash
# 临时关闭 SELinux（需要 root）
adb shell su -c "setenforce 0"

# 检查状态
adb shell getenforce
# 应该输出: Permissive
```

### 3. Frida 连接失败

**问题：** `Failed to spawn: unable to find process with name 'com.example.app'`

**解决：**
```bash
# 确保 frida-server 在运行
adb shell ps | grep frida-server

# 检查 Frida 版本是否匹配
frida --version
adb shell /data/local/tmp/frida-server --version

# 重启 frida-server
adb shell su -c "killall frida-server"
adb shell su -c "/data/local/tmp/frida-server &"
```

### 4. trace 输出为空

**问题：** 回调函数没有被触发

**解决：**
```javascript
// 确保添加了正确的模块
vm.addInstrumentedModuleFromAddr(functionAddress);

// 或者添加整个模块
vm.addInstrumentedModule("libnative-lib.so");

// 检查地址范围
console.log("Instrumented ranges: " + vm.getInstrumentedRanges());
```

### 5. 性能问题

**问题：** trace 速度很慢

**优化建议：**
```javascript
// 1. 只 trace 特定范围
vm.addInstrumentedRange(startAddr, endAddr);

// 2. 减少回调
// 不要在每条指令上都执行复杂操作

// 3. 使用条件过滤
var instCallback = vm.newInstCallback(function(vm, gpr, fpr, data) {
    var inst = vm.getInstAnalysis();
    // 只记录特定类型的指令
    if (inst.mnemonic === "BL" || inst.mnemonic === "BLR") {
        console.log("Call: 0x" + inst.address.toString(16));
    }
    return VMAction.CONTINUE;
});
```

### 6. 内存访问 trace

```javascript
// 添加内存访问回调
var memCallback = vm.newMemCallback(function(vm, access, address, size, value, gpr, fpr, data) {
    var accessType = (access === MemoryAccess.READ) ? "READ" : "WRITE";
    console.log("[" + accessType + "] 0x" + address.toString(16) + 
                " (" + size + " bytes) = 0x" + value.toString(16));
    return VMAction.CONTINUE;
});

vm.addMemAccessCB(MemoryAccess.MEMORY_READ_WRITE, memCallback, null);
vm.recordMemoryAccess(MemoryAccess.MEMORY_READ_WRITE);
```

---

## 进阶技巧

### 1. 条件断点

```javascript
var breakpointAddr = ptr(0x12345678);
var hitCount = 0;

var instCallback = vm.newInstCallback(function(vm, gpr, fpr, data) {
    var inst = vm.getInstAnalysis();
    if (inst.address.equals(breakpointAddr)) {
        hitCount++;
        console.log("[*] Breakpoint hit " + hitCount + " times");
        gpr.dump();
        
        // 条件：如果 x0 寄存器的值为特定值
        if (gpr.x0.equals(ptr(0x1000))) {
            console.log("[!] Condition met!");
            // 可以修改寄存器值
            gpr.x0 = ptr(0x2000);
            vm.setGPRState(gpr);
        }
    }
    return VMAction.CONTINUE;
});
```

### 2. 函数调用追踪

```javascript
var callStack = [];

var bbCallback = vm.newVMCallback(function(vm, evt, gpr, fpr, data) {
    var inst = vm.getInstAnalysis();
    
    // ARM64 的函数调用指令
    if (inst.mnemonic === "BL" || inst.mnemonic === "BLR") {
        callStack.push({
            from: inst.address,
            to: inst.operands[0].value
        });
        console.log("CALL: 0x" + inst.address.toString(16) + 
                   " -> 0x" + inst.operands[0].value.toString(16));
    }
    // 返回指令
    else if (inst.mnemonic === "RET") {
        if (callStack.length > 0) {
            var lastCall = callStack.pop();
            console.log("RET: 0x" + inst.address.toString(16));
        }
    }
    
    return VMAction.CONTINUE;
});

vm.addVMEventCB(VMEvent.BASIC_BLOCK_ENTRY, bbCallback, null);
```

### 3. 代码覆盖率统计

```javascript
var coveredBlocks = new Set();

var bbCallback = vm.newVMCallback(function(vm, evt, gpr, fpr, data) {
    if (evt === VMEvent.BASIC_BLOCK_ENTRY) {
        var inst = vm.getInstAnalysis();
        coveredBlocks.add(inst.address.toString(16));
    }
    return VMAction.CONTINUE;
});

vm.addVMEventCB(VMEvent.BASIC_BLOCK_ENTRY, bbCallback, null);

// 运行后统计
console.log("Total basic blocks covered: " + coveredBlocks.size);
console.log("Covered addresses: ", Array.from(coveredBlocks));
```

---

## 参考资源

- **QBDI 官方文档**: https://qbdi.readthedocs.io/
- **QBDI GitHub**: https://github.com/QBDI/QBDI
- **Frida 官方文档**: https://frida.re/docs/
- **Android NDK 文档**: https://developer.android.com/ndk/

---

## 联系与支持

如有问题，请参考：
- QBDI Issues: https://github.com/QBDI/QBDI/issues
- 官方文档: https://qbdi.readthedocs.io/

## 更新日志

- 2025-11-14: 创建初始版本

