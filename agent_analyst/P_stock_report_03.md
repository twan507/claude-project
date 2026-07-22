# P_stock_report_03 — Stage 2 Compose + Output structure + 3 depth mode

File này quy định cách compose báo cáo sau khi Stage 1 (data acquisition ở `_01`) hoàn tất, theo 6-7 phần rigid + 3 depth mode + Variant Perception rule + Pair compare mode.

## 1. Stage 2 workflow overview

```
Sau Stage 1 (data đã đủ) → Stage 2:

Bước 1: Synthesis — tổng hợp findings từ 16 sub-step Stage 1 (1a-1p)
   ├── Type-specific lens (P_stock_report_02)
   ├── Industry lens (K_sector_framework mục 5.X)
   ├── Forensic findings (Stage 1i 15-point)
   └── Catalyst + bear case + disconfirming identified

Bước 2: Conviction + Recommendation determination
   ├── Long / Watch / Avoid?
   ├── Conviction HIGH / MID / LOW?
   ├── Horizon (1-3m / 3-6m / 6-12m theo pre-flight)?
   └── Apply Variant Perception rule per depth mode (xem mục 4)

Bước 3: Compose 6-7 phần draft

Bước 4: Self-audit (xem _04 mục 1)

CHECKPOINT 1 — Thesis core review (xem mục 6)

Bước 5 (sau CP1 confirm): Finalize 6-7 phần đầy đủ

CHECKPOINT 2 — Output draft review (optional cho Quick mode)
```

## 2. Output structure — 6-7 phần rigid heading

Heading bắt buộc giữ nguyên (không flex). Sub-section trong phần có thể flex theo depth mode (mục 3).

```
Phần 1 — Khuyến nghị tổng quan
Phần 2 — Phân tích doanh nghiệp (theo type-specific)
Phần 3 — Bối cảnh ngành & vĩ mô (skip Quick)
Phần 4 — Phân tích tài chính & định giá (+ peer compare nếu Standard+)
Phần 5 — Tin tức & Catalyst
Phần 6 — Bear case & Disconfirming signal
Phần 7 — Exit triggers (chỉ Long recommendation, skip Watch/Avoid)
[Optional] Appendix data sources (Deep mode)
```

### 2.1. Phần 1 — Khuyến nghị tổng quan

**Mục đích:** 1-trang executive summary đầu báo cáo. Audience đọc 1 phần này phải nắm được core thesis.

**Mandatory content:**

```
Recommendation: [Long / Watch / Avoid]
Conviction: [HIGH / MID / LOW]
Horizon: [1-3m / 3-6m / 6-12m]
Mode phân tích: [Quick / Standard / Deep]

Thesis core (1-2 dòng): [Tóm tắt 1 luận điểm chính]

Variant Perception (Standard recommended, Deep mandatory):
[Góc nhìn khác consensus + cơ sở định lượng] hoặc 
[Consensus-aligned, edge limited] nếu Standard không có VP

Key catalysts (top 2-3 với timing):
- [Catalyst 1] — [timing]
- [Catalyst 2] — [timing]

Bear case chính:
[1-2 dòng bear scenario]

Disconfirming signals (1-2 dòng measurable):
[Signal X với threshold Y]
```

**K hygiene:**
- Audience nội bộ: dùng "Long / Watch / Avoid" + "Conviction HIGH/MID/LOW"
- Audience KH: dịch "Tích cực / Theo dõi / Cẩn trọng" + "Quan điểm tích cực mạnh/trung bình/thận trọng". KHÔNG dùng "Recommendation Long" (command-style với KH).

Length: 10-15 dòng tổng cho Quick, 15-25 dòng cho Standard, 25-35 dòng cho Deep.

### 2.2. Phần 2 — Phân tích doanh nghiệp (theo type-specific)

**Mục đích:** Đào sâu business model + earnings drivers theo lens type SXKD/NH/CK/BH.

**Content theo template ở `P_stock_report_02` mục 7:**
- SXKD: business model + 4 kịch bản (Value Play / Value Trap / Growth at Premium / Cycle Top) + sub-type cycle dynamics
- NH: NIM drivers + Asset quality + Capital position
- CK: Brokerage share + Margin book + IB + Prop book risk
- BH: Premium + Underwriting + Investment + Solvency

