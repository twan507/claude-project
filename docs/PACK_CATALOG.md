# Pack Catalog

Catalog chi tiết từng pack/file trong 2 agent. Mỗi pack ghi: mục đích, files, dependencies, key constraints, output naming. Cập nhật: 2026-05-30.

---

## analysis_agent — Layered K/P/O Architecture

Multi-pack analyst agent dùng kernel routing pattern. Đọc `KERNEL_SKELETON.md` đầu session để route, sau đó activate pack tương ứng.

### Knowledge layer

#### `K_agent_db` — 7 files

**Mục đích:** Knowledge base về MongoDB `agent_db` chứng khoán VN (pipeline fnx05 v2). Schema 31 collection, query patterns, anti-patterns, methodology diễn giải chỉ báo + tin tức, tầng phase & danh mục hệ thống.

| File | Nội dung |
|---|---|
| `K_agent_db_00` | Master — mục đích, scope, manifest, domain rules (**whitelist 18 ngành** mục 4.5, **phase = tín hiệu tham chiếu** mục 4.6, luật hiệu suất 2 tầng mục 4.3), K hygiene + bảng dịch DB raw (mục 5), quy đổi đơn vị v2 điểm % (mục 6), omit-null + known gaps (mục 9), output contract |
| `K_agent_db_01` | Schema 31 collection (8 khối cũ + **Section I phase & danh mục**) + URL pattern finext.vn + **Xếp hạng ngành tự tổng hợp** |
| `K_agent_db_02` | Query patterns 13 workflow (A-M; **M = phase & danh mục**) — bao gồm `Section 3.6` rank ngành tự tổng hợp aggregate |
| `K_agent_db_03` | Anti-patterns 10 case — case 9-10 mới (bối cảnh phase khi khuyến nghị, hiệu suất 2 tầng) |
| `K_agent_db_04` | Methodology diễn giải chỉ báo (dòng tiền, trend đa khung, technical zone, PTCB 4 type doanh nghiệp) + **bảng dịch taxonomy đầu file** |
| `K_agent_db_05` | News methodology (4 loại tin, framework chấm impact, case study, bảng dịch thuật ngữ EN) |
| `K_agent_db_06` | Phase & 3 danh mục hệ thống — 4 trạng thái + exposure, 7 chỉ số, cơ chế cơ cấu, bộ số hiệu suất FROZEN + 6 disclaimer bắt buộc. **Chỉ đọc khi user hỏi đích danh; P pack không dùng tầng phase** |

**Dependencies:** Không (knowledge layer độc lập).

**Status:** ✅ Active (2026-07-13 — port từ agent_db v2). Schema được sync với `agent_db/agent_db_*` (cùng schema, framing khác).

---

#### `K_sector_framework` — 1 file

**Mục đích:** Khung phân tích ngành chuẩn institutional buy-side, chắt lọc từ CFA Sector Analysis Framework (2020). Cung cấp lens systematic để deep-dive sector-level analysis bổ trợ cho dòng tiền/PTCB của `K_agent_db_04`.

| File | Nội dung |
|---|---|
| `K_sector_framework` | (1) Universal 5-dimension framework: **Demand Drivers / Market Position / Structural Influences / Performance Metrics / ESG** với question bank distilled (~10-15 câu/dimension). (2) **Per-sector quick-reference** cho 10-12 ngành whitelist có CFA cover (NGANHANG, TIENICH, BDS, KCN, BANLE, VANTAI, CONGNGHE, XAYDUNG, THUCPHAM, NONGNGHIEP, CHUNGKHOAN, BAOHIEM override). (3) Guidance generic cho 6-7 ngành whitelist không CFA cover (DAUKHI, HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, THUYSAN, CONGNGHIEP). (4) **Industry 4.0 lens** — digital footprint, automation, AI/IoT disruption cross-sector. |

**Dependencies:** Không (knowledge layer độc lập). Reference được từ `P_invest_memo_05/06/07`, `P_vbse_strategy_04`, `P_weekly_overview_02`.

