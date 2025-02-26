{==+==}
## Specifying Dependencies
{==+==}
## 指定依赖项
{==+==}

{==+==}
Your crates can depend on other libraries from [crates.io] or other
registries, `git` repositories, or subdirectories on your local file system.
You can also temporarily override the location of a dependency — for example,
to be able to test out a bug fix in the dependency that you are working on
locally. You can have different dependencies for different platforms, and
dependencies that are only used during development. Let's take a look at how
to do each of these.
{==+==}
你的crate可以依赖另一个库，这个库可以来自 [crates.io] 或者其他 registry、git 仓库或者你文件系统中的一个子目录。你也可以暂时覆盖某个依赖的路径，这样就可以在本地检查和修复某个依赖项的bug。你可以对不同目标平台指定不同的依赖，也可以指定仅仅在开发阶段使用某个依赖。让我们来看看具体应该怎么做。
{==+==}

{==+==}
### Specifying dependencies from crates.io
{==+==}
### 指定一个 crate.io 上的依赖
{==+==}

{==+==}
Cargo is configured to look for dependencies on [crates.io] by default. Only
the name and a version string are required in this case. In [the cargo
guide](../guide/index.md), we specified a dependency on the `time` crate:
{==+==}
默认情况下，Cargo被设置为从 [crates.io] 中查找依赖。之前在 [the cargo guide](../guide/index.md) 里我们指定了 `time` crate 作为依赖: 
{==+==}

{==+==}
```toml
[dependencies]
time = "0.1.12"
```
{==+==}
```toml
[dependencies]
time = "0.1.12"
```
{==+==}

{==+==}
The string `"0.1.12"` is a version requirement. Although it looks like a
specific *version* of the `time` crate, it actually specifies a *range* of
versions and allows [SemVer] compatible updates. An update is allowed if the new
version number does not modify the left-most non-zero digit in the major, minor,
patch grouping. In this case, if we ran `cargo update -p time`, cargo should
update us to version `0.1.13` if it is the latest `0.1.z` release, but would not
update us to `0.2.0`. If instead we had specified the version string as `1.0`,
cargo should update to `1.1` if it is the latest `1.y` release, but not `2.0`.
The version `0.0.x` is not considered compatible with any other version.
{==+==}
`"0.1.12"` 这个字符串表示请求的版本。虽然它看起来表示一个具体的 `time` 版本，但实际上它表示一个范围，允许与 [SemVer] 兼容的更新。具体来说，不改变 `主版本号.次版本号.修订号` 中**第一个非零数字**的所有更新都是允许的。如果我们运行 `cargo update -p time` ，Cargo 会把 `time` 更新到 `0.1.13` (假设这是形如 `0.1.z` 的最新版本)，但不会更新到 `0.2.0`。 如果我们指定版本为 `1.0`，那么Cargo 会将其更新到 `1.1` (假设这是最新的 `1.y` 版本)，但绝不会更新到 `2.0`。`0.0.x` 这个版本和任何其他版本都不兼容。
{==+==}

{==+==}
[SemVer]: https://semver.org
{==+==}
[SemVer]: https://semver.org
{==+==}

{==+==}
Here are some more examples of version requirements and the versions that would
be allowed with them:
{==+==}
下面是一些书写版本的例子以及符合要求的版本范围:
{==+==}

{==+==}
```notrust
1.2.3  :=  >=1.2.3, <2.0.0
1.2    :=  >=1.2.0, <2.0.0
1      :=  >=1.0.0, <2.0.0
0.2.3  :=  >=0.2.3, <0.3.0
0.2    :=  >=0.2.0, <0.3.0
0.0.3  :=  >=0.0.3, <0.0.4
0.0    :=  >=0.0.0, <0.1.0
0      :=  >=0.0.0, <1.0.0
```
{==+==}
```notrust
1.2.3  :=  >=1.2.3, <2.0.0
1.2    :=  >=1.2.0, <2.0.0
1      :=  >=1.0.0, <2.0.0
0.2.3  :=  >=0.2.3, <0.3.0
0.2    :=  >=0.2.0, <0.3.0
0.0.3  :=  >=0.0.3, <0.0.4
0.0    :=  >=0.0.0, <0.1.0
0      :=  >=0.0.0, <1.0.0
```
{==+==}

