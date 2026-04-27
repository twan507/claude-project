# TEMPLATE_FINEXT — Catalog Layout Brand Finext

Pack cung cấp template visual branded Finext cho render báo cáo pptx (dark-first, violet primary, modern tech-forward). Activate ở Stage 7 của `WORKFLOW.md` khi user pick brand Finext ở CP3.

TEMPLATE_FINEXT là layout catalog độc lập — chỉ đọc 2 nguồn: file pack của chính nó + MD final do WORKFLOW.md produce.

## Manifest pack

| File | Vai trò |
|---|---|
| `TEMPLATE_FINEXT.md` | Master catalog (file này) — design tokens, 27 layout, placeholder schema, chart strategy, render rule |
| `TEMPLATE_FINEXT.pptx` | Artifact binary 27 layout |

## Khi nào activate TEMPLATE_FINEXT

Activate khi user pick "(b) Finext" ở CP3 brand pre-flight (xem `WORKFLOW.md` mục 11). Điều kiện chung activate: user đã có MD final đúng `FORMAT.md` contract sau Stage 5.

Không activate khi: user pick VBSE, hoặc skip render binary (chỉ MD).

## Pipeline render

TEMPLATE_FINEXT đứng cuối pipeline workflow, chỉ consume MD final đã đông kết:

```
[Stage 1-5 của WORKFLOW.md đã chạy xong, output: MD final đúng FORMAT contract]
       │
       ▼
TEMPLATE_FINEXT runtime (Stage 7)
1. Đọc MD final → scan section heading + chart annotation block
2. Đọc TEMPLATE_FINEXT.md (file này) → biết layout nào có sẵn + purpose
3. Match semantic: section "Cover khuyến nghị MUA mã X" → COVER_A_RECOMMENDATION
4. Mở TEMPLATE_FINEXT.pptx → clone slide layout → fill placeholder từ MD content
5. Tại chart placeholder: build native PowerPoint chart từ YAML block trong MD
6. Save thành báo cáo pptx cuối
```

TEMPLATE_FINEXT runtime CHỈ đọc 2 nguồn: file pack của chính nó + MD final. Không đọc FORMAT.md, không đọc WORKFLOW.md (đó là context của upstream stage). Mọi nội dung cần thiết (số liệu, luận điểm, chart data, citation) đã có trong MD final.

## Chart placeholder strategy

TEMPLATE_FINEXT KHÔNG vẽ chart visual fake. Layout có chart đều dùng pattern **chart placeholder**: rectangle khung trống + label `{{CHART_PLACEHOLDER}}` mô tả loại chart.

Pattern này identical với TEMPLATE_VBSE. Chart annotation chuẩn trong MD source dạng YAML block (T consume theo quy ước này — không reference pack nào quy định):

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

Loại chart support: `bar`, `line`, `pie/donut`, `scatter`, `combo`. Không support: `bubble`, `stacked_bar` (rủi ro render sai).

## Design tokens

### Color palette (extract từ Finext brand tokens)

**Primary Violet** (signature brand):

| Token | Hex | Dùng cho |
|---|---|---|
| Violet chính | `#8B5CF6` | Primary brand, accent bar, button CTA, active state |
| Violet light | `#B47EFF` | Hover, secondary accent, gradient stop, label trên dark BG |
| Violet dark | `#7C3AED` | Pressed state, depth |
| Violet deep | `#4C1D95` | Deep tint cho hierarchical depth |
| Violet glow | `#2A1F4A` | Background tint subtle (mô phỏng glassmorphism trên dark) |

**Trend colors** (KHÔNG dùng success/error generic, dùng trend):

| Token | Hex | Ý nghĩa |
|---|---|---|
| Trend up | `#25B770` | Tăng giá / dòng tiền vào |
| Trend down | `#E14040` | Giảm giá / dòng tiền ra |
| Trend ref | `#FFC752` | Tham chiếu / sideway |
| Trend floor | `#0593BB` | Sàn / cyan accent |

**Background**:

| Token | Hex | Use case |
|---|---|---|
| BG dark | `#0A0A0A` | Cover, section divider, disclaimer (signature dark-first Finext) |
| BG card dark | `#1E1E1E` | Card trên dark BG |
| BG paper | `#FAFBFC` | Light paper (content slide) |
| BG card light | `#F5F5F5` | Card trên light BG |
| BG card violet | `#F3EEFF` | Violet tint card highlight |
| BG pink | `#FDE8F0` | Highlight error/warning |

