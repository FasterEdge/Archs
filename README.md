# Archs

FasterEdge 组件的平台与处理器架构兼容性说明。

## Go 跨平台能力

Go 工具链使用两个环境变量描述目标平台：

- `GOOS`：目标操作系统或执行环境。
- `GOARCH`：目标处理器指令集架构。

在多数不依赖目标平台 C 工具链的项目中，可以通过设置 `GOOS`、`GOARCH` 并关闭 CGO，从一台开发机交叉编译其他平台的程序：

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build ./...
CGO_ENABLED=0 GOOS=windows GOARCH=arm64 go build ./...
CGO_ENABLED=0 GOOS=freebsd GOARCH=riscv64 go build ./...
```

Go 官方下载页主要提供常用平台的预编译工具链。其他官方端口请参阅 [Go Downloads](https://go.dev/dl/) 页面中的 **Other Ports**：

> Use https://go.dev/dl/ under "Other Ports".

并非 `go tool dist list` 中的每个组合都会提供可直接下载的二进制工具链；未提供预编译包时，可以使用已有 Go 工具链交叉编译，或按照 [Installing Go from source](https://go.dev/doc/install/source) 构建工具链。

## 官方目标清单

以下清单来自 Go `1.25.5` 的：

```bash
go tool dist list
```

共包含 **15 种 GOOS、14 种 GOARCH、48 个有效 GOOS/GOARCH 组合**。有效性以组合为单位，不能任意混合下表中的 GOOS 与 GOARCH。

### 按 GOOS 分类

| GOOS | 官方 GOARCH 组合 |
|---|---|
| `aix` | `ppc64` |
| `android` | `386`, `amd64`, `arm`, `arm64` |
| `darwin` | `amd64`, `arm64` |
| `dragonfly` | `amd64` |
| `freebsd` | `386`, `amd64`, `arm`, `arm64`, `riscv64` |
| `illumos` | `amd64` |
| `ios` | `amd64`, `arm64` |
| `js` | `wasm` |
| `linux` | `386`, `amd64`, `arm`, `arm64`, `loong64`, `mips`, `mips64`, `mips64le`, `mipsle`, `ppc64`, `ppc64le`, `riscv64`, `s390x` |
| `netbsd` | `386`, `amd64`, `arm`, `arm64` |
| `openbsd` | `386`, `amd64`, `arm`, `arm64`, `ppc64`, `riscv64` |
| `plan9` | `386`, `amd64`, `arm` |
| `solaris` | `amd64` |
| `wasip1` | `wasm` |
| `windows` | `386`, `amd64`, `arm64` |

### 完整 GOOS/GOARCH 组合

```text
aix/ppc64
android/386
android/amd64
android/arm
android/arm64
darwin/amd64
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
freebsd/arm64
freebsd/riscv64
illumos/amd64
ios/amd64
ios/arm64
js/wasm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/loong64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/riscv64
linux/s390x
netbsd/386
netbsd/amd64
netbsd/arm
netbsd/arm64
openbsd/386
openbsd/amd64
openbsd/arm
openbsd/arm64
openbsd/ppc64
openbsd/riscv64
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
wasip1/wasm
windows/386
windows/amd64
windows/arm64
```

## GOARCH 说明

| GOARCH | 处理器或执行环境 |
|---|---|
| `386` | 32 位 x86 |
| `amd64` | 64 位 x86-64 |
| `arm` | 32 位 ARM |
| `arm64` | 64 位 ARM / AArch64 |
| `loong64` | 64 位 LoongArch |
| `mips` | 32 位大端 MIPS |
| `mipsle` | 32 位小端 MIPS |
| `mips64` | 64 位大端 MIPS |
| `mips64le` | 64 位小端 MIPS |
| `ppc64` | 64 位大端 PowerPC |
| `ppc64le` | 64 位小端 PowerPC |
| `riscv64` | 64 位 RISC-V |
| `s390x` | IBM z/Architecture |
| `wasm` | WebAssembly |

## 查询当前工具链支持范围

当前主机：

```bash
go env GOOS GOARCH
```

当前 Go 工具链支持的全部目标：

```bash
go tool dist list
```

仅列出 JSON：

```bash
go tool dist list -json
```

检查特定组合是否存在：

```bash
go tool dist list | grep '^linux/arm64$'
```

项目文档中的静态清单代表编写时使用的 Go `1.25.5`。Go 后续版本可能新增、调整或移除端口，因此发布构建前应以正在使用的工具链输出为准。

## 交叉编译限制

### CGO

`CGO_ENABLED=0` 时，纯 Go 项目通常可以直接交叉编译。启用 CGO 后，还必须准备目标平台的 C 编译器、链接器、系统头文件和目标库：

```bash
CGO_ENABLED=1 \
GOOS=linux \
GOARCH=arm64 \
CC=aarch64-linux-gnu-gcc \
go build ./...
```

### 平台专用代码

项目仍可能因以下原因无法在某个官方目标上构建或运行：

- 使用了带 GOOS/GOARCH 构建约束的源文件；
- 依赖仅支持部分平台的第三方库；
- 调用了特定操作系统的 syscall；
- 依赖动态库、驱动、GUI 或硬件能力；
- 目标环境未实现程序所需的网络、文件系统或进程能力。

因此，Go 工具链支持某个 GOOS/GOARCH，并不自动代表每个 Go 项目都兼容该组合。

### WebAssembly

Go 官方当前包含两类 WebAssembly 目标：

- `js/wasm`：JavaScript 宿主环境；通常需要 Go 发行版配套的 `wasm_exec.js`。
- `wasip1/wasm`：WASI Preview 1 宿主环境。

两者的系统接口和宿主能力不同，不能仅通过替换 GOOS 认为运行环境等价。

## 官方资料

- [Go Downloads](https://go.dev/dl/) — 官方工具链下载及 Other Ports。
- [Installing Go from source](https://go.dev/doc/install/source) — 从源码构建 Go 和平台要求。
- [`go` command documentation](https://pkg.go.dev/cmd/go) — Go 构建命令与环境变量。
- [`go tool dist`](https://pkg.go.dev/cmd/dist) — 查询当前工具链支持的平台组合。
- [Go Porting Policy](https://go.dev/wiki/PortingPolicy) — 新端口和端口维护策略。
- [Minimum Requirements](https://go.dev/wiki/MinimumRequirements) — 各架构最低运行要求。
