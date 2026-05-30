# agent_db_01 — Collections Schema

Tài liệu mô tả schema đầy đủ của 25 collection trong `agent_db`. Mỗi mục có: cấu trúc doc, giải nghĩa field, cách agent nên dùng, và cảnh báo nếu có.

Chi tiết về cách **diễn giải** chỉ báo (ngưỡng percentile, kịch bản, PTCB theo 4 type, pitfalls) xem ở `agent_db_04`. File này tập trung vào schema và công thức gốc.

---

## Công thức các chỉ báo cốt lõi

Hiểu cách chỉ báo được tính giúp diễn giải đúng. Chi tiết về ngưỡng và cách dùng nằm ở `agent_db_04`.

### money_flow_score.day_score

```
day_score = ((C - L) - (H - C)) / TC × 1000

Trong đó:
- TC = prev_close (giá tham chiếu)
- H = max(high, TC)  — penalize gap down
- L = min(low, TC)   — reward gap up
- C = close
```

Đo vị trí close trong range ngày (có adjust gap). Unbounded. Dương = bên mua thắng, âm = bên bán thắng. Khác biệt với `pct_change`: một phiên có `pct_change` dương nhưng close gần low vẫn có thể có `day_score` âm.

### money_flow_score.week_score

Rolling cumulative sum 5 phiên gần nhất của `day_score`.

Lưu ý: 1 phiên có `day_score` rất dương có thể kéo cả tuần lên cao dù 4 phiên khác đi ngang — luôn verify độ đồng đều bằng `stock_recent`.

### price.volume_strength_index (VSI)

```
VSI = volume phiên hiện tại / SMA5 volume
```

VSI = 1.0 tương đương khối lượng hiện tại ngang trung bình 5 phiên gần nhất. Distribution thực tế exponential skewed (median ~0.75, max có thể > 50), **không phải normal quanh 1.0**.

### money_flow_score.industry_rank_pct / market_rank_pct

```
rank_pct = 1 - rank/total
```

Rank dựa trên `week_score` giảm dần, có filter thanh khoản tối thiểu.

- `rank_pct = 0.9` → mã xếp top 10% (vượt 90% còn lại)
- `rank_pct = 0` → không xếp hạng (thanh khoản quá thấp) hoặc xếp cuối — bỏ qua khi screening

### Xếp hạng ngành — KHÔNG có rank tĩnh trong DB, phải tự tổng hợp

**DB không lưu xếp hạng ngành-so-ngành tĩnh.** Khi cần biết "ngành nào dòng tiền mạnh nhất" hoặc rank theo bất kỳ tiêu chí ngành nào, agent phải **tự tổng hợp theo scope phân tích hiện tại** (default: 18 ngành whitelist; override: tuỳ user yêu cầu):

1. Query `industry_snapshot.money_flow_score.week_score` (= **dòng tiền tuần** — đây là field chuẩn để rank ngành) cho toàn bộ ngành trong scope
2. Self-sort giảm dần (rank cao = dòng tiền tuần mạnh nhất)
3. Re-rank trong-flight (vd 1..18 cho whitelist, 1..N cho danh sách custom)

**Field rank ngành mặc định:** `money_flow_score.week_score` (dòng tiền tuần). Các tiêu chí khác (biến động giá `change.w_pct`, breadth, BCTC growth từ `industry_finstats`) chỉ rank khi user yêu cầu rõ.

**Lý do design:** rank ngành phụ thuộc scope analysis (whitelist 18 vs override). Nếu DB lưu rank tĩnh 1..24, mọi báo cáo phải re-compute lại để khớp scope — rủi ro sai. Tự tổng hợp đảm bảo rank luôn align với danh sách theo dõi user.

Reference field rank stock-level vẫn còn (`industry_rank_pct`, `market_rank_pct` ở `stock_snapshot`) — đây là percentile của mã trong ngành / thị trường, khác với rank ngành-vs-ngành đã bị bỏ.

### trend (ở cấp ngành, nhóm, thị trường)

```
trend line = (đỉnh chu kỳ + đáy chu kỳ) / 2

trend value = tỷ lệ cổ phiếu có giá hiện tại > trend line
```

Khung chu kỳ: w = 5 phiên, m = 20, q = 60, y = 240.

- `w_trend = 0.35` → 35% mã đang có giá trên trend line tuần (midpoint của range tuần)
- Bounded [0, 1]
- Ý nghĩa: breadth indicator mean-reversion — ngưỡng > 0.8 quá mua, < 0.2 quá bán

Tính ở 3 cấp: thị trường (`market_snapshot.trend`), ngành (`industry_snapshot.trend`), nhóm (`group_snapshot.trend`). **Không có** ở cấp mã — mã dùng `technical_zone` thay thế.

### technical_zone.overall (rule tổng hợp)

Overall có 5 bậc: AAA > AA > A > B > C. Được tính từ 3 sub-zone (`ma_zone`, `fibonacci_zone`, `volume_profile_zone`) — mỗi sub-zone chỉ có 3 giá trị A/B/C.

Rule:

| Số sub-zone = A | Còn lại | Overall |
|---|---|---|
| 3 | — | AAA |
| 2 | 1 B/C bất kỳ | AA |
| 1 | 2 B/C bất kỳ | A |
| 0 | ≥ 1 B | B |
| 0 | 3 C | C |

### technical_indicator (công thức chuẩn)

- **ma**: SMA đơn thuần của giá đóng cửa — ma5/ma20/ma60/ma120/ma240
- **fibonacci**: 3 mức 38.2% / 50% / 61.8% retracement của range `prev_high - prev_low` trong khung
- **volume_profile**: POC/VAL/VAH tính theo Market Profile chuẩn trong window của khung
- **pivot**: Classical Pivot — `pivot = (H+L+C)/3, r1 = 2×pivot - L, s1 = 2×pivot - H`
- **ohl**: `{open, prev_high, prev_low}` — giá mở khung, đỉnh/đáy khung trước

### Các khung thời gian w/m/q/y (toàn DB)

| Khung | Số phiên | Ý nghĩa |
|---|---|---|
| w | 5 | Tuần |
| m | 20 | Tháng |
| q | 60 | Quý |
| y | 240 | Năm |

Dùng nhất quán cho `technical_indicator`, `technical_zone`, `trend`, `change` (`w_pct`, `m_pct`, `q_pct`, `y_pct`).

---

## A. Khối cổ phiếu

### `stock_info` — Thông tin tĩnh cổ phiếu

**Số lượng:** 1 doc / ticker (~674 doc)
**Cập nhật:** EOD (hoặc khi có thay đổi niêm yết/phân loại)
**Dùng khi:** giới thiệu công ty, tra cứu ngành/vốn hoá/lĩnh vực kinh doanh, cấu trúc sở hữu.

