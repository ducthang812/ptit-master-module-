# Tóm tắt Cơ sở dữ liệu phân tán

Oct 13, 2026 · @thangntd1

## 1. Tổng quan về CSDL phân tán

**Hệ phân tán**: tập các thành phần tự trị, hoạt động cộng tác qua mạng truyền thông, thể hiện với người dùng như một hệ thống đơn lẻ.

**Hai mô hình xử lý phân tán**: Client/Server (client truy nhập trực tiếp dữ liệu trên server) và giao dịch chia nhỏ xử lý trên nhiều hệ thống (có máy trung tâm giám sát), hoặc kiểu ngang hàng (peer-to-peer).

**CSDL phân tán (CSDLPT)**: tập các CSDL có quan hệ logic với nhau, phân tán trên mạng máy tính. Hệ QTCSDLPT quản trị và làm cho sự phân tán trong suốt với người dùng. Hai khía cạnh cốt lõi:

- Tính phân tán: dữ liệu nằm trên nhiều vị trí (CSDL địa phương)
- Mối quan hệ logic: dữ liệu giữa các vị trí ràng buộc, phụ thuộc lẫn nhau

### So sánh đặc điểm với CSDL tập trung

| Đặc điểm | CSDL tập trung | CSDL phân tán |
| --- | --- | --- |
| Điều khiển | Tập trung bởi người QTCSDL | Người QTCSDL toàn cục + địa phương |
| Độc lập dữ liệu | Có | Có, cộng thêm "trong suốt phân tán" |
| Dư thừa dữ liệu | Tối thiểu hóa | Có chủ đích (nhân bản) để tăng cục bộ & sẵn sàng |
| Độ tin cậy | Phụ thuộc 1 site | Cao hơn, lỗi 1 site không ảnh hưởng toàn hệ thống |
| Hiệu năng | Giới hạn bởi 1 máy | Cải thiện nhờ phân mảnh + xử lý song song |
| Mở rộng | Khó, tốn kém | Dễ hơn, tăng dần theo mạng |

**Ưu điểm CSDLPT**: chia sẻ dữ liệu + điều khiển phân tán; độ tin cậy & tính sẵn sàng cao hơn (site hỏng không sập hệ thống); tăng tốc xử lý truy vấn (chia truy vấn con chạy song song).

**Nhược điểm**: chi phí phát triển phần mềm lớn; khó phát hiện lỗi trong thuật toán song song; chi phí xử lý/truyền thông giữa các vị trí tăng.

### Mô hình tham chiếu CSDLPT (mở rộng kiến trúc 3 mức ANSI/SPARC)

Gồm 4 lớp lược đồ: **Lược đồ toàn cục** (định nghĩa toàn bộ dữ liệu như CSDL tập trung) → **Lược đồ phân mảnh** (chia quan hệ toàn cục thành các mảnh Rᵢ) → **Lược đồ cấp phát** (xác định mảnh nào đặt ở site nào, ký hiệu Rᵢⱼ = mảnh i của quan hệ R tại site j) → **Lược đồ ánh xạ cục bộ** (ánh xạ vào hệ QTCSDL cục bộ của từng site).

### Ba chiều kiến trúc của hệ QTCSDLPT

- **Tính phân tán (Distribution)**: các thành phần hệ thống có nằm trên cùng một máy hay không.
- **Tính không đồng nhất (Heterogeneity)**: khác nhau ở phần cứng, hệ điều hành, mô hình dữ liệu, ngôn ngữ truy vấn...
- **Tính tự chủ, tự trị (Autonomy)**: tự chủ thiết kế, tự chủ truyền thông, tự chủ thực thi của mỗi DBMS.

Kết hợp 3 chiều này cho ra các kiểu kiến trúc: **Client/Server**, **Peer-to-peer Distributed DBMS** (ngang hàng), **Federated DBMS**, **Multi-DBMS** và **Distributed multi-DBMS**.

### Các kiến trúc hệ thống cụ thể