**Mandatory sub-section:**

1. **Business overview** (1 đoạn): nguyên cốt lõi doanh nghiệp làm gì
2. **Earnings drivers** theo type (2-3 đoạn): KPIs chính + trend + breakdown
3. **Vị trí chuỗi giá trị (Value chain position)** — **MANDATORY cho SXKD Standard+, optional Quick; SKIP cho NH/CK/BH** (đã có lens type-specific tương đương): từ `P_stock_report_02` mục 2.6 (đầy đủ Porter + Smile Curve + GVC + Industry 4.0 + CFA Sector Analysis 2020 reference):
   - 3a. Chuỗi giá trị ngành (industry value chain map) — upstream → midstream → downstream + margin pool + bottleneck (mục 2.6.1)
   - 3b. Chuỗi giá trị doanh nghiệp Porter — **5 primary activities** (inbound, operations, outbound, marketing, service) + **4 support activities** (firm infrastructure, HR, technology development, procurement) + vertical integration backward/forward/full (mục 2.6.2)
   - 3c. Bargaining power Porter 5 forces + **GVC governance type** (market/modular/relational/captive/hierarchy) + **Tier supplier position** (mục 2.6.3) — bảng Standard+ mandatory
   - 3d. **Smile Curve / vị trí capture giá trị** (mục 2.6.4) — Stan Shih 1992. Position: smile bottom (manufacturing pure) / smile mid (climbing) / smile top (brand + service). Đặc biệt quan trọng cho VN context.
   - 3e. **Industry 4.0 / Digital footprint lens** (mục 2.6.5) — CFA Sector Analysis 2020 reference. Bảng 7 chiều verdict (Strong/Trung/Weak) + Industry 4.0 maturity (Leader/Par/Laggard).
   - 3f. Position summary 5-7 dòng — tích hợp tất cả lens trên (mục 2.6.6): firm captures margin ở đâu, vị trí Smile, GVC governance, Industry 4.0 readiness, expose risk ở đâu, pricing power verdict tổng hợp
4. **Competitive position** (1 đoạn): vị thế ngành, market share, moat (kết quả của vị trí chuỗi giá trị)
5. **Type-specific concerns** (1 đoạn): bear case typical của type này, có hiện diện không

**Cross-reference:**
- Stage 1a, 1b cho data
- Stage 1i forensic findings có liên quan business
- Stage 1p khách hàng / nhà cung cấp / channel (mandatory cho SXKD value chain)
- `P_stock_report_02` mục 2.6 cho framework chuỗi giá trị
- `K_sector_framework` mục 5.X cho industry value chain map theo ngành

**K hygiene chuỗi giá trị:**
- Audience nội bộ: dùng "supplier power Cao/Trung/Thấp", "vertical integration full/pure-play", "Porter 5 forces"
- Audience KH: dịch — "phụ thuộc nhà cung cấp mạnh/vừa/nhẹ", "tích hợp dọc đầy đủ / chuyên 1 mảng", BỎ jargon "Porter 5 forces" — chỉ ghi "phân tích 5 nhóm áp lực cạnh tranh"

Length: 0.5-1 trang Quick (skip 3c table, skim 3a + 3b), 1.5-2.5 trang Standard (full 3a + 3b + 3c + 3d), 2.5-3.5 trang Deep (full + ASCII diagram chuỗi giá trị ngành + vertical integration table).

### 2.3. Phần 3 — Bối cảnh ngành & vĩ mô (skip Quick)

**Mục đích:** Đặt firm trong context ngành + macro relevant. Audience hiểu được tailwind/headwind kéo dài 3-12 tháng.

**Content:**

1. **Industry structural** (1 đoạn): pull từ `K_sector_framework` mục 5.X
   - Demand drivers ngành
   - Cấu trúc cạnh tranh
   - Structural trends 3-5 năm
2. **Macro relevant** (1 đoạn): từ Stage 1k
   - 2-3 macro variables liên quan ngành
   - Direction + magnitude (vd lãi suất giảm 100bp 6 tháng tới → NIM compress ~20bp)
3. **ESG context** (optional, 1 đoạn): từ Stage 1o
   - ESG hotspot ngành + status firm
   - Material controversy nếu có

