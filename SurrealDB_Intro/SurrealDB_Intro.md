# SurrealDBの紹介

<aside>
👉 Abstrct**：** この資料は[関西DB勉強会 #12](https://kansaidbstudy.connpass.com/event/268133/) Jan 21 2023 でのスライドです。

</aside>

目次

# SurrealDBのライセンス

---

- SurrealDBのソースコードはBusiness Software License 1.1
- SDKやライブラリ/ドライバはMIT
- SurrealDBのBSLは、商用DBaaSとして提供しない限り、ノード数に制限なくSurrealDBを使用することがでる
    - 製品に組み込むこともOK
- SurrealDBのBSLは、4年間有効
- 2026年1月1日この制限は失効、コードは現在のApache License 2.0によるオープンソースになる
    - どんな目的にも自由に使用することができる

# SurrealDBの特徴

---

- Rustで実装
    - Segmentation Faultが発生しない
    - クロスコンパイル
- 軽量: バイナリサイズ Linux 24MB, macOS: 44MB
- ひとつのバイナリでサーバーとREPLクライアントを兼用
- 簡単なインストール
- HTTP/Restful APIをサポート
- WebSocketをサポート
- バックエンドDB: EchoDB, RocksDB, TiKV, FoundationDB, IndexedDB

# データベースとしての特徴

---

- スキーマーレス：スキーマーを定義しても問題ない
- 多様な保存形式：テーブル、ドキュメント、グラフなど
- 複数行、複数テーブルのACIDトランザクション
- レコードリンクと有向グラフ接続
    - JOINが不要、N+1問題をスマートに回避
- 分析クエリの事前定義
    - データが書き込まれると、選択、集約、グループ化、順序づけ
- 埋め込みJavaScriptでクエリを拡張できる
- クエリに正規表現を記述できる （/regex/)
- GeoJSONをサポート
- CRUD操作を並列実行できる

# SurrealDBの強み

- 差別化できる機能を3つ同時に備えている
    - ユーザーがfrontendから直接アクセスできる
    - データベース側で認証認可を適切に設定できる
    - リアルタイムでデータが同期できる
- 対抗馬は少なく Google Firestore などがあるぐらい
    - オンプレミスで使用が条件となるとオンリーワンのDB
- データベース側で認証認可ができると…
    - [ロールベースアクセス制御](https://ja.wikipedia.org/wiki/%E3%83%AD%E3%83%BC%E3%83%AB%E3%83%99%E3%83%BC%E3%82%B9%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E5%88%B6%E5%BE%A1)を提供できることになる
        - 管理者、編集者、閲覧者など定義した役割に応じたアクセス制御

# SurrealDBの弱み

---

- 公開されてからまだ日が経っていない(2022年Jul)
    - 潜在的なセキュリティの脆弱性やバグが存在する可能性がある
        - PostgreSQLの初版は1997年、前身のPostgressは1989年)
        - MySQL の初版は1995年
        - MariaDB の初版は2009年
        - MongoDB の初版は2009年
- 情報が少ない
    - 公式ドキュメントも製作中
    - 困ったらソースコードを読めということ
- 機能をすべて実装できていない
- SurrealDB社は今時点で未収益：DBaaSのサービスインは2023年

# 現在開発中の機能 (beta-9でも未完）

- 分散モードでのマルチノード対応
- レプリケーション
- ヘルスチェック
- GraphQL
- FULLTEXT - フルテキストインデクス
- LEARNフィールド
    - 指定されたフィールドの機械学習分析に基づいて自動設定
- バージョン管理された一時テーブル
    - データ参照するときに「過去にさかのぼる」ことができる
- IDEコードハイライト(Atom, VSCode, Vim…)
- ユーザI/Fアプリのリリースは1.xとして計画にはある

# SurrealDBの2-Tierアーキテクチャ概要

![surrealdb_arch.png](https://github.com/iisaka51/KansaiDB/blob/main/SurrealDB_Intro/surrealdb_arch.png)

# SurrealDBの動作概要

![surrealdb_dataflow.png](https://github.com/iisaka51/KansaiDB/blob/main/SurrealDB_Intro/surrealdb_dataflow.png)

# ソースコード

- 数値はコメントを含む行数(SurrealDB beta-8)
    
    ちなみに MariaDB 10.9 は 130万行超（クライアントだけで12,000行超）
    

```bash
|-- src            // API layer　   4172　
  　  |-- net                                    1981
　    |-- cli                                    895
　    |-- rpc                                    174
 　   ...
|-- lib/src       // BL   Layer     32261
  　  |-- sql                                    19030
 　   |-- fnc                                    3524
 　  ...
　    |-- kvs                                    3286
　          |-- indexdb                          220
　          |-- rocksdb                          316
 　         |-- tikv                             269
      ...
```

# SurrealDBで実装している機能は驚くほど少ない

- クエリ解析のために nom を利用
    - 関数を連結してインクリメンタルにパーサーを構築できる
- メモリ格納するために echodb を利用 （Tobie)
    - マルチバージョン並行処理制御が可能なin-Memory KVS DB
- KVSへ格納するために storekey を利用（Tobie)
    - 辞書順序を保持してバイナリエンコーディングする
    - ソートされたKVSのキーを作成するときに便利
- シリアライズ/ディシリアライズはMsgPack と serde を利用
- GeoJSONの解析には geo を利用
- ローカルファイルをデータストアにするときはRocksDBを利用
- 分散DBとしての機能はTiKVやFoundationDB を利用

# SurrealDBがKVSに保存する項目

- メタデータ
    - テーブル、インデックス、スコープなどの構造
- データ
    - SurealDBが保持するオブジェクトの値

# SurrealDBのKVSへの保存方法

- Rust構造体とMsgPackをserdeバインディングでマップ
    - ストレージサイズの節約
    - シリアライズ/ディシリアライズの効率化
- KVSのデータアクセスの方法は2つ
    - キーベース：特定のキーを指定して、値を取得　（速い）
    - スキャン：キーの範囲を指定して、すべての値を取得（遅い）
- 階層構造をキーに保持
    - Namespace -> Database -> Table -> ID
- SurrealDBはキーを階層的に構築することでスキャンする範囲に変換

# インストール(簡単＆速い）

- Linux

```bash
$ curl -sSf https://install.surrealdb.com | sh
```

- macOS

```bash
$ brew install surrealdb/tap/surreal
```

- WindowsPS C:\> iwr https://windows.surrealdb.com -useb | iex

```powershell
PS C:\> iwr https://windows.surrealdb.com -useb | iex
```

- Docker/Podmandocker

```bash
$ docker run --rm -p 8000:8000 surrealdb/surrealdb:latest start
$ podman run --rm -p 8000:8000 surrealdb/surrealdb:latest start
```

# SurrealDBの起動

- start サブコマンドの第1引数がデータの書き出し先
    - デフォルトは memory
    
    ```bash
    $ surreal start --user root --pass root   memory
    ```
    
