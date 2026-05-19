# O_invest_memo_02 — Stock Memo

Spec render memo đầu tư deep-dive cho 1 mã, gộp 3 state MD (tier 5A forensic + tier 5B valuation + tier 5C memo 7 phần) thành 1 deliverable hoàn chỉnh.

Đây là **deliverable quan trọng nhất** của quy trình `P_invest_memo`. Memo này là cơ sở để ra quyết định position sizing ở tier 6 và là tài liệu archive cho cycle.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_05` (tier 5A forensic), `P_invest_memo_06` (tier 5B valuation), `P_invest_memo_07` (tier 5C memo template).

## 1. Input state & mapping

**3 state file bắt buộc đầy đủ trước khi render:**

| State file | Role | Nội dung chính |
|---|---|---|
| `tier5A_<ticker>_<YYYYMMDD>_confirmed.md` | Forensic base | Red flag đã clear/vàng, key findings từ BCTC, DSO trend, bên liên quan, off-balance items |
| `tier5B_<ticker>_<YYYYMMDD>_confirmed.md` | Valuation base | Target Base/Bull/Bear, giả định key, sensitivity table, cross-check 2 method |
| `tier5C_<ticker>_<YYYYMMDD>_confirmed.md` | Memo skeleton | Memo 7 phần: Recommendation / Variant / Business / Financial / Catalysts / Bear / Exit |

**Composition logic: tier 5C là xương sống, 5A/5B là thịt bổ sung vào các phần tương ứng.**

Mapping merge:

| Phần memo cuối | Base content | Enrichment |
|---|---|---|
| 1. Recommendation + Thesis | 5C phần 1 | — |
| 2. Variant perception | 5C phần 2 | — |
| 3. Business Overview (chỉ High/Medium) | 5C phần 3 | — |
| 4. Financial Analysis + Valuation | 5C phần 4 | Từ 5B: giả định key model + sensitivity table 2D + cross-check 2 method chênh lệch. Từ 5A: số BCTC đã clean 3-5 năm |
| 5. Catalysts | 5C phần 5 | — |
| 6. Bear Case Steelmanned | 5C phần 6 | Từ 5A: red flag vàng chưa resolve (DSO trend, bên liên quan tỷ trọng cao, off-balance items) — đưa vào bear argument nếu chưa có sẵn ở 5C |
| 7. Monitoring + Exit Triggers | 5C phần 7 | Từ 5B: level cụ thể base × 0.85 entry, bear × 0.9 stop, base/bull take profit — đảm bảo số khớp với tier 5B target |

**Nếu thiếu state file:**
- Thiếu 5C → không thể render, user phải hoàn thành CP5C trước
- Thiếu 5B → render được nhưng flag rõ "Thiếu modeling tier 5B, target giá lấy từ 5C mặc định"
- Thiếu 5A → render được nhưng flag rõ "Thiếu forensic tier 5A, bear case chỉ dựa trên 5C, không có red flag bổ sung"

Agent **không tự suy luận** thay thế content thiếu — flag rõ cho user biết.

## 2. Structure memo đầu ra

Flex theo conviction tier (đã spec ở `P_invest_memo_07` mục 1):

**High conviction (15-18 điểm) — đầy đủ 7 phần, 10-15 trang, 6-8 chart:**

```
1. Recommendation + Thesis (0.5-1 trang)
2. Variant Perception (1-1.5 trang)
3. Business Overview (0.5-1 trang)
4. Financial Analysis + Valuation (2-3 trang, 3-4 chart)
5. Catalysts (0.5-1 trang)
6. Bear Case Steelmanned (1.5-2 trang, 1-2 chart sensitivity)
7. Monitoring + Exit Triggers (1 trang)
— Metadata (0.2 trang)
```

**Medium conviction (11-14 điểm) — 5 phần chính, 6-10 trang, 3-5 chart:**

```
1. Recommendation + Thesis
2. Variant Perception
4. Financial Analysis + Valuation (rút gọn, skip sensitivity detail)
5. Catalysts
7. Monitoring + Exit Triggers
— Metadata
```

Skip phần 3 Business Overview (sếp/khách đã biết mã MidCap) và phần 6 Bear case simplified (2-3 bear argument + rebut ngắn, không steelman đầy đủ).

**Low conviction (8-10 điểm) — 3 phần, 3-5 trang, 1-2 chart:**

```
1. Recommendation + Thesis (ngắn)
4. Financial Analysis + Valuation (chỉ target + key ratio)
7. Monitoring + Exit Triggers
— Metadata
```

Mã Low conviction không đủ để deep-dive — memo chỉ cần đủ audit quyết định sizing nhỏ.

## 3. Heading convention

Title memo cấp 1 (`#`) format: `Memo đầu tư — <TICKER> (<Ngành ngắn>)`

