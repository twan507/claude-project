# TEMPLATE_VBSE — Catalog Layout Brand VBSE

Pack cung cấp template visual brand VBSE cho render báo cáo pptx. Activate ở Stage 7 của `WORKFLOW.md` khi user pick brand VBSE ở CP3.

TEMPLATE_VBSE là layout catalog độc lập — chỉ đọc 2 nguồn: file pack của chính nó + MD final do WORKFLOW.md produce.

## Manifest pack

| File | Vai trò |
|---|---|
| `TEMPLATE_VBSE.md` | Master catalog (file này) — danh sách 27 layout, placeholder schema, chart strategy, design tokens, render rule |
| `TEMPLATE_VBSE.pptx` | Artifact binary 27 layout |

## Khi nào activate TEMPLATE_VBSE

Activate khi user pick "(a) VBSE" ở CP3 brand pre-flight (xem `WORKFLOW.md` mục 11). Điều kiện chung activate: user đã có MD final đúng `FORMAT.md` contract sau Stage 5.

Không activate khi: user pick Finext, hoặc skip render binary (chỉ MD).

## Pipeline render

TEMPLATE_VBSE đứng cuối pipeline workflow, chỉ consume MD final đã đông kết:

```
[Stage 1-5 của WORKFLOW.md đã chạy xong, output: MD final đúng FORMAT contract]
       │
       ▼
TEMPLATE_VBSE runtime (Stage 7)
1. Đọc MD final → scan section heading + chart annotation block
2. Đọc TEMPLATE_VBSE.md (file này) → biết layout nào có sẵn + purpose
3. Match semantic: section "Cover khuyến nghị MUA mã X" → COVER_A_RECOMMENDATION
4. Mở TEMPLATE_VBSE.pptx → clone slide layout → fill placeholder từ MD content
5. Tại chart placeholder: build native PowerPoint chart từ YAML block trong MD
6. Save thành báo cáo pptx cuối
```

TEMPLATE_VBSE runtime CHỈ đọc 2 nguồn: file pack của chính nó + MD final. Không đọc FORMAT.md, không đọc WORKFLOW.md (đó là context của upstream stage). Mọi nội dung cần thiết (số liệu, luận điểm, chart data, citation) đã có trong MD final.

## Chart placeholder strategy

TEMPLATE_VBSE KHÔNG vẽ chart visual fake. Layout có chart đều dùng pattern **chart placeholder**: rectangle khung trống + label `{{CHART_PLACEHOLDER}}` mô tả loại chart.

Pattern chart annotation chuẩn trong MD source dạng YAML block (T consume theo quy ước này — không reference pack nào quy định):

```
```chart
type: bar | line | pie | scatter | combo
title: [tiêu đề]
x_axis: [...]
y_axis: [...]
```
```

Khi AI render thực tế:
1. Mở slide template (vd `MINI_CHARTS_2X2`)
2. Tìm shape có text `{{CHART_X_PLACEHOLDER}}` → ghi nhận vị trí (left, top, width, height)
3. Remove shape placeholder
4. Đọc YAML block tương ứng từ MD
5. Add native PowerPoint chart vào đúng vị trí với data từ YAML

Lợi ích:
- Template không bị nhiễu visual fake
- Phân tách rõ: T = visual layout, MD = nội dung (content + chart data)
- Chart cuối render là native PowerPoint chart → user/AI có thể edit data sau

Loại chart support trong MD source:
- `bar` (cột) — BCTC 5 năm, top dẫn dắt ngành
- `line` (đường) — VNINDEX history, giá ticker, P/E history
- `pie` / `donut` (cơ cấu) — cổ đông, doanh thu mảng
- `scatter` — định vị peer 2 chiều
- `combo` (bar + line) — revenue + growth, VNINDEX + breadth

Không support: `bubble`, `stacked_bar` (rủi ro render sai khi tự build).

## Design tokens

**Color palette** (extract từ pptx mẫu VBSE GEL, đã verify):

| Token | Hex | Dùng cho |
|---|---|---|
| Navy chính | `#003D7A` | Title, big stat, primary text |
| Đỏ accent | `#D71249` | Tam giác signature, target accent, numbered icon |
| Xanh lá | `#059669` | Số liệu dương, badge MUA, kịch bản tích cực |
| Text đậm | `#1F2937` | Body text chính |
| Text vừa | `#4A5568` | Sub-text, label trong card |
| Text nhạt | `#9CA3AF` | Footer, muted, chart placeholder text |
| Trắng | `#FFFFFF` | Card chính, text trên nền tối |
| BG nhạt | `#F7F9FC` | Card background nhạt |
| Hồng nhạt | `#F5D5E0` | Highlight "lo ngại còn yếu", target accent |

**Font**: Calibri (default Office Windows + Mac, không cần embed).

**Slide size**: 16:9, 13.33" × 7.5".