- その他の書き出し先
    
    ```bash
    file:///path/to/data.db                   ファイルシステム(RocksDB)
    rocksdb:///path/to/data.db                RocksDB
    tikv://endpoiint TiKV                     TiKV
    fdb:[///path/to/clusterfile]              FoundationDB (リビルドが必要）
    ```
    
- STRICTモード
    - **–strict** オプションを与えて起動
    - NAMESPACE,、DATABASEを定義しないとエラーになる
    - TABLE定義をしないとエラーになる

# ダンプ/リストア

- **export** サブコマンドでファイルにダンプ
- **import** サブコマンドでファイルからリストア

```bash
$ surreal export --conn http://dev00:8000 --ns test --db test  dump.db
$ surreal import --conn http://dev00:8000 --ns test --db test  dump.db
```

# クライアントから接続

- sqlサブコマンドを実行

```bash
$ surreal sql --conn http://dev00:8000 --ns test --db test --pretty
```

- **–-user** と **–pass** ROOT認証のユーザ/パスワード
- **–-pretty** でJSON出力ば整形して表示
- **–-ns** NAMESPACE **–-db** DATABASE を指示する
- SurrealDBがSTRICTモードで起動しているときは、—ns と —db オプションは無視される

# HTTP RESTful API

| PATH | TYPE | 説明 |
| --- | --- | --- |
| /key/:table | GET | データベースからテーブル内の全レコードを取得 |
| /key/:table/:id | GET | データベースから特定のレコードを取得 |
| /key/:table | POST | データベース内のテーブルにレコードを作成 |
| /key/:table/:id | POST | データベース内のテーブルに特定のレコードを作成 |
| /key/:table | DELETE | データベースからテーブルの全レコードを削除 |
| /key/:table/:id | PUT | データベース内の指定されたレコードを更新 |
| /key/:table/:id | PATCH | データベース内の指定されたレコードを変更 |
| /key/:table/:id | DELETE | データベース内の指定されたレコードを削除 |

# ****HTTP RESTful API (cont.)****

| PATH | TYPE | 説明 |
| --- | --- | --- |
| /version | GET | SurrealDBのバージョンを返す |
| /signup | POST | SCOPE認証の登録 |
| /signin | POST | SCOPE認証でログイン |
| /rpc | POST | WebSocketへJON-RPCでリクエスト |
| /sql | POST | SurQLクエリを許可 |
| /export | GET | データベースの内容をダンプ |
| /import | POST | クエリの内容をデータベースに適用（リストア） |

# ****HTTP RESTful API (未完のもの)****

| PATH | TYPE | 説明 |
| --- | --- | --- |
| /sync | GET | レプリケーション |
| /health | GET | データベースのヘルスチェック |
| /status | GET | ステータスを返す |

# 一般的なSQLでのテーブル定義

```sql
CREATE TABLE human (
      id int,
      nickname text,
      age int,
      PRIMARY KEY(id)
   );
```

# SurealDBはスキーマーレス

- テーブルやフィールドを定義する必要がない
- フィールドを追加するときも変更する必要がない
- サーバーがSTRICTモードで起動されていると、先にテーブル定義が必要

```sql
CREATE human:freddie SET nickname="freddie", age=99 ;
CREATE human:brian SET nickname="brian", age=75, sex=true ;
```

<aside>
💡 ID = TableName:UniqID　　　　　　テーブル名が含まれていること注目

</aside>

# ****スキーマレスでフィールド型を指定****

```sql
CREATE human:freddie SET
       nickname = <string> "freddie",
       age = <int> 99 ;
```

# ****型とキャスト****

- **bool**, **int**, **float**, **string**, **number**, **decimal**, **datetime**, **duration**
- 日時文字列はISO8601に変換される： **<datetime>** でのキャストと同じ
- 日時文字列をそのまま文字列として扱いたいときは **<string>** でキャスト

```sql
SELECT * FROM "2023-01-01";
SELECT * FROM <datetime> "2023-01-01";
SELECT * FROM <string> "2023-01-01T02:03:00Z" + "-test";
```

# スキーマーの定義

```sql
DEFINE TABLE human SCHEMAFULL;
DEFINE FIELD nickname ON human TYPE string;
DEFINE FIELD age ON human TYPE int;
```

- テーブルをSCHMALESSとして[再]定義

```sql
DEFINE TABLE human SCHEMALESS ;
```

- テーブルをSCHMAFULLとして[再]定義

```sql
DEFINE TABLE human SCHEMAFULL ;
```

# SCHEMAFULL

- 定義されたフィールドで許可されたデータのみが格納されrる
- 特定のデータ型に制限することができる
- DEFINE FIELD でデータが入力されない場合のデフォルト値を設定できる
- 設定する値は $value にセットされる

```sql
DEFINE TABLE person SCHEMAFULL;
DEFINE FIELD name ON person TYPE string VALUE $value OR 'guest';
```

# データ追加

- •id フィールドの設定省略すると、IDは自動設定される

```sql
INSERT INTO human (nickname, age)
       VALUES ('brian', 75);

INSERT INTO human (id, nickname, age)
       VALUES ('human:freddie', 'freddie', -1);

CREATE human:robert SET nickname=’robert’, age=30;

CREATE human SET
       id = human:jack, nickname=’jack’, age=30;

CREATE human CONTENT
       { id: 'human:john', nickname: 'john', age: 99 };
```

# INSERT

- IDが重複する時 UPDATE することができる

```sql
INSERT INTO test (id, test, something)
    VALUES (‘tester’, true, ‘other’ ) ;
INSERT INTO test (id, test, something)
    VALLUES (‘tester’, true, ‘other’ )
    ON DUPLICATE KEY UPDATE something = ‘else’ ;
```

# フィールドのネスト

- テーブルのフィールドはネストさせることができる
- ネストされたフィールドはドット表記で参照できる

```sql
UPDATE person:test CONTENT {
              settings: {
                  nested: {
                      object: {
                          thing: 'test'
                      }
                  }
              }
          };
SELECT settings.nested.object FROM person ;
```

# NONEとNULL

- フィールドの値には**NONE**と**NULL**を持つことができる
    - **NONE**：値が設定されていない
    - **NULL**：空の値が設定されている

```sql
CREATE person:test1 SET email = 'info@example.com';
CREATE person:test2 SET email = NONE;
CREATE person:test3 SET email = NULL;
```

# USE

- 使用するNAMESPACE、DATABASEを指定
    - ROOT認証でアクセスしているときに有効

```sql
USE NAMESPACE test ;
USE NAMESPACE test DATABASE db1 ;
USE NS test DB db1 ;
```

一般的なSQLでのリレーション

```sql
CREATE TABLE armor (
    id int,
    name text,
    resistance text,
    PRIMARY KEY(id)
 );

INSERT INTO armor VALUES
    (0, "leather", 3);
    (1, "platemail", 30),
    (2, "chainmail", 20),

CREATE TABLE player (
   name text,
    strength int,
    armor_id int,
    PRIMARY KEY((name),
    CONSTRAIN fk_armor
      FOREIGN KEY(armor_id)
      REFERENCECS armor(id)
);
```

