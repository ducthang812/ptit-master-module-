# **SRS – Hệ thống quản lý chuỗi xe khách đường dài**

SRS (đặc tả yêu cầu phần mềm) này mô tả 5 màn hình, 17 use case, logic xử lý từ giao diện xuống CSDL, 18 công thức và 32 thông báo lỗi song ngữ của ứng dụng quản lý chuỗi xe khách, bám theo prototype đã chốt (menu chia Thiết lập / Vận hành). Mỗi trường ghi rõ giá trị cố định trên giao diện, cấu hình trong CSDL hay hằng số trong code, và được kiểm tra ở đâu.

## **1\. Giới thiệu và quy ước**

**Kiến trúc:** giao diện Python Streamlit → thư viện pyodbc → SQL Server. Mọi quy tắc nghiệp vụ quan trọng được kiểm tra **trong CSDL** (ràng buộc, trigger, thủ tục), giao diện chỉ kiểm tra định dạng để báo lỗi sớm. Vì vậy sửa dữ liệu trực tiếp bằng SQL vẫn bị chặn đúng như trên giao diện.

**Tài liệu liên quan:** URD CSDL; BRD; file QLXeKhach\_SQLServer.sql; luong-nghiep-vu-xe-khach-swimlane.draw io; prototype "Giao diện QL chuỗi xe khách".

**Ký hiệu cột "Nguồn giá trị"**

| Ký hiệu | Ý nghĩa | Muốn đổi thì |
| :---- | :---- | :---- |
| FIX-UI | Giá trị hoặc danh sách gắn cứng trên giao diện (nhãn, danh sách chọn cố định) | Sửa code giao diện |
| CFG | Tham số lưu trong CSDL (ThamSo, DoPhucTap, BangGiaVe, LichNgay) | Sửa trên MH01 Thiết lập, không cần sửa code |
| CONST | Hằng số nằm trong code giao diện hoặc trong ràng buộc, trigger SQL | Sửa code hoặc script SQL |
| DB | Dữ liệu nghiệp vụ đọc từ bảng (xe, tuyến, tài xế, chuyến, vé) | Thao tác nghiệp vụ trên các màn hình |
| AUTO | Hệ thống tự tính, người dùng không nhập | Đổi công thức hoặc tham số đầu vào |

**Ký hiệu cột "Kiểm tra ở"**

| Ký hiệu | Nơi kiểm tra | Khi nào chạy |
| :---- | :---- | :---- |
| FE | Code Streamlit, trước khi gọi CSDL | Ngay khi bấm nút; báo lỗi không cần gọi CSDL |
| CK | Ràng buộc khai báo (PK, FK, UNIQUE, CHECK) | Khi INSERT / UPDATE |
| TRG | Trigger TR01–TR06 | Sau INSERT / UPDATE; lỗi thì ROLLBACK |
| SP | Thủ tục lưu trữ (transaction) | Trong giao dịch; lỗi thì ROLLBACK toàn bộ |

**Ngày hiện tại:** bản thật dùng CAST(GETDATE() AS DATE); prototype cố định **02/06/2025** để khớp dữ liệu mẫu (CONST).

## **2\. Tham số cấu hình và hằng số**

### **2.1 Tham số cấu hình (CFG) – sửa trên giao diện, không sửa code**

| Mã tham số | Ý nghĩa | Mặc định | Lưu ở | Sửa tại | Dùng ở công thức |
| :---- | :---- | :---- | :---- | :---- | :---- |
| DON\_GIA\_KM | Đơn giá thù lao mỗi km | 2.000 đ | ThamSo | MH01 Tab 1 | F01 |
| HS\_LAI | Hệ số vai trò lái xe | 1,0 | ThamSo | MH01 Tab 1 | F01 |
| HS\_PHU | Hệ số vai trò phụ xe | 0,6 | ThamSo | MH01 Tab 1 | F01 |
| HeSoLuong (cấp 1/2/3) | Hệ số lương theo độ phức tạp tuyến | 1,0 / 1,2 / 1,5 | DoPhucTap | MH01 Tab 1 | F01 |
| HeSoDuongKho (cấp 1/2/3) | Hệ số quy đổi km làm việc | 1,0 / 1,3 / 1,6 | DoPhucTap | MH01 Tab 1 | F10 |
| CHU\_KY\_BD | Chu kỳ bảo dưỡng tối đa | 360 ngày | ThamSo | MH01 Tab 1 | F11 |
| KM\_MOI\_NGAY\_BD | Số km làm việc làm chu kỳ ngắn đi 1 ngày | 100 km | ThamSo | MH01 Tab 1 | F11 |
| CANH\_BAO\_NGAY (mới) | Ngưỡng "sắp đến hạn" | 15 ngày | ThamSo | MH01 Tab 1 | F14 |
| CHU\_KY\_DK\_CU (mới) | Chu kỳ đăng kiểm xe cũ | 6 tháng | ThamSo | MH01 Tab 1 | F12, F13 |
| CHU\_KY\_DK\_MOI (mới) | Chu kỳ đăng kiểm xe mới | 12 tháng | ThamSo | MH01 Tab 1 | F12, F13 |
| TUOI\_XE\_CU (mới) | Xe từ bao nhiêu năm tuổi tính là xe cũ | 5 năm | ThamSo | MH01 Tab 1 | F12 |
| GiaVe | Giá vé theo tuyến × loại ngày | Theo từng tuyến | BangGiaVe | MH01 Tab 2 | F04 |
| MaLoaiNgay theo ngày | Ngày nào là cuối tuần, mùa du lịch, lễ tết | Nạp sẵn năm 2025 | LichNgay | Chỉ sửa bằng SQL (chưa có màn hình) | F03 |

Thay đổi tham số chỉ áp dụng **từ thời điểm lưu**: thù lao đã ghi của phân công cũ và giá đã ghi trên vé cũ giữ nguyên; lịch bảo dưỡng tính lại ngay vì được tính khi xem.

### **2.2 Hằng số (CONST) – muốn đổi phải sửa code hoặc script SQL**

| Hằng số | Giá trị | Nằm ở | Ghi chú |
| :---- | :---- | :---- | :---- |
| Số ghế không bán | 2 (ghế lái, ghế phụ xe) | TR01, giao diện sơ đồ ghế | Yêu cầu của đề bài, F05 |
| Tuổi tài xế tối thiểu | 21 | CK\_TX\_Tuoi | So với ngày hiện tại |
| Năm sản xuất hợp lệ | 2005 → năm hiện tại | CK\_Xe\_NamSX |  |
| Số ghế của loại xe | 9 → 50 | CK\_LoaiXe\_SoGhe | Danh mục loại xe nạp bằng SQL |
| Định dạng CCCD | Đúng 12 chữ số | FE \+ CHAR(12) |  |
| Định dạng SĐT khách | 10 chữ số, bắt đầu bằng 0 | FE |  |
| Vận tốc tính giờ đến gợi ý | 55 km/h \+ 0,5 giờ nghỉ, làm tròn 15 phút | FE | F09; chỉ để gợi ý, người dùng sửa được ở bản thật |
| Ngưỡng hỏi lại khi gia hạn đăng kiểm sớm | Còn hạn \> 30 ngày | FE | Chỉ cảnh báo, không chặn |
| Định dạng mã | XE \+ 3 số · TX \+ 4 số · T \+ 2 số · C \+ yyMMdd \+ 3 số · HK \+ 6 số | FE sinh mã, CK\_Chuyen\_MaFormat | F07, F08 |
| Trạng thái | Xe: HoatDong / BaoDuong / NgungHD · Chuyến: KeHoach / DangChay / HoanThanh / Huy · Vé: DaBan / DaHuy · Tài xế: DangLam / NghiViec | CHECK từng bảng |  |

### **2.3 Giá trị cố định trên giao diện (FIX-UI)**

| Danh sách | Giá trị | Màn hình |
| :---- | :---- | :---- |
| Cấp độ tuyến | 1 · đường bằng / 2 · đèo nhẹ / 3 · đèo dốc | MH01 Tab 2 |
| Hạng GPLX | D (xe 10–30 chỗ) / E (trên 30 chỗ) / FC | MH01 Tab 4 |
| Kết quả đăng kiểm | Đạt / Không đạt | MH04 |
| Giờ khởi hành | Prototype: 7 mốc cố định (02, 03/06; 02, 06, 08/09/2025) để minh họa loại ngày. Bản thật: chọn ngày giờ tự do \[CẦN BỔ SUNG\] | MH02 |
| Tên loại ngày, nhãn tab, menu | Tiếng Việt cố định | Mọi màn hình |

## **3\. Công thức tính**

