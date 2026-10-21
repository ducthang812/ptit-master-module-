# TÓM TẮT ÔN THI — CƠ SỞ DỮ LIỆU PHÂN TÁN

Tài liệu gồm 4 phần, tổng cộng 271 trang slide.

| Phần | Nội dung | Số trang |
|---|---|---:|
| 1 | Tổng quan cơ sở dữ liệu phân tán | 69 |
| 2 | Thiết kế cơ sở dữ liệu — Phân mảnh ngang | 50 |
| 3 | Thiết kế cơ sở dữ liệu — Phân mảnh dọc, nhân bản và cấp phát | 58 |
| 4 | Xử lý và tối ưu hóa truy vấn | 94 |

**Ký hiệu:** $\sigma$ là phép chọn; $\pi$ là phép chiếu; $\bowtie$ là phép nối; $\ltimes$ là phép nửa nối; $\cup$ là phép hợp.

---

# PHẦN 1 — TỔNG QUAN CƠ SỞ DỮ LIỆU PHÂN TÁN

## 1. Hệ thống phân tán

**Distributed System (hệ thống phân tán)** là tập hợp các thành phần tự trị cộng tác qua mạng và được người dùng nhìn nhận như một hệ thống duy nhất.

- **Client/Server:** Máy khách gửi yêu cầu và máy chủ xử lý dữ liệu.
- **Xử lý giao dịch phân tán:** Một giao dịch được chia thành các công việc thực hiện tại nhiều hệ thống.
- **Phân tán chức năng:** Các chức năng được thực hiện ở nhiều vị trí.
- **Phân tán dữ liệu:** Dữ liệu được lưu trữ tại nhiều vị trí.
- **Điều khiển phân tán:** Việc điều khiển giao dịch được phân phối giữa nhiều vị trí.

**Ví dụ:** Một ngân hàng có nhiều chi nhánh cùng kết nối qua mạng để xử lý giao dịch.

## 2. Các mô hình xử lý dữ liệu

| Mô hình | Đặc điểm |
|---|---|
| Xử lý truyền thống | Mỗi ứng dụng quản lý tệp riêng, dễ dư thừa và phụ thuộc dữ liệu. |
| Cơ sở dữ liệu tập trung | Dữ liệu được quản lý trong một hệ cơ sở dữ liệu tập trung. |
| Cơ sở dữ liệu phân tán | Dữ liệu nằm ở nhiều vị trí nhưng có quan hệ logic với nhau. |

## 3. Cơ sở dữ liệu phân tán

**Distributed Database (DDB)** là tập hợp các cơ sở dữ liệu có quan hệ logic với nhau và được phân bố trên mạng máy tính.

**Distributed Database Management System (DDBMS)** là phần mềm quản lý cơ sở dữ liệu phân tán và che giấu sự phân tán đối với người dùng.

- **Tính phân tán:** Dữ liệu được lưu tại nhiều vị trí (site).
- **Quan hệ logic:** Dữ liệu giữa các vị trí có quan hệ và ràng buộc với nhau.

**Ví dụ slide trang 12:** Dữ liệu nhân viên, dự án và vị trí công việc được phân bố tại Hà Nội, Hải Phòng, Đà Nẵng và TP. Hồ Chí Minh.

## 4. Đặc điểm cơ sở dữ liệu phân tán

### 4.1. Điều khiển tập trung và điều khiển phân tán

- Cơ sở dữ liệu tập trung có hệ thống quản trị và điều khiển tập trung.
- Cơ sở dữ liệu phân tán có người quản trị toàn cục (Global DBA) và người quản trị cục bộ (Local DBA).
- Các vị trí cục bộ có thể có quyền tự chủ trong quản lý dữ liệu.

### 4.2. Độc lập dữ liệu và tính trong suốt

- **Data Independence (độc lập dữ liệu):** Chương trình ít bị ảnh hưởng khi thay đổi tổ chức lưu trữ dữ liệu.
- **Distribution Transparency (trong suốt phân tán):** Ứng dụng truy cập dữ liệu mà không phải quan tâm trực tiếp đến cách dữ liệu được phân bố.
- **Trong suốt phân mảnh:** Che giấu việc dữ liệu được chia thành nhiều mảnh.
- **Trong suốt vị trí:** Che giấu vị trí lưu trữ các mảnh.
- Di chuyển dữ liệu có thể không ảnh hưởng chương trình nhưng vẫn ảnh hưởng hiệu năng.

### 4.3. Dư thừa dữ liệu và nhân bản

**Replication (nhân bản dữ liệu)** là lưu các bản sao dữ liệu tại nhiều vị trí.

- Nhân bản giúp tăng tốc độ đọc và tính sẵn sàng.
- Nhân bản làm tăng chi phí cập nhật vì phải đồng bộ các bản sao.
- Nhân bản có lợi với hệ thống đọc nhiều và cập nhật ít.

### 4.4. Độ tin cậy và giao dịch phân tán

**Transaction (giao dịch)** là đơn vị xử lý gồm nhiều thao tác và phải bảo đảm yêu cầu nhất quán, độ tin cậy.

- Lỗi tại một vị trí không nhất thiết làm toàn bộ hệ thống dừng.
- Nhân bản cho phép tiếp tục truy cập khi một bản sao gặp sự cố.
- Giao dịch phân tán phải duy trì tính nhất quán khi xử lý đồng thời hoặc xảy ra lỗi.

### 4.5. Cải tiến hiệu năng

- **Data Locality (tính cục bộ):** Đặt dữ liệu gần nơi sử dụng để giảm truy cập từ xa.
- **Inter-query Parallelism (song song liên truy vấn):** Thực hiện nhiều truy vấn đồng thời.
- **Intra-query Parallelism (song song nội truy vấn):** Chia một truy vấn thành nhiều truy vấn con chạy song song.
![xx](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiA8u2N4JAXufUrcVR5kqSVhSQFsq3a2wVv6Y8HbFE1tKEe_wEpU-CfouBhTw2uC-SALxs0ndLY9R1nWiBM_t30IzpbDhoM8YCPkK1thHf4fE3Qq7ZKqpFnqEdAa7nDPTkXyB6LPQtDBCQw/s1600/Query1.JPG)

