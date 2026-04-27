# Audit Report — Claude Project — 2026-04-27

Audit comprehensive 3 agent (analysis_agent / template_agent / db_agent) theo plan tại [2026-04-27-audit-plan.md](2026-04-27-audit-plan.md). Depth L2 semantic, scope A+B+C+D, severity 4 mức.

---

## 0. Executive Summary

**Tổng số finding:** 24 (Critical: 2 / Major: 9 / Minor: 8 / Nit: 5) + 3 Confirm Intent

**Health overall:**
- **analysis_agent** — Architecture chắc, methodology sâu, có 1 Critical (Nguyên tắc 5 strict reject phá nhất quán flex+downgrade) + nhiều legacy section (Guide render docx/pptx) chưa cleanup theo lời hứa README.
- **template_agent** — FORMAT đầy đủ nhưng có drift nội bộ + cross-file: 1 Critical (FORMAT + INDEX + WORKFLOW + P_stock_pitch + O_stock_pitch không thống nhất count section của stock_pitch) + 2 Major drift (WORKFLOW thiếu cập nhật).
- **db_agent** — Sạch, ít issue. Sync với analysis_agent K pack tốt (drift được document có chủ đích).

**Top 3 cần ưu tiên xử lý:**
1. **F-04 (Critical)** — stock_pitch section count contradicts qua 5 file (15-18 vs 13-16 vs 12-15). Cần chốt 1 con số duy nhất.
2. **F-09 (Critical)** — `P_invest_memo_00` Nguyên tắc 5 (dòng tiền dương + catalyst tiêu cực → loại) là strict reject, mâu thuẫn triết lý flex+downgrade ở cả README mục 3.6/8.2 lẫn 5 nguyên tắc còn lại.
3. **F-02 (Major)** — README mục 1.1 sai số file analysis_agent (25 thực tế 29). User mới đọc README sẽ nhầm.

---

## 1. Index findings theo dimension

| ID | Sev | Dim | Title | File:line |
|---|---|---|---|---|
| F-01 | Major | A | INDEX section count cho stock_pitch sai | template_agent/INDEX.md:76 |
| F-02 | Major | A | README mục 1.1 sai tổng file analysis_agent | README.md:13 |
| F-03 | Major | A | WORKFLOW threshold "5 whitelist" cho frontmatter sai | template_agent/WORKFLOW.md:82 |
| F-04 | Critical | A | stock_pitch section count contradict qua 5 file | (đa file) |
| F-05 | Major | A | WORKFLOW Task 2 ref "deepdive/macro_sector/generic" lạc | template_agent/WORKFLOW.md:287 |
| F-06 | Major | A | KERNEL_SKELETON ref "VBSE" — leak cross-agent | analysis_agent/KERNEL_SKELETON.md:47 |
| F-07 | Minor | A | KERNEL_SKELETON section "T — Template packs" rỗng | analysis_agent/KERNEL_SKELETON.md:113 |
| F-08 | Nit | A | O_invest_memo_00 ref `PROJECT_STATUS` không tồn tại | analysis_agent/O_invest_memo_00.md:221 |
| F-09 | Critical | B | Nguyên tắc 5 strict reject mâu thuẫn flex+downgrade | analysis_agent/P_invest_memo_00.md:216-218 |
| F-10 | Major | B | WORKFLOW Task 4 spec date format "dd/mm/yyyy" trái FORMAT | template_agent/WORKFLOW.md:298 |
| F-11 | Minor | B | WORKFLOW Stage 3 confirm report_type chỉ 7 option (gộp portfolio_review_*) | template_agent/WORKFLOW.md:146 |
| F-12 | Nit | B | O_invest_memo_00 example violate locale rule của chính nó | analysis_agent/O_invest_memo_00.md:184,205 |
| F-13 | Major | B | system_prompt analysis_agent ref "JSON" nhưng pack chỉ dùng MD | analysis_agent/system_prompt.md:152 |
| F-14 | Confirm Intent | C | _04 và _05 có legacy comment "Rule 6/0" chỉ ở K_agent_db (analysis_agent) | (admit ở README mục 5.3) |
| F-15 | Confirm Intent | C | _00 mục 10 architecture khác (output contract vs output rules) | (admit ở README mục 5.3) |
| F-16 | Confirm Intent | C | _05 bước 8 có wording khác nhau cho output style | (admit ở README mục 5.3) |
| F-17 | Minor | D | 13 section "Guide render docx/pptx" chưa marked [LEGACY] | analysis_agent/O_invest_memo_*.md, O_weekly_market_00.md |
| F-18 | Minor | D | `/mnt/user-data/outputs/` — 6 file ref legacy path | (đa file) |
| F-19 | Minor | D | `present_files` — 7 file ref legacy tool | (đa file) |
| F-20 | Minor | D | `/mnt/skills/public/docx,pptx` — ref legacy skill | analysis_agent/O_invest_memo_00.md:85 |
| F-21 | Minor | D | "render binary" out-of-scope nói trong KERNEL_SKELETON ngược với O_invest_memo có nhiều mục render | (note nhất quán hoá) |
| F-22 | Major | A | INDEX brand whitelist mục có visual signature, không đồng bộ với system_prompt mục 5.4 | template_agent/INDEX.md:64 |
| F-23 | Nit | B | Số luận điểm "tóm tắt mạnh nhất" cap 4 nhưng bug khi N=7 | analysis_agent/P_stock_pitch_00.md:486-487 |
| F-24 | Nit | A | "9 loại báo cáo support" header nhưng đếm "8 preset + 1 custom" | template_agent/INDEX.md:70 |

---

## 2. Index findings theo severity

| ID | Sev | Title | Effort fix |
|---|---|---|---|
| F-04 | Critical | stock_pitch section count contradict qua 5 file | M |
| F-09 | Critical | Nguyên tắc 5 strict reject mâu thuẫn flex+downgrade | S |
| F-01 | Major | INDEX section count cho stock_pitch sai | S |
| F-02 | Major | README mục 1.1 sai tổng file analysis_agent | S |
| F-03 | Major | WORKFLOW "5 whitelist" sai (đáng lẽ 9) | S |
| F-05 | Major | WORKFLOW Task 2 ref tên type cũ | S |
| F-06 | Major | KERNEL_SKELETON ref "VBSE" cross-agent | S |
| F-10 | Major | WORKFLOW spec date "dd/mm/yyyy" trái FORMAT | S |
| F-13 | Major | system_prompt ref "JSON" speculative | S |
| F-22 | Major | INDEX brand whitelist visual signature drift | S |
| F-07 | Minor | KERNEL_SKELETON "T — Template packs" rỗng | S |
| F-11 | Minor | WORKFLOW Stage 3 confirm chỉ 7 option (gộp 3 review) | S |
| F-17 | Minor | "Guide render docx/pptx" chưa marked [LEGACY] | M |
| F-18 | Minor | `/mnt/user-data/outputs/` — 6 file legacy | S |
| F-19 | Minor | `present_files` — 7 file legacy | S |
| F-20 | Minor | `/mnt/skills/public/` legacy skill | S |
| F-21 | Minor | render binary scope không nhất quán | M |
| F-08 | Nit | `PROJECT_STATUS` ref không tồn tại | S |
| F-12 | Nit | O_invest_memo_00 example violate locale | S |
| F-23 | Nit | "4 luận điểm tóm tắt mạnh nhất" cap khi N=7 | S |
| F-24 | Nit | "9 loại báo cáo" header / "8 preset + 1 custom" body | S |

