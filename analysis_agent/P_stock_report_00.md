# P_stock_report_00 — Master file

## 1. Mục đích & scope

Pack `P_stock_report` sinh **báo cáo phân tích chuyên sâu 1 cổ phiếu** Việt Nam niêm yết, vào trực tiếp từ ticker không cần đi qua workflow đầu tư cá nhân `P_invest_memo` (Tier 0-3). Horizon 1-12 tháng, output 1-10 trang theo 3 depth mode, audience flex (nội bộ analyst hoặc KH).

**Input kỳ vọng:**
- Ticker cụ thể (vd VNM, HPG, VCB)
- Horizon mong muốn (1-3m / 3-6m / 6-12m)
- Depth mode (Quick / Standard / Deep)
- Audience (nội bộ / KH)
- **BCTC PDF gần nhất 2-3 kỳ** — **BẮT BUỘC**, không có thì REFUSE chạy
- (Optional) BCTN, báo cáo phân tích cũ, báo cáo sell-side, IR presentation
- (Optional) Pair compare với 1-2 mã khác

**Output kỳ vọng:** báo cáo MD structured 6-7 phần với recommendation (Long / Watch / Avoid) + conviction + horizon + disconfirming + audit trail data sources. Render theo `O_stock_report_00`.

**Negative scope:**
- Không phải workflow đầu tư cá nhân — đó là `P_invest_memo` (top-down portfolio cycle)
- Không phải báo cáo tuần thị trường — đó là `P_weekly_overview`
- Không phải báo cáo chiến lược tháng — đó là `P_vbse_strategy`
- Không phục vụ trading ngắn hạn <1 tháng (intraday, scalping)
- Không có short — chỉ long (Long / Watch / Avoid)
- Không size position cụ thể — đó là `P_invest_memo` Tier 6 Portfolio Construction
- Không có khuyến nghị cho audience retail thiếu chứng chỉ (constraint giống `K_agent_db_00` mục 4.4)

## 2. Triết lý

**Bottom-up, single-stock focus.** Khác `P_invest_memo` top-down. Người dùng đã có conviction (hoặc curiosity) về 1 mã cụ thể, pack giúp đào sâu mã đó nhanh và chuẩn institutional.

**Data-first, opinion-second.** Stage 1 Data Acquisition 16 sub-step (1a-1p) bắt buộc trước khi compose thesis. Không để opinion drive data selection (red flag bias kinh điển).

**Depth-flexible nhưng rigor-consistent.** 3 mode (Quick/Standard/Deep) khác về độ dài, KHÔNG khác về độ chuẩn xác. Quick mode vẫn phải có recommendation + conviction + horizon + disconfirming + data source attribution.

**BCTC PDF first principle.** Không có BCTC PDF → REFUSE chạy. Cấu trúc giá trị thực của 1 doanh nghiệp nằm trong thuyết minh BCTC (footnotes), không phải bảng tóm tắt ratios. Đào sâu thuyết minh là điểm phân biệt institutional vs retail analysis.

**Audience-aware K hygiene.** Nội bộ và KH có rule K hygiene khác nhau (xem `O_stock_report_00`). KH không xem thuật ngữ technical raw (Bucket entry, Variant Perception, Kịch bản A/B/C — taxonomy nội bộ).

**Long-only.** Không có Short. Watch = "không đủ conviction Long bây giờ, theo dõi". Avoid = "không Long ở bất kỳ giá nào hiện tại trong horizon".

## 3. Sáu nguyên tắc bất biến

Pack tuân các nguyên tắc cross-cutting đã quy định trong project + 2 nguyên tắc riêng:

### Cross-cutting (từ project)
1. **Conviction + Horizon + Disconfirming** bắt buộc mỗi recommendation (chuẩn institutional)
2. **No command words** — observation/luận điểm, không "mua/bán/giảm tỷ trọng/stop loss". Wording lịch sự, mềm
3. **No probability %** cho kịch bản — dùng if-then trigger objective
4. **K hygiene** — không lộ ký hiệu DB raw + taxonomy nội bộ (xem `K_agent_db_00` mục 5)
5. **Source attribution** — mọi claim định lượng có nguồn truy được (citation 4 nhóm — xem `O_stock_report_00` mục 6 + system prompt mục 5.2)
6. **Web search song song DB** — khi cần tin tức / context vĩ mô, bắt buộc query DB VÀ web search song song (`K_agent_db_00` mục 2)

### Pack-specific
7. **BCTC PDF mandatory** — không có thì REFUSE; thuyết minh BCTC phải được forensic theo 15-point checklist (xem `P_stock_report_01` mục 5.1i)
8. **Variant Perception aware** — Quick optional / Standard recommended / Deep mandatory. Không có VP ở Deep → auto downgrade conviction xuống MID hoặc LOW (theo triết lý flex+downgrade)

## 4. Manifest file con

Pack chia 5 file con (số hiệu reference index, không phải thứ tự thực thi):

**`P_stock_report_00`** — Master (file này): mục đích, scope, philosophy, 6 nguyên tắc, manifest, workflow tổng, depth mode definition, output contract overview, cross-reference.

