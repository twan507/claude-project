# P_weekly_overview_03 — Checkpoint 1 + Stage 2 (Phần 10-12 + Phần 1)

File này covers Checkpoint 1 (regime + sector bias với conviction + disconfirming) + compose Stage 2 (phần 10-12 + phần 1 viết cuối). Dependency: master `P_weekly_overview_00` cho philosophy + nguyên tắc bất biến (đặc biệt Nguyên tắc 3 — conviction+horizon+disconfirming bắt buộc). Phần 6-9 ở `_02`, methodology ở `_04`. Render qua `O_weekly_overview_00`.

## 1. Checkpoint 1 — Regime + Sector bias + Conviction + Disconfirming

Đây là **checkpoint duy nhất** của workflow. Đặt giữa phần 9 và phần 10. Sau khi compose xong phần 2-9, agent KHÔNG render báo cáo mà compose block call sơ bộ và xuất tại checkpoint.

### 1.1. Method regime classification

4 input + 1 reasoning. Logic chi tiết ở `P_weekly_overview_04` mục 2.

**Input 1 — Dòng tiền thị trường** (aggregate proxy 18 ngành whitelist):
- Mean/median `industry_snapshot.money_flow_score.week_score` qua 18 ngành whitelist
- Chuỗi 5 phiên `industry_recent.series[0..4].money_flow_score.day_score` aggregate mean qua 18 ngành whitelist

**Input 2 — Breadth:**
- `data_briefing` doc `core`: `market.breadth` (in/out) phiên cuối tuần
- Số ngành whitelist 18 có w_pct > 0 / 18

**Input 3 — Khối ngoại:**
- `market_nntd` aggregate 5 phiên: net_value tuần
- Aggregate 20 phiên: net_value tháng (xu hướng)

**Input 4 — Vĩ mô context** (từ phần 5):
- Lãi suất / tỷ giá ổn định / shift signal
- Biến động commodity đáng kể tuần này (đếm từ sub-section 5.4)

### 1.2. Sector bias selection

Sau khi xác định regime, chọn sector bias từ bảng 18 ngành whitelist phần 6 cross-check sub-section 5.4 phần 5.

**Quy tắc dẫn dắt thật vs trụ kéo:**
- Đa số mã trong ngành cùng tăng giá tuần (≥60%) = dẫn dắt thật
- Vài mã vốn hoá lớn tăng trong khi đa số đứng/giảm (≤30%) = nghi trụ kéo, cảnh giác
- Phân hoá gần 50/50 = rotation nội bộ, cần thêm context

Chi tiết logic sector bias ở `P_weekly_overview_04` mục 2.4.

### 1.3. Conviction + Disconfirming bắt buộc (Nguyên tắc 3 master)

**Mới so với pack `P_weekly_market` cũ.** Mỗi call ở checkpoint phải kèm:

**Regime call:**
- **Conviction:** HIGH / MID / LOW
- **Disconfirming signal:** 1-2 chỉ báo cụ thể sẽ invalidate regime

**Sector bias mỗi ngành:**
- **Conviction:** HIGH / MID / LOW
- **Disconfirming signal:** 1 chỉ báo cụ thể sẽ invalidate bias (PREFER cơ bản/catalyst/chính sách, technical phụ)

### 1.4. Block xuất tại checkpoint

```
─── REGIME + SECTOR BIAS — Call sơ bộ ───

**Regime call:** [risk-on full / risk-on selective / defensive only / đứng ngoài]
**Conviction:** HIGH / MID / LOW

**Lý do (4 input + reasoning):**
- **Dòng tiền thị trường** (aggregate 18 ngành whitelist): điểm dòng tiền tuần [dương mạnh / dương nhẹ / trung tính / âm nhẹ / âm sâu], chuỗi 5 phiên [đồng đều dương / dao động / đồng đều âm / phục hồi cuối tuần]
- **Breadth:** [đa số / quá nửa / dưới nửa / phần lớn] 18 ngành whitelist tăng giá tuần; [N] mã tăng / [M] mã giảm phiên cuối
- **NN/TD tuần:** net = U tỷ ([mua ròng / bán ròng / trung tính])
- **Vĩ mô:** [tóm tắt 1-2 dòng — biến động đáng kể nào tuần qua tác động ngành VN, hoặc "không có biến động vĩ mô đáng kể"]

**Disconfirming signal cho regime call:**
- [Signal 1 cụ thể, PREFER macro/fundamental]
- [Signal 2 cụ thể, optional]

**Sector bias đề xuất (18 ngành whitelist):**

Ngành quan tâm [N ngành]:
- [Ngành 1] — Conviction HIGH — Lý do: [1 dòng cơ bản+flow] — Disconfirming: [signal cụ thể]
- [Ngành 2] — Conviction MID — Lý do: [...] — Disconfirming: [...]

Ngành cần thận trọng [M ngành]:
- [Ngành A] — Conviction HIGH — Lý do: [1 dòng cơ bản+vĩ mô áp lực] — Disconfirming: [signal sẽ chuyển trung tính]
- [Ngành B] — Conviction MID — Lý do: [...] — Disconfirming: [...]

Confirm/Override/Bổ sung trước khi tiếp phần 10-12?
- (a) Confirm như trên
- (b) Override regime → [user nêu regime mới + lý do]
- (c) Override sector bias → [user nêu thay đổi]
- (d) Cần phân tích thêm số liệu cụ thể trước khi quyết
```

