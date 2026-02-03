# 📖 Giải Thích Chi Tiết: 01-hello-world.yml

## 📋 Tổng Quan

File `01-hello-world.yml` là một workflow GitHub Actions cơ bản nhất, được thiết kế để giúp người mới làm quen với cấu trúc và cách hoạt động của GitHub Actions. Đây là "Hello World" của thế giới CI/CD.

---

## 🔍 Phân Tích Từng Dòng

### 1. Comment và Tên Workflow

```yaml
# File: .github/workflows/01-hello-world.yml

name: 01 - Hello World
```

#### Giải thích:
- **`#`**: Ký hiệu comment trong YAML. Dòng này chỉ mang tính chất ghi chú, không ảnh hưởng đến việc thực thi.
- **`name`**: Tên hiển thị của workflow trên giao diện GitHub Actions.

#### Tại sao dùng `name`?
- **Có `name`**: Workflow hiển thị tên đẹp và dễ hiểu trên UI GitHub.
- **Không có `name`**: GitHub sẽ lấy tên file làm tên workflow (ví dụ: `01-hello-world.yml`), ít trực quan hơn.

#### Các lựa chọn khác:
| Thuộc tính | Mô tả | Sử dụng |
|------------|-------|---------|
| `name` | Tên hiển thị | Bắt buộc nên có |
| Không có | Dùng tên file | Không khuyến khích |

---

### 2. Trigger Events (Sự Kiện Kích Hoạt)

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

#### Giải thích chi tiết:

##### `on:` - Định nghĩa khi nào workflow chạy

Đây là phần **quan trọng nhất** để xác định workflow sẽ được kích hoạt trong trường hợp nào.

##### `push:` - Kích hoạt khi có code được push

```yaml
push:
  branches: [main]
```

- **`push`**: Sự kiện xảy ra khi code được push lên repository.
- **`branches: [main]`**: Chỉ chạy khi push vào nhánh `main`.

##### `workflow_dispatch:` - Cho phép chạy thủ công

- Thêm nút "Run workflow" trên giao diện GitHub.
- Rất hữu ích để test workflow mà không cần push code.

#### Tại sao dùng cách này?

| Trigger | Khi nào dùng | Ưu điểm | Nhược điểm |
|---------|--------------|---------|------------|
| `push` | Muốn tự động chạy khi có code mới | Tự động hóa hoàn toàn | Có thể chạy nhiều lần không cần thiết |
| `pull_request` | Kiểm tra PR trước khi merge | Đảm bảo chất lượng code | Chỉ chạy khi có PR |
| `workflow_dispatch` | Debug, test thủ công | Linh hoạt, kiểm soát được | Phải bấm tay |
| `schedule` | Chạy theo lịch (cron) | Tự động theo thời gian | Không phản hồi ngay với code changes |

#### Các trigger phổ biến khác:

```yaml
# Ví dụ các trigger khác có thể dùng:

on:
  # 1. Chạy khi có Pull Request
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize, reopened]
  
  # 2. Chạy theo lịch (mỗi ngày lúc 2:00 AM UTC)
  schedule:
    - cron: '0 2 * * *'
  
  # 3. Chạy khi release được tạo
  release:
    types: [published]
  
  # 4. Chạy khi có thay đổi ở paths cụ thể
  push:
    paths:
      - 'src/**'
      - '!src/tests/**'  # Loại trừ thư mục tests
  
  # 5. Chạy khi có issue được tạo
  issues:
    types: [opened, labeled]
```

---

### 3. Jobs Definition (Định Nghĩa Công Việc)

```yaml
jobs:
  hello:
    name: Say Hello
    runs-on: ubuntu-latest
```

#### Giải thích chi tiết:

##### `jobs:` - Container chứa tất cả các job

- Một workflow có thể có **nhiều jobs**.
- Mặc định, các jobs chạy **song song** (parallel).
- Có thể cấu hình chạy **tuần tự** bằng `needs`.

##### `hello:` - ID của job

- Đây là **identifier** duy nhất cho job.
- Dùng để tham chiếu trong các job khác (dependency).
- **Quy tắc đặt tên**: chữ thường, không dấu cách, có thể dùng `-` hoặc `_`.

##### `name: Say Hello` - Tên hiển thị của job

- Tương tự `name` của workflow, đây là tên đẹp hiển thị trên UI.

##### `runs-on: ubuntu-latest` - Môi trường chạy

Đây là **runner** - máy ảo sẽ thực thi job.

#### Tại sao chọn `ubuntu-latest`?

