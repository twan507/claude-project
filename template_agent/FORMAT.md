# FORMAT — MD Contract Spec

Spec định nghĩa MD chuẩn hóa mà template_agent dùng làm input cho TEMPLATE pack render binary. File này độc lập, không reference WORKFLOW hay TEMPLATE.

## 1. Mục đích

Mỗi báo cáo render qua template_agent đều phải đi qua MD chuẩn hóa theo contract ở đây. Contract đảm bảo:

- TEMPLATE pack runtime parse được MD nhất quán
- Section heading match được với layout có sẵn
- Chart placeholder build được native chart từ data trong MD
- Citation render đúng format
- Locale vi-VN đồng nhất

MD không match contract → WORKFLOW Stage 4 normalize lại; nếu không thể normalize → reject, hỏi user.

## 2. Cấu trúc file MD

### 2.1. Frontmatter (bắt buộc)

Đầu file YAML frontmatter delimit bằng `---`:

```yaml
---
report_type: stock_pitch | weekly_market | market_scan | stock_memo | portfolio_plan | portfolio_review_weekly | portfolio_review_monthly | portfolio_review_quarterly | custom
publish_date: 2026-04-27
ticker: VNM                   # required nếu report_type cần ticker
company_name: Công ty Cổ phần Sữa Việt Nam   # required cùng ticker
industry: Hàng tiêu dùng       # required cùng ticker
language: vi                   # vi | en (default vi)
brand_suggestion: vbse         # optional — gợi ý brand từ source, user vẫn pick lại
---
```

**9 loại `report_type` cho phép:**

| report_type | Mô tả | Required field thêm |
|---|---|---|
| `stock_pitch` | Khuyến nghị MUA mã đơn lẻ gửi KH | `ticker`, `company_name`, `industry` |
| `weekly_market` | Báo cáo thị trường tuần | `week_label` (vd "Tuần 17/2026") |
| `market_scan` | Báo cáo scan đầu cycle đầu tư (top-down) | `scan_date`, `regime` |
| `stock_memo` | Memo deep-dive nội bộ 1 mã (conviction memo) | `ticker`, `company_name`, `industry`, `conviction_tier` (high/medium/low) |
| `portfolio_plan` | Kế hoạch portfolio + order list | `portfolio_size_vnd` |
| `portfolio_review_weekly` | Review portfolio tuần | `week_range` |
| `portfolio_review_monthly` | Review portfolio tháng | `month_year` (vd "04/2026") |
| `portfolio_review_quarterly` | Review portfolio quý + BCTC mới | `quarter_year` (vd "Q1/2026") |
| `custom` | Báo cáo không có template sẵn — agent quiz user xây spec runtime | `custom_spec_id` (timestamp), `custom_purpose` |

`report_type` không match 1 trong 9 → reject, normalize lại hoặc chuyển sang `custom`.

### 2.2. Heading hierarchy

```
# [TITLE BÁO CÁO]                  ← H1 duy nhất, là tiêu đề chính
## [SECTION 1]                     ← H2 cho mỗi section chính
### [SUBSECTION 1.1]               ← H3 nếu cần subsection
#### Avoid                         ← Không dùng H4+ trừ trường hợp đặc biệt
```

**Quy tắc:**
- H1 chỉ xuất hiện **1 lần** ở đầu file
- H2 cho mỗi section chính theo report_type spec (xem mục 3 dưới)
- H3 cho subsection (vd "3.1 Thông tin cơ bản" trong section "3. Hồ sơ doanh nghiệp")
- Không dùng H4+ — báo cáo không nên quá sâu

### 2.3. Block khuyến nghị (chỉ stock_pitch)

Ngay sau H1 + H2 tên công ty, có blockquote khuyến nghị:

```markdown
> **Khuyến nghị:** MUA
> **Giá hiện tại:** 33.000 đ
> **Mục tiêu trung hạn:** 39.000 đ
> **Tiềm năng tăng giá:** +18,2%
```

4 dòng cố định, đơn vị đồng/+%.

### 2.4. Body sections

Mỗi section H2 chứa:
- Prose ngắn (3-15 dòng), không filler
- Bảng MD khi data tabular
- Blockquote khi callout / disclaimer
- Chart YAML block khi cần visual chart (xem mục 4)

## 3. Section structure theo report_type

Mỗi report_type có structure cụ thể. Normalize phải map content vào đúng structure tương ứng.

### 3.1. stock_pitch — Khuyến nghị mua mã đơn lẻ (15 sections, rigid)

**Audience:** khách hàng. **Length:** 12-18 trang.

