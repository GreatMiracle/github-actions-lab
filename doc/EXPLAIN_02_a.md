# 📚 Danh Sách Đầy Đủ GitHub Actions Events (100%)

## 📌 Tổng Quan

GitHub Actions hỗ trợ hơn **35+ events** khác nhau để trigger workflows. File này liệt kê **ĐẦY ĐỦ** tất cả events, phân loại theo nhóm để dễ nhớ và sử dụng.

---

## 🎯 Phân Loại Events

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GITHUB ACTIONS EVENTS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📦 CODE & REPOSITORY          👥 COLLABORATION                    │
│  ├─ push                       ├─ pull_request                     │
│  ├─ create                     ├─ pull_request_review              │
│  ├─ delete                     ├─ pull_request_review_comment      │
│  ├─ fork                       ├─ pull_request_target              │
│  ├─ gollum (wiki)              ├─ issue_comment                    │
│  ├─ release                    ├─ issues                           │
│  ├─ branch_protection_rule     └─ discussion                       │
│  └─ public                         └─ discussion_comment           │
│                                                                     │
│  ⏰ SCHEDULING & MANUAL        🔒 SECURITY                          │
│  ├─ schedule                   ├─ secret_scanning_alert            │
│  ├─ workflow_dispatch          ├─ code_scanning_alert              │
│  └─ repository_dispatch        ├─ dependabot_alert                 │
│                                └─ secret_scanning_alert_location   │
│                                                                     │
│  🔄 WORKFLOW MANAGEMENT        📋 PROJECT MANAGEMENT               │
│  ├─ workflow_run               ├─ project                          │
│  ├─ workflow_call              ├─ project_card                     │
│  └─ check_run                  ├─ project_column                   │
│      └─ check_suite            ├─ projects_v2_item                 │
│                                ├─ milestone                        │
│                                └─ label                            │
│                                                                     │
│  🚀 DEPLOYMENT                 👤 MEMBERSHIP                        │
│  ├─ deployment                 ├─ member                           │
│  ├─ deployment_status          ├─ membership                       │
│  └─ deployment_protection_rule ├─ team                             │
│                                ├─ team_add                         │
│  📦 PACKAGES                   ├─ org_block                        │
│  ├─ registry_package           ├─ organization                     │
│  └─ package                    ├─ watch                            │
│                                └─ star                             │
│  🔌 CI/CD                                                          │
│  ├─ status                                                         │
│  ├─ page_build                                                     │
│  └─ merge_group                                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📦 NHÓM 1: CODE & REPOSITORY EVENTS

### 1.1. `push` ⭐ (Rất phổ biến)

```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*']
    paths: ['src/**']
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi push commits hoặc tags lên repository |
| **Use case** | CI/CD, build, test, deploy |
| **Filters** | `branches`, `branches-ignore`, `tags`, `tags-ignore`, `paths`, `paths-ignore` |
| **Mẹo nhớ** | 🚀 **"Push = Đẩy code lên = Chạy CI"** |

---

### 1.2. `create`

```yaml
on:
  create:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi tạo branch hoặc tag MỚI |
| **Use case** | Notification, setup môi trường cho branch mới |
| **Activity types** | Không có (chỉ 1 loại) |
| **Mẹo nhớ** | 🌱 **"Create = Sinh ra cái mới"** |

**Ví dụ thực tế:**
```yaml
on:
  create:
jobs:
  notify:
    if: github.ref_type == 'branch'
    runs-on: ubuntu-latest
    steps:
      - run: echo "New branch created: ${{ github.ref_name }}"
```

---

### 1.3. `delete`

```yaml
on:
  delete:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi xóa branch hoặc tag |
| **Use case** | Cleanup resources, remove preview environments |
| **Mẹo nhớ** | 🗑️ **"Delete = Dọn dẹp"** |

**Ví dụ thực tế:**
```yaml
on:
  delete:
jobs:
  cleanup:
    if: github.event.ref_type == 'branch'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Cleaning up resources for deleted branch"
