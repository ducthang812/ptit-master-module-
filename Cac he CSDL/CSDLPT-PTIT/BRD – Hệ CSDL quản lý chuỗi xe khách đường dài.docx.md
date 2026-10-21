# **BRD – Hệ CSDL quản lý chuỗi xe khách đường dài**

*BRD (tài liệu yêu cầu nghiệp vụ) này gom toàn bộ yêu cầu đã chốt cho hệ CSDL quản lý chuỗi 3 chi nhánh xe khách: 7 mục tiêu đo được, 44 yêu cầu (10 dữ liệu, 14 quy tắc, 12 chức năng, 8 yêu cầu giáo viên), 10 yêu cầu phi chức năng, ma trận truy vết tới ca kiểm thử và 10 kịch bản nghiệm thu. Đây là căn cứ để thiết kế, viết báo cáo và vận hành trước giáo viên.*

## **1\. Thông tin tài liệu**

| Mục | Nội dung |
| :---- | :---- |
| Tên dự án | Hệ CSDL quản lý chuỗi hãng vận tải hành khách đường dài (Đề tài 1, BTL môn Các hệ thống CSDL) |
| Mã báo cáo | \[M26CQHT02\] nhóm \[X\]\_QLXeKhach |
| Phiên bản | 1.1 |
| Người soạn | Phạm Thị Hồng Nhung |
| Người duyệt | Giảng viên môn học |
| Tài liệu liên quan | URD CSDL – Chuỗi hãng xe khách đường dài; BaoCao\_BTL\_QLXeKhach.docx; QLXeKhach\_SQLServer.sql; QLXeKhach\_DuLieu.sql; QLXeKhach\_KiemThu.sql; CSDL\_XeKhach\_SoDo.drawio |

**Lịch sử thay đổi**

| Phiên bản | Ngày | Nội dung thay đổi |
| :---- | :---- | :---- |
| 1.1 | 08/10/2026 | Bổ sung bối cảnh, mục tiêu, các bên liên quan, phạm vi, quy trình, đặc tả báo cáo, yêu cầu phi chức năng, ma trận truy vết, kịch bản nghiệm thu, rủi ro |
| 1.0 | 08/10/2026 | Bảng yêu cầu dữ liệu, quy tắc, chức năng, yêu cầu giáo viên |

## **2\. Giới thiệu**

**Mục đích:** Mô tả đầy đủ nhu cầu nghiệp vụ của chuỗi xe khách mà hệ CSDL phải đáp ứng, làm căn cứ cho thiết kế E-R, cài đặt SQL, ứng dụng Python và nghiệm thu khi vận hành trước giáo viên.

**Đối tượng đọc:** thành viên nhóm (thiết kế, lập trình, kiểm thử) và giảng viên chấm bài.

**Thuật ngữ**

| Thuật ngữ | Giải thích |
| :---- | :---- |
| Chuyến xe | Một lần xe chạy một tuyến tại một giờ khởi hành cụ thể |
| Lái xe / phụ xe | Hai vai trò tài xế trên một chuyến; phụ xe hưởng 60% thù lao lái xe |
| Số chỗ khách | Số ghế của xe trừ 2 (ghế lái và ghế phụ xe) |
| Lần bán (LanBan) | Thứ tự bán của một ghế trong chuyến; ghế của vé đã hủy được bán lại với lần bán tiếp theo |
| Độ phức tạp tuyến | Cấp 1 (đường bằng), 2 (đèo nhẹ), 3 (đèo dốc); quyết định hệ số lương và hệ số đường khó |
| Km làm việc | Km thực tế × hệ số đường khó; dùng để rút ngắn chu kỳ bảo dưỡng |
| Loại ngày | Ngày thường, cuối tuần, mùa du lịch, lễ tết; quyết định giá vé |
| Chốt lương | Ghi lương tháng vào bảng BangLuong để không thay đổi khi dữ liệu sửa sau |
| Trigger | Đoạn code tự chạy khi thêm, sửa, xóa dữ liệu để kiểm tra hoặc tự tính |
| Transaction | Nhóm thao tác chạy trọn vẹn hoặc hủy toàn bộ (ROLLBACK) khi có lỗi |
| BCNF | Dạng chuẩn mà mọi phụ thuộc hàm đều có vế trái là khóa |
| Phân mảnh | Chia bảng thành các phần đặt ở các chi nhánh khác nhau (CSDL phân tán) |

## **3\. Bối cảnh và vấn đề hiện tại**