```
# BÁO CÁO PHÂN TÍCH [TICKER]
## [Tên đầy đủ công ty]
[Block khuyến nghị 4 dòng]

## 1. Tóm tắt khuyến nghị
## 2. Hồ sơ doanh nghiệp
## 3. Dữ liệu giao dịch phiên hiện tại
## 4. Luận điểm 01 — [Tiêu đề]
## 5. Luận điểm 02 — [Tiêu đề]
## 6. Luận điểm 03 — [Tiêu đề]
## 7. Luận điểm 04 — [Tiêu đề]
## 8. Luận điểm 05 — [Tiêu đề]   [optional]
## 9. Luận điểm 06 — [Tiêu đề]   [optional]
## 10. Luận điểm 07 — [Tiêu đề]  [optional]
## 11. Chiến lược thực thi — Cấu trúc mua N lớp
## 12. Mục tiêu chốt lãi
## 13. Bear Case — Phản biện và lo ngại còn yếu
## 14. Tóm tắt hành động + 3 kịch bản
## 15. Tuyên bố miễn trừ trách nhiệm
```

Số luận điểm flex 4-7. Section number adjust tương ứng.

**Ràng buộc nội dung:**
- Section 13 BẮT BUỘC có sub "Lo ngại còn yếu sau phản biện" (honest steelman, không "all win")
- Section 14 BẮT BUỘC có 3 kịch bản (cơ sở / tích cực / thận trọng), KHÔNG gán xác suất %
- Section 14 có note "kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn"
- Variant Perception lồng vào 1 luận điểm với 3 sub: consensus sell-side / consensus retail / thesis khác consensus

### 3.2. weekly_market — Báo cáo thị trường tuần (12 sections, rigid)

**Audience:** nội bộ + khách hàng. **Length:** 9-11 trang.

```
# BÁO CÁO THỊ TRƯỜNG TUẦN [TUẦN N/YYYY]

## 1. Tóm tắt điều hành
## 2. Review tuần trước
## 3. Bối cảnh quốc tế
## 4. Thị trường Việt Nam
## 5. Vĩ mô & hàng hoá
## 6. Biến động ngành
## 7. Top dẫn dắt
## 8. Tin tức & catalyst
## 9. Phân tích kỹ thuật VNINDEX + 3 kịch bản
## 10. Watchlist
## 11. Lịch sự kiện tuần tới
## 12. Tuyên bố miễn trừ trách nhiệm
```

**Ràng buộc:**
- KHÔNG dùng từ command (mua, bán, giảm tỷ trọng) — diễn đạt observation
- Sector bias dùng "quan tâm / thận trọng" thay "tập trung / tránh"
- Watchlist: mã đáng chú ý + sự kiện, không kèm level giá vào/ra/stop
- Section 9 ba kịch bản: if-then trigger objective, KHÔNG xác suất %
- KHÔNG dùng chỉ báo trend nội bộ (audience không có nền methodology để hiểu)

### 3.3. market_scan — Báo cáo scan đầu cycle đầu tư (7 sections, flex top-down)

**Audience:** nội bộ (analyst + sếp/team). **Length:** 8-15 trang tùy regime.

```
# Market Scan — [Ngày]

## 1. Executive summary (30 giây đọc)
## 2. Regime & vĩ mô
## 3. Catalysts active
## 4. Ngành shortlist
## 5. Top mã final (top 3/ngành)
## 6. Watchlist (Bucket 3, chưa vào)
## 7. Risks & cảnh báo
— Metadata
```

**Ràng buộc:**
- Section 1 là **hook chính** — block cứng có Regime + Cash buffer + Quota cycle + Top N mã + Catalyst dominant + Rủi ro lớn nhất, dài 0.5-1 trang
- Trường hợp `regime: dung_ngoai` → Section 1 chỉ có 4 dòng (Regime / Cash buffer 100% / Lý do / Khi nào cycle kế tiếp), Section 2-7 skip
- Top mã list: tối đa 7 trong Section 1, mã thứ 8+ đẩy xuống Section 5

### 3.4. stock_memo — Memo deep-dive nội bộ (flex theo conviction tier)

**Audience:** nội bộ analyst (memo conviction để ra quyết định position). **Required field:** `conviction_tier` (high/medium/low).

**High conviction (15-18 điểm)** — 7 phần đầy đủ, 10-15 trang:

```
# Stock Memo — [TICKER]
## [Tên đầy đủ công ty]

## 1. Recommendation + Thesis
## 2. Variant Perception
## 3. Business Overview
## 4. Financial Analysis + Valuation
## 5. Catalysts
## 6. Bear Case Steelmanned
## 7. Monitoring + Exit Triggers
— Metadata
```

