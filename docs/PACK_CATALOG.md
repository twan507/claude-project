# Pack Catalog

Catalog chi tiết từng pack/file trong 3 agent. Mỗi pack ghi: mục đích, files, dependencies, key constraints, output naming. Cập nhật: 2026-05-30.

---

## analysis_agent — Layered K/P/O Architecture

Multi-pack analyst agent dùng kernel routing pattern. Đọc `KERNEL_SKELETON.md` đầu session để route, sau đó activate pack tương ứng.

### Knowledge layer

#### `K_agent_db` — 6 files

**Mục đích:** Knowledge base về MongoDB `agent_db` chứng khoán VN. Schema 25 collection, query patterns, anti-patterns, methodology diễn giải chỉ báo + tin tức.

| File | Nội dung |
|---|---|
| `K_agent_db_00` | Master — mục đích, scope, manifest, domain rules, K hygiene, quy đổi đơn vị, output rules, **whitelist 18 ngành default + override** (mục 4.5) |
| `K_agent_db_01` | Schema 25 collection (8 khối: Stock/Industry/Group/Market/History/News/Other/Briefing) + URL pattern finext.vn + **Xếp hạng ngành tự tổng hợp** |
| `K_agent_db_02` | Query patterns 12 workflow (A-L) — bao gồm `Section 3.6` rank ngành tự tổng hợp aggregate |
| `K_agent_db_03` | Anti-patterns — case study lỗi quá khứ + cách sửa |
| `K_agent_db_04` | Methodology diễn giải chỉ báo (dòng tiền, trend đa khung, technical zone, PTCB 4 type doanh nghiệp) |
| `K_agent_db_05` | News methodology (4 loại tin, framework chấm impact, case study, bảng dịch thuật ngữ EN) |

**Dependencies:** Không (knowledge layer độc lập).

**Status:** ✅ Active. Schema được sync với `db_agent/agent_db_*` (cùng schema, framing khác).

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

### Meta files (analysis_agent root)

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules vận hành agent (paste vào Custom Instructions Claude project) — kiến trúc 3 layer K/P/O + index, master-first reading, render binary workflow, font Roboto |
| `KERNEL_SKELETON.md` | File index — liệt kê pack có sẵn + trigger activation. Agent đọc đầu session. |

---

## db_agent — Monolithic Knowledge Base

Single-shot stock analyst agent. Knowledge base 6 file phẳng (không có layered architecture). Dùng để tra cứu DB ad-hoc.

### Files

| File | Nội dung |
|---|---|
| `system_prompt.md` | Vai trò agent + tone + manifest 6 file + 4 meta-rules bất biến |
| `agent_db_00` | Master — mục đích, scope, manifest, domain rules (gồm 4.5 whitelist 18 ngành default + override), K hygiene, quy đổi đơn vị |
| `agent_db_01` | Schema 25 collection (đồng bộ với `K_agent_db_01`) — schema mỗi collection, **Xếp hạng ngành tự tổng hợp**, whitelist callout, History block |
| `agent_db_02` | Query patterns 12 workflow (đồng bộ với `K_agent_db_02`) |
| `agent_db_03` | Anti-patterns (đồng bộ với `K_agent_db_03`) |
| `agent_db_04` | Methodology diễn giải chỉ báo (đồng bộ với `K_agent_db_04`) |
| `agent_db_05` | News methodology (đồng bộ với `K_agent_db_05`) |

**Dependencies:** Không (standalone).

**Đồng bộ DB với analysis_agent (CHỈ là cùng query 1 DB, không phải shared knowledge):** schema content giống `K_agent_db_*` vì 2 agent cùng query MongoDB `agent_db`. KHÔNG có cross-reference giữa 2 agent. Mỗi agent đọc knowledge của riêng mình.

Khác biệt giữ riêng:
- Naming prefix `agent_db_*` (không có `K_` tiền tố)
- Framing dùng "bộ tài liệu này" thay vì "pack"
- Cross-refs nội bộ dùng `agent_db_NN`
- Khi update schema, phải update cả 2 nơi riêng biệt (không tự sync)

**Status:** ✅ Active. Schema đồng bộ với DB 2026-05-30.

---

## template_agent — Document Normalizer + Brander

Render binary (pptx/docx) branded. Nhận input bất kỳ (PDF / DOCX / MD / paste content) → chuẩn hoá theo `FORMAT.md` contract → hỏi pick brand → render binary.

### Files

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules agent (paste vào Custom Instructions) |
| `INDEX.md` | Manifest + workflow tổng quan |
| `FORMAT.md` | Spec MD chuẩn hoá — 9 report_types với section structure |
| `WORKFLOW.md` | Flow 7 stage + 3 checkpoint (Pre-flight → Normalize → Brand pick → Layout pick → Approve → Render → Output) |
| `TEMPLATE_VBSE.md` + `.pptx` | Catalog 27 layout brand VBSE (navy + đỏ + tam giác vuông cân) |
| `TEMPLATE_FINEXT.md` + `.pptx` | Catalog 27 layout brand Finext (xanh lá + xám) |

**Lưu ý:** 2 file `.pptx` không upload được vào project knowledge của Claude Desktop — user attach trong chat session khi cần render binary.

**Dependencies:** Không (standalone). Nhận input từ analysis_agent (qua copy/paste) hoặc nguồn ngoài.

**Status:** ✅ Active. Không thay đổi trong session refactor 2026-05.

---

## Cross-references nhanh

| Khi cần | Mở file |
|---|---|
| Hiểu schema DB | `K_agent_db_01` (analysis) hoặc `agent_db_01` (db) |
| Viết báo cáo tuần broadcast | `P_weekly_overview_*` + `O_weekly_overview_00` |
| Viết chiến lược tháng deep nội bộ | `P_vbse_strategy_*` + `O_vbse_strategy_00` |
| Workflow đầu tư cá nhân deep-dive | `P_invest_memo_*` + `O_invest_memo_*` |
| Query DB ad-hoc | db_agent (monolithic) |
| Render branded binary | template_agent |
| Hiểu whitelist 18 ngành rule | `K_agent_db_00` mục 4.5 + `K_agent_db_01` Section B (đầu khối) |
| Hiểu rank ngành tự tổng hợp | `K_agent_db_01` mục "Xếp hạng ngành" + `K_agent_db_02` Section 3.6 |
| Hiểu Bucket entry (1/2/3) | `P_invest_memo_03` mục 5 (master definition) — `P_vbse_strategy_06` reference |
| Hiểu tỷ trọng PTKT trong pack | Master `P_vbse_strategy_00` mục 4 + `P_weekly_overview_00` mục 4 |
