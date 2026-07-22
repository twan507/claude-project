# K_sector_framework — Khung phân tích ngành chuẩn institutional buy-side

## 1. Mục đích & scope

Pack cung cấp **khung lens systematic** để deep-dive sector-level analysis, chắt lọc từ CFA Sector Analysis Framework (2020). Mục tiêu: bổ sung góc nhìn **industry structure + competitive dynamics + ESG** cho mọi pack P khi cần đào sâu cấp ngành — KHÔNG thay thế dòng tiền/PTCB của `K_agent_db_04`.

**Input kỳ vọng:** ticker / ngành cần deep-dive, hoặc câu hỏi định tính về cấu trúc cạnh tranh / drivers cấu trúc / ESG của 1 ngành.

**Output kỳ vọng:** structured content theo 5 dimension (DD/MP/SI/PM/ESG) ráp được vào memo, sector tilt, hoặc industry brief.

**Negative scope:**
- Không cung cấp số liệu định lượng — số liệu vẫn lấy từ `agent_db` qua `K_agent_db_01/02`
- Không thay thế methodology dòng tiền + PTCB 4 type doanh nghiệp của `K_agent_db_04`
- Không phải workflow — pack này là **reference layer**, P pack chủ động pull khi cần
- Không bao trùm hết 18 ngành whitelist — chỉ direct cover 10-12 ngành; còn lại dùng universal framework + analogue

**Quan hệ với pack khác:**

| Pack | Vai trò | Quan hệ với pack này |
|---|---|---|
| `K_agent_db_04` | Methodology dòng tiền + PTCB 4 type doanh nghiệp + technical | **Bổ trợ**, không overlap. `K_agent_db_04` từ data DB, pack này từ industry structure CFA |
| `K_agent_db_01/02` | Schema + query patterns | Pack này tham chiếu khi gợi ý metric cần kéo |
| `P_invest_memo_05/06/07` | Tier 5A/B/C deep-dive memo | Tham chiếu khi compose phần "Business" của memo 7 phần |
| `P_vbse_strategy_04` | Trục 4 Sector allocation | Tham chiếu khi compose per-sector tilt rationale |
| `P_weekly_overview_02` | Phần 6 Biến động 18 ngành | Tham chiếu khi cần structural watch (không phải mặc định mỗi tuần) |
| `P_stock_report_01/02/03` | Stage 1j sector context + Phần 2 sub-type + Phần 3 industry context | Tham chiếu mạnh — pull cho Standard+ depth mode khi compose Phần 3 (Bối cảnh ngành & vĩ mô) và sub-type context Phần 2 (theo SXKD/NH/CK/BH) |

## 2. Triết lý

**Bottom-up institutional buy-side.** Không phải top-down macro analysis (đó là việc của `P_weekly_overview` phần 3-5 và `P_vbse_strategy_01`). Pack này focus vào: ngành A có cấu trúc kinh tế cụ thể nào? Pricing power ra sao? Barriers to entry là gì? Sustainable competitive advantage ở đâu?

**Universal framework first, sector-specific second.** Khung 5 dimension áp dụng cho mọi ngành. Per-sector quick-ref chỉ là "shortcut" cho 10-12 ngành đã có CFA cover; với ngành ngoài cover (DAUKHI, HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, THUYSAN, CONGNGHIEP) vẫn áp dụng universal framework, có thể mượn analogue từ ngành tương cận.

**Lens, không phải checklist.** Không bắt buộc trả lời hết 50 câu hỏi mỗi deep-dive. Pull 3-5 câu **most material** cho ngành/mã đang phân tích, ưu tiên câu có data trong `agent_db` để answer được định lượng.

**VN context override.** CFA framework là global standard. Khi conflict với thực tế VN (vd ESG ở VN chưa đủ trưởng thành như US/EU), ưu tiên VN context. Pack này note rõ chỗ nào CFA phù hợp, chỗ nào cần weight lại.

## 3. Universal 5-dimension framework

### 3.1. Demand Drivers (DD)

**Mục tiêu:** Hiểu cái gì drive doanh thu của firm trong ngành này.

**Câu hỏi cốt lõi (chọn 3-5 câu có data answer được):**

1. **Sản phẩm/dịch vụ chính** là gì? Mix doanh thu theo product line, customer segment, geographic market như thế nào? Có dependency vào 1 product/customer/region nào không?

2. **Discretionary vs non-discretionary demand**: bao nhiêu % doanh thu thuộc nhóm thiết yếu (consumer staples, utilities, healthcare), bao nhiêu % thuộc nhóm tùy ý (luxury, leisure, big-ticket)? Sensitivity với chu kỳ kinh tế?

3. **Price elasticity**: demand thay đổi bao nhiêu % với 1% thay đổi giá? Có đối xứng giữa tăng giá và giảm giá không?

4. **Demand seasonality**: cyclical hay non-cyclical? Seasonal peaks (Tết, Q4 retail, mùa khô/mưa cho construction)? Cách firm tối ưu utilization?

5. **Customer concentration**: top 3 / top 5 customer đóng góp bao nhiêu % doanh thu? Có rủi ro lock-in / lose-key-account không?

6. **Pricing model**: subscription, transactional, project-based, cost-plus, regulated, market-based? Recurring revenue mix ra sao?

7. **Demand drivers ngoài firm**: macro (GDP, lãi suất, tỷ giá, FDI), demographic (urbanization, ageing, household formation), regulatory (subsidies, tariffs, mandates), technology (disruption, adoption rate)?

### 3.2. Market Position (MP)

**Mục tiêu:** Hiểu firm đứng ở đâu trong cấu trúc ngành.