Ví dụ: `# Memo đầu tư — VNM (Tiêu dùng thiết yếu)`

Các phần chính cấp 2 (`##`) đánh số rõ:

```
## 1. Recommendation + Thesis
## 2. Variant Perception
## 3. Business Overview
## 4. Financial Analysis + Valuation
## 5. Catalysts
## 6. Bear Case Steelmanned
## 7. Monitoring + Exit Triggers
```

Subsection cấp 3 (`###`) theo spec dưới.

## 4. Compose từng phần chi tiết

### Phần 1 — Recommendation + Thesis

Format cứng phần đầu (bảng info nhanh):

```
**Recommendation:** [Buy / Pass / Watch]  
**Conviction tier (mức độ tự tin):** [High (cao) / Medium (trung bình) / Low (thấp)] — [N] điểm  
**Size target:** [X]% portfolio  
**Bucket entry:** Bucket [1/2/3] ([vào ngay / chờ pullback xác nhận / watchlist])  
**Target giá (VND/cp):** Base [X1]k — Bull [X2]k — Bear [X3]k  
**Entry trigger:** price ≤ [X]k (margin of safety 15%)  
**Horizon (khung thời gian giữ vị thế):** [1-6 tháng]
```

Tất cả thuật ngữ chuyên môn có ngoặc dịch (theo master mục 6). Lần đầu xuất hiện trong memo.

**Thesis 3-5 bullet:**

Mỗi bullet 1 câu + 1-2 số key. Ưu tiên differentiated view, tránh consensus generic. Pattern:

```
**Thesis:**
- [Bullet 1 — 1 câu thesis cốt lõi với số liệu support]
- [Bullet 2 — catalyst rõ với timing]
- [Bullet 3 — differentiation so consensus]
- [Bullet 4-5 optional]
```

### Phần 2 — Variant Perception

4 sub-section đúng structure 5C:

```
### 2.1. Consensus view

[Consensus sell-side target, rating distribution, expected growth. Nếu không có coverage: "Không có consensus chính thức — tham khảo sentiment từ forum/tin thị trường"]

### 2.2. Differentiated view

[Tôi nghĩ gì khác consensus — cụ thể về growth/margin/multiple/timing/catalyst. So với consensus chênh bao nhiêu]

### 2.3. Evidence

[Data concrete support differentiated view — số liệu từ agent_db (nguồn: Tổng hợp) + số từ BCTC (nguồn cụ thể BCTC + trang)]

### 2.4. Catalyst + timing realize

[Sự kiện nào khiến thị trường update view, horizon cụ thể theo tuần/tháng]
```

Gate assessment ở 5C (PASS / CAUTION + downgrade) **không render vào memo cuối** — đây là internal decision. Memo chỉ hiển thị kết quả (recommendation ở phần 1 đã phản ánh gate).

### Phần 3 — Business Overview (chỉ High)

4 subsection ngắn:

```
### 3.1. Business model
[Segment nào tạo revenue, tỷ trọng, trend]

### 3.2. Competitive position
[Thị phần, barrier to entry, moat (hào kinh tế)]

### 3.3. Value chain position
[Upstream/downstream, pricing power]

### 3.4. Key customers/suppliers & regulatory
[Concentration risk + thay đổi regulatory gần]
```

Độ dài: 0.5-1 trang tổng cho 4 subsection.

### Phần 4 — Financial Analysis + Valuation (merge với 5B)

Structure:

```
### 4.1. Historical 3-5 năm

[Bảng BCTC key metrics. Số từ tier 5A đã clean]

| Năm | Revenue (tỷ) | Growth | Gross Margin | EBIT Margin | ROE | Net Debt/EBITDA |
|---|---|---|---|---|---|---|
| 2021 | ... | ... | ... | ... | ... | ... |
| 2022 | ... | ... | ... | ... | ... | ... |
| 2023 | ... | ... | ... | ... | ... | ... |
| 2024 | ... | ... | ... | ... | ... | ... |
| 2025 | ... | ... | ... | ... | ... | ... |

[Chart revenue + growth combo, lấy từ data bảng trên]

### 4.2. Định giá hiện tại

- P/E hiện tại [X] vs median ngành [Y] (nguồn: Tổng hợp)
- P/B hiện tại [X] vs median ngành [Y]
- EV/EBITDA hiện tại [X] vs median ngành [Y]
- So với historical 5Y của chính mã: [đắt/rẻ/fair]

### 4.3. Target giá từ tier 5B

[3 scenario với giả định key]

| Scenario | Probability | Revenue CAGR 5Y | EBIT margin năm 5 | WACC | Terminal g | Target |
|---|---|---|---|---|---|---|
| Base | [X]% | ... | ... | ... | ... | [X1]k |
| Bull | [Y]% | ... | ... | ... | ... | [X2]k |
| Bear | [Z]% | ... | ... | ... | ... | [X3]k |

Probability-weighted target: [X]k

Cross-check 2 method ([DCF / Peer Multiples]): chênh lệch [X]% — [Xanh/Vàng/Đỏ]

### 4.4. Sensitivity analysis (chỉ High, skip cho Medium/Low)

[Bảng sensitivity 2D cho 2 biến nhạy nhất]

| WACC \ Terminal g | 2% | 3% | 4% |
|---|---|---|---|
| 10% | ... | ... | ... |
| 12% | ... | ... | ... |
| 14% | ... | ... | ... |

### 4.5. Entry / Exit levels

- Entry: price ≤ Base × 0.85 = [X]k (margin of safety 15%)
- Take profit 50%: price đạt Base = [X]k
- Take profit 80%: price đạt Bull = [X]k
- Stop loss: price < Bear × 0.9 = [X]k
```

**Chart mandatory phần 4:**
- Chart 4.1: combo bar+line revenue/growth 5 năm
- Chart 4.3 (High only): 3 scenario bar chart so target

### Phần 5 — Catalysts

2-4 catalyst, mỗi catalyst format:

```
### Catalyst [N]: [Tên ngắn]

- **Mô tả:** [1-2 câu]
- **Timing:** [ngày/tuần/tháng cụ thể]
- **Magnitude:** [ước lượng impact % doanh thu/LNST/target giá]
- **Verifiability:** [nguồn để verify — BCTC công bố, thông cáo, UBCK (Uỷ ban Chứng khoán Nhà nước)]
- **Probability realize:** [cao/trung bình/thấp] + lý do
- **If realize:** [thesis update thế nào, target move ra sao]
- **If fail:** [thesis collapse hay có fallback]
```

Catalyst tiêu cực (overhang, áp lực treo trên giá) cũng list nếu có — không chỉ catalyst tích cực.

### Phần 6 — Bear Case Steelmanned (merge với 5A red flags)

Structure:

```
### 6.1. Bear arguments (3-5 arguments mạnh nhất)

**Bear Argument [N]: [Tên]**
- Dấu hiệu: [data concrete]
- Nguồn: [agent_db "(nguồn: Tổng hợp)" / BCTC cụ thể / tier 5A findings / web external]
- Magnitude impact: [target giảm bao nhiêu nếu đúng]

### 6.2. Probability-weighted bear target

[Tính: probability × impact từng argument → target bear scenario]

### 6.3. Rebuttal 1-by-1

**Bear [N] rebuttal:** [phản biện ngắn, data hoặc logic]
```

**Enrichment từ tier 5A:** red flag vàng (chưa clear hoàn toàn) trong tier 5A phải xuất hiện ít nhất 1 lần trong bear arguments. Nếu 5A có red flag vàng mà 5C bear case không mention → agent bổ sung thành 1 bear argument.

**Chart phần 6 (chỉ High):** sensitivity chart hoặc tornado chart từ tier 5B cho biến nhạy nhất.

### Phần 7 — Monitoring + Exit Triggers

Format 3 block rõ (hard / soft / take profit):

