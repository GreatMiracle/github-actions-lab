# Giải Thích Chi Tiết: 07-use-reusable.yml

## 📋 Tổng Quan

File `07-use-reusable.yml` minh họa cách **sử dụng Reusable Workflows** trong GitHub Actions - một tính năng quan trọng giúp tái sử dụng code, giảm trùng lặp và dễ bảo trì workflow.

---

## 📁 Cấu Trúc File

### File Chính: `07-use-reusable.yml`

```yaml
name: 07 - Use Reusable Workflow

on:
    workflow_dispatch:

jobs:
    test-python-311:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"
        secrets: inherit

    test-python-312:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.12"
        secrets: inherit
```

### File Reusable: `reusable-python-test.yml`

```yaml
name: Reusable Python Test

on:
    workflow_call:
        inputs:
            python-version:
                required: true
                type: string
            working-directory:
                required: false
                type: string
                default: "."
        secrets:
            CODECOV_TOKEN:
                required: false

jobs:
    test:
        runs-on: ubuntu-latest
        defaults:
            run:
                working-directory: ${{ inputs.working-directory }}

        steps:
            - uses: actions/checkout@v4

            - uses: actions/setup-python@v5
              with:
                  python-version: ${{ inputs.python-version }}

            - name: Install and test
              run: |
                  pip install pytest pytest-cov
                  pip install -r requirements.txt || true
                  pytest --cov || echo "No tests found"
```

---

## 🔍 Giải Thích Chi Tiết Từng Phần

### 1. Tên Workflow

```yaml
name: 07 - Use Reusable Workflow
```

| Thành phần | Giải thích |
|------------|------------|
| `name` | Tên hiển thị trên GitHub Actions UI |

**Tại sao dùng cách này?**
- Đặt tên rõ ràng giúp dễ theo dõi trong danh sách workflows
- Prefix số `07` giúp sắp xếp thứ tự học tập

---

### 2. Trigger Event

```yaml
on:
    workflow_dispatch:
```

| Trigger | Mô tả |
|---------|-------|
| `workflow_dispatch` | Cho phép chạy thủ công từ GitHub UI |

**Tại sao dùng `workflow_dispatch`?**
- Phù hợp cho demo/testing
- Cho phép kiểm soát khi nào workflow chạy

**Các trigger khác có thể dùng:**

| Trigger | Khi nào dùng |
|---------|--------------|
| `push` | Tự động chạy khi push code |
| `pull_request` | Chạy khi tạo/cập nhật PR |
| `schedule` | Chạy theo lịch (cron) |
| `workflow_run` | Chạy sau khi workflow khác hoàn thành |

---

### 3. Jobs Sử Dụng Reusable Workflow

#### Job 1: test-python-311

```yaml
jobs:
    test-python-311:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"
        secrets: inherit
```

| Thành phần | Giải thích chi tiết |
|------------|---------------------|
| `test-python-311` | Tên job, tự đặt, mô tả mục đích |
| `uses` | **Quan trọng!** Chỉ định đường dẫn đến reusable workflow |
| `./.github/workflows/reusable-python-test.yml` | Đường dẫn tương đối từ root repository |
| `with` | Truyền các input parameters cho reusable workflow |
| `python-version: "3.11"` | Giá trị truyền vào input `python-version` |
| `secrets: inherit` | **Quan trọng!** Kế thừa tất cả secrets từ workflow cha |

---

### 4. Keyword `uses` - Cách Tham Chiếu Reusable Workflow

```yaml
uses: ./.github/workflows/reusable-python-test.yml
```

**Các cách tham chiếu reusable workflow:**

| Cách dùng | Ví dụ | Khi nào dùng |
|-----------|-------|--------------|
| **Local (cùng repo)** | `./.github/workflows/file.yml` | Workflow cùng repository |
| **Remote (khác repo, cùng org)** | `org/repo/.github/workflows/file.yml@main` | Workflow từ repo khác trong cùng organization |
| **Remote (public repo)** | `owner/repo/.github/workflows/file.yml@v1` | Workflow từ public repository |

**Tại sao dùng đường dẫn tương đối `./`?**
- Đơn giản, không cần version tag
- Workflow luôn lấy version mới nhất từ cùng branch
- Không cần public repository

**Ví dụ tham chiếu từ remote:**

```yaml
# Từ repo khác với tag version
uses: actions/reusable-workflows/.github/workflows/ci.yml@v1

# Từ repo khác với branch
uses: my-org/shared-workflows/.github/workflows/deploy.yml@main

# Từ repo khác với commit SHA
uses: my-org/shared-workflows/.github/workflows/deploy.yml@a1b2c3d4
```

---