- Phân mảnh hợp lý giúp giảm tranh chấp tài nguyên và chi phí truyền thông.

### 4.6. Tính toàn vẹn, phục hồi và điều khiển tương tranh

- **Atomicity (tính nguyên tố):** Giao dịch được thực hiện toàn bộ hoặc không có hiệu lực.
- **Recovery (phục hồi):** Khôi phục hoạt động hợp lệ khi xảy ra sự cố.
- **Concurrency Control (điều khiển tương tranh):** Điều phối giao dịch đồng thời để tránh kết quả sai.
- Điều khiển tương tranh trong môi trường phân tán phức tạp hơn môi trường tập trung.

## 5. Ưu điểm và nhược điểm

| Ưu điểm | Nhược điểm |
|---|---|
| Chia sẻ dữ liệu giữa nhiều vị trí. | Chi phí phát triển và quản trị cao. |
| Tăng tính sẵn sàng và độ tin cậy. | Khó phát hiện lỗi phân tán. |
| Hỗ trợ thực hiện truy vấn song song. | Tăng chi phí truyền thông và phối hợp. |
| Có khả năng mở rộng bằng nhiều vị trí. | Khó duy trì tính nhất quán dữ liệu. |

## 6. Mô hình tham chiếu cơ sở dữ liệu phân tán

### 6.1. Kiến trúc ANSI/SPARC

- **External Schema:** Lược đồ ngoài, góc nhìn dữ liệu của người dùng.
- **Conceptual Schema:** Lược đồ khái niệm, cấu trúc logic dữ liệu.
- **Internal Schema:** Lược đồ trong, tổ chức lưu trữ vật lý.

### 6.2. Mô hình tham chiếu phân tán

Sơ đồ trang 33: **Lược đồ toàn cục → Lược đồ phân mảnh → Lược đồ cấp phát → Lược đồ ánh xạ cục bộ → DBMS cục bộ → CSDL cục bộ**.

- **Global Schema:** Định nghĩa dữ liệu, quan hệ và ràng buộc logic.
- **Fragmentation Schema:** Mô tả cách chia quan hệ toàn cục thành các mảnh.
- **Allocation Schema:** Xác định mảnh nào được lưu tại vị trí nào.
- **Local Mapping Schema:** Ánh xạ dữ liệu vào hệ quản trị tại từng vị trí.

**Phân biệt:** Phân mảnh quyết định *chia dữ liệu như thế nào*, còn cấp phát quyết định *đặt dữ liệu ở đâu*.

### 6.3. Ví dụ cấp phát dữ liệu

Slide trang 38–39 chia quan hệ $R$ thành $R_1,R_2,R_3,R_4$.

| Vị trí | Các mảnh được lưu |
|---|---|
| Site 1 | $R_1,R_2$ |
| Site 2 | $R_1,R_2$ |
| Site 3 | $R_2,R_3,R_4$ |

Mảnh $R_2$ có ba bản sao, còn $R_1$ có hai bản sao.

## 7. Hệ quản trị cơ sở dữ liệu phân tán

### 7.1. Các thành phần

- **DB (Database Management):** Quản lý dữ liệu.
- **DC (Data Communication):** Truyền thông giữa các trạm.
- **DD (Data Dictionary):** Lưu metadata, gồm thông tin phân tán.
- **DDB (Distributed Database):** Thành phần quản lý dữ liệu phân tán.

### 7.2. Các dịch vụ

- Hỗ trợ truy cập cơ sở dữ liệu từ xa.
- Hỗ trợ quản trị và điều khiển dữ liệu.
- Hỗ trợ đồng thời và phục hồi giao dịch phân tán.
- Hỗ trợ mở rộng và phối hợp các hệ thống.

### 7.3. Các kiểu hệ quản trị

- **Homogeneous DDBMS (đồng nhất):** Các vị trí dùng cùng hệ quản trị.
- **Heterogeneous DDBMS (không đồng nhất):** Các vị trí có thể dùng hệ quản trị khác nhau.
- **Middleware (phần mềm trung gian):** Hỗ trợ giao tiếp và phối hợp.

## 8. Kiến trúc hệ quản trị cơ sở dữ liệu phân tán

### 8.1. Kiến trúc Client/Server

- Client gửi yêu cầu xử lý đến Server.
- Server thực hiện yêu cầu và trả kết quả.
- Có thể có nhiều Client hoặc nhiều Server.

### 8.2. Kiến trúc ngang hàng (Peer-to-Peer)

- Các vị trí có thể đồng thời cung cấp và sử dụng dịch vụ.
- Các trạm cộng tác và có quyền tự chủ tương đối.
- Không bắt buộc toàn bộ xử lý qua máy chủ trung tâm.

### 8.3. Kiến trúc đa hệ quản trị (Multidatabase)

- Kết hợp các hệ cơ sở dữ liệu tự quản.
- Hỗ trợ truy cập dữ liệu từ nhiều hệ quản trị.
- Cần cơ chế phối hợp và ánh xạ giữa các hệ thống.

## Điểm cần nhớ — Phần 1

1. Cơ sở dữ liệu phân tán phải có tính phân tán và quan hệ logic giữa dữ liệu.
2. Phân mảnh, cấp phát và nhân bản là ba vấn đề quan trọng.
3. Nhân bản tăng hiệu quả đọc nhưng làm cập nhật phức tạp hơn.
4. Kiến trúc tham chiếu gồm lược đồ toàn cục, phân mảnh, cấp phát và ánh xạ cục bộ.
5. Giao dịch phân tán cần xử lý tính nguyên tố, phục hồi và tương tranh.

---

# PHẦN 2 — THIẾT KẾ CSDL PHÂN TÁN: PHÂN MẢNH NGANG

## 1. Phân mảnh dữ liệu

**Fragmentation (phân mảnh)** là quá trình chia quan hệ toàn cục thành nhiều quan hệ con có liên hệ logic.

