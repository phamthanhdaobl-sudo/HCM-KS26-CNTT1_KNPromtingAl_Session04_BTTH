# HCM-KS26-CNTT1_KNPromtingAl_Session04_BTTH
# BÁO CÁO BÀI TẬP THỰC HÀNH TỔNG HỢP
## XÂY DỰNG GIẢI PHÁP AI AGENT BẰNG KỸ THUẬT PROMPTING NÂNG CAO (LOGIC & CHUỖI)

**Họ và tên:** Phạm Thành Đạo  
**Lớp:** HCM-KS26-CNTT1  
**Môn học:** Kỹ thuật Prompting Nâng cao – Session 04

---

# I. PHÂN TÍCH BÀI TOÁN

## 1. Tên giải pháp

Xây dựng AI Agent tự động tiếp nhận, phân tích và xử lý thư khiếu nại khách hàng tại FastDelivery.

## 2. Bối cảnh thực tế

FastDelivery là doanh nghiệp hoạt động trong lĩnh vực công nghệ và logistics. Mỗi ngày, doanh nghiệp tiếp nhận nhiều thư khiếu nại liên quan đến quá trình vận chuyển như:

- Hàng hóa bị hư hỏng.
- Đơn hàng giao chậm.
- Thất lạc hàng hóa.
- Giao sai sản phẩm.
- Khách hàng chưa nhận được hàng.

Nếu sử dụng một Mega Prompt để thực hiện tất cả công việc như đọc thư, phân loại sự cố, tra cứu chính sách, đánh giá mức độ ưu tiên và soạn email phản hồi, AI có thể bỏ sót thông tin hoặc đưa ra kết quả thiếu chính xác.

Vì vậy, em đề xuất xây dựng hệ thống AI Agent sử dụng Prompt Chaining để chia quy trình thành nhiều trạm độc lập, kết hợp Chain-of-Thought để xử lý logic và cơ chế khôi phục ngữ cảnh để duy trì công việc khi bị gián đoạn.

## 3. Mục tiêu giải pháp

- Tự động đọc và phân loại thư khiếu nại.
- Trích xuất các thông tin quan trọng của đơn hàng.
- Kiểm tra và đối chiếu chính sách xử lý.
- Đưa ra hướng giải quyết phù hợp.
- Tự động tạo bản thảo email phản hồi khách hàng.
- Phát hiện dữ liệu thiếu, sai định dạng hoặc mâu thuẫn.
- Duy trì tiến trình khi AI bị ngắt quãng hoặc mất ngữ cảnh.

## 4. Công nghệ và kỹ thuật sử dụng

| Kỹ thuật | Vai trò trong hệ thống |
|---|---|
| Chain-of-Thought | Phân tích logic và xử lý từng bước |
| Zero-shot CoT | Kích hoạt lập luận bằng chỉ dẫn suy xét từng bước |
| Prompt Chaining | Chia quy trình thành nhiều trạm |
| Data Handoff | Chuyển kết quả giữa các trạm bằng JSON |
| Validation Gates | Kiểm tra dữ liệu trước khi chuyển trạm |
| Context Anchoring | Duy trì vai trò và mục tiêu |
| Rolling Summary | Tóm tắt trạng thái xử lý |
| Continue Prompting | Tiếp tục khi phản hồi bị ngắt |
| Session Reset | Khôi phục công việc trong phiên mới |
| Human-in-the-Loop | Yêu cầu nhân viên phê duyệt trường hợp rủi ro |

# II. BƯỚC 1: THIẾT KẾ KIẾN TRÚC GIẢI PHÁP

## 1. Sơ đồ quy trình tổng thể

Hệ thống được thiết kế thành 4 trạm xử lý liên kết tuần tự.

