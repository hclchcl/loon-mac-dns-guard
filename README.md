# Loon for Mac DNS 防泄漏

面向 **Loon for Mac 0.5.0（Build 1009）** 编写的 DNS 防护插件，使用加密 DNS、明确的 DNS 出口和常见 DNS 端口阻断，减少运营商明文 DNS 与应用自带 DNS 绕行的风险。实际导入与运行效果由使用者测试确认。

**插件必须配合下面的 DNS 设置使用。导入成功不等于已经防止所有 DNS 泄漏。**

本项目根据 iPhone 镜像里“DNS防泄露”插件展示的功能方向独立编写，参考 Loon 官方语法；未复制、修改或再分发可莉插件的源码，也不代表其作者。

## 选择一个版本

| 版本 | 默认解析 | 国内解析 | 适用偏好 |
| --- | --- | --- | --- |
| [标准版](Loon-Mac-DNS-Guard.plugin) | Cloudflare DoH，经绑定的代理 | 与默认解析相同 | 尽量统一 DNS 出口 |
| [国内分流版](Loon-Mac-DNS-Guard-CN.plugin) | Cloudflare DoH，经绑定的代理 | `.cn`、中文中国域名及清单内常见国内服务使用阿里 DoH 直连 | 国内解析速度与分流 |

**两个版本只启用一个。** 国内分流版决定的是 DNS 查询路径；国内网站的实际访问出口继续由你原来的分流规则决定。[域名清单](docs/domestic-domains.txt)不保证完整，也不按 GeoIP 自动识别国内域名。

## 安装

1. 先备份当前配置。在 Loon → 插件 → 添加插件中粘贴下面一个地址。

   标准版：

   ```text
   https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/Loon-Mac-DNS-Guard.plugin
   ```

   国内分流版：

   ```text
   https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/Loon-Mac-DNS-Guard-CN.plugin
   ```

2. 打开插件详情，把 **PROXY** 指向你已有的、可用的境外代理策略。这里的 `PROXY` 是插件的策略占位名称，需要手动绑定；不要绑定 `DIRECT`，也不要绑定会回退到直连的策略组。
3. 在 Loon → 配置 → DNS 服务器中，关闭 **查询回落**，把 **DoH 流量模式**设为 **Rule**。
4. 检查主配置、其他插件、Host 映射及按 Wi-Fi 指定的 DNS。全局加密 DNS 应仅保留本插件需要的 Cloudflare 端点；不要混入其他全局 DoH/DoQ/DoH3。国内分流版通过 Host 映射指定阿里 DNS，无需把阿里 DNS 加进全局并发列表。
5. 按 [主配置补充说明](docs/main-config.txt)，在现有 `[General]` 中合并 `hijack-dns = *:53`，同名键只保留一行。插件本身无法应用这个字段。
6. 如果代理节点的服务器地址是域名，给对应节点或订阅设置独立的加密 `server-dns`，防止“建立代理需要 DNS、DNS 又依赖代理”的循环。示例与例外见补充说明。
7. 确认使用 Loon 的 TUN 接管，并排查其他同时接管 DNS/TUN 的 VPN。重新连接后按下面的方法检查。

“**包含系统 DNS 地址**”能关闭时可关闭；若 Mac 版无法关闭，可以保留，但仍须关闭查询回落，并核对实际 DNS 记录。插件不会替你清空系统 DNS，也不会写入虚假的关闭参数。

## 插件做了什么

- 向 Loon 添加 `https://1.1.1.1/dns-query` 和 `https://1.0.0.1/dns-query`，两者均为 Cloudflare DoH，经绑定的代理访问。
- 把常见检测网站的页面与解析指向代理路径，便于检查；测试网站显示正常不等于所有应用都正常。
- 拒绝实际进入 Loon 并命中规则的 53、853、784 端口连接，限制普通 DNS、常见 DoT/DoQ 绕行。
- 给阿里 `223.5.5.5:443`、`223.6.6.6:443` 保留直连加密路径，用于独立节点引导解析和可选国内分流；标准版不会因此把阿里加入全局解析列表。
- 国内分流版额外给清单内域名配置阿里 DoH。默认使用 `223.5.5.5`；`223.6.6.6` 是节点引导解析的另一个端点，不是 Host 映射的自动备用。

