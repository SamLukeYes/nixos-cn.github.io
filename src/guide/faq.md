
# 渐进式了解

## Nix 生态构成简述

Nix 生态主要由 Nix 表达式语言，NixOS，Nix 包管理器，Nixpkgs，NixOps，Hydra 构成。**以下只是梗概，并不需要你完全理解或记住它。**

| 名称           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| Nix 表达式语言 | Nix 表达式语言是一种函数式编程语言，用于描述软件包的构建过程、依赖关系和环境变量等信息。它支持函数定义、递归、模式匹配等特性，还支持嵌套语法，可描述复杂的依赖关系和构建过程。Nix 还支持原子事务，使得所有的包和环境都是原子的，不会相互影响。Nix 表达式语言可用于定义软件包和环境，也可用于描述系统配置。它是一种强大、灵活、可重复和可扩展的语言，用于管理软件包和环境。 |
| NixOS          | NixOS 是一种基于 Nix 包管理器的 Linux 发行版，具有高度可配置性、可重复性和安全性。它采用声明性配置，使用配置文件明确描述系统状态，使得配置更易于维护。NixOS 适用于需要高度可定制性的用例，如服务器和开发环境。 |
| Nix 包管理器   | Nix 是跨平台的功能强大的包管理器，采用函数式编程思想描述依赖关系和多版本软件包管理，并提供一系列跨平台工具方便管理和部署。 |
| NixOps         | NixOps是基于NixOS的云部署管理工具，支持多云平台，提供简单的命令行接口，可创建、部署、升级和回滚NixOS。用户可通过编写Nix表达式自定义部署和配置，使其成为灵活、可扩展和可定制的工具。适合需要管理大型、复杂基础设施的组织。 |
| Hydra          | Hydra 是在 NixOS 中使用的 CI/CD 系统，它可以自动构建、测试和部署软件包，并决定是否发布和部署。Hydra 可以在不同环境下测试软件包，适用于开发、测试和部署任何类型的软件。 |
| Nixpkgs        | nixpkgs 是 Nix 软件包管理器的官方软件包集合，包含数以万计的软件包，并提供了构建、测试和部署工具，支持多平台和多架构，适用于开发、测试和部署各种类型的软件。 |

## 何谓 nix-channel

`nix-channel` 既是一个命令行工具也是“软件源”的别称，这个“软件”可以是系统（系统也是由众多软件构成的）也可以是软件包仓库（社区维护软件包集合）。为了尽量避免混淆，我们接下来统一使用“频道”代称“软件源”。

### 命令行

`nix-channel` 命令行工具的使用：

```bash
nix-channel {--add url [name] | --remove name | --list | --update [names…] | --rollback [generation] }
```

看起来，这个命令行工具提供的功能有：订阅频道，退订频道，列出频道，更新频道，还能回滚“生成”。

### 官方提供的频道