**Câu hỏi cốt lõi:**

1. **Market share**: firm đứng thứ mấy trong ngành / segment / region cụ thể? Share của Top 3 / Top 5 (concentration ratio)? Trend (gaining or losing share)?

2. **Barriers to entry**: regulatory (license, capital requirement, approvals), economic (economies of scale, network effects, learning curve), structural (distribution access, brand power, IP/patents), behavioral (switching cost, customer inertia)?

3. **Competitive advantage**: cost leadership, differentiation, niche focus, network effect, brand premium, technology moat, regulatory protection, scale advantage? Sustainable bao nhiêu năm?

4. **Pricing power**: firm là price-maker, price-taker, hay price-follower? Margin trend so với peers (gross, operating, net)?

5. **Substitutes & complements**: sản phẩm thay thế (cross-elasticity) là gì? Complementary products tăng/giảm dùng có ảnh hưởng ngược lại không?

6. **Bargaining power**: với suppliers (input concentration, switching cost), với customers (customer concentration, alternatives, transparency)?

7. **Strategic posture**: ownership structure (state, family-controlled, widely-held, foreign), M&A activity, partnership/joint-venture, expansion strategy (organic vs acquisition, domestic vs overseas)?

### 3.3. Structural Influences (SI)

**Mục tiêu:** Hiểu xu hướng dài hạn shape ngành 3-10 năm tới.

**Câu hỏi cốt lõi:**

1. **Demographic trends**: dân số, urbanization rate, ageing, household formation, education level — có driver structural cho ngành?

2. **Macroeconomic trends**: GDP per capita growth, consumption vs investment mix, financial deepening, FDI flow — drive demand như thế nào?

3. **Regulatory trends**: chính sách hiện hành + dự thảo sắp ban hành. Subsidy / tariff / quota / license cap / ESG mandate? Direction: ủng hộ hay siết ngành?

4. **Technology disruption**: AI, automation, IoT, blockchain, fintech... có shift cost structure / customer behavior / competitive dynamics ngành này không? Firm vị thế leader hay laggard trong adoption?

5. **Industry consolidation**: M&A wave, top players merge, market share tập trung hơn? Hay fragmenting do new entrants tech-enabled?

6. **Globalization vs localization**: ngành chịu ảnh hưởng global cycle (commodity, tech, finance) hay domestic-driven (utilities, real estate, retail)? Trade war / supply chain shift impact?

7. **Sustainability / climate change**: ngành có exposure direct (carbon-intensive, water-intensive, deforestation-linked) hay indirect (financing those)? Stranded asset risk?

### 3.4. Performance Metrics (PM)

**Mục tiêu:** Đo lường mức độ thành công của firm so với peers + theo thời gian.

**Câu hỏi cốt lõi:**

1. **Revenue metrics**: tăng trưởng top line (organic vs inorganic, volume vs price), revenue mix evolution, recurring revenue share?

2. **Margin metrics**: gross margin, operating margin (EBIT/EBITDA), net margin. Trend + so với peers. Cost structure breakdown (raw material, labor, marketing, R&D, depreciation).

3. **Capital efficiency**: ROE, ROA, ROIC. Asset turnover, inventory days, receivable days, payable days, cash conversion cycle.

4. **Leverage & liquidity**: D/E, net debt/EBITDA, interest coverage, current ratio. Cost of debt vs ROIC (value creation gap).

5. **Cash flow quality**: FCF margin, FCF conversion (CFO/Net Income), capex intensity (capex/revenue), capex split (maintenance vs growth).

6. **Sector-specific KPIs**: từng ngành có set metric riêng (sẽ liệt kê trong mục 5 per-sector). Vd: Banking NIM/CAR/NPL, REIT FFO/Cap rate/Occupancy, Retail same-store sales/sell-through.

7. **Backlog / pipeline / book-to-bill**: với ngành project-based (E&C, shipbuilding, IT services), order book size + book-to-bill ratio là leading indicator.

### 3.5. ESG factors

**Mục tiêu:** Đánh giá rủi ro + cơ hội phi tài chính, không phải tick-box.

**Câu hỏi cốt lõi:**

1. **Environmental footprint**: emission intensity (CO2/revenue, CO2/tonne sản xuất), water intensity, waste generation, pollution incidents. Reduction roadmap?

2. **Social — labor & community**: worker safety track record (fatality, LTIFR), labor relations (strike history, union density), supply chain labor (child labor, modern slavery audit), community impact (displacement, social license)?

3. **Social — product**: product safety recalls, customer privacy/data breaches, marketing ethics (target vulnerable groups?), affordability/accessibility?

4. **Governance — structure**: board independence, related party transactions, controlling shareholder dominance (state, family), executive compensation alignment?

5. **Governance — conduct**: corruption / bribery investigations, regulatory fines, anti-competitive behavior, political donations, transparency của disclosure?

6. **Sector-specific ESG hotspots**: từng ngành có 1-2 ESG issue đặc trưng (sẽ liệt kê per-sector). Vd: Banking — money laundering / financing controversial sectors; Palm oil — deforestation / labor; Casino — addiction / problem gambling.

7. **Forward ESG risk**: regulation đang siết (carbon pricing, plastic ban, single-use), consumer sentiment shift (ESG-conscious buying), investor screening (ESG fund exclusion)?

**VN context note:** ESG ở VN chưa mandatory disclosure đầy đủ. Khi DB không có data, có thể dùng web search reports (UNGC VN, GRI VN, Forest Trends cho palm oil/timber, VietnamCredit, FiinGroup ESG score). KHÔNG bịa data.

## 4. Industry 4.0 lens

