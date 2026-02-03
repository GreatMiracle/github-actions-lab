# Giải Thích Chi Tiết: 04 - Environment Variables

## 📋 Tổng Quan

File `04-environment-vars.yml` là một workflow GitHub Actions minh họa cách sử dụng **biến môi trường (Environment Variables)** ở nhiều cấp độ khác nhau và cách truy cập các **GitHub Context Variables**.

---

## 📝 Nội Dung File Gốc

```yaml
name: 04 - Environment Variables

on:
  workflow_dispatch:

env:
  APP_NAME: my-fastapi-app
  PYTHON_VERSION: "3.11"

jobs:
  show-env:
    runs-on: ubuntu-latest

    env:
      JOB_ENV: job-specific-value

    steps:
      - name: Show Workflow Env
        run: |
          echo "App Name: $APP_NAME"
          echo "Python: $PYTHON_VERSION"
      - name: Show Job Env
        run: | 
           echo "Job Env: $JOB_ENV"
      
      - name: Step-level Env
        env:
          STEP_VAR: step-specific
        run: | 
          echo "Step Var: $STEP_VAR"

      - name: GitHub Context Variables
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Actor: ${{ github.actor }}"
          echo "Run ID: ${{ github.run_id }}"
          echo "Run Number: ${{ github.run_number }}"
          echo "Workflow: ${{ github.workflow }}"
          echo "Job: ${{ github.job }}"
          echo "Event: ${{ github.event_name }}"
          echo "Ref: ${{ github.ref }}"
          echo "SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit Message: ${{ github.event.head_commit.message }}"
          echo "PR Number: ${{ github.event.pull_request.number }}"
          echo "PR Title: ${{ github.event.pull_request.title }}"
          echo "PR Author: ${{ github.event.pull_request.user.login }}"
          echo "Schedule Time: ${{ github.triggering_actor }}"
```

---

## 🔍 Giải Thích Chi Tiết Từng Phần

### 1. Tên Workflow

```yaml
name: 04 - Environment Variables
```

| Thuộc tính | Giải thích |
|------------|------------|
| `name` | Đặt tên hiển thị cho workflow trong GitHub Actions UI |

**Tại sao dùng `name`?**
- Giúp dễ dàng nhận diện workflow trong danh sách Actions
- Không bắt buộc, nhưng nên có để quản lý tốt hơn
- Nếu không có, GitHub sẽ dùng tên file làm tên workflow

---

### 2. Trigger Event

```yaml
on:
  workflow_dispatch:
```

| Thuộc tính | Giải thích |
|------------|------------|
| `on` | Định nghĩa khi nào workflow được kích hoạt |
| `workflow_dispatch` | Cho phép chạy thủ công từ GitHub UI |

**Tại sao dùng `workflow_dispatch`?**
- Phù hợp cho việc học và thử nghiệm
- Có thể chạy bất cứ lúc nào mà không cần push code
- Có thể thêm input parameters để linh hoạt hơn

**Các trigger khác có thể dùng:**
```yaml
# Trigger khi push
on:
  push:
    branches: [main]

# Trigger khi tạo PR
on:
  pull_request:
    branches: [main]

# Trigger theo lịch
on:
  schedule:
    - cron: '0 0 * * *'  # Chạy hàng ngày lúc 00:00 UTC

# Kết hợp nhiều trigger
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:
```

---

### 3. Biến Môi Trường Cấp Workflow (Workflow-level Env)

```yaml
env:
  APP_NAME: my-fastapi-app
  PYTHON_VERSION: "3.11"
```

| Thuộc tính | Giải thích |
|------------|------------|
| `env` | Khai báo biến môi trường |
| `APP_NAME` | Tên biến tùy chỉnh |
| `PYTHON_VERSION` | Phiên bản Python (dùng dấu ngoặc kép vì số có thể bị hiểu sai) |

**Tại sao dùng biến môi trường cấp workflow?**
- **Tái sử dụng**: Dùng được ở TẤT CẢ jobs và steps trong workflow
- **Bảo trì dễ**: Thay đổi một chỗ, áp dụng mọi nơi
- **Dễ đọc**: Code rõ ràng, không có "magic numbers"

**⚠️ Lưu ý quan trọng:**
```yaml
# SAI - số có thể bị parse thành float
PYTHON_VERSION: 3.11  # → 3.11 có thể thành "3.10999..."

# ĐÚNG - dùng string
PYTHON_VERSION: "3.11"
```

**Phạm vi áp dụng của biến:**

