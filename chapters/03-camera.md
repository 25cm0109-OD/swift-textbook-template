# 第3章：カメラの利用

> 執筆者：（氏名）
> 最終更新：YYYY-MM-DD

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、PhotosPickerでフォトライブラリから写真を選択し、UIImagePickerControllerでカメラ撮影した画像を扱う方法を学ぶ。具体的には非同期で画像データを読み込み、UIViewControllerRepresentableを使ってUIKitをSwiftUIに統合し、Coordinatorパターンを使ってカメラ機能と連携するアプリを題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ここに模範コード全体を貼る
// ============================================
// 第3章（基本）：写真を選択・撮影して表示するアプリ
// ============================================
// PhotosPickerを使ってフォトライブラリから写真を選択し、
// 画面に表示します。シミュレータでも動作します。
// ============================================

import SwiftUI
import PhotosUI

// MARK: - メインビュー

struct ContentView: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImage: Image?
    @State private var isShowingCamera = false
    @State private var capturedUIImage: UIImage?

    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 画像表示エリア
                imageDisplayArea

                // ボタンエリア
                HStack(spacing: 20) {
                    // フォトライブラリから選択
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Label("ライブラリ", systemImage: "photo.on.rectangle")
                    }
                    .buttonStyle(.bordered)

                    // カメラで撮影
                    Button {
                        isShowingCamera = true
                    } label: {
                        Label("カメラ", systemImage: "camera")
                    }
                    .buttonStyle(.borderedProminent)
                }
                .padding()
            }
            .navigationTitle("写真アプリ")
            .onChange(of: selectedItem) { _, newItem in
                Task {
                    await loadImage(from: newItem)
                }
            }
            .fullScreenCover(isPresented: $isShowingCamera) {
                CameraView(capturedImage: $capturedUIImage)
            }
            .onChange(of: capturedUIImage) { _, newImage in
                if let uiImage = newImage {
                    selectedImage = Image(uiImage: uiImage)
                }
            }
        }
    }

    // MARK: - 画像表示エリア

    @ViewBuilder
    private var imageDisplayArea: some View {
        if let image = selectedImage {
            image
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(maxHeight: 400)
                .clipShape(RoundedRectangle(cornerRadius: 16))
                .shadow(radius: 4)
                .padding()
        } else {
            RoundedRectangle(cornerRadius: 16)
                .fill(.gray.opacity(0.1))
                .frame(height: 300)
                .overlay {
                    VStack(spacing: 8) {
                        Image(systemName: "photo")
                            .font(.system(size: 48))
                            .foregroundStyle(.gray)
                        Text("写真を選択または撮影してください")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
                .padding()
        }
    }

    // MARK: - 画像の読み込み

    func loadImage(from item: PhotosPickerItem?) async {
        guard let item = item else { return }

        do {
            if let data = try await item.loadTransferable(type: Data.self),
               let uiImage = UIImage(data: data) {
                selectedImage = Image(uiImage: uiImage)
            }
        } catch {
            print("画像の読み込みに失敗: \(error.localizedDescription)")
        }
    }
}

// MARK: - カメラビュー（UIKit連携）

struct CameraView: UIViewControllerRepresentable {
    @Binding var capturedImage: UIImage?
    @Environment(\.dismiss) private var dismiss

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: CameraView

        init(_ parent: CameraView) {
            self.parent = parent
        }

        func imagePickerController(
            _ picker: UIImagePickerController,
            didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
        ) {
            if let image = info[.originalImage] as? UIImage {
                parent.capturedImage = image
            }
            parent.dismiss()
        }

        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

（アプリの動作を自分の言葉で説明する。スクリーンショットを貼ってもよい。）

## コードの詳細解説

### PhotosPickerによる写真選択

```swift
// 該当部分のコードを抜粋して貼る
// フォトライブラリから選択
PhotosPicker(selection: $selectedItem, matching: .images) {
    Label("ライブラリ", systemImage: "photo.on.rectangle")
}
.buttonStyle(.bordered)
```

**何をしているか：**
（この部分が果たしている役割を説明する）


**なぜこう書くのか：**
（別の書き方ではなく、この書き方が選ばれている理由を説明する）

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### 画像の非同期読み込み

```swift
// 該当部分のコードを抜粋して貼る
func loadImage(from item: PhotosPickerItem?) async { // ← ① async がついている
    guard let item = item else { return }

    do {
        // ← ② try await を使って、重い読み込み処理を非同期で実行している
        if let data = try await item.loadTransferable(type: Data.self),
           let uiImage = UIImage(data: data) {
            selectedImage = Image(uiImage: uiImage)
        }
    } catch {
        print("画像の読み込みに失敗: \(error.localizedDescription)")
    }
}
```

**何をしているか：**
写真を安全に非同期で読み込んでいる
画像を2進数データで取り出してUIImage型に変換してSwiftUIで扱えるようにする

**なぜこう書くのか：**
画像データの変換は重たいのでUIをホールドしない為非同期(async)で行う
ユーザーが画像を選択しなかったときの為にguard letで安全に処理
SwiftUIで画像を使う為に生データ→UIImage→Imageに変換


**もしこう書かなかったら：**
コールバック関数にして非同期処理をクロージャーとして中に入れておく
→コードが若干長くなりasyncの手動呼び出しが必要 呼び出し専用の処理も無駄に必要
PhotosPickerも完全にUIKitにする
→写真選択用のViewも自作する必要があり、こーどがとてつもなく長くなり面倒

---

### UIViewControllerRepresentableによるカメラ連携

```swift
// 該当部分のコードを抜粋して貼る
func loadImage(from item: PhotosPickerItem?) async { // ← ① async がついている
    guard let item = item else { return }

    do {
        // ← ② try await を使って、重い読み込み処理を非同期で実行している
        if let data = try await item.loadTransferable(type: Data.self),
           let uiImage = UIImage(data: data) {
            selectedImage = Image(uiImage: uiImage)
        }
    } catch {
        print("画像の読み込みに失敗: \(error.localizedDescription)")
    }
}
```

**何をしているか：**


**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### Coordinatorパターン

```swift
// 該当部分のコードを抜粋して貼る
class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    let parent: CameraView

    init(_ parent: CameraView) {
        self.parent = parent
    }

    // ① 写真が撮影し終わったときにUIKitから呼ばれるデリゲートメソッド
    func imagePickerController(
        _ picker: UIImagePickerController,
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
    ) {
        if let image = info[.originalImage] as? UIImage {
            parent.capturedImage = image // SwiftUI側の変数に画像を渡す
        }
        parent.dismiss() // カメラを閉じる
    }

    // ② キャンセルされたときに呼ばれるメソッド
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        parent.dismiss()
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
| 例：`PhotosPicker` | フォトライブラリから画像を選択するコンポーネント | `PhotosPicker(selection: $selectedItem, matching: .images)` |
| 例：`UIImagePickerController` | カメラまたはフォトライブラリにアクセスするUIKitコンポーネント | `picker.sourceType = .camera` |
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




# やっぱり自分で書くことにした

## 実装手順

Phase 1: 見た目（UI）を1つずつ作る
まずは動かない「置物」としての画面を作ります。
•	Step 1: アプリの「外枠」を作る
•	やること: ContentView の中に NavigationStack を置き、ナビゲーションタイトルに「写真アプリ」と表示させる。
•	ゴール: プレビューやシミュレータを起動して、画面上に「写真アプリ」という大きなタイトルが見えればOK。
•	Step 2: 写真が表示される「エリア」の枠だけ作る

•	やること: タイトルの下に、まだ画像がない時のグレーの四角形（RoundedRectangle）を配置する。大きさ（frame）や角丸（cornerRadius）を設定し、中に「写真を選択または撮影してください」というテキストを表示する。

•	ゴール: 画面上部にグレーのエリア、その中に文字が表示されていること。

•	Step 3: 2つのボタンを並べる
•	やること: グレーの四角形の下に、HStack を使って「ライブラリ（PhotosPickerの代わりの普通のButton）」と「カメラ（Button）」を横並びにする。デザイン（.buttonStyle）も整える。
•	ゴール: 模範コードと同じ見た目の画面が完成する。この時点ではボタンを押しても何も起きない。


Phase 2: 写真を「保持する変数」と「条件分岐」を作る
ここからアプリに「状態（データ）」を持たせます。
•	Step 4: 画像を保存する「箱」を作る
•	やること: 画面の一番上に @State private var selectedImage: Image? という変数を定義する（初期値は nil）。

•	Step 5: 画像がある時とない時で画面を切り替える
•	やること: Step 2で作ったグレーの四角形の部分を、if let image = selectedImage という条件分岐（if文）で囲む。

•	image がある場合 ➔ その画像を表示し、大きさやシャドウを整えるコードを書く。
•	nil（ない）の場合 ➔ Step 2で作ったグレーの四角形を表示する。
•	ゴール: ビルドが通り、最初は画像がないのでグレーの四角形が表示されていること。
•	💡 じぶん実験: 試しに selectedImage の初期値に Image(systemName: "star.fill") などを入れてみて、星マークに切り替わるか試すと仕組みがよく分かります！

Phase 3: ライブラリから写真を選ぶ（最難関）
ここから外部のライブラリ（PhotosUI）を組み込みます。
•	Step 6: PhotosUIを導入し、ボタンを置き換える
•	やること: ファイルの最上部に import PhotosUI を書く。そして、選んだ写真の情報を入れる @State private var selectedItem: PhotosPickerItem? という変数を追加する。

•	Step 7: 「ライブラリ」ボタンを本物に切り替える
•	やること: Step 3で作った「ライブラリ」の普通のボタンを、PhotosPicker(selection: $selectedItem, matching: .images) に書き換える。

•	ゴール: ボタンを押した時に、iPhoneの写真一覧（ライブラリ）が下からひょっこり現れるようになること（まだ選んでも画面には反映されません）。

•	Step 8: 写真が選ばれたら、データを取り出す
•	やること: ContentView の末尾（.navigationTitle の下など）に .onChange(of: selectedItem) をつける。値が変わったら、Task { await loadImage(from: newItem) } を呼び出すようにする。

•	Step 9: 画像読み込み関数 loadImage を自力で書く
•	やること: body の外側に、async がついた loadImage 関数を作る。模範コードを参考に、item.loadTransferable(type: Data.self) でデータを取り出し、それを UIImage ➔ SwiftUIの Image に変換して、Step 4で作った selectedImage に代入する処理を書く。



•	ゴール: ライブラリから写真を選ぶと、画面のグレーの四角形がその写真に切り替わる！（ここまで出来たら大感動です）

Phase 4: カメラで撮影する
最後にカメラ機能を実装します。

•	Step 10: カメラ画面を呼び出す「スイッチ」を作る
•	やること: カメラ画面が開いているかを管理する @State private var isShowingCamera = false（最初は false）を作る。「カメラ」ボタンを押したら、これを true にする処理を書く。さらに、画面全体に .fullScreenCover(isPresented: $isShowingCamera) を付与する。

•	ゴール: カメラボタンを押すと、画面が全画面で切り替わる（中身はまだ仮のテキストなどでOK）。

•	Step 11: 撮影した画像を受け取って表示する
•	やること: 撮影された画像を受け取る @State private var capturedUIImage: UIImage? を作る。.fullScreenCover の中で CameraView(capturedImage: $capturedUIImage) を呼び出す（※CameraView 自体は別ファイル等で定義されている前提です）。

•	最後に、.onChange(of: capturedUIImage) を使って、カメラから画像が返ってきたらそれを Image に変換して selectedImage に代入する。
•	ゴール: カメラで撮影した写真が、メイン画面に表示される！


# 完成したコード
```swift

import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var capturedUIImage:UIImage?
    @State private var selectedImage: Image?
    @State private var selectedItem: PhotosPickerItem?
    @State private var isShowingCamera = false
    var body: some View {
        NavigationStack {
            VStack {
                if let image = selectedImage {
                    image
                        .resizable()
                        .scaledToFit()
                        .frame(height: 600)
                        .cornerRadius(16)
                }else{
                    RoundedRectangle(cornerRadius: 16)
                        .fill(.gray.opacity(0.2))
                        .frame(height: 600)
                        .overlay {
                            Text("写真を選択または撮影してください")
                                .font(.title3)
                        }
                }

                HStack{
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Text("ライブラリ")
                    }
                    .buttonStyle(.borderedProminent)
                    Button("写真を撮影") {
                        isShowingCamera = true
                        // 写真撮影の実装
                    }
                    .buttonStyle(.borderedProminent)
                }
            }
            .navigationTitle("写真アプリ")
            .onChange(of: selectedItem) { newItem in
                Task {
                    await loadImage(from: newItem)
                }
            }
        }.fullScreenCover(isPresented: $isShowingCamera){
            CameraView(capturedImage: $capturedUIImage)

        }.onChange(of: capturedUIImage) { _, newImage in
            if let uiImage = newImage {
                selectedImage = Image(uiImage: uiImage)
            }


        }
    }
        func loadImage(from item: PhotosPickerItem?) async {
            guard let item else { return }

            do{
                if let data = try await item.loadTransferable(type: Data.self){
                    if let uiImage = UIImage(data: data) {
                        selectedImage = Image(uiImage: uiImage)
                    }
                }

            } catch {

                print("画像の読み込みに失敗: \(error.localizedDescription)")
            }
        }
    }

struct CameraView: UIViewControllerRepresentable {
        @Binding var capturedImage: UIImage?
        @Environment(\.dismiss) private var dismiss

        func makeUIViewController(context: Context) -> UIImagePickerController {
            let picker = UIImagePickerController()
            picker.sourceType = .camera
            picker.delegate = context.coordinator
            return picker
        }

        func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

        func makeCoordinator() -> Coordinator {
            Coordinator(self)
        }

        class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
            let parent: CameraView

            init(_ parent: CameraView) {
                self.parent = parent
            }

            func imagePickerController(
                _ picker: UIImagePickerController,
                didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
            ) {
                if let image = info[.originalImage] as? UIImage {
                    parent.capturedImage = image
                }
                parent.dismiss()
            }

            func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
                parent.dismiss()
            }
        }
    }

#Preview {
    ContentView()
}
```