```text
          INPUT
     Thư khiếu nại khách hàng
              |
              v
   +-----------------------+
   | TRẠM 1: PHÂN LOẠI     |
   | Phân tích nội dung    |
   | Trích xuất dữ liệu    |
   +-----------------------+
              |
              v
   +-----------------------+
   | VALIDATION GATE 1     |
   | Kiểm tra dữ liệu      |
   | Phát hiện lỗi         |
   +-----------------------+
              |
              v
   +-----------------------+
   | TRẠM 2: TRA CỨU       |
   | Đối chiếu chính sách  |
   | Xác định hướng xử lý  |
   +-----------------------+
              |
              v
   +-----------------------+
   | VALIDATION GATE 2     |
   | Kiểm tra chính sách   |
   | Kiểm tra căn cứ       |
   +-----------------------+
              |
              v
   +-----------------------+
   | TRẠM 3: LẬP PHƯƠNG ÁN |
   | Đánh giá logic        |
   | Đề xuất giải quyết    |
   +-----------------------+
              |
              v
   +-----------------------+
   | TRẠM 4: SOẠN EMAIL    |
   | Tạo bản thảo phản hồi |
   +-----------------------+
              |
              v
   +-----------------------+
   | KIỂM TRA CUỐI         |
   | Kiểm tra nội dung     |
   +-----------------------+
              |
              v
       OUTPUT HOÀN CHỈNH
```

Nếu dữ liệu không hợp lệ tại bất kỳ Validation Gate nào, hệ thống sẽ dừng chuyển tiếp và yêu cầu bổ sung hoặc xác minh.

## 2. Thiết kế Input

Hệ thống tiếp nhận các trường thông tin sau:

| Trường dữ liệu | Mô tả |
|---|---|
| Mã khiếu nại | Mã định danh yêu cầu |
| Mã đơn hàng | Mã đơn cần kiểm tra |
| Tên khách hàng | Người gửi khiếu nại |
| Email | Địa chỉ liên hệ |
| Nội dung | Nội dung khách hàng phản ánh |
| Thời gian | Thời điểm phát sinh sự cố |
| Loại sự cố | Hư hỏng, giao chậm, thất lạc... |
| Chính sách | Dữ liệu chính sách được doanh nghiệp cung cấp |

## 3. Thiết kế Output

Kết quả cuối cùng phải bao gồm:

1. Mã khiếu nại.
2. Thông tin khách hàng.
3. Loại sự cố.
4. Mức độ ưu tiên.
5. Kết quả kiểm tra dữ liệu.
6. Chính sách áp dụng.
7. Phương án giải quyết.
8. Bản thảo email phản hồi.
9. Trạng thái xử lý.
10. Các thông tin cần nhân viên xác minh.

# III. BƯỚC 2: XÂY DỰNG CHUỖI PROMPT THỰC THI

## 1. Prompt hệ thống (System Prompt)

Đây là prompt nền tảng được duy trì xuyên suốt toàn bộ quy trình.

```text
VAI TRÒ:
Bạn là AI Agent chuyên xử lý khiếu nại khách hàng
của doanh nghiệp logistics FastDelivery.

MỤC TIÊU:
Tiếp nhận, phân tích, kiểm tra và đề xuất phương án
xử lý khiếu nại một cách chính xác, nhất quán.

NGUYÊN TẮC:
1. Luôn duy trì vai trò và mục tiêu xử lý khiếu nại.
2. Chỉ sử dụng dữ liệu được cung cấp.
3. Không tự tạo thông tin hoặc chính sách.
4. Không bỏ qua dữ liệu bị thiếu hoặc sai lệch.
5. Không tự xác nhận bồi thường hoặc gửi email.
6. Phải kiểm tra dữ liệu trước khi chuyển sang bước tiếp theo.
7. Mỗi trạm phải trả kết quả theo định dạng yêu cầu.

KỸ THUẬT XỬ LÝ:
- Sử dụng lập luận logic từng bước cho các nhiệm vụ
  có nhiều điều kiện.
- Phân chia công việc theo Prompt Chaining.
- Chuyển giao dữ liệu bằng JSON sạch.
- Tóm tắt trạng thái khi hội thoại kéo dài.
- Khôi phục đúng vị trí khi phản hồi bị ngắt.

XỬ LÝ LỖI:
Nếu thiếu dữ liệu quan trọng:
Trả về NEED_MORE_DATA.

Nếu dữ liệu sai hoặc mâu thuẫn:
Trả về NEED_VERIFICATION.

Nếu dữ liệu hợp lệ:
Cho phép chuyển sang trạm tiếp theo.

BẢO MẬT:
Không tiết lộ thông tin khách hàng cho người không
có thẩm quyền.
Không thực hiện hành động gửi email hoặc thanh toán
khi chưa được nhân viên phê duyệt.
```

