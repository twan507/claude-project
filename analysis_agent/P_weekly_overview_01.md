# P_weekly_overview_01 — Pre-flight + Stage 1 first half (Phần 2-5)

File này cover Pre-flight 3 câu + compose phần 2-5 của báo cáo `weekly_overview`. Dependency: master `P_weekly_overview_00` cho philosophy fundamental-driven, weight balance (cap technical ≤ 15% tổng), từ điển thuật ngữ, whitelist 18 ngành, 4 nguyên tắc bất biến. Phần 6-9 ở `P_weekly_overview_02`; Checkpoint 1 + phần 10-12 + phần 1 ở `P_weekly_overview_03`; methodology + self-audit + edge + contract ở `P_weekly_overview_04`. Render output qua `O_weekly_overview_00` → file `weekly_overview_<YYYYMMDD>.md`.

## 1. Pre-flight — hỏi user trước khi vào Stage 1

Pack `P_weekly_overview` là **broadcast tuần độc lập** — KHÔNG yêu cầu thesis monthly hoặc parent state nào. Agent có thể chạy pack này bất cứ tuần nào user trigger, không cần state file ngoài.

Agent hỏi 1 turn 3 câu, user trả lời rồi mới chạy:

```
Trước khi tôi bắt đầu compose báo cáo tổng quan tuần [DD/MM-DD/MM/YYYY], xác nhận 3 điểm:

1. File báo cáo tuần W-1:
   (a) Có, tôi gửi đính kèm
   (b) Không có / tuần đầu cycle / skip phần Review

2. Có context đặc biệt cần lưu ý không?
   (a) Không, chạy default
   (b) Có — [user nêu]: focus ngành / sự kiện đặc biệt / override scope

3. Branding & disclaimer info (báo cáo có thể dùng cho cả nội bộ và gửi khách hàng):
   (a) Có — vui lòng cung cấp: tên công ty, logo (optional), hotline, website, phòng ban biên soạn, nội dung disclaimer mong muốn
   (b) Không cần — render bản plain
```

User trả lời → agent ghi nhận + chạy Stage 1. Branding info user gửi sẽ được insert vào header và footer file MD final ở bước render (xem `O_weekly_overview_00`).

## 2. Phần 2 — Review tuần trước (scorecard format)

**Skip nếu:** user chọn (b) ở pre-flight (không có file W-1). Ghi 1 dòng: "Tuần đầu cycle hoặc không có file W-1, chưa có dữ liệu review."

**Compose nếu:** user upload file MD báo cáo W-1.

**Đọc từ file W-1:**
- 3 kịch bản VNINDEX phần 9 → so sánh với thực tế tuần vừa qua
- Sector bias phần 10 (ngành quan tâm + ngành cần thận trọng, kèm conviction cũ) → kiểm tra biến động giá tuần thực tế từng ngành
- Watchlist phần 10 (ticker + conviction + horizon cũ + luận điểm) → cross-check biến động giá + flow + có signal trigger nào kích hoạt chưa
- Risk map phần 9.4 → rủi ro nào đã materialize

**Output structured — 3 scorecard table + Best/Worst + Carry-forward:**

### 2.1. Scorecard kịch bản match

| Kịch bản W-1 | Trigger gốc | Thực tế tuần | Match (✓/✗) |
|---|---|---|---|
| Cơ sở | [trigger từ file W-1, ví dụ "VNINDEX giữ trên POC quý + flow trung tính"] | [tóm tắt thực tế tuần] | ✓ / ✗ |
| Tích cực | [trigger break-out gốc] | [thực tế] | ✓ / ✗ |
| Tiêu cực | [trigger break-down gốc] | [thực tế] | ✓ / ✗ |

Nếu cả 3 kịch bản đều miss (thị trường đi theo đường ngoài kỳ vọng): ghi 1 dòng "Cả 3 kịch bản miss — thực tế: [mô tả ngắn]" làm note dưới bảng.

### 2.2. Scorecard sector bias

**Ngành quan tâm W-1:**

| Ngành | Conviction cũ | m_pct thực tế (tuần) | Hit/Miss |
|---|---|---|---|
| [ngành 1] | HIGH/MID/LOW | [+X% hoặc -Y%] | Hit / Miss |
| ... | | | |

**Ngành cần thận trọng W-1:**

| Ngành | Conviction cũ | m_pct thực tế (tuần) | Hit/Miss |
|---|---|---|---|
| [ngành A] | HIGH/MID/LOW | [+X% hoặc -Y%] | Hit / Miss |
| ... | | | |

**Hit rate tổng:** [N/M] ngành quan tâm tăng giá tuần, [N/M] ngành thận trọng giảm giá tuần.

(Quy ước Hit: ngành quan tâm có biến động giá tuần dương; ngành thận trọng có biến động giá tuần âm. Conviction HIGH miss = penalty nặng hơn LOW miss khi rút learning.)

