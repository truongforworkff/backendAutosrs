# Yêu cầu chức năng Backend và AI Service của AutoSRE

**Phiên bản:** 1.0  
**Nguồn yêu cầu duy nhất:** `AutoSRE_Literature_Review.docx`  
**Ngôn ngữ:** Tiếng Việt

## 1. Mục đích và phạm vi

Tài liệu này đặc tả yêu cầu chức năng cho hai phần của AutoSRE:

- **Backend**: control plane quản lý sự cố, bằng chứng, trạng thái workflow, chính sách, phê duyệt, kiểm toán và điều phối thực thi.
- **AI Service**: dịch vụ phân tích bằng chứng, xếp hạng giả thuyết nguyên nhân, đề xuất khắc phục và tạo ứng viên bản vá trong phạm vi được cho phép.

Frontend, nền tảng quan sát, kho mã, CI và môi trường chạy là hệ thống bên ngoài được tích hợp, không thuộc phạm vi triển khai của hai thành phần trên. Test harness là hệ thống đánh giá độc lập; nó không phải bộ chẩn đoán, không sửa mã dự án người dùng và không được chạm vào production.

Nguyên tắc bất biến: AI Service không có quyền phê duyệt, hợp nhất mã hoặc thực thi thay đổi trên môi trường vận hành. Mọi hành động có tác động phải do Backend kiểm tra chính sách, gắn với một phê duyệt hợp lệ và chuyển cho thành phần thực thi được kiểm soát.

## 2. Quy ước

### 2.1 Mức ưu tiên

| Mức | Ý nghĩa |
|---|---|
| Must | Bắt buộc để đáp ứng phạm vi cốt lõi hoặc rào chắn an toàn của AutoSRE. |
| Should | Quan trọng, nên có trong phạm vi sản phẩm; có thể hoãn nếu có lý do và kế hoạch rõ ràng. |
| Could | Bổ sung hữu ích, chỉ thực hiện sau khi các yêu cầu Must và Should đã được kiểm chứng. |

### 2.2 Thuật ngữ

| Thuật ngữ | Định nghĩa sử dụng trong tài liệu |
|---|---|
| Sự cố | Trạng thái dịch vụ lệch khỏi điều kiện mong muốn và cần được điều tra hoặc xử lý. |
| Bằng chứng | Dữ liệu có nguồn gốc kiểm tra được, gồm log, metric, trace, sự kiện thay đổi, deployment, phiên bản, build/test result hoặc mã nguồn liên quan. |
| Giả thuyết RCA | Một nguyên nhân có thể xảy ra, kèm mức tin cậy và bằng chứng ủng hộ, bác bỏ hoặc còn thiếu. |
| Khắc phục | Hành động nhằm phục hồi dịch vụ hoặc ngăn lỗi tái diễn. |
| Compatibility bridge | Thay đổi mã nhỏ do ứng dụng sở hữu để ánh xạ giao diện hoặc hành vi cũ sang phiên bản dependency mới. |
| Hành động có tác động | Hành động làm thay đổi workload, cấu hình, deployment, mã nguồn hoặc môi trường vận hành. |

## 3. Yêu cầu chức năng Backend

### 3.1 Quản lý sự cố và bối cảnh vận hành

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-01 | Backend phải tạo và duy trì một hồ sơ riêng cho mỗi sự cố được điều tra. | Must |
| BE-FR-02 | Backend phải liên kết sự cố với ứng dụng, dịch vụ và môi trường bị ảnh hưởng. | Must |
| BE-FR-03 | Backend phải liên kết sự cố với release, deployment, commit và thay đổi dependency liên quan khi dữ liệu này tồn tại. | Must |
| BE-FR-04 | Backend phải xây dựng dòng thời gian sự cố từ cảnh báo, dữ liệu quan sát và sự kiện thay đổi. | Must |
| BE-FR-05 | Backend phải lưu dấu thời gian, nguồn, dịch vụ, môi trường, phiên bản và mức độ đầy đủ của từng tín hiệu trong bối cảnh sự cố. | Must |
| BE-FR-06 | Backend phải phân biệt và lưu riêng triệu chứng, vị trí lỗi, nguyên nhân gốc rễ và biện pháp khắc phục. | Must |
| BE-FR-07 | Backend phải hỗ trợ cập nhật trạng thái sự cố theo workflow tường minh và lưu lịch sử chuyển trạng thái. | Must |

