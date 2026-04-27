# PROJECT STATUS — 2026-04-27 (cập nhật: P_invest_memo + P_stock_pitch + O_invest_memo + O_weekly_market + O_stock_pitch audit/fix done)

> File handoff context cho cuộc hội thoại tiếp theo. Đọc file này đầu session + `CLAUDE.md` (behavioral guidelines) để có toàn bộ bối cảnh.

## 1. Tổng quan

Project gồm **3 agent độc lập** ở thư mục gốc `D:\twan-projects\claude-project\`:

| Folder | Vai trò | Status |
|---|---|---|
| `super_agent/` | Phân tích cổ phiếu VN, output MD final (báo cáo phân tích) | Active, đang audit |
| `template_agent/` | Nhận MD/document → chuẩn hoá → render binary pptx branded | Active, vừa build |
| `db_agent/` | Database agent (related to MongoDB `agent_db`) | Chưa audit, ngoài scope hiện tại |

**Nguyên tắc kiến trúc tuyệt đối:** mỗi agent **độc lập 100%** — file của agent này KHÔNG được reference agent khác (kể cả bằng tên). Communication giữa 2 agent qua MD file (user manually paste/attach), không có shared state hay call cross-agent.

---

## 2. Folder structure hiện tại

```
D:\twan-projects\claude-project\
├── CLAUDE.md                     # Behavioral guidelines (load mọi session)
├── PROJECT_STATUS.md             # File này
│
├── super_agent/                  # 25 file
│   ├── system_prompt.md          # Meta-rules 3 layer K/P/O
│   ├── KERNEL_SKELETON.md        # Index pack
│   │
│   ├── K_agent_db_00.md          # Knowledge master
│   ├── K_agent_db_01.md          # Schema 23 collections
│   ├── K_agent_db_02.md          # Query patterns
│   ├── K_agent_db_03.md          # Anti-patterns
│   ├── K_agent_db_04.md          # Methodology indicators
│   ├── K_agent_db_05.md          # News methodology
│   │
│   ├── P_invest_memo_00.md       # P pack đầu tư cá nhân — master
│   ├── P_invest_memo_01.md       # Tier 0: Gate vĩ mô
│   ├── P_invest_memo_02.md       # Tier 1: Chọn ngành
│   ├── P_invest_memo_03.md       # Tier 2: Screen mã
│   ├── P_invest_memo_04.md       # Tier 3: Chấm điểm
│   ├── P_invest_memo_05.md       # Tier 5A: PDF forensic
│   ├── P_invest_memo_06.md       # Tier 5B: Modeling
│   ├── P_invest_memo_07.md       # Tier 5C: Memo 7 phần
│   ├── P_invest_memo_08.md       # Tier 6: Portfolio
│   ├── P_invest_memo_09.md       # Tier 7: Monitoring
│   │
│   ├── P_weekly_market_00.md     # P pack báo cáo tuần (1 file)
│   ├── P_stock_pitch_00.md       # P pack khuyến nghị mua mã (1 file)
│   │
│   ├── O_invest_memo_00.md       # O master
│   ├── O_invest_memo_01.md       # Market scan
│   ├── O_invest_memo_02.md       # Stock memo deep-dive
│   ├── O_invest_memo_03.md       # Portfolio plan
│   ├── O_invest_memo_04.md       # Weekly review
│   ├── O_invest_memo_05.md       # Monthly review
│   ├── O_invest_memo_06.md       # Quarterly review
│   │
│   ├── O_weekly_market_00.md     # Weekly market spec (1 file)
│   └── O_stock_pitch_00.md       # Stock pitch spec (1 file)
│
├── template_agent/               # 8 file
│   ├── system_prompt.md          # Meta-rules
│   ├── INDEX.md                  # Manifest + workflow overview
│   ├── FORMAT.md                 # MD contract spec (9 report_types)
│   ├── WORKFLOW.md               # 7 stage + 3 checkpoint flow
│   ├── TEMPLATE_VBSE.md          # Catalog 27 layout VBSE
│   ├── TEMPLATE_VBSE.pptx        # Binary template VBSE
│   ├── TEMPLATE_FINEXT.md        # Catalog 27 layout Finext
│   └── TEMPLATE_FINEXT.pptx      # Binary template Finext
│
└── db_agent/                     # 7 file (chưa audit)
    ├── system_prompt.md
    ├── agent_db_00.md
    ├── agent_db_01.md
    ├── agent_db_02.md
    ├── agent_db_03.md
    ├── agent_db_04.md
    └── agent_db_05.md
