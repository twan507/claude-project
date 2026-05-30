# Project Status

Trạng thái dự án Claude Code Vietnam Stock Analysis. Cập nhật: 2026-05-30.

Project gồm **3 agent hoàn toàn độc lập** — mỗi agent là 1 Claude project riêng trên Claude Desktop/App với system prompt + project knowledge files riêng. **Không có cross-agent dependency hay runtime sharing** — agent này không đọc file/state của agent kia. Việc nhiều agent có nội dung tương tự (vd schema DB) chỉ là **đồng bộ với cùng nguồn DB**, không phải shared knowledge.

**3 agent = 3 Claude project độc lập:**
- Mỗi agent có system prompt riêng (paste vào Custom Instructions)
- Mỗi agent có project knowledge riêng (upload file `.md` riêng)
- Khi update DB schema → phải update cả 2 agent (analysis + db) riêng biệt, không có cơ chế sync tự động

## 1. Tổng quan 3 agent

| Agent | Directory | Vai trò | Status |
|---|---|---|---|
| **analysis_agent** | [analysis_agent/](../analysis_agent/) | Multi-pack analyst (K/P/O layered architecture) — broadcast tuần, chiến lược tháng, memo deep-dive | ✅ Active |
| **db_agent** | [db_agent/](../db_agent/) | Monolithic stock analyst — tra cứu DB + phân tích ad-hoc | ✅ Active |
| **template_agent** | [template_agent/](../template_agent/) | Document-to-pptx normalizer + brander (VBSE / Finext) | ✅ Active |

Chi tiết pack catalog từng agent: xem [PACK_CATALOG.md](./PACK_CATALOG.md).

## 2. Trạng thái pack theo agent (high-level)

### analysis_agent — Layered architecture K/P/O

| Pack | Files | Status | Last major change |
|---|---|---|---|
| `K_agent_db` | 6 (master + 5) | ✅ Active | 2026-05-30 — schema 25 collection, History block thêm, stock_highlight bỏ, rank ngành tự tổng hợp |
| `P_invest_memo` | 10 (master + 9) | ✅ Active | Pre-2026-05 — stable |
| `P_weekly_overview` | 5 (master + 4) | ✅ Active | 2026-05-30 — refactor từ `P_weekly_market` (fundamental-driven + whitelist 18 + conviction/horizon/disconfirming) |
| `P_vbse_strategy` | 10 (master + 9) | ✅ Active | 2026-05-30 — refactor từ `P_invest_strategy` (fundamental supremacy + 2-phase watchlist với Bucket entry) |
| `O_invest_memo` | 7 (master + 6) | ✅ Active | Pre-2026-05 — stable |
| `O_weekly_overview` | 1 (master) | ✅ Active | 2026-05-30 — new render spec 12 phần rigid + 3 mode branding |
| `O_vbse_strategy` | 1 (master) | ✅ Active | 2026-05-30 — new render spec 2 mode flex |

### db_agent — Monolithic knowledge base

| File | Status | Last major change |
|---|---|---|
| `system_prompt.md` | ✅ Active | 2026-05-30 — manifest 25 collection |
| `agent_db_00..05` | ✅ Active | 2026-05-30 — same schema sync với K_agent_db |

### template_agent

| File | Status | Note |
|---|---|---|
| `system_prompt.md` + 5 knowledge files | ✅ Active | Không thay đổi trong session này |

## 3. Refactor đáng chú ý đã thực thi

### 2026-05-29/30 — Reform chuẩn institutional buy-side

**Pack đã xoá:**
- `P_invest_strategy_00.md` + `O_invest_strategy_00.md` → refactor thành `P_vbse_strategy_*` + `O_vbse_strategy_00`
- `P_stock_pitch_00.md` + `O_stock_pitch_00.md` → xoá hoàn toàn (vai trò chuyển sang `P_vbse_strategy` watchlist + `P_invest_memo` memo)
- `P_weekly_market_00.md` + `O_weekly_market_00.md` → refactor thành `P_weekly_overview_*` + `O_weekly_overview_00`

**Pack mới tạo:**
- `P_vbse_strategy` 10 files với philosophy **fundamental supremacy** + **2-phase watchlist** (Phase 1 Screen cơ bản, Phase 2 Bucket entry PTKT theo `P_invest_memo_03` mục 5)
- `P_weekly_overview` 5 files với philosophy **fundamental-driven** + 12 phần rigid + Key calls/Watch/Risk executive summary

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

`analysis_agent/K_agent_db_*` và `db_agent/agent_db_*` có cùng schema content **vì cùng query 1 DB** — không phải vì có cơ chế shared knowledge. Mỗi agent đọc file knowledge của riêng mình, không có cross-reference giữa 2 agent.

Quy tắc giữ riêng:
- **Naming prefix:** `K_agent_db_*` (analysis) vs `agent_db_*` (db) — KHÔNG được mix
- **Framing:** "Pack" (analysis) vs "Bộ tài liệu này" (db)
- **Cross-refs nội bộ:** luôn dùng đúng prefix của agent đó
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

## 6. File chính cho phát triển tương lai

| Cần thay đổi gì | File nguồn |
|---|---|
| Schema DB | `K_agent_db_01` + `agent_db_01` (sync 2 file) |
| Master / philosophy | File `_00` của pack tương ứng |
| Workflow chi tiết | File `_NN` tier tương ứng của pack |
| Render output | File `O_*_00` tương ứng |
| Kernel router | `analysis_agent/KERNEL_SKELETON.md` |
| Meta rules | `analysis_agent/system_prompt.md` + `db_agent/system_prompt.md` |
| Project knowledge listing | `README.md` mục 3 |
