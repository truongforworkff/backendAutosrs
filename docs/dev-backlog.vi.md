# AutoSRE — backlog công việc phát triển

**Mục đích:** Danh sách công việc để PM phân công cho nhóm phát triển.  
**Cơ sở:** `AutoSRE_Literature_Review.docx` và tài liệu Functional Requirements Backend & AI Service đã dẫn xuất từ nguồn đó.  
**Phạm vi:** Sản phẩm end-to-end: backend/control plane, AI Service, giao diện vận hành, kết nối hệ thống, môi trường chạy và harness đánh giá.

## Cách dùng backlog

- PM tạo ticket theo ID dưới đây; giữ nguyên ID khi đồng bộ sang công cụ quản lý công việc.
- Mỗi task có phạm vi hoàn thành và tiêu chí nghiệm thu. “Owner” là vai trò phù hợp, chưa phải tên người được giao.
- `P0` là nền tảng hoặc rào chắn an toàn cần làm trước; `P1` là năng lực cốt lõi tiếp theo; `P2` là phần bổ sung có thể xếp sau MVP.
- Estimate, sprint, người nhận, deadline và nhà cung cấp/phiên bản cụ thể chưa được nguồn quyết định; PM và nhóm chốt khi refinement.
- Task có phụ thuộc chỉ nên bắt đầu sau khi dependency đã có contract/đầu ra đủ dùng. Các task đánh dấu **Spike** cần trả lại quyết định hoặc tiêu chí, không mặc định tạo framework mới.

## Bản đồ workstream

| Workstream | Kết quả |
|---|---|
| WS1 — Nền tảng và hợp đồng | Môi trường chạy lặp lại, mô hình dữ liệu, hợp đồng API giữa Backend và AI Service |
| WS2 — Incident và evidence | Hồ sơ sự cố, timeline, dữ liệu đa tín hiệu có nguồn gốc và mức độ đầy đủ |
| WS3 — RCA và remediation | Phân tích có thứ hạng, bằng chứng truy xuất được, đề xuất có giới hạn |
| WS4 — Approval và execution | Phê duyệt gắn hành động, kiểm tra chính sách, thực thi có kiểm soát và kiểm toán |
| WS5 — Patch workflow | Nhánh riêng, diff, sandbox/CI, kết quả kiểm thử và developer review |
| WS6 — Operator UI | Giao diện điều tra, xem bằng chứng, duyệt hành động và review ứng viên bản vá |
| WS7 — Evaluation | Replay có nhãn, ca kiểm tra an toàn, kịch bản breaking dependency và báo cáo |

## WS1 — Nền tảng và hợp đồng

### AS-DEV-001 — Dựng môi trường phát triển và kiểm thử lặp lại

- **Ưu tiên:** P0 · **Owner:** DevOps/Backend · **Phụ thuộc:** Không
- **Công việc:** Tạo cách chạy local cho Backend, AI Service, PostgreSQL và workload thử nghiệm bằng cấu hình có thể lặp lại. Ghi rõ cấu hình ngoài phạm vi tự động hóa.
- **Nghiệm thu:** Thành viên mới có thể khởi động môi trường theo hướng dẫn; health check của các dịch vụ chạy được; dữ liệu thử nghiệm không trỏ tới production; cấu hình bí mật không được commit.
- **Truy vết:** Literature Review — Compose cho phát triển, môi trường cô lập; FR BE-FR-26, EV-FR-07.

### AS-DEV-002 — Định nghĩa mô hình dữ liệu và migration ban đầu

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-001
- **Công việc:** Mô hình hóa Incident, service/environment, deployment/change, evidence reference, RCA hypothesis, remediation, approval, action/execution, patch candidate, test result và audit event. Dùng migration có thể nâng cấp/khôi phục.
- **Nghiệm thu:** Migration tạo được schema mới và nâng cấp từ schema trước; quan hệ incident–action–approval không thể trỏ chéo sai; có constraint/transaction cho các trạng thái an toàn cốt lõi.
- **Truy vết:** FR BE-FR-01–07, BE-FR-14–17, BE-FR-21, BE-FR-27, BE-FR-38–40.

### AS-DEV-003 — Chốt hợp đồng API Backend ↔ AI Service (**Spike**)

