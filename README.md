# sample-mysql-fulltext-search

MySQLによる全文検索を試す

＜一旦の結論＞

- データ件数が少なければLIKE検索でもいける
- 全文検索は転置インデックスを使っているので、ngramでトークン化したキーワードよりも長いキーワードでの検索だと遅くなる
- Mroongaがよさそうだけど、RDSには組み込めない。。。

## MySQLデータベース単体での検証

### DBコンテナ起動

```bash
docker compose up -d
```

### 検証用テーブル作成＆サンプルデータ登録

```sql
CREATE TABLE articles (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255),
  content TEXT,
  FULLTEXT (content) WITH PARSER ngram
) ENGINE=InnoDB;
```

```sql
-- 初期データを挿入（20件）
INSERT INTO articles (title, content)
VALUES
('MySQLの基本', 'MySQLは人気のあるオープンソースのリレーショナルデータベース管理システムです。'),
('全文検索入門', '全文検索は、大量のテキストデータから特定の単語や文章を素早く検索する技術です。'),
('パフォーマンスチューニング', 'データベースのパフォーマンスを向上させるには、インデックスの適切な使用が重要です。'),
('セキュリティ対策', 'データベースのセキュリティを確保するには、適切なアクセス制御と暗号化が必要です。'),
('バックアップと復元', '定期的なバックアップは、データ損失を防ぐための重要な対策です。'),
('SQLの基礎', 'SQLは構造化クエリ言語の略で、データベースの操作に使用されます。'),
('インデックスの最適化', 'インデックスの適切な設計は、クエリのパフォーマンスを大幅に向上させます。'),
('トランザクション管理', 'トランザクションは、データの一貫性を保つための重要な機能です。'),
('レプリケーション', 'レプリケーションはデータの冗長性と可用性を高めるための技術です。'),
('ストアドプロシージャ', 'ストアドプロシージャは、データベース内で実行される事前にコンパイルされたSQL文のセットです。'),
('NoSQLデータベース', 'NoSQLデータベースは、非リレーショナルデータモデルを使用するデータベースシステムです。'),
('データウェアハウス', 'データウェアハウスは、大規模なデータ分析のための中央リポジトリです。'),
('クエリオプティマイザ', 'クエリオプティマイザは、SQLクエリの最適な実行計画を決定します。'),
('データモデリング', '効果的なデータモデリングは、データベース設計の基礎となります。'),
('キャッシュ戦略', '適切なキャッシュ戦略は、データベースのパフォーマンスを大幅に向上させます。'),
('データ正規化', 'データ正規化は、データの冗長性を減らし、一貫性を向上させるプロセスです。'),
('ビューの活用', 'ビューは、複雑なクエリを簡略化し、データアクセスを抽象化するのに役立ちます。'),
('パーティショニング', 'テーブルパーティショニングは、大規模データベースの管理を容易にします。'),
('データ暗号化', 'データ暗号化は、機密情報を保護するための重要なセキュリティ対策です。'),
('データベース監査', 'データベース監査は、セキュリティとコンプライアンスを確保するために不可欠です。');

-- データを倍々に増やす（16回繰り返して約130万件に）
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
INSERT INTO articles (title, content) SELECT CONCAT(title, ' ', FLOOR(1 + RAND() * 1000000)), CONCAT(content, ' ', CASE WHEN RAND() < 0.3 THEN '全文検索は効率的です。' WHEN RAND() < 0.6 THEN 'パフォーマンスの向上が期待できます。' ELSE 'データベース管理は重要です。' END) FROM articles;
```

### 実行検証

#### LIKE検索

```sql
SET profiling = 1;
SELECT * FROM articles WHERE content LIKE '%全文検索%';
SHOW PROFILES;
```

#### 全文検索

```sql
SET profiling = 1;
SELECT * FROM articles WHERE MATCH(content) AGAINST('全文検索' IN BOOLEAN MODE);
SHOW PROFILES;
```
