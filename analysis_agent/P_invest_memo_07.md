# P_invest_memo_07 — Tier 5C: Investment Memo Template

Giai đoạn 5C của quy trình. Với mỗi mã đã qua tier 5A (forensic) + tier 5B (valuation), viết memo theo structure 7 phần chuẩn. Memo là **stress-test cuối** trước quyết định vào position — 2 gate mandatory (variant perception + bear case steelmanned) có thể reject mã dù đã qua tất cả tier trước.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Sáu nguyên tắc Agent bất biến (đặc biệt Nguyên tắc 1, 2, 4), `P_invest_memo_04` (tier 3 conviction), `P_invest_memo_05` (tier 5A findings), `P_invest_memo_06` (tier 5B target giá).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** viết memo đầu tư per-stock để:
- Cross-check toàn bộ thesis qua 7 phần
- Force Agent + user explicit về variant perception (differentiation) — điều thị trường đang sai mà user đang đúng
- Steelman bear case (tìm arguments phản đối mạnh nhất) trước khi finalize buy
- Định rõ exit triggers measurable trước khi mở position

**Memo là gate cuối cùng.** Sau memo, nếu confirmed → chuyển sang tier 6 (portfolio construction + position sizing). Nếu reject ở gate variant perception hoặc bear case → loại mã khỏi shortlist final dù đã đầu tư nhiều resource.

**Input:**
- File tier 3 output (`tier3_YYYYMMDD_confirmed.md`) — conviction tier + điểm 6 tiêu chí
- File tier 5A output (`tier5A_<ticker>_YYYYMMDD_confirmed.md`) — forensic findings, red flag đã clear
- File tier 5B output (`tier5B_<ticker>_YYYYMMDD_confirmed.md`) — target Base/Bull/Bear + giả định
- Web search bổ sung: consensus sell-side (VCB Securities, SSI, HSC, VND), bài phân tích công khai mới nhất, ý kiến chuyên gia ngành

**Output chính:**
- Memo 2-5 trang/mã tuỳ tier conviction
- **Decision** cuối cùng: Buy / Pass / Watch
- Entry/Exit trigger measurable cho tier 6-7
- Report Checkpoint 5C per-stock

**Scope theo tier conviction:**

| Tier | Memo độ dài | Phần bắt buộc |
|---|---|---|
| High (15-18đ) | 4-5 trang, 7 phần đầy đủ | Tất cả |
| Medium (11-14đ) | 2-3 trang, 5 phần chính | 1, 2, 4, 5, 7 (skip 3, 6 simplified) |
| Low (8-10đ) | 1-2 trang, 3 phần | 1, 4, 7 (simplified thesis + exit rule) |

**Thời gian session:** 60-90 phút/mã Agent work (viết memo + stress-test). User review checkpoint 30-45 phút/mã (challenge variant perception + bear case).

---

## 2. Triết lý

**Memo không phải báo cáo đồng thuận.** Mục đích không phải thuyết phục người đọc mua — mục đích là force Agent + user phải explicit thesis, assumptions, và rủi ro trước khi vào position. Nếu memo viết dễ quá (không có gì tricky để stress-test) → có thể thesis thiếu chiều sâu, không phải mã tốt.

**Variant perception là core.** Một investment thesis tốt không chỉ là "mã này tốt" (consensus có thể đã priced-in) — thesis tốt là "mã này tốt và thị trường đang underestimate/chưa nhận ra điều X". Không có variant perception = buy vào consensus, khó có alpha.

**Bear case steelman là bắt buộc.** Agent phải viết bear case như một short-seller thực sự — lấy arguments phản đối mạnh nhất, không soft-pedal (không làm nhẹ, không viết yếu đi để dễ phản biện). Nếu bear case > bull case sau rebut → thesis chưa đủ mạnh, reject.

**Exit triggers phải measurable.** "Bán khi có tin xấu" không phải exit trigger — đó là vague. "Bán khi Revenue Growth Q4 < 5% YoY" là exit trigger measurable. Without measurable triggers, user sẽ hold position qua deterioration vì không có rule.

**Nguyên tắc 2 của `P_invest_memo_00` — Exit triggers before position.** Memo không được kết luận "Buy" nếu chưa có exit triggers cụ thể.

---

## 3. Structure 7 phần

### Phần 1 — Recommendation + Thesis

Tóm tắt quyết định, ngắn gọn, dễ đọc. Đây là phần đầu tiên user đọc.

**Cấu trúc:**
- **Recommendation:** Buy / Pass / Watch (sau khi đi qua gate)
- **Size target:** % portfolio (từ tier 3 conviction tier × tier 6 regime adjustment)
- **Bucket entry:** 1 (vào ngay) / 2 (pullback) / 3 (watchlist)
- **Target giá:** Base / Bull / Bear (từ tier 5B)
- **Entry trigger:** price level để vào, có margin of safety (biên an toàn — khoảng chênh giữa giá mua và giá trị nội tại ước tính, đệm cho sai số giả định)
- **Horizon:** 1-6 tháng với thesis realization