| Mã | Đại lượng | Công thức | Tính ở | Lúc nào | Ví dụ |
| ----- | ----- | ----- | ----- | ----- | ----- |
| F01 | Thù lao 1 tài xế / 1 chuyến | DoDaiKm × DON\_GIA\_KM × HeSoLuong\[cấp tuyến\] × (HS\_LAI hoặc HS\_PHU), làm tròn đến đồng | TR05 | Khi INSERT PhanCong; lưu cố định | Hà Nội – Vinh 300 km, cấp 2: lái 720.000 đ, phụ 432.000 đ |
| F02 | Lương tháng | Σ ThuLao các phân công thuộc chuyến **Hoàn thành** có tháng(GioKhoiHanhDK) \= tháng chọn | sp\_R01, sp\_ChotLuongThang | Khi xem báo cáo; khi chốt thì ghi BangLuong | Tháng 05/2025 toàn chuỗi: 465.024.000 đ |
| F03 | Loại ngày | Tra LichNgay theo ngày khởi hành. Ưu tiên khi trùng: Lễ tết \> Mùa du lịch \> Cuối tuần \> Ngày thường. Ngày không có trong LichNgay \= Ngày thường | TR02 | Khi bán vé; khi xem trước | 02/06/2025 \= Mùa du lịch; 06/09/2025 \= Cuối tuần |
| F04 | Giá vé | BangGiaVe\[MaTuyen, F03(ngày khởi hành)\]; ghi vào Ve.GiaVe | TR02 | Khi INSERT Ve | T01 mùa du lịch: 200.000 đ |
| F05 | Số chỗ khách | LoaiXe.SoGhe − 2 | TR01, sơ đồ ghế | Khi bán vé, vẽ sơ đồ | Xe 45 ghế: 43 chỗ |
| F06 | Lần bán | Số vé (kể cả đã hủy) của cùng chuyến và cùng ghế \+ 1 | TR02 | Khi INSERT Ve | Ghế đã có 1 vé hủy: lần bán \= 2 |
| F07 | Mã vé | MaTuyen-MaXe-MaChuyen-SoGhe(2 số)-LanBan | TR02 | Khi INSERT Ve | T02-XE011-C250601001-07-1 |
| F08 | Mã chuyến | 'C' \+ yyMMdd(ngày khởi hành) \+ số thứ tự 3 chữ số trong ngày | FE | Khi tạo chuyến | C250602003 |
| F09 | Giờ đến dự kiến (gợi ý) | Giờ đi \+ làm tròn 15 phút của (DoDaiKm ÷ 55 \+ 0,5) giờ | FE | Khi chọn tuyến, giờ | 300 km, đi 07:00 → đến 13:00 |
| F10 | Km làm việc | Σ KmThucTe × HeSoDuongKho\[cấp tuyến\] của chuyến Hoàn thành có ngày khởi hành ≥ ngày bảo dưỡng gần nhất | vw\_LichBaoDuong | Khi xem; không lưu | 5.000 km cấp 3 \= 8.000 km làm việc |
| F11 | Ngày bảo dưỡng tiếp theo | Ngày bảo dưỡng gần nhất (chưa có thì ngày đưa vào sử dụng) \+ CHU\_KY\_BD − ⌊F10 ÷ KM\_MOI\_NGAY\_BD⌋ | vw\_LichBaoDuong | Khi xem; TR04 dùng khi tạo chuyến | 07/03/2025 \+ 360 − 296 \= 10/05/2025 |
| F12 | Chu kỳ đăng kiểm của xe | Tuổi xe \= năm hiện tại − NamSanXuat; tuổi ≥ TUOI\_XE\_CU → CHU\_KY\_DK\_CU, ngược lại CHU\_KY\_DK\_MOI | FE (xem trước), SP | Khi thêm xe, gia hạn | Xe 2017: 8 năm → 6 tháng |
| F13 | Hạn đăng kiểm | Kết quả Đạt: hạn mới \= ngày đăng kiểm \+ F12 (tháng). Hạn hiện hành \= MAX(NgayHetHan) các lần Đạt. Không đạt: hạn giữ nguyên | SP, vw\_LichBaoDuong | Khi lưu đăng kiểm | 02/06/2025 \+ 12 tháng \= 02/06/2026 |
| F14 | Tình trạng hạn (bảo dưỡng, đăng kiểm) | d \= hạn − ngày hiện tại. d \< 0: **Quá hạn** (đỏ). 0 ≤ d ≤ CANH\_BAO\_NGAY: **Sắp đến hạn** (vàng). Còn lại: Bình thường. Chưa có đăng kiểm: **Chưa đăng kiểm** (đỏ) | FE | Khi xem MH04, MH01 | Hạn 10/05/2025, ngày 02/06/2025: quá hạn 23 ngày |
| F15 | Doanh thu | Σ Ve.GiaVe của vé **Đã bán** thuộc chuyến **Hoàn thành**, theo tháng khởi hành; nhóm theo xe, tuyến hoặc loại xe | sp\_R02–R04 | Khi xem báo cáo | Tháng 05/2025: 2.906.900.000 đ |
| F16 | Mức độ sẵn sàng | 5 dòng: (1) tham số; (2) số tuyến thiếu ít nhất 1 mức giá; (3) số xe đang chạy có hạn đăng kiểm \< hôm nay hoặc chưa có; (4) số xe có F11 \< hôm nay; (5) số tài xế. Dòng (2)–(4) \> 0 thì vàng | FE | Mỗi lần mở MH01 | "⚠ 1 xe quá hạn bảo dưỡng: 29B-532.63" |
| F17 | Tuổi tài xế | Ngày sinh ≤ ngày hiện tại − 21 năm (prototype nhập năm sinh: năm sinh ≤ năm hiện tại − 21\) | CK\_TX\_Tuoi | Khi INSERT TaiXe | Sinh 2008: không hợp lệ |
| F18 | Trùng giờ | Hai chuyến (không Hủy) trùng giờ khi Đi₁ \< Đến₂ và Đi₂ \< Đến₁ (theo giờ dự kiến), xét cùng xe hoặc cùng tài xế | TR03, TR04 | Khi tạo chuyến, phân công | 13:00–19:00 trùng 14:00–17:30 |

## **4\. Use case và luồng màn hình**

### **4.1 Cấu trúc menu và màn hình**

| Nhóm menu | Mã | Màn hình | Tab | Actor chính |
| :---- | :---- | :---- | :---- | :---- |
| THIẾT LẬP | MH01 | Thiết lập hệ thống | Khung Mức độ sẵn sàng · Tab 1 Tham số · Tab 2 Tuyến và giá vé · Tab 3 Xe và đăng kiểm · Tab 4 Tài xế | Quản lý chi nhánh |
| VẬN HÀNH | MH02 | Chuyến xe | Tạo chuyến · Danh sách, hủy chuyến · Kết thúc chuyến | NV điều hành |
| VẬN HÀNH | MH03 | Bán vé | Bán vé · Hủy vé · Tra cứu vé | NV bán vé |
| VẬN HÀNH | MH04 | Kỹ thuật xe | Xe cần xử lý · Ghi nhận bảo dưỡng · Gia hạn đăng kiểm · Lịch bảo dưỡng, đăng kiểm | NV kỹ thuật |
| VẬN HÀNH | MH05 | Báo cáo | Lương · Doanh thu theo xe, tuyến, loại xe · Xe quá hạn bảo dưỡng | Quản lý chi nhánh |

Thứ tự sử dụng: MH01 (đủ Mức độ sẵn sàng) → MH02 tạo chuyến → MH03 bán vé → MH02 kết thúc chuyến → MH05 báo cáo; MH04 bảo dưỡng xong thì xe quay lại MH02. Sơ đồ chi tiết ở file luong-nghiep-vu-xe-khach-swimlane.drawio.

### **4.2 Danh sách use case**

| ID | Tên | Màn hình | Actor | Trigger | Ưu tiên | Code CSDL |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| UC-001 | Xem mức độ sẵn sàng | MH01 | Quản lý | Mở MH01 | High | Truy vấn đọc |
| UC-002 | Thiết lập tham số | MH01 Tab 1 | Quản lý | Bấm Lưu tham số | High | UPDATE ThamSo, DoPhucTap |
| UC-003 | Thêm tuyến kèm giá vé | MH01 Tab 2 | Quản lý | Bấm Lưu tuyến và giá vé | High | sp\_ThemTuyenVaGia |
| UC-004 | Sửa bảng giá vé | MH01 Tab 2 | Quản lý | Bấm Lưu thay đổi giá | Medium | MERGE BangGiaVe |
| UC-005 | Thêm xe kèm đăng kiểm lần đầu | MH01 Tab 3 | Quản lý | Bấm Lưu xe và đăng kiểm | High | sp\_ThemXeVaDangKiem |
| UC-006 | Xóa xe | MH01 Tab 3 | Quản lý | Bấm Xóa ở dòng xe | Low | DELETE Xe (FK) |
| UC-007 | Thêm tài xế | MH01 Tab 4 | Quản lý | Bấm Thêm tài xế | High | INSERT TaiXe |
| UC-008 | Tạo chuyến và phân công | MH02 | NV điều hành | Bấm Tạo chuyến và phân công | High | sp\_TaoChuyenVaPhanCong, TR03–TR05 |
| UC-009 | Hủy chuyến | MH02 | NV điều hành | Bấm Hủy chuyến | Medium | UPDATE ChuyenXe |
| UC-010 | Kết thúc chuyến | MH02 | NV điều hành | Bấm Hoàn thành chuyến | High | sp\_KetThucChuyen, TR06 |
| UC-011 | Bán vé | MH03 | NV bán vé | Bấm Bán vé ghế số … | High | sp\_BanVe, TR01, TR02 |
| UC-012 | Hủy vé | MH03 | NV bán vé | Bấm Hủy vé | Medium | UPDATE Ve |
| UC-013 | Tra cứu vé | MH03 | NV bán vé | Nhập số điện thoại | Medium | Truy vấn đọc |
| UC-014 | Ghi nhận bảo dưỡng | MH04 | NV kỹ thuật | Bấm Lưu bảo dưỡng | High | sp\_GhiNhanBaoDuong |
| UC-015 | Gia hạn đăng kiểm | MH04 | NV kỹ thuật | Bấm Lưu đăng kiểm | High | sp\_GhiNhanDangKiem |
| UC-016 | Xem báo cáo | MH05 | Quản lý | Chọn tab báo cáo | High | sp\_R01–R06 |
| UC-017 | Chốt lương tháng | MH05 | Quản lý | Bấm Chốt lương tháng | High | sp\_ChotLuongThang |

Phi chức năng chung: thao tác ghi phản hồi dưới 2 giây; báo cáo tháng (khoảng 10.000 vé) dưới 5 giây; mọi lỗi hiển thị bằng tiếng Việt, rõ lý do; mọi thao tác nhiều bước chạy trong một giao dịch.

## **5\. MH01 Thiết lập hệ thống**

Menu THIẾT LẬP · Actor: Quản lý chi nhánh · Use case UC-001 đến UC-007. Màn hình gồm khung Mức độ sẵn sàng (MH01.0) và 4 tab (MH01.1–MH01.4).

### **5.1 Business rule MH01**

| Mã | Quy tắc | Áp dụng tại | Kiểm tra ở |
| :---- | :---- | :---- | :---- |
| BR-MH01-01 | Khung Mức độ sẵn sàng tính lại mỗi lần mở MH01 và sau mỗi lần Lưu thành công ở bất kỳ tab nào | MH01.0 | Server \+ Client |
| BR-MH01-02 | Mọi tham số phải là số \> 0; hệ số đường khó ≥ 1; số thập phân nhập bằng dấu phẩy hoặc dấu chấm | MH01.1 | Client \+ CK |
| BR-MH01-03 | Tham số, giá mới chỉ áp dụng từ lúc lưu: thù lao đã ghi và giá trên vé đã bán giữ nguyên; lịch bảo dưỡng tính lại ngay | MH01.1, MH01.2 | Server |
| BR-MH01-04 | Tuyến phải có đủ 4 mức giá \> 0\. Tuyến và giá lưu trong cùng 1 giao dịch; lỗi bất kỳ thì không lưu gì | MH01.2 | Server (sp\_ThemTuyenVaGia) |
| BR-MH01-05 | Điểm đầu ≠ điểm cuối; độ dài \> 0 km; không trùng tuyến (cùng điểm đầu, điểm cuối) | MH01.2 | Client \+ CK |
| BR-MH01-06 | Giá vé chỉ phụ thuộc tuyến và loại ngày, không phụ thuộc loại xe, số ghế | MH01.2 | – |
| BR-MH01-07 | Xe phải lưu kèm lần đăng kiểm đầu (Đạt). Hạn \= ngày đăng kiểm \+ chu kỳ theo tuổi xe. Lỗi bất kỳ thì không lưu gì | MH01.3 | Server (sp\_ThemXeVaDangKiem) |
| BR-MH01-08 | Biển số duy nhất; năm sản xuất 2005 → năm hiện tại; ngày đăng kiểm ≤ hôm nay | MH01.3 | Client \+ CK \+ SP |
| BR-MH01-09 | Xe đã có chuyến không được xóa; hướng dẫn chuyển trạng thái Ngừng hoạt động | MH01.3 | FK |
| BR-MH01-10 | Tài xế từ 21 tuổi; CCCD đúng 12 chữ số và duy nhất | MH01.4 | Client \+ CK |
| BR-MH01-11 | Mã tự sinh tăng dần, không dùng lại: tuyến T \+ 2 số, xe XE \+ 3 số, tài xế TX \+ 4 số | MH01.2–MH01.4 | Client |

