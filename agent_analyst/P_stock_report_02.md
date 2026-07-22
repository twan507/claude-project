# P_stock_report_02 — Type-specific framework (4 type SXKD/NH/CK/BH)

File này quy định **lens phân tích chi tiết** theo type doanh nghiệp đã classify ở Stage 1a (`_01` mục 3.1). Mỗi type có set KPIs + analytical framework riêng. Reference `K_agent_db_04` (methodology DB) + `K_sector_framework` (industry-level lens).

## 1. Triết lý chung — Tại sao 4 type khác lens

Mỗi type doanh nghiệp có:
- **Cấu trúc bảng cân đối khác nhau** (NH dominated by loan + deposit; CK dominated by margin loan + customer deposit; BH dominated by reserve + investment; SXKD dominated by working capital + fixed asset)
- **Cấu trúc kết quả kinh doanh khác nhau** (NH = NII + NoII − OpEx − Cost of credit; CK = brokerage commission + margin yield + IB fee + trading; BH = premium earned − claims − reserve change + investment income; SXKD = revenue × margin × asset turnover)
- **Cycle dynamics khác nhau** (NH chu kỳ tín dụng + lãi suất; CK chu kỳ TT; BH chu kỳ lãi suất dài + chu kỳ thiên tai; SXKD chu kỳ commodity + business)
- **Regulatory exposure khác nhau** (NH/CK/BH heavy regulated; SXKD lighter)

→ Dùng 1 lens chung sẽ miss insight quan trọng. 4 type tương ứng cho 4 lens chuyên biệt.

**Reference base:** `K_agent_db_04` đã có methodology PTCB cho 4 type. Pack này tổ chức lại + đào sâu cho mục đích single-stock deep-dive.

## 2. Type SXKD — Sản xuất kinh doanh / General Industrial

### 2.1. Scope

Áp dụng cho 21 ngành whitelist trừ NGANHANG / CHUNGKHOAN / BAOHIEM. Bao gồm:
- Sản xuất công nghiệp (KIMLOAI, HOACHAT, DETMAY, KHOANGSAN, CONGNGHIEP, DAUKHI, TIENICH)
- Tiêu dùng (BANLE, THUCPHAM, THUYSAN, NONGNGHIEP)
- Hạ tầng / BĐS (XAYDUNG, BDS, KCN, VANTAI)
- Dịch vụ (CONGNGHE)

### 2.2. Analytical framework — 4 kịch bản (reference `K_agent_db_04`)

Đánh giá theo **2 trục:** (a) Định giá và (b) Triển vọng tăng trưởng / cycle position.

| Kịch bản | Định giá | Triển vọng | Đặc điểm | Recommendation default |
|---|---|---|---|---|
| **Value Play** | Phân vị < 40% (rẻ vs lịch sử) | Đang cải thiện / sắp inflection | Cycle bottom + business improvement | Long candidate strong |
| **Value Trap** | Phân vị < 40% (rẻ) | Suy thoái cấu trúc / không inflection | Rẻ vì lý do — earnings continued downgrade | Avoid (rẻ là bẫy) |
| **Growth at Premium** | Phân vị > 60% (đắt vs lịch sử) | Tăng trưởng cao bền vững | Premium justified by growth | Long với conviction phụ thuộc sustainable growth |
| **Cycle Top** | Phân vị > 70% (rất đắt) | Cycle peak / earnings unsustainable | Earnings sắp peak rồi giảm | Avoid / Watch (canh cycle xuống) |

**Logic phân loại:**

1. Bước 1: Pull định giá hiện tại + percentile 3-5Y (Stage 1b)
   - P/E hoặc P/B phù hợp với ngành (vd Banking dùng P/B; SXKD tăng trưởng cao dùng P/E; SXKD cyclical dùng P/B + EV/EBITDA)
2. Bước 2: Đánh giá triển vọng từ:
   - BCTC quarterly trend (Stage 1b) — earnings momentum
   - News + catalyst (Stage 1g + 1h) — forward visibility
   - Industry structural (Stage 1j) — DD/SI trends
   - Macro relevant (Stage 1k) — tailwind / headwind
   - BCTC forensic (Stage 1i) — quality of earnings, hidden flags
3. Bước 3: Map vào 4 kịch bản + recommendation default

### 2.3. KPIs chính theo Type SXKD

**Income statement quality:**
- Revenue growth YoY (organic vs inorganic split)
- Gross margin trend (pricing power signal)
- EBIT margin (operational leverage)
- Net margin (sau interest + tax)
- Non-recurring items split (one-off gains/losses)

**Balance sheet quality:**
- Net debt / EBITDA (leverage — flag nếu > 3x cho SXKD non-utility, > 5x cho utility/BĐS)
- Interest coverage (EBIT / Interest expense — flag nếu < 3x)
- Current ratio (short-term liquidity)
- Working capital cycle: receivable days + inventory days − payable days = cash conversion cycle

**Cash flow quality:**
- FCF margin (FCF / Revenue)
- FCF conversion (CFO / Net Income) — flag nếu < 70% sustained (earnings không convert thành cash)
- Capex / Revenue (capex intensity)
- Capex split: maintenance vs growth (từ thuyết minh BCTC)

**Return quality:**
- ROE TTM + trend 5Y
- ROIC TTM + trend 5Y (loại ảnh hưởng đòn bẩy)
- DuPont decomposition: ROE = Net margin × Asset turnover × Equity multiplier (xem `K_agent_db_04`)

**Cycle awareness:**
- Earnings volatility 5Y (std dev % EPS YoY)
- Margin range 5Y (peak to trough gross margin)
- Position trong cycle (peak / mid / trough) — phán đoán qua macro + commodity price

### 2.4. Sub-type cycle dynamics

Type SXKD chia tiếp 3 sub-type theo cycle nature:

**Sub-type SXKD-Cyclical (commodity-linked):**
- DAUKHI, KIMLOAI, HOACHAT, NONGNGHIEP, THUYSAN
- Earnings volatility cao theo giá commodity
- P/B + EV/EBITDA tốt hơn P/E (P/E lừa ở cycle peak / trough)
- Cycle position quan trọng nhất

**Sub-type SXKD-Consumer/Defensive:**
- THUCPHAM, BANLE, TIENICH
- Earnings ổn định hơn
- P/E benchmark tốt
- Demand resilience trong recession quan trọng
- Pricing power qua brand / scale

**Sub-type SXKD-Growth/Infrastructure:**
- CONGNGHE, BDS, KCN, XAYDUNG, VANTAI
- Earnings phụ thuộc capex cycle + project pipeline
- Order backlog + book-to-bill ratio quan trọng (XAYDUNG)
- Land bank + pre-sales velocity (BDS)
- Occupancy + WALE (KCN)
- Customer concentration + offshore contract (CONGNGHE)

Mỗi sub-type pull thêm KPI riêng theo mục 5.X tương ứng trong `K_sector_framework`.

### 2.5. Bear case typical cho Type SXKD

- **Pricing power erosion** (commodity downcycle, new entrant disrupt)
- **Capex overhang** (capex lớn nhưng ROIC thấp → value destruction)
- **Working capital deterioration** (receivable kéo dài, inventory obsolete)
- **Goodwill impairment** sau M&A lớn
- **Customer / supplier concentration** loss key account
- **Regulatory tightening** (tariff, environmental, antitrust)
- **Cycle top earnings unsustainable** (commodity rollover, EU/US recession demand cut)

### 2.6. Chuỗi giá trị (Value chain analysis) — MANDATORY cho SXKD

Đây là **lens phải có** cho SXKD vì 4 kịch bản (mục 2.2) và pricing power (mục 7) đều xuất phát từ vị trí chuỗi giá trị. Thiếu lens này → analysis bị treo lơ lửng, không trả lời được câu "vì sao biên lợi nhuận cao/thấp" và "cấu trúc lợi nhuận có bền không".

#### 2.6.0. Khung tham chiếu chuyên nghiệp (Professional reference frameworks)

Mục này tham chiếu các framework chuẩn quốc tế đã được xác thực qua academia + industry practice. Khi áp dụng, agent phải dùng đầy đủ — không bỏ sót khía cạnh nào:

1. **Porter Value Chain (Michael Porter, 1985)** — chuỗi giá trị doanh nghiệp gồm **5 primary activities** (inbound, operations, outbound, marketing, service) + **4 support activities** (firm infrastructure, HR, technology development, procurement). Áp dụng ở mục 2.6.2.
2. **Porter 5 Forces (Michael Porter, 1979)** — cấu trúc cạnh tranh ngành: supplier power, buyer power, threat of substitute, threat of new entrants, industry rivalry. Áp dụng ở mục 2.6.3.
3. **Smile Curve (Stan Shih / Acer, 1992)** — phân bổ giá trị dọc chuỗi: R&D + design (đầu chuỗi) và brand + service + after-sales (cuối chuỗi) capture margin cao nhất; manufacturing pure (giữa chuỗi) thấp nhất. **Đặc biệt quan trọng cho VN context** vì phần lớn doanh nghiệp SXKD VN vẫn ở "smile bottom" (CMT, OEM, EMS, gia công). Áp dụng ở mục 2.6.4.
4. **Global Value Chain (GVC) governance** (Gereffi, Humphrey, Sturgeon, 2005) — 5 mô hình quản trị chuỗi cung ứng toàn cầu: **market / modular / relational / captive / hierarchy**. Áp dụng ở mục 2.6.3.
5. **Industry 4.0 / Digital footprint** (CFA Sector Analysis 2020 Prelude — "Business Models Under the Onslaught of Industrial Revolution 4.0") — mọi doanh nghiệp hiện đại đều tạo digital footprint. Three Golden Steps: tokenization → multidimensional interaction → embedded feedback mechanism. Áp dụng ở mục 2.6.5.
6. **CFA Sector Analysis Framework (CFA Institute & ACCA, 2020)** — 21 industry chapters, mỗi chapter dùng 5-dimension model (DD/MP/SI/PM/ESG). Mapping ngành VN ↔ CFA chapter ở mục 2.6.9.

**VN context callout:** Khoảng 60-70% doanh nghiệp SXKD VN niêm yết vẫn ở **mắt xích manufacturing thấp value-add** (DETMAY CMT, electronics EMS/ODM cho global brand, NONGNGHIEP/THUYSAN raw export, KIMLOAI commodity). Smile Curve + Industry 4.0 lens giúp phân biệt firm nào đang **leo smile** (Long candidate) vs firm bị stuck ở smile bottom (Value Trap risk).

#### 2.6.1. Chuỗi giá trị ngành (Industry value chain map)

Vẽ chuỗi từ thượng nguồn → trung nguồn → hạ nguồn → người tiêu dùng cuối. Identify:

1. **Các mắt xích chính** trong ngành (3-5 stage tuỳ ngành):
   - Vd ngành **thép**: Quặng sắt + than cốc → Phôi/HRC → Tôn/Ống/Thanh → Phân phối B2B (xây dựng/cơ khí) → End user
   - Vd ngành **dệt may**: Bông/sợi → Vải → Cắt may CMT → FOB/ODM → Brand → Retailer → End user
   - Vd ngành **F&B đồ uống**: Nguyên liệu (đường, hương liệu, bao bì) → Sản xuất → Phân phối (NPP cấp 1/2) → Điểm bán (GT/MT/HORECA) → End user
   - Vd ngành **bán lẻ**: Brand owner / Nhà sản xuất → NPP/Importer → Retailer (chain/mom-pop) → End user
   - Vd ngành **BĐS dân cư**: Quỹ đất → Đầu tư hạ tầng + xin phép → Phát triển dự án → Phân phối (sàn F1/F2) → Khách hàng cuối
   - Vd ngành **BĐS KCN**: Đền bù đất → San lấp + hạ tầng → Cho thuê đất công nghiệp → Nhà xưởng/Tenant → End-user FDI

2. **Margin pool theo mắt xích** — phân bổ biên lợi nhuận:
   - Mắt xích nào hold biên gross > 30%? (thường có bargaining power mạnh)
   - Mắt xích nào commodity-like biên < 10%? (rủi ro cycle cao)
   - Mắt xích nào có growth driver structural? (digital, premium, export)

3. **Bottleneck / Choke point**:
   - Mắt xích nào có barriers to entry cao? (license, capital, technology, brand)
   - Mắt xích nào fragmented, dễ bị disrupted?
   - Vd thép: thượng nguồn quặng sắt VN thiếu → phụ thuộc nhập khẩu; hạ nguồn phân phối fragmented (nhiều cửa hàng vật liệu xây dựng)

4. **Cross-reference `K_sector_framework`** mục 5.X — pull industry-specific chuỗi giá trị nếu có.

**Output expected mục 2.6.1:** 1 đoạn 3-6 dòng + (optional Standard, mandatory Deep) sơ đồ ASCII đơn giản chuỗi giá trị ngành với margin pool note ở mỗi mắt xích.

#### 2.6.2. Chuỗi giá trị doanh nghiệp (Porter Value Chain — 5 primary + 4 support activities)

Identify **doanh nghiệp đang đứng ở mắt xích nào** + **mức độ + hướng tích hợp dọc** (vertical integration).

**A. 5 PRIMARY ACTIVITIES (Porter 1985):**

1. **Inbound logistics — Nguồn nguyên liệu đầu vào:**
   - Top 3-5 nhà cung cấp + % cost of goods (pull từ thuyết minh BCTC `_01` 1i + 1p)
   - Nguyên liệu key + % giá vốn (vd thép HRC chiếm 60% giá vốn tôn mạ)
   - Mức độ phụ thuộc import vs domestic (FX exposure)
   - **Tier supplier position của firm trong GVC** (xem mục 2.6.3): firm là Tier 1 / Tier 2 / Tier 3 của ai? Có sole-source nguyên liệu nào (single-point-of-failure)? Geographic concentration risk (vd 80% từ 1 quốc gia)?
   - Switching cost với nhà cung cấp (cao = bị supplier bargain; thấp = firm có thể bargain)

2. **Operations — Sản xuất / Vận hành:**
   - Quy mô công suất + tỷ lệ vận hành (utilization rate)
   - Vị trí trên cost curve (cost leader / mid-cost / high-cost)
   - Technology / process moat (vd HPG lò cao hiện đại; VNM dây chuyền UHT)
   - Geographic footprint (1 nhà máy concentration risk vs multi-plant)
   - **Industry 4.0 readiness** (cross-link mục 2.6.5): tự động hoá, robot count, IoT sensors, AI/ML deployment

3. **Outbound logistics — Phân phối:**
   - Kênh phân phối owned vs outsourced (vd VNM 250.000 điểm bán direct; PNJ 400+ store sở hữu)
   - Logistics integrated (vd MWG self-logistics) vs 3PL
   - Geographic coverage VN (Bắc/Trung/Nam %) + Export %

4. **Marketing & Sales:**
   - Brand strength + market share theo segment
   - Channel mix: B2B vs B2C, GT (general trade) vs MT (modern trade) vs Online vs HORECA vs Export
   - Top 5-10 khách hàng + % doanh thu (pull từ thuyết minh BCTC `_01` 1i + 1p)
   - Pricing power evidence: gross margin trend stable/expanding khi commodity input volatile?
   - Customer concentration verdict: top 5 KH > 50% = concentration cao (buyer power risk)

5. **Service — After-sales / CSKH:**
   - (Áp dụng cho durables, retail, B2B equipment): warranty, technical support, parts availability
   - Recurring revenue (subscription, maintenance contract) — quan trọng cho re-rating
   - Service quality + Net Promoter Score-equivalent nếu có

**B. 4 SUPPORT ACTIVITIES (Porter 1985) — thường bị bỏ sót nhưng quyết định moat dài hạn:**

6. **Firm infrastructure:**
   - Cơ cấu quản trị: HĐQT độc lập members, audit committee, transparent disclosure (review từ Stage 1e + 1i)
   - Hệ thống quản lý: ERP, governance framework, risk management
   - Quan hệ stakeholder: cổ đông lớn, ngân hàng tài trợ, regulator (review từ 1o ESG)

