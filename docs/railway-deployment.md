# Triển khai lên Railway

## Kiến trúc trên Railway

| Service | Railway addon |
|---|---|
| App (Node.js) | Web service (Docker) |
| PostgreSQL | Railway PostgreSQL plugin |
| Redis | Railway Redis plugin |
| Object storage | Cloudflare R2 (hoặc MinIO tự host) |

## Các bước triển khai

### 1. Tạo project Railway mới

```bash
# Cài Railway CLI
npm install -g @railway/cli
railway login
railway init
```

### 2. Thêm PostgreSQL và Redis

Trên Railway dashboard:
- **New** → **Database** → **PostgreSQL** → tạo trong cùng project
- **New** → **Database** → **Redis** → tạo trong cùng project

Railway tự động inject `DATABASE_URL` và `REDIS_URL` vào app service.

### 3. Cấu hình Object Storage (Cloudflare R2)

Tạo R2 bucket tại [Cloudflare Dashboard](https://dash.cloudflare.com), sau đó set các biến:

```
S3_ENDPOINT=https://<account_id>.r2.cloudflarestorage.com
S3_PUBLIC_URL=https://<custom-domain-hoặc-public-url>
S3_BUCKET=zalocrm-attachments
S3_REGION=auto
S3_ACCESS_KEY=<r2-access-key-id>
S3_SECRET_KEY=<r2-secret-access-key>
```

### 4. Biến môi trường bắt buộc trên Railway

Vào **App service** → **Variables**, thêm:

```
# Bảo mật (tạo bằng: openssl rand -hex 32)
JWT_SECRET=<random-32-chars>
ENCRYPTION_KEY=<random-32-bytes-hex>

# App
NODE_ENV=production
PORT=3000
APP_URL=https://<your-railway-domain>.railway.app

# Database — Railway tự inject DATABASE_URL
# Redis — Railway tự inject REDIS_URL

# Object Storage (xem bước 3)
S3_ENDPOINT=
S3_PUBLIC_URL=
S3_BUCKET=zalocrm-attachments
S3_REGION=auto
S3_ACCESS_KEY=
S3_SECRET_KEY=

# AI (tùy chọn)
ANTHROPIC_AUTH_TOKEN=
AI_DEFAULT_PROVIDER=anthropic
AI_DEFAULT_MODEL=claude-sonnet-4.6

# Facebook (nếu dùng)
FB_APP_ID=
FB_APP_SECRET=
FB_WEBHOOK_VERIFY_TOKEN=
FB_TOKEN_ENC_KEY=<random-32-bytes-hex>
FB_OAUTH_REDIRECT_URI=https://<your-domain>/api/v1/integrations/facebook/oauth/callback
```

### 5. Deploy

```bash
railway up
```

Hoặc kết nối GitHub repo để tự động deploy khi push.

## Auto-deploy từ GitHub

1. Railway dashboard → project → **Settings** → **Source** → kết nối GitHub repo
2. Chọn branch `main` (hoặc branch muốn deploy)
3. Railway sẽ tự build và deploy mỗi khi có commit mới

## Lưu ý

- Railway tự cấp domain dạng `<name>.railway.app`, cập nhật `APP_URL` tương ứng
- File upload lưu local (`/var/lib/zalo-crm/files`) sẽ **mất khi restart** trên Railway — nên dùng S3/R2 cho production
- Prisma migration (`db push`) chạy tự động khi app khởi động
