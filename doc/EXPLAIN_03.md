# 📚 Giải Thích Chi Tiết: 03-jobs.yml

## 🎯 Tổng Quan

File `03-jobs.yml` là một workflow demo về cách sử dụng **Jobs** trong GitHub Actions. Nó minh họa các khái niệm quan trọng:

- **Jobs chạy song song (Parallel Jobs)**
- **Jobs phụ thuộc (Dependencies với `needs`)**
- **Chạy trên nhiều hệ điều hành khác nhau**

---

## 📋 Phân Tích Từng Phần

### 1. Tên Workflow

```yaml
name: 03 - Jobs Demo
```

**Giải thích:**
- `name` định nghĩa tên hiển thị của workflow trên tab Actions của GitHub
- Tên này giúp bạn dễ dàng nhận diện workflow khi có nhiều workflows

**Tại sao dùng cách đặt tên này?**
- Prefix số `03 -` giúp sắp xếp thứ tự các workflow demo
- Tên ngắn gọn, rõ ràng mục đích

---

### 2. Trigger Event

```yaml
on:
  workflow_dispatch:
```

**Giải thích:**
- `on` xác định khi nào workflow được kích hoạt
- `workflow_dispatch` cho phép chạy workflow **thủ công** từ giao diện GitHub

**Tại sao dùng `workflow_dispatch`?**
- Phù hợp cho demo/testing vì bạn có thể chạy bất cứ lúc nào
- Không phụ thuộc vào push, pull request, hay schedule

**Các trigger alternatives khác:**

| Trigger | Mô tả | Khi nào dùng |
|---------|-------|--------------|
| `push` | Khi có commit mới | CI tự động khi push code |
| `pull_request` | Khi có PR | Review và test PR |
| `schedule` | Chạy theo lịch (cron) | Nightly builds, cleanup |
| `workflow_call` | Gọi từ workflow khác | Reusable workflows |
| `repository_dispatch` | Webhook từ bên ngoài | Trigger từ hệ thống external |

---

### 3. Jobs Block

```yaml
jobs:
```

**Giải thích:**
- `jobs` là block chứa tất cả các jobs trong workflow
- Mỗi workflow phải có ít nhất một job
- **Mặc định, tất cả jobs chạy SONG SONG (parallel)**

---

## 🅰️ Job A - Job Đơn Giản

```yaml
job-a:
  name: 🅰️ Job A
  runs-on: ubuntu-latest
  steps:
    - run: |
        echo "Job A starting..."
        sleep 5
        echo "Job A done!"
```

### Phân tích từng lệnh:

#### `job-a:`
- **Job ID** - định danh duy nhất của job
- Dùng để tham chiếu trong `needs`, outputs, v.v.
- **Quy tắc đặt tên:** chữ thường, số, dấu gạch ngang (`-`), gạch dưới (`_`)

**Tại sao dùng `job-a` thay vì `jobA` hay `Job-A`?**
- Convention trong YAML thường dùng kebab-case (chữ thường + gạch ngang)
- Dễ đọc và nhất quán với các ví dụ GitHub

#### `name: 🅰️ Job A`
- **Tên hiển thị** của job trên GitHub UI
- Có thể dùng emoji để dễ nhận diện
- **Khác với job ID:** `name` để hiển thị, job ID để tham chiếu trong code

#### `runs-on: ubuntu-latest`
- **Runner** - máy chạy job
- `ubuntu-latest` là Linux Ubuntu runner do GitHub cung cấp

**So sánh các runners:**

| Runner | Mô tả | Ưu điểm | Nhược điểm |
|--------|-------|---------|------------|
| `ubuntu-latest` | Ubuntu mới nhất | Phổ biến, nhiều tools | Có thể thay đổi version |
| `ubuntu-22.04` | Ubuntu cố định | Ổn định, predictable | Có thể outdated |
| `ubuntu-24.04` | Ubuntu 24.04 | Mới nhất | Có thể thiếu support |
| `windows-latest` | Windows Server | Cần cho Windows apps | Chậm hơn, tốn quota |
| `macos-latest` | macOS | Cần cho iOS/macOS | Đắt nhất (10x Linux) |
| `self-hosted` | Server riêng | Không giới hạn, customize | Tự quản lý |

**Tại sao dùng `ubuntu-latest`?**
- Miễn phí với public repos
- Nhanh khởi động (~30s)
- Có sẵn hầu hết các tools phổ biến
- Thích hợp cho hầu hết các tác vụ CI/CD

#### `steps:`
- Danh sách các bước thực thi trong job
- Chạy **tuần tự** từ trên xuống dưới
- Mỗi step có thể là `run` (command) hoặc `uses` (action)

#### `- run: |`
- Chạy shell commands
- Ký tự `|` (pipe) cho phép viết **multi-line commands**
- Mỗi dòng là một command riêng biệt