Áp dụng cross-sector. Cho mọi firm phân tích, đánh giá 4 dimension:

1. **Digital footprint**: firm có generate / collect / monetize data từ operations không? Loại data (transaction, behavior, sensor IoT)? Dùng nội bộ để optimize hay sold/licensed bên ngoài?

2. **Automation level**: % process đã automated (manufacturing, customer service, back-office)? Plan đầu tư automation 3-5 năm tới? Cost saving / efficiency gain mục tiêu?

3. **Customer interface digitalization**: % customer interact qua kênh digital (app, web, chatbot, self-service)? Mức độ omnichannel? Customer data platform có unified không?

4. **AI/ML deployment**: firm dùng AI cho usecase nào (demand forecasting, fraud detection, personalization, chatbot, predictive maintenance)? Build vs buy vs partner?

**Implication cho thesis:**
- Leader trong digital: thường có pricing power / cost advantage tăng dần, defensible moat
- Laggard: risk bị disrupt bởi tech-native competitor / fintech / e-commerce / aggregator
- Capex-heavy digital transformation: 2-3 năm earnings drag rồi mới thu lại lợi nhuận — cần kiểm tra ROI / payback period

**VN context note:** VN đang ở giai đoạn middle adoption — Banking và Retail dẫn dắt (fintech adoption ~60%, e-commerce penetration ~10-12%), còn Manufacturing và Construction lag. Cẩn trọng với firm "claim digital transformation" nhưng không có cost saving measurable.

## 5. Per-sector quick-reference

Cho 10-12 ngành whitelist có CFA cover direct. Mỗi ngành: 3-5 DD drivers + 3-5 MP factors + 2-4 SI trends + 4-6 PM KPIs + 2-3 ESG hotspots + VN-specific notes.

### 5.1. NGANHANG (Banking)

**DD drivers:**
- Credit growth (loan book) — sensitive với GDP, lãi suất, regulatory cap (LTV, room tín dụng)
- Net interest income (NII) — driven bởi NIM (Net Interest Margin) = (loan yield − funding cost). NIM cao khi CASA ratio cao (tiền gửi không kỳ hạn rẻ)
- Non-interest income (NoII) — fees & commission (bancassurance, cards, FX, advisory), trading income, recoveries
- Demand cho banking products: penetration rate (bank accounts/người trưởng thành, card penetration, mortgage/GDP, consumer loan/GDP)

**MP factors:**
- Market share: theo loan book / deposit / total asset / payment volume
- Funding mix: CASA ratio (cao = competitive funding cost), wholesale vs retail funding
- Distribution: branch network + digital adoption (% transaction qua digital)
- Niche positioning: corporate-led, SME-led, retail-led, hay universal?

**SI trends:**
- Digital banking & fintech disruption (eKYC, neobanks, BNPL)
- Regulatory: Basel III/IV adoption, NPL classification tightening, foreign ownership cap
- Cashless payment growth (e-wallet, QR, contactless)
- Demographic: working-age population peak rồi sẽ giảm, tăng tài sản middle class

**PM KPIs:**
- NIM (Net Interest Margin) — trend + so peers
- CIR (Cost-Income Ratio) — efficiency
- NPL ratio + LLR (Loan Loss Reserve coverage) + cost of credit
- CAR (Capital Adequacy Ratio) Tier 1 + Total, LCR (Liquidity Coverage Ratio)
- ROE / ROA, RoTE
- Loan growth vs deposit growth vs total asset growth

**ESG hotspots:**
- Financing controversial sectors (fossil fuel, coal, deforestation-linked, weapons)
- AML/CFT compliance, sanctions screening
- Data privacy + cybersecurity breach risk
- Predatory lending / vulnerable customer treatment

**VN-specific notes:**
- 4 SOCB (Vietcombank, BIDV, VietinBank, Agribank) chiếm ~50% market share; còn lại private + foreign
- Room tín dụng SBV cấp annually + theo health rating — không phải tự do tăng credit
- NPL DB (`bank_metrics`?) chú ý: NPL 3+ theo CIC classification, vs Group 2-5 theo SBV — definition khác nhau
- Bancassurance commission ratio đang siết theo Circular 50/2023, ảnh hưởng NoII

### 5.2. BAOHIEM (Insurance — override mode)

**Note:** Ngoài whitelist 18 default, nhưng vẫn quan trọng khi user override. Tham chiếu mục `K_agent_db_00` 4.5 override mode.

**DD drivers:**
- Insurance penetration (premium/GDP) + density (premium/capita) — VN ~3% GDP, thấp hơn ASEAN ~5%
- 3 segment: Life (mảng linh hồn), Non-life (xe, sức khỏe, tài sản), Reinsurance
- Affordability: cost/income ratio
- Channel mix: agency, bancassurance, broker, direct (online)

**MP factors:**
- Market share theo premium written / sum assured / policy count
- Sales channel concentration (đặc biệt bancassurance — agreement với bank nào)
- Product portfolio width
- Brand trust + claims settlement track record

**SI trends:**
- Insurtech disruption (online direct, microinsurance, P2P insurance)
- Regulatory: Bancassurance commission siết (Circular 50/2023 VN), solvency II adoption global
- Ageing → tăng demand life + health
- Climate change → tăng demand non-life property nhưng cũng tăng claim severity

**PM KPIs:**
- Annual Premium Equivalent (APE) — new business volume
- Combined Ratio (Non-life) = Loss Ratio + Expense Ratio. <100% là underwriting profit
- Persistency rate (Life) — % policies renewed
- Embedded Value (EV) — present value future profit của portfolio
- Solvency margin
- Investment yield trên reserves

