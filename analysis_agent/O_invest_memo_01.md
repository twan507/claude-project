# O_invest_memo_01 — Market Scan Report

Spec render báo cáo tổng hợp giai đoạn scan thị trường, gộp 4 state MD (tier 0 gate vĩ mô + tier 1 chọn ngành + tier 2 screen mã + tier 3 chấm điểm top 3) thành 1 deliverable top-down cho user review toàn cycle start.

Đây là deliverable **tổng quan thị trường đầu cycle**, không đi sâu từng mã (stock memo detail ở `O_invest_memo_02`). Mục đích: user (và/hoặc sếp/team) đọc 30 giây hiểu regime + top picks, đọc 10 phút audit logic chọn ngành/mã.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_01/02/03/04` (workflow tier 0/1/2/3).

## 1. Input state & mapping

**4 state file:**

| State file | Role | Nội dung key |
|---|---|---|
| `tier0_<YYYYMMDD>_confirmed.md` | Gate vĩ mô | Regime + bảng catalyst active + quota (số ngành × số mã) |
| `tier1_<YYYYMMDD>_confirmed.md` | Universe ngành | 3-5 ngành lọt universe (B∩C) + top ranking theo flow |
| `tier2_<YYYYMMDD>_confirmed.md` | Universe mã | 6-10 mã/ngành + bucket entry phân loại |
| `tier3_<YYYYMMDD>_confirmed.md` | Top 3 mã/ngành | Bảng chấm 6 tiêu chí + top 3/ngành + conviction tier |

**Structure output: top-down.** Theo quyết định thiết kế: executive summary đầu báo cáo (regime + top picks) cho reader bận, chi tiết các tier phía sau cho reader audit.

## 2. Structure báo cáo đầu ra

**Độ dài target:** 8-15 trang tuỳ regime (Risk-on full nhiều ngành/mã hơn Đứng ngoài).

**Flex structure** theo master mục 8 (market scan là one-off, không recurring).

Template structure:

```
# Market Scan — <Ngày>

## 1. Executive summary (30 giây đọc)
## 2. Regime & vĩ mô
## 3. Catalysts active
## 4. Ngành shortlist
## 5. Top mã final (top 3/ngành)
## 6. Watchlist (Bucket 3, chưa vào)
## 7. Risks & cảnh báo
— Metadata
```

## 3. Compose từng phần chi tiết

### Phần 1 — Executive summary (30 giây đọc)

Đây là **hook chính** của báo cáo. Reader bận chỉ đọc phần này hiểu đủ ra quyết định.

Format block cứng:

```
**Ngày scan:** [DD/MM/YYYY]  
**Regime (trạng thái thị trường):** [Risk-on full (ưa rủi ro toàn diện) / Risk-on selective (ưa rủi ro chọn lọc) / Defensive only (chỉ phòng thủ) / Đứng ngoài (giữ tiền mặt, không vào mới)]  
**Cash buffer target:** [X]% portfolio  
**Quota cycle:** [N ngành] × [M mã/ngành]

**Top [N] mã final (conviction tier, điểm / mã):**
1. [Ticker] — [Ngành] — Conviction [tier (dịch)] — [điểm]đ — Bucket [1/2/3 (dịch)] — Target [X]k
2. [Ticker] — ...
3. [Ticker] — ...
...

**Catalyst mạnh nhất cycle:** [1 câu catalyst dominant ảnh hưởng nhiều ngành]

**Rủi ro lớn nhất:** [1 câu rủi ro macro/tín hiệu cần watch]
```

Độ dài: 0.5-1 trang. Không bullet quá 7 mã trong top list — nếu cycle có >7 mã shortlist, đưa bảng đầy đủ xuống phần 5.

**Trường hợp regime = Đứng ngoài:**

Executive summary chỉ có:

```
**Regime:** Đứng ngoài (giữ tiền mặt, không vào mới)  
**Cash buffer target:** 100%

**Lý do:** [1-2 câu vì sao đứng ngoài — macro shock, regime shift, v.v.]