Chuỗi hãng xe khách vận hành 12 tuyến đường dài từ 3 chi nhánh, với 30 xe thuộc 4 loại (9, 16, 29, 45 ghế) và 40 tài xế. Khi quản lý thủ công hoặc rời rạc, chuỗi gặp 6 vấn đề sau; mỗi vấn đề ứng với một nhóm yêu cầu ở mục 8\.

| \# | Vấn đề | Hậu quả | Yêu cầu giải quyết |
| :---- | :---- | :---- | :---- |
| 1 | Bán vé không kiểm soát số chỗ | Bán vượt số ghế, trùng ghế, khách phải đứng hoặc xuống xe | QT01, QT02, CN05 |
| 2 | Xếp lịch thủ công | Một tài xế hoặc một xe bị xếp 2 chuyến chồng giờ | QT03, QT04, CN04 |
| 3 | Theo dõi bảo dưỡng theo ngày cố định | Xe chạy tuyến đèo mòn nhanh nhưng vẫn chờ đủ kỳ mới bảo dưỡng; xe quá hạn đăng kiểm vẫn chạy | QT05, QT11, CN06, CN10, CN11 |
| 4 | Tính lương tài xế bằng tay | Sai vai trò lái/phụ, sai hệ số tuyến, tranh chấp khi trả lương | QT09, QT10, CN07 |
| 5 | Giá vé áp không thống nhất | Ngày lễ, mùa du lịch mỗi quầy bán một giá | QT08, DL07 |
| 6 | Không có số liệu doanh thu theo xe, tuyến, loại xe | Không biết xe, tuyến nào hiệu quả để điều phối | QT12, CN08, CN09 |

## **4\. Mục tiêu nghiệp vụ và chỉ số thành công**

| Mã | Mục tiêu | Chỉ số thành công | Cách đo |
| :---- | :---- | :---- | :---- |
| MT01 | Không bán vượt chỗ, không trùng ghế | 0 chuyến có số vé đã bán \> số ghế − 2; 0 ghế có 2 vé hiệu lực | Câu kiểm tra 01–02 cuối file dữ liệu; ca C02–C04 |
| MT02 | Xếp lịch đúng | 0 tài xế, 0 xe chạy trùng giờ; 100% chuyến hoàn thành có đúng 1 lái \+ 1 phụ | Câu kiểm tra 03–05; ca B02, B05, B06, B07 |
| MT03 | Xe chạy an toàn | 0 chuyến khởi hành khi xe hết hạn đăng kiểm hoặc quá hạn bảo dưỡng | Câu kiểm tra 09; ca B03 |
| MT04 | Lương minh bạch | Lương đã chốt khớp 100% tổng thù lao theo công thức | Câu kiểm tra 08, 10; ca D03 |
| MT05 | Giá vé thống nhất | 100% vé có giá đúng bảng giá theo loại ngày khởi hành | Câu kiểm tra 07; ca C01 |
| MT06 | Có báo cáo điều hành | Đủ 6 báo cáo theo đề, chạy được cho từng tháng và chi nhánh | Màn hình 7; sp\_R01–R06 |
| MT07 | Đạt yêu cầu môn học | Đủ ≥ 5 trigger, ≥ 5 transaction, BCNF, phân tán; nộp đúng hạn 22/11/2026 | Mục 8.D |

## **5\. Các bên liên quan và người dùng**

| Bên liên quan | Vai trò | Nhu cầu chính | Use case |
| :---- | :---- | :---- | :---- |
| NV bán vé | Người dùng hệ thống | Bán vé nhanh, thấy ghế trống, hủy vé, tra cứu theo số điện thoại | UC07–UC09 |
| NV điều hành | Người dùng hệ thống | Tạo chuyến, phân công lái/phụ không trùng giờ, đóng chuyến | UC04–UC06 |
| NV kỹ thuật | Người dùng hệ thống | Biết xe nào sắp và đã quá hạn bảo dưỡng, đăng kiểm | UC10, UC11, UC16, UC17 |
| Quản lý chi nhánh | Người dùng, người ra quyết định | Quản lý xe, tài xế, tuyến; xem lương, doanh thu; chốt lương | UC01–UC03, UC12–UC15 |
| Trụ sở (Hà Nội) | Chủ dữ liệu dùng chung | Quản lý bảng giá, tuyến, tham số; tổng hợp báo cáo toàn chuỗi | Mục 9, chương 5 báo cáo |
| Tài xế | Đối tượng được quản lý | Được trả lương đúng số chuyến và vai trò | QT09, QT10 |
| Hành khách | Đối tượng được phục vụ | Có chỗ ngồi đúng vé, giá đúng ngày | QT01, QT02, QT08 |
| Giảng viên | Người nghiệm thu | Đủ yêu cầu đề bài, báo cáo đúng định dạng, vận hành được | GV01–GV08 |
| Nhóm học viên | Người thực hiện | Hoàn thành đúng hạn, mỗi thành viên vận hành được | Mục 12 |

