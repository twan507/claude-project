# P_vbse_strategy_07 — Workflow Monthly

File này chi tiết hoá chu kỳ workflow monthly của pack `P_vbse_strategy`. Dependency: master `P_vbse_strategy_00` định nghĩa triết lý (nguyên tắc cơ bản tối thượng) + weight balance + 6 trục; file này orchestrate cách 6 trục được build thành báo cáo tháng. Cross-references trục: Trục 1 (`P_vbse_strategy_01`), Trục 2 (`P_vbse_strategy_02`), Trục 3 (`P_vbse_strategy_03`), Trục 4 (`P_vbse_strategy_04`), Trục 5 (`P_vbse_strategy_05`), Trục 6 (`P_vbse_strategy_06`). Render qua `O_vbse_strategy_00`.

## 1. Overview workflow

Workflow flex 4 stage, ngăn cách bằng 2 checkpoint (sau Stage 0 evaluation + sau Stage 1 regime/themes).

```
─── Pre-flight ──────────────────────────────────────
  Hỏi user 4-5 câu (file tháng N-1, eval prior, focus tháng N, user view, branding, horizon)

─── Stage 0: Đánh giá chiến lược cũ (OPTIONAL) ──────
  Nếu user upload file tháng N-1 + chọn (a) ở pre-flight eval:
    Cross-check thesis N-1 vs actual data tháng N-1 → eval block
    Hit rate themes / sectors / watchlist + Best/Worst call
    Pitfall calibration: methodology nào đã miscalibrated
  Output: Stage 0 eval block presented to user (intermediate)

─── CHECKPOINT 0: Eval review ───────────────────────
  User accept eval / override / skip
  Learning từ eval → feed vào Stage 1

─── Stage 1: Build thesis (Trục 1-3) ────────────────
  Trục 1  Môi trường vĩ mô & tài chính
  Trục 2  Định vị thị trường VN
  Trục 3  Themes & narratives chính (2-5 themes)

─── CHECKPOINT 1: Regime vĩ mô + Top themes ─────────
  Agent xuất block call sơ bộ (regime vĩ mô + 2-5 themes)
  User confirm / override / yêu cầu đào thêm

─── Stage 2: Allocation & risk (Trục 4-5) ───────────
  Trục 4  Sector allocation
  Trục 5  Kịch bản & risk map

─── Stage 3: Watchlist + Tóm tắt (Trục 6 + Executive summary) ─
  Trục 6  High-conviction watchlist
  Executive summary (viết cuối — bao gồm note carry-forward từ eval Stage 0)

─── Render & deliver ────────────────────────────────
  Compile MD theo O_vbse_strategy_00 (mode monthly)
  Save vbse_strategy_monthly_<YYYYMM>.md
  Xuất trong message
```

**Lưu ý:** DB `agent_db` KHÔNG có collection storage cho báo cáo chiến lược cũ (theo schema `K_agent_db_01`). Vì vậy "đọc lại strategy cũ" phụ thuộc 100% vào **file user upload** trong session. Agent có thể đọc file MD user đã save trước đó (vd `vbse_strategy_monthly_202604.md`). DB chỉ dùng để query **actual data** cross-check với thesis cũ (giá, dòng tiền, BCTC, vĩ mô).

## 2. Pre-flight monthly

```
Trước khi build báo cáo chiến lược tháng [N/YYYY], xác nhận:

1. File báo cáo chiến lược tháng N-1 (parent cũ):
   (a) Có, tôi gửi đính kèm
   (b) Không có / lần đầu chạy / không muốn dùng

2. Stage 0 — Đánh giá chiến lược tháng N-1 trước khi plan tháng [N]:
   (a) Có, chạy Stage 0 (recommended nếu đã có file N-1) — agent cross-check thesis cũ với actual data tháng N-1, present eval block, user review trước khi vào Stage 1
   (b) Skip Stage 0 — chỉ embed Review hit rate ngắn trong báo cáo cuối, không dừng để user review
   (c) Không có file N-1 → tự động skip

3. Focus đặc biệt cho tháng N:
   (a) Không, build outlook tổng quát theo phát hiện thực tế
   (b) Có — [user nêu: theme/sector/sự kiện vĩ mô cụ thể cần đào sâu]

4. Quan điểm / giả thuyết anh/chị muốn inject vào báo cáo từ đầu:
   (a) Không, build từ data thuần
   (b) Có — [user nêu tự do: vd "tôi nghĩ ngành thép sắp vào uptrend vì lý do X", "có tin nội bộ về theme Y", "đừng quên rủi ro Z mà tôi đang nhìn"]

   *Quan điểm user sẽ được agent ghép nối theo phương pháp ở `P_vbse_strategy_09` (User overlay). Báo cáo cuối sẽ ghi rõ phần nào do agent phát hiện, phần nào do user inject, kèm cross-check của agent.*

5. Branding & disclaimer (báo cáo có thể dùng cả nội bộ và gửi khách hàng):
   (a) Có — vui lòng cung cấp: tên công ty, logo, hotline, website, phòng ban biên soạn, custom disclaimer (nếu có)
   (b) Không cần — render bản plain nội bộ

6. Time horizon ưu tiên:
   (a) 1 tháng (focus catalyst sắp đến)
   (b) 1-3 tháng (default — outlook trung hạn)
   (c) 3-6 tháng (focus chu kỳ + theme dài hơi)
```

