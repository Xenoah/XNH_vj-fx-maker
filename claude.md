# VJ FX Maker — 実装メモ / フェーズ計画

## ゴール
ブラウザ単体で動作する **AE 風 UI + タイムライン + キーフレーム** を持つ VJ 用エフェクトメーカー。
`index.html` 単一ファイル、外部 CDN 無し、HTML/CSS/JS のみで完結。

---

## フェーズ計画 (確定スコープ)

> 進捗ステータス: ⏳=未着手 / 🚧=実装中 / ✅=完了

### Phase 0 — 基盤再構成 (UI レイアウト + ジャンル分け + パネルリサイズ)  ✅
- [x] CSS を AE 風 5 ペイン (Top / Left / Center / Right / Bottom) に再構築
- [x] 全 50 効果に `cat` (カテゴリ) 付与: Transform / Color / Light / Blur / Distort / Glitch / Time
- [x] 左パネルにカテゴリ別折畳式エフェクトライブラリ
- [x] 右パネルは「選択中レイヤのコントロール」専用に
- [x] 下パネルにタイムライン枠 (中身は Phase 2)
- [x] 上ツールバーに BPM/録画/全画面/プリセット集約
- [x] **E10**: 左/右/下パネルの境界をドラッグでリサイズ

### Phase 1 — レイヤースタック & 効果選択  ✅
- [x] エフェクトを「レイヤー」概念に変更 (同じ効果を複数追加可能)
- [x] ライブラリの効果クリックで「+ 追加」ボタン → レイヤー追加
- [x] レイヤー一覧 (タイムラインの縦軸) で選択状態管理
- [x] 右ペインに選択レイヤのパラメータ詳細表示
- [x] レイヤー削除/複製/並び替え/有効化トグル
- [x] **E9**: 効果の複製 (同じレイヤをコピー)

### Phase 2 — タイムライン基盤  ⏳
- [ ] コンポジション: duration (4〜60秒), timeMode (sec | beat)
- [ ] プレイヘッド (横線), ドラッグでスクラブ
- [ ] 再生/停止/巻き戻し/ループ ON-OFF ボタン
- [ ] 現在時刻表示 (timecode)
- [ ] **E2**: ショートカット (Space=再生, ←→=±1f, Home=先頭, End=末尾, I/O=In/Out 設定)
- [ ] In/Out ポイント (デフォルト 0〜duration)
- [ ] BPM グリッドスナップ (1拍/半拍/8分音符単位選択)
- [ ] スクラブ中もリアルタイム反映
- [ ] 時間軸の目盛り描画 (秒表示 / ビート表示)

### Phase 3 — キーフレーム  ⏳
- [ ] 各レイヤを展開 → パラメータ毎にキーフレーム行
- [ ] ◆追加 ボタン (現在時刻に現在値で打つ)
- [ ] キーをクリックして選択 → ドラッグで時刻移動
- [ ] 削除 (Delete キー)
- [ ] 線形補間 (デフォルト)
- [ ] レンダ時はキーフレーム評価値を `eff.values` に上書き
- [ ] **E4**: キーフレームコピペ (Ctrl+C/V) — 現在選択キー → 現在時刻に貼付

### Phase 4 — 録画範囲 & MP4 出力  ⏳
- [ ] In/Out 区間だけ録画 (再生位置を In に飛ばし→ Out に到達で自動停止)
- [ ] **A1**: `MediaRecorder` で `video/mp4;codecs=avc1` 優先 → WebM フォールバック
- [ ] 録画中の状態表示・進捗バー
- [ ] 拡張子は実際の出力形式に合わせて `.mp4` or `.webm`

### Phase 5 — プリセット拡張  ⏳
- [ ] **E3**: プリセット JSON にキーフレームと composition / scenes 設定を含める
- [ ] バージョン番号 v2 で旧版 v1 (キーフレ無し) も読込可
- [ ] ランダム生成: キーフレームもランダム化 (各レイヤ 2〜4個ランダム時刻に打つ)

### Phase 6 — Undo/Redo  ⏳
- [ ] **E1**: コマンドパターン (操作ごとに undo/redo 関数を持つコマンドを履歴に積む)
- [ ] 対象操作: レイヤ追加/削除/移動/有効化、パラメータ変更、キーフレーム追加/削除/移動
- [ ] Ctrl+Z / Ctrl+Y (Ctrl+Shift+Z) で巻き戻し/やり直し
- [ ] スクラブやスライダー連続入力は debounce してまとめる

### Phase 7 — A/B シーン + クロスフェーダ  ⏳
- [ ] **E5**: scenes = [A, B] 2スロット、each に独立した layers
- [ ] アクティブシーン (編集対象) 切替ボタン (A | B)
- [ ] クロスフェーダ (上ツールバーの大きいスライダ): 0=A / 1=B / 中間=blend
- [ ] レンダ: 必要な側のみ描画 (両端なら片側スキップ)、最終ステージで blend
- [ ] プリセットに両シーン保存