**Text**:

| Token | Hex | Use case |
|---|---|---|
| Text dark | `#1F2937` | Body text trên light BG |
| Text mid | `#6B7280` | Sub-text trên light BG |
| Text light | `#9CA3AF` | Footer, muted, chart placeholder |
| Text inverse | `#F0F0F0` | Text chính trên dark BG |
| Text inverse mid | `#B8B8B8` | Sub text trên dark BG |
| Text inverse subtle | `#707070` | Disabled trên dark BG |

**Font**: Calibri (default Office, không cần embed).

**Slide size**: 16:9, 13.33" × 7.5".

### Signature pattern Finext (khác VBSE)

| Yếu tố | VBSE | Finext |
|---|---|---|
| Cover background | Half-bleed navy + white | **Full dark `#0A0A0A`** |
| Vertical bar accent | Đỏ solid | **Gradient violet → violet light** (2-stack) |
| Decorative shape | Tam giác vuông cân đỏ | **Chevron `>>`** (semantic "next/forward" — brand "Finext") |
| Numbered icon | Square đỏ | **Circle violet** (mềm mại, modern) |
| Card style | Flat rectangle | **Rounded corner** (signature border-radius) |
| Title divider | Solid red line | **Gradient 3-segment violet** (deep → main → light) |
| Trend display | Standard | Theo Finext trend palette (`up=#25B770`, `down=#E14040`) |

## Layout pattern toàn cục

Mọi slide content (trừ cover/divider/disclaimer) tuân theo cấu trúc 3-band:

```
┌─ Slide 13.33" × 7.5" (16:9) ────────────────────────────┐
│                                                          │
│ ▌  {{TITLE}}                          ← top=0.30"       │
│ ▌  {{SUB-TITLE}}                                         │
│ ────────── divider light ───────────                     │
│                                                          │
│   Content area                                           │
│     left=0.60"   width=12.13"                            │
│     top=1.40"    height=5.30"                            │
│                                                          │
│                                                          │
│   {{FOOTER_LEFT}}              {{PAGE_NUM}}  ← top=7.10" │
└──────────────────────────────────────────────────────────┘
```

**Title bar** — cao 1.10" từ top:
- Vertical violet **gradient bar** (2-stack `0.08" × 0.70"`) cạnh trái title (signature Finext)
- `{{TITLE}}` font Pt(24) bold dark
- `{{SUB-TITLE}}` font Pt(11) italic mid

**Content area** — `GRID = {content_l: 0.60, content_top: 1.40, content_w: 12.13, content_h: 5.30}`.

**Footer** — `top=7.10`.

**Cover slides (1-5)**: dark BG full, không tuân pattern title bar.
**Section divider (6) + Disclaimer (27)**: full dark BG, chevron decorative.

## Nội dung fix cứng (không placeholder)

| Trường | Giá trị fix cứng |
|---|---|
| Brand name | "FINEXT" / "Finext" |
| Phòng ban | "Phòng Phân tích Đầu tư" |
| Website | "www.finext.vn" |
| Hotline | "1900 0000" |
| Disclaimer body | 3 đoạn pháp lý chuẩn Finext (entity: "Công ty Cổ phần Chứng khoán Finext") |
| Tiêu đề "BÁO CÁO PHÂN TÍCH & KHUYẾN NGHỊ" trong COVER_A | Cố định |
| Tiêu đề "PHÂN TÍCH CHUYÊN SÂU" trong COVER_B | Cố định |
| Tiêu đề "BÁO CÁO THỊ TRƯỜNG TUẦN" trong COVER_C | Cố định |
| Header "TUYÊN BỐ MIỄN TRỪ TRÁCH NHIỆM" | Cố định |

## Cách AI parse template

Mỗi slide trong `TEMPLATE_FINEXT.pptx` có:

1. **Slide name** (XML attribute `cSld@name`) = LAYOUT_ID. AI dùng để identify layout khi clone.
2. **Placeholder** dạng `{{NAME}}` trong text run. AI regex match `\{\{[A-Z_0-9]+\}\}` → biết shape nào cần fill.
3. **Chart placeholder**: shape rectangle có text `{{CHART_X_PLACEHOLDER}}` đánh dấu chỗ AI build native chart.