**Thesis — 3-5 bullets ngắn:**
- Mỗi bullet 1 câu + 1-2 số liệu key
- Ưu tiên thesis mang tính differentiated (variant perception), không generic

Ví dụ thesis tốt:
> - Chuyển đổi segment BHX từ loss-making sang profitable trong 2026 sớm hơn consensus 1 năm, do cost per store đã giảm 12% QoQ 3 quý liên tiếp
> - Catalyst room NN upgrade 30% → 49% đã công bố chính thức, hiệu lực tháng 6/2026 — flow NN dự kiến tăng 40% trong 6 tháng
> - Định giá sector retail đã điều chỉnh -18% YTD trong khi tăng trưởng ngành vẫn dương → mismatch

Ví dụ thesis yếu (tránh):
> - Công ty đầu ngành, thương hiệu mạnh
> - Tăng trưởng tốt, biên lợi nhuận ổn định
> - Mã blue-chip đáng sở hữu dài hạn

Thesis yếu = consensus generic, không có variant perception.

### Phần 2 — Variant Perception (GATE MANDATORY)

**Đây là gate đầu tiên.** Nếu không xác định được variant perception cụ thể → reject memo, loại mã khỏi shortlist.

**Structure 4 câu hỏi:**

**2.1. Thị trường đang nghĩ gì?** (Consensus view)
- Sell-side consensus target (trung bình analyst)
- Media/retail sentiment (từ news DB + web search)
- Định giá hiện tại implies điều gì (dùng implied multiples back-solve)

Ví dụ: "Consensus sell-side target 75k (upside 10%), VCB/SSI đều Neutral rating với outlook tăng trưởng 8-10% 2026. Sentiment retail cautious do concern về cạnh tranh online."

**2.2. Tôi đang nghĩ gì khác?** (Differentiated view)
- Cụ thể điểm tôi thấy khác consensus
- Định giá của tôi vs consensus chênh bao nhiêu
- Đây là differentiation về growth / margin / multiple / thời điểm / catalyst?

Ví dụ: "Tôi tin Revenue CAGR 12-15% (cao hơn consensus 8-10%) do BHX break-even năm 2026 sớm 1 năm. Target base 85k (cao hơn consensus 75k 13%)."

**2.3. Tại sao tôi đúng?** (Evidence)
- Data điểm cụ thể support (từ tier 5A, 5B, DB)
- Nguồn information edge (cụ thể: tôi đọc được gì trong BCTC mà analyst report không mention?)

Ví dụ: "BCTC Q4/2025 soát xét cho thấy cost per BHX store giảm 12% QoQ, trend này chưa được analyst quantify. Nếu duy trì 2 quý nữa, break-even Q2-Q3/2026."

**2.4. Khi nào thị trường nhận ra?** (Catalyst/timing)
- Sự kiện nào khiến thị trường update view
- Horizon cụ thể (tuần/tháng)

Ví dụ: "BCTC Q1/2026 (công bố cuối T4/2026) dự kiến show BHX contribution positive → re-rating đợt 1 (thị trường định giá lại mã lên mức cao hơn do thông tin mới làm thay đổi thesis). BCTC Q2/2026 show break-even — re-rating đợt 2, target 85k realize trong 3-6 tháng."

**Gate assessment — flex + cảnh báo, không auto-reject:**

| Dấu hiệu | Action |
|---|---|
| Không có differentiation (variant view ≈ consensus) | **Flag cảnh báo** "buy vào consensus, thiếu alpha" + **downgrade conviction 1 bậc** (High → Medium hoặc Medium → Low) + user quyết định proceed hay không |
| Differentiation "tôi nghĩ mạnh hơn" nhưng không evidence cụ thể | **Flag cảnh báo** "wishful thinking" + **downgrade conviction 1 bậc** + user xem lại có data ẩn không Agent chưa query |
| Differentiation về giá (undervalued) nhưng không có catalyst để re-rate | **Flag "value trap risk"** + **downgrade sang Watch list** + user quyết định có vào sớm hay chờ catalyst |
| Differentiation rõ + evidence + catalyst + timing cụ thể | **Pass**, proceed memo |

Nguyên tắc: Agent không tự reject — đưa thông tin đầy đủ, highlight rủi ro, user quyết định cuối. Nếu user chọn proceed với cảnh báo → ghi audit log cụ thể.

### Phần 3 — Business Overview (skip cho Medium/Low)

Cho High conviction. Mô tả ngắn gọn:

- **Business model:** segment nào tạo revenue, tỷ trọng, trend
- **Competitive position:** thị phần, barrier to entry, moat (hào kinh tế — rào cản cạnh tranh bền vững như thương hiệu, network effect, cost advantage, switching cost, giúp doanh nghiệp duy trì biên lợi nhuận cao lâu dài)
- **Value chain position:** upstream/downstream, pricing power
- **Key customers/suppliers:** concentration risk
- **Regulatory environment:** gần đây có thay đổi gì relevant
- **Industry-level lens (tuỳ chọn):** 1-2 đoạn ngắn theo khung DD/MP/SI/PM/ESG từ `K_sector_framework` — pull 3-5 câu most material cho ngành/mã đang phân tích (vd Banking: NIM trend + CASA ratio + digital adoption; Real Estate: land bank + pre-sales velocity + financing access). Mục tiêu: đặt firm vào bối cảnh ngành chứ không lặp lại số liệu tài chính. Áp K hygiene — không lộ tên dimension "DD/MP/SI/PM/ESG" ra output, viết bằng ngôn ngữ tự nhiên.

Nguồn:
- stock_info overview + business_area
- industry_info.value_chain
- BCTN từ tier 5A
- `K_sector_framework` mục 5 per-sector quick-ref (nếu ngành có CFA cover) hoặc mục 3 universal framework (ngành ngoài cover)
- Web search bổ sung (báo cáo ngành, annual report)

Độ dài: 0.5-1 trang (1 trang nếu có industry-level lens).

### Phần 4 — Financial Analysis + Valuation

Tóm tắt 3-5 năm BCTC + target giá từ tier 5B.

**Historical (bảng 3-5 năm):**

| Năm | Revenue (tỷ) | Growth | Gross Margin | EBIT Margin | ROE | Net Debt/EBITDA |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... |

**Định giá hiện tại:**
- P/E, P/B, EV/EBITDA vs median ngành (từ industry_finstats)
- So với historical 5Y của mình
- So với peer quốc tế (web search)

**Target giá từ tier 5B:**
- Base: X (upside Y%)
- Bull: X (upside Y%)
- Bear: X (downside Y%)

**Entry / Exit levels:**
- Entry: price ≤ Base × 0.85 (margin of safety 15%)
- Take profit 50%: price đạt Base
- Take profit 80%: price đạt Bull
- Stop loss: price < Bear × 0.9

**Method mismatch note** (nếu có):
- Nếu tier 5B chênh 2 method > 15%, ghi rõ range target
- Ví dụ: "DCF cho 72k, Peer Multiples 85k — range target 72-85k, dùng median 78k để đánh giá entry/exit"

Độ dài: 1 trang.

### Phần 5 — Catalysts

Danh sách 2-4 catalyst cụ thể với 3 đặc tính (từ `P_invest_memo_00` Phần 2 Từ điển thuật ngữ mục Catalyst):
- **Timing:** ngày/tuần/tháng cụ thể
- **Magnitude:** ước lượng impact doanh thu/LNST/định giá
- **Verifiability:** làm sao biết sự kiện đã xảy ra + thành công hay thất bại

**Structure mỗi catalyst:**

```
Catalyst 1: [Tên]
- Mô tả: [1-2 câu mô tả sự kiện]
- Timing: [ngày cụ thể hoặc range tháng]
- Magnitude: [ước lượng impact %, dẫn số liệu cụ thể]
- Verifiability: [nguồn để verify — BCTC công bố, thông cáo HĐQT, UBCK...]
- Probability realize: [cao/trung bình/thấp với lý do]
- If realize: [thesis update thế nào]
- If fail: [thesis collapse không? có fallback không?]
```

**Ví dụ:**

```
Catalyst 1: BCTC Q1/2026 công bố segment BHX break-even
- Mô tả: BHX (Bách Hóa Xanh) chuyển từ loss sang profit operation trong Q1/2026
- Timing: Cuối tháng 4/2026 (BCTC quý công bố trong 30 ngày sau kết thúc quý)
- Magnitude: BHX chiếm 15% revenue; break-even → contribution margin +3-5%, EPS +20-25%
- Verifiability: BCTC quý có segment reporting; verify qua management commentary
- Probability: Trung bình-cao (cost per store đã giảm 3Q liên tiếp, trend đủ để extrapolate)
- If realize: target re-rate từ 75k lên 85k (+13%) trong 1-2 tháng
- If fail: thesis chính collapse, target về 60-65k (downside -15%)
```

**Lưu ý:**
- Catalyst phải cụ thể, không phải "KQKD tốt" chung chung
- Nếu không có catalyst cụ thể trong 1-6 tháng → memo có risk vì horizon không match. Cân nhắc downgrade conviction hoặc reject
- Catalyst tiêu cực (overhang — áp lực treo trên giá, sự kiện tiêu cực tiềm tàng chưa resolved mà thị trường đang lo ngại) chưa resolve cũng phải list — ví dụ: "Sàn NYSE delisting nếu chỉ số xuống dưới $1 trong Q2"

Độ dài: 0.5-1 trang.

### Phần 6 — Bear Case Steelmanned (GATE MANDATORY)

**Đây là gate thứ hai.** Viết bear case như short-seller thực sự, không soft-pedal.

**Structure 3 tầng:**

**6.1. Thesis bear case (3-5 arguments mạnh nhất)**

