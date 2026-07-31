# 第4章：データの永続化

> 執筆者：（氏名）
> 最終更新：YYYY-MM-DD

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、AppStorageとSwiftDataを使ってアプリのデータを端末に永続的に保存する方法を学ぶ。具体的にはSwiftDataを使ったメモアプリを題材として、@Modelでデータモデルを定義し、modelContextを使ったデータ操作、@Queryによる動的なデータ取得、そして@AppStorageによるユーザー設定の保存を実装する。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第4章：データの永続化（AppStorage + SwiftData）
// ============================================

import SwiftUI
import SwiftData

@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool

    init(title: String, content: String, createdAt: Date = .now, isFavorite: Bool = false) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}

struct ContentView: View {
    @Environment(\.modelContext) private var modelContext
    @Query(sort: \Memo.createdAt, order: .reverse) private var memos: [Memo]
    @AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
    @AppStorage("userName") private var userName: String = ""
    @State private var isShowingAddSheet = false
    @State private var isShowingSettings = false

    var displayedMemos: [Memo] {
        if sortByFavorite {
            return memos.sorted { $0.isFavorite && !$1.isFavorite }
        }
        return memos
    }

    var body: some View {
        NavigationStack {
            Group {
                if memos.isEmpty {
                    ContentUnavailableView(
                        "メモがありません",
                        systemImage: "note.text",
                        description: Text("右上の＋ボタンからメモを追加してください")
                    )
                } else {
                    List {
                        ForEach(displayedMemos) { memo in
                            NavigationLink(destination: MemoEditView(memo: memo)) {
                                MemoRow(memo: memo)
                            }
                        }
                        .onDelete(perform: deleteMemos)
                    }
                }
            }
            .navigationTitle(userName.isEmpty ? "メモ帳" : "\(userName)のメモ帳")
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    Button { isShowingSettings = true } label: {
                        Image(systemName: "gear")
                    }
                }
                ToolbarItem(placement: .topBarTrailing) {
                    Button { isShowingAddSheet = true } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $isShowingAddSheet) {
                MemoAddView()
            }
            .sheet(isPresented: $isShowingSettings) {
                SettingsView(userName: $userName, sortByFavorite: $sortByFavorite)
            }
        }
    }

    func deleteMemos(at offsets: IndexSet) {
        for index in offsets {
            modelContext.delete(displayedMemos[index])
        }
    }
}

struct MemoRow: View {
    let memo: Memo

    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(memo.title).font(.headline)
                Text(memo.content)
                    .font(.caption)
                    .foregroundStyle(.secondary)
                    .lineLimit(2)
                Text(memo.createdAt, style: .date)
                    .font(.caption2)
                    .foregroundStyle(.tertiary)
            }
            Spacer()
            if memo.isFavorite {
                Image(systemName: "star.fill").foregroundStyle(.yellow)
            }
        }
        .padding(.vertical, 2)
    }
}

struct MemoAddView: View {
    @Environment(\.modelContext) private var modelContext
    @Environment(\.dismiss) private var dismiss
    @State private var title = ""
    @State private var content = ""

    var body: some View {
        NavigationStack {
            Form {
                Section("タイトル") { TextField("メモのタイトル", text: $title) }
                Section("内容") { TextEditor(text: $content).frame(minHeight: 200) }
            }
            .navigationTitle("新しいメモ")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("キャンセル") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        modelContext.insert(Memo(title: title, content: content))
                        dismiss()
                    }
                    .disabled(title.isEmpty)
                }
            }
        }
    }
}

struct MemoEditView: View {
    @Bindable var memo: Memo

    var body: some View {
        Form {
            Section("タイトル") { TextField("タイトル", text: $memo.title) }
            Section("内容") { TextEditor(text: $memo.content).frame(minHeight: 200) }
            Section { Toggle("お気に入り", isOn: $memo.isFavorite) }
        }
        .navigationTitle("メモを編集")
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct SettingsView: View {
    @Binding var userName: String
    @Binding var sortByFavorite: Bool
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            Form {
                Section("ユーザー設定") {
                    TextField("あなたの名前", text: $userName)
                }
                Section("表示設定") {
                    Toggle("お気に入りを上に表示", isOn: $sortByFavorite)
                }
                Section {
                    Text("設定はアプリを閉じても保存されます")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .navigationTitle("設定")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("完了") { dismiss() }
                }
            }
        }
    }
}