### 1.5. Xử lý phản hồi user

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 2 chạy thẳng với regime + sector bias đã call |
| (b) Override regime | Ghi inline note trong báo cáo cuối phần 10: "Regime chốt sau review: X. Override note: [lý do user]". Stage 2 chạy với regime mới. Log trong metadata user overlay. |
| (c) Override sector bias | Ghi inline note tương tự. Stage 2 chạy với sector bias mới. Log overlay. |
| (d) Phân tích thêm | Query bổ sung theo yêu cầu user, refine call, hỏi lại cùng pattern |

### 1.6. Edge cases checkpoint

- **Conflict regime call** (combo 4 input cho 2 regime tương đương): xuất 2 candidate, không chọn 1, để user quyết. Format: "Call có 2 candidate khả thi: [A] vs [B]. Lý do A: ... Lý do B: ... Cần user quyết."
- **User không phản hồi:** agent dừng, không tự chuyển Stage 2. Đợi user trong session sau cũng được — pack không có timeout.

## 2. Phần 10 — Watchlist (UPGRADE tách 2 hướng + conviction + horizon + disconfirming)

**Input:** regime đã chốt + sector bias đã chốt + dữ liệu mã từ phần 6, 7, 8.

**Wording rule (giống pack cũ):**
- Dùng dạng observation/luận điểm
- KHÔNG dùng từ command (mua/bán/giảm tỷ trọng/stop loss)
- KHÔNG có level giá vào/ra/stop cụ thể
- Diễn đạt qua catalyst + flow + sự kiện sắp tới

### 2.1. Sub-section 10.1 — Bối cảnh sector bias (intro 4-6 dòng)

- 1-2 dòng nêu regime tuần + conviction + ngụ ý chiến lược (universe rộng/hẹp, mức độ thận trọng)
- **Ngành quan tâm:** list ngắn tên ngành kèm 1 câu lý do + conviction marker mỗi ngành
- **Ngành cần thận trọng:** list ngắn tên ngành kèm 1 câu lý do + conviction marker
- 1 dòng risk-reward định tính tuần tới ("rủi ro xuống lớn hơn tiềm năng tăng" / "cân bằng" / "tiềm năng tăng có ưu thế")

### 2.2. Sub-section 10.2 — Mã cơ hội tăng (3-5 mã)

**Filter universe:** mã thuộc ngành quan tâm (whitelist 18) + thanh khoản ≥ 5 tỷ ADV trung bình tuần.

**Query DB:**
- `stock_snapshot` filter `industry ∈ ngành quan tâm whitelist 18`
- Sort theo combo `money_flow_score.week_score` + `money_flow_score.market_rank_pct`
- Cross-check cơ bản qua `stock_finstats` (EPS Q YoY, biên gộp, ROE) cho mã top
- Cross-check catalyst qua `news_history_feed` filter ticker

**Mỗi mã 4 dòng** (UPGRADE từ 2 dòng cũ):

1. **Ticker (ngành) — Luận điểm 1 dòng** (cơ bản-first: catalyst / cơ bản strength / chính sách hỗ trợ, không chỉ flow/technical)
2. **Conviction:** HIGH / MID / LOW   |   **Horizon:** 1-2 tuần / 2-4 tuần
3. **Signal theo dõi** (3 chỉ báo, PREFER cơ bản/catalyst/chính sách): vd "BCTC Q1 release tuần tới, EPS consensus +18%" / "Dòng tiền tuần dương 2 tuần liên tiếp" / "Ngày chốt cổ tức 7% — DD/MM"
4. **Disconfirming signal** (1 dòng cụ thể, PREFER cơ bản): vd "BCTC Q1 EPS miss consensus ≥ 10%" / "Catalyst X delay sang tháng sau"