```

---

## 3. `super_agent/` — kiến trúc chi tiết

### 3.1. Layer architecture

3 layer + 1 index:

- **K (Knowledge)** — schema, methodology, translation rules, query patterns. "biết gì"
- **P (Process)** — workflow pipeline có thứ tự, checkpoint, audit. "làm theo bước nào"
- **O (Output)** — structure rigid của deliverable, tone, format. "trình bày gì ở đâu"
- **Index:** `KERNEL_SKELETON.md` — đọc đầu session, mỗi session 1 lần

**Output cuối:** MD final. KHÔNG render binary (pptx/docx/xlsx) — đó là concern downstream, ngoài scope.

### 3.2. Pack có sẵn

| Pack | Files | Mục đích |
|---|---|---|
| `K_agent_db` | 6 (00-05) | Knowledge MongoDB `agent_db` chứng khoán VN |
| `P_invest_memo` | 10 (00 master + 01-09) | Quy trình đầu tư cá nhân, horizon 1-6 tháng, long only |
| `P_weekly_market` | 1 (00) | Báo cáo thị trường tuần 12 phần |
| `P_stock_pitch` | 1 (00) | Báo cáo khuyến nghị MUA mã đơn lẻ gửi KH 15 mục |
| `O_invest_memo` | 7 (00 master + 01-06) | Render spec 6 deliverables của P_invest_memo |
| `O_weekly_market` | 1 (00) | Render spec báo cáo tuần |
| `O_stock_pitch` | 1 (00) | Render spec khuyến nghị mã |

### 3.3. P_invest_memo — workflow tóm tắt

5 giai đoạn + giai đoạn 6 song song (numbering files dùng "Tier" với offset):

```
Giai đoạn 1 (Tier 0)  — Gate vĩ mô + catalyst (file 01) → CP1
Giai đoạn 2 (Tier 1)  — Chọn 3-5 ngành (file 02)        → CP2
Giai đoạn 3 (Tier 2)  — Screen 6-10 mã/ngành (file 03)  → CP3
Giai đoạn 4 (Tier 3)  — Chấm điểm top 3/ngành (file 04) → CP4
Giai đoạn 5           — Memo deep-dive:
  Tier 5A (file 05)   — PDF forensic                     → CP5A per-stock
  Tier 5B (file 06)   — Valuation modeling               → CP5B per-stock
  Tier 5C (file 07)   — Memo 7 phần                      → CP cuối per-stock
Giai đoạn 6 (song song):
  Tier 6 (file 08)    — Portfolio construction
  Tier 7 (file 09)    — Monitoring + exit (4 review cycles: daily/weekly/monthly/quarterly)
```

**6 nguyên tắc Agent bất biến** (file 00 mục 5):
1. Không skip variant perception
2. Không vào position nếu chưa viết exit trigger
3. Không size > 5% ADV 20 phiên
4. Bear case steelman trước khi long
5. Dòng tiền dương + catalyst tiêu cực → loại
6. Mỗi giai đoạn kết bằng checkpoint, không tự chuyển tier

### 3.4. P_weekly_market — 12 phần (đã restructure rev mới)

```
Pre-flight: hỏi file W-1 + context + branding info

Stage 1: Compose phần 2-9
  Phần 2  Review tuần trước
  Phần 3  Bối cảnh quốc tế
  Phần 4  Thị trường Việt Nam
  Phần 5  Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành
  Phần 6  Biến động ngành
  Phần 7  Top dẫn dắt — 2 góc nhìn
  Phần 8  Tin tức & catalyst
  Phần 9  Phân tích kỹ thuật VNINDEX + 3 kịch bản + Risk map (9.1-9.4)

CHECKPOINT 1: Regime + Sector bias

Stage 2: Compose phần 10-12 + Phần 1
  Phần 10  Watchlist — Mã đáng chú ý (sector bias intro + 5-8 mã)
  Phần 11  Lịch sự kiện tuần tới (macro + corporate)
  Phần 12  Tuyên bố miễn trừ trách nhiệm (3 trường hợp: custom/default branded/plain)
  Phần 1   Tóm tắt điều hành (viết cuối)
