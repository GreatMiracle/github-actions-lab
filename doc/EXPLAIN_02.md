# Giải Thích Chi Tiết: 02-triggers.yml

## 📌 Tổng Quan

File `02-triggers.yml` minh họa các loại **Triggers (Events)** trong GitHub Actions - đây là phần quan trọng nhất quyết định **KHI NÀO** workflow sẽ được kích hoạt chạy.

GitHub Actions hỗ trợ hơn 30 loại events khác nhau, và file này demo 4 loại phổ biến nhất.

---

## 📝 Phân Tích Từng Phần

### 1. Tên Workflow

```yaml
name: 02 - Triggers Demo
```

**Giải thích:**
- Tên hiển thị trên GitHub Actions UI
- Nên đặt tên mô tả rõ mục đích của workflow

---

## 🔔 PHẦN 1: PUSH TRIGGER

```yaml
on:
  push:
    branches: 
      - main
      - 'feature/**'
    paths:
      - 'src/**'
      - '!src/**/*.md'
```

### 1.1. `push` Event

**Giải thích:**
- Workflow sẽ chạy khi có code được push lên repository

**Tại sao dùng `push`?**

| Event | Mô tả | Use Case |
|-------|-------|----------|
| `push` | Khi push commits | ✅ **CI/CD phổ biến nhất** - Build, test code mới |
| `pull_request` | Khi tạo/update PR | Review code trước khi merge |
| `workflow_dispatch` | Manual trigger | Testing, deployment thủ công |

### 1.2. `branches` Filter

```yaml
branches: 
  - main
  - 'feature/**'
```

**Giải thích:**
- Chỉ trigger khi push vào các branches được chỉ định
- `main` - Branch chính xác tên "main"
- `'feature/**'` - Tất cả branches bắt đầu bằng `feature/` (glob pattern)

**Tại sao dùng array syntax?**

| Cách viết | Ví dụ | Khi nào dùng |
|-----------|-------|--------------|
| Inline `[]` | `branches: [main, develop]` | Danh sách ngắn, 1-3 items |
| Multi-line `-` | `branches:` <br> `  - main` <br> `  - develop` | ✅ **Được chọn** - Dễ đọc, dễ comment |

**Các Pattern có thể dùng:**

| Pattern | Ý nghĩa | Matches |
|---------|---------|---------|
| `main` | Chính xác | `main` |
| `feature/**` | Tất cả sub-paths | `feature/login`, `feature/api/users` |
| `feature/*` | Chỉ 1 level | `feature/login` (KHÔNG match `feature/api/users`) |
| `release-[0-9]+` | Character class | `release-1`, `release-23` |
| `!hotfix/**` | Exclude (phủ định) | Loại trừ các branch bắt đầu bằng `hotfix/` |

**Tại sao không dùng `branches-ignore`?**

```yaml
# Alternative 1: branches (whitelist)
branches:
  - main
  - 'feature/**'

# Alternative 2: branches-ignore (blacklist)
branches-ignore:
  - 'wip/**'
  - 'temp/**'
```

| Approach | Ý nghĩa | Khi nào dùng |
|----------|---------|--------------|
| `branches` | Whitelist - CHỈ các branch này | ✅ **Được chọn** - Kiểm soát chặt chẽ hơn |
| `branches-ignore` | Blacklist - TẤT CẢ NGOẠI TRỪ | Khi muốn chạy trên hầu hết branches |

⚠️ **QUAN TRỌNG:** Không thể dùng cả `branches` VÀ `branches-ignore` cùng lúc!

### 1.3. `paths` Filter

```yaml
paths:
  - 'src/**'
  - '!src/**/*.md'
```

**Giải thích:**
- `src/**` - Chỉ trigger khi có thay đổi trong folder `src/`
- `!src/**/*.md` - NGOẠI TRỪ các file `.md` trong `src/`

**Tại sao dùng `paths`?**

