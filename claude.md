# VJ FX Maker — 実装メモ

## ゴール
ブラウザ単体で動作する VJ 素材用エフェクトメーカー。`index.html` 単一ファイル、外部CDN無し、HTML/CSS/JS のみで完結。

## アーキテクチャ

### レンダリングパイプライン
```
[ソース(image|video|demo)]
       │
       ▼ texImage2D(srcTex)
    [srcTex]
       │
       ▼ enabled なエフェクトを順番にチェーン
[FBO ping]→[FBO pong]→…  (u_tex=直前出力, u_prev=前フレ最終出力)
       │
       ▼ 最終出力を画面に blit、prevFBO に保存
    [canvas]
```

- **ping-pong FBO**: `fboA` ↔ `fboB` を毎パスで swap。
- **prevFBO**: 1フレーム前の最終出力を保持。`trail` / `feedback` 効果はここから読む。
- **解像度スケール**: `resScale` スライダ (0.25–1.0) で内部解像度を縮小、軽量化。

### 共通フラグメントヘッダ
全効果のシェーダに以下を prepend:
```
precision highp float;
varying vec2 v_uv;
uniform sampler2D u_tex, u_prev;
uniform vec2 u_res;
uniform float u_time, u_beat, u_audio;
uniform float u_amount, u_speed, u_mix;
uniform float u_p1, u_p2, u_p3, u_p4;
uniform vec3 u_colorA, u_colorB;
// helpers: rgb2hsv, hsv2rgb, rand, noise, sampleSrc(clamp版)
void main(){ ... 各効果の本体 ... }
```

### 効果定義
50件を `EFFECTS` 配列で宣言:
```js
{ id, name, st: 'full'|'lite'|'mon', extras: [...], frag: '...' }
```
- `extras` は追加スライダ/カラー定義。最大4つが順に `u_p1..u_p4` にマップされる。
- `colorA/colorB` は HEX で持ち、レンダ時に vec3 0–1 へ変換。

### BPM / Audio 変調
全効果が共通で持つ `bpmSync` / `audioReactive` パラメータ。CPU側で:
```
amt = clamp(amount + bpmSync*0.5*sin(2π*beat) + audioReactive*0.5*level, 0, 1)
```
を計算し `u_amount` として渡す。`strobe`/`flash` などビート同期型は `u_beat` 自体を直接使う。

### Audio
- マイク: `getUserMedia({audio:true})` → AnalyserNode (destination 接続せず)
- ファイル: `<audio>` + MediaElementSource → AnalyserNode → AudioContext.destination
- `getByteFrequencyData()` の平均 → `audioLevel` (0..1)、感度スライダで線形ゲイン

### BPM
- `bpmInput` で BPM 設定
- `tap` ボタンで複数回タップから自動算出 (直近4回の平均)
- `beatPhase = ((now-start)/1000 * BPM/60) mod 1` を毎フレーム計算

### 録画
- `canvas.captureStream(60)` → `MediaRecorder({mimeType:'video/webm;codecs=vp9'})`
- フォールバック: vp8 → 既定。停止時に Blob を生成、a タグで自動DL

### プリセット
- 状態を JSON エクスポート/インポート。
- ランダム生成: 3-7個の効果をランダムに enable し、各パラメータをランダム化。

## 効果ステータス
- **full** (実装): 緑バッジ。WebGL シェーダが期待通り動作。
- **lite** (簡易): 黄バッジ。簡略化された見た目で動作。
- **mon** (監視): 青バッジ。レンダパス無し、状態表示用。

## 既知の制限
- WebGL1 想定 (互換性優先)。
- 各 blur/glow は単一パス内で複数タップしている (簡易実装)。本格的にやるなら separable + downsample。
- `lens-flare`, `light-leak`, `vhs-noise`, `liquify-warp` は手続き的な簡易表現。
- 動画の音声は別途 audio file 読み込みでないと取得できない (HTMLVideoElement の音声を AudioContext に取れない実装にはしていない、簡易のため)。
- シェーダコンパイルは効果有効化時の遅延コンパイル。初回 enable 時に若干フレーム落ちあり。

## ファイル構成
- `index.html` — 単一ファイル本体
- `README.md` — 使い方
- `claude.md` — 本ファイル (実装メモ)

## 進捗
- [x] パイプライン基盤 (WebGL + ping-pong)
- [x] 50効果のUIスロット
- [x] 25必須効果のシェーダ実装
- [x] 残り25効果も簡易実装で動作
- [x] BPM tap / 同期
- [x] Audio reactive (mic + file)
- [x] プリセット保存/読込/ランダム
- [x] MediaRecorder 録画
- [x] フルスクリーン
- [x] 解像度スケール
- [x] デモパターン自動表示
