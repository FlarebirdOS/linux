## 使用 Archlinux 的 config 要修改以下内容

### 例如 内核版本为 6.16.5

```diff
-# Linux/x86 6.16.5-arch1 Kernel Configuration
+# Linux/x86 6.16.5 Kernel Configuration
```

#### 系统当前 Gcc 版本 15.2.0
```diff
-CONFIG_CC_VERSION_TEXT="gcc (GCC) 15.2.1 20250813"
+CONFIG_CC_VERSION_TEXT="gcc (Flarebird Gcc 15.2.0) 15.2.0"
-CONFIG_GCC_VERSION=150201
+CONFIG_GCC_VERSION=150200
```

#### 系统当前 Rust 版本 1.89.0 
```diff
-CONFIG_RUSTC_VERSION=108800
+CONFIG_RUSTC_VERSION=108900
```

#### 系统当前 Llvm 版本 21.1.0
```diff
-CONFIG_RUSTC_LLVM_VERSION=200108
+CONFIG_RUSTC_LLVM_VERSION=210100
```

#### 用来指定系统启动时 默认主机名
```diff
-CONFIG_DEFAULT_HOSTNAME="archlinux"
+CONFIG_DEFAULT_HOSTNAME="flarebird"
```

#### 系统当前 Rust 版本 1.89.0 
#### 系统当前 Rust-bindgen 版本 0.72.1
```diff
-CONFIG_RUSTC_VERSION_TEXT="rustc 1.89.0 (29483883e 2025-08-04) (Arch Linux rust 1:1.89.0-1)"
-CONFIG_BINDGEN_VERSION_TEXT="bindgen 0.72.0"
+CONFIG_RUSTC_VERSION_TEXT="rustc 1.89.0 (29483883e 2025-08-04) (Flarebird Linux rust 1.89.0-1)"
+CONFIG_BINDGEN_VERSION_TEXT="bindgen 0.72.1"
```

#### 系统发生内核崩溃（panic）时，在崩溃屏幕上显示的 QR 码所指向的 URL 基础地址。
```diff
-CONFIG_DRM_PANIC_SCREEN_QR_CODE_URL="https://panic.archlinux.org/panic_report#"
+CONFIG_DRM_PANIC_SCREEN_QR_CODE_URL="https://FlarebirdOS.github.io"
```
