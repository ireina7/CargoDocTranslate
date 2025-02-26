## 安装

### 安装 Rust 和 Cargo

获得Cargo的最简单的方法是通过[rustup]来安装当前[Rust]稳定版本。
`rustup` 安装Rust时也会安装 `cargo` 。

在Linux和macOS系统上，可以这样:

```console
curl https://sh.rustup.rs -sSf | sh
```

它将下载一个脚本，并开始安装。如果一切顺利，你会看到:

```console
Rust is installed now. Great!
```

在Windows操作系统下，下载并运行 [rustup-init.exe] 。它将在控制台中开始安装，并在成功时显示上述消息。

之后，就可以使用 `rustup` 命令来安装 Rust 或 Cargo  `beta` "测试" 或 `nightly` "每日" 版本。

关于其他安装选项和信息，请访问 Rust 网站的 [install][install-rust] 页面。

### 从源代码构建和安装Cargo

做为选择，你可以[从源代码构建Cargo][compiling-from-source]

[rust]: https://www.rust-lang.org/
[rustup]: https://rustup.rs/
[rustup-init.exe]: https://win.rustup.rs/
[install-rust]: https://www.rust-lang.org/tools/install
[compiling-from-source]: https://github.com/rust-lang/cargo#compiling-from-source
