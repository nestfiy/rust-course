# 附录 F：Rust 版本发布

## Rust 版本说明
早在第一章，我们见过 `cargo new` 在 *Cargo.toml* 中增加了一些有关 `edition` 的元数据。本附录将解释其意义！

与其它语言相比，Rust 的更新迭代较为频繁（得益于精心设计过的发布流程以及 Rust 语言开发者团队管理）：

- 每 6 周发布一个迭代版本
- 2 - 3 年发布一个新的大版本：每一个版本会结合已经落地的功能，并提供一个清晰的带有完整更新文档和工具的功能包。新版本会作为常规的 6 周发布过程的一部分发布。

好处在于，可以满足不同的用户群体的需求：

- 对于活跃的 Rust 用户，他们总是能很快获取到新的语言内容，毕竟，尝鲜是技术爱好者的共同特点:)
- 对于一般的用户，edition 的发布会告诉这些用户：Rust 语言相比上次大版本发布，有了重大的改进，值得一看
- 对于 Rust 语言开发者，可以让他们的工作成果更快的被世人所知，不必锦衣夜行


在本文档编写时，Rust 已经有三个版本：Rust 2015、2018、2021。本书基于 `Rust 2021 edition` 编写。

*Cargo.toml* 中的 `edition` 字段表明代码应该使用哪个版本编译。如果该字段不存在，其默认为 `2021` 以提供后向兼容性。

每个项目都可以选择不同于默认的 `Rust 2021 edition` 的版本。这样，版本可能会包含不兼容的修改，比如新版本中新增的关键字可能会与老代码中的标识符冲突并导致错误。不过，除非你选择应用这些修改，否则旧代码依然能够被编译，即便你升级了编译器版本。

所有 Rust 编译器都支持任何之前存在的编译器版本，并可以链接任何支持版本的包。编译器修改只影响最初的解析代码的过程。因此，如果你使用 `Rust 2021` 而某个依赖使用 `Rust 2018`，你的项目仍旧能够编译并使用该依赖。反之，若项目使用 `Rust 2018` 而依赖使用 `Rust 2021` 亦可工作。

有一点需要明确：大部分功能在所有版本中都能使用。开发者使用任何 Rust 版本将能继续接收最新稳定版的改进。然而在一些情况，主要是增加了新关键字的时候，则可能出现了只能用于新版本的功能。只需切换版本即可利用新版本的功能。

