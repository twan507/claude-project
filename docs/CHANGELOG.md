# Changelog

Lịch sử thay đổi major cross-pack. Mỗi entry ghi: date + summary + files affected + rationale.

---

## 2026-05-30 (rev 5) — Refactor Value chain framework: tham chiếu chuyên nghiệp đầy đủ (Porter + Smile Curve + GVC + Industry 4.0 + CFA Sector Analysis 2020)

### Mở rộng value chain analysis với 6 framework chuẩn quốc tế

**Files affected:**
- `analysis_agent/P_stock_report_02.md` — **Refactor mục 2.6** từ 6 sub-mục → 10 sub-mục:
  - **NEW 2.6.0** Khung tham chiếu chuyên nghiệp (Porter, Smile Curve, GVC, Industry 4.0, CFA)
  - 2.6.1 industry value chain map (unchanged, minor enhancement)
  - **Enhanced 2.6.2** Porter Value Chain — thêm **4 support activities** (firm infrastructure, HR, technology development, procurement) + **forward/backward vertical integration distinction**
  - **Enhanced 2.6.3** thêm **GVC governance type** (market/modular/relational/captive/hierarchy — Gereffi 2005) + **Tier supplier position** (Tier 1/2/3) + switching costs depth
  - **NEW 2.6.4** Smile Curve / Vị trí capture giá trị (Stan Shih 1992) — ASCII diagram + 3 zones + VN context callout + ví dụ doanh nghiệp VN ở mỗi zone + migration strategy check
  - **NEW 2.6.5** Industry 4.0 / Digital footprint lens (CFA Sector Analysis 2020) — Three Golden Steps + 7-dimension digital readiness table + Leader/Par/Laggard verdict
  - **Enhanced 2.6.6** Position summary — tích hợp tất cả 6 lens (industry chain + Porter + Smile + GVC + Industry 4.0 + pricing power)
  - **Enhanced 2.6.7** Data sourcing — thêm R&D ratio + Industry 4.0 readiness + CFA chapter mapping reference
  - 2.6.8 Cross-link 4 kịch bản SXKD (enhanced với Smile + Industry 4.0 verdict)
  - **NEW 2.6.9** CFA Sector Analysis 2020 — Industry mapping table (18 VN whitelist + 3 financial + 2 override ↔ 21 CFA chapters)
  - Update template SXKD mục 7 — thêm Porter support activities + Smile Curve position + Industry 4.0 readiness + GVC governance render block
- `analysis_agent/P_stock_report_03.md` — Phần 2 sub-section 3 enhanced từ 4 sub-sub → 6 sub-sub (3a-3f) với Smile + Industry 4.0
- `analysis_agent/P_stock_report_04.md` — Section 1.3b value chain audit từ 5 điểm → **12 điểm**: Porter 4 (4 điểm) + Competitive 3 (3 điểm) + Smile/Industry 4.0 (2 điểm) + Synthesis 1 (1 điểm) + Professional standards check (2 điểm). Total SXKD self-audit: 40 → 47 điểm
- `analysis_agent/O_stock_report_00.md` — Render template enhanced với Porter support activities + Smile Curve section + Industry 4.0 readiness 7-row table + GVC governance + Tier position. K hygiene rule expanded — dịch framework jargon sang tiếng Việt tự nhiên cho audience KH (BỎ tên academic Porter/Stan Shih/Gereffi/CFA, chỉ giữ concept).
- `analysis_agent/K_sector_framework.md` — Điểm C mục 7.4 enhanced — list 6 framework chuẩn + 3 ví dụ industry value chain map (thép, F&B, dệt may) với CFA chapter mapping + Smile Curve position note

**Rationale:**

User feedback yêu cầu value chain phải tham chiếu **CFA Sector Analysis 2020** (200 trang, 21 industry chapters) + tiêu chuẩn chuyên nghiệp trên internet. Audit phát hiện rev 4 còn thiếu 5 elements quan trọng:

1. **Industry 4.0 / Digital footprint** — THE central concept của CFA Sector Analysis 2020 Prelude ("Business Models Under the Onslaught of Industrial Revolution 4.0"). Pack rev 4 0 reference.
2. **Smile Curve (Stan Shih 1992)** — phân bổ value-add dọc chuỗi (R&D + brand + service capture cao, manufacturing pure thấp). **Cực kỳ quan trọng cho VN context** vì hầu hết SXKD VN vẫn ở "smile bottom" (CMT/OEM/EMS). Rev 4 không có.
3. **Porter Support Activities (4)** — Rev 4 chỉ có 5 primary activities. Thiếu firm infrastructure / HR management / Technology development / Procurement → không complete Porter Value Chain.
4. **GVC governance type (Gereffi 2005)** — Market/Modular/Relational/Captive/Hierarchy. Rev 4 có vertical integration 4 mức nhưng thiếu directional analysis (forward vs backward) và GVC governance.
5. **CFA Sector Analysis chapter mapping** — Rev 4 reference chung chung K_sector_framework. Rev 5 add mapping table cụ thể 18 ngành VN whitelist ↔ 21 CFA chapters để pull additional questions specific.
6. **Tier suppliers (Tier 1/2/3)** — Modern global supply chain analysis. Rev 4 chỉ có top KH/NCC % không phân Tier.

**Key concepts thêm:**

- **Three Golden Steps** (CFA 2020 Prelude): tokenization → multidimensional interaction → embedded feedback mechanism
- **4 management DNA elements** (CFA 2020): sustainable customer interaction, decentralisation + humanistic outreach, brand togetherness, organic gel of prototyping-research-design (do-and-plan approach)
- **Smile Curve 3 zones** (Stan Shih 1992): upstream R&D top + midstream manufacturing bottom + downstream brand/service top
- **VN context callout**: 60-70% SXKD VN niêm yết ở smile bottom (manufacturing pure / CMT / OEM / EMS / raw export) — structural challenge cần leo smile để escape Value Trap
- **GVC governance 5 mô hình** với VN ví dụ: market (commodity), modular (FPT services), relational (rare in VN), captive (DETMAY CMT), hierarchy (VNM full integration)
- **Tier supplier position** với VN ví dụ: FPT Tier 1 modular, Garmex Tier 2 captive, nhiều electronics EMS Tier 2-3 captive
- **Industry 4.0 maturity 7 chiều**: digital footprint generation, production automation, mass customisation, IoT, AI/ML, business model (B2C/C2B), feedback loop speed
- **CFA chapter mapping** 21 ngành: từ CONGNGHE (chapter 1 Mobile Gaming + 2 Cashless + 5 Semiconductors) đến TIENICH (chapter 21 Utilities), với khía cạnh CFA đặc biệt useful cho value chain

**Tradeoff:**
- Phần 2 báo cáo dài thêm 0.5-1 trang cho Standard mode (1.5-2.5 → 2-3 trang) và Deep mode (2.5-3.5 → 3-4 trang) vì 6-row sub-structure
- Self-audit nặng hơn (40 → 47 điểm cho SXKD) — risk agent vướng audit nhiều hơn → nhưng đảm bảo professional rigor
- K hygiene complexity tăng — phải dịch nhiều framework jargon sang tiếng Việt cho KH audience (Porter, Smile Curve, GVC, Industry 4.0)
- Framework professional reference giúp output có credibility cao hơn khi internal review với analyst senior + giúp KH (đặc biệt KH đầu tư chuyên nghiệp) dễ hiểu basis của phân tích

**Reference frameworks (academic + professional):**
- Porter, M.E. (1985) — "Competitive Advantage: Creating and Sustaining Superior Performance" (Value Chain)
- Porter, M.E. (1979) — "How Competitive Forces Shape Strategy" (5 Forces)
- Shih, S. (1992) — Acer Smile Curve concept
- Gereffi, G., Humphrey, J., Sturgeon, T. (2005) — "The Governance of Global Value Chains"
- CFA Institute & ACCA (2020) — "Sector Analysis: A Framework for Investors" (200 pages, 21 industry chapters)

---

## 2026-05-30 (rev 4) — Bổ sung Value chain analysis cho P_stock_report (SXKD lens)

### Mở rộng P_stock_report cho phân tích chuỗi giá trị ngành + doanh nghiệp

