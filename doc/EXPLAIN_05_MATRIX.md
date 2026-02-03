# Giải Thích Chi Tiết: 05-matrix.yml

## 📌 Tổng Quan

File `05-matrix.yml` minh họa tính năng **Matrix Builds** trong GitHub Actions - một trong những tính năng mạnh mẽ nhất cho phép chạy cùng một job trên nhiều cấu hình khác nhau một cách song song.

---

## 📝 Phân Tích Từng Phần

### 1. Tên Workflow

```yaml
name: 05 - Matrix Builds
```

**Giải thích:**
- `name` định nghĩa tên hiển thị của workflow trên GitHub Actions UI
- Tên này xuất hiện trong tab "Actions" của repository

**Tại sao dùng lệnh này?**
- Đây là cách tiêu chuẩn để đặt tên workflow trong GitHub Actions
- Không có lệnh thay thế nào khác cho mục đích này

---

### 2. Trigger Event

```yaml
on:
    workflow_dispatch:
```

**Giải thích:**
- `on` xác định các sự kiện sẽ kích hoạt workflow chạy
- `workflow_dispatch` cho phép chạy workflow thủ công từ GitHub UI

**Tại sao dùng `workflow_dispatch`?**

| Trigger | Mô tả | Lý do chọn/không chọn |
|---------|-------|----------------------|
| `workflow_dispatch` | Chạy thủ công | ✅ **Được chọn** - Phù hợp cho demo/testing, kiểm soát khi nào workflow chạy |
| `push` | Chạy khi push code | ❌ Không phù hợp vì matrix builds tốn nhiều resources, không nên chạy mỗi lần push |
| `pull_request` | Chạy khi có PR | ❌ Có thể dùng nhưng trong context học tập, `workflow_dispatch` cho phép test linh hoạt hơn |
| `schedule` | Chạy theo lịch | ❌ Không phù hợp cho demo |

---

### 3. Job Definition

```yaml
jobs:
    test-matrix:
        runs-on: ${{ matrix.os }}
```

**Giải thích:**
- `jobs` là container chứa tất cả các job trong workflow
- `test-matrix` là ID của job (có thể đặt tên tùy ý)
- `runs-on: ${{ matrix.os }}` - Runner sẽ được xác định bởi giá trị từ matrix

**Tại sao dùng `${{ matrix.os }}`?**

| Cách viết | Mô tả | Lý do chọn/không chọn |
|-----------|-------|----------------------|
| `${{ matrix.os }}` | Động, lấy từ matrix | ✅ **Được chọn** - Cho phép chạy trên nhiều OS khác nhau |
| `ubuntu-latest` | Cố định một OS | ❌ Không tận dụng được matrix, chỉ chạy trên 1 OS |
| `self-hosted` | Runner tự host | ❌ Không cần thiết cho demo |

---

### 4. Strategy & Matrix Configuration

```yaml
strategy:
    fail-fast: false
    matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]
```

#### 4.1 `strategy`
**Giải thích:**
- `strategy` định nghĩa cách GitHub Actions xử lý việc chạy nhiều instances của job

#### 4.2 `fail-fast: false`

**Giải thích:**
- Khi một job trong matrix fails, các job khác **vẫn tiếp tục chạy**

**Tại sao dùng `fail-fast: false`?**

| Giá trị | Hành vi | Lý do chọn/không chọn |
|---------|---------|----------------------|
| `false` | Tiếp tục chạy tất cả jobs | ✅ **Được chọn** - Cho phép xem tất cả kết quả test, biết được version nào pass/fail |
| `true` (default) | Dừng tất cả jobs khi 1 job fail | ❌ Tiết kiệm resources nhưng không biết được trạng thái của các combination khác |

**Ví dụ thực tế:**
- Nếu Python 3.10 trên Ubuntu fail, bạn vẫn muốn biết Python 3.11, 3.12 có pass không
- Điều này quan trọng khi debug compatibility issues

#### 4.3 `matrix`

**Giải thích:**
- Định nghĩa các biến và giá trị để tạo ra ma trận các jobs
- GitHub Actions sẽ tự động tạo ra **tất cả các combinations** có thể

**Ma trận được tạo ra:**

| # | OS | Python Version |
|---|-----|----------------|
| 1 | ubuntu-latest | 3.10 |
| 2 | ubuntu-latest | 3.11 |
| 3 | ubuntu-latest | 3.12 |
| 4 | windows-latest | 3.10 |
| 5 | windows-latest | 3.11 |
| 6 | windows-latest | 3.12 |