**Khi nào cycle kế tiếp:** [điều kiện để chạy lại tier 0 — ví dụ "Khi VN-Index xác nhận uptrend tuần + Fed dovish rõ"]
```

Các phần 3-6 (catalysts, ngành, mã, watchlist) **không render** vì không có shortlist. Báo cáo Đứng ngoài dài 2-3 trang là đủ.

### Phần 2 — Regime & vĩ mô

Structure:

```
### 2.1. Regime kết luận

[1 đoạn 3-4 câu: regime là gì, dựa vào tín hiệu nào (vĩ mô + breadth + trend đa khung + flow)]

### 2.2. Vĩ mô quốc tế

[Bảng hoặc bullet các yếu tố key: Fed, DXY (US Dollar Index — chỉ số USD), hàng hoá chính, risk sentiment. Số cụ thể.]

### 2.3. Vĩ mô nội địa

[Bullet các yếu tố key: lãi suất NHNN, tỷ giá USD/VND, CPI, tăng trưởng GDP gần nhất, thông cáo chính sách. Dẫn link finext.vn cho tin chính sách.]

### 2.4. Trend & breadth thị trường

[Zone thị trường (tuần/tháng/quý/năm) diễn giải tự nhiên — không dùng tên zone A/B/C raw. Breadth: tỷ lệ mã trên SMA20, % mã tăng, % ngành trend up.]

[Chart thị trường trend — nếu cần]
```

**Chart đề xuất phần 2:**

Chart 1 — VN-Index + breadth (combo):

````
```chart
type: combo
title: VN-Index và Breadth thị trường 60 phiên
x_axis: [danh sách ngày]
y_axis:
  - name: VN-Index
    type: line
    data: [...]
    y_label: Điểm
  - name: % mã trên SMA20
    type: line
    data: [...]
    y_label: '%'
    axis: secondary
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

### Phần 3 — Catalysts active

Structure bảng rõ:

```
### 3.1. Catalyst vĩ mô 0-3 tháng

| Catalyst | Cấp | Timing | Magnitude | Ngành ảnh hưởng |
|---|---|---|---|---|
| [Tên ngắn] | Vĩ mô quốc tế / nội địa / ngành | Q2/2026 | Cao/Trung/Thấp | [Ngành 1, Ngành 2] |
| ... | ... | ... | ... | ... |

### 3.2. Catalyst ngành

[Bullet các catalyst ảnh hưởng ngành cụ thể đã vào shortlist. Mỗi catalyst 2-3 câu mô tả + link finext nếu có bài]
```

Không giới hạn số catalyst — list hết theo tier 0 output.

### Phần 4 — Ngành shortlist

Structure:

```
### 4.1. Universe filter — 3-5 ngành lọt

| Ngành | Funnel B Fundamental | Funnel C Catalyst | Pass universe |
|---|---|---|---|
| [Ngành 1] | Đạt | Đạt (điểm [X]/6) | ✓ |
| [Ngành 2] | Đạt | Đạt (điểm [X]/6) | ✓ |
| ... | ... | ... | ... |

### 4.2. Ranking trong universe — theo flow & kỹ thuật

| Hạng | Ngành | Industry rank | Điểm dòng tiền tuần | Zone quý / năm | Ghi chú |
|---|---|---|---|---|---|
| 1 | [Ngành] | [X] | [Y] | [z mô tả tự nhiên] | [1 câu strength] |
| ... | ... | ... | ... | ... | ... |

### 4.3. Ngành lọt exception (nếu có)

[1 đoạn: ngành nào không đạt Funnel B nhưng được override do catalyst score = 6 + xu hướng 20 phiên bật từ đáy. Nêu lý do cụ thể.]

### 4.4. Ngành không qua universe

[Bullet hoặc bảng rút gọn các ngành fail + lý do chính. Đủ để user audit, không cần detail.]
```

**Trường hợp "không ngành nào qua universe":**

