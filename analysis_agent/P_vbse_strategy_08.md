# P_vbse_strategy_08 — Workflow Weekly Update

File này chi tiết hoá chu kỳ workflow weekly update của pack `P_vbse_strategy` — chu kỳ con tracking thesis monthly (xem `P_vbse_strategy_07`). Dependency: master `P_vbse_strategy_00` định nghĩa triết lý + weight balance + 6 trục. File này có HARD GATE pre-flight (refuse nếu không có monthly active) + technical-as-noise rule (BẤT BIẾN khi đánh giá status Hold/Shift/Materialize).

**CRITICAL CALLOUT (theo `P_vbse_strategy_00` mục 4.3):** Weekly status Hold/Shift/Materialize phải kèm signal vĩ mô / cơ bản / chính sách. Technical shift đơn độc (VNINDEX MA tụt, industry rank tụt, technical_zone yếu đi) → "noise tạm thời, status Hold". **Ngoại lệ duy nhất:** rebucket entry (Bucket 3 → Bucket 1 khi w/m bật A) — tracking entry timing trong watchlist, KHÔNG phải shift thesis.

## 1. Overview workflow

Workflow 2 stage (Stage 0 eval W-1 optional + Stage 1 tracking), có HARD GATE pre-flight: phải xác định tuần thứ mấy của tháng nào + có monthly active không. Không có monthly active → refuse + redirect sang monthly cycle.

```
─── Pre-flight (HARD GATE) ──────────────────────────
  Bước 1  Agent compute: hôm nay DD/MM/YYYY → tuần thứ [N] của tháng [M/YYYY]
  Bước 2  Hỏi user: có monthly active cho tháng [M/YYYY] chưa?
          - Có → upload file → tiếp Bước 3
          - Không → REFUSE, đề xuất chạy monthly cycle trước (xem mục 2 case "no monthly")
  Bước 3  Hỏi: file W-1 (tuần trước trong cùng tháng) + có muốn chạy Stage 0 eval W-1 không
  Bước 4  Hỏi: context tuần + user view inject

─── Stage 0: Đánh giá tuần W-1 (OPTIONAL) ───────────
  Nếu user upload file W-1 + chọn (a) ở pre-flight eval:
    Cross-check thesis W-1 vs actual data tuần W-1
    Hit rate trục Hold/Shift/Materialize đã đặt
    Watchlist refresh W-1: mã Hold/Watch/Out/In đã chạy đúng chưa
  Output: Stage 0 eval block (intermediate)

─── CHECKPOINT 0 (chỉ khi chạy Stage 0): Eval review ─
  User accept eval / refine / skip carry-forward

─── Stage 1 — Tracking & Update ─────────────────────
  Bước 1  Đọc monthly active, extract đầy đủ 6 trục thesis + signals
  Bước 2  Quét nhanh 6 trục với data tuần qua → status Hold/Shift/Materialize
  Bước 3  Watchlist refresh 4 trạng thái
  Bước 4  1-2 action item tuần tới

─── Render ──────────────────────────────────────────
  Compile MD ngắn theo O_vbse_strategy_00 (mode weekly update)
  Save vbse_strategy_weekly_<YYYYMMDD>.md
```

## 2. Pre-flight — HARD GATE

**Bước 1 — Agent compute tuần của tháng:**

Hôm nay là ngày [DD/MM/YYYY]. Agent compute:
- **Tuần của tháng:** `ceiling(day_of_month / 7)` — vd ngày 14/05 → tuần 2 của tháng 5; ngày 22/05 → tuần 4
- **Tuần ISO** (tham khảo, nếu user hỏi): tuần ISO bắt đầu thứ Hai, week 1 = tuần chứa thứ Năm đầu tiên của năm

Convention dùng trong báo cáo: **tuần của tháng** theo công thức `ceiling(day/7)` cho đơn giản, ghi rõ "tuần [N] của tháng [M/YYYY]" ngay đầu báo cáo.

**Bước 2 — HARD GATE: check monthly active:**

```
Hôm nay [DD/MM/YYYY] — đây là tuần [N] của tháng [M/YYYY].

Để chạy weekly update, cần có báo cáo chiến lược tháng [M/YYYY] làm parent. Bạn có file monthly tháng [M/YYYY] chưa?

1. Trạng thái monthly:
   (a) Có — tôi gửi đính kèm file vbse_strategy_monthly_<YYYYMM>.md
   (b) Chưa có / chưa chạy monthly cycle cho tháng [M/YYYY]
   (c) Tôi có monthly nhưng của tháng khác (vd vẫn dùng tháng M-1 vì tháng M chưa kịp build)
```

**Xử lý theo case:**