#Preview {
    ContentView()
        .modelContainer(for: Memo.self, inMemory: true)
}
```

**このアプリは何をするものか：**

（アプリの動作を自分の言葉で説明する。スクリーンショットを貼ってもよい。）

## コードの詳細解説

### SwiftDataモデル（@Model）

```swift
@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool

    init(title: String, content: String, createdAt: Date = .now, isFavorite: Bool = false) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}
```

**何をしているか：**
（この部分が果たしている役割を説明する）

**なぜこう書くのか：**
（別の書き方ではなく、この書き方が選ばれている理由を説明する）

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### データの追加・削除（modelContext）

```swift
@Environment(\.modelContext) private var modelContext

func deleteMemos(at offsets: IndexSet) {
    for index in offsets {
        let memo = displayedMemos[index]
        modelContext.delete(memo)
    }
}

// MemoAddView 内
Button("保存") {
    let memo = Memo(title: title, content: content)
    modelContext.insert(memo)
    dismiss()
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### @Queryによるデータ取得

```swift
@Query(sort: \Memo.createdAt, order: .reverse) private var memos: [Memo]

List {
    ForEach(displayedMemos) { memo in
        NavigationLink(destination: MemoEditView(memo: memo)) {
            MemoRow(memo: memo)
        }
    }
    .onDelete(perform: deleteMemos)
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### @AppStorageによる設定保存

```swift
@AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
@AppStorage("userName") private var userName: String = ""

struct SettingsView: View {
    @Binding var userName: String
    @Binding var sortByFavorite: Bool

    var body: some View {
        Form {
            Section("ユーザー設定") {
                TextField("あなたの名前", text: $userName)
            }
            Section("表示設定") {
                Toggle("お気に入りを上に表示", isOn: $sortByFavorite)
            }
        }
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
| 例：`@Model` | SwiftDataでオブジェクトを永続化するためのマクロ | `@Model final class Memo { ... }` |
| 例：`@Query` | データベースからデータを取得し、変更を自動で反映するプロパティラッパー | `@Query var memos: [Memo]` |
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

# AIに作って貰った実装手順

メモアプリ実装手順
Step 1：SwiftUIプロジェクトを作る

Xcodeで新規プロジェクトを作成します。

App
Interface：SwiftUI
Language：Swift
Storage：SwiftData を選べる場合は有効にする

まずは ContentView.swift を開きます。

Step 2：必要なフレームワークを読み込む

ContentView.swift の先頭に書きます。

import SwiftUI
import SwiftData
SwiftUI：画面を作る
SwiftData：メモなどのデータを保存する
Step 3：メモのデータモデルを作る

画面より先に、「メモには何が必要か」を決めます。

今回のメモには以下があります。

タイトル
内容
作成日時
お気に入りかどうか

ContentView の上に書きます。

@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool

    init(
        title: String,
        content: String,
        createdAt: Date = .now,
        isFavorite: Bool = false
    ) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}

ここで重要なのは @Model です。

@Model
class Memo

これを付けると、SwiftDataがこのクラスを保存できるデータとして扱います。

Step 4：AppファイルにSwiftDataの保存場所を設定する

○○App.swift を開きます。

たとえば MemoApp.swift なら、こうします。

import SwiftUI
import SwiftData

@main
struct MemoApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Memo.self)
    }
}

この部分が大事です。

.modelContainer(for: Memo.self)

意味は、

Memo型のデータをアプリ内に保存できるようにする

です。

これがないと、@Query や modelContext が使えません。

Step 5：まずはメモ一覧画面だけ作る

最初は保存や追加ボタンを考えず、画面の土台だけ作ります。

struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                Text("テストメモ")
                Text("買い物リスト")
            }
            .navigationTitle("メモ帳")
        }
    }
}

ここで確認すること：

上に「メモ帳」と表示される
リスト形式で文字が表示される
NavigationStack が使えている
Step 6：保存されているメモを取得する

次に、仮の Text ではなくSwiftDataからメモを読み込みます。

struct ContentView: View {
    @Query(sort: \Memo.createdAt, order: .reverse)
    private var memos: [Memo]

    var body: some View {
        NavigationStack {
            List {
                ForEach(memos) { memo in
                    Text(memo.title)
                }
            }
            .navigationTitle("メモ帳")
        }
    }
}

ここでの重要ポイント：

