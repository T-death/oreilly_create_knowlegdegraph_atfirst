# はじめての知識グラフ構築ガイド

# 第1章
* 知識グラフには、利用者が潜在的をデータ推論できるようにするために、構成原則を設けておく必要がある

# 第2章
* 古典的なグラフ
    * 一瞥するだけでグラフの意味が理解できるため、構成原則が隠蔽されていると解釈できる
    * グラフの構成原則が把握できていないと、他者に理解されにくい可能性がある
        * 作業を引き継ぐときや、リバースエンジニアリングをするときに弊害になってしまう
* プロパティグラフ
    * ラベル付きノードやリレーション型を持っている
    * それぞれのノードがkey:value型のデータを保持でき、有向グラフを構築できる
* 「aはbの一種である」という推論を実施するには、`タクソノミー`と呼ばれる構造を利用する
    * `タクソノミー` -> カテゴリを「広義/狭義」といった具合に、階層的に分類するスキーム
        * 類似する性質を持つものを同一のカテゴリーに分類し、他の分類との蘇澳語リレーション
        * 具体的なモノほど下層に来て、抽象的なモノほど上層に来る
        * `タクソノミー`は、異なるタクソノミー同士で簡単に単一のグラフにまとめることができる
        * タクソノミーでよく使われる類似度 -> `パス類似度 / Leacook-Chadorow類似度 / Wu-Palmer類似度`
          * あとで調べる
    * `オントロジー` -> タクソノミーと同様に階層的に分類することに加えて、多様な結合性を記述可能
        * タクソノミーに比べて、詳細にラベルを記述できるのがオントロジーってこと
        * ラベル情報の付与により、階層構造が垂直方向だけでなく、水平方向へリレーションを接続できる
        * ドメイン知識が異なるタクソノミー/オントロジー同士の橋渡し（=セマンティックブリッジ）として、大きなオントロジーにすることも可能
            * これによって多種多様なドメインを持つオントロジーが作れる！
    * 完璧なオントロジーを目指しすぎて、肥大化しまくったグラフができるのは良くない
        * 長期的な目的を持って構築することが望ましい
    * 学術分野や金融ビジネスなどといったドメイン分野でオントロジーが使われてる
    * 構成原則の作成
        1. 自然言語を利用してセマンティクスを作成する
            * 人間目線だと導入しやすいが、機械側が認識できないため、実装時に人的ミスが生じる可能性あり
        2. 初めから機械側が認識できる言語を使用する
            * `RDFスキーマ`、`Web Ontology Language(OWL)`、`Simple Knowledge Organization System（SKOS）`などなど
            * ただ自然言語に比べると習熟のハードルが難点
            * ちなみに、`Web Ontology Language`の略称は`OWL`で合ってる。なんで`W`と`O`逆なん？
        * RDFは、データの主語/述語/目的語のトリプルを持つデータ構造のこと
            * LLMでグラフDBを活用する際に、SVOのリレーションを持たせたグラフDBを見たことがあるけれど、たぶんそれのことか
# 第3章
* 知識グラフを開発するには、`Cypher`言語と呼ばれるクエリ言語を使うことが多い
* `Cypher`言語
    * `()` -> ノード
    * `-[]->` -> リレーションのラベル
    * `:xxx` -> ノード内のラベル
    * `{key: value}` -> ノード内に含まれているkey-valueプロパティ
    * 拡張子は`.cypher`
* `MERGE`句は、関連するグラフが存在しない場合は、`CREATE`句と同じ振る舞いをする
    * `MERGE`句は`MATCH`句と`CREATE`句の組み合わせのような挙動を起こす
    * パターン全体にマッチしないと、新規作成をしてしまう
        * 部分的なノードが一致していても、そのノードに結ばれることなく新規作成される
* `グラフローカルクエリ` <-> `グラフグローバルクエリ`
    * グラフの局所的な内容に関するクエリ <-> グラフ全体の内容に関するクエリ