### Phase 8 — MIDI 入力  ⏳
- [ ] **E6**: Web MIDI API で MIDIAccess 取得
- [ ] パネルから MIDI Learn モード ON
- [ ] パラメータ右の「⚡」ボタン → 次に来た CC/Note を bind
- [ ] CC: 0–127 を min..max にマップしてパラメータ更新
- [ ] Note On: enabled トグルや「キーフレーム追加」のトリガに割当可
- [ ] バインド一覧パネル + 解除

### Phase 9 — オーディオ波形をタイムライン表示  ⏳
- [ ] **E7**: 音声ファイル読込時に AudioContext.decodeAudioData で peaks 抽出
- [ ] タイムライン背景に波形 (グレー) を描画
- [ ] BPM bar との位置合わせ (オーディオ start = composition 0 と仮定)

### Phase 10 — グラフエディタ  ⏳
- [ ] **E8**: パラメータ行にグラフモード切替 (▼)
- [ ] 値の時系列を曲線として描画 (canvas overlay)
- [ ] キーフレーム点をドラッグ (時刻 + 値)
- [ ] 補間: 線形 (Ease/Hold は今回スコープ外、後日)

---

## アーキテクチャ詳細

### レンダリングパイプライン
```
[ソース(image|video|demo)]
       │
       ▼ texImage2D(srcTex)
    [srcTex]
       │       ┌─── シーンA layers ───┐
       ├──────▶│ FBO ping ↔ pong       │──▶ outA
       │       └───────────────────────┘
       │       ┌─── シーンB layers ───┐
       └──────▶│ FBO ping ↔ pong       │──▶ outB
                                              │
                            ┌─── crossfade ───┤
                            ▼                 ▼
                         canvas ← [outA, outB, mix]
                            │
                            └──▶ prevFBO 保存 (trail/feedback 用)
```

### 状態モデル (新)
```js
state = {
  source: {...},                         // 素材
  bpm, beatStart, beatPhase, audio,      // BPM/音声
  resScale,
  comp: { duration, timeMode, inPoint, outPoint, loop, snap },
  play: { playing, time, startWall },
  scenes: [
    { layers: [ {lid, effId, enabled, values, keyframes: {paramKey: [{t,v},...]}} ] },
    { layers: [...] }
  ],
  activeScene: 0,
  crossfader: 0,
  selectedLayer: null,                   // lid (currently editing)
  selectedKey: null,                     // {layer, paramKey, idx}
  midi: { access, learnTarget, bindings: [{deviceIdx, type, num, layer, paramKey, range}] },
  history: { past, future },
  panels: { leftW, rightW, bottomH },    // resize state
  rec: {...}
}
```

### キーフレーム評価
レンダ前に毎フレーム実行:
```
for each layer:
  for each paramKey in keyframes:
    layer.values[paramKey] = sample(keyframes[paramKey], state.play.time)
```
`sample()` は配列から現時刻の前後キーを二分探索し線形補間。

### Undo/Redo コマンド形式
```js
{ do: () => {...}, undo: () => {...}, label: '...' }
```
連続操作 (スライダ ドラッグ) は同種なら 200ms 以内のもの 1 コマンドに合体。

### MP4 出力
```js
const mimes = [
  'video/mp4;codecs=avc1.42E01E',
  'video/mp4;codecs=avc1',
  'video/webm;codecs=vp9',
  'video/webm;codecs=vp8',
  'video/webm'
];
const supported = mimes.find(m => MediaRecorder.isTypeSupported(m));
```

---

## 効果カテゴリ (50)
| カテゴリ | 個数 | 効果 ID |
|---|---|---|
| Transform | 10 | zoom, pan, crop, rotate, mirror, tile, kaleido, split, pip, wrapscroll |
| Color     | 10 | hue, sat, contrast, bright, gamma, invert, mono, gradmap, poster, lut |
| Light     | 8  | glow, edgeglow, bloom, flare, leak, vignette, strobe, flash |
| Blur      | 4  | gblur, mblur, rblur, zblur |
| Distort   | 6  | wave, lensd, swirl, noisedisp, liquify, shake |
| Glitch    | 9  | rgbshift, chroma, glitch, slice, block, pixel, scan, grain, vhs |
| Time      | 3  | trail, feedback, audiobpm |

---

## ファイル
- `index.html` — 単一ファイル本体
- `README.md` — 使い方
- `claude.md` — 本ファイル (実装メモ + フェーズ進捗)

---

## 既知の制限 (継続)
- WebGL1 想定、blur/glow は単一パスでの近似。
- MP4 出力はブラウザ依存 (Chrome/Edge 130+ で `video/mp4`、それ以外 webm フォールバック)。
- 動画素材自身の音声は AudioContext に取り込まず、別途音声ファイル/マイクで Audio Reactive。
- グラフエディタは線形補間のみ (ベジェ/Ease は将来拡張)。
