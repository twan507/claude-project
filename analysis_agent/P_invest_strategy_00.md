# P_invest_strategy_00 — Master Workflow

Pack `P_invest_strategy` build báo cáo **chiến lược đầu tư** theo 2 chu kỳ lồng nhau: tháng (parent) và tuần (child). Khác biệt cốt lõi với `P_weekly_market` — pack này KHÔNG phải báo cáo tổng hợp thông tin thị trường, mà là báo cáo **định vị chiến lược**: thị trường VN đang ở đâu trong chu kỳ, theme nào đang chi phối, sector nào ưu tiên, kịch bản nào cần dự phòng, mã nào đại diện theme.

Pack 1 file. Toàn bộ workflow + khung tư duy + hướng tìm dữ liệu nằm ở file này. Render spec ở `O_invest_strategy_00`.

**Triết lý thiết kế:** khung tư duy tổng quát, structure flex theo phát hiện thực tế của tháng/tuần. Không ép 12 phần rigid. Agent được khuyến khích đào sâu trục có signal mạnh, lướt nhanh trục không có gì đặc biệt. Số lượng theme/sector/mã linh hoạt theo bối cảnh, không ép con số cứng.

## 1. Mục đích & scope

**Mục đích:** sinh báo cáo chiến lược đầu tư VN, horizon 1-3 tháng forward-looking, audience analyst nội bộ + có thể swap branding cho audience khách hàng. Mục tiêu cao nhất:

- Định vị thị trường VN trong chu kỳ vĩ mô + chu kỳ thanh khoản + chu kỳ định giá
- Chốt 2-5 themes & narratives chi phối tháng/quý tới
- Đề xuất sector allocation (ngành quan tâm / trung tính / cần thận trọng)
- Build risk framework đa kịch bản với trigger signal cụ thể
- Đưa high-conviction watchlist 5-12 mã đại diện theme (observation, không level giá)

**Quan hệ tháng ↔ tuần:**
- Báo cáo tháng = parent, chạy 1 lần đầu tháng (tuần đầu tiên). Hình thành thesis tổng + 6 trục đầy đủ.
- Báo cáo tuần = child, chạy mỗi tuần trong tháng. **Đọc báo cáo tháng đang active** → track signals đã đặt → flag shift / hold → cập nhật watchlist & risk map. Không build thesis từ đầu.
- Sau 4-5 tuần update, đến đầu tháng mới chạy lại monthly cycle (refresh full thesis).

**Input kỳ vọng:**

| Mode | Input bắt buộc | Input optional |
|---|---|---|
| Monthly | Trigger user "báo cáo chiến lược tháng" / "monthly strategy" / "outlook tháng N" + ngày kết thúc tháng N-1 hoặc đầu tháng N | Branding info, focus theme/sector user gợi ý, file báo cáo tháng N-1 (để review hit rate thesis cũ) |
| Weekly | Trigger user "update tuần [DD/MM]" / "weekly strategy update" + **file báo cáo tháng đang active** (user upload) | File báo cáo tuần W-1, sự kiện tuần qua user nhấn mạnh |

**Output kỳ vọng:** file MD theo structure flex của `O_invest_strategy_00`. Monthly target 8-12 trang, weekly update target 3-5 trang (gọn hơn vì là tracking).

**Negative scope:**

- Không phải báo cáo tổng hợp thông tin thị trường tuần — đó là job của `P_weekly_market`. Pack này không tracking giá ngành, breadth phiên, top biến động tuần.
- Không phải pipeline screen + deep-dive mã cụ thể cho portfolio — đó là job của `P_invest_memo`. Watchlist ở pack này dừng ở mức luận điểm theme (5-12 mã observation), không có entry/stop/target/size.
- Không phải pitch mã đơn lẻ gửi KH — đó là job của `P_stock_pitch`.
- Không khuyến nghị giao dịch ngắn hạn (intraday, T+) — horizon 1-3 tháng forward.
- Không gán % xác suất kịch bản (tuân `K_agent_db_00` mục 4.3).
- Không dùng từ command (mua/bán/giảm tỷ trọng/stop loss) — diễn đạt qua observation/luận điểm.
- **Không sử dụng chỉ báo trend nội bộ** (`*.trend`, `recent_trend`) nếu báo cáo render branded cho audience KH. Khi render plain (nội bộ) thì được dùng, nhưng phải dịch ra ngôn ngữ tự nhiên.

**Dependency:** `K_agent_db`. Đọc trước `K_agent_db_00` master, file con tra theo nhu cầu (chủ yếu `_01` schema, `_04` methodology, `_05` news methodology).

## 2. Naming & lưu trữ

**Naming file output:**
- Monthly: `invest_strategy_monthly_<YYYYMM>.md` — YYYYMM là tháng báo cáo (vd `invest_strategy_monthly_202605.md` cho tháng 5/2026)
- Weekly update: `invest_strategy_weekly_<YYYYMMDD>.md` — YYYYMMDD là ngày kết thúc tuần update

**Lưu trữ:** agent KHÔNG lưu file qua session. User tự archive. Mode weekly bắt buộc user upload file monthly đang active — pack không tự nhớ thesis tháng.

## 3. Khung tư duy — 6 trục cốt lõi

Đây là "spine" của báo cáo. Cả monthly và weekly đều dùng cùng 6 trục, khác ở độ sâu và mode tương tác:
- Monthly: đào đầy đủ 6 trục, build thesis từ đầu
- Weekly: đọc monthly → quét nhanh trục nào có signal shift → cập nhật trục đó, các trục khác ghi note "không đổi"

**Agent được phép merge / split / reorder trục theo phát hiện thực tế của tháng đó.** Khung này là gợi ý, không phải cấu trúc rigid 6 section.

### Weight balance — Trọng số 3 tầng phân tích theo horizon

Đây là rule quan trọng nhất của pack — quyết định signal nào driver chính, signal nào confirmation.

**Monthly horizon (1-3 tháng / 3-6 tháng) — driver chính phải dài hơi:**

