# 中兴 U60 Pro (MU5250) 免登录接口字段手册

> 测试设备：ZTE U60 Pro（型号 MU5250，固件 `BD_CNMU5250V1.0.0B31`）
> 测试日期：2026-09-07，以下所有接口均实测于**未登录状态**（浏览器打开登录页就能看到的数据）。
>
> 配套的 LiteMonitor 插件：`CPEMonitor.json`

## 调用方式

**POST `http://<设备IP>/ubus/`**，`Content-Type: application/json`，body 是 JSON-RPC 数组：

```json
[{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "call",
  "params": ["00000000000000000000000000000000", "<服务名>", "<方法名>", { "参数": "值" }]
}]
```

- 第 1 个参数是登录会话 token（stok），未登录固定填 **32 个 0**。
- 一个 POST 可以塞多个 call（数组），响应也是按顺序对应的数组。
- 成功：`{"id":N,"result":[0,{...数据...}]}`；无权限：`{"id":N,"error":{"code":-32002,"message":"Access denied"}}`。

---

## 一、电池（zwrt_mc.device.manager / get_device_info）

参数：`{"deviceInfoList": ["字段名1","字段名2",...]}`，按需取。已验证可用的字段名：

| 字段 | 示例值 | 含义 |
|---|---|---|
| `bat_percent` | `"52"` | **电量百分比** (0-100) |
| `bat_level` | `"3"` | 电池格数 (0-5 档) |
| `bat_online` | `"1"` | 电池在位 |
| `bat_charger_connect` | `"1"` | **充电器已插入** (1=插着) |
| `bat_charger_status` | `"0"` | 充电中状态 |
| `bat_ui_charger_type` | `"2"` | UI 显示的充电器类型 |
| `bat_mode` | `"1"` | 电池模式 |
| `bat_health` | `"1"` | 电池健康状态 |
| `bat_temperature` | `"31"` | **电池温度 ℃** |
| `bat_temperature_level` | `"0"` | 温度档位 (0=正常) |
| `bat_time_to_full` | `"-1"` | 距充满分钟数 (-1=未知/未充) |
| `bat_time_to_empty` | `"-1"` | 距耗尽分钟数 |
| `external_charging_flag` | `"0"` | 对外供电(反向充电)标志 |
| `power_saver_mode` | `""` | 省电模式 |
| `hightemp_datalimit_status` | `"0"` | 高温限流状态 |
| `meminfo` | `{total:1628400, free:404592, avaliable:801600}` | **内存 KB**：总量/空闲/可用 |
| `flashinfo` | `[{filesystem,size,used,avail,use,mounted_on}]` | 存储分区用量 |

> 注意：传入不认识的字段名会导致整个调用返回 `0`（失败），字段名只能用上面这些。

## 二、蜂窝网络（zte_nwinfo_api / nwinfo_get_netinfo）

参数：`{}`。实测返回（5G SA 状态下）：

| 字段 | 示例值 | 含义 |
|---|---|---|
| `network_type` | `"SA"` | **网络类型**（SA/NSA=5G，LTE=4G…） |
| `signalbar` | `"5"` | 信号格数 (0-5) |
| `network_provider` | `"CMCC"` | 运营商简称 |
| `network_provider_fullname` | `"China Mobile"` | **运营商全名** |
| `simcard_roam` | `"Home"` | 漫游状态（Home=本地） |
| `wan_active_band` | `"n41"` | 当前 5G 频段 |
| `nr5g_rsrp` | `-83` | **5G 信号强度 dBm**（越接近0越好，<-100 偏差） |
| `nr5g_rsrq` | `-11` | 5G 信号质量 dB |
| `nr5g_snr` | `"22.5"` | **5G 信噪比 dB** |
| `nr5g_rssi` | `-72` | 5G 接收强度 dBm |
| `nr5g_cell_id` | `13313155135` | 5G 小区 ID |
| `nr5g_pci` | `991` | 5G 物理小区标识 |
| `nr5g_action_channel` | `504990` | 5G 频点号 |
| `nr5g_bandwidth` | `"100"` | 5G 带宽 MHz |
| `lte_rsrp` / `lte_rsrq` / `lte_rssi` / `lte_snr` | `0` | 4G 对应信号值（走5G时为0） |
| `lte_band` | `"1,2,3,..."` | 4G 可用频段列表 |
| `nr5g_sa_band_lock` / `nr5g_nsa_band_lock` | `"1,2,3,..."` | 锁频配置 |
| `lteca_state` / `nrca` | `0` | 载波聚合状态 |
| `net_select` / `net_select_mode` | `"WL_AND_5G"` / `"auto_select"` | 选网方式 |
| `domain_stat` | `"PS_ONLY"` | 域状态（仅分组域） |
| `rmcc` / `rmnc` | `460` / `0` | 移动国家码/网络码（460+0=中国移动） |
| `lac_code` | `1448128` | 位置区码 |
| `nitz_timezone` | `"8.0"` | 网络下发时区 |

