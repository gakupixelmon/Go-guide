## 第9章：フロントエンドとの共通言語！〜JSONデータの扱い方〜

ハッカソンでは、Go言語で作ったサーバー（バックエンド）と、スマートフォンアプリやWeb画面（フロントエンド）を連携させて一つのプロダクトを作り上げます。

例えば、カメラを用いたドローンの制御アプリを開発するとしましょう。ドローンの「現在のバッテリー残量」や「カメラの稼働状態」、「現在地の座標」といった情報をスマホ画面にリアルタイムで表示するには、サーバーとスマホの間でデータをやり取りするための「世界共通語」が必要になります。

現在、その共通語として最も広く使われているデータ形式が**JSON（ジェイソン：JavaScript Object Notation）**です。今回は、Go言語でJSONデータを自在に操る方法を学びます。

---

### 9.1 JSONとは？

JSONは、データを「キー（名前）」と「値」のペアで表現する、人間にもコンピュータにも読みやすいテキスト形式のデータです。

**JSONのデータ例（ドローンの状態データ）**
~~~json
{
  "battery_level": 85,
  "camera_active": true,
  "status_message": "飛行は安定しています"
}
~~~

Go言語の「構造体（struct）」と非常に似ていますが、JSONはあくまで「文字列（テキスト）」であるという点が重要です。

---

### 9.2 Goの構造体をJSONに変換する（Marshal）

バックエンドからフロントエンドへデータを「送信（レスポンス）」する際は、Go言語の構造体をJSON形式の文字列（正確にはバイトデータの集まり）に変換します。この変換作業を**Marshal（マーシャル：直列化）**と呼びます。

Neovim（`nvim main.go`）で以下のコードを書いてみましょう。

~~~go
package main

import (
    "encoding/json" // JSONを扱うための標準パッケージ
    "fmt"
)

// DroneTelemetry（ドローンの遠隔測定データ）構造体の定義
// バッククォート(`)で囲まれた部分は「構造体タグ」と呼ばれます
type DroneTelemetry struct {
    Battery int    `json:"battery_level"` // JSONでは "battery_level" というキーにする
    Camera  bool   `json:"camera_active"`
    Message string `json:"status_message"`
}

func main() {
    // 1. 送信したいデータを構造体で作成する
    telemetry := DroneTelemetry{
        Battery: 85,
        Camera:  true,
        Message: "飛行は安定しています",
    }

    // 2. 構造体をJSON形式のバイトデータに変換（Marshal）する
    // 戻り値は「変換後のデータ」と「エラー情報」の2つ
    jsonData, err := json.Marshal(telemetry)
    
    // エラーハンドリング（ハッカソンでも非常に重要です！）
    if err != nil {
        fmt.Println("JSON変換エラー:", err)
        return
    }

    // 3. バイトデータ([]byte)を文字列(string)に直して表示する
    fmt.Println("生成されたJSONデータ:")
    fmt.Println(string(jsonData))
}
~~~

**解説：構造体タグ（Struct Tag）**
`json:"battery_level"` のように書くことで、「Go言語の世界では `Battery` という変数名だけど、JSONに変換する時は `battery_level` という名前にしてね」と指示（マッピング）することができます。他の言語と連携する際、命名規則の違いを吸収する強力な機能です。

---

### 9.3 JSONをGoの構造体に変換する（Unmarshal）

逆に、フロントエンドから「ドローンを前進させろ」といった指示（リクエスト）がJSON形式で送られてきた場合は、それをGo言語が理解できる構造体に変換し直す必要があります。これを**Unmarshal（アンマーシャル：非直列化）**と呼びます。

~~~go
package main

import (
    "encoding/json"
    "fmt"
)

// 受信用構造体
type Command struct {
    Action string `json:"action"`
    Speed  int    `json:"speed"`
}

func main() {
    // フロントエンドから送られてきたと仮定するJSONデータ（文字列）
    // ※文字列の中にダブルクォーテーションを含めるため、バッククォート(`)で全体を囲んでいます
    jsonString := `{"action": "forward", "speed": 30}`

    // JSON文字列をバイトデータに変換
    jsonBytes := []byte(jsonString)

    // 受け皿となる構造体を用意
    var cmd Command

    // バイトデータを構造体に変換（Unmarshal）する
    // ※第2引数には、受け皿の変数に「&」をつけて渡します（ポインタ渡し）
    err := json.Unmarshal(jsonBytes, &cmd)
    
    if err != nil {
        fmt.Println("データ解析エラー:", err)
        return
    }

    // Go言語のデータとして扱うことができるようになる
    fmt.Println("受信したコマンド:", cmd.Action)
    fmt.Println("設定スピード:", cmd.Speed)
}
~~~