{==+==}
This compatibility convention is different from SemVer in the way it treats
versions before 1.0.0. While SemVer says there is no compatibility before
1.0.0, Cargo considers `0.x.y` to be compatible with `0.x.z`, where `y ≥ z`
and `x > 0`.
{==+==}
这种兼容规则与SemVer有些不同，SemVer认为在1.0.0之前没有任何兼容性，而Cargo认为 `0.x.y` 是与 `0.x.z`兼容的 (只要 `y ≥ z` 且 `x > 0`)。
{==+==}

{==+==}
It is possible to further tweak the logic for selecting compatible versions
using special operators, though it shouldn't be necessary most of the time.
{==+==}
可以通过一些特殊操作符来调整选择兼容版本的逻辑，但在大多数时候都不需要。
{==+==}

{==+==}
### Caret requirements
{==+==}
### 使用 `^` 符号
{==+==}

{==+==}
**Caret requirements** are an alternative syntax for the default strategy,
`^1.2.3` is exactly equivalent to `1.2.3`.
{==+==}
 `^` 符号是默认策略的另一种写法，`^1.2.3` 和 `1.2.3` 的意义是一样的。
{==+==}

{==+==}
### Tilde requirements
{==+==}
### 使用 `~` 符号
{==+==}

{==+==}
**Tilde requirements** specify a minimal version with some ability to update.
If you specify a major, minor, and patch version or only a major and minor
version, only patch-level changes are allowed. If you only specify a major
version, then minor- and patch-level changes are allowed.
{==+==}
`~` 符号表示在某个最小版本的基础上可以做一定的更新。如果你指定了一个形如 `主.次.修` 或者 `主.次` 的版本，那么修订号是可以变的。如果是只指定了主版本号，那么次版本号和修订号都是都可以改变。
{==+==}

{==+==}
`~1.2.3` is an example of a tilde requirement.
{==+==}
`~1.2.3` 的例子:
{==+==}

{==+==}
```notrust
~1.2.3  := >=1.2.3, <1.3.0
~1.2    := >=1.2.0, <1.3.0
~1      := >=1.0.0, <2.0.0
```
{==+==}
```notrust
~1.2.3  := >=1.2.3, <1.3.0
~1.2    := >=1.2.0, <1.3.0
~1      := >=1.0.0, <2.0.0
```
{==+==}

{==+==}
### Wildcard requirements
{==+==}
### 通配符 `*`
{==+==}

{==+==}
**Wildcard requirements** allow for any version where the wildcard is
positioned.
{==+==}
通配符出现的地方可以表示任意的数字。
{==+==}

{==+==}
`*`, `1.*` and `1.2.*` are examples of wildcard requirements.
{==+==}
比如 `*`、`1.*`、 `1.2.*`。
{==+==}

{==+==}
```notrust
*     := >=0.0.0
1.*   := >=1.0.0, <2.0.0
1.2.* := >=1.2.0, <1.3.0
```
{==+==}
```notrust
*     := >=0.0.0
1.*   := >=1.0.0, <2.0.0
1.2.* := >=1.2.0, <1.3.0
```
{==+==}

{==+==}
> **Note**: [crates.io] does not allow bare `*` versions.
{==+==}
> **注意**: [crates.io] 不允许只写一个单独的 `*`。
{==+==}

{==+==}
### Comparison requirements
{==+==}
### 使用比较符号
{==+==}

{==+==}
**Comparison requirements** allow manually specifying a version range or an
exact version to depend on.
{==+==}
使用比较符号可以手动指定一个版本范围或一个具体的版本。
{==+==}

{==+==}
Here are some examples of comparison requirements:
{==+==}
下面是一些例子:
{==+==}

{==+==}
```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```
{==+==}
```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```
{==+==}

{==+==}
### Multiple requirements
{==+==}
### 多重约束
{==+==}

{==+==}
As shown in the examples above, multiple version requirements can be
separated with a comma, e.g., `>= 1.2, < 1.5`.
{==+==}
就像上面的例子中一样，多个版本约束可以用逗号分隔，如 `>= 1.2, < 1.5`。
{==+==}

{==+==}
### Specifying dependencies from other registries
{==+==}
### 指定来自其他registry的依赖
{==+==}

{==+==}
To specify a dependency from a registry other than [crates.io], first the
registry must be configured in a `.cargo/config.toml` file. See the [registries
documentation] for more information. In the dependency, set the `registry` key
to the name of the registry to use.
{==+==}
为了指定一个非 [crates.io] 的依赖，首先这个registry必须在 `.cargo/config.toml` 中设置，参见 [registries documentation]。在依赖项中，将 `registry` 字段设置为你想用的registry。
{==+==}

