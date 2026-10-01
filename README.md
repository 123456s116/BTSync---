

# BTSync - 蓝牙通知同步工具

在两台 Android 手机之间通过蓝牙（RFCOMM SPP）实时同步通知消息。
A 机收到微信/短信/QQ 等通知后，B 机自动重建同一条通知，保留原应用图标、标题、正文，点击可跳回原应用。

## 边界声明（重要）

- 本工具仅用于同步**你自己名下两台设备**的通知，请勿用于截取或伪造他人消息。
- Android 安全机制规定：第三方 App 发出的通知在通知栏归属处固定显示本 App 名称（BTSync），
  无法用普通权限完全冒充微信等来源标签；本方案已做到内容、图标、点击跳转与原应用一致。
  若追求"连来源标签都一模一样"，需要系统签名 / root / Xposed 注入，不在本工程范围内。

## 使用步骤

1. 用 Android Studio 打开本工程，Build 出 APK，安装到两台手机（或直接导入源码）。
2. 两台手机都开启蓝牙并互相配对。
3. 两台手机都进入"设置 → 通知使用权（Notification access）"，开启 BTSync。
4. 手机 A 打开 App 点【作为接收端：等待蓝牙连接】；手机 B 点【作为发送端：选择设备连接】。
5. 连接建立后：A 机任意 App 来通知 → B 机同步弹出相同通知（图标/标题/正文一致）；
   B 机来通知 → A 机同步弹出。双向同步。

## 权限说明

| 权限 | 用途 |
|------|------|
| BLUETOOTH_CONNECT / BLUETOOTH_SCAN | Android 12+ 蓝牙连接与扫描 |
| BLUETOOTH / BLUETOOTH_ADMIN | Android 11- 经典蓝牙 |
| POST_NOTIFICATIONS | Android 13+ 发送通知 |
| BIND_NOTIFICATION_LISTENER_SERVICE | 通知使用权（监听通知） |

## 技术要点

- 蓝牙经典模式 RFCOMM + SPP UUID（00001101-0000-1000-8000-00805F9B34FB），无需配对后的额外鉴权。
- 消息格式：JSON，字段 type/pkg/title/text/time，通过 DataOutputStream.writeUTF 定长传输。
- 发送端：NotificationListenerService 监听 onNotificationPosted，忽略自身通知防止回环。
- 接收端：NotificationManager 重建通知，大图标用 PackageManager.getApplicationIcon(包名) 取原应用图标；
  点击使用 getLaunchIntentForPackage 跳回原应用。