**Medium conviction (11-14 điểm)** — 5 phần, 6-10 trang. Skip section 3 + section 6 simplified:

```
## 1. Recommendation + Thesis
## 2. Variant Perception
## 4. Financial Analysis + Valuation (rút gọn)
## 5. Catalysts
## 7. Monitoring + Exit Triggers
```

**Low conviction (8-10 điểm)** — 3 phần, 3-5 trang:

```
## 1. Recommendation + Thesis (ngắn)
## 4. Financial Analysis + Valuation (chỉ target + key ratio)
## 7. Monitoring + Exit Triggers
```

**Ràng buộc:**
- Section 2 Variant Perception BẮT BUỘC explicit (3 sub: consensus / retail / thesis khác)
- Section 6 Bear Case có honest steelman 1-2 lo ngại còn yếu (high tier)
- Section 7 Exit Triggers measurable: target / thesis broken / better opportunity / stop loss
- Section 4 có target Base/Bull/Bear với assumption rõ

### 3.5. portfolio_plan — Kế hoạch portfolio + order list (8 sections, flex)

**Audience:** nội bộ (actionable, đặt lệnh broker). **Length:** 6-10 trang.

```
# Portfolio Plan — [Ngày]

## 1. Executive summary
## 2. Portfolio allocation
## 3. Constraint check
## 4. Mã trong portfolio — thesis tóm tắt
## 5. Sequence entry plan (Phase 1/2/3)
## 6. Limit orders — bảng đọc
## 7. Order list — block copy cho broker
## 8. Cảnh báo & lựa chọn sát nút
— Metadata
```

**Ràng buộc:**
- Section 1 Executive summary block cứng: Portfolio size / Regime / Phase 1 deployment % / Cash buffer / Phân bổ theo conviction tier / theo ngành / Top concentration
- Section 7 Order list format chuẩn để copy thẳng vào broker — ticker + side + quantity + limit price + valid until
- Position sizing tuân ràng buộc: ≤5% ADV 20 phiên (mỗi mã)

### 3.6. portfolio_review_weekly — Review portfolio tuần (6 sections, rigid)

**Audience:** nội bộ. **Length:** 0.5-2 trang. Đọc 12-24 lần/cycle nên format nhất quán.

```
# Weekly Review — [Ngày]

## 1. Alert level
## 2. Position health
## 3. Triggers matched tuần này
## 4. Phase 2 / Phase 3 review
## 5. Portfolio summary
## 6. Actions needed
— Metadata
```

**Ràng buộc:**
- Đầy đủ 6 section kể cả khi rỗng (viết "Không có" thay vì skip)
- Section 1 Alert level: Green / Yellow / Red — rule fixed theo trigger summary
- Section 3 Triggers matched: hard trigger (mức giá, thesis broken) + soft trigger (drift, news)

### 3.7. portfolio_review_monthly — Review portfolio tháng (8 sections, rigid)

**Audience:** nội bộ. **Length:** 3-5 trang.

```
# Monthly Review — [MM/YYYY]

## 1. Executive summary
## 2. Portfolio performance
## 3. Concentration & drift
## 4. Memo refresh status
## 5. Regime re-check
## 6. Missed opportunities
## 7. Rebalance proposal (nếu cần)
## 8. Actions needed
— Metadata
```

**Ràng buộc:**
- Section 2 Performance: Return MoM, YTD, Win rate, Sharpe (nếu đủ data)
- Section 4 Memo refresh: list mã có memo > 30 ngày — flag cần refresh
- Section 5 Regime re-check: confirm regime giữ nguyên hay shift, lý do
- Section 7 chỉ có khi drift / regime shift đáng kể; tuần thường không có

### 3.8. portfolio_review_quarterly — Review portfolio quý (9 sections, rigid)

**Audience:** nội bộ. **Length:** 5-8 trang. Báo cáo sâu nhất chu kỳ monitoring vì tích hợp BCTC quý mới.

```
# Quarterly Review — [Q[N]/YYYY]

## 1. Executive summary
## 2. Performance quarterly
## 3. BCTC quý mới — per-stock
## 4. Thesis verification
## 5. Forensic partial re-check
## 6. Valuation update
## 7. Exit levels update
## 8. Rebalance proposal
## 9. Actions needed
— Metadata
```