| Tầng | Trọng số chỉ định | Loại signal | Lý do |
|---|---|---|---|
| **PRIMARY (~60-70%)** | Trục 1 + 3 + một phần Trục 4 | **Vĩ mô** (lãi suất / tỷ giá / FII flow / chu kỳ vĩ mô) + **Cơ bản** (BCTC, tăng trưởng EPS, ROE, biên, định giá vs lịch sử) + **Chính sách** (Nghị quyết / luật / quyết định bộ ngành / chính sách Fed-ECB-PBOC) + **Catalyst dài hơi** (M&A, niêm yết mới, upgrade FTSE/MSCI, mùa BCTC, dividend cycle, capacity online) | Đây là factor có time-to-play-out khớp horizon 1-3 tháng. Re-rating equity, sector rotation lớn, theme dominant tháng — đều dẫn dắt bởi nhóm này. |
| **SECONDARY (~20-25%)** | Trục 2 + một phần Trục 4 | **Định vị thị trường** (P/E phân vị lịch sử, dòng tiền tuần aggregate 4 tuần, NN tháng/quý, breadth tổng thể, sentiment proxy) | Chu kỳ tâm lý + định vị flow định hướng dòng tiền chung. Trung gian giữa fundamental dài hơi và technical ngắn hạn. |
| **TERTIARY (~10-15%)** | Một phần Trục 4 + Trục 5 confirmation | **Technical** (technical_zone đa khung, MA, Fibonacci, POC, breadth phiên, money_flow_score tuần đơn lẻ) | Chỉ làm **confirmation timing** cho thesis đã build từ fundamental. KHÔNG để technical làm primary trigger trong báo cáo tháng. |

**Hệ quả thiết kế cho báo cáo tháng:**
- Trục 5 kịch bản phải dùng trigger **vĩ mô / cơ bản / chính sách / catalyst** (vd "Fed cut 25bp + ECB dovish tone", "BCTC Q1 ngành ngân hàng beat consensus ≥10%", "Nghị quyết gói hỗ trợ tài khoá thông qua"). Technical chỉ confirmation phụ (vd "thêm xác nhận: VNINDEX đóng cửa trên POC quý").
- Trục 6 watchlist signal theo dõi + disconfirming phải gồm **earnings / catalyst / chính sách / định giá** trước, technical sau.
- Sector tilts disconfirming signal phải tham chiếu **cơ chế cơ bản** (vd "quặng sắt giảm >15% kéo 1 tháng → biên gộp Q2 ngành thép thu hẹp"), không chỉ technical/flow (vd "industry rank tụt").

**Weekly horizon (tracking 1 tuần) — technical/flow được phép weight cao hơn:**

| Tầng | Trọng số | Loại signal |
|---|---|---|
| PRIMARY (~50%) | Vĩ mô shift trong tuần (release CPI, FOMC, Fed speech, chính sách mới) + Catalyst materialize (BCTC release, M&A close, sự kiện trong tuần) |
| SECONDARY (~30%) | Định vị thị trường shift (dòng tiền tuần, NN tuần, breadth) |
| TERTIARY (~20%) | Technical (technical_zone shift đa khung, break/giữ vùng quan trọng) — weight cao hơn monthly vì tracking ngắn hạn cần signal nhanh |

Weekly được phép lean technical hơn vì mục đích là track shift ngắn hạn — nhưng anchor vẫn là thesis monthly (vĩ mô/cơ bản/chính sách driver chính). Nếu technical shift mà thesis vĩ mô + cơ bản không thay đổi → status Hold, technical noise tạm thời.

**Self-audit trọng số:** trước khi xuất báo cáo tháng, agent đếm — trong nội dung Trục 5 + 6 + sector tilts, % chỗ dùng signal vĩ mô/cơ bản/chính sách/catalyst so với % chỗ dùng signal technical/flow. Nếu technical > 30% trong báo cáo tháng → re-weight, đào sâu thêm fundamental.

### Trục 1 — Môi trường vĩ mô & tài chính

**Câu hỏi cốt lõi:** môi trường lãi suất, thanh khoản, tỷ giá, dòng vốn đang **hỗ trợ** hay **siết** equity VN trong 1-3 tháng tới?

**Lăng kính:**
- Chính sách tiền tệ trong nước: lãi suất điều hành NHNN, OMO, tăng trưởng tín dụng, M2, dự trữ ngoại hối
- Chính sách tiền tệ quốc tế: Fed (FOMC dot plot, balance sheet), ECB, PBOC — kỳ vọng cắt/tăng
- Tỷ giá USD/VND: áp lực phá giá, can thiệp NHNN
- Dòng vốn FII: net flow EM tháng/quý, beta VN với MSCI EM
- Vĩ mô thực: CPI, GDP, PMI, XNK, FDI, bán lẻ
- Hàng hoá có ảnh hưởng cross-sector: dầu, kim loại, nông sản — chỉ những item đang biến động đáng kể

**Output diễn giải:** kết luận **regime vĩ mô** — môi trường đang ở giai đoạn nào (early cycle nới lỏng / mid cycle ổn định / late cycle thắt chặt / shock). Không cần phân loại chính thức, dùng ngôn ngữ định tính phù hợp tháng đó.

### Trục 2 — Định vị thị trường VN trong chu kỳ

**Câu hỏi cốt lõi:** VNINDEX đang ở **đâu** trong chu kỳ giá / chu kỳ định giá / chu kỳ dòng tiền?

**Lăng kính:**
- **Định giá tổng thể:** P/E VNINDEX, P/B, P/E forward (nếu có) so với median 3-5 năm. Phân vị (percentile) đang ở đâu trong dải lịch sử.
- **Dòng tiền cấp thị trường:** xu hướng `industry_snapshot.money_flow_score.week_score` aggregate qua các tuần trong tháng (proxy thị trường — vì `market_snapshot` không có money_flow_score). Mean/median 24 ngành theo tuần, vẽ trend 4-8 tuần gần nhất.
- **Khối ngoại tháng/quý:** `market_nntd` aggregate, có break trend không (chuyển bán → mua, hoặc ngược lại)
- **Breadth tổng thể:** % ngành tăng giá tháng / quý, % mã trên MA60/MA120
- **Sentiment proxy:** thanh khoản trung bình tháng vs trung bình 6 tháng, volume profile các vùng quan trọng

**Output diễn giải:** thị trường đang ở giai đoạn nào — tích luỹ sau giảm / phục hồi sớm / uptrend khoẻ / quá mua cảnh báo / phân phối / suy yếu. Dùng ngôn ngữ định tính, có cơ sở số.

### Trục 3 — Themes & narratives chính

**Câu hỏi cốt lõi:** trong 1-3 tháng tới, **2-5 câu chuyện lớn** nào sẽ chi phối dòng tiền thị trường VN?

**Lăng kính:**
- **Chính sách trong nước:** dự thảo luật/nghị quyết/quyết định mới (vd cải cách thuế, sửa luật chứng khoán, gói hỗ trợ tài khoá), tiến độ đầu tư công, cải cách doanh nghiệp nhà nước
- **Sự kiện vĩ mô lớn sắp đến:** họp NHNN, FOMC, ECB; release CPI/GDP/PMI; bầu cử; geopolitics
- **Mùa BCTC:** đang trong/sắp vào mùa quý nào, sector nào kỳ vọng tăng trưởng nổi bật, sector nào áp lực
- **Tái cấu trúc ngành:** M&A, divestment, niêm yết mới, upgrade thị trường (FTSE/MSCI)
- **Catalyst commodity:** chu kỳ giá hàng hoá đang chuyển pha (vd dầu vào downcycle, thép vào upcycle), tác động chuỗi giá trị
- **Catalyst quốc tế:** chính sách Trump/Tập/EU tác động VN, supply chain shift, China stimulus