### 3.2 Thu thập và quản lý bằng chứng

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-08 | Backend phải tiếp nhận hoặc truy xuất log, metric, trace và sự kiện deployment/thay đổi liên quan đến sự cố. | Must |
| BE-FR-09 | Backend phải tương quan bằng chứng theo thời gian, định danh dịch vụ, môi trường, phiên bản, trace ID và commit/deployment ID khi các định danh này tồn tại. | Must |
| BE-FR-10 | Backend phải giữ tham chiếu đến dữ liệu gốc và metadata nguồn để người vận hành có thể kiểm tra lại bằng chứng. | Must |
| BE-FR-11 | Backend phải biểu diễn riêng ba trạng thái của một tín hiệu: có bằng chứng ủng hộ, có bằng chứng bác bỏ và không có hoặc thiếu bằng chứng. | Must |
| BE-FR-12 | Backend phải phát hiện và ghi nhận tín hiệu thiếu, lỗi thu thập hoặc không thể tương quan; không được diễn giải dữ liệu vắng mặt là hệ thống bình thường. | Must |
| BE-FR-13 | Backend phải cung cấp cho AI Service đúng bối cảnh và bằng chứng thuộc phạm vi sự cố, kèm định danh truy vết. | Must |

### 3.3 Quản lý kết quả RCA và đề xuất khắc phục

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-14 | Backend phải tiếp nhận và lưu danh sách giả thuyết RCA có thứ hạng từ AI Service. | Must |
| BE-FR-15 | Backend phải lưu với từng giả thuyết: nguyên nhân đề xuất, thành phần liên quan, mức tin cậy, bằng chứng ủng hộ, bằng chứng bác bỏ và bằng chứng còn thiếu. | Must |
| BE-FR-16 | Backend phải cho phép truy ngược từ kết luận RCA hoặc khuyến nghị về các bằng chứng nguồn đã sử dụng. | Must |
| BE-FR-17 | Backend phải lưu đề xuất khắc phục tách biệt với kết luận RCA và nêu rõ đề xuất là phục hồi tạm thời hay sửa lỗi lâu dài. | Must |
| BE-FR-18 | Khi sự cố đang ảnh hưởng môi trường chạy, Backend phải ưu tiên workflow ổn định hoặc rollback dịch vụ trước workflow tạo bản vá lâu dài. | Must |
| BE-FR-19 | Backend phải chuyển trường hợp không đủ bằng chứng hoặc độ tin cậy không đạt ngưỡng cấu hình sang trạng thái cần con người điều tra. | Must |

### 3.4 Phê duyệt và an toàn thực thi

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-20 | Backend phải yêu cầu phê duyệt của con người trước mọi hành động có tác động lên môi trường vận hành thực tế. | Must |
| BE-FR-21 | Mỗi quyết định phê duyệt hoặc từ chối phải gắn với đúng sự cố, đúng phiên bản hành động, phạm vi cho phép, người quyết định, thời điểm và thời hạn hiệu lực. | Must |
| BE-FR-22 | Backend phải từ chối thực thi nếu phê duyệt thiếu, hết hạn, đã bị thu hồi, không đúng hành động, không đúng phiên bản hoặc vượt phạm vi được cấp. | Must |
| BE-FR-23 | Backend phải kiểm tra lại trạng thái sự cố, phiên bản mục tiêu, chính sách, quyền và hiệu lực phê duyệt ngay trước khi gửi lệnh thực thi. | Must |
| BE-FR-24 | Backend phải bảo đảm một phê duyệt chỉ mở đúng hành động được duyệt và không cấp quyền rộng hơn cho AI Service, worker, CI hoặc service account. | Must |
| BE-FR-25 | Backend phải tách vai trò đề xuất, phê duyệt và thực thi; AI Service không được gọi trực tiếp đường thực thi production. | Must |
| BE-FR-26 | Backend phải hỗ trợ timeout, hủy và chạy lại tác vụ dài mà không làm mất trạng thái hoặc thực thi trùng hành động. | Must |
| BE-FR-27 | Backend phải ghi nhận trạng thái trước và sau thực thi, kết quả, lỗi và liên kết chúng với quyết định phê duyệt. | Must |
| BE-FR-28 | Backend phải theo dõi tín hiệu sau hành động để xác định tác động; khi điều kiện an toàn thất bại, Backend phải dừng bước tiếp theo và đưa workflow về xử lý thủ công hoặc rollback theo chính sách đã duyệt. | Must |