```

---

### 1.4. `fork`

```yaml
on:
  fork:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi ai đó fork repository của bạn |
| **Use case** | Tracking, welcome message, analytics |
| **Mẹo nhớ** | 🍴 **"Fork = Nĩa = Ai đó copy repo"** |

---

### 1.5. `gollum` (Wiki)

```yaml
on:
  gollum:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi tạo/update wiki page |
| **Use case** | Backup wiki, sync documentation |
| **Mẹo nhớ** | 📖 **"Gollum = Wiki engine name = Sửa wiki"** |

---

### 1.6. `release` ⭐ (Phổ biến)

```yaml
on:
  release:
    types: [published, created, released, prereleased, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có hoạt động release |
| **Activity types** | `published`, `unpublished`, `created`, `edited`, `deleted`, `prereleased`, `released` |
| **Use case** | Deploy production, publish packages, generate changelogs |
| **Mẹo nhớ** | 🎉 **"Release = Phát hành = Deploy"** |

**Activity Types chi tiết:**

| Type | Mô tả |
|------|-------|
| `published` | Release được publish (bao gồm từ draft) |
| `unpublished` | Release bị unpublish |
| `created` | Release được tạo (kể cả draft) |
| `edited` | Release được edit (title, body) |
| `deleted` | Release bị xóa |
| `prereleased` | Pre-release được publish |
| `released` | Release/Pre-release được publish hoặc edit từ pre-release thành release |

---

### 1.7. `public`

```yaml
on:
  public:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi repository chuyển từ private → public |
| **Use case** | Security scan, notification |
| **Mẹo nhớ** | 🌍 **"Public = Công khai cho mọi người"** |

---

### 1.8. `branch_protection_rule`

```yaml
on:
  branch_protection_rule:
    types: [created, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi branch protection rule được tạo/sửa/xóa |
| **Use case** | Audit logging, compliance checking |
| **Mẹo nhớ** | 🛡️ **"Bảo vệ branch = Cần audit"** |

---

## 👥 NHÓM 2: COLLABORATION EVENTS

### 2.1. `pull_request` ⭐ (Rất phổ biến)

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches: [main]
    paths: ['src/**']
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có hoạt động trên Pull Request |
| **Default types** | `opened`, `synchronize`, `reopened` |
| **Filters** | `branches`, `branches-ignore`, `paths`, `paths-ignore` |
| **Mẹo nhớ** | 🔀 **"PR = Review code = Test trước khi merge"** |

**Tất cả Activity Types:**

| Type | Mô tả | Phổ biến |
|------|-------|----------|
| `opened` | PR được tạo mới | ⭐⭐⭐ |
| `synchronize` | Có commits mới push vào PR | ⭐⭐⭐ |
| `reopened` | PR bị close được mở lại | ⭐⭐⭐ |
| `closed` | PR bị đóng (merge hoặc không merge) | ⭐⭐ |
| `assigned` | PR được assign cho ai đó | ⭐ |
| `unassigned` | PR bị bỏ assign | ⭐ |
| `labeled` | Label được thêm vào PR | ⭐⭐ |
| `unlabeled` | Label bị remove | ⭐ |
| `edited` | Title hoặc body được sửa | ⭐ |
| `ready_for_review` | Draft PR → Ready for review | ⭐⭐ |
| `converted_to_draft` | PR → Draft | ⭐ |
| `locked` | PR conversation bị lock | ⭐ |
| `unlocked` | PR conversation được unlock | ⭐ |
| `review_requested` | Request review từ ai đó | ⭐⭐ |
| `review_request_removed` | Remove review request | ⭐ |
| `auto_merge_enabled` | Auto-merge được bật | ⭐ |
| `auto_merge_disabled` | Auto-merge bị tắt | ⭐ |

---

### 2.2. `pull_request_target`

```yaml
on:
  pull_request_target:
    types: [opened, synchronize]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Như `pull_request` nhưng chạy trong context của BASE repo |
| **Use case** | Labeling PRs từ forks, requires write access |
| **⚠️ Cảnh báo** | **RẤT NGUY HIỂM** nếu checkout PR code - có thể bị code injection |
| **Mẹo nhớ** | 🎯 **"Target = Base repo context = Cần cẩn thận security"** |

**So sánh:**

| Feature | `pull_request` | `pull_request_target` |
|---------|---------------|----------------------|
| Context | HEAD (fork) | BASE (your repo) |
| Secrets | ❌ Không access | ✅ Có access |
| Write permission | ❌ Không | ✅ Có |
| Safe for forks | ✅ An toàn | ⚠️ Cần cẩn thận |

---

### 2.3. `pull_request_review`

```yaml
on:
  pull_request_review:
    types: [submitted, edited, dismissed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có review trên PR |
| **Activity types** | `submitted`, `edited`, `dismissed` |
| **Use case** | Auto-merge khi approved, notifications |
| **Mẹo nhớ** | 👀 **"Review = Ai đó đang xem PR"** |

---

### 2.4. `pull_request_review_comment`

```yaml
on:
  pull_request_review_comment:
    types: [created, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có comment trong diff của PR |
| **Activity types** | `created`, `edited`, `deleted` |
| **Use case** | Track code discussions, analytics |
| **Mẹo nhớ** | 💬 **"Review comment = Comment trên từng dòng code"** |

---

### 2.5. `issues` ⭐ (Phổ biến)

```yaml
on:
  issues:
    types: [opened, labeled, closed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có hoạt động trên Issues |
| **Activity types** | `opened`, `edited`, `deleted`, `pinned`, `unpinned`, `closed`, `reopened`, `assigned`, `unassigned`, `labeled`, `unlabeled`, `locked`, `unlocked`, `transferred`, `milestoned`, `demilestoned` |
| **Use case** | Auto-labeling, triage, notifications |
| **Mẹo nhớ** | 🐛 **"Issues = Bug reports = Cần xử lý"** |

---

### 2.6. `issue_comment`

```yaml
on:
  issue_comment:
    types: [created, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có comment trên issue HOẶC PR |
| **Activity types** | `created`, `edited`, `deleted` |
| **Use case** | Bot commands (e.g., `/deploy`), auto-responses |
| **Mẹo nhớ** | 💭 **"Comment = Ai đó nói gì đó"** |

**⚠️ Lưu ý:** `issue_comment` trigger cho CẢ issues VÀ PRs!

```yaml
# Chỉ respond cho PR comments
- if: github.event.issue.pull_request
  run: echo "This is a PR comment"
```

---

### 2.7. `discussion`

```yaml
on:
  discussion:
    types: [created, edited, answered, category_changed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có hoạt động trên Discussions |
| **Activity types** | `created`, `edited`, `deleted`, `transferred`, `pinned`, `unpinned`, `labeled`, `unlabeled`, `locked`, `unlocked`, `category_changed`, `answered`, `unanswered` |
| **Use case** | Community support automation |
| **Mẹo nhớ** | 🗣️ **"Discussion = Thảo luận cộng đồng"** |

---

### 2.8. `discussion_comment`

```yaml
on:
  discussion_comment:
    types: [created, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi có comment trên Discussion |
| **Use case** | Track engagement, auto-responses |
| **Mẹo nhớ** | 💬 **"Discussion comment = Reply trong thảo luận"** |

---

## ⏰ NHÓM 3: SCHEDULING & MANUAL TRIGGERS

### 3.1. `schedule` ⭐ (Phổ biến)

```yaml
on:
  schedule:
    - cron: '0 2 * * *'    # 2 AM UTC daily
    - cron: '0 0 * * 0'    # Midnight UTC every Sunday
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Theo lịch cron định kỳ |
| **Timezone** | Luôn là UTC (không thể đổi) |
| **Branch** | Chỉ chạy trên default branch |
| **Mẹo nhớ** | ⏰ **"Schedule = Đồng hồ báo thức = Chạy định kỳ"** |

**Cron Cheat Sheet:**

| Expression | Ý nghĩa |
|------------|---------|
| `0 0 * * *` | Nửa đêm UTC hàng ngày |
| `0 */6 * * *` | Mỗi 6 giờ |
| `0 9 * * 1-5` | 9 AM UTC, Mon-Fri |
| `0 0 1 * *` | Ngày 1 hàng tháng |
| `*/15 * * * *` | Mỗi 15 phút |

---

### 3.2. `workflow_dispatch` ⭐ (Rất phổ biến)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]
      debug:
        type: boolean
        default: false
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Manual từ GitHub UI hoặc API |
| **Input types** | `string`, `choice`, `boolean`, `number`, `environment` |
| **Use case** | Manual deployments, testing, one-time tasks |
| **Mẹo nhớ** | 🖱️ **"Dispatch = Gửi đi bằng tay"** |

---

### 3.3. `repository_dispatch`

```yaml
on:
  repository_dispatch:
    types: [deploy, build, custom-event]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Từ external API call |
| **Use case** | Trigger từ external services, webhooks |
| **Mẹo nhớ** | 🌐 **"Repository dispatch = External systems gọi vào"** |

**Cách trigger từ API:**
```bash
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token YOUR_PAT" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"deploy","client_payload":{"env":"prod"}}'
```

---

## 🔄 NHÓM 4: WORKFLOW MANAGEMENT EVENTS

### 4.1. `workflow_run` ⭐ (Phổ biến)

```yaml
on:
  workflow_run:
    workflows: ["Build", "Test"]
    types: [completed]
    branches: [main]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi workflow khác hoàn thành |
| **Activity types** | `requested`, `in_progress`, `completed` |
| **Use case** | Chained workflows, deploy sau khi test pass |
| **Mẹo nhớ** | 🔗 **"Workflow run = Nối tiếp workflows"** |

**Ví dụ: Deploy chỉ khi Build success:**
```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [completed]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying because build succeeded!"
```

---

### 4.2. `workflow_call` (Reusable Workflows)

```yaml
on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true
    secrets:
      deploy_key:
        required: true
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi workflow khác gọi workflow này |
| **Use case** | Reusable/shared workflows |
| **Mẹo nhớ** | 📞 **"Workflow call = Gọi workflow như function"** |

---

### 4.3. `check_run`

```yaml
on:
  check_run:
    types: [created, rerequested, completed, requested_action]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi check run được tạo hoặc hoàn thành |
| **Use case** | Custom CI integrations |
| **Mẹo nhớ** | ✅ **"Check run = Kiểm tra CI"** |

---

### 4.4. `check_suite`

```yaml
on:
  check_suite:
    types: [completed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi tất cả check runs trong suite hoàn thành |
| **Use case** | Aggregate CI results |
| **Mẹo nhớ** | 📦 **"Check suite = Nhóm các checks"** |

---

## 🔒 NHÓM 5: SECURITY EVENTS

### 5.1. `secret_scanning_alert`

```yaml
on:
  secret_scanning_alert:
    types: [created, resolved, reopened]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi phát hiện secret trong code |
| **Use case** | Auto-rotate secrets, notify security team |
| **Mẹo nhớ** | 🔑 **"Secret scan = Phát hiện API keys lộ"** |

---

### 5.2. `secret_scanning_alert_location`

```yaml
on:
  secret_scanning_alert_location:
    types: [created]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi phát hiện vị trí mới của secret đã biết |
| **Use case** | Track secret spread |
| **Mẹo nhớ** | 📍 **"Location = Vị trí secret trong code"** |

---

### 5.3. `code_scanning_alert`

```yaml
on:
  code_scanning_alert:
    types: [created, reopened_by_user, closed_by_user, fixed, appeared_in_branch]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi CodeQL hoặc tools khác phát hiện vulnerability |
| **Use case** | Security response automation |
| **Mẹo nhớ** | 🔍 **"Code scan = Quét lỗ hổng bảo mật"** |

---

### 5.4. `dependabot_alert`

```yaml
on:
  dependabot_alert:
    types: [created, dismissed, fixed, reintroduced]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi Dependabot phát hiện vulnerable dependency |
| **Use case** | Auto-upgrade dependencies, notify team |
| **Mẹo nhớ** | 📦 **"Dependabot = Robot kiểm tra dependencies"** |

---

## 🚀 NHÓM 6: DEPLOYMENT EVENTS

### 6.1. `deployment`

```yaml
on:
  deployment:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi deployment được tạo |
| **Use case** | Custom deployment workflows |
| **Mẹo nhớ** | 🚀 **"Deployment = Triển khai ứng dụng"** |

---

### 6.2. `deployment_status`

```yaml
on:
  deployment_status:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi deployment status thay đổi |
| **Use case** | Post-deployment actions, notifications |
| **Mẹo nhớ** | 📊 **"Deployment status = Trạng thái deploy"** |

---

### 6.3. `deployment_protection_rule`

```yaml
on:
  deployment_protection_rule:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi deployment cần approval |
| **Use case** | Custom approval workflows |
| **Mẹo nhớ** | 🛡️ **"Protection = Cần approval trước khi deploy"** |

---

## 📋 NHÓM 7: PROJECT MANAGEMENT EVENTS

### 7.1. `project`

```yaml
on:
  project:
    types: [created, closed, reopened, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi Project board thay đổi |
| **Use case** | Project tracking, notifications |
| **Mẹo nhớ** | 📊 **"Project = Kanban board"** |

---

### 7.2. `project_card`

```yaml
on:
  project_card:
    types: [created, edited, moved, converted, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi card trong Project thay đổi |
| **Use case** | Sync với external tools |
| **Mẹo nhớ** | 🃏 **"Card = Thẻ trong Kanban"** |

---

### 7.3. `project_column`

```yaml
on:
  project_column:
    types: [created, edited, moved, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi column trong Project thay đổi |
| **Mẹo nhớ** | 📑 **"Column = Cột trong Kanban"** |

---

### 7.4. `projects_v2_item`

```yaml
on:
  projects_v2_item:
    types: [created, edited, deleted, archived, restored, converted, reordered]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi item trong Projects V2 (new) thay đổi |
| **Use case** | Projects V2 automation |
| **Mẹo nhớ** | 🆕 **"V2 = Projects mới của GitHub"** |

---

### 7.5. `milestone`

```yaml
on:
  milestone:
    types: [created, closed, opened, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi Milestone thay đổi |
| **Use case** | Release planning notifications |
| **Mẹo nhớ** | 🏁 **"Milestone = Cột mốc release"** |

---

### 7.6. `label`

```yaml
on:
  label:
    types: [created, edited, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi label được tạo/sửa/xóa |
| **Use case** | Sync labels across repos |
| **Mẹo nhớ** | 🏷️ **"Label = Nhãn phân loại"** |

---

## 👤 NHÓM 8: MEMBERSHIP EVENTS

### 8.1. `member`

```yaml
on:
  member:
    types: [added, edited, removed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi collaborator được thêm/bỏ |
| **Use case** | Onboarding, audit logging |
| **Mẹo nhớ** | 👤 **"Member = Người trong repo"** |

---

### 8.2. `membership` (Organization)

```yaml
on:
  membership:
    types: [added, removed]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi member được thêm/bỏ khỏi team |
| **Use case** | Organization management |
| **Mẹo nhớ** | 👥 **"Membership = Thành viên org"** |

---

### 8.3. `team`

```yaml
on:
  team:
    types: [created, deleted, edited, added_to_repository, removed_from_repository]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi team thay đổi |
| **Use case** | Team management automation |
| **Mẹo nhớ** | 👥 **"Team = Nhóm người dùng"** |

---

### 8.4. `team_add`

```yaml
on:
  team_add:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi team được add vào repository |
| **Use case** | Access management |
| **Mẹo nhớ** | ➕ **"Team add = Thêm team vào repo"** |

---

### 8.5. `organization`

```yaml
on:
  organization:
    types: [deleted, renamed, member_added, member_removed, member_invited]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi organization thay đổi |
| **Use case** | Organization audit |
| **Mẹo nhớ** | 🏢 **"Organization = Tổ chức"** |

---

### 8.6. `org_block`

```yaml
on:
  org_block:
    types: [blocked, unblocked]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi user bị block/unblock khỏi org |
| **Use case** | Security audit |
| **Mẹo nhớ** | 🚫 **"Block = Chặn user"** |

---

### 8.7. `watch`

```yaml
on:
  watch:
    types: [started]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi ai đó watch (star cũ) repository |
| **Use case** | Thank you messages, analytics |
| **Mẹo nhớ** | 👁️ **"Watch = Theo dõi repo"** |

---

### 8.8. `star`

```yaml
on:
  star:
    types: [created, deleted]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi ai đó star/unstar repository |
| **Use case** | Thank you messages, analytics |
| **Mẹo nhớ** | ⭐ **"Star = Yêu thích repo"** |

---

## 📦 NHÓM 9: PACKAGE EVENTS

### 9.1. `registry_package`

```yaml
on:
  registry_package:
    types: [published, updated]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi package được publish lên GitHub Packages |
| **Use case** | Post-publish actions |
| **Mẹo nhớ** | 📦 **"Registry = Nơi lưu packages"** |

---

## 🔌 NHÓM 10: CI/CD EVENTS

### 10.1. `status`

```yaml
on:
  status:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi commit status thay đổi |
| **Use case** | React to external CI/CD systems |
| **Mẹo nhớ** | 🚦 **"Status = Trạng thái commit (✓/✗)"** |

---

### 10.2. `page_build`

```yaml
on:
  page_build:
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi GitHub Pages được build |
| **Use case** | Post-deploy notifications |
| **Mẹo nhớ** | 📄 **"Page build = Build website GitHub Pages"** |

---

### 10.3. `merge_group`

```yaml
on:
  merge_group:
    types: [checks_requested]
```

| Thuộc tính | Mô tả |
|------------|-------|
| **Khi nào trigger** | Khi PR được thêm vào merge queue |
| **Use case** | Merge queue CI |
| **Mẹo nhớ** | 🔀 **"Merge group = Hàng đợi merge"** |

---

## 📊 Bảng Tổng Hợp Nhanh

| Event | Phổ biến | Use Case Chính |
|-------|----------|----------------|
| `push` | ⭐⭐⭐ | CI/CD cho mỗi commit |
| `pull_request` | ⭐⭐⭐ | Review và test PRs |
| `schedule` | ⭐⭐⭐ | Cron jobs, nightly builds |
| `workflow_dispatch` | ⭐⭐⭐ | Manual triggers |
| `release` | ⭐⭐ | Deploy khi release |
| `workflow_run` | ⭐⭐ | Chained workflows |
| `issues` | ⭐⭐ | Issue automation |
| `issue_comment` | ⭐⭐ | Bot commands |
| `create/delete` | ⭐ | Branch lifecycle |
| Còn lại | ⭐ | Specialized use cases |

---

## 💡 Tips Nhớ Events

1. **CODE changes** = `push`, `pull_request`
2. **TIME-based** = `schedule`
3. **MANUAL** = `workflow_dispatch`, `repository_dispatch`
4. **CHAINED** = `workflow_run`, `workflow_call`
5. **RELEASES** = `release`, `deployment`
6. **SECURITY** = `*_alert` events
7. **PEOPLE** = `member`, `team`, `organization`

---

## 🔗 Tài Liệu Tham Khảo

- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Webhook events and payloads](https://docs.github.com/en/webhooks/webhook-events-and-payloads)
- [GitHub Actions documentation](https://docs.github.com/en/actions)
