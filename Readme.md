## 简介

这个小项目展现了一种情况：
```
master —o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—o—>
           \
            `— hardware —o—o—o—o—o—o—o—o—o—o—o—o—o—o—>
                            \
                             `— ability —o—o—o—o—o—o—o—>
```
在最初的设想中，ability 将会合入 hardware，而 hardware 将会合入 master
但在实际操作过程中，由于各个分支承担作用的原因，导致原来的设想实现困难

## 分支作用

整个系统可以抽象成3个部分: 硬件、底层软件、上层软件
| 分支 | 硬件 | 底层软件 | 上层软件 | 作用 |
| ---- | --- | -------- | ------- | ---- |
| master | 旧版本 | 稳定版本 | 新版本 | 使用稳定的底层环境，支持快速重构迭代上层软件 |
| hardware | 新版本(仿真) | 适配新版本硬件 | 稳定版本 | 使用稳定的上层软件，支持新版本硬件的适配 |
| ability | 新版本(仿真) | 同步 hardware 必要部分 | 同步 master 必要部分 | 同步需验规格相关的新版本硬件和代码，支持规格验证 |

## 项目现状

**原计划:**
1. ability 分支验证完规格后，合入 hardware 分支，并用编译宏区分 ability 分支和 hardware 分支的代码
2. master 分支与 hardware 分支长期共存，在新版本硬件实物能够使用前，硬件、底层软件相关验证主要使用 hardware 分支，
    上层软件相关验证主要使用 master 分支
3. 在新版本硬件实物投入使用前，将 hardware 分支合入 master 分支，使 master 分支支持新版本硬件，此时 master 分支为全量新版本代码

**遇到困难:**
1. 由于 ability 分支同步了 master 分支中较多上层软件重构后的新代码，ability 合入 hardware 时，上层软件部分会有很多冲突
2. 就算 ability 分支使用编译宏的方式，规避了所有上层软件冲突（也避免合入 hardware 分支时误丢失代码），
    合入后 hardware 分支的代码也会充斥着编译宏，可读性差（或者直接丢弃 hardware 分支中与 ability 分支不同的代码？）
3. （假如不丢弃 hardware 分支用编译宏区分开的与 ability 分支不同的代码，即虽然是一个分支，但事实上作为2个分支看待）
    在 ability 分支合入 hardware 分支后，如果还有其他验证规格的需求，还需要再在 hardware 分支用编译宏区分原 ability 分支用编译宏区分的代码，
    这种写法会逐渐增大维护的难度
4. 按照 hardware 分支的作用来说，不同步新版本的上层软件没关系，但对 ability 部分代码来说，最好是使用新版本的上层软件代码，
    这就会导致 hardware 与 ability 差距逐渐扩大

## 根因分析

分支合并困难的根源在于，一开始对 ability 分支的作用没有明确的认识。

master 分支合 hardware 分支分别承担了上层软件和底层软件的开发工作。

而 ability 分支的作用更多地只是做规格验证，对验证有问题的规格，解决后同步其他分支即可，ability 分支更多的工作是同步 master 分支和
hardware 分支相关代码，以支持规格验证。直到新版本硬件实物投入使用后，规格验证的工作才会转交给 master 分支。

做规格验证的 ability 分支从 master 分支或 hardware 分支拉出来都可以，区别只是：
从 master 分支拉出来，底层软件相关代码会同步更多；从 hardware 分支拉出来，上层软件相关代码会同步更多。
但如果让 ability 分支要直接合入到 master 或 hardware 分支都不合适，
如果合入到 master 分支，而当前 master 分支并不支持新版本硬件，那么 ability 底层软件相关的配置都要用编译宏区分开；
如果合入到 hardware 分支，由于 ability 分支同步了 master 分支大量上层软件修改代码，直接合入容易出问题，但假如用编译宏区分，
代码的可维护性又变得更差。

## 解决办法

综上，也许保留 ability 分支是更好的解决办法
1. 保留 ability 分支供规格验证使用，ability 分支从 master 分支和 hardware 分支同步新代码
2. ability 通过规格验证发现的问题，进行修复的代码量与 master 和 hardware 这样的开发分支相比较少，原计划将 ability 分支合入 hardware 分支，
    大部分等效于将 master 分支代码同步到 hardware 分支，小部分 ability 分支修复的问题，及时同步修复问题代码到 hardware 分支即可