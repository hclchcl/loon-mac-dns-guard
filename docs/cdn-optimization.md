# 国内网页 CDN 解析优化

## v1.0.2 · 2026-10-28

国内分流版原有列表覆盖了抖音主站与视频域名，但遗漏了部分网页脚本、静态资源与直播资源。启用全局 Cloudflare 代理解析后，这些遗漏主机可能得到境外 CDN 地址，再命中原配置的兜底代理。

本次仅新增以下精确主机名的 `server:https://223.5.5.5/dns-query` 映射：

- `lf-c-flwb.bytetos.com`
- `lf3-webcast.bytetos.com`
- `lf-cdn-tos.bytescm.com`
- `lf-security.bytegoofy.com`
- `lf-douyin-pc-web.douyinstatic.com`
- `lf-normal-gr-sourcecdn.bytegecko.com`

这些主机均在抖音网页实际加载中出现，并完成了阿里与 Cloudflare 的加密 DNS 结果对照。部分关键脚本经阿里返回地址访问更快；结果受网络、CDN 调度和缓存影响，不代表所有环境的固定速度。

没有扩大到共享 CDN 的整个域名后缀，没有固定 CDN IP，也没有新增强制直连业务规则。原有 DNS 端点、端口防护、HTTPDNS 插件、策略组和业务分流继续保留。

## B 站图片部分加载失败

`bilibili.com`、`hdslb.com`、`bilivideo.com` 等已在国内解析列表内。若只有某个图片主机反复超时，应结合仪表中的 HostMap、实际分流和连接日志检查具体 CDN 地址，不能仅因页面慢就替换全局 DNS。

在本次网络中，`i2.hdslb.com` 的国内 CDN 地址连接超时，另一条加密解析路径返回的地址可访问。因此仅在该设备主配置中添加以下临时例外；它没有写入公共插件：

```ini
[Host]
i2.hdslb.com = server:https://1.1.1.1/dns-query
```

这条例外继续使用已有的 Cloudflare 代理加密 DNS，业务访问仍由原分流决定，可能使用代理并消耗代理流量。其他 B 站域名继续使用阿里 DoH。不同网络不应照搬；国内 CDN 恢复后可删除本地例外。

## 更新与回滚

在 Loon 中更新国内分流版插件后，核对新增主机名的 HostMap 和 DNS 记录。若旧 DNS 或连接仍被复用，可重新连接 Loon 并刷新网页。

回滚公共插件可使用上一提交的文件；回滚本地 B 站例外只需在 Host 映射中移除 `i2.hdslb.com`，不要恢复或覆盖整份配置。
