# Vibe4 — 4 键 Vibe Coding 小键盘（ZMK / nRF52840 SuperMini / macOS）

一台只有 4 个键、走蓝牙的 vibe coding 触发器，专为 macOS 上"召唤输入法 / 撤销 / 看下一段 / 提交"四个高频动作设计。

- 主控：`SuperMini nRF52840`（与 Nice!Nano v2 引脚兼容，编译时复用 `nice_nano_v2`）
- 固件：`ZMK` v0.3.0
- 连接：USB-C 有线 + BLE（最多 5 个蓝牙 profile）
- 系统：主要在 macOS，副带 iOS / iPadOS / Android

---

## 1. 硬件接线

板子丝印是 `D18 / D19 / D20 / D21`，对应到 nRF52840 的真实 GPIO 是：

| 物理键 | 板子丝印 | nRF52840 GPIO | 接法 |
|--------|----------|---------------|------|
| K1     | D18      | P0.31         | 一端 → D18，另一端 → GND |
| K2     | D19      | P0.29         | 一端 → D19，另一端 → GND |
| K3     | D20      | P0.02         | 一端 → D20，另一端 → GND |
| K4     | D21      | P1.15         | 一端 → D21，另一端 → GND |

> 全部直连，不需要二极管，不需要矩阵。GPIO 内部上拉，按下 → 拉低。

可选：`BAT+` / `GND` 接 3.7V 锂电（≥110 mAh 即够）；`BAT+`–`RAW` 之间串一个滑动开关做硬关机。

---

## 2. 按键映射

### BASE 层（默认）

| 键 | 行为 | 用途 |
|----|------|------|
| K1 | **短按 = `Globe`（macOS Fn / 输入法切换）**，长按 = 进入 FN 层 | 单按激活/切换输入法；长按时切到 FN |
| K2 | `⌘ Z` | 撤销（vibe coding 救命键） |
| K3 | `↓` 方向键 | 翻日志 / 看 AI 回复 / 选下一个候选 |
| K4 | `Enter` | 提交 / 发送（Cursor、ChatGPT、Copilot Chat、终端） |

> K1 用的是 ZMK 的 hold-tap：短按下去松开 = Fn 键（macOS 上即"地球键"，长按等于触发 macOS 的 Fn 行为）；按住不放 = 进入 FN 层。`tapping-term-ms = 200`，超过 200ms 才算"长按"。

### FN 层（按住 K1 时）

| 键 | 行为 | 用途 |
|----|------|------|
| K1 | —（保持 Fn） | 维持层切换 |
| K2 | `BT_SEL 0` | 切到蓝牙 profile 0（比如电脑） |
| K3 | `BT_SEL 1` | 切到蓝牙 profile 1（比如手机） |
| K4 | `BT_NXT`   | 顺序切到下一个蓝牙 profile |

### 组合键

| 组合 | 行为 | 用途 |
|------|------|------|
| 按住 K1 + 同时按 K4 | `BT_CLR` | 清掉当前 profile 的配对（救急用，必须在 FN 层内） |

---

## 3. 蓝牙使用流程（重点）

ZMK 把 BLE 分成 5 个独立的 **profile**，每个 profile 记一台主机。设备**不是自动跳台**的，必须你主动切。

### 3.1 第一次配对电脑（profile 0）

1. USB 接电脑或上电。
2. 按住 K1 → 按 K2 → 松开（= `BT_SEL 0`）。
3. 电脑蓝牙里搜 `Vibe4`，点击连接。本固件已禁用 6 位数字配对码（`Just Works`），直接连上即可。
4. 测试：按 K2 = 撤销，按 K3 = 向下，按 K4 = Enter。

### 3.2 第一次配对手机（profile 1）

1. 按住 K1 → 按 K3 → 松开（= `BT_SEL 1`）。这一步会把当前广播切到 profile 1。
2. 手机蓝牙搜 `Vibe4`，连上。

> 关键：profile 0 和 profile 1 是**互不影响的两套配对**。你切到 profile 1 时，电脑那边会断开，但电脑的配对记录仍然保留在 profile 0 上。

### 3.3 日常切换电脑 / 手机

- 切回电脑：按住 K1 → 按 K2 → 松开。
- 切到手机：按住 K1 → 按 K3 → 松开。
- 不知道当前在哪台？按住 K1 连按 K4 几次（= `BT_NXT`）轮询。

### 3.4 重新配对（故障恢复）

某台主机怎么都连不上时（典型现象：能搜到 `Vibe4` 但连不上 / 一直转圈）：

1. 按住 K1 进入 FN 层。
2. 用 FN 层的 `BT_SEL` 切到那一台对应的 profile（K2=0、K3=1）。
3. 在 FN 层里**同时按住 K1 + K4 约 50ms** → 触发 `BT_CLR`，清掉这个 profile 的旧配对。
4. 在主机蓝牙里删除原来的 `Vibe4` 条目。
5. 重新配对一次即可。

---

## 4. 续航与休眠

- 无操作 15 分钟自动 deep sleep，按任意键唤醒。
- macOS / iOS 蓝牙菜单可看到电池百分比（已开 `CONFIG_ZMK_BATTERY_REPORTING`）。
- 长期不用建议串个滑动开关物理断电，自放电几乎可以忽略。

---

## 5. 烧录方法（之后改键也是这一套）

1. 改 `config/vibe4.keymap` 后 `git push`。
2. 等 GitHub Actions 跑完（一般 1～2 分钟）。
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
    ├── vibe4.keymap                 # 键位（你最常改这个）
    ├── vibe4.conf                   # BLE / 省电 / 配对策略
    └── boards/shields/vibe4/
        ├── Kconfig.shield
        ├── Kconfig.defconfig
        ├── vibe4.zmk.yml
        └── vibe4.overlay            # 直连扫描 + GPIO 映射
```

### 6.1 关键决策（踩过的坑，别忘）

- **板子选 `nice_nano_v2`**：SuperMini 引脚兼容，复用 Nice!Nano v2 的 board 定义，省得自己写 board。
- **GPIO 用 `&gpio0/&gpio1` 直接寻址**：不用 `&pro_micro` 别名，因为 SuperMini 的 D18~D21 实际接到 P0.31 / P0.29 / P0.02 / P1.15，跟正版 Nice!Nano 的 D18~D21（P0.20/17/15/13）**不一样**，用别名会错位。
- **`CONFIG_ZMK_BLE_PASSKEY_ENTRY=n`**：4 个键没法输 6 位 PIN，必须强制 Just Works。
- **ZMK 锁定 `v0.3.0`**：main 分支已经切到 Zephyr 4.1 的 board variant，老的 user-config 会编译失败。`west.yml` 和 workflow 都要锁同一个 tag。

---

## 7. 一行小抄

> **K1 短按 = 切输入法；按住 K1 + K2 / K3 = 切到电脑 / 手机；按住 K1 + K4 = 清当前蓝牙配对。**
