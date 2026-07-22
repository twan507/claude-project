# KERNEL SKELETON

File index của project knowledge. Agent đọc file này đầu session để biết pack nào available và trigger nào activate pack nào. Chi tiết nội dung pack nằm trong file `_00` master của pack tương ứng, không duplicate vào đây.

## Cách dùng file này

1. Agent scan file này đầu session, cùng với `OUTPUT_MASTER.md` (xem mục "Global output convention" bên dưới).
2. Khi query user đến, match với trigger của pack để quyết định activate pack nào.
3. Activate pack thì đọc `_00` master của pack đó trước (bắt buộc theo rule master-first reading trong system prompt mục 5.7).
4. Nếu không match pack nào, fall back về Default inline hoặc Default report (xem system prompt mục 6).
5. Pack không có trong file này = không tồn tại. Agent không được suy diễn pack ngoài danh sách.
6. Khi render deliverable cuối, áp glossary EN→VN ở `OUTPUT_MASTER.md` (system prompt mục 5.8).

## Global output convention

### OUTPUT_MASTER

**Mục đích:** Chốt cách dịch term EN → VN khi render output cuối cho user đọc (memo / weekly / stock report / strategy). Áp cross-pack — không thuộc O pack riêng nào. Tách glossary khỏi O pack để: (a) consistency cross-O, (b) sửa 1 chỗ áp toàn deliverable, (c) O pack focus vào structure/tone, glossary tách layer riêng.

**File:** `OUTPUT_MASTER.md` (file đơn, không có `_00` master).

**Nội dung:** 3 nhóm term — A (dịch luôn), B (dịch + ngoặc EN lần đầu), C (giữ EN). Polysemy rules + conflict resolution với O pack K hygiene riêng.

**Trigger:** Đọc đầu session. Re-queryable khi compose deliverable cuối ở bất kỳ O pack nào active.

**Depends:** Không có. Đứng độc lập tương tự `KERNEL_SKELETON.md`.

**Override:** O pack có K hygiene table riêng (vd `O_stock_report_00` mục 5 với audience-aware translation) override OUTPUT_MASTER trong scope O pack đó.

**Status:** Active. Created 2026-05-30.

## K — Knowledge packs

### K_agent_db

**Mục đích:** Knowledge base về dữ liệu chứng khoán Việt Nam trong MongoDB `agent_db` (pipeline fnx05 v2). Bao gồm schema 35 collection (gồm khối phase & danh mục hệ thống, khối lịch sử định giá và lịch sử khối ngoại/tự doanh), query patterns 13 workflow A-M, anti-patterns, methodology diễn giải chỉ báo (dòng tiền, technical zone, trend đa khung, PTCB theo 4 type doanh nghiệp, định giá tương đối theo lịch sử), methodology phân tích tin tức 4 loại (doanh nghiệp, quốc tế, trong nước, thông cáo), và tầng phase & 3 danh mục hệ thống.

**Master:** `K_agent_db_00` (7 file: master + `_01` đến `_06`)

**Trigger:** Mọi query về cổ phiếu Việt Nam, thị trường VN, ticker, ngành, BCTC, dòng tiền, khối ngoại, technical, tin tức chứng khoán VN, hoặc khi cần số liệu định lượng từ `agent_db`. Riêng `K_agent_db_06` (phase & danh mục): chỉ đọc khi user hỏi đích danh về pha thị trường / danh mục hệ thống / hiệu suất / sổ lệnh, hoặc cần bối cảnh phase cho khuyến nghị inline (`K_agent_db_00` mục 4.6). **Các P pack không dùng tầng phase** — giữ methodology regime riêng của từng pack.

**Depends:** Không có.

**Status:** Active.

### K_sector_framework

**Mục đích:** Khung phân tích ngành chuẩn institutional buy-side, chắt lọc từ CFA Sector Analysis Framework (2020). Cung cấp lens systematic cho deep-dive sector-level analysis qua 5 dimension: **Demand Drivers / Market Position / Structural Influences / Performance Metrics / ESG**, cộng Industry 4.0 lens cross-sector. Có per-sector quick-reference cho 10-12 ngành whitelist có CFA cover trực tiếp (NGANHANG, TIENICH, BDS, KCN, BANLE, VANTAI, CONGNGHE, XAYDUNG, THUCPHAM, NONGNGHIEP, CHUNGKHOAN, BAOHIEM override) + guidance generic cho 6 ngành còn lại (DAUKHI, HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, CONGNGHIEP).