### 2.3. Scorecard watchlist

| Ticker | Conviction cũ | Horizon cũ | m_pct thực tế (tuần) | Signal trigger | Hit/Miss |
|---|---|---|---|---|---|
| [TICKER 1] | HIGH/MID/LOW | 1-2 tuần / 2-4 tuần | [+X% / -Y%] | [signal disconfirming kích hoạt chưa? catalyst dự đoán đã có chưa?] | Hit / Miss |
| ... | | | | | |

(Hit: hướng tích cực → giá tăng tuần hoặc signal catalyst kích hoạt; hướng tiêu cực → giá giảm tuần hoặc signal rủi ro materialize.)

### 2.4. Best/Worst call + Carry-forward

- **Best call:** 1-2 dòng — call nào của tuần trước đúng nhất (kịch bản / ngành / mã), ngắn gọn lý do tại sao nó đúng.
- **Worst call:** 1-2 dòng honest — call nào lệch xa nhất so thực tế, lý do (vĩ mô bất ngờ / catalyst miss / flow đảo ngược / technical false signal). Không che giấu, không spin.
- **Carry-forward:** 1-3 dòng learning áp dụng cho tuần mới — ví dụ "ngành A vẫn giữ conviction HIGH vì disconfirming chưa kích", "ngành B downgrade từ quan tâm xuống trung tính do flow tuần qua yếu rõ", "thông điệp policy NHNN tuần tới là input quyết định regime".

## 3. Phần 3 — Bối cảnh quốc tế

**Cap technical phần này: 0%** — pure macro data, không có vùng kỹ thuật chỉ số quốc tế.

**Query DB:**

`other_data` filter:
- group `international.global_index`: S&P 500, Dow Jones, Nasdaq, Nikkei 225, Shanghai Composite
- group `international.fx`: EUR/USD, GBP/USD, USD/JPY, USD/CNY, DXY
- group `international.bonds`: TPCP Mỹ 10Y
- group `macro.monetary` của các NHTW lớn (FED, ECB, PBOC) — nếu có trong DB; nếu không, web search

Lấy field: `name`, `value`, `unit`, `pct_change` (phiên gần nhất), `w_pct`, `m_pct`, `update_date`.

**Web search nếu cần:** lãi suất điều hành FED/ECB/PBOC nếu DB không có, hoặc cập nhật phát biểu Fed/ECB tuần qua.

**Output structured:**
- Bảng chứng khoán quốc tế: 5-6 chỉ số, biến động tuần + tháng
- Bảng tỷ giá quốc tế: 5 cặp + DXY
- Bảng lãi suất quốc tế: 4 mục
- Tổng kết prose 4-5 dòng: tâm lý chung quốc tế, áp lực USD, kỳ vọng lãi suất. **Câu cuối bắt buộc:** kết luận **implied conviction (HIGH/MID/LOW) cho forward bias EM equity** từ context quốc tế tuần này (ví dụ: "Context quốc tế tuần ủng hộ EM equity với conviction MID — DXY giảm tuần + lợi suất TPCP 10Y Mỹ ổn định, nhưng tín hiệu Fed chưa rõ ràng đủ để upgrade HIGH"). Đây là input cho Checkpoint 1 regime call (theo Nguyên tắc 3 master).

## 4. Phần 4 — Thị trường Việt Nam (aggregate qua 18 ngành whitelist)

**Note bắt buộc đầu phần:** Aggregate proxy thị trường tính trên **18 ngành whitelist** (xem `K_agent_db_01` Section B), **KHÔNG 24 ngành raw** — theo Nguyên tắc 2 master `P_weekly_overview_00`. Mọi ranking, mean/median dòng tiền cấp thị trường, breadth ngành, top NN/TD đều filter trong scope 18 ngành.

**Query DB:**

1. `market_snapshot` (1 doc duy nhất):
   - `price.close` (VNINDEX), `price.pct_change`, `price.trading_value` (GTGD phiên cuối tuần)
   - `change.w_pct`, `m_pct`, `q_pct`, `y_pct`
   - `breadth.breadth_in/out/neu` (rổ FNXINDEX, không phải toàn HOSE — xem `K_agent_db_01` mục D)

2. `market_recent` slice 5 phiên (`recent_price[0..4]`):
   - Tính GTGD trung bình tuần, biến động tuần từ `price.close`, `price.trading_value`
   - **Lưu ý:** `market_recent` KHÔNG có `money_flow_score` (xem `K_agent_db_01` mục D). Để có chuỗi 5 phiên dòng tiền cấp thị trường, dùng aggregate từ `industry_snapshot.money_flow_score.week_score` **18 ngành whitelist** (cấp ngành) hoặc `group_snapshot.money_flow_score.day_score` 6 nhóm.

