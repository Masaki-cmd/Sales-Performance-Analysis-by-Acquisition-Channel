プロジェクトタイトル：ECサイトにおける流入経路別・売上改善分析
分析の目的
Google Merchandise Storeの1ヶ月分のログデータを分析し、広告・集客予算の最適化に向けた意思決定を支援する。

1. ビジネス課題と仮説

課題: 全体の訪問者数に対して購入率（CVR）が低い可能性がある。

仮説: 流入数が多いチャネル（YouTube等）において、ユーザーのニーズとサイトの提供価値にミスマッチが生じているのではないか。

2. 使用技術

データ基盤: Google BigQuery (Google Analytics 360 Sample Dataset)

言語: SQL

3. 分析プロセスと結果

ステップ1：チャネル別売上集計

結果：YouTubeからの流入は180名と多いが、売上は0ドルであった。一方で、メール経由の購入率は極めて高かった。

ステップ2：YouTubeユーザーの行動分析

結果：平均ページビュー（1.66）、平均滞在時間（190秒）。

洞察：ユーザーはページをじっくり読んでいるが、次のアクション（商品選択・購入）に繋がらず離脱している。

4. 具体的提案（Actionable Insights）

YouTube経由のユーザー向けに、動画内容と連動した専用ランディングページを構築し、導線を簡略化することを提案。

5. 使用したSQLクエリ

SELECT
  date,
  COUNT(fullVisitorId) AS user_count
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
GROUP BY
  date

行	date	user_count
1	20170801	2556
   
SELECT
  trafficSource.medium AS medium,
  COUNT(fullVisitorId) AS user_count
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170801' AND '20170831'
GROUP BY
  medium
ORDER BY
  user_count DESC
  行	medium	user_count
1	(none)	2166
2	referral	313
3	affiliate	52
4	cpm	15
5	organic	10

SELECT
  trafficSource.medium AS medium,
  COUNT(fullVisitorId) AS user_count,
  SUM(totals.transactions) AS total_orders,
  -- 100万で割っているのは、データがマイクロ単位で保存されているためです
  SUM(totals.totalTransactionRevenue) / 1000000 AS total_revenue
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170801' AND '20170831'
GROUP BY
  medium
ORDER BY
  total_revenue DESC
  行	medium	user_count	total_orders	total_revenue
1	(none)	2166	44	8872.04
2	referral	313	1	17.96
3	organic	10	null	null
4	cpm	15	null	null
5	affiliate	52	null	null

SELECT
  trafficSource.source AS source, -- 具体的なサイト名
  COUNT(fullVisitorId) AS user_count,
  SUM(totals.transactions) AS total_orders,
  SUM(totals.totalTransactionRevenue) / 1000000 AS total_revenue
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170801' AND '20170831'
  AND trafficSource.medium = 'referral' -- 紹介ルートだけに絞る
GROUP BY
  source
ORDER BY
  user_count DESC
  行	source	user_count	total_orders	total_revenue
1	youtube.com	180	null	null
2	analytics.google.com	57	null	null
3	google.com	12	null	null
4	sites.google.com	8	null	null
5	facebook.com	7	null	null
6	quora.com	6	null	null
7	qiita.com	5	null	null
8	m.facebook.com	5	null	null
9	groups.google.com	4	null	null
10	reddit.com	4	null	null
11	blog.golang.org	3	null	null
12	l.facebook.com	3	null	null
13	adwords.google.com	2	null	null
14	mail.google.com	2	1	17.96
15	lm.facebook.com	2	null	null
16	productforums.google.com	1	null	null
17	datastudio.google.com	1	null	null
18	sashihara.jp	1	null	null
19	m.baidu.com	1	null	null
20	ph.search.yahoo.com	1	null	null
21	support.google.com	1	null	null
22	google.com.vn	1	null	null
23	google.com.tw	1	null	null
24	int.search.tb.ask.com	1	null	null
25	docs.google.com	1	null	null
26	away.vk.com	1	null	null
27	pinterest.com	1	null	null
28	dealspotr.com	1	null	null

SELECT
  trafficSource.source AS source,
  AVG(totals.pageviews) AS avg_pageviews,
  AVG(totals.timeOnSite) AS avg_time_on_site
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170801' AND '20170831'
  AND trafficSource.source = 'youtube.com'
GROUP BY
  source
  行	source	avg_pageviews	avg_time_on_site
1	youtube.com	1.6611111111111108	190.95121951219511