### **5.2 MH01.0 Khung Mức độ sẵn sàng**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề | Label | Read Only | Y | \-- | textVIE: Thiết lập hệ thống |
| 2 | Mô tả phụ | Label | Read Only | Y | \-- | textVIE: Làm trước khi vận hành · đi lần lượt 4 tab |
| 3 | **Mức độ sẵn sàng vận hành** |  |  |  |  |  |
| 4 | Dòng 1 – Tham số | Button dạng dòng | Click | \-- | \-- | Luôn nền xanh: "✓ 1\. Tham số: đơn giá {DON\_GIA\_KM} đ/km, chu kỳ bảo dưỡng {CHU\_KY\_BD} ngày, đăng kiểm {CHU\_KY\_DK\_CU}/{CHU\_KY\_DK\_MOI} tháng" Click: mở Tab 1 |
| 5 | Dòng 2 – Tuyến | Button dạng dòng | Click | \-- | \-- | Nền xanh khi mọi tuyến đủ 4 giá: "✓ 2\. {n} tuyến đều đủ 4 mức giá" Nền vàng khi có tuyến thiếu: "⚠ 2\. {k} tuyến thiếu giá: {danh sách mã} → bấm để nhập giá" Click: mở Tab 2 |
| 6 | Dòng 3 – Đăng kiểm xe | Button dạng dòng | Click | \-- | \-- | Xét xe khác Ngừng hoạt động có hạn đăng kiểm \< hôm nay hoặc chưa có Nền xanh: "✓ 3\. {n} xe đều còn hạn đăng kiểm" – click mở Tab 3 Nền vàng: "⚠ 3\. {k} xe chưa/hết hạn đăng kiểm: {biển số} → gia hạn ở Kỹ thuật xe" – click mở MH04 |
| 7 | Dòng cảnh báo bảo dưỡng | Button dạng dòng | Click | \-- | \-- | Chỉ hiển thị khi có xe có ngày bảo dưỡng tiếp theo (F11) \< hôm nay Nền vàng: "⚠ {k} xe quá hạn bảo dưỡng: {biển số} → Kỹ thuật xe" Click: mở MH04 |
| 8 | Dòng 4 – Tài xế | Button dạng dòng | Click | \-- | \-- | Nền xanh: "✓ 4\. {n} tài xế" Click: mở Tab 4 |
| 9 | Thanh tab | Tab | Click | Y | \-- | 4 tab: 1\. Tham số / 2\. Tuyến và giá vé / 3\. Xe và đăng kiểm / 4\. Tài xế Mặc định: Tab 1 |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng mở MH01 | Client | Gửi yêu cầu lấy dữ liệu Mức độ sẵn sàng đến Server |
|  |  | Server | Đọc ThamSo (DON\_GIA\_KM, CHU\_KY\_BD, CHU\_KY\_DK\_CU, CHU\_KY\_DK\_MOI) Đếm tuyến có ít hơn 4 dòng BangGiaVe Lấy xe (TrangThai ≠ NgungHD) có HanDangKiem \< hôm nay hoặc NULL (vw\_LichBaoDuong) Lấy xe có NgayBDTiepTheo \< hôm nay Đếm TaiXe trạng thái DangLam; trả kết quả cho Client |
|  |  | Client | Hiển thị 5 dòng theo F16: có phần tử thì nền vàng, không có thì nền xanh |
| 2 | Người dùng click 1 dòng | Client | Chuyển tới tab hoặc màn hình tương ứng (mục 4–8 bảng trên); không gọi Server |

### **5.3 MH01.1 Tab Tham số**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | **Thù lao lái xe, phụ xe** |  |  |  |  |  |
| 2 | Đơn giá mỗi km (đ) | Textbox | Input | Y | 9 | • Số nguyên \> 0 Mặc định: giá trị đang lưu ThamSo.DON\_GIA\_KM (2.000) |
| 3 | Hệ số vai trò lái xe | Textbox | Input | Y | 5 | • Số thập phân \> 0, chấp nhận "," hoặc "." • Mặc định: ThamSo.HS\_LAI (1,0) |
| 4 | Hệ số vai trò phụ xe | Textbox | Input | Y | 5 | • Như trên · mặc định ThamSo.HS\_PHU (0,6) |
| 5 | Hệ số lương tuyến cấp 1 / 2 / 3 | 3 Textbox | Input | Y | 5 | • Số thập phân \> 0 · mặc định DoPhucTap.HeSoLuong (1,0 / 1,2 / 1,5) |
| 6 | Khung ví dụ | Label | Read Only | \-- | \-- | • Dòng 1 cố định: "Thù lao \= Độ dài tuyến × Đơn giá/km × Hệ số cấp tuyến × Hệ số vai trò"  •Dòng 2, 3: ví dụ Hà Nội – Vinh (300 km, cấp 2), tính lại ngay khi gõ (F01) • Giá trị chưa hợp lệ: "Nhập số hợp lệ để xem ví dụ." |
| 7 | Button Lưu tham số thù lao | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |
| 8 | **Bảo dưỡng và đăng kiểm** |  |  |  |  |  |
| 9 | Chu kỳ bảo dưỡng tối đa (ngày) | Textbox | Input | Y | 4 | • Số nguyên \> 0 · mặc định CHU\_KY\_BD (360) |
| 10 | Số km làm việc giảm 1 ngày | Textbox | Input | Y | 5 | • Số nguyên \> 0 · mặc định KM\_MOI\_NGAY\_BD (100) |
| 11 | Ngưỡng cảnh báo sắp đến hạn (ngày) | Textbox | Input | Y | 3 | • Số nguyên \> 0 · mặc định CANH\_BAO\_NGAY (15) |
| 12 | Hệ số đường khó cấp 1 / 2 / 3 | 3 Textbox | Input | Y | 5 | • Số thập phân ≥ 1 · mặc định DoPhucTap.HeSoDuongKho (1,0 / 1,3 / 1,6) |
| 13 | Chu kỳ đăng kiểm xe cũ / xe mới (tháng) | 2 Textbox | Input | Y | 3 | • Số nguyên \> 0 · mặc định 6 / 12 |
| 14 | Xe từ bao nhiêu năm tuổi là xe cũ | Textbox | Input | Y | 2 | • Số nguyên \> 0 · mặc định TUOI\_XE\_CU (5) |
| 15 | Khung công thức | Label | Read Only | \-- | \-- | textVIE cố định: công thức F11 (bảo dưỡng tiếp theo), F10 (km làm việc), F13 (hạn đăng kiểm) |
| 16 | Button Lưu tham số bảo dưỡng, đăng kiểm | Button | Click | \-- | \-- | Xem logic xử lý mục 2 |
| 17 | **Lịch loại ngày** |  |  |  |  |  |
| 18 | Bảng lịch loại ngày | Table | Read Only | \-- | \-- | • Cột: Loại ngày, Áp dụng cho ngày, Mức so với ngày thường • Nguồn LichNgay; chỉ xem, sửa bằng SQL |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường nhập | Vui lòng nhập \<tên\_trường\> (VD: Vui lòng nhập Đơn giá mỗi km) | Hiển thị chữ đỏ dưới ô nhập |
| 2 | Nhập không phải số | \<tên\_trường\> phải là số (VD: Hệ số vai trò phụ xe phải là số) | Hiển thị chữ đỏ dưới ô nhập |
| 3 | Giá trị ≤ 0 | \<tên\_trường\> phải lớn hơn 0 | Hiển thị chữ đỏ dưới ô nhập |
| 4 | Hệ số đường khó \< 1 | Hệ số đường khó phải từ 1 trở lên | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng sửa giá trị và nhấn Lưu tham số thù lao | Client | • Đổi dấu "," thành "." rồi kiểm tra theo bảng kiểm tra dữ liệu |
|  |  |  | ○ Không hợp lệ: hiển thị lỗi tương ứng |
|  |  |  | ○ Hợp lệ: gửi 6 giá trị đến Server |
|  |  | Server | • UPDATE ThamSo: DON\_GIA\_KM, HS\_LAI, HS\_PHU |
|  |  |  | • UPDATE DoPhucTap.HeSoLuong cấp 1, 2, 3 |
|  |  |  | • Lỗi CHECK: trả mã lỗi E02 · thành công: trả OK |
|  |  | Client | • Thành công: thông báo xanh "Đã cập nhật tham số thù lao"; tính lại khung ví dụ và xem trước ở MH02 |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10 |
| 2 | Người dùng nhấn Lưu tham số bảo dưỡng, đăng kiểm | Client | • Kiểm tra như mục 1, thêm trường hợp 4; hợp lệ thì gửi 9 giá trị |
|  |  | Server | • UPDATE ThamSo: CHU\_KY\_BD, KM\_MOI\_NGAY\_BD, CANH\_BAO\_NGAY, CHU\_KY\_DK\_CU, CHU\_KY\_DK\_MOI, TUOI\_XE\_CU |
|  |  |  | • UPDATE DoPhucTap.HeSoDuongKho cấp 1, 2, 3 |
|  |  | Client | • Thông báo xanh; làm mới Mức độ sẵn sàng (lịch bảo dưỡng tính lại ngay theo tham số mới) |

### **5.4 MH01.2 Tab Tuyến và giá vé**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | **Thêm tuyến kèm giá vé** |  |  |  |  |  |
| 2 | Điểm đầu | Textbox | Input | Y | 50 | • Placeholder: Hà Nội • Không được trùng Điểm cuối |
| 3 | Điểm cuối | Textbox | Input | Y | 50 | • Placeholder: Nam Định |
| 4 | Độ dài (km) | Textbox | Input | Y | 4 | • Số nguyên \> 0 |
| 5 | Cấp độ | Combobox | Select | Y | \-- | • Cho phép chọn 1 giá trị trong danh sách:  ○ 1 · đường bằng  ○ 2 · đèo nhẹ  ○  3 · đèo dốc • Mặc định: 1 |
| 6 | Nhãn giá vé | Label | Read Only | \-- | \-- | textVIE: Giá vé theo loại ngày (đ) |
| 7 | Ngày thường | Textbox | Input | Y | 9 | • Số nguyên \> 0; cho phép gõ dấu chấm phân cách, hệ thống tự bỏ |
| 8 | Cuối tuần | Textbox | Input | Y | 9 | • Như trên |
| 9 | Mùa du lịch | Textbox | Input | Y | 9 | • Như trên |
| 10 | Lễ tết | Textbox | Input | Y | 9 | • Như trên |
| 11 | Button Lưu tuyến và giá vé | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |
| 12 | **Danh sách tuyến và bảng giá** |  |  |  |  |  |
| 13 | Bảng tuyến và giá | Table có ô nhập | Input | \-- | \-- | • Cột: Tuyến (mã · tên · km · cấp), Ngày thường, Cuối tuần, Mùa du lịch, Lễ tết • 4 cột giá là ô nhập; ô trống hiện placeholder "Chưa có" • Sắp xếp theo mã tuyến tăng dần |
| 14 | Button Lưu thay đổi giá | Button | Click | \-- | \-- | Xem logic xử lý mục 2 |
| 15 | Ghi chú | Label | Read Only | \-- | \-- | textVIE: Giá không phụ thuộc loại xe hay số ghế. Giá mới áp dụng cho vé bán sau khi lưu. |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường nhập | Vui lòng nhập \<tên\_trường\> (VD: Vui lòng nhập Giá lễ tết) | Hiển thị chữ đỏ dưới ô nhập |
| 2 | Bỏ trống trường chọn | Vui lòng chọn \<tên\_trường\> (VD: Vui lòng chọn Cấp độ) | Hiển thị chữ đỏ dưới ô chọn |
| 3 | Độ dài không phải số nguyên \> 0 | Độ dài tuyến phải là số nguyên lớn hơn 0 | Hiển thị chữ đỏ dưới ô nhập |
| 4 | Giá không phải số hoặc ≤ 0 | \<tên\_trường\> phải là số lớn hơn 0 | Hiển thị chữ đỏ dưới ô nhập |
| 5 | Điểm đầu trùng điểm cuối (không phân biệt hoa thường) | Điểm đầu và điểm cuối phải khác nhau | Hiển thị chữ đỏ dưới ô Điểm cuối |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhập tuyến, 4 giá và nhấn Lưu tuyến và giá vé | Client | • Lưu ý: mã tuyến tự sinh \= T \+ (số lớn nhất hiện có \+ 1), 2 chữ số. VD: đang có T04 → T05 |
|  |  |  | • Check validate theo bảng kiểm tra dữ liệu |
|  |  |  | ○ Không hợp lệ: hiển thị lỗi tương ứng |
|  |  |  | ○ Hợp lệ: gửi thông tin đến Server |
|  |  | Server | Server nhận thông tin, gọi sp\_ThemTuyenVaGia: |
|  |  |  | • BEGIN TRANSACTION |
|  |  |  | • INSERT TuyenDuong; kiểm tra điểm đầu ≠ cuối (E04), độ dài \> 0 (E05), không trùng tuyến |
|  |  |  | • Thiếu giá → THROW 57001 (E06) |
|  |  |  | • INSERT BangGiaVe × 4 (THUONG, CUOITUAN, MUADL, LETET); CK giá \> 0 |
|  |  |  | • Lỗi bất kỳ → ROLLBACK, tuyến cũng không được lưu · hợp lệ → COMMIT |
|  |  | Client | • Thành công: thông báo xanh "Đã thêm tuyến {mã} kèm đủ 4 mức giá"; xóa form; thêm dòng vào bảng; cập nhật Mức độ sẵn sàng |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10, giữ nguyên dữ liệu đã nhập |
| 2 | Người dùng sửa ô giá trong bảng và nhấn Lưu thay đổi giá | Client | • Bỏ ký tự không phải số trong từng ô; ô ≤ 0: báo lỗi, không gửi; hợp lệ: gửi các ô có giá |
|  |  | Server | • MERGE BangGiaVe theo (MaTuyen, MaLoaiNgay): có thì UPDATE, chưa có thì INSERT |
|  |  | Client | • Thông báo xanh "Đã cập nhật bảng giá vé"; cập nhật dòng Tuyến của Mức độ sẵn sàng |