* バインドは変数に入れるみたいな操作
* `グラフグローバルクエリ`において、ノードラベルの指定はしなくてもむしろパフォーマンスの最適化が可能だが、リレーションのラベルを指定しないと、探索時間が増える
* `APOC`ライブラリを使うことで、様々な処理（スキーマの確認や文字列結合など）を手軽に呼び出せる
* `EXPLAIN`句 -> クエリを実行することなく、実行計画を視認性高く出力する
* `PROFILE`句 -> クエリを実行し、その実行時の挙動を分析する
* Neo4jでは、ノードとリレーションが別々のメモリに格納されている
    * このとき、レコードIDをバイトサイズで掛けることで、対応する情報が得られる = `非索引型隣接（index-free adjacency）`
    * [このサイト](https://kenwagatsuma.com/ja/blog/index-free-adjacency-explained)がわかりやすい説明してくれてた
    * グラフDBであれば、インデックスがなくてもあらかじめ隣接する情報を持っている、ということ
    * しかも探索時間がO(1)である
* Neo4jは、プロパティの持ち方がノードやリレーションと若干異なる
    * 柔軟性を持たせるため
    * プロパティの読み取りはO(n)になり、書き込みはO(1)
        * ノードやリレーションと違って、プロパティは単にリストとしてデータを持っているため
        * 書き込む際には、ノードを辿った先に追加するだけなので、時間がかからない
* ナレッジグラフを構築する際には、なるべくノードとリレーションによる探索で1つのプロパティに辿り着くように設計した方が良い
    * 場合によっては、途中でプロパティの値を使う必要があるかもしれないけれど、計算時間的にはプロパティの参照は最後だけにするのが望ましい
* ACIDトランザクション
    * ログ先行書き込み（WAL）を採用しているため、万が一の場合でもログから復旧が可能
# 第4章
* `Data Importer`を使うと、GUIでcsvファイルに記述されている内容に基づいたグラフを作成してくれる
    * インポートしてきたファイルの内容から、自動でcypherコードを生成してくれる
* `MERGE`句を使うことで、重複したグラフの構築を避けることができる
    * もし既存のグラフに気づかずに`CREATE`句を実行してしまうことを懸念できる
* `LOAD CSV`句によって、GUIだけでなくクエリ言語としてcsvファイルのインポートも可能
    * ヘッダーの有無次第で、`LOAD CSV WITH HEADERS`を使う場合もある
    * 当然、csvファイルをNeo4j側にアップロードしておく必要があり、ローカルや外部からcsvファイルを使用する場合は、相応の作業が必要
* csvファイルでプロパティの値が省略されている場合は、`SET`句に記述する
    * 要するに、csvファイル内でデータが欠けている箇所があるとエラーになるので、SET句を使うってこと
* `CALL {...} IN TRANSACTIONS OF ... ROWS`を使用することで、大量のデータをバッチ処理できる
    * CALL関数の中に、通常のCypherコードを記述し、IN TRANSACTIONS OF 1 ROWSのようにCALL関数の外に記述すると、1行ずつバッチ処理される
* `neo4j-admin`コマンドは、高速にデータを読み込むことができる
    * だいたい100万レコード/秒くらいの速さ（DBがオフライン時）
    * S3バケットのような、ファイル形式でないデータはインポート不可
    * gzip圧縮されたcsvは対応可能
# 第5章
* データファブリック -> 組織横断型のデータアクセス層
    * 横断型の中でも、マスターデータ層として扱われる
    * データの持ち方がRDBだったり、NoSQLだったり様々な場面において、それらを全て統合したデータファブリックとしてアクセスされる
    * データファブリックにグラフ構造が使われるメリットとして、追加データの結合が容易だったり、事前設計が不要な点が挙げられる
        * RDBやkey-value型といった構造も、グラフ構造で再現できるからこそって感じかな
* Neo4jのブラウザで使用しているCypherコードは、直接DBを操作しているわけではなく、間にドライバが噛んでいて、そこからネットワークを経由して操作している
    * Neo4jのドライバとしてサポートしている言語には、Python / Go / JavaScriptなどがある
* フェデレーション -> 複数のグラフデータソースを統合して、それに対して単一のアクセスポイントを提供する技術
    * 統合するといっても、物理的にではなく仮想的に統合する
* Cypherコード実行時にプロシージャを利用することで、RDBからのデータ抽出が可能になる
    *  `APOC`を使用
    * ウェブURLからレスポンスされるjsonデータを読み込むことも可能
* `GraphQL` -> API構築のフレームワークやAPI呼び出しのための実行環境の一種