**Pack chỉ có 1 file (không có `_00` master riêng vì pack đơn file):** `K_sector_framework`

**Trigger:** P pack chủ động pull khi cần industry-level lens — không tự activate. Cụ thể:
- `P_invest_memo_05/06/07` (Tier 5A/B/C memo deep-dive): khi compose phần "Business" của memo 7 phần
- `P_vbse_strategy_04` (Trục 4 Sector allocation): khi compose per-sector tilt rationale
- `P_weekly_overview_02` (Phần 6 Biến động 18 ngành): chỉ khi ngành có biến động bất thường cần structural watch
- Standalone: khi user hỏi "phân tích sâu ngành X" hoặc "outlook ngành X 12 tháng tới"

**Quan hệ với `K_agent_db_04`:** Bổ trợ, không overlap. `K_agent_db_04` chuyên dòng tiền + PTCB 4 type doanh nghiệp + technical từ data DB; `K_sector_framework` chuyên industry structure + competitive dynamics + ESG chuẩn CFA.

**Depends:** Không có. Reference được `K_agent_db_01/02` khi gợi ý metric cần kéo từ DB.

**Status:** Active. Created 2026-05-30.

## P — Process packs

### P_invest_memo

**Mục đích:** Workflow đầu tư cổ phiếu Việt Nam niêm yết, horizon 1-6 tháng, chỉ long, portfolio dưới 1 triệu USD. Pipeline 5 giai đoạn: gate vĩ mô → chọn 3-5 ngành → screen 6-10 mã/ngành → chấm điểm top 3/ngành → memo deep-dive 7 phần. Bổ sung song song: portfolio construction + monitoring & exit. Mỗi giai đoạn kết bằng checkpoint review 6 phần, chờ user confirm trước khi qua tier kế.

**Master:** `P_invest_memo_00`

**Trigger:** User mention tier hoặc giai đoạn cụ thể (ví dụ "tier 3", "giai đoạn 2", "gate vĩ mô"); yêu cầu "chạy quy trình đầu tư", "viết memo deep-dive nội bộ", "deep-dive mã X", "screen ngành", "shortlist", "portfolio construction", "monitoring danh mục", "rebalance".

**Lưu ý:** "memo" ở đây là conviction memo deep-dive nội bộ (tier 5C), không phải broadcast tuần hay báo cáo chiến lược tháng.

**Depends:** `K_agent_db`. Có thể pull thêm `K_sector_framework` ở Tier 5C (memo 7 phần, phần "Business") để bổ sung industry-level lens.

**Status:** Active.

### P_weekly_overview

**Mục đích:** Sinh báo cáo **tổng quan thị trường tuần** dạng deliverable MD 9-11 trang, broadcast tuần độc lập (không cần thesis cycle nào), dùng được cho cả họp nội bộ và gửi khách hàng. Báo cáo gồm 12 phần rigid, mục tiêu (a) thống kê dữ liệu thị trường tuần qua và (b) đưa regime call + sector bias + watchlist cho tuần tới với conviction + horizon + disconfirming signal (chuẩn institutional buy-side).

Pack chia 5 file con: `_00` master + `_01` pre-flight + Stage 1 first half (phần 2-5) + `_02` Stage 1 second half (phần 6-9 với phần 9 fundamental-driven 3 kịch bản) + `_03` checkpoint + Stage 2 (phần 10-12 + phần 1 Key calls/Watch/Risk) + `_04` methodology + self-audit + edge + contract. Workflow 2 stage, ngăn cách 1 checkpoint sau khi quyết regime + sector bias.