**Files affected:**
- `analysis_agent/P_stock_report_00.md` — manifest update (`_01` 14 sub-step → 16; `_02` thêm mục 2.6; `_03` thêm sub-section value chain)
- `analysis_agent/P_stock_report_01.md` — thêm sub-step **1p "Khách hàng / Nhà cung cấp / Channel mix"** (SXKD mandatory Standard+); update coverage matrix Quick/Standard/Deep + fail-soft rule + audit trail
- `analysis_agent/P_stock_report_02.md` — thêm **mục 2.6 "Chuỗi giá trị (Value chain analysis)"** cho SXKD: 2.6.1 industry value chain map + 2.6.2 firm value chain 5 primary activities + 2.6.3 bargaining power 5 forces matrix + 2.6.4 position summary + 2.6.5 data sourcing + 2.6.6 cross-link 4 kịch bản. Update template SXKD mục 7 thêm 3 block render.
- `analysis_agent/P_stock_report_03.md` — Phần 2 mandatory sub-section thêm **(3) "Vị trí chuỗi giá trị"** với 4 sub-sub (3a industry map, 3b firm 5 activities, 3c 5 forces, 3d position summary) — SXKD Standard+ mandatory, NH/CK/BH skip. Update length matrix.
- `analysis_agent/P_stock_report_04.md` — thêm section **1.3b Value chain audit (5 điểm SXKD only)**, update total self-audit (SXKD 40 điểm, NH/CK/BH 35 điểm)
- `analysis_agent/O_stock_report_00.md` — render template thêm section "Vị trí chuỗi giá trị" với 4 sub-heading + K hygiene rule cho audience KH
- `analysis_agent/K_sector_framework.md` — bidirectional reference: mục 7.4 thêm **Điểm C "Phần 2 Chuỗi giá trị"** với ví dụ industry value chain map cho 3 ngành (thép, F&B, dệt may)

**Rationale:** Audit trail phát hiện gap: pack P_stock_report là deep-dive single-stock, nhưng cho SXKD type thiếu explicit framework phân tích chuỗi giá trị (cả industry value chain map lẫn firm value chain). Trước đó chỉ có:
- KPIs working capital cycle (DSO/DIO/DPO) — implies value chain nhưng không explicit
- "Pricing power: strong/moderate/weak" 1 dòng trong template — không có justification framework
- "Customer/supplier concentration" chỉ xuất hiện trong bear case

Trong khi đó `P_invest_memo_07` Phần 3 đã có "Value chain position + Key customers/suppliers" và render trong `O_invest_memo_02` mục 3.3 + 3.4 → P_stock_report phải đầy đủ hơn, không kém hơn invest_memo.

**Key concepts thêm:**
- **Industry value chain map** — upstream → midstream → downstream + margin pool theo mắt xích + bottleneck/choke point (ví dụ thép, dệt may, F&B, bán lẻ, BĐS dân cư, BĐS KCN trong mục 2.6.1)
- **Firm value chain (Porter 5 primary activities)** — inbound (top NCC + % giá vốn), operations (capacity, utilization, cost curve), outbound (kênh phân phối), marketing/sales (top KH, B2B/B2C/Export mix), service (after-sales)
- **Vertical integration assessment** — 4 mức (pure-play / 2 mắt xích / full integration / conglomerate) với ưu/nhược
- **5 forces matrix** — bargaining power suppliers + buyers, threat of substitute + new entrants, industry rivalry (Cao/Trung/Thấp với 1 dòng justify)
- **Position summary** — firm captures margin ở đâu + expose risk ở đâu + pricing power verdict (justified)
- **Data sub-step 1p** — pull top KH/NCC từ thuyết minh BCTC + BCTN + web search; fail-soft note nếu data gap

**Tradeoff:** Output Phần 2 dài thêm 0.5-1 trang cho SXKD (Standard 1.5-2.5 trang; Deep 2.5-3.5 trang) → output Standard có thể vượt 5 trang, cần monitor. NH/CK/BH skip 1p và Phần 2 sub-section value chain (vì 1i type-specific FA đã cover tương đương).

---

## 2026-05-30 (rev 3) — Thêm pack P_stock_report + O_stock_report (single-stock deep analysis)

### Pack mới `P_stock_report` (5 file) + `O_stock_report` (1 file)