**Các cách viết `run`:**

```yaml
# Cách 1: Single line
- run: echo "Hello"

# Cách 2: Multi-line với |
- run: |
    echo "Line 1"
    echo "Line 2"

# Cách 3: Multi-line với >
- run: >
    echo "Tất cả trên
    một dòng"

# Cách 4: Với tên step
- name: Say Hello
  run: echo "Hello"
```

#### `sleep 5`
- Tạm dừng 5 giây
- **Mục đích demo:** Giả lập tác vụ tốn thời gian
- Trong thực tế: build, test, deploy thường mất thời gian

---

## 🅱️ Job B - Chạy Song Song

```yaml
job-b:
  name: 🅱️ Job B
  runs-on: ubuntu-latest
  steps:
    - run: |
        echo "Job B starting..."
        sleep 3
        echo "Job B done!"
```

**Điểm quan trọng:**
- Job B **KHÔNG** có `needs`, nên nó chạy **SONG SONG** với Job A
- Sleep 3 giây (ngắn hơn Job A)
- Job B sẽ hoàn thành trước Job A (3s < 5s)

**Tại sao cho Job B ngắn hơn?**
- Demo rằng các jobs chạy độc lập
- Jobs kết thúc theo thứ tự hoàn thành, không phải thứ tự định nghĩa

---

## 🅲 Job C - Phụ Thuộc Nhiều Jobs

```yaml
job-c:
  name: 🅲 Job C (depends on A & B)
  needs: [job-a, job-b]
  runs-on: ubuntu-latest
  steps:
    - run: echo "Job C - After A and B!"
```

### `needs: [job-a, job-b]`

**Đây là lệnh QUAN TRỌNG NHẤT của file này!**

**Giải thích:**
- `needs` xác định **dependencies** (phụ thuộc)
- Job C **CHỈ CHẠY** khi cả Job A VÀ Job B đều **thành công**
- Cú pháp array `[job-a, job-b]` cho phép phụ thuộc nhiều jobs

**Các cách viết `needs`:**

```yaml
# Cách 1: Phụ thuộc 1 job
needs: job-a

# Cách 2: Phụ thuộc nhiều jobs (array)
needs: [job-a, job-b]

# Cách 3: Nhiều dòng
needs:
  - job-a
  - job-b
```

**Tại sao dùng `needs`?**
- **Đảm bảo thứ tự**: Build trước → Test sau → Deploy cuối
- **Tiết kiệm resources**: Không chạy test nếu build fail
- **Logic phụ thuộc**: Job deploy cần kết quả từ job build

**Điều gì xảy ra nếu Job A hoặc B fail?**
- Job C sẽ **KHÔNG chạy** và được đánh dấu `skipped`
- Toàn bộ workflow sẽ fail

---

## 🅳 Job D - Chuỗi Dependencies

```yaml
job-d:
  name: 🅳 Job D (depends on C)
  needs: job-c
  runs-on: ubuntu-latest
  steps:
    - run: echo "Job D - Final job!"
```

**Giải thích:**
- Job D phụ thuộc Job C
- Tạo thành **chuỗi** dependencies: A+B → C → D
- Mô phỏng pipeline thực tế: Build → Test → Deploy

**Dependency Graph:**

```
      ┌─────────┐
      │  Job A  │──────┐
      │ (5 sec) │      │
      └─────────┘      ▼
                   ┌─────────┐     ┌─────────┐
                   │  Job C  │────▶│  Job D  │
                   │         │     │ (final) │
      ┌─────────┐  └─────────┘     └─────────┘
      │  Job B  │──────┘
      │ (3 sec) │
      └─────────┘
```

---

## 🪟 Job Windows - Cross-Platform

```yaml
job-windows:
  name: 🪟 Windows Job
  runs-on: windows-latest
  steps:
    - run: Write-Output "Hello from Windows!"
      shell: pwsh
```

### `runs-on: windows-latest`

**Giải thích:**
- Chạy job trên Windows Server runner
- Có sẵn PowerShell, cmd, bash (via Git Bash)

### `shell: pwsh`

**Đây là điểm quan trọng cho cross-platform!**

**Giải thích:**
- `pwsh` = **PowerShell Core** (cross-platform)
- `Write-Output` là cmdlet của PowerShell

**So sánh các shell options:**

| Shell | Runner | Mô tả |
|-------|--------|-------|
| `bash` | Linux/macOS/Windows | Bash shell (default trên Linux/macOS) |
| `pwsh` | Tất cả | PowerShell Core 7+ (cross-platform) |
| `powershell` | Windows | Windows PowerShell 5.1 (legacy) |
| `cmd` | Windows | Command Prompt |
| `python` | Tất cả | Python script |