**Philosophy fundamental-driven:** 3 kịch bản phần 9 trigger PRIMARY là vĩ mô/cơ bản/chính sách/catalyst, technical chỉ confirmation phụ ≤30%. Cap technical toàn báo cáo ≤15%. Whitelist 18 ngành áp dụng default; user yêu cầu override được cho ngành ngoài whitelist. Mỗi call có conviction HIGH/MID/LOW + horizon 1-2 tuần / 2-4 tuần + 1-2 disconfirming signal.

Pack độc lập với `P_invest_memo` và `P_vbse_strategy` — không đọc state file invest cycle hay thesis monthly. **Không sử dụng chỉ báo trend nội bộ** (`market_snapshot.trend`, `industry_snapshot.trend`, `series[].trend` trong các collection `*_recent`) — audience có thể là KH.

**Master:** `P_weekly_overview_00`

**Trigger:** User yêu cầu "viết báo cáo tuần", "weekly overview report", "báo cáo tổng quan thị trường tuần", "tổng quan tuần", "broadcast tuần".

**Depends:** `K_agent_db`. Có thể pull thêm `K_sector_framework` ở Phần 6 (Biến động 18 ngành whitelist) khi có ngành biến động bất thường cần structural watch — không phải mặc định mỗi tuần (broadcast tuần ưu tiên ngắn gọn).

**Status:** Active.

### P_vbse_strategy

**Mục đích:** Sinh báo cáo **chiến lược đầu tư VBSE** VN theo 2 chu kỳ lồng nhau — báo cáo tháng (parent, đầu tháng, hình thành thesis) và báo cáo tuần (child, tracking trong tháng, đọc lại monthly để cập nhật). Horizon 1-3 tháng forward-looking. Khác `P_weekly_overview` (đó là broadcast tổng quan tuần độc lập); pack này là **định vị chiến lược deep nội bộ** — VN đang ở đâu trong chu kỳ vĩ mô + chu kỳ thanh khoản + chu kỳ định giá, theme nào chi phối, sector nào ưu tiên, kịch bản nào dự phòng, mã nào đại diện theme.

Khung tư duy 6 trục cốt lõi: (1) môi trường vĩ mô & tài chính, (2) định vị thị trường VN, (3) themes & narratives, (4) sector allocation, (5) kịch bản & risk map, (6) high-conviction watchlist 2-phase (Phase 1 Screen cơ bản + Phase 2 Bucket entry). **Structure flex** — sub-section trong từng trục, độ sâu, số theme/sector/mã linh hoạt theo phát hiện thực tế. Không ép số rigid như `P_weekly_overview`.

**Weight balance — fundamental supremacy:** báo cáo tháng (horizon 1-3, 3-6 tháng) phải dùng signal **vĩ mô + cơ bản + chính sách + catalyst dài hơi làm PRIMARY (~70-75%)** — đây là factor có time-to-play-out khớp horizon. **Định vị thị trường + flow là SECONDARY (~15-20%)**. **Technical là TERTIARY (~10-15%) — chỉ tồn tại hợp pháp ở Phase 2 Bucket entry Trục 6** (phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản). KHÔNG làm confirmation timing tổng quát; KHÔNG quyết định regime/theme/sector/risk. Trục 5 kịch bản trigger primary BẮT BUỘC là macro/fundamental/policy/catalyst (technical = 0% — cấm tuyệt đối). Weekly mode áp **Technical-as-noise rule** (`_08` mục 4): status Shift/Materialize bắt buộc kèm signal vĩ mô/cơ bản/chính sách; technical shift đơn độc = noise tạm thời, status Hold. Ngoại lệ duy nhất: rebucket entry watchlist.

**Chuẩn institutional output:** mỗi theme/sector/mã đều có **conviction level** (HIGH/MID/LOW) + **time horizon** (1m / 1-3m / 3-6m) + **disconfirming signals** ("what would change our mind" — reference field data cụ thể). Trục 4 có sector tilts consolidated table chuẩn buy-side. Trục 6 watchlist có ADV tháng cho liquidity awareness. Review N-1 có Best call / Worst call honesty attribution. Disclaimer có forward-looking statement chuẩn institutional.