**Quan hệ với `K_agent_db_04`:**
- `K_agent_db_04`: chuyên dòng tiền + PTCB 4 type doanh nghiệp + chỉ báo technical (từ data DB)
- `K_sector_framework`: chuyên industry structure + competitive dynamics + ESG (chuẩn CFA)
- 2 pack **bổ trợ**, không thay thế nhau

**Status:** ✅ Active. Created 2026-05-30.

---

### Process layer

#### `P_invest_memo` — 10 files

**Mục đích:** Workflow đầu tư cá nhân VN, horizon 1-6 tháng, chỉ long, portfolio <1 triệu USD. Pipeline 5 giai đoạn + giai đoạn 6 song song.

| File | Nội dung |
|---|---|
| `_00` | Master — 6 nguyên tắc bất biến + flow tổng + từ điển |
| `_01` | Tier 0: Gate vĩ mô + Catalyst scan |
| `_02` | Tier 1: Chọn 3-5 ngành (rank ngành tự tổng hợp theo `week_score` qua 18 whitelist) |
| `_03` | Tier 2: Screen mã + Bucket entry 1/2/3 (entry timing — reference cho `P_vbse_strategy_06`) |
| `_04` | Tier 3: Chấm điểm top 3/ngành |
| `_05` | Tier 5A: PDF Deep-dive Forensic |
| `_06` | Tier 5B: Valuation Modeling |
| `_07` | Tier 5C: Investment Memo Template 7 phần |
| `_08` | Tier 6: Portfolio Construction |
| `_09` | Tier 7: Monitoring + Exit Execution |

**Dependencies:** `K_agent_db`.

**Key constraints:** 6 nguyên tắc bất biến (variant perception, exit trigger, ADV constraint, bear steelman, dòng tiền + catalyst tiêu cực → loại, checkpoint discipline).

**Output naming:**
- `tier{N}_<YYYYMMDD>_confirmed.md`
- `tier5C_<TICKER>_<YYYYMMDD>_confirmed.md`
- `tier6_portfolio_<YYYYMMDD>_confirmed.md`
- `tier7_<weekly|monthly|quarterly>_<period>.md`

**Status:** ✅ Active. Stable, không thay đổi trong session 2026-05.

---

#### `P_weekly_overview` — 5 files

**Mục đích:** Báo cáo tổng quan thị trường tuần broadcast độc lập (không cần thesis cycle). 12 phần rigid, 9-11 trang, audience nội bộ + KH. Refactor từ `P_weekly_market` cũ với fundamental-driven philosophy + conviction labeling chuẩn institutional.

| File | Nội dung |
|---|---|
| `_00` | Master — philosophy fundamental-driven, weight balance cap technical, 4 nguyên tắc bất biến (fundamental trigger / whitelist 18 / conviction+horizon+disconfirming bắt buộc / checkpoint), từ điển, file index, cross-ref với `P_vbse_strategy` |
| `_01` | Pre-flight 3 câu + Phần 2 Review W-1 (3 scorecard tables) + Phần 3 Quốc tế + Phần 4 VN (aggregate 18 whitelist) + Phần 5 Vĩ mô-hàng hoá (institutional table 5 cột: Magnitude + Persistence) |
| `_02` | Phần 6 Biến động 18 ngành whitelist (+ earnings beat candidate) + Phần 7 Top dẫn dắt + cảnh báo trap setup + Phần 8 Tin tức + conviction impact + Phần 9 Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map cross-link theme |
| `_03` | Checkpoint 1 (regime + sector bias + conviction + disconfirming) + Phần 10 Watchlist tách 2 hướng (4 dòng mỗi mã + Bucket optional) + Phần 11 Lịch sự kiện + Phần 12 Disclaimer 3 mode + Phần 1 Executive summary Key calls/Watch/Risk |
| `_04` | Methodology (regime classification 4 mức + sector bias logic + mapping vĩ mô-18 ngành table) + Technical-as-noise rule + Self-audit 12 item + Edge cases + Output contract |

**Dependencies:** `K_agent_db`.