| Scenario | Có dùng `paths` | Kết quả |
|----------|-----------------|---------|
| Chỉ sửa README.md | Không filter | Workflow chạy (lãng phí) |
| Chỉ sửa README.md | ✅ `paths: ['src/**']` | Workflow KHÔNG chạy (tiết kiệm) |
| Sửa src/app.py | ✅ `paths: ['src/**']` | Workflow chạy ✓ |

**Các Pattern paths thường dùng:**

```yaml
# Chỉ khi thay đổi code Python
paths:
  - '**.py'

# Chỉ khi thay đổi trong src hoặc tests
paths:
  - 'src/**'
  - 'tests/**'

# Loại trừ documentation
paths:
  - '**'
  - '!docs/**'
  - '!**.md'

# Chỉ khi thay đổi Dockerfile
paths:
  - '**/Dockerfile'
  - 'docker-compose.yml'
```

**Alternative: `paths-ignore`**

```yaml
# Thay vì liệt kê paths cần trigger
paths:
  - 'src/**'

# Có thể liệt kê paths KHÔNG cần trigger
paths-ignore:
  - 'docs/**'
  - '**.md'
  - '.gitignore'
```

| Approach | Khi nào dùng |
|----------|--------------|
| `paths` | Khi chỉ quan tâm một vài folders cụ thể |
| `paths-ignore` | Khi muốn trigger cho hầu hết files |

---

## 🔔 PHẦN 2: PULL REQUEST TRIGGER

```yaml
pull_request:
  branches: [main]
  types: [opened, synchronize, reopened]
```

### 2.1. `pull_request` Event

**Giải thích:**
- Trigger khi có Pull Request tới các branches được chỉ định

**Tại sao dùng `pull_request`?**

| Event | Mô tả | Use Case |
|-------|-------|----------|
| `pull_request` | PR events (có security restrictions) | ✅ **Code review, testing PRs từ forks** |
| `pull_request_target` | PR events (runs in base repo context) | Advanced: workflows cần write access |

### 2.2. `branches: [main]`

**Giải thích:**
- Chỉ trigger cho PRs targeting branch `main`
- PRs vào `develop`, `feature/*` sẽ KHÔNG trigger workflow này

### 2.3. `types` - Activity Types

```yaml
types: [opened, synchronize, reopened]
```

**Giải thích:**
- `opened` - Khi PR được tạo mới
- `synchronize` - Khi có commits mới push vào PR (update PR)
- `reopened` - Khi PR bị closed được mở lại

**Tại sao dùng 3 types này?**

Đây là **default types** của `pull_request` event. Nếu không specify, GitHub Actions sẽ dùng đúng 3 types này!

```yaml
# Hai cách viết này TƯƠNG ĐƯƠNG:
pull_request:
  branches: [main]

pull_request:
  branches: [main]
  types: [opened, synchronize, reopened]
```

**Tại sao viết explicit?**
- ✅ Rõ ràng, dễ hiểu cho người đọc
- ✅ Dễ thêm/bớt types sau này
- ✅ Best practice cho team collaboration

**Tất cả Activity Types của `pull_request`:**

| Type | Mô tả | Khi nào cần |
|------|-------|-------------|
| `opened` | PR được tạo | ✅ Default, hầu hết cần |
| `synchronize` | PR có commits mới | ✅ Default, re-test khi update |
| `reopened` | PR được mở lại | ✅ Default |
| `closed` | PR bị đóng | Cleanup, notification |
| `assigned` | PR được assign | Notification workflows |
| `unassigned` | PR bỏ assign | Notification workflows |
| `labeled` | Label được thêm | Conditional workflows |
| `unlabeled` | Label bị xóa | Conditional workflows |
| `edited` | Title/body được sửa | Update tracking |
| `ready_for_review` | Draft → Ready | Start reviews |
| `converted_to_draft` | Ready → Draft | Pause reviews |
| `review_requested` | Request review | Notify reviewers |

**Ví dụ thực tế:**

```yaml
# Chỉ chạy khi PR ready for review (không chạy cho draft PRs)
pull_request:
  types: [opened, synchronize, reopened, ready_for_review]

# Deploy preview khi PR opened, cleanup khi closed
pull_request:
  types: [opened, closed]
```

---

## 🔔 PHẦN 3: SCHEDULE TRIGGER

