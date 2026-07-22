# P_invest_memo_06 — Tier 5B: Valuation Modeling

Giai đoạn 5B của quy trình. Chỉ làm cho top 3-5 mã High conviction từ tier 3 đã clear forensic ở tier 5A. Build 3-statement projection + DCF (hoặc phương pháp tương đương theo type) + peer multiples cross-check → target giá base/bull/bear.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, `P_invest_memo_04` (tier 3 conviction + flags), `P_invest_memo_05` (tier 5A forensic findings), `K_agent_db_04` D1-D4 (methodology 4 type).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** đưa ra target giá **base / bull / bear** cho mỗi mã, từ **ít nhất 2 phương pháp độc lập** để cross-check. Target giá là input cho:
- Memo tier 5C phần 4 (định giá) và phần 7 (exit trigger)
- Quyết định size position ở tier 6 (% portfolio)
- Exit rule khi price đạt base/bull target

**Scope:** chỉ làm cho top 3-5 mã **High conviction** (15-18 điểm) hoặc Medium conviction với thesis mạnh (12-14 điểm với catalyst rõ). Mã Low conviction (8-10 điểm) **không cần** DCF đầy đủ — dùng peer multiples cross-check nhanh ở tier 5C là đủ.

**Input:**

1. File output tier 5A (`tier5A_<ticker>_YYYYMMDD_confirmed.md`) — red flag đã clear + key findings (giả định BCTC cần lưu ý, số liệu extract từ PDF)
2. Số liệu BCTC từ DB (stock_finstats) — tier 5A đã verify qua PDF, giờ dùng để extract chuỗi dài
3. Bảng catalyst mã từ tier 3 — ảnh hưởng giả định tăng trưởng
4. Benchmark ngành (industry_finstats) — từ tier 2
5. Web search: dự báo tăng trưởng ngành, peer quốc tế, consensus sell-side

**Output chính:**

1. **Bảng target giá** base/bull/bear với mỗi phương pháp
2. **Bảng giả định** chi tiết (tăng trưởng, biên, WACC, terminal)
3. **Bảng sensitivity** 2 chiều cho biến quan trọng nhất
4. **Cross-check analysis** — so 2 phương pháp, chênh bao nhiêu, diễn giải
5. **Report Checkpoint 5B per-stock** — user challenge giả định trước khi chốt

**Tần suất:** per-stock. Mỗi mã 1 session.

**Thời gian session:** 90-120 phút/mã Agent work (build model + sensitivity). User review checkpoint 30-45 phút (challenge giả định).

---

## 2. Triết lý

**Modeling là cross-check, không phải source of truth.** Target giá DCF không chính xác đến đồng — thị trường thực tế có thể lệch 20-30% so với DCF do sentiment, flow, catalyst chưa priced-in. Vai trò của modeling:

- **Thiết lập anchor:** biết mức giá hợp lý khoảng nào (không phải điểm chính xác)
- **Phát hiện mispricing:** nếu thị trường giá cao hơn bull target nhiều → overvalued, có thể loại; nếu thấp hơn bear target → có thể undervalued
- **Stress-test thesis:** build model forces Agent + user phải explicit về giả định. Giả định không hợp lý → thesis yếu
- **Exit trigger:** base target đạt → cân nhắc chốt phần, bull target đạt → cân nhắc thoát

**Không over-engineer model.** Model 50 dòng giả định với 10 scenario không có ý nghĩa nếu các giả định không defensible. Tốt hơn là 5-10 giả định rõ ràng, mỗi giả định có lý do cụ thể, với 3 scenario base/bull/bear + 1-2 sensitivity table.

**Cross-check 2 phương pháp là bắt buộc.** 1 DCF đơn lẻ dễ bị over-fitting giả định. Ít nhất 1 phương pháp cross-check (peer multiples, sum-of-parts, residual income) để calibrate.

---

## 3. Bốn approach theo type

Không áp cùng 1 DCF cho mọi mã. Mỗi type có method chính phù hợp cấu trúc kinh doanh, + method cross-check.

### 3.1. SXKD — DCF FCFF + Peer Multiples

**Method chính: Discounted Cash Flow (FCFF)**

Free Cash Flow to Firm (FCFF — dòng tiền tự do về doanh nghiệp, đo cash dùng được cho cả chủ nợ và cổ đông sau khi trừ CapEx và đầu tư vốn lưu động) 5 năm + terminal value, chiết khấu về hiện tại với WACC (Weighted Average Cost of Capital — chi phí vốn bình quân gia quyền, đại diện cho tỷ suất sinh lời yêu cầu trung bình của toàn bộ nguồn vốn).

Công thức:
```
Firm Value = Σ (FCFF_t / (1+WACC)^t) + Terminal Value / (1+WACC)^5
Equity Value = Firm Value − Debt + Cash
Target Price = Equity Value / Outstanding Shares
```

Trong đó:
- **FCFF** = EBIT × (1−Tax Rate) + D&A (Depreciation & Amortization — khấu hao hữu hình và vô hình) − CapEx − ΔWorking Capital
- **WACC** = weighted(Cost of Equity, Cost of Debt after tax)
- **Terminal Value** = FCFF_5 × (1+g) / (WACC − g), với g = perpetual growth (2-4% cho VN)

