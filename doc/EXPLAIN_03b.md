# Giải Thích Chi Tiết: 03b - Conditionals Demo

## 📋 Tổng Quan

File `03b-conditionals.yml` minh họa cách sử dụng **điều kiện (conditionals)** trong GitHub Actions để kiểm soát khi nào một job hoặc step được chạy. Đây là một tính năng cực kỳ quan trọng để tạo ra các workflow linh hoạt và tối ưu.

---

## 📝 Nội Dung File Gốc

```yaml
name: 03b - Conditionals Demo

on:
  workflow_dispatch:
    inputs:
      run_deploy:
        description: 'Run deploy job?'
        type: boolean
        default: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  # Chỉ chạy nếu input = true
  deploy:
    needs: build
    if: ${{ inputs.run_deploy == true }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."

  # Luôn chạy, kể cả khi job trước fail
  notify:
    needs: [build, deploy]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Build status: ${{ needs.build.result }}"
          echo "Deploy status: ${{ needs.deploy.result }}"
```

---

## 🔍 Giải Thích Chi Tiết Từng Phần

### 1. Tên Workflow

```yaml
name: 03b - Conditionals Demo
```

| Thành phần | Giải thích |
|------------|------------|
| `name` | Tên hiển thị của workflow trên GitHub Actions UI |
| `03b - Conditionals Demo` | Tên mô tả mục đích: demo về điều kiện |

**Tại sao dùng `name`?**
- Giúp dễ dàng nhận diện workflow trong danh sách
- Nếu không có `name`, GitHub sẽ sử dụng tên file làm tên mặc định

---

### 2. Event Trigger với Inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      run_deploy:
        description: 'Run deploy job?'
        type: boolean
        default: false
```

#### 📌 `workflow_dispatch`

| Thuộc tính | Giải thích |
|------------|------------|
| `workflow_dispatch` | Cho phép chạy workflow thủ công từ GitHub UI |
| `inputs` | Định nghĩa các tham số đầu vào khi chạy thủ công |

**Tại sao dùng `workflow_dispatch` thay vì các trigger khác?**

| Trigger | Khi nào dùng | So sánh |
|---------|--------------|---------|
| `workflow_dispatch` | Chạy thủ công, cần nhập tham số | ✅ Phù hợp cho demo, deploy thủ công |
| `push` | Tự động khi push code | ❌ Không cho phép nhập input |
| `pull_request` | Khi tạo/update PR | ❌ Không cho phép nhập input |
| `schedule` | Chạy theo lịch (cron) | ❌ Không tương tác được |

#### 📌 Input Parameters

```yaml
inputs:
  run_deploy:
    description: 'Run deploy job?'
    type: boolean
    default: false
```

| Thuộc tính | Giải thích |
|------------|------------|
| `run_deploy` | Tên của input parameter |
| `description` | Mô tả hiển thị trên UI khi chạy workflow |
| `type: boolean` | Kiểu dữ liệu là true/false |
| `default: false` | Giá trị mặc định nếu không chọn |

**Các kiểu `type` có thể dùng:**

| Type | Mô tả | Ví dụ |
|------|-------|-------|
| `string` | Chuỗi văn bản | Tên version, branch |
| `boolean` | True/False | Bật/tắt tính năng |
| `choice` | Chọn từ danh sách | Chọn môi trường (dev/staging/prod) |
| `environment` | Chọn environment có sẵn | Chọn deployment environment |

**Ví dụ với `choice`:**
```yaml
inputs:
  environment:
    description: 'Select environment'
    type: choice
    options:
      - development
      - staging
      - production
    default: development
```

---

### 3. Job `build` - Job Cơ Bản

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
```

| Thành phần | Giải thích |
|------------|------------|
| `build` | Tên định danh của job |
| `runs-on: ubuntu-latest` | Chạy trên máy ảo Ubuntu mới nhất |
| `steps` | Danh sách các bước thực hiện |
| `run: echo "Building..."` | In ra thông báo "Building..." |

**Đây là job đơn giản không có điều kiện**, luôn chạy khi workflow được trigger.

---

### 4. Job `deploy` - Conditional Job ⭐

```yaml
deploy:
  needs: build
  if: ${{ inputs.run_deploy == true }}
  runs-on: ubuntu-latest
  steps:
    - run: echo "Deploying..."
```

#### 📌 `needs: build`

