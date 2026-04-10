# 🕸️ Graphify — AI 知識圖譜生成器

> 將任意程式碼庫或文件夾轉換為可查詢的知識圖譜，讓 AI 助手讀懂你的程式碼。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-green.svg)](https://www.python.org/downloads/)

---

## 📖 專案簡介

Graphify 是一款 **AI 編碼助手技能插件**。在 Claude Code、Codex、OpenCode、Trae 或 Cursor 中輸入 `/graphify`，它能讀取任意資料夾（程式碼、文件、PDF、圖片、截圖、筆記等），自動構建可查詢的「知識圖譜（Knowledge Graph）」。

> **"Andrej Karpathy 會維護一個 `/raw` 資料夾，把論文、推文、截圖和筆記都丟進去。Graphify 就是在解決這類問題 —— 相比直接讀取原始檔案，每次查詢的 Token 消耗可降低 71.5 倍，結果還能跨會話持久保存。"**

### 它能解決什麼問題？

| 痛點 | Graphify 的解法 |
|------|----------------|
| 接手舊專案，搞不清楚程式架構 | 自動生成視覺化圖譜，一眼看穿模組關聯 |
| AI 助手讀不懂你的程式碼，亂耗 Token | 圖譜查詢比直接讀檔省 **71.5 倍** Token |
| 想知道「為什麼這樣設計」而非只是「做了什麼」 | 提取 `# WHY:`、`# NOTE:` 等開發者註解 |
| 文件、圖片、程式碼散落各處，關聯斷裂 | 跨模態整合，自動推論隱含關係 |

---

## ✨ 核心特性

### 🧠 雙階段智慧處理架構

```
階段 1: 本地 AST 解析 (100% 離線)
   └─ 使用 tree-sitter 解析 20+ 種語言
   └─ 提取: 類別、函數、匯入、呼叫圖、Docstring、註解

階段 2: LLM 語意提取 (需 API Key)
   └─ 平行處理文件、論文、圖片
   └─ 提取跨模態概念、設計理念、隱含關聯
```

### 🎯 精準關係標記

每條邊都會標記來源類型，明確區分事實與猜測：

| 類型 | 說明 | 置信度 |
|------|------|--------|
| `EXTRACTED` | 直接從程式碼/文件提取，確定存在 | 100% 事實 |
| `INFERRED` | 由 AI 推論出的潛在關聯 | 附 0.0~1.0 分數 |
| `AMBIGUOUS` | 存疑的關聯，需要人工確認 | 待驗證 |

### 📊 拓撲導向分群 (無需向量資料庫)

- 使用 **Leiden 社群偵測演算法**（源自 NetworkX + graspologic）
- 基於邊密度自動分群，語意相似的節點自然聚集
- **省去 Embedding 與向量資料庫**，純圖論計算

### ⚡ 極致 Token 優化

```
傳統方式: AI 讀完整個資料夾 → 耗費大量 Token
Graphify: 查詢知識圖譜    → 僅返回相關節點
```

實測 Token 消耗減少約 **71.5 倍**，搭配 SHA256 快取，僅增量處理變更檔案。

### 🔮 你會得到什麼 (進階洞察)

- **超邊 (Hyperedges)**：用來表達 3 個以上節點的群組關係。例如：一組類別共同實現一個協議、認證鏈路裡的一組函數。
- **置信度分數 (Confidence Score)**：每條 `INFERRED` 邊都有 `0.0-1.0` 的評分。你不只知道哪些是猜出來的，還知道 AI 有多大把握。
- **語義相似邊**：即使結構上沒有直接依賴，也能建立關聯。例如：兩個函數實作了相同的演算法概念。
- **「為什麼」 (Rationale)**：自動提取 `# WHY:`、`# NOTE:`、`# HACK:` 等註解，讓你理解設計動機。

### 🔗 AI 助手深度整合 (Always-on)

支援以下平台自動注入圖譜規則：

- **Claude Code** → 寫入 `CLAUDE.md` + PreToolUse Hook
- **Codex CLI** → 寫入 `AGENTS.md` + Hook
- **Cursor** → 專案規則注入
- **Gemini CLI** → 對應規則生成

讓 AI 助手在搜尋檔案前，**優先讀取圖譜報告**，以結構化知識取代盲目 `grep`。

### 🔒 隱私設計

| 類型 | 處理方式 |
|------|----------|
| 程式碼檔案 | **100% 本地解析**，絕不外送 |
| 非程式碼檔案 | 僅透過**你自己的 API Key** 送至 LLM |
| 遙測/追蹤 | **完全沒有** |
| 雲端儲存 | **不涉及** |

---

## 🚀 快速開始

### 安裝

```bash
# ⚠️ 要求 Python 3.10+
pip install graphifyy

# 初始化環境
graphify install
```

> **PyPI 包目前名為 `graphifyy` (多一個 y)**，因為 `graphify` 名稱正在回收中。CLI 命令仍為 `graphify`。

`graphify install` 執行結果(自動更新 CLAUDE.md 與 skill)：
```bash
graphify install                                                                                         ✔  took 5s  at 09:30:27 
  skill installed  ->  /Users/liao-eli/.claude/skills/graphify/SKILL.md
  CLAUDE.md        ->  skill registered in /Users/liao-eli/.claude/CLAUDE.md

Done. Open your AI coding assistant and type:

  /graphify .
```

### 基本使用

```bash
# 為當前目錄建立知識圖譜
graphify .

# 增量更新 (僅處理變更的檔案)
graphify . --update

# 背景監控，檔案異動自動同步
graphify . --watch
```

### 查詢圖譜

```bash
# 自然語言查詢
graphify query "auth 流程如何運作？"

# 追蹤兩個節點之間的路徑
graphify path "DigestAuth" "Response"

# 解釋特定節點的含義與關聯
graphify explain "SwinTransformer"

# 拉取外部資源並更新圖譜
graphify add https://arxiv.org/abs/1706.03762
graphify add https://x.com/karpathy/status/...
```

### 啟動 MCP 伺服器

```bash
# 暴露圖譜為 Model Context Protocol 服務
python -m graphify.serve graphify-out/graph.json
```

---

## 📁 輸出內容說明

執行後會生成 `graphify-out/` 目錄：

```
graphify-out/
├── graph.html        # 🌐 互動式圖譜網頁 (點擊、搜尋、社群過濾)
├── GRAPH_REPORT.md   # 📋 核心節點報告、意外關聯、建議問題
├── graph.json        # 💾 持久化圖譜資料 (供 CLI/MCP 使用)
└── cache/            # ⚡ SHA256 快取 (增量重建用)
```

### 直接開啟 `graph.html` 即可在瀏覽器中探索你的知識圖譜！

---

## 🔌 AI 平台整合

### 平台支援矩陣

| 平台 | 狀態 | 安裝指令 |
|------|------|------|
| Claude Code | ✅ 已支援 | `graphify install --platform claude` |
| Codex CLI | ✅ 已支援 | `graphify install --platform codex` |
| Trae | ✅ 已支援 | `graphify install --platform trae` |
| Trae CN | ✅ 已支援 | `graphify install --platform trae-cn` |
| Opencode | ✅ 已支援 | `graphify install --platform opencode` |
| OpenClaw | ✅ 已支援 | `graphify install --platform claw` |
| Factory Droid | ✅ 已支援 | `graphify install --platform droid` |
| Cursor | ✅ 已支援 | `graphify cursor install` |
| Gemini CLI | ✅ 已支援 | `graphify gemini install` |

> **Codex 用戶注意**：需要在 `~/.codex/config.toml` 的 `[features]` 下開啟 `multi_agent = true` 以支援並行提取。

### Opencode 整合方案

Opencode 與 Codex、Qwen 共用 **方案 C：AGENTS.md 注入**，並額外使用 `.opencode/plugins/graphify.js` 插件實現 bash 鉤子。

**原理**：Opencode 會在啟動時自動讀取專案根目錄的 `AGENTS.md`，將其內容作為專案級別的系統指令。

**設定步驟**：

1. **執行安裝指令**：

```bash
graphify opencode install
```

此命令會自動：
- 寫入 graphify 規則到 `AGENTS.md`
- 建立 `.opencode/plugins/graphify.js`（tool.execute.before 鉤子）
- 註冊插件到 `opencode.json`

2. **驗證**：啟動 Opencode 後，詢問架構相關問題（例如：「這個專案的 auth 流程如何運作？」），Opencode 會優先讀取 `graphify-out/GRAPH_REPORT.md` 而非盲目搜尋檔案。

**優勢**：
- ✅ 一套設定，多平台共用（Codex、OpenCode、Qwen 等均讀取 `AGENTS.md`）
- ✅ 無需修改 Opencode 本身的設定檔
- ✅ 專案級別隔離，不同專案可有不同圖譜規則
- ✅ 額外 bash 鉤子：執行命令時自動提醒圖譜可用

#### ⚠️ Opencode MCP 配置注意

若手動編輯 `~/.opencode/opencode.json` 新增 MCP 伺服器時遇到 `Invalid input mcp.xxx` 錯誤，請檢查環境變數欄位名稱：

- **❌ 錯誤**：使用 `"env": { ... }`
- **✅ 正確**：使用 **`"environment": { ... }`**

這是因為 Opencode 的 Schema 強制要求使用完整單字 `environment`，若欄位名稱錯誤會導致設定檔驗證失敗。

### Qwen Code 整合方案

Qwen Code 原生支援讀取 `AGENTS.md`，因此 graphify 可透過 **方案 C：AGENTS.md 注入** 與 Qwen 整合，無需額外安裝指令。

#### 方案 C：AGENTS.md 注入（推薦 ✅）

**原理**：Qwen Code 會在啟動時自動讀取專案根目錄的 `AGENTS.md`，將其內容作為專案級別的系統指令。

**設定步驟**：

1. **確保專案根目錄存在 `AGENTS.md`**，並包含以下內容：

```markdown
## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `python3 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"` to keep the graph current
```

2. **自動安裝**：執行以下指令即可（與 Codex 共用）：

```bash
graphify codex install
```

此命令會自動：
- 寫入上述內容到 `AGENTS.md`
- 安裝 Codex 專屬的 PreToolUse hook（`.codex/hooks.json`）

3. **驗證**：啟動 Qwen Code 後，詢問架構相關問題（例如：「這個專案的 auth 流程如何運作？」），Qwen 會優先讀取 `graphify-out/GRAPH_REPORT.md` 而非盲目搜尋檔案。

**優勢**：
- ✅ 一套設定，多平台共用（Codex、Opencode、Qwen 等均使用 `AGENTS.md` 方案）
- ✅ 無需修改 Qwen Code 本身的設定檔
- ✅ 專案級別隔離，不同專案可有不同圖譜規則

#### 全域啟用（Opencode 選用）

若希望 **所有專案** 都啟用 graphify 規則（不限單一專案），可在 Opencode 的全域規則檔 `~/.opencode/AGENTS.md` 中加入相同區塊：

```markdown
## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `python3 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"` to keep the graph current
```

> **注意**：全域啟用後，Opencode 會在「所有專案」中嘗試讀取 `graphify-out/` 目錄。若某個專案沒有圖譜，該規則不會觸發錯誤，但可能產生多餘的檔案檢查。

#### 未來官方支援（規劃中）

若 graphify 套件未來新增 `graphify qwen install` 指令，將會：
1. 在 `_PLATFORM_CONFIG` 新增 `"qwen"` 条目
2. 建立 `skill-qwen.md` 檔案（若 Qwen 支援 Skill 工具機制）
3. 安裝到 `~/.qwen/skills/graphify/SKILL.md` 或寫入專案的 `QWEN.md`

目前使用 `AGENTS.md` 方案已完全滿足日常開發需求。

---

### Opencode 與 Qwen 差異比較

| 平台 | AGENTS.md | 額外插件/鉤子 | 實作細節 |
|------|-----------|--------------|----------|
| **Qwen** | ✅ | ❌ 無 | 純靠 `AGENTS.md` 規則 |
| **Opencode** | ✅ | ✅ `.opencode/plugins/graphify.js` | 修改 bash 命令，注入 `echo` 提示 |
| **Codex** | ✅ | ✅ `.codex/hooks.json` | PreToolUse hook，返回 `systemMessage` |

#### 鉤子實作機制說明

**Opencode 的 bash 鉤子**：
```javascript
// .opencode/plugins/graphify.js
export const GraphifyPlugin = async ({ directory }) => {
  return {
    "tool.execute.before": async (input, output) => {
      if (input.tool === "bash") {
        output.args.command =
          'echo "[graphify] Knowledge graph available..." && ' +
          output.args.command;
      }
    },
  };
};
```
- **實作方式**：直接修改 bash 命令字串，在前面加上 `echo` 提示
- **觸發時機**：每次執行 bash 命令前（僅第一次會話）
- **效果**：終端機會顯示提示訊息，然後執行原始命令

**Codex 的 PreToolUse hook**：
```json
// .codex/hooks.json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "echo '{\"systemMessage\":\"graphify: ...\"}'"
      }]
    }]
  }
}
```
- **實作方式**：透過 hook 返回 `systemMessage`，不修改命令本身
- **觸發時機**：在 bash 工具執行前，作為系統訊息注入
- **效果**：AI 助手收到系統訊息，終端機不顯示額外輸出

兩者目的相同（提醒 AI 查閱圖譜），但因平台架構不同而採用不同實作方式。

---

### Claude Code

```bash
graphify claude install    # 啟用 Always-on 模式
graphify claude disable    # 停用
```

`/graphify` 是一個 **Skill** 的 Claude Code 擴展，將完整 9 步知識圖譜流程自動化為單一指令。

**Available commands in Claude Code / 在 Claude Code 中可用的指令：**

```bash
/graphify .                      # Full analysis of current directory / 完整分析當前目錄
/graphify . --update             # Incremental update (only changed files) / 增量更新（只分析改變的檔案）
/graphify . --update --mode deep # Deep semantic extraction / 深度語義提取
```

**How it works / 運作方式：**

1. **Automatic installation / 自動安裝**：首次使用時，Skill 會自動下載到 `~/.claude/skills/graphify/`。
2. **One-command pipeline / 單一命令流程**：執行完整 9 步（detection、extraction、clustering、visualization）。
3. **AI-friendly / AI 友善**：適合 Agent 在程式碼變更後快速刷新架構理解。

**Direct command alternative (not recommended for Claude Code) / 直接命令方式（Claude Code 中不建議）：**

```bash
python -m graphify . --mode deep
```

建議在 Claude Code 中優先使用 `/graphify` 以取得最佳體驗。

### Codex CLI

```bash
graphify codex install     # 安裝鉤子 (指令為 $graphify)
```
`graphify codex install` 執行結果(自動更新 AGENTS.md 與 hook)：
```bash
graphify codex install                                                                                            ✔  at 09:30:30 
graphify already configured in AGENTS.md
  .codex/hooks.json  ->  hook already registered (no change)