**Files affected:**
- `analysis_agent/P_stock_report_00.md` (251 dòng) — Master
- `analysis_agent/P_stock_report_01.md` (628 dòng) — Pre-flight + Stage 1 Data Acquisition (14+ sub-step) + Type classification
- `analysis_agent/P_stock_report_02.md` (548 dòng) — Type-specific framework cho 4 type SXKD/NH/CK/BH
- `analysis_agent/P_stock_report_03.md` (540 dòng) — Stage 2 Compose + Output structure 6-7 phần + 3 depth mode + VP rule + Pair compare
- `analysis_agent/P_stock_report_04.md` (461 dòng) — Self-audit checklist + Edge cases + 10 failure modes + Output contract
- `analysis_agent/O_stock_report_00.md` (645 dòng) — Render spec 6-7 phần rigid + 3 depth mode + audience flex
- `analysis_agent/KERNEL_SKELETON.md` — register pack mới
- `docs/PROJECT_STATUS.md`, `docs/PACK_CATALOG.md`, `docs/CHANGELOG.md` (file này)

**Rationale:** Project có gap về **single-stock deep analysis standalone** — `P_invest_memo` Tier 5C bắt buộc đã qua Tier 0-3 (full portfolio cycle), nhưng nhiều use case khác cần phân tích sâu 1 mã: KH hỏi nhanh, pre-screening trước memo cycle, pitch ad-hoc, so sánh 2-3 mã trước khi pick. Pack `P_stock_report` complement (không thay thế) Tier 5C, cover gap này.

**Key features pack mới:**

1. **Stage 1 Data Acquisition 14+ sub-step** (chi tiết ở `_01`):
   - 1a Stock info + type classification (SXKD/NH/CK/BH)
   - 1b FA data DB (BCTC quarterly 8-12Q + annual 3-5Y + valuation)
   - 1c Dòng tiền + technical zone snapshot
   - 1d Khối ngoại + tự doanh net position
   - 1e Major shareholders + ownership structure
   - 1f Corporate actions recent (rolling 12 tháng)
   - 1g News DB (rolling 30-90 ngày)
   - 1h **Web search news** (VN equity + EN macro cho ngành tài chính/dầu khí/kim loại)
   - 1i **BCTC PDF forensic 15-point checklist** đào sâu thuyết minh
   - 1j Sector context (pull `K_sector_framework`)
   - 1k Macro relevant
   - 1l **Peer compare internet-first** + thanh khoản filter
   - 1m ADV liquidity tier
   - 1n Earnings calendar
   - 1o ESG controversy scan

2. **4 type framework** (chi tiết ở `_02`):
   - SXKD: 4 kịch bản (Value Play / Value Trap / Growth at Premium / Cycle Top) + 3 sub-type cycle dynamics
   - NH: NIM drivers + asset quality + capital
   - CK: brokerage share + margin book + IB + prop book
   - BH: combined ratio + APE + persistency + embedded value

3. **3 depth mode:**
   - Quick 1-2 trang: 11/15 sub-step, VP optional
   - Standard 3-5 trang: full 15 sub-step, peer 3 mã, VP recommended (không có → flag)
   - Deep 5-10 trang: full + ESG kỹ + peer 5 mã + macro sens + data appendix, **VP mandatory** (không có → auto downgrade conviction)

4. **6-7 phần output structure rigid:**
   - 1 Khuyến nghị / 2 Doanh nghiệp / 3 Bối cảnh ngành & vĩ mô (skip Quick) / 4 Tài chính & định giá / 5 Tin tức & Catalyst / 6 Bear case & Disconfirming / 7 Exit triggers (chỉ Long) / Phụ lục data sources (Deep)

5. **Audience flex (nội bộ / KH):**
   - Wording + K hygiene khác nhau
   - KH KHÔNG nhận TP1/TP2/SL số cụ thể (chỉ "Tín hiệu cần theo dõi để xem xét lại")
   - Forward-looking statement bắt buộc cho Deep + KH

6. **Pair compare mode** support 2-3 mã cùng ngành / cùng theme / cùng value chain (apple-to-orange REFUSE)

