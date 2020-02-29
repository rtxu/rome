# 项目规模

```bash
[rtxu@wcmbp rome]$ cloc --exclude-dir=node_modules .
    8738 text files.
    7176 unique files.
    3679 files ignored.

github.com/AlDanial/cloc v 1.85  T=5.55 s (912.7 files/s, 19513.8 lines/s)
--------------------------------------------------------------------------------
Language                      files          blank        comment           code
--------------------------------------------------------------------------------
TypeScript                     1341          13073          10524          76583
JavaScript                     3362            219            527           4976
JSON                            336              1              0           1739
Markdown                         13            119              0            270
YAML                              1             10              0             60
Bourne Again Shell                7             26             28             44
Bourne Shell                      2              4              8             13
--------------------------------------------------------------------------------
SUM:                           5062          13452          11087          83685
--------------------------------------------------------------------------------
```

# 代码结构

## 入口

从 [CONTRIBUTING](.github/CONTRIBUTING.md) 得知, 开发板 rome 的入口文件为 [scripts/dev-rome](scripts/dev-rome)

## scripts 文件夹

- ast
- commit-rome
- node -- 开发环境检查, 检查 node 版本, 调用符合版本的 node
- prettier-format
- vendor -- 目前只包含上一版本 build 好的 rome 包
- build-release
- dev-rome -- 用 ./vendor-rome 打包正在开发的 rome, 并执行之, 由此找到 rome binary 的入口文件: [@romejs/cli/bin/rome.ts](packages/@romejs/cli/bin/rome.ts)
- prettier-check
- ts-vscode
- vendor-rome -- 调用上一个版本 build 好的 rome

## rome binary 入口: [@romejs/cli/bin/rome.ts](packages/@romejs/cli/bin/rome.ts)

根据 process.env.ROME_PROCESS_TYPE 进行路由:

- master
- worker
- test-worker
- cli (默认)

:question: 其他 ROME_PROCESS_TYPE 是干嘛的?
A: cli 进程会 fork 进程，新 fork 的进程分为 master/worker/test-worker 三种角色

见 [@romejs/core/common/utils/fork.ts](packages/@romejs/core/common/utils/fork.ts)

## cli 入口: [@romejs/cli/cli.ts](packages/@romejs/cli/cli.ts)

### 1. 大量篇幅用来处理 command-line，其中抽象出三个概念：

- flags，--long_form_flag 和 -short_form_flag，取值支持 boolean 和 string
- commands，rome 本身类似于 npm，支持很多子命令，这些命令有些 cli 本身即可完成，有些需要与 master 通信，cli 仅作为一个可以与 master 通信的 client。  
  这种架构是为了以后方便支持 IDE，如 vscode，master 作为一个单独的进程，用来实现 LSP（Language Server Protocol）  
  commands 根据接收对象不同，分为三类：
  - master commands，接收者为 master 进程
  - client commands，接收者为 cli 进程本身，只不过此时 cli 的角色是 client，负责 start/stop/status/restart master 进程
  - cli commands，接收者为 cli 进程本身
- args

**blocked**

- [@romejs/cli-flags/Parser.ts](packages/@romejs/cli-flags/Parser.ts) 中的 Consumer 是干什么的？有什么作用？

### 2. 启动 master daemon

### 3. client 向 master 发送 command

### 4. 显示 command 执行结果
