# hermes-gh-demo-tools

一组围绕 Hermes CLI 封装的 GitHub 仓库研究脚本，目标不是替代作者体验仓库，而是把「事实抓取 + 选题前判断」这段重复工作工具化。

## 解决什么问题

原始的 Hermes TUI 在这个场景里不够顺手：

- 会话管理重
- 复制粘贴别扭
- 长 prompt 重复输入
- 模型容易在缺失事实时脑补结论

这套工具把流程拆成两层：

1. 外层脚本先抓真实 GitHub 事实
   - GitHub API
   - `git ls-remote`
   - `git clone --depth 1`
   - README 摘要
   - 顶层目录 / manifest / demo hints
2. 再把事实喂给 Hermes，只让模型做「选题体验前判断」

这样可以避免「模型联网失败后仍硬写结论」这种废稿。

## 目录结构

```text
bin/
  gh-demo
  gh-demo-open
  gh-demo-save
  gh-demo-save-open
  gh-demo-batch
  gh-demo-batch-open
  gh-demo-stars-top
  gh-demo-x
  gh-demo-x-open
  gh-demo-x-title-open
```

## 核心命令

### 1. 单仓库快速闭环

老版本快速闭环，主要输出：

- 它解决什么问题
- 目标用户是谁
- demo 是否存在
- 最小闭环怎么跑

```bash
gh-demo https://github.com/owner/repo
```

### 2. 单仓库选题判断卡

当前最有价值的命令。它会：

- 先抓 GitHub 事实
- 再输出一张「是否值得进入体验阶段」的判断卡
- 结果写入 `~/Obsidian/GitHub 仓库闭环/`

```bash
gh-demo-x https://github.com/owner/repo
gh-demo-x-open https://github.com/owner/repo
```

可自定义输出标题：

```bash
gh-demo-x --title "自定义标题" https://github.com/owner/repo
gh-demo-x-title-open --title "自定义标题" https://github.com/owner/repo
```

### 3. 批量处理

支持 `txt/csv`，会自动提取 GitHub 仓库 URL。

```bash
gh-demo-batch --dry-run /path/to/repos.csv
gh-demo-batch-open /path/to/repos.csv
```

### 4. GitHub Star Top 专用入口

默认：

- `--limit 5`
- `--skip-existing`

```bash
gh-demo-stars-top /path/to/export.csv
gh-demo-stars-top --dry-run /path/to/export.csv
gh-demo-stars-top --limit 10 /path/to/export.csv
gh-demo-stars-top --no-skip-existing /path/to/export.csv
```

## `gh-demo-x` 当前输出结构

最终 Markdown 只保留决策信息，不再整段抄 README：

- Repo
- Generated
- Stage
- README 链接
- 已验证事实
- 判断结果

判断结果固定要求模型输出：

1. 一句话判断：值得写 / 待定 / 不值得写
2. 值得写的核心原因
3. 最适合的内容角度
4. 用户必须亲自验证的 2-3 个点
5. 最小体验路径
6. 写作风险 / 为什么可能不值得写
7. 最终建议：入池 / 不入池 / 先观察

## 依赖

- `zsh`
- `git`
- `curl`
- `python3`
- `hermes`
- macOS `open`

并且要求 Hermes 已经配置好模型 provider。

## 已知边界

- 如果 Hermes provider 欠费 / 额度异常，事实抓取可以成功，但模型判断阶段会失败。
- `gh-demo-x` 依赖 GitHub API 和 `git clone`；如果本机网络确实不可达，最终只会产出事实失败卡。
- 这套工具不能替代真人体验仓库，它只负责把「值不值得花时间体验」判断得更快。

## 为什么值得单独存档

这不是一组随手 alias，而是一套已经跑通过、并且踩过坑后收敛出的工作流：

- 从 Hermes TUI 迁移到命令式用法
- 从 README 搬运，收敛成判断卡
- 从模型自由联网，改为外层先抓事实
- 从全量批处理，收敛到 limit / skip-existing / dry-run

它已经不再是「试试看」，而是一套可继续演进的工具链。