**Output diễn giải:** 2-5 themes có tên gọi rõ ràng (vd "Sóng đầu tư công Q2", "Margin ngân hàng cải thiện cuối chu kỳ hạ lãi suất", "Phục hồi xuất khẩu thuỷ sản theo USD/VND"). Mỗi theme bắt buộc đủ 5 thành phần:

1. **Cơ chế tác động** — mạch logic nguyên nhân → hệ quả (3-5 dòng)
2. **Conviction level — HIGH / MID / LOW:**
   - HIGH = cơ chế rõ + catalyst đã hoặc đang materialize + cross-check ≥2 trục khác đồng thuận
   - MID = cơ chế rõ + catalyst chưa rõ thời điểm, hoặc 1 trục khác chưa đồng thuận
   - LOW = early thesis, signals còn yếu, mang tính "watch list" theme
3. **Time horizon** — 1 tháng / 1-3 tháng / 3-6 tháng (theo timing catalyst materialize)
4. **Catalyst trigger** — sự kiện / mức số / chính sách cụ thể đã hoặc sắp xảy ra; nếu có ngày dự kiến, ghi ngày
5. **Disconfirming signals** — 2-3 chỉ báo cụ thể (reference data field) mà nếu xuất hiện sẽ invalidate theme. Vd "Industry rank ngành thép tụt khỏi top 8 trong 2 tuần liên tiếp", "USD/VND vượt 26500 + can thiệp NHNN", "Brent về dưới 65 USD/thùng kéo 1 tháng". Đây là "what would change our mind" — chuẩn institutional research.

### Trục 4 — Sector allocation strategy

**Câu hỏi cốt lõi:** với regime vĩ mô + định vị thị trường + themes chốt được, **ngành nào nên quan tâm**, ngành nào trung tính, ngành nào cần thận trọng tháng/quý tới?

**Lăng kính (theo Weight balance mục 3 — PRIMARY: vĩ mô/chính sách/cơ bản, SECONDARY: định vị/flow, TERTIARY: technical):**

**Lăng kính PRIMARY (đào sâu, là driver chính):**
- **Cross-check 3 trục trên:** ngành ở giao của (vĩ mô hỗ trợ + chính sách hỗ trợ + có theme + earnings outlook tích cực) → ứng viên quan tâm
- **Tăng trưởng cơ bản ngành:** `industry_finstats.financial_statements.quarterly` — tăng trưởng EPS / doanh thu / biên gộp Q gần nhất, xu hướng 4-8 quý, kỳ vọng Q tới (nếu data có hoặc web search consensus)
- **Định giá ngành tương đối lịch sử:** `industry_finstats.valuation_ratios` P/E + P/B median so với median 3-5 năm — phân vị hiện tại (vd P/E ở phân vị 30% = rẻ tương đối, phân vị 80% = đắt). Re-rating opportunity hay risk?
- **Chính sách / catalyst ngành cụ thể:** Nghị quyết / Luật / Quyết định bộ ngành mới (tham chiếu `news_history_feed` filter sector + web search), mùa BCTC quý nào sắp đến, dividend cycle, M&A pipeline, niêm yết mới
- **Sensitivity vĩ mô (cross với Trục 1):** ngành nhạy gì (lãi suất / tỷ giá / commodity / chính sách thuế) — tham chiếu `K_agent_db_01` mục F mapping ngành

**Lăng kính SECONDARY (cross-check, không quyết định độc lập):**
- Dòng tiền ngành: `industry_snapshot.money_flow_score.week_score` tuần gần + xu hướng 4 tuần, `industry_rank` — confirmation cho thesis cơ bản, không quyết định độc lập
- Breadth nội bộ ngành: `industry_snapshot.breadth` + count `stock_snapshot.change.m_pct > 0` — phân biệt dẫn dắt thật (đa số mã tăng) vs trụ kéo (vài mã lớn kéo)
- Crowding check: cross-reference `market_nntd` aggregate + `industry_snapshot.breadth` để biết ngành đang "consensus crowded" hay "contrarian". Crowded long → rủi ro thoái lui khi thesis lệch.

**Lăng kính TERTIARY (chỉ làm confirmation timing):**
- Technical zone đa khung của ngành: `industry_snapshot.technical_zone.overall` — confirmation cho timing entry chứ không quyết định bias. Một ngành có cơ bản tốt + định giá rẻ nhưng technical yếu vẫn quan tâm (entry timing khác), không loại.

**Output diễn giải:** phân tầng định tính (KHÔNG ép số ngành mỗi tầng):
- **Ngành quan tâm:** cross check cả 3 trục thuận lợi, có theme dẫn dắt rõ
- **Ngành trung tính:** signal hỗn hợp, không có theme, hoặc đang chuyển pha
- **Ngành cần thận trọng:** vĩ mô áp lực + định vị xấu + không có theme, hoặc định giá quá cao

Bias diễn đạt "quan tâm / thận trọng" — không dùng "overweight/underweight" để tương thích cả audience nội bộ và KH.

**Bắt buộc kèm bảng tilts tổng hợp** (single-page scannable, chuẩn buy-side):

Mỗi ngành 1 dòng với: tên ngành | bias | conviction (HIGH/MID/LOW) | theme/driver chính | signal hỗ trợ (1 dòng kèm số) | disconfirming signal (1 dòng cụ thể). Render chi tiết ở `O_invest_strategy_00`.

### Trục 5 — Risk scenarios & contingency

**Câu hỏi cốt lõi:** kịch bản nào có thể **đảo ngược** thesis tháng, signal **vĩ mô / cơ bản / chính sách / catalyst** nào báo hiệu, phản ứng định tính ra sao?

**Lăng kính — TRIGGER PHẢI LÀ MACRO/FUNDAMENTAL/POLICY/CATALYST, không phải break-out kỹ thuật** (theo Weight balance mục 3):