7. **Variant Perception** discipline:
   - Concept từ Steinhardt/Cohen institutional buy-side
   - Pack apply rule per depth mode (Quick optional / Standard recommended / Deep mandatory)
   - Không có VP ở Deep → auto downgrade conviction HIGH→MID / MID→LOW / LOW→Watch

8. **Key constraints:**
   - **BCTC PDF mandatory** — không upload thì REFUSE chạy (gate strict tuyệt đối)
   - **Long-only** (Long / Watch / Avoid)
   - **Web search VN cho equity, EN cho macro** chỉ với ngành liên quan tài chính/commodity
   - **Peer internet-first** + filter ADV ≥ 30 tỷ/ngày + market cap top 50, exclude small cap unknown
   - **Pattern strict reject Long:** dòng tiền dương + catalyst tiêu cực material → auto downgrade Watch
   - **Conviction CAP** at LOW cho penny stock, at MID cho newly listed
   - **Bear case mandatory** cho mọi recommendation
   - **Disconfirming signal measurable** với threshold cụ thể

9. **Self-audit 35 điểm** + **10 failure modes** + edge cases (conglomerate / holding / newly listed / suspended / penny / mã ngoài whitelist / hệ tập đoàn / ETF / multi-listing / delisting)

**Quan hệ với pack khác:**
- Complement `P_invest_memo` Tier 5C (không auto-escalate)
- Reference `K_sector_framework` cho Phần 3 industry context
- Reference `K_agent_db_04` cho 4 type methodology gốc
- Reference `K_agent_db_00/01/02` cho data acquisition + K hygiene

**Use case điển hình:**
- "Phân tích VNM" → Standard mode
- "Brief VNM cho KH" → Standard mode + audience KH
- "Quick check VNM" → Quick mode
- "Đánh giá VNM trước khi vào Tier 5C" → Deep mode (pre-Tier 5C)
- "So sánh VNM vs MSN" → Pair Standard mode

---

## 2026-05-30 (rev 2) — Xoá template_agent + thêm K_sector_framework

### template_agent — XOÁ HOÀN TOÀN

**Files affected:** Toàn bộ folder `template_agent/` (system_prompt.md, INDEX.md, FORMAT.md, WORKFLOW.md, TEMPLATE_VBSE.md, TEMPLATE_FINEXT.md, 2 file `.pptx` catalog)

**Rationale:** Render binary (pptx/docx branded) là concern downstream tách biệt với analysis quality. MD final từ `analysis_agent` đã đủ structured (heading hierarchy + chart annotation YAML + citation + locale) để tool render bên ngoài consume. Việc duy trì 1 agent dedicated cho rendering làm tăng maintenance burden mà không đem lại giá trị tương xứng — pptx layout/brand catalog cũng có thể outsource sang Canva/PowerPoint template thông thường.

**Tác động cross-doc:** README.md, docs/PROJECT_STATUS.md, docs/PACK_CATALOG.md đã được clean reference. Số agent từ 3 → **2** (analysis_agent + db_agent).

**Pack catalog cũ (archive cho tra cứu sau):**
- `system_prompt.md` — Meta-rules agent (paste vào Custom Instructions)
- `INDEX.md` — Manifest + workflow tổng quan
- `FORMAT.md` — Spec MD chuẩn hoá 9 report_types
- `WORKFLOW.md` — Flow 7 stage + 3 checkpoint
- `TEMPLATE_VBSE.md` + `.pptx` — Catalog 27 layout brand VBSE
- `TEMPLATE_FINEXT.md` + `.pptx` — Catalog 27 layout brand Finext
- 9 report_type: stock_pitch, weekly_market, market_scan, stock_memo, portfolio_plan, portfolio_review_weekly/monthly/quarterly, custom
- 7 stage workflow: Ingest → Parse → Clarify (CP1) → Normalize → MD review (CP2) → Brand pre-flight (CP3) → Render
- Brand whitelist strict: chỉ VBSE và Finext

### K_sector_framework — pack K mới

**Files affected:** `analysis_agent/K_sector_framework.md` (1 file), `analysis_agent/KERNEL_SKELETON.md` (thêm entry), pointer trong `P_invest_memo_05/07` + `P_vbse_strategy_04`.