## **6\. Phạm vi**

| Trong phạm vi | Ngoài phạm vi (lý do) |
| :---- | :---- |
| CSDL 19 bảng trên SQL Server, đạt BCNF, 93 ràng buộc, 6 trigger, 6 transaction | Đăng nhập, phân quyền (đề không yêu cầu; tốn thời gian) |
| Dữ liệu mẫu: chuyến từ 01/01/2025, vé và lương tháng 4–5/2025 | Màn hình quản lý loại xe, bảng giá vé, hành khách (nhập bằng SQL; khách tự tạo khi bán vé) |
| Ứng dụng Python Streamlit 7 màn hình, 17 chức năng | Màn hình đổi vé (giữ thủ tục sp\_DoiVe, demo bằng SQL) |
| 6 báo cáo: lương, doanh thu theo xe, tuyến, loại xe, lịch bảo dưỡng, xe quá hạn | Bán vé trực tuyến, thanh toán, hoàn tiền vé |
| Thiết kế CSDL phân tán 3 chi nhánh (phân mảnh, định vị, nhân bản) | Cài đặt thật CSDL phân tán trên nhiều máy chủ |
| Báo cáo Word, file sơ đồ draw.io | Chuyển sang NoSQL (điểm cộng, để hướng phát triển) |

## **7\. Quy trình nghiệp vụ**

Quy trình mới (TO-BE) gồm 5 bước chính chạy từ trái sang phải; số km của mỗi chuyến hoàn thành quay lại cập nhật lịch bảo dưỡng, quyết định xe có được xếp chuyến tiếp hay không.

![Quy trình nghiệp vụ TO-BE · 5 bước chính và vòng bảo dưỡng][image1]

Quy trình nghiệp vụ TO-BE · 5 bước chính và vòng bảo dưỡng

| Bước | Người thực hiện | Đầu vào | Kết quả | Kiểm soát |
| :---- | :---- | :---- | :---- | :---- |
| 1\. Thiết lập danh mục, giá | Quản lý chi nhánh, trụ sở | Xe, tài xế, tuyến, bảng giá, lịch ngày | Danh mục dùng chung | PK, FK, UNIQUE, CHECK |
| 2\. Tạo chuyến, phân công | NV điều hành | Tuyến, xe, giờ, lái, phụ | Chuyến Kế hoạch, thù lao tự tính | TR03, TR04, TR05, sp\_TaoChuyenVaPhanCong |
| 3\. Bán, hủy vé | NV bán vé | Chuyến Kế hoạch, ghế, khách | Vé có mã và giá tự sinh | TR01, TR02, sp\_BanVe |
| 4\. Kết thúc chuyến | NV điều hành | Giờ thực tế, km, chi phí | Chuyến Hoàn thành | TR06, sp\_KetThucChuyen |
| 5\. Báo cáo, chốt lương | Quản lý chi nhánh | Chuyến hoàn thành, vé đã bán | Lương, doanh thu, BangLuong | sp\_R01–R06, sp\_ChotLuongThang |
| Vòng bảo dưỡng | Hệ thống, NV kỹ thuật | Km làm việc từ bước 4 | Xe quá hạn bị chặn ở bước 2 đến khi bảo dưỡng | vw\_LichBaoDuong, TR04, sp\_GhiNhanBaoDuong |

## **8\. Yêu cầu nghiệp vụ chi tiết**

Cột Nguồn ghi "Đề bài" khi yêu cầu lấy từ đề tài 1, "Nhóm" khi do nhóm tự xác định theo phần đề để mở.

### **A. Yêu cầu dữ liệu**

