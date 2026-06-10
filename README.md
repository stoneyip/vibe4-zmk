# Vibe4 — 5 键 Vibe Coding 小键盘（ZMK / nRF52840 SuperMini / macOS）

5 个键的蓝牙小键盘，专为 macOS 上 vibe coding 的高频动作设计：召唤语音输入、提交、向下翻、撤销、Esc / 蓝牙切换。

- 主控：`SuperMini nRF52840`（与 Nice!Nano v2 引脚兼容，按 `nice_nano_v2` 编译）
- 固件：`ZMK` v0.3.0
- 连接：USB-C 有线 + BLE（5 个 profile）

---

## 1. 硬件接线（直连，不需要二极管）

| 物理键 | 板子丝印 | nRF52840 GPIO | 接法 |
|--------|----------|---------------|------|
| K1     | D18      | P0.31         | 一端 → D18，另一端 → GND |
| K2     | D19      | P0.29         | 一端 → D19，另一端 → GND |
| K3     | D20      | P0.02         | 一端 → D20，另一端 → GND |
| K4     | D21      | P1.15         | 一端 → D21，另一端 → GND |
| K5     | —        | P1.13         | 一端 → P1.13，另一端 → GND |

> 全部 GPIO 内部上拉，按下即拉低。可选：`BAT+` / `GND` 接 LiPo（≥110 mAh），`BAT+`–`RAW` 之间串滑动开关做硬关机。

---

## 2. 按键映射

### BASE 层（默认）

| 键 | 行为 | 用途 |
|----|------|------|
| K1 | `Globe`（macOS Fn） | 激活语音输入 / 切换输入法 |
| K2 | `Enter` | 提交 / 发送（Cursor、ChatGPT、Copilot Chat、终端） |
| K3 | `↓` | 向下翻日志、看 AI 回复、选下一项 |
| K4 | `⌘ Z` | 撤销 |
| K5 | **短按 = `Esc`**，**长按 ≥500ms = 进入 FN 层** | 关弹窗 / 退出 vim / 长按时进入蓝牙控制层 |

> K5 用 hold-tap：`tapping-term-ms = 500`。500ms 以内松开 = Esc；按住超过 500ms = 进 FN 层（FN 层期间松手即退出）。

### FN 层（按住 K5 时）

| 键 | 行为 | 用途 |
|----|------|------|
| K1 | `BT_SEL 0` | 切到蓝牙 profile 0（电脑） |
| K2 | `BT_SEL 1` | 切到蓝牙 profile 1（手机） |
| K3 | `BT_NXT`   | 顺序切到下一个 profile |
| K4 | `BT_CLR`   | 清掉**当前** profile 的配对（救急用） |
| K5 | —（保持 Fn） | 维持层切换 |

---

## 3. 蓝牙使用流程

ZMK 把 BLE 分成 5 个独立的 **profile**，每个 profile 对应一台主机。设备**不自动跳台**，需要手动切。

### 3.1 第一次配对电脑（profile 0）

1. USB 接电脑或上电。
2. 按住 K5（≥500ms）→ 按 K1 → 松手（= `BT_SEL 0`）。
3. 电脑蓝牙搜 `Vibe4`，点击连接。本固件已禁用 6 位数字配对码（Just Works），直接连上。
4. 测试 K1~K5 是否生效。

### 3.2 第一次配对手机（profile 1）

1. 按住 K5 → 按 K2 → 松手（= `BT_SEL 1`）。当前广播切到 profile 1。
2. 手机蓝牙搜 `Vibe4`，连上。

> profile 0 和 profile 1 是互不影响的两套配对。切到 profile 1 时电脑会断开，但电脑的配对记录仍保留在 profile 0 上。

### 3.3 日常切换电脑 / 手机

- 切回电脑：按住 K5 → 按 K1 → 松手。
- 切到手机：按住 K5 → 按 K2 → 松手。
- 不知道当前在哪台：按住 K5 连按 K3 几次（`BT_NXT`）轮询。

### 3.4 重新配对（故障恢复）

某台主机怎么都连不上时（典型现象：搜得到 `Vibe4` 但连不上 / 一直转圈）：

1. 按住 K5 进入 FN 层。
2. 在 FN 层用 K1~K3 切到那台对应的 profile。
3. **保持按住 K5**，按 K4 → 触发 `BT_CLR`，清掉这个 profile 的旧配对。
4. 在主机蓝牙里删除原来的 `Vibe4` 条目。
5. 重新配对一次。

---

## 4. 续航与休眠

- 5 分钟无操作自动 deep sleep（按任意键唤醒）。
- macOS / iOS 蓝牙菜单可看到电池百分比（已开 `CONFIG_ZMK_BATTERY_REPORTING`）。
- 110~250 mAh 的 LiPo 一般可用 1 个月以上。长期不用建议用滑动开关物理断电。

---

## 5. 烧录方法

1. 改 `config/vibe4.keymap` 后 `git push`。
2. 等 GitHub Actions 跑完（一般 1~2 分钟）。
3. Actions 页面 → 最新 run → Artifacts → 下载 `firmware`。
4. 解压得到 `vibe4-nice_nano_v2-zmk.uf2`。
5. 板子双击 `RST` 进 bootloader，电脑出现 `NICENANO` 盘符。
6. 把 `.uf2` 拖进去，自动重启即生效。

---

## 6. 仓库结构

```
zmk-config/
├── .github/workflows/build.yml      # 调 zmk@v0.3.0 的 build-user-config
├── build.yaml                       # board=nice_nano_v2, shield=vibe4
└── config/
    ├── west.yml                     # 锁定 zmk v0.3.0
    ├── vibe4.keymap                 # 键位（最常改这个）
    ├── vibe4.conf                   # BLE / 省电 / 配对策略
    └── boards/shields/vibe4/
        ├── Kconfig.shield
        ├── Kconfig.defconfig
        ├── vibe4.zmk.yml
        └── vibe4.overlay            # 直连扫描 + GPIO 映射
```

### 关键决策（踩过的坑）

- **板子选 `nice_nano_v2`**：SuperMini 引脚兼容，复用 Nice!Nano v2 的 board 定义，省得自己写 board。
- **GPIO 用 `&gpio0/&gpio1` 直接寻址**：SuperMini 上 D18~D21 实际接到 P0.31 / P0.29 / P0.02 / P1.15，与正版 Nice!Nano（P0.20/17/15/13）不同，用 `&pro_micro` 别名会错位。
- **`CONFIG_ZMK_BLE_PASSKEY_ENTRY=n`**：4~5 个键没法输 6 位 PIN，必须强制 Just Works。
- **ZMK 锁定 `v0.3.0`**：main 分支已切到 Zephyr 4.1 board variant，`west.yml` 和 workflow 都要锁同一个 tag。
- **K5 长按 500ms 才触发 Fn**：避免误触。FN 层只放蓝牙控制（BT_SEL / BT_NXT / BT_CLR），不放电源键，避免误关机。

---

## 7. 一行小抄

> **K5 短按 = Esc；按住 K5 + K1 / K2 = 切到电脑 / 手机；按住 K5 + K3 = 轮询；按住 K5 + K4 = 清当前蓝牙配对。**