7. **HR management:**
   - Thu hút + giữ nhân tài: turnover rate, chính sách ESOP/RSU (review từ 1f Corporate actions)
   - Đào tạo + năng lực kỹ thuật (đặc biệt quan trọng ngành tech-heavy: CONGNGHE, KIMLOAI lò cao, DAUKHI E&P)
   - Văn hóa doanh nghiệp + leadership pipeline (succession planning)

8. **Technology development:**
   - **R&D / Revenue ratio** (vd VNM ~0.5%, FPT ~5-7%, traditional VN manufacturing thường < 1% — quá thấp = không leo smile được)
   - Patent portfolio (nếu có) — đặc biệt cho HOACHAT, CONGNGHE, DAUKHI
   - Digital transformation roadmap (cross-link Industry 4.0 — mục 2.6.5)
   - Innovation pipeline + new product launches frequency

9. **Procurement (strategic sourcing):**
   - Sourcing strategy: single-source vs multi-source, just-in-time vs strategic stockpile
   - Supplier base diversity + power balance
   - Long-term contracts vs spot market mix (mức độ chốt giá đầu vào)
   - Strategic raw material hedging (vd HPG hedge quặng sắt + than cốc; PNJ hedge vàng SJC; thực phẩm hedge đường + sữa nguyên liệu)

**C. VERTICAL INTEGRATION — hướng + mức độ:**

Phân biệt **2 hướng tích hợp** (cực kỳ quan trọng cho phân tích moat):

- **Backward integration (tích hợp về thượng nguồn):** firm sở hữu/control supply
  - Vd: **HPG** sở hữu mỏ quặng + cảng (backward) → giảm dependency supplier + đảm bảo cost
  - Vd: **PNJ** sở hữu nhà máy tinh luyện vàng (backward) → control chất lượng + chi phí
  - Vd: **VNM** sở hữu đàn bò sữa (backward) → đảm bảo nguồn nguyên liệu organic
  - **Implication:** giảm supplier power, bảo vệ margin khi input price volatile

- **Forward integration (tích hợp về hạ nguồn):** firm sở hữu/control distribution / customer
  - Vd: **VNM** sở hữu 250.000 điểm bán direct + chuỗi Giấc Mơ Sữa Việt (forward) → control margin + customer data
  - Vd: **HPG** có chuỗi phân phối thép cuộn + tôn (forward)
  - Vd: **PNJ** 400+ cửa hàng retail (forward)
  - Vd: **MWG** retail chain (vốn dĩ là forward integration của brand owners qua MWG)
  - **Implication:** capture downstream margin + first-party customer data (enabler Industry 4.0)

- **Full integration (cả 2 hướng):** firm control nhiều mắt xích
  - Vd: **VNM** gần như full (đàn bò → chế biến → phân phối → retail)

**Bảng đánh giá vertical integration:**

| Mức tích hợp | Đặc điểm | Hướng chính | Ưu | Nhược |
|---|---|---|---|---|
| Pure-play 1 mắt xích | Single stage focus | None | Specialization, capex hiệu quả | Bị squeeze 2 đầu khi giá biến động |
| Backward (1 hướng) | VD HPG (lò cao + mỏ quặng) | Backward | Cost + supply security | Capex nặng, cycle risk amplified |
| Forward (1 hướng) | VD VNM owned distribution | Forward | Margin capture + customer data + Industry 4.0 enabler | Vốn lớn, retail execution risk |
| Full integration (cả 2) | VD VNM full chain | Both | Cost + quality + data + brand | Vốn rất lớn, ROIC dễ giảm |
| Conglomerate cross-industry | VD VIC, MSN | N/A (diversification) | Diversification | Complexity, holding discount, segment dilution |

→ Note rõ vị trí firm + hướng tích hợp + impact lên earnings volatility + Smile Curve position (mục 2.6.4).

**Output expected mục 2.6.2:**
- Quick: skim 2-3 primary activity material nhất + 1 dòng vertical integration verdict
- Standard: full 5 primary activities (1 đoạn ngắn mỗi cái) + 2-3 support activities relevant + bảng vertical integration
- Deep: full 5 primary + full 4 support activities + bảng vertical integration + cross-link Smile Curve + Industry 4.0

#### 2.6.3. Bargaining power matrix (Porter 5 forces + GVC governance + Tier supplier position)

**A. Porter 5 Forces (Porter 1979):**

Đánh giá 5 lực cho firm, mỗi lực **Cao / Trung / Thấp** + 1 dòng justify:

| Force | Đánh giá | Lý do (1 dòng) | Implication |
|---|---|---|---|
| **Bargaining power suppliers** | Cao/Trung/Thấp | [vd: 80% HRC nhập từ 3 nhà cung cấp China = supplier power cao] | Margin compress khi input giá tăng |
| **Bargaining power buyers** | Cao/Trung/Thấp | [vd: top 5 khách = 45% doanh thu → buyer power cao] | Pricing power giới hạn, dễ bị ép giá |
| **Threat of substitute** | Cao/Trung/Thấp | [vd: tôn mạ thay thế bằng nhựa composite trong 1 số ứng dụng] | TAM erosion dài hạn |
| **Threat of new entrants** | Cao/Trung/Thấp | [vd: capex lò cao $500M + 5 năm permitting → barrier cao] | Protect market share |
| **Industry rivalry** | Cao/Trung/Thấp | [vd: top 4 chiếm 75% market share, oligopoly] | Pricing discipline + margin sustainable |

**B. GVC governance type (Gereffi, Humphrey, Sturgeon 2005):**

5 mô hình quản trị chuỗi cung ứng toàn cầu — verdict cho firm:

| Mô hình | Đặc điểm | Ví dụ VN context | Implication margin |
|---|---|---|---|
| **Market** | Arm's-length, dễ switch supplier, sản phẩm chuẩn hoá | Commodity (KIMLOAI HRC, NONGNGHIEP gạo, THUYSAN raw) | Pricing power thấp, biên mỏng (5-10%) |
| **Modular** | Tiêu chuẩn hoá interface, supplier có năng lực kỹ thuật cao | FPT software services cho global enterprise | Margin trung bình (15-25%), có moat technical |
| **Relational** | Mutual dependency, long-term contract, joint dev | Aerospace components, automotive Tier 1 (chưa phổ biến VN) | Margin tốt (20-30%), switching cost cao 2 đầu |
| **Captive** | Supplier phụ thuộc nặng 1 lead firm, lead firm dictate spec | DETMAY CMT cho Uniqlo/Nike/H&M; Electronics EMS cho Samsung | Margin thấp (5-12%), dễ bị squeeze hoặc bị thay thế |
| **Hierarchy** | Full vertical integration, owned subsidiary | VNM (đàn bò → retail), HPG (mỏ → cán → phân phối) | Margin cao (20%+), nhưng vốn nặng |

**C. Tier supplier position:**

Quan trọng để hiểu vị trí firm trong global supply chain:

- **Tier 1:** cung cấp trực tiếp cho brand owner / end-customer
  - Vd: FPT là Tier 1 software vendor cho global enterprise; HPG cung cấp thép trực tiếp cho công ty xây dựng
- **Tier 2:** cung cấp cho Tier 1, không tiếp xúc end customer
  - Vd: Garmex Saigon (CMT) là Tier 2 cho Uniqlo (Tier 1 brand)
  - Vd: Nhiều electronics EMS VN là Tier 2-3 trong supply chain Apple/Samsung
- **Tier 3+:** cung cấp nguyên liệu / linh kiện cấp thấp hơn cho Tier 2

**Implication:**

- Tier 1 (đặc biệt với own brand) → pricing power tốt, captured customer data, leo smile dễ
- Tier 2 captive → margin mỏng + switching risk (brand owner có thể chuyển sang Bangladesh, Pakistan cheaper)
- Tier 2 modular (có năng lực kỹ thuật riêng) → moat tốt hơn captive

**D. Switching costs depth check (Porter classic):**