**Rationale:** Bổ sung knowledge layer chuyên cho **industry structure + competitive dynamics + ESG** chuẩn institutional buy-side, chắt lọc từ CFA Sector Analysis Framework (2020). Pack bổ trợ (không thay thế) `K_agent_db_04` — `K_agent_db_04` chuyên dòng tiền/PTCB/technical từ data DB, còn `K_sector_framework` chuyên industry-level lens (DD/MP/SI/PM/ESG).

**Cấu trúc:**
1. Universal 5-dimension framework: Demand Drivers / Market Position / Structural Influences / Performance Metrics / ESG (~10-15 câu/dimension)
2. Per-sector quick-reference cho 10-12 ngành whitelist có CFA cover direct (NGANHANG, TIENICH, BDS, KCN, BANLE, VANTAI, CONGNGHE, XAYDUNG, THUCPHAM, NONGNGHIEP, CHUNGKHOAN, BAOHIEM override)
3. Guidance generic cho 6-7 ngành whitelist không có CFA cover (DAUKHI, HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, THUYSAN, CONGNGHIEP)
4. Industry 4.0 lens — digital footprint, automation, AI/IoT disruption cross-sector

**Trigger:** P pack tham chiếu khi cần deep-dive sector-level analysis trong các điểm cụ thể:
- `P_invest_memo_05/06/07` (Tier 5A/B/C memo deep-dive) — phần "Business" của memo 7 phần
- `P_vbse_strategy_04` (Trục 4 Sector allocation) — per-sector analytical lens
- `P_weekly_overview_02` (Phần 6 Biến động 18 ngành) — structural watch khi có chuyển động bất thường

---

## 2026-05-30 — Reform institutional buy-side + schema update

### Schema update (cả 2 agent)

**Files:** `K_agent_db_00/01/02/04` (analysis_agent) + `agent_db_00/01/02/04` (db_agent) + downstream pack files

**Thay đổi:**
- 23 → **25 collection**
- Thêm khối **History** (3 collection): `history_index`, `history_industry`, `history_stock` — full historical OHLCV+pct_change cho on-demand chart dài hạn
- **Bỏ `stock_highlight`** (top tăng/giảm pre-compute) — thay bằng aggregate from `stock_snapshot` filter `industry`
- **Bỏ `industry_rank` static** trong `industry_snapshot.money_flow_score` và `industry_recent.series.money_flow_score` — rank ngành phải **tự tổng hợp** theo `week_score` (dòng tiền tuần) trong scope phân tích (default: 18 whitelist; override: tuỳ user)

**Rationale:** rank ngành phụ thuộc scope analysis. Nếu DB lưu rank tĩnh 1..24, mọi báo cáo phải re-compute lại để khớp scope (18 whitelist vs override custom) — rủi ro sai. Tự tổng hợp đảm bảo rank luôn align với danh sách theo dõi.

### Whitelist 18 ngành — Default + Override mode

**Files:** `K_agent_db_00 mục 4.5` + `K_agent_db_01 Section B callout` + `agent_db_00 mục 4.5` + `agent_db_01 Section B callout`

**Thay đổi:**
- **Default mode:** mọi query/aggregate/ranking/báo cáo cấp ngành filter 18 whitelist
- **Override mode:** user yêu cầu cụ thể ngành ngoài whitelist (vd "phân tích ngành Bảo hiểm") → agent vẫn query và trả lời bình thường, kèm note "ngoài scope whitelist mặc định"

**18 ngành whitelist:**
BANLE, BDS, CHUNGKHOAN, CONGNGHE, CONGNGHIEP, DAUKHI, DETMAY, HOACHAT, KCN, KHOANGSAN, KIMLOAI, NGANHANG, NONGNGHIEP, THUCPHAM, THUYSAN, TIENICH, VANTAI, XAYDUNG.

Ngành ngoài whitelist (vẫn có trong DB 24 ngành): BAOHIEM, VLXD, NHUA, CAOSU, DULICH, YTEGD.

### Pack rename + refactor (analysis_agent)

#### `P_invest_strategy` → `P_vbse_strategy` (split 1 file 832 LOC → 10 files ~4760 LOC)

**Old:** `P_invest_strategy_00.md` + `O_invest_strategy_00.md` (xoá hoàn toàn)

**New:** `P_vbse_strategy_00..09` + `O_vbse_strategy_00`