```yaml
schedule:
  - cron: '0 2 * * *'
```

### 3.1. `schedule` Event

**Giải thích:**
- Trigger workflow theo lịch định kỳ
- Sử dụng cron syntax chuẩn POSIX

### 3.2. Cron Syntax

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
│ │ │ │ │
* * * * *
```

**`'0 2 * * *'` có nghĩa:**
- Minute: `0` (phút 0)
- Hour: `2` (2 giờ sáng)
- Day of month: `*` (mọi ngày)
- Month: `*` (mọi tháng)
- Day of week: `*` (mọi ngày trong tuần)

→ **Chạy lúc 2:00 AM UTC mỗi ngày**

**Các ví dụ cron thường dùng:**

| Cron Expression | Ý nghĩa |
|-----------------|---------|
| `'0 2 * * *'` | 2:00 AM UTC mỗi ngày |
| `'0 0 * * *'` | Nửa đêm UTC mỗi ngày |
| `'0 */6 * * *'` | Mỗi 6 giờ (0:00, 6:00, 12:00, 18:00) |
| `'0 0 * * 0'` | Mỗi Chủ Nhật lúc nửa đêm |
| `'0 0 1 * *'` | Ngày 1 mỗi tháng lúc nửa đêm |
| `'*/15 * * * *'` | Mỗi 15 phút |
| `'0 9 * * 1-5'` | 9:00 AM các ngày trong tuần (Mon-Fri) |

**Tại sao dùng `schedule`?**

| Use Case | Ví dụ |
|----------|-------|
| Nightly builds | Build và test mỗi đêm |
| Data sync | Sync data từ external sources |
| Report generation | Tạo báo cáo định kỳ |
| Dependency updates | Check updates mỗi tuần |
| Cleanup jobs | Xóa artifacts, caches cũ |

**⚠️ Lưu ý quan trọng:**

1. **Timezone:** Luôn là **UTC**, không thể thay đổi
2. **Guaranteed execution:** KHÔNG đảm bảo chạy đúng giờ (có thể delay khi GitHub load cao)
3. **Default branch only:** Schedule chỉ chạy trên **default branch** (thường là `main`)
4. **Minimum interval:** Tối thiểu 5 phút giữa các lần chạy

**Multiple schedules:**

```yaml
schedule:
  - cron: '0 2 * * *'   # Mỗi ngày 2:00 AM
  - cron: '0 14 * * 5'  # Mỗi thứ 6 lúc 2:00 PM
```

---

## 🔔 PHẦN 4: WORKFLOW_DISPATCH TRIGGER

```yaml
workflow_dispatch:
  inputs:
    greeting:
      description: 'Your greeting message'
      required: true
      default: 'Hello'
      type: string
    environment:
      description: 'Environment to run'
      required: true
      type: choice
      options:
        - development
        - staging
        - production
```

### 4.1. `workflow_dispatch` Event

**Giải thích:**
- Cho phép trigger workflow thủ công từ GitHub UI hoặc API
- Có thể định nghĩa các **inputs** để user nhập khi trigger

**Tại sao dùng `workflow_dispatch`?**

| Use Case | Mô tả |
|----------|-------|
| Manual deployments | Deploy khi team quyết định |
| Testing workflows | Test workflow mà không cần push code |
| One-time tasks | Chạy migration, data fix |
| Parameterized runs | Chạy với các cấu hình khác nhau |

### 4.2. Input Types

**`type: string`**

```yaml
greeting:
  description: 'Your greeting message'
  required: true
  default: 'Hello'
  type: string
```

- Input dạng text tự do
- User có thể nhập bất kỳ text nào

**`type: choice`**

```yaml
environment:
  description: 'Environment to run'
  required: true
  type: choice
  options:
    - development
    - staging
    - production
