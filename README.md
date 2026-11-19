# Copper

**Copper**（原 CW Script）—— 一门中文编程语言，出自小方工作室。词法 → 语法 → 字节码 → 虚拟机；另有一条 Rust AOT 通道，把脚本直接编成原生程序。编译器本身也是用 Copper 写的（`bootstrap/colang.co`），能自举。

- 官网 / 下载：<https://github.com/xiaofangstudio/Copper/releases>
- 配套 IDE：[Float IDE](https://github.com/xiaofangstudio/Float-IDE)

---

## 下载

每个 Release 挂三种系统的安装包：

| 系统 | 文件 | 内容 |
|---|---|---|
| Windows x64 | `Copper-<版本>-windows-x64.zip` | `copper.exe`、`copper-self.exe`、`copper-run.exe`、`copper-embed.exe` + `co.cmd` / `copper.cmd` |
| Linux x64 | `Copper-<版本>-linux-x64.tar.gz` | `copper`、`copper-self`、`copper-run`、`copper-embed` + `co.sh` / `copper.sh` |
| macOS arm64 | `Copper-<版本>-macos-arm64.tar.gz` | 同上（Mach-O） |

解压后把 `bin/` 加进 `PATH`，或者直接用包里的启动脚本：

```bash
# Linux / macOS
tar -xzf Copper-*.tar.gz
export PATH="$PWD/copper/bin:$PATH"
co --version
```

```powershell
# Windows
Expand-Archive Copper-*-windows-x64.zip -DestinationPath .\copper
.\copper\bin\co.cmd --version
```

> macOS 的二进制需要在 Mac 上编（CI 的 macOS runner 产出），Windows 上交叉编译不出来。`copper_env/bin/linux-x64/` 与 `bin/macos-*` 都是原生产物，**清理仓库时不要删**。

---

## 快速开始

```bash
co                      # 版本 + 用法
co hello.co             # 解释执行（COVM）
co build hello.co       # AOT 编成原生程序（Linux/macOS 加 -o 指定产物名）
co --repl               # 交互模式
co check hello.co       # 静态检查
co env                  # 环境详情
```

一段最小程序：

```
/* 一行一句，缩进成块
ask "你叫什么名字？"
str name = ans
print(join("你好，", name))
```

```
fn greet(name, greeting = "你好")
    return join(greeting, "，", name)

print(greet("小明"))
print(greet("小红", "晚上好"))
```

完整的 16 组示例在 [`language/samples/`](language/samples)，从排序、猜数字、对话游戏到指针对战都有。

---

## 语言概况

| 能力 | 说明 |
|---|---|
| 中文关键字 | `如果 / 否则 / 当 / 重复 / 函数 / 类` 都有，英文写法同时可用 |
| 函数 | 多参数、默认值（可引用前面的参数）、`return`、递归、先用后定义 |
| 内联与闭包 | `fn(x) return x + base`，内层函数记住创建时的环境 |
| 高阶函数 | `data.map` / `filter` / `reduce` / `groupby` / `index_by` / `any` / `all` / `find` / `find_index` |
| 惰性参数 | `lazy bool y` 按需求值，`AND2` / `OR2` 真短路 |
| 指针 | `&` 取址、`*` 读写，列表元素可取地址，VM 与 AOT 语义一致 |
| 类 | 字段默认值、`init`、方法互调、`self`、继承与 `super(…)` |
| 错误处理 | `try` / `catch`，错误带行列号与中文说明 |
| 跨平台内置 | `exists` / `isfile` / `isdir` / `filesize` / `getenv` / `pathjoin` / `runcmd` / `osname` / `strfind` / `writefile` |

整数是 64 位精确整数：超出 `i64` 的字面量在解析期就报错，不会悄悄变成浮点。

---

## 两个编译器

| 程序 | 来源 | 用途 |
|---|---|---|
| `copper` | `bootstrap/colang.co` 经 AOT 编成 | 环境编译器，编译用户的 Copper 程序 |
| `copper-self` | 用上面的 `copper` 再编一次自己 | 自举版，输出与 `copper` 逐字节一致 |
| `copper-run` | `examples/run.rs` | 解释执行、`--repl`、跑脚本 |
| `copper-embed` | `examples/embed.rs` | 把脚本焊进宿主程序的嵌入 API |

自举不动点：`copper` 编自己得到 `main.rs(A)`，`copper-self` 再编自己得到 `main.rs(B)`，两份 SHA256 必须完全相同。`scripts/rebuild-linux.sh` 每次都会验这一条。

---

## 构建

内核是纯 Rust、零外部 crate，离线也能编：

```bash
cd float-lang
cargo build --release --examples     # 编出 aot / run / embed
cargo test                           # 371 项
```

重新产出某一代编译器（以 Linux 为例）：

```bash
bash copper_env/scripts/rebuild-linux.sh          # 全流程，含自举不动点复验
bash copper_env/scripts/rebuild-linux.sh --fast   # 跳过不动点
```

三平台安装包由 GitHub Actions 产出：打一个 `v*` 标签，或手动 Run workflow（见 [`.github/workflows/release.yml`](.github/workflows/release.yml)）。

---

## 目录

```
float-lang/            语言内核（独立 crate，零外部依赖）
  src/                 lexer → parser → sema → bytecode → vm，另有 aot
  tests/               语言行为、AOT、作用域、F5 游戏链路
  examples/            aot / run / embed / ast_dump / dump_tokens
  bootstrap/           用 Copper 写的编译器（colang.co 及词法 / 语法 / 代码生成）
copper_env/            可直接分发的环境
  bin/<platform>/      各平台四个可执行
  lib/colang_rt.rs     生成程序的运行时前奏
  scripts/             重编脚本与平台说明
language/              标准库目录（catalog）与 16 组示例
docs/                  教程、标准库、支持矩阵、插件与 MCP
```

---

## 性能

`fib(30)` 递归，扣掉进程启动开销：

| 实现 | 耗时 |
|---|---|
| Copper AOT（整型特化） | **4.0 ms** |
| Copper VM | 93.1 ms |
| CPython 3.13 | 124.4 ms |

AOT 会把参数、局部、返回值全是 `i64`（或 `f64`）的函数整体编译成原生数值路径，递归上限提到 100 万层。

---

## 许可

MIT © 2026 小方工作室