| User chọn | Action |
|---|---|
| (a) Có file đúng tháng | Tiếp Bước 3, chạy bình thường |
| (b) Chưa có monthly | **REFUSE weekly mode**. Reply: "Để chạy weekly update tháng [M/YYYY] cần có thesis monthly làm parent. Đề xuất 2 hướng: (i) Chạy monthly cycle cho tháng [M/YYYY] trước, sau đó weekly update — recommended. (ii) Override — vẫn muốn chạy weekly độc lập, không có parent thesis, không recommended vì sẽ thiếu context để track shift/materialize." Hỏi user chọn hướng nào. |
| (c) Có monthly tháng khác | Hỏi: "File monthly của tháng M-1 đã chạy được [X] tuần qua mốc 1 tháng — thesis có thể đã decay. Đề xuất 2 hướng: (i) Chạy monthly cycle cho tháng [M] hiện tại trước. (ii) Override — vẫn dùng monthly cũ làm parent, ghi rõ trong báo cáo 'thesis từ tháng M-1 carry-over, có thể bị decay sau [X] tuần'. Bạn chọn?" |

**Bước 3 — Pre-flight các câu còn lại (chỉ chạy nếu Bước 2 chọn (a) hoặc override):**

```
Tiếp tục pre-flight tuần [N] tháng [M/YYYY]:

2. File báo cáo tuần W-1 (tuần trước trong cùng tháng [M], nếu N > 1):
   (a) Có, tôi gửi đính kèm
   (b) Không có / đây là tuần 1 của tháng [M] (vừa chạy monthly tuần này)

3. Stage 0 — Đánh giá tuần W-1 trước khi tracking tuần [N]:
   (a) Có, chạy Stage 0 (recommended nếu đã có file W-1)
   (b) Skip Stage 0
   (c) Không có file W-1 → tự động skip

4. Context bổ sung tuần qua:
   (a) Không có gì đặc biệt — quét default 6 trục
   (b) Có — [user nêu: sự kiện lớn / signal materialize / theme shift quan trọng]

5. Quan điểm / quan sát anh/chị muốn ghi nhận tuần này:
   (a) Không, agent tự quét
   (b) Có — [user nêu tự do: vd "tôi thấy theme A đã bị priced-in", "BCTC mã X gây surprise", "có rủi ro mới Y chưa nằm trong risk map tháng"]

   *Quan điểm user sẽ được agent xử lý theo `P_vbse_strategy_09` (User overlay).*
```

## 3. Stage 0 — Đánh giá tuần W-1 (OPTIONAL)

**Skip nếu:** user chọn (b) hoặc (c) ở pre-flight câu 3, hoặc đây là tuần 1 tháng (vừa chạy monthly, W-1 không tồn tại).

**Chạy nếu:** user upload file W-1 + chọn (a) ở pre-flight câu 3.

**Workflow Stage 0 (weekly):**

**Bước 1 — Extract thesis W-1:**

Parse file MD tuần W-1, extract:
- Status 6 trục W-1 (Hold / Shift / Materialize)
- Trục Shift cụ thể: shift gì, ngụ ý gì
- Trục Materialize: rủi ro nào trigger, phản ứng đã đề xuất
- Watchlist refresh W-1: 4 nhóm (Hold / Watch closely / Out / Vào mới)
- 1-2 action item W-1 đã đặt

**Bước 2 — Query actual data tuần W-1:**

- Với trục Shift W-1: shift đó có tiếp diễn / đảo chiều / fizzle trong tuần [N] này không
- Với trục Materialize W-1: rủi ro tiếp tục materialize / stabilize / reverse
- Với mã Watch closely W-1: tuần [N] có invalidate (Out) hay confirm (Hold) hay vẫn watch
- Với mã Vào mới W-1: tuần [N] performance ra sao
- Action item W-1: đã materialize chưa, kết quả gì

**Bước 3 — Compose eval block (gọn):**

Eval block 4 phần (ngắn hơn monthly eval):

1. **Status carry-over** — bảng: trục W-1 | status W-1 | thực tế tuần [N] | đánh giá (đúng / lệch / chưa rõ)
2. **Watchlist W-1 tracking** — bảng: ticker | trạng thái W-1 | biến động tuần [N] | đánh giá
3. **Action item W-1** — đã materialize chưa, kết quả
4. **Carry-forward** — 2-3 dòng learning cho tuần [N] tracking

**Bước 4 — Checkpoint 0:**

```
─── ĐÁNH GIÁ TUẦN W-1 — Eval block ───

[Eval block 4 phần]

Confirm hay refine trước khi tiếp Stage 1 tracking tuần [N]?
- (a) Accept eval, integrate vào Stage 1
- (b) Refine — [user nêu]
- (c) Skip carry-forward
```