| Runner | Khi nào dùng | Ưu điểm | Nhược điểm |
|--------|--------------|---------|------------|
| `ubuntu-latest` | Đa số trường hợp | Miễn phí nhiều phút nhất, phổ biến | Không có Windows/macOS tools |
| `ubuntu-22.04` | Cần version cụ thể | Ổn định, không thay đổi | Có thể outdated |
| `windows-latest` | Build .NET, PowerShell | Có Windows tools | Tốn phút hơn Linux |
| `macos-latest` | Build iOS, macOS apps | Có Xcode | Tốn phút nhiều nhất |
| `self-hosted` | Cần phần cứng đặc biệt | Tùy chỉnh hoàn toàn | Phải tự quản lý |

#### So sánh chi phí (GitHub-hosted runners):

| Runner | Phút miễn phí/tháng | Tỷ lệ tiêu thụ |
|--------|---------------------|---------------|
| Linux | 2,000 | 1x |
| Windows | 2,000 | 2x (tốn gấp đôi) |
| macOS | 2,000 | 10x (tốn gấp 10) |

---

### 4. Steps (Các Bước Thực Thi)

#### Step 1: Hello World

```yaml
- name: 👋 Hello World
  run: echo "Hello, GitHub Actions!"
```

##### Giải thích:
- **`- name:`**: Mỗi step bắt đầu bằng dấu `-` (YAML array).
- **`run:`**: Thực thi lệnh shell trực tiếp.
- **`echo`**: Lệnh in ra console (stdout).

##### Tại sao dùng `run` mà không dùng `uses`?

| Keyword | Khi nào dùng | Ví dụ |
|---------|--------------|-------|
| `run` | Chạy lệnh shell đơn giản | `run: echo "Hello"` |
| `uses` | Dùng action có sẵn | `uses: actions/checkout@v4` |

```yaml
# Ví dụ sự khác biệt:

# Dùng run - chạy lệnh shell
- name: Clone repo manually
  run: git clone https://github.com/my/repo.git

# Dùng uses - dùng action (khuyến khích)
- name: Clone repo with action
  uses: actions/checkout@v4
```

---

#### Step 2: Print Date

```yaml
- name: 📅 Print Date
  run: date
```

##### Giải thích:
- **`date`**: Lệnh Linux hiển thị ngày giờ hiện tại.
- Hữu ích để debug, biết workflow chạy lúc nào.

##### Các lệnh tương tự:

```bash
# Các cách khác để lấy thời gian:
date                          # Mặc định: Tue Feb  3 08:40:51 UTC 2026
date +"%Y-%m-%d %H:%M:%S"     # Format: 2026-02-03 08:40:51
date -u                       # UTC time
TZ="Asia/Ho_Chi_Minh" date    # Theo timezone Việt Nam
```

---

#### Step 3: Print System Info (Multi-line Command)

```yaml
- name: 💻 Print System Info
  run: |
    echo "OS: $(uname -a)"
    echo "User: $(whoami)"
    echo "Directory: $(pwd)"
```

##### Giải thích:
- **`run: |`**: Ký hiệu `|` cho phép viết **multi-line script**.
- **`$(...)`**: Command substitution - chạy lệnh bên trong và lấy output.
- **`uname -a`**: Thông tin hệ điều hành.
- **`whoami`**: User đang chạy (thường là `runner`).
- **`pwd`**: Thư mục hiện tại.

##### Tại sao dùng `|` (pipe)?

| Ký hiệu | Tên gọi | Hành vi | Khi nào dùng |
|---------|---------|---------|--------------|
| `|` | Literal Block | Giữ nguyên xuống dòng | Multi-line scripts |
| `>` | Folded Block | Gộp thành 1 dòng | Text dài nhưng là 1 lệnh |
| `|-` | Literal Strip | Như `|` + bỏ newline cuối | Khi cần output sạch |
| `>-` | Folded Strip | Như `>` + bỏ newline cuối | Khi cần gộp + sạch |

```yaml
# Ví dụ so sánh:

# Dùng | - mỗi echo là 1 dòng riêng
run: |
  echo "Line 1"
  echo "Line 2"

# Dùng > - tất cả thành 1 dòng (KHÔNG DÙNG cho multi-command)
run: >
  echo "This will
  become one line"

# Chạy nhiều lệnh trên 1 dòng (alternative)
run: echo "Line 1" && echo "Line 2"
```

---

#### Step 4: Check Python Version

```yaml
- name: 🐍 Check Python Version
  run: python3 --version
```