- Giảm dữ liệu cần xử lý tại mỗi vị trí.
- Tăng khả năng xử lý truy vấn song song.
- Tạo điều kiện thực hiện nhiều giao dịch đồng thời.
- Có thể giảm hiệu năng nếu truy vấn thường xuyên cần nối nhiều mảnh.

### 1.1. Các loại phân mảnh

| Loại | Cách chia | Phép đại số quan hệ |
|---|---|---|
| Horizontal Fragmentation | Chia theo hàng (bộ). | Phép chọn $\sigma$. |
| Vertical Fragmentation | Chia theo cột (thuộc tính). | Phép chiếu $\pi$. |
| Hybrid Fragmentation | Kết hợp chia hàng và cột. | Kết hợp chọn và chiếu. |

## 2. Ba quy tắc phân mảnh

### 2.1. Tính đầy đủ (Completeness)

Mọi mục dữ liệu của quan hệ gốc phải xuất hiện trong các mảnh.

### 2.2. Tính phục hồi (Reconstruction)

- Phân mảnh ngang: $R=R_1\cup R_2\cup\cdots\cup R_n$.
- Phân mảnh dọc: $R=R_1\bowtie R_2\bowtie\cdots\bowtie R_n$, với phép nối thích hợp theo khóa chung.

### 2.3. Tính tách biệt (Disjointness)

- Phân mảnh ngang: $R_i\cap R_j=\varnothing$ với $i\ne j$.
- Phân mảnh dọc: Các mảnh chỉ chung thuộc tính khóa, không lặp thuộc tính không khóa.

## 3. Phân mảnh ngang

**Horizontal Fragmentation** là phân chia quan hệ theo điều kiện chọn các bộ.

- **Primary Horizontal Fragmentation:** Dựa vào thuộc tính của chính quan hệ.
- **Derived Horizontal Fragmentation:** Dựa vào quan hệ khác thông qua phép nối hoặc nửa nối.

## 4. Phân mảnh ngang nguyên thủy

### 4.1. Vị từ đơn giản (Simple Predicate)

Vị từ đơn giản có dạng $p_j:A_i\ \theta\ value$, trong đó $\theta$ là toán tử so sánh.

Ví dụ với `PROJ(PNO, PNAME, BUDGET)`:

- $p_1:PNAME=\text{"Maintenance"}$.
- $p_2:BUDGET\le200000$.

### 4.2. Vị từ hội sơ cấp (Minterm Predicate)

Một vị từ hội sơ cấp là phép AND các vị từ đơn giản hoặc phủ định của chúng.

- $m_1=p_1\land p_2$.
- $m_2=p_1\land\neg p_2$.
- $m_3=\neg p_1\land p_2$.
- $m_4=\neg p_1\land\neg p_2$.

Với $n$ vị từ độc lập, có tối đa $2^n$ tổ hợp hội sơ cấp trước khi loại tổ hợp không hợp lệ.

### 4.3. Độ tuyển và tần số truy nhập

- **Minterm Selectivity:** $sel(m)$ là số bộ thỏa mãn $m$.
- **Access Frequency:** $acc(q_i)$ là tần số thực hiện truy vấn $q_i$.
- $acc(m)$ là tần số truy nhập của vị từ hội sơ cấp $m$.

## 5. Tính đầy đủ và tính cực tiểu của tập vị từ

### 5.1. Tính đầy đủ (Completeness)

Tập vị từ đầy đủ khi các bộ trong mỗi mảnh được ứng dụng truy nhập đồng nhất về thống kê.

**Ví dụ:** Truy vấn phân biệt dự án theo `LOC` và `BUDGET` thì tập vị từ phải phản ánh cả hai tiêu chí.

### 5.2. Tính cực tiểu (Minimality)

- Tập vị từ cực tiểu khi mọi vị từ đều có tính liên đới (Relevant).
- Một vị từ liên đới nếu nó tạo các mảnh có cách truy cập khác nhau bởi ít nhất một ứng dụng.
- Vị từ không làm thay đổi cách truy cập có thể bị loại.

## 6. Thuật toán COM_MIN

**COM_MIN** tìm tập vị từ đơn giản đầy đủ và cực tiểu.

1. Chọn một vị từ phân hoạch quan hệ theo cách truy cập của ứng dụng.
2. Đưa vị từ vào $Pr'$ và tạo mảnh tương ứng.
3. Xét các vị từ còn lại để phân hoạch tiếp những mảnh hiện có.
4. Loại bỏ vị từ không có tính liên đới.
5. Dừng khi $Pr'$ đầy đủ và cực tiểu.

## 7. Thuật toán PHORIZONTAL

**PHORIZONTAL** xác định tập vị từ hội sơ cấp hợp lệ.

1. Chạy COM_MIN để tìm $Pr'$.
2. Sinh các hội sơ cấp từ $Pr'$.
3. Xác định các phép kéo theo (Implication).
4. Loại bỏ hội sơ cấp mâu thuẫn.
5. Tạo các mảnh ngang theo tập hội sơ cấp còn lại.

## 8. Ví dụ phân mảnh ngang

### 8.1. Phân mảnh PAY

- $PAY_1=\sigma_{SAL\le30000}(PAY)$.
- $PAY_2=\sigma_{SAL>30000}(PAY)$.

| PAY₁ | SAL | PAY₂ | SAL |
|---|---:|---|---:|
| Mech. Eng. | 27000 | Elect. Eng. | 40000 |
| Programmer | 24000 | Syst. Anal. | 34000 |

Hai mảnh hợp lại để khôi phục quan hệ PAY.

### 8.2. Phân mảnh PROJ

- Xét `PROJ(PNO, PNAME, BUDGET, LOC)` tại Montreal, New York, Paris.
- Chia theo `LOC` và `BUDGET ≤ 200000` hoặc `BUDGET > 200000`.
- Có $3\times2=6$ vị từ hội sơ cấp.
- Ví dụ: $PROJ_1=\sigma_{LOC="Montreal"\land BUDGET\le200000}(PROJ)$.
- Trong dữ liệu mẫu trang 42–43, $PROJ_2$ và $PROJ_5$ rỗng.