3. `industry_snapshot` filtered to **18 ngành whitelist** (xem `K_agent_db_01` Section B) aggregate `money_flow_score.week_score` lấy mean/median làm proxy điểm dòng tiền thị trường tuần. Aggregate `money_flow_score.day_score` 18 ngành làm proxy điểm dòng tiền phiên cuối tuần.

4. `industry_recent` slice 5 phiên × **18 ngành whitelist** — aggregate `series[i].money_flow_score.day_score` mean theo phiên để có chuỗi 5 phiên dòng tiền proxy thị trường.

5. `market_nntd` slice 5 phiên — net_value tuần, top mua/bán ròng tuần

6. `data_briefing` block market — breadth_in/out toàn thị trường phiên cuối tuần

7. `other_data` filter group `macro.exchange_rate` + `macro.monetary` (lãi suất liên ngân hàng, OMO, tỷ giá VCB)

**Methodology aggregate NN/TD tuần:**

Tổng net_value 5 phiên gần nhất = NN net tuần. Top 5 mã NN mua ròng tuần lấy từ `stock_nntd` aggregate net_value 5 phiên (sort desc), top 5 bán ròng tương tự (sort asc). Filter:
- Thanh khoản tối thiểu 5 tỷ/phiên trung bình tuần (loại nhiễu penny)
- **Filter `industry ∈ 18 ngành whitelist`** — mã thuộc ngành ngoài whitelist không xuất hiện trong bảng top NN/TD (theo Nguyên tắc 2)

**Output structured:**
- VNINDEX: giá đóng, biến động tuần/tháng/quý
- Thanh khoản: GTGD phiên cuối tuần + GTGD trung bình tuần + biến động vs trung bình tháng
- Dòng tiền nội: điểm dòng tiền tuần thị trường (proxy aggregate **18 ngành whitelist**), chuỗi điểm dòng tiền phiên 5 phiên (mô tả pattern: đồng đều dương / đồng đều âm / dao động / phục hồi cuối tuần)
- Breadth phiên cuối tuần: số ngành (trong 18) tăng/giảm, số mã tăng/giảm toàn thị trường
- NN/TD tuần: tổng mua/bán ròng + top 5 mỗi chiều (trong scope 18 ngành whitelist)
- Tỷ giá VN, lãi suất liên ngân hàng các kỳ hạn, OMO

## 5. Phần 5 — Vĩ mô & hàng hoá — institutional impact table 5 cột

**Phần quan trọng nhất để chuẩn bị regime call.** Phần này thiết lập context "ngành nào đang được vĩ mô tác động đáng kể tuần này" trước khi đọc bảng 18 ngành ở phần 6.

**Cap technical: 0%** — pure macro/commodity data.

**Workflow logic (reverse — quét chỉ số trước, suy ngành sau):**

1. Quét toàn bộ chỉ số vĩ mô + commodity tuần qua, **detect chỉ số nào có biến động đáng kể** theo 2 chiều Magnitude + Persistence (định nghĩa sub-section 5.4)
2. Với mỗi chỉ số đáng kể → suy ra ngành VN bị tác động qua mapping cơ chế
3. Output bảng kết luận chỉ liệt kê các ngành CÓ signal vĩ mô tuần này, không liệt kê 18 ngành cố định
4. Nếu tuần không có biến động vĩ mô nào đáng kể → bỏ qua sub-section 5.4, không ghi "không có gì đặc biệt"

**Query DB:**

`other_data` filter:
- **Lãi suất**: `macro.monetary` đầy đủ — lãi suất tái cấp vốn, chiết khấu, liên ngân hàng các kỳ hạn (2W/1M/3M), huy động, cho vay qua đêm
- **Tỷ giá**: `macro.exchange_rate` đầy đủ + `international.fx` (EUR/USD, USD/CNY)
- **Hàng hoá**:
  - `commodities.energy`: Dầu Brent, WTI, khí tự nhiên, than nhiệt, than cốc
  - `commodities.metals`: Quặng sắt, thép HRC, vàng, bạc, đồng
  - `commodities.chemical`: Urea Trung Đông, urea Trung Quốc, phốt pho vàng, nhựa PP/PVC/PET
  - `commodities.agriculture`: cà phê, hồ tiêu, cao su, gạo XK, đường, ngô, đậu tương, heo hơi, tôm thẻ

**Output structured:**

### 5.1. Lãi suất

Bảng số liệu (lãi suất điều hành + liên ngân hàng các kỳ hạn + huy động/cho vay) + 2-3 dòng diễn giải biến động tuần qua.

### 5.2. Tỷ giá

Bảng số liệu (USD/VND VCB, USD/VND tự do, EUR/VND, CNY/VND + cặp quốc tế liên quan) + 2-3 dòng diễn giải.

### 5.3. Hàng hoá