### **5.5 MH01.3 Tab Xe và đăng kiểm**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | **Thêm xe kèm đăng kiểm lần đầu** |  |  |  |  |  |
| 2 | Biển số | Textbox | Input | Y | 12 | • Placeholder: VD 29B-777.88 • Không được trùng biển số đã có |
| 3 | Loại xe | Combobox | Select | Y | \-- | • Cho phép chọn 1 giá trị trong danh sách LoaiXe: ○ Limousine 9 chỗ ○ Xe 16 chỗ ○ Xe 29 chỗ ○ Xe 45 chỗ • Mặc định: Xe 29 chỗ |
| 4 | Năm sản xuất | Textbox | Input | Y | 4 | • 4 chữ số, từ 2005 đến năm hiện tại • Mặc định: năm hiện tại − 1 |
| 5 | Ngày đăng kiểm đạt lần đầu | Datepicker | Select | Y | \-- | • Định dạng dd/mm/yyyy • Giới hạn chọn lịch: không chọn được ngày sau hôm nay • Mặc định: hôm nay |
| 6 | Khung xem trước hạn | Label | Read Only | \-- | \-- | • Tự tính khi đổi Năm sản xuất hoặc Ngày đăng kiểm (F12, F13) • VD: "Xe 1 năm tuổi (xe mới) → chu kỳ 12 tháng → hạn đăng kiểm 02/06/2026" • Thiếu dữ liệu: "Nhập năm sản xuất và ngày đăng kiểm để xem hạn." |
| 7 | Button Lưu xe và đăng kiểm | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |
| 8 | Ghi chú | Label | Read Only | \-- | \-- | textVIE: Gia hạn đăng kiểm định kỳ làm ở màn Kỹ thuật xe. |
| 9 | **Danh sách xe** |  |  |  |  |  |
| 10 | Tìm theo biển số | Textbox | Input | N | 12 | • Lọc danh sách ngay khi gõ, theo chứa chuỗi • Mặc định: trống (hiện tất cả) |
| 11 | Bảng danh sách xe | Table | Read Only | \-- | \-- | • Cột: Mã xe, Biển số, Loại xe, Năm SX, Trạng thái, Hạn đăng kiểm, Số chuyến • Trạng thái: Hoạt động / Bảo dưỡng / Ngừng hoạt động • Sắp xếp theo mã xe tăng dần |
| 12 | Button Xóa (mỗi dòng) | Button | Click | \-- | \-- | Xem logic xử lý mục 2 |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường nhập | Vui lòng nhập \<tên\_trường\> (VD: Vui lòng nhập Biển số) | Hiển thị chữ đỏ dưới ô nhập |
| 2 | Bỏ trống trường chọn | Vui lòng chọn \<tên\_trường\> (VD: Vui lòng chọn Ngày đăng kiểm đạt lần đầu) | Hiển thị chữ đỏ dưới ô chọn |
| 3 | Năm sản xuất ngoài 2005 – năm hiện tại | Năm sản xuất phải từ 2005 đến năm hiện tại | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhập xe, ngày đăng kiểm và nhấn Lưu xe và đăng kiểm | Client | • Lưu ý: mã xe tự sinh \= XE \+ (số lớn nhất hiện có \+ 1), 3 chữ số. VD: đang có XE011 → XE012 |
|  |  |  | • Check validate theo bảng kiểm tra dữ liệu |
|  |  |  | ○ Không hợp lệ: hiển thị lỗi tương ứng |
|  |  |  | ○ Hợp lệ: gửi thông tin đến Server |
|  |  | Server | Server nhận thông tin, gọi sp\_ThemXeVaDangKiem: |
|  |  |  | • BEGIN TRANSACTION |
|  |  |  | • INSERT Xe (TrangThai \= HoatDong, NgayDuaVaoSD \= hôm nay, MaCN \= chi nhánh người dùng); kiểm tra trùng biển số (E07), năm SX (E08) |
|  |  |  | • Ngày đăng kiểm \> hôm nay → THROW 58001 (E09) |
|  |  |  | • Tính chu kỳ: tuổi xe ≥ TUOI\_XE\_CU → CHU\_KY\_DK\_CU, ngược lại CHU\_KY\_DK\_MOI |
|  |  |  | • INSERT DangKiem (NgayHetHan \= ngày \+ chu kỳ, KetQua \= Dat) |
|  |  |  | • Lỗi bất kỳ → ROLLBACK, xe cũng không được lưu · hợp lệ → COMMIT |
|  |  | Client | • Thành công: thông báo xanh "Đã thêm xe {mã} · {biển số}, đăng kiểm đến {hạn}. Xe xếp chuyến được ngay."; xóa ô Biển số; thêm dòng vào bảng |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10, giữ nguyên dữ liệu đã nhập |
| 2 | Người dùng nhấn Xóa ở một dòng xe | Client | • Gửi mã xe đến Server (không hỏi lại ở bản prototype) |
|  |  | Server | • DELETE Xe; FK\_Chuyen\_Xe chặn nếu xe đã có chuyến (E10) |
|  |  | Client | • Thành công: xóa dòng khỏi bảng |
|  |  |  | • Lỗi: hộp đỏ "Không xóa được xe đã có chuyến. Hãy đổi trạng thái sang Ngừng hoạt động." |

### **5.6 MH01.4 Tab Tài xế**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | **Thêm tài xế** |  |  |  |  |  |
| 2 | Họ và tên | Textbox | Input | Y | 70 | • Placeholder: VD Nguyễn Văn An • Khi lưu tách thành HoDem (phần trước) và Ten (từ cuối) |
| 3 | Năm sinh | Textbox | Input | Y | 4 | • 4 chữ số; tài xế phải từ 21 tuổi |
| 4 | CCCD | Textbox | Input | Y | 12 | • Đúng 12 chữ số; không trùng |
| 5 | Hạng GPLX | Combobox | Select | Y | \-- | • Cho phép chọn 1 giá trị trong danh sách: ○ D – xe 10 đến 30 chỗ ○ E – xe trên 30 chỗ ○ FC • Mặc định: D |
| 6 | Button Thêm tài xế | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |
| 7 | **Danh sách tài xế** |  |  |  |  |  |
| 8 | Bảng tài xế | Table | Read Only | \-- | \-- | • Cột: Mã TX, Họ tên, Năm sinh, CCCD, Hạng, Số chuyến, Thù lao tháng • Thù lao tháng \= tổng thù lao chuyến Hoàn thành trong tháng (F02) |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường nhập | Vui lòng nhập \<tên\_trường\> (VD: Vui lòng nhập Họ và tên) | Hiển thị chữ đỏ dưới ô nhập |
| 2 | CCCD không đủ 12 chữ số | CCCD phải gồm đúng 12 chữ số | Hiển thị chữ đỏ dưới ô nhập |
| 3 | Năm sinh không phải 4 chữ số | Năm sinh không hợp lệ | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhập thông tin và nhấn Thêm tài xế | Client | • Lưu ý: mã tài xế tự sinh \= TX \+ (số lớn nhất hiện có \+ 1), 4 chữ số |
|  |  |  | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin và thực hiện kiểm tra: |
|  |  |  | • INSERT TaiXe (TrangThai \= DangLam, NgayVaoLam \= hôm nay, MaCN \= chi nhánh người dùng) |
|  |  |  | • CK\_TX\_Tuoi: dưới 21 tuổi → lỗi E13 |
|  |  |  | • UQ\_TX\_CCCD: trùng CCCD → lỗi E12 |
|  |  | Client | • Thành công: thông báo xanh "Đã thêm tài xế {mã}"; tài xế có ngay trong danh sách chọn Lái xe, Phụ xe ở MH02 |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10 |

## **6\. MH02 Chuyến xe**

Menu VẬN HÀNH · Actor: NV điều hành · Use case UC-008, UC-009, UC-010. Gồm 3 tab: MH02.1 Tạo chuyến, MH02.2 Danh sách và hủy chuyến, MH02.3 Kết thúc chuyến.

### **6.1 Business rule MH02**

| Mã | Quy tắc | Áp dụng tại | Kiểm tra ở |
| :---- | :---- | :---- | :---- |
| BR-MH02-01 | Mỗi chuyến có đúng 1 lái xe và 1 phụ xe, là 2 người khác nhau, cố định suốt chuyến | MH02.1 | Client \+ SP \+ UQ\_PC\_ChuyenVaiTro |
| BR-MH02-02 | Xe phải đang Hoạt động, còn hạn đăng kiểm và chưa quá hạn bảo dưỡng tại ngày khởi hành | MH02.1 | TR04 |
| BR-MH02-03 | Xe và tài xế không được chạy 2 chuyến chồng giờ (F18); chuyến Hủy không tính | MH02.1 | TR03, TR04 |
| BR-MH02-04 | Tạo chuyến và phân công là 1 giao dịch: lỗi ở bất kỳ bước nào thì không lưu chuyến, không lưu phân công | MH02.1 | sp\_TaoChuyenVaPhanCong |
| BR-MH02-05 | Thù lao tính lúc phân công theo tham số hiện hành (F01) và lưu cố định | MH02.1 | TR05 |
| BR-MH02-06 | Mã chuyến \= C \+ yyMMdd \+ số thứ tự 3 chữ số trong ngày | MH02.1 | Client \+ CK\_Chuyen\_MaFormat |
| BR-MH02-07 | Chỉ hủy được chuyến đang Kế hoạch; chuyến Hủy không bán vé, không tính lương | MH02.2 | Client \+ Server |
| BR-MH02-08 | Chỉ kết thúc được chuyến đang Kế hoạch hoặc Đang chạy; phải có km \> 0, đủ lái \+ phụ. Hệ thống không tự kết thúc | MH02.3 | TR06, sp\_KetThucChuyen |
| BR-MH02-09 | Chuyến Hoàn thành mới được tính vào lương, doanh thu và km làm việc của xe | MH02.3 | Báo cáo, vw\_LichBaoDuong |