## 2. Trạm 1 – Phân loại và trích xuất thông tin

**Mục tiêu:** Đọc nội dung thư, nhận diện loại sự cố và trích xuất dữ liệu quan trọng.

**Prompt 1:**

```text
Bạn là chuyên viên tiếp nhận khiếu nại FastDelivery.

Hãy phân tích nội dung thư khách hàng cung cấp.

YÊU CẦU:
1. Xác định loại sự cố.
2. Trích xuất mã đơn hàng.
3. Trích xuất tên khách hàng.
4. Xác định nội dung khiếu nại.
5. Đánh giá mức độ ưu tiên:
   - Cao: Thất lạc hoặc hư hỏng nghiêm trọng.
   - Trung bình: Giao hàng chậm hoặc sai sản phẩm.
   - Thấp: Yêu cầu thông tin thông thường.

Sử dụng tư duy logic từng bước để kiểm tra thông tin.

Nếu thiếu mã đơn hàng hoặc thông tin cần thiết,
đánh dấu NEED_MORE_DATA.

Nếu dữ liệu mâu thuẫn, đánh dấu NEED_VERIFICATION.

ĐẦU RA:
Chỉ trả về JSON hợp lệ, không thêm lời chào
hoặc nội dung giải thích bên ngoài JSON.

CẤU TRÚC:
{
  "complaint_id": "",
  "order_id": "",
  "customer_name": "",
  "issue_type": "",
  "priority": "",
  "description": "",
  "validation_status": ""
}
```

## 3. Validation Gate 1 – Kiểm định dữ liệu

**Mục tiêu:** Ngăn dữ liệu lỗi từ Trạm 1 truyền sang Trạm 2.

**Prompt kiểm tra:**

```text
Bạn là bộ phận kiểm định dữ liệu FastDelivery.

Hãy kiểm tra JSON từ Trạm 1.

ĐIỀU KIỆN:
- Mã đơn hàng không được để trống.
- Tên khách hàng không được để trống.
- Loại sự cố phải xác định được.
- Mức độ ưu tiên phải thuộc Cao, Trung bình hoặc Thấp.
- Không chấp nhận JSON sai cấu trúc.

KẾT QUẢ:
Nếu hợp lệ:
VALIDATED = TRUE.

Nếu thiếu thông tin:
VALIDATED = FALSE.
ERROR = NEED_MORE_DATA.

Nếu dữ liệu mâu thuẫn:
VALIDATED = FALSE.
ERROR = NEED_VERIFICATION.

Chỉ cho phép chuyển sang Trạm 2 khi dữ liệu
đã vượt qua tất cả điều kiện kiểm tra.
```

## 4. Trạm 2 – Tra cứu chính sách

**Mục tiêu:** Đối chiếu thông tin khiếu nại với chính sách doanh nghiệp.

**Prompt 2:**

```text
Bạn là chuyên viên tra cứu chính sách FastDelivery.

INPUT:
Nhận JSON đã được kiểm định từ Trạm 1
và dữ liệu chính sách do doanh nghiệp cung cấp.

NHIỆM VỤ:
1. Xác định chính sách liên quan đến sự cố.
2. Đối chiếu điều kiện áp dụng.
3. Kiểm tra thời hạn khiếu nại nếu có dữ liệu.
4. Xác định khách hàng có đáp ứng điều kiện không.
5. Ghi rõ căn cứ cho kết luận.

RÀNG BUỘC:
- Không tự sáng tạo chính sách.
- Không suy đoán điều kiện chưa được cung cấp.
- Nếu không tìm thấy chính sách phù hợp,
  ghi POLICY_NOT_FOUND.
- Nếu thiếu dữ liệu để kết luận,
  ghi NEED_VERIFICATION.

ĐẦU RA JSON:
{
  "order_id": "",
  "policy_name": "",
  "policy_condition": "",
  "eligibility": "",
  "evidence": "",
  "validation_status": ""
}
```

## 5. Validation Gate 2 – Kiểm tra chính sách

**Mục tiêu:** Kiểm tra tính phù hợp giữa khiếu nại và chính sách trước khi đề xuất phương án.

**Prompt kiểm tra:**

