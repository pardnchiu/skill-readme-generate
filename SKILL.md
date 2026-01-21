---
name: readme-generate
description: Automated bilingual README generation from source code analysis. Use when user requests README creation for a project, needs to generate README.md (English) and README.zh.md (Chinese) from codebase, or wants consistent documentation across multiple languages for their library/package.
---

# README Generator

Generate professional bilingual README documentation by analyzing project source code.

## Command Syntax

```
/readme-generate [private] [LICENSE_TYPE] [REPO_PATH]
```

### Parameters (All Optional)

| Parameter | Format | Example | Behavior |
|-----------|--------|---------|----------|
| `private` | Keyword | `private` | Generate without badges and star history |
| `LICENSE_TYPE` | License identifier | `MIT`, `Apache-2.0` | Generate LICENSE file |
| `REPO_PATH` | `github.com/{owner}/{repo}` | `github.com/foo/bar` | Override default owner/repo |

### Parameter Detection Rules

| Pattern | Detected As |
|---------|-------------|
| `private` (case-insensitive) | `PRIVATE_MODE` flag |
| Contains `github.com/` | `REPO_PATH` |
| Matches known license type (case-insensitive) | `LICENSE_TYPE` |

**Order-independent**: Parameters can appear in any order.

### Supported LICENSE Types

| Type | Aliases (case-insensitive) |
|------|----------------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`, `apache2`, `apache-2.0` |
| GPL-3.0 | `gpl`, `gpl3`, `gpl-3.0` |
| BSD-3-Clause | `bsd`, `bsd3`, `bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`, `public-domain` |
| Proprietary | `proprietary` (implies `private` mode) |

### Examples

```bash
/readme-generate                              # README only, public mode
/readme-generate MIT                          # README + MIT LICENSE
/readme-generate private                      # README without badges/star history
/readme-generate private MIT                  # Private README + MIT LICENSE
/readme-generate proprietary                  # Private README + Proprietary LICENSE
/readme-generate github.com/foo/bar           # README with custom repo path
/readme-generate private github.com/foo/bar   # Private README + custom path
```

---

## CRITICAL: Required Outputs

**ALWAYS generate BOTH files - this is mandatory:**

| File | Language | Purpose |
|------|----------|---------|
| `README.md` | English | Primary documentation |
| `README.zh.md` | Traditional Chinese (ZH-TW) | Chinese documentation |

**Never generate only one file. Both files MUST be created and saved to the project directory.**

---

## Params (Extract from Project)

| Param | Source | Example |
|-------|--------|---------|
| `{owner}` | `REPO_PATH` override OR Fixed value | `pardnchiu` |
| `{author_name}` | Fixed value | `邱敬幃 Pardn Chiu` |
| `{author_url}` | Fixed value | `https://linkedin.com/in/pardnchiu` |
| `{repo}` | `REPO_PATH` override OR Folder name or `git remote get-url origin` | `go-scheduler` |
| `{package}` | `package.json` name, `go.mod` module, `pyproject.toml` name | `@aspect/utils` |
| `{year}` | Existing README year OR `git log --reverse --format=%ai \| head -1` OR current year | `2024` |

**Priority**: Command-line `REPO_PATH` > Local git remote > Folder name

---

## Workflow

```
1. Parse      →  Extract PRIVATE_MODE, LICENSE_TYPE, REPO_PATH from command
2. Analyze    →  Run analyze_project.py on target project
3. Extract    →  Get {repo}, {package}, {year} from project (or use REPO_PATH override)
4. Review     →  Check existing docs, LICENSE, examples
5. Generate   →  Create README.zh.md FIRST (Chinese)
6. Translate  →  Create README.md from Chinese version
7. License    →  [If LICENSE_TYPE specified] Generate LICENSE file
8. Validate   →  Verify all required sections exist
9. Save       →  Write files to project root
```

---

## Step 1: Analyze Project

```bash
python3 /mnt/skills/user/readme-generator/scripts/analyze_project.py /path/to/project
```

Output: JSON with language, name, version, types, functions, dependencies.

---

## Step 2: Extract Params