## 9. Phân mảnh ngang dẫn xuất

**Derived Horizontal Fragmentation** chia quan hệ theo các mảnh của một quan hệ khác.

### 9.1. Phân mảnh EMP theo PAY

- $PAY_1=\sigma_{SAL\le30000}(PAY)$.
- $PAY_2=\sigma_{SAL>30000}(PAY)$.
- $EMP_1=EMP\ltimes PAY_1$.
- $EMP_2=EMP\ltimes PAY_2$.

Phép nửa nối $\ltimes$ giữ các bộ của EMP nối phù hợp với mảnh PAY.

### 9.2. Phân mảnh ASG theo PROJ và EMP

- Phân mảnh PROJ theo LOC.
- Phân mảnh ASG theo PROJ bằng nửa nối.
- Có thể phân mảnh ASG tiếp theo EMP.
- Phân mảnh dẫn xuất có thể tạo chuỗi phụ thuộc PAY → EMP → ASG.

## 10. Bài tập kiểm tra trong slide

Quan hệ `QLSV(MA, HT, QQ, NS, GT, DT, TB)` xuất hiện ở trang 50.

1. Xác định tập vị từ đơn giản đầy đủ và cực tiểu.
2. Sinh tập vị từ hội sơ cấp $M$.
3. Tạo các mảnh ngang nguyên thủy.

Slide chưa cung cấp tập truy vấn hay tần số nên không xác định duy nhất phương án tối ưu chỉ từ lược đồ.

## Điểm cần nhớ — Phần 2

1. Phân mảnh phải đầy đủ, phục hồi được và tách biệt.
2. Phân mảnh ngang nguyên thủy dùng phép chọn, dẫn xuất thường dùng nửa nối.
3. Vị từ hội sơ cấp được tạo bằng phép AND các vị từ hoặc phủ định.
4. COM_MIN tìm tập vị từ đầy đủ và cực tiểu; PHORIZONTAL loại tổ hợp mâu thuẫn.
5. Cần luyện xác định $Pr$, $Pr'$, $M$ và các mảnh quan hệ.

---

# PHẦN 3 — THIẾT KẾ CSDL PHÂN TÁN: PHÂN MẢNH DỌC VÀ CẤP PHÁT

## 1. Phân mảnh dọc (Vertical Fragmentation)

**Phân mảnh dọc** chia quan hệ thành nhiều mảnh, mỗi mảnh chứa tập con thuộc tính và khóa quan hệ gốc.

- Gom thuộc tính thường được truy xuất cùng nhau vào một mảnh.
- Giảm truy cập thuộc tính không cần thiết và tăng hiệu năng.
- Giữ khóa để có thể nối và khôi phục quan hệ ban đầu.

**Ví dụ:** `NHANVIEN(MANV,TENNV,DIACHI,LUONG)` tạo `R1(MANV,TENNV,DIACHI)` và `R2(MANV,LUONG)`.

## 2. Các bước thiết kế phân mảnh dọc

**Sơ đồ Split Approach:** Ma trận sử dụng thuộc tính A → Ma trận lực hút AA → Thuật toán BEA tạo CA → Thuật toán phân mảnh dọc tìm điểm chia.

1. **A:** Xác định truy vấn sử dụng thuộc tính nào.
2. **AA:** Đo mức độ các thuộc tính được truy cập cùng nhau.
3. **BEA:** Sắp xếp thuộc tính có lực hút lớn gần nhau.
4. **Phân mảnh:** Chọn vị trí chia các nhóm thuộc tính.

## 3. Ma trận sử dụng thuộc tính (Attribute Usage Matrix)

Cho $R(A_1,\ldots,A_n)$ và $Q=\{q_1,\ldots,q_m\}$; ma trận A kích thước $m\times n$.

$use(q_i,A_j)=1$ nếu truy vấn $q_i$ dùng thuộc tính $A_j$, và bằng 0 nếu ngược lại.

**Ví dụ:** `PROJ(PNO,PNAME,BUDGET,LOC)`.

- $q_1$: Tìm kinh phí theo mã dự án.
- $q_2$: Lấy tên và kinh phí các dự án.
- $q_3$: Tìm tên dự án theo thành phố.
- $q_4$: Tính tổng kinh phí theo thành phố.

| Truy vấn | A₁: PNO | A₂: PNAME | A₃: BUDGET | A₄: LOC |
|---|---:|---:|---:|---:|
| q₁ | 1 | 0 | 1 | 0 |
| q₂ | 0 | 1 | 1 | 0 |
| q₃ | 0 | 1 | 0 | 1 |
| q₄ | 0 | 0 | 1 | 1 |

## 4. Ma trận lực hút thuộc tính (Attribute Affinity Matrix — AA)

**Affinity** đo mức độ hai thuộc tính được truy cập cùng nhau.

$$
aff(A_i,A_j)=\sum_{\substack{q_k:\ use(q_k,A_i)=1\\use(q_k,A_j)=1}}\sum_{S_l}ref_l(q_k)\,acc_l(q_k)
$$

- $ref_l(q_k)$ là số lần truy xuất cho mỗi lần thực hiện $q_k$ tại site $S_l$.
- $acc_l(q_k)$ là tần số thực hiện $q_k$ tại site $S_l$.
- Chỉ truy vấn dùng cả $A_i$ và $A_j$ mới đóng góp lực hút.

### Ví dụ tính AA

Giả sử $ref_l(q_k)=1$, tổng tần số $q_1=45$, $q_2=5$, $q_3=75$, $q_4=3$.

$aff(A_1,A_3)=15+20+10=45$.

| AA | A₁ | A₂ | A₃ | A₄ |
|---|---:|---:|---:|---:|
| A₁ | 45 | 0 | 45 | 0 |
| A₂ | 0 | 80 | 5 | 75 |
| A₃ | 45 | 5 | 53 | 3 |
| A₄ | 0 | 75 | 3 | 78 |

Hai cặp $A_1,A_3$ và $A_2,A_4$ có lực hút cao.

