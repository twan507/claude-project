# P_vbse_strategy_04 — Trục 4: Sector allocation strategy

File này detail Trục 4 của framework 6 trục. Dependency: `P_vbse_strategy_00` master (philosophy fundamental-driven + weight balance + Nguyên tắc 3 whitelist 18 ngành).

**Master weight balance áp dụng cho Trục 4:**
- Trục 4 thuộc tầng **PRIMARY** (cross 3 trục đầu + BCTC + định giá + catalyst) + một phần SECONDARY (flow)
- **Cap technical: ≤ 5% nội dung trục** — gần như 0
- **TERTIARY technical zone ngành: BỎ HOÀN TOÀN** so với pack invest_strategy cũ — sector bias KHÔNG bị technical quyết

> ## ⚠️ FUNDAMENTAL-FIRST OVERRIDE — BỎ TECHNICAL CONFIRMATION
>
> Trong pack `P_invest_strategy` cũ, Trục 4 có lăng kính TERTIARY "technical zone đa khung của ngành" làm confirmation timing. **Pack này BỎ HOÀN TOÀN lăng kính đó.**
>
> Sector bias (quan tâm / trung tính / cần thận trọng) chỉ chốt bằng:
> 1. Cross 3 trục đầu (vĩ mô + định vị + theme)
> 2. BCTC ngành (`industry_finstats` quarterly EPS / doanh thu / biên)
> 3. Định giá ngành phân vị lịch sử
> 4. Chính sách / catalyst ngành cụ thể
> 5. Cross-check dòng tiền ngành (SECONDARY, không quyết định độc lập)
>
> Ngành có cơ bản tốt + technical_zone yếu → **vẫn vào "ngành quan tâm"**. Technical timing để **Trục 6 Phase 2 Bucket entry** handle cho mã cụ thể.

## 1. Mục tiêu & câu hỏi cốt lõi

**Câu hỏi:** với regime vĩ mô + định vị thị trường + themes đã chốt, **ngành nào trong whitelist 18 nên quan tâm**, ngành nào trung tính, ngành nào cần thận trọng tháng/quý tới?

**Scope ngành: 18 ngành whitelist** (xem `P_vbse_strategy_00` Nguyên tắc 3 + `K_agent_db_01` Section B). Ngành ngoài whitelist KHÔNG vào sector tilts.

**Output:** phân tầng định tính 18 ngành whitelist → 3 nhóm (quan tâm / trung tính / cần thận trọng) kèm bảng tilts tổng hợp.

## 2. Lăng kính phân tích (theo Weight balance)

### 2.1. PRIMARY — Driver chính (~80-85% nội dung trục)

**(a) Cross-check 3 trục đầu**

Ngành ở **giao** của 4 điều kiện sau → ứng viên "ngành quan tâm":
- Vĩ mô hỗ trợ (Trục 1 — vd lãi suất hạ + tỷ giá ổn → ngân hàng + BĐS hưởng lợi)
- Định vị thị trường thuận lợi (Trục 2 — định giá phân vị thấp + dòng tiền cải thiện)
- Có theme dẫn dắt rõ (Trục 3 — vd "Sóng đầu tư công Q2" → CONGNGHIEP + XAYDUNG + KIMLOAI)
- Earnings outlook tích cực (BCTC ngành Q gần nhất + kỳ vọng Q tới)

**(b) Tăng trưởng cơ bản ngành**

`industry_finstats.financial_statements.quarterly`:
- Tăng trưởng EPS Q gần nhất YoY
- Tăng trưởng doanh thu Q gần nhất YoY
- Xu hướng biên gộp / biên ròng 4-8 quý
- ROE trung bình ngành (TTM)
- Kỳ vọng Q tới (consensus qua web search nếu data DB chưa có)

**Diễn giải fundamental:** EPS YoY ≥ 15% liên tục 2-3 quý + biên gộp mở rộng → cơ bản đang accelerate → ứng viên "quan tâm". EPS YoY giảm rộng + biên thu hẹp → ứng viên "cần thận trọng".

**(c) Định giá ngành tương đối lịch sử**

`industry_finstats.valuation_ratios`:
- P/E hiện tại so median 3-5 năm — phân vị
- P/B hiện tại so median 3-5 năm — phân vị
- Re-rating opportunity hay risk?

**Diễn giải:**
- P/E phân vị < 30% + cơ bản đang accelerate → re-rating opportunity → upgrade conviction
- P/E phân vị > 70% + cơ bản chưa accelerate tương ứng → de-rating risk → downgrade conviction hoặc loại