```json
{
  "ticker": "VNM",
  "ticker_name": "CTCP Sữa Việt Nam",
  "exchange": "HSX",
  "overview": "Mô tả doanh nghiệp, năng lực cạnh tranh, thị phần...",
  "business_area": "Sản xuất và kinh doanh sữa, sản phẩm từ sữa...",
  "industry": "Thực phẩm Đồ uống",
  "marketcap": "LargeCaps",
  "category": "Dòng tiền Ổn định",
  "outstandingShare": 2089645967,
  "freeFloatRate": 0.45,
  "statePercentage": 0.36,
  "foreignerPercentage": 0.55,
  "foreignerRoom": 104823000,
  "maximumForeignPercentage": 1.0,
  "majorHoldings": 0.41
}
```

**Field quan trọng:**
- `overview`, `business_area`: prose tiếng Việt, dài — chỉ lấy khi thực sự cần giới thiệu công ty, tránh kéo khi chỉ cần tra ngành.
- `industry`, `marketcap`, `category`: đã resolve sang tên hiển thị, không phải code. Dùng trực tiếp để filter các collection khác.
- `foreignerRoom`: room còn lại nước ngoài được phép mua (số cổ phiếu).
- `majorHoldings`: tỷ lệ sở hữu của cổ đông lớn.

---

### `stock_snapshot` — Trạng thái phiên mới nhất

**Số lượng:** 1 doc / ticker
**Cập nhật:** realtime trong phiên + EOD
**Dùng khi:** cần trạng thái "hôm nay" của 1 mã — giá, điểm dòng tiền, các vùng kỹ thuật.

```json
{
  "ticker": "VNM",
  "snapshot_date": "2026-04-17",
  "price": {
    "open": 61.3, "high": 62.7, "low": 61.3, "close": 62.0,
    "volume": 1591100, "trading_value": 98.58,
    "diff": 0.9, "pct_change": 0.0147,
    "volume_strength_index": 1.21
  },
  "money_flow_score": {
    "day_score": 3.3, "week_score": -8.04,
    "industry_rank_pct": 0.6, "market_rank_pct": 0.55
  },
  "change": {
    "w_pct": -0.0127, "m_pct": 0.0333,
    "q_pct": -0.1092, "y_pct": 0.1811
  },
  "technical_indicator": {
    "ohl":       { "w": {...}, "m": {...}, "q": {...}, "y": {...} },
    "ma":        { "ma5": 62, "ma20": 61.4, "ma60": 65.2, "ma120": 63.4, "ma240": 60.0 },
    "fibonacci": { "w": {...}, "m": {...}, "q": {...}, "y": {...} },
    "volume_profile": { "w": {...}, "m": {...}, "q": {...}, "y": {...} },
    "pivot":     { "w": {...}, "m": {...}, "q": {...}, "y": {...} }
  },
  "technical_zone": {
    "overall": { "w": "AAA", "m": "AA", "q": "A", "y": "AAA" },
    "ma_zone": { "w": "A", "m": "A", "q": "A", "y": "A" },
    "fibonacci_zone": { "w": "A", "m": "B", "q": "B", "y": "A" },
    "volume_profile_zone": { "w": "A", "m": "A", "q": "B", "y": "A" }
  }
}
```

**Chi tiết nested (mỗi khung `w/m/q/y`):**

- `technical_indicator.ohl.{w|m|q|y}`: `{open, prev_high, prev_low}` — giá mở khung, đỉnh khung trước, đáy khung trước.
- `technical_indicator.fibonacci.{w|m|q|y}`: `{f382, f500, f618}` — mốc Fibonacci hồi (38.2/50/61.8%).
- `technical_indicator.volume_profile.{w|m|q|y}`: `{poc, val, vah}` — giá tập trung nhất, biên dưới, biên trên vùng giá chấp nhận.
- `technical_indicator.pivot.{w|m|q|y}`: `{pivot, r1, s1}` — cân bằng, kháng cự 1, hỗ trợ 1.

**Không có** các field `stocks` (danh sách mã) hay `breadth` trong `stock_snapshot`. Breadth chỉ có ở cấp ngành/nhóm/thị trường.

---

### `stock_recent` — Chuỗi 20 phiên gần nhất

**Số lượng:** 1 doc / ticker
**Cập nhật:** realtime + EOD
**Dùng khi:** vẽ chart giá ngắn hạn, tính biến động, so sánh day_score giữa các phiên.

```json
{
  "ticker": "VNM",
  "series": [
    {
      "date": "2026-04-17",
      "price": { "open": ..., "high": ..., "low": ..., "close": ..., "volume": ...,
                 "trading_value": ..., "volume_strength_index": ..., "diff": ..., "pct_change": ... },
      "money_flow_score": { "day_score": ..., "week_score": ...,
                            "industry_rank_pct": ..., "market_rank_pct": ... }
    },
    { "date": "2026-04-16", ... },
    ...
  ]
}
```

**Thứ tự:** `series[0]` là phiên mới nhất, `series[19]` là phiên xa nhất (đã sort giảm dần theo ngày).

**Lưu ý:** chỉ cần `$slice: [0, N]` khi lấy N phiên gần nhất, không sort lại.

---

### `stock_finstats` — BCTC + Định giá + Các tỷ số

**Số lượng:** 1 doc / ticker (~665 doc — một số ticker thiếu BCTC)
**Cập nhật:** EOD (giá trị `valuation_ratios` thay đổi theo giá; BCTC chỉ thay đổi khi có công bố quý/năm mới)
**Dùng khi:** định giá doanh nghiệp, so sánh BCTC theo kỳ, so sánh ngành.

```json
{
  "ticker": "VNM",
  "ticker_name": "CTCP Sữa Việt Nam",
  "industry": "Thực phẩm Đồ uống",
  "type": "SXKD",

  "valuation_ratios": [
    { "vi_name": "Vốn hóa thị trường", "en_name": "Market Capitalization", "value": 129577.24 },
    { "vi_name": "P/E", "en_name": "Price-to-Earnings (P/E)", "value": 15.2 },
    { "vi_name": "P/B", "en_name": "Price-to-Book (P/B)", "value": 4.1 },
    { "vi_name": "P/S", "en_name": "Price-to-Sales (P/S)", "value": 2.3 },
    { "vi_name": "EPS", "en_name": "Earnings per Share (EPS)", "value": 4080 },
    { "vi_name": "BVPS", "en_name": "Book Value per Share (BVPS)", "value": 15120 },
    { "vi_name": "PEG", "en_name": "Price/Earnings to Growth (PEG)", "value": ... },
    { "vi_name": "P/CF", "en_name": "Price-to-Cash-Flow (P/CF)", "value": ... },
    { "vi_name": "EV/EBITDA", "en_name": "EV/EBITDA", "value": ... }
  ],

  "financial_statements": {
    "quarterly": [
      {
        "period": "2023_4",
        "data": [
          { "vi_name": "ROE", "en_name": "Return on Equity (ROE)", "value": 0.216 },
          { "vi_name": "ROA", "en_name": "Return on Assets (ROA)", "value": 0.142 },
          { "vi_name": "Biên lợi nhuận gộp", "en_name": "Gross Margin", "value": 0.179 },
          { "vi_name": "Doanh thu thuần", "en_name": "Net Revenue", "value": 9864419377152 },
          ...
        ]
      },
      { "period": "2024_1", "data": [...] },
      ...
      { "period": "2025_4", "data": [...] }
    ],
    "yearly": [
      { "period": "2020_5", "data": [...] },
      ...
      { "period": "2025_5", "data": [...] }
    ]
  }
}
```

