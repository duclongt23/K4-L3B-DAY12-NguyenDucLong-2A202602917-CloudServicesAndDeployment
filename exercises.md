# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng [Câu trả lời của bạn] bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Đức Long  Mã học viên: 2A202602917

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Tình huống: Khi deploy app lên Cloud nhưng quên set biến `AGENT_API_KEY`. Nếu có mặc định `"changeme"`, app vẫn khởi động bình thường và kẻ xấu có thể dùng key `"changeme"` để gọi API làm tiêu tốn toàn bộ tiền LLM của bạn mà bạn không hay biết. Việc "chết sớm" khiến container crash ngay lập tức, giúp ta phát hiện và bổ sung secret ngay trước khi nhận traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Dòng log thu được: `{"event": "service_started", "level": "info", "timestamp": "2026-09-29T04:22:38.822702+00:00", "service": "day12-agent", "version": "1.0.0"}`

Dòng log dạng JSON chứa dữ liệu có cấu trúc bao gồm các trường chi tiết như event, level và timestamp, giúp các hệ thống quản lý log (như ELK, Datadog) tự động phân tích để thực hiện hai việc mà lệnh print thông thường không thể làm được: truy vấn, lọc và phân loại dữ liệu tự động dựa trên các trường thông tin (ví dụ: lọc theo level hay service), đồng thời giám sát và cảnh báo theo mốc thời gian chuẩn (timestamp) để dựng timeline sự cố hoặc đo lường hiệu năng hệ thống.


---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.6 GB (1600 MB) |
| Multi-stage | 272 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Dung lượng chênh lệch (~1.3 GB) bao gồm hệ điều hành Linux đầy đủ, C/C++ compiler, build tools, pip cache và các file rác không cần thiết cho môi trường chạy ứng dụng (runtime).



---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

- Các layer được dùng lại từ cache: `FROM`, `WORKDIR`, `COPY requirements.txt`, `RUN pip install`. Layer phải chạy lại từ đầu: `COPY . .` và các lệnh phía sau.
- Nếu đặt `COPY . .` lên trước `RUN pip install`: Mỗi lần sửa 1 dòng code, Docker không tận dụng được cache của bước `COPY . .` trở đi, khiến bước `RUN pip install` phải tải và cài lại toàn bộ thư viện từ đầu, làm quá trình build cực kỳ chậm.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

- Chuỗi sự kiện: Lỗi RCE trong code Python -> Kẻ tấn công thực thi lệnh hệ thống bên trong container với quyền `root` -> Khai thác lỗ hổng container escape để thoát ra máy host -> Do container chạy bằng `root`, kẻ tấn công nghiễm nhiên chiếm luôn quyền `root` trên máy host.
- Lệnh `USER appuser` cắt đứt ở bước cuối: Dù kẻ tấn công chiếm được container, chúng chỉ có quyền của user thường (`appuser`), không thể gây hại cho máy host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

- Tối đa: 20 request trong 2 giây liên tiếp.
- Giải thích: User gửi 10 request ở giây `10:00:59` (cuối phút thứ nhất) và gửi tiếp 10 request nữa ở giây `10:01:01` (đầu phút thứ hai). Vì đếm theo phút đồng hồ reset lúc 00s, cả 2 đợt đều hợp lệ -> Tổng cộng 20 request trong 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

- Khác nhau: Rate limit giới hạn *số lượng request/thời gian*. Cost guard giới hạn *tổng số tiền tiêu tốn (USD)/tháng*.
- Rate limit cho qua nhưng Cost guard chặn: User gửi 1 request/phút (đúng hạn mức rate limit), nhưng prompt quá dài ngốn 100k token tiêu hết ngân sách tháng -> Cost guard chặn (402).
- Cost guard cho qua nhưng Rate limit chặn: User mới tiêu 0.1$ (còn nhiều ngân sách), nhưng gửi liên tục 20 request trong 5 giây -> Rate limit chặn (429).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

1. `/health` thất bại trả về 503 khi Redis ngắt kết nối.
2. Orchestrator (Docker/Kubernetes) thấy `/health` lỗi nên tưởng container bị treo, lập tức kill và restart lại cả 3 container `agent`.
3. Cả cụm 3 container liên tục bị kill & restart liên tục trong 30s, làm sập hoàn toàn hệ thống ngay cả khi ứng dụng Python vẫn chạy bình thường.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Giá trị `history_length` sẽ nhảy thất thường không theo thứ tự (ví dụ: request 1 vào agent-1 trả về 1, request 2 vào agent-2 trả về 0, request 3 vào agent-1 trả về 2). Agent bị "mất trí nhớ" luân phiên do mỗi container giữ một RAM riêng.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

- Thông báo lỗi: `NotImplementedError: TODO (CP4): cài đặt install` khi ứng dụng khởi động trên Render.
- Nguyên nhân: Chưa `git push` code mới từ máy cục bộ lên GitHub, khiến Render pull bản code cũ về build.
- Cách sửa: Chạy `git add .`, `git commit -m "update code"`, `git push origin main` lên GitHub rồi bấm Manual Deploy lại trên Render.