**Method cross-check: Peer Multiples**

- EV/EBITDA (Enterprise Value / Earnings Before Interest, Tax, Depreciation & Amortization — giá trị doanh nghiệp trên lợi nhuận trước lãi vay, thuế, khấu hao; phản ánh mức định giá của toàn doanh nghiệp không phụ thuộc cấu trúc vốn) so với median ngành (industry_finstats)
- P/E so với median ngành
- P/B so với median ngành (bổ sung nếu industry có pattern rõ)
- Cross-check với 3-5 peer quốc tế cùng ngành (web search)

Target giá multiples = EBITDA kỳ forward × EV/EBITDA target − Net Debt, chia số cổ phiếu.

**Cross-check:** DCF target và Multiples target không nên chênh > 30%. Chênh lớn = re-examine giả định.

### 3.2. NGANHANG — Residual Income hoặc P/B với ROE Model

**Method chính: Residual Income Model (RIM)**

Value = BVPS + Σ (RI_t / (1+r)^t) + Terminal RI / (r−g) / (1+r)^5

Trong đó:
- **BVPS** (Book Value Per Share — giá trị sổ sách trên mỗi cổ phiếu = vốn chủ / số cổ phiếu đang lưu hành)
- **RI_t** (Residual Income — lợi nhuận thặng dư) = (ROE_t − Cost of Equity) × BVPS_(t−1)
- Cost of Equity = Risk-free rate + Beta × Equity Risk Premium
- Không dùng WACC — bank không có Capex/WC truyền thống

**Lý do dùng RIM cho bank:** EPS bank bị distort bởi provision + tax timing, khó project chính xác. BVPS ổn định hơn vì reflect capital base. RIM focus vào spread ROE − Cost of Equity × capital.

**Method cross-check: P/B với ROE Model**

Công thức Gordon Growth cho bank:
```
Fair P/B = (ROE_sustainable − g) / (Cost of Equity − g)
```

Target giá = Fair P/B × BVPS.

ROE sustainable = ROE trung bình 3-5 năm adjust cho cycle (bank ở đỉnh cycle → adjust xuống 10-15%).

**Cross-check:** RIM và P/B-ROE nên cho target tương đương (chênh < 20%). Nếu lớn → kiểm tra ROE sustainable có đúng không.

### 3.3. CHUNGKHOAN — P/B với Cyclical ROE

**Method chính: P/B-based với ROE cycle-adjusted**

CK có P&L biến động cực mạnh theo thị trường (FVTPL gain/loss, margin lending thay đổi theo thanh khoản). DCF quý đơn không hợp lý vì FCF biến động ± 50% năm-tới-năm.

Công thức:
```
Target P/B = (ROE_mid_cycle − g) / (Cost of Equity − g)
Target Price = Target P/B × BVPS
```

Với:
- **ROE_mid_cycle** = trung bình ROE annualized qua 1-2 chu kỳ thị trường (không lấy ROE spike năm bull hay crash năm bear)
- g = 4-6% cho CK VN (ngành tăng trưởng nhờ market cap hoá + thanh khoản tăng)

**Method cross-check: EV/Revenue + Sum-of-Parts**

Sum-of-Parts (SOP — định giá từng mảng kinh doanh riêng rồi cộng lại, phù hợp với doanh nghiệp có nhiều segment không đồng nhất): CK có nhiều mảng doanh thu (môi giới, margin lending, tự doanh, IB). Sum-of-parts:
- Doanh thu môi giới × multiple phù hợp (thường 3-5x revenue)
- Margin loans balance × 1.2-1.5x book (nếu rủi ro thấp)
- FVTPL portfolio × 1x book
- IB revenue × 2-4x revenue

**Cross-check:** P/B-ROE và Sum-of-parts chênh < 25%.

### 3.4. BAOHIEM — Embedded Value + P/B

**Method chính: Embedded Value (cho Life), hoặc P/B-ROE (cho Non-life + Life)**

**Life Insurance (ít mã VN):** Embedded Value = Adjusted Net Worth + Present Value of In-Force Business. Phức tạp, cần model actuarial — hiếm khi analyst individual làm được. Dùng public EV disclosure của công ty nếu có.

**Non-life Insurance (phổ biến VN):** P/B-ROE model tương tự bank, với adjustment:
```
Target P/B = (ROE_normalized − g) / (Cost of Equity − g)
```

Với:
- ROE_normalized = ROE trung bình 5 năm, loại năm bất thường (bão lũ lớn, claims spike)
- Leverage cao đặc thù (7-11x) — không so với SXKD/CK

**Method cross-check: P/E với Cyclical Earnings**

Peer multiples so với ngành (BH ngành VN có 10 mã — pool peer nhỏ).

**Cross-check:** P/B và P/E chênh < 30%.

---

## 4. Build 3-Statement Projection — nguyên tắc (chủ yếu cho SXKD)

