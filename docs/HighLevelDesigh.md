# 楽曲・アルバム管理システム 基本設計書
1. システム概要
本システムは、Spring Boot、MyBatis、Thymeleaf、H2 Databaseを用いて構築される。 「アルバム」という親エンティティに対し、複数の「楽曲」を紐付けて管理する1対多のリレーションシップを基本構造とする。

2. 画面遷移設計
ユーザーが直感的にアルバムの管理と、それに付随する楽曲の管理を行える動線を定義する。

graph TD
    Index[アルバム一覧画面 /albums] -- "詳細ボタン" --> Detail[アルバム詳細画面 /albums/{id}]
    Index -- "新規作成ボタン" --> AlbumForm[アルバム登録画面 /albums/new]
    
    Detail -- "編集ボタン" --> AlbumEdit[アルバム編集画面 /albums/{id}/edit]
    Detail -- "曲を追加ボタン" --> SongForm[楽曲登録画面 /albums/{id}/songs/new]
    Detail -- "一覧へ戻る" --> Index
    
    AlbumForm -- "保存(POST)" --> Index
    AlbumEdit -- "更新(POST)" --> Detail
    SongForm -- "保存(POST)" --> Detail

    
    3. データモデル設計（ER図）
データベースの物理構造およびエンティティ間のリレーションを定義する。


4. 外部インターフェース設計（エンドポイント）
ブラウザからの要求（HTTP Request）に対して、サーバがどのように応答するかを定義する。

4.1 通信シーケンスの例
アルバム詳細画面を開いた際の内部処理フロー：
sequenceDiagram
    participant B as ブラウザ (User)
    participant C as Controller
    participant M as Mapper (MyBatis)
    participant D as DB (H2)

    B->>C: GET /albums/1
    C->>M: getAlbumById(1)
    M->>D: SELECT * FROM albums WHERE id = 1
    D-->>M: Albumデータ返却
    C->>M: getSongsByAlbumId(1)
    M->>D: SELECT * FROM songs WHERE album_id = 1
    D-->>M: Songリスト返却
    C-->>B: Thymeleafにより結合されたHTMLを返却

    4.2 主要エンドポイント一覧

    機能,メソッド,パス,備考
アルバム一覧表示,GET,/albums,全アルバム取得
アルバム詳細表示,GET,/albums/{id},アルバム情報＋楽曲リスト表示
アルバム登録実行,POST,/albums,新規保存処理
楽曲登録実行,POST,/albums/{id}/songs,特定アルバムへの曲追加
楽曲削除実行,POST,/songs/{id}/delete,完了後、アルバム詳細へリダイレクト

5. クラス構成（パッケージ構成）
com.example.mugic_management 配下の構成案。

entity: Album.java, Song.java (Lombok利用)

controller: AlbumController.java, SongController.java

mapper: AlbumMapper.java, SongMapper.java (MyBatisインターフェース)

resources:

mapper/*.xml: SQL定義

templates/*.html: Thymeleafテンプレート

schema.sql: テーブル作成用SQL

6. 実装上の留意点
カスケード削除: アルバムを削除した際、紐づく楽曲を同時に削除するか、削除不可にするかのロジックをサービス層またはDB制約で実装する。

H2 Console: 開発中は localhost:8080/h2-console でデータの状態を直接確認できるように設定する。

バリデーション: アルバムタイトルや曲名が空文字のまま保存されないよう、@NotBlank 等のアノテーションを活用する。