# P_stock_report_01 — Pre-flight + Stage 1 Data Acquisition + Type classification

File này quy định toàn bộ data acquisition checklist trước khi compose analysis (Stage 2 ở `_03`).

## 1. Pre-flight (6 câu)

Agent hỏi 6 câu trước khi bắt đầu. Có thể gom 2-3 câu trong 1 turn nếu user chưa cung cấp.

### Câu 1: Ticker

> **Mã anh muốn phân tích là gì?**

Validation:
- Ticker UPPERCASE, 3 ký tự (vd VNM, HPG) hoặc 4 ký tự (rất hiếm)
- Cross-check tồn tại trong `stock_info`. Không tồn tại → "Mã [X] không có trong hệ thống, anh kiểm tra lại giúp"
- Mã suspended / cảnh báo / kiểm soát đặc biệt → flag big warning trước khi tiếp tục

### Câu 2: Horizon

> **Anh muốn horizon nào? (1-3 tháng / 3-6 tháng / 6-12 tháng)**

Mặc định nếu user không trả lời: **3-6 tháng** (middle ground hợp lý cho institutional buy-side).

Lưu ý: horizon ảnh hưởng:
- Cách select catalyst (timing phải khớp horizon)
- Cách build bear case (rủi ro nào trong horizon)
- Exit trigger time-stop

### Câu 3: Depth mode

> **Depth mode? (Quick 1-2 trang / Standard 3-5 trang / Deep 5-10 trang)**

Mặc định: **Standard**.

Quick mode đặc biệt: dùng cho ad-hoc câu hỏi nhanh (KH hỏi 1 mã, đồng nghiệp hỏi review). Skip nhiều sub-step.

Deep mode đặc biệt: gần Tier 5C level nhưng vẫn flex hơn. Dùng khi pre-screening trước memo cycle, hoặc khi cần đào sâu để pitch lớn.

### Câu 4: Audience

> **Audience nội bộ analyst hay KH? (mặc định nội bộ)**

Audience ảnh hưởng:
- **K hygiene strict** (xem `O_stock_report_00`): KH không xem thuật ngữ raw (Bucket entry, Variant Perception, Kịch bản A/B/C/E1/E2/E3, framework chấm điểm). Phải dịch sang ngôn ngữ tự nhiên.
- **Wording mềm hơn** với KH: "có thể xem xét theo dõi" thay vì "Watch", "rủi ro đáng lưu ý" thay vì "downgrade conviction".
- **Disclaimer chuẩn KH** cuối báo cáo (xem `O_stock_report_00`).

Mặc định: **Nội bộ analyst**.

### Câu 5: Pair compare (optional)

> **Anh có muốn so sánh với mã khác không? (nếu có, cho 1-2 ticker khác)**

Nếu user cung cấp 1-2 ticker khác → activate **Pair compare mode** (xem `_03` mục Pair compare).

Pair compare yêu cầu:
- Cả 3 mã (gồm mã chính) trong cùng ngành hoặc liên quan logic (vd HPG vs HSG cùng thép, hoặc CTD vs HBC cùng contractor)
- Nếu mã ngoài ngành / không liên quan → hỏi user xác nhận lý do compare (tránh apple-to-orange)
- Pair compare chỉ available ở Standard và Deep mode (Quick mode không support)

### Câu 6: File request — BẮT BUỘC BCTC

> **Anh cần upload các file sau cho phân tích:**
>
> **🔴 BẮT BUỘC:**
> - BCTC gần nhất 2-3 kỳ (PDF) — đầy đủ Bảng cân đối / Kết quả KD / Lưu chuyển tiền + **Thuyết minh BCTC** (quan trọng nhất)
>
> **⚪ Optional (recommended cho Deep mode):**
> - BCTN (annual report) — có chiến lược + thuyết minh ESG
> - Báo cáo phân tích cũ về mã này → Stage 0 eval cross-check thesis cũ
> - Báo cáo sell-side / IR presentation gần đây

**Rule strict:**
- **Không có BCTC PDF → REFUSE chạy.** Đây là gate tuyệt đối, không flex+downgrade.
  - Wording refusal: "Em chưa nhận được file BCTC gần nhất của [X]. Phân tích chuyên sâu cần đào sâu thuyết minh BCTC, anh upload giúp em rồi mình tiếp tục nhé."
