# P_stock_report_04 — Self-audit + Edge cases + Failure modes + Output contract

File này quy định self-audit checklist trước khi finalize, edge cases handling, failure modes thường gặp, và output contract chi tiết.

## 1. Self-audit checklist trước khi finalize

Agent **bắt buộc** chạy self-audit này sau Stage 2 Bước 3 (compose draft) và trước Checkpoint 1.

### 1.1. Data quality audit (10 điểm)

- [ ] **BCTC PDF đã forensic 15-point checklist** (Stage 1i)? Audit opinion check? Going concern?
- [ ] **Thuyết minh BCTC** đã đào sâu các phần material (related party, contingent, off-balance, segment, capex commitment)?
- [ ] **News DB pull** rolling 30-90 ngày (Stage 1g)? Có filter ticker + industry?
- [ ] **Web search news** đã chạy song song DB (Stage 1h)? Nguồn VN cho equity, EN cho macro nếu ngành cần?
- [ ] **Sell-side consensus** đã search (broker VN: SSI, VCSC, HSC, MBS...)?
- [ ] **Peer comparison** internet-first + thanh khoản filter (Stage 1l)? 3-5 peer đạt filter?
- [ ] **Macro variables relevant** ngành đã pull (Stage 1k)?
- [ ] **Earnings calendar** đã check (Stage 1n)?
- [ ] **ESG controversy** đã scan web search (Stage 1o)?
- [ ] **Audit trail metadata** đầy đủ data sources (DB collections, web queries, PDF files)?

### 1.2. Thesis quality audit (8 điểm)

- [ ] **Recommendation rõ ràng** (Long / Watch / Avoid, không phải "tích cực vừa", "có thể xem xét")?
- [ ] **Conviction tier** đã chốt (HIGH/MID/LOW)?
- [ ] **Horizon cụ thể** (1-3m / 3-6m / 6-12m)?
- [ ] **Thesis core 1-2 dòng** rõ ràng, không chung chung?
- [ ] **Variant Perception** đã apply rule per mode (Quick optional, Standard recommended với flag, Deep mandatory với auto downgrade)?
- [ ] **Top 2-3 catalyst** với **timing cụ thể** (ngày/quý)?
- [ ] **Bear case steelmanned** — không soft-pedal, trình bày như người tin Long phản đối?
- [ ] **2-3 disconfirming signal MEASURABLE** với threshold cụ thể (số / sự kiện)?

### 1.3. Type-specific audit (4 điểm)

- [ ] **Type classification đúng** (SXKD/NH/CK/BH) match với industry?
- [ ] **Phần 2 báo cáo dùng template tương ứng type** (xem `P_stock_report_02` mục 7)?
- [ ] **KPIs đã pull đúng theo type** (NH: NIM/CASA/CAR/NPL; CK: market share/margin/IB; BH: combined ratio/APE; SXKD: ROE/margins/working capital)?
- [ ] **Bear case typical theo type** đã consider?

### 1.3b. Value chain audit — SXKD only (10 điểm, SKIP cho NH/CK/BH)

**Porter framework (4 điểm):**
- [ ] **Chuỗi giá trị ngành** đã map (upstream → midstream → downstream + margin pool note + bottleneck)? (`P_stock_report_03` mục 2.2 sub-section 3a)
- [ ] **Porter primary activities (5)** đã cover (inbound, operations, outbound, marketing, service)? (sub-section 3b)
- [ ] **Porter support activities (4)** đã cover (firm infrastructure, HR, technology dev, procurement) — đặc biệt **R&D / Revenue ratio**? (sub-section 3b)
- [ ] **Vertical integration** đã phân định hướng (backward / forward / full / conglomerate)? (sub-section 3b)

**Competitive analysis (3 điểm):**
- [ ] **Porter 5 forces** đã có bảng (Standard+) hoặc note (Quick)? (sub-section 3c)
- [ ] **GVC governance type** đã verdict (market / modular / relational / captive / hierarchy)? (sub-section 3c, reference Gereffi 2005)
- [ ] **Tier supplier position** đã note (Tier 1 own brand / Tier 1 modular / Tier 2 captive / Tier 3 commodity)? (sub-section 3c)