Codex will now check the knowledge graph before answering
codebase questions and rebuild it after code changes.
```

#### Codex 專案規則（graphify）

- 回答架構或程式碼問題前，先讀取 `graphify-out/GRAPH_REPORT.md`（了解 god nodes 與社群結構）。
- 若存在 `graphify-out/wiki/index.md`，優先走 Wiki 導覽，不直接掃描原始檔。
- 本次會話若有修改程式碼檔，完成後執行以下指令同步圖譜：

```bash
python3 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"
```

> Windows + Codex 常見問題（cp950 編碼）
>
> 若在 Windows PowerShell 看到 `Rebuild failed: 'cp950' codec can't encode character ...`，先切換 UTF-8 再執行重建：
>
> ```powershell
> chcp 65001
> $env:PYTHONUTF8 = "1"
> $env:PYTHONIOENCODING = "utf-8"
> [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
> python -X utf8 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"
> ```
>
> 若使用 `python3`，將最後一行改為 `python3 -X utf8 ...`。

### Cursor

```bash
graphify cursor install    # 注入專案規則
```

### Gemini CLI

```bash
graphify gemini install    # 生成對應規則
```

安裝後，AI 助手會自動在搜尋檔案前讀取圖譜，大幅提升理解效率。

---

## 🛠️ 技術架構