**User overlay (PM input):** user có thể inject view ở 3 channel — pre-flight, mid-flow (interrupt session), checkpoint override. Agent xử lý theo matrix 5 trạng thái (Confirm / Partial / Conflict / Flag / Out of scope), không silently override agent finding bằng view user. Báo cáo cuối có badge inline + User overlay log table trong metadata làm audit trail.

**Stage 0 evaluation (đánh giá chiến lược cũ):** cả monthly và weekly mode có optional Stage 0 — agent đọc file báo cáo cũ (N-1 hoặc W-1) user upload, cross-check thesis với actual data từ `agent_db` (giá / dòng tiền / BCTC / vĩ mô), compose eval block 6 phần (monthly) hoặc 4 phần (weekly), present tại Checkpoint 0, user accept / refine / skip carry-forward trước khi build cycle mới. Lưu ý: DB không có collection storage cho báo cáo cũ — chỉ user upload file MD trong session.

**Monthly mode:** workflow 4 stage + 2 checkpoint (Checkpoint 0 sau Stage 0 eval, Checkpoint 1 sau Stage 1 regime/themes). Output 8-12 trang.

**Weekly update mode:** workflow 2 stage + HARD GATE pre-flight. HARD GATE bắt buộc: agent compute hôm nay → tuần thứ [N] của tháng [M/YYYY], hỏi user có monthly active đúng tháng chưa. **Không có monthly → REFUSE chạy weekly**, đề xuất 3 path (chạy monthly trước / dùng `P_weekly_overview` / override với note decay). Header báo cáo bắt buộc ghi "Tuần [N] của tháng [M/YYYY]" + link file monthly tham chiếu. Mỗi trục có status Hold / Shift / Materialize; watchlist refresh 4 trạng thái (Hold / Watch closely / Out / Vào mới); 1-2 action item định tính. Output 3-5 trang.

Wording observation/luận điểm, không command (mua/bán/giảm tỷ trọng/stop loss). Watchlist không entry/stop/target/size — chỉ luận điểm theme + signal theo dõi + disconfirming signal + ADV + Bucket entry (1/2/3). Kịch bản if-then trigger, không % xác suất. Branding & disclaimer optional, render branded khi user cung cấp ở pre-flight.

**Triết lý fundamental supremacy + 2-phase watchlist:** Phase 1 Screen cơ bản-only (cấm PTKT), Phase 2 Bucket entry PTKT-driven. Cap technical toàn báo cáo ≤15%; Trục 2 ≤20%; Trục 4, 5, Phase 1 Trục 6 ≈ 0% (cấm tuyệt đối ở Risk + Screen).

Pack chia 10 file con: `_00` master + `_01..06` mỗi trục một file + `_07` Workflow Monthly + `_08` Workflow Weekly + `_09` Overlay + Self-audit + Edge + Contract. Pack độc lập với `P_invest_memo` và `P_weekly_overview` — không share state. Watchlist ở pack này là theme play observation + bucket entry timing, khác bản chất với portfolio deep-dive của `P_invest_memo`.

**Master:** `P_vbse_strategy_00`

**Trigger:**
- Monthly: "báo cáo chiến lược tháng", "monthly strategy", "outlook tháng [N]", "chiến lược đầu tư tháng", "định vị thị trường tháng [N]", "vbse strategy monthly"
- Weekly update: "update tuần [DD/MM] chiến lược", "weekly strategy update", "cập nhật tuần báo cáo tháng [N]", "weekly check chiến lược", "vbse strategy weekly"

**Depends:** `K_agent_db`. Có thể pull thêm `K_sector_framework` ở Trục 4 (Sector allocation) để compose per-sector tilt rationale với lens industry structure.

**Status:** Active.

### P_stock_report

**Mục đích:** Sinh **báo cáo phân tích chuyên sâu 1 cổ phiếu** Việt Nam niêm yết. Vào trực tiếp từ ticker, không cần qua workflow `P_invest_memo` Tier 0-3. Horizon 1-12 tháng, output 1-10 trang theo 3 depth mode, audience flex (nội bộ analyst hoặc KH), support pair compare 2-3 mã.