```bash
# If REPO_PATH provided, parse it:
# github.com/owner/repo → {owner}=owner, {repo}=repo

# Otherwise fallback to:

# Get repo name
basename $(pwd)
# OR
git remote get-url origin | sed 's/.*\/\([^\/]*\)\.git/\1/'

# Get first commit year
git log --reverse --format=%ai | head -1 | cut -d'-' -f1

# Get package name (Node.js)
jq -r '.name' package.json

# Get module name (Go)
grep '^module' go.mod | awk '{print $2}'
```

---

## Step 3: Generate Chinese Version FIRST

**Create `README.zh.md` with EXACT section order:**

### Section Order (MANDATORY)

| Order | Section | Required | Public Mode | Private Mode |
|-------|---------|----------|-------------|--------------|
| 0 | LLM generation notice | **YES** | ✓ | ✓ |
| 1 | Cover image | No | ✓ | ✓ |
| 2 | Title + badges | **YES** | Title + badges | **Title only** |
| 3 | Brief description | **YES** | ✓ | ✓ |
| 4 | Table of contents | No | ✓ | ✓ |
| 5 | Core features | **YES** | ✓ | ✓ |
| 6 | Architecture / Flowchart | No | ✓ | ✓ |
| 7 | Installation | **YES** | ✓ | ✓ |
| 8 | Usage examples | **YES** | ✓ | ✓ |
| 9 | Reference | **YES** | ✓ | ✓ |
| 10 | Use cases / Scenarios | No | ✓ | ✓ |
| 11 | License | **YES** | ✓ | ✓ |
| 12 | Author | **YES** | ✓ | ✓ |
| 13 | Star History | No | ✓ | **SKIP** |
| 14 | Copyright footer | No | ✓ | ✓ |

---

## MANDATORY SECTIONS (Copy Exactly)

### Order 0: LLM Generation Notice

**English (README.md):**
```markdown
> [!NOTE]
> This README was generated by [Claude Code](https://gist.github.com/pardnchiu/b09c9bf1166ec7759cbbeeae2e4e93df), get the ZH version from [here](./README.zh.md).
```

**Chinese (README.zh.md):**
```markdown
> [!NOTE]
> 此 README 由 [Claude Code](https://gist.github.com/pardnchiu/b09c9bf1166ec7759cbbeeae2e4e93df) 生成，英文版請參閱 [這裡](./README.md)。
```

### Order 2: Title Section

**Public Mode (with badges):**
```markdown
# {repo}

[![pkg](https://pkg.go.dev/badge/github.com/{owner}/{repo}.svg)](https://pkg.go.dev/github.com/{owner}/{repo})
[![license](https://img.shields.io/github/license/{owner}/{repo})](LICENSE)
```

**Private Mode (title only):**
```markdown
# {repo}
```

### Order 12: Author Section (NEVER TRANSLATE OR MODIFY)

**Use this EXACT format in BOTH files:**
```markdown
## Author

<img src="https://avatars.githubusercontent.com/u/25631760" align="left" width="96" height="96" style="margin-right: 0.5rem;">

<h4 style="padding-top: 0">邱敬幃 Pardn Chiu</h4>

<a href="mailto:dev@pardn.io" target="_blank">
<img src="https://pardn.io/image/email.svg" width="48" height="48">
</a> <a href="https://linkedin.com/in/pardnchiu" target="_blank">
<img src="https://pardn.io/image/linkedin.svg" width="48" height="48">
</a>
```

### Order 13: Star History Section (PUBLIC MODE ONLY)

**Skip entirely in private mode.**

```markdown
## Stars

[![Star](https://api.star-history.com/svg?repos={owner}/{repo}&type=Date)](https://www.star-history.com/#{owner}/{repo}&Date)
```

### Order 14: Copyright Footer

```markdown
***

©️ {year} [{author_name}]({author_url})
```

---

## Badge Templates by Language (PUBLIC MODE ONLY)

### Go
```markdown
[![pkg](https://pkg.go.dev/badge/github.com/{owner}/{repo}.svg)](https://pkg.go.dev/github.com/{owner}/{repo})
[![card](https://goreportcard.com/badge/github.com/{owner}/{repo})](https://goreportcard.com/report/github.com/{owner}/{repo})
[![codecov](https://img.shields.io/codecov/c/github/{owner}/{repo}/master)](https://app.codecov.io/github/{owner}/{repo}/tree/master)
```