| Mã | Yêu cầu | Chi tiết | Nguồn | Đáp ứng bởi | Tiêu chí chấp nhận |
| :---- | :---- | :---- | :---- | :---- | :---- |
| DL01 | Lưu thông tin xe khách | Biển số, loại xe (số ghế), chi nhánh, năm sản xuất, ngày đưa vào sử dụng, trạng thái | Đề bài | Bảng Xe, LoaiXe | Không có 2 xe trùng biển số |
| DL02 | Lưu thông tin tài xế | Họ tên, ngày sinh, CCCD, số và hạng GPLX, địa chỉ, nhiều số điện thoại, chi nhánh | Đề bài | Bảng TaiXe, TaiXe\_SDT | Tài xế từ 21 tuổi; không trùng CCCD, GPLX |
| DL03 | Lưu tuyến đường | Điểm đầu, điểm cuối, độ dài, độ phức tạp cấp 1/2/3 | Đề bài | Bảng TuyenDuong, DoPhucTap | Điểm đầu khác điểm cuối; cấp độ chỉ 1–3 |
| DL04 | Lưu chuyến xe | Tuyến, xe, giờ đi/đến dự kiến và thực tế, số km thực tế, chi phí vận hành, trạng thái | Đề bài | Bảng ChuyenXe | Mã chuyến dạng C \+ yyMMdd \+ 3 số |
| DL05 | Lưu phân công tài xế | Mỗi chuyến 1 lái \+ 1 phụ, kèm thù lao từng người | Đề bài | Bảng PhanCong | Không chuyến hoàn thành nào thiếu lái hoặc phụ |
| DL06 | Lưu vé từng hành khách trên mỗi chuyến | Mã vé chứa thông tin tuyến, xe, chuyến; ghế, lần bán, khách, nhân viên bán, giá, trạng thái | Đề bài | Bảng Ve, HanhKhach | Mã vé dạng T01-XE001-C250401001-07-1 |
| DL07 | Lưu giá vé theo tuyến và thời điểm | Ngày thường, cuối tuần, mùa du lịch, lễ tết | Đề bài | Bảng BangGiaVe, LoaiNgay, LichNgay | Mỗi tuyến có đủ 4 mức giá |
| DL08 | Lưu bảo dưỡng và đăng kiểm | Ngày, nội dung, chi phí; ngày hết hạn đăng kiểm, kết quả | Đề bài | Bảng BaoDuong, DangKiem | Có dữ liệu từ đầu năm 2025 |
| DL09 | Lưu cơ cấu chuỗi | Chi nhánh, nhân viên, trưởng chi nhánh | Đề bài | Bảng ChiNhanh, NhanVien | Mỗi chi nhánh có đúng 1 trưởng |
| DL10 | Dữ liệu mẫu đủ quy mô | Chuyến, phân công, bảo dưỡng từ 01/01/2025; vé và lương tháng 4–5/2025 | Đề bài | QLXeKhach\_DuLieu.sql | 10 câu kiểm tra đều có SoLoi \= 0 |

### **B. Quy tắc nghiệp vụ**

| Mã | Quy tắc | Chi tiết | Nguồn | Đáp ứng bởi | Tiêu chí chấp nhận |
| :---- | :---- | :---- | :---- | :---- | :---- |
| QT01 | Giới hạn số khách | Số vé đã bán của 1 chuyến ≤ số ghế − 2 | Đề bài | TR01 | Bán ghế vượt số chỗ bị chặn |
| QT02 | Không trùng ghế | Mỗi ghế chỉ 1 vé đang hiệu lực; ghế của vé hủy được bán lại với mã đuôi \-2 | Nhóm | Unique index có lọc, TR02 | Bán trùng ghế bị chặn; bán lại ra mã \-2 |
| QT03 | Đúng 1 lái, 1 phụ mỗi chuyến | Cố định suốt chuyến; lái khác phụ | Đề bài | UNIQUE(MaChuyen, VaiTro), PK PhanCong, TR06 | Thêm lái thứ 2 bị chặn |
| QT04 | Không trùng giờ | Tài xế, xe không chạy 2 chuyến chồng giờ | Nhóm | TR03, TR04 | Phân công trùng giờ bị hủy cả giao dịch |
| QT05 | Xe đủ điều kiện chạy | Xe hoạt động, còn hạn đăng kiểm và chưa quá hạn bảo dưỡng tại ngày khởi hành | Đề bài | TR04 | Xếp chuyến cho xe quá hạn bị chặn |
| QT06 | Chỉ bán cho chuyến Kế hoạch | Chuyến hoàn thành hoặc hủy thì không bán vé | Nhóm | TR01, sp\_BanVe | Bán vé chuyến đã hoàn thành bị chặn |
| QT07 | Điều kiện hoàn thành chuyến | Có đủ lái, phụ, số km và giờ thực tế | Nhóm | TR06 | Hoàn thành chuyến thiếu phụ xe bị chặn |
| QT08 | Giá vé theo loại ngày | Thường 100%, cuối tuần 110%, mùa du lịch 115%, lễ tết 130% | Đề bài, Nhóm | TR02, BangGiaVe | Giá trên vé khớp bảng giá theo ngày khởi hành |
| QT09 | Thù lao mỗi chuyến | km × 2.000 đ × hệ số lương theo cấp (1,0 / 1,2 / 1,5) × vai trò (lái 1,0; phụ 0,6) | Đề bài, Nhóm | TR05, ThamSo | Hà Nội – Vinh: lái 720.000 đ, phụ 432.000 đ |
| QT10 | Lương tháng tài xế | Tổng thù lao các chuyến hoàn thành trong tháng | Đề bài | sp\_R01, sp\_ChotLuongThang | Lương đã chốt khớp tổng thù lao |
| QT11 | Lịch bảo dưỡng | Chu kỳ 360 ngày, giảm 1 ngày mỗi 100 km làm việc; km làm việc \= km thực tế × hệ số đường khó (1,0 / 1,3 / 1,6) | Đề bài, Nhóm | vw\_LichBaoDuong | XE004, XE008, XE012 quá hạn tại 31/05/2025 |
| QT12 | Doanh thu | Tổng giá vé đã bán của chuyến hoàn thành, tính theo tháng khởi hành | Đề bài | sp\_R02–R04 | Tổng doanh thu tháng 5/2025 \= 2.906.900.000 đ |
| QT13 | Cấu trúc mã | Chi nhánh CN01, xe XE001, tài xế TX0001, tuyến T01, chuyến C250601001, vé MaTuyen-MaXe-MaChuyen-SoGhe-LanBan | Đề bài, Nhóm | CHECK, TR02 | Mọi mã vé đúng cấu trúc |
| QT14 | Xóa an toàn | Không xóa dữ liệu đang được tham chiếu (xe đã có chuyến…); đổi trạng thái thay vì xóa | Nhóm | FOREIGN KEY | Xóa xe đã có chuyến bị chặn |

