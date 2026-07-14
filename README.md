# claude-code-hardworking-paper-writer

A **Claude Code skill** for interactive, sentence-by-sentence academic paper revision — strips AI writing tells while preserving the author's own voice.

> 这是为 **Claude Code** 设计的 skill：逐句改论文，去 AI 味、保留作者本人的文风，每一句都由作者拍板。

## 出处与许可

改编自 [caidish/cAI-tools](https://github.com/caidish/cAI-tools) 中的 `plugins/science-skill/skills/hardworking-paper-writer`（下载于 2026-07-14）。**原仓库未附开源许可证，所有权利归原作者**，故本仓库保持私有、仅供个人使用，请勿公开分发。

## 相对上游的本地改动

1. **选项呈现改造**：改写候选先在正文完整展示——原句 + 各候选完整句（inline diff：~~删除线~~ 标删除、**加粗** 标新增）+ 每项一行改动说明；弹框只留 Light / Medium / Bold / Keep original 短标签。
2. 模板小修复：示例状态 `done` → `kept`，与定义的状态值一致。
3. stop-slop 插件引用改为条件式，未安装该插件时不再去找失效路径。
4. 新增合并句规则：被 Bold 改写吞并的句子标记 `merged into SXXX` 并跳过，ID 永不重排。
5. 新增语言承诺：交流用作者聊天的语言，改写用论文本身的语言。
6. 开场一次性提问：选节奏（逐句都停 / 只停需要改的句子）+ 快速校准（目标期刊、禁区章节、初始激进度），记入 style-profile 的 `Session settings`。

## 安装

```bash
git clone git@github.com:jxzhang97/claude-code-hardworking-paper-writer.git
ln -s "$(pwd)/claude-code-hardworking-paper-writer" ~/.claude/skills/hardworking-paper-writer
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