- Switching cost với supplier (firm-side): cao = firm bị supplier lock-in; thấp = firm bargain được
- Switching cost với customer (customer-side): cao = customer lock-in vào firm = pricing power; thấp = customer dễ bỏ
- Network effect (nếu có): mỗi user thêm tăng giá trị cho user khác → tự nhiên build moat (ít gặp ở SXKD traditional, nhiều ở CONGNGHE platform)

**Output expected mục 2.6.3:**
- Quick: 2-3 force material nhất + 1 dòng GVC governance verdict + Tier position
- Standard: Bảng 5 forces đầy đủ + bảng GVC + 1-2 dòng Tier position + switching cost note
- Deep: Bảng 5 forces + bảng GVC + Tier analysis chi tiết + switching cost depth + network effect (nếu applicable)

#### 2.6.4. Smile Curve / Vị trí capture giá trị (Stan Shih 1992)

**Concept:** Trong chuỗi giá trị ngành (đặc biệt manufacturing-heavy), giá trị (margin/value-add) phân bổ KHÔNG đều dọc chuỗi:

```
  Margin (%)
  ↑
  │ R&D / Design                              Brand / Service / After-sales
  │    ╲                                              ╱
  │     ╲                                            ╱
  │      ╲___________________________________________╱
  │              Manufacturing pure (low value-add)
  │              Assembly, fabrication, CMT, OEM/EMS
  │
  └──────────────────────────────────────────────────────→ Stage trong chuỗi
   Upstream                                    Downstream
   (đầu chuỗi: R&D, design,                   (cuối chuỗi: brand, service,
   technology, IP)                            customer relationship, data)
```

**3 zones trên Smile Curve:**

1. **Smile top — Upstream (R&D / Design / Technology / IP):** margin cao 20-40%+
   - Vd ngành: chip design (CONGNGHE fabless), pharma (R&D HOACHAT), patent-holding firms
   - VN ít doanh nghiệp ở zone này (FPT software design là 1 trong số ít)

2. **Smile bottom — Midstream (Manufacturing pure / Assembly):** margin thấp 3-12%
   - Vd ngành: DETMAY CMT, electronics EMS/ODM, raw commodity processing
   - Phần lớn doanh nghiệp SXKD VN niêm yết ở zone này — đây là **structural challenge**

3. **Smile top — Downstream (Brand / Service / After-sales / Customer data):** margin cao 20-50%+
   - Vd ngành: VNM (THUCPHAM own brand + retail), PNJ (BANLE jewelry brand), Apple (CONGNGHE brand + service)
   - VN có 1 số case nổi bật (VNM, MWG, PNJ, FPT) nhưng chưa phổ biến

**Implication cho VN context:**

- Phần lớn doanh nghiệp SXKD VN vẫn ở **"smile bottom"** — pure manufacturing/OEM/CMT/gia công, biên mỏng và phụ thuộc cost competitiveness (lao động rẻ + tỷ giá VND)
- **Đi lên smile** đòi hỏi:
  - Đầu tư R&D + brand building (chi phí cao trong ngắn hạn, payoff 5-10Y)
  - Phát triển dịch vụ giá trị gia tăng (after-sales, customisation, technical support)
  - Chuyển từ B2B unbranded → B2B branded / B2C own brand
  - Build customer data + Industry 4.0 capability (cross-link mục 2.6.5)

**Câu hỏi check Smile Curve position:**

| Câu hỏi | Smile bottom (yếu) | Smile top (mạnh) |
|---|---|---|
| Gross margin trend 3-5Y | 5-12%, biến động theo commodity | 25%+, ổn định hoặc expand |
| R&D / Revenue ratio | < 1% | 2-7% (varies industry) |
| Brand recognition + market share own brand | Không có own brand hoặc < 10% market | Own brand top 3 segment |
| Customer concentration | Top 3-5 KH = 50%+ doanh thu | Diversified, top 5 < 30% |
| Switching cost (customer side) | Thấp, dễ bị thay | Cao (brand loyalty, data, ecosystem) |
| Service / Recurring revenue | Không có | 20%+ recurring (subscription, maintenance) |

**Ví dụ VN minh hoạ vị trí trên Smile Curve:**

- **Smile bottom (low margin, structural challenge):**
  - Garmex Saigon (DETMAY CMT cho Uniqlo/Nike)
  - Most electronics EMS/ODM (subcontract Samsung/Apple)
  - HSG tôn mạ commodity (KIMLOAI midstream)
  - Many THUYSAN raw frozen export
- **Smile mid (improving, leo smile):**
  - TNG (DETMAY FOB + own brand TNG Fashion)
  - HPG (KIMLOAI integrated upstream + own distribution)
  - DBC (NONGNGHIEP feed + own brand pig farming)
- **Smile top (high margin, brand + service captured):**
  - VNM (THUCPHAM full brand + retail + Vinamilk equity)
  - PNJ (BANLE own jewelry brand + design)
  - FPT software (technical service + own platforms)
  - MWG (BANLE retail brand + customer data + ecosystem)

**Strategy migration check:**

- Firm có **strategy đi lên smile** không? (M&A brand, build R&D capability, develop service line)
- **Capex + opex** cho transition vs current margin profile — feasible không?
- **Tốc độ migration** dự kiến (3-5Y, 5-10Y, > 10Y)?
- **Competitive threat** từ players khác cũng cố gắng leo smile (race-up)?

**Output expected mục 2.6.4:**
- Quick: 2-3 dòng position (bottom/mid/top) + 1 dòng strategy verdict
- Standard: Bảng position + 3-4 dòng strategy migration + verdict (Smile bottom stuck / Smile climbing / Smile top defended)
- Deep: Bảng position chi tiết + competitive landscape Smile race + 5-10Y migration roadmap analysis

**Cross-reference:** Industry 4.0 lens (mục 2.6.5) — digitalisation thường là enabler chính để leo smile.

#### 2.6.5. Industry 4.0 / Digital footprint lens (CFA Sector Analysis 2020)

**Concept (từ CFA Sector Analysis 2020 Prelude — "Business Models Under the Onslaught of Industrial Revolution 4.0"):**

Mọi doanh nghiệp hiện đại đều tạo ra **digital footprint** và phải thích nghi với Industrial Revolution 4.0. Câu hỏi không còn là "có nên digital không" mà là "đã digital đến mức nào và còn thiếu gì". Đây là **non-negotiable** cho mọi firm — digital laggard sẽ bị disrupt.

**Three Golden Steps (CFA framework):**

1. **Tokenization (số hoá thông tin):** mọi giao dịch, hoạt động, tương tác stakeholder được số hoá → big data nền tảng
2. **Multidimensional interaction:** tương tác đa chiều, real-time với supplier + customer + employee + investor → enabled by 5G + IoT
3. **Embedded feedback mechanism:** vòng phản hồi tự động → quyết định AI-driven → mass customisation

**4 management DNA elements (CFA):**

- Desire to maintain sustainable ongoing interaction with consumers
- Willingness to decentralise + erect humanistic outreach
- Vision to create brands that breed togetherness (not just functionality)
- Capability to organically gel prototyping + research + design + refinement + version upgrade + promotion (do-and-plan approach)

**Lens phân tích Industry 4.0 cho firm SXKD VN — Bảng 7 chiều:**

| Chiều | Câu hỏi check | Vd Strong (VN) | Vd Weak (VN) |
|---|---|---|---|
| **Digital footprint generation** | Firm có tạo big data từ customer behavior + production line không? | MWG (app + loyalty), VNM (250K điểm bán data) | Small fragmented retailers, traditional manufacturing |
| **Production automation** | Tỷ lệ tự động hoá dây chuyền? Robot count? Cost curve position? | HPG dây chuyền lò cao tự động cao; VNM UHT line | DETMAY CMT phụ thuộc nhân công, low automation |
| **Mass customisation capability** | Khả năng customise sản phẩm cho từng khách hàng với cost gần như mass production? | Limited ở VN (FPT software tailored); MWG cá nhân hoá recommendation | Đa số firm VN chưa có |
| **IoT deployment** | Sensors trong nhà máy + sản phẩm + chuỗi cung ứng? | TIENICH (smart meter rollout), KCN (smart park), modern logistics | Traditional manufacturing |
| **AI/ML deployment** | Dùng AI cho pricing, demand forecast, quality control, recommendation? | Banking + e-commerce leading; MWG recommendation AI | SXKD traditional lag, < 1% revenue spend AI |
| **Mô hình kinh doanh** | Vẫn B2C / B2B truyền thống hay đã C2B (customer-driven feedback loop)? | Mobile gaming, e-commerce, some online retail C2B | Traditional B2C product manufacturers |
| **Tốc độ feedback loop** | Bao lâu firm nhận và phản ứng với customer feedback? | Real-time (e-commerce, fintech) | Hàng tháng/quý (traditional manufacturing) |