## 5. Thuật toán tụ nhóm BEA

**Bond Energy Algorithm (BEA)** hoán vị thuộc tính để đưa các thuộc tính có lực hút cao gần nhau.

### 5.1. Độ đo cầu nối (Bond)

$$bond(A_x,A_y)=\sum_{z=1}^{n}aff(A_z,A_x)\,aff(A_z,A_y)$$

Bond bằng tổng tích các phần tử cùng hàng của hai cột trong AA.

### 5.2. Độ đo đóng góp (Contribution)

$$cont(A_i,A_k,A_j)=2[bond(A_i,A_k)+bond(A_k,A_j)-bond(A_i,A_j)]$$

Chọn vị trí chèn có `cont` lớn nhất.

### 5.3. Các bước BEA

1. Đặt hai cột đầu của AA vào CA.
2. Lần lượt lấy thuộc tính chưa được sắp xếp.
3. Tính `cont` ở tất cả vị trí có thể chèn, gồm hai vị trí biên.
4. Chèn thuộc tính tại nơi có `cont` lớn nhất.
5. Sắp xếp hàng theo thứ tự cột để tạo CA cuối cùng.

### 5.4. Ví dụ BEA trang 24–32

- Với $A_3$, vị trí tốt nhất cho $cont(A_1,A_3,A_2)=10150$.
- Với $A_4$, giá trị lớn nhất là 23730 khi đặt bên phải $A_2$.
- Thứ tự cuối: $A_1,A_3,A_2,A_4$.

| CA | A₁ | A₃ | A₂ | A₄ |
|---|---:|---:|---:|---:|
| A₁ | 45 | 45 | 0 | 0 |
| A₃ | 45 | 53 | 5 | 3 |
| A₂ | 0 | 5 | 80 | 75 |
| A₄ | 0 | 3 | 75 | 78 |

## 6. Thuật toán phân mảnh dọc

### 6.1. Các tập cần xác định

- **TA (Top Attributes):** Nhóm thuộc tính trước điểm chia.
- **BA (Bottom Attributes):** Nhóm thuộc tính sau điểm chia.
- **TQ (Top Queries):** Truy vấn chỉ dùng TA.
- **BQ (Bottom Queries):** Truy vấn chỉ dùng BA.
- **OQ (Overlap Queries):** Truy vấn dùng cả TA và BA.

### 6.2. Chi phí truy cập

- $CTQ$ là tổng trọng số truy cập của TQ.
- $CBQ$ là tổng trọng số truy cập của BQ.
- $COQ$ là tổng trọng số truy cập của OQ.
- Trọng số được cộng từ $ref_j(q_i)\times acc_j(q_i)$ trên truy vấn và site thích hợp.

### 6.3. Hàm mục tiêu

$$\boxed{z=CTQ\times CBQ-COQ^2}$$

- Thử các vị trí chia trong CA.
- Tính CTQ, CBQ và COQ cho từng vị trí.
- Chọn điểm chia có $z$ lớn nhất trong các phương án hợp lệ.
- Bổ sung khóa vào các mảnh để phục hồi được quan hệ.

### 6.4. Ví dụ PROJ

Thứ tự $A_1,A_3,A_2,A_4$ gợi ý hai nhóm $\{A_1,A_3\}$ và $\{A_2,A_4\}$.

Phương án minh họa: `PROJ1(PNO,BUDGET)` và `PROJ2(PNO,PNAME,LOC)`.

Để khẳng định tối ưu, cần tính hàm $z$ ở tất cả điểm chia.

## 7. Phân mảnh lai (Hybrid Fragmentation)

**Hybrid Fragmentation** kết hợp phân mảnh ngang và dọc.

- Có thể chia ngang trước rồi chia dọc từng mảnh.
- Có thể chia dọc trước rồi chia ngang từng mảnh.
- Sơ đồ trang 46 minh họa $R$ chia ngang thành $R_1,R_2$ rồi tiếp tục chia dọc.
- Khôi phục bằng phép nối các mảnh dọc và phép hợp các mảnh ngang.

## 8. Nhân bản và cấp phát dữ liệu

### 8.1. Nhân bản dữ liệu (Replication)

- **Full Replication:** Nhân bản toàn phần.
- **Selective Replication:** Chỉ nhân bản một số mảnh.
- Nhân bản cải thiện đọc nhưng tăng công việc đồng bộ cập nhật.

### 8.2. Bài toán cấp phát (Allocation Problem)

Cho $F=\{F_1,\ldots,F_n\}$, $S=\{S_1,\ldots,S_m\}$ và tập ứng dụng $Q$.

Mục tiêu là đặt các mảnh để tối thiểu chi phí và thỏa ràng buộc hệ thống.

## 9. Thông tin phục vụ cấp phát

| Ký hiệu | Ý nghĩa |
|---|---|
| $Sel_i(F_j)$ | Số bộ của $F_j$ được $q_i$ truy xuất. |
| $Size(F_j)$ | Kích thước mảnh $F_j$. |
| $RR_{ij}$ | Số lần $q_i$ đọc $F_j$. |
| $UR_{ij}$ | Số lần $q_i$ ghi $F_j$. |
| $USC_k$ | Chi phí lưu trữ đơn vị tại site $S_k$. |
| $LPC_k$ | Chi phí xử lý đơn vị tại site $S_k$. |
| $g_{ij}$ | Chi phí truyền thông giữa hai site. |

$Size(F_j)=Card(F_j)\times length(F_j)$.

## 10. Mô hình tối ưu cấp phát

### 10.1. Biến quyết định

$x_{jk}=1$ nếu $F_j$ lưu tại site $S_k$, ngược lại $x_{jk}=0$.

### 10.2. Chi phí lưu trữ

$$STC_{jk}=USC_k\times Size(F_j)\times x_{jk}$$

### 10.3. Chi phí xử lý

$PC_i=AC_i+IE_i+CC_i$.

- **AC (Access Cost):** Chi phí đọc/ghi.
- **IE (Integrity Enforcement):** Chi phí bảo đảm toàn vẹn.
- **CC (Concurrency Control):** Chi phí điều khiển tương tranh.