**Smile Curve + Industry 4.0 (2 điểm):**
- [ ] **Smile Curve position** đã verdict (smile bottom / mid / top + migration strategy)? (sub-section 3d, reference Stan Shih 1992) — **đặc biệt quan trọng cho VN context**
- [ ] **Industry 4.0 readiness** đã verdict (Leader / Par / Laggard) qua 7 chiều? (sub-section 3e, reference CFA Sector Analysis 2020)

**Synthesis + data (1 điểm):**
- [ ] **Position summary** tích hợp tất cả lens trên + trả lời: firm captures margin ở mắt xích nào + Smile position + GVC governance + Industry 4.0 + risk + pricing power verdict tổng hợp? (sub-section 3f)
- [ ] **Data sub-step 1p** đã pull (top KH, top NCC, channel mix, geographic mix, capacity, nguyên liệu key, R&D ratio, Industry 4.0 readiness)? Nếu data gap → đã flag conviction downgrade?

**Reference professional standards check:**
- [ ] Đã apply ít nhất **3 framework** trong: Porter Value Chain (1985), Porter 5 Forces (1979), Smile Curve (Stan Shih 1992), GVC governance (Gereffi 2005), Industry 4.0 (CFA Sector Analysis 2020)?
- [ ] Đã pull **CFA Sector Analysis chapter** tương ứng ngành (xem `P_stock_report_02` mục 2.6.9 mapping table)?

### 1.4. Strict reject pattern audit (3 điểm)

Các pattern strict reject (không flex+downgrade):

- [ ] **Dòng tiền dương + catalyst tiêu cực material** → strict reject Long, auto downgrade Watch? (`P_stock_report_03` mục 8)
- [ ] **Bear cumulative target < current price** → đây là red flag mạnh, conviction phải downgrade?
- [ ] **Audit opinion Qualified / Adverse / Disclaimer** hoặc **Going concern note** → flag bigaba (recommendation phải > MID + caveat rõ)?

### 1.5. K hygiene + Citation audit (5 điểm)

- [ ] **DB raw symbols đã dịch** (`vsi`, `day_score`, `week_score`, `zone`, percentile)?
- [ ] **Taxonomy nội bộ đã dịch** (Kịch bản A/B/C/E1/E2/E3, framework chấm điểm, HIGH/MID/LOW impact)?
- [ ] **Thuật ngữ EN đã dịch sang VN** (mean-reversion, exhaustion, dead-cat bounce, etc.) — đặc biệt audience KH?
- [ ] **Unit conversion** theo `K_agent_db_00` mục 6 (BCTC sang tỷ đồng; `*_pct` ĐÃ là điểm % — không nhân 100; ratio finstats + lãi suất `other_data` còn thập phân — nhân 100)?
- [ ] **Citation 4 nhóm** đầy đủ (Tổng hợp DB / finext.vn URL / PDF user upload / Web external)?

### 1.6. Audience awareness audit (3 điểm)

- [ ] Audience KH? Wording mềm (observation, không command)?
- [ ] Audience KH? KHÔNG render TP1/TP2/SL số cụ thể (chỉ "tín hiệu cần theo dõi")?
- [ ] Audience KH? Disclaimer chuẩn KH đầy đủ (xem `O_stock_report_00`)?

### 1.7. Flex+downgrade audit (2 điểm)

- [ ] Nếu data thiếu (BCTC chưa soát xét, peer < 3, etc.) → đã downgrade depth mode hoặc conviction tương ứng?
- [ ] Nếu Variant Perception thiếu ở Deep mode → đã auto downgrade conviction HIGH→MID / MID→LOW / LOW→Watch?

**Total self-audit:**
- SXKD: 47 điểm (gồm 12 điểm value chain audit 1.3b — Porter 4 + Competitive 3 + Smile/Industry 4.0 2 + Synthesis 1 + Professional standards 2). Pass ≥40/47.
- NH/CK/BH: 35 điểm (skip 1.3b). Pass ≥30/35.

Dưới ngưỡng pass → quay lại Bước 1 hoặc 2 Stage 2.

## 2. Edge cases

### 2.1. Conglomerate / Holding company