**Ràng buộc:**
- Section 3 BCTC quý mới: per-stock BCTC actual vs assumption ở memo gốc
- Section 4 Thesis verification: thesis còn đứng vững không, có cần điều chỉnh
- Section 6 Valuation update: re-run target Base/Bull/Bear với data actual
- Quý mã chưa công bố BCTC → flag rõ "chờ BCTC", không tự suy

### 3.9. custom — Quiz-driven, không có template sẵn

**Audience:** tùy user. **Length:** tùy user.

Khi user pick `custom`, agent KHÔNG tự build structure mặc định. Thay vào đó, ở Stage 3 của WORKFLOW (xem `WORKFLOW.md` mục 5), agent chạy **bộ trắc nghiệm custom** để xác định cụ thể spec báo cáo. Sau khi user trả lời quiz, agent build spec runtime + lưu vào frontmatter:

```yaml
---
report_type: custom
custom_spec_id: 20260427_153022    # timestamp định danh spec
custom_purpose: "Báo cáo phân tích ngành ngân hàng Q1/2026"
custom_audience: "Khách hàng VIP"
custom_sections:
  - "1. Tóm tắt"
  - "2. Bối cảnh ngành Q1"
  - "3. So sánh 5 mã top"
  - "4. Định giá ngành"
  - "5. Triển vọng"
  - "6. Khuyến nghị mã"
  - "7. Disclaimer"
custom_length_target: 8-10 trang
custom_chart_count: 4
custom_citation_style: footnote
publish_date: 2026-04-27
---
```

**Ràng buộc nội dung custom:**

- Section count linh hoạt (3-15), nhưng mỗi section H2 phải match được với 1 layout có sẵn trong TEMPLATE pack (xem catalog `TEMPLATE_VBSE.md` / `TEMPLATE_FINEXT.md` 27 layout)
- Nếu user yêu cầu section purpose không có layout phù hợp → agent flag, đề xuất section thay thế hoặc fallback về `BULLET_LIST_SUMMARY` layout neutral
- Frontmatter `custom_spec_id` để trace lại quiz answers
- Disclaimer cuối nếu audience là khách hàng (custom_audience chứa "khách" / "client" / "external")

**Re-render với spec đã có:**

Nếu user upload lại MD `custom` (đã có `custom_spec_id`), Stage 1.5 detect skip-normalize → đi thẳng Stage 6 brand pre-flight. Không chạy lại quiz.

## 4. Chart annotation YAML

Tại chỗ cần chart trong MD, chèn YAML block. TEMPLATE pack runtime đọc block này để build native chart.

### 4.1. Syntax bắt buộc

````
```chart
type: bar | line | pie | scatter | combo | stacked_bar | donut
title: [tiêu đề chart, tiếng Việt]
x_axis: [array hoặc field name]
y_axis: [array hoặc field name, có thể multiple series]
y_label: [unit label, ví dụ "Tỷ VND", "%"]
x_label: [optional]
source: [nguồn data]
note: [optional, chú thích dưới chart]
```
````

### 4.2. Chart types support

| Type | Use case |
|---|---|
| `bar` | Cột đứng, so sánh giữa categories |
| `line` | Trend theo thời gian |
| `pie` / `donut` | Cơ cấu, composition |
| `scatter` | Correlation 2 biến (peer comparison) |
| `combo` | Bar + line combined (revenue + growth rate) |
| `stacked_bar` | Cột chồng, composition theo time |

### 4.3. Ví dụ

````
```chart
type: combo
title: VNM Revenue & Growth 2021-2025
x_axis: [2021, 2022, 2023, 2024, 2025]
y_axis:
  - name: Revenue
    type: bar
    data: [10200, 12800, 14200, 16500, 18200]
    y_label: Tỷ VND
  - name: YoY Growth
    type: line
    data: [null, 25.5, 10.9, 16.2, 10.3]
    y_label: '%'
    axis: secondary
source: BCTC VNM 2021-2025
note: CAGR 15.5%
```
````

## 5. Citation format

4 nhóm nguồn, format khác nhau:

### 5.1. Nhóm 1 — Dữ liệu tổng hợp

Số liệu định lượng từ database/aggregation: ghi `(nguồn: Tổng hợp)` cuối câu.

```
Revenue VNM 2025 đạt 18.200 tỷ VND, tăng 10,3% YoY (nguồn: Tổng hợp).
```

### 5.2. Nhóm 2 — Tin tức báo cáo

Dẫn link đầy đủ với markdown:

```
NVL công bố bổ sung nội dung ĐHCĐ ([Finext, 18/4/2026](https://finext.vn/news/nvl-truoc-dhcd)).
```

### 5.3. Nhóm 3 — Tài liệu user upload