不含脚本、MitM、证书、节点或订阅凭据，不添加 `FINAL`，不改 AI、广告过滤等原有业务规则；只有表内检测网站会新增业务代理规则。

## 怎样验证

1. 在 Loon 仪表中产生一组新查询，核对解析服务器、DNS 端点的规则和实际出口。标准版默认解析应使用 Cloudflare，`1.1.1.1:443` / `1.0.0.1:443` 应经绑定代理；国内分流版的清单内域名应使用阿里 DoH，端点应为 DIRECT。
2. 使用 [DNSLeakTest](https://www.dnsleaktest.com/) 或 [IPLeak](https://ipleak.net/) 多轮检查。运营商普通 DNS 不应因回落而出现；公共 DNS 的机房位置与代理节点位置可能不同，不能只按国家判断泄漏。
3. 对硬编码 DNS 的应用或命令，检查请求是否被接管或拒绝。单次超时不证明已经安全；应结合 Loon 请求记录及物理网络接口抓包判断。
4. 若需要更高保证，还要检查 IPv6、局域网 DNS、网络切换、代理失效、重新连接和冷启动。测试全部通过后才能对这些具体场景下结论。

本次实际完成的检查与尚未验证的范围见 [验证记录](docs/validation.md)。

## 已知边界与故障处理

- Loon 插件的 DNS 列表是**追加合并**，不能覆盖主配置、其他插件或系统 DNS；关闭回落也是 App 内设置，不能靠插件注释代替。
- Loon 本地规则优先于插件规则。已有本地放行规则、其他插件的冲突映射、绕过 TUN 的地址，都可能使本插件的预期不成立。
- `hijack-dns` 是 UDP 接管；TCP 53 与没有被接管的 UDP 53 可能直接被拒绝。依赖这些查询的应用可能无法联网。
- 应用内置的 DoH/HTTPDNS 可以使用普通 HTTPS 443，本插件不能完整识别或阻止。需要时单独管理应用的安全 DNS 或使用可核实的专门规则。
- 不拦截 5353 端口的本地 mDNS；局域网发现功能及没有进入 Loon 的局域网 DNS 不在保证范围内。IPv6 接管必须另行核对。
- 公司/校园内网域名、认证门户可能依赖本地 DNS，启用前评估；需要例外时应明确接受该域名使用本地解析，不把例外当作全局无泄漏。
- 节点引导 DNS 在建立代理前直连使用加密端点，解析器仍能看到节点域名，这是公开说明的引导路径。
- 代理不可用且关闭回落时可能解析失败；本项目没有断开 VPN 后继续阻断 DNS 的系统级开关。
- 回滚时关闭或移除此插件，再根据备份恢复主配置合并项及你手动改过的 DNS 设置；不要覆盖整份配置。

## 依据与许可

- [Loon：插件 General 的支持字段与追加合并](https://nsloon.app/docs/Plugin/)
- [Loon：DNS 查询与回落](https://nsloon.app/docs/DNS/)
- [Loon：主配置及 UDP DNS 接管](https://nsloon.app/docs/General/)
- [Loon：DNS 映射](https://nsloon.app/docs/DNS/hostmap/)
- [Loon：规则来源优先级](https://nsloon.app/docs/Rule/)
- [Loon：节点订阅与 server-dns](https://nsloon.app/docs/Node/subscription/)
- [Cloudflare：DoH 请求格式](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/make-api-requests/)

原创文件按 [MIT License](LICENSE) 发布；本仓库许可证不适用于其他作者的插件。