```
### 7.1. Hard triggers (bán ngay khi chạm)

| # | Trigger | Ngưỡng | Source verify | Action |
|---|---|---|---|---|
| 1 | Price drop | < [X]k (Bear × 0.9) | Price hàng ngày | Bán 100% |
| 2 | KPI fail BCTC Q | [Revenue decline / margin drop / specific] | BCTC quý công bố | Bán 70% |
| 3 | Governance event | CEO/CFO từ chức không lý do | News finext.vn + UBCK disclosure | Bán 100% |

### 7.2. Soft triggers (cảnh báo, review thesis)

| # | Trigger | Ngưỡng | Source | Action đề xuất |
|---|---|---|---|---|
| 1 | Flow yếu | ... | (nguồn: Tổng hợp) | Review trong 1 tuần, cân nhắc giảm 30-50% |
| 2 | Zone quý chuyển | A → B | (nguồn: Tổng hợp) | Giảm size 30-50% nếu confirm downtrend |
| 3 | BCTC deterioration | DSO +10 ngày vs cùng kỳ | BCTC quý | Re-check forensic |

### 7.3. Take profit triggers

| # | Trigger | Ngưỡng | Action |
|---|---|---|---|
| 1 | Đạt Base target | price = [X]k | Bán 50% position |
| 2 | Đạt Bull target | price = [X]k | Bán thêm 30% (còn 20%) |
| 3 | Vượt Bull +15% | price > [X]k | Re-examine thesis, nếu không justify → bán phần còn lại |

### 7.4. Review cycle

- **Hàng tuần:** price + flow + zone. Check soft trigger.
- **Hàng tháng:** review thesis vs catalyst schedule, NN/TD flow 1 tháng.
- **Hàng quý:** BCTC quý mới, verify thesis assumptions vs actual.
```

### Metadata cuối file

Format cố định:

```
---

**Metadata**

- **Ngày viết memo:** [DD/MM/YYYY]
- **Mã:** [TICKER]
- **Type:** [SXKD / NGANHANG / CHUNGKHOAN / BAOHIEM]
- **Memo version:** 1.0 (hoặc 1.1, 2.0 nếu refresh)
- **Expiry:** 30 ngày kể từ ngày viết, refresh nếu quá hạn hoặc có BCTC mới
- **State file nguồn:**
  - `tier5A_<ticker>_<YYYYMMDD>_confirmed.md`
  - `tier5B_<ticker>_<YYYYMMDD>_confirmed.md`
  - `tier5C_<ticker>_<YYYYMMDD>_confirmed.md`
```

Metadata 0.2 trang, audit trail đủ cho cycle review sau.

## 5. Chart annotation — danh sách chart tiêu chuẩn cho stock memo

Theo rule chart annotation ở master mục 7. Memo High conviction nên có 6-8 chart. Dưới đây là bộ chart mặc định theo phần:

**Phần 4 Financial:**

Chart 1 — Revenue & Growth 5 năm (combo):

````
```chart
type: combo
title: [TICKER] Revenue & Growth 2021-2025
x_axis: [2021, 2022, 2023, 2024, 2025]
y_axis:
  - name: Revenue (bar)
    type: bar
    data: [...]
    y_label: Tỷ VND
  - name: YoY Growth (line)
    type: line
    data: [...]
    y_label: '%'
    axis: secondary
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

Chart 2 — Margin trend (line):

````
```chart
type: line
title: [TICKER] Margin Trend 2021-2025
x_axis: [2021, 2022, 2023, 2024, 2025]
y_axis:
  - name: Gross Margin
    data: [...]
  - name: EBIT Margin
    data: [...]
  - name: Net Margin
    data: [...]
y_label: '%'
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

Chart 3 — 3 scenario target (bar):

````
```chart
type: bar
title: [TICKER] Target Price Scenarios
x_axis: [Bear, Base, Bull]
y_axis:
  - name: Target Price
    data: [X3, X1, X2]
y_label: VND/cp (k)
source: Tier 5B modeling
render_in_md: skip
render_in_docx: true
render_in_pptx: true
note: Probability-weighted [X]k, current price [Y]k, upside Base [Z]%
```
````

Chart 4 — Peer multiples comparison (bar):