**ESG hotspots:**
- Mis-selling / inappropriate product complaint (đặc biệt life + bancassurance)
- ESG screening trong investment portfolio (large institutional money)
- Climate risk underwriting

**VN-specific notes:**
- 2023 bancassurance crisis: scandal ép khách bank mua bảo hiểm → siết regulation
- Life market dominated bởi foreign (Prudential, AIA, Manulife, Dai-ichi); Non-life dominated bởi nội (Bao Viet, PVI, PJICO)

### 5.3. TIENICH (Utilities — Electricity / Water / Gas)

**DD drivers:**
- Per capita consumption — VN tăng ~7-9%/năm theo GDP
- Customer mix: residential (low margin, regulated), industrial (volume cao, sensitive với GDP), commercial
- Seasonality: summer peak điện, mùa khô peak nước
- Regulatory tariff structure (cross-subsidy industrial → residential)

**MP factors:**
- Natural monopoly characteristics (transmission, distribution)
- License area exclusivity
- Generation mix (coal, hydro, gas, solar, wind, nuclear) — cost competitiveness
- Vertical integration: generation + transmission + distribution + retail (unbundled hay bundled)

**SI trends:**
- Energy transition: coal phase-out, renewables tăng (VN PDP8 plan)
- Distributed generation (rooftop solar) đe doạ central utility model
- Smart grid + smart meter rollout
- Carbon pricing / emission cap upcoming

**PM KPIs:**
- Capacity utilization rate (CF — capacity factor)
- AT&C losses (Aggregate Technical & Commercial losses) — đặc biệt EVN-related
- Allowed Rate of Return (ARoR) — regulated return mỗi rate case
- Regulated Capital Value (RCV) — asset base recoverable through tariff
- Net debt/RCV (gearing benchmark từ regulator)
- Operating cash flow stability (key cho dividend yield play)

**ESG hotspots:**
- Carbon footprint per kWh (coal heavy = high), water consumption per kWh, ash disposal
- Health hazard near generation plants (PM2.5, mercury)
- Hydropower: displacement của community, biodiversity impact
- Nuclear: long-term radioactive waste

**VN-specific notes:**
- EVN gần như monopoly dọc, gọi thầu IPP (Independent Power Producer) cho generation
- PPA (Power Purchase Agreement) thường có FIT (Feed-in Tariff) — solar FIT 9.35 cent/kWh từng tạo boom 2019-2020
- Power tariff regulated bởi MOIT, retail tariff residential cross-subsidy bởi industrial

### 5.4. BDS (Property Development) + KCN (Industrial Real Estate / REIT)

#### BDS — Residential & Commercial Property Development

**DD drivers:**
- Demand residential: household formation + income growth + financing accessibility + investment demand
- Demand commercial: office vacancy + capex spending corporate + foreign company expansion (đặc biệt KCN nearby)
- Pricing power: location (vị trí), brand, product positioning (affordable / mid / luxury)
- Sales velocity: time-to-sell một dự án

**MP factors:**
- Land bank size + location quality
- Project execution track record + brand
- Distribution: sales channel (own salesforce, distributor, MGA)
- Financing access (bank relationship, bond issuance capability)

**SI trends:**
- Urbanization rate VN ~37% (thấp hơn ASEAN ~50%) → còn nhiều dư địa
- Demographics: household formation + first-time buyer demand
- Regulatory tightening: Circular 22 LTV cap, real estate lending cap, second-home tax proposal
- Speculation bubble risk + government cooling measures

**PM KPIs:**
- Pre-sales / contract sales — leading indicator
- Inventory days (đặc biệt unsold inventory) — overhang risk
- Sell-through rate per launch
- Land bank value (RNAV — Revalued Net Asset Value vs market cap)
- Net debt / total equity (cao = stressed cycle)
- Operating cash flow / contracted sales

**ESG hotspots:**
- Land acquisition controversies (forced eviction, undervaluation)
- Construction labor safety (rate fatality)
- Sustainability of design (energy efficiency, LEED)
- Greenfield (chiếm forest/farm) vs brownfield (urban renewal)

**VN-specific notes:**
- "Hệ Vingroup", "Hệ Novaland", "Hệ Sungroup" — cẩn trọng concentration risk + related-party transactions
- Bond default crisis 2022-2023 (Tan Hoang Minh, Van Thinh Phat) — credit risk cao
- Lift LTV và stricter classification 2023-2024 → tighter financing

#### KCN — Industrial Real Estate (cận REIT)

**DD drivers:**
- FDI inflow VN (Korea, Japan, China supply chain shift, Taiwan tech, US semiconductor) → demand factory + warehouse
- Manufacturing GDP growth
- Logistics infrastructure (port, highway, airport) gần KCN

**MP factors:**
- Available land bank trong/gần các tỉnh KCN strategic (Bắc Ninh, Bình Dương, Long An, Hải Phòng)
- Existing tenant mix + occupancy rate
- Infrastructure quality (điện, nước, ETP, port access)

**SI trends:**
- China+1 / Friend-shoring tăng FDI manufacturing VN
- Logistics warehouse demand từ e-commerce (Lazada, Shopee, Tiki) tăng nhanh
- ESG-compliant industrial park (green KCN) bắt đầu thành differentiator

**PM KPIs:**
- Occupancy rate
- Land bank remaining (years of sales)
- Rental rate per sqm + trend
- Lease tenor (typical 30-50 năm KCN VN)
- NOI (Net Operating Income) margin
- WALE (Weighted Average Lease Expiry)

**ESG hotspots:**
- Effluent treatment (industrial waste water)
- Tenant ESG compliance (firm có cho phép factory pollution không?)
- Land use change (farm → industrial — compensation, displacement)

