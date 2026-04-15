# Domain Classification Rules

When extracting content from conversation history, use these rules to automatically assign each topic to the correct category, subcategory, and article slot.

---

## Decision Tree

```
Is the topic about Qualcomm SoC internals, boot, DSP, MCU, or camera?
  → Category: QCOM  (color: "green", icon: "Cpu")
  → See "QCOM Subcategory Rules" below

Is the topic about a physical sensor IC, driver, protocol, or calibration?
  → Category: Sensor  (color: "blue", icon: "Gauge")
  → See "Sensor Subcategory Rules" below

Is the topic about both (e.g., SLPI running on QCOM but handling sensor data)?
  → Primary: QCOM › Sensor Subsystem
  → Cross-link note in Sensor › General article
```

---

## QCOM Subcategory Rules

| Keywords in conversation | Subcategory id | Subcategory title |
|---|---|---|
| non-HLOS, HLOS, image file, `NON-HLOS.bin`, heterogeneous | `non-hlos-overview` | non-HLOS 架構總覽 |
| PBL, XBL, SBL, TrustZone, TZ, HYP, secure boot, chain of trust, UEFI, fastboot | `bootloader` | 啟動流程（Boot Chain） |
| SLPI, SSC, ADSP, LPASS, sensor hub, always-on, island mode, sensor fusion | `sensor-subsystem` | 感測器子系統 |
| CDSP, Hexagon, HVX, HTA, NPU, AI engine, SNPE, QNN, FastRPC | `dsp-subsystems` | DSP 子系統 |
| SMCU, SAIL MCU, ISO 26262, ASIL, functional safety, automotive | `mcu-subsystems` | MCU 子系統 |
| CMCU, Wi-Fi, Bluetooth, BLE, WPSS, WCN, connectivity | `mcu-subsystems` | MCU 子系統 |
| IFE, IPE, BPS, ISP, MIPI CSI, camera pipeline, cam-server, Camera HAL | `camera-isp` | 相機 ISP 管線 |
| AOP, RPM, PMIC, regulator, clock, power management, rpmh | `power-modem` | 電源與通訊子系統 |
| MPSS, modem, baseband, 4G, 5G, LTE, NR, QMI, RIL, VoLTE | `power-modem` | 電源與通訊子系統 |

### QCOM Article Placement Rules

- **One concept = one article.** If a conversation thread covers both SLPI and ADSP in depth, create two separate articles and link them via a `callout-info` cross-reference.
- **Correction articles**: If the user asked a question that revealed a misconception (e.g., "CMCU handles camera?"), the article correcting it MUST open with a `callout-warning` block.
- **Architecture overview articles** go in `non-hlos-overview`; subsystem-specific deep dives go in their own subcategory.

---

## Sensor Subcategory Rules

| Keywords in conversation | Subcategory id | Subcategory title |
|---|---|---|
| CapSense, SAR, SX9204, SX9324, SX9378, proximity, capacitive, CSIO, phase, scan period | `capsense` | CapSense / SAR |
| IMU, accelerometer, gyroscope, magnetometer, BMI, LSM, ICM | `imu` | IMU / 慣性感測器 |
| pressure, barometer, BMP, LPS | `environmental` | 環境感測器 |
| ToF, LiDAR, VL53, distance, ranging | `proximity-optical` | 光學近接 / ToF |
| I2C, SPI, UART, register map, driver, kernel, IIO, input subsystem | `driver-protocol` | 驅動與協定 |
| calibration, offset, threshold, noise, SNR, tuning | `calibration` | 校準與調校 |

### Sensor Article Placement Rules

- **Register-level questions** (e.g., "what does PHEN register do?") → `capsense` or the relevant IC subcategory, not `driver-protocol`.
- **Kernel driver architecture questions** (e.g., "how does the IIO framework work?") → `driver-protocol`.
- **Timing / scan period questions** → same subcategory as the IC (e.g., `capsense` for SX9204 scan period).
- If a subcategory does not yet exist in the data, create it with a placeholder article rather than forcing content into a wrong subcategory.

---

## Shared Classification Rules (apply to both domains)

### Keyword Signals That Indicate a New Article Is Needed

- User asks "what is X?" or "what does X do?" → create a concept article
- User asks "what is the difference between X and Y?" → create a comparison article (use a table)
- User asks "why does X happen?" or "how does X work?" → create a deep-dive article
- User corrects a previous answer → update or replace the existing article; add `callout-warning`

### Tags

Assign 3–6 short tags per article. Use English for technical acronyms, Traditional Chinese for descriptive terms:

```typescript
tags: ["SLPI", "ADSP", "感測器", "低功耗", "Island Mode"]
tags: ["SX9204", "CapSense", "SAR", "Scan Period", "Phase"]
tags: ["XBL", "PBL", "啟動", "TrustZone", "安全"]
```

### Placeholder Subcategories

If a domain (e.g., `Sensor › IMU`) is referenced in conversation but has no content yet, always create the subcategory with one placeholder article. Never omit a subcategory that was mentioned by the user.

```typescript
{
  id: "imu",
  title: "IMU / 慣性感測器",
  description: "加速度計、陀螺儀、磁力計的驅動與融合",
  icon: "Activity",
  articles: [
    {
      id: "imu-placeholder",
      title: "IMU 感測器內容（即將加入）",
      // ... placeholder content
    }
  ]
}
```

---

## Complete Category Skeleton

Use this as the starting `categories` array in `kb-data.ts`. Remove subcategories that have no content and no placeholder need.

```typescript
export const categories: Category[] = [
  {
    id: "qcom",
    title: "Qualcomm",
    description: "Qualcomm Snapdragon SoC 平台架構、non-HLOS 韌體與子系統解析",
    icon: "Cpu",
    color: "green",
    subcategories: [
      { id: "non-hlos-overview", title: "non-HLOS 架構總覽",  icon: "Layers",    articles: [] },
      { id: "bootloader",        title: "啟動流程（Boot Chain）", icon: "Play",  articles: [] },
      { id: "sensor-subsystem",  title: "感測器子系統",        icon: "Activity",  articles: [] },
      { id: "dsp-subsystems",    title: "DSP 子系統",          icon: "Zap",       articles: [] },
      { id: "mcu-subsystems",    title: "MCU 子系統",          icon: "Microchip", articles: [] },
      { id: "camera-isp",        title: "相機 ISP 管線",       icon: "Camera",    articles: [] },
      { id: "power-modem",       title: "電源與通訊子系統",    icon: "Radio",     articles: [] },
    ]
  },
  {
    id: "sensor",
    title: "Sensor",
    description: "感測器 IC 驅動開發、校準、協定與應用",
    icon: "Gauge",
    color: "blue",
    subcategories: [
      { id: "capsense",          title: "CapSense / SAR",      icon: "Waves",     articles: [] },
      { id: "imu",               title: "IMU / 慣性感測器",    icon: "Activity",  articles: [] },
      { id: "driver-protocol",   title: "驅動與協定",          icon: "Code",      articles: [] },
      { id: "calibration",       title: "校準與調校",          icon: "Sliders",   articles: [] },
    ]
  }
];
```