**Giải mã `period`:**
- `2025_4` = Q4 năm 2025 (quarterly).
- `2025_5` = dữ liệu năm 2025 (yearly — suffix `_5` là convention).

**Giải mã `type`:**
- `SXKD`: doanh nghiệp sản xuất kinh doanh thông thường — có đầy đủ biên lợi nhuận gộp/EBIT/ròng, vòng quay tài sản...
- `NGANHANG`: ngân hàng — có các chỉ tiêu đặc thù (CASA, NIM, nợ nhóm 1-5...).
- `CHUNGKHOAN`: công ty chứng khoán.
- `BAOHIEM`: bảo hiểm.

Mỗi `type` có bộ field khác nhau nên khi so sánh cross-type phải cẩn thận.

**Cảnh báo:**
- Đơn vị `value` cho BCTC: **đồng Việt Nam** (không phải nghìn đồng hay triệu đồng). Agent cần chia cho 10^9 để có đơn vị tỷ đồng khi trình bày.
- Đơn vị `value` cho `Vốn hóa thị trường` trong `valuation_ratios`: **tỷ đồng**.
- Các tỷ lệ (ROE, ROA, biên, tăng trưởng): **dạng thập phân** (0.216 nghĩa là 21.6%), cần nhân 100 khi trình bày.
- Một số `value` có thể là `NaN` — khi gặp, coi như không có data.

---

### `stock_nntd` — Khối ngoại & Tự doanh

**Số lượng:** 1 doc / ticker
**Cập nhật:** realtime + EOD
**Dùng khi:** phân tích dòng tiền nước ngoài / tự doanh trên 1 mã.

```json
{
  "ticker": "VNM",
  "nn": {
    "latest": { "date": "2026-04-17", "buy_value": 12.5, "sell_value": -8.3, "net_value": 4.2 },
    "week":   { "buy_value": 45.2, "sell_value": -38.1, "net_value": 7.1 },
    "month":  { "buy_value": 180.5, "sell_value": -165.3, "net_value": 15.2 }
  },
  "td": {
    "latest": { "date": "2026-04-17", "buy_value": 2.1, "sell_value": -1.8, "net_value": 0.3 },
    "week":   { ... },
    "month":  { ... }
  }
}
```

**Đơn vị:** tỷ đồng.
**Dấu:** `sell_value` luôn âm. `net_value = buy_value + sell_value`.
**week/month:** tổng cộng dồn 5 phiên / 20 phiên gần nhất (rolling).

---

### `stock_itd` — Intraday cổ phiếu

**Số lượng:** 1 doc / ticker
**Cập nhật:** 5 phút / lần trong phiên
**Dùng khi:** phân tích biến động trong phiên, xác định giờ nào có volume spike.

```json
{
  "ticker": "VNM",
  "series": [
    {
      "datetime": ISODate("2026-04-17T15:00:00Z"),
      "open": 62.0, "high": 62.7, "low": 61.1, "close": 62.0,
      "volume": 32200, "trading_value": 2.0,
      "diff": 0.9, "pct_change": 0.0147, "vsi": 1.21
    },
    ...
  ]
}
```

**Thứ tự:** `series[0]` là snapshot gần nhất (đã sort giảm dần).
**Lưu ý:** số lượng element trong `series` có thể rất lớn (tầm 50-80 điểm/ngày). Khi trả về agent, nên `$slice` lấy 10-20 điểm đầu hoặc filter theo datetime.

---

## B. Khối ngành

> ### ⚠️ WHITELIST 18 NGÀNH PHÂN TÍCH — DEFAULT SCOPE, USER OVERRIDE ĐƯỢC PHÉP
>
> DB lưu **24 ngành** (số doc của `industry_info`, `industry_snapshot`, `industry_recent`, `industry_finstats`, `history_industry`). **Mặc định khi user không nói gì cụ thể, phân tích chỉ dùng 18 ngành trong whitelist dưới đây** để so sánh, xếp hạng, aggregate, và build báo cáo. Các ngành ngoài whitelist (vd "Bảo hiểm", "Y tế Dược phẩm"…) **không xuất hiện** trong báo cáo / nhận định / xếp hạng / aggregate tổng quát.
>
> **NHƯNG khi user yêu cầu cụ thể** ngành ngoài whitelist (vd "phân tích ngành Bảo hiểm", "so sánh BVH vs VNM", "dữ liệu BCTC ngành Y tế Dược phẩm Q1"), agent **vẫn query được và trả lời bình thường** — dữ liệu 24 ngành đầy đủ vẫn có trong DB. Khi đó:
> - Query thẳng `industry_*` collection theo `industry_name` cụ thể user yêu cầu (không apply $in whitelist filter)
> - Ghi note rõ trong output: "Ngành [X] nằm ngoài scope whitelist mặc định — trả lời theo yêu cầu cụ thể của anh/chị, không tự động đưa vào báo cáo tổng quát"
> - KHÔNG tự ý so sánh với các ngành khác trong whitelist trừ khi user yêu cầu rõ
> - Mã thuộc ngành ngoài whitelist vẫn phân tích đơn lẻ được nếu user hỏi đích danh ticker
>
> | Mã ngắn (user nhập) | Tên chuẩn trong DB (`industry_name` ở khối ngành / `industry` ở `stock_info`) |
> |---|---|
> | BANLE | Bán lẻ Tiêu dùng |
> | BDS | Bất động sản Dân dụng |
> | CHUNGKHOAN | Công ty Chứng khoán |
> | CONGNGHE | Công nghệ Viễn thông |
> | CONGNGHIEP | Thiết bị Công nghiệp |
> | DAUKHI | Dịch vụ Dầu khí |
> | DETMAY | Dệt may Xuất khẩu |
> | HOACHAT | Hóa chất Phân bón |
> | KCN | Bất động sản Khu công nghiệp |
> | KHOANGSAN | Tài nguyên cơ bản |
> | KIMLOAI | Kim loại công nghiệp |
> | NGANHANG | Tài chính ngân hàng |
> | NONGNGHIEP | Nông nghiệp Chăn nuôi |
> | THUCPHAM | Thực phẩm Đồ uống |
> | THUYSAN | Chế biến Thủy sản |
> | TIENICH | Hạ tầng Tiện ích |
> | VANTAI | Vận tải Kho bãi |
> | XAYDUNG | Thi công Xây dựng |
>
> **Cách áp dụng khi query — DEFAULT mode (user không nói gì cụ thể):**
> - Mọi `find` / `aggregate` trên `industry_info`, `industry_snapshot`, `industry_recent`, `industry_finstats`, `history_industry` phải kèm `$match: { industry_name: { $in: [<18 tên chuẩn>] } }`.
> - Khi screen mã từ `stock_snapshot` / `stock_info` theo ngành (cross-section watchlist, sector tilts), filter `industry` ∈ 18 tên chuẩn — mã thuộc ngành ngoài whitelist bị loại khỏi xếp hạng theme/sector (vẫn được phân tích đơn lẻ nếu user hỏi đích danh ticker).
>
> **Cách áp dụng khi query — OVERRIDE mode (user yêu cầu ngành cụ thể ngoài whitelist):**
> - Query thẳng theo `industry_name` user yêu cầu, KHÔNG apply `$in` whitelist filter
> - Ghi note "ngoài scope whitelist mặc định" trong output để user biết
> - Vẫn cung cấp đầy đủ data như ngành trong whitelist
>
> **Ảnh hưởng tới aggregate cấp thị trường:** mean/median `money_flow_score`, breadth ngành dùng proxy thị trường (xem `agent_db_04`) **phải tính trên 18 ngành whitelist**, không phải 24.
>
> **Xếp hạng ngành tự tổng hợp:** DB **không lưu** `industry_rank` ngành-vs-ngành — khi báo cáo cần rank (vd "ngành nào dòng tiền mạnh nhất"), agent tự query `week_score` cho 18 ngành whitelist (default mode) hoặc danh sách user yêu cầu (override mode), sort, re-rank 1..N. Chi tiết ở mục "Xếp hạng ngành" đầu file.
>
> **Khi user nhập mã ngắn** (vd "phân tích ngành DAUKHI", "so sánh NGANHANG vs CHUNGKHOAN"): map sang tên chuẩn DB ở cột phải để query. Khi xuất báo cáo cho user dùng tên đầy đủ (vd "Dịch vụ Dầu khí"), không lộ mã ngắn.