Ghi tên tài liệu + trang/mục cụ thể:

```
DSO tăng từ 45 lên 52 ngày (BCTC VNM Q4/2025 soát xét, mục 8).
```

### 5.4. Nhóm 4 — Web external

Markdown link:

```
ERP VN 2025 là 8,35% ([NYU Stern country risk](https://pages.stern.nyu.edu/...)).
```

### 5.5. Footnote khi nhiều citation

Body có ≥3 citation external trong 1 đoạn → dùng footnote `[^1]` cuối câu, list source cuối section/file.

## 6. Locale vi-VN

### 6.1. Số

- Dấu phẩy ngăn nghìn: `18.200 tỷ` (KHÔNG `18,200`)
- Dấu phẩy thập phân: `15,5%` (KHÔNG `15.5%`)
- Phần trăm có dấu rõ: `+18,2%` / `-3,5%`

### 6.2. Đơn vị

- Tiền VND: `tỷ VND` cho tổng, `đồng` cho giá per share, `nghìn`/`k` cho giá ngắn (`33.000 đ` hoặc `33k`)
- Tỷ lệ: 1 chữ số thập phân (`15,5%`)
- Date: `Q1/2026`, `tháng 4/2026`, `ngày 27/4/2026`

### 6.3. Ticker

- UPPERCASE, không nháy: `VNM` (KHÔNG `'VNM'`, `vnm`)

## 7. K hygiene — không lộ ký hiệu raw

MD chuẩn hóa **không được chứa** ký hiệu DB raw hoặc taxonomy nội bộ:

**Cấm:**
- `vsi: 2.1`, `week_score: 18`, `day_score: 68`
- `zone: AAA`, `technical_zone.overall.w: A`
- `f382`, `f500`, `f618`, `poc`, `val`, `vah`, `r1`, `s1`
- `period: "2025_4"`, `m_pct: 0.062`
- `Kịch bản A/B/C`, `Pitfall F1-F12`, `HIGH/MID/LOW impact`

**Thay bằng ngôn ngữ tự nhiên:**
- `vsi: 2.1` → "thanh khoản gấp 2.1 lần trung bình 5 phiên"
- `zone: AAA` → "vùng kỹ thuật rất mạnh"
- `f382` → "Fibonacci 38.2%"
- `period: "2025_4"` → "Q4/2025"

Nếu input có ký hiệu raw → WORKFLOW Stage 4 LLM-translate sang ngôn ngữ tự nhiên trước khi xuất MD final.

## 8. Thuật ngữ chuyên môn

### 8.1. Widely adopted — không dịch

Giữ nguyên: P/E, P/B, ROE, ROA, ROIC, EBIT, EBITDA, YoY, QoQ, MoM, CAGR, ADV, NPL, NIM, CASA, FCF, DCF, IRR, BVPS.

### 8.2. Cần dịch inline lần đầu

Pattern: `English term (dịch ngắn)`. Lần xuất hiện sau trong cùng báo cáo không cần dịch lại.

| Term gốc | Dịch inline |
|---|---|
| Variant perception | Variant perception (góc nhìn khác consensus) |
| Catalyst | Catalyst (chất xúc tác đẩy giá) |
| Steelman bear case | Bear case steelmanned (luận điểm phản đối ở dạng mạnh nhất) |
| Conviction | Conviction (mức độ tự tin) |
| Horizon | Horizon (khung thời gian giữ vị thế) |

## 9. Validation checklist

Trước khi MD final đi vào Stage 7 render, agent self-check:

- [ ] Frontmatter đầy đủ + `report_type` thuộc 9 loại whitelist (8 preset + custom)
- [ ] H1 duy nhất, H2 theo structure của report_type
- [ ] Số section khớp spec (vd stock_pitch có 13-16 section flex theo số luận điểm 4-7; weekly_market 12 rigid; v.v.)
- [ ] Block khuyến nghị 4 dòng (stock_pitch)
- [ ] Chart YAML block đúng syntax + type thuộc whitelist
- [ ] Citation format đúng 4 nhóm
- [ ] Locale vi-VN đồng nhất (số, %, đơn vị)
- [ ] Không lộ ký hiệu DB raw
- [ ] Thuật ngữ EN dịch inline lần đầu
- [ ] Disclaimer cuối báo cáo (bắt buộc với mọi report_type gửi KH như stock_pitch / weekly_market; tùy chọn với report_type nội bộ như market_scan / stock_memo / portfolio_*; custom theo `custom_audience`)

Vi phạm câu nào → quay lại Stage 4 normalize fix, hoặc clarify với user.