```
### 4.1. Universe filter — kết quả

Không có ngành nào thỏa mãn universe filter do [lý do cụ thể — ví dụ: "tăng trưởng ngành toàn thị trường âm YoY 3 quý liên tiếp, không có ngành nào có catalyst mạnh đủ mức 3/6"].

**Hành động đề xuất:** chuyển regime sang Đứng ngoài hoặc Defensive only, đợi cycle kế tiếp khi macro cải thiện.
```

Phần 5-6 skip, phần 7 Risks vẫn render.

### Phần 5 — Top mã final (top 3/ngành)

**Đây là phần quan trọng nhất cho actionable decision.** User đọc phần này biết mã nào vào memo deep-dive tier 5A/5B/5C.

Structure:

```
### 5.1. Bảng tổng top mã

| # | Ticker | Ngành | Conviction tier | Điểm 6 tiêu chí | Bucket entry | Target giá sơ bộ | Thanh khoản TB (tỷ/phiên) |
|---|---|---|---|---|---|---|---|
| 1 | [X1] | [Ngành A] | High (cao) | 16/18 | Bucket 1 (vào ngay) | [X]k-[Y]k | [Z] |
| 2 | ... | ... | ... | ... | ... | ... | ... |

### 5.2. Ghi chú từng mã

[Mỗi mã 2-3 câu ngắn — thesis 1 câu + catalyst cá thể 1 câu + entry timing 1 câu. Không phải memo đầy đủ (để ở `O_invest_memo_02`). Chỉ đủ để user quyết định mã nào deep-dive trước.]

**[Ticker 1]** — [1 câu thesis]. Catalyst cá thể: [1 câu]. Entry: [thesis bucket 1/2/3 tóm gọn].

**[Ticker 2]** — ...

### 5.3. Distribution Conviction & Bucket

[Pie hoặc bảng đếm: X mã High / Y mã Medium / Z mã Low; A mã Bucket 1 / B mã Bucket 2 / C mã Bucket 3. Giúp user hình dung shape of pipeline.]
```

**Chart đề xuất phần 5:**

Chart 2 — Distribution top mã theo conviction (pie hoặc stacked bar):

````
```chart
type: pie
title: Distribution Conviction Top 9-15 mã
x_axis: [High (cao), Medium (trung bình), Low (thấp)]
y_axis:
  - name: Số mã
    data: [X, Y, Z]
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

### Phần 6 — Watchlist (Bucket 3, chưa vào)

Structure ngắn:

```
| Ticker | Ngành | Điểm | Lý do watchlist | Condition upgrade sang Bucket 2 |
|---|---|---|---|---|
| [X] | [Ngành] | [điểm] | Zone tuần và tháng đều yếu (C) | Zone tuần bật B trở lên 2-3 phiên + week_score chuyển dương + day_score dương 3 phiên liên tiếp (mô tả tự nhiên theo K hygiene) |
```

Nếu không có mã Bucket 3 → phần này rỗng, viết 1 dòng "Không có mã watchlist trong cycle này."

### Phần 7 — Risks & cảnh báo

Structure:

```
### 7.1. Rủi ro macro
[Bullet 2-3 rủi ro vĩ mô lớn nhất trong horizon 1-3 tháng có thể làm regime shift]

### 7.2. Rủi ro ngành/mã concentration
[Nếu top picks concentrate vào 1-2 ngành → flag. Nếu shortlist có mã catalyst play rủi ro cao → flag.]

### 7.3. Rủi ro thanh khoản
[Mã nào trong shortlist có ADV (Average Daily Volume — khối lượng giao dịch trung bình) sát ngưỡng 10 tỷ/phiên → flag. User cần aware size position phải giảm theo rule 5% ADV.]

### 7.4. Rủi ro timing / pullback
[Nếu top picks Bucket 1 đang ở zone đỉnh (tuần+tháng A+ dài) → flag risk pullback sau entry. Bucket 2 đang pullback sâu → flag risk thesis fail nếu Phase 2 (tranche kế tiếp chờ confirm) không match.]
```

### Metadata cuối file

```
---

**Metadata**