- **Kiến trúc ngang hàng**: mọi site bình đẳng, mỗi site có External Schema (ES), Global Conceptual Schema (GCS) dùng chung, Local Conceptual Schema (LCS) và Local Internal Schema (LIS) riêng.
- **Kiến trúc khách/chủ (Client/Server)**: tách chức năng thành 2 lớp — phía chủ (quản lý dữ liệu, tối ưu truy vấn, quản lý giao dịch) và phía khách (giao diện + một phần quản lý dữ liệu). Có hai dạng: Multi-client/Single-server (server dễ thành nút thắt cổ chai, điểm lỗi đơn) và Multi-client/Multi-server. Ưu điểm: phân chia công việc hiệu quả, dễ mở rộng tài nguyên, tỷ lệ giá/hiệu năng tốt hơn.
- **Kiến trúc đa hệ quản trị CSDL (Multi-DBMS)**: có một lớp Multi-DBMS Layer trung gian điều phối nhiều DBMS độc lập bên dưới (mỗi DBMS có CSDL riêng).

## 2. Thiết kế CSDLPT — Phân mảnh ngang

**Thiết kế CSDLPT gồm hai bước**: Phân mảnh (chia quan hệ toàn cục thành các mảnh có quan hệ logic) và Cấp phát (đặt mảnh vào site). Mục tiêu: chi phí truy vấn thấp nhất, cho phép nhiều giao dịch đồng thời và xử lý song song.

### Ba kiểu phân mảnh

- **Phân mảnh ngang**: thực chất là phép chọn (σ) — lấy các bộ thỏa điều kiện.
- **Phân mảnh dọc**: thực chất là phép chiếu (π) — lấy tập con thuộc tính + khóa.
- **Phân mảnh hỗn hợp**: kết hợp cả hai.

### Bốn quy tắc đảm bảo phân mảnh đúng đắn

1. **Tính đầy đủ**: mọi mục dữ liệu (bộ với phân mảnh ngang, thuộc tính với phân mảnh dọc) của R phải thuộc ít nhất một mảnh con.
2. **Tính phục hồi**: quan hệ toàn cục phục hồi được bằng hợp (ngang) hoặc kết nối tự nhiên (dọc) các mảnh con.
3. **Tính tách biệt**: các mảnh con không chồng lấp (ngang: Rᵢ∩Rⱼ=∅; dọc: chỉ chung thuộc tính khóa).

### Phân mảnh ngang: hai phương pháp

- **Phân mảnh ngang nguyên thủy**: dựa trên vị từ của chính quan hệ đó.
- **Phân mảnh ngang dẫn xuất**: dựa trên vị từ của quan hệ khác có liên kết (thường theo khóa ngoài), qua kết nối bằng nhau (equijoin) hoặc nửa nối (semijoin).

**Các khái niệm then chốt**: vị từ đơn giản (pⱼ: Aᵢ θ value), vị từ hội sơ cấp (hội các vị từ đơn giản, ở dạng tự nhiên hoặc phủ định), mảnh hội sơ cấp fᵢ = σ\_mᵢ(R). Số mảnh ngang = số vị từ hội sơ cấp.

**Tập vị từ cần đầy đủ và cực tiểu**:

- Đầy đủ: mọi bộ trong cùng một mảnh có xác suất được truy xuất như nhau bởi mọi ứng dụng.
- Cực tiểu: mọi vị từ đều có tính liên đới (relevant) — nghĩa là có ít nhất một ứng dụng truy xuất khác nhau giữa 2 mảnh con do vị từ đó tách ra.

**Thuật toán COM\_MIN**: sinh tập vị từ Pr' đầy đủ + cực tiểu từ tập vị từ đơn giản Pr ban đầu (lặp: tìm vị từ có tính liên đới, loại các vị từ không còn liên đới).

**Thuật toán PHORIZONTAL**: sau COM\_MIN, sinh tập M các vị từ hội sơ cấp, sau đó loại bỏ các vị từ hội sơ cấp mâu thuẫn với tập các phép kéo theo (implication) I giữa các vị từ trong Pr' — tránh sinh ra các mảnh vô nghĩa (rỗng).

**Phân mảnh ngang dẫn xuất**: quan hệ R được phân mảnh dựa theo phân mảnh (đã có) của quan hệ S liên quan, ví dụ: R₁ = R ⋉ S₁, R₂ = R ⋉ S₂. Có thể xảy ra **phân mảnh dây chuyền** (một phân mảnh kéo theo phân mảnh của quan hệ khác, cứ thế lan truyền). Một quan hệ có thể có nhiều cách phân mảnh khác nhau; lựa chọn tối ưu phụ thuộc vào tập ứng dụng và chiến lược cấp phát.

## 3. Phân mảnh dọc, phân mảnh hỗn hợp, cấp phát & nhân bản dữ liệu