- **3 kịch bản** (không gán % xác suất):
  - **Cơ sở:** môi trường vĩ mô + chính sách + earnings hiện tại tiếp diễn → thesis tháng vận hành đúng. Vùng VNINDEX/sector kỳ vọng.
  - **Tích cực:** trigger **vĩ mô/chính sách/catalyst tích cực** cụ thể materialize → re-rating + theme củng cố. Ví dụ trigger: "Fed cắt 25bp + ECB dovish tone tại FOMC tháng tới" / "BCTC Q1 ngành ngân hàng beat consensus ≥10% với NIM cải thiện ≥20bp" / "Nghị quyết gói hỗ trợ tài khoá X nghìn tỷ thông qua" / "FII chuyển sang mua ròng ≥X nghìn tỷ trong tháng sau khi đã bán ròng 3 tháng" / "Brent về dưới 70 USD/thùng kéo 1 tháng → biên gộp thép cải thiện". Technical (vd VNINDEX đóng cửa trên POC quý) chỉ là **confirmation phụ**, KHÔNG phải primary trigger.
  - **Tiêu cực:** trigger **vĩ mô/chính sách/catalyst tiêu cực** cụ thể materialize → de-rating + theme invalidate. Ví dụ trigger: "Fed phát signal hawkish hơn dự kiến tại FOMC" / "BCTC Q1 nhiều mã top sector miss consensus" / "USD/VND vượt 26800 + NHNN can thiệp >2 tỷ USD" / "Chính sách thắt tín dụng BĐS mới ban hành" / "China stimulus bị delay quý sau". Technical chỉ confirmation.

- **Risk map** 3-7 rủi ro (flex theo bối cảnh), mỗi rủi ro phải **gắn cơ chế cơ bản/vĩ mô/chính sách** (không phải technical/flow đơn thuần):
  - **Rủi ro vĩ mô:** lãi suất tăng ngoài kỳ vọng, tỷ giá shock, geopolitics, suy thoái Mỹ/EU, China hard-landing
  - **Rủi ro chính sách:** thắt tín dụng đột ngột, chính sách thuế mới gây bất ngờ, đối thoại chính sách Mỹ-VN xấu đi
  - **Rủi ro cơ bản:** mùa BCTC Q1 ngành Y miss consensus rộng, biên gộp ngành Z thu hẹp do commodity X
  - **Rủi ro thanh khoản/flow (secondary):** FII bán ròng kéo dài 2+ tháng, margin call cấp sàn
  - **Rủi ro thesis-specific:** theme bị "priced-in" sớm, catalyst chính sách delayed sang quý sau

- **Mỗi rủi ro 4 thành phần:** (1) bản chất + cơ chế cơ bản, (2) signal materialize cụ thể (PREFER macro/fundamental signal: vd "CPI YoY tháng N+1 vượt 5%", "EPS Q1 ngành thép giảm >15% YoY") — chấp nhận technical signal làm secondary, (3) phản ứng định tính (giảm exposure / chuyển defensive / đứng ngoài), (4) theme bị invalidate nếu rủi ro materialize.

**Output diễn giải:** 3 kịch bản if-then trigger (macro/fundamental/policy primary, technical confirmation) + risk map. Phục vụ trục 6 + weekly tracking.

### Trục 6 — High-conviction watchlist (theme play)

**Câu hỏi cốt lõi:** 5-12 mã đại diện cho themes & sector bias đã chốt, **observation** từng mã ra sao?

**Lăng kính:**
- Mỗi theme/sector quan tâm: 1-3 mã đại diện
- Mã phải có thanh khoản tối thiểu (≥ 5 tỷ/phiên trung bình tháng để loại nhiễu penny)
- Cross-check: định vị kỹ thuật (technical_zone w/m không quá yếu), dòng tiền ngành rank top, định giá không quá cao bất thường

**Output diễn giải mỗi mã** (bắt buộc 6 thành phần — chuẩn buy-side watchlist):

1. **Ticker (ngành) — theme đại diện**
2. **Conviction:** HIGH / MID / LOW (cùng định nghĩa Trục 3 theme — HIGH cần cross-check ≥2 trục đồng thuận + có catalyst cơ bản/chính sách rõ + định giá hợp lý + dòng tiền không ngược chiều)
3. **Horizon:** 1m / 1-3m / 3-6m (theo timing catalyst materialize của theme)
4. **Luận điểm** (1-2 câu, observation — không command): cơ chế theme → lý do mã hưởng lợi cụ thể về **cơ bản** (tăng trưởng / biên / chính sách hỗ trợ / catalyst), không chỉ technical
5. **Signal theo dõi** (3-5 chỉ báo cụ thể — **PREFER cơ bản / catalyst / chính sách / định giá**, technical là phụ):
   - **Cơ bản (must-have ≥1):** vd "BCTC Q1 EPS growth ≥ 20% YoY", "Biên gộp Q1 cải thiện ≥ 50bp QoQ", "ROE TTM mở rộng từ 15% lên ≥18%", "Doanh thu Q1 vượt consensus ≥ 8%"
   - **Catalyst / chính sách (must-have ≥1):** vd "Dự án X capacity Y MW online Q2", "Nghị quyết về sector Z thông qua tháng 5", "M&A close công ty con tháng 6", "Ngày chốt cổ tức tiền mặt 7%", "Earnings call ngày DD/MM"
   - **Định giá (recommended):** vd "P/E forward về 9x vs median 5Y là 13x", "P/B 1.1x vs ngành 1.6x"
   - **Định vị/flow (secondary):** vd "Dòng tiền tuần duy trì dương ≥ 3/4 tuần", "FII mua ròng tháng"
   - **Technical (tertiary, confirmation only):** vd "Vùng kỹ thuật khung tháng giữ A trở lên" — chỉ làm confirmation thêm, không phải primary
6. **Disconfirming signal** (1-2 dòng cụ thể — **PREFER cơ bản / catalyst / chính sách**): vd "BCTC Q1 EPS growth < 5% hoặc miss consensus ≥ 10%", "Biên gộp Q1 thu hẹp ≥ 100bp QoQ", "Dự án X delay sang Q4", "Chính sách Y bị huỷ bỏ". Technical disconfirming chỉ phụ (vd "kèm vùng kỹ thuật khung tháng tụt khỏi A").

**Bắt buộc kèm metadata cuối mỗi entry:** **ADV tháng** (average daily value — `stock_recent.series[0..19].price.trading_value` mean) — bắt buộc hiển thị để user biết liquidity tier (vd "ADV 28 tỷ" = mid-cap liquid; "ADV 6 tỷ" = small-cap sát ngưỡng filter).

**KHÔNG có:** entry zone, stop, target giá, kích thước vị thế, ưu tiên thứ tự. Đây là watchlist quan sát chiến lược, không phải lệnh giao dịch ngắn hạn.

## 4. Hướng tìm dữ liệu

Pack không liệt kê field-by-field như `P_weekly_market`. Thay vào đó cho danh mục collection + ý nghĩa, agent tự quyết query nào cần cho trục nào.

**MongoDB `agent_db` — nguồn chính:**