**Conviction logic:**
- **HIGH:** cơ bản strong + catalyst rõ ngày + định giá hợp lý + dòng tiền không ngược chiều
- **MID:** cơ bản OK + catalyst rõ nhưng chưa có ngày, hoặc dòng tiền chưa confirm
- **LOW:** signal early, cần thêm verify

### 2.3. Sub-section 10.3 — Mã cảnh báo áp lực (2-3 mã)

**Filter:** mã thuộc ngành cần thận trọng (whitelist 18) — đặc biệt mã vốn hoá lớn `market_rank_pct` cao trong ngành (bị nắm giữ rộng, áp lực bán nếu ngành xấu thêm).

**Mỗi mã 4 dòng** (tương tự 10.2 nhưng hướng tiêu cực):

1. **Ticker (ngành) — Luận điểm 1 dòng** hướng tiêu cực: "áp lực từ X → mã chịu nặng" / "cơ bản Q1 có thể yếu hơn kỳ vọng"
2. **Conviction (áp lực):** HIGH / MID / LOW   |   **Horizon:** 1-2 tuần / 2-4 tuần
3. **Signal theo dõi** (3 chỉ báo PREFER cơ bản/macro/catalyst): vd "BCTC Q1 EPS dự kiến miss" / "Dòng tiền tuần âm tiếp diễn" / "Chính sách thắt X có hiệu lực tuần tới"
4. **Disconfirming signal** (1 dòng — điều kiện nào sẽ xoá áp lực): vd "Nghị quyết nới X được ban hành" / "BCTC Q1 surprise beat consensus"

### 2.4. Bucket entry timing (OPTIONAL — default OFF)

Pack default KHÔNG render bucket entry. User có thể request "thêm bucket entry cho watchlist" để agent thêm cột Bucket (1/2/3) cho mỗi mã 10.2 + 10.3.

Định nghĩa bucket (inline pack-internal):

| Bucket | Điều kiện | Observation |
|---|---|---|
| **1 — Vào ngay được** | zone w ∈ {A, AA, AAA} VÀ zone m ∈ {A, AA, AAA} VÀ week_score ≥ 6 | Momentum đa khung đồng thuận. Sẵn sàng cho lệnh. |
| **2 — Chờ xác nhận (pullback)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w HOẶC m ∈ {B, C} | Uptrend dài hạn còn nguyên, ngắn hạn pullback. |
| **3 — Watchlist (chưa sẵn)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w VÀ m ĐỀU ∈ {C} | Pullback sâu, chờ phục hồi. |

Bucket KHÔNG nâng/giảm conviction — chỉ entry timing observation.

### 2.5. Cross-link Phần 9 risk map

Mỗi mã watchlist cross-link với risk map phần 9.4: nếu rủi ro Y materialize → mã Z bị invalidate ra sao. Cross-link inline 1 dòng nếu relevant; nếu không có rủi ro liên quan trực tiếp, bỏ qua.

## 3. Phần 11 — Lịch sự kiện tuần tới (UPGRADE thêm conviction impact)

**Input:** sự kiện corporate từ phần 8 + macro events từ web search.

**Web search bổ sung:**
- Macro release lịch tuần: FOMC minutes, CPI Mỹ, NFP, GDP, PMI Mỹ/EU/TQ/VN
- Họp chính sách tuần: FOMC, ECB, NHNN VN
- Mùa BCTC: nếu đang trong mùa, lịch công bố BCTC các mã top vốn hoá trong sector bias quan tâm (whitelist 18)

**Query DB:**
- `news_today_feed` + `news_history_feed` filter type báo cáo/thông cáo có ngày dự kiến tuần tới
- Sự kiện corporate: BCTC dự kiến công bố, ĐHCĐ, divestment, M&A, niêm yết — chỉ ngày + sự kiện + ngành/mã liên quan (whitelist 18)

**Output structured:**

### 3.1. Sub-section 11.1 — Lịch macro

Bảng UPGRADE thêm cột conviction impact:

| Ngày | Sự kiện | Ngành VN ảnh hưởng (whitelist 18) | **Conviction impact (LOW/MID/HIGH)** |

### 3.2. Sub-section 11.2 — Lịch corporate

| Ngày | Ticker | Sự kiện (BCTC/ĐHCĐ/M&A/etc.) | **Conviction impact (LOW/MID/HIGH)** |

