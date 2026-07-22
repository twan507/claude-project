# P_vbse_strategy_02 — Trục 2: Định vị thị trường VN trong chu kỳ

File này detail Trục 2 của framework 6 trục. Dependency: `P_vbse_strategy_00` master (philosophy fundamental-driven + weight balance + nguyên tắc bất biến).

**Master weight balance áp dụng cho Trục 2:**
- Trục 2 thuộc tầng **SECONDARY** (~15-20% tổng báo cáo)
- **Cap technical: ≤ 20% nội dung trục** — chỉ minh hoạ phụ, không phải driver
- PRIMARY của Trục 2: định giá phân vị + dòng tiền aggregate + FII + breadth

> ## ⚠️ FUNDAMENTAL-FIRST OVERRIDE
>
> Trục 2 trong nhiều framework retail VN thường bị technical/flow dominate (vocab "tích luỹ / phân phối / quá mua / breakout"). **Pack này đảo ngược pattern đó:** Trục 2 phải dẫn dắt bằng **định giá phân vị lịch sử + dòng tiền aggregate 18 ngành whitelist + FII flow + breadth fundamental**. Technical_zone đa khung VNINDEX + chart pattern chỉ làm **minh hoạ phụ** (≤20%), KHÔNG quyết định định vị.
>
> Vocab "tích luỹ / phân phối / quá mua" được phép dùng làm shorthand sau khi đã chốt định vị bằng định giá + flow, KHÔNG được dùng làm trigger độc lập.

## 1. Mục tiêu & câu hỏi cốt lõi

**Câu hỏi:** VNINDEX đang ở **đâu** trong chu kỳ **định giá** / chu kỳ **dòng tiền** / chu kỳ **fundamental breadth** của thị trường VN?

**Output mong đợi:** kết luận định vị thị trường định tính (vd "định giá rẻ phân vị 28% + dòng tiền cải thiện 4 tuần liên tiếp + FII chuyển mua ròng + breadth ngành whitelist 65% tăng giá tháng → đang ở giai đoạn phục hồi sớm có cơ sở fundamental") — dùng làm input cho Trục 4 sector + Trục 5 risk + Trục 6 watchlist.

## 2. Lăng kính phân tích (theo Weight balance)

### 2.1. PRIMARY — Driver chính (~75-80% nội dung trục)

**(a) Định giá tổng thể — phân vị lịch sử**

- P/E VNINDEX hiện tại so với median 3-5 năm — phân vị (percentile) bao nhiêu?
- P/B VNINDEX hiện tại so với median 3-5 năm
- P/E forward (nếu có forward EPS consensus) so với median historical
- **Diễn giải fundamental, không technical:** phân vị < 30% = rẻ tương đối → re-rating opportunity; phân vị > 70% = đắt tương đối → de-rating risk; phân vị 40-60% = trung tính

**(b) Dòng tiền cấp thị trường — aggregate 18 ngành whitelist**

- Mean/median `industry_snapshot.money_flow_score.week_score` qua **18 ngành whitelist** (xem `P_vbse_strategy_00` Nguyên tắc 3 + `K_agent_db_01` Section B) — KHÔNG aggregate trên 24 ngành raw
- Xu hướng 4-8 tuần gần nhất — improving / stable / deteriorating
- **Lý do dùng aggregate ngành làm proxy:** `market_snapshot` không có `money_flow_score` cấp thị trường (xem `K_agent_db_01`). Aggregate ngành whitelist là proxy hợp lệ duy nhất.

**(c) Khối ngoại tháng/quý**

- `market_nntd` aggregate tháng/quý — net flow VN
- Break trend không (chuyển bán → mua, hoặc ngược lại)
- Beta VN với MSCI EM (cross-check qua web search)

**(d) Breadth tổng thể — fundamental angle**

- % ngành **whitelist 18** có biến động tháng > 0 (`industry_snapshot.change.m_pct > 0`)
- % mã trong universe whitelist tăng giá tháng/quý
- **Breadth fundamental:** % ngành có `industry_finstats.financial_statements.quarterly` cho thấy EPS YoY > 0 quý gần nhất — bổ sung breadth giá bằng breadth cơ bản