| Collection | Dùng cho trục nào | Ghi chú |
|---|---|---|
| `market_snapshot` | Trục 2 (định vị) — giá VNINDEX, biến động đa khung, technical đa khung | Không có `money_flow_score` cấp thị trường, dùng aggregate từ industry |
| `market_recent` | Trục 2 — chuỗi giá + volume 20 phiên | Phân tích vận động trong tháng |
| `industry_snapshot` (24 doc) | Trục 2 (proxy thị trường) + Trục 4 | Aggregate cho proxy dòng tiền thị trường; cross-section cho ngành |
| `industry_recent` | Trục 4 — xu hướng dòng tiền ngành 4-8 tuần | Slice deeper hơn vs weekly_market |
| `industry_finstats` | Trục 4 — định giá ngành so lịch sử | P/E/P/B median + phân vị |
| `group_snapshot` (6 nhóm vốn hoá / theme nhóm) | Trục 4 — chéo ngành theo nhóm vốn hoá | Phát hiện rotation large/mid/small cap |
| `stock_snapshot` | Trục 6 — screen mã đại diện theme | Filter theo ngành + dòng tiền + thanh khoản + technical zone |
| `stock_finstats` | Trục 6 — định giá + tăng trưởng mã cụ thể | Dùng khi cần check EPS growth, ROE, margin trends mã trong watchlist |
| `stock_nntd` | Trục 6 — flow NN/TD mã | Cross-check thesis flow |
| `market_nntd` | Trục 2 — FII xu hướng tháng/quý | Aggregate dài hạn hơn |
| `other_data` | Trục 1 — vĩ mô + commodity + quốc tế | Filter group `macro.*`, `commodities.*`, `international.*` |
| `news_history_feed` | Trục 3 — themes & catalyst | Rolling 30 ngày, filter `news_type` |
| `news_history_content` | Trục 3 — đọc sâu tin quan trọng | Khi cần verify cơ chế tác động |
| `data_briefing` | Trục 2 — breadth tổng thị trường | Cross-check breadth_in/out |

**Web search — bổ sung bắt buộc khi:**

- Tin tức quốc tế ảnh hưởng VN (FOMC minutes, Fed speech, ECB statement, China data, geopolitics, commodities news)
- Chính sách VN mới công bố (Nghị quyết, dự thảo luật, quyết định bộ ngành) chưa có trong DB
- Macro release lịch tháng/tuần tới (CPI, NFP, PMI Mỹ/EU/TQ/VN)
- Benchmark ngành quốc tế khi cần so VN (vd P/E banking sector EM)
- Xác minh tin có trong DB nhưng có biến động lớn (vd theo thấy báo X có tin Y, web search cross-check)

**Khoảng tự do của agent:**

- Quyết định trục nào đào sâu / lướt nhanh dựa trên phát hiện thực tế tháng đó
- Tự quyết bao nhiêu theme, bao nhiêu sector mỗi tầng, bao nhiêu mã watchlist (trong khoảng gợi ý)
- Tự đề xuất visualization phù hợp khi cần (bảng/biểu đồ time-series qua chart annotation YAML — xem `O_invest_strategy_00`)
- Có thể merge 2 trục nếu thấy ranh giới mờ trong tháng đó (vd vĩ mô + themes hợp nhất nếu cả tháng dominated bởi 1 chính sách lớn)

## 5. Workflow Monthly

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
  Compile MD theo O_invest_strategy_00 (mode monthly)
  Save invest_strategy_monthly_<YYYYMM>.md
  Xuất trong message
```

**Lưu ý:** DB `agent_db` KHÔNG có collection storage cho báo cáo chiến lược cũ (theo schema `K_agent_db_01`). Vì vậy "đọc lại strategy cũ" phụ thuộc 100% vào **file user upload** trong session. Agent có thể đọc file MD user đã save trước đó (vd `invest_strategy_monthly_202604.md`). DB chỉ dùng để query **actual data** cross-check với thesis cũ (giá, dòng tiền, BCTC, vĩ mô).

### 5.1. Pre-flight monthly

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

   *Quan điểm user sẽ được agent ghép nối theo phương pháp ở mục 7 (User overlay). Báo cáo cuối sẽ ghi rõ phần nào do agent phát hiện, phần nào do user inject, kèm cross-check của agent.*

5. Branding & disclaimer (báo cáo có thể dùng cả nội bộ và gửi khách hàng):
   (a) Có — vui lòng cung cấp: tên công ty, logo, hotline, website, phòng ban biên soạn, custom disclaimer (nếu có)
   (b) Không cần — render bản plain nội bộ

6. Time horizon ưu tiên:
   (a) 1 tháng (focus catalyst sắp đến)
   (b) 1-3 tháng (default — outlook trung hạn)
   (c) 3-6 tháng (focus chu kỳ + theme dài hơi)
```

### 5.2. Stage 0 — Đánh giá chiến lược tháng N-1 (OPTIONAL)

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
- Watchlist 5-12 mã (luận điểm + signal theo dõi + disconfirming signal + ADV tại thời điểm N-1)

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

**Output Stage 0 trong báo cáo cuối:** mục "Review tháng trước" render đầy đủ 6 phần eval (xem render spec `O_invest_strategy_00`). Nếu user skip Stage 0 entirely (pre-flight câu 2 chọn b), Review section render ngắn (hit rate + best/worst call) như format cũ.

### 5.3. Stage 1 — Build thesis (Trục 1-3)

Agent compose lần lượt trục 1 → 2 → 3. Mỗi trục đào theo lăng kính ở mục 3, độ sâu flex theo phát hiện.

**Quy tắc đào sâu vs lướt nhanh:**
- Trục có biến động đáng kể (regime vĩ mô đang chuyển pha, định vị thị trường vào vùng cực đoan, theme mới nổi mạnh) → đào sâu, có thể chia sub-section
- Trục ổn định, không có gì mới → 3-5 dòng kết luận đủ, không ép viết dài

**Carry-forward từ Stage 0 (nếu có):** Stage 1 phải tính đến learning từ Stage 0 — không lặp lại pitfall đã identify, đào sâu themes/risks carry-forward, refine calibration conviction cho themes mới dựa trên track record N-1.

### 5.4. Checkpoint 1 — Regime vĩ mô + Top themes

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

### 5.5. Stage 2 — Allocation & risk (Trục 4-5)

Compose trục 4 (sector allocation) cross-check với 3 trục đã chốt + trục 5 (kịch bản & risk map).