**VN-specific notes:**
- KBC, SZC, IDC, BCM, VGC, GVR, PHR là các tên lớn
- Lease cycle: sales mảng đất 50 năm — revenue lumpy theo timing booking

### 5.5. BANLE (Retail)

**DD drivers:**
- Per capita disposable income + propensity to consume
- Modern trade penetration (VN ~25%, thấp hơn ASEAN, dư địa lớn)
- Channel shift: traditional → modern → online
- Discretionary vs non-discretionary mix

**MP factors:**
- Store network density (sqm/capita trong target city)
- Private label penetration (margin booster)
- E-commerce capability + omnichannel
- Supplier relationship (bargaining power, slotting fee)

**SI trends:**
- E-commerce penetration tăng nhanh (VN ~10-12%, target 25% vào 2030)
- Live-streaming commerce (TikTok Shop, Shopee Live)
- D2C model bypass traditional retailers
- Inflation pressure → consumer trade-down

**PM KPIs:**
- Same-store sales growth (SSSG) — like-for-like organic
- Store count net additions + new store productivity ramp
- Inventory days + cash conversion cycle
- Gross margin (private label boost) + EBITDA margin
- Capex / new store
- Revenue per sqm + revenue per employee

**ESG hotspots:**
- Supply chain labor practices
- Packaging waste (single-use plastic)
- Food safety (đặc biệt fresh + grocery)
- Worker turnover + low wages

**VN-specific notes:**
- MWG, FRT, PNJ là leader theo segment khác nhau (ICT, pharmacy, jewelry)
- Modern grocery: MWG Bach Hoa Xanh, MSN WinCommerce — Long Châu pharmacy đứng đầu
- Retailer + landlord crossover: VRE (Vincom Retail), HBC

### 5.6. VANTAI (Transportation — Logistics + Aviation + Marine)

**DD drivers:**
- Trade volume (export-import) + GDP growth
- Tourism (aviation passenger)
- E-commerce parcel volume (last-mile)
- Manufacturing relocation → tăng intra-Asia trade

**MP factors:**
- Network / route coverage
- Asset utilization (fleet, vessel, truck)
- Hub strategic (Tân Sơn Nhất, Nội Bài, Cái Mép port)
- Cost competitive (fuel hedging, fleet age)

**SI trends:**
- Container shipping cycle (2-3 năm boom/bust)
- Aviation: low-cost carrier penetration, regional vs international mix
- Logistics tech (TMS, route optimization)
- Carbon pricing on aviation + maritime upcoming (IMO 2050)

**PM KPIs:**
- Load factor (aviation, container), capacity utilization
- ASK / RPK (aviation), TEU throughput (port + container line)
- Yield per passenger-km / per TEU
- Fuel cost as % revenue
- Net debt / EBITDA (capital intensive)

**ESG hotspots:**
- Fuel emission + carbon pricing exposure
- Aging fleet → higher emission + maintenance
- Worker safety (truck driver, pilot fatigue)

**VN-specific notes:**
- HVN (Vietnam Airlines) bị Covid hit nặng — recovery cycle play
- VSC, GMD là port operator chính
- VTP (Viettel Post) là last-mile leader
- Shipping container VN bị phụ thuộc carrier nước ngoài

### 5.7. CONGNGHE (Technology — Software / IT services / Telecom-adjacent)

**DD drivers:**
- IT spending corporate VN (penetration thấp so peer)
- Outsourcing offshore demand (US/EU/Japan client)
- Cloud adoption + digital transformation budget enterprise
- Telecom-adjacent: 5G rollout, fintech infrastructure

**MP factors:**
- Talent pool (recruit + retain dev engineer)
- Customer concentration (offshore client list)
- Certifications + capability (CMMI, partnerships AWS/Azure/GCP)
- IP / proprietary technology

**SI trends:**
- AI/ML demand exploding — firm có competitive AI capability?
- Talent war + wage inflation
- Geopolitical: US-China tech decoupling tạo opportunity VN
- Cybersecurity demand tăng

**PM KPIs:**
- Revenue growth (organic, in USD if offshore)
- EBITDA margin trend
- Headcount + revenue per employee
- Attrition rate
- Bench utilization
- Order book / book-to-bill (project-based)

**ESG hotspots:**
- Talent labor practices (overtime, wellness)
- Data privacy / customer confidentiality
- Energy consumption data center (nếu có)

**VN-specific notes:**
- FPT là leader rõ ràng (IT services + software, retail FRT spinoff)
- CMG, ELC, HPT smaller
- Telecom operator (VGI, VTP từ Viettel group) — tách biệt với pure IT

### 5.8. XAYDUNG (Construction & Engineering)

**DD drivers:**
- Public infrastructure capex (highway, airport, metro, port) — government plan
- Private real estate development (residential + commercial)
- FDI manufacturing → KCN/factory build
- Backlog rollover

**MP factors:**
- Contractor tier (Tier 1 đấu thầu được dự án lớn, Tier 2-3 subcontract)
- License + experience track record
- Working capital capacity (project funding gap)
- Equipment fleet age + quality

**SI trends:**
- Infrastructure push (HSR North-South, expressway, airport expansion)
- Green building / LEED demand
- Modular / prefab construction
- Material price volatility (steel, cement)

**PM KPIs:**
- Order book size + backlog/revenue ratio
- Book-to-bill (>1 = backlog growing)
- Gross margin per project type (infrastructure vs residential vs industrial)
- Working capital intensity (cash flow from operations vs revenue)
- Days sales outstanding (delayed payment risk đặc biệt government client)

