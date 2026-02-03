# Giải Thích Chi Tiết: 04b-secrets.yml

## 📌 Tổng Quan

File `04b-secrets.yml` minh họa cách sử dụng **Secrets** trong GitHub Actions - một tính năng quan trọng để quản lý các thông tin nhạy cảm như API keys, passwords, tokens một cách an toàn.

---

## 🔐 Secrets Là Gì?

**Secrets** là các biến được mã hóa (encrypted variables) mà bạn tạo trong repository hoặc organization. GitHub Actions sử dụng libsodium sealed box để mã hóa secrets.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SECRETS FLOW                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐     ┌───────────────┐     ┌──────────────────┐   │
│  │   Bạn tạo    │────▶│ GitHub mã hóa │────▶│ Lưu trữ an toàn  │   │
│  │   Secret     │     │   (encrypt)   │     │   trên server    │   │
│  └──────────────┘     └───────────────┘     └────────┬─────────┘   │
│                                                      │              │
│                                                      ▼              │
│  ┌──────────────┐     ┌───────────────┐     ┌──────────────────┐   │
│  │  Runner sử   │◀────│ GitHub giải   │◀────│ Workflow request │   │
│  │  dụng secret │     │ mã (decrypt)  │     │     secret       │   │
│  └──────────────┘     └───────────────┘     └──────────────────┘   │
│                                                                     │
│  ⚠️ Secret được mask trong logs với ***                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Phân Tích Từng Phần

### 1. Tên Workflow và Trigger

```yaml
name: 04b - Secrets Demo

on:
  workflow_dispatch:
```

**Giải thích:**
- `workflow_dispatch` cho phép chạy workflow thủ công
- Phù hợp cho demo vì bạn kiểm soát khi nào workflow chạy

**Tại sao dùng `workflow_dispatch`?**

| Trigger | Phù hợp | Lý do |
|---------|---------|-------|
| `workflow_dispatch` | ✅ | An toàn để test secrets, kiểm soát được |
| `push` | ⚠️ | Có thể vô tình expose secrets trong logs |
| `pull_request` | ❌ | **KHÔNG có access secrets** từ forks (security) |

---

### 2. Job Definition

```yaml
jobs:
  use-secrets:
    runs-on: ubuntu-latest
```

**Giải thích:**
- `use-secrets` - Tên job mô tả chức năng
- `ubuntu-latest` - GitHub-hosted runner tiêu chuẩn

---

### 3. Step 1: Sử Dụng User-Defined Secret

```yaml
- name: 🔐 Use Secret
  env:
    SECRET_KEY: ${{ secrets.MY_SECRET_KEY }}
  run: |
    echo "Secret length: ${#SECRET_KEY}"
    # Không được echo trực tiếp secret!
    # GitHub sẽ che nó với ***
    echo "Secret: $SECRET_KEY"
```

#### 3.1. `env` Block

```yaml
env:
  SECRET_KEY: ${{ secrets.MY_SECRET_KEY }}
```

**Giải thích:**
- `env` - Định nghĩa environment variables cho step này
- `SECRET_KEY` - Tên biến môi trường (bạn tự đặt)
- `${{ secrets.MY_SECRET_KEY }}` - Truy cập secret có tên `MY_SECRET_KEY`

**Tại sao dùng `env` thay vì inline?**

| Cách dùng | Ví dụ | Khuyến nghị |
|-----------|-------|-------------|
| Qua `env` | `env: SECRET_KEY: ${{ secrets.X }}` | ✅ **Best Practice** |
| Inline | `run: echo ${{ secrets.X }}` | ⚠️ **Không khuyến nghị** |

**Lý do dùng `env`:**

1. **An toàn hơn:** Tránh command injection
2. **Dễ debug:** Dễ thấy biến nào được set
3. **Tái sử dụng:** Dùng được nhiều lần trong `run`
4. **Không bị expose:** GitHub sẽ mask trong logs

**Ví dụ nguy hiểm khi dùng inline:**

```yaml
# ❌ KHÔNG NÊN - Có thể bị command injection
run: |
  curl -u user:${{ secrets.PASSWORD }} https://api.example.com

# ✅ NÊN DÙNG
env:
  PASSWORD: ${{ secrets.PASSWORD }}
run: |
  curl -u user:$PASSWORD https://api.example.com
```

#### 3.2. `${#SECRET_KEY}` - Lấy độ dài

