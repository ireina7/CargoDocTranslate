## Cargo 构建目标

Cargo 包由 *构建目标* 组成，构建目标对应着可以编译成 crate 的源文件。
包中可以包含 [library](#library), [binary](#binaries),
[example](#examples), [test](#tests) 和 [benchmark](#benchmarks) 的构建目标。
构建目标列表可以在 `Cargo.toml` 目录清单中配置，但总是由源文件的 [目录结构][package layout]
[inferred automatically](#target-auto-discovery)

关于构建目标的配置，查看下面的 [Configuring a target](#configuring-a-target)。

### 类库

类库的构建目标定义了 "类库" 的概念，即可以被其他类库和可执行程序使用和链接的库。
默认文件名会是 `src/lib.rs` ，类库的名词默认为包的名称。
包可以只有一个类库。你可以在 `Cargo.toml` 的 `[lib]` 标签下 [自定义] 类库的设置。

```toml
# 在 Cargo.toml 自定义类库设置的例子。
[lib]
crate-type = ["cdylib"]
bench = false
```

### 二进制构建目标

二进制构建目标为编译后可以运行的可执行程序。
默认的二进制文件名为 `src/main.rs` ，它默认是包的名称。
额外的二进制文件会放在 [`src/bin/` 目录][package layout]。
你可以在 `Cargo.toml` 的 `[[bin]]` 标签下为每个二进制文件的 [自定义] 设置。

二进制文件可以使用包的类库提供的公共 API。
它们通过 `Cargo.toml` 中定义的 [`[dependencies]`][dependencies] 链接起来。

你可以使用带 `--bin <bin-名称>` 参数的 [`cargo run`] 命令来运行单个二进制文件。
可以使用 [`cargo install`] 将可以执行文件赋值到通用目录。

```toml
# 在 Cargo.toml 自定义二进制文件设置的例子。
[[bin]]
name = "cool-tool"
test = false
bench = false

[[bin]]
name = "frobnicator"
required-features = ["frobnicate"]
```

### 例子

[`examples` 目录][package layout] 下的文件便是使用类库功能的例子。
编译后，文件会生成在 [`target/debug/examples` 目录][build cache]。

例子可以使用包中类库的公共 API 。它们通过 `Cargo.toml` 中定义的 [`[dependencies]`][dependencies] 和
[`[dev-dependencies]`][dev-dependencies] 链接起来。

例子默认是二进制可执行文件(带有 `main()` 函数)。
你可以指定 [`crate-type` 字段](#the-crate-type-field) 另一个例子编译为类库:

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

你可以使用带 `--example <例子名>` 参数的 [`cargo run`] 命令来运行单个例子。
使用带 `--example <例子名>` 参数的 [`cargo install`] 命令来将二进制可执行文件复制到通用目录。
使用 [`cargo test`] 命令编译例子可以防止位衰减。
如果你想运行 [`cargo test`] 的例子中存在 `#[test]` 函数，你需要将 [the `test` field](#the-test-field) 为 `true` 。

### 测试

Cargo 项目中有两种测试的方式:

* *单元测试* 为在类库或二进制(或任何启用了 [the `test` field](#the-test-field) 的构建目标)中
使用 [`#[test]` 属性][test-attribute] 标记的函数，这种测试可以访问构建目标中定义的私有 API 。
* *集成测试* 为单独的与项目类库链接的二进制可执行文件。
它也包含了 `#[test]` 函数并且仅能访问 *公有* API 。

使用 [`cargo test`] 命令运行测试。默认情况下，Cargo 和 `rustc` 使用 [libtest harness] ，
它负责收集和并行执行带有 [`#[test]` 属性][test-attribute] 标记的函数，并报告所有测试的成功或失败情况。
如果你想使用不同的使用或测试策略可以查看 [the `harness` field](#the-harness-field) 。

> **注意**: Cargo 有另一种特殊的测试方式:
> [documentation tests][documentation examples]。
> 它们通过 `rustdoc` 处理并且使用略有不同的执行模型。
> 你可以在 [`cargo test`][cargo-test-documentation-tests] 查看更多信息。

[libtest harness]: ../../rustc/tests/index.html
[cargo-test-documentation-tests]: ../commands/cargo-test.md#documentation-tests

#### 集成测试

[`tests` 目录][package layout] 目录下的文件便是集成测试。
当你运行 [`cargo test`] 时，Cargo 会将该目录下的每个文件编译为单独的 crate 并执行。

集成测试可以使用包中类库的公有 API 。
它们也通过 `Cargo.toml` 中定义的 [`[dependencies]`][dependencies] 和
[`[dev-dependencies]`][dev-dependencies] 链接起来。

如果你想在多个集成测试中复用代码，你可以将代码放在一个单独的模块中，
比如 `tests/common/mod.rs` 并在测试中使用 `mod common;` 导入这个模块。

每个单独二进制可执行文件产生一个集成测试结果，[`cargo test`] 命令会串行执行集成测试。
在某些情况下可能效率较低，比如编译耗时较长或运行时没有充分利用CPU多个核心。
如果你需要进行大量集成测试，推荐创建单个集成测试，并将测试分割成多个模块。
libtest harness 会自动寻找和并行执行所有 `#[test]` 标记的函数。
你可以将模块名传递给 [`cargo test`] 来将测试的范围限定在模块内。

二进制构建目标的集成测试是自动构建的。
这样集成测试可以检验二进制文件执行的行为。
集成测试在构建时会设置 `CARGO_BIN_EXE_<name>` [environment variable] ，这样它可以使用 [`env` macro] 来确定可执行文件的位置。

[environment variable]: environment-variables.md#environment-variables-cargo-sets-for-crates
[`env` macro]: ../../std/macro.env.html

### 性能测试

使用 [`cargo bench`] 命令来测试代码的性能。它与 [tests](#tests) 遵循相同的结构，
需要性能测试的函数要用 `#[bench]` 标记。
以下几点与测试相似:

* 性能测试的文件放置在 [`benches` 目录][package layout] 目录下。
* 类库和二进制文件中定义的性能测试可以访问构建目标中定义的 *私有* API。
`benches` 目录下的性能测试仅能访问 *公有* API 。
* [The `bench` field](#the-bench-field) 可以用来定义默认对哪个构建目标进行性能测试。
* [The `harness` field](#the-harness-field) 可以用来禁用内置的 harness 。

> **注意**: [`#[bench]`
> attribute](../../unstable-book/library-features/test.html) 目前还不稳定并且仅在 [nightly channel] 中提供。
> [crates.io](https://crates.io/keywords/benchmark) 提供的一些包也许有助于在 stable channel 中运行性能测试，
> 比如 [Criterion](https://crates.io/crates/criterion)。

### 配置构建目标

对于指定构建目标应该如何构建，`Cargo.toml` 中所有的 `[lib]`, `[[bin]]`,
`[[example]]`, `[[test]]` 和 `[[bench]]` 标记都支持相似的配置。
比如 `[[bin]]` 这种双层括号标记代表 [array-of-table of TOML](https://toml.io/en/v1.0.0-rc.3#array-of-tables)，
意味着你可以在 crate 添加多个 `[[bin]]` 来生成多个可执行文件。
比如 `[lib]` 代表普通 TOML 列表，意味着你仅可以制定一个类库。

下面是 TOML 中构建目标设置的概览，针对每个字段都有详细介绍。

```toml
[lib]
name = "foo"           # 构建目标的名称
path = "src/lib.rs"    # 构建目标的源文件相对路径
test = true            # 是否默认进行测试
doctest = true         # 文档示例是否默认进行测试
bench = true           # 是否默认进行性能测试
doc = true             # 是否默认带有文档
plugin = false         # 是否用作编译器插件(已弃用)
proc-macro = false     # proc-macro 类库要设置为 `true`
harness = true         # 是否使用 libtest harness
edition = "2015"       # 构建目标的版本
crate-type = ["lib"]   # 要生成的 crate 类型
required-features = [] # 构建此目标需要使用的特性 (类库不适用)
```

### `name` 字段

`name` 字段指定构建目标的名称，它对应着自动生成的 artifact 的文件名。
对类库来说，这个字段是它的 crate 名称，其他依赖会使用这个名称来引用它。

对于 `[lib]` 和默认的二进制程序(`src/main.rs`)来说，这个字段默认就是包的名称，
其中的破折号会被替换成下划线。对于其他 [auto discovered](#target-auto-discovery) 构建目标，
它默认是目录或文件的名称。

这个字段对所有构建目标是必填的，除了 `[lib]` 。

#### `path` 字段

`path` 字段指定 crate 的源代码相对于 `Cargo.toml` 的相对路径。

若此字段未指定，则默认会使用 [inferred path](#target-auto-discovery) 基于构建目标的名称自动填充。

#### `test` 字段

`test` 字段指明构建目标是否默认使用 [`cargo test`] 命令进行测试。
对类库、可执行程序和测试来说，默认为 `true` 。

> **注意**: 默认会使用 [`cargo test`] 构建例子来保证它们可以通过编译，
> 但是默认情况下，它们没有 *测试* 过运行时是否符合预期。
> 为例子设置 `test = true` 也会将它构建为测试并且运行例子中所有带 [`#[test]`][test-attribute] 标记的方法。

#### `doctest` 字段

`doctest` 字段指明 [文档例子(documentation examples)] 是否默认使用 [`cargo test`] 测试。
这个字段只与类库有关，它对其他部分的配置没有任何作用。
对类库来说，此字段默认为 `true` 。

#### `bench` 字段

`bench` 字段指明构建目标是否默认使用 [`cargo bench`] 进行性能测试。
对类库、二进制程序和性能测试程序来说，此字段默认为 `true` 。

#### `doc` 字段

`doc` 字段指明构建目标是否包括在 [`cargo doc`] 默认生成的文档中。
对类库和二进制程序来说，此字段默认为 `true` 。

> **注意**: 若二进制程序的名称与类库的名称相同，二进制程序则会被跳过。

#### `plugin` 字段

此字段用于 `rustc` 的插件，现在已被弃用。

#### `proc-macro` 字段

`proc-macro` 字段指明类库是 [procedural macro]([reference][proc-macro-reference]) 。
此字段仅适用于 `[lib]` 构建目标。

#### `harness` 字段

`harness` 字段指明 [`--test` 参数] 会传递给 `rustc` 令其自动包含 libtest 类库。
libtest 类库用来收集并运行标有 [`#[test]` 属性][test-attribute] 的测试或标有 `#[bench]` 属性的性能测试。
对所有构建目标来说，此字段默认为 `true` 。

如果此字段设置为 `false` ，则你需要负责定义 `main()` 函数来运行测试和性能测试。

无论 harness 字段是否启用，测试都会启用 [`cfg(test)` 条件表达式][cfg-test] 字段。

#### `edition` 字段

`edition` 字段定义了构建目标将会使用的 [Rust edition] 。
若未指定，默认会使用 `[package]` 的 [`edition` 字段][package-edition] 。
此字段通常不需设置，它只用于类似大型包增量迁移到一个新版本这样的高级场景。

#### `crate-type` 字段

`crate-type` 字段定义了构建目标将生成的 [crate types] 。
它是字符串数组，用于在一个构建目标中指定多个 crate 类型 。
此字段仅用于类库和例子。对于二进制程序，测试和性能测试，此字段总是为 "bin" crate 类型。
默认值为:

构建目标 | Crate Type
-------|-----------
普通类库 | `"lib"`
Proc-macro 类库 | `"proc-macro"`
例子 | `"bin"`

此字段可用的值有 `bin`, `lib`, `rlib`, `dylib`, `cdylib`, `staticlib` 和 `proc-macro` 。
关于各种 crate 类型的更多信息在 [Rust Reference Manual][crate types] 。

#### `required-features` 字段

`required-features` 字段指定了构建目标完成构建所需要的 [features] 。
如果要求的特性都没有启用，则此构建目标将被跳过。
此字段仅与 `[[bin]]`, `[[bench]]`, `[[test]]` 和 `[[example]]` 有关。
此字段对 `[lib]` 没有影响。

```toml
[features]
# ...
postgres = []
sqlite = []
tools = []

[[bin]]
name = "my-pg-tool"
required-features = ["postgres", "tools"]
```


### 构建目标自动探测

Cargo 默认会根据文件系统上的 [文件结构][package layout] 推断构建目标。
可以在构建目标的配置表中添加不同于标准目录结构的额外构建目标，比如使用
`[lib]`, `[[bin]]`, `[[test]]`, `[[bench]]` 或 `[[example]]` 标记。

可以禁用目标探测，这样只有手动配置的构建目标会进行构建。
要禁用对应构建目标类型的自动探测，需在 `[package]` 部分将
`autobins`, `autoexamples`, `autotests` 或 `autobenches` 设置为 `false` 。

```toml
[package]
# ...
autobins = false
autoexamples = false
autotests = false
autobenches = false
```

禁用自动探测应该仅用于一些特殊场景。
比如，如果你想在类库中给一个 *模块* 命名为 `bin` ，
这样是不行的，因为 Cargo 通常会将 `bin` 目录下的文件编译为可执行文件。
这个例子的文件结构如下:

```text
├── Cargo.toml
└── src
    ├── lib.rs
    └── bin
        └── mod.rs
```

在 `Cargo.toml` 中声明 `autobins = false` 禁用自动探测来防止 Cargo 将 `src/bin/mod.rs` 推断为可执行文件:

```toml
[package]
# …
autobins = false
```

> **注意**: 对于 2015 版本的包，如果 `Cargo.toml` 中手动设置了构建目标，
> 则自动推断字段默认会设置为 `false` 。
> 自从 2018 版本，此字段默认设置为 `true` 。


[Build cache]: ../guide/build-cache.md
[Rust Edition]: ../../edition-guide/index.html
[`--test` flag]: ../../rustc/command-line-arguments.html#option-test
[`cargo bench`]: ../commands/cargo-bench.md
[`cargo build`]: ../commands/cargo-build.md
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo install`]: ../commands/cargo-install.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo test`]: ../commands/cargo-test.md
[cfg-test]: ../../reference/conditional-compilation.html#test
[crate types]: ../../reference/linkage.html
[crates.io]: https://crates.io/
[customized]: #configuring-a-target
[dependencies]: specifying-dependencies.md
[dev-dependencies]: specifying-dependencies.md#development-dependencies
[documentation examples]: ../../rustdoc/documentation-tests.html
[features]: features.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[package layout]: ../guide/project-layout.md
[package-edition]: manifest.md#the-edition-field
[proc-macro-reference]: ../../reference/procedural-macros.html
[procedural macro]: ../../book/ch19-06-macros.html
[test-attribute]: ../../reference/attributes/testing.html#the-test-attribute