**Sector allocation flow:**
1. List 24 ngành — đọc bảng cross-section (dòng tiền rank, biến động tháng, định giá)
2. Highlight ngành ở giao của (vĩ mô thuận lợi + định vị thuận lợi + có theme) → ứng viên quan tâm
3. Highlight ngành ngược lại (vĩ mô áp lực + định vị xấu + không theme) → ứng viên thận trọng
4. Kiểm tra dẫn dắt thật vs trụ kéo (đếm % mã trong ngành tăng giá tháng) — quy tắc giống `P_weekly_market` mục 6.3 (đa số mã tăng = dẫn dắt thật, vài mã lớn kéo = trụ kéo, gần 50/50 = rotation nội bộ)
5. Phân tầng cuối: quan tâm / trung tính / cần thận trọng (số ngành flex theo regime — environment thuận lợi rộng thì 4-6 ngành quan tâm, môi trường khó thì 2-3 ngành defensive)

**Risk framework flow:**
1. 3 kịch bản if-then VNINDEX (cơ sở / tích cực / tiêu cực) — dùng trigger kỹ thuật + flow + sự kiện. Không gán % xác suất.
2. Risk map 3-7 rủi ro, mỗi rủi ro: bản chất → signal materialize cụ thể → phản ứng định tính
3. Cross-link rủi ro với theme (nếu rủi ro X materialize → theme Y bị invalidate)

### 5.6. Stage 3 — Watchlist + Executive summary

**Watchlist:**

Cho mỗi ngành quan tâm + theme đại diện, screen mã:
- `stock_snapshot` filter industry, sort theo combo `money_flow_score.week_score` + `technical_zone.overall.w/m` ∈ (A, AA, AAA) + thanh khoản ≥ 5 tỷ/phiên trung bình tháng
- Top 1-3 mã/ngành, total 5-12 mã

Mỗi mã 2-4 dòng theo spec trục 6 mục 3.

**Executive summary** (viết cuối, sau khi đã có 6 trục):

3-6 bullet, mỗi bullet 1-2 dòng:
- Regime vĩ mô + định vị thị trường (1 dòng tổng)
- Top 2-3 themes
- Sector bias (quan tâm + thận trọng, ngắn gọn)
- 1-2 risk chính
- Watchlist 1-3 mã tiêu biểu nhất (optional)

### 5.7. Render monthly

Gọi `O_invest_strategy_00` render mode monthly. File `invest_strategy_monthly_<YYYYMM>.md`. Self-audit (mục 8) trước khi xuất.

## 6. Workflow Weekly Update

Workflow 2 stage (Stage 0 eval W-1 optional + Stage 1 tracking), có HARD GATE pre-flight: phải xác định tuần thứ mấy của tháng nào + có monthly active không. Không có monthly active → refuse + redirect sang monthly cycle.

```
─── Pre-flight (HARD GATE) ──────────────────────────
  Bước 1  Agent compute: hôm nay DD/MM/YYYY → tuần thứ [N] của tháng [M/YYYY]
  Bước 2  Hỏi user: có monthly active cho tháng [M/YYYY] chưa?
          - Có → upload file → tiếp Bước 3
          - Không → REFUSE, đề xuất chạy monthly cycle trước (xem mục 6.1 case "no monthly")
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
  Compile MD ngắn theo O_invest_strategy_00 (mode weekly update)
  Save invest_strategy_weekly_<YYYYMMDD>.md
```

### 6.1. Pre-flight weekly — HARD GATE

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
   (a) Có — tôi gửi đính kèm file invest_strategy_monthly_<YYYYMM>.md
   (b) Chưa có / chưa chạy monthly cycle cho tháng [M/YYYY]
   (c) Tôi có monthly nhưng của tháng khác (vd vẫn dùng tháng M-1 vì tháng M chưa kịp build)
```

**Xử lý theo case:**

| User chọn | Action |
|---|---|
| (a) Có file đúng tháng | Tiếp Bước 3, chạy bình thường |
| (b) Chưa có monthly | **REFUSE weekly mode**. Reply: "Để chạy weekly update tháng [M/YYYY] cần có thesis monthly làm parent. Đề xuất 3 hướng: (i) Chạy monthly cycle cho tháng [M/YYYY] trước, sau đó weekly update — recommended. (ii) Nếu chỉ cần tracking 1 tuần ad-hoc không gắn với thesis tháng, gợi ý dùng `P_weekly_market` (báo cáo tổng hợp thông tin tuần thị trường). (iii) Override — vẫn muốn chạy weekly độc lập, không có parent thesis, không recommended vì sẽ thiếu context để track shift/materialize." Hỏi user chọn hướng nào. |
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

   *Quan điểm user sẽ được agent xử lý theo mục 7 (User overlay).*
```

### 6.2. Stage 0 — Đánh giá tuần W-1 (OPTIONAL)

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

**Output Stage 0 trong báo cáo cuối:** mục "Review tuần W-1" render đầy đủ 4 phần (xem render spec O pack). Nếu user skip Stage 0, không render mục này — đi thẳng vào tóm tắt tuần [N].

### 6.3. Stage 1 — Tracking & Update

**Bước 1 — Extract từ monthly:**

Đọc file monthly user upload, extract:
- Regime vĩ mô đã call
- 2-5 themes đã chốt (tên + cơ chế)
- Sector bias (quan tâm / trung tính / thận trọng)
- 3 kịch bản VNINDEX (trigger từng kịch bản)
- Risk map (3-7 rủi ro + signal materialize)
- Watchlist (5-12 mã + signals theo dõi từng mã)

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

**Bước 5 — Action item tuần tới (1-2 item định tính):**

Vd:
- "Theo dõi FOMC minutes thứ Tư, signal cho theme 'Margin ngân hàng cải thiện cuối chu kỳ hạ lãi suất'"
- "Quan sát phản ứng nhóm thép sau release CPI Trung Quốc, có thể trigger shift sector bias"

**KHÔNG có:** entry/exit cụ thể, level giá, kích thước vị thế, %.

### 6.4. Render weekly update

Gọi `O_invest_strategy_00` render mode weekly. Target 3-5 trang MD (ngắn hơn monthly nhiều vì là tracking, không build từ đầu). File `invest_strategy_weekly_<YYYYMMDD>.md`.

## 7. User overlay handling — Ghép nối quan điểm user vào báo cáo

Báo cáo chiến lược là sản phẩm hợp tác analyst + PM. Pack cho phép user inject view ở **3 channel** + có methodology chuẩn để agent xử lý.

### 7.1. Three channel injection

1. **Pre-flight injection** (mục 5.1 câu 3 monthly / mục 6.1 câu 3 weekly) — user nêu view từ đầu, agent build báo cáo có tính đến view này
2. **Mid-flow injection** — user có thể interrupt bất cứ lúc nào trong session để add view. Agent dừng turn hiện tại, tiếp nhận, xử lý qua matrix mục 7.2, rồi tiếp tục từ điểm dừng
3. **Checkpoint injection** (mục 5.4 monthly Checkpoint 1 regime + themes; mục 5.2 monthly Checkpoint 0 eval; mục 6.2 weekly Checkpoint 0 eval) — user override / refine / bổ sung tại checkpoint