- **Ưu tiên:** P0 · **Owner:** Backend + AI · **Phụ thuộc:** AS-DEV-002
- **Công việc:** Chốt schema request/response cho gói incident context/evidence, RCA hypotheses, remediation và patch candidate; bao gồm version, trace/correlation ID, validation errors, timeout và idempotency.
- **Nghiệm thu:** Có schema máy đọc được và ví dụ hợp lệ/không hợp lệ; hai dịch vụ validate cùng bộ ví dụ; response luôn giữ được ID bằng chứng nguồn; không có trường biểu diễn approval hoặc quyền execution trong payload AI.
- **Truy vết:** FR BE-FR-13–16, AI-FR-01–03, AI-FR-07–10, AI-FR-15, AI-FR-19, AI-FR-23.

### AS-DEV-004 — Thiết lập xác thực dịch vụ và kiểm soát quyền

- **Ưu tiên:** P0 · **Owner:** Backend/Platform · **Phụ thuộc:** AS-DEV-001, AS-DEV-003
- **Công việc:** Thiết lập danh tính riêng cho Backend, AI Service, worker, CI runner và execution adapter; chỉ cấp quyền tối thiểu theo vai trò.
- **Nghiệm thu:** AI Service không có credential gọi production executor hoặc merge/deploy; CI runner không đọc secret production; các request không có danh tính hợp lệ bị từ chối và ghi nhận.
- **Truy vết:** FR BE-FR-24–25, BE-FR-33; AI-FR-04, AI-FR-21; EV-FR-06.

## WS2 — Incident và evidence

### AS-DEV-005 — API vòng đời incident và timeline

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-002
- **Công việc:** Tạo, đọc, cập nhật trạng thái incident; liên kết service/environment, release/deployment/commit và ghi lịch sử chuyển trạng thái.
- **Nghiệm thu:** Incident có ID ổn định; timeline sắp xếp được theo timestamp/source; phân biệt symptom, fault location, suspected cause và remediation; trạng thái chuyển theo workflow được định nghĩa.
- **Truy vết:** FR BE-FR-01–07.

### AS-DEV-006 — Adapter ingest telemetry và deployment events

- **Ưu tiên:** P0 · **Owner:** Backend/Integration · **Phụ thuộc:** AS-DEV-002, AS-DEV-005
- **Công việc:** Tiếp nhận/truy xuất log, metric, trace và deployment/change events từ nguồn observability/SCM/CI được chọn ở refinement.
- **Nghiệm thu:** Bản ghi giữ source, timestamp, service/environment, release/version và correlation IDs khi có; payload nguồn hoặc đường dẫn tới payload có thể truy xuất lại; lỗi connector được lưu thay vì bỏ qua.
- **Truy vết:** Literature Review — liên kết source/deployment/telemetry; FR BE-FR-03–05, BE-FR-08–10.

### AS-DEV-007 — Tương quan bằng chứng và xử lý dữ liệu thiếu

- **Ưu tiên:** P0 · **Owner:** Backend/Data · **Phụ thuộc:** AS-DEV-006
- **Công việc:** Chuẩn hóa định danh, tương quan theo thời gian, dịch vụ, phiên bản, trace ID và commit/deployment ID; biểu diễn rõ bằng chứng ủng hộ, bác bỏ, chưa có hoặc không thu thập được.
- **Nghiệm thu:** Ca kiểm thử chứng minh timestamp lệch hoặc thiếu trace ID không tạo liên kết nhân quả giả; thiếu tín hiệu được gắn trạng thái unknown/incomplete, không thành “bình thường”; mỗi evidence có nguồn.
- **Truy vết:** FR BE-FR-05, BE-FR-09–12.

### AS-DEV-008 — API/context builder cho AI Service

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-003, AS-DEV-007
- **Công việc:** Đóng gói incident context và evidence đúng phạm vi, cắt dữ liệu theo chính sách đã định và gửi yêu cầu phân tích có thể truy vết.
- **Nghiệm thu:** Request chỉ chứa dữ liệu của incident được chọn; mọi item có evidence ID/source; lỗi thiếu dữ liệu được trả lại rõ ràng; cùng request ID có thể truy vết từ Backend tới AI Service.
- **Truy vết:** FR BE-FR-13; AI-FR-01–04.

## WS3 — RCA và remediation

### AS-DEV-009 — Dịch vụ phân tích RCA đa tín hiệu

- **Ưu tiên:** P1 · **Owner:** AI · **Phụ thuộc:** AS-DEV-003, AS-DEV-008
- **Công việc:** Phân tích log, metric, trace, deployment history và thay đổi mã/dependency; tạo hypotheses có thứ hạng, lý do và evidence references.
- **Nghiệm thu:** Output schema hợp lệ; mỗi hypothesis có suspect component/change, confidence và bằng chứng ủng hộ; symptom/fault location không bị ghi như root cause nếu chưa có căn cứ; chạy được bộ ca kiểm thử mẫu.
- **Truy vết:** FR AI-FR-05–10.