```

**Constraint chính:**
- KHÔNG dùng chỉ báo trend (`market_snapshot.trend`, `industry_snapshot.trend`, `*_recent.recent_trend`) — pháp pháp nội bộ, audience cuối (KH) không hiểu. Trend chỉ dành cho `P_invest_memo` (audience analyst nội bộ).
- Wording observation, không command (mua/bán/giảm tỷ trọng)
- Không gán xác suất % cho kịch bản
- Watchlist không kèm level giá vào/ra/stop

### 3.5. P_stock_pitch — 15 mục

Workflow 5 stage, 2 checkpoint:
- Pre-flight: ticker / memo tier 5C có sẵn (optional) / branding info
- Stage 1: Foundation (mục 3-4: hồ sơ + dữ liệu giao dịch)
- Stage 2: Thesis & Variant Perception (mục 5-10, 4-7 luận điểm flex) → CP1
- Stage 3: Execution (mục 11-12: cấu trúc mua + chốt lãi)
- Stage 4: Steelman Bear Case (mục 13: 5-7 lo ngại + 1-2 còn yếu honest) → CP2
- Stage 5: Closing (mục 1, 2, 14, 15)

**ABORT possible** ở CP1/CP2 nếu thesis yếu hoặc conviction không đủ — không gửi KH sản phẩm half-baked.

**Constraint:** chỉ MUA, không BÁN/GIỮ/short/derivative. Không xác suất %. Không dùng trend.

---

## 4. `template_agent/` — kiến trúc chi tiết

### 4.1. Vai trò

Agent thuần **trình bày**. Nhận input bất kỳ định dạng (PDF/DOCX/MD/text/paste) → chuẩn hoá thành MD theo contract → render binary branded.

KHÔNG phân tích nội dung, KHÔNG sinh insight, KHÔNG query database.

### 4.2. File flat structure (không dùng pack convention)

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules vận hành agent |
| `INDEX.md` | Manifest + workflow tổng quan |
| `FORMAT.md` | Spec MD chuẩn hoá — 9 report_types với section structure |
| `WORKFLOW.md` | Flow 7 stage + 3 checkpoint |
| `TEMPLATE_VBSE.md/.pptx` | Catalog 27 layout brand VBSE (navy + đỏ + tam giác) |
| `TEMPLATE_FINEXT.md/.pptx` | Catalog 27 layout brand Finext (dark + violet + chevron) |

### 4.3. 9 report_type trong FORMAT.md mục 3

| report_type | Section count | Length |
|---|---|---|
| `stock_pitch` | 15-18 (rigid + 3 luận điểm flex) | 12-18 trang |
| `weekly_market` | 12 rigid | 9-11 trang |
| `market_scan` | 7 flex top-down | 8-15 trang |
| `stock_memo` | 3-7 theo conviction tier | 3-15 trang |
| `portfolio_plan` | 8 flex | 6-10 trang |
| `portfolio_review_weekly` | 6 rigid | 0.5-2 trang |
| `portfolio_review_monthly` | 8 rigid | 3-5 trang |
| `portfolio_review_quarterly` | 9 rigid | 5-8 trang |
| `custom` | flex 3-15 (quiz-driven) | tùy user |

### 4.4. Workflow 7 stage

```
Stage 1   Ingest (đọc input, extract content thô)
Stage 1.5 Detect skip-normalize (≥4/6 signals match → skip Stage 2-5)
Stage 2   Parse (LLM analyze report_type + section + ambiguities)
Stage 3   Clarify (multi-choice questions, gom 3-5 câu/turn; nếu custom: quiz 7 câu trong 2 turn)
CP1       Clarification confirm
Stage 4   Normalize (LLM produce MD theo FORMAT contract)
CP2       MD draft review (user confirm/edit/fix)
Stage 5   Finalize MD
Stage 6   Brand pre-flight (VBSE / Finext / chỉ MD)
CP3       Brand confirm
Stage 7   Render binary
```

**Brand whitelist strict:** chỉ VBSE và Finext. Brand khác → reject, không fallback render plain branded.

### 4.5. Custom quiz (FORMAT mục 3.9 + WORKFLOW mục 5.4-5.5)

Khi user pick `custom` (hoặc Stage 2 detect không match preset), agent chạy quiz 7 câu:
- Turn 1 (4 câu): Mục đích / Audience / Length target / Tone
- Turn 2 (3 câu + bonus): Số section / Chart count / Citation style + (optional) section list user paste

Sau quiz, agent build spec runtime + lưu `custom_spec_id` (timestamp) trong frontmatter. Re-render cùng MD `custom` đã có spec → skip-normalize, đi thẳng brand pre-flight.

### 4.6. TEMPLATE pack độc lập tuyệt đối

TEMPLATE pack runtime chỉ đọc 2 nguồn:
1. File pack của chính nó (catalog `.md` + binary `.pptx`)
2. MD final do upstream pipeline produce làm input

KHÔNG đọc FORMAT.md, KHÔNG đọc WORKFLOW.md (đó là context của upstream stage). Mọi nội dung cần thiết đã có trong MD final.

---

## 5. Key architectural decisions đã chốt

### 5.1. Independence giữa 2 agent

**Tuyệt đối:** mỗi agent self-contained, không reference tên agent khác trong bất kỳ file nào. Communication qua MD file (user manually pass).

Lý do: tránh coupling implicit, agent có thể swap/extend độc lập.

### 5.2. K → P → O strict one-way

Pack super_agent: dependency direction K → P → O. Không có call ngược (P không sửa K, O không sửa P). K **re-queryable** nhiều lần xuyên suốt session (thư viện), nhưng chỉ P và O đọc K, không ai sửa K.

### 5.3. T pack đã tách khỏi super_agent

Layer T (Template) ban đầu là layer thứ 4 trong super_agent. Đã tách sang `template_agent/` ở rev 6 vì:
- T natural boundary "không đọc K/P/O" được enforce structurally khi tách agent
- Brand presentation là concern riêng với phân tích/content
- Reusability cao hơn (nhận MD từ source bất kỳ)

### 5.4. Render binary out of scope của super_agent

super_agent xuất MD final là output cuối. Render binary (pptx/docx/xlsx) là concern downstream, ngoài scope. MD final đã đủ structured (heading + chart YAML + citation + locale) để consume bằng tool render bên ngoài.

### 5.5. Naming changes đã apply

- `P_recommendation_memo` → `P_stock_pitch` (ngắn hơn, term "stock pitch" chuẩn finance)
- `O_recommendation_memo` → `O_stock_pitch`
- File output: `recommendation_<TICKER>_<YYYYMMDD>.md` → `stock_pitch_<TICKER>_<YYYYMMDD>.md`
- T_vbse → TEMPLATE_VBSE (đặt lại vì naming convention template_agent dùng UPPERCASE)
- T_finext → TEMPLATE_FINEXT

### 5.6. Trend indicator restriction

Chỉ báo trend (`market_snapshot.trend`, `industry_snapshot.trend`, `group_snapshot.trend`, `*_recent.recent_trend`) là **methodology nội bộ** chỉ dành cho `P_invest_memo` (audience: analyst nội bộ).

`P_weekly_market` và `P_stock_pitch` (audience: KH) **CẤM dùng trend** vì KH không có nền methodology để hiểu chỉ báo này.

### 5.7. Conventions output

- **Locale vi-VN**: dấu `.` ngăn nghìn (`18.200 tỷ`), dấu `,` thập phân (`15,5%`), `+/-X%` rõ ràng
- **Ticker UPPERCASE**, không nháy
- **Date format**: `Q1/2026`, `tháng 4/2026`, `ngày 27/4/2026`
- **K hygiene**: không lộ ký hiệu DB raw (`vsi`, `zone: AAA`, `f382`, `period: "2025_4"`) — phải dịch sang ngôn ngữ tự nhiên theo bảng `K_agent_db_00` mục 5.2
- **Citation**: 4 nhóm — Tổng hợp / finext.vn link / PDF user / Web external

---

## 6. Audit progress

### Đã làm xong (4/8 phần)

| Phần | File | Status |
|---|---|---|
| 1 | `system_prompt.md` | ✓ Audit + Fix |
| 2 | `KERNEL_SKELETON.md` | ✓ Audit + Fix (rev 5, rev 6) |
| 3 | `P_invest_memo` (10 file) | ✓ Audit + Fix (H1, H2, H3, M1, M2, M3, M4, M5 áp 2026-04-27) |
| 4 | `P_weekly_market` (1 file) | ✓ Audit + Fix (restructure sections 10-12) |

### Đã làm xong (8/9 phần — bổ sung 2026-04-27)

| Phần | File | Status |
|---|---|---|
| 5 | `P_stock_pitch` (1 file) | ✓ Audit + Fix (M1-M6 áp 2026-04-27) |
| 6 | `O_invest_memo` (7 file) | ✓ Audit + Fix (M1 CP5C sync, M2 Weekly Review Session sync; M3 legacy specs defer Priority 2) |
| 7 | `O_weekly_market` (1 file) | ✓ Audit + Fix (M1 disclaimer duplicate, M2 raw field K hygiene, M3 K hygiene reference, L4 CP1 block render spec; M4 legacy specs defer Priority 2) |
| 8 | `O_stock_pitch` (1 file) | ✓ Audit + Fix (M1 numbering 13-16 mục flex sync, M3 Section 13 heading align; M2 legacy specs defer Priority 2). M5+M6 đã apply qua P_stock_pitch fix. |

### Chưa làm

| Phần | File | Status |
|---|---|---|
| 9 | `template_agent` (4 file: system_prompt, INDEX, FORMAT, WORKFLOW + 2 TEMPLATE pack) | ⏳ Pending audit (đã build mới, chưa audit) |

### Phần 3 audit findings — ĐÃ FIX (2026-04-27)

Decisions chốt theo logic: flex+downgrade thay vì strict reject (consistent với triết lý "checkpoint = user quyết"); keep Tier numbering hiện tại + thêm note Tier 4 missing là legacy renumbering; clarify ADV per-session × N; P/E benchmark intentional khác (historical ngành / peer mã); catalyst play giữ ở file 08 như tactical convention.

#### High — đã fix

**H1. Variant Perception — flex+downgrade.** Master Nguyên tắc 1 update: nếu không có variant perception rõ → flag cảnh báo + downgrade conviction 1 bậc, user quyết định proceed với size nhỏ hoặc loại mã. Khớp với file 07 Gate 1.

**H2. Bear Case — flex+downgrade.** Master Nguyên tắc 4 update: phản biện yếu → flag + downgrade size 30-50%; bear target dưới giá hiện tại → flag nghiêm trọng + downgrade size 50-70% (coin-flip bet). Khớp với file 07 Gate 2.

**H3. Naming Tier 4 missing.** Giữ nguyên Tier 5A/5B/5C trong file con (tránh phá cross-reference). Master phần 3 thêm ghi chú explicit: numbering Tier không liên tục do giai đoạn 5 (deep-dive) đã được tách thành 3 sub-tier ở refactor lịch sử. "Giai đoạn" = grouping high-level; "Tier" = reference cụ thể.

#### Medium — đã fix

**M1. ADV per-session × N.** Master Nguyên tắc 3 clarify: max size **mỗi phiên giải ngân** ≤ 5% × ADV; build full position phân 2-4 phiên → max tổng vị thế = 5% × ADV × N. Khớp file 08 Section 3.4.

**M2. P/E benchmark intentional khác.** File 02 B3 và file 03 B3 thêm note giải thích: tier 1 (ngành) dùng historical median 3Y của chính ngành (ngành đắt hơn quá khứ chính nó); tier 2 (mã) dùng cross-sectional median ngành hiện tại (mã đắt hơn peer). Hai mục đích khác nhau, intentional.

**M3. Checkpoint 7 → Weekly Review Session.** File 09 mục 10 đổi title + body từ "Checkpoint 7 — Weekly Review" sang "Weekly Review Session" + thêm note explicit tier 7 không có checkpoint cứng (consistent master Phần 6). Section 3 các "Checkpoint weekly/monthly/quarterly" output đổi sang "Weekly/Monthly/Quarterly review session".

**M4. CP cuối → CP5C.** Master từ điển + bảng "Danh sách 7 checkpoint" đổi "CP cuối" sang "CP5C". Bảng tóm tắt giai đoạn cũng update.

**M5. Catalyst play cap.** Master Phần 5 sau Nguyên tắc 6 thêm note explicit: catalyst play ≤ 15% portfolio là tactical convention nội bộ pack ở `P_invest_memo_08` Section 7.4, không phải nguyên tắc bất biến universal.

#### Low

**L1.** File 01 mixed style: ngưỡng cứng (trend/score) vs định tính (NN). Có thể accept nếu ghi rõ lý do.

**L2.** File 09 introduce `lessons_learned_cycle_<X>_<YYYYMM>.md` không có trong master mục 7 "Setup project knowledge".

**L3.** Wording cosmetic: master mục 4 ghi "P_invest_memo_05, 06, 07" với dấu phẩy (cosmetic).

**L4.** File 07 mục 8 vs file 09 mục 8 — end-of-cycle reconcile có overlap nhẹ.

**L5.** Master mục 6 chưa note tier 7 = continuous monitoring với 4 review cycles (daily/weekly/monthly/quarterly), khác tier 0-5C one-time.

### Phần 4 audit findings — đã apply hoặc partially fixed

- High issues về section list mismatch và render binary out of scope: ✓ FIXED
- Medium issues:
  - M3 Pre-flight workflow diagram sync: ✓ FIXED
  - M4 VBSE wording outdated: ✓ FIXED  
  - M5 Branding frontmatter bridge: ⏳ chưa add frontmatter chuẩn vào MD output, cần làm khi audit O packs
  - M6 Regime quota khác P_invest_memo: ⏳ chưa sync, decide later
- Low issues: pending decision

---

## 7. Tasks remaining (priority order)

### Phần 5 audit findings — ĐÃ FIX (2026-04-27)

P_stock_pitch (1 file 609 dòng) audit + fix cùng phiên. **0 high issue** (logic vững, meta-rule consistent với KERNEL + system_prompt + P_weekly_market pattern). 6 medium issues + 6 low issues, fix 6 medium ngay:

- **M1** Numbering "15 mục" cứng → "13-16 mục flex theo số luận điểm" (4 LĐ=13, 5=14, 6=15 default, 7=16). P pack mục 1 + 6.4 update.
- **M2** R/R threshold mục 8.2: auto-reject ("không nên recommend") → flex+flag (downgrade self-conviction, user quyết). Consistent triết lý P_invest_memo. Mục 13.2 sync.
- **M3** Memo tier 5C absorption mapping table: thêm subsection 13.5 với 7-row mapping (memo 7 phần → stock pitch 15 mục) + constraint quan trọng (không dùng probability-weighted target, abort nếu memo Pass/Watch).
- **M4** Variant Perception assessment "Mạnh/Vừa/Yếu" subjective → 4 dấu hiệu objective copy từ P_invest_memo_07 Gate 1 (4 dấu hiệu → 4 action: differentiation+evidence+catalyst → PASS; 3 dấu hiệu yếu → CAUTION+downgrade).
- **M5** O pack thêm subsection 3.12 spec block render cho checkpoint 1/2 (intermediate output, không phải MD final).
- **M6** K hygiene self-check field list cứng → reference `K_agent_db_00 mục 5.2`. Áp cả P pack câu 12 + O pack checklist.

**Low issues (defer):** L1 verify reference K_agent_db_00 mục 4.3 khi audit K pack; L2 disclaimer template thiếu element; L3 spec abort data state; L4 không validate format memo input; L5 wording self-audit literal; L6 O pack mục 8 mention legacy (cleanup Priority 6).

---

### Phần 6 audit findings — ĐÃ FIX (2026-04-27)

`O_invest_memo` (7 file: master + 6 sub) audit + fix cùng phiên. **0 high issue** — pack chỉn chu nhất trong các pack đã audit (master rules clear, cross-reference đầy đủ K/P/O, audience tone unified). 3 medium + 6 low. Fix 2 medium (M1, M2); M3 defer Priority 2 cleanup legacy.

- **M1** `O_invest_memo_02` mục 1 "Thiếu 5C → user phải hoàn thành CP cuối trước" → đổi sang "**CP5C**" (sync với P master sau fix M4 phần 3).
- **M2** `O_invest_memo_04` header reference "tier 7 weekly workflow (mục 3.2 + mục 10 template **Checkpoint 7 Weekly**)" → đổi sang "**Weekly Review Session**" (sync với P fix M3 phần 3).
- **M3 (defer)** Master `O_invest_memo_00` header có note "Render binary out of scope" nhưng body content (mục 1 manifest, mục 4-5 spec 3 format MD/docx/pptx, mục 6 master rules cho docx/pptx) + 6 file con đều có "Guide render docx/pptx" sections. Mâu thuẫn rõ với rev 6 — cần cleanup ở Priority 2.

**Low (defer):** L1 K hygiene field list cứng (đã có "v.v." + reference); L2 triple backtick lồng nhau trong O_03; L3 cycle recap wording overlap với P_09 mục 8; L4 audience tone scope; L5-L6 reference verify OK.

---

### Phần 7 audit findings — ĐÃ FIX (2026-04-27)

`O_weekly_market` (1 file 638 dòng) audit + fix cùng phiên. **0 high issue**. 4 medium + 6 low. Fix 3 medium + 1 low (M1, M2, M3, L4); M4 legacy specs defer Priority 2.

- **M1** Disclaimer duplicate giữa phần 12 (mục 3.13) và Metadata footer (mục 3.14) → bỏ disclaimer block khỏi metadata, giữ chỉ phần 12.
- **M2** Field name raw `day_score`/`week_score` lộ trong template phần 4.2 + 3 trigger phần 9.3 → đổi sang "điểm dòng tiền phiên"/"điểm dòng tiền tuần" (K hygiene compliance).
- **M3** Self-check K hygiene field list cứng → reference `K_agent_db_00 mục 5.2` (consistent pattern với P_stock_pitch + O_stock_pitch fix).
- **L4** Thêm subsection 3.14 "Block render cho checkpoint 1 (intermediate output)" — spec format block CP1 (heading, độ dài, multi-choice, không phải MD final). Consistent với O_stock_pitch fix M5. Metadata renumber 3.14 → 3.15.
- **M4 (defer)** Render binary legacy (mục 5-6 + Output contract mục 8) mâu thuẫn với header "out of scope" — cleanup ở Priority 2.

**Low (defer):** L1 edge case "tuần thị trường yếu" không spec render note; L2 lịch sự kiện thiếu "đáo hạn hợp đồng tương lai"; L3 header disclaimer ngắn vs phần 12 đầy đủ (acceptable purpose khác); L5 ngày format cosmetic; L6 sub-issue M4.

---

### Phần 8 audit findings — ĐÃ FIX (2026-04-27)

`O_stock_pitch` (1 file 612 dòng) audit + fix cùng phiên. Pack đã partially synced via P_stock_pitch fix (M5 block CP1/CP2 render spec + M6 K hygiene reference). **0 high issue**. 3 medium + 8 low. Fix 2 medium (M1, M3); M2 legacy defer Priority 2.

- **M1** Numbering "15 mục cứng" mục 1 + opening mục 2 → clarify "13-16 mục flex theo số luận điểm 4-7" (sync với P pack fix M1).
- **M3** Section 13 heading inconsistency: mục 2 structure ghi `## 13. Bear Case — Phản biện và lo ngại còn yếu` nhưng mục 3.9 actual heading dùng `## 13. Phản biện các lo ngại` → chốt wording mục 2 (đầy đủ hơn, capture honest steelman requirement), update mục 3.9.
- **M2 (defer)** Render binary legacy (mục 5-6 + Output contract mục 8 + workflow Bước 16 + checklist mục 7) — cleanup ở Priority 2. Mục 6 docx chưa marked LEGACY explicit (mục 5 đã marked) — minor inconsistency có thể fix khi cleanup.