### 7.2. Synthesis matrix — agent xử lý view user thế nào

Mỗi view user inject, agent chạy 3 bước:

**Bước 1 — Parse view:** xác định view thuộc trục nào (1-6) và bản chất gì (factual claim, predictive thesis, risk concern, theme suggestion, ticker idea).

**Bước 2 — Cross-check với data agent_db + web:** agent query data hiện có để verify view.

**Bước 3 — Phân loại + xử lý theo matrix:**

| Trạng thái | Tình huống | Cách integrate vào báo cáo |
|---|---|---|
| **Confirm** | Data agent_db hoặc web search xác nhận view user | Integrate view vào trục liên quan như evidence chính. Render trong báo cáo kèm badge `[Synthesized from PM input + data confirm]` và cite data field. |
| **Partial confirm** | Một phần view có data ủng hộ, một phần không | Render cả 2 phần: phần data confirm → integrate; phần không có data → ghi rõ "PM view, data chưa confirm — cần monitor [signal cụ thể]". Badge `[Mixed: PM input + partial data]`. |
| **Conflict** | Data có sẵn ngược chiều view user | KHÔNG silently override. Present cả 2 view trong trục liên quan: "PM view: [X]. Agent finding from data: [Y]. Tension chưa resolve, đề xuất user quyết." Badge `[Conflict: PM view ↔ data]`. Đưa vào checkpoint nếu là monthly + chưa qua checkpoint. |
| **No data** | Không có data để verify (vd "tôi nghe tin nội bộ về M&A") | Ghi nhận như standalone "PM flag" — không treat như fact đã verify. Trong báo cáo: render trong trục liên quan với badge `[PM flag — chưa có data verify]`. Vẫn integrate vào risk map hoặc themes "watching" tier (LOW conviction). |
| **Out of scope** | View không thuộc 6 trục (vd "tôi muốn add panel mới về crypto") | Báo user view ngoài scope pack hiện tại, đề xuất ghi nhận như note phụ chân báo cáo, hoặc gợi ý pack/workflow khác phù hợp hơn. |

### 7.3. Audit trail trong báo cáo cuối

Pack bắt buộc render audit trail cho user contribution:

- **Trong nội dung báo cáo:** mỗi điểm tích hợp view user có inline badge (như mục 7.2)
- **Trong metadata cuối:** mục "User overlay log" liệt kê tất cả view user đã inject + trạng thái xử lý (Confirm / Partial / Conflict / Flag / Out of scope)
- **Trong executive summary:** nếu có view user ở conviction HIGH hoặc Conflict chưa resolve, mention 1 dòng để leadership đọc

### 7.4. Quy tắc bất di bất dịch

- **Không silently override agent finding bằng user view.** Cả 2 phải xuất hiện trong báo cáo có thể truy được.
- **Không treat "PM flag" (no data) như fact verified.** Phải có badge phân biệt.
- **Không bỏ qua user view vì agent thấy không hợp lý.** Nếu disagree, present tension, để user quyết.
- **Không edit retroactively báo cáo đã render** khi user inject view muộn — render version mới với log thay đổi, hoặc note "addendum" cuối file.

## 8. Self-audit trước khi xuất file

Chạy checklist trước khi render:

### Monthly mode
1. 6 trục đầy đủ (trục lướt nhanh vẫn có 3-5 dòng kết luận)?
1b. Nếu user chọn Stage 0 (eval N-1): có checkpoint 0 đã user phản hồi? Eval block đủ 6 phần (Regime/Themes/Sectors/Watchlist/Risk/Calibration)? Carry-forward đã feed vào Stage 1?
1c. **Weight balance** (theo mục 3): trong nội dung Trục 5 + 6 + sector tilts, signal **vĩ mô/cơ bản/chính sách/catalyst** chiếm ≥ 65% tổng signal sử dụng? Technical/flow ≤ 30%? Nếu technical > 30% → re-weight, đào sâu thêm fundamental trước khi xuất.
2. Checkpoint 1 regime + themes đã có user phản hồi (confirm/override)?
3. Sector bias có cross-check 3 trục đầu (vĩ mô + định vị + theme)?
4. Trục 3 — mỗi theme có đủ 5 thành phần (cơ chế / conviction / horizon / catalyst / disconfirming signals)?
5. Trục 4 — có bảng sector tilts tổng hợp với conviction + driver + disconfirming signal mỗi ngành?
6. Trục 5 — 3 kịch bản dùng if-then trigger, **không có % xác suất**? Risk map mỗi rủi ro đủ 3-4 thành phần (bản chất / signal / phản ứng định tính / theme bị invalidate)?
7. Trục 6 — watchlist 5-12 mã, mỗi mã đủ 6 thành phần (ticker-ngành-theme / conviction / horizon / luận điểm / signal theo dõi / disconfirming signal) + ADV tháng; không có level giá vào/ra/stop?
8. Review N-1 (nếu có) đã có best call / worst call honesty?
9. **User overlay** (nếu có view inject): mỗi view có trạng thái xử lý rõ (Confirm / Partial / Conflict / Flag / Out of scope)? Audit trail trong metadata đầy đủ?
10. K hygiene: ký hiệu DB raw đã dịch, taxonomy nội bộ không lộ ra?
11. Số liệu đã quy đổi đơn vị (tỷ đồng, % thập phân nhân 100)?
12. Mỗi claim định lượng có nguồn (collection + field / URL web search)?
13. Tin có dẫn link finext.vn hoặc URL gốc?
14. Branding info đã render đúng (custom / default branded / plain)?
15. Disclaimer có forward-looking statement language?
16. Executive summary 3-6 bullet đứng đầu, đọc 30 giây hiểu toàn báo cáo? Mention user overlay HIGH-conviction hoặc Conflict (nếu có)?

### Weekly update mode
1. Pre-flight HARD GATE đã pass: agent compute đúng tuần [N] tháng [M/YYYY]? User đã confirm có monthly active đúng tháng (hoặc đã chọn override path với note)?
1b. Header báo cáo có ghi rõ "tuần [N] của tháng [M/YYYY]" và link tham chiếu monthly?
1c. Nếu user chọn Stage 0 (eval W-1): checkpoint 0 đã user phản hồi? Eval block đủ 4 phần (Status carry-over / Watchlist W-1 tracking / Action item W-1 / Carry-forward)?
2. Đã đọc file monthly active user upload? Extract đủ 6 trục?
3. Mỗi trục có 1 trong 3 status rõ (Hold / Shift / Materialize)?
4. Watchlist refresh có phân loại 4 trạng thái (Hold / Watch closely / Out / Vào mới)? Mã "Vào mới" có đủ 6 thành phần monthly format + ADV?
5. Action item tuần tới định tính, không có command/level giá?
6. **User overlay** (nếu có view inject tuần): mỗi view có trạng thái xử lý + audit trail?
7. K hygiene + nguồn đầy đủ?
8. Độ dài 3-5 trang — không phình ra mode monthly?