```text
Bạn là bộ phận kiểm soát chất lượng FastDelivery.

Hãy kiểm tra kết quả từ Trạm 2.

KIỂM TRA:
- Chính sách có được cung cấp không?
- Chính sách có phù hợp với loại sự cố không?
- Có căn cứ xác định điều kiện áp dụng không?
- Kết luận có mâu thuẫn với dữ liệu đầu vào không?

Nếu tất cả hợp lệ:
Cho phép chuyển sang Trạm 3.

Nếu không có chính sách:
Dừng và yêu cầu nhân viên tra cứu.

Nếu thiếu căn cứ:
Dừng và yêu cầu xác minh.

Không tự ý phê duyệt bồi thường.
```

## 6. Trạm 3 – Phân tích logic và đề xuất giải pháp

**Mục tiêu:** Sử dụng Chain-of-Thought để đánh giá các điều kiện và đề xuất hướng xử lý.

**Prompt 3:**

```text
Bạn là chuyên viên xử lý sự cố FastDelivery.

INPUT:
- Dữ liệu khách hàng từ Trạm 1.
- Kết quả chính sách từ Trạm 2.

NHIỆM VỤ:
Hãy phân tích theo trình tự:

BƯỚC 1:
Xác định loại sự cố khách hàng gặp phải.

BƯỚC 2:
Đối chiếu điều kiện của chính sách.

BƯỚC 3:
Đánh giá mức độ phù hợp giữa sự cố và chính sách.

BƯỚC 4:
Đưa ra phương án giải quyết.

CÁC TRƯỜNG HỢP:
- Đủ điều kiện: Đề xuất xử lý theo chính sách.
- Chưa đủ dữ liệu: Yêu cầu bổ sung.
- Không đủ căn cứ: Chuyển nhân viên xác minh.
- Chính sách không áp dụng: Đề xuất hướng xử lý khác
  nhưng không được tự tạo chính sách.

Sử dụng lập luận logic đa bước.
Kiểm tra lại kết luận trước khi xuất kết quả.

ĐẦU RA JSON:
{
  "order_id": "",
  "issue_analysis": "",
  "policy_result": "",
  "proposed_solution": "",
  "approval_required": true,
  "status": ""
}
```

## 7. Trạm 4 – Soạn email phản hồi khách hàng

**Mục tiêu:** Tạo bản thảo email dựa trên kết quả đã được kiểm định.

**Prompt 4:**

```text
Bạn là chuyên viên chăm sóc khách hàng FastDelivery.

INPUT:
Nhận dữ liệu khách hàng từ Trạm 1
và phương án xử lý từ Trạm 3.

NHIỆM VỤ:
Soạn email phản hồi khách hàng.

YÊU CẦU:
1. Tiêu đề email rõ ràng.
2. Lời chào lịch sự.
3. Xác nhận đã tiếp nhận khiếu nại.
4. Tóm tắt đúng vấn đề khách hàng gặp phải.
5. Thông báo hướng xử lý dựa trên kết quả đã kiểm định.
6. Hướng dẫn khách hàng cung cấp thêm thông tin nếu cần.
7. Kết thúc bằng lời cảm ơn.

PHONG CÁCH:
Chuyên nghiệp, thân thiện, rõ ràng và chân thành.

RÀNG BUỘC:
- Không tự hứa hẹn bồi thường.
- Không đưa ra thời hạn chưa được xác nhận.
- Không tự khẳng định đã xử lý xong.
- Không thay đổi chính sách đã được kiểm định.
- Nếu chưa được phê duyệt, chỉ tạo bản thảo email.

ĐẦU RA:
{
  "subject": "",
  "email_draft": "",
  "approval_status": "PENDING"
}
```

# IV. BƯỚC 3: THIẾT KẾ CƠ CHẾ XỬ LÝ ĐỨT GÃY NGỮ CẢNH

## 1. Continue Prompting – Tiếp tục khi bị ngắt

Nếu AI đang tạo kết quả nhưng bị ngắt giữa chừng, sử dụng chính xác cụm từ cuối cùng để tiếp tục.

**Prompt khôi phục:**

```text
Bạn đang tiếp tục quy trình xử lý khiếu nại FastDelivery.

Phản hồi trước đó bị ngắt tại cụm:
"validation_status":

Hãy tiếp tục ngay sau cụm từ trên.

YÊU CẦU:
- Không viết lại nội dung đã hoàn thành.
- Giữ nguyên cấu trúc JSON.
- Giữ nguyên dữ liệu đã xử lý.
- Không thay đổi kết quả các trạm trước.
- Kiểm tra lại cú pháp trước khi hoàn thành.
```