### 10.4. Chi phí truyền thông

$TC_i=TCU_i+TCR_i$.

- $TCU_i$ là chi phí truyền phục vụ cập nhật.
- $TCR_i$ là chi phí truyền phục vụ truy vấn đọc.

### 10.5. Các ràng buộc

- Thời gian thực hiện truy vấn không vượt mức cho phép.
- Dung lượng lưu trữ tại từng site không bị vượt.
- Khối lượng xử lý không vượt khả năng của site.

### 10.6. Độ phức tạp

- Slide xếp bài toán cấp phát vào loại **NP-complete**.
- Bài toán có liên hệ với Knapsack và Network Flow.
- Có thể dùng heuristic hoặc giải bài toán không nhân bản trước rồi mới xét nhân bản.

## Điểm cần nhớ — Phần 3

1. Phân mảnh dọc gom các thuộc tính thường truy cập cùng nhau.
2. Quy trình: A → AA → BEA → CA → tìm điểm chia.
3. Thuộc công thức `bond`, `cont` và $z=CTQ\times CBQ-COQ^2$.
4. Cấp phát quyết định vị trí đặt mảnh nhằm giảm chi phí lưu trữ, xử lý và truyền thông.
5. Luyện tính AA, chạy BEA, tìm điểm chia và tính chi phí cấp phát.

---

# PHẦN 4 — XỬ LÝ VÀ TỐI ƯU HÓA TRUY VẤN

## 1. Mục đích và chức năng xử lý truy vấn

**Query Processing (xử lý truy vấn)** là biến đổi và thực hiện truy vấn để có kết quả đúng với chi phí hợp lý.

- Giảm thời gian thực thi truy vấn.
- Giảm bộ nhớ trung gian.
- Giảm lượng dữ liệu truyền giữa các site.
- Giảm CPU và I/O sử dụng.

### 1.1. Các phương pháp xử lý truy vấn

| Phương pháp | Nội dung |
|---|---|
| Algebraic Transformation | Biến đổi đại số để đơn giản hóa truy vấn tương đương. |
| Cost Estimation | Ước lượng chi phí dựa trên kích thước dữ liệu và phép toán. |

## 2. Quy trình thực hiện một truy vấn

**Sơ đồ trang 5–7:** SQL → Preprocessor → Query Optimizer → Query Code Generator → Runtime Database Processor → Kết quả.

- **Scanning:** Nhận diện từ khóa, bảng và thuộc tính.
- **Parsing:** Kiểm tra cú pháp, xây cây phân tích.
- **Validating:** Kiểm tra ngữ nghĩa, kiểu dữ liệu và tham chiếu.
- **Optimizer:** Chọn kế hoạch thực hiện.
- **Code Generator:** Sinh mã thực thi.
- **Runtime Processor:** Thực hiện truy vấn và trả kết quả.

## 3. Chuyển đổi SQL sang đại số quan hệ

| Phép toán | Ký hiệu | Ý nghĩa |
|---|---|---|
| Selection | $\sigma$ | Lọc bộ theo điều kiện. |
| Projection | $\pi$ | Lấy thuộc tính cần thiết. |
| Join | $\bowtie$ | Nối các quan hệ. |
| Semijoin | $\ltimes$ | Giữ bộ bên trái có giá trị nối phù hợp. |
| Union | $\cup$ | Hợp hai quan hệ. |
| Cartesian Product | $\times$ | Tích Descartes. |

Ví dụ: `SELECT TENNV FROM NHANVIEN WHERE LUONG > 2000000` tương ứng $\pi_{TENNV}(\sigma_{LUONG>2000000}(NHANVIEN))$.

## 4. Xử lý truy vấn tập trung và phân tán

### 4.1. Môi trường tập trung

- Tối ưu phép toán đại số quan hệ và thứ tự thực hiện.
- Chọn phương án thực hiện có chi phí phù hợp.
- Chủ yếu xét CPU, I/O và dữ liệu trung gian.

### 4.2. Môi trường phân tán

- Kế thừa kỹ thuật tối ưu tập trung.
- Xác định vị trí lưu dữ liệu.
- Chọn site xử lý và phương án truyền dữ liệu.
- Xét thêm chi phí truyền thông giữa các site.

## 5. Thuật toán INGRES

**INGRES-QOA** tối ưu truy vấn nhờ phân rã và phép thế bộ.

### 5.1. Phép tách (Detachment)

- Chia truy vấn phức tạp thành các truy vấn con.
- Ưu tiên phép chọn đơn quan hệ khi có thể.
- Dùng kết quả trung gian cho truy vấn tiếp theo.

### 5.2. Phép thế bộ (Tuple Substitution)

- Chọn quan hệ trong truy vấn đa quan hệ.
- Thay biến quan hệ bằng các bộ cụ thể.
- Tạo truy vấn ít biến hơn.

### 5.3. Quy trình INGRES-QOA

1. Nhận truy vấn đa quan hệ MRQ.
2. Nếu chỉ có một quan hệ, thực hiện trực tiếp.
3. Nếu nhiều quan hệ, áp dụng phép tách khi có thể.
4. Chọn quan hệ cho phép thế bộ.
5. Đệ quy xử lý các truy vấn còn lại.
6. Tổng hợp kết quả.

### 5.4. Ví dụ trong slide

Yêu cầu: Tìm tên nhân viên làm dự án tên `CSDL`, với `E=NHANVIEN`, `G=HOSO`, `J=DUAN`.

- $q_{11}$: Tìm MADA của dự án tên `CSDL`.
- $q_{12}$: Từ MADA, tìm MANV trong HOSO.
- $q_{13}$: Từ MANV, tìm TENNV trong NHANVIEN.

Chuỗi xử lý: $q_{11}\rightarrow q_{12}\rightarrow q_{13}$.

## 6. Sơ đồ phân lớp xử lý truy vấn phân tán

**Sơ đồ trang 15 và 27:** Phân rã truy vấn → Định vị dữ liệu → Tối ưu hóa toàn cục → Tối ưu hóa cục bộ.