**Low (defer):** L1 mapping table 7 luận điểm shift section 11→12 cosmetic; L2 block khuyến nghị 1 target vs Section 1 2 target (purpose khác OK); L3-L5 acceptable; L6-L7 sub-issues legacy; L8 reference verify OK.

---

### Priority 1 — Audit phần 9: `template_agent` (4 file architectural + 2 TEMPLATE pack)

4 file architectural + 2 TEMPLATE pack. Đã build mới, chưa audit chính thức.
- `system_prompt.md` — meta-rules
- `INDEX.md` — manifest
- `FORMAT.md` — 9 report_type spec
- `WORKFLOW.md` — 7 stage flow + custom quiz
- `TEMPLATE_VBSE.md` + `TEMPLATE_FINEXT.md` — catalog layouts

Issues nghi ngờ cần check:
- FORMAT.md mục 3.5 portfolio_plan section list có khớp với O_invest_memo_03 không
- FORMAT.md mục 3.4 stock_memo flex theo conviction tier có khớp với O_invest_memo_02 không
- Custom quiz có các edge case cần handle không

### Priority 2 — Cleanup legacy pptx render specs

Các file đã đánh dấu `[LEGACY]` cần dọn ở audit pass tiếp theo:
- `O_stock_pitch_00.md` mục 5 "Guide render pptx [LEGACY]"
- `O_invest_memo_00.md` notes về pptx/docx render
- `O_weekly_market_00.md` notes tương tự