### 3.5 Workflow bản vá và compatibility bridge

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-29 | Backend phải chỉ khởi tạo workflow bản vá khi bằng chứng cho thấy nguyên nhân có liên quan đến mã nguồn hoặc dependency. | Must |
| BE-FR-30 | Backend phải cấp cho AI Service phạm vi sửa đổi rõ ràng gồm repository, base revision, tệp hoặc mô-đun được phép và giới hạn loại thay đổi. | Must |
| BE-FR-31 | Backend phải tạo hoặc yêu cầu tạo ứng viên bản vá trên nhánh tạm riêng; không được ghi trực tiếp vào nhánh được bảo vệ. | Must |
| BE-FR-32 | Backend phải lưu diff, dependency/lockfile trước và sau, provenance của công cụ hoặc mô hình và liên kết ứng viên bản vá với sự cố. | Must |
| BE-FR-33 | Backend phải kích hoạt build và kiểm thử trong sandbox hoặc runner cô lập, không cấp bí mật hay quyền production cho mã do AI tạo. | Must |
| BE-FR-34 | Backend phải thu thập và gắn kết quả build, kiểm thử chức năng và kiểm thử hồi quy với đúng commit của ứng viên bản vá. | Must |
| BE-FR-35 | Backend chỉ được chuyển bản vá sang trạng thái sẵn sàng review khi bản vá nằm trong phạm vi cho phép và các kiểm tra bắt buộc đã hoàn thành. | Must |
| BE-FR-36 | Backend phải yêu cầu lập trình viên review trước khi merge; build/test thành công không được xem là quyền tự động merge hoặc deploy. | Must |
| BE-FR-37 | Backend phải chuyển trường hợp vượt phạm vi sửa nhỏ — như thay đổi kiến trúc, dữ liệu lâu dài, xác thực hoặc nhiều mô-đun không liên quan — sang xử lý thủ công. | Must |

### 3.6 Kiểm toán và tái lập

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| BE-FR-38 | Backend phải lưu nhật ký kiểm toán bất biến cho thay đổi trạng thái, yêu cầu AI, đề xuất, quyết định phê duyệt, lệnh thực thi và kết quả. | Must |
| BE-FR-39 | Backend phải cho phép truy vết hai chiều từ sự cố đến bằng chứng, RCA, đề xuất, phê duyệt, bản vá, kết quả kiểm thử và hành động thực thi. | Must |
| BE-FR-40 | Backend phải lưu phiên bản workflow, chính sách, mô hình và cấu hình cần thiết để giải thích hoặc chạy lại một lần phân tích. | Should |

## 4. Yêu cầu chức năng AI Service

### 4.1 Tiếp nhận và chuẩn hóa đầu vào

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| AI-FR-01 | AI Service phải tiếp nhận gói phân tích có định danh sự cố, phạm vi, timeline, topology/phụ thuộc, deployment context và các bằng chứng được Backend cấp. | Must |
| AI-FR-02 | AI Service phải kiểm tra schema, nguồn và mức đầy đủ của đầu vào trước khi phân tích; đầu vào không hợp lệ phải được trả về bằng lỗi có cấu trúc. | Must |
| AI-FR-03 | AI Service phải bảo toàn tham chiếu tới bằng chứng nguồn trong toàn bộ quá trình phân tích. | Must |
| AI-FR-04 | AI Service phải áp dụng giới hạn dữ liệu được phép gửi tới nhà cung cấp mô hình và không được tự mở rộng quyền truy cập dữ liệu. | Must |

### 4.2 Phân tích nguyên nhân gốc rễ

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| AI-FR-05 | AI Service phải phân tích kết hợp log, metric, trace, lịch sử deployment và thay đổi mã/dependency thay vì kết luận chỉ từ một cảnh báo. | Must |
| AI-FR-06 | AI Service phải tạo danh sách giả thuyết RCA có thứ hạng và không đồng nhất triệu chứng hoặc vị trí phát hiện lỗi với nguyên nhân gốc rễ. | Must |
| AI-FR-07 | Mỗi giả thuyết phải nêu thành phần hoặc thay đổi nghi ngờ, lập luận, mức tin cậy và tham chiếu bằng chứng ủng hộ. | Must |
| AI-FR-08 | Mỗi giả thuyết phải nêu bằng chứng bác bỏ và bằng chứng còn thiếu khi có; không có dữ liệu không được coi là bằng chứng xác nhận. | Must |
| AI-FR-09 | Khi dữ liệu không đủ để đưa ra kết luận đáng tin cậy, AI Service phải trả trạng thái chưa đủ bằng chứng và đề xuất dữ liệu cần thu thập thêm. | Must |
| AI-FR-10 | AI Service phải trả kết quả theo hợp đồng dữ liệu có cấu trúc để Backend có thể lưu, hiển thị, kiểm tra và chấm tự động. | Must |