```yaml
echo "Secret length: ${#SECRET_KEY}"
```

**Giải thích:**
- `${#VAR}` là bash syntax để lấy **độ dài** của string
- An toàn vì không expose giá trị thực

**Tại sao check độ dài?**
- Verify secret đã được set đúng
- Debug mà không expose secret
- Kiểm tra format (ví dụ: API key phải có 32 ký tự)

#### 3.3. GitHub Auto-Masking

```yaml
echo "Secret: $SECRET_KEY"
```

**Kết quả trong logs:**
```
Secret: ***
```

**Giải thích:**
- GitHub **tự động detect và mask** (che) giá trị secrets trong logs
- Bất kỳ output nào match với secret value sẽ được thay thế bằng `***`

**⚠️ Lưu ý quan trọng về Masking:**

| Trường hợp | Có mask không | Giải thích |
|------------|---------------|------------|
| Echo trực tiếp | ✅ Có | `echo $SECRET` → `***` |
| In từng ký tự | ❌ KHÔNG | `echo ${SECRET:0:1}` → Có thể lộ |
| Base64 encode | ❌ KHÔNG | `echo $SECRET \| base64` → Lộ |
| Reverse string | ❌ KHÔNG | Có thể lộ |
| Write to file | ✅ Có (nếu cat file) | Nhưng file vẫn chứa secret |

**Best Practices để tránh lộ secrets:**

```yaml
# ❌ KHÔNG NÊN
run: |
  echo ${MY_SECRET:0:5}...  # In 5 ký tự đầu
  echo $MY_SECRET | base64  # Encode base64

# ✅ NÊN LÀM
run: |
  echo "Secret is set: yes"
  echo "Secret length: ${#MY_SECRET}"
```

---

### 4. Step 2: Sử Dụng GITHUB_TOKEN

```yaml
- name: 🔐 Use GITHUB_TOKEN
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    echo "Token available: yes"
    # Dùng để call GitHub API
```

#### 4.1. `GITHUB_TOKEN` là gì?

**`GITHUB_TOKEN`** là một **automatic token** được GitHub tạo tự động cho mỗi workflow run. Bạn KHÔNG cần tạo nó.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GITHUB_TOKEN LIFECYCLE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐                                                   │
│  │ Workflow     │                                                   │
│  │   starts     │                                                   │
│  └──────┬───────┘                                                   │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────────────────────────────┐                          │
│  │ GitHub tự động tạo GITHUB_TOKEN      │                          │
│  │ với permissions dựa trên:            │                          │
│  │ - Repository settings                │                          │
│  │ - Workflow permissions config        │                          │
│  └──────┬───────────────────────────────┘                          │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────────────────────────────┐                          │
│  │ Token được inject vào workflow       │                          │
│  │ Accessible via:                      │                          │
│  │ - ${{ secrets.GITHUB_TOKEN }}        │                          │
│  │ - ${{ github.token }}                │                          │
│  └──────┬───────────────────────────────┘                          │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────┐                                                   │
│  │ Workflow     │                                                   │
│  │   ends       │                                                   │
│  └──────┬───────┘                                                   │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────────────────────────────┐                          │
│  │ Token TỰ ĐỘNG HẾT HẠN (revoked)      │                          │
│  └──────────────────────────────────────┘                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 4.2. GITHUB_TOKEN vs Personal Access Token (PAT)

| Feature | `GITHUB_TOKEN` | Personal Access Token (PAT) |
|---------|---------------|----------------------------|
| **Tạo bởi** | GitHub tự động | User tạo thủ công |
| **Scope** | Chỉ repository hiện tại | Có thể cross-repo |
| **Lifetime** | Hết hạn khi workflow kết thúc | Tùy config (có thể vĩnh viễn) |
| **Cần lưu trữ** | Không | Có, phải add vào secrets |
| **Security** | ✅ An toàn hơn | ⚠️ Rủi ro nếu lộ |
| **Cross-repo actions** | ❌ Không | ✅ Có |

**Tại sao dùng `GITHUB_TOKEN`?**

| Trường hợp | Dùng gì |
|------------|---------|
| Push code trong cùng repo | `GITHUB_TOKEN` ✅ |
| Create/update issues, PRs | `GITHUB_TOKEN` ✅ |
| Trigger workflow ở repo khác | PAT (cần tạo) |
| Access private repo khác | PAT (cần tạo) |

