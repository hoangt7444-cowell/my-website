# Query quản lý pages

Các câu lệnh dưới đây dùng để reset trạng thái crawl và thêm các page mẫu vào bảng `pages`.

> Kiểm tra lại `site_id` trước khi chạy query trong môi trường thật.

## 1. Reset trạng thái crawl

Reset thông tin crawl và hash nội dung của toàn bộ page:

```sql
UPDATE pages
SET
  last_crawled_at = NULL,
  last_new_text_stored_at = NULL,
  last_content_hash = NULL;
```

Chỉ reset thời điểm crawl gần nhất:

```sql
UPDATE pages
SET last_crawled_at = NULL;
```

## 2. Thêm danh sách page test-dom

Thêm các page kiểm thử DOM Processor cho `site_id = 1`. Những URL đã tồn tại sẽ được bỏ qua.

```sql
WITH params AS (
  SELECT 1::bigint AS site_id
),
page_list (url, name) AS (
  VALUES
    ('https://lanlt6782.github.io/my-website/test-dom/01-basic-structure', '基本構造テスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/02-inline-markup', 'インライン要素テスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/03-attributes', '属性収集テスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/04-head-metadata', '収集対象ページのタイトル'),
    ('https://lanlt6782.github.io/my-website/test-dom/05-ignored-content', '収集対象外コンテンツのテスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/06-links-and-empty-nodes', 'リンクと空ノードのテスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/07-css-phrasing', 'CSSフレージングテスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/08-unicode-and-filters', 'Unicodeとフィルターのテスト'),
    ('https://lanlt6782.github.io/my-website/test-dom/09-mixed-all-cases', '全ケース総合テスト')
),
created_pages AS (
  INSERT INTO pages (
    site_id,
    url,
    name,
    setup_state,
    alive,
    ignore_param,
    touched,
    match_regex,
    hosted,
    created_at,
    updated_at
  )
  SELECT
    params.site_id,
    page_list.url,
    page_list.name,
    'initial',
    TRUE,
    FALSE,
    FALSE,
    '',
    FALSE,
    CURRENT_TIMESTAMP,
    CURRENT_TIMESTAMP
  FROM params
  CROSS JOIN page_list
  WHERE NOT EXISTS (
    SELECT 1
    FROM pages
    WHERE pages.site_id = params.site_id
      AND pages.url = page_list.url
  )
  RETURNING id, site_id
)
INSERT INTO page_to_langs (
  page_id,
  lang,
  enabled,
  created_at,
  updated_at
)
SELECT
  created_pages.id,
  site_to_langs.lang,
  TRUE,
  CURRENT_TIMESTAMP,
  CURRENT_TIMESTAMP
FROM created_pages
INNER JOIN site_to_langs
  ON site_to_langs.site_id = created_pages.site_id
ON CONFLICT (page_id, lang)
DO UPDATE SET
  enabled = TRUE,
  updated_at = CURRENT_TIMESTAMP;
```

## 3. Thêm page theo khoảng số

Sinh và thêm các page từ `Page 1` đến `Page 4` cho `site_id = 1`.

```sql
WITH params AS (
  SELECT
    123::bigint AS site_id,
    2::integer AS page_count
),
page_list AS (
  SELECT
    params.site_id,
    generate_series(1, params.page_count) AS page_number
  FROM params
),
created_pages AS (
  INSERT INTO pages (
    site_id,
    url,
    name,
    setup_state,
    alive,
    ignore_param,
    touched,
    match_regex,
    hosted,
    created_at,
    updated_at
  )
  SELECT
    site_id,
    format('https://lanlt6782.github.io/my-website/auto-crawl/index?page=%s', page_number),
    format('ページ %s', page_number),
    'initial',
    TRUE,
    FALSE,
    FALSE,
    '',
    FALSE,
    CURRENT_TIMESTAMP,
    CURRENT_TIMESTAMP
  FROM page_list
  RETURNING id, site_id
)
INSERT INTO page_to_langs (
  page_id,
  lang,
  enabled,
  created_at,
  updated_at
)
SELECT
  created_pages.id,
  site_to_langs.lang,
  TRUE,
  CURRENT_TIMESTAMP,
  CURRENT_TIMESTAMP
FROM created_pages
INNER JOIN site_to_langs
  ON site_to_langs.site_id = created_pages.site_id;
```