---

## 3. Findings detail

### Phase A — Consistency

#### F-01 [Major] — INDEX section count cho stock_pitch sai

**What:** [INDEX.md:76](../../template_agent/INDEX.md#L76) ghi `stock_pitch | Khuyến nghị MUA mã đơn lẻ gửi KH | 15-18 (rigid + 3 luận điểm flex) | 12-18 trang`. "3 luận điểm flex" là sai — spec ở [P_stock_pitch_00.md:9](../../analysis_agent/P_stock_pitch_00.md#L9) và [README.md:144](../../README.md#L144) là 4-7 luận điểm flex.

**Why it matters:** User đọc INDEX để hiểu capability template_agent có thể nhầm thành flex 3 luận điểm. Ngoài ra "15-18" tổng số mục cũng sai (đúng là 13-16).

**Suggested fix:** Sửa INDEX.md:76 thành `13-16 (rigid + 4-7 luận điểm flex)`.

---

#### F-02 [Major] — README mục 1.1 sai tổng file analysis_agent

**What:** [README.md:13](../../README.md#L13) ghi analysis_agent có **25 file**. Đếm thực tế filesystem: **29 file** (1 KERNEL + 6 K + 9 O + 12 P + 1 system_prompt). README mục 3.2 (bảng pack) lại đúng — sum của bảng đó cho ra 27 pack files + KERNEL + system_prompt = 29. Internal contradiction trong chính README.

**Why it matters:** Khi user/AI session mới vào project, README là điểm vào đầu tiên. Số sai làm sai expectation upload. README mục 2.1 đề xuất "upload 24 file" (= 25-1) — user làm theo sẽ thiếu 4 file knowledge.

**Suggested fix:** Sửa README.md:13 và README.md:42 thành `29` và `28` tương ứng. Verify lại số ở các bảng khác.

---

#### F-03 [Major] — WORKFLOW threshold "5 whitelist" cho frontmatter sai

**What:** [WORKFLOW.md:82](../../template_agent/WORKFLOW.md#L82) trong bảng Stage 1.5 detection signals ghi `Frontmatter | Top file có '---' block với report_type thuộc 5 whitelist`. Nhưng FORMAT.md mục 2.1 + INDEX.md đều xác nhận có **9 loại report_type** (8 preset + 1 custom).

**Why it matters:** Detection logic skip-normalize sẽ false-negative cho 4 type không nằm trong "5 whitelist" — agent từ chối skip dù input đúng contract.

**Suggested fix:** Sửa thành `9 whitelist`.

---

#### F-04 [Critical] — stock_pitch section count contradict qua 5 file

**What:** Cùng 1 spec stock_pitch nhưng 5 file ghi khác nhau:

| File | Claim |
|---|---|
| [README.md:144,156](../../README.md#L144) | 13-16 mục (4-7 luận điểm) |
| [P_stock_pitch_00.md:9](../../analysis_agent/P_stock_pitch_00.md#L9) | 13-16 mục (4 LĐ=13, 5=14, 6=15, 7=16) |
| [O_stock_pitch_00.md:3](../../analysis_agent/O_stock_pitch_00.md#L3) | 13-16 mục |
| [O_stock_pitch_00.md:66](../../analysis_agent/O_stock_pitch_00.md#L66) | "15-18 section tổng" (cùng file mâu thuẫn line 3!) |
| [INDEX.md:76](../../template_agent/INDEX.md#L76) | 15-18 (3 luận điểm flex) |
| [FORMAT.md:91](../../template_agent/FORMAT.md#L91) | "(15 sections, rigid)" header + body 4-7 LĐ |
| [FORMAT.md:503](../../template_agent/FORMAT.md#L503) | "13-16 section flex theo số luận điểm 4-7" (cùng file mâu thuẫn line 91!) |
| [WORKFLOW.md:84](../../template_agent/WORKFLOW.md#L84) | "stock_pitch 15±3" → 12-18 |

**Why it matters:** template_agent runtime detect skip-normalize (Stage 1.5) match section count với spec — nếu spec drift, false-negative phổ biến. Self-audit ở FORMAT mục 9 và P_stock_pitch mục 14 cũng dùng số khác nhau, agent self-audit có thể pass ở 1 nguồn fail ở nguồn khác.

**Suggested fix:** Chốt 1 con số canonical (đề xuất **13-16 với 4-7 luận điểm**, theo P_stock_pitch_00 — file authority cao nhất cho pack này). Sửa đồng bộ:
- INDEX.md:76 → "13-16 (rigid + 4-7 luận điểm flex)"
- O_stock_pitch_00.md:66 → bỏ câu "11-14 + 4 = 15-18"
- FORMAT.md:91 header → "(13-16 sections, flex 4-7 luận điểm)"
- WORKFLOW.md:84 → "stock_pitch 13-16"

---

#### F-05 [Major] — WORKFLOW Task 2 ref "deepdive / macro_sector / generic"

**What:** [WORKFLOW.md:287](../../template_agent/WORKFLOW.md#L287) ghi `H2 = section theo rigid structure (nếu stock_pitch / weekly_market) hoặc flex (deepdive / macro_sector / generic)`. Tên `deepdive`, `macro_sector`, `generic` **KHÔNG** nằm trong 9 report_type ở FORMAT mục 2.1. Có vẻ là tên cũ trước khi rename thành `stock_memo`, `market_scan`, `custom`.

**Why it matters:** Drift giữa WORKFLOW và FORMAT. Agent đọc WORKFLOW khi normalize có thể tạo MD với tên type cũ → vi phạm FORMAT contract.

**Suggested fix:** Sửa thành: `flex (stock_memo / market_scan / portfolio_plan / portfolio_review_* / custom)`.

---

#### F-06 [Major] — KERNEL_SKELETON ref "VBSE" — leak cross-agent

**What:** [KERNEL_SKELETON.md:47](../../analysis_agent/KERNEL_SKELETON.md#L47) trong description của P_weekly_market: `**Không sử dụng chỉ báo trend** (...) — phần này dành cho báo cáo bên VBSE.` "VBSE" là brand ở template_agent (mục 4.2 README). analysis_agent đáng lẽ độc lập 100% với template_agent (README mục 1.3).

**Why it matters:** Vi phạm "agent độc lập 100%" rule. AI session đọc KERNEL của analysis_agent sẽ tự hỏi VBSE là gì → confusion.

**Suggested fix:** Bỏ ", phần này dành cho báo cáo bên VBSE" — nếu cần giải thích lý do, viết generic: "phần này thuộc methodology nội bộ analyst, không dành cho audience KH cuối".

---

#### F-07 [Minor] — KERNEL_SKELETON section "T — Template packs" rỗng

**What:** [KERNEL_SKELETON.md:113](../../analysis_agent/KERNEL_SKELETON.md#L113) có header `## T — Template packs` không có nội dung gì bên dưới. Section ngay sau là "## Render binary — out of scope".

**Why it matters:** Orphan section, có thể là vestige của design cũ (khi định gộp T pack vào analysis_agent rồi đổi ý).

**Suggested fix:** Xoá header "T — Template packs" hoặc thay bằng comment ngắn giải thích "T pack moved to template_agent project".

---

#### F-08 [Nit] — O_invest_memo_00 ref `PROJECT_STATUS` không tồn tại

**What:** [O_invest_memo_00.md:221](../../analysis_agent/O_invest_memo_00.md#L221) ghi `**Number formatting (locale vi-VN, sync PROJECT_STATUS root convention):**`. File `PROJECT_STATUS` không tồn tại trong filesystem dự án.

**Why it matters:** Reference dead, có thể từ deployment cũ hoặc convention file khác.

**Suggested fix:** Thay bằng `sync README.md mục 7.1 Locale vi-VN` (file thay thế chính xác trong dự án hiện tại).

---

#### F-22 [Major] — INDEX brand whitelist visual signature drift

**What:** [INDEX.md:64-66](../../template_agent/INDEX.md#L64) ghi visual signature:
- VBSE: `Navy #003D7A + đỏ #D71249 + tam giác vuông cân`
- Finext: `Dark #0A0A0A + violet #8B5CF6 + chevron >>`

system_prompt mục 5.4 không nêu màu cụ thể, chỉ "VBSE và Finext". README mục 4.2 mô tả nhẹ hơn: `navy + đỏ + tam giác vuông cân` / `dark + violet + chevron >>` (không có hex). Hex code chỉ xuất hiện ở INDEX.

**Why it matters:** Single source of truth cho design tokens phải là `TEMPLATE_VBSE.md` / `TEMPLATE_FINEXT.md` (catalog file). Hex ở INDEX là duplicate, dễ drift khi sửa template.

**Suggested fix:** Bỏ hex code khỏi INDEX:64-66, giữ description ngôn ngữ tự nhiên thôi. Hex giữ duy nhất trong TEMPLATE_*.md.

---

#### F-24 [Nit] — INDEX "9 loại báo cáo" header / body "8 preset + 1 custom"

**What:** [INDEX.md:70](../../template_agent/INDEX.md#L70) header `## 9 loại báo cáo support`, body `8 preset type + 1 custom`. Bảng dưới list đúng 9 entry (gồm `custom`). OK về số, nhưng wording "8 preset + 1 custom" chỉ làm reader phải tự cộng để hiểu "9".

**Why it matters:** Cosmetic; reader đọc lướt có thể tưởng "9" và "8" là hai count khác nhau.

**Suggested fix:** Đổi thành `9 loại báo cáo (8 preset + 1 custom quiz-driven)`.

---

### Phase B — Methodology integrity

#### F-09 [Critical] — Nguyên tắc 5 strict reject mâu thuẫn flex+downgrade

**What:** [P_invest_memo_00.md:216-218](../../analysis_agent/P_invest_memo_00.md#L216) Nguyên tắc 5: `Dòng tiền dương + catalyst tiêu cực → loại, không bàn thêm. (...) Quy tắc: loại thẳng, không override (...)` — đây là **strict reject**.

Nhưng:
- README mục 3.6: `triết lý flex+downgrade ... agent KHÔNG tự reject mã ... User quyết định cuối`
- README mục 8.2: `Triết lý flex+downgrade thay strict reject (analysis_agent) ... agent không tự reject`
- Nguyên tắc 1 (P_invest_memo_00:202): "flag cảnh báo + downgrade conviction 1 bậc ... user quyết định proceed"
- Nguyên tắc 4 (P_invest_memo_00:214): "flag cảnh báo + downgrade size 30-50% ... User quyết định cuối"

Nguyên tắc 5 là OUTLIER: là nguyên tắc duy nhất trong 6 nguyên tắc bất biến vẫn dùng strict reject.

**Why it matters:** Agent bị contradicting instruction. Khi gặp case "dòng tiền dương + catalyst tiêu cực", phải chọn theo nguyên tắc 5 (loại) hay theo triết lý chung (flag + user quyết)? Inconsistency làm agent runtime behavior không deterministic.

**Suggested fix:** Một trong 2 hướng:
1. **Convert sang flex+downgrade**: "Dòng tiền dương + catalyst tiêu cực → flag CẢNH BÁO MẠNH (likely retail trap) + downgrade conviction 2 bậc + recommend strongly ngại loại. User quyết định cuối."
2. **Giữ strict reject** nhưng explicit ở README mục 8.2 + mục 3.6 rằng "trừ Nguyên tắc 5 vẫn strict reject vì pattern này có base rate lỗi rất cao".

Đề xuất hướng 2 (giữ behavior, thêm exception explicit) — vì pattern này bản chất khác Variant Perception/Bear Case (đó là chủ quan, đây là pattern objective historical).

---

#### F-10 [Major] — WORKFLOW Task 4 spec date "dd/mm/yyyy"

**What:** [WORKFLOW.md:298](../../template_agent/WORKFLOW.md#L298) Task 4 Locale vi-VN: `Date format dd/mm/yyyy`. Trái với:
- [FORMAT.md:454](../../template_agent/FORMAT.md#L454): `Date: Q1/2026, tháng 4/2026, ngày 27/4/2026. Không dùng "Q1 2026" hay "2026-04-15" trong prose (OK trong bảng)`
- README.md:333: `Date: Q1/2026, tháng 4/2026, ngày 27/4/2026`

**Why it matters:** Agent normalize sẽ produce MD với date sai locale theo WORKFLOW, fail FORMAT contract khi self-audit.

**Suggested fix:** Sửa WORKFLOW.md:298 thành `Date: ngày DD/MM/YYYY trong prose, Q1/YYYY hoặc tháng MM/YYYY khi nói quý/tháng`.

---

#### F-11 [Minor] — WORKFLOW Stage 3 confirm chỉ 7 option (gộp portfolio_review_*)

**What:** [WORKFLOW.md:146](../../template_agent/WORKFLOW.md#L146) ví dụ option "(a) stock_pitch (b) weekly_market (c) market_scan (d) stock_memo (e) portfolio_plan (f) review portfolio (g) custom" — **7 option**, gộp 3 portfolio_review_* (weekly/monthly/quarterly) thành 1.

**Why it matters:** User confirm "review portfolio" thì agent vẫn phải hỏi tiếp "tuần/tháng/quý" — thêm 1 round-trip. Hoặc agent default sang 1 type cụ thể, dễ sai.

**Suggested fix:** Tách thành 9 option đầy đủ, hoặc giữ 7 option và bổ sung sub-question khi pick (f).

---

#### F-12 [Nit] — O_invest_memo_00 example violate locale rule của chính nó

**What:** [O_invest_memo_00.md:184,205](../../analysis_agent/O_invest_memo_00.md#L184) có 2 ví dụ: `Revenue VNM 2025 đạt 18,200 tỷ VND` và `Revenue 2025 đạt 18,200 tỷ VND`. Đây dùng dấu phẩy ngăn nghìn (American). Nhưng chính file này (mục 6 line 222) quy định: `Dấu chấm ngăn cách nghìn: "18.200 tỷ" không "18200 tỷ" hay "18,200 tỷ"`.

**Why it matters:** Inconsistency nội bộ trong file master. AI session đọc spec sẽ thấy contradiction: rule nói chấm, ví dụ dùng phẩy.

**Suggested fix:** Sửa 2 ví dụ thành `18.200 tỷ VND`. File này có comment "ví dụ trong các file con (...) hiện có thể còn dùng convention cũ" admit debt — nhưng debt này nằm chính trong master file.

---

#### F-13 [Major] — system_prompt analysis_agent ref "JSON" speculative

**What:** [system_prompt.md:152](../../analysis_agent/system_prompt.md#L152) mục 8 Interface contracts: `P sinh structured content (markdown có schema rõ, hoặc JSON nếu pack quy định)`. Nhưng không có pack P nào trong analysis_agent dùng JSON — toàn bộ output là MD.

**Why it matters:** Speculative spec, có thể mislead AI/dev tin rằng JSON là output format hợp lệ.

**Suggested fix:** Xoá clause "hoặc JSON nếu pack quy định". Nếu tương lai muốn support JSON, thêm lại khi có pack thực sự cần.

---

#### F-23 [Nit] — "Số luận điểm tóm tắt mạnh nhất cap 4" có bug khi N=7

**What:** [P_stock_pitch_00.md:486-487](../../analysis_agent/P_stock_pitch_00.md#L486) Mục 2 spec: `04 [Tiêu đề L4 nếu có, max 4 luận điểm in tóm tắt — nếu báo cáo có 5-7 luận điểm thì gộp vào 4 tóm tắt mạnh nhất]`. Khi N=7, "gộp" 7 luận điểm vào 4 tóm tắt là việc subjective; spec không rõ cách gộp.

**Why it matters:** Agent runtime sẽ improvise → output không deterministic giữa các session cùng input.

**Suggested fix:** Một trong:
1. Cap "4 luận điểm cốt lõi cho phép tóm tắt" và rõ rule chọn 4 mạnh nhất theo headline data point lớn nhất.
2. Cho phép 4-N tóm tắt (linh hoạt theo N).

---

### Phase C — Cross-agent sync

#### F-14 [Confirm Intent] — _04 và _05 có legacy comment "Rule 6/0" chỉ ở K_agent_db (analysis_agent)

**What:** [K_agent_db_04.md:14](../../analysis_agent/K_agent_db_04.md#L14) có paragraph `**Lưu ý về nhãn "Rule 6"...**` (4 dòng). agent_db_04 không có. Tương tự [K_agent_db_05.md:9-15](../../analysis_agent/K_agent_db_05.md#L9) có paragraph `**Lưu ý về nhãn "Rule N"...**`. agent_db_05 không có.

**Status:** README mục 5.3 đã admit: "1 vài legacy comment ('Rule 6') trong K_agent_db_04 (analysis_agent)". → Confirm intent nếu user vẫn muốn giữ; nếu muốn cleanup full thì remove.

**Suggested action:** User confirm — giữ (acceptable debt) hay clean up (xoá 2 paragraph này khỏi K_agent_db_04 và K_agent_db_05)?

---

#### F-15 [Confirm Intent] — _00 mục 10 architecture khác

**What:**
- [K_agent_db_00.md:227-237](../../analysis_agent/K_agent_db_00.md#L227): mục 10 = `Output contract — Pack này sinh ra structured content để layer trên (P pack, O pack) tiêu thụ. (...) Pack KHÔNG tự quyết format output cuối...`
- [agent_db_00.md:227-235](../../db_agent/agent_db_00.md#L227): mục 10 = `Output rules khi trả lời user — Mọi output tới user phải đảm bảo: ...`

**Status:** Đây là khác biệt CÓ CHỦ ĐÍCH theo architectural — analysis_agent có layered K/P/O, db_agent là single-shot conversational. README mục 5.3 mention briefly nhưng không call out cụ thể chỗ này.

**Suggested action:** Confirm intent. Đề xuất add 1 dòng note ở README mục 5.3 cho rõ: "_00 mục 10 ở 2 agent có nội dung khác — analysis_agent là output contract cho layer trên, db_agent là output rules trực tiếp cho user".

---

#### F-16 [Confirm Intent] — _05 bước 8 wording khác về output style

**What:**
- [K_agent_db_05.md:545](../../analysis_agent/K_agent_db_05.md): `Format output theo layer style đang active (O pack nếu có, hoặc Default của Kernel theo system prompt mục 6).`
- [agent_db_05.md:538](../../db_agent/agent_db_05.md): `Tone và format theo system prompt mục 2 — direct, expert-level, concise, evidence-based.`

**Status:** Khác biệt do architecture khác — analysis_agent có O pack layer, db_agent không. Intentional.

**Suggested action:** Confirm intent. Document briefly nếu cần.

---

### Phase D — Legacy / dead-code

#### F-17 [Minor] — 13 section "Guide render docx/pptx" chưa marked [LEGACY]

**What:** README mục 8.1 và mục 10 admit: "Một số file P/O packs ... còn legacy section 'Guide render docx/pptx' được marked `[LEGACY]`". Thực tế:

| File | Section | Có [LEGACY] marker? |
|---|---|---|
| O_invest_memo_01.md:339,355 | Guide render docx/pptx | ❌ |
| O_invest_memo_02.md:485,504 | Guide render docx/pptx | ❌ |
| O_invest_memo_03.md:330,348 | Guide render docx/pptx | ❌ |
| O_invest_memo_04.md:258,272 | Guide render docx/pptx | ❌ |
| O_invest_memo_05.md:390,406 | Guide render docx/pptx | ❌ |
| O_invest_memo_06.md:399,417 | Guide render docx/pptx | ❌ |
| O_weekly_market_00.md:573,590 | Guide render docx/pptx | ❌ |
| O_stock_pitch_00.md:525 | Guide render pptx | ✅ `[LEGACY]` |
| O_stock_pitch_00.md:568 | Guide render docx (optional) | ❌ |

**Why it matters:** README claim "marked [LEGACY]" không đúng cho 13/14 section. AI/dev đọc thấy section đầy đủ format, có thể tưởng spec runtime hợp lệ và áp dụng → output sai (render binary out-of-scope).

**Suggested fix:** Hoặc:
1. Add `[LEGACY]` marker vào 13 section còn lại (1-line fix mỗi file).
2. Xoá toàn bộ 13 section (commit cleanup pass riêng) — README mục 8.1 đã chốt rev 6 "MD final là output cuối" nên các section này không cần thiết.

Đề xuất hướng 2 (xoá hẳn) vì giữ "[LEGACY]" trong knowledge base vẫn ăn token và có thể confuse AI runtime.

---

#### F-18 [Minor] — `/mnt/user-data/outputs/` — 6 file ref legacy path

**What:** Path Linux từ deployment Claude Code/Skill cũ, không tồn tại trên Claude Desktop:

| File:line | Context |
|---|---|
| analysis_agent/P_stock_pitch_00.md:547 | `save vào /mnt/user-data/outputs/, gọi present_files` |
| analysis_agent/P_weekly_market_00.md (multiple) | trong workflow render |
| analysis_agent/O_stock_pitch_00.md:521,566 | save MD và pptx |
| analysis_agent/O_weekly_market_00.md (multiple) | save reports |
| template_agent/WORKFLOW.md:379,430 | Stage 5 + 7 save |
| README.md:481 | Note legacy ở mục 10 (đã admit) |

**Why it matters:** Output flow đúng trên Claude Desktop là return MD trong message. Path này khi agent runtime gặp sẽ confuse (path nào thực ra exist?).

**Suggested fix:** Thay bằng `[output trong message để user copy/save thủ công]` hoặc clean up cả block save.

---

#### F-19 [Minor] — `present_files` tool — 7 file ref

**What:** Tool `present_files` không tồn tại trên Claude Desktop. Files: P_stock_pitch_00, P_weekly_market_00, O_invest_memo_00, O_stock_pitch_00, O_weekly_market_00, template_agent/WORKFLOW.md (line 430), README (admit).

**Why it matters:** Tương tự F-18.

**Suggested fix:** Xoá ref hoặc replace bằng "agent return file content trong message".

---

#### F-20 [Minor] — `/mnt/skills/public/docx,pptx` — legacy skill ref

**What:** [O_invest_memo_00.md:85](../../analysis_agent/O_invest_memo_00.md#L85) Bước 7: `Đọc skill /mnt/skills/public/docx hoặc /mnt/skills/public/pptx trước khi build`. Skill path này từ deployment Claude Code, không có trên Claude Desktop.

**Why it matters:** Tương tự F-18, F-19. Cũng là dấu hiệu "Bước 7" này là legacy section (render binary out-of-scope).

**Suggested fix:** Xoá toàn bộ Bước 7 + 8 (render binary + present_files) khỏi mục 3 Workflow render output, vì rev 6 đã chốt MD final là output cuối. Workflow chỉ còn Bước 1-6 (đến compose MD).

---

#### F-21 [Minor] — render binary scope không nhất quán

**What:** Có sự không nhất quán giữa các statement scope:

- KERNEL_SKELETON.md:115-117: `Render binary — out of scope` (rõ ràng)
- system_prompt.md:54-57: từ chối render pptx/docx/xlsx (rõ ràng)
- O_invest_memo_00 vẫn có mục 5 (3 format MD/Docx/Pptx), mục 7 (chart annotation render docx/pptx), Bước 7 render binary (legacy)
- O_invest_memo_00 box top: admit "Mục pptx/docx render ở các file con là legacy spec, sẽ được dọn"

**Why it matters:** Master O pack vừa tuyên bố out-of-scope vừa giữ đủ spec render. Reader phải đọc kỹ box admit mới hiểu.

**Suggested fix:** Đồng bộ với F-17 — cleanup pass xoá hẳn render binary spec khỏi O packs. Hoặc nếu giữ, gộp tất cả vào 1 section "[LEGACY] Render binary (out-of-scope)" để dễ reader biết tránh.

---

## 4. Methodology

### Files đã đọc full (10 file)
- README.md
- CLAUDE.md
- analysis_agent: system_prompt, KERNEL_SKELETON, K_agent_db_00, P_invest_memo_00, P_stock_pitch_00 (full), O_invest_memo_00, O_stock_pitch_00 (head + tail spot)
- template_agent: system_prompt, INDEX, FORMAT, WORKFLOW
- db_agent: system_prompt, agent_db_00

### Files spot-checked (grep / spot read)
- analysis_agent/K_agent_db_01..05 (line counts + diff với agent_db)
- db_agent/agent_db_01..05 (diff với K_agent_db)
- analysis_agent/O_invest_memo_01..06 (grep render section)
- analysis_agent/O_weekly_market_00, P_weekly_market_00 (grep trend ban)

### Files NOT deep-read (out of L2 scope)
- analysis_agent/P_invest_memo_01..09 (file con tier — chỉ inspect master)
- analysis_agent/K_agent_db_01..05 detail content (chỉ diff)
- template_agent/TEMPLATE_VBSE.md, TEMPLATE_FINEXT.md (catalog file, low audit priority)

### Tools dùng
- `Grep` (ripgrep) cho dead-code patterns + cross-reference
- `diff -u` cho cross-agent sync
- `wc -l` cho size compare
- File read full cho master files

### Risk acknowledged
Phase B chỉ đến L2 — không đọc kỹ P_invest_memo_01..09. Có thể miss methodology drift trong file con. Đề xuất Phase B+ ở session sau nếu user muốn cover.

---

## 5. Recommendations

### 5.1. Priority order fix (suggest)

**Wave 1 — Critical/blocking (1 commit, S effort):**
- F-09 (decision: hướng 1 hay 2 cho Nguyên tắc 5)
- F-04 (chốt 13-16, sửa 5 file)

**Wave 2 — Major consistency (1-2 commit, S-M effort):**
- F-01, F-02, F-03, F-05, F-10, F-13, F-22 — tất cả là sửa 1 line / 1 paragraph
- F-06 (KERNEL ref VBSE) — 1 line

**Wave 3 — Legacy cleanup (1 commit, M effort):**
- F-17, F-18, F-19, F-20, F-21 — gộp 1 cleanup pass xoá hẳn legacy sections + path/tool ref. Hứa "audit pass tới" ở README mục 8.1 chính là pass này.

**Wave 4 — Confirm Intent + Nit (1 commit, S effort):**
- F-14, F-15, F-16 — user xác nhận trước, fix nếu muốn cleanup
- F-07, F-08, F-11, F-12, F-23, F-24 — gộp cleanup nit

### 5.2. Fix nào nên gộp / tách

- **Gộp 1 commit:** Wave 2 (đều là 1-line fix consistency) — gọi 1 commit "doc: fix consistency drift across 3 agents"
- **Gộp 1 commit:** Wave 3 legacy cleanup — gọi 1 commit "cleanup: remove legacy render binary spec + dead path/tool refs"
- **Tách commit riêng:** F-09 (Nguyên tắc 5) — quyết định methodology, deserve own commit + decision note
- **Tách commit riêng:** F-04 (stock_pitch count) — touches 5 file, deserve own commit

### 5.3. Fix nào cần user confirm intent trước

- F-14, F-15, F-16 — đã document "Confirm Intent". User confirm hướng (giữ hay clean up) trước khi fix.
- F-09 — user quyết hướng 1 hay 2 (convert flex+downgrade hay keep strict + document exception).
- F-21 — user quyết extent cleanup render binary (chỉ marker [LEGACY] hay xoá hẳn section).

---

## 6. Appendix

### 6.1. Bảng line count K_agent_db ↔ agent_db

| File | K_agent_db | agent_db | Diff lines |
|---|---|---|---|
| _00 | 237 | 235 | 2 |
| _01 | 992 | 992 | 0 |
| _02 | 1420 | 1420 | 0 |
| _03 | 328 | 328 | 0 |
| _04 | 1106 | 1104 | 2 |
| _05 | 565 | 558 | 7 |

Tất cả diff đều là expected (filename ref + system_prompt ref + 2 paragraph legacy comment + mục 10 architecture). Không phát hiện drift methodology.

### 6.2. Filesystem actual count

```
analysis_agent/   29 files (1 KERNEL + 6 K + 9 O + 12 P + 1 system_prompt)
  - KERNEL_SKELETON.md
  - K_agent_db_00..05 (6)
  - O_invest_memo_00..06 (7)
  - O_stock_pitch_00, O_weekly_market_00 (2)
  - P_invest_memo_00..09 (10)
  - P_stock_pitch_00, P_weekly_market_00 (2)
  - system_prompt.md

template_agent/  8 files (6 .md + 2 .pptx)
  - system_prompt.md
  - INDEX.md, FORMAT.md, WORKFLOW.md
  - TEMPLATE_VBSE.md/.pptx, TEMPLATE_FINEXT.md/.pptx

db_agent/        7 files
  - system_prompt.md
  - agent_db_00..05 (6)
```

### 6.3. Pattern dead-code matches summary

| Pattern | Files matched | Severity |
|---|---|---|
| `/mnt/user-data/outputs/` | 6 (excl. README + audit-plan) | Minor |
| `present_files` | 7 (excl. README + audit-plan) | Minor |
| `[LEGACY]` marker | 1 (O_stock_pitch_00) | (note: should be 14) |
| `/mnt/skills/` | 1 (O_invest_memo_00) | Minor |
| `Guide render docx\|pptx` | 14 sections in 8 files | Minor (cluster) |
| `5 whitelist` | 1 (WORKFLOW.md:82) | Major |
| `deepdive\|macro_sector\|generic` | 1 (WORKFLOW.md:287) | Major |

---

## 7. Phase B+ — Deep methodology audit P_invest_memo_01..09

Audit phase mở rộng (run sau khi user request). Scope: 9 file con của pack `P_invest_memo` (~5900 dòng). Methodology: deep read, dimension D1-D8 (xem [Phase B+ prompt context](#)).

### 7.1. Summary Phase B+

**21 findings** (Critical: 1 / Major: 6 / Minor: 9 / Nit: 3 / Confirm Intent: 2)

**Top 3 most concerning:**
1. **F-B+01 (Critical)** — Master vs file 03 contradict liquidity threshold (10 tỷ vs 5 tỷ/phiên).
2. **F-B+02 (Major)** — Pervasive checkpoint-template drift: file 01-08 đều có 7-10 sections thay vì 6 phần master quy định.
3. **F-B+04 (Major)** — File 04/03 ADV examples drop `× N (2-4 phiên)` multiplier → undersize position 2-4×.

### 7.2. Files passing all dimensions

- **File 09** clean trừ F-B+10 (raw token alert templates) và F-B+16 (4-tuần timeout convention).
- **D2 (flex+downgrade)** mostly clean — file 07 Gate 1/2 exemplary; chỉ file 04 R3/R5 và file 05:196-197 deviate.
- **D7 (legacy patterns)** clean — không tìm thấy `/mnt/user-data/outputs`, `present_files`, `/mnt/skills/`, `output_format pdf|docx|xlsx` trong file 01-09. Wave 3 cleanup successful.
- **D5 (CP IDs)** clean — CP1, CP2, CP3, CP4, CP5A, CP5B, CP5C nhất quán file 01-07.
- **D8 (file naming)** clean — `tier{N}_<YYYYMMDD>_confirmed.md` consistent.

### 7.3. 6 nguyên tắc cross-reference scoreboard

| Nguyên tắc | Enforce | Status |
|---|---|---|
| 1. Variant perception flex+downgrade | file 07 Gate 1 | ✓ Correctly enforced |
| 2. Exit trigger before position | file 07 Gate 3 | ✓ Correctly enforced |
| 3. 5% ADV per phiên | file 08 Section 3.4 | ✓ formula đúng. Examples ở file 04, 03 drop N=2-4 multiplier (F-B+04) |
| 4. Bear case steelman flex+downgrade | file 07 Gate 2 | ✓ Correctly enforced |
| 5. Dòng tiền + catalyst tiêu cực STRICT REJECT | file 04 R4 | ✓ Correctly enforced (no drift) |
| 6. No auto-tier transition | end-session pattern across all CP | ✓ Correctly enforced |

### 7.4. Findings detail Phase B+

#### F-B+01 [Critical] — Master vs file 03 contradict liquidity threshold (5 vs 10 tỷ/phiên)

**File:** [P_invest_memo_00.md:158](../../analysis_agent/P_invest_memo_00.md#L158) vs [P_invest_memo_03.md:164](../../analysis_agent/P_invest_memo_03.md#L164)

**What:** Master line 158: `Funnel D Thanh khoản ≥ 10 tỷ/phiên`. File 03 D1 line 164: `Trading value trung bình 20 phiên ≥ 5 tỷ đồng/phiên`. Examples xuyên file 03 (line 510, 627) và file 04 (line 115, 441) consistently dùng 5 tỷ.

**Why it matters:** Universe filter D là hard reject. 2× difference threshold materially thay đổi universe size. Mã 5-10 tỷ/phiên pass theo file 03 nhưng fail theo master. CP3 output phụ thuộc vào doc nào agent load trước.

**Suggested fix:** Reconcile 1 con số. Đề xuất 5 tỷ (file con là practical source of truth, có nhiều examples). Sửa master:158 thành `≥ 5 tỷ/phiên`.

---

#### F-B+02 [Major] — Checkpoint template drift across all file con (7-10 sections vs master 6 phần)

**File:** P_invest_memo_01..08 (CP block của mỗi file)

**What:** Master mục 6.1 (line 230-241) định nghĩa khung **6 phần** cứng. Mỗi file template insert thêm phần non-standard:
- 01: 6 thành "Bảng catalyst active"
- 02: 6 thành "Flags bối cảnh chuyển sang tier 2"
- 03: 6 thành "Flags kỹ thuật chuyển sang tier 3"
- 04: 5+6+7 ("Mã bị loại bắt buộc", "Lựa chọn sát nút", "Flags quan trọng") = 8 sections
- 05: 5 thành "Key findings cho tier 5B/5C" = 7 sections
- 06: **10 sections** (thêm "Cross-check", "Sensitivity", "Discussion")
- 07: 7 sections (thêm "Gate assessment chi tiết")
- 08: 10 sections (Constraint check, Sequence entry, Limit orders, Diversification, Cảnh báo đặc biệt riêng)

**Why it matters:** Khung 6 phần master là audit/discipline anchor. File con drift → reader không biết nên audit theo doc nào. Discipline nguyên tắc 6 (checkpoint format) bị diluted.

**Suggested fix:** 2 hướng:
- **(a)** Update master mục 6.1 acknowledge tier-specific template có thể insert "data sections" giữa Phần 5 (Lựa chọn sát nút) và "Câu hỏi user", giữ 6 named anchors.
- **(b)** Restructure mỗi template fold non-standard content vào Phần 4 (Số liệu key) hoặc Phần 5 (Lựa chọn sát nút).

Đề xuất (a) — file con đã ổn định, sửa master ít disruption hơn.

---

#### F-B+03 [Major] — File 03 + 04 reference "tier 4-5" và "tier 5-6", contradict master "no Tier 4"

**File:** [P_invest_memo_03.md:14,61,331](../../analysis_agent/P_invest_memo_03.md#L14); [P_invest_memo_04.md:179,209,224,233,243,511,574](../../analysis_agent/P_invest_memo_04.md#L179) (7 occurrences)

**What:** Master line 122 explicitly: *"không có 'Tier 4'"*. File 03:14 ghi `chia 3 bucket dựa trên zone tuần+tháng cho tier 4-5 sizing`. File 04 có 7 lần `tier 5-6 sizing`. Sizing thực tế xảy ra ở Tier 6 (file 08).

**Why it matters:** "Tier 4-5" là stale leftover taxonomy cũ. "Tier 5-6" imprecise. Agent reader confused về cross-reference.

**Suggested fix:** Replace `tier 4-5 sizing` → `tier 6 sizing` (file 03:14). Replace `tier 5-6` → `tier 6` ở các vị trí ám chỉ portfolio sizing (file 04 nhiều chỗ).

---

#### F-B+04 [Major] — File 04 + 03 ADV examples drop N=2-4 phiên multiplier, undersize position 2-4×

**File:** [P_invest_memo_04.md:441](../../analysis_agent/P_invest_memo_04.md#L441), [P_invest_memo_04.md:115](../../analysis_agent/P_invest_memo_04.md#L115); [P_invest_memo_03.md:510,627](../../analysis_agent/P_invest_memo_03.md#L510)

**What:** Master Nguyên tắc 3 (line 210): `max tổng vị thế = 5% × ADV × N với N=2-4`. File 08:113-116 implements correctly. Nhưng examples ở file 04, 03 drop multiplier:
- File 04:441 — `trading value TB 7 tỷ/phiên — tier 5B sizing max 350 triệu/position (5% ADV)`. 350M = 5%×7tỷ chỉ cho 1 phiên. Nguyên tắc đúng → 700M-1.4B với N=2-4.
- File 04:115 — `5% ADV = 250-500 triệu — với portfolio 1 triệu USD, mã này chỉ vào được 2-4% size` (compute per-phiên, không phải total)
- File 03:510 — `5% ADV (max ~300 triệu/position)` (same error)
- File 03:627 — `tier 5 sizing 5% ADV = 250-400 triệu/mã` (same error)

**Why it matters:** Agent reading examples sẽ undersize position 2-4×, lead to systematic under-deployment. Conflict trực tiếp với file 08 formula.

**Suggested fix:** Update mỗi example show `max per phiên = X, max position với N=3 phiên = 3X` hoặc reference file 08 Section 3.4 cho công thức đầy đủ.

---

#### F-B+05 [Major] — File 04 Section 6 "hard reject" R3/R5 phá triết lý flex+downgrade master

**File:** [P_invest_memo_04.md:259-292](../../analysis_agent/P_invest_memo_04.md#L259)

**What:** Section 6 introduce 5 hard-reject categories R1-R5. Trong đó:
- R3 (scandal): `Loại thẳng, không kháng cự. User có thể override...`
- R5 (BCTC mismatch): `Loại`
- R4 = Nguyên tắc 5 strict reject (đúng theo master)

R3 và R5 là NEW strict-reject rules invent ở file con — master triết lý (line 200-214) là flex+downgrade trừ Nguyên tắc 5. R3, R5 không phải bất biến đã chốt.

**Why it matters:** File 07 Gate 1/2 implement flex+downgrade chuẩn (đúng master), nhưng file 04 flip sang strict reject cho governance/BCTC issue. R3 đặc biệt debatable — mã đang điều tra UBCK có thể warrant Watch+downgrade thay vì auto-reject.

**Suggested fix:** 2 hướng:
- **(a)** Elevate R3, R5 thành nguyên tắc bất biến level (update master mục 5 thành 8 nguyên tắc).
- **(b)** Convert thành `flag mạnh + downgrade conviction 2 bậc + user quyết định` consistent file 07 pattern. R4 (Nguyên tắc 5) giữ strict.

Đề xuất (b) — keep master tinh gọn, file con consistency.

---

#### F-B+06 [Major] — File 03 D constraint "không có ngoại lệ" wording không reference master Nguyên tắc 3

**File:** [P_invest_memo_03.md:196](../../analysis_agent/P_invest_memo_03.md#L196)

**What:** Line 196 — `Ngoại lệ: không có. D áp cứng cho mọi type và mọi regime`. Master line 158 confirm D không nới → consistent. Nhưng wording "không có exception" reads strict-reject without flagging là cascading từ Nguyên tắc 3 (slippage protection).

**Why it matters:** Reader không biết rule này là bất biến vs convention. Nếu user override D, agent không có cơ sở từ chối explicit ngoài "file ghi vậy".

**Suggested fix:** Add reference: `D constraint không override (cascading từ Nguyên tắc 3 master mục 5 — slippage protection per phiên)`.

---

#### F-B+07 [Minor] — File 02:197 "không cho phép override bất kể user nói gì" mâu thuẫn user override mechanism master

**File:** [P_invest_memo_02.md:197](../../analysis_agent/P_invest_memo_02.md#L197)

**What:** `Không cho phép override nếu tổng điểm catalyst < 6, bất kể user nói gì. Đây là rule cứng để tránh confirmation bias.`

**Why it matters:** Master mục 6.3 says user có quyền override BẤT KỲ checkpoint với audit log. Wording "bất kể user nói gì" contradict.

**Suggested fix:** `Agent không tự suy luận override; user có thể override với audit log nêu lý do cụ thể (insider info, theo dõi catalyst lâu) — Agent ghi cảnh báo confirmation bias risk` consistent file 01:502-509.

---

#### F-B+08 [Minor] — File 03:196 "Ngoại lệ không có" wording redundant với F-B+06 (combined)

**File:** [P_invest_memo_03.md:196](../../analysis_agent/P_invest_memo_03.md#L196)

**What:** Same line as F-B+06, different angle — phrasing forbids any user override conflict với master audit log mechanism.

**Suggested fix:** `D không nới tự động bởi Agent; user override yêu cầu audit log + lý do mạnh (Nguyên tắc 3 cascade).`

---

#### F-B+09 [Minor] — File 08:243, 09:311 leak raw "VSI" trong user-facing output spec

**File:** [P_invest_memo_08.md:243](../../analysis_agent/P_invest_memo_08.md#L243), [P_invest_memo_09.md:311](../../analysis_agent/P_invest_memo_09.md#L311)

**What:** `VSI (Volume Strength Index — chỉ số cường độ volume...) ≥ 1.2`. VSI là raw indicator name. Token này xuất hiện trong output spec Phase 2 confirm — surfaced qua tier 7 monitoring report.

**Why it matters:** K-hygiene policy: raw DB tokens dịch trước user-facing output. Đã có inline definition nhưng vẫn dùng VSI làm primary handle.

**Suggested fix:** Output spec dùng translated phrase: `Cường độ volume ≥ 1.2× trung bình (VSI nội bộ)` — keep parenthetical traceability, lead với translation.

---

#### F-B+10 [Minor] — File 09 templates expose raw `market_rank_pct`, `pct_change`, `technical_zone.overall.q` trong alert user-facing

**File:** [P_invest_memo_09.md:95,227,228,311,332,334,339,561](../../analysis_agent/P_invest_memo_09.md#L95)

**What:** Section 4.2 soft trigger output template line 332 — `market_rank_pct rơi xuống < 0.3 trong 2 tuần liên tiếp`. Section 5.1 Phase 2 confirm line 311 — `Zone tuần (technical_zone.overall.w) bật lên A trở lên`. Tier 7 daily/weekly alert templates (Section 10:561) embed tokens trong user-facing alert.

**Why it matters:** Raw DB tokens. Per K hygiene, output spec phải show translated form.

**Suggested fix:** Tier 7 alert dùng translated phrasing: `Xếp hạng dòng tiền thị trường rơi xuống < 0.3...`, `Vùng kỹ thuật quý chuyển A→B...`. Raw token chỉ trong audit log.

---

#### F-B+11 [Minor] — File 04 R2 (trading value < 5 tỷ) duplicate D1 tier 2

**File:** [P_invest_memo_04.md:267-269](../../analysis_agent/P_invest_memo_04.md#L267)

**What:** R2 nói re-check trading value < 5 tỷ tại tier 3 phòng tier 2 D1 dùng 1-phiên spike. Nhưng tier 2 D1 (file 03:164) đã spec "trung bình 20 phiên" — average. R2 hoặc là stale re-check, hoặc imply tier 2 dùng point-in-time.

**Suggested fix:** Remove R2 (trust tier 2 average), hoặc rename `R2. Verify tier 2 dùng 20-phiên average — sanity check`.

---

#### F-B+12 [Minor] — File 02 ranking filter implicit cascade vào tier 3 không enforce ở file 03

**File:** [P_invest_memo_02.md:296-299](../../analysis_agent/P_invest_memo_02.md#L296)

**What:** Section 6:298 — `Tier 2 cân nhắc giảm 1 bucket mỗi mã` cho ngành flag y_trend > 0.8. Nhưng file 03 không có rule "tier 2 đọc tier 1 flag → auto-downgrade bucket". Implicit cascade.

**Why it matters:** Master Nguyên tắc 6 says no auto-cascade giữa tier. Flag không trigger downstream → informational noise.

**Suggested fix:** Add explicit handler file 03 Section 7 `if tier 1 ngành flag y_trend > 0.8 received, default downgrade Bucket 1 → Bucket 2`, hoặc remove implicit cascade ở file 02.

---

#### F-B+13 [Minor] — File 05:196-197 "Đỏ nghiêm trọng — loại thẳng" wording inconsistent với Section 5 override allowance

**File:** [P_invest_memo_05.md:196-197](../../analysis_agent/P_invest_memo_05.md#L196), [P_invest_memo_05.md:376-385](../../analysis_agent/P_invest_memo_05.md#L376)

**What:** Section 3:196-197 phrase `Đỏ nghiêm trọng — loại thẳng` reads strict-reject. Section 5:376-385 says `User có thể override với audit log (hiếm, cần lý do mạnh)` — override IS allowed.

**Suggested fix:** Section 3:196-197 thành `Đỏ nghiêm trọng — đề xuất loại, user override yêu cầu audit log mạnh`.

---

#### F-B+14 [Minor] — File 08:336-341 catalyst-play 15% cap không reference master "convention nội bộ"

**File:** [P_invest_memo_08.md:336-341](../../analysis_agent/P_invest_memo_08.md#L336)

**What:** Master:224 lưu ý: `convention nội bộ pack ngoài 6 nguyên tắc bất biến`. File 08:340 ghi `Tổng ≤ 15% portfolio trong mọi regime, Max 2-3 mã catalyst play` — match exactly. Nhưng file 08 không reference caveat. Reader file 08 standalone treats như bất biến.

**Suggested fix:** Add line file 08:336: `(Convention từ master mục 5 lưu ý — không phải nguyên tắc bất biến, user có thể override với audit log).`

---

#### F-B+15 [Confirm Intent] — File 06 finance jargon (FCFF, WACC, RIM) trong user-facing output

**File:** [P_invest_memo_06.md:62-100,487-628](../../analysis_agent/P_invest_memo_06.md#L62)

**What:** Standard finance jargon glossed inline lần đầu. Nhưng checkpoint 5B template propagate unfiltered. Audience tier 5B = analyst nội bộ → likely intentional.

**Status:** Confirm Intent — confirm tier 5B audience analyst, no change needed.

---

#### F-B+16 [Minor] — File 09 "Phase 2 timeout 4 tuần" introduce ở file con không trong master

**File:** [P_invest_memo_09.md:337-381](../../analysis_agent/P_invest_memo_09.md#L337) vs [P_invest_memo_08.md:247-258](../../analysis_agent/P_invest_memo_08.md#L247)

**What:** File 08:247 `Timeout: sau 4 tuần ... Chi tiết rule trong tier 7`. File 09 implement. Cross-ref correct. Nhưng master `_00` không mention 4-tuần timeout — portfolio rule introduce file con only.

**Why it matters:** Reader master không biết về 4-tuần rule.

**Suggested fix:** Add master mục 5 lưu ý: `Bucket 2 timeout 4 tuần (chi tiết tier 6, tier 7) — convention không phải bất biến`.

---

#### F-B+17 [Minor] — File 04 size target overlap với file 08 — 2 source of truth

**File:** [P_invest_memo_04.md:236-239](../../analysis_agent/P_invest_memo_04.md#L236) vs [P_invest_memo_08.md:72-82](../../analysis_agent/P_invest_memo_08.md#L72)

**What:** File 04 Section 5 table: `High 15-18 → 6-8% portfolio | Medium 11-14 → 3-5% | Low 8-10 → 1-2%`. File 08 Section 3.1 identical + finer-grained `15-16đ: 6%, 17đ: 7%, 18đ: 8%`. Numbers consistent. File 04 says "guideline", file 08 treat as formula base. Ambiguity về authoritative source.

**Suggested fix:** Add note file 04 Section 5: `Size target chi tiết tính ở tier 6 (file 08 Section 3) — đây là guideline base, regime/bucket/ADV adjustment ở tier 6`.

---

#### F-B+18 [Nit] — Master từ điển "Catalyst score 0-6" vs file 01 Section 4 chấm 1-3/catalyst

**File:** [P_invest_memo_00.md:52](../../analysis_agent/P_invest_memo_00.md#L52) vs [P_invest_memo_01.md:347-353](../../analysis_agent/P_invest_memo_01.md#L347)

**What:** Master từ điển: `Catalyst score: thang 0-6 ở tier 0`. File 01 Section 4 chấm 1-3/catalyst. File 02:173 says `max 6 điểm/ngành (2 catalyst × 3 điểm)`. Master 0-6 ám chỉ ngành aggregate (2 catalyst × max 3đ). Wording master ambiguous.

**Suggested fix:** Update master:52 thành `Catalyst score: thang 1-3/catalyst, tổng cộng max 6 điểm/ngành (2 catalyst × 3 điểm) ở tier 0/1`.

---

#### F-B+19 [Nit] — Step count inconsistent across tier workflow headings

**File:** Headers Section "Workflow đầy đủ — N bước" trong file 03,04,05,06,08

**What:** File 03 = "6 bước", file 04 = "7 bước", file 05 = "8 bước", file 06 = "8 bước", file 08 = "8 bước". Inconsistent step counts, không có master template.

**Suggested fix:** Cosmetic — không cần fix unless standardize desired.

---

#### F-B+20 [Confirm Intent] — File 07 Gate 3 "REWRITE đến khi đạt chuẩn" là QC không phải flex

**File:** [P_invest_memo_07.md:385-388](../../analysis_agent/P_invest_memo_07.md#L385)

**What:** Gate 3 `Trigger vague → REWRITE đến khi đạt chuẩn measurable`. File explicit note line 385 `đây là quality control technical, không phải judgment`.

**Status:** Intentional — author đã distinguish QC vs judgment. No change.

---

#### F-B+21 [Confirm Intent] — File 06 quote specific ticker WACC (HPG, VND, VIC GuruFocus)

**File:** [P_invest_memo_06.md:293-296](../../analysis_agent/P_invest_memo_06.md#L293)

**What:** Section 5 cite valueinvesting.io HPG WACC 7.7%, VND, VIC GuruFocus 5.88% với critique. Methodology files thường avoid concrete ticker examples.

**Status:** Confirm Intent — likely intentional benchmark-anchoring với critique.

### 7.5. Recommendations Phase B+

**Wave 5 — Critical/Major Phase B+ (1-2 commit):**
- **F-B+01** — Reconcile master vs file 03 liquidity threshold (decide 5 or 10 tỷ, fix 1 file).
- **F-B+02** — Update master mục 6.1 acknowledge tier-specific data sections (single edit).
- **F-B+03** — Replace stale "tier 4-5", "tier 5-6" references (8 file con edits).
- **F-B+04** — Update 4 ADV examples to include N=2-4 multiplier hoặc reference file 08.
- **F-B+05** — Convert R3/R5 file 04 sang flex+downgrade hoặc elevate thành nguyên tắc bất biến.
- **F-B+06** — Add cascade reference D constraint file 03.

**Wave 6 — Minor/Nit Phase B+ (1 commit):**
- F-B+07 đến F-B+19 — toàn bộ S effort, gộp 1 commit "doc: methodology consistency cleanup tier 1-7".

**Skip (Confirm Intent):** F-B+15, F-B+20, F-B+21 — design decision có chủ đích.

---

**End of report.**