| Cấp độ | Phạm vi |
|--------|---------|
| Workflow-level | Toàn bộ workflow (tất cả jobs, steps) |
| Job-level | Chỉ trong job đó |
| Step-level | Chỉ trong step đó |

---

### 4. Định nghĩa Job

```yaml
jobs:
  show-env:
    runs-on: ubuntu-latest
```

| Thuộc tính | Giải thích |
|------------|------------|
| `jobs` | Khối chứa tất cả các jobs |
| `show-env` | ID của job (dùng cho tham chiếu, needs, outputs) |
| `runs-on` | Chỉ định runner để chạy job |
| `ubuntu-latest` | Máy ảo Ubuntu phiên bản mới nhất |

**Tại sao dùng `ubuntu-latest`?**
- Miễn phí cho public repositories
- Có đầy đủ công cụ cài sẵn
- Phù hợp với hầu hết các dự án

**Các runner khác có thể dùng:**
```yaml
# Windows
runs-on: windows-latest

# macOS
runs-on: macos-latest

# Ubuntu phiên bản cụ thể
runs-on: ubuntu-22.04

# Self-hosted runner
runs-on: self-hosted

# Nhiều runner
runs-on: [self-hosted, linux, x64]
```

---

### 5. Biến Môi Trường Cấp Job (Job-level Env)

```yaml
    env:
      JOB_ENV: job-specific-value
```

| Thuộc tính | Giải thích |
|------------|------------|
| `env` | Biến môi trường cấp job |
| `JOB_ENV` | Biến chỉ khả dụng trong job `show-env` |

**Tại sao dùng biến cấp job?**
- Khi biến chỉ cần thiết cho một job cụ thể
- Tránh "ô nhiễm" namespace của workflow
- Có thể override biến cấp workflow

**Ví dụ override:**
```yaml
env:
  APP_NAME: global-app

jobs:
  job1:
    env:
      APP_NAME: job1-app  # Override biến workflow
    steps:
      - run: echo $APP_NAME  # In ra: job1-app

  job2:
    steps:
      - run: echo $APP_NAME  # In ra: global-app
```

---

### 6. Hiển Thị Biến Workflow

```yaml
      - name: Show Workflow Env
        run: |
          echo "App Name: $APP_NAME"
          echo "Python: $PYTHON_VERSION"
```

| Thuộc tính | Giải thích |
|------------|------------|
| `name` | Tên hiển thị của step |
| `run` | Thực thi lệnh shell |
| `\|` | Multi-line string trong YAML |
| `$APP_NAME` | Truy cập biến môi trường theo cú pháp shell |

**Tại sao dùng `$VARIABLE`?**
- Cú pháp chuẩn của shell (bash)
- Đơn giản, dễ đọc
- Hoạt động với biến môi trường hệ thống

**Cách khác để truy cập biến:**
```yaml
# Cú pháp shell (phổ biến nhất)
run: echo $APP_NAME

# Cú pháp GitHub Expression
run: echo ${{ env.APP_NAME }}

# Trong điều kiện if
if: ${{ env.APP_NAME == 'my-app' }}
```

**⚠️ Khi nào dùng cú pháp nào:**

| Cú pháp | Khi nào dùng |
|---------|--------------|
| `$VAR` | Trong lệnh `run` (shell script) |
| `${{ env.VAR }}` | Trong `if`, `with`, hoặc khi cần evaluate expression |

---

### 7. Biến Môi Trường Cấp Step (Step-level Env)

```yaml
      - name: Step-level Env
        env:
          STEP_VAR: step-specific
        run: | 
          echo "Step Var: $STEP_VAR"
```

| Thuộc tính | Giải thích |
|------------|------------|
| `env` | Biến cấp step, chỉ khả dụng trong step này |
| `STEP_VAR` | Biến chỉ tồn tại trong step "Step-level Env" |

**Tại sao dùng biến cấp step?**
- Khi biến chỉ cần cho một step duy nhất
- Giữ biến cục bộ, tránh xung đột
- Thường dùng cho API keys, tokens kết hợp với secrets

**Ví dụ thực tế với secrets:**
```yaml
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          deploy.sh --api-key $API_KEY --token $DEPLOY_TOKEN
```

---

### 8. GitHub Context Variables

```yaml
      - name: GitHub Context Variables
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Actor: ${{ github.actor }}"
          # ... và nhiều biến khác
```

**GitHub Context** là một object chứa thông tin về workflow run, repository, event, v.v.

#### Bảng Giải Thích Các GitHub Context Variables:

| Variable | Giải thích | Ví dụ giá trị |
|----------|------------|---------------|
| `github.repository` | Tên owner/repo | `GreatMiracle/github-actions-lab` |
| `github.ref_name` | Tên branch hoặc tag | `main`, `feature/login` |
| `github.ref` | Ref đầy đủ | `refs/heads/main` |
| `github.actor` | User kích hoạt workflow | `GreatMiracle` |
| `github.run_id` | ID duy nhất của run | `1234567890` |
| `github.run_number` | Số thứ tự run (tăng dần) | `42` |
| `github.workflow` | Tên workflow | `04 - Environment Variables` |
| `github.job` | ID của job hiện tại | `show-env` |
| `github.event_name` | Tên event kích hoạt | `push`, `pull_request`, `workflow_dispatch` |
| `github.sha` | Commit SHA | `a1b2c3d4e5f6...` |
| `github.triggering_actor` | User thực sự trigger (có thể khác actor) | `GreatMiracle` |

#### Event-specific Variables:

| Variable | Khi nào có giá trị | Giải thích |
|----------|-------------------|------------|
| `github.event.head_commit.message` | Push event | Message của commit cuối |
| `github.event.pull_request.number` | PR event | Số PR |
| `github.event.pull_request.title` | PR event | Tiêu đề PR |
| `github.event.pull_request.user.login` | PR event | Username người tạo PR |

**⚠️ Lưu ý quan trọng:**
- Các biến event-specific chỉ có giá trị khi workflow được trigger bởi event tương ứng
- Nếu chạy bằng `workflow_dispatch`, các biến như `pull_request.number` sẽ trống

---

## 📊 So Sánh Các Cấp Độ Biến Môi Trường

```
┌─────────────────────────────────────────────────────────┐
│                    WORKFLOW LEVEL                        │
│  env:                                                    │
│    APP_NAME: my-app                                      │
│    ↓ Có thể truy cập từ TẤT CẢ jobs và steps            │
├─────────────────────────────────────────────────────────┤
│                      JOB LEVEL                           │
│  jobs:                                                   │
│    build:                                                │
│      env:                                                │
│        BUILD_TYPE: production                            │
│        ↓ Chỉ trong job này                              │
├─────────────────────────────────────────────────────────┤
│                     STEP LEVEL                           │
│      steps:                                              │
│        - name: Deploy                                    │
│          env:                                            │
│            API_KEY: ${{ secrets.KEY }}                   │
│            ↓ Chỉ trong step này                         │
└─────────────────────────────────────────────────────────┘
```

### Thứ Tự Ưu Tiên (Override):
```
Step-level > Job-level > Workflow-level
```

Nếu cùng tên biến được định nghĩa ở nhiều cấp, cấp thấp hơn sẽ override cấp cao hơn.

---

## 🎯 Các Context Khác Trong GitHub Actions

### 1. `env` Context
```yaml
${{ env.MY_VAR }}
```
Truy cập biến môi trường đã định nghĩa.

### 2. `secrets` Context
```yaml
${{ secrets.API_KEY }}
```
Truy cập secrets đã cấu hình trong Settings > Secrets.

### 3. `vars` Context
```yaml
${{ vars.CONFIG_VALUE }}
```
Truy cập configuration variables (Settings > Variables).

### 4. `runner` Context
```yaml
${{ runner.os }}        # Linux, Windows, macOS
${{ runner.arch }}      # X64, ARM64
${{ runner.temp }}      # Thư mục temp
${{ runner.tool_cache }} # Thư mục tool cache
```

### 5. `job` Context
```yaml
${{ job.status }}       # success, failure, cancelled
${{ job.container.id }} # ID container nếu dùng container
```

### 6. `steps` Context
```yaml
${{ steps.step_id.outputs.output_name }}
${{ steps.step_id.outcome }}  # success, failure, skipped
```

### 7. `matrix` Context
```yaml
${{ matrix.os }}
${{ matrix.node-version }}
```

### 8. `needs` Context
```yaml
${{ needs.job_id.outputs.output_name }}
${{ needs.job_id.result }}  # success, failure, skipped
```

---

## 💡 Best Practices

### 1. Đặt Tên Biến
```yaml
# ĐÚNG - uppercase với underscore
APP_NAME: my-app
PYTHON_VERSION: "3.11"
DATABASE_URL: postgres://...

# SAI - lowercase hoặc camelCase
appName: my-app
pythonVersion: "3.11"
```

### 2. Sensitive Data → Dùng Secrets
```yaml
# SAI - không bao giờ hardcode secrets
env:
  API_KEY: sk-abc123xyz

# ĐÚNG - dùng secrets
env:
  API_KEY: ${{ secrets.API_KEY }}
```