{==+==}
```toml
[dependencies]
some-crate = { version = "1.0", registry = "my-registry" }
```
{==+==}
```toml
[dependencies]
some-crate = { version = "1.0", registry = "my-registry" }
```
{==+==}

{==+==}
> **Note**: [crates.io] does not allow packages to be published with
> dependencies on other registries.
{==+==}
> **注意**: [crates.io] 不允许发布带着其他registry依赖的包。
{==+==}

{==+==}
[registries documentation]: registries.md
{==+==}
[registries documentation]: registries.md
{==+==}

{==+==}
### Specifying dependencies from `git` repositories
{==+==}
### 指定来自 `git` 的依赖
{==+==}

{==+==}
To depend on a library located in a `git` repository, the minimum information
you need to specify is the location of the repository with the `git` key:
{==+==}
要指定位于某个 `git` 仓库的依赖库，需要的最小信息是在 `git` 字段中写出该仓库的位置:
{==+==}

{==+==}
```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```
{==+==}

{==+==}

{==+==}
Cargo will fetch the `git` repository at this location then look for a
`Cargo.toml` for the requested crate anywhere inside the `git` repository
(not necessarily at the root - for example, specifying a member crate name
of a workspace and setting `git` to the repository containing the workspace).
{==+==}
Cargo 会 fetch 这个 `git` 仓库，并在仓库中查找与所需crate对应的 `Cargo.toml` 文件(这个文件不要求必须在地址根目录下，比如说指定的是某个workspace的一个成员crate，这时 `git` 字段只需写workspace的地址就可以)。
{==+==}

{==+==}
Since we haven’t specified any other information, Cargo assumes that
we intend to use the latest commit on the main branch to build our package.
You can combine the `git` key with the `rev`, `tag`, or `branch` keys to
specify something else. Here's an example of specifying that you want to use
the latest commit on a branch named `next`:
{==+==}
因为我们没有指定其他信息，Cargo假设我们想用的是该仓库主分支的最新commit。你也可以在 `git`字段后加上 `rev`、`tag` 或 `branch` 这些字段，来指定你想要的commit。下面是一个指定 `next` 分支上的最新commit的例子:
{==+==}

{==+==}
```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", branch = "next" }
```
{==+==}
```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", branch = "next" }
```
{==+==}

{==+==}
Anything that is not a branch or tag falls under `rev`. This can be a commit
hash like `rev = "4c59b707"`, or a named reference exposed by the remote
repository such as `rev = "refs/pull/493/head"`. What references are available
varies by where the repo is hosted; GitHub in particular exposes a reference to
the most recent commit of every pull request as shown, but other git hosts often
provide something equivalent, possibly under a different naming scheme.
{==+==}
如果想要指定的依赖版本不是某个branch的头，也没有标记tag，那么就需要用 `rev` 来指定。`rev` 字段可以是像 `rev = "4c59b707"` 这样的 commit hash 值，也可以是像 `rev = "refs/pull/493/head"` 这样的命名引用。哪些引用是合法的取决于这个git仓库具体的管理机构。Github 暴露出了每个pull requeset 最新commit的引用，其他git机构一般也提供了类似的，只是可能有不同的命名规则。
{==+==}

{==+==}
Once a `git` dependency has been added, Cargo will lock that dependency to the
latest commit at the time. New commits will not be pulled down automatically
once the lock is in place. However, they can be pulled down manually with
`cargo update`.
{==+==}
一旦某个 `git` 依赖被添加，Cargo会立刻锁定该依赖到当前最新的commit。之后即使有新的commit被提交，Cargo也不会再自动拉取。不过，你可以通过 `cargo update` 命令来手动拉取。
{==+==}

{==+==}
See [Git Authentication] for help with git authentication for private repos.
{==+==}
对于私有仓库的身份验证，参考 [Git 身份验证][Git Authentication]。
{==+==}