- BCTC đầy đủ phải có:
  - 3 báo cáo chính: Bảng cân đối / Kết quả KD / Lưu chuyển tiền
  - **Thuyết minh BCTC** (footnotes) — đây là điểm quan trọng nhất
  - Báo cáo của KTV (Auditor's report) — để check audit opinion
- Chỉ tóm tắt 1-2 trang ratios không đủ — REFUSE.
- BCTC quý gần nhất chưa soát xét vẫn OK, nhưng note rõ "BCTC chưa soát xét" trong báo cáo.

**Fail-soft cho optional files:**
- Không có BCTN → vẫn proceed, downgrade depth nếu Deep (Deep cần BCTN cho chiến lược + ESG note)
- Không có báo cáo cũ → skip Stage 0
- Không có sell-side → web search bù

## 2. Stage 0 — Evaluation prior analysis (optional)

**Activate khi:** user upload báo cáo phân tích cũ về mã này (của firm khác hoặc của chính user trước đây).

**Output:** 1-block ngắn 4-6 dòng:

```
Prior analysis date: [DD/MM/YYYY]
Prior recommendation: [Long/Watch/Avoid + conviction]
Prior thesis core: [1 dòng]
Cross-check với data hiện tại:
  ✓ [Element pass — vẫn đúng]
  ⚠ [Element changed — cần update]
  ✗ [Element failed — thesis cũ disconfirm]
Carry-forward / Refresh:
  - [Element nào carry, element nào refresh]
```

**Logic cross-check:**
- Compare thesis cũ với current FA data (Stage 1b), dòng tiền (1c), news 30-90 ngày (1g + 1h)
- Identify which elements đã change, which still valid
- Output: list element carry-forward + element refresh

**Present tại Checkpoint 0** (giữa Stage 0 và Stage 1): "Đây là eval báo cáo cũ. Anh có muốn (a) Carry-forward các element pass + refresh các element fail, (b) Refresh toàn bộ thesis từ đầu, hay (c) Skip eval, không dùng báo cáo cũ?"

**Skip Stage 0 + Checkpoint 0 hoàn toàn nếu:** user không upload báo cáo cũ.

## 3. Stage 1 — Data Acquisition (16 sub-step)

Đây là **trung tâm của pack**. Mọi sub-step phải thực hiện theo depth mode (xem mục 4 dưới về depth mode coverage).

### 3.1. Sub-step 1a — Stock info + Type classification

**Source:** `stock_info` (`agent_db`)

**Pull fields:**
- `ticker`, `company_name`, `industry_name`, `industry_short` (mã ngắn)
- `business_area` (mô tả business)
- `listing_date`, `exchange` (HOSE/HNX/UPCOM)
- `overview` (giới thiệu công ty)
- `major_shareholders` nếu có
- `subsidiaries` nếu có

**Logic Type classification:**

```
Map industry_name → Type:

NGANHANG (Banking) → Type NH
CHUNGKHOAN (Securities) → Type CK
BAOHIEM (Insurance) → Type BH
Tất cả 21 ngành còn lại → Type SXKD (Sản xuất kinh doanh / General industrial)
```

**Output cho user:**

> Mã **[VNM]** thuộc ngành **[THUCPHAM]**, áp dụng framework **[SXKD]**.
> Em sẽ phân tích theo lens chính: business model + 4 kịch bản SXKD (Value Play / Value Trap / Growth at Premium / Cycle Top) + lens ngành Thực phẩm.

**Edge cases:**
- **Conglomerate** (VIC, MSN, SAB...): pick type theo segment chiếm >60% doanh thu. Nếu mixed, fall back SXKD + note rõ "doanh nghiệp đa ngành, phân tích chính theo segment X chiếm Y% doanh thu". Xem `_04` mục Edge cases.
- **Holding company** (vd HDG): classification theo BCTC consolidated.
- **Mã không có `industry_name`** trong DB: REFUSE phân tích, request user clarify hoặc check lại ticker.

### 3.2. Sub-step 1b — FA data từ DB

**Source:**
- `stock_finstats` — BCTC quarterly (`financial_statements.quarterly`) + annual (`financial_statements.annual`)
- `stock_snapshot.valuation_ratios` — định giá hiện tại
- `industry_finstats` — median + percentile 3-5Y cho ngành

**Pull:**
- BCTC quarterly **8-12 kỳ gần nhất** (2-3 năm)
- BCTC annual **3-5 năm** gần nhất
- Định giá hiện tại: P/E, P/B, EV/EBITDA, P/S (nếu có)
- Định giá lịch sử: percentile 3Y và 5Y của bản thân mã + so với industry median
- ROE, ROA, ROIC, gross margin, EBIT margin, net margin (trend)
- Doanh thu YoY, EBIT YoY, Net Income YoY
- Net debt / EBITDA, interest coverage, current ratio
- FCF margin, FCF conversion (CFO / Net Income)
- Working capital cycle (receivable days, inventory days, payable days)
- Capex / revenue

**Output expected:**
- Bảng historical 3-5 năm (1 row/năm) với key metrics
- Bảng quarterly 8 kỳ gần nhất (cho trend ngắn hạn)
- Valuation table: hiện tại vs phân vị 3Y, 5Y vs industry median
- 1-2 đoạn highlight trend material nhất

### 3.3. Sub-step 1c — Dòng tiền + Technical zone snapshot

**Source:**
- `stock_snapshot` — `money_flow_score.day_score`, `money_flow_score.week_score`, `technical_zone.overall.d/w/m/q` (zone đa khung), `industry_rank_pct`, `market_rank_pct`
- `stock_recent` — `series` 4 tuần gần nhất cho trend dòng tiền

**Pull:**
- `day_score` + `week_score` hiện tại
- `technical_zone.overall.d/w/m/q` (zone day/week/month/quarter)
- `industry_rank_pct` + `market_rank_pct` (percentile mã trong ngành + thị trường)
- 4-tuần `week_score` recent → xu hướng dòng tiền tăng/giảm
- VSI (volume strength index — thanh khoản)

**Output expected:**
- 1 đoạn 3-5 dòng diễn giải dòng tiền + technical zone (theo K hygiene `K_agent_db_00` mục 5.2)
- Vd: "Dòng tiền tuần dương, điểm 32, mã đứng top 15% mạnh nhất ngành. Vùng kỹ thuật khung tuần ở mức tích cực (A), khung tháng vùng mạnh (AA)."

**Lưu ý K hygiene:** Không lộ `vsi`, `day_score`, `week_score`, `zone AAA`, `industry_rank_pct` raw.

### 3.4. Sub-step 1d — Khối ngoại + Tự doanh

**Source:**
- `stock_snapshot.nn` — khối ngoại (`week.net_value`, `month.net_value`, `quarter.net_value`)
- `stock_snapshot.td` — tự doanh (tương tự)

**Pull:**
- Net position khối ngoại: tuần, tháng, quý (tỷ VND)
- Net position tự doanh: tuần, tháng, quý
- Xu hướng (mua ròng / bán ròng) liên tục bao nhiêu phiên

**Output expected:**
- 1-2 dòng cho mỗi nhóm
- Vd: "Khối ngoại mua ròng 245 tỷ tuần qua, lũy kế tháng dương 580 tỷ — xu hướng dồn dập 8 phiên liên tiếp mua ròng. Tự doanh trung tính tuần."

### 3.5. Sub-step 1e — Major shareholders + Ownership structure

**Source:**
- `stock_info.major_shareholders` (nếu DB có)
- Web search bổ sung nếu DB không đủ

**Pull:**
- Top 5-10 cổ đông lớn + % sở hữu
- Free float (cổ phiếu lưu hành tự do)
- State ownership %
- Foreign ownership % + foreign cap headroom (room còn lại theo regulation)

**Output expected:**
- Bảng top 5 cổ đông + %
- 1 dòng: "Free float [X]%, sở hữu nhà nước [Y]%, sở hữu nước ngoài [Z]% / cap [W]%"
- Flag nếu:
  - State ownership > 50% (rủi ro political)
  - Foreign ownership chạm cap (rủi ro pressure khi foreigner muốn mua thêm — vd VNM, MWG)
  - 1 cổ đông tư nhân nắm > 60% (rủi ro key person)

### 3.6. Sub-step 1f — Corporate actions recent

**Source:**
- `data_briefing` block corporate actions nếu có
- Web search bổ sung: "[Tên công ty] cổ tức / phát hành / ESOP / mua lại 2024 2025 2026"

**Pull (rolling 12 tháng):**
- Cổ tức tiền mặt + cổ tức cổ phiếu (tỷ lệ + ngày chia)
- Phát hành riêng lẻ / chào bán công chúng / chào bán cho cổ đông hiện hữu
- ESOP (mức dilution, giá phát hành, điều kiện vesting)
- Mua lại cổ phiếu quỹ (volume, giá, status)
- M&A: thâu tóm hoặc bị thâu tóm
- Chia tách / sáp nhập subsidiary

**Output expected:**
- Bảng 3-5 corporate action recent với ngày
- Flag nếu dilution lớn (>5% từ ESOP/private placement), share buyback active, hoặc M&A pending

### 3.7. Sub-step 1g — News từ DB (rolling 30-90 ngày)

**Source:**
- `news_today_feed` (tin hôm nay)
- `news_history_feed` (lịch sử rolling 30 ngày)
- `data_briefing.news` block nếu có

**Filter:**
- Filter ticker = mã đang phân tích
- Filter industry = ngành mã đang phân tích
- Filter date: 30 ngày Standard, 90 ngày Deep
- Sort by impact (HIGH > MID > LOW theo `K_agent_db_05` framework chấm impact) + date desc

**Pull 5-10 tin material nhất theo `news_type`:**
- `doanh_nghiep` (tin doanh nghiệp): kết quả kinh doanh, công bố thông tin, dự án, M&A
- `quoc_te` (tin quốc tế): tác động tới ngành / công ty (vd Fed cho ngân hàng, OPEC cho dầu khí)
- `trong_nuoc` (tin trong nước): chính sách, vĩ mô VN
- `thong_cao` (thông cáo SBV/UBCK): regulatory action

**Output expected per news:**
- Ngày + headline + 1-2 dòng impact + nguồn (URL finext.vn nếu có article_slug)
- Diễn giải impact (HIGH/MID/LOW theo `K_agent_db_05`) — K hygiene: KHÔNG lộ "HIGH/MID/LOW" raw, dịch "tác động mạnh / vừa / nhẹ"

### 3.8. Sub-step 1h — Web search news (VN equity + EN macro tuỳ ngành)

**BẮT BUỘC theo `K_agent_db_00` mục 2** — không được dùng trí nhớ, phải web search song song DB.

**Query strategy:**

| Loại tin | Query VN | Query EN (nếu cần) |
|---|---|---|
| Tin công ty cụ thể | "[Tên công ty VN] tin tức [2025/2026]"; "[Ticker] tin tức gần nhất" | (không cần — VN focus) |
| Sell-side reports | "[Tên công ty] báo cáo phân tích [tên broker VN: SSI, VCSC, HSC, MAS, MBS]" | "[Ticker] Vietnam research report" (cho large cap có EN coverage) |
| Tin ngành VN | "Ngành [tên] Việt Nam tin tức 2026" | (không cần) |
| **Macro toàn cầu (ngành tài chính)** | (kèm nguồn VN) | "FOMC dot plot latest", "Fed funds rate", "ECB rate decision", "US 10Y Treasury yield" |
| **Macro toàn cầu (ngành dầu khí)** | (kèm nguồn VN) | "OPEC+ production cut", "Brent oil price", "WTI crude", "EIA inventory" |
| **Macro toàn cầu (ngành kim loại)** | (kèm nguồn VN) | "LME copper / aluminum / zinc price", "Iron ore 62%", "HRC China", "US Section 232 tariff" |
| **Macro toàn cầu (ngành nông nghiệp / thực phẩm)** | (kèm nguồn VN) | "USDA report", "ICCO cocoa", "ICO coffee", "TOCOM rubber" |
| **Macro toàn cầu (ngành vận tải)** | (kèm nguồn VN) | "Baltic Dry Index", "container freight rate", "IATA passenger traffic" |
| **Macro toàn cầu (ngành công nghệ)** | (kèm nguồn VN) | "TSMC capex", "NVIDIA earnings", "AI capex global" (signal indirect) |

**Nguồn VN ưu tiên:**
- CafeF, Vietstock, Finext, NDH (Người Đồng Hành), ĐTCK (Đầu tư Chứng khoán), Báo Đầu Tư, VnEconomy, BizLive
- Báo chuyên ngành: VnExpress, Tuổi Trẻ kinh tế, Thanh Niên kinh tế
- Broker VN: ssi.com.vn, vcsc.com.vn, hsc.com.vn, mbs.com.vn (sell-side reports)

**Nguồn EN cho macro (chỉ khi ngành thực sự liên quan):**
- Bloomberg, Reuters, FT, official Fed/ECB/OPEC/IEA/USDA statements
- KHÔNG dùng EN cho tin equity VN (vì nguồn VN đầy đủ + chuẩn local hơn)

**Output expected:**
- 3-7 tin web search bổ sung DB (tin mới hơn DB hoặc nguồn ngoài DB)
- 2-3 sell-side view nếu có (broker VN nói gì về mã / ngành)
- 1-2 macro insight EN nếu ngành liên quan financial / commodity

### 3.9. Sub-step 1i — BCTC PDF forensic (15-point checklist)

**Source:** PDF user upload (BẮT BUỘC).

**15-point checklist** theo thứ tự ưu tiên, focus **thuyết minh BCTC**:

1. **Audit opinion**
   - Loại: Unqualified (sạch) / Qualified (có ngoại trừ) / Adverse (không chấp nhận) / Disclaimer (từ chối)
   - Going concern note (cảnh báo khả năng hoạt động liên tục)?
   - Emphasis of matter (chú ý vấn đề trọng yếu)?

2. **Chính sách kế toán** thay đổi gần đây
   - Thay đổi method nhận doanh thu (revenue recognition)?
   - Thay đổi inventory method (FIFO/Weighted average)?
   - Thay đổi cách trích khấu hao?
   - Thay đổi cách đánh giá impairment?

3. **Doanh thu segment + customer concentration**
   - Breakdown doanh thu theo segment
   - Top 5 / Top 10 khách hàng chiếm bao nhiêu % (red flag nếu Top 1 > 20% hoặc Top 5 > 50%)

4. **Related party transactions (giao dịch bên liên quan)**
   - Tổng giá trị giao dịch bên liên quan
   - % so với tổng doanh thu / tổng chi phí (red flag nếu > 20%)
   - Loại giao dịch: bán hàng, mua hàng, cho vay, đi vay, bảo lãnh

5. **Off-balance sheet items**
   - Bảo lãnh cho bên thứ ba (vd bảo lãnh cho công ty con / liên kết)
   - Cam kết tài chính chưa thực hiện
   - SPV / fund không hợp nhất

6. **Contingent liabilities**
   - Kiện tụng đang xử lý + estimated liability
   - Tax claim chưa giải quyết
   - Bảo hành sản phẩm (warranty provision)

7. **Capex commitment + lease commitment**
   - Capex đã ký hợp đồng chưa giải ngân (future cash drain)
   - Lease commitment dài hạn (theo IFRS 16 / VAS tương ứng)

8. **Debt covenant compliance**
   - Loại covenant: net debt/EBITDA, interest coverage, current ratio
   - Hiện tại có gần breach không?
   - Có buộc trả sớm (acceleration) nếu breach không?

9. **Inventory aging + provisioning**
   - Inventory days trend (tăng nhanh = tồn kho cũ)
   - Provisioning ratio
   - **Đặc biệt ngành BĐS, SXKD công nghiệp** (tồn kho có thể obsolete)

10. **Receivable aging + provisioning**
    - Receivable days trend
    - Aging breakdown: <90 ngày, 90-180 ngày, >180 ngày, >1 năm
    - Provisioning ratio
    - **Đặc biệt ngành XAYDUNG** (delay payment government)

11. **Deferred tax + tax dispute**
    - Deferred tax asset / liability balance
    - Tax position aggressive (vd tax holiday, transfer pricing concerns)
    - Tax dispute pending với cơ quan thuế

12. **Goodwill / intangible impairment risk**
    - Goodwill balance từ M&A trước
    - Có test impairment hàng năm chưa? Kết quả?
    - **Đặc biệt sau M&A lớn** (vd MSN sau M&A WinCommerce)

13. **ESOP / share-based payment dilution risk**
    - Total ESOP outstanding + tỷ lệ dilution so với số CP đang lưu hành
    - Vesting schedule
    - Giá phát hành so với market price

14. **Subsequent events** (sự kiện sau ngày BCTC)
    - Sau ngày kết thúc kỳ BCTC nhưng trước ngày phát hành báo cáo
    - Material events (M&A, kiện tụng, regulatory action)

15. **Note dành riêng theo Type:**

    **NH (Banking):**
    - LDR (Loan-to-Deposit Ratio)
    - CAR breakdown: Tier 1 + Total
    - NPL theo nhóm 2-5 (Group 2, 3, 4, 5) — chú ý: NPL = Group 3-5 theo SBV
    - LLR (Loan Loss Reserve) coverage
    - Loan concentration (top 10 khách / total loan)
    - Fee income mix (bancassurance, cards, FX, advisory)
    - Off-balance sheet exposure (L/C, guarantees)

    **CK (Securities):**
    - Margin loan breakdown by collateral + concentration
    - Prop book composition (equity, bond, derivative)
    - Market risk VaR
    - Customer money safekeeping breakdown
    - Bond underwriting commitments outstanding

    **BH (Insurance):**
    - Technical reserve (UPR + IBNR + Outstanding claims)
    - Claim reserve adequacy
    - Investment portfolio breakdown (govt bond, corp bond, equity, real estate)
    - Solvency margin
    - Reinsurance recoverable

    **SXKD (General):**
    - Working capital cycle trend
    - Capex split (maintenance vs growth)
    - Segment profitability
    - Customer/supplier concentration

**Output expected:**
- 1 bảng 15-point checklist với status (✓ Pass / ⚠ Flag / ✗ Concern)
- Mỗi flag/concern có 1-2 dòng giải thích + page reference trong BCTC
- 1 đoạn tổng kết "Forensic verdict" 3-5 dòng

**Citation format khi cite BCTC:**
- "BCTC [Q kỳ] [VNM] soát xét, [Tên thuyết minh] trang [X]"

### 3.10. Sub-step 1j — Sector context

**Source:** `K_sector_framework`

**Logic:**
- Industry có CFA cover (mục 5.X trong `K_sector_framework`) → pull bullet DD/MP/SI/PM/ESG most material (3-5 câu)
- Industry không có cover → pull universal framework mục 3 + chọn 3-5 câu most material

**Output expected:** 1-2 đoạn (3-7 dòng tổng) về industry context.

**K hygiene:** KHÔNG lộ tên dimension "DD/MP/SI/PM/ESG" trong output user-facing. Viết tự nhiên: "Drivers chính của ngành...", "Vị thế cạnh tranh...", "Trend dài hạn..."

### 3.11. Sub-step 1k — Macro context relevant

**Source:**
- `other_data` filter theo mapping ngành (xem `K_agent_db_00` mục 8)
- Web search EN nếu ngành cần (financial → Fed, oil → OPEC, metal → LME, etc.)

**Mapping ngành ↔ Macro variables:**

| Ngành | Macro variables key |
|---|---|
| NGANHANG | Lãi suất điều hành SBV, lãi suất huy động/cho vay, tỷ giá USD/VND, USD index, US Treasury yield, Fed funds rate |
| BAOHIEM | Lãi suất 10Y (cho life), GDP growth |
| CHUNGKHOAN | ADV thị trường, margin balance thị trường, VN-Index level, foreign flow |
| BDS | Lãi suất huy động/cho vay, lãi suất mortgage, Circular regulation update |
| XAYDUNG | Giá thép HRC, giá cement, giá nhựa đường |
| DAUKHI | Brent, WTI, gas TTF, OPEC+ production |
| KIMLOAI | LME (đồng, kẽm, nhôm), iron ore 62%, HRC China |
| THUCPHAM | Giá nguyên liệu (sữa, đường, gạo, dầu thực vật) + chuỗi cung ứng |
| NONGNGHIEP | Giá cao su (TOCOM), cà phê (ICO), gạo (USDA), hồ tiêu, điều |
| THUYSAN | Giá cá tra / tôm xuất khẩu + anti-dumping tariff US/EU |
| VANTAI | Baltic Dry Index, container freight, giá dầu, IATA passenger |
| BANLE | Tổng mức bán lẻ VN, CPI, disposable income trend |
| CONGNGHE | (mostly company-specific, ít macro driver direct) |
| HOACHAT | Giá feedstock + giá end product (NH3, urê, P2O5) |
| DETMAY | Giá cotton, USD/VND, US/EU order flow |
| KHOANGSAN | Giá commodity tương ứng (apatite, titan, đồng, vàng) |
| TIENICH | Giá điện EVN, giá nước, giá khí, regulation tariff |

**Output expected:** 1-2 đoạn (3-6 dòng) về macro context liên quan đến horizon phân tích.

### 3.12. Sub-step 1l — Peer comparison (internet-first + thanh khoản filter)

**Logic 3 bước:**

**Bước 1: Identify peer via web search**

Query:
- "[Tên công ty] peer / cạnh tranh / cùng ngành Việt Nam"
- "[Ticker] competitor Vietnam stock market"
- "Báo cáo phân tích [Tên công ty] [tên broker VN] universe coverage"
- "Top doanh nghiệp ngành [tên ngành] Việt Nam niêm yết"

Tham khảo:
- Sell-side coverage universe (SSI, VCSC, HSC...)
- Wikipedia / báo chuyên ngành VN list top players
- Báo cáo phân tích ngành định kỳ của các broker

Output Bước 1: **5-8 candidate peer** ticker.

**Bước 2: Filter thanh khoản + market cap**

Cross-check với DB:
- ADV (Average Daily Volume) ≥ 30 tỷ VND/ngày (pull từ `history_stock` 60 phiên gần nhất aggregate volume × price). Mid/Large cap có thanh khoản đủ
- Market cap top 50 trong ngành (cross-check `stock_snapshot.valuation_ratios.market_cap`)
- **EXCLUDE:**
  - Market cap < 1.000 tỷ VND (small cap, info quality kém, dễ manipulation)
  - Mã có scandal / suspended / chậm BCTC / cảnh báo / kiểm soát đặc biệt
  - Mã mới list < 2 năm (chưa đủ history)
  - Mã không có analyst coverage (no sell-side report = info kém)

Output Bước 2: **3-5 peer** đạt filter.

**Bước 3: Pull data peer + compose compare table**

Source: `stock_finstats` + `stock_snapshot.valuation_ratios` + `industry_finstats` (median ngành)

Pull cho mỗi peer:
- Market cap (tỷ VND)
- ADV tháng (tỷ VND/ngày)
- P/E TTM
- P/B
- EV/EBITDA TTM
- ROE TTM
- Revenue growth YoY
- Net profit growth YoY
- Gross margin
- Net debt/EBITDA

Output expected: **Bảng peer compare** (1 row = 1 mã, gồm mã chính + 3-5 peer + median ngành) với 8-10 cột metrics. Render rigid (mọi mã đủ data hoặc note "n/a").

### 3.13. Sub-step 1m — ADV / Liquidity tier

**Source:** `history_stock` 60 phiên gần nhất

**Pull:**
- ADV tháng = avg(price × volume) trong 60 phiên gần nhất → tỷ VND/ngày
- ADV tuần (10 phiên gần nhất) cho trend
- Volume volatility (std dev của daily volume)

**Tier classification:**
- **Large cap thanh khoản cao:** ADV ≥ 200 tỷ VND/ngày
- **Mid cap thanh khoản tốt:** ADV 50-200 tỷ VND/ngày
- **Mid cap thanh khoản vừa:** ADV 20-50 tỷ VND/ngày
- **Small cap thanh khoản thấp:** ADV < 20 tỷ VND/ngày (flag risk khi sizing)

**Output expected:** 1 dòng — "ADV tháng [X] tỷ VND/ngày, [tier]. Liquidity awareness: [hệ luỵ cho sizing — vd nếu mã nhỏ, single trade > 5% ADV gây slippage]."

### 3.14. Sub-step 1n — Earnings calendar

**Source:**
- `data_briefing` block earnings calendar nếu DB có
- Web search: "[Tên công ty] họp ĐHCĐ [năm] / công bố BCTC quý [N]"
- IR section trên website công ty (qua web search)

**Pull:**
- Next BCTC reporting date (quý kế tiếp + năm)
- Consensus EPS (nếu có từ sell-side)
- AGM (ĐHCĐ thường niên) date upcoming
- Major announcement scheduled (vd hợp đồng ký kết dự kiến, IPO subsidiary)

**Output expected:** 1 đoạn 2-4 dòng với 1-3 calendar event upcoming và date cụ thể.

### 3.15. Sub-step 1o — ESG controversy scan

**Source:** Web search (DB ít có data này)

**Query (2-3 query):**
- "[Tên công ty] vi phạm / xử phạt / scandal [năm]"
- "[Tên công ty] môi trường / lao động / tranh chấp"
- "[Tên công ty] kiện tụng / điều tra"

**Cross-reference:** `K_sector_framework` mục 5.X ESG hotspots ngành tương ứng.

**Output expected:**
- Nếu không có controversy → 1 dòng: "Quét tin ESG controversy 12 tháng qua không phát hiện sự cố material."
- Nếu có → list 1-5 controversy với:
  - Date + headline
  - Severity (HIGH / MID / LOW) — K hygiene: dịch "nghiêm trọng / trung bình / nhẹ"
  - Status (resolved / ongoing / settled)
  - Impact tới thesis (downgrade conviction nếu HIGH ongoing)

### 3.16. Sub-step 1p — Khách hàng / Nhà cung cấp / Channel mix (Value chain data)

**MANDATORY cho SXKD Standard+; optional cho Quick; SKIP cho NH/CK/BH** (đã có sub-step 1i type-specific FA cho 4 nhóm tài chính).

**Mục đích:** Pull data hỗ trợ phân tích chuỗi giá trị doanh nghiệp ở Stage 2 Phần 2 sub-section 3 (xem `P_stock_report_03` mục 2.2 sub-section 3 + `P_stock_report_02` mục 2.6).

**Source:**
- Thuyết minh BCTC mới nhất (sub-step 1i đã có PDF) — đào sâu các phần:
  - Phải thu khách hàng (Receivables) — list top khách hàng nếu thuyết minh có disclosure
  - Phải trả người bán (Payables) — list top nhà cung cấp nếu thuyết minh có
  - Doanh thu chi tiết theo khu vực địa lý / theo segment / theo loại khách hàng
  - Mua bán nội bộ (Related party transactions) — đã pull ở 1i point 4, cross-link lại
- BCTN (Annual Report) năm gần nhất — phần Business overview thường liệt kê khách hàng chính, kênh phân phối
- Web search (2-3 query):
  - "[Tên công ty] khách hàng lớn / đối tác / hợp đồng"
  - "[Tên công ty] kênh phân phối / hệ thống phân phối / điểm bán"
  - "[Tên công ty] nhà cung cấp / nguyên liệu / nguồn nhập"
- IR presentation gần nhất nếu có (web search)

**Pull (mandatory items SXKD):**

1. **Top khách hàng:** 3-10 khách hàng lớn nhất + % doanh thu (nếu disclose). Note nếu top 5 > 50% → concentration cao.
2. **Top nhà cung cấp:** 3-10 nhà cung cấp lớn + % giá vốn (nếu disclose). Note nếu top 3 > 60% nguyên liệu chính → supplier power cao.
3. **Channel mix:** breakdown % doanh thu theo kênh:
   - B2B vs B2C (nếu applicable)
   - GT (General trade — chợ/cửa hàng nhỏ) vs MT (Modern trade — siêu thị/CVS) vs Online vs HORECA vs Export — tuỳ ngành
   - Bán trực tiếp vs qua nhà phân phối (NPP)
4. **Geographic mix:** % doanh thu Bắc / Trung / Nam VN + Export %
5. **Segment mix:** breakdown doanh thu / lợi nhuận theo segment (nếu doanh nghiệp đa ngành — pull từ thuyết minh BCTC consolidated segment note)
6. **Capacity + utilization rate:** công suất thiết kế + tỷ lệ vận hành hiện tại (từ thuyết minh tài sản cố định + BCTN)
7. **Nguyên liệu key + cost structure:** 2-3 nguyên liệu chính + % giá vốn + tỷ lệ import vs domestic (FX exposure)

**Pull (optional items Deep mode):**

8. Lock-up contract / commitment dài hạn với KH lớn / NCC lớn (vd LNG offtake, mining royalty)
9. Backlog order / pipeline (cho XAYDUNG, CONGNGHE, BDS)
10. Pricing strategy / price list change history (vd VNM tăng giá sữa Q2/2026)

**Fail-soft rule:**
- Thuyết minh không disclose top KH/NCC cụ thể → search BCTN + web; nếu vẫn không có → note "Thuyết minh BCTC không disclose top khách hàng/nhà cung cấp chi tiết; pull proxy từ [BCTN / báo chí / sell-side report]"
- Web search 0 result → note "Không tìm được thông tin chi tiết về [X]; phân tích chuỗi giá trị dựa trên industry-level từ K_sector_framework"
- Mã quá kín thông tin → flag conviction downgrade do data gap value chain (note rõ trong audit trail)

**Output expected:**

```
**Top khách hàng (SXKD):**
- KH1: [Tên] — [%] doanh thu (nguồn: thuyết minh BCTC trang X / BCTN)
- KH2: ...
- ...
- Concentration verdict: top 5 chiếm [X]% → [cao/trung/thấp]

**Top nhà cung cấp:**
- NCC1: [Tên] — [%] giá vốn (nếu disclose) / nguyên liệu [Y]
- ...
- Concentration verdict: ...

**Channel mix:**
- B2C vs B2B: [X%] / [Y%]
- Modern trade vs General trade vs Online: [...]
- Export %: [...]

**Geographic mix VN:** Bắc [X%] / Trung [Y%] / Nam [Z%] | Export: [W%]

**Capacity:** [công suất tỷ tấn/năm / triệu sản phẩm] | Utilization: [X%]

**Nguyên liệu key:**
- [Tên NL1]: [X%] giá vốn — [Y%] import / [Z%] domestic
- ...
```

**K hygiene:**
- Audience nội bộ: keep tên KH/NCC cụ thể
- Audience KH: nếu KH/NCC là **đối thủ của KH** hoặc thông tin nhạy cảm thương mại → ẩn tên cụ thể, dùng generic "1 tập đoàn FDI lớn ngành điện tử", "1 chuỗi bán lẻ top 3 VN". Giữ % concentration.

**Cross-link:**
- Output sub-step 1p → feed vào Stage 2 Phần 2 sub-section 3 "Vị trí chuỗi giá trị" (`P_stock_report_03` mục 2.2)
- Reference framework: `P_stock_report_02` mục 2.6
- Industry context: `K_sector_framework` mục 5.X (industry value chain map)

## 4. Depth mode coverage cho Stage 1

| Sub-step | Quick | Standard | Deep |
|---|---|---|---|
| 1a Stock info + Type | ✓ | ✓ | ✓ |
| 1b FA DB | ✓ (basic 3Y annual only) | ✓ (3-5Y annual + 8Q quarterly) | ✓ (5Y annual + 12Q quarterly) |
| 1c Dòng tiền + Tech zone | ✓ | ✓ | ✓ |
| 1d Khối ngoại + Tự doanh | ✓ | ✓ | ✓ |
| 1e Major shareholders | (skip) | ✓ | ✓ |
| 1f Corporate actions | (skip) | ✓ | ✓ |
| 1g News DB | ✓ (30 ngày) | ✓ (30-60 ngày) | ✓ (90 ngày) |
| 1h Web search news | ✓ (2 query) | ✓ (4-5 query) | ✓ (full 5-7 query + EN macro nếu cần) |
| **1i BCTC PDF forensic** | (skip — nhưng BCTC PDF vẫn bắt buộc upload) | ✓ (15-point checklist) | ✓ (15-point + đào sâu thuyết minh) |
| 1j Sector context | (skip) | ✓ | ✓ |
| 1k Macro relevant | (skip) | ✓ | ✓ |
| 1l Peer compare | (skip) | ✓ (3 peer) | ✓ (5 peer) |
| 1m ADV / Liquidity | ✓ | ✓ | ✓ |
| 1n Earnings calendar | ✓ | ✓ | ✓ |
| 1o ESG controversy | (skip) | ✓ (basic) | ✓ (full + cross-ref K_sector_framework) |
| **1p Value chain data (KH/NCC/channel)** | (skip — SXKD; N/A NH/CK/BH) | ✓ SXKD (basic) | ✓ SXKD (full + optional items 8-10) |

**Quick mode:** 11/16 sub-step active. Skip 1e, 1f, 1i, 1j, 1k, 1l, 1o, 1p. Output 1-2 trang. Đủ cho ad-hoc câu hỏi nhanh.

**Standard mode:** Full 15/16 sub-step (1p chỉ SXKD; NH/CK/BH skip 1p — đã có 1i type-specific). Output 3-5 trang. Mid-ground hợp lý.

**Deep mode:** Full 15-16/16 sub-step với độ sâu cao nhất. Output 5-10 trang. Pre-Tier 5C level.

**Lưu ý Quick mode:** BCTC PDF vẫn bắt buộc upload (gate strict). Chỉ là không làm 15-point forensic detailed. Vẫn dùng PDF để verify số liệu DB nhanh.

## 5. Fail-soft rule khi thiếu data

| Thiếu | Action |
|---|---|
| BCTC PDF | REFUSE chạy — gate strict, không flex |
| Ticker không tồn tại | REFUSE — request user check |
| Mã suspended / cảnh báo / kiểm soát đặc biệt | Fail-soft với big warning prepend, vẫn phân tích nhưng note rõ trạng thái + recommendation Avoid as default |
| BCTC quý chưa soát xét | Proceed, note "BCTC quý [N] chưa soát xét" |
| DB không có news ticker | Web search bù, note trong audit trail |
| Web search không trả result | Note "Không tìm được tin web về [X]", không bịa |
| Sell-side không có coverage | Skip sell-side view, đề cập "Không có analyst coverage broker VN cho mã này" |
| Peer < 3 mã đạt filter | Mở rộng filter (giảm ADV threshold xuống 10 tỷ) hoặc dùng peer ngành tương cận, note rõ |
| Industry không có K_sector_framework cover | Dùng universal framework mục 3 của `K_sector_framework` |
| Macro EN search timeout | Skip macro EN, dùng VN equivalent + note |
| Earnings calendar không tìm được | Note "Chưa rõ schedule BCTC kế tiếp" |
| ESG controversy search không kết quả | Note "Không phát hiện controversy material" (positive signal) |
| 1p value chain data: top KH/NCC không disclose + BCTN + web không có | Note "Data gap value chain — phân tích chuỗi giá trị dựa trên industry-level từ K_sector_framework"; flag downgrade conviction (HIGH → MID) cho SXKD vì value chain là lens mandatory |

## 6. Output audit trail bắt buộc

Cuối báo cáo, render block metadata audit trail (xem `O_stock_report_00` mục Metadata):

```yaml
data_sources:
  db_collections: [stock_info, stock_finstats, stock_snapshot, ...]
  web_search_queries: [...]
  pdf_files: [BCTC_VNM_Q4_2025_soát xét.pdf, ...]
  external_sources: [...]
data_freshness:
  snapshot_date: 2026-05-30
  bctc_latest: Q4/2025 soát xét
  news_window: 60 ngày
  macro_data: 2026-05-29
sub_steps_executed: [1a, 1b, 1c, ..., 1n, 1o, 1p]
sub_steps_skipped: [1e, 1f, 1p] (Quick mode) hoặc [1p] (NH/CK/BH Standard+)
flags:
  - Foreign cap chạm 49% (sub-step 1e)
  - Related party transaction 18% revenue (sub-step 1i point 4)
  - Top 5 khách hàng chiếm 62% doanh thu — concentration cao (sub-step 1p)
```

Khi audience = KH: audit trail trong báo cáo có thể rút gọn hoặc tách ra file metadata riêng (theo `O_stock_report_00`).