### **C. Yêu cầu chức năng**

| Mã | Chức năng | Chi tiết | Nguồn | Đáp ứng bởi | Tiêu chí chấp nhận |
| :---- | :---- | :---- | :---- | :---- | :---- |
| CN01 | Quản lý xe khách | Thêm, sửa, xóa, tìm kiếm (UC01) | Đề bài | Màn hình 1 | Lỗi ràng buộc hiện hộp thông báo đỏ |
| CN02 | Quản lý tài xế | Thêm, sửa, xóa, tìm kiếm (UC02) | Đề bài | Màn hình 2 | Như trên |
| CN03 | Quản lý tuyến đường | Thêm, sửa, xóa, tìm kiếm (UC03) | Đề bài | Màn hình 3 | Như trên |
| CN04 | Quản lý chuyến xe | Tạo chuyến kèm phân công, sửa, tìm, hủy, kết thúc chuyến (UC04–06) | Đề bài | Màn hình 4; sp\_TaoChuyenVaPhanCong, sp\_KetThucChuyen | Tạo chuyến lỗi thì không lưu gì |
| CN05 | Bán, hủy, tra cứu vé | Sơ đồ ghế, giá tự tính, khách mới tự tạo (UC07–09) | Đề bài | Màn hình 5; sp\_BanVe | Ghế đã bán tô xám, không chọn được |
| CN06 | Ghi nhận bảo dưỡng, đăng kiểm | UC10–11 | Nhóm | Màn hình 6; sp\_GhiNhanBaoDuong | Xe bảo dưỡng xong trở lại Hoạt động |
| CN07 | Lương tài xế theo tháng | Tên tài xế cùng lương tháng; nút chốt lương (UC12) | Đề bài | Màn hình 7; sp\_R01, sp\_ChotLuongThang | Chọn được tháng, chi nhánh |
| CN08 | Doanh thu từng xe đang hoạt động | Theo số chuyến và số vé trong tháng (UC13) | Đề bài | sp\_R02\_DoanhThuXe | Chỉ liệt kê xe Hoạt động |
| CN09 | Doanh thu theo tuyến, loại xe | UC14–15 | Đề bài | sp\_R03, sp\_R04 | Có biểu đồ cột |
| CN10 | Lịch bảo dưỡng, hạn đăng kiểm | Ngày bảo dưỡng tiếp theo, hạn đăng kiểm từng xe (UC16) | Đề bài | vw\_LichBaoDuong, sp\_R05 | Đúng công thức QT11 |
| CN11 | Danh sách xe quá hạn bảo dưỡng | Theo ngày xét (UC17) | Đề bài | sp\_R06\_XeQuaHanBaoDuong | Ngày xét 31/05/2025 ra 3 xe |
| CN12 | Thể hiện ràng buộc số lượng trên ứng dụng | Số chỗ, trùng ghế, trùng giờ… hiện trên giao diện | Đề bài | Sơ đồ ghế, hộp thông báo lỗi | Mọi lỗi trigger hiện nguyên văn tiếng Việt |

### **D. Yêu cầu của giáo viên**

