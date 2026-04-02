# Falcon_script

`Falcon_script/` は、Falcon 自動検査テスター向けの実行資材・設定・ドライバ・検査スクリプト・デバッグ用ワークスペースをまとめた作業ディレクトリです。  
この README は、現在確認できるフォルダ構造と主要ファイルにもとづいて、**このフォルダが何を持っているか**を把握するための概要だけを整理しています。

## このフォルダの見かた

大きく見ると、役割は次の 3 層に分かれます。

1. **テスター実行基盤**: `Driver/`, `Tester/`, `Programs/Falcon/`, `Tester.cfg`
2. **周辺資材**: `Device/`, `RELAY/`, `IMG/`, `Media/`, `Data/`
3. **開発・デバッグ作業領域**: `PgDebug/Falcon/`

## 主要ディレクトリ

```text
Falcon_script/
├── Data/                検査データ格納
├── Device/              Falcon 周辺デバイスの資材
├── Driver/              ドライバ / モニタ / 起動関連ファイル
├── EthLog/              Ethernet ログ置き場
├── IMG/                 画像検査関連パラメータ
├── Meas/                計測後処理向けファイル
├── Media/               効果音などのメディア
├── PgDebug/Falcon/      スクリプト変換・デバッグ用ワークスペース
├── Programs/Falcon/     テスター側プログラム配置先
├── RELAY/               リレー設定
├── ReprintData/         再印刷関連データ
├── SAV/ Temp/ Wave/ Work/
├── Tester/              テスターアプリ本体
├── VxWorks/             RTOS 関連置き場
├── debug_result/        デバッグ結果メモ
├── Offset.h             オフセット定義ヘッダ
└── Tester.cfg           テスター全体設定
```

## 中心になるコンポーネント

### `Tester.cfg`

テスター全体の設定ファイルです。確認できた代表値は以下の通りです。

- `ProgramType=Falcon`
- `MachineType=PCS52`
- `TesterPgStartApp=Driver\testerPgStart.exe`
- `LoadDllName=AtelierCtrlWrapper2.dll`
- `ConnectCmd=ateliercmd -connect -falcon2`

このファイルには、運転モード、DIO 配置、プリンタ、ローダ、作業 ID 読み取り、外部アプリ接続など、実機運用に関わる設定がまとまっています。

### `Driver/`

テスターのドライバ群と起動補助資材です。

- `Driver/Device.cfg`: 接続デバイスと Windows 側モニタアプリの対応
- `Driver/testerPgStart.exe`: `Tester.cfg` から参照される起動アプリ
- `Driver/Windows/`: CANFD / Relay / ADC などのモニタや時刻同期バッチ
- `Driver/Dll/`, `Driver/Include/`, `Driver/Lib/`: DLL / ヘッダ / ライブラリ群

`Device.cfg` では、たとえば ADC, CANFD, PO, Relay128 などに対応するモニタアプリ名が定義されています。

### `Tester/`

テスターアプリケーション本体です。`Tester/Bin/` には次のような実行資材が確認できます。

- `Tester.exe`
- `ScriptCompiler.exe`
- `WorkIF.exe`
- `Compile.bat`
- `AtelierCtrlWrapper2.dll`

実行ファイルに加えて、レイアウト XML、`WORKID_*.fmt`、各種 DLL が含まれます。

### `Programs/Falcon/`

テスター側プログラム資材の配置先として存在するディレクトリです。確認できた構成は次の通りです。

```text
Programs/Falcon/
├── Info/
├── Machine/
├── Process/
├── Product/
└── Script/PCS52/
    ├── FH/
    ├── FL/
    └── FR/
```

`Info/`, `Machine/`, `Process/`, `Product/`, `Script/PCS52/` という区切りを持つ、テスター側の資材ツリーとして参照できます。

### `PgDebug/Falcon/`

このフォルダが、開発・変換作業の中心です。

```text
PgDebug/Falcon/
├── 00_need_deploy/
├── Info/
├── Machine/
├── Process/
├── Product/
├── Script/
├── src/
├── dst/
└── modify_script.py
```

ここには、作業用のソース／出力先と、変換補助ファイルがまとまっています。

## `PgDebug/Falcon/` の役割分担

- `src/`: 変換元。`Info/PCS52/{ETH,FH,FL,FR}` と `Script/PCS52/{Common,ETH,FH,FL,FR}` を持つ
- `dst/`: 変換後の出力先。`src/` と同系統の構造を持つ
- `00_need_deploy/`: 必要ヘッダのデプロイ元
- `Info/`, `Script/`, `Machine/`, `Process/`, `Product/`: ワークスペース内の資材置き場
- `modify_script.py`: `src` から `dst` を生成し、家系別の置換や追記を自動化するスクリプト

## 検査工程の区分

`PgDebug/Falcon/src/Info/PCS52/` と `src/Script/PCS52/` では、少なくとも次の区分が確認できます。

- `ETH`
- `FH`
- `FL`
- `FR`

`FH`, `FL`, `FR` には `H1`〜`H7` と `MH2`, `MH4`, `MH5`, `MH6` を含むスクリプト名が見られます。  
また `Common/ADU/` には、共通ヘッダとして `EtherControl.h`, `EtherMacros.h`, `Ether_IF_3TEMP_H*.h`, `USBControl*.h` などがあります。

## `modify_script.py` の確認できた処理フロー

`PgDebug/Falcon/modify_script.py` は、バージョン番号の入力を受け取り、`src` 配下を `dst` 配下へコピーしたうえで、Falcon スクリプト群に補正を加える Python スクリプトです。

コード中では `C:\Falcon_script\PgDebug\Falcon\...` の固定 Windows パスが使われており、このディレクトリ構成を前提に動く作りになっています。

`main()` で確認できた主な流れ:

1. バージョン番号を入力
2. 必要ディレクトリを作成
3. `src/Info` と `src/Script` を `dst/` へコピー
4. コピー済み `.falcon` / `.script` を加工
5. 追加コマンドやパッチを適用
6. `.tii` を補正
7. `00_need_deploy/` の必要ファイルを配置

コード上で確認できた自動処理の例:

- `Info` / `Script` のコピー
- 家系別トークン置換
- `CMD13`, `eMMC_tap` 取得系の追記
- TPVM1 デバッグ用の置換
- `STEP900` / `STEP999` 周辺の調整
- デバッグブレーク挿入
- `.tii` 内の電源定義補正

## 生成元と生成先

README を読むときは、`PgDebug/Falcon/` で次の区別をすると追いやすくなります。

- **編集・保守対象の元データ**: `src/`
- **変換後の出力**: `dst/`
- **追加投入する補助ヘッダ**: `00_need_deploy/`
- **テスター実行側の資材置き場**: `Programs/Falcon/`, `Tester/`, `Driver/`

## 補足

- `Falcon_script/` 全体には `.script`, `.falcon`, `.h`, `.def`, `.dll`, `.exe` が多く含まれます
- `Device/` には ADC / CAN / Relay など複数デバイス用の資材があります
- `RELAY/`, `IMG/`, `Media/`, `Data/` などは、検査機運用を支える周辺フォルダです

## この README の範囲

この README は、**現在のフォルダ構造と主要ファイルの概要**に限定しています。  
実機セットアップ手順、Falcon スクリプト言語の文法、個別工程の詳細ロジックまでは扱いません。