### `industry_info` — Tổng quan ngành

**Số lượng:** 24 doc
**Cập nhật:** EOD (gần như static)
**Dùng khi:** giới thiệu ngành, hiểu chuỗi giá trị, động lực, rủi ro.

```json
{
  "industry_name": "Bán lẻ Tiêu dùng",
  "overview": "Ngành bán lẻ trong danh mục bao gồm...",
  "sub_segments": "Bán lẻ điện máy, ICT và đa ngành: MWG, FRT...",
  "value_chain": "Bán lẻ điện máy và ICT (MWG, FRT, DGW): Nhập hàng từ hãng...",
  "drivers": "Thu nhập tiêu dùng và niềm tin người tiêu dùng...",
  "risks": "Rủi ro chu kỳ tiêu dùng: rất nhạy...",
  "typical_tickers": "Đầu ngành bán lẻ, đa ngành: MWG (lớn nhất)...",
  "full_ticker_list": ["PNJ", "TNA", "PIT", "MWG", "FRT", "DGW", ...]
}
```

**Các field prose** (overview, sub_segments, value_chain, drivers, risks, typical_tickers) đều là text dài. Khi agent chỉ cần danh sách mã → projection `{full_ticker_list: 1, _id: 0}`.

---

### `industry_snapshot` — Chỉ số ngành phiên mới nhất

**Số lượng:** 24 doc
**Cập nhật:** realtime + EOD
**Dùng khi:** so sánh sức mạnh các ngành, xếp hạng dòng tiền ngành.

```json
{
  "industry_name": "Bán lẻ Tiêu dùng",
  "snapshot_date": "2026-04-17",
  "price": { ... },
  "money_flow_score": {
    "day_score": -1.88,
    "week_score": 11.74
  },
  "breadth": { "breadth_in": 6, "breadth_out": 7, "breadth_neu": 2 },
  "change": { "w_pct": 0.022, "m_pct": 0.039, "q_pct": 0.027, "y_pct": 0.284 },
  "technical_zone": {
    "overall": { "w": "AAA", "m": "AAA", "q": "AA", "y": "AAA" },
    "ma_zone": { ... }, "fibonacci_zone": { ... }, "volume_profile_zone": { ... }
  },
  "trend": { "w_trend": 0.53, "m_trend": 0.73, "q_trend": 0.27, "y_trend": 0.60 }
}
```

**Khác biệt so với stock_snapshot:**
- Có thêm `breadth` (độ rộng trong nội bộ ngành).
- Có thêm `trend` (tỷ lệ mã trong ngành có giá trên trend line = midpoint range của khung; xem công thức ở đầu file).
- `money_flow_score` **KHÔNG có** `industry_rank` (DB không lưu rank ngành-vs-ngành — phải tự tổng hợp theo scope phân tích, xem mục "Xếp hạng ngành" ở đầu file).
- **Không có** `technical_indicator` (ohl/ma/fibonacci/volume_profile/pivot).

---

### `industry_recent` — Chuỗi ngành 20 phiên

**Số lượng:** 24 doc
**Cập nhật:** realtime + EOD

```json
{
  "industry_name": "Bán lẻ Tiêu dùng",
  "series": [
    {
      "date": "2026-04-17",
      "price": { ... },
      "money_flow_score": { "day_score": ..., "week_score": ... },
      "trend": { "w_trend": ..., "m_trend": ..., "q_trend": ..., "y_trend": ... }
    },
    ...
  ]
}
```

**Thứ tự:** mới nhất ở đầu. Đã sort sẵn.

---

### `industry_finstats` — BCTC ngành

**Số lượng:** 24 doc
**Cập nhật:** EOD (số định giá đổi theo giá; BCTC đổi khi công bố mới)
**Dùng khi:** benchmark định giá/hiệu quả của 1 mã so với ngành.

Schema tương đồng `stock_finstats` nhưng theo ngành:

```json
{
  "industry_name": "Bán lẻ Tiêu dùng",
  "type": "SXKD",
  "valuation_ratios": [...],
  "financial_statements": {
    "quarterly": [...],
    "yearly": [...]
  }
}
```

**Lưu ý:** ngành `Tài chính ngân hàng` sẽ có `type: NGANHANG` → bộ chỉ tiêu khác các ngành SXKD. So sánh ngân hàng với ngành khác phải dùng các chỉ tiêu chung (ROE, tổng tài sản, tăng trưởng), tránh các chỉ tiêu đặc thù.

---

## C. Khối nhóm

### `group_snapshot` — Chỉ số nhóm phiên mới nhất

**Số lượng:** 6 doc
**Cập nhật:** realtime + EOD

**Giá trị `group_name` khả dĩ:**
- Nhóm vốn hoá: `LargeCaps`, `MidCaps`, `SmallCaps`
- Nhóm dòng tiền: `Dòng tiền Vượt trội`, `Dòng tiền Ổn định`, `Dòng tiền Sự kiện`

```json
{
  "group_name": "LargeCaps",
  "snapshot_date": "2026-04-17",
  "price": { ... },
  "money_flow_score": { "day_score": -6.2, "week_score": -18.1 },
  "breadth": { "breadth_in": 14, "breadth_out": 27, "breadth_neu": 2 },
  "change": { "w_pct": 0.010, "m_pct": 0.045, "q_pct": -0.018, "y_pct": 0.322 },
  "technical_zone": { "overall": {...}, "ma_zone": {...}, "fibonacci_zone": {...}, "volume_profile_zone": {...} },
  "trend": { "w_trend": 0.37, "m_trend": 0.77, "q_trend": 0.37, "y_trend": 0.53 }
}
```