**Output Stage 0 trong báo cáo cuối:** mục "Review tuần W-1" render đầy đủ 4 phần (xem render spec `O_vbse_strategy_00`). Nếu user skip Stage 0, không render mục này — đi thẳng vào tóm tắt tuần [N].

## 4. Stage 1 — Tracking & Update

**Bước 1 — Extract từ monthly:**

Đọc file monthly user upload, extract:
- Regime vĩ mô đã call
- 2-5 themes đã chốt (tên + cơ chế)
- Sector bias (quan tâm / trung tính / thận trọng)
- 3 kịch bản VNINDEX (trigger từng kịch bản)
- Risk map (3-7 rủi ro + signal materialize)
- Watchlist 20 mã (10 Tier 1 + 10 Tier 2 mặc định; điều chỉnh theo regime Bear 8/12 hoặc quá mua 6/14 — signals theo dõi từng mã + tier marker + bucket entry)

Hệ thống signals này là "spine" để tuần này tracking.

**Bước 2 — Quét nhanh 6 trục tuần qua:**

Cho mỗi trục, agent query nhanh data tuần qua, so với thesis monthly:

| Trục | Câu hỏi check |
|---|---|
| 1 — Vĩ mô | Lãi suất / tỷ giá / FII / dòng vốn quốc tế tuần qua có shift gì không so với regime đã call? |
| 2 — Định vị thị trường | VNINDEX tuần qua: định giá / dòng tiền / breadth có chuyển pha không? |
| 3 — Themes | Theme nào có catalyst materialize tuần qua (tích cực)? Theme nào fizzle / bị đảo? |
| 4 — Sector | Ngành quan tâm có duy trì dòng tiền + giá không? Ngành thận trọng có tệ hơn? Có sector rotation mới? |
| 5 — Risk | Rủi ro nào trong risk map có signal materialize tuần qua? |
| 6 — Watchlist | Mã nào trong watchlist có signal hold / invalidate? Có mã mới đáng vào theo theme cũ? |

**Bước 3 — Update từng trục:**

**Technical-as-noise rule (BẤT BIẾN, theo `P_vbse_strategy_00` mục 4.3):** Status Shift/Materialize bắt buộc kèm signal vĩ mô / cơ bản / chính sách. Technical shift đơn độc (VNINDEX MA tụt, industry rank tụt, technical_zone yếu đi) → "noise tạm thời, status Hold". Ngoại lệ: rebucket entry watchlist.

Cho mỗi trục, 1 trong 3 status:
- **Hold:** không có shift, thesis monthly còn valid → ghi 1-2 dòng note
- **Shift:** có thay đổi cần điều chỉnh → mô tả shift + ngụ ý mới (vd "Theme A weaken vì catalyst bị delay → giảm priority")
- **Materialize (risk):** rủi ro đã xảy ra → cảnh báo + phản ứng định tính

**Bước 4 — Refresh watchlist:**

| Trạng thái | Action |
|---|---|
| Hold | Mã + thesis còn valid, signals theo dõi vẫn ổn |
| Watch closely | Có dấu hiệu yếu, signal cảnh báo bắt đầu xuất hiện nhưng chưa invalidate |
| Out | Signal invalidate đã xảy ra (vd BCTC Q1 ra dưới kỳ vọng, dòng tiền tuần âm 2 tuần liên tiếp, technical break-down) |
| Vào mới | Mã mới phát hiện trong tuần, đáp ứng tiêu chí theme hiện hữu — ghi rõ theme nào, signals theo dõi |

**Rebucket entry (Trục 6 Phase 2):** với mã Hold/Watch closely trong watchlist, re-check technical_zone w/m/q/y → rebucket Bucket 1/2/3 (xem `P_vbse_strategy_06`). Đây là **ngoại lệ duy nhất** được PTKT-driven trong weekly tracking. Mã shift bucket KHÔNG nâng/giảm conviction theme — chỉ thay đổi entry timing.

**Bước 5 — Action item tuần tới (1-2 item định tính):**

Vd:
- "Theo dõi FOMC minutes thứ Tư, signal cho theme 'Margin ngân hàng cải thiện cuối chu kỳ hạ lãi suất'"
- "Quan sát phản ứng nhóm thép sau release CPI Trung Quốc, có thể trigger shift sector bias"

**KHÔNG có:** entry/exit cụ thể, level giá, kích thước vị thế, %.

## 5. Render weekly update

Gọi `O_vbse_strategy_00` render mode weekly. Target 3-5 trang MD (ngắn hơn monthly nhiều vì là tracking, không build từ đầu). File `vbse_strategy_weekly_<YYYYMMDD>.md`.