## 3. Stage 0 — Đánh giá chiến lược tháng N-1 (OPTIONAL)

**Skip nếu:** user chọn (b) hoặc (c) ở pre-flight câu 2, hoặc không có file N-1.

**Chạy nếu:** user upload file MD tháng N-1 + chọn (a) ở pre-flight câu 2.

**Mục đích:** đánh giá chính thức thesis cũ trước khi plan thesis mới — không phải embed review ngắn trong báo cáo, mà là **stage độc lập có checkpoint** để user review eval trước khi vào Stage 1.

**Workflow Stage 0:**

**Bước 1 — Extract thesis tháng N-1 từ file:**

Agent parse file MD N-1, extract đầy đủ:
- Regime vĩ mô call + conviction
- 2-5 themes (tên + cơ chế + catalyst + horizon + conviction + disconfirming signals)
- Sector bias (quan tâm / trung tính / thận trọng — kèm conviction từng ngành)
- 3 kịch bản VNINDEX (trigger từng kịch bản)
- Risk map (3-7 rủi ro + signal materialize)
- Watchlist 20 mã (10 Tier 1 + 10 Tier 2, có thể 8/12 hoặc 6/14 nếu Bear regime hoặc quá mua — luận điểm + signal theo dõi + disconfirming signal + ADV tại thời điểm N-1)

**Bước 2 — Query actual data tháng N-1 từ agent_db:**

Cross-check thesis cũ với actual:
- **Regime:** vĩ mô tháng N-1 thực tế (lãi suất / tỷ giá / FII flow / commodities) có khớp regime đã call không
- **Themes:** với mỗi theme:
  - Catalyst trigger đã materialize / delay / fizzle?
  - Ngành liên quan có chạy đúng hướng tháng N-1 không (`industry_snapshot.change.m_pct` + `industry_recent` 20 phiên)
  - Disconfirming signals có signal nào trigger không
- **Sector bias:** với mỗi ngành quan tâm, query `industry_snapshot` + `industry_recent`:
  - % biến động tháng (`change.m_pct`)
  - Xu hướng dòng tiền tháng (`money_flow_score.week_score` aggregate 4 tuần)
  - Industry rank đầu tháng vs cuối tháng — improved / stable / deteriorated
- **Kịch bản VNINDEX:** so VNINDEX thực tế tháng N-1 (`market_recent` 20 phiên) với 3 kịch bản đã đặt — kịch bản nào match
- **Risk map:** rủi ro nào đã materialize (có signal trigger), rủi ro nào còn nguyên, có rủi ro mới chưa có trong map cũ không
- **Watchlist:** với mỗi mã, query `stock_snapshot` + `stock_recent` + `stock_nntd`:
  - Biến động tháng (`change.m_pct`) — chạy đúng hướng luận điểm không
  - Signal theo dõi có trigger không (BCTC release, dòng tiền tuần, vùng kỹ thuật)
  - Disconfirming signal có materialize không
  - ADV tháng N-1 vs ngưỡng filter

**Bước 3 — Compose eval block:**

Eval block 6 phần:

1. **Regime evaluation** — đúng / lệch nhẹ / sai rõ; nếu sai, do trục nào (vĩ mô / định vị / cả 2)
2. **Themes evaluation** — bảng N theme: theme | conviction cũ | trạng thái thực tế (materialize / partial / fizzle / disconfirming triggered) | hit hay miss
3. **Sector tilts evaluation** — bảng: ngành quan tâm | conviction cũ | biến động m_pct thực tế | hit hay miss; tương tự cho ngành cần thận trọng
4. **Watchlist evaluation** — bảng: ticker | conviction cũ | horizon cũ | biến động m_pct thực tế | signal trigger | hit hay miss
5. **Risk map evaluation** — rủi ro nào materialize, signal nào trigger, có rủi ro nào agent đã miss
6. **Calibration learning** — 2-4 dòng:
   - **Best call** (theme/sector/mã đúng nhất + lý do calibration đúng)
   - **Worst call** (theme/sector/mã sai nhất + lý do calibration sai, có gắn với pitfall methodology nào không)
   - **Carry-forward** vào tháng N: themes/sectors/risks nào còn nguyên cần tiếp tục đào sâu

**Bước 4 — Checkpoint 0:**

Agent xuất eval block trong message (0.5-1 trang), hỏi user:

```
─── ĐÁNH GIÁ CHIẾN LƯỢC THÁNG [N-1] — Eval block ───

[Eval block 6 phần ở trên]

Confirm hay refine trước khi tiếp Stage 1 build tháng [N]?
- (a) Accept eval, integrate learning vào Stage 1
- (b) Refine eval — [user nêu phần nào cần điều chỉnh, vd "Worst call về theme Y, agent miss yếu tố Z"]
- (c) Skip carry-forward — chỉ giữ Best/Worst call cho Review section, không feed vào Stage 1
```

**Bước 5 — Xử lý phản hồi:**

| User chọn | Action |
|---|---|
| (a) Accept | Stage 1 chạy với eval learning làm context background; Review section cuối báo cáo full 6 phần eval |
| (b) Refine | Agent revise eval theo user, hỏi lại; user OK → tiếp Stage 1 |
| (c) Skip carry-forward | Stage 1 chạy độc lập, Review section chỉ giữ Best/Worst call ngắn |

**Output Stage 0 trong báo cáo cuối:** mục "Review tháng trước" render đầy đủ 6 phần eval (xem render spec `O_vbse_strategy_00`). Nếu user skip Stage 0 entirely (pre-flight câu 2 chọn b), Review section render ngắn (hit rate + best/worst call) như format cũ.

## 4. Stage 1 — Build thesis (Trục 1-3)

Agent compose lần lượt trục 1 → 2 → 3. Mỗi trục đào theo lăng kính ở file con tương ứng, kèm weight balance rule ở `P_vbse_strategy_00` mục 4. Cross-reference detail:
- Trục 1 detail → `P_vbse_strategy_01`
- Trục 2 detail → `P_vbse_strategy_02`
- Trục 3 detail → `P_vbse_strategy_03`

**Quy tắc đào sâu vs lướt nhanh:**
- Trục có biến động đáng kể (regime vĩ mô đang chuyển pha, định vị thị trường vào vùng cực đoan, theme mới nổi mạnh) → đào sâu, có thể chia sub-section
- Trục ổn định, không có gì mới → 3-5 dòng kết luận đủ, không ép viết dài

**Carry-forward từ Stage 0 (nếu có):** Stage 1 phải tính đến learning từ Stage 0 — không lặp lại pitfall đã identify, đào sâu themes/risks carry-forward, refine calibration conviction cho themes mới dựa trên track record N-1.

## 5. Checkpoint 1 — Regime vĩ mô + Top themes

Sau khi xong Stage 1, agent xuất block ngắn (0.5-1 trang trong message), KHÔNG render full MD:

```
─── REGIME VĨ MÔ + TOP THEMES — Call sơ bộ ───

**Regime vĩ mô:** [ngôn ngữ định tính — vd "đầu chu kỳ nới lỏng còn kéo dài", "cuối chu kỳ thắt chặt, kỳ vọng đảo chiều", "ổn định mid-cycle"]
**Lý do (3 lăng kính chính):** [lãi suất / dòng vốn / tỷ giá — mỗi cái 1 dòng]

**Định vị thị trường VN:** [định tính — vd "tích luỹ sau giảm 6%", "uptrend khoẻ chưa cực đoan", "phân phối sau đỉnh quý"]
**Lý do:** [định giá / dòng tiền / breadth — mỗi cái 1 dòng]

**Top themes đề xuất cho tháng [N]:**
1. **[Tên theme 1]** — [cơ chế + ngành liên quan + catalyst trigger, 2 dòng]
2. **[Tên theme 2]** — ...
3. **[Tên theme 3]** — ...
[2-5 themes, không ép số]

Confirm hay override trước khi tiếp Stage 2 (allocation + risk + watchlist)?
- (a) Confirm như trên
- (b) Override regime / định vị → [user nêu]
- (c) Override / bổ sung themes → [user nêu]
- (d) Cần đào thêm dữ liệu cụ thể trước khi quyết
```