**`P_stock_report_01`** — Pre-flight + Stage 1 Data Acquisition:
- Pre-flight 6 câu hỏi + file request rule (BCTC mandatory)
- Stage 0 evaluation prior analysis (optional)
- Stage 1 sub-steps 1a-1p (16 sub-step): stock info → FA DB → flow + technical → khối ngoại + tự doanh → ownership → CA → news DB → web search → BCTC forensic → sector → macro → peer compare → ADV → earnings calendar → ESG controversy → value chain data (KH/NCC/channel — SXKD mandatory)
- Type classification logic (SXKD/NH/CK/BH)
- Fail-soft rule khi thiếu data

**`P_stock_report_02`** — Type-specific framework:
- SXKD lens: 4 kịch bản (Value Play / Value Trap / Growth at Premium / Cycle Top) reference `K_agent_db_04` + **mục 2.6 chuỗi giá trị (industry value chain map + firm value chain 5 primary activities + bargaining power 5 forces + vertical integration + position summary)**
- NH lens: NIM/CASA/CAR/NPL/Cost of risk + bank-specific KPIs
- CK lens: brokerage share/margin book/IB pipeline/prop book
- BH lens: combined ratio/APE/persistency/embedded value
- Cross-reference `K_sector_framework` 5.X tương ứng

**`P_stock_report_03`** — Stage 2 Compose + Output structure:
- Stage 2 workflow
- 6-7 phần output structure rigid headings (Phần 2 có sub-section **"Vị trí chuỗi giá trị"** mandatory SXKD Standard+)
- 3 depth mode (Quick / Standard / Deep) — section in/out
- Variant Perception rule per mode
- Pair compare mode logic
- Bear case + Disconfirming signal rules
- Checkpoint 1 + Checkpoint 2

**`P_stock_report_04`** — Self-audit + Edge cases + Output contract:
- Self-audit checklist trước khi finalize
- Edge cases (conglomerate, holding, newly listed, suspended, mã ngoài whitelist, etc.)
- Output contract chi tiết
- Failure modes

## 5. Workflow tổng (overview, chi tiết ở `_01` đến `_04`)

```
PRE-FLIGHT (6 câu)
├── Ticker / Horizon / Depth mode / Audience / Pair compare?
└── File request: BCTC PDF BẮT BUỘC + optional files
        ├── Không có BCTC → REFUSE, đề nghị user upload
        └── Có BCTC → tiếp Stage 0

STAGE 0 (optional) — Eval prior analysis
├── Nếu user upload báo cáo cũ → cross-check thesis với data hiện tại
└── Skip nếu không có

STAGE 1 — Data Acquisition (16 sub-step: 1a-1p)
├── 1a. Stock info + Type classification (SXKD/NH/CK/BH)
├── 1b. FA data DB (BCTC quarterly + annual + valuation)
├── 1c. Dòng tiền + technical zone snapshot
├── 1d. Khối ngoại + tự doanh net position
├── 1e. Major shareholders + ownership structure
├── 1f. Corporate actions recent (rolling 12 tháng)
├── 1g. News từ DB (rolling 30-90 ngày)
├── 1h. Web search news (VN equity + EN macro tuỳ ngành)
├── 1i. BCTC PDF forensic (15-point checklist thuyết minh)
├── 1j. Sector context (K_sector_framework)
├── 1k. Macro context relevant (other_data + EN nếu cần)
├── 1l. Peer comparison (internet-first + thanh khoản filter)
├── 1m. ADV / liquidity tier
├── 1n. Earnings calendar (next reporting date + consensus)
├── 1o. ESG controversy scan (web search)
└── 1p. Value chain data — top KH/NCC/channel/capacity/R&D/Industry 4.0 (SXKD mandatory Standard+; SKIP NH/CK/BH)

STAGE 2 — Compose analysis 6-7 phần
├── 1. Recommendation snapshot
├── 2. Type-specific business analysis
├── 3. Industry & Macro context
├── 4. Financial + Valuation + Peer compare
├── 5. News & Catalysts
├── 6. Bear case + Disconfirming
└── 7. Exit triggers (chỉ Long, skip Watch/Avoid)
+ optional Data appendix (Deep mode)

CHECKPOINT 1 — Thesis core review
├── Present: Reco + Conviction + Thesis 1-2 dòng + Bear chính + Exit chính + Variant Perception (nếu có)
└── User: Confirm / Refine / Reject

STAGE 3 — Finalize
├── Render theo O_stock_report_00 + depth mode + audience
└── Metadata + audit trail data sources

CHECKPOINT 2 — Output review (optional cho Quick mode)
└── User review draft cuối, edit/approve
```

## 6. Ba depth mode (definition)