````
```chart
type: bar
title: [TICKER] vs Peer Median — P/E and P/B
x_axis: [P/E, P/B, EV/EBITDA]
y_axis:
  - name: [TICKER]
    data: [...]
  - name: Peer median ngành
    data: [...]
y_label: Ratio
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

**Phần 6 Bear case (chỉ High):**

Chart 5 — Sensitivity 2D heatmap hoặc scatter với 2 biến nhạy nhất.

**Optional cho High:**

Chart 6-8 tuỳ nội dung mã: ROE trend vs ngành, segment revenue breakdown (pie), catalyst timeline visualization.

Medium/Low tier: chỉ Chart 1, 3, và skip các chart còn lại.

## 6. Compose workflow step-by-step

Khi user yêu cầu render stock memo (ví dụ "xuất memo VNM"):

**Bước 1 — Xác định mã + format + template** (theo master workflow Bước 1-3):
- Mã user chỉ định (VNM)
- Hỏi format (MD / docx / pptx)
- Nếu docx/pptx → hỏi template

**Bước 2 — Load 3 state file:**
- Tìm trong project knowledge: `tier5A_<VNM>_*`, `tier5B_<VNM>_*`, `tier5C_<VNM>_*`
- Lấy version confirmed mới nhất (theo YYYYMMDD)
- Nếu thiếu state → flag user theo rule ở mục 1

**Bước 3 — Đọc nội dung 3 state:**
- 5C phần 1-7 là skeleton
- 5A: red flag vàng list + key findings BCTC + DSO/off-balance
- 5B: target Base/Bull/Bear + giả định + sensitivity + cross-check

**Bước 4 — Xác định conviction tier** (từ 5C phần 1 hoặc state tier 3): High/Medium/Low. Set scope structure theo mục 2.

**Bước 5 — Compose từng phần theo mapping ở mục 1 + structure mục 4:**
- Phần 1-3: copy từ 5C, apply K hygiene + dịch thuật ngữ
- Phần 4: merge 5C phần 4 + 5B detail (giả định, sensitivity, cross-check)
- Phần 5: copy từ 5C
- Phần 6: merge 5C phần 6 + 5A red flag vàng → bổ sung bear argument
- Phần 7: copy từ 5C + number cụ thể từ 5B
- Metadata: auto-generate

**Bước 6 — Chèn chart annotation YAML** tại các vị trí ở mục 5 với data từ state file (5A clean BCTC + 5B scenarios).

**Bước 7 — Self-check K hygiene + dịch thuật ngữ + citation:**
- Không còn `w_trend`, `zone: A`, `Kịch bản B`, `Pitfall F5` → nếu còn, dịch lại
- Conviction/Bucket/Phase/Regime lần đầu xuất hiện có ngoặc dịch
- Số từ agent_db có "(nguồn: Tổng hợp)"
- Số từ PDF BCTC có tên tài liệu + trang
- Tin finext có link đầy đủ
- Ticker uppercase, VND tỷ/k, % 1 decimal

**Bước 8 — Render format cuối:**
- MD → output trực tiếp file `.md`, chart annotation giữ nguyên YAML
- Docx → load skill, template, convert MD với chart thật
- Pptx → load skill, template, slide layout (xem guide render pptx dưới)

**Bước 9 — Present file cho user.**

## 7. Guide render docx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

Khi format cuối là docx:

**Layout đề xuất:**
- **Trang 1 — Cover:** Title memo + ticker + ngày + logo (từ template). Các trường còn lại của phần 1 Recommendation bảng info ở trang 2.
- **Trang 2+ — Body:** render theo 7 phần, mỗi phần bắt đầu heading mới. Không break page giữa phần, page break chỉ giữa các phần chính (1→2, 2→3, v.v.)
- **Chart:** render inline trong section tương ứng, kích thước 80% page width, caption bên dưới = title chart + "(nguồn: Tổng hợp)" hoặc source cụ thể
- **Bảng:** render Markdown table sang Word table, header row bold, zebra shading optional tuỳ template
- **Footnote citation Nhóm 3/4:** cuối từng trang (Word footnote feature)
- **Metadata:** trang cuối, font nhỏ hơn body 1-2pt

**Template cần có:**
- Heading styles (H1/H2/H3) đã set font/size/color theo brand
- Body style default
- Table style
- Footer có page number + "Memo [TICKER] v[X.Y] — [ngày]"
- Header optional có logo

## 8. Guide render pptx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

Khi format cuối là pptx (pitch deck 1 mã):

**Slide layout đề xuất cho High conviction (15-20 slide):**

| Slide | Nội dung |
|---|---|
| 1 | Cover — ticker + ngành + ngày + recommendation |
| 2 | Executive summary (dù memo MD không có, pptx pitch cần) — target + thesis 3 bullet + horizon |
| 3 | Thesis chi tiết (phần 1 của memo) |
| 4-5 | Variant perception (4 sub: consensus / my view / evidence / timing) |
| 6 | Business overview |
| 7-8 | Financial historical + chart revenue/growth + chart margin |
| 9-10 | Valuation + 3 scenario chart + sensitivity |
| 11-12 | Catalysts (2 slide, mỗi slide 1-2 catalyst) |
| 13-14 | Bear case + chart sensitivity |
| 15 | Monitoring & exit triggers |
| 16 | Summary + decision recap |
| 17-18 | Q&A hoặc appendix chart |

Pitch pptx **có executive summary slide** dù memo MD không có — đây là exception vì pitch cần hook ngay slide đầu.

Medium tier: 10-12 slide (bỏ variant perception detail, business overview, bear case simplified).
Low tier: không render pptx (mã Low không đủ warrant pitch).

**Template cần có:**
- Master slide với logo + color palette + font
- Section divider slide
- Title + content layout
- Chart placeholder layout
- Table layout
- Final slide layout

## 9. Ví dụ memo đầu ra (fragment, demo)

Đoạn ngắn để illustrate convention, không phải memo đầy đủ:

```markdown
# Memo đầu tư — VNM (Tiêu dùng thiết yếu)

