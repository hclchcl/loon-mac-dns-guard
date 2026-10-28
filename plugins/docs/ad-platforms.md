# Mac 广告平台拦截器

导入地址：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/plugins/Loon-Mac-Ad-Platforms-Block.plugin
```

目标版本：Loon for Mac 0.5.0（1009）。包含 77 条原生规则，无脚本、无复写、无 MitM，全部使用 `REJECT`，无需配置 `PROXY`。

## 处理范围

从 blackmatrix7/ios_rule_script 的 AdvertisingLite 固定快照中，选取常见广告投放、素材、归因和广告统计平台条目，包括 Google/DoubleClick、腾讯广告、百度广告、阿里妈妈的已列广告端点、穿山甲、京东广告、小米广告，以及 AppLovin、AppNexus、Criteo、Rubicon、PubMatic、Taboola、Outbrain、InMobi、Vungle 等相关条目。

这是配置的来源范围，不代表已经验证所有这些平台在 Mac 上的实际拦截效果；也不表示完整覆盖某个平台的全部地址。准确条目以插件明文和 [源条目](../sources/advertising-domain-selection.list) 为准。

## 选择方式

- 来源中以点号开头的域名保留为 `DOMAIN-SUFFIX`；其他条目保留为精确 `DOMAIN`，没有把精确匹配自动扩大为整个后缀。
- 只抽取部分有明确平台名称的关键词，没有直接导入上游全部泛化关键词。
- 没有整体封禁 `qq.com`、`baidu.com`、`snssdk.com` 等业务主域；没有复制原插件中把业务配置直连的例外规则。
- 没有使用广告 IP 大列表，减少共享 IP、云地址迁移和内网网段导致的额外影响。

若平台使用同一请求返回广告和正常内容，域名拦截可能不能精确处理。需要按服务处理的广告路径由第三个插件承担；本插件不承诺删除浏览器中的空白占位。

## 安装与回滚

导入后关闭原广告平台拦截器，再启用本插件。可以搭配第一、第三个 Mac 插件；不存在必须先配置某个 `PROXY` 的步骤。

关键词规则也可能命中不希望拦截的站点，广告归因域名的阻断可能影响推广跳转。如有异常，先停用本插件或为具体域名添加你需要的本地策略例外。不要把“已添加规则”当成全部应用已经去广告。

本次没有进行实机导入或广告效果测试。规则数据是固定快照，来源清单的更新时间与本次制作日期分别记录，不承诺持续自动更新。
