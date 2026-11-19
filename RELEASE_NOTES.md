# Copper 1.0.0

Copper（原 CW Script）第一个独立发行版：语言内核与 IDE 分开，环境编译器单列一仓。

## 下载

| 系统 | 文件 |
|---|---|
| Windows x64 | `Copper-1.0.0-windows-x64.zip` |
| Linux x64 | `Copper-1.0.0-linux-x64.tar.gz` |
| macOS arm64 | `Copper-1.0.0-macos-arm64.tar.gz` |

解压后把包里的 `bin/` 加进 `PATH`，敲 `co --version` 验证。

## 内容

- `copper`：环境编译器（`bootstrap/colang.co` 经 AOT 编出）
- `copper-self`：自举版，输出与 `copper` 逐字节一致
- `copper-run`：解释执行、`--repl`、跑脚本
- `copper-embed`：把脚本焊进宿主程序的嵌入 API
- `lib/colang_rt.rs`：AOT 生成程序的运行时前奏
- `bin/co` / `bin/copper`：一行命令的启动器

## 语言

- 中文关键字，英文写法同时可用
- 多参数、默认值、递归、内联函数与闭包
- 九个回调式高阶函数、`lazy` 惰性参数（`AND2` / `OR2` 真短路）
- 指针 `&` / `*`、类与继承、`try` / `catch`
- 跨平台内置：`exists` / `isfile` / `isdir` / `filesize` / `getenv` / `pathjoin` / `runcmd` / `osname` / `strfind` / `writefile`

## 性能

`fib(30)` 递归（扣掉启动开销）：AOT 整型特化 **4.0 ms**，VM 93.1 ms，CPython 124.4 ms。

## 验证

`cargo test` 371 项全过；自举不动点两代 `main.rs` SHA256 一致。