# ****SurrealQL(SurQL)でのリレーション****

```sql
CREATE armor:leather SET registance = 3;
CREATE armor:chainmail SET registance = 20;
CREATE armor:platemail SET registance = 30;
CREATE player:jack SET strength = 22, armor = armor:platemail;
CREATE player:brian SET strength = 20, armor = armor:leather;
```

<aside>
💡 ID にテーブル名が含まれていること利用

</aside>

# ****SurQL: スキーマーを定義してのリレーション****

```sql
DEFINE TABLE armor SCHEMAFULL;
DEFINE FIELD resistance ON armor TYPE int;

CREATE armor:leather SET resistance = 3;
CREATE armor:chainmail SET resistance = 20;
CREATE armor:platemail SET resistance = 30;

DEFINE TABLE player SCHEMAFULL;
DEFINE FIELD strength ON player TYPE int;
DEFINE FIELD armor ON player TYPE record(armor);

CREATE player:jack SET strength = 22, armor = armor:platemail;
CREATE player:brian SET strength = 20, armor = armor:leather;
```

# ****一般的なSQLでのリレーション：JOIN****

```sql
SELECT
    player.name,
    player.strength,
    armor.name AS armor_name,
    armor.resistance AS armor_resistance
 FROM player
 JOIN armor
 ON armor.id = player.armor_id
```

# ****SurQLでのリレーション：JOINが不要****

- FETCHで指定したフィールドが展開される

```sql
SELECT * FROM player FETCH armor;
```

# レコードリンク

```sql
CREATE armor:leather SET registance = 3;
CREATE armor:chainmail SET registance = 20;
CREATE armor:platemail SET registance = 30;

CREATE player:jack SET strength = 22, armor = armor:platemail;
CREATE player:brian SET strength = 20, armor = armor:leather;
```

<aside>
💡 **Foreign Key == Record Link**

</aside>

# ****リレーション: ONE-TO-ONE****

```sql
CREATE human:freddie SET nickname="freddie", age=99 ;
CREATE human:brian SET nickname="brian", age=75
```

```sql
UPDATE human:brian SET bff = human:freddie;

SELECT bff.nickname, bff.age FROM human:brian
```

<aside>
💡 外部テーブルのフィールドをドットで繋いで指示

</aside>

# リレーション: ONE-TO-MANY

```sql
CREATE car:tesla  SET model='Model S', ev=True, price=99000;
CREATE car:mustang SET model='Mustang Cobra', ev=False, price=60000;

UPDATE human:brian SET cars=["car:tesla"];
UPDATE human:freddie SET cars=["car:mustang"];
UPDATE car:tesla SET owner = human:brian;
UPDATE car:mustang SET owner = human:freddie;

CREATE parts:tire SET brand='Michelin', size=5;
CREATE parts:gastank SET brand='Tanksy', size=10;
CREATE parts:battery SET brand='Xi Ping', size=20;

UPDATE car:mustang SET parts = ['parts:tire', 'parts:gastank'];
UPDATE car:tesla SET parts = ['parts:tire', 'parts:battery'];
```

# リレーション: ONE-TO-MANY

```sql
SELECT parts FROM car:mustang
SELECT cars.parts.brand FROM human:brian ;
```

<aside>
💡 外部テーブルのフィールドをドットで繋いで指示

</aside>

# グラフコネクション

```sql
RELATE player:jack -> wants_to_buy -> armor:dragon;
RELATE player:jack -> wants_to_buy -> armor:platemail;
 
SELECT * FROM wants_to_buy;
SELECT id, -> wants_to_buy -> armor AS wtb FROM player;
SELECT id, <- wants_to_buy <- player AS players FROM armor:dragon
```

<aside>
💡 外部テーブルの　”->” や “<-” で繋いで指示

</aside>

# LIMIT

- SELECTで返す結果の数を指定する
- 今時点ではFETCHを指示するとうまく動作しない

```sql
CREATE tag:rs SET name = 'Rust';
CREATE tag:go SET name = 'Golang';
CREATE tag:js SET name = 'JavaScript';

CREATE person:tobie SET tags = [tag:rs, tag:go, tag:js];
CREATE person:jaime SET tags = [tag:js];

SELECT * FROM person LIMIT 1;
SELECT * FROM (SELECT * FROM person FETCH tags) LIMIT 1;
// SELECT * FROM person LIMIT 1 FETCH tags;
```

# START

- SELECTで返す結果の始めの位置を指定する（ゼロはじまり）
- FETCHを指示してもうまく動作する

```sql
SELECT * FROM person START AT 1;
SELECT * FROM (SELECT * FROM person FETCH tags) START 1;
SELECT * FROM person START 1 FETCH tags;
```

# ****SurQL: WHERE, ORDER BY, GROUP BY****

```sql
SELECT * FROM armor ;

SELECT * armor WHERE resistance >= 30 ;
SELECT math::sum(strength) FROM player GROUP BY ALL ;

SELECT * FROM armor ORDER BY RAND();
SELECT * FROM armor ORDER RAND();

SELECT * FROM armor ORDER resistance NUMERIC ASC ;
SELECT * FROM armor ORDER resistance NUMERIC DESC ;
```

# ****SurQL: BEFORE, AFTER, DIFF****

- CREATE、UPDATE、DELETE でクエリの前後や差分を返すことができる

```sql
UPDATE human:freddie SET email = 'freddie@example.com';
UPDATE human:freddie SET email = 'freddie@dummy.com' RETURN DIFF ;
```

# ****分析クエリの事前定義****

```sql
DEFINE TABLE person SCHEMALESS;
DEFINE TABLE person_by_age AS
    SELECT
          count(),
          age,
          math::sum(age) AS total,
          math::mean(age) AS average
          FROM person
          GROUP BY age ;
```

# EVENT

- ON TABLE で指定したテーブルの内容：$before, $after
- イベントが発生したID：$this
- 発生したイベント： $event

```sql
UPDATE human:freddie SET email = 'freddie@example.com';
UPDATE human:brian SET email = 'brian@example.com';

DEFINE EVENT changelog ON TABLE human
       WHEN $before.email != $after.email 
       THEN ( CREATE changelog SET
              time = time::now(),
              email = $after.email );
```

# ****RANGE　コロンで指定した回数だけ繰り返す****

```json
> CREATE |test:10| SET time = time::now();
[
  {
    "result": [
      {
        "id": "test:g9hq0dowz77us5yvxnst",
        "time": "2022-12-20T04:10:19.282031670Z"
      },
      {
        "id": "test:nk45tn46dy2bn1hd6zj9",
        "time": "2022-12-20T04:10:19.282450969Z"
      },
```

# ****RANGE　初期値と終了値を指示して繰り返す****

