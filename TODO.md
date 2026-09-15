# TODO / 已知待改进项

记录 `dev` 分支 Garmin 国际版支持的 Code Review 发现，后续改进时逐项处理。

## 待修复

### 1. `is_garmin_logged_in` 不识别国际版活动页 `/app/activities`（Critical）
- 位置：`SyncOnelapToXoss.py` 的 `is_garmin_logged_in()`
- 现象：`https://connect.garmin.com/app/activities` 既不含 `import-data` 也不含 `/modern/`，导致对已登录的活动页判为未登录；`get_latest_activity_garmin` 在非登录态起始路径下会直接 `return None`。
- 建议：末尾识别改为
  `return ('import-data' in current_url) or ('/modern/' in current_url) or ('/app/' in current_url)`

### 2. 活动列表解析依赖 CSS class 哈希后缀（Medium）
- `ActivityListItem_activityDate__HjqpL` 等由 CSS Modules 生成，Garmin 前端改版会失效。
- 已用 `[class*=...]` 前缀匹配缓解，仍需留意。

### 3. `_wait_garmin_activity_list` 返回值被忽略（Medium）
- 超时后仍继续解析空 HTML 再走 usageIndicators，功能正确；建议超时打一条 warn 便于排查。

### 4. `force_click_garmin_login_button` 是尽力而为（Medium）
- Stencil `<g-button>` 若用私有状态锁 disabled，`host.disabled=false` 未必生效；仅作兜底，无害。

## 待观察 / 低优先级

### 5. `parse_garmin_iso_datetime` 按 naive 处理时间戳（Low）
- 剥离 `Z`/时区后按本地 naive 处理，与 OneLap 本地时间比对可能有时区偏差（属既有问题，未放大）。

### 6. `settings.ini.example` 注释仍写旧域名（Low）
- `connect.garmin.cn/com` 实际已跳转到 `connectus.*`，文档可更新。

## 待调研

### 7. 国际版精确到秒的活动时间
- 当前活动列表只有日期粒度（如 `Sep 13`）；如需精确到秒，需接入国际版 `activitylist-service` 接口，其鉴权/路径与国区可能不同。

### 8. 阻止自动跳转国区
- 现象：登录 SSO 时 `service` 指向 `connectus.garmin.cn`（`clientId=GarminConnectUSCN`）。
- 分两阶段：登录前由 IP/地区 Cookie 决定，登录后由账号注册区域决定（后者无法代码绕过）。
- 待诊断：未登录时 `service` 后缀、挂代理后是否变化，据此定位是 IP 还是账号问题。
