# Giải Thích Chi Tiết: 08 - Complete CI/CD Pipeline

## 📋 Tổng Quan

File `08-complete-pipeline.yml` là một **workflow CI/CD hoàn chỉnh** mô phỏng quy trình phát triển phần mềm thực tế với 4 giai đoạn:

1. **Lint** - Kiểm tra chất lượng code
2. **Test** - Chạy unit tests
3. **Build** - Xây dựng ứng dụng
4. **Deploy** - Triển khai lên môi trường production

---

## 📁 Cấu Trúc File

```yaml
name: 08 - Complete CI/CD Pipeline

on:
    push:
        branches: [main]
    pull_request:
        branches: [main]
    workflow_dispatch:

env:
    PYTHON_VERSION: "3.11"
```

### Giải thích từng phần:

### 1. `name: 08 - Complete CI/CD Pipeline`
- **Mục đích**: Đặt tên hiển thị cho workflow trong tab Actions của GitHub
- **Tại sao dùng**: Giúp dễ dàng nhận diện workflow trong danh sách nhiều workflows

### 2. `on:` - Trigger Events

```yaml
on:
    push:
        branches: [main]
    pull_request:
        branches: [main]
    workflow_dispatch:
```

| Trigger | Giải thích | Tại sao dùng |
|---------|------------|--------------|
| `push: branches: [main]` | Chạy khi có code push vào branch main | Đảm bảo code trên main luôn được kiểm tra |
| `pull_request: branches: [main]` | Chạy khi có PR vào main | Kiểm tra code TRƯỚC khi merge |
| `workflow_dispatch` | Cho phép chạy thủ công | Debug, test workflow, hoặc deploy lại |

**❓ Tại sao không dùng các trigger khác?**

| Trigger không dùng | Lý do |
|--------------------|-------|
| `push: branches: ["*"]` | Sẽ chạy trên TẤT CẢ branch → tốn resource, không cần thiết |
| `schedule` | CI/CD nên chạy theo event, không theo lịch |
| `release` | Có thể thêm nếu muốn deploy khi tạo release |

### 3. `env:` - Biến môi trường cấp Workflow

```yaml
env:
    PYTHON_VERSION: "3.11"
```