| Mã | Yêu cầu | Chi tiết | Nguồn | Đáp ứng bởi | Tiêu chí chấp nhận |
| :---- | :---- | :---- | :---- | :---- | :---- |
| GV01 | Lược đồ E-R đủ loại | Đủ loại thực thể, thuộc tính, liên kết; có mô tả | Đề bài | Chương 2, file drawio trang 1 | Có thực thể yếu, thuộc tính phức hợp, đa trị, dẫn xuất, liên kết 1–1, 1–N, N–N |
| GV02 | Lược đồ quan hệ có khóa chính, khóa ngoại | Ánh xạ từ E-R | Đề bài | Chương 3, file drawio trang 3 | 19 lược đồ, 20 khóa ngoại |
| GV03 | Phụ thuộc hàm, chuẩn BCNF | Từng lược đồ | Đề bài | Chương 3 | Mọi vế trái là siêu khóa |
| GV04 | Ràng buộc viết bằng SQL | Trình bày cụ thể trong báo cáo | Đề bài | QLXeKhach\_SQLServer.sql, mục 4.3 | 93 ràng buộc khai báo có tên |
| GV05 | Ít nhất 5 transaction | Thực hiện nghiệp vụ | Đề bài | 6 thủ tục TX01–TX06 | Có ROLLBACK khi lỗi |
| GV06 | Ít nhất 5 trigger | Thực hiện nghiệp vụ dữ liệu | Đề bài | TR01–TR06 | 27 ca kiểm thử đạt |
| GV07 | Thiết kế CSDL phân tán | Số liệu giả định, phân mảnh, định vị, nhân bản, sơ đồ tổng quát | Đề bài | Chương 5 | Đủ 3 tính chất đầy đủ, tái tạo, tách biệt |
| GV08 | Nộp và vận hành | 1 file Word/PDF trước 23h59 ngày 22/11/2026, tiêu đề \[M26CQHT02\] nhóm X\_…; mọi thành viên vận hành được | Đề bài | BaoCao\_BTL\_QLXeKhach.docx | Đúng định dạng tiêu đề và hạn nộp |

## **9\. Đặc tả báo cáo**

Mọi báo cáo có bộ lọc chi nhánh (để trống \= toàn chuỗi). Doanh thu và lương chỉ tính chuyến **Hoàn thành**, theo tháng của **ngày khởi hành**; doanh thu chỉ tính vé **Đã bán**.

| Mã | Báo cáo | Bộ lọc | Cột hiển thị | Công thức | Sắp xếp |
| :---- | :---- | :---- | :---- | :---- | :---- |
| R01 | Lương tháng tài xế | Tháng, chi nhánh | Mã TX, họ tên, chi nhánh, số chuyến lái, số chuyến phụ, lương tháng | Σ ThuLao của các phân công thuộc chuyến hoàn thành trong tháng (QT09, QT10) | Lương giảm dần |
| R02 | Doanh thu từng xe đang hoạt động | Tháng, chi nhánh | Mã xe, biển số, loại xe, số chuyến, số vé, doanh thu | Σ GiaVe vé đã bán của chuyến hoàn thành; chỉ xe trạng thái Hoạt động | Doanh thu giảm dần |
| R03 | Doanh thu theo tuyến | Tháng, chi nhánh | Mã tuyến, tên tuyến, số chuyến, số vé, doanh thu | Như R02, gom theo tuyến | Doanh thu giảm dần |
| R04 | Doanh thu theo loại xe | Tháng, chi nhánh | Loại xe, số xe, số chuyến, số vé, doanh thu | Như R02, gom theo loại xe | Doanh thu giảm dần |
| R05 | Lịch bảo dưỡng và đăng kiểm | Chi nhánh | Mã xe, biển số, ngày bảo dưỡng gần nhất, km làm việc, ngày bảo dưỡng tiếp theo, hạn đăng kiểm | Ngày BD tiếp \= ngày BD gần nhất \+ 360 − ⌊km làm việc ÷ 100⌋ (QT11) | Ngày BD tiếp tăng dần |
| R06 | Xe quá hạn bảo dưỡng | Ngày xét (mặc định hôm nay), chi nhánh | Như R05 thêm số ngày quá hạn | Lọc xe có ngày BD tiếp \< ngày xét | Số ngày quá hạn giảm dần |

**Số liệu mẫu đối chiếu (tháng 05/2025, toàn chuỗi):** doanh thu 2.906.900.000 đ; 10.540 vé; tổng lương tài xế 465.024.000 đ; tuyến doanh thu cao nhất Hà Nội – Sa Pa 486.550.000 đ; R06 tại 31/05/2025 ra 3 xe XE004, XE008, XE012.