Mỗi argument cần:
- **Dấu hiệu:** data concrete support argument
- **Nguồn:** DB / PDF tier 5A / web search
- **Magnitude impact:** giả sử argument đúng, target giảm bao nhiêu

Nguồn arguments:
- Tier 5A red flag vàng (từ forensic)
- Tier 5B sensitivity analysis (biến nhạy nhất)
- Web search: short-seller report, analyst downgrade, negative news
- DB: catalyst tiêu cực, ngành trend xấu, pitfall từ `K_agent_db_04`

**Ví dụ bear case:**

```
Bear Argument 1: BHX break-even delay > thesis giả định
- Dấu hiệu: cost per store đã giảm nhưng revenue per store đang stagnant. Nếu revenue không tăng, cost giảm không đủ để break-even đúng Q1/2026
- Nguồn: Tier 5A BCTC Q4/2025 thuyết minh — revenue per store YoY -2%
- Impact: delay break-even 2-3 quý → target Base giảm từ 85k về 70k (-18%)

Bear Argument 2: Cạnh tranh ICT online ngày càng siết
- Dấu hiệu: Shopee Electronics gross margin tăng từ 5% lên 8%, TGDD mất thị phần 2% trong 2025
- Nguồn: Web search "TGDD market share 2025", FiinGroup quarterly retail report
- Impact: gross margin giảm 1-2% → EBIT margin giảm → target -10-15%

Bear Argument 3: Tier 5A noted DSO tăng trend
- Dấu hiệu: DSO 45 → 52 trong 2 năm
- Nguồn: Tier 5A forensic
- Impact: công nợ xấu tăng → provision → EBIT giảm 0.5-1%

Bear Argument 4: SmallCap retail trading pattern — liquidity thấp
- Dấu hiệu: 6-10 tỷ/phiên trading value sát ngưỡng D
- Impact: không phải bear giả năng lực DN, mà là bear về exit liquidity
```

**6.2. Probability-weighted bear target**

Tính: probability × impact của từng bear argument → expected target từ bear scenario.

Nếu bear arguments cumulative bring target xuống dưới current price → rủi ro lớn, reject hoặc watch list.

**6.3. Rebuttal 1-by-1**

Mỗi argument, đưa counter:
- Data hoặc logic phản biện
- Tại sao argument bear không decisive

**Ví dụ:**

```
Bear 1 rebuttal: Revenue per BHX store actually đã tăng 3% QoQ trong Q4/2025 sau downtrend, inflection point có thể đã qua
Bear 2 rebuttal: TGDD vẫn là leader (30% share); Shopee dùng ICT làm traffic driver, không có profit motive dài hạn
Bear 3 rebuttal: DSO tăng là do mở rộng B2B segment, không phải công nợ xấu — tier 5A đã verify 0 bad debt provision spike
Bear 4 rebuttal: thanh khoản đủ cho size position 3-5% portfolio; với size lớn hơn thì giảm size, không loại mã
```

**Gate assessment — flex + cảnh báo, không auto-reject:**

| Dấu hiệu | Action |
|---|---|
| Bear arguments quá mạnh, rebuttal yếu (dưới 1 in 3 rebut thuyết phục) | **Flag cảnh báo mạnh** + **downgrade size 30-50%** so với tier conviction gốc + user quyết định |
| Probability-weighted bear target < current price | **Flag cảnh báo nghiêm trọng** "downside > upside" + **downgrade size 50-70%** (coin-flip bet, size nhỏ) + user quyết định. Nếu user đồng ý vào → ghi audit log rõ |
| Bear có 1-2 argument strong nhưng rebuttal solid | **Pass**, note trong final recommendation để user biết |
| Bear chỉ có arguments generic (ngành khó, macro risk) | **Pass**, nhưng flag rằng bear case chưa đủ mạnh — có thể Agent đã soft-pedal, yêu cầu Agent stress-test lại |

Nguyên tắc: Agent đưa ra đánh giá rõ ràng về cường độ bear case và impact lên size, nhưng không tự reject. User nhìn full picture và quyết định. Nếu proceed với rủi ro cao → size nhỏ lại đáng kể và có audit log.

### Phần 7 — Monitoring + Exit Triggers

**Nguyên tắc 2 của `P_invest_memo_00`.** Memo không được kết luận "Buy" nếu chưa có exit triggers measurable.

**Structure 3 loại trigger:**

**7.1. Hard triggers — bán ngay khi chạm (không thảo luận thêm):**

Mỗi trigger:
- Chỉ số measurable
- Ngưỡng cụ thể
- Source để verify (DB query, BCTC công bố, price level)

**Ví dụ:**

```
Hard trigger 1: Price < Bear × 0.9 = 43k
- Source: stock_snapshot.price.close hàng ngày
- Action: bán toàn bộ position

Hard trigger 2: BCTC Q2/2026 BHX operating loss tăng vs Q1
- Source: BCTC quý công bố cuối T7
- Action: bán 70% position (thesis chính đã fail)

Hard trigger 3: Lãnh đạo CEO/CFO từ chức bất thường
- Source: news DB + UBCK disclosure
- Action: bán 100% position
```