Vi phạm câu nào sửa rồi mới render.

## 9. Edge cases & xử lý

### 9.1. Monthly mode

- **Thiếu dữ liệu trục 1 (vĩ mô):** chỉ số vĩ mô tháng có thể release chậm 2-3 tuần. Dùng số gần nhất, ghi rõ ngày cập nhật. Web search bổ sung nếu DB lỗi thời.
- **Tháng "không có gì đặc biệt" (không có theme rõ):** ghi explicit "Tháng N không có theme dominant. Chiến lược: chờ tín hiệu mới, sector bias trung tính, watchlist defensive." Không bịa theme.
- **Conflict regime vĩ mô vs định vị thị trường:** vd vĩ mô tích cực nhưng định vị thị trường quá mua → ghi rõ tension, cho cả 2 view, để checkpoint user quyết.
- **Tháng đầu cycle (không có file N-1):** skip Stage 0 eval + skip Review section, ghi 1 dòng "Lần đầu chạy monthly cycle, chưa có dữ liệu review tháng trước".
- **Stage 0 eval phát hiện thesis cũ sai rõ:** không cố biện minh. Trong eval, ghi worst call honest + pitfall identified. Carry-forward learning: tháng N tránh lặp pitfall đó, có thể nâng/giảm conviction methodology tương ứng.
- **File N-1 user upload không parse được (corrupt MD / sai format):** báo user file lỗi, hỏi có muốn skip Stage 0 không. Không tự đoán nội dung file.

### 9.2. Weekly update mode

- **HARD GATE — không có monthly active:** REFUSE chạy weekly, đề xuất 3 path (monthly trước / dùng P_weekly_market / override). Tuyệt đối không silently chạy weekly độc lập — pack không build weekly không có parent thesis.
- **HARD GATE — monthly active của tháng khác:** flag rõ, present 2 path. Nếu user chọn override với monthly cũ, header báo cáo cuối có note "Thesis carry-over từ tháng M-1, đã decay [X] tuần, conviction tổng thể giảm 1 bậc so monthly gốc".
- **Tuần 1 của tháng (vừa chạy monthly):** không có W-1 → tự động skip Stage 0 eval. Weekly chạy nhẹ vì thesis tươi.
- **Stage 0 eval phát hiện trục có shift lớn ở W-1 nhưng tuần [N] không tiếp diễn:** ghi rõ "shift W-1 đã reverse, không carry-forward; quay lại baseline monthly".
- **Tuần "không có shift":** vẫn render report ngắn 2-3 trang, ghi rõ tất cả trục Hold, watchlist Hold, action item "tiếp tục theo dõi signals đã đặt". Không bịa shift.
- **Materialize risk lớn (≥2 rủi ro materialize trong cùng tuần):** flag rõ ở đầu báo cáo + Tóm tắt tuần, đề xuất user xem xét chạy lại monthly cycle giữa kỳ (mid-month refresh).
- **Mâu thuẫn weekly update vs monthly (≥3 trục materialize/shift mạnh):** không tự "viết lại" monthly. Báo user: "Nhiều shift lớn tuần này, đề xuất chạy lại monthly cycle thay vì tiếp tục weekly update".
- **File W-1 user upload không parse được:** báo user, hỏi có skip Stage 0 không.

### 9.3. Trùng pack

- Có thể chạy song song với `P_invest_memo` và `P_weekly_market`. Independent, không share state. Watchlist của pack này khác bản chất với portfolio của invest_memo (theme play vs deep-dive picks).

## 10. Output contract

Pack sinh structured content cho `O_invest_strategy_00` render. Ràng buộc:

**Monthly mode:**
- 6 trục đầy đủ (kể cả trục Hold không có gì đặc biệt vẫn có 3-5 dòng kết luận)
- **Weight balance:** Trục 5 kịch bản + Trục 6 watchlist + sector tilts dùng signal vĩ mô/cơ bản/chính sách/catalyst làm PRIMARY (≥65%); technical/flow chỉ confirmation phụ (≤30%). Technical chiếm > 30% → vi phạm contract, re-weight trước khi xuất
- Stage 0 evaluation (nếu user chọn chạy): có checkpoint 0 user phản hồi, eval block đủ 6 phần, carry-forward feed vào Stage 1
- Checkpoint 1 regime + themes có user phản hồi trước khi vào Stage 2
- Trục 3 themes — mỗi theme đủ 5 thành phần (cơ chế / conviction HIGH-MID-LOW / horizon 1m-1to3m-3to6m / catalyst trigger / disconfirming signals)
- Trục 4 sector — bảng tilts tổng hợp với conviction + driver + disconfirming signal mỗi ngành
- Trục 5 — 3 kịch bản if-then trigger, không % xác suất; risk map 3-7 rủi ro với theme invalidation cross-link
- Trục 6 watchlist 5-12 mã — mỗi mã 6 thành phần (ticker-ngành-theme / conviction / horizon / luận điểm / signal theo dõi / disconfirming) + ADV tháng; observation only
- Review N-1 (nếu có) có best call / worst call
- **User overlay** (nếu có): mỗi view có badge trạng thái xử lý + integrate đúng matrix; metadata có User overlay log
- Executive summary viết cuối, mention user overlay HIGH-conviction hoặc Conflict nếu có
- Số liệu đã quy đổi, ký hiệu đã dịch (K hygiene)
- Mỗi claim định lượng có nguồn
- Disclaimer có forward-looking statement language

**Weekly update mode:**
- **HARD GATE pre-flight:** agent compute tuần [N] tháng [M/YYYY] + check user có monthly active đúng tháng. Không có monthly → REFUSE, không silently chạy weekly độc lập
- Header báo cáo ghi rõ "tuần [N] của tháng [M/YYYY]" + link tham chiếu file monthly
- File monthly active user upload là input bắt buộc (trừ override path đã được agreed với note rõ)
- Stage 0 evaluation W-1 (nếu user chọn chạy): checkpoint 0 user phản hồi, eval block đủ 4 phần
- 6 trục mỗi trục có status Hold/Shift/Materialize
- Watchlist refresh 4 trạng thái; mã Vào mới có đủ 6 thành phần monthly format + ADV
- 1-2 action item định tính
- **User overlay** (nếu có) tương tự monthly
- Độ dài 3-5 trang
- K hygiene + nguồn đầy đủ

Pack KHÔNG tự quyết heading style / xưng hô / tone / length cuối — `O_invest_strategy_00` quyết.