Vì super_agent đã chốt scope là MD final, các spec render binary trong O packs không còn cần thiết. Có thể xoá hoặc move sang documentation riêng.

### Priority 3 — `db_agent/` (out of current scope)

Folder `db_agent/` có 7 file (`agent_db_00.md` đến `_05` + `system_prompt.md`). Chưa audit, có vẻ là agent riêng cho database operations. Mối quan hệ với `super_agent/K_agent_db_*` chưa rõ — có thể là:
- (a) Agent độc lập query MongoDB cho super_agent consume
- (b) Bản gốc của K_agent_db trong super_agent
- (c) Agent khác hoàn toàn

Nếu user muốn tích hợp/audit `db_agent/` cũng cần làm rõ relationship trước.

---

## 8. Open decisions cần user quyết

User đã clarify các decision sau, lưu lại để AI session sau biết:

1. **Mỗi agent độc lập tuyệt đối** — không reference agent khác trong bất kỳ file nào ✓ đã apply
2. **Brand whitelist strict** — chỉ VBSE/Finext, brand khác reject ✓ đã apply
3. **Skip-normalize fast path** — input đã match contract → đi thẳng brand pre-flight ✓ đã apply
4. **Trend indicator** — chỉ dành cho P_invest_memo (analyst nội bộ); P_weekly_market + P_stock_pitch cấm ✓ đã apply
5. **Custom report_type** — quiz-driven 7 câu, agent không tự bịa structure ✓ đã apply