**Definition:** Doanh nghiệp có operations across multiple types/sub-types với segment chính chiếm <60% doanh thu.

**Ví dụ VN:**
- VIC (Vingroup): BĐS dominant (~50-60%) + manufacturing (VinFast) + consumer (Vinpearl, retail)
- MSN (Masan Group): FMCG (MasanConsumer) + Retail (WinCommerce) + Mining (Masan High-Tech) — truly diversified
- HDG (Hà Đô): Real estate + Hydropower + Renewable + Hotel — diversified
- HPX (Hải Phát): Real estate + Industrial real estate

**Handling rule:**

1. Pull BCTC consolidated + segment breakdown từ thuyết minh BCTC
2. **Identify primary segment** (>60% revenue hoặc >60% net profit):
   - Có primary segment → áp framework type tương ứng (SXKD/NH/CK/BH) + sub-type
   - Không có primary segment → fall back SXKD framework + render segment breakdown chi tiết
3. **Note rõ trong báo cáo:**
   - "Doanh nghiệp đa ngành, phân tích chính theo segment [X] chiếm [Y%] doanh thu"
   - Liệt kê segment khác như SECONDARY context (1-2 dòng/segment)
4. **Bear case riêng cho conglomerate:**
   - Conglomerate discount (sum-of-parts valuation < market cap segments riêng)
   - Capital allocation issues across segments
   - Cross-subsidy giữa segment yếu và segment mạnh
   - Governance complexity

### 2.2. Holding company thuần (parent only operations)

**Definition:** Holding công ty chỉ có operations trading + investment management, không direct operating (vd holding HDG parent only chỉ có equity investment vào subsidiary hydropower / BĐS).

**Handling rule:**

1. **Dùng BCTC consolidated cho phân tích** (reflect total value creation)
2. BCTC parent only chỉ dùng để check:
   - Dividend distribution capacity (parent có đủ retained earnings để chia cổ tức cho shareholder không)
   - Intercompany flow (loan parent → subsidiary, dividend subsidiary → parent)
3. Note: "Phân tích dựa trên BCTC hợp nhất. BCTC riêng lẻ chỉ tham chiếu cho capacity chia cổ tức"

### 2.3. Newly listed (< 2 năm)

**Definition:** Mã list trên HOSE/HNX < 24 tháng.

**Handling rule:**

1. Web search BCTN tiền-IPO (annual report private 2-3 năm trước listing) nếu có public
2. **Conviction CAP tại MID** (không cho phép HIGH cho newly listed):
   - Lý do: governance evolving, audit quality chưa proven multi-year, lock-up expiry risk
3. **Bear case typical bổ sung:**
   - Lock-up expiry → supply pressure
   - Insider selling sau lock-up
   - Earnings beat year 1 (window dressing) → revert mean year 2-3
   - Audit transition (đổi auditor sau listing)
4. Note: "Newly listed (list [DD/MM/YYYY], dưới 2 năm), history limited, conviction capped at MID, monitor lock-up expiry [DD/MM/YYYY]"

### 2.4. Mã suspended / Cảnh báo / Kiểm soát đặc biệt

**Definition:** Status đặc biệt từ HOSE/HNX:
- Cảnh báo (Warning)
- Kiểm soát (Control)
- Kiểm soát đặc biệt (Special control)
- Hạn chế giao dịch
- Tạm ngưng giao dịch (Suspended)
- Hủy niêm yết (Delisted)

**Handling rule:**

1. **Big warning prepend** đầu báo cáo:
   ```
   ⚠️ Mã [X] hiện ở trạng thái [cảnh báo/kiểm soát/etc.] từ [DD/MM/YYYY]. 
   Lý do: [chậm BCTC / lỗ liên tiếp / thiếu thông tin / kiểm toán từ chối / etc.]. 
   Phân tích này mang tính tham khảo, KHÔNG khuyến nghị giao dịch mới.
   ```
2. **Default recommendation: Avoid**. Long không khả thi trong status này.
3. Vẫn proceed analysis nếu user yêu cầu — không refuse, nhưng note rõ caveat lớn

### 2.5. Mã ngoài 18 whitelist (BAOHIEM, VLXD, NHUA, CAOSU, DULICH, YTEGD)