| Mode | Length | Stage 1 coverage | Phần output | VP rule | Audience preferred |
|---|---|---|---|---|---|
| **Quick** | 1-2 trang | Skip 1e, 1f, 1i (forensic), 1j, 1k, 1l (peer), 1o (ESG), 1p (value chain). 8/16 sub-step active | 1, 2, 4 (vắn), 5 (1-2 catalyst), 6 (1-2 disconfirming), 7 (nếu Long) | Optional | Nội bộ — answer câu hỏi nhanh |
| **Standard** | 3-5 trang | Full 16 sub-step SXKD / 15 sub-step NH/CK/BH (skip 1p). Peer compare 3 mã. **SXKD: Phần 2 mục 2.6 value chain MANDATORY** (Porter + Smile + GVC + Industry 4.0) | 1, 2 (SXKD có sub-section 3 value chain 6 sub-sub), 3 (vắn), 4 (full + peer table), 5 (2-4 catalyst), 6 (2-3 disconfirming), 7 (nếu Long) | Recommended (không có → flag "Consensus-aligned thesis, edge limited") | Nội bộ + KH OK |
| **Deep** | 5-10 trang | Full 16 sub-step SXKD / 15 NH/CK/BH + ESG controversy kỹ + value chain đầy đủ 6 framework. Peer compare 5 mã | 1, 2 (SXKD full value chain), 3 (full), 4 (full + peer + macro sensitivity), 5 (2-4 catalyst with timing), 6 (2-3 disconfirming + bear scenario steelmanned), 7 (đầy đủ), + Data appendix | Bắt buộc (không có → auto downgrade conviction MID/LOW) | Nội bộ — pre-Tier 5C analysis |

Default khi user không specify: **Standard mode** (mid-ground hợp lý).

## 7. Output contract overview

Chi tiết ở `O_stock_report_00`. Tóm tắt:

- **MD final** là output cuối (như mọi pack khác trong project)
- Structure 6-7 phần rigid heading (không flex structure)
- Mỗi phần ghi rõ **data source attribution** (DB / Web / PDF / Sector framework)
- Audit trail metadata cuối báo cáo (audit trail data sources + timestamps)
- Audience flex (nội bộ / KH) → ảnh hưởng K hygiene + wording mềm/cứng
- Render branded optional (VBSE / Finext / plain) — render binary out of scope, MD final là deliverable
- Citation 4 nhóm (xem `O_stock_report_00` mục 6 + system prompt mục 5.2)
- Naming: `stock_report_<TICKER>_<YYYYMMDD>_<mode>.md` (mode = quick/standard/deep)

## 8. Trigger / Activation

Trigger phrase điển hình:
- "Phân tích mã [X]" / "Phân tích cổ phiếu [X]"
- "Đánh giá [X]"
- "[X] có nên mua không"
- "Brief [X] cho KH"
- "Quick check [X]"
- "Stock report [X]"
- "So sánh [X] vs [Y]" (mode pair)
- "[X] horizon [Y] tháng"

Conflict resolution:
- "Phân tích [X]" generic → `P_stock_report`

## 9. Dependencies

- `K_agent_db` (mandatory) — schema, query patterns, methodology, đặc biệt `K_agent_db_04` cho 4 type framework
- `K_sector_framework` (recommended) — pull khi compose phần 2 (type-specific) và phần 3 (industry context)
- `O_stock_report` (mandatory ở Stage 3) — render spec

## 10. Quan hệ với 3 lăng kính phân tích cốt lõi

Pack tuân **3 lăng kính theo thứ tự ưu tiên** của `K_agent_db_00` mục 8:
1. **Dòng tiền** (trước) — từ Stage 1c
2. **Kỹ thuật** (sau) — chỉ snapshot zone đa khung từ Stage 1c, không deep PTKT
3. **Cơ bản** (deep) — Stage 1b + 1i (forensic) + 1l (peer)

Tuy nhiên với pack này, **trọng số bottom-up cao hơn**: cơ bản và forensic BCTC PDF chiếm ~50-60% công sức, dòng tiền + technical zone là context snapshot (~15-20%), industry/macro context (~20-25%), ESG/controversy (~5-10%).

Khác `P_weekly_overview` (technical ≤15%, fundamental-driven primary) — pack này cũng fundamental-driven nhưng deep hơn về company-level. Dòng tiền vẫn quan trọng nhưng là 1 input, không phải primary lens.

## 11. Flex+downgrade triết lý (như mọi pack analysis_agent)

Khi gate methodology không pass strict:
- **Variant Perception yếu hoặc thiếu (ở Deep mode):** auto downgrade conviction từ HIGH → MID, MID → LOW, LOW → "không đủ conviction kết luận, recommendation Watch"
- **Bear case rebuttal yếu:** downgrade size (nếu có) hoặc flag "thesis vulnerable to [X]"
- **Data thiếu (vd BCTC PDF không có thuyết minh, hoặc peer không tìm được):** downgrade depth mode (Deep → Standard, Standard → Quick) hoặc flag "limited data, conviction capped at LOW"

User quyết định cuối: proceed với downgrade + audit log, hoặc reject báo cáo.

**Exception strict reject (không flex+downgrade):**
- BCTC PDF không có → REFUSE chạy (không downgrade — đây là bắt buộc tuyệt đối)
- Ticker không tồn tại trong DB → REFUSE
- Mã suspended / cảnh báo / kiểm soát đặc biệt → REFUSE hoặc fail-soft với big warning
