# Changelog

Lịch sử thay đổi major cross-pack. Mỗi entry ghi: date + summary + files affected + rationale.

---

## 2026-05-30 — Reform institutional buy-side + schema update

### Schema update (cả 2 agent)

**Files:** `K_agent_db_00/01/02/04` (analysis_agent) + `agent_db_00/01/02/04` (db_agent) + downstream pack files

**Thay đổi:**
- 23 → **25 collection**
- Thêm khối **History** (3 collection): `history_index`, `history_industry`, `history_stock` — full historical OHLCV+pct_change cho on-demand chart dài hạn
- **Bỏ `stock_highlight`** (top tăng/giảm pre-compute) — thay bằng aggregate from `stock_snapshot` filter `industry`
- **Bỏ `industry_rank` static** trong `industry_snapshot.money_flow_score` và `industry_recent.series.money_flow_score` — rank ngành phải **tự tổng hợp** theo `week_score` (dòng tiền tuần) trong scope phân tích (default: 18 whitelist; override: tuỳ user)

**Rationale:** rank ngành phụ thuộc scope analysis. Nếu DB lưu rank tĩnh 1..24, mọi báo cáo phải re-compute lại để khớp scope (18 whitelist vs override custom) — rủi ro sai. Tự tổng hợp đảm bảo rank luôn align với danh sách theo dõi.

### Whitelist 18 ngành — Default + Override mode

**Files:** `K_agent_db_00 mục 4.5` + `K_agent_db_01 Section B callout` + `agent_db_00 mục 4.5` + `agent_db_01 Section B callout`

**Thay đổi:**
- **Default mode:** mọi query/aggregate/ranking/báo cáo cấp ngành filter 18 whitelist
- **Override mode:** user yêu cầu cụ thể ngành ngoài whitelist (vd "phân tích ngành Bảo hiểm") → agent vẫn query và trả lời bình thường, kèm note "ngoài scope whitelist mặc định"

**18 ngành whitelist:**
BANLE, BDS, CHUNGKHOAN, CONGNGHE, CONGNGHIEP, DAUKHI, DETMAY, HOACHAT, KCN, KHOANGSAN, KIMLOAI, NGANHANG, NONGNGHIEP, THUCPHAM, THUYSAN, TIENICH, VANTAI, XAYDUNG.

Ngành ngoài whitelist (vẫn có trong DB 24 ngành): BAOHIEM, VLXD, NHUA, CAOSU, DULICH, YTEGD.

### Pack rename + refactor (analysis_agent)

#### `P_invest_strategy` → `P_vbse_strategy` (split 1 file 832 LOC → 10 files ~4760 LOC)

**Old:** `P_invest_strategy_00.md` + `O_invest_strategy_00.md` (xoá hoàn toàn)

**New:** `P_vbse_strategy_00..09` + `O_vbse_strategy_00`

**Philosophy mới:**
- **Fundamental supremacy:** PTKT không có vai trò quyết định trong bất kỳ trục nào (Trục 1-5 + Phase 1 Screen Trục 6)
- **PTKT vai trò duy nhất:** Phase 2 Bucket entry Trục 6 — phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản (theo `P_invest_memo_03` mục 5)
- Cap technical siết: Trục 5 = 0% (cấm tuyệt đối), Trục 4 ≤5%, Trục 3 ≤5%, Trục 2 ≤20%, báo cáo tổng ≤15% (trừ Phase 2 Bucket)
- Weekly **Technical-as-noise rule:** Shift bắt buộc kèm signal vĩ mô/cơ bản/chính sách. Technical shift đơn độc = noise tạm thời

#### `P_weekly_market` → `P_weekly_overview` (split 1 file 616 LOC → 5 files ~1860 LOC)

**Old:** `P_weekly_market_00.md` + `O_weekly_market_00.md` (xoá hoàn toàn)

**New:** `P_weekly_overview_00..04` + `O_weekly_overview_00`

**Philosophy mới:**
- **Fundamental-driven:** 3 kịch bản phần 9 trigger primary là vĩ mô/cơ bản/chính sách/catalyst, technical chỉ confirmation phụ ≤30%
- **Conviction + Horizon + Disconfirming** bắt buộc mỗi call (chuẩn institutional)
- **Executive summary Key calls / Watch / Risk** (thay format bullet rời cũ)
- **Review W-1 thành 3 scorecard tables**
- **Vĩ mô-ngành table 5 cột** (Magnitude + Persistence — chuẩn institutional macro impact)
- Bảng 18 ngành thêm cột P/E phân vị 3Y + EPS Q YoY + earnings beat candidate section
- Cảnh báo trap setup (mã top dòng tiền thuộc ngành thận trọng)
- Watchlist tách 2 hướng (cơ hội tăng + cảnh báo áp lực) + Bucket entry OPTIONAL

#### `P_stock_pitch` + `O_stock_pitch` — XOÁ

**Rationale:** vai trò pitch mã đơn lẻ gửi KH đã được cover bởi:
- `P_vbse_strategy` Trục 6 watchlist (theme play + bucket entry observation)
- `P_invest_memo` Tier 5C memo (deep-dive nội bộ)

Pack dedicated cho pitch KH có thể tái sinh sau nếu có nhu cầu rõ.

### Cross-cutting updates

**Files affected:**
- `analysis_agent/KERNEL_SKELETON.md` — refactor toàn bộ section P + O (P_invest_strategy → P_vbse_strategy, P_weekly_market → P_weekly_overview, xoá P_stock_pitch + O_stock_pitch + O_invest_strategy + O_weekly_market blocks)
- `analysis_agent/system_prompt.md` — update O pack examples (line 60)
- `db_agent/system_prompt.md` — schema manifest 25 collection
- `README.md` — pack list + workflow + naming + constraint cuối
- `docs/PROJECT_STATUS.md` (new)
- `docs/PACK_CATALOG.md` (new)
- `docs/CHANGELOG.md` (this file)

---

## Pre-2026-05 — Initial state (baseline)

**Packs active:**
- `K_agent_db` (analysis_agent) + `agent_db` (db_agent) — 23 collection schema
- `P_invest_memo` (10 files) — workflow đầu tư cá nhân
- `P_weekly_market` (1 file) — báo cáo thị trường tuần 12 phần
- `P_stock_pitch` (1 file) — pitch mã đơn lẻ gửi KH
- `P_invest_strategy` (1 file 832 LOC) — chiến lược đầu tư VN 2 mode

**Architecture stable:** K/P/O layered cho analysis_agent, monolithic cho db_agent.

**Outstanding issues identified before reform:**
- `P_invest_strategy` quá dài 832 LOC trong 1 file, methodology lẫn workflow lẫn output contract
- `P_weekly_market` technical-driven trigger 3 kịch bản phần 9 — không nhất quán với institutional buy-side standard
- Không có whitelist ngành rule — agent có thể aggregate trên ngành ngoài focus
- `industry_rank` static gây confusion khi scope thay đổi (18 vs 24)