| Thuộc tính | Giải thích |
|------------|------------|
| `needs` | Khai báo job phụ thuộc |
| `build` | Job `deploy` CHỈ chạy SAU KHI `build` hoàn thành thành công |

**Tại sao dùng `needs`?**
- Đảm bảo thứ tự thực hiện: build trước, deploy sau
- Nếu `build` fail, `deploy` sẽ bị skip (mặc định)

**Các cách khai báo `needs`:**
```yaml
# Phụ thuộc 1 job
needs: build

# Phụ thuộc nhiều job
needs: [build, test]

# Hoặc viết dạng danh sách
needs:
  - build
  - test
```

#### 📌 `if: ${{ inputs.run_deploy == true }}` ⭐⭐⭐

Đây là **trọng tâm của file này** - câu lệnh điều kiện!

| Thành phần | Giải thích |
|------------|------------|
| `if` | Keyword để định nghĩa điều kiện |
| `${{ }}` | Cú pháp expression của GitHub Actions |
| `inputs.run_deploy` | Truy cập giá trị input đã định nghĩa |
| `== true` | So sánh bằng với giá trị `true` |

**Cách hoạt động:**
- Nếu `run_deploy == true` → Job `deploy` được chạy
- Nếu `run_deploy == false` → Job `deploy` bị **skip**

**Tại sao viết `${{ inputs.run_deploy == true }}`?**

| Cách viết | Kết quả | Giải thích |
|-----------|---------|------------|
| `${{ inputs.run_deploy == true }}` | ✅ Đúng | So sánh boolean rõ ràng |
| `${{ inputs.run_deploy }}` | ✅ Cũng đúng | Boolean tự động đánh giá |
| `inputs.run_deploy == true` | ✅ Cũng đúng | GitHub tự thêm `${{ }}` cho `if` |

> **Lưu ý:** Với `if`, bạn có thể bỏ `${{ }}` vì GitHub tự động wrap. Tuy nhiên, viết đầy đủ sẽ rõ ràng hơn.

---

### 5. Job `notify` - Always Run Job ⭐⭐

```yaml
notify:
  needs: [build, deploy]
  if: always()
  runs-on: ubuntu-latest
  steps:
    - run: |
        echo "Build status: ${{ needs.build.result }}"
        echo "Deploy status: ${{ needs.deploy.result }}"
```

#### 📌 `needs: [build, deploy]`

Job `notify` phụ thuộc vào CẢ HAI job `build` và `deploy`.

#### 📌 `if: always()` ⭐⭐⭐

| Hàm | Giải thích |
|-----|------------|
| `always()` | Job này LUÔN chạy, bất kể job trước thành công, thất bại, hay bị skip |

**Tại sao dùng `always()`?**
- Mặc định, nếu job trước fail hoặc bị skip, job sau sẽ không chạy
- `always()` đảm bảo job notify luôn gửi thông báo

**So sánh các Status Check Functions:**

| Function | Khi nào chạy | Use case |
|----------|--------------|----------|
| `always()` | Luôn luôn | Gửi notification, cleanup |
| `success()` | Tất cả job trước thành công | Xác nhận hoàn thành (mặc định) |
| `failure()` | Có ít nhất 1 job trước fail | Gửi alert khi lỗi |
| `cancelled()` | Workflow bị cancel | Cleanup khi cancel |

**Ví dụ sử dụng:**
```yaml
# Chỉ chạy khi có lỗi
cleanup-on-failure:
  needs: [build, test]
  if: failure()
  runs-on: ubuntu-latest
  steps:
    - run: echo "Something failed! Cleaning up..."

# Chạy khi thành công
send-success-notification:
  needs: [build, test, deploy]
  if: success()
  runs-on: ubuntu-latest
  steps:
    - run: echo "All jobs succeeded!"
```

#### 📌 `${{ needs.build.result }}` và `${{ needs.deploy.result }}`

| Expression | Giải thích |
|------------|------------|
| `needs.<job_id>.result` | Lấy kết quả của job đã chạy trước đó |

**Các giá trị result có thể có:**

| Result | Nghĩa |
|--------|-------|
| `success` | Job hoàn thành thành công |
| `failure` | Job thất bại |
| `cancelled` | Job bị hủy |
| `skipped` | Job bị bỏ qua (do điều kiện không thỏa) |

---

## 🎯 Các Lệnh Điều Kiện Nâng Cao

### 1. Operators (Toán tử)