### **6.2 MH02.1 Tab Tạo chuyến**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề | Label | Read Only | Y | \-- | textVIE: Chuyến xe |
| 2 | Thanh tab | Tab | Click | Y | \-- | • 3 tab: Tạo chuyến / Danh sách, hủy chuyến / Kết thúc chuyến · mặc định Tạo chuyến |
| 3 | **Thông tin chuyến** |  |  |  |  |  |
| 4 | Tuyến | Combobox | Select | Y | \-- | • Danh sách tuyến: "{mã} · {điểm đầu} – {điểm cuối} · {km} km" • Mặc định: tuyến đầu tiên |
| 5 | Xe | Combobox | Select | Y | \-- | • Danh sách tất cả xe: "{biển số} · {loại xe} · {trạng thái}" • Hiện cả xe không hoạt động để TR04 báo lỗi rõ ràng |
| 6 | Giờ khởi hành | Combobox (prototype) / Datetimepicker (bản thật) | Select | Y | \-- | • Hiển thị "dd/mm/yyyy hh:mm · {loại ngày}" • Prototype: 7 mốc cố định; bản thật: chọn ngày giờ, không chọn được thời điểm đã qua |
| 7 | Lái xe | Combobox | Select | Y | \-- | • Danh sách tài xế Đang làm: "{mã} · {họ tên} · hạng {hạng}" |
| 8 | Phụ xe | Combobox | Select | Y | \-- | • Như Lái xe; không được trùng Lái xe |
| 9 | Khung xem trước | Label | Read Only | \-- | \-- | • Dòng 1: "Thù lao lái xe \= {km} km × {đơn giá} × {hệ số cấp} (cấp {n}) × {HS lái} (lái) \= {số tiền}" • Dòng 2: thù lao phụ xe, tương tự với HS phụ • Dòng 3: "Đến lúc {giờ đến F09} · ngày {dd/mm/yyyy} là {loại ngày} → giá vé {giá F04}" hoặc "chưa có giá" • Tính lại ngay khi đổi Tuyến hoặc Giờ khởi hành |
| 10 | Button Tạo chuyến và phân công | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường chọn | Vui lòng chọn \<tên\_trường\> (VD: Vui lòng chọn Phụ xe) | Hiển thị chữ đỏ dưới ô chọn |
| 2 | Lái xe trùng Phụ xe | Lái xe và phụ xe phải là 2 người khác nhau | Hiển thị chữ đỏ dưới ô Phụ xe |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng chọn tuyến, xe, giờ, lái, phụ và nhấn Tạo chuyến và phân công | Client | • Lưu ý: mã chuyến \= C \+ yyMMdd \+ số thứ tự 3 chữ số trong ngày. VD: ngày 02/06/2025 đã có 2 chuyến → C250602003 |
|  |  |  | • Giờ đến dự kiến \= giờ đi \+ làm tròn 15 phút của (km ÷ 55 \+ 0,5) giờ (F09) |
|  |  |  | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin, gọi sp\_TaoChuyenVaPhanCong: |
|  |  |  | • BEGIN TRANSACTION; lái \= phụ → THROW 53001 (E14) |
|  |  |  | • INSERT ChuyenXe (TrangThai \= KeHoach) → TR04 kiểm tra: |
|  |  |  | ○ Xe không Hoạt động (E15) |
|  |  |  | ○ Xe trùng giờ chuyến khác chưa hủy (E16) |
|  |  |  | ○ Không có lần đăng kiểm Đạt bao trùm ngày khởi hành (E17) |
|  |  |  | ○ Ngày bảo dưỡng tiếp theo \< ngày khởi hành (E18) |
|  |  |  | • INSERT PhanCong LAI, PHU → UQ (chuyến, vai trò) (E20); TR03: tài xế đang làm, không trùng giờ (E19); TR05 tính thù lao F01 |
|  |  |  | • Lỗi bất kỳ → ROLLBACK, chuyến cũng không được lưu · hợp lệ → COMMIT |
|  |  | Client | • Thành công: thông báo xanh "Đã tạo chuyến {mã}. Sang trang Bán vé để bán vé cho chuyến này."; tự chọn chuyến mới ở MH03 và MH02.3 |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10, giữ nguyên lựa chọn |

### **6.3 MH02.2 Tab Danh sách, hủy chuyến**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Bảng chuyến | Table | Read Only | \-- | \-- | • Cột: Mã, Tuyến, Xe, Giờ đi, Giờ đến, Lái, Phụ, Đã bán, Trạng thái • Đã bán \= số vé Đã bán / số chỗ khách (F05) • Trạng thái: Kế hoạch / Đang chạy / Hoàn thành / Hủy; dòng Hủy chữ xám • Sắp xếp giờ đi mới nhất trên cùng |
| 2 | Button Hủy chuyến (mỗi dòng) | Button | Click | \-- | \-- | • Chỉ hiển thị ở dòng trạng thái Kế hoạch |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhấn Hủy chuyến | Client | • Gửi mã chuyến đến Server |
|  |  | Server | • Kiểm tra chuyến đang KeHoach; khác → lỗi "Chỉ hủy được chuyến đang Kế hoạch" |
|  |  |  | • UPDATE ChuyenXe SET TrangThai \= Huy; vé đã bán giữ nguyên trạng thái (hoàn tiền ngoài phạm vi) |
|  |  | Client | • Dòng chuyển chữ xám, ẩn nút Hủy; chuyến biến khỏi danh sách mở bán ở MH03 |

### **6.4 MH02.3 Tab Kết thúc chuyến**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Chuyến đang Kế hoạch | Combobox | Select | Y | \-- | • Danh sách chuyến trạng thái Kế hoạch: "{mã} · {dd/mm hh:mm} · {tuyến}" • Mặc định: chuyến vừa tạo hoặc chuyến đầu tiên |
| 2 | Số km thực tế | Textbox | Input | Y | 4 | • Số nguyên \> 0 |
| 3 | Chi phí vận hành (đ) | Textbox | Input | N | 12 | • Số ≥ 0 · mặc định 0 |
| 4 | Giờ đi, giờ đến thực tế | Datetimepicker | Select | Y (bản thật) | \-- | • Prototype: lấy bằng giờ dự kiến · bản thật: giờ đến \> giờ đi \[CẦN BỔ SUNG\] |
| 5 | Button Hoàn thành chuyến | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường chọn | Vui lòng chọn Chuyến | Hiển thị chữ đỏ dưới ô chọn |
| 2 | Km bỏ trống, không phải số hoặc ≤ 0 | Số km thực tế phải lớn hơn 0 | Hiển thị chữ đỏ dưới ô nhập |
| 3 | Chi phí âm hoặc không phải số | Chi phí phải là số không âm | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng chọn chuyến, nhập km, chi phí và nhấn Hoàn thành chuyến | Client | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin, gọi sp\_KetThucChuyen: |
|  |  |  | • BEGIN TRANSACTION; chuyến phải đang KeHoach hoặc DangChay |
|  |  |  | • UPDATE GioKhoiHanhTT, GioDenTT, KmThucTe, ChiPhiVanHanh, TrangThai \= HoanThanh |
|  |  |  | • TR06: đủ 1 lái \+ 1 phụ, có km và giờ thực tế; thiếu → ROLLBACK (E21) |
|  |  |  | • Hợp lệ → COMMIT |
|  |  | Client | • Thông báo xanh kèm 3 tác động: lương 2 tài xế (+ thù lao), doanh thu (+ vé đã bán), km làm việc của xe (F10) |
|  |  |  | • Chuyến biến khỏi danh sách Kế hoạch; cập nhật MH04, MH05 |

## **7\. MH03 Bán vé**

Menu VẬN HÀNH · Actor: NV bán vé · Use case UC-011, UC-012, UC-013. Gồm 3 tab: MH03.1 Bán vé, MH03.2 Hủy vé, MH03.3 Tra cứu vé.

### **7.1 Business rule MH03**

| Mã | Quy tắc | Áp dụng tại | Kiểm tra ở |
| :---- | :---- | :---- | :---- |
| BR-MH03-01 | Chỉ bán vé cho chuyến đang Kế hoạch | MH03.1 | sp\_BanVe, TR01 |
| BR-MH03-02 | Số ghế bán được \= số ghế của loại xe − 2 (ghế lái, ghế phụ xe); tổng vé Đã bán không vượt số chỗ | MH03.1 | TR01 |
| BR-MH03-03 | Mỗi ghế chỉ có 1 vé Đã bán; hai người bán cùng ghế cùng lúc thì chỉ 1 người thành công | MH03.1 | sp\_BanVe (UPDLOCK), UX\_Ve\_GheDangBan |
| BR-MH03-04 | Giá vé \= giá tuyến theo loại ngày của ngày khởi hành (F03, F04); lưu cố định trên vé | MH03.1 | TR02 |
| BR-MH03-05 | Mã vé \= MaTuyen-MaXe-MaChuyen-SoGhe-LanBan; lần bán \= số vé (kể cả đã hủy) của ghế \+ 1 | MH03.1 | TR02 |
| BR-MH03-06 | Khách nhận diện theo số điện thoại; chưa có thì tự tạo mới | MH03.1 | sp\_BanVe |
| BR-MH03-07 | Chỉ hủy vé Đã bán của chuyến còn Kế hoạch; vé không bị xóa, chỉ đổi trạng thái; ghế mở bán lại | MH03.2 | Client \+ Server |
| BR-MH03-08 | Hủy vé luôn đi kèm tra cứu vé trước | MH03.2 | – |

