> [!NOTE]
> 此 README 由 [Claude Code](https://gist.github.com/pardnchiu/b09c9bf1166ec7759cbbeeae2e4e93df) 生成，英文版請參閱 [這裡](./README.md)。

# readme-generate

[![license](https://img.shields.io/github/license/pardnchiu/readme-generate)](LICENSE)
[![version](https://img.shields.io/github/v/tag/pardnchiu/readme-generate?label=release)](https://github.com/pardnchiu/readme-generate/releases)

> 自動化雙語 README 文件產生器，透過原始碼分析自動生成專業的 README.md（英文）與 README.zh.md（繁體中文）文件。

## 功能特點

- **多語言專案分析**：支援 Go、Python、JavaScript/TypeScript、PHP、Swift 專案
- **自動化類型提取**：從原始碼中提取 struct、class、interface 等類型定義
- **函式簽名擷取**：自動識別 exported 函式並提取完整簽名
- **雙語文件產出**：同時產生英文與繁體中文版本的 README
- **智慧忽略機制**：自動跳過 `node_modules`、`vendor`、`__pycache__` 等目錄
- **依賴分析**：從 `go.mod`、`package.json`、`pyproject.toml` 提取相依套件

## 安裝

```bash
# 複製此 skill 到 Claude Code skills 目錄
cp -r readme-generate ~/.claude/skills/
```

## 使用方法

在 Claude Code 中執行：

```bash
# 基本用法 - 僅產生 README
/readme-generate

# 產生 README + MIT LICENSE
/readme-generate MIT

# Private 模式（無 badges 與 star history）
/readme-generate private

# 指定自訂 GitHub 路徑
/readme-generate github.com/your-org/your-repo

# 組合使用
/readme-generate private MIT github.com/your-org/your-repo
```

### 支援的 LICENSE 類型

| 類型 | 別名 |
|------|------|
| MIT | `mit` |
| Apache-2.0 | `apache`, `apache2`, `apache-2.0` |
| GPL-3.0 | `gpl`, `gpl3`, `gpl-3.0` |
| BSD-3-Clause | `bsd`, `bsd3`, `bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`, `public-domain` |
| Proprietary | `proprietary`（自動啟用 private 模式） |

## 命令列參考

### analyze_project.py

用於分析專案結構的獨立腳本（Script）。

```bash
python3 scripts/analyze_project.py /path/to/project
```

**輸出格式**：JSON

```json
{
  "language": "go",
  "name": "project-name",
  "description": "",
  "version": "1.0.0",
  "files": ["main.go", "lib/utils.go"],
  "types": [
    {
      "name": "Config",
      "kind": "struct",
      "fields": [{"name": "Host", "type": "string", "tag": "json:\"host\""}],
      "doc": "Config holds application settings",
      "file": "config.go"
    }
  ],
  "functions": [
    {
      "name": "NewConfig",
      "signature": "func NewConfig(path string) (*Config, error)",
      "doc": "NewConfig creates a new Config instance",
      "exported": true,
      "file": "config.go",
      "line": 0
    }
  ],
  "dependencies": ["github.com/pkg/errors"]
}
```

### 專案類型偵測

| 檔案指標 | 偵測語言 |
|----------|----------|
| `go.mod` | Go |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python |
| `package.json` | JavaScript |
| `tsconfig.json` | TypeScript |
| `composer.json` | PHP |
| `Package.swift` | Swift |

## 授權

此專案採用 MIT 授權條款，詳見 [LICENSE](LICENSE)。

## Author

<img src="https://avatars.githubusercontent.com/u/25631760" align="left" width="96" height="96" style="margin-right: 0.5rem;">

<h4 style="padding-top: 0">邱敬幃 Pardn Chiu</h4>

<a href="mailto:dev@pardn.io" target="_blank">
<img src="https://pardn.io/image/email.svg" width="48" height="48">
</a> <a href="https://linkedin.com/in/pardnchiu" target="_blank">
<img src="https://pardn.io/image/linkedin.svg" width="48" height="48">
</a>

## Stars

[![Star](https://api.star-history.com/svg?repos=pardnchiu/readme-generate&type=Date)](https://www.star-history.com/#pardnchiu/readme-generate&Date)

***

©️ 2026 [邱敬幃 Pardn Chiu](https://linkedin.com/in/pardnchiu)