### 4.3 Đề xuất khắc phục

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| AI-FR-11 | AI Service phải tạo đề xuất khắc phục gắn với một hoặc nhiều giả thuyết RCA và nêu mục tiêu, phạm vi, điều kiện tiên quyết, rủi ro cùng tín hiệu kiểm chứng. | Must |
| AI-FR-12 | AI Service phải phân biệt biện pháp phục hồi tạm thời như rollback với sửa lỗi lâu dài như compatibility bridge. | Must |
| AI-FR-13 | AI Service chỉ được đề xuất hành động thuộc danh mục và phạm vi Backend cung cấp; hành động ngoài phạm vi phải được đánh dấu cần xử lý thủ công. | Must |
| AI-FR-14 | AI Service không được biểu diễn đề xuất hoặc kiểm thử thành công như một quyết định phê duyệt hay lệnh thực thi. | Must |

### 4.4 Sinh và kiểm tra ứng viên bản vá

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| AI-FR-15 | AI Service chỉ được sinh ứng viên bản vá khi nhận yêu cầu có phạm vi sửa mã hợp lệ từ Backend. | Must |
| AI-FR-16 | Với breaking dependency update, AI Service phải phân tích manifest, lockfile, build/test output, API bị thay đổi và các vị trí gọi liên quan trước khi tạo bản vá. | Must |
| AI-FR-17 | AI Service phải ưu tiên thay đổi nhỏ, có thể khoanh vùng và kiểm chứng; ứng viên compatibility bridge phải nêu cách ánh xạ đầu vào, đầu ra hoặc lỗi giữa giao diện cũ và mới. | Must |
| AI-FR-18 | AI Service phải từ chối tự động sửa và yêu cầu developer xử lý khi giải pháp cần thay đổi kiến trúc, dữ liệu lâu dài, xác thực hoặc nhiều mô-đun không liên quan. | Must |
| AI-FR-19 | AI Service phải trả bản vá dưới dạng diff cùng giải thích, danh sách tệp thay đổi, giả định, test đề xuất và bằng chứng liên quan. | Must |
| AI-FR-20 | AI Service phải hỗ trợ tiếp nhận kết quả build/test để đánh giá lại ứng viên; không được coi bản vá là an toàn chỉ vì mã đã được sinh thành công. | Must |
| AI-FR-21 | AI Service không được merge, deploy, rollback hoặc gọi trực tiếp môi trường vận hành. | Must |

### 4.5 Khả năng tái lập và kiểm soát mô hình

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| AI-FR-22 | AI Service phải ghi nhận model provider, model/version, prompt hoặc workflow version, tham số suy luận, thời lượng và mức sử dụng chi phí cho mỗi lần chạy. | Must |
| AI-FR-23 | AI Service phải hỗ trợ hủy, timeout và định danh idempotency cho yêu cầu phân tích hoặc sinh bản vá. | Must |
| AI-FR-24 | AI Service phải dùng giao diện đầu ra ổn định để Backend không phụ thuộc trực tiếp vào định dạng riêng của một nhà cung cấp mô hình. | Should |

## 5. Yêu cầu kiểm thử và đánh giá

### 5.1 Đánh giá RCA

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| EV-FR-01 | Hệ thống phải hỗ trợ chạy lại các ca có nhãn từ nguồn đánh giá RCA và so kết quả với đáp án được giữ kín đối với AI Service. | Must |
| EV-FR-02 | Kết quả đánh giá RCA phải tính ít nhất tỷ lệ nguyên nhân đúng ở Top-1 và Top-3. | Must |
| EV-FR-03 | Hệ thống phải báo cáo riêng kết quả theo nguồn dữ liệu và mức chi tiết của nhãn; không được gộp thành một chỉ số che khuất khác biệt phạm vi. | Must |
| EV-FR-04 | Hệ thống phải lưu đầu vào, phiên bản mô hình/workflow, đầu ra và kết quả chấm để lần đánh giá có thể được tái lập. | Must |

