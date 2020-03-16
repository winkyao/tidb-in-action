在各种系统软件和应用软件的安装管理中，包管理器均有着广泛的应用，包管理工具的出现也大大简化了软件安装升级维护的工作。例如，几乎所有使用 RPM 的 Linux 都会使用 Yum 来进行包管理，Anaconda 可以非常方便的管理 python 的环境和相关软件包。在早期的 TiDB 生态中，没有专门的包管理工具，使用者只能通过相应的配置文件和文件夹命名来手动管理，像 Prometheus 等第三方监控报表工具甚至需要额外的特殊管理，这样大大提高了相应的运维管理工作。

如今，在 TiDB4.0 的生态系统里，TiUP作为新的工具，承担着包管理器的角色，管理着 TiDB 生态下众多的组件（例如 TiDB、PD、TiKV），用户想要运行 TiDB 生态中任何东西的时候，只需要执行 TiUP 的一行命令即可，相比之前极大的降低了管理难度。用户可以访问 [https://tiup.io/](https://tiup.io/) 来查看相应的文档。本文基于v0.0.2版本 tiup 编写。

**命令安装**

TiUP 当前支持在平台 Linux/MacOS/Windows WSL 运行，登陆终端控制台执行下面命令，如果无错误输出即完成tiup的安装：

```
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```
**功能介绍**

通过tiup --help 查看其支持的指令和参数：

```
Usage:
  tiup [flags] <command> [args...]
  tiup [flags] <component> [args...]

Available Commands:
  install     Install a specific version of a component
  list        List the available TiDB components or versions
  uninstall   Uninstall components or versions of a component
  update      Update tiup components to the latest version
  status      List the status of instantiated components
  clean       Clean the data of instantiated components
  help        Help about any command or component

Components Manifest:
  use "tiup list --refresh" to fetch the latest components manifest

Flags:
  -B, --binary <component>[:version]   Print binary path of a specific version of a component <component>[:version]
                                       and the latest version installed will be selected if no version specified
  -h, --help                           help for tiup
      --mirror mirror                  Overwrite default mirror or TIUP_MIRRORS environment variable (default "https://tiup-mirrors.pingcap.com/")
      --rm                             Remove the data directory when the component instance finishes its run
      --skip-version-check             Skip the strict version check, by default a version must be a valid SemVer string
  -T, --tag string                     Specify a tag for component instance
      --version                        version for tiup

Examples:
  $ tiup playground                    # Quick start
  $ tiup playground nightly            # Start a playground with the latest nightly version
  $ tiup install <component>[:version] # Install a component of specific version
  $ tiup update --all                  # Update all installed components to the latest version
  $ tiup update --nightly              # Update all installed components to the nightly version
  $ tiup update --self                 # Update the "tiup" to the latest version
  $ tiup list --refresh                # Fetch the latest supported components list
  $ tiup status                        # Display all running/terminated instances
  $ tiup clean <name>                  # Clean the data of running/terminated instance (Kill process if it's running)
  $ tiup clean --all                   # Clean the data of all running/terminated instances

Use "tiup [command] --help" for more information about a command.

```

命令参数解释如下：

flags:
* --bin: 打印某个组件的二进制文件存放位置
* -h, --help: 打印 help 信息
* --mirror: 指定一个镜像替换默认的官方镜像
command:
* install: 安装某个组件的某个版本
* list: 查看有哪些组件可以安装，以及这些组件有哪些版本可选
* uninstall: 删除某个组件
* update: 升级某个组件到最新的版本
* status: 查看组件组件的运行状态/运行历史
* clean: 清除某次运行后的数据
* help: 打印 help 信息，后面跟子命令则是打印该子命令的使用方法
args：
通常用于对command进行补充，例如我们想要知道某个子命令的具体用法，执行 tiup subcommand -h/--help 就可以看到

**使用参考**

* 查询组件列表：tiup list

命令帮助信息：

```
tiup list --help
Usage:
  tiup list [component] [flags]
Flags:
  -h, --help        help for list
      --installed   List installed components only.
      --refresh     Refresh local components/version list cache.
```
从 tiup list [component] [flags] 可以看出，tiup list 有以下两种用法：
* tiup list: 查询支持的组件列表
* tiup list <component>: 查询某个组件的支持的版本

对于上面两种使用方法，可以组合使用两个 flag:

* --installed: 本地已经安装了哪些组件，或者某个组件的哪些版本
* --refresh: 刷新本地缓存的组件以及版本信息

示例一：查看当前已经安装的所有组件，命令如下：

```
tiup list --installed
```
示例二：从服务器获取 TiKV 所有可安装版本组件列表，命令如下：
```
tiup list tikv --refresh
```
* 安装组件：tiup install

命令帮助信息：

```
tiup install -h
Usage:
  tiup install <component1>:[version] [component2...N] [flags]
Flags:
  -h, --help   help for install
```
install 的使用方式主要以下两种：
* tiup install <component>：安装指定组件的最新稳定版
* tiup install <component>:[version]: 安装指定组件的指定版本

示例一：使用TiUP安装TiDB：

```
tiup install tidb
```
示例二：使用TiUP安装nightly版本的TiDB：
```
tiup install tidb:nightly
```
示例三：使用TiUP安装3.0.6版本的TiKV：
```
tiup install tikv:v3.0.6
```
* 升级组件：tiup update

命令帮助信息：

```
tiup update -h
Usage:
  tiup update [component1]:[version] [component2..N] [flags]
Flags:
      --all       Update all components
      --force     Force update a component to the latest version
  -h, --help      help for update
      --nightly   Update the components to nightly version
      --self      Update tiup to the latest version
```
使用方式上和 install 基本相同，不过它支持几个额外的 flag:
* --all: 升级所有组件
* --nightly: 升级至 nightly 版本（若无此参数则升级到最新稳定版）
* --self: 升级 tiup 自己
* --force：强制升级至最新版本（若无此参数则本地有最新版本了就不更新）

示例一：升级所有组件至最新版本：

```
tiup update --all
```
示例二：升级所有组件至最新nightly版本：
```
tiup update --nightly --all
```
示例三：升级TiUP至最新版本：
```
tiup update --self
```

* 删除组件：tiup uninstall 

TiUP 安装的组件是要占用本地磁盘空间的，如果不想要那么多老版本的组件，可以先查看当前安装了哪些版本的组件，然后在删除某个组件的某个版本，同时也支持删除所有版本。命令帮助信息：

```
Usage:
  tiup uninstall <component1>:<version> [flags]
Flags:
      --all    Remove all components or versions.
  -h, --help   help for uninstall
      --self   Uninstall tiup and clean all local data
```
示例一：删除3.0.8版本的 TiDB：
```
tiup uninstall tidb:v3.0.8
```
示例二：删除所有版本的TiKV：
```
tiup uninstall tikv --all
```
示例三：删除所有已经安装的组件：
```
tiup uninstall --all
```
* 数据清除：tiup clean

tiup uninstall 只是将组件的二进制文件从系统中删除了，但是组件运行时生成的数据仍然存在，如果想要删除，需要使用 tiup clean。命令帮助信息：

```
Usage:
  tiup clean [flags]
Flags:
      --all    Clean all data of instantiated components
  -h, --help   help for clean
```

如果需要清除所有组件的所有运行数据，加上 --all 即可。

示例一：根据实例 tag 清除数据：

```
tiup clean experiment
```

* 查询已经安装的组件状态

TiUP提供查看所有实例（包括正在运行）的功能，帮助信息如下：

```
tiup status
```