### AS-DEV-010 — Nhánh từ chối kết luận khi bằng chứng chưa đủ

- **Ưu tiên:** P1 · **Owner:** AI · **Phụ thuộc:** AS-DEV-009
- **Công việc:** Bổ sung kết quả “insufficient evidence”, liệt kê dữ liệu cần thu thêm và lý do làm giảm độ tin cậy.
- **Nghiệm thu:** Ca không có telemetry hoặc telemetry lỗi trả trạng thái chưa đủ bằng chứng; không tự điền giả định thành evidence; Backend nhận được trạng thái có cấu trúc.
- **Truy vết:** FR BE-FR-11–12, BE-FR-19; AI-FR-08–09.

### AS-DEV-011 — Lưu, truy xuất và xem lại RCA/remediation

- **Ưu tiên:** P1 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-009
- **Công việc:** Lưu hypotheses, thứ hạng, confidence, evidence links, missing/disconfirming evidence và đề xuất remediation riêng biệt.
- **Nghiệm thu:** Mỗi kết luận truy ngược được tới evidence gốc; remediation chỉ ra mục tiêu, phạm vi, rủi ro và tín hiệu kiểm chứng; phân biệt phục hồi tạm thời với sửa lâu dài.
- **Truy vết:** FR BE-FR-14–19.

### AS-DEV-012 — Xây dựng danh mục remediation và phạm vi action

- **Ưu tiên:** P1 · **Owner:** Backend + AI · **Phụ thuộc:** AS-DEV-003, AS-DEV-011
- **Công việc:** Khai báo loại action được đề xuất và giới hạn cho từng action; các action ngoài danh mục được đưa sang người vận hành.
- **Nghiệm thu:** Mọi action proposal có loại, target, phạm vi, tiền điều kiện, rủi ro và tín hiệu xác nhận; AI không thể nới phạm vi bằng text tự do; không action nào tự tạo authorization.
- **Truy vết:** FR AI-FR-11–14; BE-FR-20–25.

### AS-DEV-013 — Adapter rollback và khôi phục trong môi trường thử nghiệm

- **Ưu tiên:** P1 · **Owner:** Backend/Platform · **Phụ thuộc:** AS-DEV-012, AS-DEV-017
- **Công việc:** Thực thi rollback hoặc khôi phục đã được cấp phép trên workload thử nghiệm, thu trạng thái trước/sau và tín hiệu sức khỏe.
- **Nghiệm thu:** Chỉ nhận action được phép và phê duyệt hợp lệ; thao tác ngoài sandbox bị từ chối trong môi trường thử; lưu kết quả và health signal; lỗi thì dừng workflow/đưa về manual handling.
- **Truy vết:** Literature Review — ưu tiên ổn định trước sửa lâu dài; FR BE-FR-18, BE-FR-23, BE-FR-27–28.

## WS4 — Approval và execution safety

### AS-DEV-014 — Mô hình action và quyết định approval

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-002, AS-DEV-012
- **Công việc:** Lưu action version/hash, incident ID, target/scope, approver, decision, timestamp, expiry và revoke state.
- **Nghiệm thu:** Quyết định gắn duy nhất với nội dung action đã duyệt; thay đổi action làm approval cũ không còn hợp lệ; decision và lịch sử không bị ghi đè mất dấu.
- **Truy vết:** FR BE-FR-21, BE-FR-27, BE-FR-38–39.

### AS-DEV-015 — Gate phê duyệt và kiểm tra ngay trước thực thi

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-014
- **Công việc:** Thực thi policy gate cho missing/denied/expired/revoked/mismatched/out-of-scope approvals; kiểm tra lại incident, target version và quyền ngay trước lệnh.
- **Nghiệm thu:** Mỗi trường hợp âm tính đều bị chặn; approval không tái sử dụng cho action khác; validation chạy sát điểm thực thi; tất cả quyết định được audit.
- **Truy vết:** FR BE-FR-20–25; EV-FR-05.

