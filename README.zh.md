> [!NOTE]
> 此 README 由 [SKILL](https://github.com/pardnchiu/skill-readme-generate) 生成，英文版請參閱 [這裡](./README.md)。

# skill-readme-generate

[![license](https://img.shields.io/github/license/pardnchiu/skill-readme-generate)](LICENSE)

> Claude Code 技能（Skill），透過原始碼分析自動生成專業的雙語 README 文件。<br>
> 此專案主要由 [Claude Code](https://claude.ai/claude-code) 生成，作者僅做部分調整。

## 目錄

- [功能特點](#功能特點)
- [安裝](#安裝)
- [使用方法](#使用方法)
- [命令列參考](#命令列參考)
- [授權](#授權)

## 功能特點

- **雙語輸出**：同時生成 `README.md`（英文）與 `README.zh.md`（繁體中文）
- **原始碼分析**：自動解析專案結構，提取類型定義、函式簽章與相依套件
- **多語言支援**：支援 Go、Python、JavaScript、TypeScript、PHP、Swift 專案
- **LICENSE 生成**：支援 MIT、Apache-2.0、GPL-3.0、BSD-3-Clause、ISC、Unlicense 等授權類型
- **彈性參數**：可指定公開/私有模式、自訂 repo 路徑、授權類型
- **徽章自動產生**：根據專案語言自動產生對應的徽章（Badge）

## 安裝

將此技能放置於 Claude Code 的技能目錄：

```bash
~/.claude/skills/readme-generate/
```

目錄結構：

```
readme-generate/
├── SKILL.md              # 技能定義檔
├── scripts/
│   └── analyze_project.py  # 專案分析腳本
├── LICENSE
├── README.md
└── README.zh.md
```

## 使用方法

### 基本語法

```bash
/readme-generate [private] [LICENSE_TYPE] [REPO_PATH]
```

### 參數說明

| 參數 | 格式 | 範例 | 說明 |
|------|------|------|------|
| `private` | 關鍵字 | `private` | 生成不含徽章與 Star History 的版本 |
| `LICENSE_TYPE` | 授權識別碼 | `MIT`、`Apache-2.0` | 同時生成 LICENSE 檔案 |
| `REPO_PATH` | `github.com/{owner}/{repo}` | `github.com/foo/bar` | 覆寫預設的 owner/repo |

### 使用範例

```bash
# 僅生成 README（公開模式）
/readme-generate

# 生成 README + MIT LICENSE
/readme-generate MIT

# 私有模式（不含徽章與 Star History）
/readme-generate private

# 私有模式 + MIT LICENSE
/readme-generate private MIT

# 指定自訂 repo 路徑
/readme-generate github.com/foo/bar

# 完整參數
/readme-generate private MIT github.com/foo/bar
```

## 命令列參考

### 支援的授權類型

| 類型 | 別名（不區分大小寫） |
|------|---------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`、`apache2`、`apache-2.0` |
| GPL-3.0 | `gpl`、`gpl3`、`gpl-3.0` |
| BSD-3-Clause | `bsd`、`bsd3`、`bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`、`public-domain` |
| Proprietary | `proprietary`（自動啟用私有模式） |

### 分析腳本

```bash
python3 scripts/analyze_project.py /path/to/project
```

輸出 JSON 格式，包含：
- `language`：專案語言
- `name`：專案名稱
- `version`：版本號
- `types`：類型定義
- `functions`：函式簽章
- `dependencies`：相依套件

## 授權

本專案採用 [MIT LICENSE](LICENSE)。