@Query(sort: \Memo.createdAt, order: .reverse)
private var memos: [Memo]

これは、

保存されているMemoを、作成日時が新しい順に取得する

という意味です。

Step 7：メモが0件のときの画面を作る

メモがないと空のリストだけで寂しいので、空状態を表示します。

Group {
    if memos.isEmpty {
        ContentUnavailableView(
            "メモがありません",
            systemImage: "note.text",
            description: Text("右上の＋ボタンからメモを追加してください")
        )
    } else {
        List {
            ForEach(memos) { memo in
                Text(memo.title)
            }
        }
    }
}

これを NavigationStack の中に入れます。

Step 8：＋ボタンを追加する

追加画面を開くための状態を作ります。

@State private var isShowingAddSheet = false

次にツールバーを追加します。

.toolbar {
    ToolbarItem(placement: .topBarTrailing) {
        Button {
            isShowingAddSheet = true
        } label: {
            Image(systemName: "plus")
        }
    }
}

まだ追加画面はありませんが、タップすると true になります。

Step 9：メモ追加画面を作る

新しく MemoAddView を作ります。

struct MemoAddView: View {
    @State private var title = ""
    @State private var content = ""

    var body: some View {
        NavigationStack {
            Form {
                Section("タイトル") {
                    TextField("メモのタイトル", text: $title)
                }

                Section("内容") {
                    TextEditor(text: $content)
                        .frame(minHeight: 200)
                }
            }
            .navigationTitle("新しいメモ")
        }
    }
}

まずは入力欄が表示されることを確認します。

TextField：一行の入力欄
TextEditor：複数行の入力欄
$title：入力内容と変数をつなぐ
Step 10：追加画面をシート表示する

ContentView に戻って、sheet を追加します。

.sheet(isPresented: $isShowingAddSheet) {
    MemoAddView()
}

これで＋ボタンを押すと、追加画面が下から表示されます。

Step 11：保存処理を追加する

MemoAddView でSwiftDataを使えるようにします。

@Environment(\.modelContext) private var modelContext
@Environment(\.dismiss) private var dismiss

保存ボタンを作ります。

.toolbar {
    ToolbarItem(placement: .cancellationAction) {
        Button("キャンセル") {
            dismiss()
        }
    }

    ToolbarItem(placement: .confirmationAction) {
        Button("保存") {
            let memo = Memo(
                title: title,
                content: content
            )

            modelContext.insert(memo)
            dismiss()
        }
        .disabled(title.isEmpty)
    }
}

流れはこうです。

let memo = Memo(title: title, content: content)
modelContext.insert(memo)
dismiss()
入力内容を使って Memo を作る
SwiftDataに保存する
追加画面を閉じる

保存後は @Query が自動更新され、一覧にも表示されます。

Step 12：一覧用の行デザインを分ける

MemoRow を作ります。

struct MemoRow: View {
    let memo: Memo

    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(memo.title)
                    .font(.headline)

                Text(memo.content)
                    .font(.caption)
                    .foregroundStyle(.secondary)
                    .lineLimit(2)

                Text(memo.createdAt, style: .date)
                    .font(.caption2)
                    .foregroundStyle(.tertiary)
            }

            Spacer()

            if memo.isFavorite {
                Image(systemName: "star.fill")
                    .foregroundStyle(.yellow)
            }
        }
        .padding(.vertical, 2)
    }
}

一覧側をこう変更します。

ForEach(memos) { memo in
    MemoRow(memo: memo)
}

役割を分けると、ContentView が読みやすくなります。

Step 13：メモ編集画面を作る

一覧のメモをタップしたら編集画面に移動させます。

まず編集画面を作ります。

struct MemoEditView: View {
    @Bindable var memo: Memo

    var body: some View {
        Form {
            Section("タイトル") {
                TextField("タイトル", text: $memo.title)
            }

            Section("内容") {
                TextEditor(text: $memo.content)
                    .frame(minHeight: 200)
            }

            Section {
                Toggle("お気に入り", isOn: $memo.isFavorite)
            }
        }
        .navigationTitle("メモを編集")
        .navigationBarTitleDisplayMode(.inline)
    }
}

重要なのはこれです。

@Bindable var memo: Memo

普通の let memo だと編集できません。

@Bindable にすると、

$memo.title
$memo.content
$memo.isFavorite

のように、SwiftDataのデータを直接編集できます。

Step 14：一覧から編集画面へ遷移する