## 三、流量统计（zwrt_data / get_wwandst）

参数：`{"source_module":"web","cid":1,"type":N}`，**type=4 一次返回全部**。

| type | 内容 |
|---|---|
| 1 | 本次连接实时统计 |
| 2 | 本月统计 |
| 3 | 累计统计 |
| **4** | **全部（含今日）** |
| 7 | 仅今日 |

| 字段 | 含义（单位均为字节 bytes） |
|---|---|
| `real_tx_bytes` / `real_rx_bytes` | 本次连接 上行/下行 累计字节 |
| **`day_tx_bytes` / `day_rx_bytes`** | **今日 上行/下行 已用流量** |
| **`month_tx_bytes` / `month_rx_bytes`** | **本月（套餐周期内）上行/下行 已用流量** |
| `total_tx_bytes` / `total_rx_bytes` | 设备总累计 |
| `real_tx_speed` / `real_rx_speed` | **实时 上行/下行 速率（B/s）** |
| `real_max_tx_speed` / `real_max_rx_speed` | 历史峰值速率 |
| `real_time` / `day_time` / `month_time` / `total_time` | 对应统计时长（秒） |
| `*_packets` / `*_drop_packets` / `*_error_packets` | 各周期的包数/丢包/错包 |

## 四、拨号连接（zwrt_data / get_wwaniface）

参数：`{"source_module":"web","cid":1,"connect_status":""}`。

| 字段 | 示例值 | 含义 |
|---|---|---|
| `connect_status` | `"ipv4_ipv6_connected"` | **联网状态** |
| `ipv4_address` / `ipv4_gateway` / `ipv4_netmask` | `10.93.10.127` … | 运营商侧 IPv4（大内网地址） |
| `ipv6_address` / `ipv6_gateway` | `2409:...` | IPv6 地址 |
| `ipv4_dns_prefer` / `ipv4_dns_standby` | `211.136.150.86` | DNS |
| `ipv4_dev_name` | `"rmnet_data0"` | 数据网卡名 |

## 五、WAN/工作模式（zwrt_router.api / router_get_status_no_auth）

参数：`{}`。

| 字段 | 示例值 | 含义 |
|---|---|---|
| `current_wan_status` | `"ipv4_ipv6_connected"` | WAN 连接状态 |
| `opms_wan_mode` | `"PPP"` | 当前工作模式 |
| `opms_wan_auto_mode` | `"AUTO_LTE_GATEWAY"` | 自动模式判定结果 |

## 六、SIM 卡（zwrt_zte_mdm.api / get_sim_info_before）

参数：`{}`（before = 登录前可用；`get_sim_info` 需登录）。

| 字段 | 示例值 | 含义 |
|---|---|---|
| `sim_states` / `sim2_states` | `"sim undetected"` / `"sim ready"` | 卡1/卡2 状态 |
| `Operator` / `Operator2` | `"CMCC"` | 各卡运营商 |
| `current_sim_slot` | `"2"` | 当前使用的卡槽 |
| `support_dual_sim` | `"1"` | 支持双卡 |
| `sim1_provision_state` / `sim2_provision_state` | `"1"` | 卡启用状态 |
| `switch_card_status` | `"1"` | 切卡状态 |
| `modem_main_state` | `"modem_init_complete"` | 模块初始化状态 |
| `wlan_mac_address` | `"5c:7d:ae:..."` | 无线 MAC |
| `mdm_mcc` / `mdm_mnc` | `""` | 卡侧 MCC/MNC |

## 七、Wi-Fi（zwrt_wlan / report）

参数：`{}`。

