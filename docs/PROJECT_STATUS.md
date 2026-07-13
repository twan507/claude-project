# Project Status

Trạng thái dự án Claude Code Vietnam Stock Analysis. Cập nhật: 2026-07-13 (rev 6 — port agent_db v2/fnx05 vào cả 2 agent).

Project gồm **2 agent hoàn toàn độc lập** — mỗi agent là 1 Claude project riêng trên Claude Desktop/App với system prompt + project knowledge files riêng. **Không có cross-agent dependency hay runtime sharing** — agent này không đọc file/state của agent kia. Việc 2 agent có nội dung tương tự (vd schema DB) chỉ là **đồng bộ với cùng nguồn DB**, không phải shared knowledge.

**2 agent = 2 Claude project độc lập:**
- Mỗi agent có system prompt riêng (paste vào Custom Instructions)
- Mỗi agent có project knowledge riêng (upload file `.md` riêng)
- Khi update DB schema → phải update cả 2 agent (analysis + db) riêng biệt, không có cơ chế sync tự động

## 1. Tổng quan 2 agent

| Agent | Directory | Vai trò | Status |
|---|---|---|---|
| **analysis_agent** | [analysis_agent/](../analysis_agent/) | Multi-pack analyst (K/P/O layered architecture) — broadcast tuần, chiến lược tháng, memo deep-dive | ✅ Active |
| **db_agent** | [agent_db/](../agent_db/) | Monolithic stock analyst v2 — tra cứu DB + phân tích ad-hoc, audience NĐT khách Finext, tầng phase & danh mục | ✅ Active |

Chi tiết pack catalog từng agent: xem [PACK_CATALOG.md](./PACK_CATALOG.md).

## 2. Trạng thái pack theo agent (high-level)

### analysis_agent — Layered architecture K/P/O

| Pack | Files | Status | Last major change |
|---|---|---|---|
| `K_agent_db` | 7 (master + 6) | ✅ Active | 2026-07-13 — port bộ agent_db v2 (fnx05): 31 collection (+6 phase), đơn vị `*_pct` = điểm % / `rank_pct` 0-100, 13 workflow A-M, `data_briefing` 2 doc, omit-null, thêm `K_agent_db_06` phase & danh mục (tín hiệu tham chiếu, P pack không dùng) |
| `K_sector_framework` | 1 | ✅ Active | 2026-05-30 — new pack distill CFA Sector Analysis Framework (universal DD/MP/SI/PM/ESG + per-sector quick-ref cho 18 ngành whitelist + Industry 4.0 lens) |
| `P_invest_memo` | 10 (master + 9) | ✅ Active | Pre-2026-05 — stable; 2026-05-30 thêm pointer K_sector_framework ở `_07` Phần 3 |
| `P_weekly_overview` | 5 (master + 4) | ✅ Active | 2026-05-30 — refactor từ `P_weekly_market` (fundamental-driven + whitelist 18 + conviction/horizon/disconfirming) |
| `P_vbse_strategy` | 10 (master + 9) | ✅ Active | 2026-05-30 — refactor từ `P_invest_strategy` (fundamental supremacy + 2-phase watchlist với Bucket entry); thêm pointer K_sector_framework ở `_04` Trục 4 |
| `P_stock_report` | 5 (master + 4) | ✅ Active | 2026-05-30 (rev 5) — new pack single-stock deep analysis. **Stage 1 16 sub-step (1a-1p)** data acquisition (BCTC PDF bắt buộc + forensic thuyết minh + news DB + web VN equity + EN macro + peer internet-first + ESG + **1p value chain data**) + 4 type framework (SXKD/NH/CK/BH) + **SXKD có mục 2.6 Chuỗi giá trị 10 sub-mục áp dụng 6 framework chuẩn quốc tế (Porter Value Chain + 5 Forces + Smile Curve Stan Shih + GVC governance Gereffi + Industry 4.0 CFA Sector Analysis 2020 + CFA chapter mapping 21 ngành)** + 3 depth mode (Quick/Standard/Deep) + audience flex (nội bộ/KH) + pair compare 2-3 mã. Self-audit 47 điểm SXKD / 35 điểm NH/CK/BH |
| `O_invest_memo` | 7 (master + 6) | ✅ Active | Pre-2026-05 — stable |
| `O_weekly_overview` | 1 (master) | ✅ Active | 2026-05-30 — new render spec 12 phần rigid + 3 mode branding |
| `O_vbse_strategy` | 1 (master) | ✅ Active | 2026-05-30 — new render spec 2 mode flex |
| `O_stock_report` | 1 (master) | ✅ Active | 2026-05-30 — new render spec 6-7 phần rigid + 3 depth mode + audience flex (K hygiene khác nội bộ vs KH) |