ForEach を次の形に変えます。

ForEach(memos) { memo in
    NavigationLink(destination: MemoEditView(memo: memo)) {
        MemoRow(memo: memo)
    }
}

これでメモをタップすると編集画面が開きます。

Step 15：削除機能を追加する

ContentView にSwiftData操作用のコードを追加します。

@Environment(\.modelContext) private var modelContext

次に削除関数を作ります。

func deleteMemos(at offsets: IndexSet) {
    for index in offsets {
        let memo = memos[index]
        modelContext.delete(memo)
    }
}

そして ForEach に追加します。

.onDelete(perform: deleteMemos)

完成形はこうです。

List {
    ForEach(memos) { memo in
        NavigationLink(destination: MemoEditView(memo: memo)) {
            MemoRow(memo: memo)
        }
    }
    .onDelete(perform: deleteMemos)
}

これでスワイプ削除ができます。

Step 16：AppStorageで設定を保存する

次はメモ本体ではなく、アプリ設定を保存します。

ContentView に書きます。

@AppStorage("userName") private var userName: String = ""
@AppStorage("sortByFavorite") private var sortByFavorite: Bool = false

AppStorage は、

ユーザー名
ダークモード設定
表示順設定
初回起動済みかどうか

のような、簡単な設定保存に向いています。

Step 17：ユーザー名をタイトルに反映する
.navigationTitle(
    userName.isEmpty
    ? "メモ帳"
    : "\(userName)のメモ帳"
)

これで、

名前が空 → メモ帳
名前がある → ○○のメモ帳

になります。

Step 18：お気に入り順の表示用配列を作る

memos はSwiftDataから取った元データです。

表示順だけ変えたいので、別の計算プロパティを作ります。

var displayedMemos: [Memo] {
    if sortByFavorite {
        return memos.sorted {
            $0.isFavorite && !$1.isFavorite
        }
    }

    return memos
}

一覧は memos ではなく displayedMemos を使います。

ForEach(displayedMemos) { memo in
    NavigationLink(destination: MemoEditView(memo: memo)) {
        MemoRow(memo: memo)
    }
}

削除も同じ並び順を使う必要があります。

func deleteMemos(at offsets: IndexSet) {
    for index in offsets {
        let memo = displayedMemos[index]
        modelContext.delete(memo)
    }
}
Step 19：設定画面を作る
struct SettingsView: View {
    @Binding var userName: String
    @Binding var sortByFavorite: Bool
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            Form {
                Section("ユーザー設定") {
                    TextField("あなたの名前", text: $userName)
                }

                Section("表示設定") {
                    Toggle(
                        "お気に入りを上に表示",
                        isOn: $sortByFavorite
                    )
                }

                Section {
                    Text("設定はアプリを閉じても保存されます")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .navigationTitle("設定")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("完了") {
                        dismiss()
                    }
                }
            }
        }
    }
}

ここでは @Binding を使います。

@Binding var userName: String

これは、

ContentViewが持っている値を、SettingsViewから直接変更する

という意味です。

Step 20：設定画面を開くボタンを作る

ContentView に状態を追加します。

@State private var isShowingSettings = false

ツールバーに歯車ボタンを追加します。

ToolbarItem(placement: .topBarLeading) {
    Button {
        isShowingSettings = true
    } label: {
        Image(systemName: "gear")
    }
}

シートを追加します。

.sheet(isPresented: $isShowingSettings) {
    SettingsView(
        userName: $userName,
        sortByFavorite: $sortByFavorite
    )
}
最終的な学習順

この順番で、毎回コードを見ずに書く練習をすると覚えやすいです。

Memo モデルを作る
App に .modelContainer(for: Memo.self) を付ける
ContentView に @Query を書く
List + ForEach で一覧表示する
MemoAddView を作る
modelContext.insert() で保存する
MemoEditView を作る
@Bindable で編集できるようにする
modelContext.delete() で削除する
@AppStorage で設定を保存する
SettingsView + @Binding を作る
お気に入り順の並び替えを追加する

特に覚えるべき3つはこれです。

@Query

保存済みデータを読む。

modelContext.insert(memo)

新しいデータを保存する。

@AppStorage("キー名")

アプリ設定を保存する。



