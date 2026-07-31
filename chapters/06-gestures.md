# 第6章：ジェスチャー操作

> 執筆者：（氏名）
> 最終更新：YYYY-MM-DD

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、ユーザーの指の動きを検出するジェスチャー認識の方法を学ぶ。タップ・ロングプレス・ドラッグ・拡大縮小・回転など、各ジェスチャーの実装方法を学び、最終的にTinder風のスワイプUIで複数のジェスチャーを組み合わせた実装を題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// 詳細解説の各コードを順に組み合わせたものが模範コードです。
```

**このアプリは何をするものか：**

（アプリの動作を自分の言葉で説明する。スクリーンショットを貼ってもよい。）

## コードの詳細解説

### 基本ジェスチャー（タップ、ロングプレス）

```swift
RoundedRectangle(cornerRadius: 16)
    .fill(backgroundColor)
    .frame(width: 200, height: 200)
    .onTapGesture {
        tapCount += 1
        backgroundColor = Color(hue: Double.random(in: 0...1), saturation: 0.7, brightness: 0.9)
    }

Circle()
    .fill(isPressed ? .green : .orange)
    .frame(width: 120, height: 120)
    .onLongPressGesture(minimumDuration: 1.0) {
        isPressed = true
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { isPressed = false }
    }
```

**何をしているか：**
（この部分が果たしている役割を説明する）

**なぜこう書くのか：**
（別の書き方ではなく、この書き方が選ばれている理由を説明する）

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### ドラッグジェスチャーとオフセット管理

```swift
@State private var offset: CGSize = .zero
@State private var lastOffset: CGSize = .zero

.offset(offset)
.gesture(
    DragGesture()
        .onChanged { value in
            offset = CGSize(
                width: lastOffset.width + value.translation.width,
                height: lastOffset.height + value.translation.height
            )
        }
        .onEnded { _ in lastOffset = offset }
)
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### 拡大縮小と回転

```swift
.scaleEffect(scale)
.gesture(
    MagnifyGesture()
        .onChanged { value in scale = lastScale * value.magnification }
        .onEnded { _ in lastScale = scale }
)

.rotationEffect(angle)
.gesture(
    RotateGesture()
        .onChanged { value in angle = lastAngle + value.rotation }
        .onEnded { _ in lastAngle = angle }
)
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### ジェスチャーの組み合わせとアニメーション

```swift
.gesture(
    DragGesture()
        .onChanged { value in
            offset = CGSize(
                width: lastOffset.width + value.translation.width,
                height: lastOffset.height + value.translation.height
            )
        }
        .onEnded { _ in lastOffset = offset }
)
.simultaneousGesture(
    MagnifyGesture()
        .onChanged { value in scale = lastScale * value.magnification }
        .onEnded { _ in lastScale = scale }
)
.simultaneousGesture(
    RotateGesture()
        .onChanged { value in angle = lastAngle + value.rotation }
        .onEnded { _ in lastAngle = angle }
)

Button("リセット") {
    withAnimation(.spring) {
        offset = .zero; lastOffset = .zero
        scale = 1.0; lastScale = 1.0
        angle = .zero; lastAngle = .zero
    }
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`DragGesture` | ドラッグジェスチャーを認識するジェスチャーレコグナイザー | `.gesture(DragGesture().onChanged { ... })` |
| 例：`MagnificationGesture` | ピンチジェスチャーで拡大・縮小を認識 | `.gesture(MagnificationGesture().onChanged { scale in ... })` |
| | | |
| | | |
| | | |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 結果：
- わかったこと：

**実験2：**
- やったこと：
- 結果：
- わかったこと：

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