**Default rule** (`K_agent_db_00` mục 4.5): pack `K_agent_db` filter whitelist 18 ngành.

**Override mode khi user yêu cầu mã cụ thể:**
- Pack `P_stock_report` SỬ DỤNG override mode — phân tích bình thường
- Note đầu báo cáo: "Mã thuộc ngành [BAOHIEM/VLXD/...], ngoài scope whitelist mặc định 18 ngành. Phân tích theo override mode."
- Sector context (Stage 1j):
  - BAOHIEM: dùng `K_sector_framework` mục 5.2 (Insurance override)
  - VLXD, NHUA, CAOSU: dùng SXKD-Cyclical framework + universal mục 3 (không có CFA cover direct)
  - DULICH: dùng SXKD-Consumer framework + universal mục 3
  - YTEGD: pack `K_sector_framework` không có cover — dùng universal framework

### 2.6. Mã có related party complexity (hệ MSN, hệ Vingroup, etc.)

**Definition:** Mã thuộc "hệ" tập đoàn lớn với related party transactions material.

**Ví dụ:**
- "Hệ Vingroup": VIC, VHM, VRE, VEF, VPL
- "Hệ Masan": MSN, MCH (MasanConsumer Holdings), MML (Masan MeatLife)
- "Hệ Sungroup": (chưa list trực tiếp nhưng có 1-2 sub-listing)

**Handling rule:**

1. **Stage 1i forensic point 4 (Related party)** đào kỹ:
   - Tỷ trọng related party / total revenue
   - Tỷ trọng related party / total expense
   - Loại giao dịch chi tiết
2. **Pull thêm các mã cùng "hệ" làm peer compare** (Stage 1l):
   - VIC pair với VHM, VRE (concentration risk hệ)
3. **Bear case bổ sung:**
   - Concentration risk hệ tập đoàn (vd VPB exposure to VinGroup nếu có)
   - Governance independence concerns
   - Cross-subsidy risk
4. Note đầu Phần 6 (Bear case): "Thuộc hệ tập đoàn [X], cần cẩn trọng concentration risk + related party"

### 2.7. ETF / Quỹ / Mã không phải doanh nghiệp đơn

**Handling rule:** REFUSE phân tích — pack chỉ hỗ trợ single-stock equity. Đề xuất user:
- ETF VN30 → cần công cụ phân tích market-level riêng (ngoài scope pack)
- Quỹ mở / ETF ngành → đề xuất search sell-side reports + IR factsheet quỹ

### 2.8. Penny stock / Mã rất nhỏ

**Definition:** Market cap < 500 tỷ VND hoặc giá dưới 5.000 VND.

**Handling rule:**

1. **Big warning prepend:**
   ```
   ⚠️ Mã [X] thuộc nhóm small/micro cap (market cap [Y] tỷ VND, giá [Z]). 
   Thanh khoản hạn chế (ADV [W] tỷ), thông tin minh bạch có thể kém. 
   Phân tích này KHÔNG khuyến nghị giao dịch quy mô lớn.
   ```
2. **Conviction CAP at LOW** (không HIGH, không MID cho penny)
3. **Bear case mặc định nặng:**
   - Liquidity risk (single trade > 5% ADV gây slippage lớn)
   - Manipulation risk (low float dễ bị pump & dump)
   - Info quality risk (sell-side coverage = 0, BCTC chậm, audit quality)
4. **Default recommendation: Avoid hoặc Watch** (không Long).

### 2.9. Mã đa list (HOSE + HNX + UPCOM cùng lúc)

Hiếm — không phổ biến VN. Nếu gặp → pull data từ exchange chính (HOSE > HNX > UPCOM), note multi-listing.

### 2.10. Mã chuẩn bị huỷ niêm yết / chuyển sàn

**Handling rule:**
- Web search check status (vd "[Ticker] huỷ niêm yết / chuyển sàn")
- Nếu pending → big warning + default Avoid
- Nếu vừa chuyển sàn → conviction cap MID + monitor recurring report quality

## 3. Failure modes thường gặp + Cách tránh

### 3.1. Failure: BCTC PDF thiếu thuyết minh (chỉ có 3 báo cáo chính)