**K hygiene:**
- KHÔNG lộ "DD/MP/SI/PM/ESG" — viết tự nhiên: "Drivers chính của ngành...", "Trend dài hạn..."
- Dịch macro EN terms sang VN: "Fed funds rate" → "lãi suất Fed", "Brent" → "giá dầu Brent"

Length: SKIP Quick mode. 0.5 trang Standard, 1-1.5 trang Deep.

### 2.4. Phần 4 — Phân tích tài chính & định giá (+ peer compare)

**Mục đích:** Quantitative deep-dive vào historical financials + valuation hiện tại + peer benchmark.

**Mandatory sub-section:**

1. **Historical financials (3-5 năm)** — bảng từ Stage 1b:
   - Quick: 3Y annual basic (Revenue, EBIT, Net Income, ROE)
   - Standard: 3-5Y annual + 8Q quarterly đầy đủ KPIs theo type
   - Deep: 5Y annual + 12Q quarterly + DuPont decomposition + segment breakdown

2. **Định giá hiện tại** (1 đoạn):
   - P/E, P/B, EV/EBITDA hiện tại
   - Phân vị 3Y, 5Y so với lịch sử của mã
   - So với industry median + peer trung vị

3. **Peer compare table** (Standard+ only, từ Stage 1l):
   - 3 peer Standard, 5 peer Deep
   - 8-10 cột metrics
   - Note ưu/khuyết của mã so với peer

4. **Macro sensitivity** (Deep only, từ Stage 1k):
   - Sensitivity table: vd ngân hàng → NIM sensitivity to 50bp rate change

5. **Forensic flags** (1 đoạn, từ Stage 1i):
   - Audit opinion + going concern note
   - Top 3-5 flag material từ 15-point checklist
   - Page reference BCTC

6. **Valuation conclusion** (1 đoạn):
   - Định giá hiện tại fair / expensive / cheap so với historical + peer
   - Implied upside/downside range (Bear / Base / Bull) — chỉ Standard+ mode
   - Vd: "P/E hiện tại 18x, phân vị 65% lịch sử 5Y, ở trung bình peer. Implied upside Base ~12%, Bull ~25%, Bear -15% trong horizon 6 tháng"

**K hygiene:**
- Quy đổi đơn vị (`K_agent_db_00` mục 6): BCTC sang tỷ đồng, ratios sang %
- KHÔNG lộ ký hiệu raw: `m_pct`, `q_pct`, percentile 0-100 raw (`industry_rank_pct: 90` → "top 10% ngành")

Length: 0.5 trang Quick (skip peer + macro sens + forensic), 1-2 trang Standard, 2-3 trang Deep.

### 2.5. Phần 5 — Tin tức & Catalyst

**Mục đích:** Forward-looking — sự kiện sắp tới có thể đẩy/kéo giá.

**Mandatory sub-section:**

1. **News recap (rolling 30-90 ngày)** — 3-5 tin material nhất từ Stage 1g + 1h:
   - Format: ngày | headline | 1 dòng impact | nguồn (link nếu có)
   - Sort by impact + date desc

2. **Sell-side consensus** (Standard+ if available, từ Stage 1h):
   - Broker VN view (SSI, VCSC, HSC, MBS, MAS...)
   - Foreign broker view nếu có (cho large cap)
   - Spread của consensus (median / range target price nếu có)

3. **Catalyst pipeline** — 2-4 catalyst với timing:
   - Catalyst phải có **timing cụ thể** (ngày / quý / kỳ vọng)
   - Vd:
     - Q1/2026 BCTC publish (Earnings beat expected): cuối tháng 1/2026
     - SOCB privatization announcement: nửa cuối 2026
     - ESOP vesting tranche 2: tháng 6/2026
   - Mỗi catalyst note **conviction impact** (mạnh/vừa/nhẹ — dịch từ HIGH/MID/LOW)

4. **Earnings calendar** (1 dòng, từ Stage 1n):
   - Next BCTC reporting date
   - Consensus EPS nếu có

5. **Macro catalyst** (Deep only):
   - Macro event upcoming có thể ảnh hưởng (Fed meeting, OPEC, Circular SBV)

Length: 0.5 trang Quick (chỉ 1-2 catalyst + earnings date), 1-1.5 trang Standard, 1.5-2 trang Deep.