### 2.2. SECONDARY — Cross-check, không quyết định độc lập (~10% nội dung)

**(e) Sentiment proxy thanh khoản**

- Thanh khoản trung bình tháng vs trung bình 6 tháng — phản ánh đám đông tham gia
- Cross-check với `data_briefing` doc `core` (`market.breadth.in/out`) cho breadth phiên (chỉ làm cross-check, không driver)

### 2.3. TERTIARY — Minh hoạ phụ (~5-10% nội dung, BẮT BUỘC ≤20%)

**(f) Technical đa khung (chỉ visualization, không driver)**

- `market_snapshot.technical_zone.overall.w/m/q/y` — chỉ ghi nhận xếp hạng kỹ thuật đa khung làm minh hoạ
- `market_recent.series` 20 phiên — chart price + volume vận động (visualization)
- Volume profile, POC, MA — chỉ ghi nhận, KHÔNG dùng làm trigger định vị

**Rule cứng cho Trục 2:**
- KHÔNG bắt đầu kết luận định vị bằng cụm technical (vd "VNINDEX đang trong vùng tích luỹ technical zone w=AA")
- BẮT ĐẦU kết luận bằng cụm fundamental (vd "Định giá phân vị 28% + dòng tiền cải thiện 4 tuần + FII chuyển mua → định vị phục hồi sớm có cơ sở")
- Nhắc technical chỉ sau khi đã chốt định vị bằng PRIMARY, dạng "kèm minh hoạ kỹ thuật: technical_zone w=A/m=A — đồng thuận"

## 3. Hướng tìm dữ liệu

| Loại data | Collection / nguồn | Field cụ thể | Tầng |
|---|---|---|---|
| P/E / P/B VNINDEX phân vị | `market_snapshot` + lịch sử `market_recent` aggregate; nếu thiếu, aggregate từ `industry_finstats.valuation_ratios` (18 ngành whitelist) | `valuation_ratios.PE`, `PB`, percentile vs median 3-5Y | PRIMARY |
| Dòng tiền aggregate thị trường | `industry_snapshot` (18 doc whitelist) | `money_flow_score.week_score` mean/median; trend 4-8 tuần qua `industry_recent` | PRIMARY |
| FII tháng/quý | `market_nntd` | aggregate net flow tháng/quý, break trend detect | PRIMARY |
| Breadth giá ngành | `industry_snapshot` (18 doc whitelist) | `change.m_pct`, `change.q_pct` | PRIMARY |
| Breadth giá mã | `stock_snapshot` filter `industry ∈ 18` | `change.m_pct > 0`, `change.q_pct > 0` count / total | PRIMARY |
| **Breadth cơ bản** | `industry_finstats` (18 doc whitelist) | `financial_statements.quarterly` EPS YoY > 0 count | PRIMARY |
| Sentiment thanh khoản | `market_recent.series` (20 phiên) + `market_snapshot.volume_profile` | volume trung bình tháng vs trung bình 6 tháng | SECONDARY |
| Breadth phiên cross-check | `data_briefing` doc `core` | `market.breadth.in/out` | SECONDARY |
| Beta VN vs MSCI EM | Web search | — | SECONDARY |
| Technical đa khung VNINDEX | `market_snapshot` | `technical_zone.overall.w/m/q/y`, `ma_zone`, `fibonacci_zone`, `volume_profile_zone` | TERTIARY (minh hoạ) |
| Chuỗi giá+volume 20 phiên | `market_recent` | `series` (sort giảm dần) | TERTIARY (visualization) |

**Trọng số nguồn ước:** ~95% DB + ~5% web search (beta EM).

## 4. Output diễn giải

**Format diễn đạt định vị:**

1. **Mở đầu bằng PRIMARY (fundamental + flow):** câu chốt 2-3 dòng dùng định giá phân vị + dòng tiền aggregate + FII + breadth.

   Ví dụ:
   - "Định giá VNINDEX P/E forward 11.2x ở phân vị 32% so median 5 năm (rẻ tương đối) + dòng tiền aggregate 18 ngành whitelist cải thiện 4 tuần liên tiếp (week_score median tăng từ -2.1 lên +5.8) + FII chuyển mua ròng 3.4 nghìn tỷ tháng (break trend bán ròng 3 tháng) + breadth ngành whitelist 67% tăng giá tháng → định vị **phục hồi sớm có cơ sở fundamental**."