### Phân mảnh dọc

Gom các thuộc tính **thường được truy xuất cùng nhau** vào một mảnh (mỗi mảnh giữ khóa chung để phục hồi bằng kết nối). Quy trình gồm 3 bước:

1. **Ma trận sử dụng thuộc tính A** = (use(qᵢ, Aⱼ)): ghi 1 nếu ứng dụng qᵢ tham chiếu thuộc tính Aⱼ.
2. **Ma trận lực hút thuộc tính AA** (Attribute Affinity Matrix): đo mức độ hai thuộc tính Aᵢ, Aⱼ "đi cùng nhau" trong các ứng dụng, tính theo tần số truy cập ref và acc tại từng site.
3. **Thuật toán tụ nhóm BEA** (Bond Energy Algorithm): hoán vị hàng/cột của AA để tối đa số đo lực hút chung AM, tạo ra **ma trận lực hút tụ nhóm CA** (Clustered Affinity Matrix) — các thuộc tính có lực hút cao được xếp cạnh nhau.

**Thuật toán PARTITION**: từ CA, xác định điểm tách tối ưu k (1≤k≤n) sao cho z = CTQ×CBQ − COQ² đạt giá trị lớn nhất, trong đó:

- CTQ = chi phí truy xuất của các ứng dụng chỉ dùng tập đỉnh TA (Top)
- CBQ = chi phí của các ứng dụng chỉ dùng tập đáy BA (Bottom)
- COQ = chi phí của các ứng dụng dùng cả hai tập (overlap — muốn nhỏ)

Kết quả: R₁ = π\_TA(R) ∪ K, R₂ = π\_BA(R) ∪ K (K là khóa chính).

### Phân mảnh lai (hỗn hợp)

Kết hợp phân mảnh ngang và dọc theo hai hướng: ngang trước rồi dọc, hoặc ngược lại. Khôi phục mảnh lai dùng **phép kết nối (VF)** cho nhánh dọc và **phép hợp (HF)** cho nhánh ngang.

### Cấp phát và nhân bản dữ liệu

- **Nhân bản (Replication)**: quyết định mảnh nào lưu nhiều bản sao — nhân bản toàn phần hoặc nhân bản có lựa chọn.
- **Cấp phát (Allocation)**: quyết định mảnh nào đặt ở site nào.

**Bài toán cấp phát**: cho F={F₁,...,Fₙ} (mảnh), S={S₁,...,Sₘ} (site mạng), Q={q₁,...,qᵩ} (ứng dụng) → tìm cách phân bổ F vào S tối ưu theo:

- Chi phí tối thiểu (truyền thông + lưu trữ + xử lý đọc/ghi)
- Hiệu năng (thời gian đáp ứng, thông lượng)
- Ràng buộc (dung lượng lưu trữ, khả năng xử lý tại mỗi site)

**Hàm tổng chi phí TOC** = Σ chi phí lưu trữ (STC) + Σ chi phí truy vấn (QPC). Chi phí truy vấn QPCᵢ = chi phí xử lý PCᵢ (truy cập + toàn vẹn + điều khiển tương tranh) + chi phí truyền thông TCᵢ (cập nhật TCU + truy xuất TCR).

Bài toán cấp phát dữ liệu là **NP-complete** (tương đồng bài toán knapsack, luồng mạng). Cách tiếp cận thực tế thường dùng kinh nghiệm (heuristic): giải bài toán cấp phát không nhân bản trước, rồi xử lý nhân bản ở bước 2.

## 4. Xử lý truy vấn và tối ưu hóa truy vấn

### Mục đích & chức năng

Giảm thiểu thời gian xử lý, vùng nhớ trung gian, chi phí truyền thông giữa các trạm và tài nguyên sử dụng — bằng cách biến đổi truy vấn phức tạp thành truy vấn tương đương đơn giản hơn, đúng đắn và hiệu quả.

**Hai phương pháp cơ bản**: biến đổi đại số (đơn giản hóa bằng các phép biến đổi tương đương, không quan tâm kích thước dữ liệu) và ước lượng chi phí (tính kích thước dữ liệu + thời gian thực hiện từng phép toán).

### Quy trình thực hiện 1 truy vấn trong DBMS