### **7.2 MH03.1 Tab Bán vé**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề | Label | Read Only | Y | \-- | textVIE: Bán vé |
| 2 | Thanh tab | Tab | Click | Y | \-- | • 3 tab: Bán vé / Hủy vé / Tra cứu vé · mặc định Bán vé |
| 3 | **Chọn chuyến** |  |  |  |  |  |
| 4 | Chuyến đang mở bán | Combobox | Select | Y | \-- | • Danh sách chuyến trạng thái Kế hoạch: "{mã} · {dd/mm hh:mm} · {tuyến}" • Mặc định: chuyến vừa tạo ở MH02, không có thì chuyến đầu tiên • Không có chuyến: "Chưa có chuyến nào đang mở bán. Hãy tạo chuyến ở trang Chuyến xe." |
| 5 | Dòng thông tin chuyến | Label | Read Only | \-- | \-- | • "{tuyến} · xe {biển số} ({loại xe}) · khởi hành {dd/mm/yyyy hh:mm} · đã bán {x}/{số chỗ}" • Hết chỗ thì thêm " — HẾT CHỖ |
| 6 | Dòng giá vé | Label nền xám | Read Only | \-- | \-- | • "Giá vé: ngày {dd/mm/yyyy} thuộc loại "{loại ngày}" → giá tuyến {mã} {loại ngày} \= {giá} đ." • Chưa có giá: "CHƯA CÓ (nhập ở Thiết lập › Tuyến và giá vé)" |
| 7 | Sơ đồ ghế | Lưới nút, 10 cột | Click | Y | \-- | • Ghế 1 → số ghế − 2 (F05) • Trắng: ghế trống, bấm được · Xám: đã bán, không bấm được · Đỏ: đang chọn • Chú thích dưới sơ đồ: "Ghế lái và ghế phụ xe không bán: {số ghế} − 2 \= {số chỗ} chỗ." |
| 8 | **Thông tin khách** |  |  |  |  |  |
| 9 | Họ tên khách | Textbox | Input | Y | 100 |  |
| 10 | Số điện thoại | Textbox | Input | Y | 10 | • 10 chữ số, bắt đầu bằng 0; dùng để tìm khách cũ |
| 11 | Button Bán vé | Button | Click | \-- | \-- | • Chưa chọn ghế: nhãn "Chọn ghế trên sơ đồ" • Đã chọn: nhãn "Bán vé ghế số {n}" |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Chưa chọn ghế | Chọn một ghế trống trên sơ đồ | Hiển thị chữ đỏ dưới sơ đồ ghế |
| 2 | Bỏ trống trường nhập | Vui lòng nhập \<tên\_trường\> (VD: Vui lòng nhập Họ tên khách) | Hiển thị chữ đỏ dưới ô nhập |
| 3 | Số điện thoại sai định dạng | Số điện thoại phải gồm 10 chữ số, bắt đầu bằng 0 | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng chọn chuyến | Client | • Gọi Server lấy thông tin chuyến, giá theo loại ngày và danh sách ghế đã bán; vẽ sơ đồ ghế |
| 2 | Người dùng chọn ghế, nhập khách và nhấn Bán vé | Client | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin, gọi sp\_BanVe: |
|  |  |  | • BEGIN TRANSACTION; khóa dòng chuyến (UPDLOCK, HOLDLOCK) |
|  |  |  | • Chuyến không tồn tại hoặc không Kế hoạch → lỗi (E25) |
|  |  |  | • Ghế đã có vé Đã bán → lỗi (E24) |
|  |  |  | • Tìm khách theo SĐT; chưa có thì INSERT HanhKhach (mã HK \+ 6 số) |
|  |  |  | • INSERT Ve → TR02 tính lần bán (F06), mã vé (F07), giá (F04); không có giá → lỗi (E27) |
|  |  |  | • TR01: ghế ≤ số ghế − 2, tổng vé ≤ số chỗ; vi phạm → lỗi (E26) |
|  |  |  | • Lỗi bất kỳ → ROLLBACK · hợp lệ → COMMIT, trả mã vé và giá |
|  |  | Client | • Thành công: thông báo xanh "Đã bán vé {mã vé} · {giá} đ ({loại ngày})."; ghế chuyển xám; xóa ô khách; cập nhật "đã bán x/y" |
|  |  |  | • Lỗi: hộp thông báo đỏ theo mục 10; tải lại sơ đồ ghế |

### **7.3 MH03.2 Tab Hủy vé**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Bảng vé của chuyến đang chọn | Table | Read Only | \-- | \-- | • Cột: Mã vé, Khách, SĐT, Ghế, Giá, Trạng thái • Chuyến lấy theo ô "Chuyến đang mở bán" ở tab Bán vé • Không có vé: "Chuyến này chưa có vé nào." |
| 2 | Button Hủy vé (mỗi dòng) | Button | Click | \-- | \-- | • Chỉ hiển thị ở vé Đã bán của chuyến Kế hoạch |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhấn Hủy vé | Client | • Gửi mã vé đến Server |
|  |  | Server | • Vé phải Đã bán và chuyến còn Kế hoạch; khác → lỗi |
|  |  |  | • UPDATE Ve SET TrangThai \= DaHuy (không xóa vé) |
|  |  | Client | • Thông báo xanh "Đã hủy vé {mã}. Ghế {n} được mở bán lại." |
|  |  |  | • Ghế trở lại trắng; lần bán sau của ghế có mã đuôi tăng (VD: \-2) |

### **7.4 MH03.3 Tab Tra cứu vé**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | **Điều kiện tìm kiếm** |  |  |  |  |  |
| 2 | Số điện thoại khách | Textbox | Input | Y | 10 | • Tìm theo chứa chuỗi, lọc ngay khi gõ |
| 3 | **Kết quả** |  |  |  |  |  |
| 4 | Bảng kết quả | Table | Read Only | \-- | \-- | • Cột: Mã vé, Khách, Chuyến, Ghế, Giá, Trạng thái • Không có kết quả: "Không tìm thấy vé nào." |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng nhập số điện thoại | Client | • Gửi chuỗi tìm kiếm đến Server (bỏ qua khi trống) |
|  |  | Server | • SELECT Ve JOIN HanhKhach WHERE SoDienThoai LIKE '%{chuỗi}%'; chỉ đọc |
|  |  | Client | • Hiển thị bảng kết quả; vé Đã hủy ghi rõ trạng thái |

## **8\. MH04 Kỹ thuật xe**

Menu VẬN HÀNH · Actor: NV kỹ thuật · Use case UC-014, UC-015. Màn hình gồm 4 khung trên cùng một trang: MH04.1 Xe cần xử lý, MH04.2 Ghi nhận bảo dưỡng, MH04.3 Gia hạn đăng kiểm, MH04.4 Lịch bảo dưỡng và đăng kiểm.

### **8.1 Business rule MH04**

| Mã | Quy tắc | Áp dụng tại | Kiểm tra ở |
| :---- | :---- | :---- | :---- |
| BR-MH04-01 | Ngày bảo dưỡng tiếp theo \= ngày bảo dưỡng gần nhất \+ CHU\_KY\_BD − ⌊km làm việc ÷ KM\_MOI\_NGAY\_BD⌋ (F10, F11); tính khi xem, không lưu | MH04.1, MH04.4 | vw\_LichBaoDuong |
| BR-MH04-02 | Tình trạng: quá hạn khi hạn \< hôm nay (đỏ); sắp đến hạn khi còn ≤ CANH\_BAO\_NGAY ngày (vàng); chưa có đăng kiểm là đỏ (F14) | MH04.1, MH04.4 | Client |
| BR-MH04-03 | "Xe cần xử lý" chỉ gồm xe khác Ngừng hoạt động có tình trạng đỏ hoặc vàng; mỗi xe tối đa 2 dòng (bảo dưỡng, đăng kiểm) | MH04.1 | Client |
| BR-MH04-04 | Ngày bảo dưỡng ≤ hôm nay, \> ngày bảo dưỡng gần nhất, không trùng ngày đã có; chi phí ≥ 0 | MH04.2 | sp\_GhiNhanBaoDuong, PK, CK |
| BR-MH04-05 | Ghi nhận bảo dưỡng xong: km làm việc tính lại từ 0; xe đang Bảo dưỡng chuyển về Hoạt động | MH04.2 | sp\_GhiNhanBaoDuong |
| BR-MH04-06 | Ngày đăng kiểm ≤ hôm nay; chi phí ≥ 0 | MH04.3 | sp\_GhiNhanDangKiem, CK |
| BR-MH04-07 | Đạt: hạn \= ngày \+ chu kỳ theo tuổi xe (F12); hạn hiện hành \= hạn lớn nhất, đăng kiểm sớm không làm ngắn hạn. Không đạt: NgayHetHan để trống, hạn hiện hành giữ nguyên | MH04.3 | sp\_GhiNhanDangKiem, CK\_DK\_Ngay |
| BR-MH04-08 | Còn hạn trên 30 ngày vẫn cho gia hạn, chỉ hiện lưu ý | MH04.3 | Client |

### **8.2 MH04.1 Xe cần xử lý**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề | Label | Read Only | Y | \-- | textVIE: Kỹ thuật xe |
| 2 | Mô tả phụ | Label | Read Only | Y | \-- | textVIE: UC10 Ghi nhận bảo dưỡng · UC11 Gia hạn đăng kiểm — ngày hiện tại {dd/mm/yyyy} |
| 3 | Tiêu đề khung | Label | Read Only | Y | \-- | textVIE: Xe cần xử lý — quá hạn hoặc còn ≤ {CANH\_BAO\_NGAY} ngày (bấm Ghi nhận để điền sẵn form) |
| 4 | Bảng xe cần xử lý | Table | Read Only | \-- | \-- | • Cột: Xe, Biển số, Việc cần làm (Bảo dưỡng / Đăng kiểm), Hạn, Tình trạng, Ảnh hưởng • Tình trạng: "Quá hạn {n} ngày" (dòng đỏ) / "Còn {n} ngày" (dòng vàng) / "Chưa đăng kiểm" (dòng đỏ) • Ảnh hưởng: "Bị chặn xếp chuyến" (đỏ) / "Sắp bị chặn" (vàng) • Ảnh hưởng: "Bị chặn xếp chuyến" (đỏ) / "Sắp bị chặn" (vàng) • Sắp xếp: dòng đỏ trước, rồi theo hạn tăng dần • Sắp xếp: dòng đỏ trước, rồi theo hạn tăng dần |
| 5 | Button Ghi nhận (mỗi dòng) | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng mở MH04 | Client | • Gọi Server lấy lịch của tất cả xe |
|  |  | Server | • SELECT vw\_LichBaoDuong (NgayBDGanNhat, KmLamViec, NgayBDTiepTheo, HanDangKiem) và ThamSo.CANH\_BAO\_NGAY |
|  |  | Client | • Tính tình trạng F14 cho từng xe; dựng bảng Xe cần xử lý và bảng lịch MH04.4 |
| 2 | Người dùng nhấn Ghi nhận ở một dòng | Client | • Dòng Bảo dưỡng: điền xe và ngày hôm nay vào form MH04.2 |
|  |  |  | • Dòng Đăng kiểm: điền xe, ngày hôm nay, kết quả Đạt vào form MH04.3 |
|  |  |  | • Thông báo: "Đã chọn xe {biển số} ở form … Kiểm tra rồi bấm Lưu."; không gọi Server |

### **8.3 MH04.2 Ghi nhận bảo dưỡng**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề khung | Label | Read Only | Y | \-- | textVIE: Ghi nhận bảo dưỡng |
| 2 | Xe | Combobox | Select | Y | \-- | • Danh sách tất cả xe: "{mã} · {biển số} · {trạng thái}" • Mặc định: xe đầu tiên trong Xe cần xử lý, nếu có |
| 3 | Ngày bảo dưỡng | Datepicker | Select | Y | \-- | • Định dạng dd/mm/yyyy • Giới hạn chọn lịch: không chọn được ngày sau hôm nay • Mặc định: hôm nay |
| 4 | Chi phí (đ) | Textbox | Input | N | 12 | • Số ≥ 0 |
| 5 | Nội dung | Textbox | Input | N | 200 | • Placeholder: VD Bảo dưỡng định kỳ, kiểm tra phanh |
| 6 | Khung xem trước | Label nền xám | Read Only | \-- | \-- | • "Hạn hiện tại: {F11} ({tình trạng}). Sau khi lưu: hạn mới \= {ngày} \+ {CHU\_KY\_BD} ngày \= {ngày mới}, km làm việc về 0." • Tính lại khi đổi Xe hoặc Ngày |
| 7 | Button Lưu bảo dưỡng | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường chọn | Vui lòng chọn \<tên\_trường\> (VD: Vui lòng chọn Ngày bảo dưỡng) | Hiển thị chữ đỏ dưới ô chọn |
| 2 | Chi phí không phải số hoặc âm | Chi phí phải là số không âm | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng chọn xe, ngày, nhập chi phí, nội dung và nhấn Lưu bảo dưỡng | Client | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin, gọi sp\_GhiNhanBaoDuong: |
|  |  |  | • BEGIN TRANSACTION |
|  |  |  | • Ngày \> hôm nay → THROW 55001 (E28) |
|  |  |  | • Đã có lần bảo dưỡng sau ngày này → THROW 55002 (E29) |
|  |  |  | • INSERT BaoDuong; trùng (xe, ngày) → PK\_BaoDuong (E30); chi phí âm → CK (E31) |
|  |  |  | • Xe đang BaoDuong → UPDATE TrangThai \= HoatDong |
|  |  |  | • Lỗi bất kỳ → ROLLBACK · hợp lệ → COMMIT |
|  |  | Client | • Thông báo xanh "Đã ghi nhận bảo dưỡng xe {biển số} ngày {ngày}. Hạn bảo dưỡng tiếp theo: {ngày mới}. Xe được xếp chuyến trở lại." |
|  |  |  | • Xe biến khỏi Xe cần xử lý; cập nhật MH04.4 và Mức độ sẵn sàng ở MH01 |