### 技術棧

| 層級 | 技術 | 用途 |
|------|------|------|
| 語法解析 | `tree-sitter` | 20+ 種語言的 AST 提取 |
| 圖計算 | `NetworkX` | 圖譜資料結構與演算法 |
| 社群分群 | `graspologic` | Leiden 社群偵測 |
| 視覺化 | `vis.js` | 互動式 HTML 圖譜 |
| AI 提取 | Anthropic/OpenAI API | 非程式碼檔案的語意分析 |
| 通訊協定 | MCP | 標準 IO 伺服器暴露圖譜 |

### 核心流程

```
┌─────────────┐
│  多模態輸入   │  (程式碼、文件、圖片、筆記...)
└──────┬──────┘
       │
  ┌────┴────┐
  │  提取器  │
  ├─────────┤
  │ AST 解析 │ ← 本地執行，100% 離線
  │ LLM 提取 │ ← 平行 Subagents，需 API Key
  └────┬────┘
       │
  ┌────┴──────┐
  │ 圖譜引擎   │
  ├───────────┤
  │ 節點/邊合併│
  │ 超邊組裝   │
  │ 置信度評分 │
  │ Leiden 分群│
  └────┬──────┘
       │
  ┌────┴─────────┐
  │  輸出與快取   │
  ├──────────────┤
  │ HTML/JSON/MD │
  │ SHA256 快取  │
  │ Git Hooks    │
  └──────────────┘
```