Bảng phân theo nhóm (năng lượng / kim loại / hoá chất / nông sản) — số liệu thuần (giá, đơn vị, % tuần, % tháng), không có cột "ngành nhạy" trong bảng này. 2-3 dòng diễn giải nhóm nào biến động đáng kể nhất tuần.

### 5.4. Tác động lên ngành VN tuần này — institutional impact table 5 cột

Bảng 5 cột (UPGRADE từ bảng 3 cột cũ):

| Chỉ số biến động đáng kể (tuần) | Magnitude | Persistence | Ngành VN bị tác động (whitelist 18) | Hướng + cơ chế (1 dòng) |
|---|---|---|---|---|
| [Tên chỉ số + % tuần cụ thể] | Small / Material / Significant | Transitory / Trending / Structural | [Ngành whitelist 18] | [tích cực / tiêu cực] + cơ chế ngắn |

**Magnitude — định nghĩa định tính:**
- **Small:** biến động trong phạm vi dao động bình thường của chỉ số đó. Không thay đổi narrative.
- **Material:** biến động ngoài phạm vi bình thường nhưng chưa shock. Bắt đầu tạo áp lực biên gộp / chi phí / tâm lý đáng chú ý.
- **Significant:** shock — vượt ngưỡng tâm lý (dầu Brent vượt 100 USD, USD/VND vượt 26500, vàng vượt mốc tròn) hoặc pattern đảo chiều rõ. Tạo re-rating ngành rõ rệt.

**Persistence — định nghĩa định tính:**
- **Transitory:** biến động 1 lần, có thể revert tuần tới. Tác động ngắn hạn lên giá cổ phiếu nhưng không đổi fundamental.
- **Trending:** xu hướng đang hình thành, có thể kéo dài 1-2 tháng. Bắt đầu đổi mô hình earnings.
- **Structural:** shift cấu trúc dài hạn (chu kỳ chuyển pha — ví dụ chu kỳ lãi suất Fed bắt đầu hạ; chu kỳ commodity siêu vòng). Đổi narrative ngành nhiều quý.

**Quy tắc lọc bảng:**

Bảng chỉ liệt kê chỉ số có biến động **Magnitude ≥ Material** HOẶC **Persistence ≥ Trending**. Loại Small + Transitory ra khỏi bảng để giữ signal-to-noise cao.

Nếu tuần không có chỉ số nào pass ngưỡng lọc → bỏ qua sub-section 5.4 hoàn toàn (không render bảng rỗng, không ghi "không có gì đặc biệt").

**Quy tắc cột 4 (Ngành VN):**

Ngành VN BẮT BUỘC thuộc **whitelist 18 ngành** (`K_agent_db_01` Section B). Nếu cơ chế tác động chạm ngành ngoài whitelist (ví dụ giá đường tăng tác động ngành Mía đường — ngoài 18), vẫn được ghi nhưng đánh note "(ngoài scope pack)" — agent ghi để hoàn chỉnh chain logic vĩ mô, không dùng cho sector bias.

**Mapping cơ chế chỉ số → ngành VN:** tham chiếu trực tiếp `P_weekly_overview_04` mục 3 (Mapping vĩ mô-18 ngành) — pack **KHÔNG redefine bảng mapping trong file này**. File `_04` giữ single source of truth cho mapping; tránh duplicate maintenance.

**Mục đích sub-section 5.4:** input trực tiếp cho phần 6 (đọc bảng 18 ngành kèm context vĩ mô) và Checkpoint 1 (regime + sector bias + conviction). Ngành không xuất hiện ở 5.4 = vĩ mô không tác động trực tiếp tuần này, đánh giá qua flow + tin tức ở phần 6, 8.

## Rules cho file này

1. **Voice/tone:** Vietnamese terse, institutional buy-side voice, không emoji
2. **Cross-references đã chuẩn hoá:**
   - K pack: `K_agent_db_00`, `K_agent_db_01` (Section B = whitelist 18), `K_agent_db_05` — giữ intact
   - P pack: master `P_weekly_overview_00`, file này `_01`, `_02` (phần 6-9), `_03` (checkpoint + 10-12 + phần 1), `_04` (methodology + self-audit + mapping)
   - Render spec: `O_weekly_overview_00`
   - File output: `weekly_overview_<YYYYMMDD>.md`
3. **Cap technical phần 2-5:** Phần 3 = 0%, Phần 4 ≤ 10% (chỉ vùng kỹ thuật VNINDEX phụ nếu cần), Phần 5 = 0%. Tuân `P_weekly_overview_00` mục 4.1.
4. **18 ngành whitelist applied:** mọi aggregate cấp thị trường + ranking + filter NN/TD + cột Ngành VN bảng 5.4 đều scope 18, tham chiếu `K_agent_db_01` Section B (theo Nguyên tắc 2 master).