**Lưu ý:** không có field `group_type` để phân biệt "vốn hoá" vs "dòng tiền". Agent tự nhận diện qua tên:
- Bắt đầu "Dòng tiền " → nhóm dòng tiền.
- Kết thúc "Caps" → nhóm vốn hoá.

---

### `group_recent` — Chuỗi nhóm 20 phiên

**Số lượng:** 6 doc
**Cập nhật:** realtime + EOD

```json
{
  "group_name": "LargeCaps",
  "group_type": "Nhóm vốn hoá",    // collection này CÓ group_type, snapshot thì không
  "series": [
    {
      "date": "2026-04-17",
      "price": { ... },
      "money_flow_score": { "day_score": ..., "week_score": ... },
      "trend": { ... }
    },
    ...
  ]
}
```

Giá trị `group_type`: `"Nhóm vốn hoá"` hoặc `"Nhóm dòng tiền"`.

---

## D. Khối thị trường

**Lưu ý chung:** tất cả collection khối này chỉ đại diện cho VNINDEX. HNX và UPCOM không được track trong DB này.

### `market_snapshot` — VNINDEX phiên mới nhất

**Số lượng:** 1 doc
**Cập nhật:** realtime + EOD

```json
{
  "index": "VNINDEX",
  "snapshot_date": "2026-04-17",
  "price": { ... },
  "breadth": { "breadth_in": 127, "breadth_out": 171, "breadth_neu": 33 },
  "change": { "w_pct": 0.038, "m_pct": 0.103, "q_pct": -0.033, "y_pct": 0.465 },
  "technical_indicator": {
    "ohl": {...}, "ma": {...}, "fibonacci": {...}, "volume_profile": {...}, "pivot": {...}
  },
  "technical_zone": {
    "overall": {...}, "ma_zone": {...}, "fibonacci_zone": {...}, "volume_profile_zone": {...}
  },
  "trend": { "w_trend": 0.35, "m_trend": 0.68, "q_trend": 0.28, "y_trend": 0.32 }
}
```

**Lưu ý kỹ thuật:**
- `breadth` được tính từ rổ FNXINDEX (rổ lọc nội bộ), **không** phải từ toàn sàn HOSE. Con số phản ánh độ rộng cổ phiếu "chất lượng" trong rổ.
- `trend` cũng tính trên rổ FNXINDEX (tỷ lệ mã trong rổ có giá trên trend line khung tương ứng).
- Tên field là `index` (không phải `ticker`).

---

### `market_recent` — VNINDEX chuỗi 20 phiên

**Số lượng:** 1 doc
**Cập nhật:** realtime + EOD

```json
{
  "ticker": "VNINDEX",
  "recent_price": [
    {
      "date": "2026-04-17",
      "price": { "open": ..., "high": ..., "low": ..., "close": ..., "volume": ...,
                 "trading_value": ..., "volume_strength_index": ..., "diff": ..., "pct_change": ... }
    },
    ...
  ],
  "recent_trend": [
    {
      "date": "2026-04-17",
      "market_trend": { "w_trend": ..., "m_trend": ..., "q_trend": ..., "y_trend": ... }
    },
    ...
  ]
}
```

**Lưu ý không đồng nhất:**
- `market_recent` có field `ticker` (khác `market_snapshot` và `market_itd` dùng `index`).
- Có 2 array riêng biệt: `recent_price` (giá) và `recent_trend` (xu hướng). Index của 2 array khớp nhau theo ngày.
- **Không có** `money_flow_score` (khác `stock_recent`, `industry_recent`, `group_recent`).

---

### `market_nntd` — Khối ngoại / Tự doanh toàn thị trường

**Số lượng:** 1 doc
**Cập nhật:** realtime + EOD

```json
{
  "nn": {
    "latest": { "date": "2026-04-17", "buy_value": 3192.3, "sell_value": -3140.6, "net_value": 51.7 },
    "week":   { "buy_value": 12427.8, "sell_value": -17116.4, "net_value": -4688.6 },
    "month":  { "buy_value": 62872.0, "sell_value": -74043.9, "net_value": -11171.9 }
  },
  "td": {
    "latest": { ... }, "week": { ... }, "month": { ... }
  }
}
```

**Đơn vị:** tỷ đồng. Được tổng hợp từ VNINDEX + HNX + UPCOM (toàn thị trường).
**Không có** field định danh ticker/index (1 doc duy nhất = toàn thị trường).

---

### `market_itd` — Intraday VNINDEX

**Số lượng:** 1 doc
**Cập nhật:** 5 phút / lần trong phiên

```json
{
  "index": "VNINDEX",
  "series": [
    {
      "datetime": ISODate("2026-04-17T15:00:00Z"),
      "open": 1820.0, "high": 1846.2, "low": 1812.6, "close": 1817.2,
      "volume": 0, "trading_value": NaN,
      "diff": -2.66, "pct_change": -0.0015, "vsi": 0.89
    },
    ...
  ]
}
```

**Lưu ý:**
- `volume` và `trading_value` trong index thường là `0` hoặc `NaN` (index là tính toán, không có KL giao dịch).
- `series[0]` là snapshot gần nhất.

---

## E. Khối lịch sử (History)

Toàn bộ chuỗi giá lịch sử dài hạn (index / ngành / mã). Dùng cho query on-demand khi cần chart dài hạn, phân tích cycle, backtest, hoặc so sánh giai đoạn — KHÔNG dùng cho phân tích realtime/EOD ngắn hạn (các collection `*_recent` đã cover 20 phiên gần nhất).

**Schema item chung** (áp dụng cho cả 3 collection history):

```json
{
  "date": "YYYY-MM-DD",
  "price": {
    "open": ..., "high": ..., "low": ..., "close": ...,
    "volume": ..., "pct_change": ...
  },
  "change": {"w_pct": ..., "m_pct": ..., "q_pct": ..., "y_pct": ...}
}
```

**Lưu ý chung:**
- `series` chứa toàn bộ lịch sử có sẵn — số lượng phần tử lớn (vài trăm đến vài nghìn phiên). **Luôn projection + `$slice` hoặc filter theo date range** khi query.
- Item thiếu dữ liệu phiên cụ thể → field `null`.
- Cấu trúc đơn giản hơn `*_recent`: KHÔNG có `money_flow_score`, `trend`, `technical_zone`, `volume_strength_index`.

### `history_index` — Lịch sử chỉ số thị trường

**Số lượng:** 1 doc (hiện tại chỉ `VNINDEX`)
**Cập nhật:** EOD (append phiên mới)
**Dùng khi:** chart VNINDEX nhiều năm, phân tích chu kỳ thị trường, so sánh giai đoạn, backtest macro.

