# VJ FX Maker — 実装メモ / フェーズ計画

## ゴール
ブラウザ単体で動作する **AE 風 UI + タイムライン + キーフレーム** を持つ VJ 用エフェクトメーカー。
`index.html` 単一ファイル、外部 CDN 無し、HTML/CSS/JS のみで完結。

---

## フェーズ計画 (確定スコープ)
フェーズ完了ごとcommentしてcommitしpushして
> 進捗ステータス: ⏳=未着手 / 🚧=実装中 / ✅=完了

### Phase 0 — 基盤再構成 (UI レイアウト + ジャンル分け + パネルリサイズ)  ✅
- [x] CSS を AE 風 5 ペイン (Top / Left / Center / Right / Bottom) に再構築
- [x] 全 51 効果に `cat` (カテゴリ) 付与: Generate / Transform / Color / Light / Blur / Distort / Glitch / Time
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

### Phase 2 — タイムライン基盤  ✅
- [x] コンポジション: duration (4〜60秒), timeMode (sec | beat)
- [x] プレイヘッド (横線), ドラッグでスクラブ
- [x] 再生/停止/巻き戻し/ループ ON-OFF ボタン
- [x] 現在時刻表示 (timecode)
- [x] **E2**: ショートカット (Space=再生, ←→=±1f, Home=先頭, End=末尾, I/O=In/Out 設定)
- [x] In/Out ポイント (デフォルト 0〜duration)
- [x] BPM グリッドスナップ (1拍/半拍/8分音符単位選択)
- [x] スクラブ中もリアルタイム反映
- [x] 時間軸の目盛り描画 (秒表示 / ビート表示)
- [x] 各エフェクトレイヤーをクリップ化し、開始/終了範囲をタイムライン上でドラッグ編集

### Phase 3 — キーフレーム  ✅
- [x] 各レイヤを展開 → パラメータ毎にキーフレーム行
- [x] ◆追加 ボタン (現在時刻に現在値で打つ)
- [x] キーをクリックして選択 → ドラッグで時刻移動
- [x] 削除 (Delete キー)
- [x] 線形補間 (デフォルト)
- [x] レンダ時はキーフレーム評価値を `eff.values` に上書き
- [x] **E4**: キーフレームコピペ (Ctrl+C/V) — 現在選択キー → 現在時刻に貼付

### Phase 4 — 録画範囲 & MP4 出力  ✅
- [x] In/Out 区間だけ録画 (再生位置を In に飛ばし→ Out に到達で自動停止)
- [x] **A1**: `MediaRecorder` で `video/mp4;codecs=avc1` 優先 → WebM フォールバック
- [x] 録画中の状態表示・進捗バー
- [x] 拡張子は実際の出力形式に合わせて `.mp4` or `.webm`

### Phase 5 — プリセット拡張  ✅
- [x] **E3**: プリセット JSON にキーフレーム、クリップ範囲、composition / scenes / FFT トリガー設定 / MIDI バインドを含める
- [x] バージョン番号 v3 で旧版 v1 / v2 も読込可
- [x] ランダム生成: キーフレームもランダム化 (各レイヤ 2〜4個ランダム時刻に打つ)

### Phase 6 — Undo/Redo  ✅
- [x] **E1**: コマンドパターン (操作ごとに undo/redo 関数を持つコマンドを履歴に積む)
- [x] 対象操作: レイヤ追加/削除/移動/有効化、クリップ範囲変更、パラメータ変更、キーフレーム追加/削除/移動
- [x] Ctrl+Z / Ctrl+Y (Ctrl+Shift+Z) で巻き戻し/やり直し
- [x] スクラブやスライダー連続入力は debounce してまとめる

### Phase 7 — A/B シーン + クロスフェーダ  ✅
- [x] **E5**: scenes = [A, B] 2スロット、each に独立した layers
- [x] アクティブシーン (編集対象) 切替ボタン (A | B)
- [x] クロスフェーダ (上ツールバーのスライダ): 0=A / 1=B / 中間=blend
- [x] レンダ: 必要な側のみ描画 (両端なら片側スキップ)、最終ステージで blend
- [x] プリセットに両シーン保存

### Phase 7.5 — FFT トリガー + パーティクル  ✅
- [x] `AnalyserNode.getByteFrequencyData()` による指定周波数帯レベル取得
- [x] FFT Hz / width / threshold / edge (rise | fall | both) の UI
- [x] 短い trigger envelope を生成し、各レイヤー共通 `fftTrig` で amount を変調
- [x] Generate カテゴリに `particles` を追加
- [x] density / size / spread / glow / spin / tunnel / streak / style / color A/B を持つパーティクルシェーダ

### Phase 8 — MIDI 入力  ✅
- [x] **E6**: Web MIDI API で MIDIAccess 取得
- [x] パネルから MIDI Learn モード ON
- [x] パラメータ右の「M」ボタン → 次に来た CC/Note を bind
- [x] CC: 0–127 を min..max にマップしてパラメータ更新
- [x] Note On: enabled トグルや「キーフレーム追加」のトリガに割当可
- [x] バインド一覧パネル + 解除

