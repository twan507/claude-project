# P_vbse_strategy_05 — Trục 5: Kịch bản & risk map

File chi tiết Trục 5 của khung 6 trục pack `P_vbse_strategy`. Phụ thuộc master `P_vbse_strategy_00` — đọc trước để nắm triết lý thiết kế, scope monthly/weekly, và **Weight balance** (mục 4 của master). Theo Weight balance: **Trục 5 thuộc tầng PRIMARY**, **technical cap = 0% (CẤM TUYỆT ĐỐI)** — trigger kịch bản phải là macro / fundamental / policy / catalyst. Technical chỉ được phép xuất hiện làm **confirmation phụ** (vd "thêm xác nhận: VNINDEX đóng cửa trên POC quý"), KHÔNG bao giờ làm primary trigger.

## 1. Mục tiêu & câu hỏi cốt lõi

Kịch bản nào có thể **đảo ngược** thesis tháng, signal **vĩ mô / cơ bản / chính sách / catalyst** nào báo hiệu, phản ứng định tính ra sao?

## 2. Rule trigger — BẤT BIẾN

**TRIGGER PHẢI LÀ MACRO/FUNDAMENTAL/POLICY/CATALYST, không phải break-out kỹ thuật.** Tuân **Weight balance mục 4 của `P_vbse_strategy_00`**. Vi phạm rule này = re-write trục 5 trước khi render (self-audit trước render: nếu trigger primary là technical → reject, đào sâu thêm fundamental driver).

Self-audit checklist trước khi xuất:
- Mỗi kịch bản (Cơ sở / Tích cực / Tiêu cực) có ít nhất 1 trigger macro/fundamental/policy/catalyst làm primary?
- Technical xuất hiện ≤1 lần per kịch bản và chỉ ở vị trí "confirmation phụ"?
- Risk map: mỗi rủi ro có gắn cơ chế cơ bản/vĩ mô/chính sách (không phải technical/flow đơn thuần)?

## 3. Three kịch bản if-then (không gán % xác suất)

- **Cơ sở:** môi trường vĩ mô + chính sách + earnings hiện tại tiếp diễn → thesis tháng vận hành đúng. Vùng VNINDEX/sector kỳ vọng.
- **Tích cực:** trigger **vĩ mô/chính sách/catalyst tích cực** cụ thể materialize → re-rating + theme củng cố. Ví dụ trigger:
  - "Fed cắt 25bp + ECB dovish tone tại FOMC tháng tới"
  - "BCTC Q1 ngành ngân hàng beat consensus ≥10% với NIM cải thiện ≥20bp"
  - "Nghị quyết gói hỗ trợ tài khoá X nghìn tỷ thông qua"
  - "FII chuyển sang mua ròng ≥X nghìn tỷ trong tháng sau khi đã bán ròng 3 tháng"
  - "Brent về dưới 70 USD/thùng kéo 1 tháng → biên gộp thép cải thiện"
  - Technical (vd VNINDEX đóng cửa trên POC quý) chỉ là **confirmation phụ**, KHÔNG phải primary trigger.
- **Tiêu cực:** trigger **vĩ mô/chính sách/catalyst tiêu cực** cụ thể materialize → de-rating + theme invalidate. Ví dụ trigger:
  - "Fed phát signal hawkish hơn dự kiến tại FOMC"
  - "BCTC Q1 nhiều mã top sector miss consensus"
  - "USD/VND vượt 26800 + NHNN can thiệp >2 tỷ USD"
  - "Chính sách thắt tín dụng BĐS mới ban hành"
  - "China stimulus bị delay quý sau"
  - Technical chỉ confirmation.

## 4. Risk map — 3-7 rủi ro (flex theo bối cảnh)

Mỗi rủi ro phải **gắn cơ chế cơ bản/vĩ mô/chính sách** (không phải technical/flow đơn thuần). 5 nhóm rủi ro:

- **Rủi ro vĩ mô:** lãi suất tăng ngoài kỳ vọng, tỷ giá shock, geopolitics, suy thoái Mỹ/EU, China hard-landing
- **Rủi ro chính sách:** thắt tín dụng đột ngột, chính sách thuế mới gây bất ngờ, đối thoại chính sách Mỹ-VN xấu đi
- **Rủi ro cơ bản:** mùa BCTC Q1 ngành Y miss consensus rộng, biên gộp ngành Z thu hẹp do commodity X
- **Rủi ro thanh khoản/flow (secondary):** FII bán ròng kéo dài 2+ tháng, margin call cấp sàn
- **Rủi ro thesis-specific:** theme bị "priced-in" sớm, catalyst chính sách delayed sang quý sau

**Mỗi rủi ro 4 thành phần bắt buộc:**
1. Bản chất + cơ chế cơ bản
2. Signal materialize cụ thể (PREFER macro/fundamental signal: vd "CPI YoY tháng N+1 vượt 5%", "EPS Q1 ngành thép giảm >15% YoY") — chấp nhận technical signal làm secondary
3. Phản ứng định tính (giảm exposure / chuyển defensive / đứng ngoài)
4. Theme bị invalidate nếu rủi ro materialize

**Whitelist scope cho sector-level risk:** khi rủi ro gắn ngành cụ thể (vd "biên gộp ngành thép thu hẹp"), chỉ tham chiếu trong scope **18 ngành whitelist** của `K_agent_db_01 Section B`. Ngành ngoài whitelist không xuất hiện trong risk map cấp pack.

## 5. Hướng tìm dữ liệu

| Nguồn | Mục đích | Ghi chú |
|---|---|---|
| `other_data.macro.*`, `other_data.fx.*` + web | Macro/policy trigger — CPI, lãi suất, FX shock; Fed minute, CPI release | PRIMARY cho trigger vĩ mô |
| `news_history_feed` + web | Policy trigger — Nghị quyết, QĐ bộ ngành, dự thảo luật | PRIMARY cho trigger chính sách |
| `industry_finstats.financial_statements.quarterly`, `other_data.commodities.*` | Fundamental trigger — BCTC miss, biên gộp thu hẹp do commodity | PRIMARY cho rủi ro cơ bản |
| `market_nntd` aggregate 2+ tháng | Flow trigger (secondary) — FII bán ròng kéo dài | SECONDARY, không quyết định độc lập |
| Cross-reference Trục 3 output + `industry_snapshot.change.m_pct` | Thesis-specific risk — theme bị priced-in, catalyst delay | Theme-by-theme map disconfirming signals |
| Web search | Forward macro lịch release, Fed/ECB/PBOC speech, geopolitics | Risk vốn forward-looking → web heavy |
| File user upload (optional) | Brief user nhấn mạnh / nội dung tháng cụ thể | Nếu có |

**Tỷ trọng nguồn (per master index):** ~50% DB + ~45% web + ~5% file user upload.

## 6. Output diễn giải

3 kịch bản if-then trigger (macro/fundamental/policy primary, technical confirmation only if absolutely needed) + risk map 3-7 rủi ro với 4 thành phần đầy đủ mỗi rủi ro. **Cross-link với theme từ Trục 3** — mỗi rủi ro ghi rõ nếu materialize sẽ invalidate theme nào trong Trục 3.

Output Trục 5 phục vụ Trục 6 (signal theo dõi + disconfirming watchlist) và weekly tracking (mỗi tuần re-check trigger nào đã materialize / shift).

## 7. Edge cases

- **Mâu thuẫn 2 kịch bản đồng materialize** (vd Fed dovish nhưng đồng thời BCTC ngành ngân hàng miss): present **cả 2** kịch bản với cơ chế riêng, **không pick winner**. Để user/PM tự assess net impact.
- **Rủi ro mới chưa có trong map cũ** (phát hiện trong tuần): flag explicit "rủi ro mới X xuất hiện, chưa nằm trong risk map tháng N". Đề xuất add weekly tracking signal cho rủi ro mới này; carry vào risk map tháng N+1.
- **Trigger đã materialize một phần** (vd Fed cắt 25bp nhưng ECB chưa dovish): note "kịch bản tích cực materialize 50%, đợi điều kiện thứ 2"; không tuyên bố full materialize cho đến khi đủ điều kiện. Render qua `O_vbse_strategy_00`.