#### 4.3. GITHUB_TOKEN Permissions

**Default Permissions:**

```yaml
# Trong workflow, có thể customize
permissions:
  contents: read
  issues: write
  pull-requests: write
```

**Tất cả Permissions có thể set:**

| Permission | Mô tả | Values |
|------------|-------|--------|
| `actions` | Manage GitHub Actions | `read`, `write`, `none` |
| `checks` | Manage check runs | `read`, `write`, `none` |
| `contents` | Repository contents | `read`, `write`, `none` |
| `deployments` | Deployments | `read`, `write`, `none` |
| `discussions` | Discussions | `read`, `write`, `none` |
| `id-token` | OIDC token | `write`, `none` |
| `issues` | Issues | `read`, `write`, `none` |
| `packages` | GitHub Packages | `read`, `write`, `none` |
| `pages` | GitHub Pages | `read`, `write`, `none` |
| `pull-requests` | Pull Requests | `read`, `write`, `none` |
| `repository-projects` | Projects | `read`, `write`, `none` |
| `security-events` | Security events | `read`, `write`, `none` |
| `statuses` | Commit statuses | `read`, `write`, `none` |

**Ví dụ customize permissions:**

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    permissions:
      contents: write      # Để push code
      pull-requests: write # Để comment on PRs
      issues: read         # Chỉ đọc issues
    steps:
      - uses: actions/checkout@v4
      - run: git push  # Works vì có contents: write
```

---

## 🎓 PHẦN MỞ RỘNG: SECRETS BEST PRACTICES

### 1. Các Loại Secrets

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SECRETS HIERARCHY                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ORGANIZATION SECRETS                            │   │
│  │  - Shared across all/selected repos                         │   │
│  │  - Admin creates                                             │   │
│  │  - Ví dụ: DOCKER_HUB_TOKEN, AWS_ACCESS_KEY                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              REPOSITORY SECRETS                              │   │
│  │  - Chỉ repo này access được                                 │   │
│  │  - Repo admin creates                                        │   │
│  │  - Ví dụ: DEPLOY_KEY, API_SECRET                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ENVIRONMENT SECRETS                             │   │
│  │  - Chỉ jobs targeting environment này                       │   │
│  │  - Có thể require approval                                   │   │
│  │  - Ví dụ: PROD_DATABASE_URL, STAGING_API_KEY                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2. Cách Tạo Secrets

#### 2.1. Repository Secrets (UI)

```
Repository → Settings → Secrets and variables → Actions → New repository secret
```

#### 2.2. Repository Secrets (CLI)

```bash
# Sử dụng GitHub CLI
gh secret set MY_SECRET_KEY --body "your-secret-value"

# Từ file
gh secret set MY_SECRET_KEY < secret.txt

# Interactive (không hiển thị input)
gh secret set MY_SECRET_KEY
```

#### 2.3. Environment Secrets

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Sử dụng environment
    steps:
      - name: Deploy
        env:
          # Access environment secret
          DB_URL: ${{ secrets.DATABASE_URL }}
        run: ./deploy.sh
```

### 3. Secrets trong Reusable Workflows

```yaml
# Caller workflow
jobs:
  call-workflow:
    uses: ./.github/workflows/reusable.yml
    secrets:
      api_key: ${{ secrets.MY_API_KEY }}
      # Hoặc pass tất cả secrets
    secrets: inherit
```

```yaml
# Reusable workflow (được gọi)
on:
  workflow_call:
    secrets:
      api_key:
        required: true
        description: 'API key for service X'

jobs:
  use-secret:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Using API key"
        env:
          API_KEY: ${{ secrets.api_key }}
```

### 4. Secrets với Pull Requests từ Forks

```
┌─────────────────────────────────────────────────────────────────────┐
│            SECRETS & FORK PULL REQUESTS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PR từ FORK                          PR từ SAME REPO                │
│  ┌─────────────────┐                ┌─────────────────┐             │
│  │ `pull_request`  │                │ `pull_request`  │             │
│  │                 │                │                 │             │
│  │ Secrets: ❌ NO  │                │ Secrets: ✅ YES │             │
│  │ (Security)      │                │                 │             │
│  └─────────────────┘                └─────────────────┘             │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ `pull_request_target`                                        │   │
│  │                                                             │   │
│  │ Secrets: ✅ YES (base repo context)                         │   │
│  │ ⚠️ NGUY HIỂM nếu checkout PR code và chạy nó!              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**An toàn với `pull_request_target`:**

```yaml
# ✅ AN TOÀN - Chỉ chạy trusted code từ base branch
on:
  pull_request_target:
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v4  # Trusted action, không checkout PR code