### AS-DEV-016 — Orchestration job bền vững, timeout, cancel và idempotency

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-002, AS-DEV-003
- **Công việc:** Chạy tác vụ suy luận/build/execution dài ngoài vòng đời HTTP request và lưu trạng thái job để tiếp tục hoặc hủy có kiểm soát.
- **Nghiệm thu:** Timeout và cancel ghi được trạng thái; retry cùng idempotency key không tạo action trùng; worker restart không làm mất quyết định/phê duyệt; lỗi có thể truy vết.
- **Truy vết:** Literature Review — tác vụ dài cần trạng thái bền vững; FR BE-FR-07, BE-FR-26; AI-FR-23.

### AS-DEV-017 — Execution Engine giới hạn quyền cho môi trường thử nghiệm

- **Ưu tiên:** P0 · **Owner:** Backend/Platform · **Phụ thuộc:** AS-DEV-004, AS-DEV-015
- **Công việc:** Tạo cổng thực thi duy nhất cho action được phép; ban đầu gắn với test environment và quyền tối thiểu.
- **Nghiệm thu:** Không có đường gọi trực tiếp từ AI Service; mọi lệnh có approval reference hợp lệ; target/scope được kiểm tra server-side; action, trạng thái trước/sau, kết quả và lỗi được ghi audit.
- **Truy vết:** FR BE-FR-23–28; EV-FR-06–07.

### AS-DEV-018 — Audit trail end-to-end

- **Ưu tiên:** P0 · **Owner:** Backend · **Phụ thuộc:** AS-DEV-005, AS-DEV-008, AS-DEV-014–17
- **Công việc:** Ghi sự kiện phân tích, quyết định, thực thi, test, patch và mọi chuyển trạng thái bằng correlation IDs.
- **Nghiệm thu:** Từ incident có thể truy đến evidence, RCA, remediation, approval, action/patch, test results và outcome; audit record ghi actor/time/version; không có thao tác state transition quan trọng bị thiếu.
- **Truy vết:** FR BE-FR-27, BE-FR-38–40; AI-FR-22.

## WS5 — Patch workflow

### AS-DEV-019 — Phân tích dependency breaking change

- **Ưu tiên:** P1 · **Owner:** AI · **Phụ thuộc:** AS-DEV-003, AS-DEV-008
- **Công việc:** Dùng manifest, lockfile, build/test failure, API change và điểm gọi ứng dụng để xác định vùng sửa nhỏ có thể kiểm chứng.
- **Nghiệm thu:** Kết quả phân tích chỉ ra dependency/version, API hoặc call sites liên quan và evidence; nếu liên quan chưa rõ thì yêu cầu điều tra thêm; không mặc định minor/patch update là an toàn.
- **Truy vết:** Literature Review — dependency breakage; FR AI-FR-16–18.

### AS-DEV-020 — Sinh patch candidate hoặc compatibility bridge

- **Ưu tiên:** P1 · **Owner:** AI · **Phụ thuộc:** AS-DEV-019
- **Công việc:** Tạo diff nhỏ trong vùng cho phép, giải thích mapping API cũ/mới, liệt kê assumption và test cần chạy.
- **Nghiệm thu:** Output là diff áp dụng được trên base revision xác định; tệp thay đổi nằm trong allowlist; ngoài phạm vi thì từ chối và giải thích; không có lệnh merge/deploy.
- **Truy vết:** FR AI-FR-15, AI-FR-17–21.

### AS-DEV-021 — Quản lý patch branch và review handoff

- **Ưu tiên:** P1 · **Owner:** Backend/SCM Integration · **Phụ thuộc:** AS-DEV-014, AS-DEV-020
- **Công việc:** Tạo branch riêng cho candidate, giữ base revision/diff, đính kèm incident và cung cấp đường review cho developer.
- **Nghiệm thu:** Candidate không sửa protected branch; mọi commit gắn incident và action version; không tự merge; developer có thể xem diff và kết quả CI trước quyết định.
- **Truy vết:** FR BE-FR-30–32, BE-FR-35–37.

### AS-DEV-022 — Sandbox build/test cho patch do AI đề xuất

- **Ưu tiên:** P0 · **Owner:** CI/Platform · **Phụ thuộc:** AS-DEV-004, AS-DEV-021
- **Công việc:** Chạy build, test chức năng và hồi quy trên runner cô lập với dependency/build input có thể tái lập.
- **Nghiệm thu:** Runner không có secret/quyền production; test result gắn commit chính xác; ghi lại version/build metadata; failure không tạo merge/deploy; có thể dọn workload sandbox.
- **Truy vết:** FR BE-FR-32–36; EV-FR-09–11.

### AS-DEV-023 — Tiếp nhận kết quả CI và khóa trạng thái review