**Symptom:** User upload BCTC nhưng PDF chỉ có Bảng cân đối / Kết quả KD / Lưu chuyển tiền — không có Thuyết minh.

**Action:**
- REFUSE chạy + request: "PDF anh upload thiếu phần Thuyết minh BCTC — phần này quan trọng nhất cho forensic. Anh upload bản đầy đủ giúp em."
- Nếu user khẳng định "đây là bản đầy đủ rồi" → fail-soft với big caveat: "Phân tích này dựa trên BCTC summary, KHÔNG có forensic thuyết minh — conviction capped at LOW"

### 3.2. Failure: BCTC quá cũ (>6 tháng từ ngày kết quả)

**Symptom:** User upload BCTC Q4/2024 vào tháng 5/2026.

**Action:**
- Hỏi user: "BCTC anh upload là Q4/2024, đến hôm nay đã >12 tháng. Có BCTC gần hơn không?"
- Nếu không có → fail-soft với warning "BCTC reference quá cũ, conviction capped at LOW + caveat data freshness"

### 3.3. Failure: Thesis dựa chủ yếu vào định giá rẻ

**Symptom:** Phần 1 Recommendation Long với thesis chỉ là "P/E phân vị 15% lịch sử, rẻ".

**Cách tránh:**
- Self-audit câu hỏi "Nếu mã rẻ thế thì tại sao consensus chưa pick lên?" (red flag Value Trap)
- Phải có **catalyst forward-looking** + **earnings turnaround signal** ngoài "rẻ"
- Nếu chỉ rẻ → classification = Value Trap (xem `P_stock_report_02` mục 2.2), default Avoid

### 3.4. Failure: Bear case soft-pedaled

**Symptom:** Bear case ghi 1-2 dòng chung chung "rủi ro vĩ mô", không steelmanned.

**Cách tránh:**
- Bear case PHẢI steelmanned — trình bày như chính người tin Long phản đối
- Mỗi bear case có:
  - Mechanism (cơ chế xảy ra)
  - Impact magnitude (mức độ ảnh hưởng — số cụ thể nếu có)
  - Probability qualitative (đang play out, có khả năng cao, đuôi rủi ro)
- Self-audit: "Nếu Bear materialize, mã sẽ về vùng giá nào?" — phải answer được con số

### 3.5. Failure: Catalyst không có timing cụ thể

**Symptom:** Catalyst ghi "kỳ vọng cải thiện BCTC", "macro thuận lợi" — không có ngày/quý.

**Cách tránh:**
- Mỗi catalyst PHẢI có timing:
  - Ngày cụ thể (BCTC Q1 publish ~30/04/2026)
  - Quý (catalyst materializes Q3/2026)
  - Range (6-12 tháng tới)
- Nếu không có timing → xoá khỏi catalyst list, chuyển sang "long-term tailwind" trong Phần 3 (industry context)

### 3.6. Failure: Disconfirming signal mơ hồ

**Symptom:** Disconfirming ghi "NIM xấu đi", "kinh tế VN suy thoái".

**Cách tránh:**
- Disconfirming PHẢI measurable:
  - Số cụ thể: "NIM Q2 thu hẹp ≥ 20bp QoQ"
  - Threshold + duration: "Group 2 inflow tăng ≥ 30% QoQ trong 2 quý liên tiếp"
  - Sự kiện cụ thể: "SBV ban hành Circular siết classification loan trước Q3"
- Self-audit: "Nếu disconfirming materialize, em có hard exit không?" — yes = good, no = signal yếu

### 3.7. Failure: Variant Perception fake (claim but actually consensus)

**Symptom:** VP ghi "Em thấy mã sẽ outperform vì fundamentals tốt" — nhưng đó chính là consensus.

**Cách tránh:**
- VP PHẢI khác consensus có **cơ sở định lượng** (số liệu specific)
- Self-audit:
  - "Consensus đang nghĩ gì cụ thể về EPS / margin / valuation năm tới?" (search sell-side)
  - "Quan điểm em khác như thế nào?" (số liệu cụ thể)
  - "Cơ sở định lượng nào support quan điểm khác?" (data point từ Stage 1)