2. **Cross-check SECONDARY:** 1-2 dòng về thanh khoản + breadth phiên.

3. **Minh hoạ TERTIARY (optional, ≤20% nội dung trục):** 1-2 dòng technical đa khung làm visualization. Phải prefix "Kèm minh hoạ kỹ thuật:" để rõ vai trò minh hoạ.

   Ví dụ: "Kèm minh hoạ kỹ thuật: technical_zone w=A/m=AA/q=A/y=AA — đồng thuận multi-frame, không mâu thuẫn định vị fundamental phục hồi sớm."

**Vocab định vị được phép (sau khi đã chốt bằng PRIMARY):**
- Tích luỹ sau giảm
- Phục hồi sớm
- Uptrend khoẻ (có cơ sở fundamental)
- Quá mua cảnh báo (định giá phân vị > 75%)
- Phân phối sau đỉnh quý
- Suy yếu

**Vocab CẤM dùng làm trigger định vị độc lập:**
- "Breakout vùng kháng cự" (technical-only)
- "Tích luỹ wyckoff phase X" (technical-only)
- "Quá bán kỹ thuật" (technical-only)
- "MA20 cắt lên MA60" (technical-only)

Vocab cấm chỉ được dùng làm **đóng góp phụ** sau khi PRIMARY đã chốt.

## 5. Cross-reference đầu ra Trục 2

Output Trục 2 feed vào:
- **Trục 4 sector** (`_04`): định vị thuận lợi → sector tilt rộng hơn; định vị áp lực → defensive only
- **Trục 5 risk** (`_05`): định vị quá mua (phân vị > 75%) → thêm risk de-rating
- **Trục 6 watchlist** (`_06`): định vị thuận lợi → Bucket 1 active; định vị áp lực → ưu tiên Bucket 2-3 chờ pullback
- **Workflow Monthly Stage 1** (`_07`): output Trục 2 + Trục 1 + Trục 3 là input cho Checkpoint 1

## 6. Edge cases

- **P/E forward consensus không có trong DB:** dùng web search benchmark hoặc fallback dùng P/E trailing. Ghi rõ nguồn + ngày update.
- **Mâu thuẫn PRIMARY:** vd định giá rẻ phân vị 25% nhưng dòng tiền aggregate xấu + FII bán ròng → ghi rõ tension, present cả 2 view, kết luận định vị "trung tính chuyển động phức tạp", feed vào Trục 5 risk.
- **Mâu thuẫn fundamental vs technical:** vd PRIMARY chốt phục hồi sớm nhưng technical_zone w=C đa khung → vẫn chốt định vị theo PRIMARY (phục hồi sớm), kèm note "technical chưa đồng thuận, theo dõi xác nhận cuối Trục 6 Bucket entry".
- **Dữ liệu thiếu cho 1 lăng kính PRIMARY:** ghi rõ "thiếu data X, dùng N lăng kính còn lại". Không bịa số.
- **Sentiment proxy bất thường:** vd thanh khoản tháng vọt 2x trung bình 6 tháng nhưng PRIMARY trung tính → flag inline "thanh khoản tăng đột biến cần monitor breadth phiên để phân biệt rotation vs FOMO".

## 7. Self-audit Trục 2 (trước khi xuất)

- [ ] Câu chốt định vị mở đầu bằng PRIMARY (định giá + flow + FII + breadth), KHÔNG bằng technical
- [ ] Aggregate dòng tiền tính trên **18 ngành whitelist**, không 24
- [ ] Technical xuất hiện sau PRIMARY, có prefix "Kèm minh hoạ kỹ thuật:" hoặc tương đương
- [ ] % nội dung technical trong trục ≤ 20%
- [ ] Vocab cấm (breakout / wyckoff / MA cắt) không xuất hiện làm trigger độc lập
- [ ] Có breadth cơ bản (EPS YoY > 0 % ngành) bổ sung cho breadth giá
- [ ] Cross-reference Trục 4/5/6 đã ghi rõ implication

Vi phạm bất kỳ item nào → re-weight + đào thêm PRIMARY trước khi render.