Khi fill: thay nguyên text `{{NAME}}` bằng giá trị thực, giữ font size + color + alignment của run gốc.

## Danh sách 27 layout

### Cover (5 layout)

#### LAYOUT 01 — `COVER_A_RECOMMENDATION`
**Slide #:** 1. Cover khuyến nghị mua mã đơn lẻ. Full dark BG + violet glow tint top + ticker XL trái + MUA badge violet rounded + 2 stat dark card (giá hiện tại, giá mục tiêu) + thesis section.

**Placeholders:**
- `{{REPORT_SUBTITLE}}`
- `{{TICKER}}` (≤ 5 ký tự, font 96pt)
- `{{COMPANY_FULL_NAME}}`, `{{INDUSTRY_LABEL}}`
- `{{PRICE_CURRENT}}`, `{{PRICE_TARGET}}`
- `{{UPSIDE_LINE}}`
- `{{THESIS_HEADLINE}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 02 — `COVER_B_DEEPDIVE`
**Slide #:** 2. Cover phân tích chuyên sâu mã đơn lẻ. Full dark BG + gradient violet bar top + ticker centered + 4 stat dark card với violet bar trái.

**Placeholders:**
- `{{TICKER}}` (≤ 6 ký tự)
- `{{REPORT_SUBTITLE}}`, `{{COMPANY_FULL_NAME}}`, `{{INDUSTRY_LABEL}}`
- `{{PRICE_CURRENT}}`, `{{PRICE_DATE}}`
- `{{TARGET_BASE}}`, `{{UPSIDE_BASE}}`
- `{{STOPLOSS}}`, `{{DOWNSIDE_STOP}}`
- `{{RR_RATIO}}`, `{{RR_VERDICT}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 03 — `COVER_C_WEEKLY_MARKET`
**Slide #:** 3. Cover báo cáo thị trường tuần. Top section violet glow tint + week label XL + regime badge violet rounded + 3 KPI dark card.

**Placeholders:**
- `{{WEEK_LABEL}}`, `{{HEADLINE_LINE}}`, `{{REGIME_BADGE}}`
- `{{VNINDEX_CLOSE}}`, `{{VNINDEX_CHANGE}}`
- `{{LIQUIDITY_AVG}}`, `{{LIQUIDITY_NOTE}}`
- `{{FOREIGN_NET}}`, `{{FOREIGN_NOTE}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 04 — `COVER_D_MACRO_SECTOR`
**Slide #:** 4. Cover chủ đề vĩ mô / phân tích ngành. Full dark BG + vertical gradient bar trái full height + 3 takeaway numbered (circle icon).

**Placeholders:**
- `{{THEME_TAG}}`, `{{REPORT_TITLE}}`, `{{REPORT_SUBTITLE}}`
- `{{TAKEAWAY_1}}` ... `{{TAKEAWAY_3}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 05 — `COVER_E_GENERIC`
**Slide #:** 5. Cover tổng quát light BG minimalist. Khác biệt với 4 cover dark khác — dùng cho báo cáo định kỳ, ad-hoc với tone nhẹ hơn.

**Placeholders:**
- `{{REPORT_KIND_TAG}}`, `{{REPORT_TITLE}}`, `{{REPORT_SUBTITLE}}`
- `{{PUBLISH_DATE}}`

---

### Section divider (1 layout)

#### LAYOUT 06 — `SECTION_DIVIDER`
**Slide #:** 6. Full dark BG + chevron decorative phải dưới + section info trái.

**Placeholders:**
- `{{SECTION_TAG}}`, `{{SECTION_TITLE}}`, `{{SECTION_DESC}}`

---

### Content layout (12)

Cấu trúc giống TEMPLATE_VBSE 1-1, chỉ khác visual style (violet thay đỏ navy, rounded card thay flat, circle icon thay square, gradient bar thay solid).

#### LAYOUT 07 — `BULLET_LIST_SUMMARY`
**Slide #:** 7. 5 bullet với violet circle marker. Dùng cho Weekly Exec Summary.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{BULLET_1_LABEL}}` ... `{{BULLET_5_LABEL}}`
- `{{BULLET_1_BODY}}` ... `{{BULLET_5_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 08 — `STAT_CALLOUT_GRID_4CELL`
**Slide #:** 8. 4 stat callout rounded + section band violet + 4 numbered cards (circle icon). Phù hợp cho slide tóm tắt khuyến nghị (4 mức giá target/stop + 4 luận điểm cốt lõi).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{TARGET_SHORT}}`, `{{UPSIDE_SHORT}}`, `{{TARGET_MID}}`, `{{UPSIDE_MID}}`
- `{{STOPLOSS}}`, `{{DOWNSIDE_STOP}}`, `{{RR_RATIO}}`, `{{RR_VERDICT}}`
- `{{POINT_1_TITLE}}` ... `{{POINT_4_TITLE}}`, `{{POINT_1_BODY}}` ... `{{POINT_4_BODY}}` (≤ 100 ký tự)
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 09 — `BIG_STAT_SUBPOINTS`
**Slide #:** 9. Big stat panel violet rounded trái (KPI_VALUE 72pt center) + 3 sub-points cards rounded + bảng KPI bg violet tint.
- `{{LUANDIEM_TITLE}}`, `{{LUANDIEM_SUB}}`
- `{{KPI_LABEL}}`, `{{KPI_VALUE}}` (≤ 6 ký tự, 72pt), `{{KPI_NOTE}}`, `{{KPI_SUB}}`
- `{{SUBPOINT_1_TITLE}}` ... `{{SUBPOINT_3_TITLE}}`
- `{{SUBPOINT_1_BODY}}` ... `{{SUBPOINT_3_BODY}}`
- `{{KPI_ROW_1_LABEL}}` ... `{{KPI_ROW_5_LABEL}}`
- `{{KPI_ROW_1_VALUE}}` ... `{{KPI_ROW_5_VALUE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 10 — `TWO_COLUMN_INFO_BUSINESS`
**Slide #:** 10. Hồ sơ DN 2 cột rounded card.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{INFO_1}}` ... `{{INFO_9}}` (Mã CP, Ngày NY, Vốn ĐL, Số CP LH, Vốn hoá, Tự do CN, Cổ đông lớn, NN%, Ngành)
- `{{SEGMENT_1_TITLE}}` ... `{{SEGMENT_5_TITLE}}`
- `{{SEGMENT_1_BODY}}` ... `{{SEGMENT_5_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 11 — `COMPARISON_3CELL_SCENARIOS`
**Slide #:** 11. Top action band violet rounded + 3 kịch bản: Cơ sở (violet) / Tích cực (xanh) / Thận trọng (đỏ).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`, `{{ACTION_RECOMMENDED}}`
- `{{BASE_TIMEFRAME}}`, `{{BASE_TARGET}}`, `{{BASE_RETURN}}`, `{{BASE_BIG_PCT}}`, `{{BASE_BODY}}`
- `{{BULL_TIMEFRAME}}`, `{{BULL_TARGET}}`, `{{BULL_RETURN}}`, `{{BULL_BIG_PCT}}`, `{{BULL_BODY}}`
- `{{BEAR_TIMEFRAME}}`, `{{BEAR_TARGET}}`, `{{BEAR_RETURN}}`, `{{BEAR_BIG_PCT}}`, `{{BEAR_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** KHÔNG gán xác suất % vào kịch bản. Header dùng "Khả năng cao nhất / vừa phải / thấp" qualitative.

#### LAYOUT 12 — `STAT_TABLE_COMBO_SESSION`
**Slide #:** 12. 4 stat callout rounded + bảng 8 mức kỹ thuật header violet.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CLOSE_PRICE}}`, `{{CLOSE_CHANGE}}`, `{{HIGH_PRICE}}`, `{{HIGH_NOTE}}`
- `{{LOW_PRICE}}`, `{{LOW_NOTE}}`, `{{VALUE_AMOUNT}}`, `{{LIQUIDITY_NOTE}}`
- `{{LEVEL_1_NAME}}` ... `{{LEVEL_8_NAME}}` (cao→thấp: Kháng cự R1, Fibo 38.2%, Biên trên, POC, Giá hiện tại, Biên dưới, Hỗ trợ S1, MA60)
- `{{LEVEL_1_PRICE}}` ... `{{LEVEL_8_PRICE}}`, `{{LEVEL_1_DELTA}}` ... `{{LEVEL_8_DELTA}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 13 — `DATA_TABLE_FULL`
**Slide #:** 13. Bảng full 8×8 header violet.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{COL_1_HEAD}}` ... `{{COL_8_HEAD}}`
- `{{R{i}_C{j}}}` cho i=1..8, j=1..8
- `{{TABLE_NOTE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 14 — `TIMELINE_NEWS`
**Slide #:** 14. Timeline 4 sự kiện trục dọc violet light, marker circle violet, card rounded.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{EVENT_1_DATE}}` ... `{{EVENT_4_DATE}}`
- `{{EVENT_1_TITLE}}` ... `{{EVENT_4_TITLE}}`
- `{{EVENT_1_BODY}}` ... `{{EVENT_4_BODY}}`
- `{{EVENT_1_TAG}}` ... `{{EVENT_4_TAG}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 15 — `RISK_GRID_STEELMAN`
**Slide #:** 15. Grid 6 cards lo ngại + phản biện. 2 card cuối highlight pink (lo ngại còn yếu).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CONCERN_1}}` ... `{{CONCERN_6}}` (CONCERN_5, 6 = lo ngại còn yếu)
- `{{REBUT_1}}` ... `{{REBUT_6}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** CONCERN_5/6 BẮT BUỘC là lo ngại thực sự yếu sau phản biện (honest steelman), không "all win" — đây là yêu cầu nội dung chung cho mọi báo cáo dùng layout này.

#### LAYOUT 16 — `VARIANT_PERCEPTION`
**Slide #:** 16. Insight box violet rounded với quote mark + 3 sub-section: đồng thuận / tâm lý NĐT / góc nhìn khác.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{INSIGHT_KEY}}`
- `{{CONSENSUS_VIEW}}`, `{{RETAIL_VIEW}}`, `{{DIFFERENT_VIEW}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** Cả 3 sub-section bắt buộc fill.

#### LAYOUT 17 — `BUY_STRUCTURE_FLEX`
**Slide #:** 17. Cấu trúc mua N lớp linh hoạt 2-5 lớp + risk band. Header layer violet rounded với label tag + action prominent.
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

#### LAYOUT 18 — `TAKE_PROFIT_TARGETS`
**Slide #:** 18. 4 mốc chốt M1-M4 với tag violet rounded + big price center + bottom band violet R/R summary.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{M1_NUM}}` ... `{{M4_NUM}}`
- `{{M1_PRICE}}` ... `{{M4_PRICE}}`
- `{{M1_PROFIT_PCT}}` ... `{{M4_PROFIT_PCT}}`
- `{{M1_RATIO}}` ... `{{M4_RATIO}}`
- `{{M1_BASIS}}` ... `{{M4_BASIS}}`
- `{{AVG_PROFIT}}`, `{{MAX_RISK}}`, `{{RR_RATIO}}`, `{{MAX_TARGET}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 19 — `THESIS_B_PANEL_RIGHT`
**Slide #:** 19. Alternative cho LAYOUT_09. Big stat panel violet phải + 4 sub-points 2x2 trái với circle numbered icon.
- `{{LUANDIEM_TITLE}}`, `{{LUANDIEM_SUB}}`
- `{{SUBPOINT_1_TITLE}}` ... `{{SUBPOINT_4_TITLE}}`
- `{{SUBPOINT_1_BODY}}` ... `{{SUBPOINT_4_BODY}}`
- `{{KPI_LABEL}}`, `{{KPI_VALUE}}` (≤ 6 ký tự, 56pt), `{{KPI_NOTE}}`, `{{KPI_LONG_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 20 — `HEATMAP_INDUSTRY`
**Slide #:** 20. 24 ngành 6×4 grid rounded card + legend thang màu 5 mức.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{IND_1_NAME}}` ... `{{IND_24_NAME}}`
- `{{IND_1_PCT}}` ... `{{IND_24_PCT}}`
- `{{IND_1_NOTE}}` ... `{{IND_24_NOTE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Constraint:** AI override màu cell theo magnitude %:
- `>+3%` → fill `#8B5CF6` (violet), text trắng
- `+1% to +3%` → fill `#25B770` (xanh), text trắng
- `-1% to +1%` → fill `#F5F5F5` (trung tính), text đậm
- `-3% to -1%` → fill `#FDE8F0` (hồng), text đậm
- `<-3%` → fill `#E14040` (đỏ), text trắng

#### LAYOUT 21 — `PEER_COMPARE_TABLE`
**Slide #:** 21. So sánh 8 mã header violet rounded. P1 highlight bg violet tint.
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
**Slide #:** 22. 4 chart placeholder 2×2, card rounded + title bar violet rounded. Phù hợp cho block 4 chart chỉ số tài chính 5 năm hoặc 4 stat trend song song.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_1_TITLE}}` ... `{{CHART_4_TITLE}}`
- `{{CHART_1_UNIT}}` ... `{{CHART_4_UNIT}}`
- **`{{CHART_1_PLACEHOLDER}}` ... `{{CHART_4_PLACEHOLDER}}`** — AI thay bằng native chart
- `{{CHART_1_LATEST}}` ... `{{CHART_4_LATEST}}`
- `{{CHART_1_CHANGE}}` ... `{{CHART_4_CHANGE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `bar` (column clustered), 5 năm, color violet.

#### LAYOUT 23 — `CALENDAR_WEEK`
**Slide #:** 23. Lịch 7 ngày Thứ 2 - CN. Header violet (T2-T6) hoặc text light grey (T7-CN).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{DAY_1_DATE}}` ... `{{DAY_7_DATE}}`
- `{{DAY_1_EVT_1_TIME}}` ... `{{DAY_7_EVT_4_TIME}}`
- `{{DAY_1_EVT_1_TITLE}}` ... `{{DAY_7_EVT_4_TITLE}}`
- `{{DAY_1_COUNT}}` ... `{{DAY_7_COUNT}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

---

### Chart placeholder layouts (3 layout)

#### LAYOUT 24 — `LINE_CHART_FULL`
**Slide #:** 24. Chart đường full width 2/3 + 3 commentary cards phải, single layer (title bar rounded violet + chart placeholder rounded).
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_TITLE}}`
- **`{{CHART_PLACEHOLDER}}`** — AI build native line chart
- `{{CHART_SOURCE}}`
- `{{NOTE_1_TITLE}}`, `{{NOTE_1_BODY}}`
- `{{NOTE_2_TITLE}}`, `{{NOTE_2_BODY}}`
- `{{NOTE_3_TITLE}}`, `{{NOTE_3_BODY}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

#### LAYOUT 25 — `SCATTER_PEER`
**Slide #:** 25. Scatter plot định vị peer 2D + bảng peer rút gọn phải.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{CHART_TITLE}}`, `{{Y_AXIS_LABEL}}`, `{{X_AXIS_LABEL}}`
- **`{{CHART_PLACEHOLDER}}`** — AI build native scatter
- `{{CHART_SOURCE}}`
- `{{PEER_1_TICKER}}` ... `{{PEER_6_TICKER}}` (PEER_1 = highlight violet)
- `{{PEER_1_NOTE}}` ... `{{PEER_6_NOTE}}`
- `{{PEER_VERDICT}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `scatter`, mã đang phân tích = marker violet, peer khác = marker dark.

#### LAYOUT 26 — `DONUT_COMPOSITION`
**Slide #:** 26. 2 donut chart side-by-side + legend với circle dot violet.
- `{{SECTION_TITLE}}`, `{{SECTION_SUB}}`
- `{{DONUT_1_TITLE}}`, `{{DONUT_2_TITLE}}`
- **`{{DONUT_1_PLACEHOLDER}}`, `{{DONUT_2_PLACEHOLDER}}`** — AI build native donut/pie
- `{{DONUT_1_SLICE_1_LABEL}}` ... `{{DONUT_1_SLICE_5_LABEL}}`
- `{{DONUT_1_SLICE_1_PCT}}` ... `{{DONUT_1_SLICE_5_PCT}}`
- `{{DONUT_1_SLICE_1_NOTE}}` ... `{{DONUT_1_SLICE_5_NOTE}}`
- (tương tự cho DONUT_2)
- `{{DONUT_1_SOURCE}}`, `{{DONUT_2_SOURCE}}`
- `{{FOOTER_LEFT}}`, `{{PAGE_NUM}}`

**Chart spec mặc định:** type `pie` hoặc `donut` (hole 0.5), 5 slice max, palette violet gradient.

---

### Disclaimer (1 layout)

#### LAYOUT 27 — `DISCLAIMER`
**Slide #:** 27. Full dark BG + chevron decorative phải dưới + disclaimer body fix cứng. Toàn bộ nội dung fix cứng, chỉ 1 placeholder.
- `{{PUBLISH_DATE}}`

---

## Decoupling — TEMPLATE_FINEXT hoàn toàn độc lập

TEMPLATE_FINEXT là layout catalog đứng một mình. File này KHÔNG reference FORMAT.md, WORKFLOW.md, hay file pack khác trong template_agent — đó là backward dependency vi phạm rule architecture (xem `system_prompt.md` mục 2).

TEMPLATE_FINEXT chỉ đọc 2 nguồn khi runtime render: (a) file pack của chính nó (catalog `.md` + binary `.pptx`), (b) MD final do WORKFLOW Stage 5 produce làm input nội dung.

Mỗi layout mô tả PURPOSE semantic của chính nó (cover khuyến nghị mã / scenario comparison 3-cell / heatmap 24 ngành / etc.) ở section "Danh sách 27 layout" trên. Agent runtime: scan MD final → match section heading + chart annotation với layout có sẵn theo semantic → clone layout → fill placeholder. Cùng 1 layout có thể fit nhiều loại MD content khác nhau, runtime tự match.

Cấu trúc layout TEMPLATE_FINEXT 1-1 với TEMPLATE_VBSE (cùng LAYOUT_ID), khác visual style (dark/violet/chevron vs navy/red/triangle) — runtime chọn T pack nào dựa trên brand audience user yêu cầu, không phải dựa trên loại báo cáo.

## Quy tắc bắt buộc khi AI render báo cáo qua TEMPLATE_FINEXT

1. **Layout ID là duy nhất**. Slide name = LAYOUT_ID viết HOA_SNAKE. AI không được tự đổi tên slide khi clone.

2. **Placeholder pattern**. Mọi placeholder phải dạng `{{NAME}}` với UPPER_SNAKE_CASE. AI regex `\{\{[A-Z_0-9]+\}\}` để parse. Không tự thêm placeholder mới.

3. **Reuse layout có sẵn**. TEMPLATE_FINEXT có 27 layout cover gần hết use case. AI KHÔNG tự tạo layout mới — clone layout phù hợp nhất rồi fill placeholder. Cần layout mới → flag để user quyết định.

4. **Chart placeholder strategy**. Layout có chart đặt rectangle `{{CHART_X_PLACEHOLDER}}`. AI khi render: tìm shape có text này → ghi nhận vị trí → remove shape → đọc YAML block từ MD → add native PowerPoint chart vào đúng vị trí. KHÔNG vẽ chart bằng shape.

5. **Không hardcode hex color khi fill**. Dùng đúng màu theo design token (`violet / violet_light / trend_up / trend_down / text_dark / text_inverse / bg_card_*`). KHÔNG hardcode `#FF0000` cho số dương hay `#0000FF` cho số âm.

6. **Trend color helper**. Giá CP, %change, dòng tiền → dùng đúng `trend_up / trend_down / trend_ref / trend_floor`. KHÔNG ternary `value > 0 ? green : red` hardcode.

7. **Format số locale vi-VN**. Tiền/giá: `33.000 đ`. Phần trăm có dấu `+/-` (vd `+18,0%`). Chỉ số dùng dấu phẩy thập phân (`1,37x`).

8. **Constraint kịch bản 3 cell** (`COMPARISON_3CELL_SCENARIOS`). KHÔNG gán xác suất % vào kịch bản. Header dùng "Khả năng cao nhất / vừa phải / thấp" qualitative. Đây là yêu cầu nội dung chung — MD final từ upstream pipeline phải đảm bảo không có % xác suất; TEMPLATE_FINEXT chỉ render visual.

9. **Honest steelman 6-cell** (`RISK_GRID_STEELMAN`). 2 card cuối highlight pink BẮT BUỘC là lo ngại còn thực sự yếu sau phản biện, không "all win". TEMPLATE_FINEXT chỉ render visual; nội dung CONCERN_5/6 trong MD final phải honest do upstream pipeline đảm bảo.

10. **Variant Perception 3 sub bắt buộc**. Cả 3 sub-section phải fill. Không skip vì "không có data" — nếu không có thông tin, ghi "Không quan sát được dữ liệu rõ rệt".

11. **Heatmap override màu theo magnitude** (`HEATMAP_INDUSTRY`). Override theo % biến động (xem LAYOUT 20 constraint).

12. **Big stat constraint**. Vùng `{{KPI_VALUE}}` font 72pt panel hẹp — value tối đa 6-8 ký tự. Nếu dài hơn → AI truncate hoặc dùng layout khác.

13. **Chevron signature**. Helper `add_chevron_decorative()` đã định nghĩa 4 vị trí chuẩn (`bottom_right`, `bottom_left`, `top_right`, `dark_cover_accent`) — dùng `MSO_SHAPE.CHEVRON`. AI không tự thêm chevron ở vị trí khác.

14. **Self-check trước khi save**:
   - Mọi placeholder `{{...}}` đã được fill (regex check còn `\{\{[A-Z_]+\}\}` không?)
   - Các chart placeholder đã thay thế bằng native chart
   - Slide name (LAYOUT_ID) còn nguyên, không bị đổi
   - Số slide đầu cuối khớp với section count trong MD final
   - Format số tiếng Việt đồng nhất

15. **File source of truth — đừng tự sửa**:
   - `TEMPLATE_FINEXT.pptx` là binary template, AI không sửa file này khi render báo cáo cá nhân (chỉ clone slide). Sửa template chỉ khi user yêu cầu update T pack.
   - Disclaimer body fix cứng — nếu user yêu cầu đổi disclaimer, sửa trực tiếp trong template, không tạo placeholder mới.

## Lịch sử update

- **2026-04-26 rev 1**: Khởi tạo TEMPLATE_FINEXT pack pptx. Build 27 layout cấu trúc 1-1 mapping với TEMPLATE_VBSE: 5 cover (4 dark + 1 light) + 1 section divider + 12 content + 5 extension + 3 chart placeholder + 1 disclaimer. Visual style khác biệt với TEMPLATE_VBSE: dark-first cover (signature Finext), violet primary thay đỏ navy, chevron decorative `>>` thay tam giác (semantic "next/forward"), rounded card thay flat rectangle, circle numbered icon thay square, vertical gradient bar 2-stack thay solid bar, gradient 3-segment violet thay solid line. Trend color theo Finext palette (`up=#25B770`, `down=#E14040`, `ref=#FFC752`, `floor=#0593BB`). Color BG dark `#0A0A0A` cho cover/divider/disclaimer. Nội dung lặp lại fix cứng: brand name "Finext", phòng ban "Phòng Phân tích Đầu tư", website "www.finext.vn", hotline "1900 0000", disclaimer body Finext entity. Chart placeholder strategy identical với TEMPLATE_VBSE (decoupled): AI build native chart từ YAML block trong MD source. Tài liệu master gồm: design tokens, layout pattern toàn cục với ASCII diagram, 27 layout với placeholder schema, mapping layout → loại báo cáo, 15 quy tắc bắt buộc khi AI render. Decoupling rule giữ nguyên: O ⟂ T, một O có thể render qua nhiều T (VBSE/Finext/internal), một T layout fit nhiều O pack.
- **2026-04-27 rev 2**: Register vào kernel skeleton (rev 5) + siết rule strict independence của T pack:
  - Bỏ tất cả backward references lên K/P/O packs trong file: bỏ "Dùng cho: P_xxx" ở mỗi layout description; bỏ bảng "Mapping layout → loại báo cáo"; thay reference "do O pack quy định" / "theo O pack spec" bằng wording neutral (chart annotation chuẩn trong MD source / section count trong MD final).
  - Pipeline render redraw: TEMPLATE_FINEXT runtime chỉ đọc 2 nguồn = file pack chính nó + MD final. KHÔNG đọc K/P/O file. O ra MD final là chốt — T thuần visual filler trên content đông kết.
  - Thêm section "Decoupling — TEMPLATE_FINEXT hoàn toàn độc lập" thay cho mapping table cũ.
  - Self-check item "khớp với O pack spec" → "khớp với section count trong MD final".
- **2026-04-27 rev 3**: Pack restructure. Rename `T_finext_00.md` → `TEMPLATE_FINEXT.md`, `T_finext_01.pptx` → `TEMPLATE_FINEXT.pptx`. Update body content: pipeline render reference `WORKFLOW.md` Stage 7 + decoupling rule reference `system_prompt.md` mục 2. Bỏ note "Pack code FE đặt thành T_finextapp_00" — naming convention dùng TEMPLATE_X. Pack vẫn giữ nguyên 27 layout + design tokens + render rule — không thay đổi visual spec.
