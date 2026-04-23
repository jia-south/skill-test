# skill-test

本仓库用于测试 **Skills Management Platform** 的 GitHub 同步功能（**release 模式**）。

在 release 模式下，平台通过扫描 GitHub Releases 的附件（assets）来发现并同步 skill。每次发布新 skill 或更新已有 skill，只需将打包好的 `.zip` 文件作为 Release asset 上传即可，平台会在下次定时同步时自动将其拉取入库。

---

## 仓库目录结构

本仓库保留 skill 源文件，方便版本对比和 PR review。打包后的 zip 通过 GitHub Release 发布。

```
skill-test/
├── README.md
└── skills/                        # 所有 skill 源文件
    ├── hello-world/
    │   ├── SKILL.md               # 必须，skill 元数据 + 使用说明
    │   └── scripts/
    │       └── main.py            # 可选，执行脚本
    └── data-analyzer/
        ├── SKILL.md
        ├── scripts/
        │   └── run.py
        └── requirements.txt       # 可选，Python 依赖
```

> `skills/` 目录仅作为源码管理用途，平台**不会**直接扫描此目录。平台只识别 GitHub Release 中的 `.zip` 附件。

---

## Skill 压缩包规范

### 命名规则

```
{skill-name}-{version}.zip
```

| 示例 | 说明 |
|------|------|
| `hello-world-1.0.0.zip` | skill 名 + 版本号，中间用连字符拼接 |
| `data-analyzer-2.1.0.zip` | 版本号遵循 semver，不带 `v` 前缀 |

> 版本号格式：`主版本.次版本.修订号`，支持预发布后缀，如 `1.0.0-beta.1`。

### zip 内部结构

**SKILL.md 必须位于 zip 的根目录**，不能套父文件夹。

```
hello-world-1.0.0.zip
├── SKILL.md               ← 必须，在根目录
├── scripts/               ← 可选
│   └── main.py
└── requirements.txt       ← 可选
```

错误示例（平台会拒绝）：
```
hello-world-1.0.0.zip
└── hello-world/           ← 不能有父目录包裹
    └── SKILL.md
```
 
---

## SKILL.md 格式

每个 zip 包内的 `SKILL.md` 必须以 YAML frontmatter 开头，frontmatter 中 `name`、`version`、`description` 为**必填字段**。

```markdown
---
name: hello-world
version: "1.0.0"
description: A simple greeting skill that outputs Hello World.
author: your-github-username
category: demo
tags: ["hello", "greeting", "demo"]
---

## 使用说明

在此处写 skill 的详细说明、参数介绍、使用示例等内容。
```

### frontmatter 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | ✅ | skill 唯一标识，全小写，单词用连字符，需与 zip 文件名前缀一致 |
| `version` | ✅ | 语义化版本号，如 `1.0.0`，用引号包裹避免 YAML 解析歧义 |
| `description` | ✅ | 英文简短描述，一句话说明 skill 的功能 |
| `author` | 可选 | 作者名或 GitHub 用户名 |
| `category` | 可选 | 分类，如 `demo`、`data`、`automation` |
| `tags` | 可选 | 标签数组，用于搜索过滤 |

---

## 发布流程

### 第一步：打包 skill

将 skill 目录内的所有文件直接压缩，确保 `SKILL.md` 在 zip 根目录。

**Windows（PowerShell）：**
```powershell
cd skills/hello-world
Compress-Archive -Path * -DestinationPath ../../releases/hello-world-1.0.0.zip
```

**macOS / Linux：**
```bash
cd skills/hello-world
zip -r ../../releases/hello-world-1.0.0.zip .
```

### 第二步：创建 GitHub Release

1. 进入仓库页面，点击右侧 **Releases → Draft a new release**
2. 创建一个新 tag，建议命名为日期格式，如 `release-2026-04-23`
3. 在 **Assets** 区域上传打包好的 `.zip` 文件
4. 点击 **Publish release**

### 第三步：等待平台同步

平台会在下次定时同步时（根据订阅源配置的 `interval_hours`）自动：

1. 拉取所有 Release 的 assets 列表
2. 过滤出 `.zip` 文件
3. 下载 → 解压 → 校验结构 → AI 审核 → 入库

也可以在平台管理界面手动触发同步。

---

## 去重机制

平台以 `name + version`（来自 SKILL.md frontmatter）作为唯一键。

- 同一个 `name + version` 的 skill 只会入库一次，重复上传会被跳过（`skipped`）。
- 更新 skill 内容时，必须**同时升级 version**，否则平台视为已存在而跳过。

---

## 示例 skill

参考 [skills/hello-world/](skills/hello-world/) 目录获取一个最简 skill 的完整示例。