### Node.js
```markdown
[![npm](https://img.shields.io/npm/v/{package})](https://www.npmjs.com/package/{package})
[![downloads](https://img.shields.io/npm/dm/{package})](https://www.npmjs.com/package/{package})
```

### Python
```markdown
[![pypi](https://img.shields.io/pypi/v/{package})](https://pypi.org/project/{package})
[![python](https://img.shields.io/pypi/pyversions/{package})](https://pypi.org/project/{package})
```

### Universal (Always Include in Public Mode)
```markdown
[![license](https://img.shields.io/github/license/{owner}/{repo})](LICENSE)
[![version](https://img.shields.io/github/v/tag/{owner}/{repo}?label=release)](https://github.com/{owner}/{repo}/releases)
```

---

## Translation Guidelines

### ZH-TW Conventions (README.zh.md)

| Element | Rule | Example |
|---------|------|---------|
| Technical terms | English + Chinese annotation (first use) | `Worker 池（Pool）` |
| Subsequent uses | Chinese only | `Worker 池` |
| Function names | Keep original | `Enqueue()`, `Shutdown()` |
| Code blocks | Unchanged, translate comments only | `// 啟動佇列` |
| Section headers | Translate | `## 安裝` for `## Installation` |

### EN Conventions (README.md)

| Rule | Example |
|------|---------|
| Active voice | "The queue processes tasks" not "Tasks are processed" |
| Imperative mood | "Run the command" not "You should run" |
| Consistent terminology | Same term throughout document |
| No parenthetical translations | No "(佇列)" after "queue" |

---

## Code Block Guidelines

- Always specify language identifier for syntax highlighting
- Include import/require statements
- Keep examples minimal but complete
- Production examples must include error handling
- Comments translated in ZH version, code unchanged

---

## Mermaid Diagram Types

| Type | Directive | Use Case |
|------|-----------|----------|
| Flowchart (TB) | `graph TB` | Top-to-bottom flow |
| Flowchart (LR) | `graph LR` | Left-to-right flow |
| Sequence | `sequenceDiagram` | Interaction sequences |
| State Machine | `stateDiagram` | State transitions |
| Class Diagram | `classDiagram` | Type relationships |

---

## Reference Section Guidelines

| Project Type | Reference Content |
|--------------|-------------------|
| Library/SDK | Exported types, functions, methods with signatures |
| CLI Tool | Commands table, flags/options, environment variables |
| Framework | Lifecycle hooks, middleware interface, plugin API |
| Config-based | Config file schema with defaults and validation |
| Desktop App | Preferences, shortcuts, scripting API (if any) |
| Embedded/IoT | Communication protocols, hardware interface specs |

### Detection Heuristics
- `main()` + `flag`/`cobra`/`argparse` → CLI Tool
- Only exported types, no `main()` → Library
- `plugin`/`middleware`/`hook` patterns → Framework
- `.config.json`/`.yaml` templates → Config-driven

### Reference Section Titles by Project Type

| Project Type | EN Title | ZH-TW Title |
|--------------|----------|-------------|
| Library/SDK | `## API Reference` | `## API 參考` |
| CLI Tool | `## CLI Reference` | `## 命令列參考` |
| Framework | `## Interface Reference` | `## 介面參考` |
| Config-based | `## Configuration Reference` | `## 設定參考` |
| Desktop App | `## Preferences Reference` | `## 偏好設定參考` |
| Embedded/IoT | `## Protocol Reference` | `## 協定參考` |

---

## LICENSE Generation (Optional)

**Only executed if `LICENSE_TYPE` parameter is provided.**

### LICENSE Templates

#### MIT
```
MIT License

Copyright (c) {year} {author_name}

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

#### Apache-2.0
```
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION
[Full Apache 2.0 text...]

Copyright {year} {author_name}

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
```

#### GPL-3.0
```
                    GNU GENERAL PUBLIC LICENSE
                       Version 3, 29 June 2007

