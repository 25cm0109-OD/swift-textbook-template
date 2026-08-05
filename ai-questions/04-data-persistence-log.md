# AI質問ログ：第4章 データの永続化

## 使用した生成AIツール

ChatGPT

## 質問と回答の記録

### Q1

**質問：**

`ContentView` の左上に歯車ボタン、右上に追加ボタンを表示するには、どこにコードを追加すればよい？

**AIの回答の要点：**

`NavigationStack` の中にある `.toolbar { }` へ `ToolbarItem` を追加する。`placement: .topBarLeading` は左上、`placement: .topBarTrailing` は右上に配置する指定である。

```swift
.toolbar {
    ToolbarItem(placement: .topBarLeading) {
        Button {
            isShowingSettings = true
        } label: {
            Image(systemName: "gear")
        }
    }

    ToolbarItem(placement: .topBarTrailing) {
        Button {
            isShowingAddSheet = true
        } label: {
            Image(systemName: "plus")
        }
    }
}
```

歯車ボタンで設定画面を表示するため、`ContentView` のプロパティとして次も必要になる。

```swift
@State private var isShowingSettings = false
```

**自分の理解：**

ナビゲーションバーに置くボタンは `.toolbar` の中に書く。ボタンを押したときに設定画面を開く状態は、`@State` で最初に用意しておく必要があると理解した。

### Q2

**質問：**

`@Model`って何？

**AIの回答の要点：**

`@Model` は、SwiftData に保存したいデータのクラスに付ける印である。

```swift
@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool
}
```

`class Memo` だけなら通常の Swift のクラスだが、`@Model` を付けると SwiftData が保存・読み込み・編集・削除の対象として扱える。`Memo` はメモ帳の1件分のデータを表す。

```swift
modelContext.insert(memo)       // 保存
@Query private var memos: [Memo] // 読み込み
modelContext.delete(memo)       // 削除
```

**自分の理解：**

`@Model` は「このクラスのデータを保存したい」と SwiftData に伝えるためのものだと理解した。`Memo` のインスタンス1個が、メモ帳のメモ1件になる。

### Q3

**質問：**

SwiftDataって何？

**AIの回答の要点：**

SwiftData は、アプリのデータを端末内に保存し、アプリを閉じた後も残せる Apple 公式の仕組みである。メモアプリでは、タイトル・本文・作成日時・お気に入り状態などを保存できる。

```swift
let memo = Memo(title: "買い物", content: "牛乳を買う")
modelContext.insert(memo)
```

保存済みのメモは `@Query` で取得し、不要になったメモは `modelContext.delete(memo)` で削除する。`AppStorage` は名前やオン／オフ設定のような単純な値向けで、SwiftData はメモ・タスク・投稿のような複数件のデータを扱う場合に向いている。

**自分の理解：**

SwiftData はアプリ内のデータベースのような仕組みで、メモをアプリ終了後も残すために使う。`@Model` で保存するデータの形を決め、`modelContext` と `@Query` で操作する。

（質問は何個でも追加してください。多ければ多いほど良いです。）

## 今日の質問を振り返って

画面に部品を追加する場所と、SwiftData の基本用語を分けて質問したことで、役割の違いを理解できた。特に、`@Model` は保存するデータの定義、`modelContext` は保存や削除の操作、`@Query` は保存済みデータの取得に使うと整理できた。

次回は、歯車ボタンを押した後に設定画面を `.sheet` で表示する方法や、SwiftData をアプリ全体で使えるように設定する `modelContainer` について質問したい。
