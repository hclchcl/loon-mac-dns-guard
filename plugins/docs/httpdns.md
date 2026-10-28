# Mac HTTPDNS 拦截器

导入地址：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/plugins/Loon-Mac-HTTPDNS-Block.plugin
```

目标版本：Loon for Mac 0.5.0（1009）。包含 81 条网络规则、42 条明文 HTTP 复写，无脚本、无 MitM、无需绑定代理策略。

## 覆盖方向

来源清单包含阿里、百度、哔哩哔哩、华为、爱奇艺、京东、酷我、美团、网易、腾讯云、字节跳动、微博、微信、企业微信等 HTTPDNS 端点。每条具体规则以插件明文为准，不表示已经在各家 Mac 客户端实际触发。

- 对清单内的专用 HTTPDNS 域名使用原生域名拒绝规则。
- 对清单内硬编码 IPv4/IPv6 地址，将阻断范围限定为目标端口 80、443、8080，避免把同一地址的所有端口都直接封禁。
- 另选取来源中的 HTTPDNS 请求路径，转为 Loon 原生 Rewrite；仅匹配明文 HTTP，并返回 503，使请求立即失败。

## 相对来源的修改

以 VirgilClyne/GetSomeFries 的 `HTTPDNS.Block.sgmodule` 为依据；保留完整上游快照及 GPL-3.0 许可。

移除了 Surge 专属的 `force-http-engine-hosts`、`pre-matching`、`extended-matching` 和 `%APPEND%`；没有把不属于 Loon 插件支持范围的参数直接搬入。去除重复项，增加路径边界，排除了上游含非法 IPv4 地址 `182.256.116.116` 的复写及未按 URL 方括号格式书写的 IPv6 路径规则，没有擅自猜测正确地址。原来的独立 IPv6 网络规则仍保留。

HTTPS 路径复写和 Map Local 没有纳入；专用域名和清单内 IP 的网络规则仍可拦截对应 HTTP(S) 连接。其他共享域名上的加密 HTTPDNS 不保证覆盖。逐条选择记录见 [httpdns-changes.json](../sources/httpdns-changes.json)。

## 安装与回滚

导入后建议关闭原 HTTPDNS 拦截器，再启用本插件。不需要修改现有加密 DNS 配置。应继续保留你已经配置好的 DNS 防泄漏措施。

部分应用可能不接受 HTTPDNS 失败，出现登录、聊天、图片、播放或联网问题。此时先停用本插件恢复，再由你检查具体命中的请求；不能把应用报错说成正常回落。需要恢复原行为时，停用本插件并重新启用原插件即可。

未进行实机导入、联网、回落或泄漏测试。