### 2.6. Phần 6 — Bear case & Disconfirming signal

**Mục đích:** Bear case steelmanned + disconfirming signal measurable. Đây là **gate quality** của báo cáo — bear case yếu = thesis vulnerable.

**Mandatory sub-section:**

1. **Bear case** (1-2 đoạn, theo type):
   - **Steelmanned**: trình bày bear case như chính người tin Long phản đối — không soft-pedal
   - Pull từ:
     - Stage 1i forensic flags (red flag từ thuyết minh BCTC)
     - Stage 1h negative news (tin tiêu cực web search)
     - Stage 1k macro headwind
     - Stage 1c dòng tiền âm nếu có
     - Stage 1o ESG controversy nếu có
   - Bear case typical theo type (xem `P_stock_report_02`):
     - SXKD: pricing power erosion, capex overhang, working capital deterioration, goodwill impairment, customer concentration
     - NH: NIM compression, NPL spike, CAR breach, regulatory tightening
     - CK: market crash margin call, commission compression, prop book loss
     - BH: catastrophe claim spike, mis-selling, interest down

2. **Bear target price** (Standard+ mode):
   - Implied downside Bear scenario
   - **Quan trọng:** nếu Bear cumulative target < current price → đây là red flag mạnh, phải downgrade conviction

3. **Disconfirming signals — 2-3 signal MEASURABLE**:
   - Mỗi signal có **threshold cụ thể** (số / sự kiện)
   - Vd:
     - "NIM Q2/2026 thu hẹp ≥ 20bp QoQ" (số cụ thể)
     - "NPL Group 2 inflow tăng ≥ 30% QoQ trong 2 quý liên tiếp" (số + duration)
     - "SBV ban hành Circular siết classification loan trước Q3/2026" (sự kiện)
   - KHÔNG dùng disconfirming mơ hồ: "NIM xấu đi", "kinh tế VN suy thoái" — không actionable

**Format:**

```
**Bear case steelmanned:**
[2-3 đoạn]

**Bear target price (range):** [X - Y tỷ đồng/cp] (downside Z%)

**Disconfirming signals (cái gì khiến thesis sai):**
1. [Signal 1 với threshold cụ thể]
2. [Signal 2 với threshold cụ thể]
3. [Signal 3 với threshold cụ thể] (optional cho Deep)
```

Length: 0.3 trang Quick (1 bear + 1 disconfirming), 0.7-1 trang Standard, 1-1.5 trang Deep.

### 2.7. Phần 7 — Exit triggers (CHỈ Long recommendation)

**Skip cho Watch / Avoid.**

**Mục đích:** Nếu recommendation Long → cần exit plan trước khi vào position.

**Mandatory sub-section:**

1. **Take-profit triggers**:
   - TP1: price reach Base target → reduce 50%
   - TP2: price reach Bull target → reduce 80%
   - Total exit: price reach > Bull × 1.1

2. **Stop-loss triggers**:
   - SL technical: price break support tuần / Fibonacci 61.8% retracement
   - SL fundamental: 1+ disconfirming signal materialized
   - Combined: cả 2 → exit hard

3. **Time-stop**:
   - Horizon expiry: nếu kết thúc horizon (3-6m) mà thesis chưa play out → exit hoặc re-evaluate
   - Re-evaluate date: cụ thể (vd "Review tại Q3/2026 BCTC")

4. **Catalyst trigger**:
   - Nếu catalyst chính fail (vd earnings miss consensus) → exit
   - Nếu catalyst chính materialize → review xem có hold tiếp hay book profit

**K hygiene + Wording:**
- Audience nội bộ: dùng đủ TP1/TP2/SL với số cụ thể
- Audience KH: BỎ exit trigger cụ thể (số) — chỉ ghi "Cần theo dõi các tín hiệu sau để xem xét lại quan điểm: [signal list]". KHÔNG render TP1/TP2/SL số cho KH (rule cross-pack: KH không nhận command).

Length: 0.3 trang Quick (basic TP/SL), 0.5-0.7 trang Standard, 0.7-1 trang Deep.

### 2.8. Appendix — Data sources (Deep mode only)

**Mục đích:** Full audit trail data sources cho transparency.

**Content:**