**Key constraints:**
- 3 kịch bản phần 9 trigger primary là vĩ mô/cơ bản/chính sách/catalyst (technical confirmation phụ ≤30%)
- Cap technical toàn báo cáo ≤15%
- Whitelist 18 ngành default + override
- Mỗi call có conviction HIGH/MID/LOW + horizon (1-2w / 2-4w) + 1-2 disconfirming
- Không dùng chỉ báo trend nội bộ (audience cuối có thể là KH)
- Rank ngành tự tổng hợp theo `week_score`

**Output naming:** `weekly_overview_<YYYYMMDD>.md` (YYYYMMDD = ngày Chủ Nhật kết thúc tuần).

**Status:** ✅ Active. Refactored 2026-05-30.

---

#### `P_vbse_strategy` — 10 files

**Mục đích:** Báo cáo chiến lược đầu tư VBSE VN. 2 cycle lồng nhau: monthly parent (đầu tháng, hình thành thesis) + weekly child (tracking trong tháng). Horizon 1-3 tháng. Audience PM/analyst deep nội bộ. Refactor từ `P_invest_strategy` cũ với philosophy **fundamental supremacy** + **2-phase watchlist**.

| File | Nội dung |
|---|---|
| `_00` | Master — philosophy fundamental supremacy (PTKT chỉ ở Phase 2 Bucket entry), weight balance cap technical, 4 nguyên tắc bất biến, từ điển, file index, cross-ref với `P_weekly_overview` |
| `_01` | Trục 1 — Môi trường vĩ mô & tài chính (PRIMARY ~70-75%) |
| `_02` | Trục 2 — Định vị thị trường VN (fundamental-first: định giá phân vị + dòng tiền aggregate + FII + breadth làm PRIMARY; technical ≤20% chỉ minh hoạ) |
| `_03` | Trục 3 — Themes & narratives (5 thành phần mỗi theme) |
| `_04` | Trục 4 — Sector allocation (whitelist 18, cap technical ≤5%, mapping vĩ mô-ngành đầy đủ, rank ngành tự tổng hợp) |
| `_05` | Trục 5 — Risk scenarios (3 kịch bản if-then trigger macro/fundamental/policy ONLY, cấm technical primary) |
| `_06` | Trục 6 — Watchlist 2-phase (Phase 1 Screen cơ bản-only cấm PTKT + Phase 2 Bucket entry PTKT-driven theo `P_invest_memo_03` mục 5) |
| `_07` | Workflow Monthly (Pre-flight + Stage 0 eval N-1 + CP0 + Stage 1 build thesis + CP1 + Stage 2 allocation+risk + Stage 3 watchlist+exec sum + render) |
| `_08` | Workflow Weekly (HARD GATE pre-flight + Stage 0 eval W-1 + Stage 1 tracking với technical-as-noise rule + rebucket entry) |
| `_09` | User overlay (3 channel + matrix 5 trạng thái) + Self-audit 27 item monthly / 13 item weekly + Edge cases + Output contract |

**Dependencies:** `K_agent_db` (+ reference `P_invest_memo_03` cho Bucket entry definition).

**Key constraints:**
- **Fundamental supremacy:** mọi quyết định driver bởi vĩ mô/cơ bản/chính sách/catalyst
- **PTKT vai trò duy nhất:** Phase 2 Bucket entry Trục 6 (phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản)
- **Cap technical:** Trục 1 = 0%, Trục 2 ≤20%, Trục 3 ≤5%, Trục 4 ≤5%, Trục 5 = 0%, Phase 1 Trục 6 = 0%, Phase 2 Trục 6 = 80-100% (vùng hợp pháp PTKT). Báo cáo tổng ≤15%
- **Whitelist 18 ngành** default + override
- **Weekly HARD GATE:** không có monthly active → REFUSE
- **Technical-as-noise rule** weekly: Shift bắt buộc kèm signal vĩ mô/cơ bản/chính sách. Technical shift đơn độc = noise tạm thời
- Conviction HIGH/MID/LOW + horizon 1m/1-3m/3-6m + disconfirming bắt buộc mỗi theme/sector/mã

**Output naming:**
- Monthly: `vbse_strategy_monthly_<YYYYMM>.md`
- Weekly: `vbse_strategy_weekly_<YYYYMMDD>.md`