```json
{
  "index": "VNINDEX",
  "series": [
    {
      "date": "YYYY-MM-DD",
      "price": {"open", "high", "low", "close", "volume", "pct_change"},
      "change": {"w_pct", "m_pct", "q_pct", "y_pct"}
    }
  ]
}
```

**Lưu ý:** `volume` index thường là `0` hoặc `NaN` (index là tính toán). Dùng `close` để chart trend dài hạn.

---

### `history_industry` — Lịch sử 24 ngành

**Số lượng:** ~24 doc (skip ngành không có data)
**Cập nhật:** EOD
**Dùng khi:** chart ngành nhiều năm, phân tích chu kỳ ngành, so sánh ngành qua giai đoạn dài.

```json
{
  "industry_name": "Ngân hàng",
  "series": [ ... item theo schema chung ... ]
}
```

**Whitelist 18 ngành áp dụng:** khi query lịch sử cho phân tích pack, filter `industry_name ∈ 18 whitelist` (xem mục B đầu khối ngành).

---

### `history_stock` — Lịch sử ~500 mã

**Số lượng:** ~500 doc (skip ticker không có data)
**Cập nhật:** EOD
**Dùng khi:** chart mã nhiều năm, phân tích pattern dài hạn, backtest, kiểm tra biến động qua các giai đoạn lớn (vd 2020 covid, 2022 sụp, 2023 phục hồi).

```json
{
  "ticker": "VCB",
  "series": [ ... item theo schema chung ... ]
}
```

**Lưu ý quan trọng về performance:**
- Tổng dung lượng collection lớn (~500 mã × vài trăm phiên/mã). KHÔNG `find({})` không projection.
- Luôn filter theo `ticker` cụ thể.
- Khi query nhiều mã song song, dùng `$in` + projection + `$slice` để giới hạn output.

---

## F. Khối tin tức

### `news_today_feed` — Tin hôm nay (metadata)

**Số lượng:** vài chục doc/ngày (phụ thuộc crawler)
**Cập nhật:** realtime trong ngày
**Dùng khi:** duyệt nhanh tiêu đề để chọn tin liên quan trước khi đọc full.

```json
{
  "article_slug": "vnm-cong-bo-ket-qua-q1-2026",
  "news_type": "doanh_nghiep",           // hoặc "quoc_te", "trong_nuoc", "thong_cao"
  "category_name": "Doanh nghiệp niêm yết",
  "title": "VNM công bố kết quả kinh doanh Q1/2026",
  "sapo": "Doanh thu tăng 15% so với cùng kỳ...",
  "tickers": ["VNM"],
  "created_at": "2026-04-17T09:30:00+07:00",
  "is_processed": false
}
```

**Field quan trọng:**
- `article_slug`: khoá nối với `news_today_content` (tương tự primary key).
- `tickers`: mảng tickers liên quan, có thể rỗng (tin vĩ mô chẳng hạn).
- `news_type`: 4 giá trị — `doanh_nghiep` (tin doanh nghiệp niêm yết, ~55% tổng tin), `quoc_te` (tin tài chính quốc tế, ~25%), `trong_nuoc` (tin nội địa nói chung, ~25%, bucket RỘNG gồm nhiều category con), `thong_cao` (tổng hợp chỉ đạo điều hành Chính phủ/Thủ tướng hàng ngày, ~1%, hầu hết không có ticker). Khi lọc `trong_nuoc`, thường phải lọc thêm qua `category_name` để tách tin vĩ mô/chính sách liên quan TTCK khỏi tin chính trị/đối ngoại/thời sự không liên quan.
- `is_processed`: cờ nội bộ, agent không cần quan tâm.

---

### `news_today_content` — Nội dung tin hôm nay

**Số lượng:** khớp với `news_today_feed` (cùng số doc)
**Cập nhật:** realtime

```json
{
  "article_slug": "vnm-cong-bo-ket-qua-q1-2026",
  "plain_content": "Nội dung đầy đủ bài báo..."
}
```

**Pattern dùng:** truy `news_today_feed` trước để lọc bài liên quan → lấy `article_slug` → query `news_today_content` theo slug để lấy full text.

---

### `news_history_feed` — Tin lịch sử 30 ngày (metadata)

**Số lượng:** ~1100 doc (1000 tin + 100 báo cáo)
**Cập nhật:** EOD (rolling 30 ngày)

```json
{
  "article_slug": "nvl-truoc-dhcd-novaland-bo-sung-noi-dung-quan-trong",
  "news_type": "doanh_nghiep",
  "category_name": "Doanh nghiệp niêm yết",
  "title": "NVL: Trước ĐHCĐ, Novaland công bố bổ sung nhiều nội dung quan trọng",
  "sapo": "...",
  "tickers": ["NVL"],
  "created_at": "2026-04-18T23:59:00+07:00",
  "is_processed": false,
  "type": "news_feed"     // hoặc "report_feed"
}
```

**Hai loại doc phân biệt qua `type`:**
- `news_feed`: tin thường, có `article_slug`.
- `report_feed`: báo cáo tổng hợp, có `report_slug` thay vì `article_slug`, thêm fields `report_type` (daily/weekly), không có `news_type`.

Khi agent muốn duyệt riêng tin hoặc riêng báo cáo, luôn thêm filter `{type: "news_feed"}` hoặc `{type: "report_feed"}`.

---

### `news_history_content` — Nội dung tin lịch sử

**Số lượng:** ~1100 doc
**Cập nhật:** EOD

```json
// Tin thường:
{
  "article_slug": "...",
  "plain_content": "...",
  "type": "news_content"
}

// Báo cáo:
{
  "report_slug": "...",
  "report_markdown": "Markdown đầy đủ báo cáo...",
  "type": "report_content"
}
```

**Field nội dung khác nhau:**
- Tin: `plain_content` (text thuần).
- Báo cáo: `report_markdown` (markdown có heading, bảng).

---

### URL pattern — Dẫn link finext.vn

`article_slug` và `report_slug` có thể ghép trực tiếp với base URL để tạo link bài báo/báo cáo gốc:

| Loại | Pattern | Ví dụ |
|---|---|---|
| Tin thường (`type: news_feed`) | `https://finext.vn/news/{article_slug}` | `https://finext.vn/news/nfc-dat-ke-hoach-lai-ky-luc-mot-doanh-nghiep-lan-chot-tra-co-tuc-tien-mat-50-ngay-trong-thang-5` |
| Báo cáo tổng hợp (`type: report_feed`) | `https://finext.vn/reports/{report_slug}` | `https://finext.vn/reports/bao-cao-tong-hop-tin-tuc-quoc-te-ngay-21-04-2026` |

**Khi nào dẫn link:**
- User yêu cầu explicit ("cho link bài báo", "dẫn link báo cáo")
- Output liệt kê nhiều tin/báo cáo dạng bảng hoặc danh sách → bổ sung link cho mỗi entry để user verify nhanh
- Khi cite claim cụ thể từ tin/báo cáo cho analysis → dẫn link bên cạnh để audit trail

