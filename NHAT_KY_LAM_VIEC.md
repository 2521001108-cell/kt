# NHẬT KÝ LÀM VIỆC DỰ ÁN TIỂU LUẬN — NHÓM 6

* **Cơ sở đào tạo:** Trường Đại học Tài chính - Marketing (UFM)
* **Khoa:** Khoa học Dữ liệu
* **Học phần:** Lập trình Python cho Khoa học Dữ liệu (Mã lớp học phần: 2631101190501)
* **Tên đề tài tiểu luận:** Xây dựng mô hình dự báo chi phí bồi thường bảo hiểm
* **Giảng viên hướng dẫn:** ThS. Trần Mạnh Tường
* **Nhóm sinh viên thực hiện (Nhóm 6):**
  1. **Phan Thị Quế Trân** (MSSV: 2521001108) — *Trưởng nhóm*
  2. **Nguyễn Ngọc Phương Thảo** (MSSV: 2521001099) — *Thành viên*
* **Môi trường & Công cụ phát triển:** Python 3.11+, Quarto CLI, Microsoft Edge Headless, Antigravity IDE
* **Quy chuẩn lưu trữ nhật ký:** Thứ tự thời gian đảo ngược (Reverse Chronological Order) — Phiên làm việc mới nhất luôn nằm ở trên cùng ngay sau phần Header.

---

## PHIÊN 7: KIỂM TOÁN CHẤT LƯỢNG DỮ LIỆU, CÔ LẬP DỊ THƯỜNG SỐ HỌC VÀ LÀM SẠCH TẬP NGOẠI TRÚ

* **Thời gian ghi nhận:** `[2026-11-19 01:50]`
* **Mục tiêu phiên:** 
  1. Phân tích sự cần thiết của cột phân loại `ClaimType` và tối ưu bộ nhớ RAM cho tập dữ liệu Ngoại trú.
  2. Rà soát, kiểm toán toàn diện các dạng dữ liệu bẩn (khuyết thiếu, chuỗi giả lập, trùng lặp, sai định dạng, khoảng trắng, chữ hoa/thường lẫn lộn, ngoại lai IQR).
  3. Kiểm toán dị thường số học (giá trị số âm trong cột tiền tệ bồi thường và khấu trừ), làm rõ nguyên nhân nghiệp vụ bảo hiểm y tế.
  4. Lập trình và thực thi đường ống làm sạch dữ liệu chuẩn mực, cô lập hồ sơ dị thường và xuất bản các tập dữ liệu sạch phục vụ mô hình hóa.
