# P_vbse_strategy_01 — Trục 1: Môi trường vĩ mô & tài chính

File chi tiết Trục 1 của khung 6 trục pack `P_vbse_strategy`. Phụ thuộc master `P_vbse_strategy_00` — đọc trước để nắm triết lý thiết kế, scope monthly/weekly, naming output, và đặc biệt là **Weight balance** (mục 4 của master). Theo Weight balance: **Trục 1 thuộc tầng PRIMARY (~70-75%)** trong báo cáo monthly, **technical cap = 0%** ở trục này — môi trường vĩ mô/tài chính không có khái niệm "technical zone", toàn bộ signal phải là số macro thực, chính sách, hoặc giá hàng hoá thực.

## 1. Mục tiêu & câu hỏi cốt lõi

Môi trường lãi suất, thanh khoản, tỷ giá, dòng vốn đang **hỗ trợ** hay **siết** equity VN trong 1-3 tháng tới?

Output Trục 1 là input cốt lõi cho Trục 4 (sector sensitivity mapping — ngành nào hưởng lợi/thiệt do regime hiện tại) và Trục 5 (risk trigger — kịch bản đảo thesis dựa vào shift vĩ mô/chính sách).

## 2. Lăng kính phân tích

- **Chính sách tiền tệ trong nước:** lãi suất điều hành NHNN, OMO, tăng trưởng tín dụng, M2, dự trữ ngoại hối
- **Chính sách tiền tệ quốc tế:** Fed (FOMC dot plot, balance sheet), ECB, PBOC — kỳ vọng cắt/tăng
- **Tỷ giá USD/VND:** áp lực phá giá, can thiệp NHNN
- **Dòng vốn FII:** net flow EM tháng/quý, beta VN với MSCI EM
- **Vĩ mô thực:** CPI, GDP, PMI, XNK, FDI, bán lẻ
- **Hàng hoá có ảnh hưởng cross-sector:** dầu, kim loại, nông sản — chỉ những item đang biến động đáng kể

## 3. Hướng tìm dữ liệu

| Nguồn | Mục đích | Ghi chú |
|---|---|---|
| `other_data` group `macro.*` | CPI, GDP, PMI, XNK, FDI, bán lẻ — vĩ mô thực VN | PRIMARY cho lăng kính "vĩ mô thực" |
| `other_data` group `macro.fx.*` | USD/VND, dự trữ ngoại hối, can thiệp NHNN | PRIMARY cho lăng kính "tỷ giá" |
| `other_data` group `commodities.*` | Dầu, kim loại, nông sản — giá + xu hướng | PRIMARY cho lăng kính "hàng hoá cross-sector" |
| `other_data` group `international.*` | Số liệu Fed, ECB, PBOC, China data | PRIMARY cho lăng kính "chính sách tiền tệ quốc tế" |
| `market_nntd` | FII aggregate tháng/quý, beta VN với MSCI EM (qua xu hướng dài) | PRIMARY cho lăng kính "dòng vốn FII" |
| Web search **BẮT BUỘC** | FOMC minutes, Fed speech, ECB statement, China data release, lịch macro tháng/tuần tới (CPI, NFP, PMI Mỹ/EU/TQ/VN), Nghị quyết/QĐ NHNN chưa kịp vào DB | Forward-looking + cross-check release chậm |

**Tỷ trọng nguồn (per master index):** ~40% DB + ~60% web search. DB cung cấp số liệu lịch sử + xu hướng; web phục vụ forward catalyst, release mới, và xác minh tin lớn.

## 4. Output diễn giải

Kết luận **regime vĩ mô** — môi trường đang ở giai đoạn nào: early cycle nới lỏng / mid cycle ổn định / late cycle thắt chặt / shock. Không cần phân loại chính thức, dùng ngôn ngữ định tính phù hợp tháng đó.

Output Trục 1 feed sang:
- **Trục 4 sector allocation:** mapping ngành nhạy với lãi suất / tỷ giá / commodity / chính sách thuế (tham chiếu `K_agent_db_01 Section B` — whitelist 18 ngành) — regime quyết định ngành nào quan tâm/thận trọng
- **Trục 5 risk triggers:** shift vĩ mô/chính sách (vd "Fed phát signal hawkish hơn dự kiến", "USD/VND vượt 26800 + can thiệp >2 tỷ USD") là PRIMARY trigger cho 3 kịch bản if-then

## 5. Edge cases

- **Macro release chậm 2-3 tuần:** số liệu CPI/PMI/GDP VN thường công bố trễ. Khi data DB chưa cập nhật → dùng web search cross-check số mới nhất từ TCTK / NHNN / nguồn quốc tế. Ghi rõ trong báo cáo "số tháng N-1 đã chốt, tháng N tracking qua [nguồn]".
- **Commodity biến động đột ngột:** nếu dầu/thép/USD chuyển pha trong tuần báo cáo, đào sâu cơ chế cross-sector (qua Trục 4) thay vì chỉ note giá. Cross-link đến đâu trong chuỗi giá trị bị ảnh hưởng.
- **Geopolitics shock (chiến tranh, sanctions, election surprise):** flag riêng, không gộp vào "rủi ro vĩ mô" thường. Web search BẮT BUỘC để có thông tin mới nhất; impact thường bất đối xứng, không suy luận từ pattern lịch sử đơn thuần.