- **Ngày scan:** [DD/MM/YYYY]
- **Regime confirmed:** [Risk-on full/selective/Defensive only/Đứng ngoài]
- **Cycle ID:** [Xác định theo convention user — ví dụ "Cycle 2026-Q2"]
- **State file nguồn:**
  - `tier0_<YYYYMMDD>_confirmed.md`
  - `tier1_<YYYYMMDD>_confirmed.md`
  - `tier2_<YYYYMMDD>_confirmed.md`
  - `tier3_<YYYYMMDD>_confirmed.md`
- **Next step:** render stock memo chi tiết cho top 3-5 mã High conviction qua `O_invest_memo_02`
```

## 4. Quy tắc xử lý tier thiếu data

Theo rule bạn đã quyết: **produce được thì produce, tier sau thiếu data thì note rõ, không bỏ luôn báo cáo.**

4 tình huống có thể gặp:

**Tình huống 1 — Chỉ có tier 0, regime = Đứng ngoài:**
Render phần 1 (executive summary Đứng ngoài) + phần 2 (regime & vĩ mô) + phần 7 (risks). Không có phần 3-6. Báo cáo 2-3 trang.

**Tình huống 2 — Có tier 0, tier 1 không ngành nào qua universe:**
Render phần 1 (executive summary với top list trống), phần 2, phần 3 (catalysts), phần 4 với note "Không có ngành nào thỏa mãn điều kiện do [lý do]", phần 7. Skip phần 5-6. Báo cáo 4-6 trang.

**Tình huống 3 — Có tier 0/1/2 nhưng tier 3 chưa chấm xong:**
Render đầy đủ phần 2/3/4 + phần 5 dùng tier 2 shortlist thay vì tier 3 top 3 (note rõ: "Danh sách này là tier 2 shortlist 6-10 mã/ngành, chưa qua chấm điểm tier 3. Sẽ refine sau checkpoint CP4."). Metadata ghi rõ tier 3 pending.

**Tình huống 4 — Tier 0/1/2/3 đầy đủ (chuẩn):**
Render đầy đủ 7 phần theo structure chính.

**Không render khi:** thiếu tier 0 (không có regime thì không có base — yêu cầu user hoàn thành CP1 trước).

## 5. Compose workflow step-by-step

Khi user yêu cầu render market scan:

**Bước 1 — Xác định format + template** (theo master workflow Bước 2-3): hỏi MD/docx/pptx + template nếu docx/pptx.

**Bước 2 — Load state file:**
- Tìm `tier0_*_confirmed.md` mới nhất (theo YYYYMMDD)
- Kiểm tra tier 1/2/3 có hay không, version mới nhất
- Xác định tình huống (1/2/3/4 theo mục 4)

**Bước 3 — Đọc regime từ tier 0.** Nếu "Đứng ngoài" → skip phần chi tiết, render báo cáo ngắn.

**Bước 4 — Compose executive summary phần 1** trước, làm hook. Data: regime + top list từ tier 3 (hoặc tier 2 fallback) + catalyst dominant + rủi ro lớn nhất.

**Bước 5 — Compose phần 2-6** theo thứ tự, merge data từ 4 tier state file.

**Bước 6 — Chèn chart annotation:**
- Phần 2: chart VN-Index + breadth
- Phần 5: chart distribution top mã (pie)
- Optional: chart ngành ranking nếu muốn visual hơn

**Bước 7 — Compose phần 7 Risks & cảnh báo.** Cross-check với shortlist để phát hiện concentration, thanh khoản, timing risk.

**Bước 8 — Self-check (như stock memo):** K hygiene, dịch thuật ngữ, citation 4 nhóm, unit format.

**Bước 9 — Render format cuối** (MD / docx / pptx với template user chỉ định).

**Bước 10 — Present file.**

## 6. Guide render docx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

**Layout:**
- Cover (trang 1): Title "Market Scan — [ngày]" + logo + regime confirm
- Executive summary (trang 2) — phần 1 đầy đủ, bảng top picks. Đây là trang user đọc lâu nhất, layout rộng, font dễ đọc
- Body (trang 3+): phần 2-7 theo thứ tự, mỗi phần chính bắt đầu page mới
- Chart inline 80% page width
- Bảng top mã phần 5 có thể cần landscape page nếu nhiều cột — tuỳ template
- Metadata trang cuối

**Template cần có:**
- Heading styles H1/H2/H3
- Body + table + caption style
- Footer: page number + "Market Scan [ngày]"
- Header optional: logo

## 7. Guide render pptx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

**Slide layout đề xuất (12-18 slide tuỳ regime):**

| Slide | Nội dung |
|---|---|
| 1 | Cover — ngày scan + regime badge lớn |
| 2 | Executive summary — regime + top [N] picks bullet + catalyst mạnh nhất |
| 3 | Regime & vĩ mô quốc tế |
| 4 | Vĩ mô nội địa |
| 5 | Trend & breadth thị trường + chart VN-Index/breadth |
| 6-7 | Catalysts active (bảng) |
| 8 | Ngành shortlist — bảng universe filter |
| 9 | Ngành ranking — bảng + flow chart |
| 10-13 | Top mã: mỗi 2-3 mã 1 slide với thesis 1 câu + catalyst + bucket |
| 14 | Distribution top mã (pie chart) |
| 15 | Watchlist |
| 16-17 | Risks & cảnh báo |
| 18 | Next step — stock memo deep-dive cho top [N] mã High conviction |

Regime Đứng ngoài: 4-6 slide (cover + exec summary + vĩ mô + risks + next step).

## 8. Ví dụ executive summary (fragment, demo)

```markdown
# Market Scan — 22/04/2026

