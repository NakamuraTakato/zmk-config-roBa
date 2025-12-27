# zmk-config-roBa プロジェクトガイド

## プロジェクト概要

このリポジトリは、**roBa（ロバ）**という名前の分割型キーボード用のZMKファームウェア設定です。

### roBaキーボードの特徴

- **分割型エルゴノミックキーボード**: 左右が分かれた40キー構成
- **右手側にトラックボール統合**: マウス不要で作業効率アップ
- **完全ワイヤレス**: Bluetooth接続で配線不要
- **高度なカスタマイズ**: ZMK Studioで動的に設定変更可能
- **ロータリーエンコーダー搭載**: 音量調整やスクロールに便利

### 対象ユーザー

- プログラマーや長時間キーボード作業をする方
- 手の移動を最小化したい方
- カスタマイズ性の高いキーボードを求める方

## 技術スタック

### ハードウェア

- **MCU**: Seeeduino XIAO BLE (nRF52840)
  - Bluetooth Low Energy対応
  - 省電力設計
- **トラックボールセンサー**: PMW3610 (右手側のみ)
  - 解像度: 400 CPI
  - SPI接続
- **ロータリーエンコーダー**: 左右両側に搭載可能

### ソフトウェア

- **ファームウェア**: ZMK Firmware v0.3-branch
  - オープンソースのキーボードファームウェア
  - Zephyr RTOSベース
- **ビルドシステム**: West (Zephyrのメタツール)
- **CI/CD**: GitHub Actions
- **キーマップ可視化**: keymap-drawer

## ディレクトリ構造

```
zmk-config-roBa/
├── .github/workflows/      # GitHub Actions設定
│   ├── build.yml          # ファームウェア自動ビルド
│   └── draw.yml           # キーマップ図生成
│
├── boards/shields/roBa/   # ハードウェア定義
│   ├── roBa.dtsi         # 共通ハードウェア設定（Device Tree）
│   ├── roBa_L.conf       # 左手側ファームウェア設定
│   ├── roBa_L.overlay    # 左手側ハードウェアレイアウト
│   ├── roBa_R.conf       # 右手側ファームウェア設定
│   ├── roBa_R.overlay    # 右手側ハードウェアレイアウト
│   └── ...               # その他Kconfig関連ファイル
│
├── config/                # ユーザー設定
│   ├── roBa.keymap       # キーマップ定義（メイン設定ファイル）
│   ├── roBa.json         # キーボードレイアウト情報
│   └── west.yml          # 依存関係管理
│
├── keymap-drawer/         # キーマップ可視化
│   ├── roBa.svg          # キーマップ図（画像）
│   └── roBa.yaml         # 描画設定
│
├── zephyr/
│   └── module.yml        # Zephyrモジュール定義
│
├── build.yaml            # ビルドマトリクス設定
├── README.md             # プロジェクト説明
└── claude.md             # このファイル（開発ガイド）
```

### 主要ファイルの役割

| ファイル | 役割 | 編集頻度 |
|---------|------|---------|
| `config/roBa.keymap` | キー配置の設定 | 高 |
| `boards/shields/roBa/roBa_R.conf` | 右手側の機能設定 | 中 |
| `boards/shields/roBa/roBa_L.conf` | 左手側の機能設定 | 中 |
| `boards/shields/roBa/*.overlay` | ハードウェアピン配置 | 低 |
| `build.yaml` | ビルド対象の定義 | 低 |
| `config/west.yml` | ZMK本体のバージョン管理 | 低 |

## キーマップ構成

### 7レイヤーシステム

| レイヤー番号 | 名前 | 用途 | アクセス方法 |
|------------|------|------|------------|
| 0 | default_layer | 通常のQWERTY配列 | デフォルト |
| 1 | FUNCTION | ファンクションキー (F1-F13) | レイヤーキーで切替 |
| 2 | NUM | テンキー・記号入力 | レイヤーキーで切替 |
| 3 | ARROW | 矢印キー・ナビゲーション | レイヤーキーで切替 |
| 4 | MOUSE | マウスクリック | トラックボール動作時自動 |
| 5 | SCROLL | スクロールモード | レイヤーキーで切替 |
| 6 | layer_6 | Bluetooth設定・リセット | 特殊キーコンボ |

### 特殊機能

#### 1. コンボキー
複数のキーを同時押しすることで、別のキーを出力する機能。

例:
- キー11 + キー12 → TAB
- キー20 + キー21 → ダブルクォーテーション

#### 2. Mod-Tap（モッドタップ）
- **短押し**: 通常のキーとして動作
- **長押し**: モディファイアキー（Shift、Ctrl等）として動作

#### 3. 自動レイヤー切り替え
トラックボールを動かすと、自動的にマウスレイヤー(4)に遷移します。
700ms操作がないと元のレイヤーに戻ります。

## ビルド方法

### 自動ビルド（推奨）

1. **GitHub Actionsで自動ビルド**
   - `main`ブランチにpushすると自動的にビルドが開始
   - ビルド完了後、Actionsタブから`.uf2`ファイルをダウンロード