**ESG hotspots:**
- Worker safety (fatality, LTIFR cao trong ngành)
- Corruption / kickback (đặc biệt government project)
- Environmental impact construction (dust, noise, waste)

**VN-specific notes:**
- CTD (Coteccons), HBC (Hoa Binh) là Tier 1 residential
- CC1, VCG, LCG là contractor infrastructure
- Payment delay từ developer client + government luôn là vấn đề working capital

### 5.9. THUCPHAM + THUYSAN (Food Producers + Seafood)

**DD drivers:**
- Per capita food spending (Engel's law: food share giảm khi income tăng, nhưng absolute spending tăng)
- Shift sang processed / packaged / convenience food
- Export demand (seafood) — US/EU/Japan/China
- Health/wellness trend → tăng demand healthy food, giảm soda/sugar

**MP factors:**
- Brand power (consumer goods)
- Distribution network (modern trade + traditional + online)
- Vertical integration (farm-to-fork)
- Raw material sourcing (input price hedge)

**SI trends:**
- Sugar tax / fat tax mandates upcoming
- Plant-based, organic, no-GMO trend
- Climate change → fish stock pressure (seafood)
- E-commerce grocery channel growth

**PM KPIs:**
- Same-store volume growth + price growth split
- Gross margin (input cost pass-through)
- Inventory turnover
- Export/domestic mix (FX exposure)
- Distribution cost/revenue

**ESG hotspots:**
- Sustainable sourcing (palm oil deforestation, fish bycatch, seafood IUU)
- Plastic packaging waste
- Labor practices upstream supply chain
- Food safety + recall history

**VN-specific notes:**
- VNM (Vinamilk), MSN (Masan), KDC, QNS (đường), SBT (đường)
- Seafood: VHC (Vinh Hoan), MPC (Minh Phu), ANV, FMC — export-driven, FX và anti-dumping tariff risk
- Sugar industry consolidation đang diễn ra (TTC, SBT, QNS)

### 5.10. NONGNGHIEP (Agriculture — Crops, Plantation, Livestock)

**DD drivers:**
- Food consumption growth domestic + export
- Commodity price (cao su, cà phê, gạo, hồ tiêu, điều)
- Yield improvement potential (still low vs region cho nhiều crop)

**MP factors:**
- Land bank size + quality (đặc biệt cao su, cà phê — perennial crop có cycle 7-25 năm)
- Vertical integration (sản xuất + chế biến + xuất khẩu)
- Buyer concentration (xuất khẩu ai mua?)

**SI trends:**
- Climate change → crop yield volatility tăng
- Carbon credit cho rừng / cao su sustainable
- Vertical farming + tech (limited VN adoption)
- ESG screening commodities (đặc biệt từ EU)

**PM KPIs:**
- Yield per hectare
- Mature/immature land mix (cycle position)
- Cost per tonne sản xuất
- ASP (Average Selling Price) trend
- Inventory cycle (đặc biệt cao su tồn kho theo cycle giá)

**ESG hotspots:**
- Deforestation (cao su, cà phê expansion)
- Pesticide / chemical residue
- Labor: smallholder fairness, indigenous community
- Water usage (rice especially)

**VN-specific notes:**
- DPR, PHR, GVR là plantation cao su lớn
- HAG (Hoang Anh Gia Lai) farm sang Lào/Cambodia
- Cà phê: dominated bởi tư nhân + cooperative, ít listed pure-play

### 5.11. CHUNGKHOAN (Securities — Brokerage + IB)

**DD drivers:**
- Stock market turnover (ADV — Average Daily Value)
- New investor account opening rate
- Margin lending balance growth
- IPO + corporate bond issuance pipeline

**MP factors:**
- Market share theo brokerage volume
- Margin book size + risk management
- IB league table position
- Retail vs institutional client mix

**SI trends:**
- Retail investor boom 2020-2021, normalize 2022-onwards
- Tech disruption (mobile-first broker, robo-advisory)
- Derivative product expansion (covered warrants, futures)
- Foreign ownership cap upcoming changes

**PM KPIs:**
- Brokerage market share + commission per trade
- Margin book / equity (leverage)
- Margin yield + cost of funding
- IB fee income mix
- CIR (Cost-Income Ratio)
- VaR (Value at Risk) cho prop book

**ESG hotspots:**
- Mis-selling / unsuitable product to retail
- Conflict of interest (research vs IB)
- Cybersecurity + customer data
- Market manipulation enforcement track

**VN-specific notes:**
- SSI, VCI, HCM, SHS, VND, MBS là top theo market share
- Margin lending là driver lợi nhuận chính khi market tăng
- Foreign broker (Mirae Asset, KIS) cạnh tranh fee aggressive

### 5.12. DAUKHI (Oil & Gas — partial coverage via Offshore Marine)

**Note:** CFA framework không có direct cover cho integrated O&G, nhưng có Shipbuilding & Offshore Marine cho upstream services. Khung dưới mượn analogue từ commodity producer general.

**DD drivers:**
- Oil price (Brent/WTI) cycle
- VN domestic gas demand (power generation, petrochemical)
- E&P spending của PVN / PetroVietnam group
- Downstream petrochemical margin

**MP factors:**
- PVN ownership / partnership (sub of PVN có monopoly status nhiều mảng)
- Reserve life (P1/P2 reserve / annual production)
- Asset quality (cost per barrel produced)

**SI trends:**
- Energy transition risk (long-term demand cap)
- Decarbonization mandate
- New field discovery declining trong VN, dependency on PSC (Production Sharing Contract)
- LNG terminal upcoming → tăng demand gas

**PM KPIs:**
- Production volume (kbbl/d, mmscf/d)
- Realized price / Brent gap
- Lifting cost per barrel
- F&D cost (Finding & Development)
- Reserve replacement ratio
- FCF generation

**ESG hotspots:**
- Carbon emission (scope 1+2+3)
- Spill / pollution track record
- Stranded asset risk long-term
- Community impact near offshore platform

**VN-specific notes:**
- PVD (drilling), PVS (subsea services), PVT (transport), GAS, BSR (refining), PLX (distribution)
- PVN (parent) controlled — related-party transaction nhiều
- Block ngoài khơi Đông Nam (Lan Tây, Lan Đỏ) declining; Block 06.1 + 11.2 (Sao Vàng Đại Nguyệt) là new growth

## 6. Sectors không cover trực tiếp — guidance

Cho 6 ngành whitelist không có CFA cover (HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, CONGNGHIEP) + 1 ngành đã partially cover trên (DAUKHI), áp dụng universal framework + mượn analogue ngành tương cận:

| Ngành VN | Analogue mượn | Drivers cốt lõi cần check |
|---|---|---|
| HOACHAT (Chemicals) | Food Producers (input) + Oil & Gas (feedstock) | Capacity utilization, ASP-feedstock spread, capacity addition vs demand, environmental compliance |
| KIMLOAI (Metals — thép, kẽm, đồng, nhôm) | Commodity producer general | Spot price cycle, scrap vs ore route mix, energy cost, dumping tariff risk export, carbon emission |
| DETMAY (Garment & Textile) | Light manufacturing | Order book (US/EU client), labor cost competitive vs Bangladesh/India/Cambodia, FTA utilization (EVFTA, CPTPP), sustainability cert (BCI, GOTS) |
| KHOANGSAN (Mining) | Commodity producer + Oil & Gas | Reserve life, extraction cost, environmental damage (rất cao!), royalty/tax regime, social license |
| CONGNGHIEP (Industrial — đa dạng, cross-cutting) | Tùy sub-sector (electronics, machinery, automotive parts) | Khó generic — nên break down sang sub-sector cụ thể trước khi apply lens |

**Rule:** Khi gặp ngành ngoài cover, default về universal framework mục 3 + Industry 4.0 lens mục 4. Chọn 3-5 câu most material từ DD/MP/SI/PM/ESG.

## 7. Cách dùng pack này

### 7.1. Khi P_invest_memo deep-dive (Tier 5C memo 7 phần)

Trong phần "Business" của memo (1 trong 7 phần), bổ sung:
- 1 đoạn DD (3-5 dòng): demand drivers chính + sensitivity
- 1 đoạn MP (3-5 dòng): vị thế firm trong ngành, sustainable advantage
- 1 đoạn SI (2-3 dòng): 1-2 structural trend material nhất 3-5 năm tới
- PM: đã có trong phần "Financial" — không duplicate, chỉ note sector-specific KPI cần highlight
- 1 đoạn ESG (2-3 dòng): 1-2 hotspot có thể trigger downgrade conviction

Pull từ mục 5 per-sector quick-ref nếu ngành có cover; ngược lại từ universal framework mục 3.

### 7.2. Khi P_vbse_strategy Trục 4 Sector allocation

Mỗi sector tilt trong table 18 ngành whitelist, bổ sung 1 dòng "Structural lens" (3-5 từ) gọi tên SI driver dominant. Vd:
- NGANHANG: "Digital banking + credit growth recovery"
- BDS: "Affordability cycle + financing tightening"
- BANLE: "Modern trade penetration + e-commerce share gain"

Nguồn: mục 5.X SI bullets cho từng ngành.

### 7.3. Khi P_weekly_overview Phần 6 (Biến động 18 ngành)

**Default:** KHÔNG dùng pack này (weekly broadcast cần ngắn gọn 9-11 trang).

**Exception:** Khi 1 ngành có biến động bất thường tuần này (vd dòng tiền tuần top 1 hoặc bottom 1, hoặc có catalyst lớn) → có thể compose 1-2 dòng "Structural watch" từ SI hoặc ESG bullets của pack này, không quá 1 dòng/ngành.

### 7.4. Khi P_stock_report (single-stock deep analysis)

Pack này được pull mạnh nhất từ `P_stock_report` — qua 2 điểm:

**Điểm A: Stage 1j (Sector context) — `P_stock_report_01` mục 3.10**

Pull từ mục 5.X per-sector quick-ref nếu ngành có cover; ngược lại từ universal framework mục 3:
- Quick mode: skip 1j (đảm bảo output 1-2 trang)
- Standard mode: pull 3-5 câu most material → output 0.5 trang Phần 3
- Deep mode: pull đầy đủ 5 dimension → output 1-1.5 trang Phần 3

**Điểm B: Phần 2 Sub-type context — `P_stock_report_02` mục 7**

Trong Phần 2 (Phân tích doanh nghiệp theo type), pack `P_stock_report_02` đã có template cho 4 type (SXKD/NH/CK/BH). Nhưng để đào sâu sub-type (vd SXKD-Cyclical vs SXKD-Consumer/Defensive vs SXKD-Growth/Infrastructure), pack pull thêm context ngành từ pack này:
- Vd: mã ngành NGANHANG → pull mục 5.1 (NIM drivers, asset quality, capital position context bổ sung)
- Vd: mã ngành BDS → pull mục 5.4 (demand residential, structural urbanization trend)
- Vd: mã ngành NONGNGHIEP → pull mục 5.10 (yield + commodity price + climate exposure)

**Điểm C: Phần 2 Chuỗi giá trị (Value chain) — `P_stock_report_02` mục 2.6 (MANDATORY SXKD)**

Pack `P_stock_report_02` mục 2.6 áp dụng **6 framework chuẩn quốc tế** + reference K_sector_framework:

1. **Porter Value Chain (1985)** — 5 primary + 4 support activities (`P_stock_report_02` mục 2.6.2)
2. **Porter 5 Forces (1979)** — supplier/buyer/substitute/entrant/rivalry (mục 2.6.3)
3. **Smile Curve (Stan Shih 1992)** — vị trí capture giá trị (mục 2.6.4) — đặc biệt quan trọng cho VN context (nhiều SXKD VN ở smile bottom)
4. **GVC governance (Gereffi 2005)** — market/modular/relational/captive/hierarchy (mục 2.6.3)
5. **Industry 4.0 / Digital footprint** (CFA Sector Analysis 2020 Prelude) — Three Golden Steps + 7 chiều digital readiness (mục 2.6.5)
6. **CFA Sector Analysis 2020** — 21 industry chapters mapping với 18 ngành VN whitelist (mục 2.6.9)

K_sector_framework cung cấp **VN nuance** bổ sung cho framework global. Pull từ pack này khi áp dụng vào value chain:

- Mục 5.X tương ứng ngành — barriers to entry (mục 1.2), bargaining power (mục 1.6), competitive advantage (mục 1.3) feed vào 5 forces matrix
- SI (structural influences) per ngành → input cho Smile Curve migration analysis (vị trí firm có đang leo smile theo trend ngành không)
- MP (market position) per ngành → input cho Industry 4.0 competitive landscape

**Ví dụ industry value chain map cho 3 ngành tiêu biểu:**

- Vd ngành **thép (KIMLOAI):**
  - Industry value chain: upstream quặng sắt + than cốc (import phụ thuộc) → midstream phôi/HRC (capital-intensive, top 4 = 75% share) → downstream tôn/thép xây dựng (fragmented distribution)
  - Smile Curve: midstream commodity = smile bottom; downstream branded steel + customer service = smile top (ít doanh nghiệp VN ở zone này)
  - CFA chapter: Engineering & Construction (chapter 10) — selective application

- Vd ngành **F&B đồ uống (THUCPHAM):**
  - Industry value chain: upstream nguyên liệu sữa/đường (commodity volatility, mục 5.9) → midstream chế biến (brand-driven, mass customisation potential) → downstream phân phối GT/MT (NPP layer 1-2, mục 5.5 BANLE overlap)
  - Smile Curve: midstream + downstream với own brand = smile top (VNM, MSN); pure manufacturing without brand = smile bottom
  - CFA chapter: Food Producers (chapter 8) — đầy đủ framework

- Vd ngành **dệt may (DETMAY):**
  - Industry value chain: upstream bông/sợi (FX exposure, phần lớn nhập China) → midstream CMT/FOB (low margin, captive GVC) → downstream brand owner global captures most margin
  - Smile Curve: CMT = smile bottom rõ rệt; FOB cao hơn; ODM + own brand = smile top (TNG là 1 trong số ít)
  - CFA chapter: Luxury Products (chapter 6) — selective downstream + Food Producers (chapter 8) supply chain

Nếu industry chưa có cover trong mục 5.X → dùng universal framework mục 1.6 (bargaining power) + 1.2 (barriers to entry) làm khung phân tích chuỗi giá trị generic, kết hợp với CFA Sector Analysis chapter tương ứng.

**Output format khi P_stock_report pull pack này:**
- Phần 3 báo cáo stock_report đặt heading "Bối cảnh ngành & vĩ mô" với 3 sub-section:
  - Cấu trúc ngành (1 đoạn DD/MP)
  - Xu hướng dài hạn 3-5 năm (1 đoạn SI)
  - ESG context (Deep mode đầy đủ)
- Apply K hygiene mạnh: KHÔNG lộ "DD/MP/SI/PM/ESG" raw — viết tự nhiên (xem mục 8 output contract dưới)

### 7.5. Khi standalone deep-dive ngành

User hỏi "phân tích sâu ngành X" hoặc "outlook ngành X 12 tháng tới" → compose full 5 dimension brief theo template:
1. Demand Drivers (1 paragraph)
2. Market Position + Competition (1 paragraph)
3. Structural Influences 3-5 năm (1 paragraph)
4. Performance Metrics ngành tracking (1 bullet list)
5. ESG hotspots + risks (1 paragraph)
6. Industry 4.0 implication (1 paragraph nếu material)

Length tổng: 1-2 trang. Mỗi dimension pull 3-5 câu most material từ universal framework + per-sector quick-ref.

## 8. Output contract

Pack này là **reference layer**, không sinh deliverable riêng. Khi P pack pull content từ pack này:

- **Phải apply K hygiene** (`K_agent_db_00` mục 5) — không lộ ký hiệu "DD/MP/SI/PM/ESG" ra output user-facing. Viết bằng ngôn ngữ tự nhiên: "Drivers chính của ngành là...", "Vị thế firm trong ngành...", "Trend dài hạn 3-5 năm...".
- **Phải gắn citation** khi đưa số liệu định lượng: nguồn DB (mục 4 citation contract trong `K_agent_db_00`), web report cho data ngoài DB.
- **Phải VN-context override** khi CFA framework conflict với thực tế VN — note rõ trong phần phân tích.
- **Conviction / horizon / disconfirming** vẫn theo rule cross-pack (mỗi call có 3 yếu tố này — chuẩn institutional). Pack này không thay thế rule đó.
- **Không command words** — observation only (như mọi P pack).
