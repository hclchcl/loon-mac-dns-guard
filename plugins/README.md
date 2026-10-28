# Loon for Mac 去广告插件

参照 Mac Loon 当前前三个插件的功能分工，面向 **Loon for Mac 0.5.0（Build 1009）、macOS 13 及以上** 制作的三个独立插件。规则来自注明许可的公开项目，经选择、收窄和语法转换后发布；**未复制可莉插件源码，不是原作者的官方 Mac 版，也不承诺与原版覆盖完全相同。**

按用户要求，这次只制作和发布，未导入、启用或进行实机效果测试。下表是文件内配置数量，不是实际拦截数量。

| 插件 | 对应的功能方向 | 本次配置 | 是否需要证书 |
| --- | --- | --- | --- |
| [Mac HTTPDNS 拦截器](Loon-Mac-HTTPDNS-Block.plugin) | HTTPDNS 拦截器 | 81 条规则、42 条明文 HTTP 复写 | 不需要 |
| [Mac 广告平台拦截器](Loon-Mac-Ad-Platforms-Block.plugin) | 广告平台拦截器 | 77 条广告平台规则 | 不需要 |
| [Mac 应用与网页广告过滤器](Loon-Mac-Ad-Filter.plugin) | 可莉广告过滤器的按服务处理广告这一方向 | 20 条专用域名规则、15 条路径复写、34 个精确 MitM 域名 | 域名规则不需要；HTTPS 路径复写需要 |

三个插件可以搭配，也可以分别启用；它们不需要配置 `PROXY`，不修改 DNS 服务器，不改变已有 DNS 防泄漏插件的策略。

## 分别导入

在 Mac Loon → **插件 → 添加插件**，逐个粘贴下面的地址。建议按 HTTPDNS → 广告平台 → 应用与网页过滤的顺序排列，并在切换时关闭相应的原插件，避免两套规则和复写叠加。

HTTPDNS 拦截器：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/plugins/Loon-Mac-HTTPDNS-Block.plugin
```

广告平台拦截器：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/plugins/Loon-Mac-Ad-Platforms-Block.plugin
```

应用与网页广告过滤器：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/plugins/Loon-Mac-Ad-Filter.plugin
```

无需一次性停用其他不相干插件，也不要删除原插件。可以先添加新文件、暂不启用，保存原设置后逐个切换和自行测试。文件更新使用 Loon 自带的插件更新功能；本项目采用固定来源快照，不会自动追随其他作者规则的每次变化。

## 逐项说明

- [HTTPDNS：覆盖对象、HTTP 路径及可能的联网影响](docs/httpdns.md)
- [广告平台：平台清单、关键词和覆盖范围](docs/ad-platforms.md)
- [应用与网页过滤：服务清单、MitM 与例外](docs/ad-filter.md)
- [版本差异与待测范围](docs/compatibility.md)
- [来源、署名和许可](NOTICE.md)

## HTTPS 路径复写

第三个插件的路径复写只有在 Loon 能读取对应 HTTP 请求时才会生效。对于 HTTPS，需要你在 Loon 的域名与证书设置中启用 MitM、安装并信任自己的 Loon CA。**这个仓库不提供证书或私钥，也不需要关闭 TLS 证书验证。**

第三个插件只列出选中服务的精确域名，没有添加全站 `*`、银行或支付站点。尚未启用 MitM 时，域名规则仍可独立工作；HTTPS 路径规则不会因此自动生效。证书固定、自定义加密、HTTP/3 或未进入 Loon 的流量可能无法处理。

## 使用边界

- 阻止 HTTPDNS 后，应用可能改用系统解析，也可能直接报错；不保证所有应用都能自动回落，更不等于完整的 DNS 防泄漏。
- 仅处理进入 Loon 且命中配置的请求。已有本地放行规则、其他插件、进程绕过、局域网绕过和 IPv6 接管会影响结果。
- 广告与正常内容共用一个响应时，这组文件不会猜测其 JSON 结构，也不保证去掉信息流广告、视频贴片或页面空白。没有提供浏览器元素隐藏规则。
- 如果出现误拦截，先停用对应新插件；需要例外时使用更具体的本地 `DIRECT` 或既有业务策略规则，而不是把整个平台或全部流量改为直连。
- 没有添加 `FINAL`、节点、订阅地址、定时脚本、会员修改或内容解锁逻辑。

## 许可

HTTPDNS 插件及其上游来源按 [GPL-3.0](licenses/GPL-3.0.txt) 发布；另两个插件及其规则选择按 [GPL-2.0](licenses/GPL-2.0.txt) 发布。它们在同一仓库中分别发行，根目录 DNS 插件的 MIT 许可不覆盖这些派生规则。具体署名、版本和修改记录见 [NOTICE](NOTICE.md) 与 [来源清单](sources/manifest.json)。