**解説：ポインタ渡し（`&`）**
`json.Unmarshal` の第2引数に `&cmd` と渡しているのは、「この `cmd` という変数の入っているメモリの場所（アドレス）を教えるから、そこに変換したデータを直接書き込んでね」という指示です。

---

### 9.4 第9章の確認問題

本章の理解度を確認しましょう。

**問題1**
Go言語でJSONデータをエンコード（変換）したりデコード（解析）したりするためにインポートする標準パッケージは次のうちどれですか？
A) `net/http`
B) `encoding/json`
C) `text/template`

**問題2**
Go言語の「構造体（struct）」のデータを、JSON形式のデータ（バイト列）に変換するための関数はどれですか？
A) `json.Marshal`
B) `json.Unmarshal`
C) `json.Stringify`

**問題3**
JSON形式のデータ（バイト列）を読み解いて、Go言語の「構造体（struct）」に割り当てる（パースする）ための関数はどれですか？
A) `json.Parse`
B) `json.Marshal`
C) `json.Unmarshal`

**問題4**
Goの構造体のフィールド（変数）が、JSONのどのキーに対応するかを明示的に指定（マッピング）するための仕組みを何と呼びますか？（例：`` `json:"battery_level"` ``）
A) 構造体メソッド (Struct Method)
B) 構造体インターフェース (Struct Interface)
C) 構造体タグ (Struct Tag)

**問題5**
`json.Marshal` 関数が正常に処理を終えたときに返す、JSONデータの実体の型（データ形式）は何ですか？
A) `string` （文字列）
B) `[]byte` （バイトスライス / バイト列）
C) `map[string]interface{}` （マップ）

---

### 9.5 解答と解説

**問題1の正解：B**
*解説：*
* **A) `net/http`**：第8章で学んだ、Webサーバーを立ち上げるためのパッケージです。
* **B) `encoding/json`**：JSONデータの生成と解析（エンコード/デコード）を行うパッケージです。ハッカソンでは `net/http` とセットで頻繁に使われます。
* **C) `text/template`**：テキストベースのテンプレートを処理するためのパッケージです（動的なHTMLの生成などに使われます）。

**問題2の正解：A**
*解説：*
* **A) `json.Marshal`**：直列化（メモリ上のデータをネットワークで送れる形式に並べること）を行う関数です。
* **B) `json.Unmarshal`**：非直列化（届いたデータをメモリ上の構造体に復元すること）を行う関数です。
* **C) `json.Stringify`**：JavaScriptなどの別言語でJSON文字列にする際によく使われる関数名ですが、Go言語には存在しません。

**問題3の正解：C**
*解説：*
* **A) `json.Parse`**：これも他の言語（JavaScriptなど）でよく使われる名称です。
* **B) `json.Marshal`**：Go -> JSON の変換です。
* **C) `json.Unmarshal`**：JSON -> Go の変換です。Un（逆の）+ Marshal（直列化）という意味合いです。

**問題4の正解：C**
*解説：*
* **A) 構造体メソッド**：構造体に紐づいた関数（振る舞い）のことです。
* **B) 構造体インターフェース**：Go言語で「どのようなメソッドを持っているべきか」を定義するルールのことです。
* **C) 構造体タグ**：フィールドの宣言の右側にバッククォートで記述するメタデータです。データベースの列名と紐付ける際などにも応用される、非常に拡張性の高い機能です。

**問題5の正解：B**
*解説：*
* **A) `string`**：文字列です。`string(jsonData)` とキャスト（型変換）することで、ターミナルで人間が読みやすい形で表示できます。
* **B) `[]byte`**：バイト（8ビットのデータ単位）の可変長リスト（スライス）です。ネットワーク越しの通信やファイルの読み書きは、文字列ではなくこの `[]byte` 形式で行われるのがGo言語の基本です。
* **C) `map[string]interface{}`**：どんな値でも入るマップです。構造体を定義せずに強引にJSONを受け取る奥の手として使われることがありますが、基本は `[]byte` を返します。
