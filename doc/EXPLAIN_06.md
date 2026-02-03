# Giải Thích Chi Tiết: 06-artifacts.yml

## 📋 Tổng Quan

File `06-artifacts.yml` minh họa cách sử dụng **Artifacts** trong GitHub Actions - một tính năng quan trọng cho phép **lưu trữ và chia sẻ dữ liệu** giữa các jobs trong cùng một workflow hoặc lưu lại kết quả build để tải về sau.

---

## 🔍 Phân Tích Từng Phần

### 1. Tên Workflow

```yaml
name: 06 - Artifacts Demo
```

**Giải thích:**
- Đặt tên mô tả cho workflow để dễ nhận biết trong GitHub Actions UI

---

### 2. Trigger (Kích hoạt)

```yaml
on:
    workflow_dispatch:
```

**Giải thích:**
- `workflow_dispatch`: Cho phép kích hoạt workflow **thủ công** từ giao diện GitHub

**Tại sao dùng `workflow_dispatch`?**
- Phù hợp cho demo/testing vì bạn có thể chạy bất cứ lúc nào
- Không cần push code hay tạo PR để test

**Các lựa chọn thay thế:**
```yaml
# Kích hoạt khi push
on:
  push:
    branches: [main]

# Kích hoạt khi có PR
on:
  pull_request:
    branches: [main]

# Kích hoạt theo lịch (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Chạy lúc 00:00 UTC hàng ngày
```

---

### 3. Job 1: `create-artifact`

```yaml
jobs:
    create-artifact:
        runs-on: ubuntu-latest
```

**Giải thích:**
- `create-artifact`: Tên job đầu tiên - tạo và upload artifact
- `runs-on: ubuntu-latest`: Chạy trên máy ảo Ubuntu mới nhất

**Tại sao dùng `ubuntu-latest`?**
- Miễn phí nhiều phút chạy nhất (2,000 phút/tháng cho free tier)
- Hỗ trợ đầy đủ các công cụ phát triển
- Boot nhanh hơn Windows/macOS

**Các lựa chọn thay thế:**
```yaml
runs-on: ubuntu-22.04      # Version cụ thể
runs-on: ubuntu-20.04      # Version cũ hơn
runs-on: windows-latest    # Windows Server
runs-on: macos-latest      # macOS
runs-on: self-hosted       # Runner tự host
```

---

### 4. Step 1: Tạo Files

```yaml
steps:
    - name: 📝 Create files
      run: |
          mkdir -p output
          echo "Build output content" > output/build.txt
          echo '{"version": "1.0.0"}' > output/metadata.json
          date > output/timestamp.txt
```

**Phân tích từng lệnh:**

| Lệnh | Giải thích | Tại sao dùng? |
|------|------------|---------------|
| `mkdir -p output` | Tạo thư mục `output`, `-p` = không lỗi nếu đã tồn tại | `-p` an toàn hơn `mkdir` đơn thuần |
| `echo "..." > file.txt` | Ghi nội dung vào file (ghi đè) | Cách đơn giản nhất để tạo file với nội dung |
| `echo '{...}' > file.json` | Tạo file JSON | Dùng single quote `'...'` để tránh bash xử lý ký tự đặc biệt |
| `date > file.txt` | Ghi thời gian hiện tại vào file | Hữu ích để tracking thời điểm build |

**Tại sao dùng `>`?**
- `>` : Ghi đè file (overwrite)
- `>>` : Thêm vào cuối file (append)

**Các lựa chọn thay thế:**
```bash
# Sử dụng printf (có định dạng tốt hơn)
printf "Build version: %s\n" "1.0.0" > output/build.txt

# Sử dụng cat với heredoc (cho nội dung nhiều dòng)
cat << EOF > output/metadata.json
{
    "version": "1.0.0",
    "build_number": "${GITHUB_RUN_NUMBER}"
}
EOF

# Sử dụng tee (vừa ghi file vừa hiển thị output)
echo "Build content" | tee output/build.txt
```

---

### 5. Step 2: Upload Artifact

```yaml
- name: 📤 Upload artifact
  uses: actions/upload-artifact@v4
  with:
      name: build-output
      path: output/
      retention-days: 5
```

**Giải thích các tham số:**