**Philosophy mới:**
- **Fundamental supremacy:** PTKT không có vai trò quyết định trong bất kỳ trục nào (Trục 1-5 + Phase 1 Screen Trục 6)
- **PTKT vai trò duy nhất:** Phase 2 Bucket entry Trục 6 — phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản (theo `P_invest_memo_03` mục 5)
- Cap technical siết: Trục 5 = 0% (cấm tuyệt đối), Trục 4 ≤5%, Trục 3 ≤5%, Trục 2 ≤20%, báo cáo tổng ≤15% (trừ Phase 2 Bucket)
- Weekly **Technical-as-noise rule:** Shift bắt buộc kèm signal vĩ mô/cơ bản/chính sách. Technical shift đơn độc = noise tạm thời

#### `P_weekly_market` → `P_weekly_overview` (split 1 file 616 LOC → 5 files ~1860 LOC)

**Old:** `P_weekly_market_00.md` + `O_weekly_market_00.md` (xoá hoàn toàn)

**New:** `P_weekly_overview_00..04` + `O_weekly_overview_00`

**Philosophy mới:**
- **Fundamental-driven:** 3 kịch bản phần 9 trigger primary là vĩ mô/cơ bản/chính sách/catalyst, technical chỉ confirmation phụ ≤30%
- **Conviction + Horizon + Disconfirming** bắt buộc mỗi call (chuẩn institutional)
- **Executive summary Key calls / Watch / Risk** (thay format bullet rời cũ)
- **Review W-1 thành 3 scorecard tables**
- **Vĩ mô-ngành table 5 cột** (Magnitude + Persistence — chuẩn institutional macro impact)
- Bảng 18 ngành thêm cột P/E phân vị 3Y + EPS Q YoY + earnings beat candidate section
- Cảnh báo trap setup (mã top dòng tiền thuộc ngành thận trọng)
- Watchlist tách 2 hướng (cơ hội tăng + cảnh báo áp lực) + Bucket entry OPTIONAL

#### `P_stock_pitch` + `O_stock_pitch` — XOÁ

**Rationale:** vai trò pitch mã đơn lẻ gửi KH đã được cover bởi:
- `P_vbse_strategy` Trục 6 watchlist (theme play + bucket entry observation)
- `P_invest_memo` Tier 5C memo (deep-dive nội bộ)

Pack dedicated cho pitch KH có thể tái sinh sau nếu có nhu cầu rõ.

### Cross-cutting updates

**Files affected:**
- `analysis_agent/KERNEL_SKELETON.md` — refactor toàn bộ section P + O (P_invest_strategy → P_vbse_strategy, P_weekly_market → P_weekly_overview, xoá P_stock_pitch + O_stock_pitch + O_invest_strategy + O_weekly_market blocks)
- `analysis_agent/system_prompt.md` — update O pack examples (line 60)
- `db_agent/system_prompt.md` — schema manifest 25 collection
- `README.md` — pack list + workflow + naming + constraint cuối
- `docs/PROJECT_STATUS.md` (new)
- `docs/PACK_CATALOG.md` (new)
- `docs/CHANGELOG.md` (this file)

---

## Pre-2026-05 — Initial state (baseline)

**Packs active:**
- `K_agent_db` (analysis_agent) + `agent_db` (db_agent) — 23 collection schema
- `P_invest_memo` (10 files) — workflow đầu tư cá nhân
- `P_weekly_market` (1 file) — báo cáo thị trường tuần 12 phần
- `P_stock_pitch` (1 file) — pitch mã đơn lẻ gửi KH
- `P_invest_strategy` (1 file 832 LOC) — chiến lược đầu tư VN 2 mode

**Architecture stable:** K/P/O layered cho analysis_agent, monolithic cho db_agent.

**Outstanding issues identified before reform:**
- `P_invest_strategy` quá dài 832 LOC trong 1 file, methodology lẫn workflow lẫn output contract
- `P_weekly_market` technical-driven trigger 3 kịch bản phần 9 — không nhất quán với institutional buy-side standard
- Không có whitelist ngành rule — agent có thể aggregate trên ngành ngoài focus
- `industry_rank` static gây confusion khi scope thay đổi (18 vs 24)