```

- Input dạng dropdown/select
- User chỉ có thể chọn từ các options được định nghĩa

**Tất cả Input Types:**

| Type | Mô tả | Ví dụ |
|------|-------|-------|
| `string` | Free text | Tên, message, version |
| `choice` | Dropdown selection | Environment, region |
| `boolean` | Checkbox true/false | Enable debug, dry-run |
| `number` | Số | Batch size, timeout |
| `environment` | GitHub Environments | (special type) |

**Ví dụ đầy đủ các types:**

```yaml
workflow_dispatch:
  inputs:
    # String input
    version:
      description: 'Version to deploy'
      required: true
      type: string
      default: 'latest'
    
    # Choice input
    environment:
      description: 'Target environment'
      required: true
      type: choice
      options:
        - development
        - staging
        - production
    
    # Boolean input
    dry_run:
      description: 'Run in dry-run mode?'
      required: false
      type: boolean
      default: false
    
    # Number input
    replicas:
      description: 'Number of replicas'
      required: false
      type: number
      default: 3
```

### 4.3. Input Properties

| Property | Bắt buộc | Mô tả |
|----------|----------|-------|
| `description` | ❌ | Mô tả hiển thị trên UI |
| `required` | ❌ | `true` = bắt buộc nhập |
| `default` | ❌ | Giá trị mặc định |
| `type` | ❌ | Loại input (default: string) |
| `options` | Chỉ cho `choice` | Danh sách options |

---

## 📋 PHẦN 5: JOBS VÀ STEPS

### 5.1. Job Definition

```yaml
jobs:
  show-triggers:
    runs-on: ubuntu-latest
```

**Giải thích:**
- `jobs` - Container cho tất cả jobs
- `show-triggers` - Job ID (tên tùy chọn)
- `runs-on: ubuntu-latest` - Chạy trên Ubuntu runner mới nhất

### 5.2. Step 1: Show Trigger Info

```yaml
- name: 📋 Show Trigger Info
  run: |
    echo "Trigger Type: ${{ github.event_name }}"
    echo "Event: ${{ github.event }}"
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

**Giải thích các Context Variables:**

| Variable | Mô tả | Ví dụ giá trị |
|----------|-------|---------------|
| `github.event_name` | Tên event đã trigger | `push`, `pull_request`, `schedule` |
| `github.event` | Full event payload (object) | `{ "push": {...} }` |
| `github.ref` | Full ref name | `refs/heads/main` |
| `github.sha` | Commit SHA | `a1b2c3d4e5f6...` |
| `github.actor` | User trigger workflow | `username` |
| `github.repository` | Repository name | `owner/repo` |
| `github.ref_name` | Branch/tag name only | `main`, `v1.0.0` |
| `github.event.head_commit.message` | Commit message (chỉ cho push) | `"Fix bug #123"` |
| `github.event.pull_request.number` | PR number (chỉ cho PR events) | `42` |
| `github.triggering_actor` | User thực sự trigger (kể cả scheduled) | `username` hoặc `github-actions` |

**Tại sao dùng `${{ }}` syntax?**

| Syntax | Mô tả | Khi nào dùng |
|--------|-------|--------------|
| `${{ }}` | Expression syntax | ✅ **Truy cập context, variables** |
| `$VAR` | Shell variable | Chỉ cho shell environment variables |
| `${VAR}` | Shell variable (explicit) | Chỉ cho shell environment variables |

### 5.3. Step 2: Conditional Step

```yaml
- name: 👋 Show Manual Inputs
  if: github.event_name == 'workflow_dispatch'
  run: |
    echo "Greeting: ${{ github.event.inputs.greeting }}"
    echo "Environment: ${{ github.event.inputs.environment }}"
```

**Giải thích:**
- `if` condition: Step này **CHỈ chạy** khi trigger là `workflow_dispatch`
- Access inputs qua `github.event.inputs.<input_name>`

**Tại sao cần `if` condition?**

| Scenario | Không có `if` | Có `if` |
|----------|---------------|---------|
| Push trigger | Error vì không có inputs | ✅ Step bị skip |
| PR trigger | Error vì không có inputs | ✅ Step bị skip |
| Manual trigger | ✅ Chạy bình thường | ✅ Chạy bình thường |

---

## 🎓 PHẦN MỞ RỘNG: CÁC EVENTS KHÁC

### Các Events Phổ Biến Khác