## 2. Rolling Summary – Tóm tắt cuộn

Sau khoảng 8–10 lượt tương tác, AI phải tạo bản tóm tắt trạng thái.

**Prompt:**

```text
Hãy tạo bản tóm tắt trạng thái dự án FastDelivery.

Bao gồm:
1. Vai trò và mục tiêu hệ thống.
2. Mã khiếu nại đang xử lý.
3. Dữ liệu đã tiếp nhận.
4. Các trạm đã hoàn thành.
5. Kết quả kiểm định.
6. Các lỗi chưa giải quyết.
7. Trạm tiếp theo cần thực hiện.

YÊU CẦU:
- Ngắn gọn.
- Không bỏ sót dữ liệu quan trọng.
- Không tạo thêm thông tin mới.
- Trình bày theo cấu trúc JSON.
```

## 3. Context Anchoring – Neo bối cảnh

Sử dụng khi AI có dấu hiệu quên vai trò hoặc mục tiêu.

**Prompt:**

```text
[CONTEXT ANCHORING]

Nhắc lại:
Bạn là AI Agent xử lý khiếu nại của FastDelivery.

Mục tiêu:
Phân loại, kiểm tra chính sách, đề xuất xử lý
và tạo bản thảo email.

Quy tắc:
- Sử dụng dữ liệu đã được xác nhận.
- Không tự tạo chính sách.
- Không bỏ qua Validation Gates.
- Không thay đổi kết quả các trạm đã hoàn thành.

Tiếp tục thực hiện từ trạng thái gần nhất.
```

## 4. Session Reset – Làm sạch phiên chat

Khi phiên hội thoại quá dài hoặc xuất hiện hiện tượng Context Drift, sử dụng bản tóm tắt Rolling Summary để bắt đầu một phiên mới.

Không yêu cầu AI xử lý lại từ đầu. Chỉ chuyển dữ liệu trạng thái đã xác nhận sang phiên mới.

# V. BƯỚC 4: XÂY DỰNG CƠ CHẾ KIỂM SOÁT LỖI

## 1. Các trường hợp dữ liệu dị biệt

| Tình huống | Cách xử lý |
|---|---|
| Thiếu mã đơn hàng | NEED_MORE_DATA |
| Thiếu tên khách hàng | NEED_MORE_DATA |
| Sai cấu trúc JSON | Dừng và yêu cầu tạo lại JSON |
| Mã đơn hàng không thống nhất | NEED_VERIFICATION |
| Không tìm thấy chính sách | POLICY_NOT_FOUND |
| Chính sách không đủ căn cứ | NEED_VERIFICATION |
| AI bị ngắt giữa chừng | Continue Prompting |
| Hội thoại quá dài | Rolling Summary / Session Reset |
| Giá trị bồi thường lớn | Human-in-the-Loop |

## 2. Quy tắc kiểm định

Hệ thống chỉ được chuyển sang trạm tiếp theo khi dữ liệu đã vượt qua điều kiện kiểm tra.

Nguyên tắc quan trọng:

- Không để dữ liệu lỗi lan truyền qua các trạm.
- Không để kết quả AI tự động trở thành quyết định nghiệp vụ.
- Không gửi email chính thức khi chưa được phê duyệt.
- Ghi nhận trạng thái xử lý để có thể khôi phục.

## 3. Mã giả điều phối hệ thống

```python
def process_complaint(data):
    try:
        # Trạm 1: Phân loại và trích xuất
        result1 = classify_complaint(data)

        if not validate_stage1(result1):
            return {
                "status": "NEED_VERIFICATION",
                "stage": 1
            }

        # Trạm 2: Tra cứu chính sách
        result2 = lookup_policy(result1)

        if not validate_stage2(result2):
            return {
                "status": "NEED_VERIFICATION",
                "stage": 2
            }

        # Trạm 3: Phân tích và đề xuất
        result3 = analyze_solution(result1, result2)

        if not result3:
            return {
                "status": "PROCESSING_ERROR",
                "stage": 3
            }

        # Trạm 4: Soạn email
        result4 = draft_email(result1, result3)

        return {
            "status": "SUCCESS",
            "complaint": result1,
            "policy": result2,
            "solution": result3,
            "email": result4,
            "approval": "PENDING"
        }

    except Exception as error:
        return {
            "status": "ERROR",
            "message": "Cần kiểm tra dữ liệu hoặc hệ thống."
        }
```

