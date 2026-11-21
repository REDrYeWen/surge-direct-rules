# surge-direct-rules（私有）

Surge 直连（放行）规则库。与拦截库 [surge-reject-rules] 配套使用。
本库所有文件均为「未带策略」的规则集（ruleset）格式，在 Surge 配置中通过 RULE-SET 引用时指定 DIRECT 策略，例如：

```
RULE-SET,<本仓库 raw 地址>/rulesets/loyalsoldier-private.txt,DIRECT
```

## rulesets/（远程规则集的存档副本，2026-11-21 下载）

| 文件 | 来源 | 内容 | 规模 |
| --- | --- | --- | --- |
| chinamax-all-no-resolve.list | blackmatrix7/ios_rule_script · ChinaMax_All_No_Resolve | 中国大陆直连大全：域名 + 中国 IP 段（IP 规则带 no-resolve）；另含少量 USER-AGENT、PROCESS-NAME（Mac）规则 | 123,125 条（DOMAIN-SUFFIX 110,207 / IP-CIDR 8,244 / IP-CIDR6 4,312 / DOMAIN 271 / 其他约 91） |
| loyalsoldier-direct.txt | Loyalsoldier/surge-rules · direct | 大陆网站直连域名表（纯域名，无 IP） | 110,250 行（DOMAIN-SUFFIX 109,688 / DOMAIN 562） |
| loyalsoldier-private.txt | Loyalsoldier/surge-rules · private | 局域网/私有域名：路由器后台、.local/.lan/home.arpa 等保留后缀 | 132 行 |
| loyalsoldier-cncidr.txt | Loyalsoldier/surge-rules · cncidr | 中国大陆 IPv4 地址段（纯 IP-CIDR，无 no-resolve 标记） | 9,665 条 |

注意重叠：ChinaMax 与 loyalsoldier-direct 的域名有 107,879 个相同（约占 direct 的 98%），ChinaMax 基本覆盖 direct 并额外提供 IP 段与 UA/进程规则；两者同时引用没有坏处，但没有必要。cncidr 的 IP 段与 ChinaMax 的 IP-CIDR 部分同样高度重叠。日常更新建议直接引用上游地址，本仓库副本仅作存档与对照。

## custom/（从个人现有配置拆出的直连规则）

| 文件 | 内容 |
| --- | --- |
| talkatone-lan-private.list | Talkatone 相关域名、路由器/内网域名、私有与保留 IP 段（带 no-resolve）、IPv6 私有段、in-addr.arpa 反向解析后缀 |
| media-misc.list | 「奶昔」段的几个媒体/订阅域名与从抓包手动添加的域名（baidu.com 后缀也在其中，按源文件位置归入直连，使用时请留意其覆盖面） |

## 已知冲突（与拦截库之间）

- `inmobi.com`、`kochava.com`、`mobilefuse.com`、`crashlytics.com` 相关域名在本库 Talkatone 段（直连）与拦截库的广告表（REJECT）中同时出现。Surge 按规则顺序先匹配先执行，最终效果取决于配置中两条 RULE-SET 的先后。整理于 2026-11-21，冲突按源文件原样保留，未擅自删改。

## 重整化记录（2026-11-21）

全库做过一轮匹配清理（脚本 normalize.py，可重复运行）：删除精确重复、被 DOMAIN-SUFFIX 父后缀覆盖的域名、被更短关键字包含的 DOMAIN-KEYWORD、被大网段包含的小网段（仅限 no-resolve 选项相同者）。结果（规则条数）：

| 文件 | 清理前 | 清理后 |
| --- | --- | --- |
| chinamax-all-no-resolve.list | 123,125 | 123,107 |
| loyalsoldier-direct.txt | 110,250 | 2,146（与 ChinaMax 重复 107,877 条、被覆盖 227 条，本文件现仅保留 ChinaMax 未覆盖的部分） |
| loyalsoldier-private.txt | 132 | 114 |
| loyalsoldier-cncidr.txt | 9,665 | 9,665（未删，见下） |
| custom/talkatone-lan-private.list | 49 | 28 |
| custom/media-misc.list | 7 | 2 |

- cncidr 未删的原因：它的网段不带 no-resolve，与 ChinaMax 的同范围网段选项不同，脚本按保守原则不跨选项删除。实测 cncidr 的 IPv4 网段有 6,192/6,221（99.5%）被 ChinaMax 的网段包含——若确认不需要 cncidr 的「触发 DNS 解析」行为，可整表停用，只留 ChinaMax。
- 与拦截库的冲突已按拦截优先处理：`DOMAIN-SUFFIX,inmobi.com`、`DOMAIN-SUFFIX,kochava.com` 已从 custom/talkatone-lan-private.list 删除（拦截库中为明确的 REJECT）。Talkatone 段其余广告 SDK 域名（agkn、cohere、crashlytics、inner-active、mobilefuse）在拦截库中只有子域名级条目、没有整后缀拦截，故保留直连并在此标注。
- 每 15 天自动更新：重新下载四个上游文件、比较新增、重跑重整化并推送本仓库。