**Status:** ✅ Active. Refactored 2026-05-30.

---

#### `P_stock_report` — 5 files

**Mục đích:** Sinh **báo cáo phân tích chuyên sâu 1 cổ phiếu** Việt Nam niêm yết. Vào trực tiếp từ ticker, không cần qua workflow `P_invest_memo` Tier 0-3. Horizon 1-12 tháng, output 1-10 trang theo 3 depth mode, audience flex (nội bộ analyst / KH), support pair compare 2-3 mã. Complement (không thay thế) Tier 5C của `P_invest_memo` — dùng cho pre-screening / pitch nhanh / ad-hoc analysis ngoài full memo cycle.

| File | Nội dung |
|---|---|
| `_00` | Master — mục đích, scope, differentiate với Tier 5C, philosophy, 6 nguyên tắc bất biến (cross-cutting + pack-specific), manifest (5 file con với Stage 1 16 sub-step + mục 2.6 chuỗi giá trị SXKD), workflow tổng, 3 depth mode definition, cross-reference, flex+downgrade triết lý |
| `_01` | Pre-flight 6 câu (ticker / horizon / depth mode / audience / pair / **file request BCTC PDF bắt buộc**) + Stage 0 optional eval prior analysis + **Stage 1 Data Acquisition 16 sub-step (1a-1p)** (1a stock info + type classification → 1b FA data DB → 1c dòng tiền + tech zone → 1d khối ngoại + tự doanh → 1e major shareholders → 1f corporate actions → 1g news DB → 1h web search news (VN equity + EN macro tuỳ ngành) → 1i **BCTC PDF forensic 15-point checklist đào sâu thuyết minh** → 1j sector context K_sector_framework → 1k macro relevant → 1l **peer compare internet-first + filter thanh khoản** → 1m ADV liquidity → 1n earnings calendar → 1o ESG controversy scan → **1p Value chain data top KH/NCC/channel/capacity/R&D ratio/Industry 4.0 readiness — SXKD mandatory Standard+; SKIP NH/CK/BH**) + depth mode coverage matrix + fail-soft rule |
| `_02` | **Type-specific framework cho 4 type:** SXKD (4 kịch bản Value Play / Value Trap / Growth at Premium / Cycle Top + 3 sub-type cycle dynamics + **mục 2.6 Chuỗi giá trị 10 sub-mục: 2.6.0 khung tham chiếu chuyên nghiệp / 2.6.1 industry value chain map / 2.6.2 Porter Value Chain 5 primary + 4 support + forward/backward integration / 2.6.3 Porter 5 forces + GVC governance + Tier suppliers / 2.6.4 Smile Curve Stan Shih / 2.6.5 Industry 4.0 CFA Sector Analysis 2020 / 2.6.6 position summary tích hợp / 2.6.7 data sourcing / 2.6.8 cross-link 4 kịch bản / 2.6.9 CFA chapter mapping 21 ngành ↔ VN whitelist**) + NH (NIM/CASA/CAR/NPL + bank-specific FA) + CK (brokerage share/margin/IB/prop book) + BH (combined ratio/APE/persistency/embedded value) + cross-type decision rule (conglomerate / holding / newly listed) + output template phần 2 theo type |
| `_03` | Stage 2 workflow + **6-7 phần output structure rigid** (1 Khuyến nghị / 2 Doanh nghiệp với **sub-section 3 Vị trí chuỗi giá trị MANDATORY SXKD Standard+ với 6 sub-sub 3a-3f** / 3 Bối cảnh ngành & vĩ mô / 4 Tài chính & định giá / 5 Tin tức & Catalyst / 6 Bear case & Disconfirming / 7 Exit triggers chỉ Long) + 3 depth mode coverage + **Variant Perception rule per mode** (Quick optional / Standard recommended với flag / **Deep mandatory với auto downgrade**) + **Pair compare mode** (2-3 mã, same industry / theme constraint) + Checkpoint 1 thesis core review + Checkpoint 2 optional output review + bear case strict reject pattern (dòng tiền dương + catalyst tiêu cực) + K hygiene checklist |
| `_04` | **Self-audit checklist 47 điểm SXKD / 35 điểm NH/CK/BH** (data quality 10 + thesis quality 8 + type-specific 4 + **value chain audit 1.3b SXKD 12 điểm: Porter 4 + Competitive 3 + Smile/Industry 4.0 2 + Synthesis 1 + Professional standards 2** + strict reject pattern 3 + K hygiene + citation 5 + audience awareness 3 + flex+downgrade 2) + **Edge cases** (conglomerate / holding / newly listed / suspended / penny / mã ngoài whitelist / related party hệ tập đoàn / ETF / multi-listing / delisting) + **10 failure modes** (BCTC thiếu thuyết minh / BCTC quá cũ / thesis dựa rẻ / bear soft-pedal / catalyst không timing / disconfirming mơ hồ / fake VP / skip bear khi HIGH / auto-escalate Long / KH render technical raw) + output contract chi tiết + limitations + honest disclosure |

