---

name: git-commit-assistant

description: 当用户明确请求“提交/commit/git commit/提交代码/创建一个 commit/审查并提交当前修改”等时，执行一次安全、简洁的 git 提交流程：检查暂存区→（必要时）让用户确认提交范围→快速审查→生成符合仓库风格的提交信息→展示摘要→等待确认→执行 git commit 并验证。

metadata:

short-description: 一键走完安全的 git commit 工作流

---

# Git Commit Assistant（提交助手）

## 触发条件（非常重要）

仅在用户“明确要提交”时执行，例如包含：

- 提交、commit、git commit、提交代码、创建一个 commit、审查并提交当前的修改、把这些改动提交到 git

如果用户只是询问差异/想看改动，不要直接提交。

## 总原则

- **先看暂存区**：优先提交已暂存（staged）的内容。

- **暂存区为空才询问范围**：列出工作区变更，让用户选择“全部/指定文件/取消”。

- **快速审查不深挖**：做基础 lint/check、明显调试残留、敏感信息检查；不做大型重构建议。

- **提交信息跟随仓库风格**：先看最近提交历史的 message 风格，再生成一致的格式。

- **最后一步必须二次确认**：在执行 `git commit` 前，向用户展示摘要+commit message 并询问是否确认。

---

## 步骤 1：检查暂存区状态

运行：

- `git status --porcelain`

- `git diff --cached --stat`

决策：

- 如果暂存区 **非空**：继续步骤 3（审查）与步骤 4（生成提交信息）。

- 如果暂存区 **为空**：继续步骤 2（让用户确认提交范围）。

---

## 步骤 2：暂存区为空 → 询问用户提交范围

运行：

- `git status --short`

- `git diff --name-status`

将文件清单展示给用户（包含状态：M/A/D/??），并询问用户选择：

1) 提交所有修改（包含新增/删除/未跟踪）

2) 只提交指定文件（让用户勾选或点名）

3) 取消提交

根据用户选择执行：

- 全部：`git add -A`

- 指定文件：`git add <file1> <file2> ...`（删除文件也要包含在 add 里）

- 取消：输出“已取消提交操作。”并停止

完成后再次运行：

- `git diff --cached --stat`

确认暂存区现在非空。

---

## 步骤3

## 步骤 3：简单代码审查（快速）

目标：在不拖慢流程的前提下，减少明显错误/泄露风险。

### 3.1 基础健康检查（按项目生态选择）

目的：在提交前自动统一 C/Python 风格，避免格式化 diff 混进后续提交。

执行：

运行 devfmt（如失败，停止并提示用户查看输出）

运行后检查是否产生新改动：

git status --porcelain

git diff --stat

若 devfmt 产生改动：

如果当前提交范围是“已暂存内容”：只重新暂存已暂存文件里被格式化影响的部分（见下条）

最简实现：直接 git add -u（只 add 已跟踪文件的修改，避免把新文件误加进来）

然后 git diff --cached --stat 重新确认

决策：

devfmt 失败：停止，提示“devfmt 失败，修复后再提交”

devfmt 成功：继续 Step 执行

--

### 3.1.1 自动修复轻微问题 严重问题手动确认

目标：对“低风险可自动修”的问题直接修复并更新暂存区；对“可能改变语义/需要人为判断”的问题停止并征求确认。

自动修范围（默认启用）：

Python：ruff check --fix（仅 safe fixes）

可选：ruff format（如果你用 ruff-format；不强制）

执行顺序（若命令不存在则跳过但记录）：

ruff check --fix .

重新暂存：git add -u

重新输出：git diff --cached --stat

需要用户确认的情况（触发则停止到 Step 6 询问）：

ruff/black 报错且无法自动修复

发现潜在高风险（密钥、证书私钥、.env、token）

发现大范围重排（例如一次性改动大量无关文件、include 顺序大量变化），提示用户是否继续

输出要求：

自动修发生时，提示“已自动修复：xxx（文件数/规则）”

需要确认时，列出文件与行号（不要泄露敏感内容），然后询问是否继续

### 3.2 低成本风险扫描

在 diff 里检查：

- 调试残留：`console.log` / `TODO` / `FIXME`

- 敏感信息：疑似 token、key、password、.env等

如果发现高风险（例如疑似密钥、.env、私钥）：

- 明确指出文件与片段位置（不要在输出中完整泄露密钥）

- 建议先移除/加入 `.gitignore`/使用示例文件（如 `.env.example`）

- 询问用户：**“仍要继续提交吗？”**

---

## 步骤 4：分析提交历史（对齐风格）

运行：

- `git log --oneline -10`

- `git log --format="%s" -20`

从最近 20 条 subject 里归纳：

- 是否使用 Conventional Commits（feat/fix/chore 等）

- scope 是否常见（如 feat(api): ...）

- 使用中文还是英文

- 是否有多行 body

---

## 步骤 5：生成提交信息（默认 Conventional Commits）

如果仓库历史明显使用 Conventional Commits，则遵循：

- `<type>(<scope>): <subject>`

否则使用仓库常见格式（例如纯 subject、或带前缀）。

要求：

- subject 简洁、描述“目的”，避免堆技术细节

- 多项改动：在 body 用要点列出（- ...）

- 如有 breaking change：加 `BREAKING CHANGE:`（仅在确实破坏兼容时）

生成提交信息前，先阅读：

- `git diff --cached`

提炼主要目的、影响模块、文件范围。

---

## 步骤 6：展示摘要并请求确认

向用户展示：

- 变更摘要（文件数、+/- 行数、主要文件列表）

- 建议的 commit message（含 body）

- 审查结论（通过/发现问题与建议）

然后询问：

- “是否确认提交？（是/否）”

- 若否：停止；若是：进入步骤 7。

---

## 步骤 7：执行提交并验证

执行 commit（多行 message）：

- `git commit -m "$(cat <<'EOF'

<COMMIT_MESSAGE>

EOF

)"`

验证：

- `git log -1 --oneline`

输出最终结果：最新 commit 的 hash 与 subject。

---

## 输出规范（给用户看的格式）

尽量使用如下结构：

📊 变更摘要：

- 修改文件：N

- 新增行：+X

- 删除行：-Y

📝 提交信息：

<type(scope): subject>

- bullet 1

- bullet 2

✅ 快速审查：通过

或

⚠️ 快速审查：发现问题（列出）

👉 是否确认提交？（是/否）