* **Nội dung đã hoàn thành:**
  1. **Làm rõ cấu trúc biến phân loại `ClaimType`:**
     - Đánh giá mã nguồn tiền xử lý `pd.concat` và tối ưu hóa bộ nhớ.
     - Khẳng định: Nếu chỉ phân tích đơn lẻ tệp Ngoại trú (`PRJ04 Outpatient`), việc bổ sung `ClaimType = 'Outpatient'` và `Is_Inpatient = 0` là dư thừa (biến hằng số có phương sai bằng 0, không có giá trị phân biệt trong mô hình). Các cột này chỉ bắt buộc khi ghép nối hai tệp Ngoại trú và Nội trú.
  2. **Kiểm toán chất lượng dữ liệu bẩn trên 517.737 dòng Ngoại trú:**
     - **Trùng lặp (Duplicates):** 0 dòng trùng lặp 100% trên 27 cột thuộc tính. Tình trạng trùng lặp `ClaimID` (181.712 lượt) thực chất do các chuỗi khuyết giả lập bị lặp lại; toàn bộ mã `CLM...` chuẩn là duy nhất 100%.
     - **Nhãn không nhất quán & Khoảng trắng:** 25/27 cột bị dính khoảng trắng đầu/cuối; 106.143 dòng `BeneID` viết thường (20,50%), 121.476 dòng `ClaimID` viết thường (23,46%), 119.602 dòng `Provider` viết thường (23,10%).
     - **Lỗi định dạng đơn vị tiền tệ:** 32.361 dòng `InscClaimAmtReimbursed` và 40.312 dòng `DeductibleAmtPaid` dính chuỗi đơn vị (`USD`, `kg`, dấu phẩy).
     - **Dữ liệu khuyết và chuỗi giả lập:** Phát hiện hàng chục nghìn giá trị dạng `'?'`, `'-'`, `'unknown'`, `'None'`, `'N/A'`, ô trống tại các cột mã bệnh, mã bác sĩ và mã định danh.
  3. **Kiểm toán dị thường số học (Chi phí bồi thường và khấu trừ mang giá trị âm):**
     - `InscClaimAmtReimbursed < 0`: 7.344 dòng (1,42%), giá trị nhỏ nhất: -15.900,00 USD.
     - `DeductibleAmtPaid < 0`: 4.578 dòng (0,88%), giá trị nhỏ nhất: -200,00 USD.
     - Luận giải nghiệp vụ bảo hiểm y tế: Giá trị âm đại diện cho các khoản thu hồi bồi thường (clawback/retroactive void), hoàn tiền hủy hồ sơ sau thanh tra (claim cancellation), hoặc sai sót hạch toán. Do hàm mất mát Log-likelihood của các mô hình định phí (GLM Gamma, Tweedie) yêu cầu biến mục tiêu $y \ge 0$, toàn bộ 7.344 hồ sơ này được cô lập sang tập kiểm toán riêng để không làm biến dạng mô hình.
  4. **Kiểm toán giá trị ngoại lai (IQR Outliers):**
     - $Q_1 = 40,00$ USD; Trung vị $Q_2 = 80,00$ USD; $Q_3 = 300,00$ USD; $IQR = 260,00$ USD.
     - Ngưỡng ngoại lai trên ($Q_3 + 1,5 \times IQR$): 690,00 USD.
     - Ghi nhận 62.457 hồ sơ ngoại lai (12,86%), khoản bồi hoàn lớn nhất đạt 10.300.000,00 USD.
     - Quyết định phương pháp luận: Bảo toàn toàn bộ các hồ sơ ngoại lai chi phí cao vì đây là đặc tính phân phối đuôi dày (fat tail) thực tế trong y tế; xử lý bằng log-transform và mô hình phân phối Tweedie ở giai đoạn sau.
  5. **Thực thi script làm sạch và xuất bản dữ liệu:**
     - Xây dựng hoàn chỉnh script [kiem_toan_va_lam_sach_du_lieu.py](file:///c:/Users/Admin/Downloads/Aa/kiem_toan_va_lam_sach_du_lieu.py).
     - Áp dụng Regex `([-+]?\d*\.?\d+)` phục hồi chính xác giá trị số thuần từ các chuỗi lẫn đơn vị đo lường.
     - Chuẩn hóa viết hoa toàn diện và làm sạch khoảng trắng thừa bằng `.str.strip()`.
     - Xuất tệp dữ liệu sạch hợp lệ: [df_outpatient_clean.csv](file:///c:/Users/Admin/Downloads/Aa/df_outpatient_clean.csv) gồm **485.776 dòng $\ge 0$** (thu hồi thành công 93,83% dữ liệu ban đầu).
     - Xuất tệp dị thường số âm phục vụ thanh tra: [df_negative_claims.csv](file:///c:/Users/Admin/Downloads/Aa/df_negative_claims.csv) gồm **7.344 dòng**.
     - Xuất báo cáo tổng hợp: [bao_cao_kiem_toan_lam_sach.csv](file:///c:/Users/Admin/Downloads/Aa/bao_cao_kiem_toan_lam_sach.csv).
* **Các tệp liên kết cập nhật:**
  - Script kiểm toán và làm sạch: [kiem_toan_va_lam_sach_du_lieu.py](file:///c:/Users/Admin/Downloads/Aa/kiem_toan_va_lam_sach_du_lieu.py)
  - Tập dữ liệu sạch hợp lệ: [df_outpatient_clean.csv](file:///c:/Users/Admin/Downloads/Aa/df_outpatient_clean.csv)
  - Tập hồ sơ dị thường số âm: [df_negative_claims.csv](file:///c:/Users/Admin/Downloads/Aa/df_negative_claims.csv)
  - Báo cáo kiểm toán tổng hợp: [bao_cao_kiem_toan_lam_sach.csv](file:///c:/Users/Admin/Downloads/Aa/bao_cao_kiem_toan_lam_sach.csv)
* **Kế hoạch tiếp nối cho phiên kế tiếp:**
  - Triển khai **Kỹ thuật trích xuất đặc trưng (Feature Engineering)** trên tập dữ liệu sạch `df_outpatient_clean.csv`: tính toán thời lượng khám (`ClaimDuration`), tần suất khám và lũy kế chi phí theo bệnh nhân (`BeneID`) và cơ sở y tế (`Provider`), đếm số mã bệnh và mã thủ thuật, trích xuất đặc trưng thời gian (tháng, quý, ngày trong tuần).
  - Tích hợp số liệu kiểm toán và làm sạch vào Chương 3 của báo cáo tiểu luận (`tiểu luận.qmd`).

---

## PHIÊN 6: TÍCH HỢP BÁO CÁO PHÂN TÍCH THỰC NGHIỆM VÀ TỪ ĐIỂN 27 BIẾN VÀO TIỂU LUẬN

* **Thời gian ghi nhận:** `[2026-11-19 01:15]`
* **Mục tiêu phiên:** 
  - Xây dựng chương trình Python phân tích toàn diện tệp Ngoại trú (`06_2521001108_PRJ04_Phan Thị Quế Trân (1).csv`) tại `phan_tich_du_lieu_ngoai_tru.py`.
  - Kiểm chứng 4 nội dung cốt lõi: Nguồn gốc dữ liệu CMS DE-SynPUF, Quy trình phát sinh dữ liệu (từ khám lâm sàng đến bồi thường), Bản chất đơn vị quan sát (Claim-level, quan hệ 1-N), và Từ điển 27 biến.
  - Chép và tích hợp toàn bộ nội dung báo cáo hoàn chỉnh vào Mục 1.4 của tệp `tiểu luận.qmd`.
* **Nội dung đã hoàn thành:**
  1. **Nội dung phân tích thực nghiệm:**
     - Nguồn gốc dữ liệu CMS Medicare DE-SynPUF, tỷ lệ tiền tố chuẩn BENE (79,45%), CLM (75,98%), PRV (76,12%), PHY (92,77%), khung thời gian 12/2008 - 12/2009, và tuân thủ Non-PHI theo HIPAA.
     - Quy trình phát sinh dữ liệu: thời lượng khám ngoại trú (87,54% hoàn tất trong ngày), phân công bác sĩ (98,17% có bác sĩ chính), mã hóa bệnh án ICD-9/ICD-9-CM, và quyết toán tài chính.
     - Đơn vị quan sát: Mỗi dòng là một đợt yêu cầu bồi thường (`ClaimID`), không phải cấp độ bệnh nhân (140.646 người, trung bình 2,58 hồ sơ/người), phục vụ tại 8.831 cơ sở y tế.
     - Xây dựng Bảng 1.3 - Từ điển dữ liệu và hệ thống phân loại 27 biến thuộc tính Ngoại trú.
  2. **Khôi phục trạng thái ban đầu:**
     - Theo yêu cầu của người dùng, đã xóa phần nội dung nháp tại Mục 1.4 và khôi phục nguyên bản nội dung chuẩn ban đầu (dung lượng 53.725 bytes) cho toàn bộ các tệp `tiểu luận.qmd` và `tieu_luan.qmd`.
* **Các tệp liên kết:**
  - Script phân tích ngoại trú: [phan_tich_du_lieu_ngoai_tru.py](file:///c:/Users/Admin/Downloads/Aa/phan_tich_du_lieu_ngoai_tru.py)
  - Tệp nguồn Quarto chính thức: [tiểu luận.qmd](file:///c:/Users/Admin/Downloads/Aa/kt/ti%E1%BB%83u%20lu%E1%BA%ADn.qmd)

---

## PHIÊN 5: HOÀN THIỆN ĐỒNG BỘ 11 VẤN ĐỀ HỌC THUẬT & PHƯƠNG PHÁP LUẬN GIỮA CHƯƠNG 1 VÀ CHƯƠNG 2

* **Thời gian ghi nhận:** `[2026-11-17 13:40]`
* **Mục tiêu phiên:** Tiếp thu, hiệu chỉnh toàn diện 11 vấn đề học thuật phát sinh do nhóm rà soát; đồng bộ hóa tuyệt đối danh sách mô hình, đặc trưng y tế, thuật ngữ và chỉ số đánh giá giữa Chương 1 (`tiểu luận.qmd`) và Chương 2 (`06_2521001099_PRJ04_Chuong2.qmd`); biên dịch tái xuất bản ấn phẩm PDF chuẩn A4 đúng 16 trang.
* **Nội dung đã hoàn thành:**
  1. **Đồng bộ mã thủ thuật và loại bỏ biến không có trong dữ liệu:** Thay thế toàn bộ "HCPCS" thành mã thủ thuật can thiệp lâm sàng "ICD-9-CM" (trong Thách thức 3, RQ1, Mục tiêu 2, Mục 1.5.1 và Danh mục chữ viết tắt). Loại bỏ các đề cập đến "độ tuổi / nhân khẩu học" (do tệp PRJ04 không chứa thông tin nhân khẩu học; sửa "nhân khẩu học tương đương" ở Trụ cột 3 thành "đặc điểm điều trị và bệnh lý tương đương").
  2. **Giải quyết triệt để mâu thuẫn "dữ liệu thực tế" vs "dữ liệu tổng hợp":** Thay thế cụm từ "Big Data với hơn 520.000 dòng" tại Mục 1.1.2 thành "bài toán phân tích dữ liệu quy mô lớn"; sửa Mục 1.2.1 thành "bộ dữ liệu bồi thường được mô phỏng phục vụ nghiên cứu"; tại Mục 1.4 trích dẫn rõ căn cứ từ đề bài của giảng viên phụ trách học phần, khẳng định là dữ liệu tổng hợp (Synthetic Data) và bỏ chữ "hoàn toàn".
  3. **Tách bạch bản chất "thanh toán sai" (Improper Payments) và "gian lận" (Fraud):** Làm rõ tại Trụ cột 3 (Mục 1.1.1) theo đúng báo cáo thanh tra của CMS và HHS-OIG rằng phần lớn thanh toán sai xuất phát từ thiếu chứng từ y tế hợp lệ hoặc sai sót mã hóa hành chính, phân biệt rõ với các hành vi gian lận trục lợi có chủ đích (kê khống viện phí, Upcoding).
  4. **Chuẩn hóa khái niệm "Số bản ghi hợp lệ ban đầu" và bảo toàn dữ liệu dính đơn vị:** Tại Bảng 1.1 và Mục 1.4, chuẩn hóa tên gọi thành "Số bản ghi hợp lệ ban đầu (thuần số ≥ 0, trước khi xử lý đơn vị)" gồm 452.487 ngoại trú và 6.982 nội trú; nêu rõ các dòng dính đơn vị (33.289 ngoại trú và 453 nội trú) sẽ được bóc tách phục hồi bằng biểu thức chính quy (Regex) và tính toán lại quy mô mẫu đầy đủ ở Chương 3.
  5. **Đính chính số lượng biến độc lập từ 57 xuống 30 biến:** Phân tích cấu trúc dữ liệu cho thấy 27 cột thuộc tính của tệp ngoại trú là tập con của 30 cột nội trú (nội trú chỉ có thêm 3 cột: `AdmissionDt`, `DischargeDt`, `DiagnosisGroupCode`). Đã đính chính tại Mục tiêu 1 (Mục 1.3.2) và Bảng 1.2 thành "30 biến thuộc tính khác nhau".
  6. **Đồng bộ hóa đặc trưng y tế thực tế với Chương 2:** Thay thế "tỷ lệ trùng khớp chức danh bác sĩ" tại Mục tiêu 2 bằng hệ thống đặc trưng thực tế của Chương 2 gồm: thời gian nằm viện nội trú `LengthOfStay` (LOS), thời gian đợt bồi thường `ClaimDuration`, cờ bác sĩ phẫu thuật/hỗ trợ, tháng phát sinh hồ sơ, `Diagnosis_Count` và `Procedure_Count`.
  7. **Thống nhất hệ thống 4 chỉ số đo lường hiệu năng:** Bổ sung chỉ số MAPE (Mean Absolute Percentage Error - Sai số phần trăm tuyệt đối trung bình) vào Mục 1.3.2, Bảng 1.2 và Danh mục chữ viết tắt, tạo nên bộ 4 chỉ số chuẩn: RMSE, MAE, MAPE, R².
  8. **Khóa danh sách chuẩn 4 mô hình và đồng bộ Chương 2:** Giữ vững 4 mô hình tại Chương 1 (OLS logarit kết hợp Smearing, GLM Gamma & Tweedie, Random Forest, XGBoost). Hiệu chỉnh đồng bộ Chương 2 (`06_2521001099_PRJ04_Chuong2.qmd`), loại bỏ toàn bộ các mục Hồi quy Ridge/Lasso và mô hình LightGBM để hai chương thống nhất 100%.
  9. **Trung tính hóa văn phong khoa học trước thực nghiệm:** Thay thế các khẳng định chủ quan trước khi có kết quả ("tính ưu việt", "giải pháp đột phá", "dự phóng chính xác", "hạn chế tối đa rủi ro vỡ quỹ", "quan trọng nhất") sang văn phong khoa học chuẩn mực ("kỳ vọng", "hỗ trợ", "so sánh thực nghiệm hiệu năng").
  10. **Cập nhật Tóm tắt đồ án (Executive Summary):** Cập nhật tỷ lệ khuyết thiếu thực tế ở các cột khóa (26%–30% ở tệp ngoại trú và 11%–21% ở tệp nội trú); lược bỏ dòng đếm từ hành chính; tích hợp đầy đủ các đặc trưng và 4 mô hình thực nghiệm.
  11. **Chuẩn hóa Bảng 1.1:** Bổ sung phương pháp phân định tệp dữ liệu (tệp nội trú nhận diện bằng 3 cột đặc thù: `AdmissionDt`, `DischargeDt`, `DiagnosisGroupCode`), viết gọn tên tệp và cập nhật cơ cấu biến / số dòng hợp lệ ban đầu.
* **Các tệp liên kết cập nhật:**
  * Tệp nguồn Quarto chính thức: [tiểu luận.qmd](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd)
  * Tệp nguồn Quarto ASCII: [tieu_luan.qmd](file:///c:/Users/Admin/Downloads/Aa/tieu_luan.qmd)
  * Tệp nguồn Quarto Chương 2 đã đồng bộ: [06_2521001099_PRJ04_Chuong2.qmd](file:///c:/Users/Admin/Downloads/06_2521001099_PRJ04_Chuong2.qmd)
  * Ấn phẩm HTML chính thức: [tiểu luận.html](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.html)
  * Ấn phẩm PDF xuất bản chính thức: [tiểu luận.pdf](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.pdf) (16 trang chuẩn A4)
  * Ấn phẩm PDF ASCII: [tieu_luan.pdf](file:///c:/Users/Admin/Downloads/Aa/tieu_luan.pdf)

---

## PHIÊN 4: HIỆU CHỈNH TOÀN DIỆN CHƯƠNG 1 & PHẦN MỞ ĐẦU THEO GÓP Ý HỌC THUẬT

* **Thời gian ghi nhận:** `[2026-11-16 15:15]`
* **Mục tiêu phiên:** Tiếp thu, giải quyết trọn vẹn 10 góp ý chi tiết từ sinh viên về Chương 1 và Phần mở đầu; tối ưu hóa ngắt trang và biên dịch xuất bản ấn phẩm PDF 16 trang chuẩn A4.
* **Nội dung đã hoàn thành:**
  * **Đồng bộ tên bảng biểu:** Chuẩn hóa tên [Bảng 1.1](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd#L487-L490) (*"So sánh đặc trưng và cơ cấu chi phí giữa hai tập dữ liệu khám chữa bệnh Ngoại trú và Nội trú"*) và [Bảng 1.2](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd#L663-L675) (*"Ma trận liên kết Mục tiêu nghiên cứu, Câu hỏi nghiên cứu và Phương pháp thực hiện"*) khớp chính xác 100% giữa Danh mục bảng biểu và nội dung thân bài.
  * **Thống nhất Mục lục:** Chuyển đổi toàn bộ các đề mục trong Mục lục từ *"Phần 1–5"* sang cấu trúc học thuật chuẩn *"Chương 1–5"*.
  * **Bổ sung Danh mục chữ viết tắt:** Thêm 6 từ viết tắt chuyên ngành quan trọng: `HHS`, `OLS`, `RQ`, `RBNS`, `IBNR`, `R²`; đưa công thức IQR về dạng ký hiệu toán học Unicode: `IQR = Q₃ - Q₁`.
  * **Thống nhất định dạng dấu số:** Đồng nhất trên toàn bài (từ Tóm tắt đến Thân bài) cách dùng **dấu chấm (`.`) phân cách hàng nghìn** (`517.737`, `9.551`, `527.288`, `7.690`, `7.344`, `346`, `452.487`, `6.982`, `1.157.614.040 USD`, `650.224.658 USD`) và **dấu phẩy (`,`) cho phần thập phân**.
  * **Hiệu chỉnh Mục 1.1:** Đổi tiêu đề thành *“1.1. Bối cảnh nghiên cứu và lý do chọn đề tài”*; bổ sung mục 1.1.2 phân tích rõ 3 lý do chọn đề tài theo khung yêu cầu; cập nhật số liệu Medicare vĩ mô (*hơn 65 triệu người thụ hưởng*, *ngân sách chi trả hàng năm xấp xỉ 1.000 tỷ USD* theo báo cáo chính thức của CMS năm 2023); thay nguồn HHS 2016 bằng báo cáo thanh toán sai lệch thực tế của CMS (2023) và HHS-OIG (2023); đổi thuật ngữ *"trục lợi"* thành *"hồ sơ bất thường cần rà soát"*; loại bỏ các từ ngữ cảm tính (*"hóc búa"*, *"sống còn"*); nêu rõ số liệu trung bình và trung vị được tính toán trên **các bản ghi hợp lệ** ($\ge 0$).
  * **Hiệu chỉnh Mục 1.2:** Đổi *"OLS thất bại hoàn toàn"* thành *"kém phù hợp"*; liệt kê đầy đủ tất cả các dạng lỗi dữ liệu thực tế trong bộ dữ liệu PRJ04 (`None`, `N/A`, `?`, `-`, `null`, `NULL`, `nan`, `NAN`, `unknown`, `chưa rõ`, ô trống, số tiền dính đơn vị đo lường `USD`, `VND`, `đ`, `kg`, `%`, khoảng trắng thừa, chữ hoa/thường lẫn lộn, ngày bị cắt cụt, và 7.690 giá trị âm ở biến mục tiêu); tách thành 4 câu hỏi nghiên cứu riêng biệt (**RQ1** - Feature Drivers, **RQ2** - Inpatient vs Outpatient Differences, **RQ3** - Model Benchmarking, **RQ4** - Anomaly Detection & Provider Screening); dẫn chiếu Bảng 1.1 trong bài và chuẩn hóa công thức `Q₃ + 1,5×IQR`.
  * **Hiệu chỉnh Mục 1.3:** Sửa mục tiêu tổng quát thành *"đạt độ chính xác cao nhất trong các mô hình thử nghiệm"*; cập nhật danh mục lỗi và cụm từ *"7.690 giá trị âm ở biến mục tiêu"*; đổi *"Top 1% cơ sở trục lợi"* thành *"Top 1% cơ sở có chi phí lệch bất thường"*.
  * **Hiệu chỉnh Mục 1.4:** Ghi rõ nguồn là **“Bộ dữ liệu PRJ04 do giảng viên cung cấp”** (dữ liệu tổng hợp synthetic data); thêm khoảng thời gian thực tế ghi nhận trong file từ **tháng 11/2008 đến tháng 12/2009**; báo cáo số bản ghi hợp lệ còn lại sau làm sạch (**452.487** ngoại trú và **6.982** nội trú).
  * **Hiệu chỉnh Mục 1.5 & Bảng 1.2:** Đổi *"trục lợi"* thành *"bất thường"*; loại bỏ tên hàm cài đặt `audit_dataset()` khỏi ma trận tổng quan; gán câu hỏi nghiên cứu cho Mục tiêu 1 và bổ sung RQ4 cho Mục tiêu 4; giữ nguyên số lượng 57 biến kiểm toán.
  * **Chuẩn hóa Tài liệu tham khảo theo APA 7th Edition:** Sửa tên tác giả `Goldburd, M., Khare, A., & Tevet, D. (2016)`; thêm `(Eds.)` cho `Frees et al. (2014)`; thay thế HHS (2016) bằng `CMS (2023)` và `HHS-OIG (2023)`; bổ sung mục trích dẫn bộ dữ liệu `PRJ04 do giảng viên cung cấp (2026)`.
  * **Tối ưu hóa ngắt trang & Biên dịch PDF:** Cấu hình `page-break-after: avoid` cho tiêu đề và `page-break-inside: avoid` cho bảng biểu; tinh chỉnh padding bảng viết tắt để loại bỏ hoàn toàn trang trắng dôi dòng, biên dịch xuất bản ấn phẩm [tiểu luận.pdf](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.pdf) gồm đúng **16 trang A4** hoàn chỉnh.
  * **Các tệp liên kết:**
    * Tệp nguồn Quarto chính thức: [tiểu luận.qmd](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd)
    * Tệp nguồn Quarto ASCII dự phòng: [tieu_luan.qmd](file:///c:/Users/Admin/Downloads/Aa/tieu_luan.qmd)
    * Cơ sở dữ liệu trích dẫn APA 7th: [references.bib](file:///c:/Users/Admin/Downloads/Aa/references.bib)
    * Script biên dịch tự động: [render_tieu_luan_pdf.py](file:///c:/Users/Admin/Downloads/Aa/render_tieu_luan_pdf.py)
    * Ấn phẩm xuất bản chính thức: [tiểu luận.pdf](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.pdf)
    * Ấn phẩm xuất bản ASCII dự phòng: [tieu_luan.pdf](file:///c:/Users/Admin/Downloads/Aa/tieu_luan.pdf)

---

## PHIÊN 3: SOẠN THẢO SƠ BỘ CHƯƠNG 1 (TỔNG QUAN ĐỀ TÀI & MỤC TIÊU NGHIÊN CỨU)

* **Thời gian ghi nhận:** `[2026-11-16 12:45]`
* **Mục tiêu phiên:** Soạn thảo sơ bộ toàn văn Chương 1 theo 5 phân mục học thuật, tổng hợp số liệu tài chính thực nghiệm cho Bảng 1.1 và thiết lập ma trận nghiên cứu Bảng 1.2.
* **Nội dung đã hoàn thành:**
  * Soạn thảo 5 phân mục cốt lõi của Chương 1: 1.1 Thị trường BHYT và vai trò dự báo chi phí bồi thường; 1.2 Thách thức kỹ thuật và các câu hỏi nghiên cứu ban đầu; 1.3 Mục tiêu tổng quát và 4 mục tiêu cụ thể; 1.4 Đối tượng và phạm vi nghiên cứu; 1.5 Ý nghĩa lý luận và thực tiễn.
  * Trích xuất các chỉ số tài chính từ 2 tệp CSV để xây dựng Bảng 1.1 đối chiếu đặc trưng giữa hồ sơ Ngoại trú và Nội trú.
  * Xây dựng Bảng 1.2 liên kết Mục tiêu nghiên cứu, Câu hỏi nghiên cứu, Kỹ thuật Python và Kết quả kỳ vọng.
  * Đồng bộ vào mã nguồn Quarto và chạy thử nghiệm render PDF nháp.
  * **Các tệp liên kết:**
    * Tệp nguồn tiểu luận: [tiểu luận.qmd](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd)
    * Notebook phân tích dữ liệu: [Bao_Cao_Tieu_Luan_Nhom6.ipynb](file:///c:/Users/Admin/Downloads/Aa/Bao_Cao_Tieu_Luan_Nhom6.ipynb)
    * Script render PDF: [render_tieu_luan_pdf.py](file:///c:/Users/Admin/Downloads/Aa/render_tieu_luan_pdf.py)

---

## PHIÊN 2: KHỞI TẠO KHUNG QUY CHUẨN TIỂU LUẬN & SOẠN THẢO 7 TRANG PHẦN ĐẦU

* **Thời gian ghi nhận:** `[2026-11-16 11:45]`
* **Mục tiêu phiên:** Thiết lập tệp Quarto `tiểu luận.qmd` đáp ứng 100% các tiêu chuẩn kỹ thuật khắt khe của trường UFM; soạn thảo toàn bộ 7 trang phần đầu bài báo cáo.
* **Nội dung đã hoàn thành:**
  * Cấu hình quy chuẩn văn bản học thuật trong YAML front-matter và CSS in ấn: Khổ A4 in dọc (210 mm × 297 mm), lề trên 2.0 cm, lề dưới 2.0 cm, lề trái 3.0 cm (đóng gáy), lề phải 2.0 cm, phông Times New Roman 13pt, khoảng cách dòng 1.4 lines, khoảng cách đoạn after 6pt, thụt đầu dòng 0.8 cm.
  * Cấu hình vị trí đặt tên: `tbl-cap-location: top` (tên bảng đặt ở trên) và `fig-cap-location: bottom` (tên hình đặt ở dưới).
  * Tích hợp hệ thống trích dẫn chuẩn APA 7th Edition với tệp định dạng [apa.csl](file:///c:/Users/Admin/Downloads/Aa/apa.csl) và cơ sở dữ liệu [references.bib](file:///c:/Users/Admin/Downloads/Aa/references.bib).
  * Soạn thảo chỉn chu 7 trang đầu theo xác nhận chính xác từ sinh viên:
    1. Trang 1: Trang bìa chính có logo trường UFM [Ảnh1.png](file:///c:/Users/Admin/Downloads/Aa/Ảnh1.png), viền đôi trang trọng.
    2. Trang 2: Lời cam đoan liêm chính học thuật (có chữ ký 2 thành viên).
    3. Trang 3: Lời cảm ơn Nhà trường, Khoa và ThS. Trần Mạnh Tường.
    4. Trang 4: Mục lục nội dung tự động căn lề số trang bằng đường chấm nối (dot leaders).
    5. Trang 5: Danh mục chữ viết tắt chuyên ngành.
    6. Trang 6: Danh mục bảng biểu và Danh mục hình vẽ / biểu đồ dự kiến.
    7. Trang 7: Tóm tắt đồ án (Executive Summary) chuẩn xác 300 từ.
  * Xây dựng script [render_tieu_luan_pdf.py](file:///c:/Users/Admin/Downloads/Aa/render_tieu_luan_pdf.py) điều khiển Microsoft Edge Headless chuyển đổi HTML sang PDF vector sắc nét.
  * **Các tệp liên kết:**
    * Tệp nguồn Quarto: [tiểu luận.qmd](file:///c:/Users/Admin/Downloads/Aa/tiểu%20luận.qmd)
    * Tệp kiểu trích dẫn APA: [apa.csl](file:///c:/Users/Admin/Downloads/Aa/apa.csl)
    * Tệp BibTeX trích dẫn: [references.bib](file:///c:/Users/Admin/Downloads/Aa/references.bib)
    * Logo trường UFM: [Ảnh1.png](file:///c:/Users/Admin/Downloads/Aa/Ảnh1.png)
    * Script render: [render_tieu_luan_pdf.py](file:///c:/Users/Admin/Downloads/Aa/render_tieu_luan_pdf.py)

---

## PHIÊN 1: KHAI PHÁ DỮ LIỆU TỪ NOTEBOOK, XÂY DỰNG DASHBOARD & TRÍCH XUẤT BIỂU ĐỒ

* **Thời gian ghi nhận:** `[2026-11-16 09:30]`
* **Mục tiêu phiên:** Phân tích dữ liệu thực tế từ `Bao_Cao_Tieu_Luan_Nhom6.ipynb` và 2 file CSV bồi thường PRJ04; xây dựng Dashboard tương tác HTML và xuất bản biểu đồ PNG.
* **Nội dung đã hoàn thành:**
  * Phân tích cấu trúc 2 tệp dữ liệu thực nghiệm:
    * File Ngoại trú: 517.737 dòng, 27 cột thuộc tính, 140.650 bệnh nhân duy nhất, 8.835 cơ sở y tế (Provider).
    * File Nội trú: 9.551 dòng, 30 cột thuộc tính, 7.714 bệnh nhân duy nhất, 723 cơ sở y tế.
  * Kiểm toán chất lượng dữ liệu: Phát hiện 7.690 giá trị âm ở biến mục tiêu `InscClaimAmtReimbursed` (7.344 ở ngoại trú và 346 ở nội trú); phát hiện các chuỗi dữ liệu thiếu giả lập (`'None'`, `'N/A'`, `'?'`, `'-'`); ghi nhận phân phối lệch phải cực đoan với giá trị chi trả tối đa lên tới 47.000.000 USD.
  * Phát triển Dashboard tương tác trực quan [Dashboard_Bao_Cao_Tieu_Luan_Nhom6.html](file:///c:/Users/Admin/Downloads/Aa/Dashboard_Bao_Cao_Tieu_Luan_Nhom6.html) bằng script [build_dashboard.py](file:///c:/Users/Admin/Downloads/Aa/build_dashboard.py), cho phép người dùng chuyển đổi linh hoạt qua 5 phần của báo cáo.
  * Trích xuất 2 biểu đồ PNG chất lượng cao:
    * [bieu_do_ty_le_thieu_du_lieu.png](file:///c:/Users/Admin/Downloads/Aa/bieu_do_ty_le_thieu_du_lieu.png): Biểu đồ thanh ngang so sánh Top 15 cột thiếu dữ liệu nhiều nhất.
    * [bieu_do_boxplot_outliers_tai_chinh.png](file:///c:/Users/Admin/Downloads/Aa/bieu_do_boxplot_outliers_tai_chinh.png): Biểu đồ hộp phát hiện các khoản chi trả ngoại lai tài chính.
  * **Các tệp liên kết:**
    * Notebook nguồn: [Bao_Cao_Tieu_Luan_Nhom6.ipynb](file:///c:/Users/Admin/Downloads/Aa/Bao_Cao_Tieu_Luan_Nhom6.ipynb)
    * Script tạo Dashboard: [build_dashboard.py](file:///c:/Users/Admin/Downloads/Aa/build_dashboard.py)
    * Dashboard tương tác: [Dashboard_Bao_Cao_Tieu_Luan_Nhom6.html](file:///c:/Users/Admin/Downloads/Aa/Dashboard_Bao_Cao_Tieu_Luan_Nhom6.html)
    * Biểu đồ tỷ lệ thiếu: [bieu_do_ty_le_thieu_du_lieu.png](file:///c:/Users/Admin/Downloads/Aa/bieu_do_ty_le_thieu_du_lieu.png)
    * Biểu đồ hộp ngoại lai: [bieu_do_boxplot_outliers_tai_chinh.png](file:///c:/Users/Admin/Downloads/Aa/bieu_do_boxplot_outliers_tai_chinh.png)

---
*Nhật ký được lập và cập nhật tự động vào ngày 19/11/2026 bởi Antigravity AI Coding Assistant.*