#### 1. `create` / `delete`

```yaml
on:
  create:  # Khi tạo branch hoặc tag
  delete:  # Khi xóa branch hoặc tag
```

#### 2. `release`

```yaml
on:
  release:
    types: [published, created, released]
```

**Use case:** Deploy khi release mới được publish

#### 3. `issues` / `issue_comment`

```yaml
on:
  issues:
    types: [opened, labeled]
  issue_comment:
    types: [created]
```

**Use case:** Automation cho issue management

#### 4. `workflow_run`

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [completed]
```

**Use case:** Trigger workflow B sau khi workflow A hoàn thành

#### 5. `repository_dispatch`

```yaml
on:
  repository_dispatch:
    types: [my-custom-event]
```

**Use case:** Trigger từ external services (API call)

---

## 🔄 Luồng Thực Thi

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Workflow: 02 - Triggers Demo                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     TRIGGER EVENTS                           │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │                                                             │   │
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌─────────┐ │   │
│  │  │  PUSH    │  │ PULL_REQUEST │  │ SCHEDULE │  │ MANUAL  │ │   │
│  │  │          │  │              │  │          │  │         │ │   │
│  │  │ main     │  │ → main       │  │ 2AM UTC  │  │ Inputs: │ │   │
│  │  │ feature/*│  │              │  │ daily    │  │ greeting│ │   │
│  │  │ src/**   │  │ opened       │  │          │  │ env     │ │   │
│  │  │ !*.md    │  │ synchronize  │  │          │  │         │ │   │
│  │  │          │  │ reopened     │  │          │  │         │ │   │
│  │  └────┬─────┘  └──────┬───────┘  └────┬─────┘  └────┬────┘ │   │
│  │       │               │               │              │      │   │
│  └───────┴───────────────┴───────────────┴──────────────┴──────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    JOB: show-triggers                        │   │
│  │                    Runner: ubuntu-latest                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │                                                             │   │
│  │  Step 1: 📋 Show Trigger Info                               │   │
│  │  ├─ Display github.event_name                               │   │
│  │  ├─ Display github.ref                                      │   │
│  │  ├─ Display github.sha                                      │   │
│  │  └─ Display other context variables                         │   │
│  │                                                             │   │
│  │  Step 2: 👋 Show Manual Inputs                              │   │
│  │  └─ if: github.event_name == 'workflow_dispatch'            │   │
│  │     ├─ Display greeting input                               │   │
│  │     └─ Display environment input                            │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

### 1. Kết hợp nhiều triggers

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

### 2. Sử dụng paths để tiết kiệm resources

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'tests/**'
      - 'requirements.txt'
    paths-ignore:
      - '**.md'
      - 'docs/**'
```

### 3. Giới hạn PR types khi cần

```yaml
on:
  pull_request:
    types: [opened, synchronize]  # Không trigger khi reopened
```

### 4. Sử dụng conditional steps thay vì conditional workflows

```yaml
# ✅ Một workflow với conditional steps
steps:
  - name: Deploy
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    run: ./deploy.sh
```

### 5. Validate inputs trong workflow_dispatch

```yaml
- name: Validate inputs
  run: |
    if [[ "${{ inputs.environment }}" == "production" ]]; then
      echo "⚠️ Deploying to PRODUCTION!"
    fi
```

---

## 📊 So Sánh Tổng Hợp Các Events

| Event | Auto/Manual | Use Case | Frequency |
|-------|-------------|----------|-----------|
| `push` | Auto | CI trên mỗi commit | Rất thường xuyên |
| `pull_request` | Auto | PR review/testing | Thường xuyên |
| `schedule` | Auto | Cron jobs | Định kỳ |
| `workflow_dispatch` | Manual | Deployments, testing | Theo nhu cầu |
| `release` | Auto | Release deployments | Ít thường xuyên |
| `workflow_run` | Auto | Chained workflows | Khi cần |

---

## 🔗 Tài Liệu Tham Khảo

- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Workflow syntax: on](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)
- [Context and expression syntax](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [Crontab.guru - Cron expression editor](https://crontab.guru/)