- **Mục đích**: Định nghĩa biến dùng chung cho TẤT CẢ jobs
- **Tại sao dùng**: 
  - **DRY (Don't Repeat Yourself)**: Chỉ cần thay đổi 1 chỗ khi upgrade Python
  - **Nhất quán**: Tất cả jobs dùng cùng version
- **Cách truy cập**: `${{ env.PYTHON_VERSION }}`

**❓ Tại sao không dùng `strategy.matrix`?**
- Matrix phù hợp khi muốn test NHIỀU versions cùng lúc
- Ở đây chỉ cần 1 version nên dùng env đơn giản hơn

---

## 🔍 STAGE 1: Lint Job

```yaml
lint:
    name: 🔍 Lint
    runs-on: ubuntu-latest
    steps:
        - uses: actions/checkout@v4

        - uses: actions/setup-python@v5
          with:
              python-version: ${{ env.PYTHON_VERSION }}
              cache: "pip"

        - name: Install linters
          run: pip install ruff black

        - name: Run Ruff
          run: ruff check . || true

        - name: Check Black formatting
          run: black --check . || true
```

### Giải thích từng bước:

#### Step 1: `actions/checkout@v4`
```yaml
- uses: actions/checkout@v4
```
- **Mục đích**: Clone source code vào runner
- **Tại sao cần**: Runner bắt đầu là máy ảo trống, không có code của bạn
- **@v4**: Version mới nhất, hỗ trợ sparse checkout, submodules

**❓ Tại sao không dùng `git clone`?**
```yaml
# Không nên dùng:
- run: git clone https://github.com/${{ github.repository }}
```
- Phải xử lý authentication thủ công
- Không tự động checkout đúng commit/branch
- `actions/checkout` đã tối ưu sẵn

#### Step 2: `actions/setup-python@v5`
```yaml
- uses: actions/setup-python@v5
  with:
      python-version: ${{ env.PYTHON_VERSION }}
      cache: "pip"
```

| Option | Giải thích |
|--------|------------|
| `python-version` | Version Python cần cài |
| `cache: "pip"` | Cache thư mục pip để tăng tốc lần chạy sau |

**❓ Tại sao không tự cài Python?**
```yaml
# Không nên:
- run: sudo apt update && sudo apt install python3.11
```
- Chậm hơn (phải compile/download)
- Không có cache
- Khó quản lý multiple versions

**❓ Cache types có thể dùng:**
| Cache | Khi nào dùng |
|-------|--------------|
| `pip` | Project Python thuần |
| `pipenv` | Dùng Pipfile |
| `poetry` | Dùng Poetry |

#### Step 3: Install linters
```yaml
- name: Install linters
  run: pip install ruff black
```

| Tool | Mục đích |
|------|----------|
| `ruff` | Linter cực nhanh (thay thế flake8, isort, pylint) |
| `black` | Code formatter tự động |

**❓ Tại sao chọn Ruff thay vì Flake8?**
| So sánh | Ruff | Flake8 |
|---------|------|--------|
| Tốc độ | 10-100x nhanh hơn | Chậm |
| Tích hợp | Thay thế nhiều tools | Chỉ lint |
| Cấu hình | pyproject.toml | Nhiều file config |

#### Step 4-5: Run linters
```yaml
- name: Run Ruff
  run: ruff check . || true

- name: Check Black formatting
  run: black --check . || true
```

**`|| true` là gì?**
- `||` = OR operator trong shell
- Nếu lệnh trước FAIL → chạy `true` (luôn thành công)
- **Kết quả**: Step LUÔN pass, kể cả khi có lỗi lint

**❓ Tại sao dùng `|| true`?**
- Demo purpose: Không muốn workflow fail hoàn toàn
- **Production**: BỎ `|| true` để workflow fail khi có lỗi lint

```yaml
# Production version:
- name: Run Ruff
  run: ruff check .  # Sẽ fail nếu có lỗi
```

---

## 🧪 STAGE 2: Test Job

```yaml
test:
    name: 🧪 Test
    needs: lint
    runs-on: ubuntu-latest
    steps:
        - uses: actions/checkout@v4

        - uses: actions/setup-python@v5
          with:
              python-version: ${{ env.PYTHON_VERSION }}
              cache: "pip"

        - name: Install dependencies
          run: |
              pip install pytest pytest-cov
              pip install -r requirements.txt || true

        - name: Run tests
          run: pytest --cov --cov-report=xml || echo "No tests"

        - name: Upload coverage
          uses: actions/upload-artifact@v4
          with:
              name: coverage-report
              path: coverage.xml
          if: always()
```

### Phần quan trọng:

#### `needs: lint`
```yaml
needs: lint
```
- **Mục đích**: Job `test` CHỈ chạy SAU khi `lint` hoàn thành thành công
- **Tại sao**: Tạo pipeline tuần tự (Lint → Test → Build → Deploy)

**❓ Các cách define dependencies:**
```yaml
# Một dependency
needs: lint

# Nhiều dependencies
needs: [lint, security-scan]

# Job chạy song song
# Không có needs → chạy song song với jobs khác
```

#### Install dependencies
```yaml
- name: Install dependencies
  run: |
      pip install pytest pytest-cov
      pip install -r requirements.txt || true
```

| Package | Mục đích |
|---------|----------|
| `pytest` | Framework chạy tests |
| `pytest-cov` | Plugin đo coverage |

**`pip install -r requirements.txt || true`**
- Install dependencies từ file requirements.txt
- `|| true`: Không fail nếu file không tồn tại

#### Run tests với coverage
```yaml
- name: Run tests
  run: pytest --cov --cov-report=xml || echo "No tests"
```

| Option | Giải thích |
|--------|------------|
| `--cov` | Đo code coverage |
| `--cov-report=xml` | Xuất report dạng XML (cho CI tools) |
| `|| echo "No tests"` | Không fail nếu không có tests |

**❓ Các format coverage report:**
```yaml
--cov-report=xml      # Cho CI/CD tools
--cov-report=html     # Cho người đọc
--cov-report=term     # In ra terminal
```

#### Upload coverage artifact
```yaml
- name: Upload coverage
  uses: actions/upload-artifact@v4
  with:
      name: coverage-report
      path: coverage.xml
  if: always()
```

- **Mục đích**: Lưu coverage report để xem sau
- **`if: always()`**: Upload KỂ CẢ KHI tests fail

**❓ Tại sao dùng `if: always()`?**
| Điều kiện | Khi nào chạy |
|-----------|--------------|
| `if: success()` (mặc định) | Chỉ khi steps trước pass |
| `if: always()` | Luôn chạy |
| `if: failure()` | Chỉ khi có step fail |
| `if: cancelled()` | Chỉ khi workflow bị cancel |

---

## 🏗️ STAGE 3: Build Job

```yaml
build:
    name: 🏗️ Build
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    outputs:
        image-tag: ${{ steps.meta.outputs.tags }}

    steps:
        - uses: actions/checkout@v4

        - name: Set image tag
          id: meta
          run: |
              echo "tags=myapp:${{ github.sha }}" >> $GITHUB_OUTPUT

        - name: Build info
          run: |
              echo "Would build: ${{ steps.meta.outputs.tags }}"
              echo "Build complete!"
```

### Phần quan trọng:

#### Job-level condition
```yaml
if: github.ref == 'refs/heads/main'
```
- **Mục đích**: TOÀN BỘ job `build` chỉ chạy khi push vào `main`
- **Tại sao**: Không cần build khi đang làm việc trên feature branch

**❓ Các giá trị `github.ref` phổ biến:**
| Ref | Khi nào |
|-----|---------|
| `refs/heads/main` | Push vào branch main |
| `refs/heads/feature-x` | Push vào branch feature-x |
| `refs/pull/123/merge` | Pull request #123 |
| `refs/tags/v1.0.0` | Push tag v1.0.0 |

#### Job outputs
```yaml
outputs:
    image-tag: ${{ steps.meta.outputs.tags }}
```
- **Mục đích**: Xuất giá trị để JOB KHÁC có thể dùng
- **Ở đây**: Xuất image tag để job `deploy` dùng

#### Tạo output trong step
```yaml
- name: Set image tag
  id: meta
  run: |
      echo "tags=myapp:${{ github.sha }}" >> $GITHUB_OUTPUT
```

| Phần | Giải thích |
|------|------------|
| `id: meta` | Đặt ID cho step để reference sau |
| `$GITHUB_OUTPUT` | File đặc biệt để set output |
| `github.sha` | SHA commit hiện tại (ví dụ: `a1b2c3d4...`) |

**❓ Cách set output (GITHUB_OUTPUT vs set-output):**
```yaml
# Cách mới (khuyên dùng):
echo "key=value" >> $GITHUB_OUTPUT

# Cách cũ (deprecated, không dùng):
echo "::set-output name=key::value"
```

#### Sử dụng step output
```yaml
echo "Would build: ${{ steps.meta.outputs.tags }}"
```
- `steps.meta.outputs.tags` = giá trị `tags` từ step có `id: meta`

---

## 🚀 STAGE 4: Deploy Job

```yaml
deploy:
    name: 🚀 Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
        name: production
        url: https://example.com

    steps:
        - name: Deploy
          run: |
              echo "Deploying ${{ needs.build.outputs.image-tag }}"
              echo "Deploy successful! 🎉"
```

### Phần quan trọng:

#### `needs: build`
- Deploy CHỈ chạy SAU khi build thành công

#### Environment
```yaml
environment:
    name: production
    url: https://example.com
```

| Field | Mục đích |
|-------|----------|
| `name: production` | Tên môi trường (hiển thị trong GitHub) |
| `url` | URL của deployment (hiển thị link trong PR/commit) |

**❓ Tại sao dùng Environment?**
1. **Protection rules**: Yêu cầu approval trước khi deploy
2. **Secrets riêng**: Mỗi environment có secrets riêng
3. **Deployment history**: Theo dõi lịch sử deploy
4. **Traceability**: Biết commit nào đang deploy ở đâu

**Cấu hình Protection Rules trong GitHub:**
1. Settings → Environments → production
2. ✅ Required reviewers: Thêm người phải approve
3. ✅ Wait timer: Delay trước khi deploy
4. ✅ Deployment branches: Chỉ cho phép từ main

#### Sử dụng output từ job khác
```yaml
echo "Deploying ${{ needs.build.outputs.image-tag }}"
```
- `needs.build.outputs.image-tag` = output từ job `build`

**Cú pháp:**
```
needs.<job_id>.outputs.<output_name>
```

---

## 🔄 Luồng Pipeline

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  LINT   │ ──▶ │  TEST   │ ──▶ │  BUILD  │ ──▶ │ DEPLOY  │
│  🔍     │     │  🧪     │     │  🏗️     │     │  🚀     │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                                    │               │
                            chỉ trên main     chỉ trên main
```

---

## 📊 So Sánh Các Cách Tiếp Cận

### Job Dependencies

| Pattern | Code | Use case |
|---------|------|----------|
| Sequential | `needs: previous-job` | CI/CD pipeline |
| Parallel | Không có `needs` | Independent checks |
| Diamond | `needs: [job1, job2]` | Merge sau nhiều jobs |

```yaml
# Sequential:  A → B → C
job-b:
  needs: job-a
job-c:
  needs: job-b

# Parallel:    A
#             B   (cùng lúc)
#             C
job-a:
job-b:
job-c:
# Không có needs

# Diamond:    A → C
#             B → C
job-c:
  needs: [job-a, job-b]
```

### Conditional Execution

```yaml
# Job level - toàn bộ job
job:
  if: github.ref == 'refs/heads/main'

# Step level - một step
- run: echo "deploy"
  if: github.event_name == 'push'
```

---

## 🛠️ Mở Rộng Pipeline

### Thêm Security Scan
```yaml
security:
    name: 🔒 Security
    needs: lint
    runs-on: ubuntu-latest
    steps:
        - uses: actions/checkout@v4
        - name: Run Trivy
          uses: aquasecurity/trivy-action@master
          with:
              scan-type: 'fs'
```

### Thêm Matrix Testing
```yaml
test:
    strategy:
        matrix:
            python-version: ["3.10", "3.11", "3.12"]
            os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
        - uses: actions/setup-python@v5
          with:
              python-version: ${{ matrix.python-version }}
```

### Thêm Multiple Environments
```yaml
deploy-staging:
    environment:
        name: staging
        url: https://staging.example.com
    needs: build

deploy-production:
    environment:
        name: production
        url: https://example.com
    needs: deploy-staging  # Deploy staging trước
```

---

## 📚 Các Lệnh/Concepts Liên Quan

### 1. GitHub Context Objects

| Context | Chứa | Ví dụ |
|---------|------|-------|
| `github` | Event, repo info | `github.sha`, `github.ref` |
| `env` | Environment variables | `env.PYTHON_VERSION` |
| `steps` | Outputs từ steps | `steps.meta.outputs.tags` |
| `needs` | Outputs từ jobs | `needs.build.outputs.image-tag` |
| `secrets` | Secrets | `secrets.DEPLOY_KEY` |
| `matrix` | Matrix values | `matrix.os` |

### 2. Expressions

```yaml
# Comparison
if: github.ref == 'refs/heads/main'

# Boolean
if: github.event_name == 'push' && github.ref == 'refs/heads/main'

# Functions
if: contains(github.event.head_commit.message, '[skip ci]')
if: startsWith(github.ref, 'refs/tags/')
if: always()  # Luôn chạy
if: failure() # Chỉ khi fail  
```

### 3. Output Patterns

```yaml
# Set output
echo "name=value" >> $GITHUB_OUTPUT

# Multi-line output
echo "json<<EOF" >> $GITHUB_OUTPUT
cat data.json >> $GITHUB_OUTPUT
echo "EOF" >> $GITHUB_OUTPUT

# Use in same job
${{ steps.step-id.outputs.name }}

# Use in different job (cần job outputs)
${{ needs.job-id.outputs.name }}
```

---

## ✅ Checklist Khi Tạo CI/CD Pipeline

- [ ] 1. Lint/Format check đầu tiên (fail fast)
- [ ] 2. Tests với coverage
- [ ] 3. Build artifacts
- [ ] 4. Deploy với environment protection
- [ ] 5. `needs:` để tạo dependencies đúng
- [ ] 6. Conditions (`if:`) để giới hạn execution
- [ ] 7. `outputs:` để pass data giữa jobs
- [ ] 8. Cache để tăng tốc
- [ ] 9. Artifacts để lưu results
- [ ] 10. Environments với protection rules

---

## 🔗 Tài Liệu Tham Khảo

- [GitHub Actions - Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Actions - Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
- [GitHub Actions - Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [GitHub Actions - Environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