- Không có cơ sở định lượng → fake VP, agent flag "VP claim chưa có evidence"

### 3.8. Failure: Skip Phần 6 Bear case khi conviction HIGH

**Symptom:** Conviction HIGH → agent skip Phần 6 vì "mã tốt quá".

**Cách tránh:**
- **Phần 6 Bear case MANDATORY cho mọi recommendation** (kể cả HIGH conviction)
- HIGH conviction càng cần Bear case mạnh để demonstrate đã consider downside
- Self-audit: "Top 3 disconfirming nào sẽ làm em downgrade từ HIGH xuống MID?" — phải answer được

### 3.9. Failure: Auto-escalate Long khi data ambiguous

**Symptom:** Data mixed, một số positive, một số negative → agent vẫn render Long.

**Cách tránh:**
- Decision rule: Long requires **majority data point positive + bear case manageable + disconfirming measurable**
- Data mixed → default Watch (không Long)
- Specifically:
  - Dòng tiền + technical zone positive + earnings momentum positive + BCTC clean → Long candidate
  - Có bất kỳ data point material negative (forensic flag, NPL spike, audit qualified) → Watch hoặc Avoid

### 3.10. Failure: KH audience nhưng render technical raw

**Symptom:** Audience = KH nhưng output có "Bucket entry 2", "Kịch bản E1", "Pitfall F3", "VSI 2.5"

**Cách tránh:**
- Self-audit K hygiene checklist (mục 1.5) cho audience KH
- Specifically: dịch hết taxonomy nội bộ sang ngôn ngữ tự nhiên
- Wording mềm: "có thể xem xét theo dõi" thay vì "Watch", "vùng kỹ thuật tích cực" thay vì "zone A"

## 4. Output contract chi tiết

Pack này sinh ra **MD final** consume bởi user (audience nội bộ analyst hoặc KH).

### 4.1. Mandatory deliverable

1. **MD báo cáo** structure 6-7 phần (heading rigid theo `_03` mục 2)
2. **Metadata block** đầu báo cáo (frontmatter):
   ```yaml
   ---
   ticker: VNM
   company_name: Công ty Cổ phần Sữa Việt Nam
   industry: THUCPHAM
   type: SXKD
   sub_type: Consumer/Defensive
   report_date: 2026-05-30
   report_mode: standard  # quick / standard / deep
   audience: internal  # internal / client
   pair_compare: false  # true nếu pair mode
   horizon: 3-6m
   recommendation: Long
   conviction: HIGH
   ---
   ```
3. **Audit trail metadata** cuối báo cáo:
   - Deep mode: full block (xem `_01` mục 6)
   - Standard mode: rút gọn 3-5 dòng key
   - Quick mode: optional

### 4.2. Length contract

| Mode | Pages | Words approx |
|---|---|---|
| Quick | 1-2 | 500-1.000 |
| Standard | 3-5 | 1.500-2.500 |
| Deep | 5-10 | 2.500-5.000 |
| Pair (Standard) | 5-7 | 2.500-3.500 |
| Pair (Deep) | 8-12 | 4.000-6.000 |

Vượt range → flag self-audit "Length over budget, review để cô đọng".

### 4.3. Disclaimer

Cuối báo cáo bắt buộc có disclaimer. 2 mode:

**Disclaimer nội bộ (audience internal):**

```
---
Báo cáo này chỉ dùng nội bộ trong tổ chức. Số liệu lấy từ MongoDB `agent_db` 
(snapshot date [YYYY-MM-DD]) và các nguồn web đã ghi rõ. BCTC reference: [period].

Phân tích phản ánh quan điểm tại thời điểm publish, không phải dự báo. 
Diễn biến thực tế có thể khác. Quyết định đầu tư cuối cùng do người đọc cân nhắc 
trên tổng thể danh mục và khẩu vị rủi ro cá nhân.

Không bảo đảm lợi nhuận. Không tư vấn pháp lý / thuế / kế toán cá nhân.
```

**Disclaimer cho khách hàng (audience client):**