2. **ビルド成果物**
   - `roBa_R-seeeduino_xiao_ble-zmk.uf2` - 右手側ファームウェア
   - `roBa_L-seeeduino_xiao_ble-zmk.uf2` - 左手側ファームウェア
   - `settings_reset-seeeduino_xiao_ble-zmk.uf2` - 設定リセット用

### ローカルビルド（上級者向け）

```bash
# 1. ZMK開発環境のセットアップ
# （公式ドキュメント参照: https://zmk.dev/docs/development/setup）

# 2. このリポジトリをクローン
git clone https://github.com/yourusername/zmk-config-roBa.git
cd zmk-config-roBa

# 3. Westで依存関係を取得
west init -l config/
west update

# 4. ビルド実行
west build -p -d build/roBa_R -b seeeduino_xiao_ble -- -DSHIELD=roBa_R
west build -p -d build/roBa_L -b seeeduino_xiao_ble -- -DSHIELD=roBa_L
```

## ファームウェアの書き込み方法

1. **RSTピンを2回短く押す** → ブートローダーモードに入る
2. PCに`XIAO-SENSE`という名前のUSBドライブが表示される
3. `.uf2`ファイルをドラッグ&ドロップ
4. 自動的に再起動してファームウェアが適用される

### 初回セットアップ手順

1. **右手側**に`roBa_R-*.uf2`を書き込み
2. **左手側**に`roBa_L-*.uf2`を書き込み
3. 両方の電源を入れる
4. 自動的にBluetooth接続が確立される

## 開発とカスタマイズ

### キーマップの変更手順

1. **`config/roBa.keymap`を編集**
   ```c
   // 例: デフォルトレイヤーのキー配置を変更
   bindings = <
       &kp TAB   &kp Q  &kp W  &kp E  // キーを変更
       // ...
   >;
   ```

2. **GitHubにpush**
   ```bash
   git add config/roBa.keymap
   git commit -m "キーマップを変更: Tabキーの位置を調整"
   git push
   ```

3. **GitHub Actionsで自動ビルド**
   - Actionsタブで進行状況を確認
   - 完了後、新しい`.uf2`ファイルをダウンロード

4. **ファームウェアを書き込み**

### ZMK Studioを使った動的設定（右手側のみ）

ZMK Studioを使えば、ファームウェアを再ビルドせずにキーマップを変更できます。

1. 右手側をUSBケーブルでPCに接続
2. ZMK Studio（Webアプリまたはデスクトップアプリ）を開く
3. GUIでキーマップを編集
4. 保存すると即座に反映

**注意**: 左手側は対応していません（右手側の設定のみ）

### トラックボール感度の調整

`boards/shields/roBa/roBa_R.conf`を編集:

```conf
# CPI値を変更（大きいほど速く動く）
CONFIG_PMW3610_CPI=400          # 現在の設定
# CONFIG_PMW3610_CPI=800        # より速い設定

# スクロール感度を変更（小さいほど敏感）
CONFIG_PMW3610_SCROLL_TICK=16   # 現在の設定
# CONFIG_PMW3610_SCROLL_TICK=8  # より敏感な設定
```

## トラブルシューティング

### Bluetooth接続が不安定

1. **設定リセット**:
   - `settings_reset-*.uf2`を両側に書き込む
   - 再度通常のファームウェアを書き込む

2. **ペアリングのやり直し**:
   - レイヤー6の`BT_CLR`キーを押す
   - PCのBluetooth設定から"roBa"を削除
   - 再ペアリング

### トラックボールが反応しない

1. **ハードウェア接続を確認**:
   - PMW3610センサーのSPI配線を確認
   - IRQピン、CSピンが正しく接続されているか

2. **設定を確認**:
   ```conf
   CONFIG_PMW3610=y                    # 有効化されているか
   CONFIG_ZMK_POINTING=y               # ポインティングデバイス有効か
   ```

### ビルドエラーが発生

1. **依存関係のバージョンを確認**:
   - `config/west.yml`のZMKブランチが`v0.3-branch`になっているか
   - PMW3610ドライバーのリポジトリURLが正しいか

2. **エラーログを確認**:
   - GitHub ActionsのLogsタブで詳細を確認
   - Device Tree関連のエラーが多い場合は`.overlay`ファイルを確認

## プロジェクトの拡張

### 新しいレイヤーを追加

`config/roBa.keymap`に新しいレイヤーを定義:

```c
compatible = "zmk,keymap";

// 既存のレイヤー...

new_layer {
    bindings = <
        // 新しいキー配置を定義
    >;
};
```

### 新しいコンボを追加

```c
/ {
    combos {
        compatible = "zmk,combos";

        new_combo {
            timeout-ms = <50>;
            key-positions = <10 11>;  // 同時押しするキーの位置
            bindings = <&kp ESC>;      // 出力するキー
        };
    };
};
```

### マクロの追加

```c
/ {
    macros {
        my_macro: my_macro {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp L &kp O &kp L>;  // "LOL"と入力
        };
    };
};
```

## Git運用

### ブランチ戦略

このプロジェクトでは以下のブランチ運用を推奨します:

- **main**: 安定版・本番用ファームウェア
- **develop**: 開発統合ブランチ（実験的機能）
- **feature/xxx**: 新機能開発用

### コミットメッセージ例

```bash
# 良い例
git commit -m "キーマップ変更: テンキーレイヤーにF13キーを追加"
git commit -m "トラックボール感度を400から600 CPIに変更"
git commit -m "修正: 左手側のロータリーエンコーダーが動作しない問題"

# 悪い例
git commit -m "update"
git commit -m "fix bug"
```

## 参考リソース

### 公式ドキュメント

- [ZMK Firmware公式サイト](https://zmk.dev/)
- [ZMK設定リファレンス](https://zmk.dev/docs/config)
- [ZMK Behavior一覧](https://zmk.dev/docs/behaviors)
- [Device Tree仕様](https://docs.zephyrproject.org/latest/build/dts/index.html)

### コミュニティ

- [ZMK Discord](https://zmk.dev/community/discord/invite)
- [ZMK GitHub Discussions](https://github.com/zmkfirmware/zmk/discussions)

### 関連プロジェクト

- [PMW3610ドライバー](https://github.com/kumamuk-git/zmk-pmw3610-driver) - このプロジェクトで使用
- [keymap-drawer](https://github.com/caksoylar/keymap-drawer) - キーマップ可視化ツール
- [ZMK Studio](https://github.com/zmkfirmware/zmk-studio) - GUIキーマップエディタ

## 用語集

| 用語 | 説明 |
|-----|------|
| **ZMK** | Zephyr RTOS Mechanical Keyboard - オープンソースのキーボードファームウェア |
| **Zephyr** | リアルタイムOS。ZMKのベースとなっている |
| **West** | Zephyrプロジェクトのメタツール。依存関係管理とビルドに使用 |
| **Device Tree** | ハードウェア構成を記述する形式。`.dtsi`や`.overlay`ファイル |
| **UF2** | USB Flashing Format。ファームウェアのバイナリ形式 |
| **Shield** | ZMKでのキーボード定義。ボードとは別の概念 |
| **Board** | マイコン本体（この場合はSeeeduino XIAO BLE） |
| **CPI** | Counts Per Inch。トラックボールの感度単位 |
| **Layer** | レイヤー。複数のキー配置を切り替える機能 |
| **Combo** | コンボ。複数キー同時押しで別のキーを出力 |
| **Mod-Tap** | モッドタップ。短押しと長押しで動作が変わるキー |
| **Behavior** | ZMKでのキー動作定義（&kp、&mo、&lt等） |
| **BLE** | Bluetooth Low Energy。低消費電力Bluetooth |
| **Split Keyboard** | 分割型キーボード。左右が分かれている |
| **Rotary Encoder** | ロータリーエンコーダー。回転式のノブ |

## プロジェクト履歴

### 主要な変更履歴（最近のコミットより）

- `194fc14` - ZMK v0.3-branchを参照するように変更
- `66454a0` - 左手側でもポインティング機能を有効化(`CONFIG_ZMK_POINTING=y`)
- `7187f0c` - 左側マウスキー割り当てを一時的に無効化（Revert）
- `9f9dad1` - boards/shieldsをconfig外に移動（リファクタリング）

### 既知の制限事項

- ZMK Studioは右手側のみ対応（左手側は非対応）
- トラックボールは右手側のみ（左手側には搭載不可）
- バッテリー駆動時間はBluetooth使用状況により変動

## 開発のヒント

### デバッグ方法

1. **ログ出力を有効化**:
   `.conf`ファイルに追加:
   ```conf
   CONFIG_ZMK_USB_LOGGING=y
   CONFIG_LOG_MODE_MINIMAL=n
   CONFIG_LOG_MODE_DEFERRED=y
   ```

2. **シリアルコンソールで確認**:
   ```bash
   # macOS/Linux
   screen /dev/tty.usbmodem* 115200

   # Windows
   # PuTTYやTera Termを使用
   ```

### パフォーマンス最適化

- **ポーリングレート調整**: `CONFIG_PMW3610_POLLING_RATE_*`
- **省電力設定**: `CONFIG_ZMK_SLEEP=y`で未使用時スリープ
- **BLE最適化**: `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`で電波強度アップ

### よくある質問

**Q: キーマップを変更したのにビルドに反映されない**
A: GitHub Actionsのキャッシュが原因の可能性があります。Actions画面で手動で再実行してください。

**Q: 左右が逆に動作する**
A: `.overlay`ファイルの`transforms`定義を確認してください。

**Q: トラックボールの動きが遅い/速い**
A: `CONFIG_PMW3610_CPI`の値を調整してください（200-1600の範囲）。

---

## まとめ

このプロジェクトは、ZMKファームウェアを使用した高度にカスタマイズ可能な分割型キーボードの設定です。トラックボール統合により、マウス不要で効率的な作業環境を実現できます。

開発を進める際は、小さな変更から始めて、GitHub Actionsでのビルドを確認しながら進めることをお勧めします。

何か問題が発生した場合は、ZMKコミュニティやこのREADMEのトラブルシューティングセクションを参照してください。

Happy Hacking! 🎹