**Dependencies:**
- `K_agent_db` (mandatory) — schema + query patterns + methodology + K hygiene
- `K_sector_framework` (mandatory cho SXKD value chain analysis, recommended ngành khác) — pull cho Phần 3 industry context + Phần 2 sub-type context + Phần 2 sub-section 3 Vị trí chuỗi giá trị (mục 7.4 Điểm A/B/C của K_sector_framework)

**Key constraints:**
- **BCTC PDF mandatory** — không upload thì REFUSE chạy (gate strict)
- **Long-only** (Long / Watch / Avoid)
- **Web search VN-only cho equity**, EN OK cho macro liên quan financial/commodity (Fed, OPEC, LME, USDA, etc.)
- **Peer compare internet-first** + filter ADV ≥ 30 tỷ/ngày + market cap top 50, exclude small cap unknown
- **Strict reject pattern Long:** dòng tiền dương + catalyst tiêu cực material → auto downgrade Watch
- **Conviction CAP** at LOW cho penny (market cap < 1.000 tỷ), at MID cho newly listed (< 2 năm)
- **Audience flex** (nội bộ / KH) — wording + K hygiene khác nhau, KH KHÔNG nhận TP/SL số cụ thể
- **Bear case mandatory** cho mọi recommendation
- **Disconfirming signal measurable** với threshold cụ thể (số / sự kiện)
- **Value chain analysis MANDATORY cho SXKD Standard+** — áp dụng 6 framework chuẩn quốc tế (Porter Value Chain 1985 + Porter 5 Forces 1979 + Smile Curve Stan Shih 1992 + GVC governance Gereffi 2005 + Industry 4.0 CFA Sector Analysis 2020 + CFA chapter mapping). SKIP NH/CK/BH (đã có lens type-specific tương đương)

**Output naming:**
- Single: `stock_report_<TICKER>_<YYYYMMDD>_<mode>.md` (mode = quick/standard/deep)
- Pair: `stock_report_<TICKER1>vs<TICKER2>_<YYYYMMDD>_pair_<mode>.md`

**Status:** ✅ Active. Created 2026-05-30.

---

### Output layer

#### `O_invest_memo` — 7 files

**Mục đích:** Render spec cho 6 deliverable của `P_invest_memo`. Quy định MD/docx/pptx structure rigid theo loại output, K hygiene, citation.

| File | Deliverable |
|---|---|
| `_00` | Master + checkpoint report template |
| `_01..06` | Tier-specific render specs |

**Dependencies:** `P_invest_memo`, `K_agent_db`.

**Status:** ✅ Active. Stable.

---

#### `O_weekly_overview` — 1 file

**Mục đích:** Render spec cho `P_weekly_overview`. 12 phần rigid MD + 3 mode branding (custom / default branded / plain).

| File | Nội dung |
|---|---|
| `_00` | Master + structure 12 phần + 3 mode disclaimer + metadata + User overlay log |

**Dependencies:** `P_weekly_overview`, `K_agent_db`.

**Output target:** 9-11 trang MD.

**Status:** ✅ Active. Created 2026-05-30.

---

#### `O_vbse_strategy` — 1 file

