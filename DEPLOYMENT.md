# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Đức Long |
| Mã học viên | 2A202602917 |
| Repo | https://github.com/duclongt23/K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-axee.onrender.com |
| Platform | Render |
| Ngày deploy | 29/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
(.venv) D:\AI_thuc_chien\lab12\K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment>curl -i https://day12-agent-axee.onrender.com/health
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:53:26 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: c5d4a904-8b9f-4222
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a4286255dfdac8b5-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}
(.venv) D:\AI_thuc_chien\lab12\K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment>curl -i https://day12-agent-axee.onrender.com/ready
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:53:29 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: a22a47b7-7046-461e
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42862685ed1a39b-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}
(.venv) D:\AI_thuc_chien\lab12\K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment>curl -i -X POST https://day12-agent-axee.onrender.com/ask -H "Content-Type: application/json" -d "{\"question\":\"Hello\"}"
HTTP/1.1 401 Unauthorized
Date: Tue, 29 Sep 2026 04:53:34 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 933a0029-de8b-4e1e
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42862862f6b9f8b-SIN
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}
(.venv) D:\AI_thuc_chien\lab12\K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment>curl -i -X POST https://day12-agent-axee.onrender.com/ask -H "Content-Type: application/json" -H "X-API-Key: tvttcute" -H "X-User-Id: sv-test" -d "{\"question\":\"Deploy là gì?\"}"
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:53:41 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: cc9abf06-f615-49ef
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42862b43aa58e13-SIN
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 6 lượt trao đổi trước đó.)","user_id":"sv-test","history_length":6,"cost_usd":4.785e-05,"tokens":{"in":139,"out":45}}
(.venv) D:\AI_thuc_chien\lab12\K4-L3B-DAY12-NguyenDucLong-2A202602917-CloudServicesAndDeployment>for /L %i in (1,1,15) do @curl -s -o NUL -w "%{http_code} " -X POST https://day12-agent-axee.onrender.com/ask -H "Content-Type: application/json" -H "X-API-Key: tvttcute" -H "X-User-Id: sv-test" -d "{\"question\":\"test\"}"
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl