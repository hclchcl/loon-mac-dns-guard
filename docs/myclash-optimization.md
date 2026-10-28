# v1.2.0：参考 MyClash 的 DNS 分工

本次优化只更新国内分流版及其说明和构建资料。沿用 v1.1.1 的 `https://223.6.6.6/dns-query`，默认解析仍为经 PROXY 的两个 Cloudflare IP 端点。标准版及其他插件保持原样，本机 Loon 的主配置、策略绑定和本地 CDN 例外未改动。

## 采用的思路

参考 [AIsouler/MyClash 的固定版本源码](https://github.com/AIsouler/MyClash/blob/071d4ec5764b4700d0c5de41797f20a8bdf6b6d8/Script/mihomoScript.js)，分别处理国内服务、默认查询和代理节点的引导解析。

| 用途 | 本插件在 Loon 中的实现 |
| --- | --- |
| 国内域名和国内 CDN | Host 指定阿里 DoH；只匹配明确的域名范围 |
| 其他普通域名 | 全局 Cloudflare DoH，端点按 PROXY 访问；需排除其他全局 DNS 混入 |
| 节点服务器域名 | 仍由用户在节点或订阅上配置独立的加密 `server-dns`；插件不获取或改写节点 |

MyClash 对 Apple、微软、NVIDIA 和游戏服务的 CN 条目做单独处理。本次借鉴分组方式，从已核验的 v2fly 数据快照提取 `APPLE@cn`、`MICROSOFT@cn`、`NVIDIA@cn`、`CATEGORY-GAMES@cn`，补齐原清单未覆盖的 **386 条 DNS 匹配项**。这些是根域名、后缀或精确主机名的匹配项，并非 386 个独立网站。

新增范围包括 `swcdn.apple.com`、`download.developer.apple.com`、`officecdn.microsoft.com`、`download.windowsupdate.com`、国内 Steam 下载主机等。源数据的精确主机条目仍保持精确匹配，不扩大为整个 `apple.com`、`microsoft.com` 或 `steamcontent.com`。业务请求继续由原规则决定走直连还是代理。

## 本次变化

| 项目 | v1.1.1 | v1.2.0 |
| --- | --- | --- |
| 国内 DNS 匹配项 | 12,215 | 12,595 |
| 检测网站专用 Host | 12 | 0 |
| 总 Host 映射 | 12,227 | 12,595 |
| 插件规则 | 13 | 7 |
| 国内 DNS 端点 | 223.6.6.6 | 223.6.6.6 |

- 删除 6 条已被同一解析器的父域通配符覆盖的抖音精确映射；这些主机仍在国内清单覆盖范围内。此处的等价性仅针对本插件自身，其他 Host 来源存在同名或更具体条目时仍需核对。
- 移除对 DNSLeakTest、IPLeak 等 6 个检测站点的专用代理规则和 12 条专用解析映射。它们改为遵循普通配置流程，减少“只让检测站点走不同路径”的偏差。网页检测仍无法单独证明客户端到解析器是否经过代理。
- `docs/domestic-domains.txt` 作为唯一域名输入，由模板生成插件，避免文档与实际插件日后漂移。

## 没有直接移植的参数

- MyClash 的 `nameserver-policy`、`direct-nameserver`、`proxy-server-nameserver`、Fake-IP 缓存等属于 Mihomo 配置；Loon 插件不能用同名参数实现这些功能。
- MyClash 某些国内和抖音策略会使用普通 DNS 或系统 DNS。本项目维持加密 DNS，不移植这些明文查询路径。
- 不加入按地区切换节点、屏蔽国外 QUIC、PCDN 拦截或固定 CDN IP；这些会改变现有业务访问行为，超出此次 DNS 插件优化范围。
- Loon 官方 Host 文档未给出单条域名映射配置多个加密 DNS 的可靠写法。本版继续使用单一国内端点，没有宣称自动故障切换。把多个端点加入全局列表会使其参与其他域名的并发解析，不能用来替代国内专用备用。

## 来源与维护

域名来自 [v2fly/domain-list-community，release 20261027070632](https://github.com/v2fly/domain-list-community/releases/tag/20261027070632)，沿用此前国内清单使用的已核验快照；`dlc.dat` 的 SHA-256 为 `6409bf0d157fc389ca0349a1da76b98d7ac2e569fe4e80a3883290e7381bde0f`。按 `cn` 属性筛选四个分组，只转换 Domain 和 Full 类型，跳过 3 条不支持的正则表达式。源码的版权与 MIT 许可保留在域名清单和生成插件中。

MyClash 提供设计参考，本插件未复制其脚本，也未把其 MRS 规则当成 Loon 文件使用。具体来源和数量见 [来源记录](domestic-domain-sources.json)。旧清单的全部国内匹配范围保留，国际 AI、GitHub 等域名没有新增业务或 DNS 专用规则。

维护者修改域名清单或模板后，在仓库根目录运行（Python 3.9+）：

```sh
python3 scripts/build-domestic-plugin.py
python3 scripts/build-domestic-plugin.py --check
```

构建工具只处理本地文件，不访问网络、不自动拉取域名，也不操作 Loon。没有新增定时更新任务。

## 更新与验证边界

继续使用原插件订阅地址即可；在 Loon 内单独更新国内分流版，并确认说明显示 `v1.2.0`。版本号位于插件描述及文件注释中；日期仍为 2026-10-28，不能只用日期判断。若 CDN 仍有旧缓存，可在导入前查看原始文件的版本，避免仅凭“更新成功”判断已加载。

本次只做文件结构、来源完整性、域名覆盖及生成一致性的静态检查。没有启用本版、重连 Loon、访问 Claude、做泄漏检测或性能测试。速度与实际解析路径由使用者验证，不能据此宣称提速或零泄漏。

回滚只需将国内分流版订阅临时指向 [v1.1.1 的固定文件](https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/2f7eeb1da025c708575196e1804d79585d28cc49/Loon-Mac-DNS-Guard-CN.plugin)，不需要覆盖主配置或恢复整份设置。

Loon 的字段与边界依据：[插件](https://nsloon.app/docs/Plugin/)、[DNS](https://nsloon.app/docs/DNS/)、[Host 映射](https://nsloon.app/docs/DNS/hostmap/)、[节点引导 DNS](https://nsloon.app/docs/Node/subscription/)。
