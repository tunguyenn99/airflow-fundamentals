# 🚀 Chạy Apache Airflow 3.0.2 bằng Docker Compose

Hướng dẫn khởi chạy nhanh Airflow sử dụng **CeleryExecutor** với **Docker Compose**. Phù hợp cho mục đích học tập, thử nghiệm và phát triển cá nhân.

> ⚠️ **Lưu ý:** Đây **không phải** là cấu hình production-ready. Để triển khai thực tế, bạn nên sử dụng [Helm Chart chính thức của Airflow](https://airflow.apache.org/docs/helm-chart/stable/index.html) với Kubernetes.

---

## 📦 Yêu cầu trước khi bắt đầu

- Docker CE đã cài đặt
- Docker Compose v2.14.0 hoặc mới hơn
- Tối thiểu **4GB RAM** cấp cho Docker (khuyến nghị 8GB)

---

## 🛠️ Các bước thiết lập

### 1. Tải file docker-compose.yaml chính thức

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.0.2/docker-compose.yaml'
```

---

### 2. Tạo thư mục & cấu hình quyền truy cập

```bash
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

> Trên macOS/Windows, bạn có thể dùng `AIRFLOW_UID=50000`

---

### 3. (Tuỳ chọn) Khởi tạo file `airflow.cfg`

```bash
docker compose run airflow-cli airflow config list
```

---

### 4. Khởi tạo database và tài khoản quản trị

```bash
docker compose up airflow-init
```

> ✅ Tài khoản mặc định:
> - Username: `airflow`
> - Password: `airflow`

---

### 5. Khởi động toàn bộ dịch vụ

```bash
docker compose up --build
```

- Truy cập giao diện: http://localhost:8080  
- Flower (nếu bật): http://localhost:5555

---

## 📁 Cấu trúc thư mục

| Thư mục      | Chức năng                                                   |
|--------------|-------------------------------------------------------------|
| `./dags`     | Nơi bạn lưu các DAG Python                                  |
| `./logs`     | Lưu log của task & scheduler                                |
| `./plugins`  | Plugin tùy chỉnh nếu có                                     |
| `./config`   | Gồm file cấu hình `airflow.cfg`, hoặc airflow_local_settings |

---

## 📦 Cài thêm thư viện Python (ví dụ SQL Server)

Tạo file `requirements.txt` như sau:

```txt
apache-airflow-providers-microsoft-mssql
pyodbc
```

Tạo `Dockerfile` tương ứng:

```dockerfile
FROM apache/airflow:3.0.2
USER root

RUN apt-get update && apt-get install -y \
    curl gnupg apt-transport-https unixodbc-dev && \
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && \
    curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list && \
    apt-get update && ACCEPT_EULA=Y apt-get install -y msodbcsql17 && \
    apt-get clean

USER airflow
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

Sau đó build lại bằng:

```bash
docker compose up --build
```

---

## 🔄 Reset môi trường (nếu cần xoá sạch và làm lại)

```bash
docker compose down --volumes --remove-orphans
rm -rf dags logs plugins config .env
```

---

## 📌 Ghi chú

- Airflow chạy với CeleryExecutor kèm PostgreSQL + Redis
- Flower là công cụ giám sát worker (tuỳ chọn, bật bằng `--profile flower`)
- Môi trường chỉ phục vụ học tập – KHÔNG dùng cho production