1. **DB collections used:** list `stock_info`, `stock_finstats`, `stock_snapshot`, `news_*`, etc.
2. **Web search queries executed:** list 10-15 query đã dùng
3. **PDF files received:** filename + ngày upload
4. **External sources:** sell-side reports, IR presentation, BCTN nếu có
5. **Data freshness:**
   - snapshot_date
   - BCTC latest period
   - News window (số ngày)
   - Macro data date
6. **Sub-steps executed:** [1a, 1b, 1c, ...] với ✓ / ⚠ / ✗ status
7. **Flags raised + caveats:**
   - Forensic flags (top 3-5)
   - Data gap notes
   - Conviction downgrade reasons (nếu có)

Length: 0.5-1 trang.

## 3. Depth mode coverage cho output structure

| Phần | Quick | Standard | Deep |
|---|---|---|---|
| 1 Khuyến nghị | ✓ (10-15 dòng) | ✓ (15-25 dòng) | ✓ (25-35 dòng) |
| 2 Doanh nghiệp (+ Value chain SXKD) | ✓ (0.5-1 trang) | ✓ (1.5-2.5 trang) | ✓ (2.5-3.5 trang) |
| 3 Bối cảnh ngành & vĩ mô | ✗ skip | ✓ (0.5 trang) | ✓ (1-1.5 trang) |
| 4 Tài chính & định giá | ✓ basic (0.5 trang) | ✓ (1-2 trang) + peer 3 | ✓ (2-3 trang) + peer 5 + macro sens |
| 5 Tin tức & Catalyst | ✓ basic (0.5 trang) | ✓ (1-1.5 trang) | ✓ (1.5-2 trang) |
| 6 Bear case & Disconfirming | ✓ basic (0.3 trang) | ✓ (0.7-1 trang) | ✓ (1-1.5 trang) |
| 7 Exit triggers (Long only) | ✓ basic (0.3 trang) | ✓ (0.5-0.7 trang) | ✓ (0.7-1 trang) |
| Appendix data sources | ✗ skip | ✗ skip | ✓ (0.5-1 trang) |
| **Tổng** | **1-2 trang** | **3-5 trang** | **5-10 trang** |

## 4. Variant Perception rule per depth mode

Đã giải thích concept ở `P_stock_report_00` mục 3 nguyên tắc 8. Apply rule:

| Mode | VP requirement | Nếu không có VP |
|---|---|---|
| Quick | Optional | Không flag |
| Standard | **Recommended** | Flag "Consensus-aligned thesis, edge limited" trong Phần 1; KHÔNG auto downgrade conviction |
| Deep | **Bắt buộc** | Auto downgrade conviction: HIGH → MID, MID → LOW, LOW → recommendation thành Watch (không Long) |

**Format VP trong Phần 1:**

Khi có VP:
```
**Variant Perception:**
Consensus thấy [X về mã/ngành].
Em thấy [Y khác consensus] vì [cơ sở định lượng cụ thể — vd: "biên lợi nhuận sẽ bật lại 3pp khi giá sữa nguyên liệu giảm 12% trong Q3 — consensus chưa price in"].
Nếu VP đúng → upside [Z%]. Nếu VP sai → cùng lắm về mức consensus [W%].
```

Khi không có VP (Standard mode, flag):
```
**Variant Perception:** Em chưa identify được góc nhìn khác consensus có cơ sở định lượng. Thesis hiện tại align với view chung thị trường — edge limited. Conviction giữ [MID/LOW] phản ánh thực tế này.
```

Khi không có VP (Deep mode, downgrade):
```
**Variant Perception:** Em không identify được góc nhìn khác consensus có cơ sở định lượng. Theo discipline pack Deep mode, conviction được downgrade từ [HIGH → MID] / [MID → LOW] / recommendation [Long → Watch] phản ánh việc thesis không có edge differentiation.
```

## 5. Pair compare mode (multi-stock)

Activate khi user cung cấp 1-2 ticker khác trong pre-flight câu 5.

### 5.1. Constraint pair compare