1. Phân rã truy vấn dựa vào lược đồ tổng thể.
2. Định vị dữ liệu dựa vào lược đồ phân mảnh.
3. Tối ưu hóa toàn cục dựa vào thống kê mảnh và truyền thông.
4. Tối ưu hóa cục bộ dựa vào lược đồ và khả năng của từng site.

## 7. Phân rã truy vấn (Query Decomposition)

### 7.1. Chuẩn hóa (Normalization)

- **CNF (Conjunctive Normal Form):** Phép AND của các nhóm OR.
- **DNF (Disjunctive Normal Form):** Phép OR của các nhóm AND.
- Ví dụ CNF: $p\land(q\lor r)$.
- Ví dụ DNF: $(p\land q)\lor(p\land r)$.

Các luật logic:

- De Morgan: $\neg(p\land q)=\neg p\lor\neg q$.
- De Morgan: $\neg(p\lor q)=\neg p\land\neg q$.
- Phân phối: $p\land(q\lor r)=(p\land q)\lor(p\land r)$.
- Phân phối: $p\lor(q\land r)=(p\lor q)\land(p\lor r)$.
- Luật nuốt: $p\lor(p\land q)=p$.

### 7.2. Phân tích (Analysis)

- Kiểm tra ngữ nghĩa truy vấn.
- Kiểm tra quan hệ và điều kiện nối.
- Dùng đồ thị truy vấn để biểu diễn liên kết.
- Phát hiện phần truy vấn không đóng góp vào kết quả.

### 7.3. Loại bỏ dư thừa (Elimination of Redundancy)

- Loại điều kiện trùng lặp.
- Rút gọn bằng luật logic.
- Không thay đổi ngữ nghĩa kết quả.

### 7.4. Viết lại (Rewriting)

- Chuyển truy vấn sang cây đại số quan hệ.
- Dùng luật tương đương để biến đổi cây.
- Giảm kích thước dữ liệu trung gian.

## 8. Các luật biến đổi đại số quan hệ

### 8.1. Đẩy phép chọn xuống thấp

Nếu $p$ chỉ dùng thuộc tính của $R$: $\sigma_p(R\bowtie S)=\sigma_p(R)\bowtie S$.

### 8.2. Đẩy phép chiếu xuống thấp

- Loại những cột không cần thiết trước phép toán lớn.
- Giữ thuộc tính cần cho phép nối và các bước tiếp theo.

### 8.3. Giao hoán và kết hợp phép nối

- Có thể đổi thứ tự hai toán hạng nối khi phù hợp.
- Có thể đổi cách nhóm các phép nối tương đương.
- Thứ tự nối ảnh hưởng kích thước quan hệ trung gian.

### 8.4. Tính lũy đẳng

- Các phép chọn trùng điều kiện có thể rút gọn.
- Phép chiếu lồng nhau có thể rút gọn nếu tập thuộc tính phù hợp.

### 8.5. Quy tắc với phép toán hai ngôi

- Phép chọn có thể phân phối qua phép hợp khi phù hợp.
- Phép chiếu có thể phân phối qua phép hợp khi phù hợp.
- Mọi biến đổi phải giữ nguyên ngữ nghĩa.

## 9. Định vị dữ liệu phân tán

**Data Localization** thay quan hệ toàn cục bằng các mảnh phù hợp.

### 9.1. Rút gọn phân mảnh ngang nguyên thủy

- Thay quan hệ gốc bằng hợp các mảnh ngang.
- Loại mảnh không thể thỏa điều kiện lọc.
- Loại cặp mảnh không thể nối vì điều kiện mâu thuẫn.

### 9.2. Rút gọn phân mảnh dọc

- Thay quan hệ toàn cục bằng phép nối các mảnh dọc.
- Xác định thuộc tính cần cho kết quả và điều kiện.
- Bỏ mảnh không chứa thuộc tính cần thiết.
- Chỉ nối các mảnh khi truy vấn cần.

### 9.3. Rút gọn phân mảnh ngang dẫn xuất

- Dựa vào quan hệ giữa các mảnh sinh bằng nửa nối.
- Xác định cặp mảnh có dữ liệu nối phù hợp.
- Loại nhánh không đóng góp kết quả.

### 9.4. Rút gọn phân mảnh hỗn hợp

- Kết hợp rút gọn ngang và dọc.
- Loại mảnh ngang không thỏa điều kiện.
- Loại mảnh dọc không chứa thuộc tính cần thiết.
- Giảm phép nối và truyền dữ liệu.

## 10. Tối ưu hóa truy vấn phân tán

### 10.1. Mô hình chi phí

- **Total Cost:** Tổng tài nguyên được sử dụng ở tất cả site.
- **Response Time:** Thời gian từ lúc bắt đầu tới khi hoàn tất truy vấn.

### 10.2. Tổng chi phí

$$
\begin{aligned}
Total\_cost={}&C_{CPU}\times\#instr+C_{I/O}\times\#I/Os\\
&+C_{MSG}\times\#msgs+C_{TR}\times\#bytes
\end{aligned}
$$

- $C_{CPU}$ là chi phí một lệnh CPU.
- $C_{I/O}$ là chi phí một thao tác nhập/xuất.
- $C_{MSG}$ là chi phí khởi đầu và nhận thông báo.
- $C_{TR}$ là chi phí truyền một đơn vị dữ liệu.

### 10.3. Chi phí truyền thông

$CC(b)=C_{MSG}+C_{TR}\times b$, với $b$ là số byte thông báo.

### 10.4. Thời gian trả lời

$$
\begin{aligned}
Response\_time={}&C_{CPU}\times seq\_\#instr+C_{I/O}\times seq\_\#I/Os\\
&+C_{MSG}\times seq\_\#msgs+C_{TR}\times seq\_\#bytes
\end{aligned}
$$

Các đại lượng `seq_` thể hiện phần công việc tuần tự quyết định thời gian trả lời trong mô hình xử lý song song.

### 10.5. Ví dụ tổng chi phí và thời gian trả lời