**Signature pattern**:
- Vertical red bar 0.08" × 0.70" cạnh trái title (mọi slide content)
- Tam giác cân đỏ (`ISOSCELES_TRIANGLE`) decorative, đỉnh nhọn rõ ràng. Áp dụng cho COVER_A (rìa navy panel), SECTION_DIVIDER, COVER_E, DISCLAIMER.

## Nội dung fix cứng (không placeholder)

Các trường sau ghi cứng trong template, AI không cần fill:

| Trường | Giá trị fix cứng |
|---|---|
| Tên phòng ban | "PHÒNG MÔI GIỚI VÀ TƯ VẤN ĐẦU TƯ" |
| Tên tổ | "Tổ tầm soát cổ phiếu" |
| Website | "www.vbse.vn" |
| Hotline | "1900 588 866" |
| Disclaimer body | 3 đoạn pháp lý chuẩn VBSE |
| Tiêu đề "BÁO CÁO PHÂN TÍCH" trong COVER_A | Cố định |
| Tiêu đề "PHÂN TÍCH CHUYÊN SÂU" trong COVER_B | Cố định |
| Tiêu đề "BÁO CÁO THỊ TRƯỜNG TUẦN" trong COVER_C | Cố định |
| Header "TUYÊN BỐ MIỄN TRỪ TRÁCH NHIỆM" | Cố định |

Lưu ý: nếu VBSE đổi tên phòng ban hoặc đổi disclaimer, sửa trực tiếp trong template `TEMPLATE_VBSE.pptx`.

## Layout pattern toàn cục

Mọi slide content (trừ cover/divider/disclaimer) tuân theo cấu trúc 3-band:

```
┌─ Slide 13.33" × 7.5" (16:9) ────────────────────────────┐
│                                                          │
│ ▌ {{TITLE}}                          ← top=0.30"        │
│ ▌ {{SUB-TITLE}}                                          │
│ ───────────────────────────────────                      │
│                                                          │
│   Content area                                           │
│     left=0.60"   width=12.13"                            │
│     top=1.40"    height=5.30"                            │
│                                                          │
│                                                          │
│   {{FOOTER_LEFT}}              {{PAGE_NUM}}  ← top=7.10" │
└──────────────────────────────────────────────────────────┘
```

**Vùng tiêu đề (Title bar)** — cao 1.10" từ top:
- Vertical red bar `0.08" × 0.70"` cạnh trái title (signature)
- `{{TITLE}}` font Pt(24) bold navy
- `{{SUB-TITLE}}` font Pt(11) italic text-mid

**Vùng nội dung (Content area)** — `left=0.60, top=1.40, width=12.13, height=5.30`:
- Đây là vùng làm việc chính cho mọi layout content
- Tham khảo qua `GRID["content_l"]`, `GRID["content_top"]`, `GRID["content_w"]`, `GRID["content_h"]`

**Vùng footer** — `top=7.10`:
- `{{FOOTER_LEFT}}` text light, italic
- `{{PAGE_NUM}}` text light, align right

**Cover slides (1-5)** không tuân theo pattern này — full bleed hoặc half-bleed navy panel.

**Section divider (6) + Disclaimer (27)** — full navy background, không có title bar / content area chuẩn.

## Cách AI parse template

Mỗi slide trong `TEMPLATE_VBSE.pptx` có:

1. **Slide name** (XML attribute `cSld@name`) = LAYOUT_ID. AI dùng để identify layout khi clone.
2. **Placeholder** dạng `{{NAME}}` trong text run. AI regex match `\{\{[A-Z_0-9]+\}\}` → biết shape nào cần fill.
3. **Chart placeholder**: shape rectangle có text `{{CHART_X_PLACEHOLDER}}` đánh dấu chỗ AI build native chart.

Khi fill: thay nguyên text `{{NAME}}` bằng giá trị thực, giữ font size + color + alignment của run gốc.

## Danh sách 27 layout

### Cover (5 layout)

#### LAYOUT 01 — `COVER_A_RECOMMENDATION`
**Slide #:** 1
**Mục đích:** Cover khuyến nghị mua mã đơn lẻ. Half-bleed navy + ticker XL + badge MUA + 2 stat (giá hiện tại + giá mục tiêu) + thesis headline.

**Placeholders:**
- `{{TICKER}}` — Mã cổ phiếu, ≤ 5 ký tự (font 96pt)
- `{{COMPANY_FULL_NAME}}` — Tên đầy đủ công ty
- `{{PRICE_CURRENT}}` — Giá hiện tại có đơn vị
- `{{PRICE_TARGET}}` — Giá mục tiêu có đơn vị
- `{{UPSIDE_LINE}}` — vd "Tiềm năng tăng giá: +18,0%"
- `{{THESIS_HEADLINE}}` — Tiêu đề thesis 1 câu, ≤ 60 ký tự
- `{{PUBLISH_DATE}}` — vd "Ngày phát hành: 22/04/2026"