**Lưu ý về K hygiene:** `article_slug` raw (dạng string code) thuộc diện cấm lộ trong output theo system prompt mục 4.5. Tuy nhiên URL `https://finext.vn/news/{slug}` là output user-facing hợp lệ, không tính vi phạm — khác biệt giữa ký hiệu nội bộ DB và URL công khai. Agent dẫn link URL đầy đủ, không để slug trần trong output dạng `article_slug: vnm-xyz`.

**Query projection:** các workflow tin ở `agent_db_02` đã include sẵn `article_slug` / `report_slug` trong projection nên agent có sẵn slug trong kết quả query, không cần query thêm.

---

## G. Khối vĩ mô & hàng hoá quốc tế

### `other_data` — Chỉ số vĩ mô, hàng hoá, thị trường quốc tế

**Số lượng:** 70 doc
**Cập nhật:** liên tục (mỗi chỉ số có `update_date` riêng). Hàng hoá và FX cập nhật hàng ngày; lãi suất liên ngân hàng cập nhật hàng ngày; chỉ số vĩ mô (CPI, xuất nhập khẩu, PMI...) cập nhật theo tháng.
**Dùng khi:** phân tích bối cảnh vĩ mô, mã/ngành có độ nhạy cao với hàng hoá, tỷ giá, lãi suất, hoặc thị trường quốc tế.

```json
{
  "name": "Vàng thế giới",
  "value": 4775.82,
  "unit": "USD/ounce",
  "pct_change": -0.0004,
  "w_pct": -0.0077,
  "m_pct": 0.0617,
  "q_pct": -0.0280,
  "y_pct": 0.3815,
  "update_date": "2026-04-19",
  "group": "commodities",
  "category": "metals"
}
```

**Giải nghĩa field:**
- `value`: giá trị hiện tại của chỉ số.
- `unit`: đơn vị gốc — đọc trực tiếp từ đây khi trình bày (USD/ounce, USD/MMBtu, USD/thùng, Đồng/kg, Triệu USD, Tỷ VNĐ, %, Nghìn người...). Với FX đơn vị có thể rỗng (`""`) vì là tỷ giá.
- `pct_change`: biến động phiên/ngày gần nhất (thập phân).
- `w_pct, m_pct, q_pct, y_pct`: biến động 1 tuần / 1 tháng / 1 quý / 1 năm (thập phân).
- `update_date`: ngày cập nhật gần nhất — **quan trọng**, vì các chỉ số vĩ mô có thể cập nhật chậm (hàng tháng, có thể cũ tới 2-3 tuần).
- `group`, `category`: dùng để filter theo nhóm.

**Cấu trúc `group` / `category`:**

| group | category | Số lượng | Chỉ số đại diện |
|---|---|---|---|
| `commodities` | `metals` | 6 | Vàng thế giới, Vàng trong nước (mua/bán), Bạc thế giới, Quặng sắt, Thép HRC |
| `commodities` | `energy` | 5 | Dầu WTI, Dầu Brent, Khí tự nhiên, Than nhiệt, Than cốc |
| `commodities` | `agriculture` | 11 | Cà phê, Hồ tiêu, Cao su, Gạo XK, Đường, Ngô, Đậu tương, Lúa mì, Sợi cotton, Heo hơi, Tôm thẻ |
| `commodities` | `chemical` | 6 | Nhựa PP, PVC, PET, Urea Trung Đông/Trung Quốc, Phốt pho vàng |
| `international` | `global_index` | 6 | S&P 500, Dow Jones, Nasdaq, NYSE Composite, Nikkei 225, Shanghai Composite |
| `international` | `fx` | 7 | EUR/USD, GBP/USD, USD/JPY, USD/CHF, USD/CAD, AUD/USD, NZD/USD |
| `international` | `crypto` | 4 | Bitcoin (BTC), Ethereum (ETH), Ripple (XRP), Litecoin (LTC) |
| `international` | `bonds` | 2 | TPCP Mỹ 5 năm, TPCP Mỹ 10 năm |
| `macro` | `exchange_rate` | 5 | Tỷ giá trung tâm, trần, sàn, USD NHTM bán, USD tự do bán |
| `macro` | `monetary` | 9 | Lãi suất tái cấp vốn, chiết khấu, liên ngân hàng các kỳ hạn, huy động các kỳ hạn, cho vay qua đêm |
| `macro` | `economy` | 9 | CPI, PMI, IIP, Xuất khẩu, Nhập khẩu, Cán cân TM, Bán lẻ HH&DV, Vận chuyển hành khách/hàng hoá |

**Sử dụng trong phân tích:**
- **Dầu khí** (PVS, PVD, BSR, PLX...) ⟵ nhạy `commodities.energy` (Dầu Brent, Dầu WTI).
- **Thép** (HPG, HSG, NKG...) ⟵ nhạy `commodities.metals` (Quặng sắt, Thép HRC) và `commodities.energy` (Than cốc).
- **Phân bón** (DCM, DPM, BFC...) ⟵ nhạy `commodities.chemical` (Urea Trung Đông/Trung Quốc).
- **Ngân hàng** (VCB, BID, CTG...) ⟵ nhạy `macro.monetary` (lãi suất) và `macro.exchange_rate` (tỷ giá).
- **Bất động sản** (VHM, VIC, NVL...) ⟵ nhạy `macro.monetary` (lãi suất huy động, cho vay).
- **Xuất khẩu** (dệt may, thuỷ sản, đồ gỗ, cao su) ⟵ nhạy `macro.exchange_rate` (USD/VND) và `international.fx`.
- **Vàng/Bạc** (PNJ có mảng trang sức, DOJI...) ⟵ nhạy `commodities.metals`.
- **Nông nghiệp** (HAG, HNG, DBC, PAN...) ⟵ nhạy `commodities.agriculture` (giá heo, cà phê, hồ tiêu, đường).
- **Tâm lý toàn cầu** ⟵ tham khảo `international.global_index` (đặc biệt S&P 500, Nikkei, Shanghai).

**Cảnh báo đơn vị:**
- Lãi suất: `value: 0.045, unit: "%"` nghĩa là 4.5% — đã dạng thập phân, nhân 100 khi trình bày.
- Tỷ giá: đơn vị thường rỗng hoặc "VND/USD". USD NHTM bán thường quanh 25,000-26,000 VND.
- Chỉ số vĩ mô tháng (xuất nhập khẩu, bán lẻ): `update_date` có thể là cuối tháng trước. Đơn vị "Triệu USD" hoặc "Tỷ VNĐ" đọc kỹ.
- CPI: giá trị là chỉ số hoặc phần trăm thay đổi YoY, đọc `unit` để biết.

---

## H. Khối briefing

### `data_briefing` — Toàn cảnh thị trường 6 block

**Số lượng:** 6 doc
**Cập nhật:** realtime + EOD
**Dùng khi:** câu hỏi chung như "thị trường hôm nay thế nào", "ngành nào mạnh nhất", "tin gì nổi bật", "bức tranh vĩ mô" — 1 collection trả lời được đa số trường hợp.

**6 loại block** (phân biệt qua `type`):