## 1. Executive summary

**Ngày scan:** 22/04/2026  
**Regime (trạng thái thị trường):** Risk-on selective (ưa rủi ro chọn lọc)  
**Cash buffer target:** 30-40% portfolio  
**Quota cycle:** 3 ngành × 3 mã/ngành

**Top 9 mã final (conviction tier, điểm / mã):**

1. VNM — Tiêu dùng thiết yếu — Conviction High (cao) — 16đ — Bucket 1 (vào ngay) — Target 85k
2. HPG — Vật liệu — High (cao) — 15đ — Bucket 2 (chờ pullback xác nhận) — Target 32k
3. VCB — Tài chính ngân hàng — High (cao) — 15đ — Bucket 2 — Target 105k
4. ...

**Catalyst mạnh nhất cycle:** Fed cut rate dự kiến tháng 6/2026 — dovish outlook thuận dòng vốn sang EM (emerging markets / thị trường mới nổi) và giảm áp lực tỷ giá USD/VND.

**Rủi ro lớn nhất:** Địa chính trị Trung Đông leo thang có thể đẩy dầu quay lại $90+ → áp lực CPI VN và chi phí logistic, ảnh hưởng mạnh mã retail và xuất nhập khẩu.

...
```

## 9. Checklist self-check cuối cùng

- [ ] Tier 0 đã có và confirmed (tối thiểu mandatory)
- [ ] Tình huống xử lý tier thiếu (nếu có) đã apply đúng theo mục 4
- [ ] Executive summary có đủ: regime + cash buffer + quota + top picks + catalyst dominant + rủi ro lớn nhất
- [ ] Thuật ngữ Regime/Bucket/Conviction tier lần đầu có ngoặc dịch
- [ ] Regime không dùng tên technical, dùng tên + dịch
- [ ] Zone/trend/dòng tiền mô tả tự nhiên, không có `w_trend`, `zone: A`, `Kịch bản X`
- [ ] Chart annotation YAML đầy đủ source + render flags + data array đúng format
- [ ] Catalyst dẫn link finext.vn đầy đủ cho tin thuộc agent_db
- [ ] Data định lượng từ agent_db có "(nguồn: Tổng hợp)"
- [ ] Metadata cuối file đầy đủ
- [ ] Format cuối đúng yêu cầu + template đúng