**Tổng cộng: 2 OS × 3 Python versions = 6 jobs**

**Tại sao dùng array syntax `[]`?**

| Cách viết | Mô tả | Lý do chọn/không chọn |
|-----------|-------|----------------------|
| `[value1, value2]` | Inline array | ✅ **Được chọn** - Ngắn gọn, dễ đọc cho danh sách nhỏ |
| YAML list với `-` | Multi-line array | Có thể dùng, nhưng dài hơn |

**Tại sao Python version trong quotes `"3.10"`?**
- **QUAN TRỌNG:** `3.10` không có quotes sẽ được YAML parse thành số `3.1` (bỏ số 0)
- `"3.10"` đảm bảo giữ nguyên string "3.10"

---

### 5. Exclude Configuration

```yaml
exclude:
    - os: windows-latest
      python-version: "3.10"
```

**Giải thích:**
- `exclude` loại bỏ các combinations cụ thể khỏi matrix
- Ở đây: **KHÔNG chạy** Python 3.10 trên Windows

**Ma trận sau khi exclude:**

| # | OS | Python Version | Status |
|---|-----|----------------|--------|
| 1 | ubuntu-latest | 3.10 | ✅ Chạy |
| 2 | ubuntu-latest | 3.11 | ✅ Chạy |
| 3 | ubuntu-latest | 3.12 | ✅ Chạy |
| 4 | windows-latest | 3.10 | ❌ **Bị exclude** |
| 5 | windows-latest | 3.11 | ✅ Chạy |
| 6 | windows-latest | 3.12 | ✅ Chạy |

**Tổng cộng: 5 jobs (thay vì 6)**

**Tại sao dùng `exclude`?**

| Tính năng | Mô tả | Lý do chọn/không chọn |
|-----------|-------|----------------------|
| `exclude` | Loại bỏ combinations | ✅ **Được chọn** - Linh hoạt, dễ maintain khi có nhiều values |
| Không khai báo combination | Tự quản lý matrix | ❌ Phức tạp hơn, dễ sai sót |

**Use cases thực tế cho `exclude`:**
- Python version không hỗ trợ trên OS cụ thể
- Combination đã biết có bug
- Tiết kiệm resources cho combinations không quan trọng

---

### 6. Include Configuration

```yaml
include:
    - os: ubuntu-latest
      python-version: "3.12"
      experimental: true
```

**Giải thích:**
- `include` thêm hoặc mở rộng combinations trong matrix
- Có thể thêm **biến mới** (`experimental`) cho combination cụ thể
- Ở đây: Thêm flag `experimental: true` cho Ubuntu + Python 3.12

**Tại sao dùng `include`?**

| Tính năng | Mô tả | Lý do chọn/không chọn |
|-----------|-------|----------------------|
| `include` | Thêm/mở rộng combinations | ✅ **Được chọn** - Cho phép thêm metadata riêng cho từng combination |
| Thêm trực tiếp vào matrix | Không linh hoạt | ❌ Không thể có biến khác nhau cho từng combination |

**Use cases thực tế cho `include`:**
- Đánh dấu experimental builds (có thể allow failure)
- Thêm extra configuration cho platform cụ thể
- Thêm completely new combination không nằm trong cartesian product

**Lưu ý quan trọng:**
- `include` ở đây **KHÔNG tạo job mới** vì combination `ubuntu-latest + 3.12` đã tồn tại
- Nó chỉ **thêm biến `experimental`** vào combination đó

---

### 7. Steps

#### 7.1 Checkout Code

```yaml
steps:
    - uses: actions/checkout@v4
```

**Giải thích:**
- Checkout source code từ repository về runner

**Tại sao dùng `actions/checkout@v4`?**

| Action | Mô tả | Lý do chọn/không chọn |
|--------|-------|----------------------|
| `actions/checkout@v4` | Official GitHub action | ✅ **Được chọn** - Stable, được maintain bởi GitHub |
| `actions/checkout@v3` | Phiên bản cũ hơn | ❌ Nên dùng version mới nhất |
| `git clone` command | Dùng lệnh trực tiếp | ❌ Phức tạp hơn, không có optimizations |

---

#### 7.2 Setup Python

```yaml
- name: 🐍 Setup Python ${{ matrix.python-version }}
  uses: actions/setup-python@v5
  with:
      python-version: ${{ matrix.python-version }}
```

**Giải thích:**
- Cài đặt Python với version từ matrix
- `${{ matrix.python-version }}` lấy giá trị động từ matrix

**Tại sao dùng `actions/setup-python@v5`?**