### 5. Keyword `with` - Truyền Inputs

```yaml
with:
    python-version: "3.11"
```

**Tại sao dùng `with`?**
- Truyền giá trị động cho reusable workflow
- Cho phép tái sử dụng workflow với các configuration khác nhau

**So sánh với env:**

| Keyword | Mục đích | Scope |
|---------|----------|-------|
| `with` | Truyền inputs cho reusable workflow | Chỉ reusable workflow |
| `env` | Biến môi trường | Steps trong job |

---

### 6. Keyword `secrets: inherit`

```yaml
secrets: inherit
```

**Đây là điểm QUAN TRỌNG!**

| Cách xử lý secrets | Mô tả |
|--------------------|-------|
| `secrets: inherit` | **Kế thừa TẤT CẢ secrets** từ workflow cha |
| `secrets:` (liệt kê cụ thể) | Chỉ truyền secrets được chỉ định |

**Ví dụ truyền secrets cụ thể:**

```yaml
# Cách 1: Kế thừa tất cả (đơn giản)
secrets: inherit

# Cách 2: Truyền từng secret (an toàn hơn, rõ ràng hơn)
secrets:
    CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
    DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

**Khi nào dùng `inherit` vs liệt kê cụ thể?**

| Tình huống | Khuyến nghị |
|------------|-------------|
| Trong cùng repo, tin tưởng | `secrets: inherit` |
| Reusable workflow từ repo khác | Liệt kê cụ thể từng secret |
| Cần kiểm soát chặt | Liệt kê cụ thể từng secret |

---

## 📖 Giải Thích File Reusable: `reusable-python-test.yml`

### 1. Trigger `workflow_call`

```yaml
on:
    workflow_call:
```

**Đây là điểm BẮT BUỘC để tạo reusable workflow!**

| Trigger | Mô tả |
|---------|-------|
| `workflow_call` | Cho phép workflow này được gọi từ workflow khác |

**Không có `workflow_call` = Không thể dùng `uses` để gọi!**

---

### 2. Định Nghĩa Inputs

```yaml
inputs:
    python-version:
        required: true
        type: string
    working-directory:
        required: false
        type: string
        default: "."
```

| Thuộc tính | Giải thích |
|------------|------------|
| `inputs` | Block định nghĩa các tham số đầu vào |
| `required: true` | Bắt buộc phải truyền khi gọi |
| `required: false` | Không bắt buộc, có thể dùng default |
| `type: string` | Kiểu dữ liệu (string, boolean, number) |
| `default: "."` | Giá trị mặc định nếu không truyền |

**Các type hỗ trợ:**

| Type | Ví dụ | Ghi chú |
|------|-------|---------|
| `string` | `"3.11"`, `"production"` | Phổ biến nhất |
| `boolean` | `true`, `false` | Cho flags |
| `number` | `8080`, `5` | Cho port, count, etc. |

---

### 3. Định Nghĩa Secrets

```yaml
secrets:
    CODECOV_TOKEN:
        required: false
```

| Thuộc tính | Giải thích |
|------------|------------|
| `secrets` | Block định nghĩa các secrets cần thiết |
| `required: false` | Secret không bắt buộc |
| `required: true` | Secret bắt buộc phải có |

**Sử dụng secret trong reusable workflow:**

```yaml
# Trong reusable workflow
- name: Upload coverage
  env:
    CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

### 4. Sử Dụng Inputs Trong Steps

```yaml
defaults:
    run:
        working-directory: ${{ inputs.working-directory }}

# ...

- uses: actions/setup-python@v5
  with:
      python-version: ${{ inputs.python-version }}
```

| Syntax | Giải thích |
|--------|------------|
| `${{ inputs.python-version }}` | Truy cập giá trị input được truyền vào |
| `${{ inputs.working-directory }}` | Truy cập input với default value |

---

## 🔄 So Sánh: Có và Không Có Reusable Workflow

### ❌ Không Dùng Reusable Workflow (Code Trùng Lặp)

```yaml
jobs:
    test-python-311:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-python@v5
              with:
                  python-version: "3.11"
            - name: Install and test
              run: |
                  pip install pytest pytest-cov
                  pip install -r requirements.txt || true
                  pytest --cov || echo "No tests found"

    test-python-312:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-python@v5
              with:
                  python-version: "3.12"
            - name: Install and test
              run: |
                  pip install pytest pytest-cov
                  pip install -r requirements.txt || true
                  pytest --cov || echo "No tests found"
```

**Vấn đề:** Code trùng lặp, khó bảo trì, dễ sai sót khi cập nhật.

### ✅ Dùng Reusable Workflow (DRY - Don't Repeat Yourself)

