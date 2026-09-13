# GitHub 语言统计使用说明

本文说明个人主页仓库 `xikao-hx/xikao-hx` 中语言统计卡片的配置、运行和维护流程。

## 工作原理

仓库通过 `.github/workflows/language-metrics.yml` 调用 `lowlighter/metrics`，统计 GitHub 公开仓库中实际检测到的语言，并生成：

```text
assets/languages.svg
```

个人主页的 `README.md` 直接引用该 SVG。工作流最多展示 8 种语言，并显示代码量和占比。

工作流支持两种触发方式：

- 在 GitHub Actions 页面手动运行；
- 每周一 UTC 03:17 自动运行，即北京时间周一 11:17。

## 首次配置

### 1. 创建 Personal Access Token

打开 GitHub 的 [Personal Access Token (classic) 创建页面](https://github.com/settings/tokens/new)，或者依次进入：

```text
GitHub 右上角头像
→ Settings
→ Developer settings
→ Personal access tokens
→ Tokens (classic)
→ Generate new token
→ Generate new token (classic)
```

建议填写：

```text
Note: GitHub profile language metrics
Expiration: 90 days
```

本工作流只统计公开仓库，因此 `Select scopes` 中的权限可以全部不勾选。点击 `Generate token` 后，立即复制生成的 Token；Token 通常以 `ghp_` 开头，并且只显示一次。

> Token 等同于密码。不得将其写入 README、工作流源码、Git 提交、终端截图或聊天内容。

### 2. 创建仓库 Secret

进入仓库：

```text
xikao-hx/xikao-hx
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

填写：

```text
Name: METRICS_TOKEN
Secret: 粘贴上一步创建的 Personal Access Token
```

点击 `Add secret`。创建成功后，页面会显示名为 `METRICS_TOKEN` 的 Repository secret，但不会再次显示其内容。

### 3. 配置工作流写权限

进入：

```text
Settings
→ Actions
→ General
→ Workflow permissions
```

选择 `Read and write permissions` 并保存。该权限用于将新生成的 `assets/languages.svg` 提交回当前仓库。

## 手动生成统计图

打开仓库顶部的 `Actions` 页面，然后执行：

```text
Language metrics
→ Run workflow
→ Branch: main
→ Run workflow
```

也可以直接打开：

```text
https://github.com/xikao-hx/xikao-hx/actions/workflows/language-metrics.yml
```

运行期间状态为 `In progress`。成功后应满足：

1. 工作流显示绿色勾号；
2. `assets/languages.svg` 被自动更新；
3. 仓库出现 `chore: update language metrics [skip ci]` 提交；
4. 刷新 GitHub 个人主页后能看到真实语言统计。

GitHub 或浏览器可能缓存图片。如果工作流已成功但个人主页仍显示旧图，可以等待几分钟后强制刷新页面。

## 调整统计显示

统计参数位于 `.github/workflows/language-metrics.yml`：

```yaml
plugin_languages_limit: 8
plugin_languages_threshold: 0.1%
plugin_languages_other: yes
plugin_languages_details: bytes-size, percentage
```

参数含义：

| 参数 | 说明 |
| --- | --- |
| `plugin_languages_limit` | 最多展示的语言数量，允许范围为 0～8 |
| `plugin_languages_threshold` | 隐藏占比低于该阈值的语言 |
| `plugin_languages_other` | 将未单独展示的语言合并为 `Other` |
| `plugin_languages_details` | 显示代码量和百分比 |

修改工作流后，需要提交并推送，再手动运行一次工作流：

```bash
git add .github/workflows/language-metrics.yml
git commit -m "调整语言统计配置"
git push
```

## Token 到期或更新

Token 到期后，自动更新会认证失败。重新创建 Token 后进入：

```text
Settings
→ Secrets and variables
→ Actions
→ Repository secrets
→ METRICS_TOKEN
→ Update secret
```

粘贴新 Token 并保存，然后手动运行一次 `Language metrics` 验证。

不再使用该功能时，应同时删除仓库中的 `METRICS_TOKEN` 和 GitHub 账户中的对应 Personal Access Token。

## 常见问题

### Actions 页面没有 `Language metrics`

确认 `.github/workflows/language-metrics.yml` 已提交并推送到默认分支 `main`，并检查：

```text
Settings → Actions → General → Actions permissions
```

仓库必须允许执行 `lowlighter/metrics` 第三方 Action。

### 提示找不到 `METRICS_TOKEN`

确认 Secret 创建在 `Repository secrets`，名称必须严格为 `METRICS_TOKEN`。不要只创建 GitHub Personal Access Token 而漏掉仓库 Secret。

### 工作流能生成 SVG，但无法提交

确认 `Workflow permissions` 已设置为 `Read and write permissions`。如果 `main` 分支启用了保护规则，还需要允许 GitHub Actions 写入该分支，或者调整工作流的产物提交方式。

### 只显示少量语言

语言由 GitHub Linguist 和公开仓库内容自动识别，不能通过填写名称伪造统计结果。可以检查：

- 相关仓库是否为公开仓库；
- 语言文件是否已提交并推送；
- 语言占比是否低于 `plugin_languages_threshold`；
- 是否超过 `plugin_languages_limit` 限制。

### 个人主页显示占位图

仓库初始的 `assets/languages.svg` 是避免图片加载失败的占位图。完成首次配置并成功运行工作流后，它会被真实统计图覆盖。

## 相关文件

| 文件 | 用途 |
| --- | --- |
| `.github/workflows/language-metrics.yml` | GitHub Actions 工作流与统计参数 |
| `assets/languages.svg` | 自动生成的语言统计图 |
| `README.md` | GitHub 个人主页及统计图引用 |

## 参考资料

- [Metrics Languages 插件](https://github.com/lowlighter/metrics/blob/master/source/plugins/languages/README.md)
- [GitHub Personal Access Token 文档](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub Actions Secrets 文档](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