**Industry 4.0 maturity verdict:**

- **Leader:** strong ở 5+ chiều — tự động hoá cao + digital footprint mạnh + data analytics deployed → moat dày, leo smile dễ
- **Par:** mid ở phần lớn chiều — bình thường so với peer ngành → defensive position
- **Laggard:** weak ở 5+ chiều — vulnerable cho disruption + cạnh tranh từ digital-native players → conviction downgrade risk

**Implication cho thesis:**

- Firm Industry 4.0 leader + digital footprint mạnh → **moat dày hơn**, khả năng leo smile curve cao (mục 2.6.4)
- Firm Industry 4.0 laggard → **vulnerable** cho disruption + competition từ digital-native players
- **Đặc biệt quan trọng cho ngành:** BANLE (e-commerce disruption), CONGNGHE (tự nó là Industry 4.0), TIENICH (smart utility), VANTAI (logistics tech), NGANHANG (fintech disruption)

**Cross-reference với Smile Curve (mục 2.6.4):**

Digital footprint + AI/analytics + customer data thường là **enabler chính** để leo smile (từ manufacturing → brand/service). Vd: VNM dùng data từ 250.000 điểm bán retail để hiểu customer behavior → develop premium products + targeting → leo smile.

**Output expected mục 2.6.5:**
- Quick: 2-3 dòng skim Industry 4.0 status (leader/par/laggard)
- Standard: Bảng 7 chiều với verdict mỗi chiều (Strong/Trung/Weak) + 2-3 dòng implication chính
- Deep: Bảng đầy đủ + competitive threat analysis từ digital-native players + roadmap firm digital transformation 3-5Y

**Reference:** CFA Sector Analysis 2020 — Prelude "Business Models Under the Onslaught of Industrial Revolution 4.0" by Dr. Alan Lok et al.

#### 2.6.6. Position summary (Enhanced — tích hợp tất cả lens)

Tóm tắt 5-7 dòng tổng hợp tất cả lens phân tích:

- **Firm captures margin ở mắt xích nào** trong chuỗi giá trị ngành (mục 2.6.1)?
- **Vị trí trên Smile Curve** (mục 2.6.4): smile bottom / mid / top? Có đang leo lên không?
- **Mức độ + hướng vertical integration** (mục 2.6.2): pure-play / backward / forward / full / conglomerate?
- **GVC governance type** (mục 2.6.3): market / modular / relational / captive / hierarchy?
- **Tier supplier position** (mục 2.6.3): Tier 1 own brand / Tier 1 modular / Tier 2 captive / Tier 3 commodity?
- **Industry 4.0 readiness** (mục 2.6.5): leader / par / laggard so với peer?
- **Firm expose risk lớn nhất ở đâu** (supplier upstream / customer downstream / substitution / digital disruption)?
- **Pricing power verdict** (tổng hợp): strong (giá tăng pass-through dễ) / moderate (pass-through 1-2 quý lag) / weak (margin nhận hết hit)

**Output expected mục 2.6.6:** 5-7 dòng kết luận tích hợp. Đây là **input chính** cho:
- Phần 1 thesis core (vị trí chuỗi giá trị justify recommendation)
- Phần 2 competitive position
- Phần 6 bear case (risk ở mắt xích nào)
- Phần 4 valuation (premium/discount vs peer dựa trên vị trí Smile + Industry 4.0)

#### 2.6.7. Data sourcing cho chuỗi giá trị (Enhanced với CFA chapter mapping)

Pull data từ Stage 1 (xem `_01`):

| Data | Sub-step | Source |
|---|---|---|
| Top khách hàng / nhà cung cấp + % | 1i point 4 + 1p | Thuyết minh BCTC + BCTN |
| **Tier supplier position** | 1p + 1h web search | BCTN + supplier websites + industry research + sell-side report |
| Channel mix (B2B/B2C/GT/MT/Export) | 1p + 1h web search | BCTN + IR presentation + báo chí |
| Capacity + utilization | 1b + 1i | BCTC + thuyết minh tài sản cố định |
| Market share | 1l peer + 1h + 1j | Sell-side report + K_sector_framework |
| Geographic footprint | 1i + 1h | Thuyết minh BCTC + BCTN |
| Vertical integration (segment mix + direction) | 1a + 1b segment + 1h web | BCTC consolidated segment note + BCTN |
| **R&D / Revenue ratio** | 1b + 1i | BCTC chi phí R&D + thuyết minh |
| **Industry 4.0 readiness** | 1h + 1i + IR presentation | Annual report digital transformation section + sustainability report + IR deck + báo chí về công nghệ |
| Industry value chain map | 1j | K_sector_framework mục 5.X |
| **CFA Sector Analysis framework (industry-specific)** | 1j | **K_sector_framework + CFA Sector Analysis 2020 chapter (mapping ở mục 2.6.9)** |

→ Nếu data gap (vd thuyết minh không list top khách hàng) → fail-soft note "Không tìm được data top khách hàng cụ thể, dùng proxy từ BCTN/báo chí".
→ Nếu data gap cho **R&D ratio** hoặc **Industry 4.0 readiness** (firm không disclose) → đây là **negative signal** (firm không đầu tư hoặc không transparent về digital/R&D) → downgrade Industry 4.0 verdict.

#### 2.6.8. Áp dụng cho 4 kịch bản SXKD (cross-link với mục 2.2, enhanced)

- **Value Play candidate** → check chuỗi giá trị tổng hợp: firm có đứng mắt xích có moat / barrier không? Smile Curve position đang leo lên (mid → top)? Industry 4.0 ready? Vertical integration backward+forward? Nếu có nhiều điểm tích cực → **Long conviction cao**. Nếu commodity midstream + smile bottom + Industry 4.0 laggard + GVC captive → có thể là **Value Trap** (rẻ vì lý do cấu trúc).
- **Value Trap** → thường có pattern: smile bottom stuck + GVC captive + Industry 4.0 laggard + bị squeeze 2 đầu (supplier power cao + buyer power cao). Avoid.
- **Growth at Premium** → cần verify pricing power thực sự bền (gross margin expand 3-5Y) qua: vị trí Smile top + brand premium + Industry 4.0 leader (digital moat) + switching cost cao customer side. Nếu verified → justify premium. Nếu không → Watch (overvalued).
- **Cycle Top** → check earnings có phụ thuộc commodity upstream + smile bottom không. Nếu yes + cycle peak + Industry 4.0 laggard → roll over imminent + structural decline risk.

#### 2.6.9. CFA Sector Analysis 2020 — Industry mapping cho 18 whitelist + 3 financial

CFA Sector Analysis 2020 (CFA Institute & ACCA) bao phủ 21 industry chapters với 5-dimension model (DD/MP/SI/PM/ESG). Bảng mapping với 18 ngành whitelist + 3 ngành tài chính trong VN context — để pull additional questions specific từ CFA framework:

| Ngành VN | CFA Chapter(s) ưu tiên | Khía cạnh CFA đặc biệt useful cho value chain |
|---|---|---|
| **CONGNGHE** | Mobile Gaming (1), Semiconductors (5), Cashless Payments (2) | Digital footprint, network effect, IDM vs fabless model, ARPU, freemium revenue model |
| **BANLE** | Traditional Retail (3), Luxury Products (6) | Channel mix online/offline, private label threat, anchor tenant, footfall, tourist traffic |
| **DAUKHI** | Shipbuilding & Offshore Marine (4) | Offshore oil services, vessel mix, deep-sea capability, oil price sensitivity |
| **THUCPHAM** | Food Producers (8) | Sugar/fat regulation, private label threat, channel power retailer, e-commerce shift |
| **THUYSAN** | Food Producers (8) — sub-application | Anti-dumping tariff US/EU, sustainability certification, raw vs value-added |
| **NONGNGHIEP** | Palm Oil (14), Food Producers (8) | Upstream/midstream/downstream split rõ ràng, climate exposure, ESG certification |
| **KIMLOAI** | (gần Engineering & Construction (10) + Shipbuilding (4) upstream) | Commodity cycle, upstream integration backward, cost curve position |
| **HOACHAT** | (food producers chain (8) + selective semiconductors materials) | Feedstock cost (NH3, P2O5), end-product mix, environmental regulation |
| **DETMAY** | (Luxury Products (6) downstream + selective Food chain) | CMT/FOB/ODM/brand smile curve migration, US/EU order flow, sustainability certification |
| **CONGNGHIEP** | Engineering & Construction (10), selective | Order book, contract types, working capital |
| **XAYDUNG** | Engineering & Construction (10) | Order book pipeline, project management, working capital, fixed price risk |
| **VANTAI** | Logistics (12), Airlines (16), Shipbuilding (4) | Route economics, fleet age, fuel cost, FSC/LCC mix, container freight rates |
| **TIENICH** | Utilities (21) | Regulatory pricing AroR, RCV (Regulated Capital Value), capex cycle, renewable transition |
| **BDS** | Property Development (17) | Lumpy revenue, JV model, land bank quality, REIT spin-off opportunity |
| **KCN** | Property Development (17) industrial sub + REITs (20) warehouse | FDI demand, occupancy, lease structure, geographic moat (proximity ports) |
| **KHOANGSAN** | (Palm Oil (14) upstream + Shipbuilding (4) offshore) | Reserve life, commodity price, mining permit, ESG sensitivity |
| **DULICH** (override) | Airlines (16), Casino (15), REITs (20) hospitality | Capacity, RevPAR, regulatory licenses, gaming tax, Chinese tourist exposure |
| **YTEGD** (override) | Healthcare (7) | Drug pipeline, FDA/regulatory approvals, generic vs original, MLR (insurance side) |
| **NGANHANG** | Banking Industry (18) | Already covered in Type NH framework (mục 3 file này) — supplement chỉ cho fee income business |
| **CHUNGKHOAN** | Banking Industry (18) IB section | Already covered in Type CK framework (mục 4 file này) |
| **BAOHIEM** | Insurance (11) | Already covered in Type BH framework (mục 5 file này) |
| **Education (override)** | Education (13) | Enrolment rates, tuition pricing, student demographics |
| **REITs (override)** | REITs (20) | Cap rate, NOI, FFO/FAD, tenant diversification |

**Khi pull CFA framework specific cho 1 ngành (workflow):**

1. **Identify chapter** CFA tương ứng (table trên)
2. **Pull 5-dimension framework:** Common to sector + sub-categories questions (DD/MP/SI/PM/ESG) từ CFA chapter
3. **Cross-check với `K_sector_framework`** mục 5.X local (VN context) — CFA cung cấp global framework, K_sector_framework cung cấp VN nuance
4. **Áp dụng vào value chain analysis** (mục 2.6.1-2.6.5):
   - CFA DD → demand drivers cho value chain firm (mục 2.6.2 Marketing & Sales)
   - CFA MP → competitive position trong chuỗi giá trị (mục 2.6.3)
   - CFA SI → structural trends ảnh hưởng vị trí Smile Curve dài hạn (mục 2.6.4)
   - CFA PM → operational + financial metrics validate vị trí value chain
   - CFA ESG → supply chain ESG risk (Scope 3, supplier compliance) — cross-link Stage 1o

**Reference:** CFA Sector Analysis: A Framework for Investors (2020) — CFA Institute & ACCA. Khi user upload CFA PDF hoặc agent có access bookmarked summary, pull chapter cụ thể từ table mapping trên.

## 3. Type NH — Ngân hàng

### 3.1. Scope

Áp dụng cho ticker thuộc industry NGANHANG. Bao gồm: VCB, BID, CTG, ACB, MBB, TCB, TPB, VPB, HDB, EIB, STB, SHB, OCB, LPB, MSB, NAB, ABB, VBB, PGB, BAB, SGB, KLB, NVB, BVB.

### 3.2. Analytical framework

**Core profit driver: NII (Net Interest Income) + NoII (Non-interest Income) − OpEx − Cost of credit**

```
Net Income = (Avg interest-earning asset × NIM) 
           + Fee income + Trading income + Other income
           − OpEx (CIR)
           − Cost of credit (NPL provisioning)
           − Tax
```

**Đánh giá theo 3 lens chính:**

1. **NIM trend + drivers** — quan trọng nhất
2. **Asset quality** (NPL, coverage, cost of credit) — gate quality
3. **Growth + capital** (loan growth, CASA, CAR headroom)

### 3.3. KPIs chính theo Type NH

**Profitability:**
- **NIM (Net Interest Margin)** — TTM + trend 8 quý. Driver:
  - Loan yield (đặc biệt retail vs corporate mix)
  - Funding cost (đặc biệt CASA ratio)
  - Spread = Loan yield − Funding cost
- **NII growth YoY** + breakdown (volume vs price)
- **NoII / Total income** — fee income mix, trading income volatility
- **CIR (Cost-Income Ratio)** — efficiency, mục tiêu < 40% cho top tier
- **ROE / ROTE** — TTM + trend
- **ROA** (cho banking < 2% là good, > 2.5% là exceptional)

**Asset quality (CRITICAL):**
- **NPL ratio** = NPL (Group 3-5 theo SBV) / Total loan
- **NPL theo nhóm** breakdown: Group 2 (cảnh báo), Group 3 (nghi ngờ), Group 4 (nghi ngờ cao), Group 5 (mất vốn)
- **LLR coverage** = Loan Loss Reserve / NPL (mục tiêu > 100%, top tier > 150%)
- **Cost of credit** = Annual NPL provisioning / Average loan (basis points)
- **Group 2 inflow trend** — leading indicator NPL formation (Group 2 → 3 → 4 → 5)
- **Restructured loan ratio** (sau Circular regulation đặc biệt như Circular 02/2023 VN)

**Growth + funding:**
- **Loan growth YoY** + **Loan-to-Deposit Ratio (LDR)** — flag nếu LDR > 85%
- **Deposit growth YoY** + **CASA ratio** (current account + savings account / total deposit) — cao = funding cost rẻ
- **Loan concentration** (top 10 borrower / total loan; ngành mix)
- **Off-balance sheet exposure** (L/C, guarantee, derivative)

**Capital + liquidity:**
- **CAR (Capital Adequacy Ratio)** — Tier 1 + Total (theo Basel II/III). Headroom vs regulatory minimum (8% min Basel II VN, > 9% recommended)
- **Tier 1 / RWA** breakdown
- **LCR (Liquidity Coverage Ratio)** — Basel III standard
- **NSFR (Net Stable Funding Ratio)**

**Bank-specific FA (từ thuyết minh BCTC 1i.15 NH):**
- LDR
- CAR breakdown
- NPL theo Group 2-5
- LLR coverage trend
- Loan concentration (top 10)
- Fee income mix (bancassurance, cards, FX, advisory) — đặc biệt bancassurance commission sau Circular 50/2023 siết
- Off-balance sheet exposure

### 3.4. NH-specific catalysts + bear case

**Positive catalysts:**
- Chu kỳ hạ lãi suất → NIM bottoming → expand
- Credit growth recovery (sau period siết Circular)
- Asset quality improvement (NPL formation ngừng → write-back)
- CAR headroom → có thể bán cổ phần private placement giá tốt
- M&A foreign partner (vd VPB-SMBC) — capital injection + know-how
- Foreign room headroom + state ownership giảm (privatization SOCB)

**Bear case typical:**
- NIM compression (cuộc đua giảm lãi suất cho vay để giữ market share)
- NPL spike từ specific exposure (BĐS, energy, restructured loan rollover)
- CAR breach → capital raise dilutive
- Group 2 inflow tăng → forecast NPL 1-2 năm tới spike
- Regulatory tightening (room tín dụng siết, classification stricter, bancassurance siết)
- Bond default exposure (vd 2022-2023 Tan Hoang Minh / Van Thinh Phat ảnh hưởng SCB, TPB)