### db_agent (`agent_db/`) — Monolithic knowledge base v2

| File | Status | Last major change |
|---|---|---|
| `system_prompt.md` | ✅ Active | 2026-07-12 (v2) + 2026-07-13 (v2.1) — gộp `agent_db_00` cũ vào system prompt; audience NĐT khách; v2.1 hạ phase từ luật subordination xuống tín hiệu tham chiếu |
| `agent_db_01..06` | ✅ Active | 2026-07-12 — schema fnx05 v2 (31 collection, đơn vị mới, phase); `_06` mới (phase & danh mục); sync với `K_agent_db_01..06` |

### template_agent — XOÁ

Pack `template_agent` (document-to-pptx normalizer + brander) đã được xoá khỏi project 2026-05-30. Lý do: render binary out of scope, MD final từ analysis_agent đã đủ structured để consume bằng tool render bên ngoài.

## 3. Refactor đáng chú ý đã thực thi

### 2026-05-29/30 — Reform chuẩn institutional buy-side

**Pack đã xoá:**
- `P_invest_strategy_00.md` + `O_invest_strategy_00.md` → refactor thành `P_vbse_strategy_*` + `O_vbse_strategy_00`
- `P_stock_pitch_00.md` + `O_stock_pitch_00.md` → xoá hoàn toàn (vai trò chuyển sang `P_vbse_strategy` watchlist + `P_invest_memo` memo)
- `P_weekly_market_00.md` + `O_weekly_market_00.md` → refactor thành `P_weekly_overview_*` + `O_weekly_overview_00`
- **Toàn bộ `template_agent/`** (document-to-pptx normalizer + brander) → out of scope, render binary chuyển sang tool ngoài

**Pack mới tạo:**
- `P_vbse_strategy` 10 files với philosophy **fundamental supremacy** + **2-phase watchlist** (Phase 1 Screen cơ bản, Phase 2 Bucket entry PTKT theo `P_invest_memo_03` mục 5)
- `P_weekly_overview` 5 files với philosophy **fundamental-driven** + 12 phần rigid + Key calls/Watch/Risk executive summary
- `K_sector_framework` 1 file — khung phân tích ngành chuẩn CFA institutional buy-side (universal DD/MP/SI/PM/ESG + per-sector quick-ref cho 18 ngành whitelist + Industry 4.0 lens)
- `P_stock_report` 5 files + `O_stock_report` 1 file — pack **single-stock deep analysis** vào trực tiếp từ ticker. Đặc trưng: BCTC PDF mandatory + forensic 15-point thuyết minh + 4 type framework (SXKD/NH/CK/BH) + 3 depth mode (Quick 1-2 / Standard 3-5 / Deep 5-10 trang) + audience flex (nội bộ/KH) + pair compare 2-3 mã. Complement (không thay thế) `P_invest_memo` Tier 5C — dùng cho pre-screening / pitch nhanh / ad-hoc analysis ngoài full memo cycle

**Schema DB update:**
- 23 → 25 collection
- Thêm khối History (3 collection): `history_index`, `history_industry`, `history_stock`
- Bỏ `stock_highlight` (top tăng/giảm pre-compute) — thay bằng aggregate from `stock_snapshot`
- Bỏ `industry_rank` static field trong `industry_snapshot` + `industry_recent` — rank ngành **tự tổng hợp** theo `week_score` (dòng tiền tuần) trong scope phân tích

**Whitelist 18 ngành (rule cross-pack):**
- **Default mode:** mọi query/aggregate/ranking cấp ngành filter 18 whitelist
- **Override mode:** user yêu cầu cụ thể ngành ngoài whitelist → agent vẫn query và trả lời, kèm note "ngoài scope whitelist mặc định"
- Áp dụng cho cả 2 pack `K_agent_db` (analysis_agent) và `agent_db` (db_agent)

**PTKT role re-definition (analysis_agent):**
- Trước: technical_zone đa khung làm TERTIARY confirmation ở mọi trục
- Sau:
  - `P_vbse_strategy`: PTKT chỉ ở Phase 2 Bucket entry Trục 6 (timing observation). Trục 4 sector, Trục 5 risk, Phase 1 Screen → cap 0% PTKT
  - `P_weekly_overview`: 3 kịch bản phần 9 trigger primary là vĩ mô/cơ bản/chính sách/catalyst, technical chỉ confirmation phụ ≤30%
  - Cap technical toàn báo cáo ≤15% (trừ Phase 2 Bucket / 9.2 Vùng giá tham chiếu)