- **Tối đa 3 mã** (gồm mã chính + 1-2 mã compare). Không quá 3 vì khó focus.
- **Chỉ available ở Standard và Deep mode.** Quick mode không support (không đủ space).
- **Mã phải cùng ngành hoặc logic compare rõ.** Vd:
  - Same industry: HPG vs HSG (cùng thép); VNM vs MSN (cùng FMCG); VCB vs ACB (cùng bank private/SOCB)
  - Same theme: VRE vs DXG (cùng BĐS retail/residential)
  - Same value chain: HSG vs HPG (input vs downstream)
- Apple-to-orange (vd VNM vs VCB) → REFUSE pair compare, suggest user pick 1 mã.

### 5.2. Pair compare output structure

Output structure khác standalone — render side-by-side hoặc table format:

**Option A (Standard mode, side-by-side):**

```
Phần 1 — Khuyến nghị tổng quan PAIR
| Aspect | Mã chính | Compare 1 | Compare 2 |
|---|---|---|---|
| Recommendation | Long | Watch | Avoid |
| Conviction | HIGH | MID | LOW |
| Horizon | 3-6m | 3-6m | 3-6m |
| Thesis core | [...] | [...] | [...] |

→ Em đề xuất focus mã [chính] với conviction HIGH...

Phần 2 — Phân tích doanh nghiệp (3 mã)
[Compose từng mã ngắn 0.5-0.7 trang]

Phần 3 — Bối cảnh ngành chung
[Compose 1 lần vì cùng ngành]

Phần 4 — Bảng peer compare CHI TIẾT
[Bảng 8-10 cột × 3 mã + median ngành]

Phần 5 — Catalyst (mỗi mã 2-3 catalyst)
Phần 6 — Bear case (mỗi mã 1-2 bear)
Phần 7 — Exit triggers (chỉ mã Long)
```

**Option B (Deep mode, sequential):**

Render full 6-7 phần cho từng mã + 1 phần tổng kết "Pair selection thesis" cuối cùng.

### 5.3. Pair selection thesis

Cuối báo cáo pair compare:

```
**Lựa chọn final:**

Top pick: [Mã X]
Lý do: [3-5 bullet so sánh ưu thế]

Underperform pick: [Mã Y]
Lý do: [3-5 bullet hạn chế]

Action gợi ý (nội bộ): [Long X, Avoid Y] hoặc [Long pair X cho upside + short Y cho hedge — KHÔNG render với KH]
```

KH version: BỎ "Underperform pick" wording — chỉ ghi "Top pick" + "Cẩn trọng với [Mã Y]".

## 6. Checkpoint 1 — Thesis core review

**Activate trước Bước 5 (finalize)**. Pack có 1 CP bắt buộc (cộng CP0 optional ở Stage 0 nếu có file cũ).

**Output Checkpoint 1:**

```
Em sẵn sàng compose báo cáo đầy đủ. Trước khi finalize, anh xác nhận giúp em thesis core:

📊 [Mã X] — [Long/Watch/Avoid] | Conviction [HIGH/MID/LOW] | Horizon [Y]

🎯 Thesis core (1 dòng):
[Tóm tắt 1 dòng]

💡 Variant Perception (nếu có):
[1-2 dòng VP]

📈 Top 2 catalyst:
1. [Catalyst 1] — [timing]
2. [Catalyst 2] — [timing]

⚠ Top 2 bear case:
1. [Bear 1]
2. [Bear 2]

🚦 Disconfirming chính:
- [Signal X với threshold Y]

Anh:
(a) Xác nhận thesis như trên → em compose đầy đủ
(b) Refine thesis [xin chỉnh] → em sửa rồi compose
(c) Reject — thesis không hợp lý, em làm lại từ Stage 2
```

**Skip Checkpoint 1 nếu:** mode = Quick (Quick fast-path, không có CP, output luôn).

## 7. Checkpoint 2 — Output draft review (optional)

**Mặc định:** Skip cho Quick. Optional cho Standard. Có thể có cho Deep nếu user request.

**Output Checkpoint 2:**

Render draft báo cáo đầy đủ → user review → user (a) Approve / (b) Edit / (c) Reject.

Wording:

```
Em đã compose draft báo cáo đầy đủ ở phần dưới. Anh:
(a) Approve — em finalize + export
(b) Edit: [chỉ ra phần cần sửa]
(c) Reject — quay lại Stage 2 với hướng khác
```

## 8. Bear case strict reject rule