## 1. Recommendation + Thesis

**Recommendation:** Buy  
**Conviction tier (mức độ tự tin):** High (cao) — 16 điểm  
**Size target:** 6% portfolio  
**Bucket entry:** Bucket 1 (vào ngay)  
**Target giá (VND/cp):** Base 85k — Bull 105k — Bear 58k  
**Entry trigger:** price ≤ 72k (margin of safety 15%)  
**Horizon (khung thời gian giữ vị thế):** 3-6 tháng

**Thesis:**
- BHX (Bách Hóa Xanh) break-even sớm 1 năm so consensus do cost per store giảm 12% QoQ 3 quý liên tiếp (nguồn: Tổng hợp)
- Catalyst room khối ngoại từ 30% lên 49% đã công bố, hiệu lực tháng 6/2026 — flow khối ngoại dự kiến tăng 40% trong 6 tháng ([Finext, 15/4/2026](https://finext.vn/news/vnm-room-nn-upgrade-49-percent))
- Định giá sector retail đã điều chỉnh -18% YTD trong khi tăng trưởng ngành vẫn dương, tạo mismatch cơ hội

## 2. Variant Perception

### 2.1. Consensus view

Consensus sell-side target trung bình 75k (upside 4%), SSI/VND đều Neutral rating với outlook tăng trưởng 8-10% 2026. Sentiment retail cautious do concern cạnh tranh online ([SSI VNM Update 03/2026](https://www.ssi.com.vn/...)).

### 2.2. Differentiated view

Tôi tin Revenue CAGR 12-15% (cao hơn consensus 8-10%) do BHX break-even năm 2026 sớm 1 năm so dự báo analyst. Target Base 85k, cao hơn consensus 13%.

...
```

## 10. Checklist self-check cuối cùng

Trước khi present file cho user, agent chạy checklist:

- [ ] 3 state file đầy đủ và confirmed. Nếu thiếu, flag đã ghi rõ ở đầu memo
- [ ] Conviction tier đã xác định, scope structure phù hợp (High 7 phần / Medium 5 phần / Low 3 phần)
- [ ] Target giá Base/Bull/Bear ở phần 1 và phần 4.3 và phần 7 khớp nhau (không có lệch số)
- [ ] Entry/TP/Stop levels ở phần 1 và phần 4.5 và phần 7 khớp nhau
- [ ] Bear case phần 6 có ít nhất 1 argument từ red flag vàng tier 5A (nếu 5A có red flag vàng)
- [ ] Chart annotation YAML: tất cả có source + render flags; data đúng format array
- [ ] Thuật ngữ Conviction/Bucket/Phase/Regime/Catalyst/Horizon/Variant perception lần đầu xuất hiện có ngoặc dịch
- [ ] Không còn ký hiệu DB raw (`w_trend`, `zone: A`, `day_score`, etc.)
- [ ] Không còn taxonomy nội bộ ("Kịch bản B", "Pitfall F5", "HIGH impact")
- [ ] Citation đúng 4 nhóm: agent_db "(nguồn: Tổng hợp)", tin finext link đầy đủ, PDF BCTC tên + trang, web external markdown link
- [ ] Metadata cuối file đầy đủ: ngày, version, 3 state file nguồn
- [ ] Format cuối đúng yêu cầu user (MD/docx/pptx). Nếu docx/pptx, đã load đúng template user chỉ định