**Giải thích:**

- Mỗi trạm thực hiện một nhiệm vụ riêng biệt.
- Validation được thực hiện trước khi chuyển dữ liệu.
- Nếu phát hiện lỗi, hệ thống dừng đúng tại trạm gặp sự cố.
- Khối `try/except` giúp chương trình xử lý ngoại lệ thay vì bị dừng đột ngột.
- Kết quả cuối cùng được đóng gói theo cấu trúc thống nhất.

*Lưu ý: Đây là mã giả minh họa kiến trúc điều phối. Các hàm xử lý và kiểm định cần được triển khai riêng khi xây dựng hệ thống thực tế.*

# VI. BƯỚC 5: CHẠY THỬ VÀ KIỂM TRA KẾT QUẢ

## 1. Dữ liệu Input thử nghiệm

Tình huống giả định: Khách hàng nhận được sản phẩm bị hư hỏng trong quá trình vận chuyển.

```text
Mã khiếu nại: KN001
Mã đơn hàng: FD202611001
Khách hàng: Nguyễn Văn A
Email: nguyenvana@example.com

Nội dung:
Tôi nhận được đơn hàng nhưng sản phẩm bên trong
bị vỡ khi mở hộp. Tôi muốn được đổi sản phẩm mới.

Thời gian nhận hàng: 17/11/2026

Dữ liệu chính sách thử nghiệm do doanh nghiệp cung cấp:
- Sản phẩm bị hư hỏng được xác nhận đủ điều kiện
  sẽ được áp dụng chính sách đổi 1-1.
- Khách hàng cần cung cấp hình ảnh sản phẩm bị hỏng.
- Nhân viên có trách nhiệm xác minh trước khi phê duyệt.

Yêu cầu:
Phân tích khiếu nại và soạn email phản hồi.
```

*Lưu ý: Chính sách trên là dữ liệu giả định phục vụ chạy thử, không phải chính sách thực tế đã được xác nhận của FastDelivery.*

## 2. Kết quả Trạm 1 – Phân loại

```json
{
  "complaint_id": "KN001",
  "order_id": "FD202611001",
  "customer_name": "Nguyễn Văn A",
  "issue_type": "Hàng hóa bị hư hỏng",
  "priority": "Cao",
  "description": "Khách hàng nhận sản phẩm bị vỡ và yêu cầu đổi sản phẩm mới.",
  "validation_status": "VALID"
}
```

**Đánh giá:** AI đã xác định đúng loại khiếu nại và trích xuất đầy đủ các trường thông tin cần thiết.

## 3. Kết quả Validation Gate 1

```json
{
  "validated": true,
  "errors": [],
  "next_stage": 2,
  "status": "APPROVED"
}
```

**Đánh giá:** Dữ liệu đầy đủ, không phát hiện lỗi cấu trúc hoặc trường thông tin bắt buộc bị trống.

## 4. Kết quả Trạm 2 – Tra cứu chính sách

```json
{
  "order_id": "FD202611001",
  "policy_name": "Chính sách đổi 1-1",
  "policy_condition": "Sản phẩm hư hỏng phải được xác nhận và khách hàng cung cấp hình ảnh.",
  "eligibility": "PENDING_VERIFICATION",
  "evidence": "Khách hàng phản ánh sản phẩm bị vỡ nhưng chưa cung cấp hình ảnh.",
  "validation_status": "VALID"
}
```

**Đánh giá:** AI xác định được chính sách liên quan nhưng chưa tự kết luận khách hàng đủ điều kiện đổi hàng vì còn thiếu bằng chứng.

## 5. Kết quả Validation Gate 2

```json
{
  "policy_found": true,
  "policy_consistent": true,
  "evidence_complete": false,
  "status": "VALID_WITH_PENDING_EVIDENCE",
  "next_stage": 3
}
```