Pack áp dụng nguyên tắc: **khi mã có dòng tiền đang mạnh nhưng catalyst ngành/mã đã chuyển tiêu cực (tin xấu chính thức, chính sách siết chặt, hàng hoá đầu vào tăng cấu trúc) → dòng tiền đang priced-in (tin đã phản ánh vào giá) thứ gì đó chưa lộ, hoặc là late money (dòng tiền vào muộn, thường là retail đuổi theo khi đà tăng đã gần hết) sẽ kẹt. Đây là tình huống bẫy retail phổ biến nhất ở thị trường VN. Quy tắc: loại thẳng, không override bằng lý do "dòng tiền mạnh hẳn biết điều gì đó tốt".**

**Cụ thể trong pack: pattern "dòng tiền dương + catalyst tiêu cực" → strict reject Long.**

- Cụ thể: Stage 1c week_score > 0 + Stage 1g/1h có catalyst tiêu cực material (HIGH impact nhưng dịch sang "tác động mạnh")
- → Auto downgrade Long thành Watch (không cho phép Long với pattern này)
- → Note rõ trong báo cáo: "Pattern dòng tiền dương + catalyst tiêu cực — base rate retail trap kinh điển VN — strict reject Long"

Đây là design decision có chủ đích (xem README.md mục 8.4 Conviction memo). Không flex+downgrade — strict reject.

## 9. K hygiene checklist trước khi render output (lần nữa)

Reference `K_agent_db_00` mục 5. Trước khi finalize, agent self-check:

- [ ] Đã dịch DB raw (`vsi`, `day_score`, `week_score`, `zone`, percentile 0-100) sang ngôn ngữ tự nhiên
- [ ] Đã dịch taxonomy nội bộ (Kịch bản A/B/C/E1/E2/E3, framework chấm điểm, HIGH/MID/LOW impact) sang mô tả trực tiếp
- [ ] Đã dịch thuật ngữ EN (mean-reversion, exhaustion, etc.) sang VN nếu audience là KH
- [ ] Đã quy đổi đơn vị BCTC (sang tỷ đồng), ratios (sang %)
- [ ] Citation 4 nhóm đầy đủ cho mọi claim định lượng
- [ ] Wording observation, không command
- [ ] Disclaimer cuối báo cáo (xem `O_stock_report_00`)
- [ ] Audit trail metadata cuối báo cáo (Deep mode đầy đủ, Standard mode rút gọn, Quick mode optional)

## 10. Output naming convention

File output naming (theo `P_stock_report_04` mục 4.6 + `O_stock_report_00`):

```
stock_report_<TICKER>_<YYYYMMDD>_<mode>.md

Vd:
stock_report_VNM_20260530_standard.md
stock_report_HPG_20260530_deep.md
stock_report_VCBvsACB_20260530_pair_standard.md   (pair mode)
```

Suffix `_pair_*` cho pair mode để tránh nhầm với standalone single-stock.

## 11. Decision flow Stage 2 tổng

```
Stage 1 done (data đủ)
   ↓
Stage 2 Bước 1: Synthesis findings
   ↓
Stage 2 Bước 2: Conviction + Recommendation determination
   ├── Apply Variant Perception rule (mục 4)
   ├── Apply bear case strict reject rule nếu pattern (mục 8)
   └── Apply flex+downgrade nếu data thiếu (xem _00 mục 13)
   ↓
Stage 2 Bước 3: Compose 6-7 phần draft (theo template mục 2)
   ├── Pair mode? → activate Section 5 structure
   └── Single mode → standard structure
   ↓
Stage 2 Bước 4: Self-audit (xem _04 mục 1)
   ↓
CHECKPOINT 1: Thesis core review (mục 6)
   ├── (a) Confirm → Bước 5
   ├── (b) Refine → quay lại Bước 1 với input mới
   └── (c) Reject → quay lại Stage 2 từ đầu
   ↓
Stage 2 Bước 5: Finalize 6-7 phần đầy đủ
   ↓
CHECKPOINT 2 (optional Standard, Deep nếu request): Output review
   ├── (a) Approve → render + export
   ├── (b) Edit → fix specific section
   └── (c) Reject → quay lại Bước 3
   ↓
Stage 3 (ở O_stock_report): Render + audit trail metadata
```