```yaml
jobs:
    test-python-311:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"
        secrets: inherit

    test-python-312:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.12"
        secrets: inherit
```

**Lợi ích:**
- Code ngắn gọn, dễ đọc
- Sửa 1 chỗ, áp dụng cho tất cả
- Dễ test và maintain

---

## 🚀 Các Lệnh Liên Quan Và Mở Rộng

### 1. Outputs Từ Reusable Workflow

Reusable workflow có thể trả về outputs:

```yaml
# Trong reusable-python-test.yml
on:
    workflow_call:
        outputs:
            test-result:
                description: "Kết quả test"
                value: ${{ jobs.test.outputs.result }}

jobs:
    test:
        runs-on: ubuntu-latest
        outputs:
            result: ${{ steps.test-step.outputs.status }}
        steps:
            - id: test-step
              run: echo "status=passed" >> $GITHUB_OUTPUT
```

**Sử dụng output từ workflow gọi:**

```yaml
jobs:
    call-reusable:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"

    use-output:
        needs: call-reusable
        runs-on: ubuntu-latest
        steps:
            - run: echo "Test result: ${{ needs.call-reusable.outputs.test-result }}"
```

---

### 2. Kết Hợp Với Matrix Strategy

```yaml
jobs:
    test-matrix:
        strategy:
            matrix:
                python-version: ["3.10", "3.11", "3.12"]
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: ${{ matrix.python-version }}
        secrets: inherit
```

**Lợi ích:** Gọn gàng hơn, dễ thêm/bớt versions.

---

### 3. Nested Reusable Workflows

Reusable workflow có thể gọi reusable workflow khác (tối đa 4 cấp):

```yaml
# level-1.yml
on:
    workflow_call:

jobs:
    job1:
        uses: ./.github/workflows/level-2.yml
```

---

### 4. Conditional Reusable Workflow

```yaml
jobs:
    test:
        if: github.event_name == 'push'
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"
```

---

## 📊 Bảng Tổng Hợp Keywords

| Keyword | Vị trí | Mục đích |
|---------|--------|----------|
| `workflow_call` | Reusable workflow | Cho phép được gọi từ workflow khác |
| `uses` | Caller workflow | Gọi reusable workflow |
| `with` | Caller workflow | Truyền inputs |
| `inputs` | Reusable workflow | Định nghĩa inputs |
| `secrets` | Cả hai | Xử lý secrets |
| `outputs` | Reusable workflow | Trả về giá trị |

---

## 💡 Best Practices

### 1. Đặt Tên Rõ Ràng
```yaml
# ✅ Tốt
name: Reusable Python Test

# ❌ Không tốt
name: workflow1
```

### 2. Document Inputs
```yaml
inputs:
    python-version:
        description: "Version của Python cần sử dụng"
        required: true
        type: string
```

### 3. Sử Dụng Default Values
```yaml
inputs:
    timeout:
        required: false
        type: number
        default: 10
```

### 4. Validate Inputs
```yaml
steps:
    - name: Validate Python version
      if: inputs.python-version == ''
      run: |
        echo "Error: python-version is required"
        exit 1
```

---

## ⚠️ Lưu Ý Quan Trọng

1. **Giới hạn độ sâu:** Tối đa 4 cấp nested reusable workflows
2. **Giới hạn số lượng:** Tối đa 20 reusable workflows trong một workflow run
3. **Secrets cần khai báo:** Reusable workflow từ public repo khác cần khai báo secrets rõ ràng
4. **Version control:** Khi dùng remote reusable workflow, luôn pin version (tag, SHA)

---

## 🎯 Tùy Biến Cho Dự Án Của Bạn

### Ví dụ 1: Thêm Node.js Testing

```yaml
# reusable-node-test.yml
on:
    workflow_call:
        inputs:
            node-version:
                required: true
                type: string
            package-manager:
                required: false
                type: string
                default: "npm"

jobs:
    test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-node@v4
              with:
                  node-version: ${{ inputs.node-version }}
            - run: ${{ inputs.package-manager }} install
            - run: ${{ inputs.package-manager }} test
```

### Ví dụ 2: Multi-Language Testing

```yaml
# Gọi nhiều reusable workflows
jobs:
    python-test:
        uses: ./.github/workflows/reusable-python-test.yml
        with:
            python-version: "3.11"

    node-test:
        uses: ./.github/workflows/reusable-node-test.yml
        with:
            node-version: "18"
            package-manager: "yarn"
```

---

## 📚 Tài Liệu Tham Khảo

- [GitHub Docs: Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [GitHub Docs: workflow_call trigger](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_call)
- [GitHub Docs: Sharing secrets with reusable workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecretsinherit)