Câu truy vấn (ngôn ngữ cấp cao) → **Preprocessor** (Scanning: nhận diện từ khóa, tên bảng/thuộc tính; Parsing: kiểm tra cú pháp, dựng Parse Tree; Validating: kiểm tra ngữ nghĩa) → **Query Optimizer** (chọn chiến lược thực hiện tối ưu) → **Query Code Generator** (sinh mã) → **Runtime Database Processor** (thực thi, trả kết quả).

### Sơ đồ xử lý truy vấn phân tán (4 lớp)

1. **Phân rã truy vấn** (tại trạm điều khiển): chuyển truy vấn phân tán → truy vấn đại số trên quan hệ phân tán. Gồm 4 bước:
   - **Chuẩn hóa**: đưa về dạng chuẩn hội hoặc chuẩn tuyển.
   - **Phân tích**: loại truy vấn sai kiểu, sai ngữ nghĩa (dùng đồ thị truy vấn — nếu không liên thông thì sai ngữ nghĩa).
   - **Loại bỏ dư thừa**: đơn giản hóa vị từ bằng các luật đại số mệnh đề (luỹ đẳng, phân phối, De Morgan...).
   - **Viết lại**: chuyển sang cây đại số quan hệ, rồi cấu trúc lại cây bằng 6 nhóm luật biến đổi (giao hoán, kết hợp, lũy đẳng, chọn/chiếu giao hoán với phép hai ngôi...) để đẩy phép chọn/chiếu xuống sâu, giảm kích thước trung gian.
2. **Định vị dữ liệu** (dùng lược đồ phân mảnh): biến truy vấn đại số tổng thể thành truy vấn trên các mảnh vật lý, rút gọn bằng cách loại các mảnh/nối vô ích:
   - Rút gọn với phân mảnh ngang nguyên thủy (loại mảnh có vị từ mâu thuẫn với điều kiện chọn, loại phép nối giữa các mảnh có điều kiện phân mảnh đối lập)
   - Rút gọn với phân mảnh dọc (loại phép chiếu vô ích không chứa thuộc tính cần)
   - Rút gọn với phân mảnh gián tiếp và phân mảnh hỗn hợp (kết hợp các luật trên)
3. **Tối ưu hóa toàn cục** (tại trạm điều khiển): chọn chiến lược thực hiện tốt nhất dựa trên thống kê dữ liệu (lực lượng, kích thước mảnh, hệ số chọn SF).
4. **Tối ưu hóa cục bộ** (tại các trạm địa phương): tối ưu truy vấn con chạy trên từng site.

### Mô hình chi phí

- **Tổng chi phí (Total cost)** = CCPU×#lệnh + CI/O×#lần vào/ra + CMSG×#thông báo + CTR×#byte truyền.
- **Thời gian trả lời (Response time)**: tính theo đường tuần tự dài nhất (các thao tác có thể song song nên thường nhỏ hơn tổng chi phí).
- Cần các **thống kê dữ liệu**: card(R) (số bộ), size(R), length thuộc tính, hệ số chọn phép nối SF\_J = card(R⋈S) / (card(R)×card(S))...
- Công thức ước lượng lực lượng kết quả cho phép chọn, chiếu, nối, nửa nối, hợp, trừ — dùng để so sánh chi phí giữa các cây truy vấn tương đương.

### Giải thuật heuristic (quy tắc suy diễn để tối ưu cây đại số)

1. Tách các phép chọn liên kết (σ\_p1∧p2) thành dãy các phép chọn đơn.
2. Đẩy phép chọn xuống càng sâu càng tốt (đến gần quan hệ gốc).
3. Tái tổ chức để thực hiện phép chọn có tính chọn lọc nhất trước.
4. Kết hợp tích Decartes với phép chọn phù hợp sau đó thành phép nối (⋈).
5. Đẩy phép chiếu xuống càng sâu càng tốt (có thể sinh phép chiếu mới).
6. Gom nhóm các phép chọn liên tiếp.
7. Loại bỏ các phép chiếu vô ích.

**Thuật toán INGRES**: phối hợp phân rã và tối ưu hóa — câu truy vấn đa quan hệ được tách dần (bằng **phép tách** detachment) thành chuỗi truy vấn con có chung 1 quan hệ; truy vấn **bất khả giản** (không tách được nữa) dùng **phép thế bộ** (tuple substitution) để biến thành truy vấn đơn quan hệ, xử lý đệ quy bởi bộ xử lý một biến (OVQP) cho đến khi không còn truy vấn đa quan hệ.