# ❌ NGUY HIỂM - Checkout và chạy untrusted code
on:
  pull_request_target:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # Checkout PR code!
      - run: npm install  # Chạy untrusted code với access to secrets!
```

### 5. Alternatives cho Secrets

| Phương pháp | Use Case | Ví dụ |
|-------------|----------|-------|
| **Secrets** | API keys, passwords | `${{ secrets.API_KEY }}` |
| **Variables** | Non-sensitive config | `${{ vars.API_URL }}` |
| **Environment files** | Static config | `.env.production` |
| **OIDC** | Cloud authentication | AWS, GCP, Azure |

#### 5.1. Variables (Không nhạy cảm)

```yaml
# Repository → Settings → Secrets and variables → Actions → Variables
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "API URL: ${{ vars.API_URL }}"
```

**So sánh:**

| Feature | Secrets | Variables |
|---------|---------|-----------|
| Encrypted | ✅ Có | ❌ Không |
| Visible in logs | ❌ Masked | ✅ Hiển thị |
| Use for | Passwords, tokens | URLs, flags |

#### 5.2. OIDC Authentication (Không cần secrets!)

```yaml
jobs:
  deploy-to-aws:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Required for OIDC
      contents: read
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActions
          aws-region: us-east-1
          # KHÔNG cần AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY!
```

---

## 🔒 Security Best Practices

### 1. Giới hạn scope của secrets

```yaml
# ❌ KHÔNG NÊN - Secret available cho tất cả steps
env:
  API_KEY: ${{ secrets.API_KEY }}
jobs:
  build:
    steps:
      - run: npm install   # Không cần API_KEY
      - run: npm test      # Không cần API_KEY
      - run: ./deploy.sh   # Cần API_KEY

# ✅ NÊN - Secret chỉ cho step cần
jobs:
  build:
    steps:
      - run: npm install
      - run: npm test
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: ./deploy.sh
```

### 2. Rotate secrets định kỳ

```yaml
# Thêm expiry date vào secret name để nhắc nhở
# MY_API_KEY_EXPIRES_2024_06
```

### 3. Sử dụng Environment với approval

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://myapp.com
    # Cần approval từ reviewer trước khi chạy
    steps:
      - run: ./deploy.sh
```

### 4. Audit secrets usage

```bash
# Xem workflows nào dùng secrets
grep -r "secrets\." .github/workflows/
```

---

## 📊 So Sánh Các Cách Truyền Secrets

| Cách | Ví dụ | Khi nào dùng |
|------|-------|--------------|
| **Step env** | `env: KEY: ${{ secrets.X }}` | ✅ Một step cần |
| **Job env** | Đặt ở job level | Nhiều steps trong 1 job |
| **Workflow env** | Đặt ở workflow level | ⚠️ Tất cả jobs, steps |
| **Action input** | `with: token: ${{ secrets.X }}` | Passing to actions |

---

## 💡 Common Patterns

### Pattern 1: Docker Login

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Pattern 2: SSH Deploy

```yaml
- name: Deploy via SSH
  env:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  run: |
    mkdir -p ~/.ssh
    echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa
    ssh user@server "cd /app && git pull"
```

### Pattern 3: Multi-environment

```yaml
jobs:
  deploy:
    strategy:
      matrix:
        environment: [staging, production]
    environment: ${{ matrix.environment }}
    steps:
      - run: echo "Deploying to ${{ matrix.environment }}"
        env:
          # Mỗi environment có secret riêng
          API_KEY: ${{ secrets.API_KEY }}
```

### Pattern 4: Conditional secrets

```yaml
- name: Use Production Secret
  if: github.ref == 'refs/heads/main'
  env:
    API_KEY: ${{ secrets.PROD_API_KEY }}
  run: ./deploy.sh

- name: Use Staging Secret
  if: github.ref != 'refs/heads/main'
  env:
    API_KEY: ${{ secrets.STAGING_API_KEY }}
  run: ./deploy.sh
```

---

## 🔗 Tài Liệu Tham Khảo

- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
- [Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [OIDC authentication](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