Pack chia 5 file: `_00` master + `_01` pre-flight + **Stage 1 data acquisition 16 sub-step (1a-1p)** + type classification (SXKD/NH/CK/BH) + `_02` type-specific framework (lens chi tiết cho 4 type, mỗi type có KPIs + bear case riêng; **SXKD có mục 2.6 Chuỗi giá trị 10 sub-mục — áp dụng 6 framework chuẩn quốc tế**) + `_03` Stage 2 compose + 6-7 phần output (Phần 2 sub-section 3 Vị trí chuỗi giá trị MANDATORY SXKD với 6 sub-sub 3a-3f) + 3 depth mode + Variant Perception rule + Pair compare logic + `_04` self-audit **47 điểm SXKD / 35 điểm NH/CK/BH** + edge cases + failure modes + output contract chi tiết.

**Stage 1 Data Acquisition 16 sub-step (chi tiết ở `_01`):** 1a stock info + type → 1b FA data DB → 1c dòng tiền + technical zone → 1d khối ngoại + tự doanh → 1e major shareholders → 1f corporate actions → 1g news DB → 1h web search news (VN equity + EN macro tuỳ ngành) → 1i **BCTC PDF forensic 15-point đào sâu thuyết minh** → 1j sector context (pull `K_sector_framework`) → 1k macro relevant → 1l peer compare (internet-first + thanh khoản filter) → 1m ADV/liquidity → 1n earnings calendar → 1o ESG controversy scan → **1p Value chain data (top KH/NCC/channel/capacity/R&D ratio/Industry 4.0 readiness — SXKD mandatory Standard+; SKIP NH/CK/BH)**.

**Value chain framework cho SXKD (chi tiết ở `_02` mục 2.6 — 10 sub-mục):** 6 framework chuẩn quốc tế áp dụng đầy đủ:
1. Porter Value Chain (1985) — 5 primary + 4 support activities + forward/backward vertical integration
2. Porter 5 Forces (1979) — supplier/buyer/substitute/entrant/rivalry
3. Smile Curve (Stan Shih 1992) — vị trí capture giá trị (smile bottom/mid/top) — **đặc biệt quan trọng cho VN context** vì hầu hết SXKD VN ở smile bottom
4. GVC governance (Gereffi, Humphrey, Sturgeon 2005) — market/modular/relational/captive/hierarchy + Tier supplier position
5. Industry 4.0 / Digital footprint (CFA Sector Analysis 2020) — Three Golden Steps + 7-dimension readiness table
6. CFA Sector Analysis 2020 — 21 industry chapter mapping với 18 ngành VN whitelist + 3 financial

**4 type framework (chi tiết ở `_02`):**
- **SXKD** (Sản xuất kinh doanh, 21 ngành whitelist trừ tài chính): 4 kịch bản Value Play / Value Trap / Growth at Premium / Cycle Top + 3 sub-type cycle dynamics (Cyclical / Consumer-Defensive / Growth-Infrastructure)
- **NH** (Banking): NIM drivers + asset quality (NPL Group 2-5, LLR coverage) + capital (CAR, LDR, LCR) + bank-specific FA
- **CK** (Chứng khoán): brokerage market share + margin book quality (leverage, yield, concentration) + IB pipeline + prop book VaR
- **BH** (Bảo hiểm — override mode): combined ratio (Loss + Expense) + APE/NBV (life) + persistency + investment yield + solvency margin

**3 depth mode (chi tiết ở `_03`):**
- **Quick** 1-2 trang: skip Stage 1i forensic, 1l peer, 1o ESG, 1p value chain. Variant Perception optional
- **Standard** 3-5 trang (SXKD 2-3 trang Phần 2 với value chain): full 16 sub-step (1p SXKD only), peer 3 mã. Variant Perception recommended (không có → flag "Consensus-aligned thesis, edge limited")
- **Deep** 5-10 trang (SXKD 3-4 trang Phần 2): full + ESG controversy kỹ + value chain đầy đủ 6 framework, peer 5 mã, macro sensitivity, data appendix. **Variant Perception bắt buộc** (không có → auto downgrade conviction HIGH→MID / MID→LOW / LOW→Watch)