**7.2. Soft triggers — cảnh báo, review thesis:**

Không bán ngay nhưng flag để review.

**Ví dụ:**

```
Soft trigger 1: market_rank_pct rơi xuống < 30 (percentile 0-100) trong 2 tuần liên tiếp
- Source: stock_snapshot.money_flow_score.market_rank_pct
- Action: review vị thế trong 1 tuần, cân nhắc giảm 30-50%

Soft trigger 2: Zone quý (technical_zone.overall.q) chuyển từ A về B
- Source: stock_snapshot.technical_zone.overall.q
- Action: review thesis, giảm size 30-50% nếu confirm downtrend

Soft trigger 3: DSO Q tăng > 10 ngày so với cùng kỳ năm trước
- Source: BCTC quý
- Action: review tier 5A forensic, có thể tái kiểm chất lượng lợi nhuận
```

**7.3. Take profit triggers — thoát khi đạt target:**

```
TP 1: Price đạt Base target 85k → bán 50% position
- Source: price hàng ngày
- Rationale: realize 50% upside thesis base

TP 2: Price đạt Bull target 105k → bán 30% position nữa (còn 20%)
- Rationale: thesis realized đầy đủ, giảm exposure

TP 3: Price vượt Bull 15% (120k+) → re-examine thesis
- Có phải thị trường đang euphoria? Có data mới justify?
- Nếu không justify → bán phần còn lại
```

**7.4. Review cycle:**

- **Hàng tuần:** price + zone + money_flow_score. Check soft trigger nào đã trigger
- **Hàng tháng:** NN/TD flow 1 tháng, review thesis vs catalyst schedule
- **Hàng quý:** BCTC quý mới công bố, verify thesis assumptions vs actual

Độ dài: 1 trang. Đây là phần quan trọng cho tier 7 (monitoring) — chuyển sang `P_invest_memo_09`.

---

## 4. Logic Gate — kiểm tra trước khi finalize

Agent kiểm tra 3 gate trước khi confirm memo. Triết lý: **flex + cảnh báo tốt hơn auto-reject**. Agent không tự ý loại mã — đưa cảnh báo rõ ràng, highlight rủi ro, user có full info để quyết định.

**Gate 1 (Variant Perception):** Memo có xác định clearly variant perception không?
- Nếu **không có** hoặc **đang nghĩ giống consensus** → **FLAG + DOWNGRADE 1 bậc** (High → Medium, Medium → Low), user quyết định proceed hay loại
- Nếu **có nhưng không có evidence** → **FLAG "wishful thinking" + DOWNGRADE 1 bậc**, user review lại có data ẩn không
- Nếu **undervalued không catalyst** → **FLAG "value trap risk" + DOWNGRADE sang Watch list**
- Nếu **có + evidence + catalyst + timing** → **PASS**

**Gate 2 (Bear Case Steelmanned):** Bear case có được viết mạnh thực sự không?
- Dấu hiệu bear case soft-pedal: chỉ 1-2 arguments, không có số liệu, rebuttal dài hơn bear argument
- Nếu bear case soft-pedal → **Agent REWRITE bear case với tư duy short-seller** (đây là quality control technical, không phải judgment — bear case phải đúng chuẩn forensic)
- Nếu **bear arguments mạnh + rebuttal solid** → **PASS**
- Nếu **bear arguments mạnh + rebuttal yếu** → **FLAG + DOWNGRADE SIZE 30-50%**, user quyết định
- Nếu **probability-weighted bear target < current price** → **FLAG NGHIÊM TRỌNG + DOWNGRADE SIZE 50-70%** (coin-flip bet), user quyết định. Nếu user vẫn vào → audit log rõ lý do

**Gate 3 (Exit Triggers Measurable):** Đây là quality control technical, **không phải judgment** — exit trigger phải measurable để dùng được, nếu không thì memo chưa hoàn thành.
- Mỗi trigger phải có: chỉ số cụ thể + ngưỡng + source verify + action
- Trigger vague ("bán khi có tin xấu", "bán khi mất niềm tin") → **REWRITE** đến khi đạt chuẩn measurable
- Không có ≥ 3 hard trigger + 2 soft trigger → **REWRITE** (memo chưa đủ risk management)

**Tổng hợp:** Agent confirm memo recommend Buy khi:
- Gate 1 + Gate 2: PASS, hoặc DOWNGRADE + user explicit approve (có audit log)
- Gate 3: PASS (đã rewrite nếu cần, technical chuẩn)

Cảnh báo + downgrade nhẹ không phải lý do reject — user có quyền proceed với size nhỏ và rủi ro explicit nếu thấy thesis vẫn valid.

---

## 5. Workflow 8 bước

**Bước 1 — Load input**