| 字段 | 示例值 | 含义 |
|---|---|---|
| `main2g_ssid` / `main5g_ssid` | `"Einhart"` | **2.4G/5G Wi-Fi 名称** |
| `main2g_authmode` / `main5g_authmode` | `"sae-mixed"` | 加密方式 |
| `wifi_onoff` | `"1"` | Wi-Fi 总开关 |
| `wifi_start_mode` | `"2"` | Wi-Fi 启动模式 |
| `lbd_enable` / `mlo_enable` | `"1"` | 双频合一/MLO |
| `radio2` / `radio5` | `"wifi0"` / `"wifi1"` | 射频名 |
| `radio2_disabled` / `radio5_disabled` | `"0"` | 射频禁用标志 |
| `dfs_status` / `load_status` | `"end"` / `"idle"` | DFS 雷达检测/加载状态 |
| `mesh_deployed` 等 | `""` | Mesh 组网状态 |

## 八、设备身份（uci / get）

参数：`{"config":"zwrt_common_info","section":"common_config"}`。
（uci 是唯一免登录开放的 config，其它如 `wireless`、`network`、`zwrt_zte_mdm` 全部返回错误码 6）

| 字段 | 示例值 | 含义 |
|---|---|---|
| `device_market_name` / `device_alias_name` | `"U60 Pro"` | 设备名 |
| `model_name` | `"MU5250"` | 型号 |
| `manufacturer` | `"ZTE"` | 厂商 |
| `hardware_version` | `"MU5250_HW1.0"` | 硬件版本 |
| `wa_inner_version` | `"BD_CNMU5250V1.0.0B31"` | **固件(内部)版本** |
| `integrate_version` | `"CN_ZTE_MU5250V1.0.0B31"` | 集成版本 |

## 九、其它可用小接口

| 服务/方法 | 返回 | 含义 |
|---|---|---|
| `zwrt_deviceui/zwrt_deviceui_direct_power_mode_get` `{moduleName:"web"}` | `{enable:"1"}` | **直供电模式**（插线绕过电池）开关 |
| `zwrt_deviceui/zwrt_deviceui_touch_status_get` `{}` | `{lcd_login:""}` | 屏幕登录状态 |
| `zwrt_fota_res.api/get_update_status` `{moduleName:"web"}` | `{status:0}` | 固件升级状态 |
| `zwrt_web/web_language_get` `{}` | `{web_language:"zh-cn"}` | 界面语言 |
| `zwrt_web/web_quick_settings_init_flag_get` `{}` | `{quick_settings_init_flag:"1"}` | 是否已过初始化向导 |
| `zwrt_web/web_login_info` `{}` | `{login_fail_num, zte_web_sault}` | 登录失败次数 + 登录盐值（登录加密用） |
| `zwrt_web/web_crt_get` `{}` | `{result:"-----BEGIN PUBLIC KEY-----..."}` | 登录 RSA 公钥（登录加密用） |
| `zwrt_nfc/zwrt_nfc_wifi_get` `{}` | `{switch:"1",flag:"2"}` | NFC 一碰连开关 |
| `zwrt_wlan/get_wifi_moving_info` `{}` | `{wifi_moving_status:""}` | Wi-Fi 迁移状态 |
| `uci get` `zwrt_sleep`/`ztmp_time` | `{SysIdTime:"10"}` | 休眠时间设置 |

## 十、实测**需要登录**才能拿的（供参考）

以下全部返回 `{"error":{"code":-32002,"message":"Access denied"}}`：

- **接入设备数量/列表**：`router_get_user_list_num`、`router_wireless_access_list`、`router_lan_access_list`、`router_get_arptable`、`router_offline_list`
- **套餐流量限额设置**：`zwrt_data/get_wwandst_monthlimit`（本月已用字节数走 type=4 可以拿，限额值拿不到）
- **短信**：`zwrt_wms/*`（收发短信、未读数）
- **APN/锁频/邻居小区**：`zwrt_apn_object/*`、`nwinfo_get_lte_nbr_contents` 等
- **系统/时间/日志**：`zwrt_sntp/*`、`router_get_syslog`、uci `zwrt_zte_mdm`/`wireless`/`network` 等
- **登录后主接口**：`zwrt_router.api/router_get_status`（含更多 WAN 信息）

> 如需这些字段，要先走 `zwrt_web/web_login` 登录流程（RSA + AES 加密握手，拿 stok），插件系统目前不支持这种复杂流程。
