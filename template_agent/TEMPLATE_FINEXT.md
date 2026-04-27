# TEMPLATE_FINEXT — Catalog Layout Brand Finext

Pack cung cấp template visual branded Finext cho render báo cáo pptx (**editorial fintech magazine style**: light BG, violet primary, asymmetric layout, typography-driven hero). Activate ở Stage 7 của `WORKFLOW.md` khi user pick brand Finext ở CP3.

**Design philosophy** — đối lập với TEMPLATE_VBSE để hai brand có visual identity rõ rệt khi cùng nội dung render ra:

| Trục | TEMPLATE_VBSE | TEMPLATE_FINEXT |
|---|---|---|
| Tinh thần | Brokerage classic — formal, conservative, dày data | Editorial fintech — modern, asymmetric, magazine-style |
| BG | White trên content + Navy half-bleed cover | **White toàn bộ** (mọi slide) |
| Hero element | Big number / ticker XL | **Thesis / quote / typography hero** (number xuống vai trò support) |
| Layout symmetry | Symmetric grid, evenly distributed | **Asymmetric** (golden ratio 1:2 hoặc 2:3, dominant + supporting) |
| Title bar | Top horizontal | **Mix top horizontal + side vertical band** (per layout) |
| Decorative | Tam giác vuông cân đỏ | **Chevron violet + horizontal/vertical violet rule** |
| Card | Flat rectangle sharp corner | **Rounded corner** (10000–25000 EMU) |
| Numbered icon | Square red | **Circle violet** |
| Bar accent | Solid red 0.08"×0.70" | **Gradient violet 2-stack** (chính + light) hoặc full-height side band |

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

### Color palette

**Primary Violet** (signature brand):

| Token | Hex | Dùng cho |
|---|---|---|
| Violet chính | `#8B5CF6` | Primary brand, accent bar, button CTA, active state |
| Violet light | `#B47EFF` | Hover, secondary accent, gradient stop dưới của 2-stack bar |
| Violet dark | `#7C3AED` | Pressed state, big stat color, header text emphasis |
| Violet deep | `#4C1D95` | Deep tint cho hierarchical depth |
| Violet tint | `#F3EEFF` | Card highlight nhạt, soft background tint, decorative chevron BG |

**Trend colors** (KHÔNG dùng success/error generic, dùng trend):

| Token | Hex | Ý nghĩa |
|---|---|---|
| Trend up | `#25B770` | Tăng giá / dòng tiền vào |
| Trend down | `#E14040` | Giảm giá / dòng tiền ra |
| Trend ref | `#FFC752` | Tham chiếu / sideway |
| Trend floor | `#0593BB` | Sàn / cyan accent (dùng cho stat trung tính) |

**Background & border**:

| Token | Hex | Use case |
|---|---|---|
| BG white | `#FFFFFF` | Slide background mặc định (mọi slide) |
| BG paper | `#FAFBFC` | Soft paper alternative khi cần chút khác biệt |
| BG card | `#F5F5F5` | Card neutral trên BG trắng |
| BG violet | `#F3EEFF` | Card highlight violet (cho luận điểm key, summary card) |
| BG pink | `#FDE8F0` | Highlight error / "lo ngại còn yếu" |
| **Border light** | **`#E5E7EB`** | Border, divider, separator nhẹ — token mới thêm |

**Text**:

| Token | Hex | Use case |
|---|---|---|
| Text dark | `#1F2937` | Body text chính trên BG light |
| Text mid | `#6B7280` | Sub-text, label |
| Text light | `#9CA3AF` | Footer, muted, chart placeholder text |
| Text inverse | `#FFFFFF` | Text trên violet fill (button, badge, side band) |

**Note**: TEMPLATE_FINEXT phiên bản trước có dark-first identity với BG `#0A0A0A`, BG card dark `#1E1E1E`, và 3 text inverse tokens (`F0F0F0`, `B8B8B8`, `707070`) — tất cả đã loại bỏ ở phiên bản hiện tại. Nếu cần text trên fill violet/ violet_dark, dùng `text_inverse = #FFFFFF`.

**Font**: Calibri (default Office, không cần embed).

**Slide size**: 16:9, 13.33" × 7.5".

### Signature pattern Finext

Đã liệt kê ở bảng đối lập VBSE/Finext đầu file. Chi tiết shape sử dụng:

- **Vertical violet bar 2-stack** `0.08" × 0.70"`: top half `#8B5CF6`, bottom half `#B47EFF`. Đặt cạnh trái title bar (mọi slide content layout 7-26).
- **Side vertical band**: `0.30"–1.50"` rộng, full height, fill `#8B5CF6`. Dùng cho COVER_B (0.30"), SECTION_DIVIDER (1.50"), DISCLAIMER (1.0").
- **Chevron decorative**: `MSO_SHAPE.CHEVRON`. Dùng ở COVER_E (top-right tint), SECTION_DIVIDER (trên side band), DISCLAIMER (trên side band).
- **Horizontal violet rule**: `0.04"` cao, full hoặc partial width. Title divider, footer separator.
- **Rounded card**: corner radius 10000–25000 EMU. Dùng cho mọi card content.
- **Circle numbered icon**: violet fill, white text. Dùng cho takeaway/subpoint/layer numbering.

## Layout pattern toàn cục

**Slide background**: `#FFFFFF` mặc định cho mọi slide (inherit từ slide master, không có `<p:bg>` hay full-slide rect cover).

**Cover slides (1-5)**: KHÔNG tuân pattern title-bar 3-band của content. Mỗi cover có editorial composition riêng (xem chi tiết LAYOUT 01-05 dưới).

**Section divider (6) + Disclaimer (27)**: KHÔNG tuân pattern title-bar 3-band. Dùng side band trái + content phải.

**Content slides (7-26)** tuân theo cấu trúc 3-band:

```
┌─ Slide 13.33" × 7.5" (16:9) — BG #FFFFFF ──────────────┐
│                                                          │
│ ▌  {{TITLE}}                          ← top=0.30"       │
│ ▌  {{SUB-TITLE}}                                         │
│ ────────── divider violet rule ───────────               │
│                                                          │
│   Content area                                           │
│     left=0.60"   width=12.13"                            │
│     top=1.40"    height=5.30"                            │
│                                                          │
│                                                          │
│   {{FOOTER_LEFT}}              {{PAGE_NUM}}  ← top=7.10" │
└──────────────────────────────────────────────────────────┘
```

- **Title bar** (cao 1.10" từ top): vertical violet bar 2-stack `0.08" × 0.70"` cạnh trái + `{{TITLE}}` Pt(24) bold dark + `{{SUB-TITLE}}` Pt(11) italic mid.
- **Content area** GRID `{content_l: 0.60, content_top: 1.40, content_w: 12.13, content_h: 5.30}`.
- **Footer** `top=7.10`.

## Nội dung fix cứng (không placeholder)

| Trường | Giá trị fix cứng |
|---|---|
| Brand name | "Finext" (Title Case — đồng nhất mọi slide) |
| Phòng ban | "Phòng Phân tích Đầu tư" |
| Website | "www.finext.vn" |
| Hotline | "1900 0000" |
| Footer brand string | `Finext • Phòng Phân tích Đầu tư` (single space + bullet + single space) |
| Disclaimer body | 3 đoạn pháp lý chuẩn Finext (entity: "Công ty Cổ phần Chứng khoán Finext"), mỗi đoạn là 1 paragraph riêng |
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
**Slide #:** 1. Cover khuyến nghị mua mã đơn lẻ — **editorial pitch style**.

**Composition** (khác VBSE classic ticker-XL hero):
- Top violet rule full-width + masthead row (FINEXT brand left, badge "KHUYẾN NGHỊ MUA" violet rounded right)
- Subtle border separator
- Subtitle line italic
- **Ticker as compact violet pill tag** (1.4"×0.45", rounded) + company full name italic (giống editorial subhead)
- **Hero zone**: large quote mark (60pt violet `"`) + `{{THESIS_HEADLINE}}` 36pt bold (thesis là hero, không phải ticker)
- Compact 3-cell stat row bottom: Giá hiện tại / Giá mục tiêu / Tiềm năng — mỗi cell có vertical violet rule trái 0.05"
- Bottom violet rule + footer date + website/hotline

**Placeholders:**
- `{{REPORT_SUBTITLE}}`
- `{{TICKER}}` (≤ 5 ký tự, font 18pt trong pill)
- `{{COMPANY_FULL_NAME}}`, `{{INDUSTRY_LABEL}}`
- `{{PRICE_CURRENT}}`, `{{PRICE_TARGET}}`
- `{{UPSIDE_LINE}}`
- `{{THESIS_HEADLINE}}` (≤ 90 ký tự, là hero element 36pt bold)
- `{{PUBLISH_DATE}}`

#### LAYOUT 02 — `COVER_B_DEEPDIVE`
**Slide #:** 2. Cover phân tích chuyên sâu — **asymmetric R/R hero style**.

**Composition** (khác VBSE 4-stat 4-cell evenly):
- Left full-height violet band 0.30" wide
- Top section: tag "PHÂN TÍCH CHUYÊN SÂU" violet + subtitle italic 22pt
- **Left side**: ticker XL 84pt bold violet_dark (hero positioning) + company italic + industry label dưới
- **Right side asymmetric**: R/R card violet_tint dominant (6.0"×2.30", rounded 15000) — `{{RR_RATIO}}` 72pt bold, `{{RR_VERDICT}}` italic
- 3 supporting stats inline below R/R: Giá hiện tại / Mục tiêu / Cắt lỗ — mỗi cell có top color rule (violet/up/down)
- Footer Finext brand string

**Placeholders:**
- `{{TICKER}}` (≤ 6 ký tự, font 84pt)
- `{{REPORT_SUBTITLE}}`, `{{COMPANY_FULL_NAME}}`, `{{INDUSTRY_LABEL}}`
- `{{PRICE_CURRENT}}`, `{{PRICE_DATE}}`
- `{{TARGET_BASE}}`, `{{UPSIDE_BASE}}`
- `{{STOPLOSS}}`, `{{DOWNSIDE_STOP}}`
- `{{RR_RATIO}}` (hero stat 72pt), `{{RR_VERDICT}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 03 — `COVER_C_WEEKLY_MARKET`
**Slide #:** 3. Cover báo cáo thị trường tuần — **magazine masthead style**.

**Composition** (khác VBSE 3-KPI ngang sau headline):
- Top masthead band violet_tint 0.50" — brand left + section name right
- **Week label as huge masthead** 72pt bold (`{{WEEK_LABEL}}` là hero)
- Regime badge violet rounded floating right of week label
- Horizontal violet rule full width
- Headline as feature title 28pt italic dark
- 3 KPI bottom row: 3 rounded card bg_card với top violet rule trái + label/value/note. KPI value 30pt bold.
- Bottom violet rule

**Placeholders:**
- `{{WEEK_LABEL}}` (hero 72pt), `{{HEADLINE_LINE}}`, `{{REGIME_BADGE}}`
- `{{VNINDEX_CLOSE}}`, `{{VNINDEX_CHANGE}}`
- `{{LIQUIDITY_AVG}}`, `{{LIQUIDITY_NOTE}}`
- `{{FOREIGN_NET}}`, `{{FOREIGN_NOTE}}`
- `{{PUBLISH_DATE}}`

#### LAYOUT 04 — `COVER_D_MACRO_SECTOR`
**Slide #:** 4. Cover chủ đề vĩ mô / phân tích ngành — **editorial pull-quote stack style**.

**Composition** (khác VBSE 3-takeaway numbered linear):
- Theme tag violet rounded top
- Report title 40pt bold (hero)
- Subtitle italic 14pt
- Partial violet horizontal rule
- **3 takeaways as staggered pull quotes** — không phải numbered list. Mỗi takeaway có violet quote mark `"` 48pt + text italic 15pt. Indent staggered: 0.6"/1.0"/1.4" (zig-zag editorial flow).
- Footer + bottom violet rule

**Placeholders:**
- `{{THEME_TAG}}`, `{{REPORT_TITLE}}` (hero 40pt), `{{REPORT_SUBTITLE}}`
- `{{TAKEAWAY_1}}` ... `{{TAKEAWAY_3}}` (mỗi takeaway ≤ 100 ký tự, italic 15pt)
- `{{PUBLISH_DATE}}`

#### LAYOUT 05 — `COVER_E_GENERIC`
**Slide #:** 5. Cover tổng quát — **asymmetric bottom-left composition**.

**Composition** (khác VBSE center-aligned classic):
- Tag violet rounded top-left
- Decorative chevron violet_tint top-right corner (oversized, partially off-canvas)
- Vertical violet rule signature 0.10"×1.50"
- **Title bottom-left aligned** 40pt bold (asymmetric, không center)
- Subtitle italic 14pt
- Footer + bottom violet rule

**Placeholders:**
- `{{REPORT_KIND_TAG}}`, `{{REPORT_TITLE}}` (hero 40pt), `{{REPORT_SUBTITLE}}`
- `{{PUBLISH_DATE}}`

---

### Section divider (1 layout)

#### LAYOUT 06 — `SECTION_DIVIDER`
**Slide #:** 6. **Magazine chapter side-band style**.

**Composition** (khác VBSE full navy + tam giác):
- Left vertical violet band 1.50" wide full height (signature chapter band)
- Section tag white text trên band (top)
- Decorative chevron white trên band (bottom)
- Right side: section title 48pt bold dark (hero) + partial violet rule + section description italic 16pt mid

**Placeholders:**
- `{{SECTION_TAG}}` (text trên band)
- `{{SECTION_TITLE}}` (hero 48pt)
- `{{SECTION_DESC}}` (italic 16pt)

---

### Content layout (12)

**Triết lý redesign**: cấu trúc layout TEMPLATE_FINEXT giữ **cùng LAYOUT_ID + cùng placeholder schema** với TEMPLATE_VBSE để runtime mapping work, NHƯNG **spatial arrangement khác hẳn** (đảo bố cục, đổi orientation, đổi hierarchy) để 2 báo cáo cùng nội dung render ra 2 brand không nhìn giống nhau. Specific differentiation per layout: xem **bảng spatial differentiation** ở phụ lục cuối file.

**Status redesign**: LAYOUT 01-27 đã redesign xong theo philosophy editorial fintech (Phase 1 + Phase 2 hoàn thành).

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
**Slide #:** 27. **Side band magazine style** (khác VBSE full navy + tam giác).

**Composition**:
- Left vertical violet band 1.0" wide full height
- 2 chevrons trắng decorative trên band
- Header "TUYÊN BỐ MIỄN TRỪ TRÁCH NHIỆM" 22pt bold violet_dark
- Partial violet rule
- **3 paragraphs disclaimer body** — mỗi đoạn là 1 text box riêng (paragraph break thật, không dùng `\n\n` literal)
- Contact card rounded bg_card với left violet rule: Website + Hotline trái, Finext brand string + Publish date phải
- Bottom violet rule

**Toàn bộ nội dung fix cứng**, chỉ 1 placeholder: `{{PUBLISH_DATE}}`

---

## Decoupling — TEMPLATE_FINEXT độc lập về authoring

TEMPLATE_FINEXT là layout catalog đứng một mình về **authoring dependency**: nội dung layout (placeholder schema, design tokens, render rule) KHÔNG được derive từ spec của FORMAT.md hay WORKFLOW.md. Đây là rule architecture (xem `system_prompt.md` mục 2 — minimal cross-reference cho clarity runtime, không backward authoring dependency).

TEMPLATE_FINEXT runtime chỉ consume 2 nguồn data:
- (a) file pack của chính nó (catalog `.md` + binary `.pptx`)
- (b) MD final đã đông kết do WORKFLOW Stage 5 produce

Reference đến WORKFLOW Stage 5/7 trong file này là pointer runtime (khi nào activate, đọc input từ đâu), không phải derive content.

Mỗi layout mô tả PURPOSE semantic của chính nó (cover khuyến nghị mã / scenario comparison 3-cell / heatmap 24 ngành / etc.) ở section "Danh sách 27 layout" trên. Agent runtime: scan MD final → match section heading + chart annotation với layout có sẵn theo semantic → clone layout → fill placeholder. Cùng 1 layout có thể fit nhiều loại MD content khác nhau, runtime tự match.

Cấu trúc layout TEMPLATE_FINEXT giữ cùng LAYOUT_ID + cùng placeholder schema với TEMPLATE_VBSE (để runtime brand routing work), nhưng spatial arrangement khác hẳn theo design philosophy editorial fintech — runtime chọn T pack nào dựa trên brand audience user yêu cầu, không phải dựa trên loại báo cáo.

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

---

## Phụ lục — Bảng spatial differentiation summary

LAYOUT 07-26 đã redesign theo bảng dưới. Mỗi row mô tả "VBSE vs Finext present cùng nội dung" — đã apply trong template hiện tại. Spec ngắn gọn (composition cụ thể đã thực thi trong `TEMPLATE_FINEXT.pptx`).

| LAYOUT | VBSE trình bày | Finext trình bày |
|---|---|---|
| 07 BULLET_LIST_SUMMARY | 5 bullet dọc với label + body | Magazine sidebar list — index "01"-"05" violet 22pt + label bold + body italic, separator border light giữa items |
| 08 STAT_CALLOUT_GRID_4CELL | 4 stat 2×2 + 4 luận điểm 2×2 | 4 stat horizontal banner row top + 4 luận điểm zig-zag indent (alternate left/right) với circle number violet |
| 09 BIG_STAT_SUBPOINTS | Big stat panel TRÁI + 3 subpoint phải | Big stat hero TOP-CENTER full-width violet_tint band (KPI 64pt) + 3 subpoint horizontal row giữa + KPI table compact bottom |
| 10 TWO_COLUMN_INFO_BUSINESS | 2 cột bằng nhau | Asymmetric 1:2 — info bảng compact LEFT (3.8") + 5 segment editorial card stack RIGHT (8.13") |
| 11 COMPARISON_3CELL_SCENARIOS | 3 cell horizontal đều | Center cell BIG (BASE 6.0" violet_tint, big_pct 44pt) + 2 cell BULL/BEAR sidebar nhỏ (2.85") |
| 12 STAT_TABLE_COMBO_SESSION | 4 stat + bảng 8 mức dọc | 4 stat horizontal top + 8-level **ladder visualization** (vertical spine + price markers, 5th level "Giá hiện tại" highlight bold violet) |
| 13 DATA_TABLE_FULL | Bảng 8×8 truyền thống | Magazine table: header band violet rounded + zebra rows bg_card + cột 1 (TICKER) violet bold + table_note italic bottom |
| 14 TIMELINE_NEWS | Timeline trục DỌC | Timeline trục **NGANG** — 4 nodes circle violet trên violet rule + alternating above/below cho date+tag và title+body (zig-zag editorial) |
| 15 RISK_GRID_STEELMAN | Grid 6 cell 3×2 | 6 hàng dọc — concern card trái + violet right-arrow giữa + rebut card phải. CONCERN_5/6 highlight bg_pink + arrow đỏ |
| 16 VARIANT_PERCEPTION | Insight box trên + 3 sub vertical | Floating violet insight bubble top-right với big quote mark 80pt + 3 cards horizontal bottom (top color band per card) |
| 17 BUY_STRUCTURE_FLEX | Bảng layer DỌC | 5 horizontal **stepped bars** (mỗi layer indent +0.20") với violet header band 1.6" + 3 column price/allocation/trigger + 3 risk band cards horizontal bottom |
| 18 TAKE_PROFIT_TARGETS | 4 milestone DỌC + R/R bottom | R/R **panel side TRÁI** (violet_tint, RR_RATIO 44pt) + 4 milestone **horizontal staircase** (height tăng dần từ M1 đến M4) phải |
| 19 THESIS_B_PANEL_RIGHT | Alt cho LAYOUT 09 | KPI panel violet PHẢI 1/3 (KPI_VALUE 64pt white) + 4 subpoint zig-zag indent trái 2/3 |
| 20 HEATMAP_INDUSTRY | 6×4 grid rounded card | 6×4 grid + **legend strip top** với 5 magnitude buckets rounded. Cell có name top + pct big + note small italic |
| 21 PEER_COMPARE_TABLE | Bảng peer 8 hàng × 8 cột | **P1 hero card** full-width violet_tint (TICKER 36pt + 6 metrics inline) + **7 peer compact cards** 4×2 grid (mini metric grid 2×3) + verdict band violet bottom |
| 22 MINI_CHARTS_2X2 | 4 chart 2×2 grid đều | **Asymmetric**: CHART_1 prominent 8.0"×5.45" trái + CHART_2/3/4 stacked vertical phải 3.98" |
| 23 CALENDAR_WEEK | 7 ngày bảng NGANG | 7 ngày **bảng DỌC** với weekday rows (T2-T6) cao 0.85" + weekend rows (T7-CN) collapsed nhỏ hơn. Day band trái violet (weekday) hoặc text_light (weekend) |
| 24 LINE_CHART_FULL | Chart 2/3 trái + 3 commentary phải | Chart **full-width top** (12.13"×3.30") + chart_source line + **3 commentary cards horizontal bottom** với circle number violet |
| 25 SCATTER_PEER | Scatter trái + bảng peer phải | Scatter **dominant trái 70%** (8.5") + 6 peer cards **stacked phải 30%** (PEER_1 highlight violet_tint) + verdict band bottom |
| 26 DONUT_COMPOSITION | 2 donut side-by-side đơn giản | 2 donut side-by-side với **violet underline title** trên mỗi donut + 5-row legend table dưới mỗi donut (zebra rows + circle dot violet marker) |