---

## 📊 視覺化範例

開啟 `graphify-out/graph.html` 後，你可以：

- 🔍 **搜尋節點**: 快速定位特定類別或函數
- 🖱️ **點擊探索**: 查看節點的所有關聯邊
- 🎨 **社群過濾**: 僅顯示特定分群的節點
- 🔗 **路徑追蹤**: 找出兩個概念間的關聯路徑

---

## 🎯 適用場景

| 場景 | 說明 |
|------|------|
| 接手舊專案 | 3 分鐘看懂整體架構 |
| 程式碼審查 | 視覺化模組依賴與耦合 |
| AI 輔助開發 | 大幅降低 Token 消耗 |
| 技術文件生成 | 自動產出架構圖與關聯報告 |
| 學術論文分析 | 提取概念網絡與引用關係 |
| Obsidian 筆記 | 匯出知識庫圖譜 |

---

## 📈 實測案例 (Worked Examples)

| 語料規模 | 檔案數 | Token 壓縮比 | 範例輸出 |
|------|--------|--------|------|
| Karpathy 倉庫 + 5 篇論文 + 4 張圖片 | 52 | **71.5x** | `worked/karpathy-repos/` |
| graphify 源碼 + Transformer 論文 | 4 | **5.4x** | `worked/mixed-corpus/` |
| httpx (合成 Python 庫) | 6 | **~1x** | `worked/httpx/` |