官方提供了[官方频道集合](https://channels.nixos.org/)。订阅了其中的频道以后，就会从相应的频道获取更新，就像你以前给其他发行版添加软件源一样。

请你尝试访问上面的 URL。正如我们所说，这是一个频道集合，根目录下的每一个子目录就代表一个频道，官方提供了若干个频道，这些频道功能和版本各异：

```
2022/8/4 22:51:25               0.1 kB         nixos-21.11
2022/8/4 22:59:10               0.1 kB         nixos-21.11-aarch64
2022/8/2 23:24:22               0.1 kB         nixos-21.11-small
2023/1/3 23:39:40               0.1 kB         nixos-22.05
2023/1/3 22:43:29               0.1 kB         nixos-22.05-aarch64
2023/1/2 04:11:11               0.1 kB         nixos-22.05-small
2023/3/16 05:19:03              0.1 kB         nixos-22.11
2023/3/16 22:18:18              0.1 kB         nixos-22.11-small
2023/3/16 14:55:20              0.1 kB         nixos-unstable
2023/3/16 21:27:06              0.1 kB         nixos-unstable-small
2023/1/2 21:05:38               0.1 kB         nixpkgs-22.05-darwin
2023/3/17 00:30:11              0.1 kB         nixpkgs-22.11-darwin
2023/3/16 04:06:35              0.1 kB         nixpkgs-unstable
```

### 项目结构

我们以官方频道 `nixpkgs-unstable` 为例，查看每个频道大致的构成。它们似乎都提供了 `nixexprs.tar.xz`，从文件名我们就了解到这是一个包含了若干 nix 文件的 tar 压缩档案（Tarball）。

::: tip Tarball

Taball 是 `tar` 文件格式的全称，不是 Nix 独有。它可以将多个文件打包在一起。如果你想在打包的时候压缩一下，还可以使用 gzip，bzip2 等软件压缩该档案。当你对 tar 文件启用压缩以后，后缀名会变更为 tar.gz, tar.bz2 等，具体取决于你使用的压缩软件。

:::

于是我们解压它，列出目录树：

```powershell
├─.github
├─doc
├─lib
├─maintainers
├─nixos
└─pkgs
    ├─applications
    │  ├─accessibility
    │  │  ├─contrast
    │  │  └─wvkbd
    │  ├─audio
    │  │  ├─a2jmidid
    │  │  ├─ario
    │  │  ├─csound
    │  │  │  ├─csound-manual
    │  │  │  └─csound-qt
    ......
```

::: note 仅供演示

以上目录树是笔者为了方便演示而精简过的部分，实际构成肯定有差别。

:::

上面的每个子目录中都有一个 `default.nix` 文件，这是导入该目录时默认被求值的文件。每个包名文件夹下面的 nix 文件，都是对于该软件包的描述。

::: tip

如果你还没有学习过 Nix 表达式语言，大抵是看不懂的，你可以前往 [Nix 语言概览](https://nixos-cn.org/guide/lang/) 进行初步学习。

:::

每个频道都应该提供一个名为 `nixexprs` 的 Tarball。其中 `default.nix`  既是根目录也是每一级目录的入口点。

## 系统频道与软件仓库频道

在上一节，我们列出了许多频道，现在请你找出它们的特点：

- 频道被分为了两类，一类 `nixos` 开头，一类 `nixpkgs` 开头。
- 频道名有版本信息，指定了频道所对应的系统版本。
- 频道名含有架构，特性等信息，比如 `aarch64`，`darwin` 之类字段的被用于描述架构。还有一个比较特殊的 `small`，可能这个频道的软件包比极少？

### 系统频道

`nixos` 打头的频道被称为系统频道。系统频道提供 NixOS Linux 发行版的软件包和配置文件，因此，通过添加 `nixos` 频道，可以让 Nix 包管理器获取 NixOS 发行版的所有软件包和系统配置文件。这些软件包和配置文件包括 NixOS 中所有的标准组件、系统服务和配置选项等。

通过 `nixos` 频道，可以方便地安装和管理 NixOS 中的软件包，也可以使用 NixOS 的系统配置文件来管理整个系统的配置。此外，`nixos` 频道也提供了 NixOS 发行版的更新，使得用户可以及时获得最新的 NixOS 软件包和系统更新。

::: tip 默认订阅的频道

NixOS 默认订阅了官方频道 `nixos`，即使你安装完系统什么都不做，它们也是存在的：

```bash
sudo nix-channel -list  # 列出频道
```

```bash
nixos https://nixos.org/channels/nixos22.11
```

这个频道提供了组成系统的一些驱动，设施等等。

::: warning

这里的 `nixos` 与 `https://nixos.org/channels/nixos22.11` 并不是并列关系，前者是频道名，后面是被订阅的 URL。

当你有两个及两个以上频道的时候，你就会理解我的意思：

```bash
nixos https://nixos.org/channels/nixos22.11
nixpkgs https://nixos.org/channels/nixpkgs-unstable
```

除非升级系统或更换镜像频道，否则不要动系统默认的 `nixos` 频道。
:::

#### 升级系统 #TODO

```bash

```

### 软件仓库频道

你还记得吗，Nix 包管理器是跨平台的也就是说可以在多种 Linux/Unix 发行版运行。软件仓库频道不仅仅可以为包含 Nix 包管理器的 NixOS 发行版所使用，还可以是安装了 Nix 包管理器的任何发行版。如果说系统频道是 NixOS 的眷属（只有 NixOS使用）的话，那 nixpkgs 频道就是运行在所有 Unix/Linux 上的 Nix 包管理器的爱恋。

你发现没有软件仓库频道只有 `nixpkgs-unstable` 和 `nixpkgs-22.05-darwin` 诸类的命名，在一般的 Linux 发行版上面我们一般订阅的就是 `nixpkgs-unstable`，为什么 macOS 上面运行的 Nix 包管理器需要单独做版本控制的软件包仓库呢呢？

由于 macOS 的不断更新和演变，不同版本的 macOS 可能会有不兼容的变化。因此，为了确保 Nixpkgs 中的软件包在不同版本的 macOS 上都能够正常工作，需要对 Darwin 版本进行版本控制。

相比之下，其他平台（如 Linux）则不需要进行版本控制，因为它们的接口和 ABI 稳定性较高，软件包之间的兼容性不像 macOS 那样容易受到影响。因此，Nixpkgs 中的 Linux 版本可以使用较新的版本，而不必过于担心兼容性问题。

#### 订阅 `nixpkgs-unstable` 频道

你好奇一个问题吗？为什么软件仓库频道竟然没有稳定版（Darwin 例外）？

因为 Nixpkgs 是一个非常活跃的软件包集合，它在持续不断地发展和更新中，这就是为什么它没有稳定版的原因。

Nixpkgs 的设计哲学是将软件包的构建与版本控制分离开来。这使得它可以轻松地添加、更新和删除软件包，同时避免了许多依赖性和版本问题。由于这种设计，Nixpkgs 不需要稳定版，因为每个用户都可以选择在任何时候使用任何特定版本的软件包。

此外，由于 Nixpkgs 的贡献者非常活跃，他们不断地提交新的软件包和更新现有软件包，因此对于一些应用场景来说，使用较新的软件包版本是更好的选择，以获取更好的功能、性能和安全性。

虽然没有稳定版，但Nixpkgs 每个发布的版本仍然被认为是有质量保证的，并且是经过测试和验证的。此外，可以通过指定版本号或通过锁定依赖来确保 Nixpkgs 中软件包的可重现性。

我们来演示一下？

```bash
nix-channel --add https://nixos.org/channels/nixpkgs-unstable  # 添加频道，不过我更喜欢称它为 “订阅”
nix-channel --update  # 更新频道
```

如此，你便订阅上了官方的 nixpkgs-unstable 软件源。

::: warning 推荐镜像频道

上文仅供教学。在下一节我们会指引大家订阅国内能正常访问的镜像频道，键入下面的命令以退订官方的 `nixpkgs-unstable` 频道：

```bash
nix-channel --remove nixpkgs-unstable
```

:::

::: tip 自定义频道名

默认情况下，频道名是截取自 URL 的最后一级：

```bash
nix-channel --add https://URL/nixpkgs-unstable
```

我们列出频道名：

```bash
nixpkgs-unstable https://URL/nixpkgs-unstable
```

如果我们需要手动命名频道，增加一个参数即可：

```bash
nix-channel --add https://URL/nixpkgs-unstable nixpkgs
```

:::

## 使用镜像频道

由于不可抗力的因素，大陆对于环大陆主机的访问显得异常艰难，所以我们需要使用国内的镜像频道来替代我们对官方频道（镜像频道通常由大学和企业公益性提供），下面列出了一些在中国可用的一些镜像频道：

- [清华大学](https://mirrors.tuna.tsinghua.edu.cn/help/nix-channels/)
- [中国科技大学](https://mirrors.ustc.edu.cn/help/nix-channels.html)
- [上海交通大学](https://sjtug.org/post/mirror-help/nix-channels/)

我们使用清华大学的镜像源作订阅演示：

```bash
nix-channel --add https://mirrors.tuna.tsinghua.edu.cn/nix-channels/nixpkgs-unstable nixpkgs  # 订阅频道，并分配 nixpkgs 命名
nix-channel --update  # 更新频道
```

::: note

特地修改频道名是因为许多表达式都会把 nixpkgs 而不是 nixpkgs-unstable 作输入。

:::

## 二进制构建缓存

在上面列出的镜像频道中，有不少频道都提供了二进制构建缓存。让我们来了解几种常见的软件分发方式，你就知道为什么我们需要二进制构建缓存了。

Gentoo 是最个性的 Linux 之一。它使用 ebuild 脚本（一种基于 bash 的脚本）来定义软件包的构建和安装过程。每个软件包都有一个相应的 ebuild 文件，其中包含有关软件包的元数据，如版本、依赖项和编译选项。Portage 根据这些 ebuild 文件自动下载源代码、解压缩并编译软件包，然后安装它们。Gentoo 社区维护了一个大型的 ebuild 仓库，其中包含数千个软件包的ebuild文件。通过使用 Portage，用户可以轻松地搜索、安装、升级和删除这些软件包。使用这个发行版的时候，你不是在构建就是在构建的路上。你可以用包管理控制编译选项，不同的编译选项会导致二进制包的性能，特性也不一样。

ArchLinux 社区是现代最闻名的科技教会之一，ArchLinux 发行版同样非常适合深入了解 Linux 的使用。ArchLinux 软件包是以 `.tar.gz` 或 `.tar.xz` 格式的二进制文件的形式存在的，属于二进制分发。需要注意的是 AUR 的存在，我们可以使用 `paru` 获取 AUR 中用户打包的软件，其中有部分软件是从源码开始构建的，附带的构建脚本会让二进制构建在本地生成。

我们将两种方式分别称为“源码分发”与“二进制分发”。

**源码分发是指将软件的源代码打包并分发给用户**。源代码是程序员编写的文本文件，它包含了程序的全部代码和逻辑。用户可以根据需要对源代码进行修改、编译和构建，以生成可执行的二进制文件。通常，源代码分发方式被用于开源软件或者需要自定义配置的软件。**二进制分发则是将已编译好的二进制程序直接分发给用户**。这种方式不需要用户进行编译，用户可以直接使用软件。二进制分发方式常见于商业软件或者用户不需要自定义配置的软件。两种分发方式各有优缺点。源码分发方式具有开放性，用户可以自由地对程序进行修改和定制，但是用户需要具备一定的编程技能和时间成本。二进制分发方式则更加方便和易用，但是用户无法自定义修改软件的源代码。

**NixOS 默认就是源码分发形式**。NixOS 使用 Nix 包管理器来管理软件包，这个管理器可以自动地从源代码构建软件包，并生成可执行的二进制文件。NixOS 中的软件包通常是通过 Nix 表达式的方式来定义的，这些表达式包含了构建软件包所需的所有信息，包括源代码、依赖项、构建脚本等等。当用户需要安装一个软件包时，Nix 包管理器会自动下载源代码，并按照表达式中的指令进行构建。构建完成后，Nix 包管理器会将生成的二进制文件保存在一个固定的路径下，这个路径可以通过环境变量进行访问。NixOS 的源代码分发方式具有很多优点，例如易于定制和升级、减少依赖关系和版本冲突等等。同时，NixOS 的包管理器还支持多版本软件共存、自动回滚等高级特性，这使得 NixOS 成为一款非常灵活和强大的操作系统。