Đọc 3 file:
- `tier3_YYYYMMDD_confirmed.md`: conviction tier, 6 điểm, bucket
- `tier5A_<ticker>_YYYYMMDD_confirmed.md`: forensic findings, red flag vàng
- `tier5B_<ticker>_YYYYMMDD_confirmed.md`: target Base/Bull/Bear, giả định key

**Bước 2 — Web search consensus**

- Sell-side target từ các nhà môi giới VN lớn: SSI, VNDirect, VCBS, BSC, HSC, ACBS, FPTS, MBS
- Query: `<ticker> target giá 2026`, `<ticker> báo cáo phân tích`, `<ticker> analyst report`
- Tin tức 30 ngày gần nhất: news_history_feed + web search
- Ghi lại consensus target trung bình (từ các broker có public report), rating distribution (bao nhiêu Buy/Hold/Sell), expected growth trung bình
- Nếu không có consensus rõ (mã Small/Mid, ít broker cover): note "no formal consensus" và dùng sentiment từ news + forum (F319, diễn đàn CafeF) làm proxy

**Bước 3 — Viết Phần 1 (Recommendation + Thesis)**

Từ tier 3 conviction + tier 5B target → ghi recommendation + size + bucket.

Thesis 3-5 bullet ngắn — ưu tiên differentiated view.

**Bước 4 — Viết Phần 2 (Variant Perception) — GATE 1**

4 câu hỏi consensus/tôi khác/tôi đúng/khi nào realize.

**Check gate 1.** Nếu không có variant perception rõ → flag cảnh báo + downgrade conviction 1 bậc, đưa user full info để quyết định. User có quyền proceed hoặc loại mã (ghi audit log).

**Bước 5 — Viết Phần 3, 4, 5 (Overview, Financial, Catalysts)**

- Phần 3: stock_info + industry_info + BCTN (skip cho Medium/Low)
- Phần 4: tier 5B target + sensitivity tóm tắt
- Phần 5: 2-4 catalyst với 3 đặc tính

**Bước 6 — Viết Phần 6 (Bear Case) — GATE 2**

Viết bear case mạnh trước, rebuttal sau.

**Check gate 2.** Nếu bear arguments mạnh + rebuttal yếu → flag cảnh báo + downgrade size 30-50%. Nếu probability-weighted bear target < current price → flag nghiêm trọng + downgrade size 50-70% (coin-flip bet). User quyết định cuối, có audit log nếu proceed với cảnh báo.

**Bước 7 — Viết Phần 7 (Exit Triggers) — GATE 3**

3 loại trigger: hard + soft + take profit. Measurable với source.

**Check gate 3.** Nếu vague → rewrite.

**Bước 8 — Xuất Checkpoint 5C**

Theo template Section 6.

---

## 6. Template Checkpoint 5C (per-stock)

```
# Checkpoint 5C — Memo [Mã X] [ngày]

## 1. Summary quyết định

Decision: [Buy / Pass / Watch]
Tier conviction: [High/Medium/Low] — tổng [N]đ (từ tier 3)
Size target: [X%] portfolio
Bucket entry: [1/2/3]
Target giá: Base [X1] / Bull [X2] / Bear [X3]
Upside/downside vs price hiện tại [X0]: [+/-X%] Base / [+/-X%] Bull / [+/-X%] Bear

3 gate status:
- Variant Perception: [PASS / CAUTION — flag cảnh báo, downgrade, user quyết định]
- Bear Case Steelmanned: [PASS / CAUTION — flag cảnh báo, size downgrade]
- Exit Triggers: [PASS / REWRITE cần hoàn thiện]

## 2. Bối cảnh đầu vào
- Mã [X] type [Y]
- Tier 5A: [Xanh / Vàng + red flag chính]
- Tier 5B: method chính [DCF FCFF/RIM/P/B-ROE], cross-check [peer multiples/SOP]
- Consensus sell-side: target [avg X], [N] buy / [M] hold / [K] sell
- Ngày viết memo: [ngày]

## 3. Memo (7 phần, độ dài theo tier)

[Memo đầy đủ 7 phần — High conviction 4-5 trang, Medium 2-3 trang, Low 1-2 trang]

## 4. Key data points
- Target Base vs consensus: chênh [X%]
- Bear target vs current price: [±X%]
- Liquidity (trading value TB): [X tỷ/phiên]
- Bucket + timing horizon: [...]

## 5. Gate assessment chi tiết

### Gate 1 — Variant Perception
- Consensus: [...]
- My view: [...]
- Evidence: [...]
- Catalyst timing: [...]
- Assessment: [PASS / CAUTION + lý do cụ thể + đề xuất size adjustment]

### Gate 2 — Bear Case
- Bear arguments (3-5): [list ngắn]
- Probability-weighted bear target: [X]
- Strongest bear argument: [...]
- Rebuttal strength: [strong/medium/weak]
- Assessment: [PASS / CAUTION + lý do + size downgrade đề xuất]

### Gate 3 — Exit Triggers
- Hard triggers (≥3): [list]
- Soft triggers (≥2): [list]
- Take profit triggers: [list]
- Assessment: [PASS / REWRITE nếu còn vague]

## 6. Lựa chọn sát nút

Nếu user không agree recommendation:
- (a) Challenge variant perception: có view khác không?
- (b) Thêm bear argument user thấy strong: [cụ thể]
- (c) Adjust size (conviction thấp hơn/cao hơn Agent đánh giá)
- (d) Downgrade sang Watch list để observe thêm trước khi vào

## 7. Câu hỏi chờ user

Memo mã [X] đã qua 3 gate. Decision Buy với size [X%] portfolio, bucket [N].

Confirm để chuyển sang tier 6 (portfolio construction)?
Hoặc muốn:
- (a) Adjust size hoặc bucket
- (b) Challenge specific gate (rewrite phần đó)
- (c) Hold memo 1 tuần trước quyết định, chờ data mới
- (d) Pass — không vào mã này, quay lại tier 3 chọn thay thế

Nếu confirm Buy → Agent chuyển mã sang shortlist final tier 6.
```