### 5.2 Đánh giá an toàn phê duyệt và thực thi

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| EV-FR-05 | Test harness phải kiểm tra các trường hợp không phê duyệt, từ chối, hết hạn, thu hồi, sai hành động, sai phiên bản và vượt phạm vi; Backend phải chặn thực thi trong mọi trường hợp này. | Must |
| EV-FR-06 | Test harness phải kiểm tra rằng AI Service không thể gọi đường thực thi production hoặc biến kết quả phân tích thành quyền thực thi. | Must |
| EV-FR-07 | Các thử nghiệm hành động phải chạy trong môi trường cách ly với kịch bản, giới hạn blast radius, timeout và phép đo trước–sau rõ ràng. | Must |
| EV-FR-08 | Kết quả replay dữ liệu phải được báo cáo riêng với kết quả thử nghiệm trên workload đang chạy; thành công ở replay không được coi là bằng chứng cơ chế phê duyệt hoặc rollback hoạt động đúng. | Must |

### 5.3 Đánh giá workflow bản vá

| ID | Yêu cầu chức năng | Ưu tiên |
|---|---|---|
| EV-FR-09 | Test harness phải có ít nhất một ca breaking dependency update gồm phiên bản cũ hoạt động, phiên bản mới gây lỗi, mã gọi API cũ và bộ kiểm thử hồi quy. | Must |
| EV-FR-10 | Ca bản vá phải kiểm tra toàn chuỗi: liên kết sự cố với thay đổi dependency, tạo ứng viên bridge/bản sửa nhỏ, build, kiểm thử sandbox và cung cấp bằng chứng cho developer review. | Must |
| EV-FR-11 | Việc chấm bản vá phải đánh giá ít nhất: build thành công, test bắt buộc đạt, test hồi quy đạt, thay đổi nằm trong phạm vi và không có quyền tự merge/deploy. | Must |
| EV-FR-12 | Kết quả trên SWE-bench Verified hoặc benchmark sửa issue tương đương phải được báo cáo bổ sung, không thay thế ca thử breaking dependency end-to-end của AutoSRE. | Should |

## 6. Quy tắc phân định trách nhiệm

| Quyết định hoặc hành động | Backend | AI Service | Con người / hệ thống ngoài |
|---|---|---|---|
| Lưu sự cố, bằng chứng và workflow | Chịu trách nhiệm | Cung cấp kết quả có cấu trúc | Nguồn dữ liệu cung cấp tín hiệu |
| Xếp hạng giả thuyết RCA | Lưu và kiểm soát vòng đời | Chịu trách nhiệm phân tích | Operator xem xét |
| Phê duyệt hành động | Xác thực và lưu quyết định | Không có quyền | Người có thẩm quyền quyết định |
| Thực thi thay đổi | Kiểm tra và điều phối | Không có quyền | Execution Engine thực hiện |
| Tạo ứng viên bản vá | Cấp phạm vi, tạo workflow, điều phối CI | Sinh diff trong phạm vi | Developer review và merge |
| Chấm đánh giá | Lưu cấu hình và kết quả | Là đối tượng được đánh giá | Test harness giữ đáp án và chấm |

## 7. Tiêu chí hoàn tất tối thiểu

Một phiên bản đáp ứng phạm vi cốt lõi khi tất cả yêu cầu **Must** liên quan đến năng lực đã triển khai đều có kiểm thử; riêng các rào chắn phê duyệt/thực thi phải đạt toàn bộ ca âm tính. Chất lượng RCA phải được báo cáo bằng Top-1/Top-3 trên dữ liệu có nhãn. Workflow bản vá chỉ được coi là hoàn tất khi có bằng chứng build, test sandbox, review của developer và không tồn tại đường tự động merge hoặc deploy production.

## 8. Cơ sở truy xuất từ Literature Review

Các nhóm yêu cầu trên được dẫn xuất duy nhất từ các nội dung sau trong Literature Review: phạm vi DevOps/SRE và chẩn đoán sự cố; RCA đa nguồn; ranh giới giữa AI và thực thi; human approval; sự cố do thay đổi phần mềm và dependency; compatibility bridge; sandbox, CI và developer review; test harness độc lập; replay có nhãn, Top-1/Top-3, chaos testing và ca breaking dependency update. Các lựa chọn công nghệ như Go/Gin, PostgreSQL, pgvector hoặc nhà cung cấp mô hình được xem là quyết định thiết kế, không được chuyển thành yêu cầu chức năng.