3-statement = Income Statement + Balance Sheet + Cash Flow Statement, tích hợp qua 5 năm.

### Lý do cần 3-statement

DCF thuần chỉ cần FCFF, không cần full 3-statement. Nhưng 3-statement buộc Agent phải **consistent giữa các giả định:**
- Tăng trưởng doanh thu (IS) phải nhất quán với tăng CapEx (CFS) và tăng Fixed Assets (BS)
- Biên lợi nhuận (IS) phải nhất quán với Working Capital (BS, CFS)
- Net Income (IS) phải nhất quán với Equity roll-forward (BS)

Nếu Agent chỉ build FCFF 1 dòng, dễ bỏ sót inconsistency.

### Bước build 3-statement 5 năm

**Bước 1 — Historical 3-5 năm từ BCTC (đã extract trong tier 5A):**
- Income Statement: Revenue, COGS, Gross Profit, OPEX, EBIT, Interest, Tax, Net Income
- Balance Sheet: Current Assets, Fixed Assets, Total Assets, Current Liab, Debt, Equity
- Cash Flow: CFO, CFI, CFF, Net Cash Change

**Bước 2 — Projection 5 năm:**

**Revenue** — driver chính. Cách project:
- **Bottom-up** (preferred): phân theo segment (VD: MWG = Điện máy + Bách Hóa Xanh + An Khang). Mỗi segment project riêng: số store × revenue per store, hoặc volume × price
- **Top-down** (fallback nếu không có segment data): Revenue = Revenue_prev × (1 + growth_rate). Growth rate lấy từ: consensus sell-side + historical CAGR + ngành growth + catalyst impact
- Catalyst: nếu có catalyst từ tier 0/2 (KQKD tăng Q, thoái vốn, luật mới hiệu lực), quantify impact — thường 3-10% bump cho năm relevant

**Gross Margin** — thường stable theo ngành, trừ khi:
- Mã có cost pressure (nguyên liệu tăng) → giảm margin 0.5-1.5%
- Mã có pricing power (thương hiệu mạnh) → ổn định hoặc tăng nhẹ
- Mã vào giai đoạn scale (công nghệ, retail) → margin mở rộng 1-2%/năm rồi ổn định

**OPEX** — thường % doanh thu hoặc tăng theo tốc độ mở rộng. Lưu ý operating leverage: doanh thu tăng 15% → OPEX tăng chậm hơn 10-12% → EBIT margin mở rộng.

**CapEx** — dựa trên:
- BCTN có guidance capex 2-3 năm tới
- Historical capex % doanh thu
- Plan mở rộng nhà máy / store / capacity

**Working Capital** — giả định DSO/DIO/DPO ổn định theo trend hoặc có adjustment nhỏ. Nếu tier 5A phát hiện DSO tăng → project thêm tăng 3-5% → WC tăng → FCF giảm.

**Net Debt** — từ schedule repay + new borrowing. Nếu công ty deleveraging → Debt giảm đều.

**Bước 3 — Reconcile:**
- Net Income (IS) → Retained Earnings (BS) roll-forward
- CFO (CFS) + CFI (CapEx) + CFF (debt change) → Net Cash Change (BS)
- Kiểm tra Balance Sheet có balance không (Assets = Liab + Equity)

Nếu unbalance > 1% → có error, tìm và sửa.

### Simplify cho Medium conviction

Cho mã Medium conviction (12-14đ), không cần 3-statement đầy đủ. Thay bằng:
- Revenue + Gross Margin + EBIT margin → EBIT
- EBIT × (1−Tax) + D&A − CapEx − ΔWC estimate → FCFF
- Skip full Balance Sheet, Working Capital chỉ estimate từ DSO/DIO/DPO historical

Approach nhẹ hơn, 4-5 giả định thay vì 15+.

---

## 5. WACC / Cost of Equity — cách tính cho VN

### Cost of Equity (CAPM)

```
Cost of Equity = Rf + Beta × ERP
```

**Rf (Risk-Free Rate):** lợi suất trái phiếu chính phủ VN 10 năm.
- Query DB: `other_data` với `name: "TPCP VN 10 năm"`
- Hoặc web search: "Vietnam 10Y government bond yield" (tradingeconomics.com, macromicro.me)
- **Mức tham khảo (2026):** ~4.0-5.0% (hiện tại khoảng 4.36% per tradingeconomics.com tháng 4/2026)
- Note: Rf VN đã tăng từ 3-3.5% trong 2024 lên 4-5% trong 2026, phản ánh policy SBV hiện tại. Cập nhật trước khi dùng vì số này biến động 0.5-1% trong năm

**Beta:**
- Ideal: regression giá mã với VNINDEX 2-3 năm. Agent có thể tính từ `stock_recent` + `market_recent` (cần extract rộng hơn 20 phiên, có thể query history).
- Fallback nhanh: dùng beta ngành từ web search (Damodaran tables, FiinGroup) — VN SXKD thường beta 0.7-1.3, NGANHANG 0.9-1.2, CK 1.3-1.8, BH 0.6-0.9.
- Adjustment: bổ sung unlever rồi relever theo D/E của mã nếu leverage khác ngành

