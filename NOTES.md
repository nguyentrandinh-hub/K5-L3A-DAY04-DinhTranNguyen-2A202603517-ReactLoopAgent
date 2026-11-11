# Ghi chú phân tích · Day 04

Họ và tên: Đinh Trần Nguyên
MSSV: 2A202603517

## 1. Case được chọn

- ID case: H1
- Câu trả lời của agent: Agent so sánh hai incident, nhưng dùng mốc `updated_at` của GitHub làm thời điểm kết thúc.
- Điểm ở lần chạy 1: 60/100.

## 2. Bằng chứng trong trace

Dẫn chứng: Kết quả GitHub dẫn tới `xr3myzj7ms51`, nhưng trace lần đầu không có lần gọi tool đọc incident này. Hai calls cùng ở step 1, trong khi mechanical check yêu cầu ít nhất hai tool steps. Dữ liệu GitHub có update `resolved` lúc 04/11/2026 18:16 UTC; `updated_at` ngày 07/11 không đủ căn cứ để coi là thời gian kết thúc.

Trích trace rút gọn, chỉ giữ các trường nhận diện call:

```json
{"event":"tool","step":1,"name":"get_incident_details","arguments":{"provider":"github","incident_id":"h02n9572mx9y"}}
{"event":"tool","step":1,"name":"get_incident_details","arguments":{"provider":"cloudflare","incident_id":"yaa0witbl2me"}}
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Trong `agent.py`, bổ sung kiểm tra incident liên kết khi câu hỏi yêu cầu theo liên kết; kiểm tra URL nguồn và yêu cầu giữ chi tiết khi sửa câu trả lời.
- Vì sao chọn thay đổi này: H1 bỏ qua incident summary được dẫn tới; kiểm tra URL nguồn cũng là contract của Bài 4.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 83.5 | 82.95 |
| Điểm case H1 | 60 | 75 |
| Model calls — agent / judge | 50 / 22 | 54 / 22 |
| Token — agent + judge | 82.545 | 100.781 |
| Chi phí — agent + judge, USD | 0.01789065 | 0.02058180 |

- Lỗi đã giải quyết: Trong lần chạy sau, H1 đọc incident được liên kết; có 3 tool calls qua 2 tool steps và không vi phạm mechanical check.
- Lỗi còn lại: Câu trả lời H1 vẫn diễn giải `updated_at` như thời gian kết thúc và chưa nêu nội dung incident liên kết bị cắt. Điểm tổng giảm 0.55 điểm.

## 5. Giới hạn

Judge dùng cùng model với agent, điểm là ước lượng và có thể thiên lệch. Snapshot lưu ngày 09/11/2026, không phản ánh trạng thái live. Bảng so sánh một lượt eval đầy đủ cho mỗi phiên bản; các lượt thử H1 và eval trung gian không chứng minh độ ổn định.