Copyright (C) 2007 Free Software Foundation, Inc. <https://fsf.org/>
[Full GPL 3.0 text from https://www.gnu.org/licenses/gpl-3.0.txt]

Copyright (C) {year} {author_name}
```

#### BSD-3-Clause
```
BSD 3-Clause License

Copyright (c) {year}, {author_name}
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

#### ISC
```
ISC License

Copyright (c) {year}, {author_name}

Permission to use, copy, modify, and/or distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```

#### Unlicense
```
This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <https://unlicense.org>
```

#### Proprietary (Auto-enables Private Mode)
```
Proprietary License

Copyright (c) {year} {author_name}. All rights reserved.

This software and associated documentation files (the "Software") are
proprietary and confidential. Unauthorized copying, modification,
distribution, or use of this Software, via any medium, is strictly
prohibited.

The Software is provided for internal use only and may not be shared
with third parties without prior written consent from the copyright
holder.

For licensing inquiries, contact: dev@pardn.io
```

---

## Validation Checklist (MUST PASS ALL)

Before completing, verify:

- [ ] `README.zh.md` created and saved
- [ ] `README.md` created and saved
- [ ] **Order 0**: LLM generation notice present (FIRST LINE after any cover image)
- [ ] **Order 2**: Title present; badges included (public) OR omitted (private)
- [ ] **Order 3**: Brief description in blockquote format
- [ ] **Order 5**: Core features section exists
- [ ] **Order 7**: Installation section exists
- [ ] **Order 8**: Usage examples section exists
- [ ] **Order 11**: License section exists
- [ ] **Order 12**: Author section uses EXACT fixed format (not translated)
- [ ] **Order 13**: Star History included (public) OR omitted (private)
- [ ] All `{owner}`, `{repo}`, `{package}`, `{year}` placeholders replaced
- [ ] Both files have identical section structure
- [ ] All code blocks match (except comments language)
- [ ] [If LICENSE_TYPE specified] LICENSE file exists with correct content
- [ ] [If REPO_PATH specified] All URLs use overridden owner/repo

---

## Example Output Structure

### Public Mode

**README.zh.md:**
```markdown
> [!NOTE]
> 此 README 由 Claude Code 生成，英文版請參閱 [這裡](./README.md)。

# {repo}

[![pkg](https://pkg.go.dev/badge/github.com/pardnchiu/{repo}.svg)](https://pkg.go.dev/github.com/pardnchiu/{repo})
[![license](https://img.shields.io/github/license/pardnchiu/{repo})](LICENSE)

> 簡短描述（1-2 句）

## 功能特點
...

## 安裝
...

## 使用方法
...

## {Reference Section Title}
...

## 授權

MIT License

## Author

<img src="https://avatars.githubusercontent.com/u/25631760" align="left" width="96" height="96" style="margin-right: 0.5rem;">

<h4 style="padding-top: 0">邱敬幃 Pardn Chiu</h4>

<a href="mailto:dev@pardn.io" target="_blank">
<img src="https://pardn.io/image/email.svg" width="48" height="48">
</a> <a href="https://linkedin.com/in/pardnchiu" target="_blank">
<img src="https://pardn.io/image/linkedin.svg" width="48" height="48">
</a>

## Stars

[![Star](https://api.star-history.com/svg?repos=pardnchiu/{repo}&type=Date)](https://www.star-history.com/#pardnchiu/{repo}&Date)

***

©️ {year} [邱敬幃 Pardn Chiu](https://linkedin.com/in/pardnchiu)
```

### Private Mode

**README.zh.md:**
```markdown
> [!NOTE]
> 此 README 由 Claude Code 生成，英文版請參閱 [這裡](./README.md)。

# {repo}

> 簡短描述（1-2 句）

## 功能特點
...

## 安裝
...

## 使用方法
...

## {Reference Section Title}
...

## 授權

MIT License

## Author

<img src="https://avatars.githubusercontent.com/u/25631760" align="left" width="96" height="96" style="margin-right: 0.5rem;">

<h4 style="padding-top: 0">邱敬幃 Pardn Chiu</h4>

<a href="mailto:dev@pardn.io" target="_blank">
<img src="https://pardn.io/image/email.svg" width="48" height="48">
</a> <a href="https://linkedin.com/in/pardnchiu" target="_blank">
<img src="https://pardn.io/image/linkedin.svg" width="48" height="48">
</a>

***

©️ {year} [邱敬幃 Pardn Chiu](https://linkedin.com/in/pardnchiu)
```