- **Ưu tiên:** P1 · **Owner:** Backend/CI Integration · **Phụ thuộc:** AS-DEV-022
- **Công việc:** Nhận status/check results và đưa candidate sang `ready_for_review` chỉ khi policy/test bắt buộc đạt.
- **Nghiệm thu:** Kết quả test cũ hoặc của commit khác không được gắn nhầm; test thiếu/failed giữ candidate ở trạng thái không đạt; review của developer là bước riêng và được lưu lại.
- **Truy vết:** FR BE-FR-34–36; EV-FR-10–11.

## WS6 — Operator UI

### AS-DEV-024 — Màn hình danh sách và chi tiết incident

- **Ưu tiên:** P1 · **Owner:** Frontend · **Phụ thuộc:** AS-DEV-005, AS-DEV-011
- **Công việc:** Tạo màn hình xem incident, service/environment, deployment context, timeline và trạng thái workflow.
- **Nghiệm thu:** Người vận hành có thể mở sự cố và thấy timeline đã sắp xếp; UI phân biệt symptom, fault location, RCA và remediation; trạng thái đang điều tra/đang chờ/đã xử lý hiển thị rõ.
- **Truy vết:** Literature Review — Control Panel; FR BE-FR-01–07.

### AS-DEV-025 — Màn hình evidence và RCA có truy xuất nguồn

- **Ưu tiên:** P1 · **Owner:** Frontend · **Phụ thuộc:** AS-DEV-007, AS-DEV-011
- **Công việc:** Hiển thị hypotheses theo thứ hạng, confidence, evidence ủng hộ/bác bỏ, missing data và liên kết evidence gốc.
- **Nghiệm thu:** Người xem mở được dữ liệu nguồn qua tham chiếu; trạng thái “không có dữ liệu” khác “bằng chứng bác bỏ”; kết luận chưa đủ dữ liệu được thể hiện rõ.
- **Truy vết:** Literature Review — giải thích evidence; FR BE-FR-10–19, AI-FR-07–10.

### AS-DEV-026 — Màn hình remediation và approval

- **Ưu tiên:** P0 · **Owner:** Frontend + Backend · **Phụ thuộc:** AS-DEV-014–15, AS-DEV-024
- **Công việc:** Hiển thị action version, target/scope, risk, preconditions, expected signals, approver, trạng thái expiry/revoke; gửi quyết định approve/reject tới Backend.
- **Nghiệm thu:** Người duyệt nhìn thấy đúng nội dung action sẽ được phép; action đổi thì approval cũ không áp dụng; UI không thể bỏ qua backend gate; trạng thái pending/approved/denied/expired/revoked rõ ràng.
- **Truy vết:** Literature Review — approval gắn incident-action; FR BE-FR-20–25.

### AS-DEV-027 — Màn hình patch diff và test results

- **Ưu tiên:** P1 · **Owner:** Frontend · **Phụ thuộc:** AS-DEV-021–23
- **Công việc:** Hiển thị patch diff, file list, assumptions, CI results và developer review status.
- **Nghiệm thu:** Diff và kết quả thuộc cùng commit; failure/pending/complete phân biệt; không có nút auto-merge/deploy production; developer review được ghi nhận.
- **Truy vết:** FR BE-FR-32–37, AI-FR-19–21.

## WS7 — Evaluation và báo cáo

### AS-DEV-028 — Replay harness cho RCA dataset

- **Ưu tiên:** P1 · **Owner:** AI/Evaluation · **Phụ thuộc:** AS-DEV-009
- **Công việc:** Chạy ca labeled từ nguồn RCA được chọn; đáp án chuẩn do harness giữ riêng, không đưa vào input AI.
- **Nghiệm thu:** Chạy lại được cùng input/version; tính Top-1 và Top-3; kết quả tách theo dataset và granularity nhãn; lưu input/model/workflow/output/score.
- **Truy vết:** Literature Review — Nezha, RCAEval, OpenRCA; FR EV-FR-01–04.

### AS-DEV-029 — Ca kiểm thử approval và execution âm tính

- **Ưu tiên:** P0 · **Owner:** QA/Backend · **Phụ thuộc:** AS-DEV-015–18
- **Công việc:** Tạo harness tests cho approval missing/denied/expired/revoked/mismatch/out-of-scope, stale action version, retry và quyền AI Service.
- **Nghiệm thu:** Mọi trường hợp âm tính đều chứng minh không có lệnh execution được gửi; retry không gây action trùng; bằng chứng test liên kết tới audit records.
- **Truy vết:** FR EV-FR-05–08.