```json
// Block 1: market_snapshot - VNINDEX hôm nay
{
  "type": "market_snapshot",
  "data": [ <1 doc tương tự market_snapshot> ]
}

// Block 2: market_nntd - NN/TD toàn thị trường
{
  "type": "market_nntd",
  "data": [ <1 doc tương tự market_nntd> ]
}

// Block 3: group_snapshot - 6 nhóm
{
  "type": "group_snapshot",
  "data": [ <6 doc tương tự group_snapshot> ]
}

// Block 4: industry_snapshot - 24 ngành
{
  "type": "industry_snapshot",
  "data": [ <24 doc tương tự industry_snapshot> ]
}

// Block 5: other_data - 70 chỉ số vĩ mô / hàng hoá / quốc tế
{
  "type": "other_data",
  "data": [ <70 doc tương tự other_data> ]
}

// Block 6: news_report - 4 báo cáo tổng hợp gần nhất
{
  "type": "news_report",
  "data": [
    {
      "report_slug": "...",
      "title": "...",
      "sapo": "...",
      "report_markdown": "### 1. MBB: ĐHĐCĐ thông qua... ###",
      "report_type": "daily",
      "category_name": "Tin tức doanh nghiệp",
      "created_at": "2026-04-18T22:00:00+07:00",
      "tickers": ["BID", "DVN", "HAG", ...]
    },
    ...  // 4 báo cáo
  ]
}
```

**Strategy dùng:**
- Câu hỏi tổng quan về **giá/biến động thị trường** → lấy `type: market_snapshot`.
- Câu hỏi về **khối ngoại** → lấy `type: market_nntd`.
- Câu hỏi **ngành nào dẫn dắt / yếu nhất** → lấy `type: industry_snapshot`.
- Câu hỏi **vốn hoá lớn hay nhỏ đang mạnh** → lấy `type: group_snapshot`.
- Câu hỏi **vĩ mô, giá dầu, giá vàng, tỷ giá, lãi suất, thị trường quốc tế** → lấy `type: other_data`.
- Câu hỏi **tin/sự kiện nổi bật** → lấy `type: news_report`.
- Câu hỏi tổng hợp "hôm nay thế nào" → lấy `type: { $in: [...] }` với nhiều block.

**Lưu ý:**
- Nội dung block chính là **clone** từ collection nguồn, độ tươi đồng bộ cập nhật.
- `news_report` chỉ có 4 bài, là các daily report mới nhất — không đầy đủ mọi tin.
- Block `other_data` clone đầy đủ 70 chỉ số, là cách nhanh nhất để lấy toàn bộ bức tranh vĩ mô trong 1 query.

---

## Tham khảo chéo — Mối quan hệ giữa collection

```
Ticker-level:
  stock_info (static)
    ├─> stock_snapshot (state hôm nay)
    ├─> stock_recent   (chuỗi 20 phiên)
    ├─> stock_finstats (BCTC)
    ├─> stock_nntd     (NN/TD)
    └─> stock_itd      (intraday)

Industry-level:
  industry_info (static)
    ├─> industry_snapshot
    ├─> industry_recent
    └─> industry_finstats

Group-level:
  group_snapshot ─> group_recent

Market-level:
  market_snapshot ─> market_recent
  market_nntd (standalone)
  market_itd (standalone)

History (lịch sử dài hạn — query on-demand):
  history_index    (VNINDEX toàn bộ lịch sử)
  history_industry (24 ngành lịch sử)
  history_stock    (~500 mã lịch sử)

Cross-level aggregates:
  data_briefing   (tổng quan thị trường + ngành + nhóm + vĩ mô + report)

Macro / international:
  other_data (hàng hoá, FX, crypto, TPCP, global index, monetary, economy, exchange_rate)

News:
  news_today_feed    ─slug─> news_today_content
  news_history_feed  ─slug─> news_history_content
```

---

## Anti-patterns — Những điều KHÔNG nên làm

1. **Không `find({})` không projection trên `stock_finstats`.** Mỗi doc có thể ~30KB (9 quý × 56 chỉ tiêu + 6 năm). Luôn projection lấy đúng phần cần.

2. **Không sort lại `series` trong các collection `*_recent`, `*_itd`.** Đã sort sẵn (mới nhất ở đầu). Sort thừa tốn CPU.

3. **Không giả định tên field đồng nhất.**
   - Ticker: `ticker` (stock), `industry_name` (industry), `group_name` (group), `index` (market_snapshot + market_itd), `ticker` (market_recent — bất đồng bộ).
   - Xử lý bằng cách đọc doc mẫu trước khi viết query phức tạp.

4. **Không dùng `today_news.is_processed`.** Đây là cờ workflow nội bộ của pipeline build, không mang ý nghĩa phân tích.

5. **Không gộp `news_history_feed` và `news_history_content` bằng `article_slug` mà không filter `type`.** Cùng collection có cả `article_slug` (tin) lẫn `report_slug` (báo cáo) — query theo slug có thể không match nếu gõ nhầm field.

6. **Không so sánh `valuation_ratios` giữa các `type` khác nhau.** Doanh nghiệp SXKD và NGANHANG có bộ chỉ tiêu khác. Ví dụ P/B của ngân hàng không cùng ý nghĩa với P/B của công ty sản xuất.

7. **Không nhân `pct_change` với 100 để có phần trăm rồi quên nhân lại khi so sánh.** `pct_change` đã ở dạng thập phân (0.07 = 7%). Giữ nhất quán đơn vị khi dùng trong pipeline.

8. **Không hiển thị ký hiệu nội bộ ra user.** `vsi`, `day_score`, `week_score`, `zone` (AAA/AA/A/B/C), `f382`, `poc`, `period: "2025_4"` — phải dịch sang ngôn ngữ tự nhiên trong câu trả lời cuối. Bảng dịch chi tiết và ngưỡng diễn giải ở `agent_db_04`.

9. **Không giả định `market_snapshot.breadth` là toàn sàn HOSE.** Đây là breadth rổ FNXINDEX (rổ lọc nội bộ), phản ánh "cổ phiếu chất lượng" chứ không toàn sàn.

10. **Không dùng `_id` của MongoDB làm khoá kinh doanh.** Chúng là random ObjectId sau mỗi lần rebuild collection, không ổn định.

11. **Không bỏ qua `update_date` khi dùng `other_data`.** Các chỉ số vĩ mô tháng có thể cũ 2-3 tuần so với thời điểm query. Khi trình bày, nếu số liệu không phải của ngày hôm nay, ghi chú ngày cập nhật để user hiểu đúng bối cảnh.

12. **Không quên `unit` khi đọc `other_data`.** Mỗi chỉ số có đơn vị riêng (USD/ounce, Triệu USD, %, Đồng/kg, Nghìn người...). Trình bày không có đơn vị = số liệu vô nghĩa.

13. **Không giả định `other_data.name` là chuẩn hoá hoàn toàn.** Ví dụ có 2 doc "Vàng trong nước" (một cho giá mua, một cho giá bán). Khi filter theo `name`, cần chính xác từng ký tự, hoặc dùng regex nếu nhóm.