Decisions chốt 2026-04-27 (P_invest_memo phần 3 fix):

- **Variant Perception + Bear Case** — flex+downgrade (master sync với file 07 Gate 1/2) ✓ apply
- **Naming Tier** — keep Tier 5A/5B/5C trong file con; master thêm note Tier 4 missing là legacy ✓ apply
- **ADV constraint** — per-session × N (master Nguyên tắc 3 clarify) ✓ apply
- **P/E benchmark** — keep different intentional (historical ngành / peer mã); add note ở file 02 + 03 ✓ apply
- **Catalyst play cap 15%** — giữ ở file 08 như tactical convention; master phần 5 add pointer ✓ apply

---

## 9. Conventions quan trọng (tóm tắt nhanh)

**System prompt rules (super_agent):**
- 3 layer K/P/O + 1 index
- Master-first reading: đọc `_00` trước file con
- Checkpoint discipline: P pack không tự chuyển giai đoạn
- K hygiene: dịch ký hiệu DB raw sang ngôn ngữ tự nhiên
- No fabrication: mọi số có nguồn truy được

**Output format:**
- Locale vi-VN strict
- Ticker UPPERCASE
- 4 nhóm citation
- Chart YAML block cho visualizable data
- Báo cáo gửi KH có disclaimer (3 trường hợp render)