Hai site truyền $x$ và $y$ byte tới một site khác.

- Tổng chi phí: $Total\_cost=2C_{MSG}+C_{TR}(x+y)$.
- Nếu truyền song song: $Response\_time=\max\{C_{MSG}+C_{TR}x,C_{MSG}+C_{TR}y\}$.
- Tổng chi phí và thời gian trả lời không nhất thiết cùng giảm.

## 11. Các thống kê dữ liệu

| Ký hiệu | Ý nghĩa |
|---|---|
| $card(R)$ | Số bộ của R. |
| $length(R)$ | Số byte mỗi bộ trong R. |
| $size(R)$ | Kích thước dữ liệu R. |
| $max(A)$ | Giá trị lớn nhất của A. |
| $min(A)$ | Giá trị nhỏ nhất của A. |
| $card(dom(A))$ | Số giá trị trong miền A. |
| $SFJ$ | Hệ số chọn phép nối. |
| $SFS$ | Hệ số chọn phép chọn. |

$size(R)=card(R)\times length(R)$.

## 12. Ước lượng lực lượng kết quả trung gian

### 12.1. Phép chọn (Selection)

$card(\sigma_F(R))=SFS(F)\times card(R)$.

| Điều kiện | Hệ số chọn ước lượng |
|---|---|
| $A=value$ | $1/card(\pi_A(R))$ |
| $A>value$ | $(max(A)-value)/(max(A)-min(A))$ |
| $A<value$ | $(value-min(A))/(max(A)-min(A))$ |
| $p\land q$ | $SFS(p)\times SFS(q)$ |
| $p\lor q$ | $SFS(p)+SFS(q)-SFS(p)SFS(q)$ |

Các ước lượng cần giả định phân bố và độc lập phù hợp.

### 12.2. Phép chiếu (Projection)

- Chiếu có loại trùng nên số bộ kết quả không vượt quan hệ gốc.
- Nếu tập chiếu chứa khóa, $card(\pi_A(R))=card(R)$.
- Tích Descartes: $card(R\times S)=card(R)\times card(S)$.

### 12.3. Phép nối (Join)

$$SFJ=\frac{card(R\bowtie S)}{card(R)\times card(S)}$$

$card(R\bowtie S)=SFJ\times card(R)\times card(S)$.

Cận trên là $card(R)\times card(S)$; nối khóa chính–khóa ngoại phụ thuộc số bộ có bản ghi tham chiếu phù hợp.

### 12.4. Phép nửa nối (Semijoin)

$card(R\ltimes S)=SFSJ(S.A)\times card(R)$.

Trong mô hình slide, $SFSJ(S.A)=card(\pi_A(S))/card(dom(A))$.

### 12.5. Phép hợp (Union)

Với quan hệ không chứa bộ lặp:

$$\max\{card(R),card(S)\}\le card(R\cup S)\le card(R)+card(S)$$

### 12.6. Phép hiệu (Difference)

$0\le card(R-S)\le card(R)$.

## 13. Ví dụ tối ưu cây truy vấn

Slide trang 82 yêu cầu tìm tên nhân viên đang quản lý một dự án.

- **Cách 1:** Tích Descartes NHANVIEN và HOSO rồi lọc.
- **Cách 2:** Lọc HOSO theo `NHIEMVU = "Quản lý"` rồi nối NHANVIEN.
- Cách 2 tránh tạo tích Descartes lớn và giảm dữ liệu trung gian.

## 14. Câu hỏi cuối chương

1. Mục đích và chức năng tối ưu hóa truy vấn phân tán.
2. Các phương pháp xử lý truy vấn cơ bản.
3. Ý tưởng INGRES và ví dụ minh họa.
4. Sơ đồ phân lớp xử lý truy vấn phân tán.
5. Chuyển SQL sang cây đại số quan hệ.
6. Các luật biến đổi cây đại số quan hệ.
7. Định vị dữ liệu và tối ưu cục bộ.
8. Tính tổng chi phí và thời gian trả lời.

## Điểm cần nhớ — Phần 4

1. Quy trình gồm phân rã, định vị, tối ưu toàn cục và tối ưu cục bộ.
2. INGRES dùng phép tách và phép thế bộ.
3. Đẩy phép chọn, phép chiếu xuống thấp để giảm dữ liệu trung gian.
4. Tổng chi phí và thời gian trả lời là hai tiêu chí khác nhau.
5. Luyện tính hệ số chọn, kích thước trung gian, chi phí truyền và so sánh kế hoạch.

---

# TỔNG HỢP TRỌNG TÂM ÔN THI CẢ 4 PHẦN

| Phần | Lý thuyết trọng tâm | Bài tập trọng tâm |
|---|---|---|
| 1. Tổng quan | DDB, DDBMS, tính trong suốt, nhân bản, kiến trúc. | Phân biệt phân mảnh và cấp phát; mô tả kiến trúc. |
| 2. Phân mảnh ngang | Vị từ đơn giản, hội sơ cấp, COM_MIN, PHORIZONTAL. | Tìm $Pr'$, $M$ và tạo các mảnh ngang. |
| 3. Phân mảnh dọc | Ma trận AA, BEA, bond, cont, hàm z. | Tính AA, chạy BEA, tìm điểm chia. |
| 3. Cấp phát | Nhân bản, chi phí lưu trữ, xử lý và truyền thông. | Tính chi phí cấp phát và kiểm tra ràng buộc. |
| 4. Tối ưu truy vấn | INGRES, cây đại số, luật biến đổi, hệ số chọn. | Tối ưu cây, ước lượng lực lượng và tính chi phí. |

**Thứ tự ưu tiên luyện bài tập:** Phân mảnh ngang → ma trận AA và BEA → phân mảnh dọc → biến đổi cây truy vấn → ước lượng và tối ưu chi phí.

**Lưu ý:** Một số công thức hoặc ký hiệu trên slide có thể thiếu hoặc không nhất quán; phần tóm tắt này là tài liệu ôn tập dựa trên nội dung đã tóm tắt trong cuộc trò chuyện.