### Phase 9 — オーディオ波形をタイムライン表示  ✅
- [x] **E7**: 音声ファイル読込時に AudioContext.decodeAudioData で peaks 抽出
- [x] タイムライン背景に波形を描画
- [x] BPM bar との位置合わせ (オーディオ start = composition 0 と仮定)

### Phase 10 — グラフエディタ  ✅
- [x] **E8**: パラメータ行にグラフモード切替 (G)
- [x] 値の時系列を曲線として描画 (canvas overlay)
- [x] キーフレーム点をドラッグ (時刻 + 値)
- [x] 補間: 線形 (Ease/Hold は今回スコープ外、後日)

### Phase 11 — AE風ワークスペース + 上部バー/右クリック操作  ✅
- [x] After Effects 風の Project / Composition / Info / Audio / Preview / Effects & Presets / Effect Controls / Timeline 構成へ整理
- [x] 2 段トップバーに素材/音声読込、プリセット、録画、MIDI、再生、A/B、XFade、エフェクト追加、Particles、音声入力、表示切替を集約
- [x] 右クリックメニューを追加し、プレビュー/タイムライン/レイヤー上で再生、In/Out、素材読込、エフェクト追加、レイヤー複製/削除、録画、PNG、表示切替を実行可能
- [x] 右サイドパネルに Info / Audio / Preview / Effects & Presets の簡易操作面を追加し、Effect Controls と併用できる構成に変更

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
  bpm, beatStart, beatPhase,             // BPM
  audio: {
    level, gain,
    freq, freqWidth, freqThreshold, freqEdge,
    freqLevel, freqGate, triggerEnv
  },
  resScale,
  comp: { duration, timeMode, inPoint, outPoint, loop, snap },
  play: { playing, time, startWall },
  scenes: [
    { layers: [ {lid, effId, enabled, start, end, values, keyframes: {paramKey: [{t,v},...]}} ] },
    { layers: [...] }
  ],
  activeScene: 0,
  crossfader: 0,
  selectedLayer: null,                   // lid (currently editing)
  selectedKey: null,                     // {layer, paramKey, idx}
  midi: { access, learnTarget, bindings: [{id, type, channel, num, kind, lid, paramKey}] },
  graphOpen: {"lid:param": true},
  history: { past, future },
  panels: { leftW, rightW, bottomH },    // resize state
  rec: {...}
}
```

### キーフレーム評価
レンダ前に毎フレーム実行:
```
for each layer:
  if layer.enabled && layer.start <= state.play.time <= layer.end:
    for each paramKey in keyframes:
      layer.values[paramKey] = sample(keyframes[paramKey], state.play.time)
```
`sample()` は配列から現時刻の前後キーを二分探索し線形補間。

### FFT トリガー評価
`pollAudio()` 内で毎フレーム実行:
```
analyser.getByteFrequencyData(buf)
binHz = sampleRate / 2 / buf.length
freqLevel = average(buf bins around freq ± width/2) * gain
gate = freqLevel >= threshold
triggerEnv = edge(rise|fall|both) ? 1 : exponentialDecay(triggerEnv)
```
各レイヤーの `fftTrig` は `effectiveAmount()` で `triggerEnv` を amount に加算する。

### MIDI バインド
```
binding = {
  id,
  type: 'cc' | 'note',
  channel,
  num,
  kind: 'param' | 'enabled' | 'key',
  lid,
  paramKey
}
```
`param` は CC/Note velocity をパラメータ範囲へ線形マップ。`enabled` は Note でトグル、CC では 64 以上を ON として扱う。`key` は Note/CC 入力で `amount` キーフレームを現在時刻へ追加。

### 波形表示
音声ファイル読み込み時に `decodeAudioData()` で PCM を取得し、固定数のピーク配列へ縮約。ルーラー背景の canvas に composition 0 起点で描画する。音声ファイル自体はプリセットには保存しない。

### グラフエディタ
数値パラメータ行の `G` ボタンで graph mode を切替。キー点の x は時刻、y は `min..max` の正規化値。ドラッグ中は canvas 曲線と点を即時更新し、pointerup で履歴化する。

### UI 操作サーフェス
上部バーは既存のボタン/入力を呼び出す統合ショートカットとして扱う。`setupTopToolbar()` で既存 DOM や操作関数へ接続し、右クリックメニューは `contextTargetFromEvent()` で preview / timeline / layer / general を判定して表示項目を切り替える。

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

## 効果カテゴリ (51)
| カテゴリ | 個数 | 効果 ID |
|---|---|---|
| Generate  | 1  | particles |
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
- FFT トリガーは指定帯域の平均レベルによる判定。キック抽出などは素材に応じて Hz / width / threshold / gain の調整が必要。
- Particles は 1 パスの手続き的シェーダ。density / glow / resolution が高いほど GPU 負荷が上がる。
- Web MIDI API はブラウザ/OS/権限に依存。
- 波形表示は読み込んだ音声ファイルのピーク解析のみ。動画素材内の音声波形は未対応。
- グラフエディタは数値パラメータのみ対応。色パラメータは通常のキー点編集。
- グラフエディタは線形補間のみ (ベジェ/Ease は将来拡張)。