**Constraint chính:**
- **BCTC PDF mandatory** — không upload thì REFUSE chạy (gate strict tuyệt đối)
- **Long-only** (Long / Watch / Avoid, không Short)
- **Web search VN cho equity, EN cho macro** chỉ với ngành có liên quan tài chính / commodity (Banking → Fed, Dầu khí → OPEC, Kim loại → LME, Thực phẩm → USDA, etc.)
- **Peer compare internet-first** + filter ADV ≥ 30 tỷ/ngày + market cap top 50 ngành, exclude small cap unknown
- **Pattern strict reject Long:** dòng tiền dương + catalyst tiêu cực material → auto downgrade Watch (như `P_invest_memo` nguyên tắc 5)
- **Conviction CAP at LOW cho penny stock** (market cap < 1.000 tỷ) hoặc **CAP at MID cho newly listed** (< 2 năm)
- **Audience flex** (nội bộ / KH) — wording + K hygiene khác nhau, đặc biệt audience KH không render TP1/TP2/SL số cụ thể
- **Bear case mandatory** cho mọi recommendation (kể cả HIGH conviction)
- **Disconfirming signal MEASURABLE** với threshold cụ thể (số / sự kiện)

**Pair compare mode:** support 2-3 mã cùng ngành hoặc cùng theme/value chain. Render side-by-side (Standard) hoặc sequential (Deep) + pair selection thesis cuối báo cáo. Apple-to-orange (vd VNM vs VCB) → REFUSE pair, suggest pick 1 mã.

**Master:** `P_stock_report_00`

**Trigger:**
- "Phân tích mã [X]", "Phân tích cổ phiếu [X]", "Đánh giá [X]"
- "[X] có nên mua không", "Brief [X] cho KH"
- "Quick check [X]", "Stock report [X]"
- "So sánh [X] vs [Y]" (mode pair)
- "[X] horizon [Y] tháng"

**Trigger không activate (conflict resolution):**
- "Memo deep-dive [X]" hoặc "Tier 5C [X]" → activate `P_invest_memo` Tier 5C
- "Báo cáo tuần" → `P_weekly_overview`
- "Chiến lược tháng" → `P_vbse_strategy`

**Depends:** `K_agent_db` (mandatory) + `K_sector_framework` (recommended cho Phần 3 industry context và Phần 2 sub-type context).

**Quan hệ với `P_invest_memo`:** Complement, không thay thế. `P_stock_report` dùng pre-screening / pitch nhanh / ad-hoc deep-dive 1 mã. Tier 5C dùng full conviction memo cycle (sau khi đã qua Tier 0-3). KHÔNG auto-escalate sang Tier 5C — user phải explicit yêu cầu.

**Status:** Active.

## O — Output packs

### O_invest_memo

**Mục đích:** Render spec cho deliverable của pack `P_invest_memo` — báo cáo checkpoint tier 0-7, memo deep-dive 7 phần, portfolio plan, weekly review portfolio. Quy định structure rigid theo loại output, format MD/docx/pptx, K hygiene, citation, chart annotation.

**Master:** `O_invest_memo_00`

**Trigger:** Activate cùng với `P_invest_memo` khi user yêu cầu deliverable file (memo, report, pitch deck) hoặc render output theo style chuẩn của workflow đầu tư.

**Depends:** `P_invest_memo`, `K_agent_db`.

**Status:** Active.

### O_weekly_overview

**Mục đích:** Render spec cho deliverable của pack `P_weekly_overview` — báo cáo tổng quan thị trường tuần 12 phần rigid, heading exact. Quy định format MD (source of truth), docx (archive formal), pptx (meeting trình bày), 3 mode branding (custom / default branded / plain), K hygiene, mapping bảng/chart cho từng phần, conviction marker + horizon + disconfirming bắt buộc cho mỗi call (regime / sector bias / watchlist mã).

**Master:** `O_weekly_overview_00`

**Trigger:** Activate cùng với `P_weekly_overview` khi user yêu cầu báo cáo tổng quan thị trường tuần.

**Depends:** `P_weekly_overview`, `K_agent_db`.

**Status:** Active.

### O_vbse_strategy