### Sync DB schema giữa 2 agent (CHỈ là đồng bộ với DB, không phải shared knowledge)

`analysis_agent/K_agent_db_*` và `agent_db/agent_db_*` có cùng schema content **vì cùng query 1 DB** — không phải vì có cơ chế shared knowledge. Mỗi agent đọc file knowledge của riêng mình, không có cross-reference giữa 2 agent.

Quy tắc giữ riêng:
- **Naming prefix:** `K_agent_db_*` (analysis) vs `agent_db_*` (db, thư mục `agent_db/`) — KHÔNG được mix
- **Framing:** "Pack" (analysis) vs "Bộ tài liệu này" (db)
- **Cross-refs nội bộ:** luôn dùng đúng prefix của agent đó; pointer system prompt khác nhau (db v2: mục 5/8.x/9; analysis: mục 5.x + `K_agent_db_00`)
- **Master file:** db agent v2 gộp master vào system_prompt; analysis giữ `K_agent_db_00` riêng (rule master-first mục 5.7)
- **Audience khác nhau:** db = NĐT khách (clarify nới lỏng); analysis = analyst nội bộ (clarify 2 câu) — KHÔNG port policy audience giữa 2 agent
- Khi update schema (vd thêm/bỏ collection), phải update cả 2 nơi riêng biệt

## 4. Cross-cutting rules áp dụng mọi pack analysis_agent

1. **Whitelist 18 ngành default + user override** (Nguyên tắc 3 master vbse_strategy + Nguyên tắc 2 master weekly_overview)
2. **Rank ngành tự tổng hợp** theo `week_score` qua scope phân tích (xem `K_agent_db_01` mục "Xếp hạng ngành")
3. **Conviction + Horizon + Disconfirming signal** bắt buộc mỗi call (chuẩn institutional)
4. **No command words** (mua/bán/giảm tỷ trọng/stop loss) — observation only
5. **No probability %** cho kịch bản — if-then trigger
6. **K hygiene** — không lộ ký hiệu DB raw ra output, dịch sang ngôn ngữ tự nhiên
7. **Source attribution** — mọi claim định lượng có nguồn truy được

## 5. Known TODO / Backlog

- [ ] Audit `P_invest_memo` xem có cần update để align fundamental-first philosophy của pack mới (hiện tại độc lập, không bị conflict)
- [ ] Decide whether `K_agent_db_03` (anti-patterns) cần case study mới cho rank ngành tự tổng hợp (vd anti-pattern: query industry_rank tĩnh vẫn nghĩ DB có)
- [ ] Consider thêm pack `P_quarterly_report` cho báo cáo quý (gap hiện tại: chỉ có monthly cycle, không có quarterly aggregation)
- [ ] Test pack `P_stock_report` end-to-end với 1-2 mã thực tế (1 SXKD value chain heavy như HPG/VNM/TNG để stress test mục 2.6 6 framework, 1 NH) để validate workflow + output quality
- [ ] Bổ sung case study `K_agent_db_03` cho failure modes `P_stock_report` mục 3 (fake Variant Perception, soft-pedal bear, catalyst không timing)
- [ ] Test value chain framework Phần 2 sub-section 3 với firm điển hình mỗi Smile Curve zone (smile bottom: Garmex/EMS; smile mid: TNG/HPG; smile top: VNM/PNJ/FPT)
- [x] ~~Bổ sung Value chain analysis cho P_stock_report SXKD lens~~ — DONE 2026-05-30 rev 4 (basic Porter framework)
- [x] ~~Tham chiếu CFA Sector Analysis 2020 + internet professional standards cho value chain~~ — DONE 2026-05-30 rev 5 (Porter full + Smile Curve + GVC governance + Industry 4.0 + CFA 21 chapter mapping)

## 6. File chính cho phát triển tương lai

| Cần thay đổi gì | File nguồn |
|---|---|
| Schema DB | `K_agent_db_01` + `agent_db_01` (sync 2 file) |
| Master / philosophy | File `_00` của pack tương ứng |
| Workflow chi tiết | File `_NN` tier tương ứng của pack |
| Render output | File `O_*_00` tương ứng |
| Kernel router | `analysis_agent/KERNEL_SKELETON.md` |
| Meta rules | `analysis_agent/system_prompt.md` + `agent_db/system_prompt.md` |
| Project knowledge listing | `README.md` mục 3 |