### **8.4 MH04.3 Gia hạn đăng kiểm định kỳ**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề khung | Label | Read Only | Y | \-- | textVIE: Gia hạn đăng kiểm định kỳ |
| 2 | Xe | Combobox | Select | Y | \-- | • Danh sách tất cả xe: "{mã} · {biển số} · {trạng thái}" |
| 3 | Ngày đăng kiểm | Datepicker | Select | Y | \-- | • Định dạng dd/mm/yyyy · không chọn được ngày sau hôm nay · mặc định hôm nay |
| 4 | Kết quả | Combobox | Select | Y | \-- | • Cho phép chọn 1 giá trị trong danh sách: ○ Đạt ○ Không đạt • Mặc định: Đạt |
| 5 | Chi phí (đ) | Textbox | Input | N | 12 | • Số ≥ 0 |
| 6 | Khung xem trước | Label nền xám | Read Only | \-- | \-- | • Đạt: "Hạn hiện tại: {hạn}. Xe sản xuất {năm} ({tuổi} năm, xe cũ/xe mới) → chu kỳ {n} tháng → hạn mới {ngày}." • Còn hạn \> 30 ngày: thêm "Lưu ý: xe còn hạn đến {hạn}, hạn hiện hành sẽ là ngày lớn hơn." • Không đạt: "Hạn hiện tại: {hạn}. Kết quả Không đạt: hạn giữ nguyên, xe phải sửa và đăng kiểm lại." |
| 7 | Button Lưu đăng kiểm | Button | Click | \-- | \-- | Xem logic xử lý mục 1 |

**Bảng kiểm tra dữ liệu phía Client**

| \# | Trường hợp | Mô tả | Xử lý (nếu có) |
| :---- | :---- | :---- | :---- |
| 1 | Bỏ trống trường chọn | Vui lòng chọn \<tên\_trường\> (VD: Vui lòng chọn Kết quả) | Hiển thị chữ đỏ dưới ô chọn |
| 2 | Chi phí không phải số hoặc âm | Chi phí phải là số không âm | Hiển thị chữ đỏ dưới ô nhập |

**Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng chọn xe, ngày, kết quả và nhấn Lưu đăng kiểm | Client | • Check validate theo bảng kiểm tra dữ liệu; hợp lệ thì gửi Server |
|  |  | Server | Server nhận thông tin, gọi sp\_GhiNhanDangKiem: |
|  |  |  | • BEGIN TRANSACTION; ngày \> hôm nay → THROW 59001 (E09) |
|  |  |  | • Kết quả Đạt: tính chu kỳ theo tuổi xe (F12); INSERT DangKiem (NgayHetHan \= ngày \+ chu kỳ, Dat) |
|  |  |  | • Kết quả Không đạt: INSERT DangKiem (NgayHetHan \= NULL, KhongDat) |
|  |  |  | • CK\_DK\_Ngay bảo đảm hạn khớp kết quả; lỗi → ROLLBACK · hợp lệ → COMMIT |
|  |  | Client | • Đạt: thông báo xanh "Đã ghi nhận đăng kiểm Đạt. Xe {biển số} được chạy đến {hạn hiện hành}." |
|  |  |  | • Không đạt: thông báo "Đã ghi kết quả Không đạt. Hạn đăng kiểm của xe {biển số} giữ nguyên {hạn}." |
|  |  |  | • Cập nhật Xe cần xử lý, MH04.4, Mức độ sẵn sàng |

### **8.5 MH04.4 Lịch bảo dưỡng và đăng kiểm**

**Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề khung | Label | Read Only | Y | \-- | textVIE: Lịch bảo dưỡng, đăng kiểm của tất cả xe |
| 2 | Bảng lịch | Table | Read Only | \-- | \-- | • Cột: Xe, Biển số, Trạng thái, BD gần nhất, Km làm việc, BD tiếp theo, Tình trạng BD, Hạn đăng kiểm, Tình trạng ĐK • Dòng đỏ: quá hạn hoặc chưa đăng kiểm · dòng vàng: sắp đến hạn |
| 3 | Dòng công thức | Label | Read Only | \-- | \-- | • "Bảo dưỡng tiếp theo \= ngày BD gần nhất \+ {CHU\_KY\_BD} − ⌊km làm việc ÷ {KM\_MOI\_NGAY\_BD}⌋ · hệ số đường khó {1/2/3} · đỏ \= quá hạn, vàng \= còn ≤ {CANH\_BAO\_NGAY} ngày" |
|  |  |  |  |  |  | • Giá trị lấy theo tham số hiện hành ở MH01.1 |

Logic xử lý của MH04.4 dùng chung mục 1 của MH04.1 (chỉ đọc).

## **9\. MH05 Báo cáo**

Menu VẬN HÀNH · Actor: Quản lý chi nhánh · Use case UC-016, UC-017. Gồm điều kiện tìm kiếm, 3 ô số tổng và 5 tab báo cáo.

### **9.1 Business rule MH05**

| Mã | Quy tắc | Áp dụng tại | Kiểm tra ở |
| :---- | :---- | :---- | :---- |
| BR-MH05-01 | Chỉ tính chuyến Hoàn thành; doanh thu chỉ tính vé Đã bán; gom theo tháng của ngày khởi hành dự kiến | Mọi báo cáo | sp\_R01–R04 |
| BR-MH05-02 | Lương tháng \= tổng thù lao đã lưu của các phân công thuộc chuyến Hoàn thành trong tháng (F02); không tính lại theo tham số mới | Tab Lương | sp\_R01 |
| BR-MH05-03 | Doanh thu theo xe chỉ liệt kê xe đang Hoạt động | Tab Doanh thu theo xe | sp\_R02 |
| BR-MH05-04 | Xe quá hạn bảo dưỡng: ngày bảo dưỡng tiếp theo \< ngày xét | Tab Xe quá hạn | sp\_R06 |
| BR-MH05-05 | Chốt lương: xóa lương cũ của tháng rồi ghi mới cho mọi tài xế (kể cả 0 đồng) trong 1 giao dịch; số đã chốt không đổi khi dữ liệu sửa sau | Nút Chốt lương | sp\_ChotLuongThang |
| BR-MH05-06 | Chi nhánh để trống \= toàn chuỗi | Mọi báo cáo | Server |

### **9.2 Mô tả màn hình**

| \# | Hạng mục | Kiểu hiển thị | Kiểu thao tác | Bắt buộc | Độ dài | Mô tả |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Tiêu đề | Label | Read Only | Y | \-- | textVIE: Báo cáo |
| 2 | **Điều kiện tìm kiếm** |  |  |  |  |  |
| 3 | Tháng | Combobox | Select | Y | \-- | • Danh sách các tháng có chuyến, định dạng yyyy-mm • Mặc định: tháng hiện tại (prototype cố định 2025-06) |
| 4 | Chi nhánh | Combobox | Select | N | \-- | • Cho phép chọn 1 giá trị trong danh sách: ○ Tất cả ○ Hà Nội ○ Đà Nẵng ○ TP.HCM • Mặc định: Tất cả |
| 5 | Ngày xét quá hạn | Datepicker | Select | Y | \-- | • Chỉ dùng cho tab Xe quá hạn · mặc định hôm nay (demo dữ liệu mẫu: 31/05/2025) |
| 6 | **Số tổng** |  |  |  |  |  |
| 7 | Ô Doanh thu | Label dạng thẻ | Read Only | \-- | \-- | • "Doanh thu tháng (chuyến hoàn thành)": tổng F15, định dạng \#.\#\#\# đ |
| 8 | Ô Số vé | Label dạng thẻ | Read Only | \-- | \-- | • "Số vé tính doanh thu": số vé Đã bán của chuyến Hoàn thành |
| 9 | Ô Tổng lương | Label dạng thẻ | Read Only | \-- | \-- | • "Tổng lương tài xế": tổng F02 |
| 10 | **Báo cáo chi tiết** |  |  |  |  |  |
| 11 | Thanh tab | Tab | Click | Y | \-- | • 5 tab: Lương tài xế / Doanh thu theo xe / Doanh thu theo tuyến / Doanh thu theo loại xe / Xe quá hạn bảo dưỡng • Mặc định: Lương tài xế |
| 12 | Bảng Lương tài xế | Table | Read Only | \-- | \-- | • Cột: Mã TX, Họ tên, Số chuyến lái, Số chuyến phụ, Lương (đ); sắp xếp lương giảm dần |
| 13 | Button Chốt lương tháng | Button | Click | \-- | \-- | • Nhãn: "Chốt lương tháng {yyyy-mm}" · xem logic xử lý mục 2 |
| 14 | Dòng trạng thái chốt | Label | Read Only | \-- | \-- | • "BangLuong đang có {n} dòng của tháng {yyyy-mm} (đã chốt)" hoặc "Tháng {yyyy-mm} chưa chốt lương" |
| 15 | Biểu đồ doanh thu theo xe / tuyến / loại xe | Biểu đồ thanh ngang | Read Only | \-- | \-- | • Mỗi dòng: tên, số chuyến, số vé, thanh dài tỷ lệ doanh thu, số tiền • Sắp xếp doanh thu giảm dần |
| 16 | Bảng Xe quá hạn bảo dưỡng | Table chữ đỏ | Read Only | \-- | \-- | • Cột: Xe, Biển số, Ngày bảo dưỡng tiếp theo, Số ngày quá hạn; sắp xếp số ngày quá hạn giảm dần • Không có xe: "Không có xe nào quá hạn tại ngày xét." |

### **9.3 Logic xử lý**

| \# | Thao tác | Tác nhân | Mô tả |
| :---- | :---- | :---- | :---- |
| 1 | Người dùng mở MH05, đổi Tháng, Chi nhánh hoặc chọn tab | Client | • Gửi Tháng, Chi nhánh (NULL nếu Tất cả), tab đang mở đến Server |
|  |  | Server | • Ô số tổng: tổng F15 và F02 theo tháng, chi nhánh |
|  |  |  | • Tab Lương: sp\_R01 @Thang, @MaCN |
|  |  |  | • Tab Doanh thu: sp\_R02 / sp\_R03 / sp\_R04 @Thang, @MaCN |
|  |  |  | • Tab Xe quá hạn: sp\_R06 @NgayXet, @MaCN |
|  |  | Client | • Vẽ bảng hoặc biểu đồ; lỗi kết nối → hộp đỏ E32 |
| 2 | Người dùng nhấn Chốt lương tháng | Client | • Gửi Tháng đến Server (bản thật nên hỏi xác nhận khi tháng đã chốt trước đó) |
|  |  | Server | Server nhận thông tin, gọi sp\_ChotLuongThang: |
|  |  |  | • BEGIN TRANSACTION |
|  |  |  | • DELETE BangLuong của tháng |
|  |  |  | • INSERT 1 dòng cho mỗi tài xế: số chuyến lái, số chuyến phụ, tổng lương (F02), ngày chốt |
|  |  |  | • Lỗi → ROLLBACK, bảng lương tháng giữ nguyên như trước · hợp lệ → COMMIT |
|  |  | Client | • Thông báo xanh; cập nhật dòng trạng thái chốt "BangLuong đang có {n} dòng…" |