**Mục đích:** Render spec cho deliverable của pack `P_vbse_strategy` — báo cáo chiến lược đầu tư VBSE VN, 2 mode (monthly parent + weekly update child). **Flex structure** thay vì rigid: 6 trục H2 cố định nhưng sub-section H3 và độ sâu flex theo phát hiện thực tế; trục Hold rút gọn 3-5 dòng, trục signal mạnh đào sâu không giới hạn. Quy định header (plain / branded), layout từng trục với fundamental-first vocabulary, format watchlist (Phase 1 Screen cơ bản + Phase 2 Bucket entry observation, không level giá), checkpoint block monthly (CP0 + CP1), status badge weekly (Hold / Shift / Materialize) với technical-as-noise rule, watchlist refresh 4 trạng thái + rebucket section, disclaimer 3 trường hợp, K hygiene, metadata mỗi mode + User overlay log table.

**Master:** `O_vbse_strategy_00`

**Trigger:** Activate cùng với `P_vbse_strategy` khi user yêu cầu báo cáo chiến lược tháng hoặc weekly update.

**Depends:** `P_vbse_strategy`, `K_agent_db`.

**Status:** Active.

### O_stock_report

**Mục đích:** Render spec cho deliverable của pack `P_stock_report` — báo cáo phân tích chuyên sâu 1 cổ phiếu VN niêm yết, 3 depth mode (Quick / Standard / Deep), audience flex (nội bộ / KH), pair compare optional. Quy định structure rigid 6-7 phần, format MD (source of truth), 2 mode branding (plain / branded optional), K hygiene + citation (4 nhóm), audit trail metadata, file naming `stock_report_<TICKER>_<YYYYMMDD>_<mode>.md`.

Output cuối là MD final trong message (Claude Desktop). Render binary pptx/docx out of scope project (xem README.md mục 8.1). Branding optional — user cung cấp brand info ở pre-flight nếu cần shell visual.

**K hygiene đặc biệt:**
- Audience nội bộ analyst: giữ raw Recommendation Long/Watch/Avoid + Conviction HIGH/MID/LOW + TP1/TP2/SL số
- Audience KH: dịch sang "Quan điểm tích cực / Theo dõi / Cẩn trọng" + KHÔNG render TP/SL số (chỉ "Tín hiệu cần theo dõi để xem xét lại quan điểm")

**Forward-looking statement** bắt buộc cho Deep mode + audience KH.

**Master:** `O_stock_report_00` (1 file đơn).

**Trigger:** Activate cùng với `P_stock_report` khi user yêu cầu báo cáo phân tích 1 cổ phiếu (single hoặc pair compare).

**Depends:** `P_stock_report`, `K_agent_db`.

**Status:** Active.

## Render binary (pptx / docx / xlsx)

MD final là source of truth. Khi user yêu cầu render binary, agent chạy theo workflow ở `system_prompt.md` mục 4 "Render binary — workflow": xác định style qua (a) O pack render spec, (b) branding info pre-flight, (c) user explicit; nếu không rõ thì hỏi clarify. **Body font chốt: Roboto** (fallback Roboto → Open Sans → Arial). Binary derive từ MD final, không edit độc lập — sửa nội dung phải sửa MD trước rồi re-render.

## Naming convention tham chiếu

Pack tương lai đặt tên theo pattern (lặp lại từ system prompt mục 2 để agent tiện đối chiếu):

```
K_{domain}_{NN}              ví dụ K_realestate_00, K_macro_00
P_{flow_name}_{NN}           ví dụ P_quick_check_00, P_earnings_review_00
O_{format_or_style}_{NN}     ví dụ O_memo_docx_00, O_inline_chat_00
```

Số `NN` ý nghĩa nội bộ pack, quy định trong file `_00` của pack đó.

**Exception cho pack 1-file:** theo system prompt mục 2 ("Pack có ≥3 file phải có master. Pack 1-2 file không bắt buộc master"), pack 1-file có thể bỏ suffix `_NN`. Hiện có 1 pack áp dụng exception này: `K_sector_framework` (1 file đơn, không có `_00` riêng).