Lịch chỉ liệt kê sự kiện đáng chú ý — bỏ qua sự kiện không impact materially. Ngành/mã thuộc whitelist 18 ưu tiên; sự kiện chạm ngành ngoài whitelist note "(ngoài scope)".

## 4. Phần 12 — Tuyên bố miễn trừ trách nhiệm

Render theo 3 trường hợp branding (lấy từ pre-flight Section 4 câu 3, xem `P_weekly_overview_01` mục 1):

**(a) User cung cấp custom disclaimer:** render đầy đủ disclaimer text user + khối liên hệ.

**(b) User cung cấp branding nhưng không có disclaimer text:** dùng template default + khối liên hệ. Có forward-looking statement language.

**(c) User chọn không cung cấp branding (pre-flight 3b):** render note plain ngắn.

Chi tiết render text 3 mode ở `O_weekly_overview_00` mục 3.12.

## 5. Phần 1 — Tóm tắt điều hành (REWRITE chuẩn Key calls / Watch / Risk)

**Viết cuối cùng**, sau khi compose xong 11 phần còn lại.

**Cấu trúc institutional weekly broadcast:**

```
## Tóm tắt điều hành

### Key calls (HIGH conviction)
- **Regime tuần tới:** [risk-on full / selective / defensive / đứng ngoài] — Conviction HIGH — [1 dòng căn cứ]
- **Sector quan tâm dẫn đầu:** [Ngành A] — Conviction HIGH — [1 dòng cơ bản+catalyst]
- **Sector quan tâm thứ 2:** [Ngành B] — Conviction HIGH — [1 dòng]
- **Mã tiêu biểu:** [Ticker X] (Ngành A) — Conviction HIGH — Horizon 1-2 tuần — [1 dòng catalyst chính]

### Watch (MID conviction — đang theo dõi)
- **Theme nổi:** [theme tuần] — [1 dòng cơ chế + signal cần theo dõi]
- **Sector chuyển pha:** [Ngành C đang chuyển từ trung tính sang quan tâm] — [1 dòng signal đang cần xác nhận]
- **Mã đang gom kín** (nếu có từ phần 7): [Ticker Y] — [1 dòng]

### Risk (chính tuần tới)
- **Rủi ro #1 từ Risk map:** [tên rủi ro] — Signal materialize: [chỉ báo cụ thể PREFER macro/fundamental] — Phản ứng: [định tính]
- **Rủi ro #2 (optional):** [tương tự]

### PM overlay note (optional — chỉ render khi có user view inject)
- [1-2 dòng tóm tắt view user inject + trạng thái xử lý — vd "PM nêu rủi ro thanh khoản TPCP DN tuần này; agent partial confirm, integrate vào Risk map item #2"]
```

**Quy tắc viết:**
- Đứng riêng đầu báo cáo — đọc 30-60 giây hiểu toàn bộ key calls
- Mỗi bullet 1 dòng, ngắn gọn, không lan man
- Conviction marker bắt buộc cho mỗi Key call
- Horizon marker bắt buộc cho mã tiêu biểu
- Risk bullet bắt buộc kèm signal materialize cụ thể (PREFER macro/fundamental, technical phụ)
- Không liệt kê quá 8 bullet tổng

## 6. Render & deliver

Sau khi có đủ 12 phần structured content + self-audit pass (theo `P_weekly_overview_04` mục 5), agent gọi `O_weekly_overview_00` để render thành file MD final.

File output: `weekly_overview_<YYYYMMDD>.md`. Trên Claude Desktop, agent xuất nội dung MD trong message để user copy/save thủ công.

Nếu user yêu cầu format khác (docx / pptx), render thêm theo `O_weekly_overview_00` mục tương ứng.

## 7. Stage 2 self-check (trước khi render)

Quick checklist phụ trợ tự audit chính ở `P_weekly_overview_04` mục 5:

- [ ] Checkpoint 1 đã có user phản hồi (confirm hoặc override)?
- [ ] Regime call có conviction HIGH/MID/LOW + disconfirming signal?
- [ ] Mỗi ngành sector bias có conviction + disconfirming?
- [ ] Phần 10.2 mỗi mã có 4 dòng (luận điểm / conviction+horizon / signal theo dõi / disconfirming)?
- [ ] Phần 10.3 mã cảnh báo cùng format 4 dòng?
- [ ] Phần 11 có cột conviction impact?
- [ ] Phần 1 dùng cấu trúc Key calls / Watch / Risk?
- [ ] Override checkpoint (nếu có) đã ghi inline note + log metadata?
- [ ] Ngành/mã trong toàn báo cáo chỉ thuộc 18 ngành whitelist?