# 完成コード
```swift
//
//  MyView.swift
//  Infinit_Data
//
//  Created by cmStudent on 2026/06/19.
//
import SwiftUI
import SwiftData

@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool

    init(
        title: String,
        content: String,
        createdAt: Date = .now,
        isFavorite: Bool = false
    ) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}

struct MyView: View {
    @Query(sort: \Memo.createdAt, order: .reverse)
    private var memos: [Memo]

    @State private var isShowingAddSheet = false
    @State private var isShowingSettings = false

    @Environment(\.modelContext) private var modelContext
    @AppStorage("userName") private var userName: String = ""
    @AppStorage("sortByFavorite") private var sortByFavorite: Bool = false

    var displayedMemos: [Memo] {
        if sortByFavorite {
            return memos.sorted {
                $0.isFavorite && !$1.isFavorite
            }
        }
        return memos
    }

    var body: some View {
        NavigationStack {
            Group {
                if memos.isEmpty {
                    ContentUnavailableView(
                        "メモがありません",
                        systemImage: "note.text",
                        description: Text("右上の＋ボタンからメモを追加してください")
                    )
                } else {
                    List {
                        ForEach(displayedMemos) { memo in
                            NavigationLink(destination: MemoEditView(memo: memo)) {
                                MemoRow(memo: memo)
                            }
                        }
                        .onDelete(perform: deleteMemos)
                    }
                }
            }
            .navigationTitle(
                userName.isEmpty
                ? "メモ帳"
                : "\(userName)のメモ帳"
            )
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button {
                        isShowingAddSheet = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
                ToolbarItem(placement: .topBarLeading) {
                        Button {
                            isShowingSettings = true
                        } label: {
                            Image(systemName: "gear")
                        }
                    }
            }
            .sheet(isPresented: $isShowingSettings) {
                SettingsView(
                    userName: $userName,
                    sortByFavorite: $sortByFavorite
                )
            }
        }
        .sheet(isPresented: $isShowingAddSheet) {
            MemoAddView()
        }
    }
    func deleteMemos(at offsets: IndexSet) {
        for index in offsets {
            let memo = displayedMemos[index]
            modelContext.delete(memo)
        }
    }
}

struct MemoAddView: View {
    @State private var title = ""
    @State private var content = ""

    @Environment(\.modelContext) private var modelContext
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            Form {
                Section("タイトル") {
                    TextField("メモのタイトル", text: $title)
                }

                Section("内容") {
                    TextEditor(text: $content)
                        .frame(minHeight: 200)
                }
            }
            .navigationTitle("新しいメモ")
        }
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button("キャンセル") {
                    dismiss()
                }
            }
            ToolbarItem(placement: .confirmationAction) {
                Button("保存") {
                    let memo = Memo(
                        title: title,
                        content: content
                    )

                    modelContext.insert(memo)
                    dismiss()
                }
                .disabled(title.isEmpty)
            }
        }
    }
}


struct MemoRow: View {
    let memo: Memo

    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(memo.title)
                    .font(.headline)

                Text(memo.content)
                    .font(.caption)
                    .foregroundStyle(.secondary)
                    .lineLimit(2)

                Text(memo.createdAt, style: .date)
                    .font(.caption2)
                    .foregroundStyle(.tertiary)
            }

            Spacer()

            if memo.isFavorite {
                Image(systemName: "star.fill")
                    .foregroundStyle(.yellow)
            }
        }
        .padding(.vertical, 2)
    }
}

struct MemoEditView: View {
    @Bindable var memo: Memo

    var body: some View {
        Form {
            Section("タイトル") {
                TextField("タイトル", text: $memo.title)
            }

            Section("内容") {
                TextEditor(text: $memo.content)
                    .frame(minHeight: 200)
            }

            Section {
                Toggle("お気に入り", isOn: $memo.isFavorite)
            }
        }
        .navigationTitle("メモを編集")
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct SettingsView: View {
    @Binding var userName: String
    @Binding var sortByFavorite: Bool
    @Environment(\.dismiss) private var dismiss
    var body: some View {
        NavigationStack {
            Form {
                Section("ユーザー設定") {
                    TextField("ユーザー名", text: $userName)

                }
                Section("表示設定") {
                    Toggle("お気に入りを優先して表示", isOn: $sortByFavorite)
                }
                Section {
                    Text("設定はアプリを閉じても保存されます")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .navigationTitle("設定")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("完了") {
                        dismiss()
                    }
                }
            }
        }
    }
}


#Preview{
    MyView()
}
```