##### Giải thích:
- **`python3 --version`**: Kiểm tra phiên bản Python đã cài sẵn.
- Ubuntu runner có sẵn Python, Node.js, và nhiều tools khác.

##### Tools có sẵn trên ubuntu-latest:

| Tool | Có sẵn | Cách kiểm tra |
|------|--------|---------------|
| Python 3 | ✅ | `python3 --version` |
| Node.js | ✅ | `node --version` |
| npm | ✅ | `npm --version` |
| Git | ✅ | `git --version` |
| Docker | ✅ | `docker --version` |
| Java | ✅ | `java --version` |
| Go | ✅ | `go version` |

---

## 🎯 Các Khái Niệm Quan Trọng Liên Quan

### 1. Cấu Trúc Thư Mục

```
.github/
└── workflows/
    ├── 01-hello-world.yml    # Workflow files phải ở đây
    ├── ci.yml
    └── deploy.yml
```

- Workflow files **bắt buộc** phải ở `.github/workflows/`.
- Extension: `.yml` hoặc `.yaml`.

### 2. YAML Syntax Cơ Bản

```yaml
# Key-Value
name: My Workflow

# Nested (thụt lề 2 spaces)
jobs:
  build:
    runs-on: ubuntu-latest

# Array (dấu -)
steps:
  - name: Step 1
  - name: Step 2

# Inline Array
branches: [main, develop]

# Multi-line String
run: |
  line 1
  line 2
```

### 3. Environment Variables

```yaml
# Định nghĩa env ở workflow level
env:
  MY_VAR: global_value

jobs:
  hello:
    # Định nghĩa env ở job level
    env:
      JOB_VAR: job_value
    
    steps:
      - name: Use env
        # Định nghĩa env ở step level
        env:
          STEP_VAR: step_value
        run: |
          echo "Global: $MY_VAR"
          echo "Job: $JOB_VAR"
          echo "Step: $STEP_VAR"
```

### 4. Context và Expressions

```yaml
# Sử dụng GitHub context
- name: Show context
  run: |
    echo "Repository: ${{ github.repository }}"
    echo "Branch: ${{ github.ref_name }}"
    echo "Commit: ${{ github.sha }}"
    echo "Actor: ${{ github.actor }}"
    echo "Event: ${{ github.event_name }}"
```

---

## 🔧 Cách Tùy Biến Workflow Này

### Thêm điều kiện chạy (Conditional)

```yaml
steps:
  - name: Only on main
    if: github.ref == 'refs/heads/main'
    run: echo "This is main branch"
  
  - name: Only on PR
    if: github.event_name == 'pull_request'
    run: echo "This is a pull request"
```

### Thêm timeout

```yaml
jobs:
  hello:
    runs-on: ubuntu-latest
    timeout-minutes: 10  # Job tự động fail sau 10 phút
```

### Thêm environment

```yaml
jobs:
  hello:
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://my-app.com
```

### Chạy nhiều jobs tuần tự

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test:
    needs: build  # Chờ build xong mới chạy
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."
  
  deploy:
    needs: [build, test]  # Chờ cả build và test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

---

## ❓ FAQ (Câu Hỏi Thường Gặp)

### Q: Workflow không chạy khi push?
**A**: Kiểm tra:
1. File phải ở `.github/workflows/`
2. Branch phải match với trigger (`branches: [main]`)
3. YAML syntax phải đúng

### Q: Làm sao debug workflow?
**A**: 
1. Dùng `workflow_dispatch` để chạy thủ công
2. Thêm các step `echo` để in biến
3. Xem logs trên tab Actions của GitHub

### Q: Tại sao dùng `ubuntu-latest` thay vì version cụ thể?
**A**: 
- `ubuntu-latest` = luôn dùng version mới nhất
- `ubuntu-22.04` = version cố định, ổn định hơn cho production

---

## 📚 Tài Liệu Tham Khảo

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
- [GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners)

---

## ✅ Checklist Học Tập

Sau khi đọc xong, bạn nên hiểu:

- [ ] Cấu trúc cơ bản của một workflow file
- [ ] Các trigger events phổ biến (`push`, `pull_request`, `workflow_dispatch`)
- [ ] Cách định nghĩa jobs và steps
- [ ] Sự khác biệt giữa `run` và `uses`
- [ ] Cách viết multi-line commands với `|`
- [ ] Các runners có sẵn và khi nào dùng từng loại
- [ ] Cách thêm điều kiện và dependency giữa các jobs