**(d) Chính sách / catalyst ngành cụ thể**

- `news_history_feed` filter sector + ticker đại diện ngành
- Web search Nghị quyết / Luật / Quyết định bộ ngành mới
- Mùa BCTC quý nào sắp đến cho ngành
- Dividend cycle, M&A pipeline, niêm yết mới, FTSE/MSCI rebalance
- Capacity online, expansion plan

**(e) Sensitivity vĩ mô — cross với Trục 1**

Reference mapping ngành ↔ sensitivity (tham khảo `K_agent_db_00` mục 8):

| Ngành whitelist | Sensitivity chính |
|---|---|
| NGANHANG | Lãi suất điều hành NHNN, tỷ giá USD/VND |
| BDS | Lãi suất huy động, lãi suất cho vay, chính sách tín dụng BĐS |
| KCN | FDI, chính sách KCN, lãi suất |
| DAUKHI | Dầu Brent, WTI |
| KIMLOAI | Quặng sắt, than cốc, HRC |
| HOACHAT | Giá ure, urê, kali, khí tự nhiên |
| THUYSAN | USD/VND, EUR/USD, giá đầu vào nông sản |
| DETMAY | USD/VND, lương tối thiểu, giá bông |
| NONGNGHIEP | Giá cà phê, gạo, cao su, đường, thức ăn chăn nuôi |
| THUCPHAM | Giá đầu vào nông sản, sức tiêu dùng |
| BANLE | Sức tiêu dùng, lãi suất cho vay |
| CHUNGKHOAN | Thanh khoản thị trường, lãi suất, NN flow |
| CONGNGHE | USD/VND (xuất khẩu IT), policy số hoá |
| CONGNGHIEP | Đầu tư công, vốn FDI, lãi suất |
| KHOANGSAN | Giá hàng hoá nông sản, chính sách XK |
| TIENICH | Lãi suất, chính sách điện-nước-môi trường |
| VANTAI | Giá dầu, thương mại quốc tế |
| XAYDUNG | Đầu tư công, giá thép-xi măng, lãi suất |

Khi Trục 1 chốt regime, agent cross-check ngành nhạy gì với regime đó để upgrade/downgrade conviction sector.

### 2.2. SECONDARY — Cross-check, không quyết định độc lập (~10-15% nội dung)

**(f) Dòng tiền ngành — cross-check thesis**

- `industry_snapshot.money_flow_score.week_score` tuần gần (dòng tiền tuần — **field chuẩn để rank ngành**)
- Xu hướng 4 tuần qua `industry_recent`
- Rank ngành — **tự tổng hợp** theo `week_score` qua 18 ngành whitelist (DB không lưu `industry_rank` tĩnh — xem `K_agent_db_01` mục "Xếp hạng ngành")

**Vai trò:** confirmation cho thesis cơ bản, không quyết định độc lập. Ngành cơ bản tốt + dòng tiền yếu → ghi rõ tension, vẫn vào "quan tâm" với conviction giảm 1 bậc (vd HIGH → MID).

**(g) Breadth nội bộ ngành — dẫn dắt thật vs trụ kéo**

- `industry_snapshot.breadth.breadth_in / breadth_out / breadth_neu` phiên gần
- Count `stock_snapshot.change.m_pct > 0` trong ngành (lấy `full_ticker_list` từ `industry_info`)
- **Quy tắc dẫn dắt thật:** đa số mã trong ngành tăng giá tháng (≥60%) → dẫn dắt thật. Vài mã lớn kéo (≤30%) → trụ kéo, conviction giảm. Gần 50/50 → rotation nội bộ, conviction trung tính.

**(h) Crowding check**

Cross-reference `market_nntd` aggregate + `industry_snapshot.breadth`:
- Ngành đã rally mạnh + FII mua dồn dập + breadth rộng → "consensus crowded" → rủi ro thoái lui khi thesis lệch nhỏ → giảm conviction
- Ngành cơ bản tốt + dòng tiền chưa vào + FII trung tính → "contrarian" → upgrade conviction nếu catalyst sắp materialize

### 2.3. TERTIARY — KHÔNG dùng (Cap technical ≤ 5%)

**Bỏ hoàn toàn:**
- Technical_zone đa khung của ngành (`industry_snapshot.technical_zone.overall`)
- MA, Fibonacci, volume profile ngành