| Phương pháp | Mô tả | Lý do chọn/không chọn |
|-------------|-------|----------------------|
| `actions/setup-python@v5` | Official action | ✅ **Được chọn** - Tự động cache, cross-platform, reliable |
| `apt-get install python` | Cài thủ công | ❌ Chỉ hoạt động trên Linux, không linh hoạt version |
| `pyenv` | Version manager | ❌ Phức tạp hơn, cần cài đặt thêm |

**Tại sao dùng `@v5`?**
- Là phiên bản mới nhất, stable
- Hỗ trợ tốt hơn cho Python 3.12+

---

#### 7.3 Show Info

```yaml
- name: 📋 Show Info
  run: |
      echo "OS: ${{ matrix.os }}"
      echo "Python: ${{ matrix.python-version }}"
      echo "Experimental: ${{ matrix.experimental }}"
      python --version
```

**Giải thích:**
- In ra thông tin về môi trường hiện tại
- Sử dụng các biến từ matrix để xác nhận configuration

**Tại sao dùng `run` với `|` (multi-line)?**

| Cách viết | Mô tả | Lý do chọn/không chọn |
|-----------|-------|----------------------|
| `run: \|` | Multi-line block | ✅ **Được chọn** - Dễ đọc khi có nhiều commands |
| `run: "cmd1 && cmd2"` | Single line | ❌ Khó đọc khi có nhiều commands |
| Nhiều `run` steps riêng | Mỗi command 1 step | ❌ Verbose, không cần thiết cho logging đơn giản |

**Về `${{ matrix.experimental }}`:**
- Chỉ có giá trị `true` cho combination Ubuntu + Python 3.12
- Các combinations khác sẽ hiển thị empty string

---

## 🔄 Luồng Thực Thi

```
┌─────────────────────────────────────────────────────────────────┐
│                    Workflow: 05 - Matrix Builds                 │
├─────────────────────────────────────────────────────────────────┤
│  Trigger: workflow_dispatch (Manual)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Matrix Expansion                      │   │
│  │                                                         │   │
│  │  OS: [ubuntu-latest, windows-latest]                    │   │
│  │  Python: ["3.10", "3.11", "3.12"]                       │   │
│  │  Exclude: windows-latest + 3.10                         │   │
│  │  Include: ubuntu-latest + 3.12 + experimental: true     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              5 Jobs Run in Parallel                      │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │                                                          │  │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────────────┐   │  │
│  │  │ Ubuntu     │ │ Ubuntu     │ │ Ubuntu              │   │  │
│  │  │ Python 3.10│ │ Python 3.11│ │ Python 3.12         │   │  │
│  │  │            │ │            │ │ experimental: true  │   │  │
│  │  └────────────┘ └────────────┘ └────────────────────┘   │  │
│  │                                                          │  │
│  │  ┌────────────┐ ┌────────────┐                          │  │
│  │  │ Windows    │ │ Windows    │                          │  │
│  │  │ Python 3.11│ │ Python 3.12│                          │  │
│  │  └────────────┘ └────────────┘                          │  │
│  │                                                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 So Sánh Với Các Alternatives

### Không dùng Matrix (Cách truyền thống)

```yaml
# ❌ KHÔNG NÊN - Repetitive và khó maintain
jobs:
  test-ubuntu-310:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: "3.10"
      # ... duplicate steps

  test-ubuntu-311:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      # ... duplicate steps

  # ... và nhiều jobs khác
```

**Vấn đề:**
- Code duplication
- Khó thêm/bớt versions
- Dễ sai sót khi update

### Dùng Matrix (Best Practice)

```yaml
# ✅ NÊN DÙNG - Clean và maintainable
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    python-version: ["3.10", "3.11", "3.12"]
```

**Ưu điểm:**
- DRY (Don't Repeat Yourself)
- Dễ thêm/bớt versions
- Tự động tạo tất cả combinations

---

## 💡 Best Practices

1. **Luôn dùng quotes cho versions:** `"3.10"` thay vì `3.10`
2. **Sử dụng `fail-fast: false`** khi muốn xem tất cả kết quả
3. **Dùng `exclude`** để bỏ các combinations không hỗ trợ
4. **Dùng `include`** để thêm metadata cho combinations cụ thể
5. **Giới hạn matrix size** để tránh quá nhiều jobs chạy song song

---

## 🔗 Tài Liệu Tham Khảo

- [GitHub Actions Matrix Strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
- [actions/setup-python](https://github.com/actions/setup-python)
- [actions/checkout](https://github.com/actions/checkout)