```json
> CREATE |test:1..10| SET time = time::now();
[
  {
    "result": [
      {
        "id": "test:1",
        "time": "2022-12-20T04:12:21.477667592Z"
      },
      {
        "id": "test:2",
        "time": "2022-12-20T04:12:21.478289880Z"
      },
```

# 正規表現

```json
> SELECT * FROM test WHERE id = /.*[24].*/
[
  {
    "result": [
      {
        "id": "test:2",
        "time": "2022-12-20T04:12:21.478289880Z"
      },
      {
        "id": "test:4",
        "time": "2022-12-20T04:12:21.478332436Z"
      }
    ],
    "status": "OK",
```

# IF THEN ELSE

```sql
UPDATE person SET classtype =
    IF age <= 10 THEN
      'junior'
    ELSE IF age <= 21 THEN
      'student'
    ELSE IF age >= 65 THEN
      'senior'
    ELSE
      NULL
    END ;
```

# MERGE

- テーブルのフィールドをマージ：追加、削除

```sql
UPDATE person:test SET   
      name.initials = 'TMH',
      name.first = 'Tobie',
      name.last = 'Morgan Hitchcock';
UPDATE person:test MERGE {
      name: {
          title: 'Mr',
          initials: NONE,
          suffix: ['BSc', 'MSc'],
          }
     };
```

# ****ASSERT テーブル制約****

- 定義された各フィールドは、**ASSERT**でデータに対する制約を定義できる

```sql
DEFINE FIELD countrycode ON user TYPE string
    // Ensure country code is ISO-3166
    ASSERT $value != NONE AND $value = /[A-Z]{3}/
    // Set a default value if empty
    VALUE $value OR 'GBR'
  ;
```

# FUTURE関数

- • テーブルのフィールドをあとで設定される値によって定義する

```sql
UPDATE person:test SET
        can_drive = <future> {
             birthday && time::now() > birthday + 18y };

UPDATE person:test SET birthday = <datetime> '2007-06-22';
UPDATE person:test SET birthday = <datetime> '2001-06-22';
```

# PERMISSIONS

- •TABLE、FIELDのCRUD操作を制限する

```sql
DEFINE TABLE user SCHEMALESS
    PERMISSIONS 
    FOR select, create, update 
        WHERE id = $auth.id 
    FOR delete 
        WHERE id = $auth.id OR $auth.admin = true ;
```

# ACIDトランザクション

```sql
BEGIN TRANSACTION;
 
UPDATE coin:one SET balance += -23.00 ;
UPDATE coin:two SET balance -= 23.00 ;
 
COMMIT TRANSACTION;
```

- あるいは

```sql
CANCEL TRANSACTION;
```