```yaml
# So sánh
if: ${{ github.event_name == 'push' }}
if: ${{ github.ref != 'refs/heads/main' }}

# Logic
if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
if: ${{ github.event_name == 'push' || github.event_name == 'workflow_dispatch' }}
if: ${{ !cancelled() }}

# Chứa
if: ${{ contains(github.event.head_commit.message, '[skip ci]') }}
if: ${{ startsWith(github.ref, 'refs/tags/') }}
if: ${{ endsWith(github.repository, '-demo') }}
```

### 2. Context Variables Thường Dùng

| Context | Mô tả | Ví dụ |
|---------|-------|-------|
| `github.event_name` | Tên event trigger | `push`, `pull_request` |
| `github.ref` | Branch/tag reference | `refs/heads/main` |
| `github.actor` | User trigger workflow | `username` |
| `github.repository` | Tên repo đầy đủ | `owner/repo` |
| `github.ref_name` | Tên branch/tag ngắn | `main`, `v1.0.0` |

### 3. Điều Kiện Phổ Biến Trong Thực Tế

```yaml
# Chỉ chạy trên main branch
if: github.ref == 'refs/heads/main'

# Chỉ chạy khi push (không phải PR)
if: github.event_name == 'push'

# Chỉ chạy khi là tag release
if: startsWith(github.ref, 'refs/tags/v')

# Skip nếu commit message chứa [skip ci]
if: ${{ !contains(github.event.head_commit.message, '[skip ci]') }}

# Chạy trên production environment
if: github.event.inputs.environment == 'production'

# Kết hợp nhiều điều kiện
if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' && success() }}
```

---

## 📊 Luồng Hoạt Động

```
┌─────────────────────────────────────────────────────────────┐
│                  Workflow Dispatch                          │
│                  (User clicks Run)                          │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Input: run_deploy = ?                              │   │
│  │  □ false (default)    ☑ true                        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    build     │
                    │  (Always)    │
                    └──────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
     run_deploy=true  run_deploy=false     │
           │               │               │
           ▼               ▼               │
    ┌──────────────┐ ┌──────────────┐      │
    │    deploy    │ │   (SKIP)     │      │
    │   (Chạy)     │ │              │      │
    └──────────────┘ └──────────────┘      │
           │               │               │
           └───────────────┼───────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    notify    │
                    │  if: always()│
                    │  (Luôn chạy) │
                    └──────────────┘
```

---

## 💡 Các Mẫu Ứng Dụng Thực Tế

### Mẫu 1: Deploy theo môi trường

```yaml
name: Deploy to Environment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Select environment'
        type: choice
        options:
          - development
          - staging
          - production
        default: development

jobs:
  deploy-dev:
    if: ${{ inputs.environment == 'development' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to DEV..."

  deploy-staging:
    if: ${{ inputs.environment == 'staging' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to STAGING..."

  deploy-prod:
    if: ${{ inputs.environment == 'production' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to PRODUCTION..."
```

### Mẫu 2: Skip CI với commit message

```yaml
jobs:
  build:
    if: ${{ !contains(github.event.head_commit.message, '[skip ci]') }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
```

### Mẫu 3: Chỉ deploy khi merge vào main

```yaml
jobs:
  deploy:
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production..."
```

### Mẫu 4: Notification đầy đủ

```yaml
jobs:
  notify:
    needs: [build, test, deploy]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Success notification
        if: success()
        run: echo "✅ All jobs succeeded!"
      
      - name: Failure notification
        if: failure()
        run: echo "❌ Some jobs failed!"
      
      - name: Cancelled notification
        if: cancelled()
        run: echo "⚠️ Workflow was cancelled!"
```

---

## 📚 Tham Khảo Thêm

- [GitHub Actions Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [GitHub Actions Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## ✅ Tóm Tắt

| Khái niệm | Cú pháp | Mục đích |
|-----------|---------|----------|
| Điều kiện cơ bản | `if: ${{ condition }}` | Kiểm tra điều kiện trước khi chạy |
| Input boolean | `inputs.name == true` | Kiểm tra input từ user |
| Luôn chạy | `if: always()` | Chạy bất kể kết quả trước đó |
| Chạy khi fail | `if: failure()` | Chạy chỉ khi có job fail |
| Lấy kết quả job | `needs.<job>.result` | Truy cập status của job trước |
| Phụ thuộc job | `needs: [job1, job2]` | Đợi các job hoàn thành |