**Mục đích:** Render spec cho `P_vbse_strategy`. 2 mode (monthly + weekly), 6 trục flex structure.

| File | Nội dung |
|---|---|
| `_00` | Master + monthly structure (8-12 trang) + weekly structure (3-5 trang) + 3 mode branding + Phase 1+2 watchlist render + rebucket section weekly + User overlay log |

**Dependencies:** `P_vbse_strategy`, `K_agent_db`.

**Status:** ✅ Active. Created 2026-05-30.

---

#### `O_stock_report` — 1 file

**Mục đích:** Render spec cho `P_stock_report`. 6-7 phần rigid structure + 3 depth mode (Quick 1-2 / Standard 3-5 / Deep 5-10 trang) + audience flex (nội bộ analyst / KH) + pair compare optional + 2 mode branding (plain / branded optional).

| File | Nội dung |
|---|---|
| `_00` | Master + frontmatter metadata format + heading hierarchy 6-7 phần + format từng phần (Khuyến nghị / Doanh nghiệp / Bối cảnh / Tài chính / Tin tức Catalyst / Bear case Disconfirming / Exit triggers / Phụ lục audit trail) + K hygiene table dịch taxonomy nội bộ ↔ wording cho 2 audience + citation 4 nhóm + 2 mode branding + length budget + disclaimer 2 mode + forward-looking statement Deep+client + output naming + self-checklist trước render |

**Key K hygiene rule:**
- Audience nội bộ: giữ raw Recommendation Long/Watch/Avoid + Conviction HIGH/MID/LOW + TP1/TP2/SL số cụ thể
- Audience KH: dịch sang "Quan điểm tích cực / Theo dõi / Cẩn trọng" + KHÔNG render TP/SL số (chỉ "Tín hiệu cần theo dõi để xem xét lại quan điểm")

**Dependencies:** `P_stock_report`, `K_agent_db`.

**Status:** ✅ Active. Created 2026-05-30.

---

### Meta files (analysis_agent root)

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules vận hành agent (paste vào Custom Instructions Claude project) — kiến trúc 3 layer K/P/O + index, master-first reading, render binary workflow, font Roboto |
| `KERNEL_SKELETON.md` | File index — liệt kê pack có sẵn + trigger activation. Agent đọc đầu session. |

---

## db_agent (`agent_db/`) — Monolithic Knowledge Base v2

Single-shot stock analyst agent, audience NĐT khách hàng Finext (v2, 2026-07-12). Knowledge base 6 file phẳng + system prompt resident (không có layered architecture). Dùng để tra cứu DB ad-hoc.

### Files

| File | Nội dung |
|---|---|
| `system_prompt.md` | v2 — file resident duy nhất, gộp `agent_db_00` cũ: vai trò + tone + bản đồ collection + đơn vị + phase (tín hiệu tham chiếu, v2.1) + khuyến nghị & hiệu suất 2 tầng + meta-rules + bảng dịch rút gọn + manifest |
| `agent_db_01` | Schema 31 collection (đồng bộ với `K_agent_db_01`) — 8 khối + Section I phase & danh mục, **Xếp hạng ngành tự tổng hợp**, whitelist callout |
| `agent_db_02` | Query patterns 13 workflow A-M (đồng bộ với `K_agent_db_02`) |
| `agent_db_03` | Anti-patterns 10 case (đồng bộ với `K_agent_db_03`) |
| `agent_db_04` | Methodology diễn giải chỉ báo + bảng dịch taxonomy đầu file (đồng bộ với `K_agent_db_04`) |
| `agent_db_05` | News methodology (đồng bộ với `K_agent_db_05`) |
| `agent_db_06` | Phase & 3 danh mục hệ thống (đồng bộ với `K_agent_db_06`) |

**Dependencies:** Không (standalone).

**Đồng bộ DB với analysis_agent (CHỈ là cùng query 1 DB, không phải shared knowledge):** schema content giống `K_agent_db_*` vì 2 agent cùng query MongoDB `agent_db`. KHÔNG có cross-reference giữa 2 agent. Mỗi agent đọc knowledge của riêng mình.