{==+==}
> **Note**: [crates.io] does not allow packages to be published with `git`
> dependencies (`git` [dev-dependencies] are ignored). See the [Multiple
> locations](#multiple-locations) section for a fallback alternative.
{==+==}
> **注意**: [crates.io] 不允许发布带 `git` 依赖的包( `git` [dev-dependencies] 除外)。对于备用选择，参考[Multiple locations](#multiple-locations)。
{==+==}

{==+==}
[Git Authentication]: ../appendix/git-authentication.md
{==+==}
[Git Authentication]: ../appendix/git-authentication.md
{==+==}

{==+==}
### Specifying path dependencies
{==+==}
### 指定路径依赖
{==+==}

{==+==}
Over time, our `hello_world` package from [the guide](../guide/index.md) has
grown significantly in size! It’s gotten to the point that we probably want to
split out a separate crate for others to use. To do this Cargo supports **path
dependencies** which are typically sub-crates that live within one repository.
Let’s start off by making a new crate inside of our `hello_world` package:
{==+==}
经过一段时间，我们的 `hello_world` 包 (来自[the guide](../guide/index.md)) 已经变得很大了。我们可能想要把其中一部分拆出来使用。Cargo为此提供了指定路径依赖的功能，最经典的情况是一个git仓库中有很多个子crate。首先我们在 `hello_world` 项目里面创建一个新的crate:
{==+==}

{==+==}
```console
# inside of hello_world/
$ cargo new hello_utils
```
{==+==}
```console
# hello_world/ 目录中
$ cargo new hello_utils
```
{==+==}

{==+==}
This will create a new folder `hello_utils` inside of which a `Cargo.toml` and
`src` folder are ready to be configured. In order to tell Cargo about this, open
up `hello_world/Cargo.toml` and add `hello_utils` to your dependencies:
{==+==}
这将创建一个新的 `hello_utils` 文件夹，其中已经设置好了 `Cargo.toml` 和 `src` 文件夹。为了让Cargo知道新crate的存在，需要在 `hello_world/Cargo.toml` 中将 `hello_utils` 添加为依赖:
{==+==}

{==+==}
```toml
[dependencies]
hello_utils = { path = "hello_utils" }
```
{==+==}
```toml
[dependencies]
hello_utils = { path = "hello_utils" }
```
{==+==}

{==+==}
This tells Cargo that we depend on a crate called `hello_utils` which is found
in the `hello_utils` folder (relative to the `Cargo.toml` it’s written in).
{==+==}
这会告诉Cargo我们依赖了一个名为 `hello_utils` 的crate，其在 `hello_utils` 文件夹中(相对于刚刚被写入的 `Cargo.toml` 的位置)。
{==+==}

{==+==}
And that’s it! The next `cargo build` will automatically build `hello_utils` and
all of its own dependencies, and others can also start using the crate as well.
However, crates that use dependencies specified with only a path are not
permitted on [crates.io]. If we wanted to publish our `hello_world` crate, we
would need to publish a version of `hello_utils` to [crates.io]
and specify its version in the dependencies line as well:
{==+==}
搞定！下次执行 `cargo build` 会自动构建 `hello_utils` 和它的所有依赖，其他包也可以使用这个crate。但是，[crates.io] 不允许发布仅以路径指定某个依赖的包。如果我们想发布 `hello_world`，就必须把 `hello_utils` 发布到 [crates.io]，然后在 `hello_word`的 toml 中同时指定依赖版本:
{==+==}

{==+==}
```toml
[dependencies]
hello_utils = { path = "hello_utils", version = "0.1.0" }
```
{==+==}
```toml
[dependencies]
hello_utils = { path = "hello_utils", version = "0.1.0" }
```
{==+==}

{==+==}
> **Note**: [crates.io] does not allow packages to be published with `path`
> dependencies (`path` [dev-dependencies] are ignored). See the [Multiple
> locations](#multiple-locations) section for a fallback alternative.
{==+==}
> **注意**: [crates.io] 不允许发布带 `path` 依赖的包( `path` [dev-dependencies] 除外)。对于备用选择，参考[Multiple locations](#multiple-locations)。
{==+==}

{==+==}
### Multiple locations
{==+==}
### 多依赖位置
{==+==}

{==+==}
It is possible to specify both a registry version and a `git` or `path`
location. The `git` or `path` dependency will be used locally (in which case
the `version` is checked against the local copy), and when published to a
registry like [crates.io], it will use the registry version. Other
combinations are not allowed. Examples:
{==+==}
可以对一个依赖同时指定一个registry和一个 `git` 或 `path` 位置。`git` 或 `path` 依赖项会被用于本地(此时 `version` 会与本地的拷贝进行对比)，而当发布到一个registry (比如[crates.io]) 时，会使用 registry 上的版本。其他组合是不允许的。例如:
{==+==}

{==+==}
```toml
[dependencies]
# Uses `my-bitflags` when used locally, and uses
# version 1.0 from crates.io when published.
bitflags = { path = "my-bitflags", version = "1.0" }

# Uses the given git repo when used locally, and uses
# version 1.0 from crates.io when published.
smallvec = { git = "https://github.com/servo/rust-smallvec.git", version = "1.0" }

# N.B. that if a version doesn't match, Cargo will fail to compile!
```
{==+==}
```toml
[dependencies]
# 在本地时会使用 `my-bitflags`
# 而上传后会使用 crates.io 中的1.0 版本
bitflags = { path = "my-bitflags", version = "1.0" }

# 在本地时会使用给定的 git 仓库
# 而上传后会使用 crates.io 中的1.0 版本
smallvec = { git = "https://github.com/servo/rust-smallvec.git", version = "1.0" }

# 如果version没有匹配，Cargo 会编译失败！
```
{==+==}

{==+==}
One example where this can be useful is when you have split up a library into
multiple packages within the same workspace. You can then use `path`
dependencies to point to the local packages within the workspace to use the
local version during development, and then use the [crates.io] version once it
is published. This is similar to specifying an
[override](overriding-dependencies.md), but only applies to this one
dependency declaration.
{==+==}
一个使用的场景是当你把一个库拆分成一个工作空间(workspace)中的多个包(package)，你可以用 `path` 在开发阶段指定工作空间内的本地包做为依赖，而当你将其发布后，就可以使用 [crates.io] 上的版本了。
{==+==}

{==+==}
### Platform specific dependencies
{==+==}
### 平台特定依赖
{==+==}

{==+==}
Platform-specific dependencies take the same format, but are listed under a
`target` section. Normally Rust-like [`#[cfg]`
syntax](../../reference/conditional-compilation.html) will be used to define
these sections:
{==+==}
平台特定依赖的书写格式没什么变化，但是要列在 `target` 部分。通常使用与rust代码类似的 [`#[cfg]` 语法](../../reference/conditional-compilation.html):
{==+==}

{==+==}
```toml
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"

[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"

[target.'cfg(target_arch = "x86")'.dependencies]
native-i686 = { path = "native/i686" }

[target.'cfg(target_arch = "x86_64")'.dependencies]
native-x86_64 = { path = "native/x86_64" }
```
{==+==}
```toml
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"

[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"

[target.'cfg(target_arch = "x86")'.dependencies]
native-i686 = { path = "native/i686" }

[target.'cfg(target_arch = "x86_64")'.dependencies]
native-x86_64 = { path = "native/x86_64" }
```
{==+==}

{==+==}
Like with Rust, the syntax here supports the `not`, `any`, and `all` operators
to combine various cfg name/value pairs.
{==+==}
如同rust代码一样，语法支持 `not`、`any` 和 `all` 操作符，从而组合不同的cfg键值对。
{==+==}

{==+==}
If you want to know which cfg targets are available on your platform, run
`rustc --print=cfg` from the command line. If you want to know which `cfg`
targets are available for another platform, such as 64-bit Windows,
run `rustc --print=cfg --target=x86_64-pc-windows-msvc`.
{==+==}
如果你想知道自己的平台支持哪些cfg目标，可以运行 `rustc --print=cfg` 来获取。如果你想知道其他平台可用的cfg目标，可以使用 `rustc --print=cfg --target=x86_64-pc-windows-msvc`。
{==+==}

{==+==}
Unlike in your Rust source code, you cannot use
`[target.'cfg(feature = "fancy-feature")'.dependencies]` to add dependencies
based on optional features. Use [the `[features]` section](features.md)
instead:
{==+==}
与rust代码不同，你不能使用 `[target.'cfg(feature = "fancy-feature")'.dependencies]` 来根据feature指定依赖，而是应该使用 [features](features.md)。
{==+==}

{==+==}
```toml
[dependencies]
foo = { version = "1.0", optional = true }
bar = { version = "1.0", optional = true }

[features]
fancy-feature = ["foo", "bar"]
```
{==+==}
```toml
[dependencies]
foo = { version = "1.0", optional = true }
bar = { version = "1.0", optional = true }

[features]
fancy-feature = ["foo", "bar"]
```
{==+==}

{==+==}
The same applies to `cfg(debug_assertions)`, `cfg(test)` and `cfg(proc_macro)`.
These values will not work as expected and will always have the default value
returned by `rustc --print=cfg`.
There is currently no way to add dependencies based on these configuration values.
{==+==}
这同样也适用于 `cfg(debug_assertions)` ， `cfg(test)` 和 `cfg(proc_macro)`。这些值不会按你的预期生效，写不写结果都和默认情况(`rustc --print=cfg`返回的结果)相同。目前没有办法根据这些设置值来添加依赖。
{==+==}

{==+==}
In addition to `#[cfg]` syntax, Cargo also supports listing out the full target
the dependencies would apply to:
{==+==}
除了 `#[cfg]` 标记，Cargo也支持直接写出target的全名来指定依赖:
{==+==}

{==+==}
```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```
{==+==}
```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```
{==+==}

{==+==}
#### Custom target specifications
{==+==}

#### 为自定义目标指定依赖
{==+==}

{==+==}
If you’re using a custom target specification (such as `--target
foo/bar.json`), use the base filename without the `.json` extension:
{==+==}
如果你使用一个自定义构建目标 (比如 `--target foo/bar.json` )，使用不含 `.json` 后缀的文件名作为名字:
{==+==}

{==+==}
```toml
[target.bar.dependencies]
winhttp = "0.4.0"

[target.my-special-i686-platform.dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
```
{==+==}
```toml
[target.bar.dependencies]
winhttp = "0.4.0"

[target.my-special-i686-platform.dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
```
{==+==}

{==+==}
> **Note**: Custom target specifications are not usable on the stable channel.
{==+==}
> **注意**: 自定义目标依赖在稳定版是不支持的。
{==+==}

{==+==}
### Development dependencies
{==+==}
### 开发依赖
{==+==}

{==+==}
You can add a `[dev-dependencies]` section to your `Cargo.toml` whose format
is equivalent to `[dependencies]`:
{==+==}
你可以在 `Cargo.toml` 中添加 `[dev-dependencies]` ，格式与 `[dependencies]` 一致:
{==+==}

{==+==}
```toml
[dev-dependencies]
tempdir = "0.3"
```
{==+==}
```toml
[dev-dependencies]
tempdir = "0.3"
```
{==+==}

{==+==}
Dev-dependencies are not used when compiling
a package for building, but are used for compiling tests, examples, and
benchmarks.
{==+==}
开发依赖不会被用在构建包时，而是用在编译 tests、emamples 和 benchmarks。
{==+==}

{==+==}
These dependencies are *not* propagated to other packages which depend on this
package.
{==+==}
这些依赖不会传播给依赖本包的那些包。
{==+==}

{==+==}
You can also have target-specific development dependencies by using
`dev-dependencies` in the target section header instead of `dependencies`. For
example:
{==+==}
你也可以为特定目标指定开发依赖，只需要把 `dependencies` 换成 `dev-denpendencies` 。
{==+==}

{==+==}
```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```
{==+==}
```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```
{==+==}

{==+==}
> **Note**: When a package is published, only dev-dependencies that specify a
> `version` will be included in the published crate. For most use cases,
> dev-dependencies are not needed when published, though some users (like OS
> packagers) may want to run tests within a crate, so providing a `version` if
> possible can still be beneficial.
{==+==}
> **注意**: 当发布一个包时，只有那些以 `version` 指定的开发依赖才会被包含在发布的包中。大部分情况下，包发布后就不需要开发依赖了，但是也有一些使用者(比如操作系统打包商)希望在crate中运行一些测试，所以给开发依赖提供一个 `version` 也是有好处的。
{==+==}

{==+==}
### Build dependencies
{==+==}
### 构建依赖
{==+==}

{==+==}
You can depend on other Cargo-based crates for use in your build scripts.
Dependencies are declared through the `build-dependencies` section of the
manifest:
{==+==}
你可以在构建脚本中依赖一些使用Cargo的crate。在清单中指定 `build-dependencies` 来声明依赖。
{==+==}

{==+==}
```toml
[build-dependencies]
cc = "1.0.3"
```
{==+==}
```toml
[build-dependencies]
cc = "1.0.3"
```
{==+==}


{==+==}
You can also have target-specific build dependencies by using
`build-dependencies` in the target section header instead of `dependencies`. For
example:
{==+==}
你也可以为特定目标指定构建依赖，只需将 `dependencies` 替换为 `build-dependencies` 。例如:
{==+==}

{==+==}
```toml
[target.'cfg(unix)'.build-dependencies]
cc = "1.0.3"
```
{==+==}
```toml
[target.'cfg(unix)'.build-dependencies]
cc = "1.0.3"
```
{==+==}

{==+==}
In this case, the dependency will only be built when the host platform matches the
specified target.
{==+==}
这样，只有当宿主平台满足指定目标要求时，相关依赖才会被构建。
{==+==}

{==+==}
The build script **does not** have access to the dependencies listed
in the `dependencies` or `dev-dependencies` section. Build
dependencies will likewise not be available to the package itself
unless listed under the `dependencies` section as well. A package
itself and its build script are built separately, so their
dependencies need not coincide. Cargo is kept simpler and cleaner by
using independent dependencies for independent purposes.
{==+==}
构建脚本无法使用 `dependencies` 和 `dev-dependencies` 中列出的依赖。同样的，包也无法使用构建依赖(`build-dependencies`)，除非该依赖也被放在了 `dependencies` 中。一个包和它的构建脚本是分开各自编译的，因此它们的依赖项也不需要相同。Cargo通过对不同的目的使用各自的依赖来保持简单和简洁。
{==+==}

{==+==}
### Choosing features
{==+==}
### 选择特性
{==+==}

{==+==}
If a package you depend on offers conditional features, you can
specify which to use:
{==+==}
如果你依赖的包提供了可选的特性(feature)，你可以选择使用哪些:
{==+==}

{==+==}
```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # do not include the default features, and optionally
                         # cherry-pick individual features
features = ["secure-password", "civet"]
```
{==+==}
```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # 不使用默认特性
                         # 手动选择单独的特性
features = ["secure-password", "civet"]
```
{==+==}

{==+==}
More information about features can be found in the [features
chapter](features.md#dependency-features).
{==+==}
关于特性的更多信息，参考 [features chapter](features.md#dependency-features)。
{==+==}

{==+==}
### Renaming dependencies in `Cargo.toml`
{==+==}
### 在 `Cargo.toml` 中重命名依赖
{==+==}

{==+==}
When writing a `[dependencies]` section in `Cargo.toml` the key you write for a
dependency typically matches up to the name of the crate you import from in the
code. For some projects, though, you may wish to reference the crate with a
different name in the code regardless of how it's published on crates.io. For
example you may wish to:
{==+==}
当你在 `Cargo.toml` 中添加 `[dependencies]` 时，给依赖项的命名就是你导入到代码中的crate名。但是在某些项目中，你可能想要用另一个名称来引用这个crate，而不是它在crates.io上的名字。例如，你可能希望:
{==+==}

{==+==}
* Avoid the need to  `use foo as bar` in Rust source.
* Depend on multiple versions of a crate.
* Depend on crates with the same name from different registries.
{==+==}
* 避免在代码中使用 `use foo as bar`。
* 同时使用某个crate的多个版本。
* 依赖不同registry中同名的包。
{==+==}

{==+==}
To support this Cargo supports a `package` key in the `[dependencies]` section
of which package should be depended on:
{==+==}
Cargo 通过在 `[dependencies]` 加入`package` 字段来支持此功能。
{==+==}

{==+==}
```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project.git", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```
{==+==}
```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project.git", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```
{==+==}

{==+==}
In this example, three crates are now available in your Rust code:
{==+==}
在这个例子中，这三个crate都在代码中可用:
{==+==}

{==+==}
```rust,ignore
extern crate foo; // crates.io
extern crate bar; // git repository
extern crate baz; // registry `custom`
```
{==+==}
```rust,ignore
extern crate foo; // crates.io
extern crate bar; // git repository
extern crate baz; // registry `custom`
```
{==+==}

{==+==}
All three of these crates have the package name of `foo` in their own
`Cargo.toml`, so we're explicitly using the `package` key to inform Cargo that
we want the `foo` package even though we're calling it something else locally.
The `package` key, if not specified, defaults to the name of the dependency
being requested.
{==+==}
这三个crate都在自己的 `Cargo.toml` 里名为 `foo` ，所以我们用 `package` 来声明我们想要的是那个名为 `foo` 的包，即使我们会在本地用另一个别名来称呼它。如果没有指定 `package` ，则认为其名称与指定的依赖名称一致。
{==+==}

{==+==}
Note that if you have an optional dependency like:
{==+==}
注意如果你有一个可选依赖:
{==+==}

{==+==}
```toml
[dependencies]
bar = { version = "0.1", package = 'foo', optional = true }
```
{==+==}
```toml
[dependencies]
bar = { version = "0.1", package = 'foo', optional = true }
```
{==+==}

{==+==}
you're depending on the crate `foo` from crates.io, but your crate has a `bar`
feature instead of a `foo` feature. That is, names of features take after the
name of the dependency, not the package name, when renamed.
{==+==}
你依赖的是 crates.io 上的 `foo` 包，但是你的包指定的是 `bar` feature 而不是`foo` feature。也就是说如果你给一个依赖改了名，feature 使用的是依赖的名字，而不是包原本的名字。
{==+==}

{==+==}
Enabling transitive dependencies works similarly, for example we could add the
following to the above manifest:
{==+==}
传递依赖也是如此，比如我们可以把下面的条目加入清单文件:
{==+==}

{==+==}
```toml
[features]
log-debug = ['bar/log-debug'] # using 'foo/log-debug' would be an error!
```
{==+==}
```toml
[features]
log-debug = ['bar/log-debug'] # 用 'foo/log-debug' 会报错!
```
{==+==}

{==+==}
### Inheriting a dependency from a workspace
{==+==}
### 从工作空间中继承依赖
{==+==}

{==+==}
Dependencies can be inherited from a workspace by specifying the
dependency in the workspace's [`[workspace.dependencies]`][workspace.dependencies] table.
After that, add it to the `[dependencies]` table with `workspace = true`.
{==+==}
可以在工作空间的 [`[workspace.dependencies]`][workspace.dependencies] 字段指定依赖，然后在crate的 `[dependencies]` 中添加 `workspace = true` 就可以继承这个依赖。
{==+==}

{==+==}
Along with the `workspace` key, dependencies can also include these keys:
- [`optional`][optional]: Note that the`[workspace.dependencies]` table is not allowed to specify `optional`.
- [`features`][features]: These are additive with the features declared in the `[workspace.dependencies]`
{==+==}
除了 `workspace` 字段，还可以加入:
- [`optional`][optional]: 注意 `[workspace.dependencies]` 不允许使用 `optional`。
- [`features`][features]:  这是对于 `[workspace.dependencies]` 中声明的依赖的补充。
{==+==}

{==+==}
Other than `optional` and `features`, inherited dependencies cannot use any other
dependency key (such as `version` or `default-features`).
{==+==}
在 `optional` 和 `features` 之外，继承依赖不允许使用任何其他字段(比如 `version` 或 `default-features`)。
{==+==}

{==+==}
Dependencies in the `[dependencies]`, `[dev-dependencies]`, `[build-dependencies]`, and
`[target."...".dependencies]` sections support the ability to reference the
`[workspace.dependencies]` definition of dependencies.
{==+==}
`[dependencies]` 、 `[dev-dependencies]` 、 `[build-dependencies]` 和
 `[target."...".dependencies]` 都可以引用 `[workspace.dependencies]` 中定义的依赖。
{==+==}

{==+==}
```toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand = { workspace = true, optional = true }
```
{==+==}
```toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand = { workspace = true, optional = true }
```
{==+==}


{==+==}
[crates.io]: https://crates.io/
[dev-dependencies]: #development-dependencies
[workspace.dependencies]: workspaces.md#the-dependencies-table
[optional]: features.md#optional-dependencies
[features]: features.md
{==+==}
[crates.io]: https://crates.io/
[dev-dependencies]: #development-dependencies
[workspace.dependencies]: workspaces.md#the-dependencies-table
[optional]: features.md#optional-dependencies
[features]: features.md
{==+==}

{==+==}
<script>
(function() {
    var fragments = {
        "#overriding-dependencies": "overriding-dependencies.html",
        "#testing-a-bugfix": "overriding-dependencies.html#testing-a-bugfix",
        "#working-with-an-unpublished-minor-version": "overriding-dependencies.html#working-with-an-unpublished-minor-version",
        "#overriding-repository-url": "overriding-dependencies.html#overriding-repository-url",
        "#prepublishing-a-breaking-change": "overriding-dependencies.html#prepublishing-a-breaking-change",
        "#overriding-with-local-dependencies": "overriding-dependencies.html#paths-overrides",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>
{==+==}
<script>
(function() {
    var fragments = {
        "#overriding-dependencies": "overriding-dependencies.html",
        "#testing-a-bugfix": "overriding-dependencies.html#testing-a-bugfix",
        "#working-with-an-unpublished-minor-version": "overriding-dependencies.html#working-with-an-unpublished-minor-version",
        "#overriding-repository-url": "overriding-dependencies.html#overriding-repository-url",
        "#prepublishing-a-breaking-change": "overriding-dependencies.html#prepublishing-a-breaking-change",
        "#overriding-with-local-dependencies": "overriding-dependencies.html#paths-overrides",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>
{==+==}