**Độ dài:** 4-5 trang/mã High, 2-3 trang Medium, 1-2 trang Low.

---

## 7. Ví dụ generic — memo case

### Case 1 — Mã High Pass all gates

Mã top tier 3 (16đ, High conviction), tier 5A clear xanh, tier 5B Base 85k (current 72k, upside 18%).

**Gate 1 — Variant Perception:**
- Consensus SSI/VND target 75-78k, outlook growth 10%
- My view: Base 85k (13% cao hơn consensus) do thesis BHX break-even Q1/2026 (sớm 1 năm)
- Evidence: tier 5A BCTC Q4/2025 show cost per store -12% QoQ 3Q liên tiếp
- Timing: BCTC Q1/2026 cuối T4/2026 → re-rating đợt 1
- **PASS**

**Gate 2 — Bear Case:**
- 3 arguments: delay break-even, online competition, DSO trend
- Probability-weighted bear target: 68k (current 72k, downside 6%)
- Rebuttal solid cho arg 1 (inflection data), arg 2 (market leader), arg 3 (B2B, không bad debt)
- **PASS**

**Gate 3 — Exit Triggers:**
- Hard: price < 48k, BCTC Q2 BHX loss tăng, CEO/CFO từ chức
- Soft: rank_pct < 30, zone q về B, DSO +10 ngày
- TP: 50% at 85k, 30% at 105k, re-examine at 120k+
- **PASS**

**Decision:** Buy, size 6-7% portfolio, Bucket 1 (vào ngay 50-70%).

### Case 2 — Mã Medium Pass Gate 1 & 3 nhưng Gate 2 có cảnh báo

Mã 13đ Medium conviction, tier 5A vàng (DSO tăng + bên liên quan 25% vốn chủ).

**Gate 1 PASS:** variant perception về chuyển đổi model có evidence

**Gate 2 có cảnh báo:**
- Bear arg 1: DSO + bên liên quan 25% = red flag có thể phản ánh revenue quality giảm
- Bear arg 2: consensus downgrade gần đây do concern về corporate governance
- Probability-weighted bear target: 58k (current 65k, downside 11%)
- Rebuttal yếu — không có data strong phản biện DSO trend

**Gate 3 PASS:** exit triggers cụ thể

**Agent action:** Flag cảnh báo "Bear rebuttal yếu, quality concern chưa resolve" + downgrade size 30-50% so với Medium gốc. User options:
- (a) Proceed size nhỏ (2-3% thay vì 3-5%) với audit log
- (b) Downgrade sang Watch list, observe 2-3 tháng xem DSO có stabilize
- (c) Loại mã, chọn thay thế

**Decision khuyến nghị:** (b) Watch list 2-3 tháng. Nếu Q tới DSO stabilize + giao dịch bên liên quan clear → upgrade lại.

### Case 3 — Mã Low Không pass Gate 1

Mã 10đ Low conviction, qua all tiers trước nhưng:

**Gate 1 không pass:**
- Consensus target 35k, Base tier 5B 37k
- Chênh chỉ 5% — không phải variant perception đáng kể
- Thesis generic "ngành tăng trưởng, mã đầu ngành" — consensus đã priced-in

**Agent action:** Flag cảnh báo "thesis = consensus, thiếu alpha" + downgrade conviction (Low → Watch list với size nhỏ 0.5-1%).

**User options:**
- (a) Proceed với size minimal (0.5-1% portfolio) chỉ để có exposure, không expect alpha — ghi audit log
- (b) Loại mã khỏi shortlist, quay lại tier 3 chọn mã thay thế (khuyến nghị)
- (c) Yêu cầu Agent search thêm data chưa query để xem có differentiation ẩn không

Agent **không tự reject** — user có full info (consensus, my view, gap nhỏ), chọn cách xử lý phù hợp với strategy portfolio.

---

## 8. Failure mode

