+++
title = "Rust termcolor 用法速查"
date = 2025-11-17
description = "错误显示颜色的用法。"

[taxonomies]
tags = ["rust", "cli","result"]
+++
# termcolor 库使用指南

`termcolor` 是一个简单、跨平台的 Rust 库，用于向终端写入彩色文本。它能自动处理 ANSI 转义序列（Unix-like 系统）和 Windows 控制台 API，确保颜色在不同操作系统上正确显示。
> 📚 [**官方文档**](https://docs.rs/termcolor/latest/termcolor/)  

## 📦 1. 添加依赖

在 `Cargo.toml` 中添加：
```shell
cargo add termcolor
```
```toml
[dependencies]
termcolor = "1.4.1" # 推荐使用最新稳定版本
```

## 🧩 2. 核心概念与类型

### `WriteColor` Trait
*   扩展了标准的 `std::io::Write` trait。
*   提供了设置前景色、背景色和重置颜色的方法。
*   任何实现了 `WriteColor` 的类型都可以写入带颜色的文本。

### `ColorChoice` (颜色选择策略)
决定是否以及如何使用颜色：
*   `Always`: **始终**使用颜色（即使输出到管道或不支持颜色的终端）。
*   `Never`: **从不**使用颜色。
*   `Auto`: **自动判断**（推荐）。会检查：
    *   `NO_COLOR` 环境变量（存在即禁用颜色）。
    *   `TERM=dumb` 环境变量。
    *   （非 Windows）`TERM` 环境变量未设置。
    *   *注意：`termcolor` 不自动检测是否为 TTY，如需此功能，使用 `std::io::IsTerminal`。*

### 主要写入类型

| 类型 | 用途 | 类比 `std::io` |
| :--- | :--- | :--- |
| `StandardStream` | 直接写入 stdout 或 stderr。支持颜色。 | `Stdout` / `Stderr` |
| `StandardStreamLock` | `StandardStream` 的锁定版本，用于多线程或需要显式锁定的场景。 | `StdoutLock` / `StderrLock` |
| `BufferWriter` + `Buffer` | 用于更复杂的场景，如多线程。先写入内存缓冲区，再统一输出。 | N/A |

## 🛠 3. 使用方法

### 方法一：使用 `StandardStream` (最常用)

适用于大多数单线程命令行应用。

```rust
use std::io::{self, Write};
use termcolor::{Color, ColorChoice, ColorSpec, StandardStream, WriteColor};

fn main() -> io::Result<()> {
    // 1. 创建带颜色的 stdout/stderr 写入器
    let mut stdout = StandardStream::stdout(ColorChoice::Auto); // 推荐 Auto
    // let mut stderr = StandardStream::stderr(ColorChoice::Auto);

    // 2. 定义颜色规格 (ColorSpec)
    let mut spec = ColorSpec::new();
    spec.set_fg(Some(Color::Green)) // 前景色 (文本颜色)
         .set_bg(Some(Color::Rgb(50, 50, 50))) // 背景色 (RGB)
         .set_bold(true)           // 加粗
         .set_underline(true)      // 下划线 (支持有限)
         .set_intense(true);       // 高亮/亮色

    // 3. 应用颜色规格
    stdout.set_color(&spec)?; // 返回 Result，需处理错误

    // 4. 写入带颜色的文本
    writeln!(&mut stdout, "这是一行绿色加粗的文本！")?;

    // 5. 重置颜色 (非常重要！)
    stdout.reset()?;

    // 6. 写入普通文本 (可选)
    writeln!(&mut stdout, "这行是普通颜色。")?;

    // 7. 刷新缓冲区 (确保立即输出)
    stdout.flush()?;

    Ok(())
}
```

### 方法二：使用 `BufferWriter` (多线程/复杂场景)

适用于需要避免输出交错或多线程写入的场景。

```rust
use std::io::{self, Write};
use termcolor::{BufferWriter, Color, ColorSpec, WriteColor};

fn main() -> io::Result<()> {
    // 1. 创建 BufferWriter (管理缓冲区)
    let mut bufwtr = BufferWriter::stdout(ColorChoice::Auto);

    // 2. 获取一个内存缓冲区
    let mut buffer = bufwtr.buffer();

    // 3. 在缓冲区内设置和写入颜色
    buffer.set_color(ColorSpec::new().set_fg(Some(Color::Red)))?;
    writeln!(&mut buffer, "红色错误信息")?;
    
    // 可以多次设置不同颜色
    buffer.set_color(ColorSpec::new().set_fg(Some(Color::Blue)))?;
    writeln!(&mut buffer, "蓝色提示信息")?;

    // 4. 将整个缓冲区内容写入 stdout
    bufwtr.print(&buffer)?;

    // 5. 重置 (通常在 BufferWriter 打印后自动处理，但显式调用也无妨)
    // 注意：这里是重置 stdout 本身，不是 buffer
    let mut stdout = StandardStream::stdout(ColorChoice::Auto);
    stdout.reset()?;
    stdout.flush()?;

    Ok(())
}
```

### 方法三：包装任意 `Write` 类型

你可以将 `termcolor` 的功能应用到任何实现了 `std::io::Write` 的类型上。

```rust
use std::io::{self, Write};
use termcolor::{Ansi, NoColor, WriteColor};

// 包装一个 Vec<u8> (内存缓冲)
let mut buf = Vec::new();
{
    let mut ansi_writer = Ansi::new(&mut buf); // 或者 NoColor::new(&mut buf)
    ansi_writer.set_color(termcolor::ColorSpec::new().set_fg(Some(termcolor::Color::Yellow)))?;
    write!(&mut ansi_writer, "黄色文本")?;
    ansi_writer.reset()?; // 重置
}
// 此时 buf 包含了带 ANSI 转义序列的字节
println!("Buffer contents: {:?}", String::from_utf8_lossy(&buf));
```

## 🎨 4. 颜色 (`Color`) 类型

*   **预定义颜色**:
    *   `Black`, `Red`, `Green`, `Yellow`, `Blue`, `Magenta`, `Cyan`, `White`
    *   `Rgb(r, g, b)`: 24位真彩色 (e.g., `Color::Rgb(255, 128, 0)`)
    *   `Ansi256(u8)`: 256色模式

## ⚠️ 5. 重要注意事项

1.  **错误处理**: `set_color`, `write!`, `reset` 等方法都返回 `io::Result<()>`，必须处理错误（通常用 `?` 操作符）。
2.  **重置颜色**: 每次设置颜色后，**务必**在适当时候调用 `reset()`。否则后续所有终端输出都可能保持该颜色，影响用户体验。
3.  **刷新缓冲区**: 调用 `flush()` 确保内容立即输出到终端，尤其是在程序结束前。
4.  **`ColorChoice::Auto`**: 对于面向用户的 CLI 工具，**强烈推荐**使用 `Auto`，尊重用户的环境设置（如 `NO_COLOR`）。
5.  **性能**: 对于简单的单线程输出，`StandardStream` 足够高效。`BufferWriter` 主要用于解决多线程输出交错问题。
6.  **TTY 检测**: `termcolor` 的 `Auto` **不**包含 TTY 检测。如果需要更精确的控制（例如，只有连接到终端时才启用颜色），结合使用 `std::io::IsTerminal` trait:
    ```rust
    use std::io::IsTerminal;
    let choice = if std::io::stdout().is_terminal() {
        ColorChoice::Auto
    } else {
        ColorChoice::Never
    };
    ```
