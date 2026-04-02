# Session Handoff Log

> これは**生の会話トランスクリプトそのものではなく**、2026-04-02 時点の実用的な引き継ぎサマリー。  
> Session ID: `fc19ed7c-4ba0-486a-87a9-12e4d37a7e6c`

## セッション概要

このセッションでは、`Falcon_script/` 配下のコードベース調査と README 再構成、TPVM1 のベンチ NG 調査、そして FR 常温 H7 script のデバッグ派生版作成まで進めた。  
GitHub 操作、commit、push は今回のスコープ外として実施していない。

## 完了済み作業

- `Falcon_script/README.md` を、確認済み事実だけに基づく概要 README に再構成
- `Falcon_script` 全体構成、`Tester.cfg`、`PgDebug/Falcon/modify_script.py`、`Programs/Falcon`、`Driver` 周辺を調査
- TPVM1 ベンチ NG について、手動ログ・ベンチログ・FR/H7 script・`.tii`・共通 macro / function を突き合わせ
- 「隠れた外部設定ファイル不足」よりも、「手動成功時の API フローと bench script フローの差」が主因候補と整理
- `falcon-h7-debug-script-variant_20260403_0002.md` を作成
- `PCS52@FR@JP039701-H7@V2-Debug.script` を新規作成し、Debug 用の handshake / wait 実験差分を追加
- 元の `PCS52@FR@JP039701-H7@V2.script` は未変更のまま保持

## 技術的結論

現時点の結論は、**PgDebug 外に設定ファイルが散っている可能性は低く、主因は手動成功時の API フローと bench script フローの不一致である可能性が高い**、というもの。

特に重要だった点:

- 手動 API ログ `対向機.txt` には `gvif_setHandShakeGPIO` がある
- bench 実行対象の FR/H7 script には、調査時点で `IMAGE_HANDSHAKE_SET(...)` 呼び出しが無かった
- `gvif_setParams(...)`、`OUTPUT_IMAGE_SET(...)`、`IMAGE_OUTPUT(...)` は bench script 側にもあるため、**handshake と timing の差**が有力
- 設定値の実体はほぼ `PgDebug/Falcon/Script/PCS52/PCS52@01@V2.tii` と `IMG/` に集約されており、PgDebug 外で見つかったのは API header 類が中心
- もしまだ見えていない差分があるとしても、テキスト設定ではなく DLL / EXE 内部実装側である可能性が高い

ユーザーには、「隠れた設定ファイルより、bench 側が手動成功 API 手順を完全再現できていないのが濃厚」と共有済み。

## 成果物

### 調査・計画

- `/home/fpxszk/.copilot/session-state/fc19ed7c-4ba0-486a-87a9-12e4d37a7e6c/checkpoints/001-falcon-readme-tpvm1.md`
  - README 作業〜TPVM1 調査中盤までの詳細チェックポイント
- `/home/fpxszk/code/twitter-auto-poster/docs/exec-plans/active/falconscript-readme-overview_20260402_1223.md`
  - Falcon README 作業 plan
- `/home/fpxszk/code/twitter-auto-poster/docs/exec-plans/active/tpvm1-bench-ng-investigation_20260402_1301.md`
  - TPVM1 調査 plan
- `/home/fpxszk/code/twitter-auto-poster/docs/exec-plans/active/falcon-h7-debug-script-variant_20260403_0002.md`
  - Debug 派生 script 作成 plan

### 実成果物

- `/home/fpxszk/code/twitter-auto-poster/Falcon_script/README.md`
  - 現行構成ベースの概要 README
- `/home/fpxszk/code/twitter-auto-poster/Falcon_script/PgDebug/Falcon/Script/PCS52/FR/PCS52@FR@JP039701-H7@V2-Debug.script`
  - Debug 専用の派生 script

### Debug script の追加内容

- `IMAGE_HANDSHAKE_SET("GVIF1-4", 30);` を `gvif_setParams(...)` 後、`OUTPUT_IMAGE_SET(...)` 群の前に追加
- `OUTPUT_IMAGE_SET(...)` 6 箇所の前後に `WAIT(1000);`
- `IMAGE_OUTPUT(...)` 6 箇所の前後に `WAIT(1000);`
- `ETH_BASE_SEND("./" CMD_B_TPVM1_mipi);` と `ETH_MID_SEND("./" CMD_M_TPVM1_mipi);` の前後に `WAIT(3000);`
- 追加箇所には `// DEBUG:` コメントを付与

## 未解決事項 / 再開時の注意点

- `IMAGE_HANDSHAKE_SET("GVIF1-4", 30)` だけで十分かは未確定
- TPVM1 に対して本当に必要なのが handshake なのか、timing なのか、両方なのかはまだ実機/bench 側で未検証
- `GVIF_INIT("GVIF1-1", 1)` と手動時 API の `initialize / selfCheck / handshake / output start` の差が完全に解けたわけではない
- ベンチ時の API 実ログが `対向機.txt` のように残っていないため、API レベルの完全比較はまだできていない
- `Falcon_script/` は repo ルート `.gitignore` 対象なので、通常の `git status` では差分確認しづらい

## 次のステップ

1. `PCS52@FR@JP039701-H7@V2-Debug.script` を bench / debug 環境で実行し、TPVM1 の NG 挙動が変わるか確認する
2. 変化が出たら、どの変更が効いたかを切り分ける
   - handshake が効いたのか
   - image set / output 周辺の 1 秒 wait が効いたのか
   - TPVM1 send 前後の 3 秒 wait が効いたのか
3. 変化が無ければ、bench 側で実際に叩かれている API 実行経路と DLL / EXE 側の差を追う
4. 効果が確認できたら、本番 script に寄せるか、追加調査を続行するかを決める

## 関連リファレンス

- Checkpoint: `001-falcon-readme-tpvm1.md`
  - `/home/fpxszk/.copilot/session-state/fc19ed7c-4ba0-486a-87a9-12e4d37a7e6c/checkpoints/001-falcon-readme-tpvm1.md`
- Exec plan: `falcon-h7-debug-script-variant_20260403_0002.md`
  - `/home/fpxszk/code/twitter-auto-poster/docs/exec-plans/active/falcon-h7-debug-script-variant_20260403_0002.md`
- Debug script:
  - `/home/fpxszk/code/twitter-auto-poster/Falcon_script/PgDebug/Falcon/Script/PCS52/FR/PCS52@FR@JP039701-H7@V2-Debug.script`
- 元 script:
  - `/home/fpxszk/code/twitter-auto-poster/Falcon_script/PgDebug/Falcon/Script/PCS52/FR/PCS52@FR@JP039701-H7@V2.script`
