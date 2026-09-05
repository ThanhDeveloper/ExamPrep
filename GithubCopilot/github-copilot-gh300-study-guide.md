# GitHub Copilot From Zero to GH-300

> Complete but concise study guide for Vietnamese developers  
> **Verified against current official documentation: 2026-09-05**  
> Scope: beginner use, professional administration, and GH-300 exam revision

GitHub Copilot changes quickly. Prices, quotas, previews, product names, and UI routes in this guide are a dated snapshot. Before the exam, recheck the [current GH-300 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-300), [Copilot plans](https://docs.github.com/en/copilot/get-started/plans), and [feature matrix](https://docs.github.com/en/copilot/reference/copilot-feature-matrix). The exam blueprint shown by Microsoft is **skills measured as of August 7, 2026**; most questions cover generally available features, but commonly used previews can appear.

## How to use this guide

- 🟢 means beginner knowledge: understand the idea and use it safely.
- 🔵 means professional/GH-300 knowledge: choose the correct plan, policy, scope, role, surface, or evidence.
- **CURRENT** means verified current behavior. **LEGACY / OLD EXAM TERM** means recognize it, but do not use it as current syntax.
- In a negative question, circle **NOT**, **EXCEPT**, or **LEAST** before reading the choices.
- Product behavior can vary by IDE/version. `✓` means supported, `P` means preview, and `✗` means not supported in the cited current matrix.

## Exam blueprint: what deserves your time

| Current skill area | Weight |
|---|---:|
| Use GitHub Copilot responsibly | 15–20% |
| Use GitHub Copilot features | 25–30% |
| GitHub Copilot features | 25–30% |
| Understand Copilot data and architecture | 10–15% |
| Apply prompt engineering and context crafting | 10–15% |
| Improve developer productivity | 10–15% |
| Configure privacy, content exclusions, and safeguards | 10–15% |

The duplicated “features” headings are present in Microsoft's current blueprint; do not invent a correction. A passing score is **700 or greater**, but that is a scaled score—not necessarily 70%. [Official GH-300 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-300)

---

# Part 1 — What is GitHub Copilot?

## 🟢 Beginner

**What:** GitHub Copilot is an AI coding assistant. It can suggest code while you type, answer questions, edit files, review code, work from a terminal, or perform a delegated task.

**Why:** It can reduce repetitive work and explain unfamiliar code. It is not an authority: generated code may be wrong, insecure, biased, outdated, or too similar to public code.

**Example:** Write `// return unique email addresses sorted alphabetically`; Copilot may suggest the function.

**What it looks like:** gray “ghost text” in an editor, a Chat panel, an agent session, comments on a code review, or a `copilot` terminal session.

**Remember:** **Copilot proposes; the human verifies and decides.** [Responsible use of inline suggestions](https://docs.github.com/en/copilot/responsible-use/inline-suggestions)

## 🔵 Professional / GH-300

- Copilot is a family of experiences across IDEs, GitHub.com, GitHub Mobile, Windows Terminal, the Copilot CLI, and the Copilot app—not one identical feature everywhere. [Copilot features](https://docs.github.com/en/copilot/get-started/features)
- Availability depends on plan, administrator policies, assigned seat, IDE/client version, repository access, and feature status.
- A language model generates likely output from the prompt and available context. It does not “know” that its answer is correct.
- **Exam keywords:** AI pair programmer, assistant, suggestions, Chat, agent, context, human review.
- **⚠️ Trap:** Copilot is not GitHub itself, GitHub Actions, Codespaces, or GitHub Code Security.

## Essential words

| Term | Simple meaning | Tiếng Việt |
|---|---|---|
| Generative AI | AI that creates new text, code, images, or other output from instructions | AI tạo sinh |
| LLM | Large language model trained to predict/generate language and code | mô hình ngôn ngữ lớn |
| Model | The AI system used to generate a result | mô hình |
| Prompt | The instruction or question sent to the AI | câu lệnh / yêu cầu |
| Response | What the AI returns | câu trả lời |
| Context | Relevant information supplied with a request | ngữ cảnh |
| Context window | The model's fixed token capacity for instructions, conversation, code, and tool results | cửa sổ ngữ cảnh |
| Token | A small unit of text/code processed by a model; not necessarily a whole word | đơn vị văn bản |
| Suggestion | Proposed code/text that you may accept, change, or reject | đề xuất |
| Code completion | Code predicted at the cursor as you type | hoàn thành mã |
| Repository context | Relevant files/information from a repository | ngữ cảnh kho mã |
| Hallucination | Plausible-looking but false or unsupported output | thông tin bịa/sai nhưng có vẻ đúng |
| AI Credit | Current Copilot billing unit; **1 credit = US$0.01** | đơn vị tính mức sử dụng AI |
| Copilot extension | IDE plugin/extension that provides Copilot features | tiện ích Copilot |

Context includes system instructions, messages, model responses, and tool calls/results; every model has a fixed capacity. More irrelevant context can reduce quality and increase cost. [Copilot CLI context management](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/context-management) AI-credit cost depends on the model and input, output, and cached tokens. [Usage-based billing](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises/usage-based-billing)

## Do not confuse the products

| Product | Main job | Not the same as |
|---|---|---|
| GitHub | Hosts repositories, issues, pull requests, collaboration, and more | Copilot AI assistance |
| GitHub Copilot | AI assistance across coding workflows | Source control or CI itself |
| GitHub Actions | Executes automation/CI workflows | Copilot, although cloud agent environments are Actions-powered |
| GitHub Codespaces | Cloud-hosted development environment | Copilot cloud agent |
| GitHub Code Security (GHCS) | Finds and helps remediate vulnerabilities and risky dependencies | Copilot subscription; some Copilot Autofix use does not require a Copilot subscription |
| GitHub.com Copilot | Copilot Chat/agents/review and other AI experiences on GitHub.com | The local IDE extension |
| Copilot CLI | Current agentic terminal interface launched with `copilot` | GitHub CLI `gh`, or IDE `@workspace` |

[GitHub security features](https://docs.github.com/en/code-security/getting-started/github-security-features) · [About Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent)

---

# Basic Copilot request lifecycle

```text
Developer
   ↓
Prompt / code input
   ↓
Context gathering
   ↓
Prompt construction
   ↓
Service/proxy processing and applicable filtering
   ↓
Model invocation
   ↓
Response generation
   ↓
Post-processing and applicable public-code/safety checks
   ↓
Suggestion / answer
   ↓
Human review → accept / reject / modify
```

1. **Input:** your text, code, selection, or task starts the request.
2. **Context gathering:** the client/service collects permitted relevant information, such as surrounding code or referenced files.
3. **Prompt construction:** instructions and context are assembled for the model.
4. **Service/proxy processing:** the request is mediated and applicable policies/filters are applied.
5. **Model:** an LLM predicts a response; it does not prove correctness.
6. **Post-processing:** formatting and applicable safety or public-code checks prepare the output.
7. **Presentation:** you see ghost text, a chat answer, a diff, review comment, or agent result.
8. **Human review:** verify correctness, security, licensing, tests, and business intent.

**⚠️ Accuracy boundary:** Official materials describe input processing, prompt building, proxy filtering, and post-processing, but do not confirm one immutable internal order for every Copilot surface and model. Learn the conceptual flow, not an invented network diagram. [Current GH-300 architecture objectives](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-300) · [Responsible use of Copilot Chat](https://docs.github.com/en/copilot/responsible-use/chat)

---

# Part 2 — Core Copilot features

## 🟢 Beginner feature map

| Feature | What / why / when | What it looks like | Often confused with |
|---|---|---|---|
| Inline suggestion | Predict code at the cursor while typing | Gray ghost text; one or several lines | Inline Chat |
| Multiple suggestions | Alternative completions for the same cursor position | Next/previous alternatives or suggestion panel | Next edit suggestion |
| Next edit suggestions (NES) | Predict the **next edit location and change**, not only text at the cursor | Highlight/gutter cue; `Tab` navigates then accepts | Ordinary completion |
| Copilot Chat | Ask, explain, design, debug, or generate | Chat panel/conversation | Inline ghost text |
| Inline Chat / Inline Edit | Prompt against selected/current code near the editor | Inline input and proposed diff | Chat panel or Edit mode |
| Ask mode | Explain/recommend without autonomously editing files | Chat response | Agent mode |
| Edit mode | Apply controlled changes across a user-defined file set; review per-file edits | Multi-file proposed diffs | Agent mode chooses work/tools itself |
| Agent mode | Locally plans multi-step work, edits files, runs tools/commands with controls, and iterates | Tool calls, terminal commands, file diffs | Asynchronous cloud agent |
| Plan mode | Produce an implementation plan before changes | Structured plan; switch to agent to implement | Agent executing immediately |
| Code review | Find issues and suggest fixes in selected code or a pull request | Review comments/findings | Security scanner guarantees |
| Copilot cloud agent | Works asynchronously in an Actions-powered environment, changes a branch, optionally opens a PR | Agent session log, commits, pull request | Local IDE Agent mode |
| Copilot CLI | Agentic Copilot in a terminal | Interactive `copilot` session or noninteractive output | `gh copilot` legacy CLI extension |

[Chat modes in IDEs](https://docs.github.com/en/copilot/how-tos/chat-with-copilot/chat-in-ide) · [Copilot feature matrix](https://docs.github.com/en/copilot/reference/copilot-feature-matrix) · [About cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent)

## 🔵 Professional / GH-300 distinctions

- **Ask:** explanation/advice; it does not autonomously modify the project.
- **Edit:** controlled multi-file editing in files you specify; currently supported in VS Code and JetBrains.
- **Agent mode:** local and synchronous/interactive; selects relevant files and tools, may run commands, and iterates toward the goal.
- **Plan mode:** separates analysis from implementation. Use it when scope/approach should be reviewed first.
- **Cloud agent:** asynchronous work on GitHub in an ephemeral, GitHub Actions-powered environment. It works on a branch and can open a PR.
- **Code review:** review assistance, not a correctness/security guarantee. A human still reviews and tests.
- **⚠️ Trap:** “agent” alone is ambiguous. Ask **where it runs** and **whether the user waits locally**.

### CURRENT vs legacy terminology

| CURRENT | LEGACY / POSSIBLE OLD EXAM TERM |
|---|---|
| **Copilot cloud agent** | **Copilot coding agent**; older documentation/questions may use this name |
| Standalone `copilot` CLI | `gh copilot suggest` / `gh copilot explain` from the retired/older GitHub CLI extension |
| VS Code `@workspace` participant is current | It is sometimes incorrectly described as CLI syntax; it is **not** the current CLI file-reference syntax |

## Which Copilot feature should I use?

| Scenario | Correct feature |
|---|---|
| Predict the next lines while typing | Inline suggestion / code completion |
| Move to the likely next change after an edit | Next edit suggestions |
| Explain selected code | Inline Chat or Chat with selection context |
| Refactor a small selected block | Inline Chat / Inline Edit |
| Change a known set of files, with controlled diffs | Edit mode |
| Change many files and run local commands | IDE Agent mode |
| Design a multi-file solution before editing | Plan mode |
| Take a GitHub issue and produce a PR asynchronously | Copilot cloud agent (legacy: coding agent) |
| Ask from or operate in a terminal | Copilot CLI |
| Inspect a PR for defects | Copilot code review |

**Memory:** `ghost text` → inline suggestion; `selected code + natural language` → inline Chat/Edit; `multi-step + local tools` → Agent mode; `issue + background + PR` → cloud agent.

## Copilot code review in one minute

- Review selected code in supported IDEs, or request Copilot as a pull-request reviewer on GitHub/through supported tools.
- It identifies possible bugs, security issues, and maintainability/style problems and may offer ready-to-apply suggestions.
- Current reviews consume AI Credits. Agentic review capabilities use GitHub Actions for richer repository context; a fallback review can still run if those Actions capabilities are unavailable.
- By default a GitHub pull-request review is a **Comment**, so it does not count as a required approval; organizations can configure approval behavior and automatic reviews.
- Content exclusions apply to Copilot code review. Some file types are excluded from review independently.
- Review standards can be supplied as repository-wide, path-specific, or agent instruction files where supported. For reviews on GitHub, Copilot reads the instructions from the PR's **head branch**, so keep instructions concise, specific, and reviewable.
- Copilot can miss issues or be wrong. Validate every finding and supplement it with human review and deterministic security/quality tools.

[About Copilot code review](https://docs.github.com/en/copilot/concepts/agents/code-review) · [Using Copilot code review](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review) · [Custom-instruction support](https://docs.github.com/en/copilot/reference/custom-instructions-support)

---

# Inline suggestions and shortcuts

## 🟢 Beginner

Start typing code or write a useful comment. Copilot may show gray **ghost text**. Accept it only after reading it.

- **Full acceptance:** insert the whole suggestion.
- **Partial acceptance:** insert the next word or line where the editor supports it.
- **Dismiss:** remove the visible proposal.
- **Navigate:** cycle through alternative proposals.
- **Multi-line suggestion:** one completion contains several lines.
- **Multiple suggestions:** several alternative completions exist.

## Current common shortcuts

| Action | Windows/Linux | macOS | Notes |
|---|---|---|---|
| Accept suggestion | `Tab` | `Tab` | Common VS Code, Visual Studio, JetBrains, Eclipse |
| Dismiss | `Esc` | `Esc` | Common supported IDEs |
| Trigger inline suggestion | `Alt+\` | `Option+\` | VS Code and JetBrains default |
| Next suggestion | `Alt+]` | `Option+]` | VS Code and JetBrains default |
| Previous suggestion | `Alt+[` | `Option+[` | VS Code and JetBrains default |
| Show alternatives | `Ctrl+Enter` | `Ctrl+Return` | VS Code |
| Accept next word | `Ctrl+Right` | `Cmd+Right` | VS Code; Eclipse uses the same OS pattern |
| Accept next line | Custom binding | Custom binding | VS Code command: `editor.action.inlineSuggest.acceptNextLine` |
| Visual Studio next/previous | `Alt+.` / `Alt+,` | N/A | Visual Studio on Windows |
| Xcode first/full suggestion | `Tab`; hold `Option`, then `Option+Tab` | Same | `Tab` accepts first line; option flow views/accepts full suggestion |
| Eclipse trigger | `Alt+Ctrl+/` | `Option+Cmd+/` | Current Eclipse default |
| Next edit suggestion | `Tab` to navigate, then `Tab` to accept | Same | Follow the highlighted next-edit cue where supported |

Shortcuts can conflict with custom keymaps and extensions. Vim/Neovim mappings are configurable; run `:help copilot`. Always treat the IDE's keybinding UI as authoritative. [Official keyboard shortcuts](https://docs.github.com/en/copilot/reference/keyboard-shortcuts) · [Getting IDE code suggestions](https://docs.github.com/en/copilot/how-tos/get-code-suggestions/get-ide-code-suggestions)

## Four easy-to-confuse experiences

| Concept | Trigger/input | Output |
|---|---|---|
| Inline suggestion | Pause/type at cursor | Ghost text at cursor |
| Next edit suggestion | Make an edit; Copilot predicts related next change | A different edit location/change |
| Inline Chat/Edit | Select/current code + natural-language prompt | Explanation or local diff near code |
| Copilot Chat | Conversation plus attached/implicit context | Rich answer in Chat UI |

**⚠️ Trap:** `Tab` can accept a completion or navigate/accept a next edit. The visible UI and question wording identify which feature is active.

---

# Part 3 — Effective prompting

## 🟢 Beginner: a practical formula

```text
GOAL + CONTEXT + CONSTRAINTS + OUTPUT FORMAT + EXAMPLE/TEST
```

**Bad prompt:** `make this better`

**Good prompt:**

```text
Refactor the selected TypeScript function to remove duplicate database calls.
Keep its public signature and error messages unchanged.
Return a minimal diff and add Jest tests for empty and duplicate inputs.
Do not add dependencies.
```

**Why better:** it states intent, boundary, expected output, and verification.

## 🔵 Professional / GH-300

1. Start broad enough to state the goal, then give specific requirements.
2. Supply **relevant** context: selected code, file, interfaces, failing test, error output, conventions.
3. State constraints: language/version, dependencies, performance, security, files not to change.
4. Show an example when format or behavior is easy to misunderstand (**few-shot**). No example is **zero-shot**.
5. Break complex work into smaller tasks; ask for a plan before high-risk edits.
6. Iterate: inspect the result, correct assumptions, add missing constraints, and retest.
7. Keep conversation history relevant; start a new session or compact it when old material distracts the model.

[Prompt engineering](https://docs.github.com/en/copilot/concepts/prompting/prompt-engineering)

### Prompt vocabulary

| Word | Simple Vietnamese | Meaning here |
|---|---|---|
| clarity | rõ ràng | easy to understand |
| specific | cụ thể | precise details, not vague |
| intent | ý định/mục tiêu | what outcome you want |
| constraint | ràng buộc | rule the answer must obey |
| surrounding | xung quanh | nearby code/context |
| craft/crafted | soạn/tạo có chủ đích | deliberately compose a prompt |
| iterate | lặp để cải tiến | prompt, inspect, improve |
| refine | tinh chỉnh | make more precise |
| relevant | liên quan | useful to the task |
| verbose | dài dòng | more words/detail than needed |
| concise | súc tích | short but sufficient |

**Remember:** more words are not automatically better; more **relevant constraints and context** are better.

## Productivity playbook

| Goal | Practical Copilot workflow | Human verification |
|---|---|---|
| Learn unfamiliar code | Ask for purpose, data flow, assumptions, and a small example | Compare the explanation with source and tests |
| Generate/refactor/document | State behavior to preserve; choose inline, Edit, or Agent mode by scope | Inspect the diff; build, lint, and run tests |
| Create sample data | State schema, boundary cases, size, and output format | Use synthetic—not real sensitive—data; validate constraints |
| Improve tests | Request unit/integration tests, success/failure paths, edge cases, and meaningful assertions | Run them; check that assertions test behavior instead of repeating implementation |
| Modernize legacy code | Explain first, map dependencies/data flow, create baseline tests, then migrate in small steps | Compare behavior before/after and keep rollback points |
| Improve security/performance | Ask for risks or optimization ideas with constraints | Confirm with code scanning, dependency/secret scanning, benchmarks, profiling, and expert review |
| Reduce context switching | Use Chat with relevant selected/repository context and reusable instructions | Do not trade speed for unreviewed output |

[Choosing the right AI tool](https://docs.github.com/en/copilot/concepts/tools/ai-tools) · [Writing tests with Copilot](https://docs.github.com/en/copilot/tutorials/write-tests) · [Modernizing legacy code](https://docs.github.com/en/copilot/tutorials/modernize-legacy-code)

---

# Part 4 — Context

## 🟢 Beginner

**What:** Context is information Copilot can use to understand the request.

**Why:** `Fix this` is meaningless without knowing what “this” is.

**Examples:** current file, selected code, nearby code, open files, referenced repository files, chat history, terminal output, custom instructions, issue/PR details.

**Remember:** Context is the AI's working material, not proof that the answer is correct.

## 🔵 Professional / GH-300

| Type | Meaning | Example |
|---|---|---|
| Explicit context | User deliberately names/attaches it | `#file:auth.ts`, `@README.md`, selected code |
| Implicit context | Client selects it automatically | current file, cursor location, conversation |
| Workspace/repository context | Indexed or searched project information | symbols and files in the codebase |
| Context window | Fixed token capacity for all material in a request/session | long logs can crowd out key requirements |
| Excluded context | Content blocked by a Business/Enterprise exclusion rule | `/secrets/**` |

- Too much irrelevant history consumes the context window and can lower answer quality.
- Repository context is constrained by access, supported indexing, current client, policy, and content exclusions.
- Custom instructions persist project/team conventions; a one-off prompt remains task-specific.
- **⚠️ Trap:** repository permission controls whether a person/service can access a repository. Content exclusion controls whether selected accessible content may be used by supported Copilot features.

## Current reference syntax by environment

| Environment | Current syntax/examples | Notes |
|---|---|---|
| VS Code | `#file`, `#selection`, `#codebase`, `#git`; `@workspace` | Type `#` or `@` to see what the installed version supports. `@workspace` is a current Chat participant for workspace questions. |
| Visual Studio | `#file`, file/line references, `#solution` | Exact suggestions appear in Chat context controls; version matters. |
| JetBrains | `@project` for project context; use the context picker/visible suggestions | Do not assume every VS Code variable exists. |
| Copilot CLI | `@path` to mention files/directories | Example: `Explain @src/auth.ts`. This is not `@workspace`. |
| GitHub.com | `@` mentions and attachments for repositories, files, issues, PRs, discussions, or extensions where supported | The current page also supplies implicit context. |
| LEGACY | Old tutorials may mix `@workspace`, `#workspace`, IDE participants, and the old `gh copilot` extension | Follow the current product's `/help`, context picker, and docs. |

[Copilot Chat cheat sheet](https://docs.github.com/en/copilot/reference/chat-cheat-sheet?tool=vscode) · [Copilot CLI quickstart](https://docs.github.com/en/copilot/get-started/cli-quickstart)

---

# Part 4A — Current blueprint features people overlook

These features appear in the current GH-300 blueprint but solve different problems:

| Feature | What it is | Best exam clue | Important limit/distinction |
|---|---|---|---|
| Custom instructions | Always-on guidance applied within a defined scope | coding standards, test framework, repository conventions | Repository-wide: `.github/copilot-instructions.md`; path-specific: `.github/instructions/*.instructions.md`; `AGENTS.md` can guide agents. Surface support varies. |
| Prompt files | Reusable, on-demand prompt templates, optionally with inputs | repeat the same task consistently | `.github/prompts/*.prompt.md`; currently public preview and available in VS Code, Visual Studio, and JetBrains IDEs. Invoke by its slash command, such as `/explain-code`. |
| Custom agents | Reusable specialist persona with selected instructions and tools | reviewer, documentation writer, security specialist | An agent configuration is not a prompt file and is not itself a runtime subagent. |
| MCP | Open standard for connecting Copilot to approved external tools and data | issue tracker, database, CI/CD tool, specialist docs | It broadens capability and trust boundaries; administrators can govern MCP access. It is not repository indexing or a custom instruction. |
| Copilot Spaces | Curated, shareable context made from repositories, code, PRs, issues, text, files, and images | reusable project knowledge or onboarding context | Anyone with a Copilot license, including Free, can use Spaces. GitHub sources stay synchronized. Space questions consume AI Credits; Free usage counts against its chat limit. |
| PR summary | AI-generated explanation of PR changes, affected files, and reviewer focus | help a reviewer understand “what changed and why” | Not available on Free. Review and edit it; it is **not** an approval, defect analysis, or replacement for code review. Existing PR-description text is not considered, so a blank field works best. |
| GitHub Spark | Natural-language app builder with live preview, managed storage/authentication, and one-click managed deployment | prototype/deploy a full-stack web app from a description | Currently in public preview and documented for **Copilot Pro+ and Copilot Enterprise**. Each Spark prompt consumes AI Credits. Test generated apps and protect sensitive data. |
| Agent session | A traceable unit of agent work with prompts, tool use, changes, and validation | monitor, steer, stop, resume, inspect, or share delegated work | Copilot app sessions can use isolated worktrees/cloud sandboxes. Cloud-agent sessions are visible to repository collaborators by default; local sessions are private by default unless shared. |
| Subagent | A separate runtime agent delegated a bounded task with its own context window | keep the main context focused or parallelize independent work | More agents mean more model calls and review work. Parallelism does not remove dependencies or human accountability. |

[Customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet) · [Prompt-file tutorial](https://docs.github.com/en/copilot/tutorials/customization-library/prompt-files/your-first-prompt-file) · [MCP concept](https://docs.github.com/en/copilot/concepts/context/mcp) · [Copilot Spaces](https://docs.github.com/en/copilot/concepts/context/spaces) · [PR summaries](https://docs.github.com/en/copilot/how-tos/copilot-on-github/copilot-for-github-tasks/create-a-pr-summary) · [GitHub Spark](https://docs.github.com/en/copilot/concepts/spark) · [Managing agent sessions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/manage-and-track-agents)

**Memory test:** instructions = automatic guidance; prompt file = reusable task; agent = specialist behavior; MCP = outside tools/data; Space = curated knowledge; PR summary = explanation; code review = findings; Spark = app building; session = one unit of agent work; subagent = delegated worker.

---

# Part 5 — GitHub Copilot CLI

## 🟢 Beginner

Copilot CLI is the current agentic terminal experience. Run `copilot`, ask a question, reference a file with `@`, and approve or deny requested tools. It can explain, plan, edit files, and run commands, so read every proposed action.

## Install, authenticate, launch

Choose one current installation method:

```powershell
# npm; requires Node.js 22+
npm install -g @github/copilot

# Windows Package Manager
winget install GitHub.Copilot

# Launch
copilot
```

```bash
# macOS Homebrew
brew install --cask copilot-cli

copilot
```

Inside the session, run `/login` if required and complete OAuth authentication. An organization/enterprise policy can disable CLI access. [Copilot CLI quickstart](https://docs.github.com/en/copilot/get-started/cli-quickstart) · [Installing Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)

## Current commands and symbols

| Command/symbol | Meaning | When to use | Tiny example |
|---|---|---|---|
| `@` | Mention file/directory context | Ground a request in local content | `Explain @src/auth.ts` |
| `# NUMBER` | Mention a GitHub issue or pull request | Add work-item context | `Explain #42` |
| `! COMMAND` | Execute a local shell command, bypassing Copilot | Run a command you explicitly chose | `! git status` |
| `$` | Temporarily hand the TTY to a real interactive shell | Need job control/full-screen shell behavior | enter `$`, then `exit` to return |
| `/` | Open slash-command menu | Discover session operations | type `/` |
| `?` | Open tabbed help | Learn keybindings/modes | type `?` |
| `/help` | Show help | Confirm current commands | `/help` |
| `/ask` | Ask without switching to an edit task | Explanation/advice | `/ask Why did this test fail?` |
| `/plan [PROMPT]` | Build a plan | Review approach before execution | `/plan migrate API to v2` |
| `/delegate [PROMPT]` or `& PROMPT` | Send work to cloud agent | Continue while a remote branch/PR task runs | `/delegate update docs` |
| `/fleet [PROMPT]` | Split suitable work among parallel subagents | Independent parts of a large task | `/fleet inspect all modules` |
| `/tasks` | View/manage subagents and shell tasks | Monitor, inspect, stop, or enter delegated work | `/tasks` |
| `/subagents` / `/agents` | Configure per-agent/default subagent models | Tune delegated-agent behavior | `/subagents` |
| `/model` | Select/inspect model | Balance capability and credits | `/model` |
| `/agent` | Manage/select agent behavior | Use a suitable configured agent | `/agent` |
| `/diff` | Inspect changes | Review before keeping work | `/diff` |
| `/permissions` | Inspect/manage tool permissions | Apply least privilege | `/permissions` |
| `/add-dir` | Add an allowed working directory | Work across an intentional extra path | `/add-dir ../shared` |
| `/mcp` | Inspect/manage MCP servers | Use approved external tools/data | `/mcp` |
| `/init` | Create/improve repository Copilot instructions | Initialize project guidance | `/init` |
| `/instructions` | View/toggle instruction files | Diagnose applied guidance | `/instructions` |
| `/sandbox` | Manage OS-level filesystem/network sandboxing | Limit tools and integrations | `/sandbox status` |
| `/settings` | Inspect/change user/repo/local CLI settings | Configure a documented setting/scope | `/settings` |
| `/review` / `/security-review` | Review local changes | Target general or security defects | `/review` |
| `/context` | Inspect context use | Diagnose a crowded session | `/context` |
| `/compact` | Summarize older context | Free space while preserving useful state | `/compact` |
| `/new` | Start a clean session | New unrelated task | `/new` |
| `/resume` | Resume a prior session | Continue earlier work | `/resume` |
| `/exit` | Leave | End session | `/exit` |
| `-p`, `--prompt` | Noninteractive prompt | Automation/single request | `copilot -p "Explain this repo"` |
| `-s` | Print response only | Reduce extra CLI metadata | `copilot -sp "Summarize @README.md"` |

Commands evolve. `/help` and the [current CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference) override memorized lists.

### Shell integration

- **Interactive:** launch `copilot`; it can propose and, with permission, execute shell tools in the working directory.
- `! command` runs a local shell command while bypassing Copilot. A lone `!` enters shell mode.
- A lone `$` hands a local trusted idle TTY to the real interactive shell; `exit` returns to Copilot. Enterprise managed settings can disable this shortcut.
- **Programmatic/noninteractive:** `-p` sends one prompt; `-s` (`--silent`) prints only the agent response without usage statistics. Treat generated commands/text as untrusted until reviewed.
- **LEGACY:** old tutorials may install a GitHub CLI extension and shell aliases around `gh copilot suggest` or `gh copilot explain`. That is not the current standalone CLI command set.

### Main agent, custom agent, and subagent

| Term | Meaning | Exam clue |
|---|---|---|
| Main agent | The primary agent handling the CLI conversation | Owns main context/history |
| Custom agent | A reusable profile defining specialist instructions/tools | reviewer, docs writer, security specialist |
| Subagent | A separate runtime agent delegated one bounded task; has its own context window | offload context, specialist work, parallelism |

The main agent may automatically delegate to a built-in/custom subagent. Separate context prevents exploration/test output from crowding the main conversation. `/fleet` explicitly asks for parallel subagent work; use it for independent parts, not tightly sequential work. More model calls can consume more AI Credits, and every result still needs review. [CLI customization comparison](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/comparing-cli-features) · [Running `/fleet`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)

## Permissions, trust, and security

- The CLI normally has access to the current working directory and its subtree plus temporary locations. Adding directories broadens scope.
- A trusted directory decision controls whether Copilot may operate there; trust only code you understand.
- On the first mutating tool request, choose the smallest approval: allow once, allow for session, or deny.
- `--allow-tool` and `--deny-tool` set rules; **deny takes precedence**. `--allow-all` is high risk.
- Use `--available-tools` to inspect tools. Do not grant broad shell/network access merely to avoid prompts.
- MCP extends tools/data. Approve servers, arguments, permissions, and secrets deliberately.
- For noninteractive authentication, current token precedence is `COPILOT_GITHUB_TOKEN`, then `GH_TOKEN`, then `GITHUB_TOKEN`. Classic `ghp_` PATs are not supported; a fine-grained PAT needs **Copilot Requests** permission. Prefer OAuth when practical.
- Business/Enterprise administrators can separately govern the CLI. [Configure Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/configure-copilot-cli)

### Content-exclusion currency warning

**CURRENT (2026-09-05):** Copilot CLI and the Copilot app respect Business/Enterprise content exclusions; this became generally available on September 2, 2026. [Current CLI documentation](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli) · [Official September 2 changelog](https://github.blog/changelog/2026-09-02-content-exclusions-generally-available-in-copilot-app-and-cli/)

**LEGACY / OLD QUESTIONS:** Earlier matrices and learning material may say the CLI/app did not support content exclusion. Use the newer, surface-specific documentation for current behavior.

### `@workspace`: exact answer

- **Current VS Code Copilot Chat:** yes, `@workspace` is a participant for workspace context.
- **Current Copilot CLI:** no, use `@file-or-directory`; there is no current CLI `@workspace` command in the official command reference.
- **Old material:** may call `@workspace` old Chat syntax. Do not therefore conclude that it is obsolete in VS Code.

**Exam trap:** “terminal, current standalone CLI, reference one file” → `@path`, not `@workspace`.

---

# Part 6 — IDE support

## 🟢 Beginner

An **IDE** (integrated development environment — môi trường phát triển tích hợp) is an application for writing, navigating, running, testing, and debugging code. Copilot capabilities are not identical in every IDE.

## Current feature matrix

This compact table uses the latest-version matrix on 2026-09-05. `✓` supported, `P` preview, `✗` unsupported. Vim and Neovim are grouped here because the official matrix lists NeoVim; completion support also exists for Vim/Neovim.

| Feature | VS Code | Visual Studio | JetBrains | Eclipse | Xcode | Vim/NeoVim |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Code completion | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Chat | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| Agent mode | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| Edit mode | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ |
| Next edit suggestions | ✓ | ✓ | P | P | P | ✗ |
| Code review | ✓ | ✓ | ✓ | ✗ | ✓ | ✗ |
| Code referencing | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |
| Custom instructions | ✓ | ✓ | P | P | P | ✗ |
| MCP | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| Workspace indexing | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |
| Cloud-agent integration | ✓ | ✓ | ✓ | ✓ | — | — |

The feature matrix itself is in **public preview** and can change. Use latest stable IDE/extension versions. The cloud-agent row is based on the current integration list rather than the IDE feature-matrix table. [Official feature matrix](https://docs.github.com/en/copilot/reference/copilot-feature-matrix) · [Using cloud agent from an IDE](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-in-your-ide)

### Content exclusion by surface

Content exclusion supports inline suggestions in VS Code, Visual Studio, JetBrains, Vim/Neovim, Xcode, and Eclipse. Chat support differs, and **IDE Agent mode does not support content exclusion**. Current Copilot CLI and Copilot app do support it; third-party agents and Spark do not. Recheck the [content-exclusion concept](https://docs.github.com/en/copilot/concepts/context/content-exclusion), [policy surface matrix](https://docs.github.com/en/copilot/reference/supported-surfaces-for-policies), and surface-specific documentation before deployment.

**⚠️ Trap:** “Copilot is supported in this IDE” does not imply every feature, policy, reference syntax, or shortcut is supported there.

---

# Part 7 — Copilot plans

## Current plan comparison

Prices and included credits below are current US list prices. Taxes and eligibility can differ. A dash means not applicable; **NC** means **Not confirmed in current official documentation**.

| Plan | Buyer / target | Price/month | AI Credits/month | Completions | Administration and key limits |
|---|---|---:|---:|---|---|
| Free | Individual trying Copilot | Free | Allowance exists; **number NC** | 2,000/month | Limited agents; auto model selection; no org policy, audit, or content exclusion |
| Student | Verified student | Free | Allowance exists; **number NC** | Unlimited | Auto model selection; cloud agent/agent mode/review/MCP; excludes third-party agents; no org governance |
| Pro | Individual developer | US$10 | Base 1,000 + flex 500 = **1,500** | Unlimited | Model selection, agents, review, CLI/app; individual controls only |
| Pro+ | Individual AI power user | US$39 | Base 3,900 + flex 3,100 = **7,000** | Unlimited | Pro plus premium models/higher allowance and Spark |
| Max | Sustained high-volume individual | US$100 | Base 10,000 + flex 10,000 = **20,000** | Unlimited | Highest individual allowance; priority premium-model access |
| Business | Organization/enterprise | US$19 per granted seat | **1,900 per user into shared pool** | Unlimited | Central policies, seats, audit, content exclusion, org custom instructions, agents/MCP |
| Enterprise | GHEC enterprise organizations | US$39 per granted seat | **3,900 per user into shared pool** | Unlimited | Everything in Business plus enterprise capabilities such as Spark, higher pool, priority premium-model access, and typically earlier features |

All current plans include Copilot CLI and the Copilot app. Copilot is not currently available for GitHub Enterprise Server (GHES). Code completions and next edit suggestions do not consume AI credits and remain unlimited on paid plans. [Plans for GitHub Copilot](https://docs.github.com/en/copilot/get-started/plans)

## 🟢 Individual vs organization plan

- **Individual:** you subscribe and control personal settings. Choose Free/Student/Pro/Pro+/Max based on eligibility and volume.
- **Organization:** an organization or enterprise grants your GitHub account a **seat**; its policies control your provided Copilot access.
- Working for a company does **not** automatically mean you personally buy Business. The company normally assigns the seat if it wants centralized governance.
- If you have an active paid individual plan and receive a Business/Enterprise seat, GitHub automatically cancels the individual plan; the organization's policies apply. [Viewing/changing a plan](https://docs.github.com/en/copilot/how-tos/manage-your-account/view-and-change-your-copilot-plan)

## Which plan fits this user?

| User/requirement | Most appropriate | Why |
|---|---|---|
| Eligible student learning programming | Student | Free student access and unlimited completions |
| Curious beginner | Free | No cost; enough to test, but 2,000 completions/month |
| Typical individual/freelancer | Pro | Paid individual baseline |
| Individual needing more premium use | Pro+ | 7,000 monthly credits |
| Very heavy individual AI user | Max | 20,000 monthly credits |
| Small organization needing central policies/content exclusion | Business | Lowest organization plan with centralized governance |
| Enterprise wanting Business features but not Enterprise extras | Business | Business can be assigned in an enterprise |
| GitHub Enterprise Cloud org needing larger pool/Spark/enterprise capabilities | Enterprise | Higher credits and enterprise-only additions |

“More expensive” is not automatically “more appropriate.” Choose the least plan that satisfies the explicit requirement.

## Business vs Enterprise: exam-focused

| Topic | Business | Enterprise |
|---|---|---|
| Price | US$19/granted seat/month | US$39/granted seat/month |
| Included pool contribution | 1,900 credits/user/month | 3,900 credits/user/month |
| Central policy management | Yes | Yes |
| Seats, usage, audit logs | Yes | Yes |
| Content exclusion | Yes | Yes |
| Cloud agent, agent mode, code review, MCP | Yes | Yes |
| Public-code policy | Yes | Yes |
| GitHub Enterprise Cloud required | Can serve organizations and dedicated enterprises | Yes |
| Distinctive value | Lower-cost organization governance | Larger pool, priority premium models, Spark, additional enterprise capabilities/earlier access |

**Correct outdated assumption:** content exclusion, organization policy management, and audit logs are **not Enterprise-only**; current Business includes them. [Current plan matrix](https://docs.github.com/en/copilot/get-started/plans) · [Choosing an enterprise Copilot plan](https://docs.github.com/en/copilot/tutorials/roll-out-at-scale/assign-licenses/choose-enterprise-plan)

---

# Part 8 — AI Credits and billing

## 🟢 Beginner

An **AI Credit** measures billable model use. One credit equals **US$0.01**. A short low-cost-model question may use less than a credit; a long agent session using a frontier model can use much more.

## What consumes credits?

| Consumes AI Credits | Does not consume AI Credits |
|---|---|
| Copilot Chat and agent-mode prompts | Code completions on paid plans |
| Copilot code review | Next edit suggestions on paid plans |
| Copilot CLI | Copilot Autofix suggestion for a code-scanning alert (not agentic autofix) |
| Copilot cloud-agent sessions | — |
| Copilot Spaces | — |
| Spark | — |
| Third-party coding agents | — |

The cost is derived from the selected model and the number/type of input, output, and cached tokens. [Organization/enterprise billing](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises/usage-based-billing) · [Copilot Autofix](https://docs.github.com/en/code-security/concepts/code-scanning/autofix-for-code-scanning)

## Pooling, reset, and overage

- Organization/enterprise licenses contribute credits to a shared **billing-entity pool**. Example: 100 Business seats contribute `100 × 1,900 = 190,000` credits.
- Adding licenses mid-cycle increases the pool immediately. Removing them reduces it at the next billing cycle.
- Included credits reset at **00:00:00 UTC on the first calendar day of each month** and do not roll over.
- After the pool is exhausted, usage continues at US$0.01/credit if additional paid usage is allowed and applicable limits permit it; otherwise credit-consuming features are blocked.
- **Additional usage is enabled by default** for organizations/enterprises. Disable the **AI credit paid usage** policy to prevent overage.
- Code completions and next edit suggestions continue even if an AI-credit budget blocks consuming features.
- The temporary June 1–September 1, 2026 promotional organization allowances have ended. Current standard amounts are 1,900 Business and 3,900 Enterprise.

[Usage-based billing for organizations and enterprises](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises/usage-based-billing)

## Budget controls

| Control | What it caps | When it acts |
|---|---|---|
| Universal user-level budget | Each licensed user's total credits | Pool and metered phases; hard stop |
| Cost-center user-level budget | Each user in that cost center | Overrides universal; hard stop |
| Individual user-level budget | One user's total credits | Most specific; overrides both; hard stop |
| Cost-center budget | Metered charges for a group | After pool exhaustion |
| Organization budget | Metered charges for org-billed seats | After pool exhaustion |
| Enterprise spending limit | Enterprise metered charges | After pool exhaustion |

For user-level budgets: `individual > cost center > universal`; the most specific applies, and US$0 blocks immediately. For organization/cost-center/enterprise spending budgets, **Stop usage when budget limit is reached is off by default**. A budget without that option stops nothing; it alerts while charges continue. [Budgets for usage-based billing](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing)

## CURRENT vs LEGACY billing

| CURRENT | LEGACY | Difference |
|---|---|---|
| AI Credits; token- and model-based | Premium requests; request/multiplier based | Current billing reflects tokens and model price |
| Base plus possible flex allotment for paid individuals | Fixed monthly premium-request allowance | Different unit; do not convert by guesswork |
| Started June 1, 2026 generally | Some existing annual Pro/Pro+ subscribers could remain until annual term ends | Legacy questions may still mention premium requests |

[What changed with Copilot billing (legacy)](https://docs.github.com/en/copilot/reference/copilot-billing/request-based-billing-legacy/what-changed-with-billing)

**Exam traps**

- **Allowance** = included amount, not unlimited paid use.
- **Pooled** does not mean each person owns an isolated 1,900/3,900 bucket.
- A spending budget without **Stop usage** is an alert, not a hard cap.
- No credits left does not disable paid-plan completions/NES.

---

# Part 9 — Responsible AI principles

## Two-level view

| Principle | Vietnamese | Simple definition | Copilot example | Exam keywords | Common confusion |
|---|---|---|---|---|---|
| Fairness | công bằng | Treat similar people/cases fairly; avoid harmful bias/discrimination | Test a hiring helper across demographic groups | bias, discrimination, unequal treatment | Inclusiveness |
| Reliability & Safety | đáng tin cậy và an toàn | Behave predictably and safely; validate correctness/security | Review, test, scan, and validate generated code | correct, test, validate, safe, robust | Accountability |
| Privacy & Security | riêng tư và bảo mật | Protect sensitive/confidential data and systems | Exclude `/secrets/**`; use least privilege | secret, private, confidential, exposure, leakage | Content safety |
| Inclusiveness | bao trùm | Work for people with different abilities/backgrounds | Accessible UI and keyboard workflow | accessibility, abilities, diverse needs | Fairness |
| Transparency | minh bạch | Make AI use, behavior, capabilities, and limits understandable | Label AI-generated review and explain limitations | explain, disclose, understandable, traceable | Accountability |
| Accountability | trách nhiệm/khả năng chịu trách nhiệm | A human/owner remains answerable for outcomes | Named reviewer owns the merge decision | oversight, owner, answerable, final decision | Reliability & Safety |

These are Microsoft's six Responsible AI principles. [Microsoft Responsible AI principles](https://learn.microsoft.com/en-us/azure/machine-learning/concept-responsible-ai) GitHub separately documents product limitations and requires users to review/validate output. [GitHub responsible use](https://docs.github.com/en/copilot/responsible-use/inline-suggestions)

## Critical distinction: Reliability vs Accountability

| Scenario | Correct principle | Why |
|---|---|---|
| Developer must **review/test/validate/scan** generated code | Reliability & Safety | The action checks whether output works safely |
| Human remains **responsible/answerable** for the final merge | Accountability | It identifies ownership of the outcome |
| Team creates mandatory tests and security gates | Reliability & Safety | Technical validation and safe operation |
| Team assigns a named approver for AI changes | Accountability | Human oversight and responsibility |
| Team tests code, and a named owner approves it | Both | Testing = reliability; named ownership = accountability |

### Quick scenarios

| Scenario | Principle |
|---|---|
| Secret folder must not be exposed to Copilot context | Privacy & Security |
| Users must understand that a review was AI-generated | Transparency |
| Avoid worse results for a protected group | Fairness |
| Support keyboard-only and screen-reader users | Inclusiveness |

---

# Part 10 — Processing lifecycle and filtering

## Memory flow

```text
INPUT → CONTEXT → PROMPT → FILTER/SERVICE → MODEL → POST-PROCESS → OUTPUT → HUMAN REVIEW
```

| Stage | What happens | Main risk/control |
|---|---|---|
| Input processing | User code/text/selection/task is received | Do not paste unnecessary secrets |
| Context gathering | Permitted current/referenced/repository context is selected | Permission and content-exclusion boundaries |
| Prompt construction | Instructions, context, history, and tool definitions are assembled | Relevant context; unambiguous constraints |
| Proxy/service processing | Service mediates the request and applies relevant controls | Policy/filter enforcement varies by surface |
| Content/safety filtering | Harmful, abusive, offensive, or inappropriate material may be filtered | Content safety, not secret-path exclusion |
| Model invocation | Model generates likely tokens | Hallucination, bias, insecure/outdated code |
| Public-code matching | Applicable products compare candidates against public GitHub code | Block or annotate/reference according to policy/support |
| Post-processing | Output is formatted and prepared for the client | References/filtering/presentation |
| Suggestion presentation | Ghost text, answer, diff, comment, or agent log appears | User should understand source/status |
| Developer review | Human accepts, rejects, modifies, tests, scans, and verifies | Reliability & Safety + Accountability |

## Four mechanisms that questions deliberately mix up

| Problem | Correct mechanism | What it is **not** |
|---|---|---|
| Harmful/offensive generated content | Content safety / toxicity filtering | Content exclusion |
| Generated suggestion resembles public code | Public-code matching / code referencing | Repository permission |
| Secret folder must not become supported Copilot context | Content exclusion | Toxicity filter |
| Developer cannot access a repository | Repository permissions | Content exclusion |
| Protect sensitive data generally | Privacy & Security principle | A single filter name |

### Definitions

- **Content filtering:** checks input/output for disallowed or inappropriate material according to product safety controls.
- **Toxicity/safety filtering:** focuses on **toxic** (độc hại), harmful (gây hại), offensive (xúc phạm), abusive (lạm dụng), or inappropriate (không phù hợp) content.
- **Public-code matching:** checks whether generated code resembles indexed public GitHub code; block or show references where supported.
- **Content exclusion:** an administrator names repository files/paths that supported Copilot surfaces must not use as context.
- **Repository permissions:** access control decides who/service may read or modify a repository.

**⚠️ Trap:** “filter” alone is insufficient. Look for the object: harmful language → safety; public source → matching; private path → exclusion; unauthorized user → permissions.

---

# Part 11 — Content exclusion

## 🟢 Beginner

**Content exclusion** means: “Tell supported GitHub Copilot features not to use selected files/folders as context.”

Example: developers may use Copilot in the repository while `/secrets/**` is excluded.

**Remember:** it protects **content used as Copilot context**; it does not remove repository access, encrypt a file, or replace secret scanning.

## 🔵 Professional / GH-300

| Question | Current answer |
|---|---|
| Plan | Copilot Business or Copilot Enterprise |
| Who configures | Repository administrator, organization owner, or enterprise owner at their scope; repository **Maintain** role can view but not edit repository rules |
| What happens in an excluded file | Inline suggestions are unavailable; content is not used for other inline suggestions or supported Chat responses and is not reviewed by Copilot code review |
| Inheritance | Enterprise, organization, and repository exclusions are additive; inherited rules appear but cannot be changed at a lower scope |
| Matching | Case-insensitive `fnmatch` path patterns |
| Propagation | Can take up to **30 minutes** in already loaded IDEs |
| Key limitations | IDE Edit/Agent modes do not support exclusion; symbolic links and repositories on remote filesystems are not supported; an IDE may indirectly expose semantic information such as type/hover/build data |

[Content exclusion concept](https://docs.github.com/en/copilot/concepts/context/content-exclusion) · [Configure content exclusion](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot)

## Where to configure it

| Scope | Exact current route | Who | Plan |
|---|---|---|---|
| Repository | Repository → **Settings** → under “Code, planning, and automation,” **Copilot** → **Content exclusion** | Repository admin | Business/Enterprise |
| Organization | Profile picture → **Organizations** → organization → **Settings** → **Copilot** → **Content exclusion** | Organization owner | Business/Enterprise |
| Enterprise | Enterprise → **AI controls** → **Copilot** → **Content exclusion** | Enterprise owner | Business/Enterprise |

UI labels can move; the [current exclusion how-to](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) is authoritative.

## Tiny syntax examples

Repository-level YAML:

```yaml
- "/secrets/**"
- "/config/production.yml"
```

Organization/enterprise mapping:

```yaml
"https://github.com/octo-org/payment-service":
  - "/secrets/**"
  - "/config/production.yml"
"*":
  - "/company-confidential/**"
```

`"*"` can define patterns across repository roots/non-Git repositories as documented. Use valid repository references and test patterns; do not assume `.gitignore` semantics are identical.

## Scope, inheritance, and precedence

```text
Enterprise exclusions
        +
Organization exclusions
        +
Repository exclusions
        =
Effective exclusions (additive)
```

- Enterprise rules apply to Copilot users governed by the enterprise.
- Organization rules apply to users whose Copilot seat is assigned by that organization.
- A repository admin cannot “unexclude” a path inherited from organization/enterprise scope.
- **⚠️ Trap:** precedence does not mean a lower level can override and allow excluded content. Exclusions accumulate.

## Supported and unsupported surfaces

| Surface | Current content-exclusion behavior |
|---|---|
| IDE inline suggestions | Supported in VS Code, Visual Studio, JetBrains, Vim/Neovim, Xcode, Eclipse |
| IDE Chat | Supported in VS Code, Visual Studio, and JetBrains; not supported in Xcode or Eclipse |
| IDE Edit mode | **Not supported** |
| IDE Agent mode | **Not supported** |
| GitHub.com Chat / GitHub Mobile | Supported in public preview |
| Copilot code review | Supported; excluded files are not reviewed |
| Copilot cloud agent | Supported in current policy surface documentation |
| Copilot CLI | Supported as of 2026-09-02 |
| Copilot app | Supported as of 2026-09-02 |
| Third-party coding agents | Not supported |
| Spark | Not supported |

The current rendered policy table may lose icon accessibility in some text views; verify the live [supported-surfaces table](https://docs.github.com/en/copilot/reference/supported-surfaces-for-policies) and [content-exclusion documentation](https://docs.github.com/en/copilot/concepts/context/content-exclusion). Surface-specific newer documentation wins over an older matrix.

## Propagation and refresh

“Up to 30 minutes” means an already running editor can temporarily keep the earlier setting. After changing a rule:

| Client | Refresh current settings |
|---|---|
| VS Code | Command Palette → **Developer: Reload Window** |
| Visual Studio | Close and reopen the application |
| JetBrains | Close and reopen the application |
| Vim/Neovim | Settings are fetched whenever a file is opened; close/reopen the file |
| Xcode | Exact manual refresh action: **Not confirmed in current official documentation** |
| Eclipse | Exact manual refresh action: **Not confirmed in current official documentation** |

Wait up to 30 minutes before treating unchanged behavior as a policy failure. [Official testing/propagation guidance](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot#testing-changes-to-content-exclusions)

## Practical verification test

1. Confirm the user has the intended Business/Enterprise seat and the rule is at the correct scope.
2. In a normal file, type a clear comment and confirm an inline suggestion can appear.
3. Open an excluded file and try the same action. No inline suggestion should be offered.
4. In supported Chat, attempt to attach/reference only the excluded file or ask Copilot to explain it. Excluded content should not be available as context/reference.
5. Check inherited rules and pattern spelling/case-insensitive match.
6. If just changed, refresh the client as above and allow up to 30 minutes.
7. Retest. Do not use a surface that ignores exclusions—especially IDE Agent mode—as proof that the rule failed.

### Required secret-folder scenario

Repository: `/src`, `/docs`, `/secrets`, `/config`; `/secrets` is confidential.

- **Feature:** Content exclusion with `- "/secrets/**"`.
- **Principle:** Privacy & Security, because the goal is to prevent sensitive context exposure—not primarily Reliability & Safety, which concerns validating safe/correct output.
- **Where/who:** repository Settings → Copilot → Content exclusion; repository administrator.
- **Plan:** Business or Enterprise.
- **Propagation:** up to 30 minutes in already-loaded IDEs; reload/reopen as documented.
- **Verify:** normal-file suggestion works; excluded-file suggestion/context does not.
- **Unsupported warning:** IDE Edit/Agent modes, third-party agents, and Spark do not honor this control. Keep repository permissions and secret-management controls too.

---

# Part 12 — Public-code matching, code referencing, and IP

## 🟢 Beginner

Generated code can resemble code in public repositories because common problems often have common solutions and models learned patterns from large datasets. Copilot can check a candidate and nearby code against indexed public GitHub code.

| Policy/result | Behavior in supported products |
|---|---|
| **Block** matching suggestions | Match/near-match is not displayed |
| **Allow** matching suggestions | Suggestion can be displayed; references can show source repository and detected license |
| Product does not support Block | Matching code may be displayed with references/annotations where supported |

The matching process compares a candidate plus about **150 surrounding characters** to an index of public GitHub repositories. Inline code referencing runs for accepted, unmodified suggestions—not code you wrote or altered. The index refreshes periodically, so references are helpful evidence, not a complete legal guarantee. [GitHub Copilot code referencing](https://docs.github.com/en/copilot/concepts/completions/code-referencing)

## 🔵 Professional / GH-300

- Individual route: profile picture → **Copilot settings** → **Suggestions matching public code** → Allow/Block.
- With an organization/enterprise seat, the providing account's policy governs this setting; the user cannot override it personally.
- Enterprise route: enterprise → **AI controls** → **Copilot** → privacy policy; Business defaults matching suggestions to **Blocked**.
- Code references can provide repository URLs and detected license information. Review source, license, attribution, compatibility, and organizational policy.
- The Block/Allow public-code policy applies in supported IDEs. Cloud agent and third-party agents support it only in **annotate mode**. The policy does **not** apply to current Copilot CLI, the Copilot app, Copilot Chat in GitHub, Copilot code review, or Spark; the app may therefore generate a match even when Block is set. Do not assume one policy applies to every surface. [Supported surfaces for policies](https://docs.github.com/en/copilot/reference/supported-surfaces-for-policies) · [Copilot app public-code behavior](https://docs.github.com/en/copilot/concepts/agents/github-copilot-app#public-code)

## Intellectual-property words

| English | Vietnamese | Practical link |
|---|---|---|
| intellectual property (IP) | sở hữu trí tuệ | creations protected by legal rights |
| intellectual property rights | quyền sở hữu trí tuệ | rights held by creators/owners |
| violate/infringe | vi phạm | use that breaches rights/terms |
| copyright | bản quyền | legal protection for original expression/code |
| license | giấy phép sử dụng | terms that permit/restrict use |
| attribution | ghi nguồn | credit required/appropriate for a source |

Code referencing supports investigation; it does not decide whether a use is legally safe. Ask legal/compliance specialists for high-stakes decisions.

## Never confuse these

| Public-code matching | Content exclusion | Content safety |
|---|---|---|
| Examines generated code for similarity to public code | Prevents named repository content becoming context on supported surfaces | Filters harmful/offensive content |
| IP/license/reference concern | Privacy/confidential-context concern | Safety/toxicity concern |
| Allow/Block/annotate | Path patterns | Safety classifiers/policies |

---

# Part 13 — Policies and governance

## 🟢 Beginner

- **Policy** (chính sách): a rule controlling whether/how Copilot may behave.
- **Governance** (quản trị): how an organization centrally assigns access, controls features, manages risk/cost, and monitors use.
- **Enforce** (bắt buộc áp dụng): make a rule mandatory.
- **Inherit** (kế thừa): receive a rule from a higher scope.
- **Precedence** (mức ưu tiên): which applicable rule wins.

## 🔵 Professional / GH-300

```text
Enterprise policy (enforce or delegate)
              ↓
Organization policy (only when allowed)
              ↓
User with seat from that organization
              ↓
Supported Copilot surface
```

- Enterprise owners can enforce a policy or leave **No policy** so organizations decide. An organization cannot override an explicitly enforced enterprise setting.
- A user's effective features normally follow the organization/enterprise providing the seat.
- When a user receives seats from multiple organizations, conflict rules are feature-specific—not one universal “most restrictive” rule. For example, suggestions matching public code uses the **most restrictive** organization; many availability features use the **least restrictive** organization.
- Across multiple enterprises, the outcome is generally the most restrictive, but use the official conflict table for the named feature.
- CLI and Copilot app availability have independent policies; enabling one does not necessarily enable the other.
- **Scope** (phạm vi) asks which account/repository/users the rule affects.

[Copilot policies](https://docs.github.com/en/copilot/concepts/policies) · [Policy conflict table](https://docs.github.com/en/copilot/reference/enterprise-administrators/policy-conflicts)

## Policy routes and roles

| Task | Role | Exact current route | Plan |
|---|---|---|---|
| Organization feature/privacy policies | Organization owner | Profile → Organizations → org → **Settings** → under “Code, planning, and automation,” **Copilot** → **Policies** | Business/Enterprise |
| Organization model availability | Organization owner | Same path → **Copilot** → **Models** | Business/Enterprise |
| Enterprise agents policies | Enterprise owner or suitable enterprise custom permission | Enterprise → **AI controls** → **Agents** | Business/Enterprise |
| Enterprise Copilot administration/privacy/models/billing/usage | Enterprise owner or suitable custom permission | Enterprise → **AI controls** → **Copilot** | Business/Enterprise |
| Enterprise feature/client policies | Enterprise owner or suitable custom permission | Enterprise → **AI controls** → **Copilot** → **Configure features & clients** | Business/Enterprise |
| Enterprise MCP policy | Enterprise owner or suitable custom permission | Enterprise → **AI controls** → **MCP** | Business/Enterprise |

[Organization policies](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-policies) · [Enterprise policies](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-enterprise-policies)

### Managing subscriptions with the REST API

The current Copilot user-management REST endpoints are in **public preview**. Organization owners can retrieve subscription/seat information, list or inspect seat assignments, and add/remove selected users or teams for Business or Enterprise. Use the current API version and a token with the documented least privilege; write operations need suitable Copilot Business or organization-administration write permission. Removing a directly assigned user's seat sets it to pending cancellation, but access can remain through a team. [Copilot user-management REST API](https://docs.github.com/en/rest/copilot/copilot-user-management?apiVersion=2026-03-10)

**⚠️ Trap:** the API's seat/activity data supports subscription management. It is not an audit-log replacement and does not reproduce prompts or accepted code.

## How to verify a policy is applied

1. Confirm **scope and role**: personal, repository, organization, or enterprise; correct administrator.
2. Confirm the user has a Copilot seat from the expected provider.
3. Inspect the higher-level policy and whether the lower scope inherits an enforced setting.
4. Check the named policy's supported surfaces; do not test in an unsupported client/mode.
5. Allow documented propagation and reload/update the IDE/extension.
6. Perform a small behavior test with that user.
7. Search the audit log for the administrative change. Audit proves the change event, while the behavior test proves the client experience.

**Scenario:** changed five minutes ago, old behavior remains → consider propagation, stale IDE session, outdated extension, wrong seat/provider, inherited policy, pattern error, or unsupported surface.

**Scenario:** User A differs from User B → compare seat assignment/provider, enterprise/org membership, policy conflict resolution, client/version, repository access, and whether both test the same surface.

---

# Part 14 — Audit logs

## 🟢 Beginner

An **audit log** is a history of important administrative, security, policy, license, and GitHub-hosted agent events. It is not a recording of every local keystroke or prompt.

## What is and is not there?

| Event/data | Audit log? | Why |
|---|:---:|---|
| Administrator Copilot policy/setting change | Yes | Administrative event |
| Seat/license assignment or removal | Yes | License event |
| Enterprise/organization Copilot setting activity | Yes | Governance event |
| Copilot cloud-agent activity on GitHub | Yes | GitHub-hosted agent event; inspect agent/session events/logs |
| Content-exclusion administrative change | Yes: `copilot.content_exclusion_changed` | Includes actor/time and `excluded_paths` details |
| Accepted-line **counts** | No, not audit; usage metrics | Adoption/acceptance aggregate |
| Exact accepted generated source line | No standard Copilot audit record | Audit is not source-content replay |
| Ordinary inline suggestion | No | Local product interaction, not admin event |
| Local prompt/session data | No enterprise audit record | Audit docs explicitly exclude client session data |
| Ordinary developer keystrokes | No | Not audited Copilot governance event |
| Active users/chat request counts | No, usage metrics | Measurement, not audit history |
| AI-credit consumption/cost | No, AI usage/billing | Financial usage data |

[Reviewing Copilot enterprise audit logs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/review-audit-logs) · [Copilot usage metrics](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics)

## Routes, access, search, retention

| Scope | Who | Route | Useful filter | Retention |
|---|---|---|---|---:|
| Enterprise | Enterprise owner or custom role with **Read enterprise audit logs** | Enterprise → **Settings** → **Audit log** | `action:copilot` | 180 days |
| Organization | Organization owner | Organization → **Settings** → under “Archive,” **Logs** → **Audit log** | `action:copilot` | 180 days |

Examples:

```text
action:copilot
action:copilot.cfb_seat_assignment_created
action:copilot.content_exclusion_changed
actor:Copilot
```

For exclusions, the settings page shows who last changed them; clicking the time opens matching organization audit entries and can show the saved `excluded_paths`. [Reviewing exclusion changes](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/review-changes)

Use the UI suggestions/current [agentic audit-event reference](https://docs.github.com/en/copilot/reference/enterprise-administrators/agentic-audit-log-events) for exact agent action fields; they evolve. Stream/export audit data to a SIEM or storage if policy needs retention beyond GitHub's period. [Enterprise Copilot audit logs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/review-audit-logs) · [Organization audit log](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization)

**⚠️ Trap:** `actor:Copilot` finds actions attributed to Copilot; `action:copilot` finds the Copilot action category. They answer different searches.

---

# Part 15 — Audit vs metrics vs billing vs history

| Question | Correct source | What it tells you |
|---|---|---|
| Who changed a policy? | Audit log | Actor, action, time, scope metadata |
| How many users use Copilot? | Usage metrics/dashboard | DAU/WAU/adoption and feature counts |
| How many suggestions were accepted? | Usage/code-generation metrics | Aggregate acceptance activity/lines, not exact code text |
| How many AI Credits were consumed? | Billing & Licensing → AI usage | Credits, models/features, spend |
| What did this GitHub-hosted agent do? | Agent session log + relevant audit events | Tools, reasoning/activity, changes |
| What exact line did a user accept locally? | None of the standard governance views | Not provided as an audit replay |
| What did I ask in a recent GitHub.com Chat? | Personal Chat history | Up to 100 conversations; current messages retained 28 days |

The usage dashboard includes suggestions offered/accepted, acceptance rate, active users, Chat requests, models/modes, and code-generation metrics. Dashboard data can lag by **up to three full UTC days**. Its dashboards do not include Copilot CLI usage, while exports/APIs can expose a `used_cli` indicator—read the requested data source carefully. [Usage dashboard](https://docs.github.com/en/copilot/how-tos/administer-copilot/view-usage-and-adoption) · [Metrics reference](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics) · [GitHub.com Chat history](https://docs.github.com/en/copilot/how-tos/copilot-on-github/chat-with-copilot/chat-in-github)

---

# Part 16 — Copilot cloud agent

## 🟢 Beginner

Copilot cloud agent can research a repository, make a plan, change a branch, and optionally open a pull request while you do other work. You review the diff and decide whether to merge.

## 🔵 Professional / GH-300

- **Plan:** all paid Copilot plans; unavailable for repositories owned by managed user accounts or where explicitly disabled.
- **Where it runs:** an ephemeral development environment powered by GitHub Actions—not your local IDE.
- **Starts from:** the Agents UI/Chat, issue assignment, IDE integration, API, GitHub CLI, GitHub MCP Server, Mobile, or supported integrations.
- **Repository workflow:** researches/plans, creates or uses an agent branch, commits/pushes changes, and can open a pull request. Session logs provide transparency.
- **Human control:** review the session log, diff, checks, code review, and tests. Apply branch protection, rulesets, required reviews, and CODEOWNERS.
- **Security:** the agent can write code and is vulnerable to prompt injection; use least privilege, review configuration changes, and avoid unnecessary network/secrets.
- **Network:** a default firewall restricts Internet access during task execution; administrators can configure it. Firewall behavior differs for setup steps and MCP connections.
- **Secrets:** it cannot read Actions, Codespaces, or Dependabot secrets. Configure dedicated **Agents** secrets/variables at repository or organization scope; values are passed as environment variables and secret values are masked in logs.
- **MCP:** repository admins can configure approved MCP servers; use `COPILOT_MCP_`-prefixed Agents secrets/variables for MCP-only credentials.
- **Instructions:** repository/organization custom instructions can guide conventions; protect instruction/setup/MCP files with rulesets/CODEOWNERS.
- **Cost:** each cloud-agent session consumes AI Credits.

[About cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent) · [Risks and mitigations](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations) · [Agent secrets and variables](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables)

## Agent comparison

| Feature | Chat / Ask | IDE Agent mode | Copilot cloud agent | Copilot CLI |
|---|---|---|---|---|
| Runs | IDE/GitHub Chat UI | Local IDE/workspace | GitHub Actions-powered cloud environment | Local terminal/worktree |
| Interaction | Conversational | Interactive multi-step | Asynchronous/delegated | Interactive or noninteractive |
| Edits files | Not in Ask; Chat may propose | Yes | Yes, on branch | Yes, with permission |
| Runs commands/tools | Limited by mode/surface | Yes, with controls | Yes in agent environment | Yes, with approvals/rules |
| Works asynchronously | No | No | **Yes** | Normally no |
| Creates PR | May explain how | Not the defining feature | **Yes/optional** | Can via tools if authorized, but not defining behavior |
| Access boundary | Supplied/permitted context | Local workspace + permissions | Enabled GitHub repository + configured resources | Trusted/allowed paths and tools |
| Content exclusion | Supported Chat surfaces only | **Not in IDE Agent mode** | Current support | Current support |
| Human review | Always | Always | Always before merge | Always |

**Exam keyword:** `GitHub issue + background/asynchronous + branch/PR` → Copilot cloud agent. `local IDE + commands + iterative edits` → Agent mode.

---

# Part 17 — GitHub Code Security (GHCS)

**CURRENT meaning here:** GHCS means **GitHub Code Security**, one of GitHub's Advanced Security products. It is not shorthand for Copilot Chat.

For GH-300, know only the boundary:

- GHCS includes vulnerability-focused capabilities such as code scanning, CodeQL CLI, Copilot Autofix, AI-powered security detections, premium Dependabot capabilities, dependency review, security campaigns, and security overview.
- Code scanning finds vulnerabilities/coding errors. **Copilot Autofix** can propose a targeted fix; a human must review it.
- Standard Copilot Autofix for a code-scanning alert does not require a Copilot subscription and does not consume AI Credits. **Agentic autofix** uses cloud agent, is billed as an agent session, and consumes credits.
- Secret exposure is also a privacy/security problem, but GitHub **Secret Protection** is a separate Advanced Security product. Content exclusion is not secret scanning.
- Generated code can still be insecure. Use tests, code review, dependency checks, code scanning, secret scanning/protection, and human judgment.

[GitHub security features](https://docs.github.com/en/code-security/getting-started/github-security-features) · [Copilot Autofix for code scanning](https://docs.github.com/en/code-security/concepts/code-scanning/autofix-for-code-scanning)

**⚠️ Trap:** Copilot code review is probabilistic review assistance. CodeQL/code scanning is security analysis. Neither makes human validation unnecessary.

---

# Part 18 — 32 GH-300-style case studies

These are original scenarios, not exam dumps. The answer follows current documentation cited in the relevant chapters.

## 1. Secret folder in one repository

**Situation:** `/src`, `/docs`, `/secrets`, and `/config` exist; Copilot should work except for `/secrets/**`.  
**Primary concern:** Confidential context exposure.  
**Correct answer:** Repository content exclusion `- "/secrets/**"`; Privacy & Security.  
**Why:** It removes the named path from supported Copilot context without disabling Copilot elsewhere.  
**Closest wrong answer:** Reliability & Safety.  
**Why that is wrong:** Testing generated output does not prevent secret content from entering context.  
**Where configured:** Repository Settings → Copilot → Content exclusion; repository admin.  
**Plan:** Business or Enterprise; allow up to 30 minutes, refresh, test normal and excluded files. IDE Edit/Agent modes, Xcode/Eclipse Chat, Azure Data Studio, third-party agents, and Spark do not honor it.  
**Exam keywords:** `secret`, `confidential`, `exclude path`, `up to 30 minutes`.

## 2. Customer data and credentials across repositories

**Situation:** Every repository in one organization uses `/private/customer/**` and `/private/credentials/**`.  
**Primary concern:** Consistent sensitive-path protection.  
**Correct answer:** Organization-level content exclusions; Privacy & Security.  
**Why:** One organization rule covers the relevant repositories/users and is inherited.  
**Closest wrong answer:** Toxicity filter.  
**Why that is wrong:** Credentials are sensitive data, not offensive content.  
**Where configured:** Organization Settings → Copilot → Content exclusion; organization owner.  
**Plan:** Business or Enterprise.  
**Exam keywords:** `all repositories in organization`, `credentials`, `common path`.

## 3. Harmful generated prose

**Situation:** A generated comment may be abusive, offensive, toxic, or inappropriate.  
**Primary concern:** Harmful output.  
**Correct answer:** Content safety/toxicity filtering.  
**Why:** It targets unsafe/offensive content.  
**Closest wrong answer:** Content exclusion.  
**Why that is wrong:** Exclusion removes named repository material from context; it is not a toxicity classifier.  
**Where configured:** Product safety system/applicable policies; no path rule.  
**Plan:** Product safety behavior, not a reason to buy Business solely for path exclusion.  
**Exam keywords:** `harmful`, `offensive`, `abusive`, `toxic`.

## 4. Block public-code matches

**Situation:** An organization does not want IDE suggestions that match public GitHub code.  
**Primary concern:** Public-source/IP risk.  
**Correct answer:** Set **Suggestions matching public code** to Block and verify the intended surface supports Block.  
**Why:** Matching/near-matching candidates are suppressed in supported products.  
**Closest wrong answer:** Content exclusion.  
**Why that is wrong:** Exclusion governs private repository context, not generated-output similarity.  
**Where configured:** Organization Settings → Copilot → Policies; organization owner.  
**Plan:** Business/Enterprise (individual subscribers can set personal Allow/Block).  
**Exam keywords:** `resembles public code`, `block match`.

## 5. Allow and investigate public code

**Situation:** A developer may use matching suggestions but must inspect source and license.  
**Primary concern:** Attribution and licensing.  
**Correct answer:** Allow matching suggestions and use code references.  
**Why:** References identify matching repositories and detected license information where supported.  
**Closest wrong answer:** Toxicity filtering.  
**Why that is wrong:** Toxicity has no relationship to copyright/license provenance.  
**Where configured:** Personal Copilot settings or provider organization/enterprise policy.  
**Plan:** Depends on provider; individual plans expose personal public-code setting.  
**Exam keywords:** `source`, `license`, `attribution`, `public repository`.

## 6. Intellectual-property risk reduction

**Situation:** Legal wants fewer accidental uses of code that might violate IP rights.  
**Primary concern:** Similarity, provenance, license review.  
**Correct answer:** Public-code matching policy plus code referencing and human/legal review.  
**Why:** It can block or identify public matches; references support a license/attribution decision.  
**Closest wrong answer:** Content safety filtering.  
**Why that is wrong:** Offensive-content controls do not establish source or license.  
**Where configured:** Copilot privacy/policy settings.  
**Plan:** Individual setting or centrally controlled Business/Enterprise policy.  
**Exam keywords:** `IP`, `copyright`, `license`, `violate`, `attribution`.

## 7. Verify an administrator's policy change

**Situation:** One admin changed the organization's Copilot policy; another wants evidence of who changed it.  
**Primary concern:** Administrative history.  
**Correct answer:** Organization audit log; filter `action:copilot`.  
**Why:** Audit records governance actions and actor/time metadata.  
**Closest wrong answer:** Usage metrics.  
**Why that is wrong:** Metrics measure adoption/activity, not who changed a setting.  
**Where configured:** Organization Settings → Logs → Audit log; organization owner.  
**Plan:** Business/Enterprise for Copilot governance events.  
**Exam keywords:** `who changed`, `administrator`, `history`.

## 8. Request exact accepted code

**Situation:** An auditor asks for the exact source line a developer accepted from ghost text.  
**Primary concern:** Evidence boundary.  
**Correct answer:** Standard Copilot audit logs/metrics do not provide exact accepted source text.  
**Why:** Metrics provide counts/lines-of-code activity; audit logs cover administrative/agent events, not local code replay.  
**Closest wrong answer:** Search `action:copilot`.  
**Why that is wrong:** That cannot reconstruct every local completion.  
**Where configured:** No standard view supplies it; inspect repository history only if the accepted code was committed, without assuming provenance.  
**Plan:** Any.  
**Exam keywords:** `exact line`, `accepted completion`, `keystroke`.

## 9. Exclusion changed five minutes ago

**Situation:** An admin updates exclusion; a developer immediately still gets old behavior.  
**Primary concern:** Propagation and stale client state.  
**Correct answer:** Confirm scope/pattern/seat/surface, reload or reopen, and allow up to 30 minutes.  
**Why:** Official docs warn that already-loaded IDEs can take up to 30 minutes.  
**Closest wrong answer:** Declare the policy unsupported immediately.  
**Why that is wrong:** Five minutes is inside the documented window.  
**Where configured:** Content exclusion at the chosen scope; IDE refresh.  
**Plan:** Business/Enterprise.  
**Exam keywords:** `immediately`, `old behavior`, `30 minutes`, `reload`.

## 10. Exclude one folder in one repository

**Situation:** Only `repo-A/generated/vendor/**` must be excluded.  
**Primary concern:** Smallest correct scope.  
**Correct answer:** Repository-level exclusion.  
**Why:** It changes only repo A.  
**Closest wrong answer:** Enterprise exclusion.  
**Why that is wrong:** It is needlessly broad.  
**Where configured:** Repo A Settings → Copilot → Content exclusion; repository admin.  
**Plan:** Business/Enterprise.  
**Exam keywords:** `one folder`, `one repository`, `least scope`.

## 11. Common exclusion throughout an organization

**Situation:** Fifty organization repositories use the same confidential path.  
**Primary concern:** Consistent central rule.  
**Correct answer:** Organization-level exclusion.  
**Why:** Avoids fifty separately maintained rules and applies to seats assigned by the organization.  
**Closest wrong answer:** Copy a repository rule fifty times.  
**Why that is wrong:** Possible but not the most appropriate centralized control.  
**Where configured:** Organization Settings → Copilot → Content exclusion; organization owner.  
**Plan:** Business/Enterprise.  
**Exam keywords:** `organization-wide`, `common rule`, `inherit`.

## 12. Enterprise enforces a feature

**Situation:** Enterprise security must disable a Copilot feature in all member organizations.  
**Primary concern:** Central enforcement.  
**Correct answer:** Enterprise AI controls; explicitly disable the policy.  
**Why:** Organizations cannot override an explicit enterprise setting.  
**Closest wrong answer:** Set enterprise to **No policy**.  
**Why that is wrong:** That delegates the choice to organizations.  
**Where configured:** Enterprise → AI controls → relevant Agents/Copilot/Features & clients page.  
**Plan:** Business or Enterprise licenses in the enterprise, depending deployment.  
**Exam keywords:** `all organizations`, `enforce`, `cannot override`.

## 13. Verified student

**Situation:** An eligible student wants unlimited completions at no monthly subscription charge.  
**Primary concern:** Cheapest eligible plan.  
**Correct answer:** Copilot Student.  
**Why:** It is free for verified students and has unlimited completions.  
**Closest wrong answer:** Pro.  
**Why that is wrong:** It costs US$10 when Student satisfies the stated need.  
**Where configured:** Individual GitHub Copilot enrollment.  
**Plan:** Student.  
**Exam keywords:** `verified student`, `free`, `unlimited completions`.

## 14. Typical freelancer

**Situation:** One developer wants paid individual Copilot and moderate AI usage without organization governance.  
**Primary concern:** Personal value.  
**Correct answer:** Pro.  
**Why:** It is the US$10 individual baseline with 1,500 total monthly credits and unlimited completions.  
**Closest wrong answer:** Business.  
**Why that is wrong:** Business is licensed by organizations for central management, not required because the person works commercially.  
**Where configured:** Personal billing/Copilot settings.  
**Plan:** Pro.  
**Exam keywords:** `individual`, `freelancer`, `no central policy`.

## 15. Heavy individual use

**Situation:** A solo developer repeatedly exhausts Pro+ credits and wants the largest included individual allowance.  
**Primary concern:** Individual volume.  
**Correct answer:** Max.  
**Why:** Current Max includes 20,000 monthly AI Credits, the highest individual amount.  
**Closest wrong answer:** Enterprise.  
**Why that is wrong:** It requires an enterprise/GHEC organization deployment and is not a personal high-volume tier.  
**Where configured:** Personal plan settings.  
**Plan:** Max.  
**Exam keywords:** `highest individual allowance`, `sustained high volume`.

## 16. Small company needs centralized controls

**Situation:** A ten-person company needs seats, central policies, audit, and content exclusion.  
**Primary concern:** Organization governance at lowest price.  
**Correct answer:** Business.  
**Why:** Current Business includes all named governance features at US$19/granted seat.  
**Closest wrong answer:** Enterprise.  
**Why that is wrong:** It costs more and no Enterprise-only need is stated.  
**Where configured:** Organization Copilot Settings and audit log.  
**Plan:** Business.  
**Exam keywords:** `small organization`, `central policy`, `content exclusion`.

## 17. Enterprise needs larger pool and Spark

**Situation:** A GHEC enterprise explicitly needs Spark plus 3,900 credits per seat in the shared pool.  
**Primary concern:** Enterprise-only additions/allowance.  
**Correct answer:** Copilot Enterprise.  
**Why:** It includes Business capabilities plus Spark, higher pool contribution, and priority premium-model access.  
**Closest wrong answer:** Business.  
**Why that is wrong:** It contributes 1,900 credits/seat and does not include Spark in the current plan matrix.  
**Where configured:** Enterprise licensing and AI controls.  
**Plan:** Enterprise; GitHub Enterprise Cloud required.  
**Exam keywords:** `GHEC`, `Spark`, `3,900`.

## 18. Validate generated code before merge

**Situation:** Policy says developers must review, test, validate, scan, and verify Copilot code.  
**Primary concern:** Correct and safe operation.  
**Correct answer:** Reliability & Safety.  
**Why:** Those verbs describe technical validation.  
**Closest wrong answer:** Accountability.  
**Why that is wrong:** Accountability is who remains answerable, not the validation method.  
**Where configured:** Engineering standards, branch/ruleset/CI controls, instructions; not one Responsible-AI toggle.  
**Plan:** Any.  
**Exam keywords:** `review`, `test`, `validate`, `scan`, `verify`.

## 19. Named human owns the decision

**Situation:** A senior engineer must approve and remains responsible for every AI-assisted merge.  
**Primary concern:** Answerability and oversight.  
**Correct answer:** Accountability.  
**Why:** A human owner bears final responsibility.  
**Closest wrong answer:** Reliability & Safety.  
**Why that is wrong:** No test/validation property is the primary wording.  
**Where configured:** Review ownership/process; CODEOWNERS/rulesets can enforce workflow.  
**Plan:** Any.  
**Exam keywords:** `human remains responsible`, `owner`, `answerable`, `oversight`.

## 20. Test plus approve

**Situation:** CI tests AI code, then a named maintainer gives final approval.  
**Primary concern:** Two responsibilities.  
**Correct answer:** Reliability & Safety **and** Accountability.  
**Why:** CI validation addresses reliability; named approval addresses answerability.  
**Closest wrong answer:** Privacy & Security alone.  
**Why that is wrong:** No sensitive-data boundary is described.  
**Where configured:** CI/rulesets/review process.  
**Plan:** Any.  
**Exam keywords:** `tests` + `final owner`.

## 21. Biased recommendation

**Situation:** A generated screening function disadvantages candidates from one group.  
**Primary concern:** Unequal treatment.  
**Correct answer:** Fairness.  
**Why:** Bias/discrimination is the strongest signal.  
**Closest wrong answer:** Inclusiveness.  
**Why that is wrong:** Inclusiveness emphasizes supporting varied users/abilities; unfair group outcomes point directly to fairness.  
**Where configured:** Dataset/design/testing/governance; no single Copilot setting guarantees fairness.  
**Plan:** Any.  
**Exam keywords:** `bias`, `discrimination`, `group`, `unequal`.

## 22. Accessible Copilot workflow

**Situation:** A tool must support keyboard-only and screen-reader users.  
**Primary concern:** Different abilities.  
**Correct answer:** Inclusiveness.  
**Why:** The system should serve a broad range of users.  
**Closest wrong answer:** Fairness.  
**Why that is wrong:** Fairness is nearby, but explicit accessibility/abilities most strongly signals inclusiveness.  
**Where configured:** Product/workflow accessibility.  
**Plan:** Any.  
**Exam keywords:** `accessibility`, `different abilities`, `inclusive`.

## 23. Explain AI limitations

**Situation:** Users must know when a review is AI-generated and understand its limits.  
**Primary concern:** Understandability and disclosure.  
**Correct answer:** Transparency.  
**Why:** It makes the AI's role and limitations visible.  
**Closest wrong answer:** Accountability.  
**Why that is wrong:** Accountability assigns responsibility; disclosure explains operation/source.  
**Where configured:** UI/process/documentation.  
**Plan:** Any.  
**Exam keywords:** `explain`, `disclose`, `understand`, `AI-generated`.

## 24. CLI needs one file

**Situation:** In current Copilot CLI, a user wants a question grounded in `src/auth.ts`.  
**Primary concern:** Explicit file context.  
**Correct answer:** `Explain @src/auth.ts`.  
**Why:** Current CLI uses `@` mentions for files/directories.  
**Closest wrong answer:** `@workspace`.  
**Why that is wrong:** That is current VS Code Chat participant syntax, not current standalone CLI file syntax.  
**Where configured:** Interactive `copilot` session.  
**Plan:** All current Copilot plans include CLI, subject to policy.  
**Exam keywords:** `CLI`, `reference file`, `@`.

## 25. CLI asks to run a destructive command

**Situation:** The CLI proposes deleting generated files, but scope looks broad.  
**Primary concern:** Tool authorization and safety.  
**Correct answer:** Deny, inspect paths/diff, narrow the prompt/permission, then approve only if safe.  
**Why:** Human review and least privilege apply to tool execution.  
**Closest wrong answer:** Start with `--allow-all`.  
**Why that is wrong:** It removes an important approval boundary.  
**Where configured:** CLI permission prompt, `/permissions`, allow/deny rules.  
**Plan:** Any plan with CLI access.  
**Exam keywords:** `command approval`, `least privilege`, `trusted directory`.

## 26. Pool exhausted but completions needed

**Situation:** An organization's AI-credit pool and hard budget are exhausted.  
**Primary concern:** Which features continue.  
**Correct answer:** Chat/agents/CLI consuming credits are blocked, but paid-plan code completions and NES continue.  
**Why:** Completions/NES do not consume credits on paid plans.  
**Closest wrong answer:** All Copilot features stop.  
**Why that is wrong:** It ignores the non-credit-billed features.  
**Where configured:** AI paid-usage policy and budgets.  
**Plan:** Business/Enterprise scenario.  
**Exam keywords:** `pool exhausted`, `budget`, `completions continue`.

## 27. Background issue-to-PR task

**Situation:** Assign a well-scoped GitHub issue and continue other work while a PR is prepared.  
**Primary concern:** Asynchronous delegation.  
**Correct answer:** Copilot cloud agent.  
**Why:** It works in a GitHub Actions-powered environment on a branch and can open a PR.  
**Closest wrong answer:** IDE Agent mode.  
**Why that is wrong:** IDE Agent mode works interactively in the local environment.  
**Where configured:** Enable cloud agent for the organization/repository; start from a supported entry point.  
**Plan:** All paid Copilot plans; org policies apply.  
**Exam keywords:** `issue`, `background`, `asynchronous`, `branch`, `PR`.

## 28. Cloud agent needs a private package token

**Situation:** Agent setup must authenticate to an internal registry.  
**Primary concern:** Correct secret type and least privilege.  
**Correct answer:** Configure a dedicated repository/organization **Agents** secret and restrict its repository access.  
**Why:** Cloud agent cannot access Actions, Codespaces, or Dependabot secrets.  
**Closest wrong answer:** Reuse an Actions secret automatically.  
**Why that is wrong:** It is not exposed to cloud agent.  
**Where configured:** Settings → Security → Secrets and variables → Agents.  
**Plan:** Paid plan with cloud agent; org controls may apply.  
**Exam keywords:** `cloud agent`, `secret`, `private registry`, `Agents`.

## 29. IDE Agent mode opens an excluded file

**Situation:** `/secret/**` is excluded, but local IDE Agent mode reads it.  
**Primary concern:** Unsupported-surface assumption.  
**Correct answer:** IDE Agent mode does not support content exclusion; do not use that mode for the sensitive repo/path, and enforce repository/OS access boundaries.  
**Why:** The documented limitation is explicit.  
**Closest wrong answer:** Wait another 30 minutes only.  
**Why that is wrong:** Propagation cannot make an unsupported mode enforce the rule.  
**Where configured:** Disable/restrict Agent mode via policy/workflow and protect data access.  
**Plan:** Business/Enterprise exclusion still does not cover IDE Agent mode.  
**Exam keywords:** `Agent mode`, `excluded file`, `limitation`.

## 30. Compare adoption, not administration

**Situation:** A leader asks how many users are active and the acceptance rate.  
**Primary concern:** Adoption measurement.  
**Correct answer:** Copilot usage metrics/dashboard.  
**Why:** It contains DAU/WAU, suggestion/acceptance, Chat, model, and mode metrics.  
**Closest wrong answer:** Audit log.  
**Why that is wrong:** Audit is governance history, not adoption analytics.  
**Where configured:** Enterprise → Insights → Copilot usage; usage-metrics policy must be enabled.  
**Plan:** Managed organization/enterprise availability.  
**Exam keywords:** `how many users`, `acceptance rate`, `adoption`.

## 31. Insecure generated code

**Situation:** Copilot generates code with a possible injection vulnerability.  
**Primary concern:** Detection and remediation.  
**Correct answer:** Human review/testing plus code scanning/CodeQL; consider Copilot Autofix for an alert.  
**Why:** Generative suggestions are not guaranteed secure; GHCS tools can detect/remediate vulnerabilities.  
**Closest wrong answer:** Public-code matching.  
**Why that is wrong:** Similarity/provenance does not determine security.  
**Where configured:** Repository code-security settings and CI/review workflow.  
**Plan:** Public repositories have several features free; private/internal advanced capabilities require relevant Code Security licensing.  
**Exam keywords:** `vulnerability`, `CodeQL`, `code scanning`, `Autofix`.

## 32. Conflicting organization policies

**Situation:** A user has Copilot seats from two organizations with different public-code policies.  
**Primary concern:** Policy conflict resolution.  
**Correct answer:** For suggestions matching public code, apply the **most restrictive** organization rule; consult the feature-specific conflict table.  
**Why:** Conflict behavior is not universal; this privacy policy is a documented exception to the common least-restrictive availability pattern.  
**Closest wrong answer:** Always use the least restrictive organization for every policy.  
**Why that is wrong:** Public-code policy specifically uses most restrictive.  
**Where configured:** Providing organizations/enterprise AI controls; verify effective behavior.  
**Plan:** Business/Enterprise seats.  
**Exam keywords:** `multiple organizations`, `conflict`, `most restrictive`, `public code`.

---

# Part 19 — Navigation route cheat sheet

Routes are current on 2026-09-05. New and original billing platforms can coexist, so follow the UI/documentation shown for your account.

| Task | Who | Exact current navigation | Plan/source |
|---|---|---|---|
| Manage organization seats | Organization owner | Profile → Organizations → org → Settings → Copilot → **Access** | [Business/Enterprise access](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-access/grant-access) |
| Automate organization seat management | Organization owner / appropriately authorized integration | Copilot user-management REST API: list/get seats; add/remove selected users or teams | [Business/Enterprise; public preview](https://docs.github.com/en/rest/copilot/copilot-user-management?apiVersion=2026-03-10) |
| Manage direct enterprise Business licenses | Enterprise owner | Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** | [Enterprise access](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-access/grant-access) |
| Configure organization policy | Organization owner | Organization → Settings → Copilot → **Policies** | [Business/Enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-policies) |
| Configure organization models | Organization owner | Organization → Settings → Copilot → **Models** | [Business/Enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-policies) |
| Configure enterprise Copilot policy | Enterprise owner/suitable custom permission | Enterprise → **AI controls** → **Copilot** | [Business/Enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-enterprise-policies) |
| Configure enterprise clients/features | Same | Enterprise → AI controls → Copilot → **Configure features & clients** | Business/Enterprise |
| Configure enterprise agent policy | Same | Enterprise → AI controls → **Agents** | Business/Enterprise |
| Configure enterprise MCP policy | Same | Enterprise → AI controls → **MCP** | Business/Enterprise |
| Exclude repository content | Repository admin | Repository → Settings → Copilot → **Content exclusion** | [Business/Enterprise](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) |
| Exclude organization content | Organization owner | Organization → Settings → Copilot → **Content exclusion** | Business/Enterprise |
| Exclude enterprise content | Enterprise owner | Enterprise → AI controls → Copilot → **Content exclusion** | Business/Enterprise |
| Review repository/org exclusion changes | Organization owner | Content-exclusion settings → bottom “last changed” time → matching audit entries | [Business/Enterprise](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/review-changes) |
| Personal public-code matching | Individual subscriber | Profile → **Copilot settings** → Suggestions matching public code | [Individual policy](https://docs.github.com/en/copilot/how-tos/manage-your-account/manage-policies) |
| Managed public-code matching | Org/enterprise owner | Organization Copilot Policies, or Enterprise AI controls → Copilot → Privacy | Business/Enterprise |
| Enterprise Copilot audit | Enterprise owner/Read enterprise audit logs | Enterprise → Settings → **Audit log**; search `action:copilot` | [Audit docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/review-audit-logs) |
| Organization audit | Organization owner | Organization → Settings → Archive → Logs → **Audit log** | [Audit docs](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization) |
| Adoption/usage metrics | Enterprise owner, organization administrator, billing manager, or authorized custom role | Enterprise → **Insights** → **Copilot usage** | [Usage dashboard](https://docs.github.com/en/copilot/how-tos/administer-copilot/view-usage-and-adoption) |
| AI-credit usage, individual | Personal account owner | Settings → Billing & Licensing → **AI usage** | [Monitor AI usage](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/monitor-ai-usage) |
| Personal credits for managed-seat user | User | Profile → **Copilot settings** → Usage → Usage this cycle | Business/Enterprise seat |
| Organization/enterprise AI usage | Owner/billing manager | Account → **Billing & Licensing** → under Usage, **AI usage** | [Metered/license usage](https://docs.github.com/en/billing/how-tos/products/view-productlicense-use) |
| Configure budgets | Account owner/billing manager as applicable | Billing & Licensing → **Budgets and alerts** → New budget | [Budget setup](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| Personal Copilot plan | User | Settings → Billing & licensing → Licensing (new) or Plans and usage (original) | [Plan management](https://docs.github.com/en/copilot/how-tos/manage-your-account/view-and-change-your-copilot-plan) |

**Navigation exam technique:** identify **task → scope → role → page**. Do not choose an enterprise route for a one-repository rule.

---

# Part 20 — Important numbers

| Number | Meaning | Memorize? | Official source | Memory trick |
|---:|---|:---:|---|---|
| 700 | Minimum GH-300 scaled passing score | Yes | [Study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-300) | “7 hundred to pass” |
| 100 minutes | Current exam time | Medium | [Certification page](https://learn.microsoft.com/en-us/credentials/certifications/github-copilot/) | Recheck before booking |
| US$0.01 | Value of one AI Credit | Yes | [Copilot billing](https://docs.github.com/en/billing/concepts/product-billing/github-copilot-billing) | 100 credits = US$1 |
| 2,000/month | Free plan completion limit | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Only Free has this completion quota |
| US$10; 1,500 | Pro price; total monthly credits (1,000 base + 500 flex) | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Pro = 10 / 1.5k |
| US$39; 7,000 | Pro+ price; total monthly credits (3,900 + 3,100) | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Plus = 39 / 7k |
| US$100; 20,000 | Max price; total monthly credits (10k + 10k) | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Max = 100 / 20k |
| US$19; 1,900 | Business seat price; pool contribution/user | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Business 19 ↔ 1900 |
| US$39; 3,900 | Enterprise seat price; pool contribution/user | Yes | [Plans](https://docs.github.com/en/copilot/get-started/plans) | Enterprise 39 ↔ 3900 |
| 00:00 UTC, day 1 | Monthly included-credit reset; no rollover | Yes | [Usage billing](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises/usage-based-billing) | Calendar-month pool |
| Up to 30 minutes | Content-exclusion propagation in loaded IDEs | Yes | [Exclusion how-to](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot#testing-changes-to-content-exclusions) | Exclusion = 30 |
| 180 days | Organization and enterprise audit retention | Yes | [Enterprise audit](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/review-audit-logs) | Export for longer retention |
| Up to 3 full UTC days | Usage dashboard data lag | Medium | [Usage dashboard](https://docs.github.com/en/copilot/how-tos/administer-copilot/view-usage-and-adoption) | Metrics are not real-time |
| About 150 characters | Surrounding code used in public-code comparison | Medium | [Code referencing](https://docs.github.com/en/copilot/concepts/completions/code-referencing) | Match candidate + nearby text |
| 75%, 90%, 100% | Optional budget alert thresholds | Low | [Budget setup](https://docs.github.com/en/billing/how-tos/set-up-budgets) | Alerts do not necessarily stop usage |
| 100 conversations / 28 days | Current GitHub.com Chat history limits/retention | Low | [Chat on GitHub](https://docs.github.com/en/copilot/how-tos/copilot-on-github/chat-with-copilot/chat-in-github) | Current snapshot; recheck |
| 2 / 4 / 8 / 16 / 32 | Default concurrent CLI subagents: Free/Education; Pro/Pro+; Max; Business; Enterprise | Medium | [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#subagent-limits) | Plan increases parallel capacity |

Free and Student AI-credit allowance numbers: **Not confirmed in current official documentation**. Do not memorize a number from old premium-request material.

---

# Part 21 — English vocabulary for GH-300

## Negative and ranking words — read these first

| Word | Simple Vietnamese | How it changes the logic |
|---|---|---|
| **NOT** | không | Choose the false/non-matching choice. |
| **EXCEPT** | ngoại trừ | Usually three fit; choose the one that does not. |
| **LEAST** | ít nhất/kém nhất | Choose the weakest or least suitable. |
| **MOST** | nhiều/đúng nhất | Choose the strongest match, even if others can happen. |
| **BEST** | tốt nhất | Optimize for every stated requirement. |
| **MOST APPROPRIATE** | phù hợp nhất | Choose the best fit, often the least sufficient scope/cost. |
| **PRIMARILY / MAINLY** | chủ yếu | Choose the dominant concern, not every related concern. |
| **MOST LIKELY** | có khả năng nhất | Several are possible; choose strongest evidence. |
| **LEAST LIKELY** | ít khả năng nhất | Choose weakest evidence/probability. |

Example: “Which is **LEAST** appropriate?” → find the worst/least suitable choice.  
Example: “Which principle is **PRIMARILY** addressed by excluding secrets?” → Privacy & Security, although good governance also supports accountability.

## Core vocabulary

| English | Simple Vietnamese | Exam example | Trap |
|---|---|---|---|
| ensure | bảo đảm | ensure tests pass | Stronger than “help”; no AI alone guarantees it |
| enforce | bắt buộc áp dụng | enterprise enforces Disabled | Not the same as recommend |
| prevent | ngăn chặn | prevent path becoming context | Strong requirement; choose a control |
| restrict | hạn chế | restrict tool/network access | May reduce, not necessarily remove all access |
| exclude | loại trừ | exclude `/secrets/**` | Content exclusion, not delete |
| retain | lưu giữ | retain audit data | Keep, not retrieve |
| retrieve | truy xuất/lấy lại | retrieve a log | Fetch existing data |
| infer | suy luận | model infers intent | An inference may be wrong |
| mitigate | giảm thiểu rủi ro | mitigate prompt injection | Reduce, not eliminate |
| adhere | tuân theo | adhere to instructions | Behavior goal, not guarantee |
| comply | tuân thủ | comply with policy/license | Compliance needs evidence |
| compliance | sự tuân thủ | compliance requirement | Broader than one setting |
| regardless | bất kể | blocked regardless of pool | Ignore the named condition |
| subsequent | tiếp theo/sau đó | subsequent retries | After the first event |
| underlying | nền tảng/bên dưới | underlying model | Not the UI feature |
| surrounding | xung quanh | surrounding code | Nearby context |
| verbose | dài dòng | avoid verbose response | Opposite of concise |
| concise | súc tích | concise plan | Short **and sufficient**, not incomplete |
| specific | cụ thể | specific file/path | Opposite of vague |
| clarity | sự rõ ràng | improve prompt clarity | Not simply more words |
| craft/crafted | soạn/tạo có chủ đích | crafted prompt | Deliberately designed |
| utilize / leverage | sử dụng/tận dụng | leverage repository context | Usually just “use” |
| encounter/encountering | gặp phải | encounter a limit | Experience a situation |
| offensive | xúc phạm | offensive output | Safety/toxicity, not IP |
| harmful | gây hại | harmful content | Safety/toxicity |
| toxic | độc hại | toxic response | Safety/toxicity |
| abusive | lạm dụng/xúc phạm | abusive language | Safety/toxicity |
| inappropriate | không phù hợp | inappropriate output | Context-sensitive safety term |
| sensitive | nhạy cảm | sensitive customer data | Privacy & Security |
| confidential | mật/bí mật | confidential code | Privacy & Security/exclusion |
| disclose | tiết lộ/công bố | disclose AI use | Transparency; may also mean data leak by context |
| oversight | giám sát | human oversight | Often Accountability |
| accountability | trách nhiệm giải trình | owner of final decision | Not the act of testing |
| reliable / reliability | đáng tin cậy/độ tin cậy | validate correct behavior | Reliability & Safety |
| fairness | công bằng | avoid unequal outcomes | Bias/discrimination signal |
| transparency | minh bạch | explain AI role/limits | Not ownership |
| bias / biased | thiên lệch | biased recommendation | Fairness |
| discrimination | phân biệt đối xử | harms a protected group | Fairness |
| violate | vi phạm | violate license/IP rights | Strong legal/policy breach term |
| intellectual property | sở hữu trí tuệ | public-code license concern | Not toxicity |
| propagate | lan truyền/cập nhật | policy takes time to propagate | Not inheritance |
| inherit | kế thừa | org receives enterprise rule | Not merely delayed update |
| precedence | thứ tự ưu tiên | which policy wins | Feature-specific conflicts |
| eligible | đủ điều kiện | verified student is eligible | Not automatically enrolled |
| entitlement | quyền được sử dụng | assigned product entitlement | Access right, not usage evidence |
| allowance | hạn mức được bao gồm | monthly credits allowance | Not unlimited usage |
| audit | kiểm tra/nhật ký kiểm toán | who changed policy | Not adoption metrics |
| governance | quản trị | central policy/monitoring | Broader than permissions |
| exposure | sự lộ/tiếp xúc | secret exposure | Privacy & Security |
| leakage | rò rỉ | credential leakage | Privacy & Security |
| attribution | ghi nguồn | credit public source | Not a license itself |
| retention | thời gian lưu giữ | 180-day audit retention | Not propagation delay |
| scope | phạm vi | repository vs organization | Choose smallest sufficient scope |
| constraint | ràng buộc | do not add dependencies | Improves prompt specificity |

---

# Part 22 — GitHub Copilot From Zero (5–10 minutes)

1. **Copilot is an AI assistant**, not a correctness oracle.
2. **Inline suggestion** is gray ghost text at the cursor; `Tab` commonly accepts and `Esc` dismisses.
3. **Chat** explains/designs/answers. Attach relevant context and state constraints.
4. **Inline Chat/Edit** targets selected/current code. **Edit mode** changes a controlled file set. **Agent mode** performs multi-step local work.
5. **Cloud agent** works asynchronously on GitHub and can create a branch/PR. **CLI** is the terminal interface launched with `copilot`.
6. **Context** is the material Copilot can use. Relevant context helps; too much irrelevant context hurts.
7. Always review, test, scan, and modify generated code. The human owns the result.
8. Protect sensitive data with access control and, on Business/Enterprise supported surfaces, **content exclusion**.
9. Public-code matching/referencing concerns similarity, source, license, and attribution—not secret paths or toxic text.
10. Start with Free/Student if eligible; Pro/Pro+/Max are individuals; Business/Enterprise are centrally managed seats.

**Start:** install the official IDE extension or current CLI, sign in, open a small test repository, ask Copilot to explain one file, accept only a reviewed suggestion, then run the tests. [Getting started](https://docs.github.com/en/copilot/get-started)

---

# Part 23 — GH-300 last-minute review

| If you see… | Think… |
|---|---|
| `ghost text`, `as you type` | Inline suggestion / code completion |
| `next likely change/location` | Next edit suggestions |
| `selected code + prompt` | Inline Chat/Edit |
| `known file set` | Edit mode |
| `local multi-step + commands` | IDE Agent mode |
| `issue + asynchronous + branch/PR` | Copilot cloud agent |
| `terminal`, `@file`, `/plan` | Current Copilot CLI |
| `/fleet`, separate context, parallel independent work | CLI subagents |
| `/delegate` or `&`, remote PR task | CLI → cloud agent |
| `@workspace` | Current VS Code Chat participant, not CLI |
| `admin policy history` | Audit log |
| `active users`, `acceptance rate` | Usage metrics |
| `credits/models/spend` | AI usage/billing |
| `exact accepted code/keystroke` | Not in standard audit/metrics |
| `secret/confidential path` | Content exclusion + Privacy & Security |
| `harmful/offensive/abusive` | Content safety/toxicity filter |
| `matches public source/license` | Public-code matching/referencing |
| `developer lacks repo access` | Repository permissions |
| `review/test/validate/scan` | Reliability & Safety |
| `human remains answerable` | Accountability |
| `bias/discrimination` | Fairness |
| `explain/disclose AI behavior` | Transparency |
| `abilities/accessibility` | Inclusiveness |
| `up to 30 minutes` | Content-exclusion propagation |
| `180 days` | Org/enterprise audit retention |
| `three UTC days behind` | Usage dashboard lag |
| `2,000 completions` | Free monthly completion limit |
| `1,900 / 3,900 pooled` | Business / Enterprise credits per seat |
| `policy conflict` | Feature-specific table; public code = most restrictive org |
| `No policy` at enterprise | Delegate decision to organizations |
| `budget reached but charges continue` | Stop usage toggle was not enabled |
| `pool empty; completion` | Paid completion/NES still works |
| `Actions/Codespaces secret for cloud agent` | Wrong secret type; use Agents secret |
| `GHCS` | GitHub Code Security |

**Five-step scenario solver:** identify the **object** → required **mechanism** → smallest correct **scope** → authorized **role** → minimum suitable **plan**. Then eliminate choices using unsupported-surface and propagation facts.

---

# Part 24 — Final concept map

```text
GitHub Copilot
├── Interaction
│   ├── Inline Suggestions / Next Edit Suggestions
│   ├── Chat / Inline Chat / Edit / Plan
│   ├── IDE Agent Mode
│   ├── Cloud Agent (legacy: Coding Agent)
│   └── Copilot CLI
├── Context
│   ├── Current File / Selection / Surrounding Code
│   ├── Repository / References / Conversation
│   ├── Custom Instructions
│   └── Content Exclusion
├── AI Processing
│   ├── Input / Context / Prompt Construction
│   ├── Service and Safety Filtering
│   ├── Model Generation
│   └── Public-Code Check / Post-processing
├── Security and Safety
│   ├── Content Exclusion
│   ├── Content Safety / Toxicity
│   ├── Public-Code Matching / Referencing
│   ├── Repository Permissions
│   └── GitHub Code Security
├── Responsible AI
│   ├── Fairness
│   ├── Reliability & Safety
│   ├── Privacy & Security
│   ├── Inclusiveness
│   ├── Transparency
│   └── Accountability
├── Governance
│   ├── Policies / Inheritance / Conflicts
│   ├── Seats and Roles
│   ├── Audit Logs
│   └── Usage Metrics
└── Plans and Billing
    ├── Individual: Free / Student / Pro / Pro+ / Max
    ├── Organization: Business / Enterprise
    └── AI Credits / Pools / Paid Usage / Budgets
```

---

# Self-test — 31 original practice questions

## Question 1 — Single answer

**Question:** A developer sees gray code appear at the cursor while typing. Which feature is this?

A. Copilot cloud agent  
B. Inline suggestion  
C. Audit log  
D. Code scanning

**Correct answer:** B.  
**Why:** Ghost text at the cursor is an inline suggestion/code completion.  
**Why the other options are wrong:** A works asynchronously on tasks; C records governance events; D finds vulnerabilities.  
**Key English words:** `gray`, `at the cursor`, `while typing`.

## Question 2 — Single answer

**Question:** Which feature is **MOST APPROPRIATE** to refactor only the currently selected method using a natural-language instruction?

A. Inline Chat/Edit  
B. Copilot cloud agent  
C. Organization audit log  
D. Content exclusion

**Correct answer:** A.  
**Why:** Selection plus a focused prompt is the classic Inline Chat/Edit case.  
**Why the other options are wrong:** B is excessive/asynchronous; C is evidence, not editing; D removes context.  
**Key English words:** `most appropriate`, `only`, `selected`.

## Question 3 — Single answer

**Question:** A developer needs autonomous multi-file edits and local command execution while staying in the IDE. What should they use?

A. Ask mode  
B. IDE Agent mode  
C. Public-code matching  
D. Copilot cloud agent

**Correct answer:** B.  
**Why:** IDE Agent mode performs iterative local edits and tool/command work.  
**Why the other options are wrong:** A answers but does not autonomously edit; C is a matching safeguard; D runs asynchronously in a GitHub-hosted environment, not the local IDE.  
**Key English words:** `autonomous`, `local`, `in the IDE`.

## Question 4 — Single answer

**Question:** Developers need Copilot in a repository, but `/secrets/**` must not be used as context. Which solution is best?

A. Disable repository access for all developers  
B. Enable toxicity filtering  
C. Add a repository content-exclusion pattern  
D. Block public-code matches

**Correct answer:** C.  
**Why:** Content exclusion is the path-specific context control.  
**Why the other options are wrong:** A prevents work rather than preserving Copilot elsewhere; B filters harmful content; D concerns generated code similarity.  
**Key English words:** `must not use`, `path`, `continue using`.

## Question 5 — Single answer

**Question:** An exclusion was changed ten minutes ago in a loaded VS Code window. Old behavior remains. What should the developer do first?

A. Buy Copilot Enterprise  
B. Reload the VS Code window, verify scope/pattern/surface, and allow up to 30 minutes  
C. Turn on `--allow-all`  
D. Search public-code references

**Correct answer:** B.  
**Why:** Exclusion propagation can take up to 30 minutes; VS Code can reload its window.  
**Why the other options are wrong:** A does not fix propagation; C is unrelated and unsafe; D checks a different mechanism.  
**Key English words:** `ten minutes`, `loaded`, `old behavior`, `first`.

## Question 6 — Single answer

**Question:** Which mechanism **primarily** addresses abusive and offensive generated text?

A. Content safety/toxicity filtering  
B. Content exclusion  
C. Repository permissions  
D. Code referencing

**Correct answer:** A.  
**Why:** Toxicity/safety filters target harmful or inappropriate content.  
**Why the other options are wrong:** B controls context paths; C controls repository access; D shows public-code provenance.  
**Key English words:** `primarily`, `abusive`, `offensive`.

## Question 7 — Single answer

**Question:** A Copilot suggestion resembles code in a public repository. Which capability helps identify the source and license?

A. Audit retention  
B. Code referencing  
C. Content exclusion  
D. Next edit suggestions

**Correct answer:** B.  
**Why:** Code references can show matching repository URLs and detected license information.  
**Why the other options are wrong:** A is log lifetime; C controls private context; D predicts edits.  
**Key English words:** `resembles`, `public repository`, `source`, `license`.

## Question 8 — Select two

**Question:** Which TWO actions best reduce risk when Copilot returns code matching a public source?

A. Inspect code references and license information  
B. Review organizational IP/compliance requirements  
C. Use a toxicity filter as the license authority  
D. Assume a reference guarantees legal permission

**Correct answer:** A and B.  
**Why:** References support provenance/licensing review, and policy/legal requirements still need human analysis.  
**Why the other options are wrong:** C addresses harmful content, not licensing; D overstates what a reference proves.  
**Key English words:** `two`, `reduce risk`, `matching`, `public source`.

## Question 9 — Single answer

**Question:** A standard requires developers to test, scan, and validate AI-generated code. Which Responsible AI principle is the strongest match?

A. Accountability  
B. Transparency  
C. Reliability & Safety  
D. Inclusiveness

**Correct answer:** C.  
**Why:** Testing and validation establish correct and safe behavior.  
**Why the other options are wrong:** A is human answerability; B is understandability/disclosure; D supports varied users/abilities.  
**Key English words:** `test`, `scan`, `validate`, `strongest`.

## Question 10 — Single answer

**Question:** A named maintainer remains answerable for the final merge decision. Which principle applies **primarily**?

A. Accountability  
B. Fairness  
C. Reliability & Safety  
D. Privacy & Security

**Correct answer:** A.  
**Why:** Named human answerability and oversight define accountability.  
**Why the other options are wrong:** B concerns bias; C concerns validation/safe operation; D concerns data/system protection.  
**Key English words:** `named`, `answerable`, `final decision`, `primarily`.

## Question 11 — Single answer

**Question:** An AI-supported assessment systematically produces worse outcomes for one protected group. Which principle is most directly involved?

A. Inclusiveness  
B. Fairness  
C. Transparency  
D. Reliability

**Correct answer:** B.  
**Why:** Unequal group treatment and discrimination are fairness concerns.  
**Why the other options are wrong:** A emphasizes broad ability/access support; C explanation; D predictable/correct operation.  
**Key English words:** `systematically`, `protected group`, `worse outcomes`.

## Question 12 — Single answer

**Question:** A Copilot workflow must support users with different abilities, including keyboard-only use. Which principle fits best?

A. Inclusiveness  
B. Accountability  
C. Public-code matching  
D. Retention

**Correct answer:** A.  
**Why:** Accessibility and differing abilities point to inclusiveness.  
**Why the other options are wrong:** B is ownership; C is provenance; D is how long data remains.  
**Key English words:** `different abilities`, `keyboard-only`, `best`.

## Question 13 — Single answer

**Question:** Users must be told that a review is AI-generated and understand the model's limitations. Which principle is primary?

A. Transparency  
B. Fairness  
C. Content exclusion  
D. Reliability & Safety

**Correct answer:** A.  
**Why:** Disclosure and understandability are transparency.  
**Why the other options are wrong:** B is equitable treatment; C is a path control; D is validation and safe behavior.  
**Key English words:** `told`, `understand`, `limitations`, `primary`.

## Question 14 — Single answer

**Question:** A verified student wants free Copilot and unlimited completions. Which plan is most appropriate?

A. Free  
B. Student  
C. Business  
D. Max

**Correct answer:** B.  
**Why:** Student is free for verified students and includes unlimited completions.  
**Why the other options are wrong:** A limits completions to 2,000/month; C is organization licensing; D is a US$100 individual tier.  
**Key English words:** `verified`, `free`, `unlimited`, `most appropriate`.

## Question 15 — Single answer

**Question:** A small company needs central policies, audit logs, and content exclusion at the lowest current organization-plan price. Which plan?

A. Pro  
B. Business  
C. Enterprise  
D. Student

**Correct answer:** B.  
**Why:** Business includes those governance features for US$19 per granted seat.  
**Why the other options are wrong:** A and D are individual; C also works but costs more without a stated Enterprise-only need.  
**Key English words:** `company`, `central`, `lowest`, `organization-plan`.

## Question 16 — Single answer

**Question:** A GHEC enterprise requires Spark and 3,900 monthly pooled credits per granted seat. Which plan?

A. Pro+  
B. Business  
C. Enterprise  
D. Free

**Correct answer:** C.  
**Why:** Copilot Enterprise supplies the named features/allowance.  
**Why the other options are wrong:** A is individual; B has 1,900 credits/seat and lacks Spark in the current matrix; D is limited individual use.  
**Key English words:** `GHEC`, `requires`, `Spark`, `3,900`.

## Question 17 — Single answer

**Question:** What is the current monthly completion limit for Copilot Free?

A. 500  
B. 1,000  
C. 2,000  
D. Unlimited

**Correct answer:** C.  
**Why:** Current official plans list 2,000 completions/month for Free.  
**Why the other options are wrong:** They do not match the current plan table. Paid plans and Student have unlimited completions.  
**Key English words:** `current`, `Free`, `completion limit`.

## Question 18 — Single answer

**Question:** What is one GitHub AI Credit worth under current billing?

A. US$0.001  
B. US$0.01  
C. US$0.10  
D. One premium request

**Correct answer:** B.  
**Why:** One AI Credit equals US$0.01.  
**Why the other options are wrong:** A/C are wrong values; D mixes the legacy request system with current token/model-based credits.  
**Key English words:** `worth`, `current billing`.

## Question 19 — Select two

**Question:** Which TWO features do **NOT** consume AI Credits on paid plans?

A. Code completions  
B. Next edit suggestions  
C. Copilot CLI interactions  
D. Cloud-agent sessions

**Correct answer:** A and B.  
**Why:** Paid-plan completions and NES are included and not AI-credit billed.  
**Why the other options are wrong:** CLI and cloud-agent work consume credits.  
**Key English words:** `two`, `do not`, `paid plans`.

## Question 20 — Single answer

**Question:** An enterprise creates a metered spending budget but leaves “Stop usage when budget limit is reached” off. What happens after the budget is reached?

A. Usage always stops immediately  
B. Charges can continue; the budget acts as an alert  
C. All completions permanently stop  
D. The pool doubles

**Correct answer:** B.  
**Why:** That stop option is off by default for enterprise/org/cost-center spending limits.  
**Why the other options are wrong:** A ignores the toggle; C confuses non-credit-billed completions; D is invented.  
**Key English words:** `leaves off`, `after`, `budget reached`.

## Question 21 — Navigation

**Question:** Where should an organization owner change Copilot feature/privacy policies?

A. Repository Actions → Runners  
B. Organization Settings → Copilot → Policies  
C. Personal profile → SSH keys  
D. Enterprise Insights → Copilot usage

**Correct answer:** B.  
**Why:** That is the current organization policy route.  
**Why the other options are wrong:** A manages Actions runners; C is personal authentication; D is enterprise adoption metrics.  
**Key English words:** `organization owner`, `change`, `policies`.

## Question 22 — Navigation

**Question:** An enterprise owner needs to discover who changed a Copilot policy. Where should they look?

A. Enterprise Settings → Audit log; filter `action:copilot`  
B. Enterprise Insights → Copilot usage only  
C. Personal Chat history  
D. Code references

**Correct answer:** A.  
**Why:** Audit logs record administrative actors/actions.  
**Why the other options are wrong:** B is adoption; C is personal conversation history; D is public-code provenance.  
**Key English words:** `who changed`, `enterprise owner`, `look`.

## Question 23 — Single answer

**Question:** Which source answers “How many weekly active Copilot users do we have?”

A. Audit log  
B. Usage metrics/dashboard  
C. Public-code reference log  
D. Content-exclusion YAML

**Correct answer:** B.  
**Why:** WAU and adoption counts are usage metrics.  
**Why the other options are wrong:** A tracks events; C tracks matching provenance; D defines excluded paths.  
**Key English words:** `how many`, `weekly active`, `users`.

## Question 24 — Single answer

**Question:** What is the current standard retention period for organization and enterprise audit logs discussed here?

A. 7 days  
B. 28 days  
C. 90 days  
D. 180 days

**Correct answer:** D.  
**Why:** Current organization and enterprise audit-log documentation states 180 days.  
**Why the other options are wrong:** 28 days relates to current GitHub.com Chat messages; the other values do not answer this audit question.  
**Key English words:** `retention`, `organization and enterprise`, `current`.

## Question 25 — CLI

**Question:** In current Copilot CLI, how should a user explicitly reference `README.md`?

A. `@README.md`  
B. `#workspace:README.md`  
C. `/audit README.md`  
D. `actor:README.md`

**Correct answer:** A.  
**Why:** CLI `@` mentions files/directories.  
**Why the other options are wrong:** B mixes contexts/syntax; C is not a current command; D is audit-search syntax.  
**Key English words:** `current`, `CLI`, `explicitly reference`.

## Question 26 — CLI

**Question:** Which current command asks Copilot CLI to develop an implementation plan before changes?

A. `/plan`  
B. `/diff`  
C. `/exit`  
D. `/limits`

**Correct answer:** A.  
**Why:** `/plan [PROMPT]` produces a plan.  
**Why the other options are wrong:** `/diff` inspects changes; `/exit` leaves; `/limits` concerns limits.  
**Key English words:** `implementation plan`, `before changes`.

## Question 27 — CLI

**Question:** Copilot CLI proposes a broad mutating command. Which response is **LEAST** appropriate?

A. Deny and inspect the exact target  
B. Narrow permissions and request  
C. Review the diff and command  
D. Use `--allow-all` without review

**Correct answer:** D.  
**Why:** Blanket authorization without review violates least privilege and human oversight.  
**Why the other options are wrong:** A–C are appropriate safeguards, so they cannot answer a **LEAST appropriate** question.  
**Key English words:** `least`, `broad`, `mutating`, `without review`.

## Question 28 — Current vs legacy

**Question:** Which statement about `@workspace` is accurate?

A. It is the current standalone CLI command for every repository  
B. It is a current VS Code Copilot Chat participant, while CLI uses `@path` mentions  
C. It is the audit filter for workspace changes  
D. It is a content-exclusion wildcard

**Correct answer:** B.  
**Why:** Current VS Code and current CLI use different context mechanisms.  
**Why the other options are wrong:** A mixes surfaces; C/D assign unrelated meanings.  
**Key English words:** `accurate`, `current`, `while`.

## Question 29 — Policy conflict

**Question:** A user has seats from two organizations with conflicting “Suggestions matching public code” settings. Which rule is used?

A. Always the newest rule  
B. The most restrictive organization rule  
C. Always the least restrictive organization rule  
D. The user's local preference always overrides both

**Correct answer:** B.  
**Why:** Public-code privacy policy uses the documented most-restrictive organization conflict rule.  
**Why the other options are wrong:** A is invented; C is common for several feature-availability policies but not this one; D ignores managed-seat policy.  
**Key English words:** `conflicting`, `public code`, `which rule`.

## Question 30 — GHCS

**Question:** A code-scanning alert needs an AI-generated targeted fix that a developer will review. Which feature is directly relevant?

A. Copilot Autofix  
B. Next edit suggestions  
C. Toxicity filtering  
D. Content exclusion

**Correct answer:** A.  
**Why:** Copilot Autofix can generate a fix suggestion for a code-scanning alert.  
**Why the other options are wrong:** B predicts editing flow; C filters harmful content; D protects context paths. Standard Autofix is distinct from agentic autofix/cloud-agent sessions.  
**Key English words:** `code-scanning alert`, `targeted fix`, `review`.

## Question 31 — CLI subagents

**Question:** A large investigation contains several independent modules, and the main Copilot CLI context should stay focused. Which option is most appropriate?

A. `/fleet` so separate subagents can handle parallel chunks  
B. Paste every build log into one prompt  
C. Content exclusion  
D. Organization audit log

**Correct answer:** A.  
**Why:** `/fleet` enables parallel subagent execution; each subagent has a separate context window.  
**Why the other options are wrong:** B crowds the main context; C protects selected repository content rather than delegating work; D records governance events.
**Key English words:** `independent`, `parallel`, `main context`, `most appropriate`.

---

# Final accuracy checklist

- [x] Current plans, prices, AI Credits, pools, and legacy premium requests distinguished.
- [x] Business vs Enterprise rechecked; governance/content exclusion/audit are not mislabeled Enterprise-only.
- [x] Current CLI installation, context syntax, commands, permissions, trust, MCP, and security covered.
- [x] Custom instructions, prompt files, custom agents, MCP, Spaces, PR summaries, Spark, agent sessions, and subagents distinguished.
- [x] Productivity workflows and Copilot subscription/seat REST endpoints covered from the current blueprint.
- [x] IDE shortcuts and feature parity tied to current official matrices.
- [x] Inline, Chat, Edit, Plan, Agent mode, cloud agent, review, and CLI separated.
- [x] Content exclusion plan, role, route, syntax, inheritance, 30-minute propagation, refresh, verification, and surface limits covered.
- [x] Current 2026-09-02 CLI/app content-exclusion change called out against legacy material.
- [x] Public-code matching/referencing, IP vocabulary, and unsupported-policy surfaces distinguished.
- [x] Policy scope, inheritance, feature-specific conflicts, routes, seats, and verification covered.
- [x] Audit contents/exclusions, roles, filters, and 180-day retention separated from metrics/billing/history.
- [x] Responsible AI principles and exam-trigger verbs mapped.
- [x] Lifecycle uses a conceptual official-doc flow without inventing a universal hidden implementation order.
- [x] GHCS identified as GitHub Code Security and separated from Copilot and Secret Protection.
- [x] Unconfirmed values/actions explicitly say **Not confirmed in current official documentation**.

## Final official-document refresh set

Use these four live pages immediately before the exam because they change most often:

1. [GH-300 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-300)
2. [Copilot plans](https://docs.github.com/en/copilot/get-started/plans)
3. [Copilot feature matrix](https://docs.github.com/en/copilot/reference/copilot-feature-matrix)
4. [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