### 8.1. Thesis hoàn toàn là consensus

Agent viết thesis "mã blue-chip, thương hiệu mạnh, tăng trưởng tốt" — consensus đã priced-in hết. Không có alpha.

**Xử lý:** Gate 1 enforce. Nếu sau search consensus mà my view ≈ consensus → **flag cảnh báo + downgrade conviction 1 bậc**, không auto-reject. Force Agent tìm differentiation (tier 5A findings, tier 5B scenario khác consensus, catalyst chưa được analyst quantify). Nếu vẫn không tìm ra differentiation, user quyết định: proceed size nhỏ với audit log, hoặc loại mã.

### 8.2. Bear case soft-pedal

Agent viết bear case kiểu "ngành có cạnh tranh, macro risk", không có data. Rebuttal dài gấp đôi bear argument.

**Xử lý:** Gate 2 enforce. Bear case phải có data concrete + nguồn. Nếu soft-pedal → rewrite với tư duy short-seller. Dùng tier 5A red flag vàng làm seed bear argument.

### 8.3. Exit trigger vague

"Bán khi có tin xấu", "bán khi mất niềm tin", "bán khi thị trường xấu" — không actionable.

**Xử lý:** Gate 3 enforce. Mỗi trigger có chỉ số measurable + ngưỡng + source. Agent rewrite đến khi đạt chuẩn.

### 8.4. Catalyst không có timing cụ thể

"Tăng trưởng sẽ tiếp tục", "M&A có thể xảy ra" — không có timing.

**Xử lý:** mỗi catalyst phải có timing range (tháng/quý). Nếu không → không tính là catalyst trong memo, chỉ là hy vọng.

### 8.5. Skip phần 6 (bear case) vì "mã tốt quá"

Agent có xu hướng viết bear case ngắn cho mã confidence cao. Đây là bias — mã confidence cao càng cần bear case mạnh.

**Xử lý:** bear case có độ dài tối thiểu theo tier:
- High: 3-5 arguments
- Medium: 2-3 arguments
- Low: 2 arguments
Ngắn hơn thì Agent chưa thực sự stress-test.

### 8.6. Thesis dựa chủ yếu vào định giá (rẻ)

Thesis "mã này rẻ vs ngành, P/E dưới median" nhưng không có catalyst để re-rating.

**Xử lý:** định giá rẻ không phải thesis — là điều kiện entry. Thesis phải có catalyst để market re-price. Nếu chỉ rẻ + không catalyst → value trap risk, downgrade sang Watch list.

### 8.7. Gate 2 bear cumulative target < current nhưng vẫn Buy

Bear arguments + probability → target < current price. Agent vẫn viết Buy vì tin Base case.

**Xử lý:** nếu probability-weighted bear target < current, downside risk > upside → **flag cảnh báo nghiêm trọng** trong checkpoint. Agent **không tự reject** nhưng **downgrade size 50-70%** (coin-flip bet, size tối thiểu) và highlight rõ cho user. User quyết định cuối — nếu vẫn Buy, ghi audit log rõ lý do override cảnh báo. Agent không được che giấu hoặc làm nhẹ cảnh báo.

### 8.8. Không update memo khi có data mới

Memo viết từ tier 5B data 2 tháng trước. BCTC quý mới đã có. Agent dùng memo cũ cho tier 6 sizing → sai assumption.

**Xử lý:** memo có timestamp expiry 30 ngày. Sau 30 ngày nếu chưa enter position → rewrite memo với data mới (giá thay đổi, consensus update, BCTC quý mới có thể đã công bố). Sau khi enter, memo archive, monitoring dùng exit triggers thay vì thesis memo.

### 8.9. Variant perception dựa wishful thinking

"Tôi tin segment X sẽ break-even sớm" — không có data, chỉ cảm tính.

**Xử lý:** Gate 1 yêu cầu evidence cụ thể (số liệu + source). Nếu không có → variant perception = wishful thinking, không đủ để buy.

---

## 9. Đầu ra chuẩn để tier 6 dùng

Output tier 5C lưu file `tier5C_<ticker>_YYYYMMDD_confirmed.md` per-stock.

1. **Header:** mã, ngày, decision (Buy/Pass/Watch), size target, bucket
2. **Memo 7 phần** đầy đủ (theo tier conviction)
3. **3 gate status** + lý do
4. **Consensus vs my view comparison**
5. **Exit triggers chi tiết** — 3 hard + 2-3 soft + 3 TP, measurable với source
6. **Review cycle** — weekly/monthly/quarterly
7. **Timestamp expiry** — 30 ngày
8. **Link tier 3, 5A, 5B** reference

Tier 6 (portfolio construction) dùng file này làm input để:
- Compute portfolio size allocation với cap 5% ADV constraint
- Sequence entry (Bucket 1 đầu, 2 chờ confirm, 3 watchlist)
- Set exit triggers vào monitoring system

Mã decision Pass/Watch → file chỉ archive, ghi nhận lý do loại.