**Xử lý phản hồi user:**

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 2 chạy với thesis đã call |
| (b)(c) Override | Ghi inline note phần trục liên quan trong MD final, Stage 2 chạy với thesis mới |
| (d) Đào thêm | Query bổ sung theo yêu cầu, refine, hỏi lại |

## 6. Stage 2 — Allocation & risk (Trục 4-5)

Compose trục 4 (sector allocation, detail → `P_vbse_strategy_04`) cross-check với 3 trục đã chốt + trục 5 (kịch bản & risk map, detail → `P_vbse_strategy_05`).

**Sector allocation flow:**
1. List 18 ngành whitelist (xem `P_vbse_strategy_00` Nguyên tắc 3 + `K_agent_db_01` Section B) — đọc bảng cross-section (dòng tiền rank, biến động tháng, định giá)
2. Highlight ngành ở giao của (vĩ mô thuận lợi + định vị thuận lợi + có theme) → ứng viên quan tâm
3. Highlight ngành ngược lại (vĩ mô áp lực + định vị xấu + không theme) → ứng viên thận trọng
4. Kiểm tra dẫn dắt thật vs trụ kéo (đếm % mã trong ngành tăng giá tháng) — đa số mã tăng = dẫn dắt thật, vài mã lớn kéo = trụ kéo, gần 50/50 = rotation nội bộ
5. Phân tầng cuối: quan tâm / trung tính / cần thận trọng (số ngành flex theo regime — environment thuận lợi rộng thì 4-6 ngành quan tâm, môi trường khó thì 2-3 ngành defensive)

**Risk framework flow:**
1. 3 kịch bản if-then VNINDEX (cơ sở / tích cực / tiêu cực) — dùng trigger kỹ thuật + flow + sự kiện. Không gán % xác suất.
2. Risk map 3-7 rủi ro, mỗi rủi ro: bản chất → signal materialize cụ thể → phản ứng định tính
3. Cross-link rủi ro với theme (nếu rủi ro X materialize → theme Y bị invalidate)

## 7. Stage 3 — Watchlist + Executive summary

**Watchlist:**

Watchlist build qua 2-tier × 2-phase: Tier 1 Priority (10 mã, cơ bản strong + catalyst rõ + ADV ≥5 tỷ) + Tier 2 Standby (10 mã, cơ bản clean + technical bottom-fishing + ADV ≥3 tỷ). Phân bổ Bear regime mode (8/12) hoặc Trục 2 quá mua (6/14). Chi tiết `P_vbse_strategy_06`. Stage 3 monthly chỉ orchestrate, không lặp lại spec.

Cho mỗi ngành quan tâm + theme đại diện, screen mã theo 2-tier filter:
- **Tier 1:** `stock_snapshot` + `stock_finstats` filter industry, ưu tiên cơ bản strong (EPS YoY ≥15% / doanh thu YoY ≥10% / biên cải thiện / ROE ngành) + catalyst rõ với ngày + thanh khoản ≥ 5 tỷ/phiên + negative catalyst gate clean
- **Tier 2:** `stock_snapshot` + `stock_finstats` filter industry, cơ bản clean (no red flag) + technical bottom-fishing setup (zone q/y tích cực + w/m pullback) + thanh khoản ≥ 3 tỷ/phiên + negative catalyst gate clean

Đa dạng sector cả 2 tier (không > 30% mã/sector). Áp Conviction CAP rules + Variant Perception mandatory Tier 1 + Bear regime mode nếu trigger.

Mỗi mã 2-4 dòng theo spec trục 6 (`P_vbse_strategy_06`).

**Executive summary** (viết cuối, sau khi đã có 6 trục):

3-6 bullet, mỗi bullet 1-2 dòng:
- Regime vĩ mô + định vị thị trường (1 dòng tổng)
- Top 2-3 themes
- Sector bias (quan tâm + thận trọng, ngắn gọn)
- 1-2 risk chính
- Watchlist 1-3 mã tiêu biểu nhất (optional)

## 8. Render monthly

Gọi `O_vbse_strategy_00` render mode monthly. File `vbse_strategy_monthly_<YYYYMM>.md`. Self-audit (`P_vbse_strategy_09`) trước khi xuất.
