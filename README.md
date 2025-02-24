# eGov-parser

## 思想
[法を分かりやすく使いやすいものにするAI駆動型リーガルテックの可能性 – 次世代知能科学研究センター](https://www.ai.u-tokyo.ac.jp/ja/activities/act-archive/act-20230313)

**Law/Rule as Code = 法律をプログラム可能なデータとして扱う考え方**
- **機械可読な法令で、リーガルチェックやコンプライアンスを自動化** **
- 法改正の影響をシミュレーションできる

### デジタル法制ロードマップ（デジタル庁参事官補佐 山内 匠 氏）
- **フェーズ0（現状）：** 人間による手作業に依存する業務が混在。法文書の機械分析性も低い。
- **フェーズ1（法令ベースレジストリ）：** 法令ベースレジストリを構築して法文書の機械分析性を向上させ、データの相互参照・相互連携を可能とする。
- **フェーズ2（コネクテッドデータ）：** 法文書間の関係がデータ化されて、統一的APIで容易に相互連携可能。法文書に関連したガイドライン等が迅速に閲覧できる。
- **フェーズ3（法令オントロジ）：** 法令用語の意義・論理関係などの意味論的情報が整備される。自然言語処理による法令の分析が可能となる。
- **フェーズ4（法令静的分析）：** 法令の論理構造がデータ化されることで、法令の矛盾等が自動的に特定できるようになる。
- **フェーズ5（制度デジタルツイン）：** 法令を仮想空間上でシミュレーション可能となる。シミュレーション結果から法令の効果を自動分析できるようになる。

Ph. 5 まで一気に駆け上がる！！ Ph4, 5の違いは
- Ph.4 : 制約充足問題 (CSP) / **「法令の無矛盾性を判定」**
- Ph.5 : モデル拡充 (ME) / **「法令の論理関係を推論」** （法律の抜け穴発見するような仕組み）

### 法改正の現場の問題点
[法令文書とバージョン管理 - 法令データ ドキュメンテーション（α版）](https://laws.e-gov.go.jp/docs/docs/ba4d819-laws-and-version-control/)

→ ソフトウェア開発現場同様に，**git とCI自動テストの導入**しかないですね！

### その他

民事判決の情報DBはまだ
[法務省：民事判決情報データベース化検討会](https://www.moj.go.jp/shingi1/shingi09900001_00004.html)

ニュージランドの取り組み
[Resources | Better Rules – Better Outcomes](https://www.betterrules.govt.nz/resource)

個人開発の政治可視化系サイト
[政治が分かるように、国会のサイトを分かりやすくしちゃうプロジェクト！ - YouTube](https://www.youtube.com/watch?v=MBjrRQnn_fQ)
[誰に投票する？ | 選挙で投票する人を選ぶときの参考サイト](https://sstjp.com/)


## DB要件
- グラフDB（Neo4j）
	- [Cypherならできる！SQLには難しいこと10選（前編） #neo4j #cypher #sql - クリエーションライン株式会社](https://www.creationline.com/tech-blog/data-management/neo4j/51530)
	- [GraphDBを学ぶ（１）neo4jのコンテナ稼働 #Docker - Qiita](https://qiita.com/myoshimi/items/83ff5541518e482c4044)
	- [Neo4jをはじめよう！グラフデータベースで最初のクエリを書くまで #neo4j - Qiita](https://qiita.com/toshiki19/items/1c84a4f49e24106ad756)
	- マイグレーションはCSVを出力するらしい [Neo4jのアップグレードとデータ移行まとめ #neo4j - Qiita](https://qiita.com/blue_islands/items/dd1fa93bae23a980640e)
- 裁判所の過去の判例PDFをLLMでパース，DB管理
- LLMによる曖昧検索
	- [Knowledge Graphを使った RAG をLangChainで実装\[前編\] #LLM - Qiita](https://qiita.com/FukuharaYohei/items/6f1d094dc33688711221)
	- [Microsoftから提供されたGraphRAGを使ってみました](https://zenn.dev/headwaters/articles/a8e08ff7314933)
	- [Langchain+Neo4j で「GraphRAG」を実装してみる](https://www.chowagiken.co.jp/blog/graph_rag)

## データフロー
1. 生XMLをAPIから取得 [Home - 法令データ ドキュメンテーション（α版）](https://laws.e-gov.go.jp/docs) [法令API仕様書](https://laws.e-gov.go.jp/file/houreiapi_shiyosyo.pdf)
	1. 交付年月日 PromulgationDate によるバージョニング
	2. 最新の現行法令しかAPIからは取得できない
	3. 法令一覧API → 法令取得API ループ
2. XMLをとりあえずどこかにテキストとしてDB保存（RDBMSでOK）
3. XMLを各条文に分割し，ノード作成．言及している法令リンク（文脈依存なのでちょっと難しい）をエッジとして，CYPHERクエリに変換するロジックを書く
4. Neo4jに格納する
5. Neo4jにクエリして，Markdownに変換するロジックを書く
6. Markdown をgithub pushする
7. Neo4jにクエリして，IDP-Z3 に変換するロジックを書く
8. 上記の自動テスト走らせる

## バックエンド（クラウド，バッチ，自動テスト）
- 官公庁向けなので国産クラウド [VPS（仮想専用サーバー）｜さくらインターネット](https://vps.sakura.ad.jp/) [クラウドサーバーはIaaS型のさくらのクラウド](https://cloud.sakura.ad.jp/)
- eGov 法令API を利用してXML取得
- 裁判所の判例PDFをLLMでパースするバッチ
- 自動テスト
	- Neo4j にクエリする
	- クエリ結果からMarkdown形式に変換する
	- クエリ結果からIDP-Z3形式に変換する（論理関係推定）
		- モデル拡充 [FO(.) - Wikipedia](https://en.wikipedia.org/wiki/FO%28.%29) [The IDP Language — IDP-Z3  documentation](https://docs.idp-z3.be/en/0.5.3/IDPLanguage.html)
- eGov-Viewer でも公開APIを用意する
	- 関連法令検索機能


## フロントエンド

既にeGov法令検索で実装されているもの
- 相互リンク
- 他法令リンク
- 法令新旧比較
- 

### Web （リッチ版）
- 2chブラウザのようにリンク先を多段ポップアップ https://chatgpt.com/share/67ba7712-5e6c-8002-b9f4-52266df3d071
- ページ内法令リンク
- 括弧内ハイライト
- ページ内ジャンプ（テンキー入力でジャンプ）
- 附則リンク対応
- （同一法令でない）参照条文リンク機能
- cssで好きなスキン対応
- VivliostyleによるCSS組版．印刷でもレイアウト崩さず．[CSSフレームワークVivliostyle Themeで簡単にページデザインを編集する | gihyo.jp](https://gihyo.jp/article/2024/04/vivliostyle-03)
- ある法案からネストした関連法令だけをプリントアウト
- eLen（全国条例データベース）での機能集 [eLen](https://elensv.e-legislation.jp/project/elen/?page=2)（Safariからアクセスする） eLen は一般公開していない（なぜ？？）
	- 全国市町村別，条例・規則別検索
	- 類似グループ表示（？）
	- 比較表表示
	- 文脈表示（？）

### Markdown＋Github pages （官公庁編集者向け）
- 相互参照
	- MarkdownファイルによってVSCode上で相互参照を可能に
	- VSCodeでは #~~ にF12 でジャンプ，Ctrl - で戻れる．Find All References も効く
	- **相互リンクで行き来しすい環境構築*
- Markdown をVSCode系ツールで直接ブラウザ上から編集
	- [https://github.com/coder/code-server](https://github.com/coder/code-server)
	- [https://github.com/hackmdio/codimd](https://github.com/hackmdio/codimd)
- Githubレポジトリで保管
- github pages 管理．[GitHub PagesとJekyllでMarkdownを静的サイト公開](https://zenn.dev/sasakiki/articles/e4d5dd28700b16)
- Component で Markdownを扱う [GitHub - mdx-js/mdx: Markdown for the component era](https://github.com/mdx-js/mdx)