> Token 壓縮效果隨語料規模增大而顯著。在 52 個混合檔案規模下，可節省超過 70 倍的 Token 消耗。

---

## 📝 常見問題

### Q: 一定要用 API Key 嗎？
A: 程式碼解析完全不需要。只有分析文件、圖片時才需要 LLM。

### Q: 支援哪些程式語言？
A: 20+ 種，包含 Python、JavaScript、TypeScript、Go、Rust、C++、Java、PHP、Ruby 等。

### Q: 處理速度如何？
A: 純 AST 解析極快；LLM 部分採用平行處理，速度取決於 API 回應時間。

### Q: 大型專案會很慢嗎？
A: 搭配 `--update` 增量模式或 `--watch` 背景同步，僅處理變更檔案，效率極高。

---

## 📄 授權

本專案採用 [MIT License](LICENSE) 開放原始碼授權。

---

## 🔗 原始專案

- **原作者**: [Safi Shamsi](https://github.com/safishamsi/graphify)
- **參考版本**: [v3 繁體/簡體中文 README](https://github.com/safishamsi/graphify/blob/v3/README.zh-CN.md)
- **PyPI**: `graphifyy` (注意多一個 y)

---

## 🌟 結語

> Graphify 不僅是工具，更是理解複雜系統的思維框架。  
> 它將「程式碼 → 結構 → 語意 → 洞察」的過程自動化，  
> 讓開發者專注在真正重要的事：做對的決策。

---

*最後更新: 2026-04-10*