| Tham số | Giá trị | Giải thích |
|---------|---------|------------|
| `uses` | `actions/upload-artifact@v4` | Action chính thức của GitHub để upload artifact |
| `name` | `build-output` | Tên định danh cho artifact (dùng để download sau) |
| `path` | `output/` | Đường dẫn đến file/thư mục cần upload |
| `retention-days` | `5` | Số ngày lưu trữ artifact (mặc định 90 ngày) |

**Tại sao dùng `actions/upload-artifact@v4`?**
- Action chính thức, được GitHub duy trì
- Hỗ trợ tốt cho việc upload file lớn
- Tự động nén files để tiết kiệm storage
- v4 là version mới nhất với nhiều cải tiến

**Các tham số nâng cao khác:**

```yaml
- uses: actions/upload-artifact@v4
  with:
      name: my-artifact
      path: |
          output/
          dist/
          !dist/**/*.tmp
      retention-days: 5
      if-no-files-found: error  # 'warn' hoặc 'ignore'
      compression-level: 6       # 0-9, mặc định 6
      overwrite: true           # Ghi đè artifact cùng tên
```

**Giải thích `if-no-files-found`:**
- `error` (mặc định): Workflow fail nếu không tìm thấy file
- `warn`: Cảnh báo nhưng không fail
- `ignore`: Bỏ qua, không thông báo

---

### 6. Job 2: `use-artifact`

```yaml
use-artifact:
    needs: create-artifact
    runs-on: ubuntu-latest
```

**Giải thích:**
- `needs: create-artifact`: Đảm bảo job này chỉ chạy **sau khi** `create-artifact` hoàn thành thành công
- Đây là cách tạo **dependency** giữa các jobs

**Tại sao cần `needs`?**
- Mặc định, các jobs chạy **song song** (parallel)
- `needs` tạo thứ tự tuần tự (sequential)
- Artifact chỉ có thể download sau khi đã được upload

**Các cách sử dụng `needs`:**

```yaml
# Phụ thuộc một job
needs: create-artifact

# Phụ thuộc nhiều jobs
needs: [build, test]

# Phụ thuộc theo chuỗi
job-a:
  runs-on: ubuntu-latest
job-b:
  needs: job-a
job-c:
  needs: [job-a, job-b]
```

---

### 7. Step 3: Download Artifact

```yaml
steps:
    - name: 📥 Download artifact
      uses: actions/download-artifact@v4
      with:
          name: build-output
          path: downloaded/
```

**Giải thích các tham số:**

| Tham số | Giá trị | Giải thích |
|---------|---------|------------|
| `uses` | `actions/download-artifact@v4` | Action chính thức để download artifact |
| `name` | `build-output` | Tên artifact cần download (phải khớp với upload) |
| `path` | `downloaded/` | Thư mục đích để giải nén artifact |

**Các tham số nâng cao:**

```yaml
- uses: actions/download-artifact@v4
  with:
      name: build-output
      path: downloaded/
      merge-multiple: true    # Gộp nhiều artifacts vào một thư mục
      
# Download tất cả artifacts
- uses: actions/download-artifact@v4
  # Không chỉ định 'name' sẽ download tất cả
  with:
      path: all-artifacts/
```

---

### 8. Step 4: Hiển Thị Nội Dung

```yaml
- name: 📋 Show contents
  run: |
      ls -la downloaded/
      cat downloaded/build.txt
      cat downloaded/metadata.json
```

**Phân tích từng lệnh:**

| Lệnh | Giải thích |
|------|------------|
| `ls -la downloaded/` | Liệt kê chi tiết tất cả files (bao gồm hidden files) |
| `cat downloaded/build.txt` | Hiển thị nội dung file text |
| `cat downloaded/metadata.json` | Hiển thị nội dung file JSON |

**Các options của `ls`:**
- `-l`: Long format (chi tiết)
- `-a`: Hiện tất cả files (kể cả hidden)
- `-h`: Human-readable sizes (KB, MB, GB)
- `-R`: Recursive (liệt kê cả subfolders)

**Các lựa chọn thay thế cho `cat`:**

```bash
# Hiển thị với line numbers
cat -n downloaded/build.txt

# Sử dụng less cho file lớn
less downloaded/build.txt

# Sử dụng jq cho JSON (format đẹp hơn)
jq '.' downloaded/metadata.json

# Chỉ xem phần đầu file
head -n 10 downloaded/build.txt

# Chỉ xem phần cuối file
tail -n 10 downloaded/build.txt
```

