# iPhone / iPad 版

iOS 版与 Mac 版使用相同的规则、DNS 端点和国内清单，只有插件信息（名称、适用系统、说明）不同。构建脚本会检查两个平台的插件正文逐字一致，因此行为、限制和测试结论与 Mac 版相同。

**本版只做了文件静态检查，作者没有在 iPhone 上导入或运行。** 实际加载、内存占用和解析路径请按下文自行确认。

## 导入地址

两个版本只启用一个，也不要同时启用 Mac 版文件。

标准版：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/Loon-iOS-DNS-Guard.plugin
```

国内分流版：

```text
https://raw.githubusercontent.com/hclchcl/loon-mac-dns-guard/main/Loon-iOS-DNS-Guard-CN.plugin
```

插件头只声明 `#!system = iOS,iPadOS`，没有填写最低 iOS 和 Loon 版本：官方文档没有说明版本不匹配时是拒绝导入还是仅提示，不写可避免误挡。

## 设置步骤

1. 先备份当前配置，再在 Loon 的插件页添加上面一个地址。
2. 在插件详情里把 **PROXY** 绑定到一个只含境外节点的组。PROXY 只承载 DNS 查询，选延迟最低的地区即可，不必与网页出口相同；最好是 url-test 组，示例写法见 [主配置补充说明](main-config.txt)。不要绑定 `DIRECT` 或会回退直连的组。
3. 在 Loon 的 DNS 服务器页面关闭 **查询回落**；若有“DoH 流量模式”，设为 **Rule**。
4. 在主配置 `[General]` 合并 `hijack-dns = *:53`，同名键只保留一行。插件不能设置这个字段。
5. 节点服务器地址是域名的订阅，都要设置直连的加密 `server-dns`，例如 `server-dns="https://223.5.5.5/dns-query,https://223.6.6.6/dns-query"`。缺少时节点域名会经 PROXY 解析，既慢又与 PROXY 绑定的节点组形成循环依赖（Mac 版实测，见 [验证记录](validation.md)）。
6. 检查主配置、其他插件、订阅中的 `[Host]`/`[DNS]` 和按 Wi-Fi 指定的 DNS，全局加密 DNS 只保留本插件的 Cloudflare 端点。

## iPhone 上需要额外注意

- **系统加密 DNS 描述文件**：若在“设置 → 通用 → VPN 与设备管理”中安装过 DNS 描述文件（如某些去广告 DNS），系统可能直接使用它。DoT 类描述文件走 853 端口，会被插件拒绝导致解析失败；建议启用本插件前移除这类描述文件。
- **内存**：iOS 对 VPN 扩展有内存上限。国内分流版含 12,595 条 Host 映射（文件约 650 KB）；Mac 上实测匹配开销可以忽略，但 iPhone 上的内存占用没有实测。若出现 Loon 频繁断开或内存提示，先改用标准版。
- **蜂窝与 Wi-Fi 切换**：`hijack-dns` 接管的是进入 Loon 的 UDP 53，与网络类型无关；按 SSID 指定的 DNS 会覆盖插件设置，切换网络后请在 DNS 记录中复查。
- **验证**：在 Loon 的请求记录或 DNS 记录里产生一组新查询，确认清单外域名由 `1.1.1.1`/`1.0.0.1` 解析并经 PROXY，国内清单域名由 `223.6.6.6` 解析并直连，且没有运营商或路由器的 DNS。

其余边界与故障处理同 Mac 版，见 [README](../README.md) 与 [主配置补充说明](main-config.txt)。