请查看 [Edition Guide](https://rust-lang-nursery.github.io/edition-guide/) 了解更多细节，这是一个完全介绍版本的书籍，包括如何通过 `cargo fix` 自动将代码迁移到新版本。


## Rust 自身开发流程

本附录介绍 Rust 语言自身是如何开发的以及这如何影响作为 Rust 开发者的你。

### 无停滞稳定

作为一个语言，Rust **十分** 注重代码的稳定性。我们希望 Rust 成为你代码坚实的基础，假如持续地有东西在变，这个希望就实现不了。但与此同时，如果不能实验新功能的话，在发布之前我们又无法发现其中重大的缺陷，而一旦发布便再也没有修改的机会了。

对于这个问题我们的解决方案被称为 “无停滞稳定”（“stability without stagnation”），其指导性原则是：无需担心升级到最新的稳定版 Rust。每次升级应该是无痛的，并应带来新功能，更少的 Bug 和更快的编译速度。

### Choo, Choo! ~~ 小火车发布流程启动

开发 Rust 语言是基于一个**火车时刻表**来进行的：所有的开发工作在 Master 分支上完成，但是发布就像火车时刻表一样，拥有不同的时间，发布采用的软件发布列车模型，被用于思科IOS和等其它软件项目。Rust 有三个 **发布通道**（*release channel*）：

* Nightly
* Beta
* Stable（稳定版）

大部分 Rust 开发者主要采用稳定版通道，不过希望实验新功能的开发者可能会使用 nightly 或 beta 版。

如下是一个开发和发布过程如何运转的例子：假设 Rust 团队正在进行 Rust 1.5 的发布工作。该版本发布于 2015 年 12 月，这个版本和时间显然比较老了，不过这里只是为了提供一个真实的版本。Rust 新增了一项功能：一个 `master` 分支的新提交。每天晚上，会产生一个新的 nightly 版本。每天都是发布版本的日子，而这些发布由发布基础设施自动完成。所以随着时间推移，发布轨迹看起来像这样，版本一天一发：

```text
nightly: * - - * - - *
```

每 6 周时间，是准备发布新版本的时候了！Rust 仓库的 `beta` 分支会从用于 nightly 的 `master` 分支产生。现在，有了两个发布版本：

```text
nightly: * - - * - - *
                     |
beta:                *
```

大部分 Rust 用户不会主要使用 beta 版本，不过在 CI 系统中对 beta 版本进行测试能够帮助 Rust 发现可能的回归缺陷（regression）。同时，每天仍产生 nightly 发布：

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

比如我们发现了一个回归缺陷。好消息是在这些缺陷流入稳定发布之前还有一些时间来测试 beta 版本！fix 被合并到 `master`，为此 nightly 版本得到了修复，接着这些 fix 将 backport 到 `beta` 分支，一个新的 beta 发布就产生了：

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

第一个 beta 版的 6 周后，是发布稳定版的时候了！`stable` 分支从 `beta` 分支生成：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

好的！Rust 1.5 发布了！然而，我们忘了些东西：因为又过了 6 周，我们还需发布 **新版** Rust 的 beta 版，Rust 1.6。所以从 `beta` 分支生成 `stable` 分支后，新版的 `beta` 分支也再次从 `nightly` 生成：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

这被称为 “train model”，因为每 6 周，一个版本 “离开车站”（“leaves the station”），不过从 beta 通道到达稳定通道还有一段旅程。

Rust 每 6 周发布一个版本，如时钟般准确。如果你知道了某个 Rust 版本的发布时间，就可以知道下个版本的时间：6 周后。每 6 周发布版本的一个好的方面是下一班车会来得更快。如果特定版本碰巧缺失某个功能也无需担心：另一个版本很快就会到来！这有助于减少因临近发版时间而偷偷释出未经完善的功能的压力。

多亏了这个过程，你总是可以切换到下一版本的 Rust 并验证是否可以轻易的升级：如果 beta 版不能如期工作，你可以向 Rust 团队报告并在发布稳定版之前得到修复！beta 版造成的破坏是非常少见的，不过 `rustc` 也不过是一个软件，可能会存在 Bug。

### 不稳定功能

这个发布模型中另一个值得注意的地方：不稳定功能（unstable features）。Rust 使用一个被称为 “功能标记”（“feature flags”）的技术来确定给定版本的某个功能是否启用。如果新功能正在积极地开发中，其提交到了 `master`，因此会出现在 nightly 版中，不过会位于一个 **功能标记** 之后。作为用户，如果你希望尝试这个正在开发的功能，则可以在源码中使用合适的标记来开启，不过必须使用 nightly 版。

如果使用的是 beta 或稳定版 Rust，则不能使用任何功能标记。这是在新功能被宣布为永久稳定之前获得实用价值的关键。这既满足了希望使用最尖端技术的同学，那些坚持稳定版的同学也知道其代码不会被破坏。这就是无停滞稳定。

本书只包含稳定的功能，因为还在开发中的功能仍可能改变，当其进入稳定版时肯定会与编写本书的时候有所不同。你可以在网上获取 nightly 版的文档。

### Rustup 和 Rust Nightly 的职责

#### 安装 Rust Nightly 版本
Rustup 使得改变不同发布通道的 Rust 更为简单，其在全局或分项目的层次工作。其默认会安装稳定版 Rust。例如为了安装 nightly：

```text
$ rustup install nightly
```

你会发现 `rustup` 也安装了所有的 **工具链**（*toolchains*， Rust 和其相关组件）。如下是一位作者的 Windows 计算机上的例子：

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

#### 在指定目录使用Rust Nightly
如你所见，默认是稳定版。大部分 Rust 用户在大部分时间使用稳定版。你可能也会这么做，不过如果你关心最新的功能，可以为特定项目使用 nightly 版。为此，可以在项目目录使用 `rustup override` 来设置当前目录 `rustup` 使用 nightly 工具链：

```text
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

现在，每次在 *~/需要nightly的项目/*下(在项目的根目录下，也就是 `Cargo.toml` 所在的目录) 调用 `rustc` 或 `cargo`，`rustup` 会确保使用 nightly 版 Rust。在你有很多 Rust 项目时大有裨益！

### RFC 过程和团队

那么你如何了解这些新功能呢？Rust 开发模式遵循一个 **Request For Comments (RFC) 过程**。如果你希望改进 Rust，可以编写一个提议，也就是 RFC。

任何人都可以编写 RFC 来改进 Rust，同时这些 RFC 会被 Rust 团队评审和讨论，他们由很多不同分工的子团队组成。这里是 [Rust 官网](https://www.rust-lang.org/governance) 上所有团队的总列表，其包含了项目中每个领域的团队：语言设计、编译器实现、基础设施、文档等。各个团队会阅读相应的提议和评论，编写回复，并最终达成接受或回绝功能的一致。

如果功能被接受了，在 Rust 仓库会打开一个 issue，人们就可以实现它。实现功能的人可能不是最初提议功能的人！当实现完成后，其会合并到 `master` 分支并位于一个特性开关（feature gate）之后，正如[不稳定功能](#不稳定功能) 部分所讨论的。

在稍后的某个时间，一旦使用 nightly 版的 Rust 团队能够尝试这个功能了，团队成员会讨论这个功能在 nightly 中运行的情况，并决定是否应该进入稳定版。如果决定继续推进，特性开关会移除，然后这个功能就被认为是稳定的了！乘着“发布的列车”，最终在新的稳定版 Rust 中出现。