---

## 🔄 Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    WORKFLOW: 06 - Artifacts Demo            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────┐                                    │
│  │  Job: create-artifact │                                  │
│  │  (ubuntu-latest)      │                                  │
│  ├─────────────────────┤                                    │
│  │ 1. 📝 Create files   │                                   │
│  │    - mkdir output    │                                   │
│  │    - echo > files    │                                   │
│  ├─────────────────────┤                                    │
│  │ 2. 📤 Upload artifact│                                   │
│  │    - name: build-output                                  │
│  │    - retention: 5 days                                   │
│  └──────────┬──────────┘                                    │
│             │                                               │
│             │ needs (chờ hoàn thành)                        │
│             ▼                                               │
│  ┌─────────────────────┐                                    │
│  │  Job: use-artifact    │                                  │
│  │  (ubuntu-latest)      │                                  │
│  ├─────────────────────┤                                    │
│  │ 1. 📥 Download artifact│                                 │
│  │    - name: build-output                                  │
│  ├─────────────────────┤                                    │
│  │ 2. 📋 Show contents  │                                   │
│  │    - ls, cat files   │                                   │
│  └─────────────────────┘                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 Các Trường Hợp Sử Dụng Artifacts Thực Tế

### 1. Build và Deploy

```yaml
jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - run: npm ci && npm run build
            - uses: actions/upload-artifact@v4
              with:
                  name: dist
                  path: dist/
    
    deploy:
        needs: build
        runs-on: ubuntu-latest
        steps:
            - uses: actions/download-artifact@v4
              with:
                  name: dist
            - run: # Deploy to server
```

### 2. Test Reports

```yaml
- name: Run tests
  run: npm test -- --coverage

- uses: actions/upload-artifact@v4
  with:
      name: coverage-report
      path: coverage/
      retention-days: 30
```

### 3. Build Artifacts cho Multiple Platforms

```yaml
jobs:
    build:
        strategy:
            matrix:
                os: [ubuntu-latest, windows-latest, macos-latest]
        runs-on: ${{ matrix.os }}
        steps:
            - run: # Build command
            - uses: actions/upload-artifact@v4
              with:
                  name: build-${{ matrix.os }}
                  path: dist/
```

---

## ⚠️ Lưu Ý Quan Trọng

### 1. Giới Hạn Storage

| Plan | Storage Limit | Lưu ý |
|------|---------------|-------|
| Free | 500 MB | Cần quản lý retention-days |
| Pro | 1 GB | |
| Team | 2 GB | |
| Enterprise | 50 GB | |

### 2. Retention (Thời gian lưu trữ)

```yaml
# Mặc định: 90 ngày
# Tối thiểu: 1 ngày
# Tối đa: 90 ngày (400 cho Enterprise)
retention-days: 5
```

### 3. Artifacts vs Cache

| Tính năng | Artifacts | Cache |
|-----------|-----------|-------|
| Mục đích | Lưu kết quả build | Tăng tốc workflow |
| Chia sẻ giữa jobs | ✅ Có | ❌ Không trực tiếp |
| Download được | ✅ Có (UI hoặc API) | ❌ Không |
| Thời gian lưu | Cấu hình được | 7 ngày |

```yaml
# Cache example
- uses: actions/cache@v4
  with:
      path: ~/.npm
      key: npm-${{ hashFiles('**/package-lock.json') }}
```

### 4. Security

- Artifacts có thể chứa sensitive data
- KHÔNG nên upload secrets, passwords, tokens
- Kiểm tra kỹ nội dung trước khi public repository

---

## 🎯 Best Practices

1. **Đặt tên artifact rõ ràng**: Dùng tên mô tả như `build-output`, `test-reports`, không dùng tên chung chung

2. **Set retention-days hợp lý**: Tránh lưu quá lâu để tiết kiệm storage

3. **Sử dụng `if-no-files-found`**: Để kiểm soát behavior khi không có files

4. **Compress artifacts lớn**: Action tự động compress nhưng bạn có thể optimize thêm

5. **Cleanup sau khi dùng**: 
```yaml
- uses: geekyeggo/delete-artifact@v2
  with:
      name: temporary-artifact
```

---

## 📚 Tài Liệu Tham Khảo

- [GitHub Docs - Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/download-artifact](https://github.com/actions/download-artifact)