**Restrictions:**
- super_agent: render binary out of scope
- P_weekly_market + P_stock_pitch: không dùng trend, không command, không xác suất %, không level giá
- P_stock_pitch: chỉ MUA, không BÁN/GIỮ/short
- template_agent: brand whitelist strict (VBSE/Finext only)

**File naming output:**
- `weekly_market_<YYYYMMDD>.md` (ngày cuối tuần - Chủ Nhật)
- `stock_pitch_<TICKER>_<YYYYMMDD>.md`
- `tier{N}_<YYYYMMDD>_confirmed.md` cho state files super_agent
- Binary từ template_agent: `<report_type>_<id>_<YYYYMMDD>_<brand>.pptx`

---

## 10. Files to read first in next conversation

Đọc theo thứ tự sau khi bắt đầu session mới:

1. `CLAUDE.md` (project root) — behavioral guidelines
2. `PROJECT_STATUS.md` (file này) — bối cảnh tổng thể
3. Tuỳ task tiếp theo:
   - **Audit P_stock_pitch** → đọc `super_agent/P_stock_pitch_00.md` + `super_agent/O_stock_pitch_00.md` + `super_agent/system_prompt.md` + `super_agent/KERNEL_SKELETON.md`
   - **Audit O_invest_memo** → đọc `super_agent/O_invest_memo_00.md` đến `_06.md` + `super_agent/P_invest_memo_00.md` (master cho cross-check)
   - **Audit template_agent** → đọc 4 file architectural + 2 TEMPLATE pack

