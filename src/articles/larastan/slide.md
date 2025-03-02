---
marp: true
---

# About PHPStan extension of Larastan

実践的 Laravel 11の型安全な書き方。

@taiki-kuraishi

---

# 型安全っていいよね

- 型安全とは
→ 一言で言うと、型エラーを起こさないということ。

- 可読性の向上
  - 返り値の `array` `Collection`の中身が一目でわかる
  - `mixed`の根絶

<br/>
→ PHPstanいれよう!

→ Laravelで型安全に書く方法を共有します。

---

# PHPstanとは

- phpの静的解析ツール
- open-sourceで、無料

# Larastanとは

- LaravelのためのPHPstanのextension
- laravelの Magic method, Facadeに対応

    → クラスをインスタンスしなくても、staticメソッドのように呼び出す機能
    → `Auth::`, `DB::`, `Gate::`, etc ...

---

# PHPstan 設定

- PHPstan Level
  - Levelによってチェックされる内容が異なる
  - Levelは累積で、Level2は0、1、2を内包する

- Level 抜粋
  0. 未定義の`class`, `function`, `method`をチェック
  2. PHP Docsの検証
  4. `dead code`のチェック
  10. mixedの根絶

※ 引用: <https://phpstan.org/user-guide/rule-levels>

---

# PHPstan Baseline

- PHPstan を後から導入したら、大量のエラーが発生してしまった

- BaseLineを定義する
  >PHPStanでは、現在報告されているエラーリストを「ベースライン」として宣言し、それ以降の実行で報告されないようにすることができる。これにより、新しいコードや変更されたコードにおける違反にのみ関心を持つことができる。
  <https://phpstan.org/user-guide/baseline>

---

# PHPStan 導入

- どのレベルを入れようか
  - 厳しすぎるチェックは、開発速度を低下させる
  - `dead code`のチェックができるLevel 4くらいでいいかなぁ

<br />

- 相談しよう
  - 倉石:
    PHPStanにLevelっていう概念がありまして...
    Level 5くらいですすめようかと....
  - Yさん: え、10(最大)でよくね
  - 倉石: ぞす

---

# Request編

- `$request-input('key')`
  - リクエストからクエリーパラメータやbodyを取り出す
  → しかし、こいつは`mixed`を返す（バリデーション済みであっても）

- `$request->safe()->string('key')`
  - こうすると、返り値が`string`になる
  - `boolean()`, `integer()`, `float()`もある。
  - しかしこれらのメソッドは、内部的には型キャストしているだけ。
    → 例えば、存在しないクエリーパラメータでも`string()`は`""`を返してくる
    →　存在確認は、`$request->safe()->exists('key')`を使いましょう。

---

# Config編

- `config('key')`
  - config( 環境変数 )を取り出す
  → こいつも`mixed`を返す

- `app('config')->string('key')`
  - こうすると、返り値が`string`になる

---

# Custom Exception

- カスタムエラー

    ```php
    class CustomException extends Exception
    {
        public string $message = "error message"
    }
    ```

- 継承元のプロパティ(属性)と競合するので以下で書く

    ```php
    class CustomException extends Exception
    {
        public function __construct()
        {
            parent::__construct('error message');
        }
    }
    ```

---

# Factory編

- `User::factory()->create()`
  - モデルの作成、主にテストでよく使われる
  - 返り値は、`Collection<int, TModel>|TModel`

- 1つだけ作りたいとき
  → `User::factory()->createOne(): TModel`
- 複数作りたいとき
  → `User::factory()->createMany(): Collection<int, TModel>`

<br />
※ phpでもジェネリック型使えます...

---

# さいごに

Laravelを型安全に書く記事がネットになかったので書きました。
お手隙で、base lineのエラーを解消していただけると助かります。
PHPstanで型エラーを減らし、綺麗なコードを書いていきましょう。
