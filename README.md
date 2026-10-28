# Eeyi

把长命令输出压成能用的摘要，不丢值得看的那几行。零依赖，单文件，Python 3.8+。

给 AI 编程助手（或任何人）看构建日志、测试输出、`git diff`、目录结构、大 JSON 时用：直接喂原文会吃掉上下文，Eeyi 只保留头尾、错误、堆栈和结构骨架，其余折叠成带行号的省略标记。

## 安装

作为技能装进 AI 助手：

```bash
python3 eeyi.py install                    # 自动找最近的 .aicode/skills
python3 eeyi.py install --scope ~/.aicode/skills
```

不需要技能机制时，直接当命令用：

```bash
<命令> 2>&1 | python3 eeyi.py trim
```

## 命令

| 命令 | 用途 |
|---|---|
| `trim` | 压缩命令输出：保留头尾、错误行、堆栈块，重复行折叠 |
| `digest` | 目录结构或单文件符号骨架（函数/类清单） |
| `json` | 大 JSON 压成「结构 + 首末样本」 |
| `count` | 估算 token 体积 |
| `install` | 安装为技能 |
| `bench` | 压缩率、保真度、耗时基准 |
| `selftest` | 内置 60 项单测 |

常用参数：`--max-lines`（默认 80）、`--head/--tail`（默认 12/12）、`--max-col`（默认 300）、`--min-repeat`（默认 3）、`--keep-all-errors`。完整策略见 [GUIDE.md](GUIDE.md)。

## 效果

200 行日志，其中 5 条连接失败、一条堆栈：

```text
$ python3 eeyi.py trim < demo.log
eeyi trim: 200→38 行, ~882→236 token (+73%)
<首个错误：第 57 行 ERROR worker 57: connection refused to db-1>
<错误位置：app/jobs.py:88>
INFO step 1 ok
...
ERROR worker 57: connection refused to db-1
ERROR worker 58: connection refused to db-1
ERROR worker 59: connection refused to db-1
<省略 3 行>
ERROR worker 63: connection refused to db-1
```

38 行里所有错误行都在，堆栈整块保留，中间 180 多行 INFO 折成标记。省 73%，且没删掉定位所需的信息。

```bash
$ python3 eeyi.py count demo.log
     882 token       3488 字符  demo.log
```

## 它不做什么

- **不是原文**。`<…>` 标记处都不是连续文本，改代码、打补丁、核对拼写必须回原文。
- `<首个错误：第 N 行>` 的行号来自折叠后的中间序列，不能拿去定位原文；要定位看 `<错误位置：文件:行号>`。
- token 数是按字数比例的估算（中文 ±10%，代码/JSON ±30%），只适合做相对比较。
- 目录 `digest` 是正则启发式，函数体会丢；关键文件读全文。

这些限制在 [GUIDE.md](GUIDE.md) 里逐条写了取证方式（C1–C17）。

## 自检

```bash
python3 eeyi.py selftest    # 60 项单测
python3 verify_usage.py     # 17 条实现约定，逐条输出证据
python3 verify_skill.py     # 技能正文覆盖与成本
python3 verify_tokens.py    # 需要 tiktoken，校准 token 估算误差
```

`verify_*.py` 只随源码分发，`install` 不会复制它们。

## 相关

[EeyiRecall](https://github.com/Ee114514x/EeyiRecall) 是这一系列的可逆版本：保留行都带原文行号，省略处标的是原文行号区间，需要时能 `fetch` 回原文。两者可以串联——Eeyi 判断什么重要，Recall 负责可逆。
