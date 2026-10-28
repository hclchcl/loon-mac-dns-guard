# 署名、来源与修改说明

本目录包含分别发行的三个 Loon 插件及其可编辑来源。适配者：hclchcl；修改日期：2026-10-28。原项目作者与本项目没有官方合作关系。

## HTTPDNS 插件

- 原作者：VirgilClyne；项目：[VirgilClyne/GetSomeFries](https://github.com/VirgilClyne/GetSomeFries)。
- 依据文件：`sgmodule/HTTPDNS.Block.sgmodule`，固定提交 `b4aa767bcbc4d39bea5b549a3277a00517891d1e`，文件 Git blob SHA `1fe7e056fe4f3db500823de44e9d1edd66f74b93`。
- 原始快照保存在 [sources/HTTPDNS.Block.sgmodule](sources/HTTPDNS.Block.sgmodule)，保留作者、资料链接及原始格式。
- 本插件及该来源按 [GNU GPL v3](licenses/GPL-3.0.txt) 发布。
- 适配：Surge 规则转为 Loon 原生规则；移除平台专属标记；IP 拦截收窄到目标端口 80/443/8080；选取明文 HTTP 复写并增加边界，去重并排除地址格式有问题的条目；没有分发原 Map Local 或 MitM 配置。
- 逐条适配记录：[sources/httpdns-changes.json](sources/httpdns-changes.json)。

## 广告平台与应用/网页过滤插件

- 规则整理者：blackmatrix7；项目：[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)。
- 固定提交：`0b6897dc17e0c513d25449c935a9694818e32f0c`。
- 依据文件：`rule/Loon/AdvertisingLite/AdvertisingLite_Domain.list`、同目录 `AdvertisingLite.list`，以及 `rewrite/Loon/AdvertisingLite/AdvertisingLite.plugin`。
- 两个插件及来源选集按该项目的 [GNU GPL v2](licenses/GPL-2.0.txt) 发行。来源的规则更新时间分别为 2025-12-08、2025-12-08、2025-07-12；本次制作日期不代表每条规则都更新到了 2026-10-28。
- 保留使用到的原始域名条目、关键词及复写原文；来源、Git blob SHA 与不可变链接记录在 [sources/manifest.json](sources/manifest.json)。
- 适配：保留精确域名/后缀语义，选取平台条目和常用服务专用广告域名；限定路径并转为 Loon 原生复写；未整体复制上游全部规则、iOS 插件或全部 MitM 主机列表。

上游说明列出的规则来源包括以下项目。感谢原作者；本项目不把这些规则选择宣称为全部原创：

- [ACL4SSR/ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- [NobyDa/ND-AD](https://github.com/NobyDa/ND-AD)
- [NobyDa/Script](https://github.com/NobyDa/Script)
- [yjqiang/surge_scripts](https://github.com/yjqiang/surge_scripts)
- [GeQ1an/Rules](https://github.com/GeQ1an/Rules)
- [GFBG-IT/QuantumultX](https://github.com/GFBG-IT/QuantumultX)
- blackmatrix7 本仓库内的 `source/rule/Advertising` 与 `source/rewrite/Advertising`。

## 可莉插件与本项目的边界

Mac Loon 中已有的 HTTPDNS拦截器、广告平台拦截器、可莉广告过滤器由可莉发布。本项目只参考其可见功能分工，没有复制、解密、修改或再分发其 `.lpx` 源文件，也不使用“可莉官方 Mac 版”等名义。

三个新插件与本仓库根目录的 DNS 插件是分别可用的配置文件，许可分别适用。根目录 MIT 许可只适用于相应原创 DNS 文件；不将 GPL 来源重新声明为 MIT。所有新插件均以可编辑明文发布，不提供效果担保。

## Loon 语法依据

- [插件及字段支持](https://nsloon.app/docs/Plugin/)
- [规则与来源优先级](https://nsloon.app/docs/Rule/)
- [逻辑规则](https://nsloon.app/docs/Rule/logic_rule/)
- [原生 Rewrite 语法与动作](https://nsloon.app/docs/Rewrite/rewrite_v2/)

许可全文随目录提供。重新分发修改后的插件时，请保留适用署名与许可，并明确记录自己的修改。