- **DROP**が設定されているテーブルで新規トランザクションが指定されると、終了していないトランザクションは破棄される
- デフォルトは、新規トランザクションはリードオンリー(可能であれば）になる

# WebSocket

- ws://dev00:8000/rpc のようにURLを指定　(ws://、wss://)
- 次のようなJSONをメッセージボディに設定してPOST送信

```json
{
  "id": <識別するためのID>,   
  "method": <コマンド>, 
  "params": <コマンドが要求するパラメタの配列> 
}
```

# ****SurrealDBの認証****

- ROOT認証
    - サーバー起動時に指定したユーザ/パスワード
    - -pass オプションを省略するとROOT認証は無効になる
- ユーザ認証
    - **DEFINE LOGIN** で作成したユーザ/パスワード
- トークン認証
    - JSON Web Token (JWT)による認証 ([RFC7519](https://www.rfc-editor.org/rfc/rfc7519)、[RFC8725](https://www.rfc-editor.org/rfc/rfc8725))
    - 3rd パーティーのOAuth認証
- SCOPE認証
    - **SIGNUP**、**SIGNIN**を事前に定義
    - アクセス期間を限定することができる

# LOGIN

- NAMESPACE や DATABASE に対してアクセス制限ができる/co
- NAMESPACEに権限がないユーザはDB作成/削除ができない

```sql
DEFINE LOGIN admin ON NAMESPACE PASSWORD “admin.admin”;
DEFINE LOGIN guest ON DATABASE PASSWORD “guest.guest”;
```

# LOGINのイメージ

![surrealdb_login.png](https://github.com/iisaka51/KansaiDB/blob/main/SurrealDB_Intro/surrealdb_login.png)

# TOKEN

- 特定のトークンをヘッダに持つリクエストだけアクセスを許可
- NAMESPACE、DATABASE、SCOPEに対して設定できる

```sql
DEFINE TOKEN my_token ON DATABASE 
    TYPE HS512 VALUE '1234567890';
```

# SCOPE

- JSON-RPC の全てのフィールドが変数にセットされる
- SCOPEはデータベースにアクセスする能力を与える
- テーブルやフィールドへのアクセスはPERMISSIONSに従う
- TOKEN付きSCOPEは、テーブルの作成/変更/削除、情報の表示ができない

```sql
DEFINE FIELD email ON TABLE user TYPE string ASSERT is::email($value);
DEFINE INDEX email ON TABLE user COLUMNS email UNIQUE;

DEFINE SCOPE account SESSION 24h
	  SIGNUP ( CREATE user SET
                email = $email, 
                pass = crypto::argon2::generate($pass) )
  	SIGNIN ( SELECT * FROM user 
                WHERE email = $email 
                AND crypto::argon2::compare(pass, $pass) );
```

# SCOPEのイメージ

![surrealdb_scope.png](https://github.com/iisaka51/KansaiDB/blob/main/SurrealDB_Intro/surrealdb_scope.png)

# SCOPE認証にSIGNUP

```jsx
let jwt = fetch('https://api.surrealdb.com/signup', {
	method: 'POST',
	headers: {
		'Accept': 'application/json',
		'NS': 'google', // Specify the namespace
		'DB': 'gmail', // Specify the database
    },
	body: JSON.stringify({
		'NS': 'google',
		'DB': 'gmail',
		'SC': 'account',
		email: 'tobie@surrealdb.com',
		pass: 'a85b19*1@jnta0$b&!'
	}),
});
```

# ****クエリからトークンや認証データを参照****

- $session、$scope、$token および $auth には、クライアントに関連する特別な情報がセットされる
- NAMESPACEやDATABASE、 TOKENを使用している間は、 $session および $token がセットされる
- $toekn にはJWT トークンのすべてのフィールドがセットされる
- $scope には SCOPE認証でのSCOPE名がセットされる
- $authは、SCOPE認証でJWTがidフィールドを持ち、idで指定されたデータがテーブルに存在するときにセットされる

```sql
SELECT * FROM $session;
SELECT * FROM $token;
SELECT * FROM $scope;
SELECT * FROM $auth;
```

# LIVEクエリ

- WebScoket 経由でアクセスしたときに有効
- データ変更は、クライアント、アプリケーション、エンドユーザーデバイス、サーバーサイドライブラリーへリアルタイムにプッシュされる
- すべてのクライアントデバイスは同期させたまま維持される
- LIVEクエリを終了するためには**、KILL**で指定する

```sql
LIVE SELECT expr FROM what
```

# PARALLEL

- CREATE、DELETE、UPDATE、SELECT で PARALLEL を付加
- クエリ実行が並列処理される

```sql
SELECT * FROM test PARALLEL ;
```

# TIMEOUT

- CREATE、DELETE、UPDATE、SELECT で TIMEOUTを付加
- クエリ実行で指定した時間だけ待つ

```sql
SELECT * FROM 
    http::get('https://ipinfo.io')
    TIMEOUT 10s;
```

# GeoJSON

- Pont, Line, Polygon, MultiPoint, MultiLine, MultiPolygon, Collection

```sql
UPDATE university:oxford SET area = {
  type: 'MultiPolygon',
  coordinates: [
    [[ [10.0, 11.2],[10.5, 11.9],[10.8, 12.0],[10.0, 11.2] ]],
    [[ [9.0, 11.2], [10.5, 11.9],[10.3, 13.0], [9.0, 11.2] ]]
    ]
};

SELECT * FROM university:oxford;
```

# ****SurQL の組み込み関数****

- **array::xxxx()**
    - combine, complement, concat, difference, disinc, intersect, len, sort::asc, sort::desc, sort, union
- **count()**
- **crypto::xxxx()**
    - argon2::compare, argon2::generate, bcrypt::compare, bcrypt::generate, md5, pdkdf2::compare, pdkdf2::generate, scrypt::compare, scrypt::generate, sha1, sha25, sha512

# ****SurQL の組み込み関数 (cont.)****

- **duration::xxxx()**
    - days, hours, mins, secs, weeks, years
- **geo::xxxx()**
    - area, bearing, centroid, distance, hash::decode, hash::encode
- **http::xxxx()**
    - head, get, put, post, patch, delete
- **is::xxxx()**
    - alphanum, alpha, domain, email, hexadecimal, latitude, longitude, numeric, semver, url, uuid

# ****SurQL の組み込み関数 (cont.)****

- **math::xxxx()**
    - abs, bottom, ceil, fixed, floor, interquartile, max, mean, midhinge, min, mode, nearestrank, percentile, round, spread, sqrt, stddev, sum, top, trimean, variance
- **parse::email()**
    - host, user
- **parse::url::xxxx)_**
    - domain, fragment, host, port, path, query, scheme

# ****SurQL の組み込み関数 (cont.)****

- **rand()**
- **rand::xxxx()**
    - bool, enum, float, guid, int, string, time, uuuid:v4, uuid:v7, uuid
- **session::xxxx()**
    - db, id, ip, ns, origin, sc, sd, token,

# ****SurQL の組み込み関数 (cont.)****

- **string::xxxx()**
    - **concat, endsWith, join, length, lowercase, repeat, replace, reverse, slice, slug, split, startsWith, trim, uppercase, words**
- **time::xxxx()**
    - day, floor, format, group, hour, minute, month, nano, now, round, second, unix, wday, yday, year
- **type::xxxx()**
    - bool, datetime, decimal, duration, float, int, number, point, regex, table, thin

# ****SurQL の組み込み常数****

- **math::xxxx**
    - **E, FRAC_1_PI, FRAC_1_SQORT_2, FRAC_2_PI, FRAC_2_SQRT_PI, FRAC_PI_2, FRAC_PI_3, FRAC_PI_4, FRAC_PI_6, FRAC_PI_8, LN_10, LN_2, LOGO10_2, LOG10_E, LOG2_10, LOG2_E, PI, SQRT_2, TAU**

# LET パラメタ設定

- 数値、文字列などオブジェクトを変数に設定できる
- クエリから $変数 として参照できる

```sql
LET $test = { some: 'thing', other: true };
SELECT * FROM $test WHERE some = 'thing';
```

# ****JavaScriptでSurQLを拡張****

- SurrealDB からの全ての値は、JavaScript の型に自動変換
- JavaScript 関数からの戻り値は、SurrealDB の値に自動変換
- ブール値、整数、フロート、文字列、配列、オブジェクト、および日付オブジェクトは、すべて自動的に SurrealDB の値に変換または、SurrealDB の値から変換される

```sql
CREATE user:test SET created_at = function() {
	return new Date();
};
```

# ****JavaScript拡張のサンプル 1****

```sql
CREATE platform:test SET version = function() {
            const { platform } = await import('os');
            return platform();
        };
```

# ****JavaScript拡張のサンプル 2****

```sql
LET $value = 'SurrealDB';
LET $words = ['awesome', 'advanced', 'cool'];
CREATE article:test SET summary = function($value, $words) {
    return `${arguments[0]} is ${arguments[1].join(', ')}`;
};
```

# ****JavaScript拡張のサンプル 3****

```sql
CREATE film:test SET
    ratings = [
        { rating: 6.3 },
        { rating: 8.7 },
    ],
    display = function() {
        return this.ratings.filter(r => {
            return r.rating >= 7;
            }).map(r => {
                return { ...r,
                    rating: Math.round(r.rating * 10) };
            });
    };
```

# ****PythonでのRETful APIの簡単な実装例****

```python
from urllib.request import Request, urlopen
import base64
import json
from pprint import pprint as pp

_BASE_URL = "http://dev00:8000"
_NAMESPACE = "test"
_DATABASE = "test"
_USER = "root"
_PASS = "root"

auth_str = f"{_USER}:{_PASS}".encode("utf-8")
credential = base64.b64encode(auth_str)
auth = "Basic " + credential.decode("utf-8")

headers = {
    "Accept": "application/json",
    "Authorization": auth,
    "NS": _NAMESPACE,
    "DB": _DATABASE,
}

url = _BASE_URL + "/key/human"
request = Request(url, headers=headers)

with urlopen(request) as res:
    data = res.read()
    pp(json.loads(data)[0]['result'])
```

# ****PythonでのRETful API経由でSQLクエリを行う例****

```python
import requests
from requests.auth import HTTPBasicAuth
from pprint import pprint as pp

_URL = "http://dev00:8000/sql"
_NAMESPACE = "test"
_DATABASE = "test"
_USER = "root"
_PASS = "root"

_headers = {
  'Content-Type': 'application/json',
  'Accept':'application/json',
  'ns': _NAMESPACE,
  'db': _DATABASE
}
_auth = HTTPBasicAuth(_USER, _PASS)

def db(query):
    res = requests.post( _URL,
          headers=_headers,
         auth = _auth,
         data=query )
  if "code" in res.json():
      raise Exception(res.json())
  return res.json()

if __name__ == '__main__':
    while True:
      sql = input('SQL> ')
      if sql.upper() == 'Q':
          break
      val = db(sql)
      pp(val)
```

# ****PythonでのWebSocket経由でSQLクエリを行う例****

```python
import asyncio
from surrealdb import WebsocketClient
from pprint import pprint as pp

_URL = "ws://dev00:8000/rpc"
_NAMESPACE = "test"
_DATABASE = "test"
_USER = "root"
_PASS = "root"

async def main():
    async with WebsocketClient( url=_URL,
        namespace=_NAMESPACE, database=_DATABASE,
        username=_USER, password=_PASS,
    ) as session:
        while True:
            sql = input('SQL> ')
            if sql.upper() == 'Q': break
            res = await session.query(sql)
            pp(res)
```

# ****PyPIのものはWebSocketでうまく接続できない****

- プルリクエスト中
- このレポジトリならとりあえず動くよ

[GitHub - iisaka51/surrealdb.py at develop](https://github.com/iisaka51/surrealdb.py/tree/develop)

# その他の実装例

- Flask
    
    [GitHub - santiagodonoso/python_flask_surrealdb](https://github.com/santiagodonoso/python_flask_surrealdb)
    
- FastAPI
    
    [GitHub - santiagodonoso/fastapi_surrealdb_v_1](https://github.com/santiagodonoso/fastapi_surrealdb_v_1)
    
- PHP
    
    [GitHub - santiagodonoso/php_surrealdb](https://github.com/santiagodonoso/php_surrealdb)
    
- React
    
    [GitHub - rvdende/surrealreact: SurrealDB explorer UI written in react.](https://github.com/rvdende/surrealreact?ref=reactjsexample.com)
    
- Deno
    - [https://github.com/officialrajdeepsingh/surrealDb-deno](https://github.com/officialrajdeepsingh/surrealDb-deno)
- Typescript
    
    [SurrealDB Explained With Express.js, Node.js, and TypeScript](https://betterprogramming.pub/surrealdb-explained-with-express-js-node-js-and-typescript-98db891389c4)
    

# 留意するべき項目

- surrealdb-1.0.0-beat8 で使用できるバックエンドDB
    - RocksDB, TiKV
    - FoundationDB、IndexedDB はリビルドが必要
    - IndexedDB は組み込み用途での利用が本来かも
- DATABASEにアクセスするためにはNAMESPACEが必要
    - NAMESPACEが異なると同じ名前のDATABASEも別物になる
- 1つのNAMESPACEにはいくつでもDATABASEを作成できる
    - ただしNAMESPACEへのアクセス権限が必要/

# これはバグでしょう！

- 一部のSurQLはCLIではうまく動作しない
    - sqlサブコマンドはリターンキー押下でHTTP RestfulAPIアクセス
    - セミコロンの入力までを1つのクエリと認識しない
    - import サブコマンドであたえるファイル中ではOK
    - Copy&Pasetでリターン押下ならOK
- コメントは受け付けつけるけど、コメントだけだとエラー
    - 空文字（’’）を実行したことになるため
- LETで定義した変数はリターンキー押下で消失する
- トランザクションは途中でリターンキー押下するとNG
- USE もリターンキー押下で宣言が消失

# まとめ

- インストールと設定が驚くほど簡単
- Webアプリとの親和性がとてもとても高い
- JavaScriptで拡張可能な強力なクエリ
- ビジネス ロジックとユーザー認証をデータベース内で直接処理が可能
- バックエンド技術スタックの簡素化→開発期間を短縮→コスト削減
- まだ未完成ではあるものの、将来性は非常に高い

# 参考資料

- SurrealDB オフィシャルサイト
    
    [GitHub - surrealdb/surrealdb: A scalable, distributed, collaborative, document-graph database, for the realtime web](https://github.com/surrealdb/surrealdb)
    
- StackOverflow
    
    [Authentication Failure when using external JWT token in SurrealDB](https://stackoverflow.com/questions/74667534/)
    
    [Traverse relations upwards in SurrealDB](https://stackoverflow.com/questions/74364065/)
    
    [How can I log into a scope with SurrealDB?](https://stackoverflow.com/questions/73720759/)
    
    [How to specify a namespace when using websockets with surrealdb](https://stackoverflow.com/questions/74005492/)
    

# オススメのRESTクライアント

- **Surrealist**
    - 複数クエリの結果をタブに分割してくれる
    - SurrealDBの実装例としても優秀
    
    [GitHub - StarlaneStudios/Surrealist: ⚡ Lightning fast graphical SurrealDB query playground for Desktop](https://github.com/StarlaneStudios/Surrealist)
    
- **Tabbed Postman - REST Client**
    - Chromeの拡張機能
    - クエリをコレクションとして保存できる
    - コレクションはexport/import できる
    
    [Tabbed Postman - REST Client](https://chrome.google.com/webstore/detail/tabbed-postman-rest-clien/coohjcphdfgbiolnekdpbcijmhambjff)
    
- **Thunder Client**
    - VSCodeの拡張機能
    
    [Thunder Client - Rest API Client Extension for VS Code](https://www.thunderclient.com/)
    

# Q&A

# 予備スライド

---

# ****SurrealDBの歴史****

- 公開されてからの期間は浅いが2016年から開発が始まっている
    - 2016年 Feb　GoLangで開発開始
    - 2017年 Jul SaaS のバックエンドDBとして運用開始
    - 2021年 Oct Open Source として公開決定、Rustで再構築
    - 2022年 Jul Beta-01 リリース
    - 2022年 Aug Beta-05リリース
    - 2022年 Oct Beta-08 リリース
    - 2023年 Feb Beta-09 リリース？
- SurrealDB 社
    - 2021年 Nov SurrealDB Ltd. をロンドンに設立
    - 2023年 Jan DBaaS ために[600万ドル調達](https://mobile.twitter.com/tobiemh)

# ****SurrealDBが生まれた背景****

- **大きなトレンド**
    - データベースの抽象化、クラウド、サーバーレス
    - DBaaSを採用する企業が増えている
    - MariaDBによる[調査](https://www.businesswire.com/news/home/20220210005498/en/The-All-Aboard-Trip-to-the-Cloud-100-Move-to-Cloud-Databases-Survey-Shows-Big-Promise-and-Surprising-Disagreement)では、開発者/運用者の61％が、DBaaSへの完全移行を完了しているか、完了しようとしている
- **DBaaSの拡大する市場規模**
    - 2025年までに[248億ドル規模](https://www.marketsandmarkets.com/Market-Reports/cloud-database-as-a-service-dbaas-market-1112.html)へ
- **DBaaS への豊富な投資環境**
    - [SingleStore](https://www.singlestore.com/) が [3000万ドルを調達](https://techcrunch.com/2022/10/04/singlestore-raises-30m-more-to-brings-its-database-tech-to-new-customers/) (2022/Oct)
    - [EdgeDB](https://www.edgedb.com/) が [1500万ドルを調達](https://techcrunch.com/2022/11/07/edgedb-raises-15m-ahead-of-the-launch-of-its-cloud-database-service/) (2022/Nov)
    - [SurrealDB](https://surrealdb.com/) が [600万ドルを調達](https://mobile.twitter.com/tobiemh) (2023/Jan)

# ****SurrealDBの注目度合い****

- Hacke’s News トップページ掲載
    - [No.4 2022/Aug](https://news.ycombinator.com/item?id=32550543)
    - [No.2 2022/Sep](https://surrealdb.com/blog/2-on-hacker-news)
- GitHub 注目レポジトリにランクイン
    - [2022/Aug](https://surrealdb.com/blog/no-1-github-trending-repository)
    - [2022/Dec](https://surrealdb.com/blog/honoured-to-be-a-trending-public-repo-once-again-on-github-worldwide)
- GitHub Star
    - 180 star から 1500 star になるまで[48時間](https://surrealdb.com/blog/1500-thank-yous)
    - 5,000 star を[3週間で達成](https://surrealdb.com/blog/5000-thank-yous)
    - 10,000 star を[4週間で達成](https://surrealdb.com/blog/10-000-thank-yous)
- Reddit のProggraming と　Rust セックションの[Hotリストで No.1](https://surrealdb.com/blog/no-1-github-trending-repository)

# ****Server のオプションと環境変数****

- **DB_PATH**: データの格納先 (memory)
- **USER** ROOT認証のユーザ名 (root)、--user/-u
- **PASS** ROOT認証のユーザに対してのパスワード、--pass/-p
- **ADDR** ROOT認証を許可するサブネット (127.0.0.1/32),、--addr
- **BIND** コネクションを待ち受けるホスト名/IPアドレス(0.0.0.0:8000)、--bind/-b
- **KEY** ON-DISK暗号化のための秘密鍵、--key/-k
- **KVS_CA** KVS接続のためのCAファイル、--kvs-ca/
- **KVS_CRT** KVS接続のためのCERTファイル、--kvs-crt
- **KVS_KEY** KVS接続のための秘密鍵、--kvs-key
- **WEB_CRT** SSL接続で待ち受けるためのCERTファイル,--web-crt
- **WEB_KEY** SSL接続で待ち受けるための秘密鍵、--web-key
- **STRICT** 設定されていればSTRICTモードで起動、--strict
- **LOG** ログレベル　"warn", ["info"], "debug", "trace", "full"、--log

# ****ACID：トランザクションを定義する4つの特性****

- **Atomicity（原子性）**
    - ランザクションの各ステートメントは1 つの単位として扱われる
- **Consistency（一貫性）**
    - トランザクションがテーブルに、事前定義された予測可能な方法でのみ変更を加えることを保証
- **Isolation（独立性）**
    - 複数のユーザーが同じテーブルで読み書きを同時に実行しても、各要求は単独で発生しているように扱われる
- **Durability（永続性**）
    - システム障害が発生した場合でも、正常に実行されたトランザクションによるデータの変更が保存されることを保証

# N+1問題

- データベースアクセスでクエリが合計 N+1 回実行されてしまう問題
    - SELECT を 1 回実行し、N レコードを取得
    - Nレコードに関連するデータを取得するSELECT を N 回実行
- ORMを使っているときに裏側で発生しやすい
- アプリケーションの動作が重く（遅く）なる原因になりやすい

# RocksDB

- 人気の高い高性能な組み込み型KVSデータベース
- Meta(Facebook)社が開発したLevelDBのフォーク
- Facebook、Yahoo!、LinkedInなど、様々なWebサービスのプロダクションで使用されている
- データの永続化の実現と同時に、性能と安全性を高めている
- CPUの数が多ければ、性能が線形に増加する
- RocksDBの性能は、そのチューニングに大きく左右される
- プラットフォームのチューニングに強く影響する
    - 設定可能なパラメータが多く複雑なため簡単ではないTi

# TiKV

- TiDBのバックエンドで動作するKVSデータベース
- データの永続化(with RocksDB)
- 分散型データベースのデータ整合性の保証
- MVCC(Multi-Version Concurrency Control)
- 分散トランザクションの実現
- Google Parcorator / 2PC(2 Phase Commit)
- Coprocessor
- TiKVはデフォルトで3ノードクラスタ構成なので注意
    - 3ノードで構成されるまでは一時ファイルに格納
    - 一時ファイルは起動時に自動的に消去される
    - 参考： [https://docs.pingcap.com/tidb/dev/tikv-configuration-file](https://docs.pingcap.com/tidb/dev/tikv-configuration-file)
    - クラスタ構成： [https://tikv.org/docs/5.1/deploy/install/production/](https://tikv.org/docs/5.1/deploy/install/production/)

# ****TiKVのアーキテクチャ****

![tikv_stack.png](https://github.com/iisaka51/KansaiDB/blob/main/SurrealDB_Intro/tikv_stack.png)

# IndexedDB

- ブラウザベースの組み込み型KVSデータベース
    - ユーザーのブラウザー内にデータを永続的に保存する
    - ネットワークの状態にかかわらず高度なクエリ機能を持つWebアプリを作成できる
- スキーマーレス
- ACIDトランザクションをサポート
- 非同期処理
- マルチバージョン並行処理
- Cookie消去する程度の気軽さでユーザがデータを削除できてしまう
- ブラウザを選ぶ
- 一定容量を超えると、IndexedDBにデータ登録できなくなる
- IndexedDBに登録できるデータ量は、環境によって変化する

# FoundationDB

- ACIDトランザクションをサポートするNoSQL
- SQLで操作可能
- キーはソートされる
- SSDを使う場合で、1コアで20,000書き込み/秒のスループット
- 500コアまでリニアにスケール
- 読み込みは1ms、書き込みは5ms
- クラスタ構成で分散/冗長化ができるようになる
    - 最低1ノードで構成可能（冗長化はない/あとからノード追加可能）
- FDBのデフォルトではうまくないので要設定
    
    ```python
    $ fdbcli –exec ‘configure ssd’
    $ fdbcli --exec 'writemode on'
    $ fdbcli --exec 'getrangekeys \x00 \xff'
    ```
    
    - 参考: [https://apple.github.io/foundationdb/command-line-interface.html](https://apple.github.io/foundationdb/command-line-interface.html)

# ****バイナリリリースはFoundationDBが無効****

- FoundationDBのバージョンに依存するため features で指定されている

```bash
$ cargo feature surreal
   Avaliable features for `surreal`
default = ["storage-rocksdb", "scripting", "http"]
http = ["surrealdb/http"]
scripting = ["surrealdb/scripting"]
storage-fdb = ["surrealdb/kv-fdb-6_3"]
storage-rocksdb = ["surrealdb/kv-rocksdb"]
storage-tikv = ["surrealdb/kv-tikv"]

$ grep kv-fdb- lib/Cargo.toml
kv-fdb-5_1 = ["foundationdb/fdb-5_1", "kv-fdb"]
kv-fdb-5_2 = ["foundationdb/fdb-5_2", "kv-fdb"]
kv-fdb-6_0 = ["foundationdb/fdb-6_0", "kv-fdb"]
kv-fdb-6_1 = ["foundationdb/fdb-6_1", "kv-fdb"]
kv-fdb-6_2 = ["foundationdb/fdb-6_2", "kv-fdb"]
kv-fdb-6_3 = ["foundationdb/fdb-6_3", "kv-fdb"]
kv-fdb-7_0 = ["foundationdb/fdb-7_0", "kv-fdb"]
kv-fdb-7_1 = ["foundationdb/fdb-7_1", "kv-fdb"]
```

# SurrealDBのリビルド

- rust の開発環境を整える (必要があれば）

```bash
$ curl -sSf https://sh.rustup.rs | sh 
$ source $HOME/.cargo/env
$ rustup install stable
```

- SurrealDBのレポジトリをクローン

```bash
$ git clone https://github.com/surrealdb/surrealdb.git
$ cd surrealdb
```

- リビルド　（約80分）

```bash
$ carrgo build –release –all-features        # 2GBメモリだと失敗する
# もしくは
$ cargo build –release –features storage-fdb # TiKVは無効になる
```

# ****TiKV vs FoundationDB****

- FDBのモニタリング機能は弱い
    - fdbcli --exec 'status json'
- TiKVにはPrometheusでデータ参照、Grafanaでステータスモニタリング
- FDBはバージョン・センシティブ
- FDBはデフォルトではメモリに格納するため設定変更が必要
- FDBはC++、TiKVはRustで実装
- FDBは最低1ノードでサービスできるが、TiKVは最低3ノードが必要

# ****RocksDBをレプリケーション****

- rocksplicator を使うとリアルタイムレプリケーションができる
    - [https://github.com/pinterest/rocksplicator](https://github.com/pinterest/rocksplicator)
- ただし、Rocksplicatorは、Pinterestによって積極的に保守・サポートされていない、アーカイブされたプロジェクトであることに注意

# ****バイナリファイルを格納することについて****

- SurrealDBはそうした用途向けに設計されていない
- オブジェクトストレージを検討するべき

[GitHub - minio/minio: Multi-Cloud Object Storage](https://github.com/minio/minio)

[GitHub - scality/Zenko: Zenko is the open source multi-cloud data controller: own and keep control of your data on any cloud.](https://github.com/scality/Zenko)

# ****余談）SQLの発音について****

SQL は日本では単純にエス・キュー・エルと発音されることが多いのですが、欧米ではほとんどの場合、シークェル（ 'sequel' ）と発音されます。

これは、SQLが1970年代はじめにIBM社によって開発されたとき、“SEQUEL (Structured English Query Language)”と名前を定義したことによります。

その後、名前が "SQL (Structured Query Language)" に変えられたのですが、もとの発音が主流のまま使われ続けています。

参考： [S.Q.L or Sequel: How to Pronounce SQL?](https://www.vertabelo.com/blog/sql-or-sequel/)

# SurQL構文

## INFO

```sql
INFO FOR [
	KV
	| NS | NAMESPACE
	| DB | DATABASE
	| SCOPE @scope
	| TABLE @table
];
```

## ****DEFINE  NAMESPACE | DATABASE …****

```sql
DEFINE [
	NAMESPACE @name
	| DATABASE @name
	| LOGIN @name ON [ NAMESPACE | DATABASE ] 
            [ PASSWORD @pass | PASSHASH @hash ]
	| TOKEN @name ON [ NAMESPACE | DATABASE ] 
            TYPE @algorithm VALUE @value
	| SCOPE @name [ SESSION @duration ]
            [ SIGNUP @expression ] [ SIGNIN @expression ] 
	| EVENT @name ON [ TABLE ] @table 
            WHEN @expression THEN @expression
```

<aside>
💡 @algorithm

EDDSA, ES256, ES384, ES512, HS256, HS384, HS512, PS256, PS384, PS512, 

RS256, RS384, RS51

</aside>

## ****DEFINE  TABLE****

```sql
DEFINE [
	| TABLE @name
		[ DROP ]
		[ SCHEMAFULL | SCHEMALESS ]
		[ AS SELECT @projections
			FROM @tables
			[ WHERE @condition ]
			[ GROUP [ BY ] @groups ]
		]
		[ PERMISSIONS [ NONE | FULL
			| FOR select @expression
			| FOR create @expression
			| FOR update @expression
			| FOR delete @expression
		] ]
;
```

## ****DEFINE  FIELD | INDEX****

```sql
DEFINE [
	| FIELD @name ON [ TABLE ] @table
		[ TYPE @type ]
		[ VALUE @expression ]
		[ ASSERT @expression ]
		[ PERMISSIONS [ NONE | FULL
			| FOR select @expression
			| FOR create @expression
			| FOR update @expression
			| FOR delete @expression
		] ]
	| INDEX @name ON [ TABLE ] @table [ FIELDS | COLUMNS ] @fields [ UNIQUE ]
] ;
```

## CREATE

```sql
CREATE @targets
	[ CONTENT @value
	  | SET @field = @value ...
	]
	[ RETURN [ NONE | BEFORE | AFTER | DIFF | @projections ... ]
	[ TIMEOUT @duration ]
	[ PARALLEL ]
;
```

## REMOVE

```sql
REMOVE [
	NAMESPACE @name
	| DATABASE @name
	| LOGIN @name ON [ NAMESPACE | DATABASE ]
	| TOKEN @name ON [ NAMESPACE | DATABASE ]
	| SCOPE @name
	| TABLE @name
	| EVENT @name ON [ TABLE ] @table
	| FIELD @name ON [ TABLE ] @table
	| INDEX @name ON [ TABLE ] @table
] ;
```

## INSERT

```sql
INSERT [ IGNORE ] INTO @what
	[ @value
	  | (@fields) VALUES (@values)
		[ ON DUPLICATE KEY UPDATE @field = @value ... ]
	]
;
```

## UPDATE

```sql
UPDATE @targets
	[ CONTENT @value
	  | MERGE @value
	  | PATCH @value
	  | SET @field = @value ...
	]
	[ WHERE @condition ]
	[ RETURN [ NONE | BEFORE | AFTER | DIFF | @projections ... ]
	[ TIMEOUT @duration ]
	[ PARALLEL ]
;
```

## DELETE

```sql
DELETE @targets
	[ WHERE @condition ]
	[ RETURN [ NONE | BEFORE | AFTER | DIFF | @projections ... ]
	[ TIMEOUT @duration ]
	[ PARALLEL ]
;
```

## SELECT

```sql
SELECT @projections
	FROM @targets
	[ WHERE @condition ]
	[ SPLIT [ AT ] @field ... ]
	[ GROUP [ BY ] @field ... ]
	[ ORDER [ BY ]
		@field [RAND()| COLLATE| NUMERIC ] [ ASC | DESC ] ...
	]
	[ LIMIT [ BY ] @limit ]
	[ START [ AT ] @start ]
	[ FETCH @field ... ]
	[ TIMEOUT @duration ]
	[ PARALLEL ]
;
```

## USE

```sql
USE 
  [ NAMESPACE | NS ] @namespace 
  [[ DATABASE | DB ] @database ] ;
```