### 3.5. Reference cross-pack

- **`K_agent_db_04`** mục PTCB Banking (4 type framework)
- **`K_sector_framework`** mục 5.1 NGANHANG
- **Macro variables** (Stage 1k): lãi suất điều hành SBV, lãi suất huy động/cho vay, US Treasury yield, Fed funds rate, USD/VND

## 4. Type CK — Chứng khoán (Securities)

### 4.1. Scope

Áp dụng cho ticker thuộc industry CHUNGKHOAN. Bao gồm: SSI, VCI, HCM, VND, SHS, MBS, FTS, BSI, CTS, AGR, BVS, ORS, VIX, APG, BMS, etc.

### 4.2. Analytical framework

**Core profit driver: Brokerage commission + Margin yield + IB fee + Trading P&L**

```
Net Income = Brokerage commission (commission rate × volume) 
           + Margin lending income (margin balance × spread)
           + IB / advisory fee
           + Prop trading P&L (volatile)
           − OpEx
           − Tax
```

**Đánh giá theo 3 lens chính:**

1. **Market share + brokerage economics** — sustainable competitive position
2. **Margin book quality + leverage** — risk-return engine
3. **Prop book + IB pipeline** — volatility + upside option

### 4.3. KPIs chính theo Type CK

**Brokerage business:**
- **Market share** (volume + value) — TTM + trend 8 quý
- **Brokerage commission rate** trung bình (đang đua giảm về 0)
- **Brokerage commission / Total revenue** — declining secular
- **Number of active accounts** — mới mở + churn
- **Revenue per active account**

**Margin lending (engine chính khi market tăng):**
- **Margin balance** (tỷ VND) — TTM + trend
- **Margin / Equity** (leverage proxy) — flag nếu > 200%
- **Margin yield** = (Margin interest income) / Avg margin balance — TTM
- **Cost of funding margin** (lãi suất huy động + bond issuance)
- **Margin spread** = Margin yield − Cost of funding
- **Margin concentration**: top 10 client / total margin; ngành mix
- **Collateral quality**: % large cap / mid cap / small cap as collateral; LTV (loan-to-value) ratio

**IB / Advisory:**
- **IB fee income** + breakdown (M&A advisory, equity capital market, debt capital market, bond underwriting)
- **Pipeline deals** (number + estimated fee)
- **Underwriting commitment outstanding** (off-balance sheet risk)

**Proprietary trading (prop book):**
- **Prop book size** (tỷ VND) + composition (equity, bond, derivative)
- **VaR (Value at Risk)** — daily, monthly
- **Trading P&L volatility** quarterly

**Operational + capital:**
- **CIR (Cost-Income Ratio)**
- **ROE / ROA** TTM
- **Capital Adequacy Ratio (CAR — UBCK requirement)**
- **Customer money safekeeping** (off-balance sheet — phải tách riêng theo regulation)

**CK-specific FA (từ thuyết minh BCTC 1i.15 CK):**
- Margin loan breakdown by collateral + concentration
- Prop book composition (equity, bond, derivative)
- Market risk VaR
- Customer money safekeeping breakdown
- Bond underwriting commitments outstanding

### 4.4. CK-specific catalysts + bear case

**Positive catalysts:**
- Market turnover bùng nổ (ADV thị trường tăng → brokerage volume tăng)
- Margin balance thị trường mở rộng (đặc biệt khi NHNN nới room margin / margin spread tốt)
- IPO / corporate bond pipeline lớn (IB fee)
- Foreign room nới (cho phép foreign mua thêm cổ phần broker)
- VN Index level lên (multiplier cho mọi business line)

**Bear case typical:**
- Market crash → margin call cascade → loss on collateral concentration
- Brokerage commission compression (zero commission war)
- Prop book loss large (đặc biệt khi thị trường crash + leveraged prop)
- Regulatory tightening (margin lending cap, classification strict hơn)
- Investor account churn (retail mệt sau drawdown 2022-2023)
- Foreign broker (Mirae Asset, KIS) cạnh tranh fee aggressive

### 4.5. Reference cross-pack

- **`K_agent_db_04`** mục PTCB CK
- **`K_sector_framework`** mục 5.11 CHUNGKHOAN
- **Macro variables** (Stage 1k): ADV thị trường, margin balance thị trường, VN-Index level, foreign flow, lãi suất

## 5. Type BH — Bảo hiểm (Insurance)

### 5.1. Scope

Áp dụng cho ticker thuộc industry BAOHIEM. Lưu ý: BAOHIEM **ngoài whitelist 18 mặc định** (theo `K_agent_db_00` mục 4.5) — chỉ phân tích khi user override.

Bao gồm: BVH (Bảo Việt — composite life + non-life), PVI (non-life + reinsurance), BIC, BMI, MIG, PGI, PRE, PTI, ABI, VNR (Vinare — reinsurance).

### 5.2. Analytical framework

Phân biệt 2 sub-type:

**Sub-type BH-Life (whole life, term life, universal life, retirement):**
- Long-duration liability
- Sensitive với lãi suất (reserve + investment yield)
- Persistency rate quan trọng
- Embedded Value framework

**Sub-type BH-Non-life (motor, property, health, marine):**
- Short-duration liability
- Sensitive với claim frequency + severity
- Combined Ratio framework
- Reinsurance treaty quan trọng

VN listed mostly Non-life (BVH có life). Pack này focus framework cả 2.

### 5.3. KPIs chính theo Type BH

**Premium business:**
- **GWP (Gross Written Premium)** — total
- **NWP (Net Written Premium)** = GWP − Premium ceded to reinsurer
- **APE (Annual Premium Equivalent)** = Annual premium + 10% × Single premium (life)
- **NBV (New Business Value)** — present value future profit của new business written
- **Persistency rate (life)** — % policies renewed at year 1, 2, 5

**Underwriting profitability (Non-life chính):**
- **Combined Ratio** = Loss Ratio + Expense Ratio
  - < 100% = underwriting profit
  - 100-105% = marginal, dựa vào investment income
  - > 105% = underwriting loss, bù bằng investment
- **Loss Ratio** = Claims incurred / Premium earned
- **Expense Ratio** = Underwriting expense / Premium earned (acquisition cost + admin)
- **Reinsurance recovery ratio**

**Investment management:**
- **AUM (Assets Under Management)** size
- **Investment yield** = Investment income / Avg AUM (TTM)
- **Investment portfolio breakdown**: govt bond, corp bond, equity, real estate, deposit
- **Duration matching** (life): asset duration vs liability duration

**Capital + reserve:**
- **Technical reserve** = UPR (Unearned Premium Reserve) + IBNR (Incurred But Not Reported) + Outstanding claims reserve
- **Claim reserve adequacy** — historical reserve runoff test
- **Solvency margin** = Available capital / Required capital (regulatory)

**Embedded Value framework (Life):**
- **EV (Embedded Value)** = Adjusted Net Asset Value + Value of In-Force business
- **NBV / APE** = NBV margin
- **EV growth YoY** — driven by NBV addition + investment variance

**BH-specific FA (từ thuyết minh BCTC 1i.15 BH):**
- Technical reserve breakdown (UPR + IBNR + Outstanding claims)
- Claim reserve adequacy (runoff test)
- Investment portfolio breakdown
- Solvency margin
- Reinsurance recoverable

### 5.4. BH-specific catalysts + bear case

**Positive catalysts:**
- Penetration tăng (VN ~3% GDP, target 5%+ — dư địa lớn)
- Premium rate hardening (sau period nhiều claim — hard market cycle)
- Lãi suất 10Y tăng → investment yield tăng → ROE improve (life)
- Bancassurance recovery sau period siết Circular 50/2023
- Health insurance demand bùng (post-COVID + ageing)