## **10\. Danh mục thông báo lỗi**

Lỗi từ CSDL (CK, TRG, SP) được giao diện bắt lại và hiện bằng câu tiếng Việt bên dưới, không hiện nguyên văn mã lỗi SQL. Cột "Mã SQL" là mã THROW hoặc tên ràng buộc để đối chiếu khi kiểm thử.

| Mã | Tình huống | Kiểm tra ở | Mã SQL | Tiếng Việt | English |
| :---- | :---- | :---- | :---- | :---- | :---- |
| E01 | Bỏ trống trường bắt buộc | FE | – | Vui lòng nhập {tên trường}. | Please enter {field}. |
| E02 | Tham số hoặc giá ≤ 0 | FE, CK | CK\_Gia\_GiaVe, CK\_DoPhucTap\_\* | Giá trị phải lớn hơn 0\. | Value must be greater than 0\. |
| E03 | Hệ số đường khó \< 1 | FE, CK | CK\_DoPhucTap\_HSDK | Hệ số đường khó phải từ 1 trở lên. | Road difficulty factor must be at least 1\. |
| E04 | Điểm đầu trùng điểm cuối | CK | CK\_Tuyen\_DiemKhacNhau | Điểm đầu và điểm cuối phải khác nhau. | Origin and destination must be different. |
| E05 | Độ dài tuyến ≤ 0 | CK | CK\_Tuyen\_DoDai | Độ dài tuyến phải lớn hơn 0 km. | Route length must be greater than 0 km. |
| E06 | Thiếu giá khi thêm tuyến | SP | sp\_ThemTuyenVaGia | Nhập đủ 4 mức giá lớn hơn 0\. Tuyến chưa được lưu. | Enter all 4 prices above 0\. The route was not saved. |
| E07 | Biển số trùng | CK | UQ\_Xe\_BienSo | Biển số {biển số} đã tồn tại. | Plate number {plate} already exists. |
| E08 | Năm sản xuất không hợp lệ | CK | CK\_Xe\_NamSX | Năm sản xuất phải từ 2005 đến năm hiện tại. | Manufacturing year must be between 2005 and this year. |
| E09 | Ngày đăng kiểm ở tương lai | SP | sp\_ThemXeVaDangKiem, sp\_GhiNhanDangKiem | Ngày đăng kiểm không được sau hôm nay. | Inspection date cannot be in the future. |
| E10 | Xóa xe đã có chuyến | CK | FK\_Chuyen\_Xe | Không xóa được xe đã có chuyến. Hãy đổi trạng thái sang Ngừng hoạt động. | This vehicle has trips and cannot be deleted. Set it to Inactive instead. |
| E11 | CCCD sai định dạng | FE | – | CCCD phải gồm đúng 12 chữ số. | Citizen ID must have exactly 12 digits. |
| E12 | CCCD trùng | CK | UQ\_TX\_CCCD | CCCD đã tồn tại. | Citizen ID already exists. |
| E13 | Tài xế dưới 21 tuổi | CK | CK\_TX\_Tuoi | Tài xế phải từ 21 tuổi trở lên. | Driver must be at least 21 years old. |
| E14 | Lái xe trùng phụ xe | FE, SP | 53001 | Lái xe và phụ xe phải là 2 người khác nhau. | Driver and assistant must be different people. |
| E15 | Xe không hoạt động | TRG TR04 | 50401 | Xe không ở trạng thái Hoạt động. | Vehicle is not active. |
| E16 | Xe trùng giờ | TRG TR04 | 50402 | Xe đã có chuyến khác trùng giờ. | Vehicle already has an overlapping trip. |
| E17 | Xe hết hạn đăng kiểm | TRG TR04 | 50403 | Xe chưa đăng kiểm hoặc đã hết hạn tại ngày khởi hành. | Vehicle inspection is missing or expired on the departure date. |
| E18 | Xe quá hạn bảo dưỡng | TRG TR04 | 50404 | Xe đã quá hạn bảo dưỡng tại ngày khởi hành. | Vehicle maintenance is overdue on the departure date. |
| E19 | Tài xế trùng giờ | TRG TR03 | 50302 | Tài xế {mã} đã có chuyến trùng giờ. Chuyến chưa được tạo. | Driver {id} has an overlapping trip. The trip was not created. |
| E20 | Thêm lái/phụ thứ 2 | CK | UQ\_PC\_ChuyenVaiTro | Chuyến đã có đủ lái xe và phụ xe. | This trip already has a driver and an assistant. |
| E21 | Kết thúc chuyến thiếu điều kiện | FE, TRG TR06 | 50601 | Cần có số km lớn hơn 0 và đủ 1 lái, 1 phụ để hoàn thành chuyến. | Enter distance above 0 and assign a driver and an assistant to complete the trip. |
| E22 | Chưa chọn ghế | FE | – | Chọn một ghế trống trên sơ đồ. | Select an available seat on the map. |
| E23 | SĐT khách sai | FE | – | Số điện thoại phải gồm 10 chữ số, bắt đầu bằng 0\. | Phone number must be 10 digits starting with 0\. |
| E24 | Ghế đã có người | SP | 51003 | Ghế đã có người đặt. Vui lòng chọn ghế khác. | This seat is taken. Please choose another seat. |
| E25 | Chuyến không mở bán | SP, TRG TR01 | 51001, 51002, 50102 | Chuyến không còn mở bán. | This trip is no longer open for sale. |
| E26 | Vượt số chỗ | TRG TR01 | 50101, 50103 | Chuyến đã hết chỗ. | This trip is fully booked. |
| E27 | Tuyến chưa có giá | TRG TR02 | 50201 | Tuyến chưa có giá cho loại ngày này. Nhập ở Thiết lập › Tuyến và giá vé. | No fare set for this route on this day type. Add it in Setup › Routes and fares. |
| E28 | Ngày bảo dưỡng ở tương lai | SP | sp\_GhiNhanBaoDuong | Ngày bảo dưỡng không được sau hôm nay. | Maintenance date cannot be in the future. |
| E29 | Ngày bảo dưỡng trước lần gần nhất | SP | sp\_GhiNhanBaoDuong | Ngày bảo dưỡng phải sau lần gần nhất ({ngày}). | Maintenance date must be after the last one ({date}). |
| E30 | Trùng ngày bảo dưỡng | CK | PK\_BaoDuong | Xe đã có lần bảo dưỡng ngày {ngày}. | This vehicle already has a maintenance record on {date}. |
| E31 | Chi phí âm | CK | CK\_BD\_ChiPhi, CK\_DK\_ChiPhi, CK\_Chuyen\_ChiPhi | Chi phí phải là số không âm. | Cost must be zero or greater. |
| E32 | Mất kết nối, quá thời gian | FE | – | Không kết nối được cơ sở dữ liệu. Vui lòng thử lại. | Cannot connect to the database. Please try again. |

## **11\. Tác động dữ liệu và vấn đề cần bổ sung**

### **11.1 Bảng bị ghi theo use case**

| Use case | Bảng ghi | Kiểu ghi | Not null quan trọng |
| :---- | :---- | :---- | :---- |
| UC-002 | ThamSo, DoPhucTap | UPDATE | GiaTri, HeSoLuong, HeSoDuongKho |
| UC-003 | TuyenDuong, BangGiaVe | INSERT 1 \+ 4 | DiemDau, DiemCuoi, DoDaiKm, CapDo, GiaVe |
| UC-004 | BangGiaVe | MERGE | GiaVe |
| UC-005 | Xe, DangKiem | INSERT 1 \+ 1 | BienSo, MaLoaiXe, MaCN, NgayDuaVaoSD; NgayDangKiem, KetQua |
| UC-006 | Xe | DELETE | – |
| UC-007 | TaiXe | INSERT | HoDem, Ten, NgaySinh, CCCD, SoGPLX, HangGPLX, NgayVaoLam, MaCN |
| UC-008 | ChuyenXe, PhanCong | INSERT 1 \+ 2 | MaTuyen, MaXe, GioKhoiHanhDK, GioDenDK; VaiTro, ThuLao |
| UC-009 | ChuyenXe | UPDATE TrangThai | – |
| UC-010 | ChuyenXe | UPDATE | KmThucTe, GioKhoiHanhTT, GioDenTT khi Hoàn thành |
| UC-011 | HanhKhach (nếu mới), Ve | INSERT | SoDienThoai; MaVe, SoGhe, LanBan, GiaVe |
| UC-012 | Ve | UPDATE TrangThai | – |
| UC-014 | BaoDuong, Xe | INSERT \+ UPDATE | MaXe, NgayBaoDuong |
| UC-015 | DangKiem | INSERT | MaXe, NgayDangKiem, KetQua |
| UC-017 | BangLuong | DELETE \+ INSERT | MaTX, Thang, TongLuong |

### **11.2 Vấn đề cần bổ sung**

| \# | Nội dung | Ảnh hưởng | Trạng thái |
| :---- | :---- | :---- | :---- |
| 1 | File SQL chưa có sp\_ThemTuyenVaGia, sp\_ThemXeVaDangKiem, sp\_GhiNhanDangKiem | UC-003, UC-005, UC-015 chưa chạy được trên SQL Server thật | Đã bổ sung (15/10/2026) |
| 2 | sp\_GhiNhanBaoDuong chưa kiểm tra ngày tương lai và ngày trước lần gần nhất | E28, E29 chưa được chặn ở CSDL | Đã bổ sung (15/10/2026) |
| 3 | ThamSo thiếu 4 khóa CANH\_BAO\_NGAY, CHU\_KY\_DK\_CU, CHU\_KY\_DK\_MOI, TUOI\_XE\_CU | Tab 1 không lưu được các tham số này | Đã bổ sung (15/10/2026) |
| 4 | DangKiem.NgayHetHan đang NOT NULL | Kết quả Không đạt chưa lưu đúng (4b ở UC-015) | Đã bổ sung: NgayHetHan cho phép trống khi Không đạt (CK\_DK\_Ngay) |
| 5 | Chưa có chức năng sửa, xóa tuyến và tài xế; sửa thông tin xe | Đề yêu cầu thêm/xóa/sửa cho xe, tài xế, tuyến | \[CẦN BỔ SUNG\] |
| 6 | Giờ khởi hành, tháng báo cáo, ngày hiện tại đang cố định trong prototype | Chỉ minh họa; bản thật cần chọn tự do | \[CẦN BỔ SUNG\] |
| 7 | Giờ đi, giờ đến thực tế chưa có ô nhập khi kết thúc chuyến | Đang lấy bằng giờ dự kiến | \[CẦN BỔ SUNG\] |
| 8 | Chưa có danh sách nhắc "chuyến quá giờ chưa kết thúc" | Dễ quên bấm Hoàn thành chuyến, sót lương và doanh thu | Đề xuất, chờ chốt |
| 9 | Không có đăng nhập, phân quyền; nhân viên bán vé đang cố định | MaNVBan trên vé chưa đúng người thao tác | Ngoài phạm vi đã chốt |
| 10 | Đổi vé (sp\_DoiVe) không có giao diện; hủy chuyến không hoàn tiền vé | Chỉ demo bằng SQL | Ngoài phạm vi đã chốt |