### 3. Giá Trị Cấu Hình → Dùng Variables
```yaml
# Trong Repository Settings > Variables
# MAX_RETRIES: 3
# TIMEOUT: 60

# Trong workflow
env:
  MAX_RETRIES: ${{ vars.MAX_RETRIES }}
  TIMEOUT: ${{ vars.TIMEOUT }}
```

### 4. Tổ Chức Biến Theo Cấp Độ Hợp Lý
```yaml
# Workflow-level: Biến dùng chung
env:
  APP_NAME: my-app
  ENVIRONMENT: production

jobs:
  build:
    # Job-level: Biến riêng cho job
    env:
      BUILD_FLAGS: --release
    steps:
      - name: Build
        # Step-level: Biến riêng cho step
        env:
          SIGNING_KEY: ${{ secrets.SIGNING_KEY }}
        run: ...
```

---

## 🔄 Ví Dụ Thực Tế

### Ví Dụ 1: CI Pipeline Với Nhiều Cấp Độ Biến

```yaml
name: CI Pipeline

on: [push, pull_request]

env:
  # Workflow-level: Dùng cho tất cả jobs
  NODE_VERSION: "18"
  REGISTRY: ghcr.io

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      # Job-level: Dùng cho tất cả steps trong job test
      TEST_REPORT_PATH: ./reports
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        env:
          # Step-level: Chỉ dùng trong step này
          CI: true
          COVERAGE: true
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    env:
      # Job-level khác với job test
      DEPLOY_ENV: production
    steps:
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          echo "Deploying ${{ env.REGISTRY }}/${{ github.repository }}"
          echo "Environment: $DEPLOY_ENV"
```

### Ví Dụ 2: Sử Dụng Dynamic Environment Variables

```yaml
name: Dynamic Env

on: push

jobs:
  set-env:
    runs-on: ubuntu-latest
    steps:
      - name: Set dynamic env
        run: |
          echo "BUILD_DATE=$(date +'%Y-%m-%d')" >> $GITHUB_ENV
          echo "SHORT_SHA=${GITHUB_SHA::7}" >> $GITHUB_ENV
      
      - name: Use dynamic env
        run: |
          echo "Build date: $BUILD_DATE"
          echo "Short SHA: $SHORT_SHA"
```

**Giải thích `$GITHUB_ENV`:**
- Đây là file đặc biệt để set biến môi trường cho các step sau
- Cú pháp: `echo "VAR_NAME=value" >> $GITHUB_ENV`
- Biến sẽ khả dụng từ step tiếp theo (không phải step hiện tại)

---

## ❓ FAQ - Câu Hỏi Thường Gặp

### Q1: Khi nào dùng `$VAR` vs `${{ env.VAR }}`?

| Trường hợp | Cú pháp |
|------------|---------|
| Trong `run:` (shell script) | `$VAR` |
| Trong `if:` condition | `${{ env.VAR }}` |
| Trong `with:` parameters | `${{ env.VAR }}` |
| String interpolation trong YAML | `${{ env.VAR }}` |

### Q2: Làm sao để truyền biến giữa các jobs?

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      my_output: ${{ steps.step1.outputs.result }}
    steps:
      - id: step1
        run: echo "result=hello" >> $GITHUB_OUTPUT

  job2:
    needs: job1
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ needs.job1.outputs.my_output }}"
```

### Q3: Làm sao để che giấu giá trị trong logs?

```yaml
      - name: Mask value
        run: |
          echo "::add-mask::my-secret-value"
          echo "This contains my-secret-value"  # Sẽ hiển thị ***
```

### Q4: Biến môi trường có phân biệt chữ hoa/thường không?

Có! `APP_NAME` và `app_name` là hai biến khác nhau. Convention là dùng UPPERCASE.

---

## 📚 Tài Liệu Tham Khảo

- [GitHub Actions Environment Variables](https://docs.github.com/en/actions/learn-github-actions/variables)
- [GitHub Actions Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [GitHub Actions Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
- [Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## ✅ Tóm Tắt

| Khái niệm | Mục đích |
|-----------|----------|
| **Workflow-level env** | Biến dùng chung cho toàn bộ workflow |
| **Job-level env** | Biến riêng cho một job |
| **Step-level env** | Biến riêng cho một step |
| **GitHub Context** | Thông tin về repo, event, run, v.v. |
| **Secrets** | Lưu trữ an toàn cho sensitive data |
| **Variables** | Configuration values (non-sensitive) |
| **$GITHUB_ENV** | Set dynamic env cho các step sau |
| **$GITHUB_OUTPUT** | Truyền data giữa steps/jobs |