**Lý do bỏ:**
1. Technical timing là job của Trục 6 Phase 2 Bucket entry cho mã cụ thể, không phải sector bias
2. Pack invest_strategy cũ có rule "ngành có cơ bản tốt + định giá rẻ nhưng technical yếu vẫn quan tâm" — đã ngầm thừa nhận technical không quyết định. Pack này làm rule đó explicit bằng cách bỏ hẳn lăng kính.
3. Sector bias horizon 1-3 tháng. Technical zone w/m biến động nhanh hơn timeline đó nhiều — không phù hợp.

**Ngoại lệ ≤5%:** chỉ ghi nhận technical zone ngành trong **disconfirming signal** nếu thực sự cần (vd "technical zone đa khung ngành tụt từ AAA → C trong 2 tuần liên tiếp KÈM dòng tiền âm 3 tuần liên tiếp" — đây là technical + flow signal combo, không phải technical đơn độc).

## 3. Hướng tìm dữ liệu

| Loại data | Collection | Field cụ thể | Tầng |
|---|---|---|---|
| BCTC ngành quarterly | `industry_finstats` (18 doc whitelist) | `financial_statements.quarterly` — EPS, doanh thu, biên gộp, ROE | PRIMARY |
| Định giá ngành lịch sử | `industry_finstats` (18 doc whitelist) | `valuation_ratios.PE`, `PB` + percentile vs median 3-5Y | PRIMARY |
| Chính sách / catalyst ngành | `news_history_feed` filter sector + web search | `news_type` + body content | PRIMARY |
| Mùa BCTC / consensus | Web search | — | PRIMARY |
| Sensitivity vĩ mô mapping | `K_agent_db_00` mục 8 + bảng mục 2.1.e trên | — | PRIMARY |
| Dòng tiền ngành tuần + xu hướng 4 tuần | `industry_snapshot` (18 doc) + `industry_recent` | `money_flow_score.week_score`; rank tự tổng hợp 1..18 theo `week_score` | SECONDARY |
| Breadth nội bộ ngành | `industry_snapshot` (18 doc) + `stock_snapshot` filter industry | `breadth.breadth_in/out/neu`, `change.m_pct` count | SECONDARY |
| Crowding | `market_nntd` aggregate + `industry_snapshot.breadth` | net flow tháng/quý + breadth | SECONDARY |
| Rotation large/mid/small | `group_snapshot` (6 nhóm vốn hoá) | so sánh nhóm vs nhóm | SECONDARY |

**Trọng số nguồn ước:** ~80% DB + ~15% web search + ~5% file user upload.

## 4. Output diễn giải — 3 tầng

Phân tầng định tính (KHÔNG ép số ngành mỗi tầng):

### 4.1. Ngành quan tâm

Tiêu chí (cross-check ≥3/4 thuận lợi):
- ✓ Vĩ mô hỗ trợ
- ✓ Định vị thị trường thuận lợi
- ✓ Có theme dẫn dắt rõ
- ✓ Earnings outlook tích cực (EPS YoY ≥ 15% liên tục, biên mở rộng)
- Định giá phân vị < 60%
- Catalyst chính sách / mùa BCTC trong 1-3 tháng tới

Conviction theo mức cross-check:
- **HIGH:** cả 4 trục thuận + định giá phân vị < 40% + có catalyst ngày cụ thể
- **MID:** 3/4 trục thuận + định giá phân vị 40-60% + catalyst rõ nhưng chưa có ngày
- **LOW:** 3/4 trục thuận, không có catalyst rõ ngày cụ thể

### 4.2. Ngành trung tính

- Signal hỗn hợp (2/4 trục thuận, 2/4 trục ngược)
- Không có theme dẫn dắt
- Đang chuyển pha (BCTC quý gần nhất bị ảnh hưởng one-off)
- Định giá phân vị 50-70% — không rẻ không đắt

### 4.3. Ngành cần thận trọng

- Vĩ mô áp lực + định vị xấu + không theme
- Định giá phân vị > 75% (đắt tương đối)
- Earnings deceleration (EPS YoY giảm rộng, biên thu hẹp 2+ quý liên tiếp)
- Catalyst tiêu cực (chính sách siết, commodity headwind cấu trúc)

### 4.4. Bias diễn đạt

- **Dùng:** "quan tâm / trung tính / cần thận trọng"
- **KHÔNG dùng:** "overweight / underweight / neutral" (vocab portfolio manager — không tương thích audience KH)

## 5. Bảng tilts tổng hợp (BẮT BUỘC)

Single-page scannable, chuẩn buy-side. Mỗi ngành 1 dòng:

| Ngành | Bias | Conviction | Theme/driver chính | Signal hỗ trợ | Disconfirming signal |
|---|---|---|---|---|---|
| NGANHANG | Quan tâm | HIGH | Margin cải thiện cuối chu kỳ hạ lãi suất | EPS Q1/26 +18% YoY (consensus), P/B 1.4x phân vị 28% | NIM Q2 thu hẹp ≥20bp QoQ; chính sách thắt tín dụng mới |
| ... | ... | ... | ... | ... | ... |

**Quy tắc:**
- Render đủ 18 ngành whitelist (kể cả ngành trung tính ghi 1 dòng ngắn)
- Signal hỗ trợ kèm **số cụ thể** (không "tốt", "cải thiện" chung chung)
- Disconfirming kèm **threshold cụ thể** (số / sự kiện cụ thể)

## 6. Cross-reference đầu ra Trục 4

Output Trục 4 feed vào:
- **Trục 6 watchlist** (`_06`): mỗi ngành quan tâm → 1-3 mã đại diện
- **Trục 5 risk** (`_05`): ngành nào "consensus crowded" → thêm risk thoái lui
- **Workflow Monthly Stage 2** (`_07`): Trục 4 chốt cùng Trục 5 trong stage 2 sau Checkpoint 1

## 7. Edge cases

- **Mâu thuẫn PRIMARY:** vd ngành có vĩ mô hỗ trợ + theme rõ nhưng BCTC Q gần nhất một-off xấu → ghi rõ "one-off, cơ chế ngành chưa đổi" → vẫn quan tâm với conviction MID + add disconfirming signal "Q tới một-off chưa được absorb"
- **Ngành nhạy commodity biến động mạnh:** vd HOACHAT phụ thuộc giá ure, ure đang up 25% trong 1 tháng nhưng chưa rõ chu kỳ → ghi rõ "catalyst commodity ngắn hạn, conviction LOW cho horizon 1-3 tháng (giá ure có thể correct), MID cho horizon 3-6 tháng nếu chu kỳ thực sự shift"
- **Mã hoặc nhóm mã trong ngành dominant kéo cả ngành:** vd NGANHANG có VCB + BID kéo 70% biến động ngành → phân tích sector tilt phải cross-check breadth nội bộ. Nếu breadth thấp → ghi rõ "trụ kéo, không phải dẫn dắt thật", conviction giảm
- **Ngành ngoài whitelist** (vd Bảo hiểm, Y tế Dược phẩm): **KHÔNG vào sector tilts**. Mã thuộc các ngành đó có thể phân tích đơn lẻ nếu user hỏi đích danh, nhưng không vào watchlist theme. Xem `K_agent_db_00` mục 4.5.
- **Catalyst override:** ngành không đạt tăng trưởng BCTC + định giá hợp lý nhưng có catalyst rất mạnh (vd FTSE/MSCI upgrade VN với ngành X nằm trong top inflow) → có thể vào "quan tâm" LOW conviction, ghi rõ "catalyst-driven, fundamental không support" + disconfirming "catalyst delay hoặc bị huỷ"
- **Dữ liệu `industry_finstats` thiếu cho 1 ngành:** ghi rõ "thiếu BCTC Q gần nhất, dùng N-1 + web consensus" hoặc xếp tạm "trung tính" cho đến khi data về

## 8. Self-audit Trục 4 (trước khi xuất)

- [ ] Bảng tilts render đủ 18 ngành whitelist (không bị thiếu, không có ngành ngoài whitelist)
- [ ] Mỗi ngành "quan tâm" có ≥3/4 trục PRIMARY thuận
- [ ] Signal hỗ trợ kèm số cụ thể, KHÔNG chung chung
- [ ] Disconfirming signal kèm threshold cụ thể
- [ ] Rank ngành đã **tự tổng hợp** theo `week_score` qua 18 ngành whitelist (DB không lưu `industry_rank` tĩnh)
- [ ] **% nội dung technical trong trục ≤ 5%** — không có lăng kính TERTIARY technical confirmation
- [ ] Bias dùng "quan tâm / trung tính / cần thận trọng", KHÔNG "overweight/underweight"
- [ ] Cross-reference Trục 6 đã ghi rõ ngành nào → mã đại diện nào (lazy, sẽ chi tiết ở Trục 6)
- [ ] Mapping sensitivity vĩ mô đúng theo bảng mục 2.1.e

Vi phạm bất kỳ item nào → re-write trước khi render.