**Tại sao dùng `pwsh` thay vì `powershell`?**
- `pwsh` (PowerShell Core) chạy được trên **mọi OS**
- `powershell` chỉ có trên Windows
- `pwsh` là version mới hơn (7.x) với nhiều tính năng

**Tại sao dùng `Write-Output` thay vì `echo`?**
- `Write-Output` là cmdlet PowerShell native
- `echo` cũng hoạt động (alias trong PowerShell)
- Demo sự khác biệt giữa Windows và Linux

---

## 🔄 Các Lệnh Liên Quan Quan Trọng

### 1. `if` - Conditional Execution

```yaml
job-x:
  needs: job-a
  if: ${{ success() }}  # Chỉ chạy nếu job-a thành công
  # hoặc
  if: ${{ failure() }}  # Chỉ chạy nếu job-a fail
  # hoặc
  if: ${{ always() }}   # Luôn chạy
```

**Các conditions phổ biến:**

| Condition | Mô tả |
|-----------|-------|
| `success()` | Job trước thành công (default) |
| `failure()` | Job trước fail |
| `always()` | Luôn chạy |
| `cancelled()` | Workflow bị cancelled |

### 2. Truy cập Outputs từ Job Khác

```yaml
job-a:
  outputs:
    version: ${{ steps.get-version.outputs.version }}
  steps:
    - id: get-version
      run: echo "version=1.0.0" >> $GITHUB_OUTPUT

job-b:
  needs: job-a
  steps:
    - run: echo "Version is ${{ needs.job-a.outputs.version }}"
```

### 3. `continue-on-error`

```yaml
job-x:
  continue-on-error: true  # Job fail không làm workflow fail
```

### 4. `timeout-minutes`

```yaml
job-x:
  timeout-minutes: 10  # Job tự động fail sau 10 phút
```

### 5. `concurrency` - Giới Hạn Song Song

```yaml
job-x:
  concurrency:
    group: my-deployment
    cancel-in-progress: true
```

---

## 🏗️ Best Practices

### 1. Đặt Tên Job Có Ý Nghĩa

```yaml
# ❌ Không tốt
job1:
  name: Job 1

# ✅ Tốt
build:
  name: 🔨 Build Application

test:
  name: 🧪 Run Tests
```

### 2. Sử Dụng Dependencies Hợp Lý

```yaml
# Typical CI/CD Pipeline
jobs:
  build:
    # Build first
  
  test:
    needs: build
    # Test after build
  
  deploy-staging:
    needs: test
    # Deploy to staging after tests
  
  deploy-prod:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    # Deploy to prod only from main branch
```

### 3. Fail Fast

```yaml
# Nếu build fail, không cần chạy test và deploy
jobs:
  build:
    # ...
  test:
    needs: build  # Skip nếu build fail
  deploy:
    needs: [build, test]  # Skip nếu bất kỳ job nào fail
```

---

## 📊 So Sánh Với Các CI/CD Khác

| Khái niệm | GitHub Actions | GitLab CI | Jenkins |
|-----------|---------------|-----------|---------|
| Dependencies | `needs` | `needs` | `dependsOn` |
| Parallel | Mặc định | Mặc định với `stages` | `parallel` |
| Runner | `runs-on` | `tags` | `agent` |
| Condition | `if` | `rules/only/except` | `when` |

---

## 💡 Tùy Biến Thực Tế

### Ví dụ 1: Pipeline Build → Test → Deploy

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    name: 🔨 Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  test:
    name: 🧪 Test
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      - run: npm test

  deploy:
    name: 🚀 Deploy
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - run: echo "Deploying..."
```

### Ví dụ 2: Matrix với Dependencies

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Building on ${{ matrix.os }}"

  deploy:
    needs: build  # Chờ TẤT CẢ matrix builds hoàn thành
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

---

## ❓ FAQ

### Q: Nếu không có `needs`, jobs có chạy song song không?
**A:** Có! Mặc định tất cả jobs chạy song song.

### Q: Có thể phụ thuộc vào job từ workflow khác không?
**A:** Không trực tiếp. Cần dùng `workflow_run` trigger hoặc `workflow_call`.

### Q: Giới hạn số job song song là bao nhiêu?
**A:** 
- Free tier: 20 concurrent jobs
- Pro: 40 concurrent jobs
- Team: 60 concurrent jobs

### Q: Job fail thì sao?
**A:** 
- Các jobs phụ thuộc sẽ `skipped`
- Toàn bộ workflow sẽ có status `failure`
- Dùng `continue-on-error: true` nếu muốn bỏ qua lỗi

---

## 📖 Tham Khảo

- [GitHub Actions Jobs Documentation](https://docs.github.com/en/actions/using-jobs)
- [GitHub-hosted Runners](https://docs.github.com/en/actions/using-github-hosted-runners)
- [Workflow Syntax - needs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)