**ERP (Equity Risk Premium) cho VN:** 2 nguồn có methodology khác nhau, đều valid

**Nguồn 1 — Damodaran (NYU Stern) 2025:**
- Mature ERP (US baseline): 4.21-4.33%
- Country Risk Premium VN: 4.02% (dựa credit rating Ba2/Moody's + equity volatility scaling)
- **Total Vietnam ERP: ~8.35%** (tháng 1/2025 data, cập nhật đầu năm + mid-year)
- Web search: "Damodaran country risk premium Vietnam" hoặc pages.stern.nyu.edu/~adamodar

**Nguồn 2 — Gurufocus historical (methodology khác):**
- Historical VN ERP: median 11%, range 7.8-13.51%, 2023 value 11.13%
- Ghi nhận ERP VN thực tế cao hơn Damodaran do higher local volatility + currency risk

**Khuyến nghị:** dùng Damodaran 2025 (~8.35%) làm mặc định — chuẩn quốc tế, được nhiều quỹ tham khảo. Với mã rủi ro đặc thù (SmallCap, turnaround, liquidity thấp), bổ sung Company-Specific Risk Premium 1-3%.

**Tổng hợp Cost of Equity VN (2026):**
- Blue-chip (beta 0.8-1.0): 4.4% + 0.8-1.0 × 8.35% = 11-13%
- Mid-cap (beta 1.0-1.3): 13-15%
- Small/risky (beta 1.3-1.8): 15-19%

### Cost of Debt (sau thuế)

- Lãi suất debt thực tế = Interest Expense / Average Debt Balance (từ BCTC)
- Sau thuế = Cost of Debt × (1 − Tax Rate)
- Tax Rate VN: 20% (chuẩn DN SXKD), 17-20% tuỳ ưu đãi

**Ngưỡng tham khảo (2026):**
- Cost of Debt pre-tax: 6-8% cho DN tốt (blue-chip, investment grade), 9-12% cho DN rủi ro cao
- Cost of Debt after-tax: 4.8-6.4% (blue-chip), 7-10% (rủi ro cao)
- Web search: "lãi suất cho vay doanh nghiệp VN 2026" hoặc cite from BCTC mã cụ thể

### WACC

```
WACC = E/(D+E) × Cost of Equity + D/(D+E) × Cost of Debt × (1−Tax)
```

D/E lấy từ BCTC năm (tier 5A). Dùng **market D/E** (market cap of equity vs BV debt), không phải book D/E.

**Ngưỡng WACC VN (2026) theo segment:**

| Segment mã | Beta điển hình | D/E typical | WACC range |
|---|---|---|---|
| LargeCaps blue-chip (VN30, ROE ổn định) | 0.8-1.0 | 15-25% | 10-13% |
| MidCaps (quality tốt, growth ổn định) | 1.0-1.3 | 25-40% | 11-14% |
| SmallCaps (liquidity sát ngưỡng D, quality trung bình) | 1.3-1.8 | 30-50% | 13-17% |
| Mã catalyst play / turnaround rủi ro cao | 1.5-2.0+ | Variable | 15-20% |

**Benchmark thực tế từ các nguồn public:**
- Utility-scale solar VN (Payne Institute 2023): WACC 10-13.5% — confirm blue-chip range
- Valueinvesting.io estimates (tự động, không chuẩn): HPG WACC 7.7% (thấp, có thể dùng Beta quốc tế), VND WACC 13.2% (cao do Beta ngành CK)
- VIC GuruFocus 2023: 5.88% — flag lỗi dùng Beta default = 1 (cảnh báo nguồn auto-calc)

**Lưu ý:** các nguồn tự động (valueinvesting.io, GuruFocus) thường dùng Beta quốc tế hoặc default — không phù hợp cho VN context. Agent ưu tiên tự tính theo CAPM với data thực của mã (regression Beta từ stock_recent + market_recent, hoặc Beta ngành Damodaran adjusted).

### Terminal Growth Rate (g)

Perpetual growth — không vượt GDP growth dài hạn của VN (~5-6%, target 2030 double-digit dài hạn).

Chuẩn:
- **2-3%** cho mã mature, ngành không tăng trưởng (điện, nước, tiêu dùng thiết yếu)
- **3-4%** cho mã trung bình, SXKD bình thường
- **4-5%** cho mã ngành tăng trưởng cao (công nghệ, bán lẻ, banking), với justification rõ
- Không dùng g ≥ 5% trừ khi có thesis rất mạnh

**Rule cứng:** g < Cost of Equity. Nếu g ≥ WACC → terminal value = infinity, model broken.

---

## 6. Scenario — Base / Bull / Bear

Mọi mã cần 3 scenario, với giả định khác biệt rõ ràng trên 2-3 biến quan trọng nhất.

### Cấu trúc 3 scenario

**Base case (probability 50-60%):**
- Giả định "most likely" — tăng trưởng doanh thu, biên, CapEx ở mức phù hợp với consensus + catalyst đã priced-in
- WACC ở mức chuẩn theo beta hiện tại
- Terminal growth 3-4%

**Bull case (probability 20-30%):**
- Catalyst thesis realized: tăng trưởng cao hơn base 30-50%, biên mở rộng thêm 1-2%
- WACC thấp hơn chút (beta giảm do rủi ro giảm sau catalyst)
- Terminal growth tăng 0.5-1%

**Bear case (probability 15-25%):**
- Thesis fail: tăng trưởng bằng 0 hoặc âm nhẹ, biên co hẹp 1-2%
- Rủi ro tăng (beta tăng → WACC tăng 1-2%)
- Terminal growth giảm 0.5-1%

### Quy tắc khác biệt giả định

Mỗi scenario phải khác base ít nhất trên **2-3 biến quan trọng**:

| Biến | Base | Bull | Bear |
|---|---|---|---|
| Revenue CAGR 5Y | 12% | 18% | 5% |
| Gross Margin avg | 25% | 27% | 22% |
| WACC | 12% | 11% | 14% |
| Terminal g | 3% | 4% | 2% |

Target giá của 3 scenario sẽ chênh 30-60% là bình thường. Chênh quá nhỏ (< 20%) = 3 scenario gần giống nhau, không có ý nghĩa stress-test.

### Probability-weighted target

```
Expected target = P(base) × Target_base + P(bull) × Target_bull + P(bear) × Target_bear
```

Đây là 1 con số tổng hợp nhưng **không phải guideline entry**. Nguyên tắc:
- Entry khi price < Base target với margin of safety 15-25%
- Chốt partial khi price đạt Base
- Chốt thêm hoặc thoát khi price đạt Bull
- Stop loss khi price giảm sâu xuống dưới Bear

---

## 7. Sensitivity Analysis

### Bảng sensitivity 2 chiều

Chọn **2 biến nhạy cảm nhất** cho target giá — thường là:
- SXKD: WACC × Terminal growth, hoặc Revenue CAGR × Gross Margin
- NGANHANG: ROE sustainable × Cost of Equity
- CK: ROE mid-cycle × Cost of Equity
- BH: ROE normalized × Cost of Equity

Ví dụ cho SXKD:

| WACC \ Terminal g | 2% | 3% | 4% |
|---|---|---|---|
| 10% | 75,000 | 92,000 | 120,000 |
| 12% | 62,000 | 72,000 | 85,000 |
| 14% | 52,000 | 58,000 | 67,000 |

**Đọc sensitivity:**
- Range target qua bảng: 52k-120k (±40% từ center ~72k)
- Giả định most likely: WACC 12%, g 3% → 72k
- Nếu thị trường cho mã giá 90k → nằm trong "upper half" của bảng, cần giả định hơi optimistic (WACC 10% hoặc g ≥ 3.5%)

Sensitivity giúp user thấy **target giá không phải 1 điểm mà là 1 range**. Mỗi giá trị trong bảng correspond với 1 giả định. User có thể pick giá trị gần với niềm tin thesis nhất.

### Tornado chart (optional)

Cho mã High conviction, có thể bổ sung tornado chart (biểu đồ tornado — show độ nhạy của target theo từng biến, xếp từ biến tác động lớn nhất đến nhỏ nhất, hình thành pattern tương tự đám mây tornado) show impact của từng biến lên target:

| Biến | Change | Impact on Target |
|---|---|---|
| Revenue CAGR ± 3% | | ± 25% |
| Gross Margin ± 2% | | ± 18% |
| WACC ± 1% | | ± 15% |
| Terminal g ± 0.5% | | ± 10% |
| CapEx/Revenue ± 1% | | ± 5% |

Biến đứng đầu = biến quan trọng nhất — focus vào assumption của biến này khi thảo luận với user.

---

## 8. Cross-check 2 phương pháp

Sau khi có target từ Method chính (DCF / RIM / P/B-ROE) và Method cross-check (Peer Multiples / P/B-ROE / Sum-of-Parts), so sánh:

**Rule:**

| Chênh lệch target | Đánh giá |
|---|---|
| < 15% | Xanh — 2 method đồng thuận, target đáng tin |
| 15-30% | Vàng — chênh trong range, ghi nhận range thay vì point. Dùng median 2 method |
| > 30% | Đỏ — re-examine. Có giả định nào không hợp lý? |

**Khi 2 method chênh lớn:**

1. Kiểm tra Method chính trước — giả định tăng trưởng, biên, WACC có defensible không?
2. Kiểm tra Method cross-check — peer multiples có include peer đúng không? Median ngành có đại diện không?
3. Nếu cả 2 vẫn chênh: ghi rõ range target trong checkpoint, không ép chọn 1 con số

### Third method (optional, khuyến khích cho mã High)

Cho mã tier High conviction, bổ sung method 3:
- **Residual Income** (SXKD conglomerate) — nếu mã có ROE - WACC spread ổn định
- **Sum-of-Parts** (conglomerate VIC, VRE, HPG group) — định giá từng mảng riêng
- **DDM (Dividend Discount Model)** — cho mã trả cổ tức đều, mature
- **Implied multiples** — giải ngược từ giá hiện tại → giả định nào thị trường đang priced in, user judge có reasonable không

3 method = 3 góc nhìn độc lập. Nếu cả 3 cho target trong range ±15% → conviction rất cao. Nếu 1 method outlier → identify lý do.

---

## 9. Workflow per-stock — 8 bước

**Bước 1 — Load input tier 5A**

Đọc file `tier5A_<ticker>_YYYYMMDD_confirmed.md`. Extract:
- Red flag đã clear / vàng acceptable / notes quan trọng
- Key findings: giả định BCTC cần lưu ý (DSO trend, off-balance, cam kết mua nguyên liệu)
- Số liệu BCTC đã extract 3-5 năm

Nếu mã tier 5A là đỏ → không có tier 5B cho mã này, skip.

**Bước 2 — Chọn approach theo type**

Dựa vào `type` từ stock_finstats:
- SXKD → DCF FCFF + Peer Multiples
- NGANHANG → RIM + P/B-ROE
- CK → P/B-ROE + Sum-of-Parts
- BH → P/B-ROE + P/E

Prepare method chính + method cross-check.

**Bước 3 — Build historical + project 5 năm**

SXKD: full 3-statement projection (High conviction) hoặc simplified FCFF (Medium).

NGANHANG/CK/BH: project BVPS + ROE 5 năm thay vì 3-statement đầy đủ.

**Bước 4 — Tính WACC / Cost of Equity**

Theo Section 5:
- Rf từ DB hoặc web search
- Beta từ regression hoặc ngành
- ERP từ Damodaran
- Cost of Debt từ BCTC (nếu SXKD)
- Tax Rate VN

**Bước 5 — Compute 3 scenario**

Base / Bull / Bear với giả định khác biệt. Tính target giá mỗi scenario.

**Bước 6 — Cross-check với method 2**

Peer Multiples (SXKD) / P/B-ROE (NGANHANG/CK/BH) hoặc Sum-of-Parts. So target với method chính.

**Bước 7 — Sensitivity analysis**

Build bảng 2D cho 2 biến quan trọng nhất. Optional tornado chart cho High conviction.

**Bước 8 — Xuất checkpoint 5B**

Theo template Section 10. User challenge giả định trước khi finalize.

---

## 10. Template báo cáo Checkpoint 5B (per-stock)

```
# Checkpoint 5B — Valuation [Mã X] [ngày]

## 1. Summary quyết định
Target giá (Base / Bull / Bear): [X1k / X2k / X3k]
Phương pháp chính: [DCF FCFF / RIM / P/B-ROE / Embedded Value]
Method cross-check: [Peer Multiples / P/B-ROE / Sum-of-Parts]
Cross-check: chênh lệch 2 method [X%] — [Xanh/Vàng/Đỏ]

Giá hiện tại: [X0k] → Upside/Downside:
- Base: [+/-X%]
- Bull: [+/-X%]
- Bear: [+/-X%]

Decision: [Proceed to memo tier 5C / Target quá gần giá hiện tại, cân nhắc loại / Cần revise giả định]

## 2. Bối cảnh đầu vào
- Mã [X] type [Y], tier conviction: [High/Medium]
- Tier 5A decision: [Xanh/Vàng]
- Red flag vàng/notes từ tier 5A: [list cụ thể, ảnh hưởng giả định]
- Ngày build model: [ngày]
- Consensus sell-side (nếu có): [nguồn, target, tăng trưởng dự kiến]

## 3. Approach + historical base

### Method chính: [DCF FCFF / ...]

Historical 3-5 năm:
| Năm | Revenue (tỷ) | EBIT margin | FCFF (tỷ) | CFO/LNST |
|---|---|---|---|---|
| 2021 | 10,500 | 12.5% | 850 | 0.92 |
| 2022 | 12,800 | 13.8% | 1,100 | 0.95 |
| 2023 | 14,200 | 13.2% | 1,180 | 0.88 |
| 2024 | 16,500 | 14.1% | 1,450 | 0.90 |
| 2025 | 18,200 | 14.5% | 1,650 | 0.89 |

### Method cross-check: [Peer Multiples]

Peer ngành (median):
- P/E: 15x / EV/EBITDA: 9x / P/B: 1.8x
- Peer quốc tế (top 3): [...]

## 4. Giả định Base case

Driver:
- Revenue CAGR 5Y: 12% (từ segment: điện máy +8%, BHX +20%, AnKhang +30%)
- Gross Margin: ổn định 22% ± 0.5%
- EBIT margin: mở rộng từ 14.5% → 16% vào năm 5 (operating leverage)
- CapEx: 4% revenue (guidance từ BCTN 2024)
- Working Capital: DSO 48 → 52 (theo trend tier 5A), DIO 85 → 82, DPO 38 → 40

WACC: 10.9%
- Rf: 4.4% (TPCP VN 10Y, tháng 4/2026)
- Beta: 0.95 (regression 3Y)
- ERP: 8.35% (Damodaran VN 2025 total: mature 4.33% + country risk 4.02%)
- Cost of Equity: 4.4% + 0.95 × 8.35% = 12.3%
- Cost of Debt pre-tax: 7.5% (từ BCTC)
- Cost of Debt sau thuế: 7.5% × (1-0.2) = 6.0%
- Market D/E: 20/80 → WACC = 80% × 12.3% + 20% × 6.0% = 11.1%
- Rounded: 11.0%

Terminal g: 3.5% (mature retail)

## 5. 3 Scenario — Base / Bull / Bear

| Biến | Base (55%) | Bull (25%) | Bear (20%) |
|---|---|---|---|
| Revenue CAGR 5Y | 12% | 18% | 5% |
| EBIT margin năm 5 | 16% | 18% | 12% |
| WACC | 11.0% | 10.0% | 13.0% |
| Terminal g | 3.5% | 4% | 2.5% |
| **Target (k/cp)** | **72** | **105** | **48** |

Probability-weighted: 55% × 72 + 25% × 105 + 20% × 48 = **75k/cp**

Giả định đặc biệt:
- Bull case assume: BHX đạt break-even sớm hơn 1 năm + thị phần ICT tăng 2%
- Bear case assume: cost pressure nguyên liệu (dầu, nhựa) + chi phí logistics + cạnh tranh online

## 6. Cross-check với Peer Multiples

### Peer Multiples approach:

EBITDA năm 5 projection: 3,850 tỷ
EV/EBITDA target ngành: 9x (median)

Firm Value = 3,850 × 9 = 34,650 tỷ
Net Debt hiện tại: 4,200 tỷ
Equity Value = 34,650 - 4,200 = 30,450 tỷ
Target Price = 30,450 / 400M shares = **76,000 VND/cp**

### So sánh:

| Method | Base target | Chênh vs DCF |
|---|---|---|
| DCF FCFF | 72,000 | — |
| Peer Multiples | 76,000 | +5.5% |

**Cross-check: Xanh** — 2 method chênh < 15%, đồng thuận quanh 72-76k.

## 7. Sensitivity Analysis (Base case)

WACC × Terminal g:

| WACC \ g | 3.0% | 3.5% | 4.0% |
|---|---|---|---|
| 10.0% | 85k | 98k | 115k |
| 10.5% | 75k | 85k | 98k |
| 11.0% | 66k | 72k (base) | 82k |
| 11.5% | 60k | 64k | 70k |
| 12.0% | 55k | 58k | 62k |

Tornado (top 3 biến):
- Revenue CAGR ± 3% → target ± 22%
- Gross Margin ± 1% → target ± 15%
- WACC ± 0.5% → target ± 8%

## 8. Discussion — key assumptions cần user challenge

Điểm User cần review kỹ:
1. **Revenue CAGR 12% Base** — hợp lý không? Consensus sell-side hiện tại là 10-14%. Segment BHX +20% có optimistic quá không?
2. **EBIT margin mở rộng 14.5% → 16%** — dựa trên operating leverage. Nếu cạnh tranh online siết, có thể chỉ giữ 14.5% → target Base giảm 8-10%
3. **WACC 11.0%** — Beta 0.95 từ regression 3Y. Nếu dùng Beta ngành retail VN (1.1-1.2), WACC tăng lên 12% → target giảm 8-10%
4. **Terminal g 3.5%** — phù hợp retail mature. Nhưng BHX còn đang scale, có thể dùng g cao hơn cho phần này?

## 9. Lựa chọn sát nút

- Nếu user nghĩ consensus Bull quá optimistic → giảm Bull target xuống 90k, probability 20%
- Nếu user có thông tin về Capex plan lớn hơn BCTN công bố → Bear case weight tăng lên 30%
- User có thể pick different Beta nếu có lý do (ví dụ Beta ngành bán lẻ VN 1.1 trung bình)

## 10. Câu hỏi chờ user

Target giá (Base 72k / Bull 105k / Bear 48k) có reasonable không?
Hoặc muốn:
- (a) Challenge giả định cụ thể (user chỉ ra biến cần revise)
- (b) Thêm scenario 4 (ví dụ "mild bear" cho bear case moderate)
- (c) Build model 3 đầy đủ (RIM hoặc Sum-of-Parts) nếu cần cross-check 3 method
- (d) Loại mã khỏi shortlist nếu target không đủ upside so với price hiện tại (upside Base < 15% → không đủ margin of safety)

Nếu confirm → proceed to tier 5C memo cho mã này.
Target giá sẽ được dùng trong memo phần 4 (định giá) + phần 7 (exit trigger).
```

**Độ dài target:** 4-5 trang/mã tuỳ complexity.

---

## 11. Failure mode

### 11.1. Over-fitting giả định để hợp giá hiện tại

Agent có xu hướng adjust giả định ngược từ target giá "phải" hợp lý so với giá thị trường. Ví dụ price đang 80k, Agent set Revenue CAGR 15% + margin 17% để ra target 85k.

**Xử lý:** giả định phải defensible độc lập, không ngược từ target. Nếu target DCF = 50k trong khi price = 80k → ghi nhận "market priced in nhiều hơn giả định của mình", không adjust giả định để match.

### 11.2. Terminal growth quá cao

g ≥ 5% cho mọi mã. Điều này giả định mã tăng trưởng mãi mãi nhanh hơn GDP — không thực tế.

**Xử lý:** g < 5% trong mọi case. g = 4-5% chỉ cho mã ngành tăng trưởng cực cao (công nghệ, digital banking) với justification rõ. Mã mature SXKD: g = 2-3%. Check rule: g < WACC.

### 11.3. WACC không phù hợp với rủi ro thực

Dùng WACC 10% cho mã SmallCap có leverage 3x + cash flow bất ổn. WACC đó phù hợp blue-chip, không phù hợp rủi ro.

**Xử lý:** adjust WACC lên 15-17% cho SmallCap rủi ro cao. Beta từ regression 3-5 năm hoặc dùng industry beta với size adjustment (+1-2% cho SmallCap).

### 11.4. Scenario Bull không justify được

Bull case assume tăng trưởng 25% + margin 20% không có basis. Đây là wish thinking, không phải stress-test.

**Xử lý:** mỗi giả định Bull phải có trigger cụ thể (catalyst X realize, thị phần Y tăng với lý do, ngành tăng trưởng Z). Nếu không list được, giảm Bull target.

### 11.5. Bỏ qua tier 5A findings

Tier 5A cảnh báo DSO tăng trend, Agent vẫn project DSO ổn định. Giả định không nhất quán với forensic.

**Xử lý:** Bước 1 workflow bắt buộc đọc key findings tier 5A và reflect vào giả định. Nếu tier 5A note DSO trend xấu → Base case project DSO tăng thêm. Nếu tier 5A note off-balance commitments cao → Bear case weight cao hơn.

### 11.6. Peer multiples dùng peer không đúng

So EV/EBITDA của mã SXKD (ROE 25%) với median ngành (ROE 12%) → mã được đánh giá xứng đáng premium. Nhưng cross-check multiples không account cho sự khác biệt ROE.

**Xử lý:** khi peer multiples, segment peer theo tier chất lượng:
- So với peer cùng ROE tier (VD: ROE 20-25%)
- Hoặc dùng adjusted multiples (PEG thay vì P/E)
- Note rõ trong checkpoint: "Mã này ROE gấp đôi median → dùng P/E 1.3x median là fair, không phải 1x"

### 11.7. Cross-check 2 method chênh lớn nhưng không examine

DCF = 50k, Peer Multiples = 95k, chênh 90%. Agent ghi trung bình 72k và proceed.

**Xử lý:** chênh > 30% là Đỏ (Section 8). Bắt buộc examine:
- DCF giả định gì optimistic/pessimistic?
- Peer multiples có đúng peer không?
- Nếu không resolve, ghi nhận **range 50-95k** trong checkpoint thay vì 1 số, và report cho user quyết định.

### 11.8. Not quarantining bank/CK DCF

Agent cố chạy DCF FCFF cho bank dù method không phù hợp — FCFF của bank không defined standard do cấu trúc working capital đặc thù.

**Xử lý:** Bước 2 workflow bắt buộc match type-method:
- SXKD: DCF FCFF
- NGANHANG: RIM (không DCF FCFF)
- CK: P/B-ROE (không DCF FCFF)
- BH: P/B-ROE hoặc Embedded Value

### 11.9. Probability weighting gán random

Probability 55/25/20 cho base/bull/bear không có basis — Agent gán arbitrary.

**Xử lý:** weight phải dựa trên:
- Historical base rate (ngành thường có đạt plan không?)
- Catalyst strength (catalyst mạnh → Bull weight cao hơn)
- Macro context (Risk-on → Bull weight cao hơn, Defensive → Bear weight cao hơn)

Không ép đưa % nếu không có basis — ghi range thay vì điểm.

---

## 12. Đầu ra chuẩn để tier 5C dùng

Output tier 5B lưu file `tier5B_<ticker>_YYYYMMDD_confirmed.md` per-stock.

1. **Header:** mã, ngày, target Base/Bull/Bear, method, conviction
2. **Bảng 3 scenario** với giả định + target
3. **Bảng cross-check 2 method** với chênh lệch
4. **Bảng sensitivity 2D** cho biến quan trọng nhất
5. **Giả định key** — list 5-8 giả định lớn với lý do
6. **Điểm mà user challenge** — ghi nhận các giả định user đã review/adjust trong checkpoint
7. **Entry + Exit triggers** cho memo tier 5C phần 7:
   - Entry: price < Base × 0.85 (15% margin of safety)
   - Take profit 50%: price đạt Base
   - Take profit 80%: price đạt Bull
   - Stop loss: price < Bear × 0.9 (10% dưới Bear target)
8. **Flags cho tier 5C (memo):**
   - Giả định rủi ro nhất (input cho phần bear case memo)
   - Cross-check 2 method đồng thuận/chênh (input cho phần variant perception)
   - Sensitivity biến quan trọng nhất (input cho phần rủi ro)
9. **Link audit log** nếu user override giả định

File này enable vào project knowledge cho tier 5C session.
