# 🚀 .NET Web API + Docker + GitHub Actions CI/CD

Triển khai ứng dụng .NET Web API lên VPS Ubuntu 24.04 bằng Docker, Docker Compose và GitHub Actions.  
Hỗ trợ domain và SSL miễn phí với Cloudflare.

---

## 🧱 Công nghệ sử dụng

- [.NET 8 Web API](https://learn.microsoft.com/en-us/aspnet/core/web-api)
- Docker + Docker Compose
- GitHub Actions (CI/CD)
- VPS Ubuntu 24.04 LTS
- SSH key-based deploy
- Cloudflare (quản lý domain + HTTPS SSL miễn phí)

---

## ⚙️ Bước 1: Tạo Project .NET Web API

```bash
dotnet new webapi -n MyApi
cd MyApi
```

---

## 🐳 Bước 2: Tạo `Dockerfile`

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApi/MyApi.csproj", "MyApi/"]
RUN dotnet restore "MyApi/MyApi.csproj"
COPY . .
WORKDIR "/src/MyApi"
RUN dotnet build "MyApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

---

## 📦 Bước 3: Tạo `docker-compose.yml`

```yaml
version: '3.8'
services:
  webapi:
    build: .
    ports:
      - "5000:80"
```

---

## 📡 Bước 4: Cài Docker & Docker Compose trên VPS

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)"   -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

docker --version
docker-compose --version
```

---

## 🔐 Bước 5: Thiết lập SSH Key để deploy

Trên máy local:

```bash
ssh-keygen -t ed25519 -C "github-deploy"
```

Copy public key lên VPS:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@<VPS_IP> -p 24700
```

---

## 🛠️ Bước 6: Cấu hình GitHub Actions

Tạo file `.github/workflows/deploy.yml`:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy over SSH
        run: |
          ssh -o StrictHostKeyChecking=no -p 24700 ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} << 'EOF'
            cd /home/code/dotnet-api-docker/MyApi
            git pull origin main
            docker-compose down
            docker-compose up -d --build
          EOF
```

---

## 🔑 Bước 7: Thêm GitHub Secrets

Vào repo GitHub → Settings → Secrets → Actions → **New Repository Secret**

| Key             | Value                                      |
|----------------|--------------------------------------------|
| SSH_PRIVATE_KEY | Nội dung file `~/.ssh/id_ed25519`         |
| VPS_USER        | `root` hoặc user VPS khác                 |
| VPS_HOST        | Địa chỉ IP của VPS                        |

---

## 🌐 Bước 8: Trỏ domain và cấu hình SSL Cloudflare

1. Tạo domain miễn phí tại [freenom.com](https://www.freenom.com/)
2. Trỏ DNS về IP VPS qua Cloudflare
3. Bật chế độ **Full SSL** trong Cloudflare
4. Cài **Cloudflare Origin Certificate** (nếu muốn dùng SSL nâng cao)

---

## ✅ Truy cập API

Truy cập qua IP:

```bash
http://<your-vps-ip>:5000/weatherforecast
```

Hoặc qua domain:

```bash
https://yourdomain.com/weatherforecast
```