---

## 11. Audit pattern đã thiết lập

Cho mỗi pack, audit theo format:

```
# Audit Phần [N] — [pack name]

## Vấn đề logic / consistency

### High
[1-N issues, mỗi issue: location, vấn đề, đề xuất sửa]

### Medium  
[1-N issues]

### Low
[1-N issues]

## Tổng kết phần [N]
- Số issue High/Medium/Low
- Đánh giá tổng quan pack (logic vững / drift master vs files / etc.)
```

Audit phần 4 đã sample workflow này. AI session sau cứ tiếp tục pattern.

---

## 12. Trạng thái git

```
Repository: D:\twan-projects\claude-project\
Branch: main
Last known status: nhiều file modified + new files created (template_agent/), chưa commit
```

User có thể commit khi hoàn thành audit toàn bộ.

---

## 13. Ngữ cảnh user

- User người Việt, làm finance/đầu tư cổ phiếu VN
- Email: finext.vn@gmail.com (suggest có liên quan tới brand Finext)
- Workflow: build agent system cho phân tích đầu tư
- Style: prefer concise, không emoji, có thể dùng Vietnamese informal
- Decision-making: thường cho directive ngắn gọn, agent cần execute + report

---

**End of PROJECT_STATUS.md**

> **Cách dùng file này ở session mới:** AI đọc đầu session, hiểu bối cảnh kiến trúc + audit progress + open decisions. Hỏi user task tiếp theo (ưu tiên Priority 1-2-3 ở mục 7), hoặc tiếp audit từ phần 5 (`P_stock_pitch`) nếu user không chỉ định.