```
---
**Tuyên bố miễn trừ trách nhiệm**

Báo cáo này được cung cấp cho mục đích tham khảo, không phải tư vấn đầu tư cá nhân. 
Số liệu và phân tích dựa trên thông tin công khai sẵn có tại thời điểm phát hành 
([YYYY-MM-DD]). Báo cáo tài chính tham chiếu kỳ [period].

Quan điểm trong báo cáo phản ánh nhận định của bộ phận phân tích tại thời điểm 
publish và có thể thay đổi theo diễn biến thị trường mà không cần báo trước.

Quyết định đầu tư cần dựa trên đánh giá đầy đủ của quý anh/chị về tình hình tài chính, 
mục tiêu đầu tư, khẩu vị rủi ro, và khung thời gian phù hợp. Quá khứ không bảo đảm 
kết quả tương lai. Không có cam kết về lợi nhuận.

Để biết thêm chi tiết, vui lòng liên hệ với chuyên viên tư vấn.
```

### 4.4. Forward-looking statement (Deep mode + audience client)

Cuối Phần 1 hoặc đầu Phần 6, bổ sung forward-looking statement chuẩn institutional:

```
**Forward-looking statement:**
Quan điểm và kỳ vọng trong báo cáo này là "forward-looking" — phụ thuộc các giả định 
về tương lai có thể không thành hiện thực. Các yếu tố làm sai lệch kỳ vọng bao gồm 
(nhưng không giới hạn): biến động lãi suất, tỷ giá, giá hàng hoá, chính sách vĩ mô, 
sự kiện địa chính trị, dịch bệnh, thiên tai, và các yếu tố bất ngờ khác. 
Vui lòng xem mục Disconfirming signal để hiểu các tín hiệu sẽ làm thesis sai.
```

### 4.5. Render output channel

Output cuối render trong message Claude Desktop (theo project deployment hiện tại). User copy/save thủ công.

Render binary (pptx/docx) **out of scope** project hiện tại — nếu user cần presentation deck, dùng tool render bên ngoài (Canva, PowerPoint từ MD).

Theo `O_stock_report_00` chi tiết hơn về render spec (heading, table, metadata format).

### 4.6. File naming

```
Single mode:
stock_report_<TICKER>_<YYYYMMDD>_<mode>.md

Pair mode:
stock_report_<TICKER1>vs<TICKER2>_<YYYYMMDD>_pair_<mode>.md
stock_report_<TICKER1>vs<TICKER2>vs<TICKER3>_<YYYYMMDD>_pair_<mode>.md  (3 mã)
```

Ví dụ:
```
stock_report_VNM_20260530_standard.md
stock_report_HPG_20260530_deep.md
stock_report_VCBvsACB_20260530_pair_standard.md
```

## 5. Dependencies summary

| Pack | Quan hệ với P_stock_report |
|---|---|
| **K_agent_db** | Mandatory dependency — schema + query patterns + methodology + K hygiene |
| **K_sector_framework** | Recommended dependency — pull cho Phần 3 (industry context) + Phần 2 (sub-type context) |
| **O_stock_report** | Mandatory dependency ở Stage 3 — render spec |

## 6. Limitations + Honest disclosure

Pack có giới hạn cố hữu:

1. **Chỉ scope thị trường VN niêm yết** — không support UPCOM unlisted, không support cổ phiếu ngoài VN
2. **Long-only** — không có Short
3. **Horizon 1-12 tháng** — không phục vụ scalping intraday hoặc holding >5 năm
4. **BCTC PDF bắt buộc** — pack từ chối analysis nếu không có
5. **Conviction phụ thuộc data quality** — newly listed / penny / suspended có cap conviction
6. **Sector framework chỉ cover 12/18 ngành whitelist trực tiếp** — 6 ngành còn lại dùng universal framework (xem `K_sector_framework`)
7. **Web search có thể timeout / partial result** — agent note rõ trong audit trail
8. **Forensic depth phụ thuộc PDF quality** — PDF scan-only OCR có thể miss số liệu
9. **Pack độc lập với DCF chuyên sâu** — pack không cover valuation DCF chi tiết, chỉ valuation multiples + peer compare
10. **Audience giả định analyst nội bộ hoặc KH có background đầu tư** — không phục vụ retail thuần chưa có chứng chỉ