### AS-DEV-030 — Kịch bản breaking dependency end-to-end

- **Ưu tiên:** P0 · **Owner:** QA + Backend + AI · **Phụ thuộc:** AS-DEV-019–23
- **Công việc:** Dựng repository thử có version dependency cũ hoạt động và version mới gây lỗi; chạy nhận diện, RCA, candidate patch, sandbox tests và review handoff.
- **Nghiệm thu:** Harness giữ ground truth; AutoSRE liên kết lỗi với dependency change; patch/test evidence có thể truy xuất; test pass không tự merge/deploy; kịch bản tái lập từ đầu.
- **Truy vết:** Literature Review — ca thử dependency update; FR EV-FR-09–12.

### AS-DEV-031 — Kịch bản lỗi vận hành cách ly và hậu kiểm

- **Ưu tiên:** P1 · **Owner:** QA/Platform · **Phụ thuộc:** AS-DEV-013, AS-DEV-017
- **Công việc:** Dùng workload thử nghiệm để kiểm tra phát hiện, telemetry, hành động được duyệt và tín hiệu sau hành động; thêm network fault injection nếu phù hợp.
- **Nghiệm thu:** Có giới hạn blast radius, thời hạn và baseline/post-action metrics; replay result tách khỏi live workload experiment; không có target production.
- **Truy vết:** Literature Review — Chaos Mesh/Toxiproxy; FR BE-FR-28, EV-FR-07–08.

### AS-DEV-032 — Báo cáo đánh giá và ngưỡng phát hành

- **Ưu tiên:** P1 · **Owner:** QA/Evaluation + PM · **Phụ thuộc:** AS-DEV-028–31
- **Công việc:** Tổng hợp RCA Top-1/Top-3, patch outcomes, false/blocked execution attempts và các giới hạn dữ liệu theo từng loại thử nghiệm.
- **Nghiệm thu:** Báo cáo tách replay, action tests, workload experiments và patch benchmark; nêu model/workflow version; không suy rộng kết quả một benchmark thành độ an toàn production; PM có checklist bằng chứng để quyết định release.
- **Truy vết:** FR EV-FR-01–12; Literature Review — giới hạn phạm vi benchmark và cần đánh giá thực nghiệm.

## Thứ tự triển khai gợi ý

1. **Nền tảng an toàn:** AS-DEV-001–004, AS-DEV-014–18 và AS-DEV-029.
2. **Đường đi RCA:** AS-DEV-005–011, AS-DEV-024–25 và AS-DEV-028.
3. **Khắc phục có kiểm soát:** AS-DEV-012–13, AS-DEV-026, AS-DEV-031.
4. **Patch lifecycle:** AS-DEV-019–23, AS-DEV-027 và AS-DEV-030.
5. **Đánh giá và release decision:** AS-DEV-032.

Các nhóm có thể chạy song song sau khi API/schema chung được thống nhất, nhưng không bật execution ngoài sandbox trước khi approval gate và quyền tối thiểu được nghiệm thu.

## Quyết định PM cần chốt khi refinement

- Người nhận task, estimate, sprint goal, deadline và Definition of Done theo năng lực nhóm.
- Nguồn telemetry/SCM/CI cụ thể, loại action được hỗ trợ đầu tiên và môi trường sandbox/Kubernetes mục tiêu.
- Model provider/version và dữ liệu nào được phép gửi tới API; ngân sách và giới hạn lưu trữ.
- Ngưỡng confidence và tiêu chí khi nào cần handoff cho con người; FR yêu cầu hỗ trợ trạng thái này nhưng nguồn không đặt giá trị ngưỡng.
- API auth, retention/redaction, dashboard UX detail và tiêu chí hiệu năng; các giá trị định lượng chưa được Literature Review quy định.

## Ranh giới MVP và ngoài phạm vi

MVP nên chứng minh được một luồng hoàn chỉnh trên môi trường thử nghiệm: ingest tín hiệu → incident/timeline → RCA có evidence → remediation candidate → approval gate → sandbox execution/test → audit/result. Ca breaking dependency update cần đi qua patch branch, build/test và developer review. Benchmark RCA và workload experiment được báo cáo riêng.

Không giao task cho AI tự quyết định hoặc tự thay đổi production; không xem test harness là thành phần vận hành; không chuyển các lựa chọn công nghệ hoặc con số chưa được nguồn xác định thành cam kết nghiệm thu.
