# claude-code-paper-writer

A **Claude Code skill** for interactive, sentence-by-sentence academic paper revision — strips AI writing tells while preserving the author's own voice.

> 这是为 **Claude Code** 设计的 skill：逐句改论文，去 AI 味、保留作者本人的文风，每一句都由作者拍板。

## 出处与许可

改编自 [caidish/cAI-tools](https://github.com/caidish/cAI-tools) 中的 `plugins/science-skill/skills/hardworking-paper-writer`（下载于 2026-07-14）。**原仓库未附开源许可证，上游内容的权利归原作者**。本仓库为其修改版，基于 GitHub 公开仓库可查看、可 fork 的平台惯例公开分享；若原作者提出异议，将立即转回私有或下架。因上游未授权，本仓库整体不附加开源许可证（`references/register-calibration.md` 除外——其内容蒸馏自 Apache-2.0 许可的 nature-skills，文内已注明来源）。

## 相对上游的本地改动

1. **选项呈现改造**：改写候选先在正文完整展示——原句 + 各候选完整句（inline diff：~~删除线~~ 标删除、**加粗** 标新增）+ 每项一行改动说明；弹框只留 Light / Medium / Bold / Keep original 短标签。
2. 模板小修复：示例状态 `done` → `kept`，与定义的状态值一致。
3. stop-slop 插件引用改为条件式，未安装该插件时不再去找失效路径。
4. 新增合并句规则：被 Bold 改写吞并的句子标记 `merged into SXXX` 并跳过，ID 永不重排。
5. 新增语言承诺：交流用作者聊天的语言，改写用论文本身的语言。
6. 开场一次性提问：选节奏（逐句都停 / 只停需要改的句子）+ 快速校准（目标期刊、禁区章节、初始激进度），记入 style-profile 的 `Session settings`。
7. **完成后自动编译 PDF**：修订全部完成时自动把 `working/` 编译成 PDF（`.tex` 用 latexmk、`.md` 用 pandoc；图/bib 在原目录时通过 `TEXINPUTS`/`BIBINPUTS` 或复制到原目录解决；先编译 `original/` 作对照，只调试修订引入的错误，最多两轮）。提前中断时改为询问是否编译。
8. **六项编辑学增强（2026-07-25，源自 Yuan1z0825/nature-skills 的 nature-polishing 对比筛选，两版同步）**：句子职责标签（一句一职，两职就拆）；first-use map 升级为**术语账本**（防作者术语漂移 + 禁止改写引入同义变体）；hedge 位置与动词-证据梯（只作带标注选项，绝不静默改）；中式英语干扰透镜（按 style-profile 的 L1 记录条件启用）；过度声明监察（只标记 + 给带边界备选）；新增 [references/register-calibration.md](references/register-calibration.md)（20 篇 2025 Nat Comms 实测措辞/时态/连接词校准，Apache-2.0 来源已注明）。逐句零新增开销，仅 Phase 1 多读一个 4KB 文件。
9. **五项健壮性升级（2026-07-17，借鉴 jxzhang97/cAI-tools 状态机版的对比结论，两版同步）**：resume 前校验源文件 SHA-256（防会话间的外部修改被静默覆盖、防 `paper.tex`/`paper.md` 同 stem 撞工作区）；Phase 0 检测多文件 LaTeX 的 `\input{}`/`\include{}`；意义保护清单补全（hedge、界、量词、单位、数字、极性、因果方向）；Keep original 选项也完整重复原句；日志常设 Pending decision 小节。运行时开销仅为每次会话 1–2 次 `shasum` + 1 次 grep，逐句循环零新增。

## 安装

```bash
git clone git@github.com:jxzhang97/claude-code-paper-writer.git
ln -s "$(pwd)/claude-code-paper-writer" ~/.claude/skills/hardworking-paper-writer
```

或直接把本目录复制为 `~/.claude/skills/hardworking-paper-writer/`。新开的 Claude Code 会话即可使用。

## 用法

```
/hardworking-paper-writer path/to/paper.tex
```

支持 `.tex` / `.md` / `.txt`。运行时会在论文旁建立 `<paper-stem>-revision/` 工作区：`original/`（备份，永不修改）、`working/`（实际编辑的副本）、`revision-log.md`（逐句日志，支持断点续改）、`style-profile.md`（文风档案，越改越懂你）。

## 文件结构

```
SKILL.md                            # 主流程与去 slop 原则
references/templates.md             # revision-log / style-profile 模板
references/segmentation.md          # LaTeX/Markdown 分句规则
references/preference-learning.md   # 如何从作者的选择中学习口味
```