## **10\. Yêu cầu phi chức năng**

| Mã | Nhóm | Yêu cầu | Cách đáp ứng |
| :---- | :---- | :---- | :---- |
| PCN01 | Toàn vẹn dữ liệu | Mọi quy tắc được chặn ở tầng CSDL, không phụ thuộc giao diện; sửa trực tiếp bằng SQL vẫn bị chặn | Ràng buộc khai báo \+ trigger; ca kiểm thử C04 chèn thẳng bảng Ve |
| PCN02 | Đồng thời | Hai nhân viên bán cùng một ghế cùng lúc thì chỉ một người thành công | sp\_BanVe dùng UPDLOCK, HOLDLOCK; unique index có lọc |
| PCN03 | Nguyên tử | Thao tác nhiều bước lỗi giữa chừng thì không lưu phần nào | 6 transaction có SET XACT\_ABORT ON, ROLLBACK |
| PCN04 | Hiệu năng | Bán vé phản hồi dưới 2 giây; báo cáo tháng (≈ 10.000 vé) dưới 5 giây | Khóa chính, khóa ngoại, unique index; truy vấn gom nhóm trong thủ tục |
| PCN05 | Dễ sử dụng | Thông báo lỗi tiếng Việt, nêu rõ lý do; ghế đã bán tô xám, không chọn được | Ứng dụng hiển thị nguyên văn thông báo của trigger |
| PCN06 | Bảo mật, riêng tư | Dữ liệu cá nhân tài xế (CCCD, địa chỉ, ngày sinh) chỉ lưu ở trụ sở | Phân mảnh dọc TaiXe\_NhanSu đặt tại S1 |
| PCN07 | Mở rộng | Đổi hệ số lương, đơn giá, chu kỳ bảo dưỡng không cần sửa code; thêm chi nhánh \= thêm một mảnh | Bảng ThamSo, DoPhucTap; phân mảnh theo MaCN |
| PCN08 | Bảo trì | Ràng buộc đặt tên thống nhất để đọc lỗi dễ: PK\_, FK\_, UQ\_, CK\_, DF\_ | QLXeKhach\_SQLServer.sql |
| PCN09 | Tương thích | SQL Server 2019+, SSMS; Python 3.10+, streamlit, pyodbc, pandas; ODBC Driver 17/18 | Hướng dẫn chạy trong phụ lục báo cáo |
| PCN10 | Lưu vết | Giữ lịch sử: vé hủy không xóa, giá vé lưu theo lúc bán, lương đã chốt không đổi | Trạng thái DaHuy; cột GiaVe trong Ve; bảng BangLuong |

## **11\. Ma trận truy vết**

Mỗi yêu cầu được nối tới use case, đối tượng CSDL thực hiện và ca kiểm tra chứng minh. Mã A01–D04 là ca trong QLXeKhach\_KiemThu.sql; KT01–KT10 là câu kiểm tra cuối QLXeKhach\_DuLieu.sql.