#### LAYOUT 02 — `COVER_B_DEEPDIVE`
**Slide #:** 2
**Mục đích:** Cover phân tích chuyên sâu mã đơn lẻ. Tone trung tính, không có badge MUA. 4 stat callout: giá hiện tại / giá kỳ vọng cơ sở / mức cắt lỗ / tỷ lệ lời-lỗ.

**Placeholders:**
- `{{TICKER}}` (≤ 6 ký tự)
- `{{REPORT_SUBTITLE}}`, `{{COMPANY_FULL_NAME}}`, `{{INDUSTRY_LABEL}}`
- `{{PRICE_CURRENT}}`, `{{PRICE_DATE}}`
- `{{TARGET_BASE}}`, `{{UPSIDE_BASE}}`
- `{{STOPLOSS}}`, `{{DOWNSIDE_STOP}}`
- `{{RR_RATIO}}`, `{{RR_VERDICT}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 03 — `COVER_C_WEEKLY_MARKET`
**Slide #:** 3
**Mục đích:** Cover báo cáo thị trường tuần. Headline regime + 3 KPI VNINDEX.

**Placeholders:**
- `{{WEEK_LABEL}}` — vd "Tuần 17/2026"
- `{{HEADLINE_LINE}}`, `{{REGIME_BADGE}}`
- `{{VNINDEX_CLOSE}}`, `{{VNINDEX_CHANGE}}`
- `{{LIQUIDITY_AVG}}`, `{{LIQUIDITY_NOTE}}`
- `{{FOREIGN_NET}}`, `{{FOREIGN_NOTE}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 04 — `COVER_D_MACRO_SECTOR`
**Slide #:** 4
**Mục đích:** Cover báo cáo chủ đề vĩ mô / phân tích ngành. 3 takeaway numbered.

**Placeholders:**
- `{{THEME_TAG}}` — vd "VĨ MÔ", "PHÂN TÍCH NGÀNH"
- `{{REPORT_TITLE}}` (≤ 60 ký tự), `{{REPORT_SUBTITLE}}`
- `{{TAKEAWAY_1}}` ... `{{TAKEAWAY_3}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 05 — `COVER_E_GENERIC`
**Slide #:** 5
**Mục đích:** Cover tổng quát, linh hoạt cho báo cáo định kỳ, ad-hoc.

**Placeholders:**
- `{{REPORT_KIND_TAG}}`, `{{REPORT_TITLE}}`, `{{REPORT_SUBTITLE}}`
- `{{PUBLISH_DATE}}`

---

### Section divider (1 layout)

#### LAYOUT 06 — `SECTION_DIVIDER`
**Slide #:** 6
**Mục đích:** Chia section lớn trong báo cáo dài (>15 slide).

**Placeholders:**
- `{{SECTION_TAG}}`, `{{SECTION_TITLE}}`, `{{SECTION_DESC}}`

---

### Content layout (12)

#### LAYOUT 07 — `BULLET_LIST_SUMMARY`
**Slide #:** 7. Tóm tắt 5 bullet label + body. Dùng cho Weekly Exec Summary.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{BULLET_1_LABEL}}` ... `{{BULLET_5_LABEL}}`
- `{{BULLET_1_BODY}}` ... `{{BULLET_5_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 08 — `STAT_CALLOUT_GRID_4CELL`
**Slide #:** 8. 4 stat callout + section band + 4 luận điểm 2x2. Phù hợp cho slide tóm tắt khuyến nghị (4 mức giá target/stop + 4 luận điểm cốt lõi).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{TARGET_SHORT}}`, `{{UPSIDE_SHORT}}`, `{{TARGET_MID}}`, `{{UPSIDE_MID}}`
- `{{STOPLOSS}}`, `{{DOWNSIDE_STOP}}`, `{{RR_RATIO}}`, `{{RR_VERDICT}}`
- `{{POINT_1_TITLE}}` ... `{{POINT_4_TITLE}}`, `{{POINT_1_BODY}}` ... `{{POINT_4_BODY}}` (≤ 100 ký tự)
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 09 — `BIG_STAT_SUBPOINTS`
**Slide #:** 9. 1 luận điểm chuyên sâu. Big stat panel navy trái + 3 sub-points + bảng KPI.
- `{{LUANDIEM_TITLE}}`, `{{LUANDIEM_SUB}}`
- `{{KPI_LABEL}}`, `{{KPI_VALUE}}` (≤ 6 ký tự, 96pt), `{{KPI_NOTE}}`, `{{KPI_SUB}}`
- `{{SUBPOINT_1_TITLE}}` ... `{{SUBPOINT_3_TITLE}}`
- `{{SUBPOINT_1_BODY}}` ... `{{SUBPOINT_3_BODY}}`
- `{{KPI_ROW_1_LABEL}}` ... `{{KPI_ROW_5_LABEL}}`
- `{{KPI_ROW_1_VALUE}}` ... `{{KPI_ROW_5_VALUE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 10 — `TWO_COLUMN_INFO_BUSINESS`
**Slide #:** 10. Hồ sơ DN 2 cột: bảng thông tin (9 dòng) + 5 mảng kinh doanh.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{INFO_1}}` ... `{{INFO_9}}` (thứ tự: Mã CP, Ngày NY, Vốn ĐL, Số CP LH, Vốn hoá, Tự do CN, Cổ đông lớn, NN%, Ngành)
- `{{SEGMENT_1_TITLE}}` ... `{{SEGMENT_5_TITLE}}`
- `{{SEGMENT_1_BODY}}` ... `{{SEGMENT_5_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 11 — `COMPARISON_3CELL_SCENARIOS`
**Slide #:** 11. 3 kịch bản: Cơ sở (navy) / Tích cực (xanh) / Thận trọng (đỏ).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`, `{{ACTION_RECOMMENDED}}`
- `{{BASE_TIMEFRAME}}`, `{{BASE_TARGET}}`, `{{BASE_RETURN}}`, `{{BASE_BIG_PCT}}`, `{{BASE_BODY}}`
- `{{BULL_TIMEFRAME}}`, `{{BULL_TARGET}}`, `{{BULL_RETURN}}`, `{{BULL_BIG_PCT}}`, `{{BULL_BODY}}`
- `{{BEAR_TIMEFRAME}}`, `{{BEAR_TARGET}}`, `{{BEAR_RETURN}}`, `{{BEAR_BIG_PCT}}`, `{{BEAR_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** KHÔNG gán xác suất % vào kịch bản. Header dùng "Khả năng cao nhất / vừa phải / thấp" qualitative.

#### LAYOUT 12 — `STAT_TABLE_COMBO_SESSION`
**Slide #:** 12. 4 stat callout phiên + bảng 8 mức kỹ thuật.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CLOSE_PRICE}}`, `{{CLOSE_CHANGE}}`, `{{HIGH_PRICE}}`, `{{HIGH_NOTE}}`
- `{{LOW_PRICE}}`, `{{LOW_NOTE}}`, `{{VALUE_AMOUNT}}`, `{{LIQUIDITY_NOTE}}`
- `{{LEVEL_1_NAME}}` ... `{{LEVEL_8_NAME}}` (cao→thấp: Kháng cự R1, Fibo 38.2%, Biên trên, POC, Giá hiện tại, Biên dưới, Hỗ trợ S1, MA60)
- `{{LEVEL_1_PRICE}}` ... `{{LEVEL_8_PRICE}}`, `{{LEVEL_1_DELTA}}` ... `{{LEVEL_8_DELTA}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 13 — `DATA_TABLE_FULL`
**Slide #:** 13. Bảng full 8×8.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{COL_1_HEAD}}` ... `{{COL_8_HEAD}}`
- `{{R{i}_C{j}}}` cho i=1..8, j=1..8
- `{{TABLE_NOTE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 14 — `TIMELINE_NEWS`
**Slide #:** 14. Timeline 4 sự kiện trục dọc.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{EVENT_1_DATE}}` ... `{{EVENT_4_DATE}}`
- `{{EVENT_1_TITLE}}` ... `{{EVENT_4_TITLE}}`
- `{{EVENT_1_BODY}}` ... `{{EVENT_4_BODY}}`
- `{{EVENT_1_TAG}}` ... `{{EVENT_4_TAG}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 15 — `RISK_GRID_STEELMAN`
**Slide #:** 15. Grid 6 cards lo ngại + phản biện. 2 card cuối highlight hồng "lo ngại còn yếu".
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CONCERN_1}}` ... `{{CONCERN_6}}` (CONCERN_5, 6 = lo ngại còn yếu)
- `{{REBUT_1}}` ... `{{REBUT_6}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** CONCERN_5/6 BẮT BUỘC là lo ngại thực sự yếu sau phản biện (honest steelman), không "all win" — đây là yêu cầu nội dung chung cho mọi báo cáo dùng layout này.

#### LAYOUT 16 — `VARIANT_PERCEPTION`
**Slide #:** 16. Insight box + 3 sub-section: đồng thuận thị trường / tâm lý NĐT cá nhân / góc nhìn khác đồng thuận.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{INSIGHT_KEY}}`
- `{{CONSENSUS_VIEW}}`, `{{RETAIL_VIEW}}`, `{{DIFFERENT_VIEW}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** Cả 3 sub-section bắt buộc fill.

#### LAYOUT 17 — `BUY_STRUCTURE_FLEX`
**Slide #:** 17. Cấu trúc mua N lớp linh hoạt 2-5 lớp + risk band.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{LAYER_COUNT}}`, `{{STRUCTURE_NOTE}}`
- `{{LAYER_1_LABEL}}` ... `{{LAYER_5_LABEL}}` (vd "LỚP 01")
- `{{LAYER_1_ACTION}}` ... `{{LAYER_5_ACTION}}` (vd "MUA THĂM DÒ")
- `{{LAYER_1_PRICE_ZONE}}` ... `{{LAYER_5_PRICE_ZONE}}`
- `{{LAYER_1_ALLOCATION}}` ... `{{LAYER_5_ALLOCATION}}`
- `{{LAYER_1_TRIGGER}}` ... `{{LAYER_5_TRIGGER}}`
- `{{STOP_HARD_PRICE}}`, `{{STOP_HARD_NOTE}}`
- `{{STOP_SOFT_PRICE}}`, `{{STOP_SOFT_NOTE}}`
- `{{HOLD_PERIOD}}`, `{{HOLD_NOTE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** Nếu < 5 lớp, fill ô thừa rỗng.

#### LAYOUT 18 — `TAKE_PROFIT_TARGETS`
**Slide #:** 18. 4 mốc chốt lãi M1-M4 + bottom band tóm tắt R/R.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{M1_NUM}}` ... `{{M4_NUM}}`
- `{{M1_PRICE}}` ... `{{M4_PRICE}}`
- `{{M1_PROFIT_PCT}}` ... `{{M4_PROFIT_PCT}}`
- `{{M1_RATIO}}` ... `{{M4_RATIO}}`
- `{{M1_BASIS}}` ... `{{M4_BASIS}}`
- `{{AVG_PROFIT}}`, `{{MAX_RISK}}`, `{{RR_RATIO}}`, `{{MAX_TARGET}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 19 — `THESIS_B_PANEL_RIGHT`
**Slide #:** 19. Alternative cho LAYOUT_09. Big stat panel phải + 4 sub-points 2x2 trái.
- `{{LUANDIEM_TITLE}}`, `{{LUANDIEM_SUB}}`
- `{{SUBPOINT_1_TITLE}}` ... `{{SUBPOINT_4_TITLE}}`
- `{{SUBPOINT_1_BODY}}` ... `{{SUBPOINT_4_BODY}}`
- `{{KPI_LABEL}}`, `{{KPI_VALUE}}`, `{{KPI_NOTE}}`, `{{KPI_LONG_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 20 — `HEATMAP_INDUSTRY`
**Slide #:** 20. Bản đồ nhiệt 24 ngành VBSE 6×4 grid + legend thang màu.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{IND_1_NAME}}` ... `{{IND_24_NAME}}`
- `{{IND_1_PCT}}` ... `{{IND_24_PCT}}`
- `{{IND_1_NOTE}}` ... `{{IND_24_NOTE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** AI override màu cell theo magnitude %:
- `>+3%` → fill `#003D7A` (navy), text trắng
- `+1% to +3%` → fill `#059669` (xanh), text trắng
- `-1% to +1%` → fill `#F7F9FC` (trung tính), text đậm
- `-3% to -1%` → fill `#F5D5E0` (hồng), text đậm
- `<-3%` → fill `#D71249` (đỏ), text trắng

#### LAYOUT 21 — `PEER_COMPARE_TABLE`
**Slide #:** 21. So sánh 8 mã trong cùng ngành. P1 highlight hồng (mã đang phân tích).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{P1_TICKER}}` ... `{{P8_TICKER}}` (P1 = highlight)
- `{{P1_MARKETCAP}}` ... `{{P8_MARKETCAP}}`
- `{{P1_PE}}` ... `{{P8_PE}}`
- `{{P1_PB}}` ... `{{P8_PB}}`
- `{{P1_ROE}}` ... `{{P8_ROE}}`
- `{{P1_ROA}}` ... `{{P8_ROA}}`
- `{{P1_REVGROWTH}}` ... `{{P8_REVGROWTH}}`
- `{{P1_NOTE}}` ... `{{P8_NOTE}}`
- `{{PEER_VERDICT}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 22 — `MINI_CHARTS_2X2`
**Slide #:** 22. 4 chart placeholder 2×2. AI build native chart từ YAML block. Phù hợp cho block 4 chart chỉ số tài chính 5 năm hoặc 4 stat trend song song.

- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_1_TITLE}}` ... `{{CHART_4_TITLE}}`
- `{{CHART_1_UNIT}}` ... `{{CHART_4_UNIT}}`
- **`{{CHART_1_PLACEHOLDER}}` ... `{{CHART_4_PLACEHOLDER}}`** — AI thay bằng native chart
- `{{CHART_1_LATEST}}` ... `{{CHART_4_LATEST}}`
- `{{CHART_1_CHANGE}}` ... `{{CHART_4_CHANGE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `bar` (column clustered), 5 năm, color navy.

#### LAYOUT 23 — `CALENDAR_WEEK`
**Slide #:** 23. Lịch 7 ngày Thứ 2 - CN, mỗi ngày 4 sự kiện.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{DAY_1_DATE}}` ... `{{DAY_7_DATE}}`
- `{{DAY_1_EVT_1_TIME}}` ... `{{DAY_7_EVT_4_TIME}}`
- `{{DAY_1_EVT_1_TITLE}}` ... `{{DAY_7_EVT_4_TITLE}}`
- `{{DAY_1_COUNT}}` ... `{{DAY_7_COUNT}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

---

### Chart placeholder layouts (3 layout)

#### LAYOUT 24 — `LINE_CHART_FULL`
**Slide #:** 24
**Mục đích:** Chart đường full width (2/3 trang) + 3 commentary cards phải. Phù hợp cho mọi chart trend theo thời gian (chỉ số, giá, ratio, equity curve).

- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_TITLE}}`
- **`{{CHART_PLACEHOLDER}}`** — AI build native line chart
- `{{CHART_SOURCE}}`
- `{{NOTE_1_TITLE}}`, `{{NOTE_1_BODY}}`
- `{{NOTE_2_TITLE}}`, `{{NOTE_2_BODY}}`
- `{{NOTE_3_TITLE}}`, `{{NOTE_3_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `line`, 1-3 series, x_axis = thời gian.

#### LAYOUT 25 — `SCATTER_PEER`
**Slide #:** 25
**Mục đích:** Scatter plot định vị peer 2 chiều (vd P/E vs ROE) + bảng peer rút gọn. Phù hợp cho mọi block peer comparison 2D.

- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_TITLE}}`, `{{Y_AXIS_LABEL}}`, `{{X_AXIS_LABEL}}`
- **`{{CHART_PLACEHOLDER}}`** — AI build native scatter
- `{{CHART_SOURCE}}`
- `{{PEER_1_TICKER}}` ... `{{PEER_6_TICKER}}` (PEER_1 = highlight đỏ)
- `{{PEER_1_NOTE}}` ... `{{PEER_6_NOTE}}`
- `{{PEER_VERDICT}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `scatter`, mã đang phân tích = marker đỏ, peer khác = marker navy.

#### LAYOUT 26 — `DONUT_COMPOSITION`
**Slide #:** 26
**Mục đích:** 2 donut chart side-by-side + legend chi tiết. Phù hợp cho mọi block composition 2 chiều (cơ cấu cổ đông + cơ cấu doanh thu, allocation portfolio + sector composition, etc.).

- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{DONUT_1_TITLE}}`, `{{DONUT_2_TITLE}}`
- **`{{DONUT_1_PLACEHOLDER}}`, `{{DONUT_2_PLACEHOLDER}}`** — AI build native donut/pie
- `{{DONUT_1_SLICE_1_LABEL}}` ... `{{DONUT_1_SLICE_5_LABEL}}`
- `{{DONUT_1_SLICE_1_PCT}}` ... `{{DONUT_1_SLICE_5_PCT}}`
- `{{DONUT_1_SLICE_1_NOTE}}` ... `{{DONUT_1_SLICE_5_NOTE}}`
- (tương tự cho DONUT_2)
- `{{DONUT_1_SOURCE}}`, `{{DONUT_2_SOURCE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `pie` hoặc `donut` (hole 0.5), 5 slice max, color palette navy gradient.

---

### Disclaimer (1 layout)

#### LAYOUT 27 — `DISCLAIMER`
**Slide #:** 27. **Toàn bộ nội dung fix cứng**, chỉ có 1 placeholder.

- `{{PUBLISH_DATE}}`

---

## Decoupling — TEMPLATE_VBSE độc lập về authoring

TEMPLATE_VBSE là layout catalog đứng một mình về **authoring dependency**: nội dung layout (placeholder schema, design tokens, render rule) KHÔNG được derive từ spec của FORMAT.md hay WORKFLOW.md. Đây là rule architecture (xem `system_prompt.md` mục 2 — minimal cross-reference cho clarity runtime, không backward authoring dependency).

TEMPLATE_VBSE runtime chỉ consume 2 nguồn data:
- (a) file pack của chính nó (catalog `.md` + binary `.pptx`)
- (b) MD final đã đông kết do WORKFLOW Stage 5 produce

Reference đến WORKFLOW Stage 5/7 trong file này là pointer runtime (khi nào activate, đọc input từ đâu), không phải derive content.

Mỗi layout mô tả PURPOSE semantic của chính nó (cover khuyến nghị mã / scenario comparison 3-cell / heatmap 24 ngành / etc.) ở section "Danh sách 27 layout" trên. Agent runtime: scan MD final → match section heading + chart annotation với layout có sẵn theo semantic → clone layout → fill placeholder. Cùng 1 layout có thể fit nhiều loại MD content khác nhau, runtime tự match.

## Quy tắc bắt buộc khi AI render báo cáo qua TEMPLATE_VBSE

1. **Layout ID là duy nhất**. Slide name = LAYOUT_ID viết HOA_SNAKE (vd `COVER_A_RECOMMENDATION`). AI không được tự đổi tên slide khi clone — sẽ làm hỏng mapping.

2. **Placeholder pattern**. Mọi placeholder phải dạng `{{NAME}}` với UPPER_SNAKE_CASE. AI regex `\{\{[A-Z_0-9]+\}\}` để parse. Không tự thêm placeholder mới ngoài bộ đã định nghĩa trong file này.

3. **Reuse layout có sẵn**. TEMPLATE_VBSE có 27 layout cover gần hết use case. AI KHÔNG tự tạo layout mới khi render báo cáo cụ thể — clone layout phù hợp nhất rồi fill placeholder. Nếu thực sự cần layout mới → flag để user quyết định, không tự ý thêm.

4. **Chart placeholder strategy**. Layout có chart đặt rectangle `{{CHART_X_PLACEHOLDER}}` trong template. AI khi render: tìm shape có text này → ghi nhận vị trí (left, top, width, height) → remove shape → đọc YAML block tương ứng từ MD → add native PowerPoint chart vào đúng vị trí. KHÔNG vẽ chart bằng shape (rectangle giả lập bar).

5. **Không hardcode hex color khi fill**. Các thẻ giá trị có biến thiên theo trạng thái (positive/negative/neutral) — AI dùng đúng màu theo design token (`navy / red / green / text_dark / text_mid / text_light / bg_card / bg_pink`). Không hardcode `#FF0000` cho số dương, hay `#0000FF` cho số âm.

6. **Format số locale vi-VN**. Tiền/giá: dấu phẩy ngăn nghìn (vd `33.000 đ`). Phần trăm có dấu `+/-` rõ (vd `+18,0%`). Chỉ số: dấu phẩy thập phân (vd `1,37x`). KHÔNG dùng locale en-US (`33,000`) trong báo cáo VBSE.

7. **Constraint kịch bản 3 cell** (`COMPARISON_3CELL_SCENARIOS`). KHÔNG gán xác suất % vào kịch bản. Header dùng "Khả năng cao nhất / vừa phải / thấp" qualitative, không "60% / 30% / 10%". Đây là yêu cầu nội dung chung — MD final từ upstream pipeline phải đảm bảo không có % xác suất; TEMPLATE_VBSE chỉ render visual.

8. **Honest steelman 6-cell** (`RISK_GRID_STEELMAN`). 2 card cuối highlight hồng BẮT BUỘC là lo ngại còn thực sự yếu (chưa bị bác bỏ thuyết phục) — không phải "all win" với mọi rebut đều solid. TEMPLATE_VBSE chỉ render visual; nội dung CONCERN_5/6 trong MD final phải honest do upstream pipeline đảm bảo.

9. **Variant Perception 3 sub bắt buộc** (`VARIANT_PERCEPTION`). Cả 3 sub-section (Đồng thuận thị trường / Tâm lý NĐT cá nhân / Góc nhìn khác đồng thuận) phải fill. Không skip vì "không có data" — nếu không có thông tin, ghi rõ "Không quan sát được dữ liệu rõ rệt" thay vì để trống.

10. **Heatmap override màu theo magnitude** (`HEATMAP_INDUSTRY`). AI không giữ màu cell mặc định (bg_card). Override theo % biến động:
   - `>+3%` → fill navy `#003D7A`, text trắng
   - `+1% to +3%` → fill xanh `#059669`, text trắng
   - `-1% to +1%` → fill nhạt `#F7F9FC`, text đậm
   - `-3% to -1%` → fill hồng `#F5D5E0`, text đậm
   - `<-3%` → fill đỏ `#D71249`, text trắng

11. **Big stat constraint**. Vùng `{{KPI_VALUE}}` font 72pt trong panel hẹp — value tối đa 6-8 ký tự (vd "1,37x", "12,5%", "33.000"). Nếu value dài hơn → AI cần truncate hoặc dùng layout khác (như `COMPARISON_3CELL_SCENARIOS` cho big_pct).

12. **Tam giác signature**. Helper `add_decorative_triangle()` đã định nghĩa 4 vị trí chuẩn (`bottom_right`, `bottom_left`, `top_right`, `left_edge_navy`) — dùng tam giác vuông cân vẽ qua freeform. AI không tự thêm tam giác ở vị trí khác hoặc tự thay đổi shape.

13. **Self-check trước khi save**:
   - Mọi placeholder `{{...}}` đã được fill (regex check còn `\{\{[A-Z_]+\}\}` không?)
   - Các chart placeholder đã thay thế bằng native chart
   - Slide name (LAYOUT_ID) còn nguyên, không bị đổi
   - Số slide đầu cuối khớp với section count trong MD final
   - Format số tiếng Việt đồng nhất

14. **File source of truth — đừng tự sửa**:
   - `TEMPLATE_VBSE.pptx` là binary template, AI không sửa file này khi render báo cáo cá nhân (chỉ clone slide). Sửa template chỉ khi user yêu cầu update T pack.
   - Disclaimer body fix cứng — nếu user yêu cầu đổi disclaimer, sửa trực tiếp trong template, không tạo placeholder mới.

## Lịch sử update

- **2026-04-26 rev 1**: Khởi tạo TEMPLATE_VBSE pack. Build 24 layout: 5 cover + 1 section divider + 12 content + 5 layout mới + 1 disclaimer.
- **2026-04-26 rev 2**: Refactor theo feedback user:
  - Tam giác signature: đổi `RIGHT_TRIANGLE` → `ISOSCELES_TRIANGLE` cho đỉnh nhọn (theo mẫu user)
  - Fix cứng nội dung lặp lại: tên phòng ban, tên tổ, hotline, website, disclaimer body, các tiêu đề báo cáo
  - Bỏ author name placeholder
  - Redesign MINI_CHARTS_2X2: bỏ bar fake → chart placeholder rectangle
  - Thêm 3 layout chart mới: LINE_CHART_FULL, SCATTER_PEER, DONUT_COMPOSITION
  - Tổng 27 layout
  - Định nghĩa Chart placeholder strategy: AI parse `{{CHART_X_PLACEHOLDER}}` shape → build native PowerPoint chart từ YAML block trong MD, đặt vào đúng vị trí
- **2026-04-26 rev 3**: Refactor lần 2 theo feedback user:
  - Tam giác signature: đổi từ `ISOSCELES_TRIANGLE` (cân nhọn ~30°) → tam giác **vuông cân** vẽ qua `slide.shapes.build_freeform()`. Góc vuông 90° tại đỉnh nhô ra ngoài, hai cạnh kề bằng nhau, cạnh huyền dọc/ngang. Áp dụng cho 3 vị trí: COVER_A `left_edge_navy`, SECTION_DIVIDER `bottom_right`, DISCLAIMER `bottom_right`
  - `BIG_STAT_SUBPOINTS`: KPI_VALUE giảm font 96pt → 72pt, center alignment, anchor middle để render value ngắn nhìn cân đối
  - `THESIS_B_PANEL_RIGHT`: KPI_VALUE giảm 72pt → 56pt cho panel phải hẹp hơn
  - `LINE_CHART_FULL` + `SCATTER_PEER`: bỏ outer panel `bg_card` lồng nhau — đơn giản hoá thành single layer (title bar + chart placeholder + source bên dưới). Layout cleaner, dễ AI parse.
  - `SCATTER_PEER` bảng peer: cột Mã từ 1.10" → 1.40" để text PEER_X_TICKER không bị wrap
  - Học pattern từ `T_finext_00.md`: thêm section "Layout pattern toàn cục" với ASCII diagram + section "Quy tắc bắt buộc khi AI render" 14 rules cover toàn bộ constraint vận hành (layout ID, placeholder format, reuse, chart strategy, color, format số, scenario constraint, steelman, variant perception, heatmap, big stat, tam giác, self-check, source of truth)
- **2026-04-27 rev 4**: Siết rule strict independence của T pack theo audit kernel rev 5:
  - Bỏ tất cả backward references lên K/P/O packs trong file: bỏ "Dùng cho: P_xxx" ở mỗi layout description; bỏ bảng "Mapping layout → loại báo cáo"; thay reference "do O pack quy định" / "theo O pack spec" bằng wording neutral (chart annotation chuẩn trong MD source / section count trong MD final).
  - Pipeline render redraw: TEMPLATE_VBSE runtime chỉ đọc 2 nguồn = file pack chính nó + MD final. KHÔNG đọc K/P/O file. O ra MD final là chốt — T thuần visual filler trên content đông kết.
  - Thêm section "Decoupling — TEMPLATE_VBSE hoàn toàn độc lập" thay cho mapping table cũ.
  - Self-check item "khớp với O pack spec" → "khớp với section count trong MD final".
- **2026-04-27 rev 5**: Pack restructure. Rename `T_vbse_00.md` → `TEMPLATE_VBSE.md`, `T_vbse_01.pptx` → `TEMPLATE_VBSE.pptx`. Update body content: pipeline render reference `WORKFLOW.md` Stage 7 + decoupling rule reference `system_prompt.md` mục 2. Pack vẫn giữ nguyên 27 layout + design tokens + render rule — không thay đổi visual spec.