Khác biệt giữ riêng:
- Naming prefix `agent_db_*` (không có `K_` tiền tố)
- Framing dùng "bộ tài liệu này" thay vì "pack"
- Cross-refs nội bộ dùng `agent_db_NN`
- Khi update schema, phải update cả 2 nơi riêng biệt (không tự sync)

**Status:** ✅ Active. Schema đồng bộ với DB 2026-05-30.

---

## template_agent — XOÁ

Pack `template_agent` (document-to-pptx normalizer + brander VBSE/Finext) đã được **xoá hoàn toàn** khỏi project 2026-05-30. Lý do: render binary out of scope, MD final từ analysis_agent đã đủ structured để tool render bên ngoài consume. Catalog cũ archive trong `docs/CHANGELOG.md` mục 2026-05-30.

---

## Cross-references nhanh

| Khi cần | Mở file |
|---|---|
| Hiểu schema DB | `K_agent_db_01` (analysis) hoặc `agent_db_01` (db) |
| Viết báo cáo tuần broadcast | `P_weekly_overview_*` + `O_weekly_overview_00` |
| Viết chiến lược tháng deep nội bộ | `P_vbse_strategy_*` + `O_vbse_strategy_00` |
| Workflow đầu tư cá nhân deep-dive (full portfolio cycle) | `P_invest_memo_*` + `O_invest_memo_*` |
| **Phân tích chuyên sâu 1 cổ phiếu standalone** (ad-hoc / pitch / pre-screening / pair compare) | `P_stock_report_*` + `O_stock_report_00` |
| Query DB ad-hoc | db_agent (monolithic) |
| Deep-dive ngành theo chuẩn CFA institutional | `K_sector_framework` (universal DD/MP/SI/PM/ESG + per-sector quick-ref) |
| Hiểu 4 type doanh nghiệp (SXKD/NH/CK/BH) | `K_agent_db_04` (methodology gốc) + `P_stock_report_02` (apply cho single-stock) |
| Hiểu whitelist 18 ngành rule | `K_agent_db_00` mục 4.5 + `K_agent_db_01` Section B (đầu khối) |
| Hiểu rank ngành tự tổng hợp | `K_agent_db_01` mục "Xếp hạng ngành" + `K_agent_db_02` Section 3.6 |
| Hiểu Bucket entry (1/2/3) | `P_invest_memo_03` mục 5 (master definition) — `P_vbse_strategy_06` reference |
| Hiểu tỷ trọng PTKT trong pack | Master `P_vbse_strategy_00` mục 4 + `P_weekly_overview_00` mục 4 |
| Hiểu Variant Perception concept | `P_invest_memo_07` Phần 2 (gate Tier 5C) + `P_stock_report_03` mục 4 (rule per depth mode) |
| BCTC PDF forensic 15-point checklist | `P_stock_report_01` mục 3.9 (Stage 1i đào sâu thuyết minh) |
| Value chain analysis SXKD 6 framework | `P_stock_report_02` mục 2.6 (Porter VC 1985 + Porter 5 Forces 1979 + Smile Curve Stan Shih 1992 + GVC governance Gereffi 2005 + Industry 4.0 CFA Sector Analysis 2020 + CFA chapter mapping 21 ngành) + render template `O_stock_report_00` |
| Sub-step 1p Value chain data (top KH/NCC/channel/R&D/Industry 4.0) | `P_stock_report_01` mục 3.16 (Stage 1p — SXKD mandatory Standard+) |
| Smile Curve concept (Stan Shih 1992) | `P_stock_report_02` mục 2.6.4 (ASCII diagram + 3 zones + VN context callout) |
| Industry 4.0 / Digital footprint lens | `P_stock_report_02` mục 2.6.5 (Three Golden Steps CFA + 7-dimension readiness table) |
| GVC governance type (Gereffi 2005) | `P_stock_report_02` mục 2.6.3 (market/modular/relational/captive/hierarchy) |
| CFA Sector Analysis 2020 industry mapping | `P_stock_report_02` mục 2.6.9 (21 CFA chapter ↔ 18 ngành VN whitelist + 3 financial + 2 override) |
