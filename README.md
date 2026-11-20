# Personal Blog&CloudDrive
Learning how to use Github and QuantumultX

## Google IP 规则集

Clash、Loon 和 Quantumult X 的规则集由同一份 [Google 官方 `goog.json`](https://www.gstatic.com/ipranges/goog.json) 生成，包含其中发布的 IPv4 和 IPv6 CIDR（也覆盖 Google Cloud 客户可用的部分地址段）。

### Clash Verge Rev

[`GoogleIP.list`](ProxyConfig/Clash/rule/GoogleIP.list) 使用 `no-resolve`，匹配连接已有的目标 IP，也可匹配前置规则已经解析出的 IP。

在 Clash Verge Rev 配置的 `rule-providers` 下添加与 `Google` 同级的 `GoogleIP`。

```yaml
rule-providers:
  GoogleIP:
    type: http
    behavior: classical
    format: text
    url: https://raw.githubusercontent.com/SwordZYZ/ProxyRules/main/ProxyConfig/Clash/rule/GoogleIP.list
    path: ./ruleset/GoogleIP.list
    interval: 86400
```

在 `rules` 中，将 `GoogleIP` 放在现有 `Google` 条目紧下一行。两条规则使用同一个 `Google` 策略组。

```yaml
  - RULE-SET,Google,Google
  - RULE-SET,GoogleIP,Google
```

`GoogleIP` 已覆盖 `142.251.157.1` 和 `2001:4860:4844:400::`。删除配置开头单独指向 `LLM` 的这两条 IP 规则后，它们会按上面的规则走 `Google` 策略组。

### Loon

将 [`Loon_GoogleIP.list`](ProxyConfig/Loon/rule/Loon_GoogleIP.list) 添加到 `[Remote Rule]`，将 `policy` 设为已有的策略组。

```ini
https://raw.githubusercontent.com/SwordZYZ/ProxyRules/main/ProxyConfig/Loon/rule/Loon_GoogleIP.list, policy=Google, tag=GoogleIP, enabled=true
```

### Quantumult X

将 [`QX_GoogleIP.list`](ProxyConfig/QuantumultX/rule/QX_GoogleIP.list) 添加到 `[filter_remote]`。清单默认使用 `proxy`，可通过 `force-policy` 指定已有的策略组。

```ini
https://raw.githubusercontent.com/SwordZYZ/ProxyRules/main/ProxyConfig/QuantumultX/rule/QX_GoogleIP.list, tag=GoogleIP, force-policy=Google, enabled=true
```

### 同步

GitHub Actions 每周一 09:17（北京时间）同步三个规则集，也可在 GitHub 的 **Actions → Sync Google IP rules** 手动运行。仅在规则内容变化时，工作流才会提交并推送更新。工作流使用 `GITHUB_TOKEN`，仓库的默认分支需允许该令牌推送。

公开仓库连续 60 天没有活动时，GitHub 会停用定时工作流，可在 Actions 页面重新启用。[GitHub 定时工作流说明](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)

在项目根目录手动更新可以运行以下命令。脚本只使用 Python 标准库。

```sh
python3 ProxyConfig/Clash/rule/update_google_ip.py
```