**Bear case typical:**
- Major catastrophe (bão, lũ, dịch bệnh) → claim spike → loss ratio blow up
- Mis-selling crisis (như 2023 bancassurance crisis VN)
- Interest rate down → investment yield compress (life)
- Persistency dropdown → NBV value erode
- Regulatory tightening (Circular 50/2023, solvency II adoption)
- Anti-fraud system fail → claim fraud increase

### 5.5. Reference cross-pack

- **`K_agent_db_04`** mục PTCB BH
- **`K_sector_framework`** mục 5.2 BAOHIEM (override mode)
- **Macro variables** (Stage 1k): lãi suất 10Y, GDP growth

## 6. Cross-type decision rule (edge cases)

### 6.1. Conglomerate / Holding company

Mã có operations across multiple types (vd VIC có BĐS + retail + manufacturing; MSN có tiêu dùng + retail + mining).

**Rule:**
1. Pull BCTC consolidated + segment breakdown từ thuyết minh
2. Identify segment > 60% doanh thu hoặc > 60% net profit
3. Áp framework type tương ứng segment đó là PRIMARY
4. Mention các segment khác là SECONDARY context
5. Nếu không có segment > 60% (truly diversified) → fall back SXKD framework + note rõ "doanh nghiệp đa ngành, không phân loại single-type"

Ví dụ:
- **VIC** (Vingroup): BDS chiếm > 60% → primary SXKD-Growth/Infrastructure framework (BDS sub-type) + secondary mention VinFast (manufacturing), Vinpearl (consumer)
- **MSN** (Masan): MasanConsumer (FMCG) + WinCommerce (retail) + Masan High-Tech (mining) — mixed → SXKD + segment breakdown chi tiết

### 6.2. Subsidiary of listed parent

Mã là subsidiary của parent niêm yết khác (vd FRT là sub của FPT? — không, FRT là spinoff, độc lập).

**Rule:**
- Phân tích standalone
- Note quan hệ với parent + related party transactions (Stage 1i point 4)
- Flag nếu parent control > 50% (potential governance issue)

### 6.3. Newly listed (< 2Y)

Mã mới list, BCTC public < 2 năm.

**Rule:**
- Web search bổ sung BCTN tiền-IPO (annual report private trước listing)
- Note "Newly listed, history limited, conviction capped at MID"
- Bear case typical cho newly listed: lock-up expiry, governance evolving, audit quality

### 6.4. Holding công ty mẹ (parent only) vs Tập đoàn (consolidated)

Một số mã có 2 set BCTC (riêng lẻ + hợp nhất). Always dùng **BCTC consolidated** cho phân tích (vì reflect total value creation). BCTC riêng lẻ chỉ dùng để check intercompany flows + dividend distribution capacity.

## 7. Output format type-specific phần 2 (cho `_03` Stage 2)

Khi compose Phần 2 (Type-specific business analysis) của báo cáo, render theo template phụ thuộc type:

### Template SXKD

```
[Tên công ty] hoạt động trong ngành [X] với [Y] segment chính. 
[Segment 1] chiếm [%], [Segment 2] chiếm [%].

Cấu trúc earnings:
- Revenue YoY: [%] (volume [%] + price [%])
- Gross margin: [%] vs [%] cùng kỳ
- EBIT margin: [%] vs [%] cùng kỳ
- ROIC TTM: [%]
- Net debt/EBITDA: [x]

Chuỗi giá trị ngành (xem mục 2.6.1):
[Upstream → midstream → downstream map ngắn 2-3 dòng + margin pool note]

Chuỗi giá trị doanh nghiệp Porter (xem mục 2.6.2):
Primary activities:
- Inbound: top NCC [...], % giá vốn [...], Tier supplier position [...]
- Operations: công suất [...], utilization [...], cost curve position [...], Industry 4.0 automation level [...]
- Outbound + Marketing: kênh [...], top KH [...] (% doanh thu [...])
- Service: recurring revenue % [...]
Support activities:
- R&D / Revenue ratio: [...] (Strong > 2% / Par 1-2% / Weak < 1%)
- Technology development + digital roadmap: [...]
Vertical integration: [pure-play / backward / forward / full / conglomerate]

5 forces verdict (xem mục 2.6.3):
- Supplier power: [Cao/Trung/Thấp] — [1 dòng]
- Buyer power: [Cao/Trung/Thấp] — [1 dòng]
- Rivalry: [Cao/Trung/Thấp] — [1 dòng]
- (substitute + new entrant — note nếu material)
GVC governance: [market / modular / relational / captive / hierarchy]
Tier position: [Tier 1 own brand / Tier 1 modular / Tier 2 captive / Tier 3 commodity]

Smile Curve position (xem mục 2.6.4):
- Vị trí: [smile bottom (manufacturing pure) / smile mid (climbing) / smile top (brand + service)]
- Migration strategy + speed: [stuck / climbing (X-Y năm) / smile top defended]

Industry 4.0 readiness (xem mục 2.6.5):
- Verdict tổng: [Leader / Par / Laggard]
- Highlights: [automation X% / digital footprint generation Y / AI deployment Z]

Cycle position: [Value Play / Value Trap / Growth at Premium / Cycle Top]
Lý do classification: [2-3 dòng justification từ định giá phân vị + earnings outlook + Smile position + Industry 4.0 readiness]

Pricing power tổng hợp: [strong/moderate/weak] dựa trên [evidence từ gross margin trend + market share + brand + Smile position + GVC governance + Industry 4.0]
```

### Template NH

```
[Tên ngân hàng] xếp thứ [N] về tổng tài sản / quy mô tín dụng. Mix retail/corporate: [%/%].

NIM TTM: [%], trend [tăng/giảm/ổn định] [N] quý qua. 
Drivers chính:
- Loan yield: [%] (retail-heavy / corporate-heavy)
- Funding cost: [%] (CASA ratio [%])
- Spread: [%]

Asset quality:
- NPL: [%] (Group 3-5)
- LLR coverage: [%]
- Group 2 inflow: [trend - leading indicator]
- Cost of credit TTM: [bps]

Capital:
- CAR Tier 1: [%]
- CAR Total: [%]
- LDR: [%]

ROE TTM: [%], ROA TTM: [%]
CIR: [%]
```

### Template CK

```
[Tên công ty CK] xếp thứ [N] về market share brokerage ([%] HOSE + [%] HNX). 
Mix client: [retail/institutional %].

Revenue breakdown:
- Brokerage commission: [%]
- Margin lending income: [%]
- IB / advisory: [%]
- Prop trading: [%]
- Other: [%]

Margin business:
- Margin balance: [tỷ VND]
- Margin / Equity: [%] (leverage)
- Margin yield: [%]
- Margin spread: [%]
- Top 10 client concentration: [%]
- Collateral quality: [large/mid cap mix]

ROE TTM: [%], CIR: [%], CAR: [%]
Prop book VaR: [tỷ VND] / [% equity]
```

### Template BH

```
[Tên công ty BH] hoạt động chủ yếu [Life / Non-life / composite].

Premium:
- GWP TTM: [tỷ VND], growth YoY [%]
- NWP / GWP: [%] (reinsurance retention)
- APE (life): [tỷ VND]

Underwriting (Non-life):
- Combined Ratio TTM: [%]
- Loss Ratio: [%]
- Expense Ratio: [%]

Investment:
- AUM: [tỷ VND]
- Investment yield TTM: [%]
- Portfolio breakdown: [Bond %, Equity %, Deposit %, RE %]

Capital:
- Solvency margin: [%]
- Technical reserve: [tỷ VND]

NBV / APE (life): [%]
EV (life): [tỷ VND]
ROE TTM: [%]
```

## 8. Decision flow Stage 1 → Type framework

```
Stage 1a output: Type = SXKD / NH / CK / BH

↓

Activate type-specific lens:
- SXKD → mục 2 above + sub-type cycle dynamics (mục 2.4)
- NH → mục 3 above
- CK → mục 4 above  
- BH → mục 5 above

↓

Compose Phần 2 báo cáo (Stage 2 — chi tiết ở `_03`):
- Pull data type-specific từ Stage 1b + 1i forensic
- Render theo template type tương ứng (mục 7 above)
- Cross-reference K_sector_framework mục 5.X cho industry lens (Phần 3)
```