| Yêu cầu | Use case | Đối tượng CSDL | Kiểm thử |
| :---- | :---- | :---- | :---- |
| DL01 Xe khách | UC01 | Xe, LoaiXe, UQ\_Xe\_BienSo, FK\_Xe\_LoaiXe | A05, A06 |
| DL02 Tài xế | UC02 | TaiXe, TaiXe\_SDT, CK\_TX\_Tuoi | A07 |
| DL03 Tuyến | UC03 | TuyenDuong, DoPhucTap, CK\_Tuyen\_DiemKhacNhau | A03, A04 |
| DL04, DL05 Chuyến, phân công | UC04–UC06 | ChuyenXe, PhanCong, CK\_Chuyen\_GioDK | A08, B01 |
| DL06 Vé | UC07–UC09 | Ve, HanhKhach, TR02 | C01, C06 |
| DL07 Giá vé | UC07 | BangGiaVe, LichNgay, TR02 | C01, KT07 |
| DL08 Bảo dưỡng, đăng kiểm | UC10, UC11 | BaoDuong, DangKiem, sp\_GhiNhanBaoDuong | D04 |
| QT01 Số chỗ | UC07 | TR01 | C02, KT01, KT02 |
| QT02 Không trùng ghế | UC07, UC08 | UX\_Ve\_GheDangBan, TR02, sp\_BanVe | C03, C04, C06 |
| QT03 1 lái \+ 1 phụ | UC04, UC06 | UQ\_PC\_ChuyenVaiTro, PK\_PhanCong, TR06 | B04, B06, B07, KT03 |
| QT04 Không trùng giờ | UC04 | TR03, TR04 | B02, B05, KT04, KT05 |
| QT05 Xe đủ điều kiện | UC04 | TR04, vw\_LichBaoDuong | B03, KT09 |
| QT06 Chỉ bán chuyến Kế hoạch | UC07 | TR01, sp\_BanVe | D02 |
| QT07 Hoàn thành chuyến | UC06 | TR06, sp\_KetThucChuyen | B07, D01 |
| QT08 Giá theo loại ngày | UC07 | TR02 | C01, KT07 |
| QT09, QT10 Thù lao, lương | UC12 | TR05, sp\_R01, sp\_ChotLuongThang | B01, D03, KT08, KT10 |
| QT11 Lịch bảo dưỡng | UC16, UC17 | vw\_LichBaoDuong, sp\_R05, sp\_R06 | D04, UAT08 |
| QT12 Doanh thu | UC13–UC15 | sp\_R02, sp\_R03, sp\_R04 | Số liệu đối chiếu mục 9 |
| QT13 Cấu trúc mã | UC04, UC07 | CK\_Chuyen\_MaFormat, TR02 | C01, KT06 |
| QT14 Xóa an toàn | UC01 | FK\_Chuyen\_Xe | B08 |
| GV05 ≥ 5 transaction | – | sp\_BanVe, sp\_DoiVe, sp\_TaoChuyenVaPhanCong, sp\_KetThucChuyen, sp\_GhiNhanBaoDuong, sp\_ChotLuongThang | C01, C07, B05, D01, D04, D03 |
| GV06 ≥ 5 trigger | – | TR01–TR06 | C02, C01, B05, B03, B01, B07 |

## **12\. Kịch bản nghiệm thu (UAT)**

Chạy trên ứng dụng sau khi nạp dữ liệu mẫu, theo đúng thứ tự; mỗi thành viên nên tự chạy được cả 10 kịch bản trước buổi vận hành. Cột Kết quả để đánh dấu khi chạy.

| Mã | Kịch bản | Bước thực hiện | Kết quả mong đợi | Kết quả |
| :---- | :---- | :---- | :---- | :---- |
| UAT01 | Bán vé đến hết chỗ | Màn hình 5, chọn chuyến Kế hoạch còn ít ghế, bán đến ghế cuối | Sơ đồ ghế toàn ô xám, báo hết chỗ; không chọn thêm được | Chưa chạy |
| UAT02 | Hủy vé và bán lại ghế | Tab Hủy vé, tra cứu rồi hủy 1 vé; quay lại bán đúng ghế đó | Vé cũ Đã hủy; vé mới có mã đuôi \-2 | Chưa chạy |
| UAT03 | Tạo chuyến hợp lệ | Màn hình 4, chọn tuyến, xe, giờ, 2 tài xế rảnh | Chuyến Kế hoạch mới; thù lao tự tính; chuyến hiện ở màn hình bán vé | Chưa chạy |
| UAT04 | Tạo chuyến cho xe quá hạn | Chọn xe XE004 (quá hạn bảo dưỡng) | Báo lỗi xe quá hạn hoặc không ở trạng thái Hoạt động; không lưu chuyến | Chưa chạy |
| UAT05 | Phân công trùng giờ | Chọn tài xế đã có chuyến chồng giờ | Báo lỗi trùng giờ; cả chuyến lẫn phân công không được lưu | Chưa chạy |
| UAT06 | Kết thúc chuyến cập nhật báo cáo | Kết thúc 1 chuyến có vé; xem màn hình 7 | Doanh thu tăng đúng tổng giá vé; lương 2 tài xế tăng đúng thù lao | Chưa chạy |
| UAT07 | Chốt lương tháng | Màn hình 7, tháng 2025-05, bấm Chốt lương | BangLuong có 40 dòng; tổng 465.024.000 đ | Chưa chạy |
| UAT08 | Xe quá hạn và bảo dưỡng | Báo cáo xe quá hạn ngày xét 31/05/2025; ghi nhận bảo dưỡng XE004 | Ban đầu 3 xe; sau bảo dưỡng XE004 không còn trong danh sách và về Hoạt động | Chưa chạy |
| UAT09 | Thêm xe trùng biển số | Màn hình 1, nhập biển số đã có | Báo lỗi trùng biển số (UQ\_Xe\_BienSo) | Chưa chạy |
| UAT10 | Thêm tài xế chưa đủ tuổi | Màn hình 2, năm sinh 2008 | Báo lỗi tài xế phải từ 21 tuổi (CK\_TX\_Tuoi) | Chưa chạy |

[image1]: <data:image/png;base64,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>