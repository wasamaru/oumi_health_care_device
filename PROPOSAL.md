# 提案書（量産版アップグレード計画）

## 概要

- 目的: 既存機の量産向けアップグレード（UI拡充、Type‑C化、音声対応、設定保存、認証対応、筐体改善）
- MCU: 当社推奨は`ESP32‑WROOM‑32E‑N16`。代替として`MBN52832 (WSM‑BL241‑ADA‑008)`を選択肢提示（最終選定は顧客）。
- 端子: 電源入力と出力ポートを`USB Type‑C`へ統一。
- 出力: 4chのうち2chをスピーカー構成と共用（自動認識）。
- モーター: シナノ電子「SD‑0820CY」採用。
- オーディオ: 受動SP + 5V Class‑D、MOSFETはアンプの`/EN`制御に使用。
- 設定保存: microSD（SPI）。
- UI: MODE/VOL/PITCHボタン追加、同時押し、LEDでモード表示。
- 認証: Bluetooth SIG は顧客側で取得。
- 筐体: 薄型・丸み・スリム化。ESP32厚さ3.1mmはType‑C等と同程度で機構設計次第で整合可能。

## アップデート内容（概要表）

| ブロック | アップデート概要 |
|---|---|
| 端子 | 入力`CN1`および出力`CN4–CN7`をUSB Type‑Cへ統一 |
| 無線／MCU | 推奨`ESP32‑WROOM‑32E‑N16`、代替`MBN52832`（最終選定は顧客） |
| 出力 4ch | 2chをスピーカーと共用（自動認識）、MOSFETはアンプ`/EN`制御に使用 |
| モーター | シナノ電子`SD‑0820CY`採用 |
| スピーカー／アンプ | 受動SP + 5V Class‑D（推奨I²Sアンプ、代替アナログIN） |
| 設定保持 | microSD（SPI）追加 |
| UI／表示 | MODE/VOL/PITCHボタン追加、同時押し、LED表示 |
| 認証 | Bluetooth SIG は顧客側で取得 |

## 構成のブロック図

```mermaid
flowchart LR
    USB_C_IN[USB Type‑C (CN1)] -->|5V| DC_DC[TPS630250 3.3V]
    DC_DC --> PWR_MGMT[LTC2955]
    PWR_MGMT --> LOAD_SW[TPS22919]
    LOAD_SW --> MCU{{ESP32‑WROOM‑32E‑N16 \n(代替: MBN52832)}}

    MCU --> UI[Buttons: MODE/VOL/PITCH \nLED Mode Indication]
    MCU --> microSD[microSD (SPI)]
    MCU --> AMP_EN[/EN Control]

    subgraph Outputs
      MOSFETs[4ch MOSFET (FDV305N)]
      USB_C_OUT[USB Type‑C x4 (CN4–CN7)\nSBU1: ID_ADC]
    end

    LOAD_SW --> MOSFETs
    MOSFETs --> USB_C_OUT

    subgraph Audio Path
      MCU -- I²S --> AMP[MAX98357A Class‑D]
      AMP --> SPK[Speaker 8Ω/0.5–1W]
      AMP_EN --> AMP
    end
```

## お見積り

- ユニットプライス（単価）
  - PoC: **¥15,000**（光造形 or FDM 筐体）
  - EVT: **¥25,000**（切削筐体）
  - DVT: **成型品（単価は別途）**／金型製造費あり
- 期間は空欄のまま記載します（数量・小計は非表示）。

### 原理試作 (PoC)

| 項目 | 単価(円) | 期間 | 備考 |
|---|---:|---|---|
| メカ | 15000 |  | 光造形 or FDM 筐体 |
| 回路設計 | 15000 |  | Type‑C/出力共用/AMP/SD 等 |
| 基板設計 | 15000 |  | 4層、I²S/USB/SD配線考慮 |
| デザイン（フルデザイン） | 15000 |  | 外観/筐体モデリング一式 |
| 簡易デザイン | 15000 |  | 形状簡易案/暫定外装 |
| FW開発 | 15000 |  | ESP32想定（I²S/SD/UI/認識） |

### EVT（Engineering Validation Test）

| 項目 | 単価(円) | 期間 | 備考 |
|---|---:|---|---|
| メカ | 25000 |  | 切削筐体 |
| 回路設計 | 25000 |  | 量産前修正/マージン確認 |
| 基板設計 | 25000 |  | DFM/テストポイント反映 |
| デザイン（フルデザイン） | 25000 |  | 量産前外装チューニング |
| 簡易デザイン | 25000 |  | ライト版意匠調整 |
| FW開発 | 25000 |  | 安定化/自動認識/モード運用確認 |

### DVT（Design Validation Test）

| 項目 | 単価(円) | 期間 | 備考 |
|---|---:|---|---|
| メカ |  |  | 成型品 |
| 回路設計 |  |  | 公差/量産バラツキ対策 |
| 基板設計 |  |  | パネル化/製造条件最適化 |
| デザイン（フルデザイン） |  |  | 最終外観/質感調整 |
| 簡易デザイン |  |  | 低コスト版検討（必要時） |
| FW開発 |  |  | 量産向け最終調整/検査機能 |
| 金型製造費 |  |  | DVT用 金型 |

## 付記

- 本提案は`docs/アップグレード計画_整理（メカ・ファーム・回路／流用・改修ブロック）.md`の内容を基に構成。
- MCU最終選定は顧客判断。ESP32厚さは他部品同等で、筐体設計で薄型化は成立可能。
- スピーカーはI²S推奨、代替としてPWM→RC→アナログINも可。