**Đánh giá:** Chính sách có căn cứ từ dữ liệu thử nghiệm. Tuy nhiên, bằng chứng hình ảnh chưa đầy đủ nên phương án xử lý chỉ được đề xuất, chưa được phê duyệt.

## 6. Kết quả Trạm 3 – Phân tích và đề xuất

```json
{
  "order_id": "FD202611001",
  "issue_analysis": "Sản phẩm bị hư hỏng trong quá trình vận chuyển theo phản ánh của khách hàng.",
  "policy_result": "Có chính sách đổi 1-1 nhưng cần hình ảnh xác minh.",
  "proposed_solution": "Yêu cầu khách hàng gửi hình ảnh sản phẩm bị hỏng để nhân viên kiểm tra và xem xét đổi sản phẩm mới.",
  "approval_required": true,
  "status": "WAITING_FOR_EVIDENCE"
}
```

**Đánh giá:** AI đưa ra hướng giải quyết phù hợp với dữ liệu và không tự động phê duyệt đổi hàng.

## 7. Kết quả Trạm 4 – Soạn email

```text
Tiêu đề: Phản hồi khiếu nại đơn hàng FD202611001

Kính gửi anh Nguyễn Văn A,

FastDelivery đã tiếp nhận thông tin phản ánh của anh
về tình trạng sản phẩm bị hư hỏng khi nhận hàng.

Trước tiên, chúng tôi thành thật xin lỗi vì trải nghiệm
chưa tốt mà anh đã gặp phải.

Theo chính sách hỗ trợ khách hàng, sản phẩm bị hư hỏng
sau khi được xác minh đủ điều kiện có thể được áp dụng
chính sách đổi mới 1-1.

Để hỗ trợ anh nhanh chóng hơn, kính mong anh gửi thêm
hình ảnh sản phẩm bị hư hỏng để bộ phận chăm sóc khách
hàng kiểm tra và xác minh thông tin.

Sau khi tiếp nhận đầy đủ thông tin, nhân viên sẽ xem xét
và phản hồi phương án xử lý phù hợp.

Cảm ơn anh đã tin tưởng và sử dụng dịch vụ FastDelivery.

Trân trọng,
Bộ phận Chăm sóc khách hàng
FastDelivery
```

Kết quả đầu ra:

```json
{
  "subject": "Phản hồi khiếu nại đơn hàng FD202611001",
  "email_draft": "Đã tạo bản thảo email phản hồi khách hàng.",
  "approval_status": "PENDING"
}
```

# VII. BƯỚC 6: KIỂM TRA CÁC TÌNH HUỐNG LỖI

## 1. Trường hợp thiếu mã đơn hàng

**Input:**

```text
Khách hàng: Trần Văn B
Email: tranvanb@example.com
Nội dung: Tôi nhận được sản phẩm bị hỏng.
```

**Output:**

```json
{
  "complaint_id": "",
  "order_id": "",
  "customer_name": "Trần Văn B",
  "issue_type": "Hàng hóa bị hư hỏng",
  "priority": "Cao",
  "validation_status": "NEED_MORE_DATA",
  "error": "Thiếu mã đơn hàng."
}
```

**Kết luận:** Hệ thống không chuyển sang Trạm 2 mà yêu cầu bổ sung mã đơn hàng.

## 2. Trường hợp dữ liệu mâu thuẫn

**Input:**

```text
Mã đơn hàng trong tiêu đề: FD001
Mã đơn hàng trong nội dung: FD002
Khách hàng: Lê Văn C
Nội dung: Tôi muốn khiếu nại đơn hàng bị thất lạc.
```

**Output:**

```json
{
  "status": "NEED_VERIFICATION",
  "error": "Mã đơn hàng không thống nhất.",
  "detected_values": ["FD001", "FD002"],
  "next_action": "Yêu cầu nhân viên xác minh mã đơn hàng."
}
```

**Kết luận:** Hệ thống phát hiện mâu thuẫn và không tự ý lựa chọn một mã đơn hàng.

## 3. Trường hợp AI bị ngắt giữa chừng

**Tình huống:** AI hoàn thành Trạm 1 và Trạm 2 nhưng bị ngắt trước khi hoàn thành Trạm 3.

**Bản tóm tắt khôi phục:**

```json
{
  "project": "FastDelivery Complaint System",
  "complaint_id": "KN001",
  "completed_stages": [1, 2],
  "current_stage": 3,
  "customer": "Nguyễn Văn A",
  "order_id": "FD202611001",
  "policy": "Đổi 1-1",
  "evidence": "Chưa có hình ảnh",
  "next_action": "Tiếp tục phân tích phương án xử lý",
  "status": "IN_PROGRESS"
}
```

**Kết luận:** Hệ thống sử dụng bản tóm tắt để tiếp tục từ Trạm 3 mà không phải thực hiện lại toàn bộ quy trình.

# VIII. ĐÁNH GIÁ HIỆU QUẢ GIẢI PHÁP

| Tiêu chí | Kết quả |
|---|---|
| Chain-of-Thought | Phân tích logic theo từng bước |
| Prompt Chaining | Hoàn thành 4 trạm liên kết |
| Data Handoff | Chuyển giao dữ liệu bằng JSON |
| Validation Gates | Phát hiện dữ liệu thiếu và mâu thuẫn |
| Context Anchoring | Duy trì vai trò và mục tiêu |
| Rolling Summary | Tóm tắt trạng thái xử lý |
| Continue Prompting | Khôi phục phản hồi bị ngắt |
| Session Reset | Tiếp tục công việc trong phiên mới |
| Human-in-the-Loop | Chờ nhân viên phê duyệt |

## 1. Ưu điểm

- Phân chia nhiệm vụ rõ ràng, tránh quá tải Mega Prompt.
- Kiểm soát chất lượng tại từng giai đoạn.
- Dữ liệu trung gian được chuẩn hóa.
- Dễ dàng phát hiện và sửa lỗi tại từng trạm.
- Có thể khôi phục khi AI bị gián đoạn.
- Hạn chế AI tự tạo thông tin hoặc chính sách không có căn cứ.

## 2. Hạn chế

- Phải thực hiện nhiều lần gọi AI nên độ trễ có thể tăng.
- Tốn thêm token cho các lần chuyển giao dữ liệu.
- Cần xây dựng chương trình điều phối và quản lý trạng thái.
- Cần nhân viên kiểm duyệt các trường hợp quan trọng.
- Kết quả vẫn phụ thuộc vào chất lượng dữ liệu đầu vào và nguồn chính sách.

## 3. Hướng phát triển

Trong tương lai, hệ thống có thể tích hợp:
- Database để tra cứu thông tin đơn hàng.
- API để kiểm tra trạng thái vận chuyển.
- Python để tự động kiểm định dữ liệu.
- Model Routing để tối ưu chi phí.
- Giao diện quản lý khiếu nại cho nhân viên.
- Hệ thống lưu trạng thái để khôi phục các quy trình đang xử lý.

# IX. KẾT LUẬN

Thông qua bài tập thực hành tổng hợp, em đã vận dụng và kết hợp các kiến thức trọng tâm của Session 04 để thiết kế một giải pháp AI Agent xử lý khiếu nại khách hàng trong doanh nghiệp logistics.

Trong đó, Chain-of-Thought giúp AI phân tích các vấn đề có nhiều điều kiện theo trình tự logic. Prompt Chaining giúp chia nhỏ nhiệm vụ thành nhiều trạm xử lý độc lập và chuyển giao dữ liệu thông qua Data Handoff. Các kỹ thuật Continue Prompting, Rolling Summary, Context Anchoring và Session Reset giúp duy trì tính nhất quán và khôi phục công việc khi hội thoại bị gián đoạn.

Bên cạnh đó, việc bổ sung Validation Gates, xử lý ngoại lệ và Human-in-the-Loop giúp hạn chế lỗi dữ liệu lan truyền và tăng khả năng kiểm soát của con người.

Qua bài thực hành, em nhận thấy để xây dựng một hệ thống AI Agent hiệu quả, không chỉ cần viết Prompt rõ ràng mà còn phải biết phân tích bài toán, thiết kế quy trình, kiểm soát dữ liệu và dự đoán các tình huống phát sinh.

**Kết luận:** Sự kết hợp giữa lập luận logic, chuỗi Prompt liên kết và quản lý ngữ cảnh là nền tảng quan trọng để xây dựng các hệ thống AI Agent có khả năng giải quyết những công việc thực tế một cách có hệ thống, ổn định và hiệu quả.
