# 📚 Danh Sách Đầy Đủ GitHub Official Actions

## 📌 Tổng Quan

GitHub cung cấp các **Official Actions** dưới organization `@actions` trên GitHub. Đây là các actions được GitHub phát triển, duy trì và hỗ trợ chính thức.

**Cách nhận biết Official Actions:**
- Thuộc organization `actions/` (ví dụ: `actions/checkout`)
- Có dấu ✓ verified trên Marketplace
- Được GitHub maintain và update thường xuyên

---

## 🗂️ Phân Loại Actions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GITHUB OFFICIAL ACTIONS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📦 CORE ACTIONS              🛠️ SETUP ACTIONS                      │
│  ├─ checkout                  ├─ setup-node                        │
│  ├─ cache                     ├─ setup-python                      │
│  ├─ upload-artifact           ├─ setup-java                        │
│  ├─ download-artifact         ├─ setup-go                          │
│  ├─ github-script             ├─ setup-dotnet                      │
│  └─ configure-pages           └─ setup-ruby (ruby org)             │
│                                                                     │
│  🏷️ ISSUE/PR MANAGEMENT       🔒 SECURITY                          │
│  ├─ labeler                   ├─ codeql-action                     │
│  ├─ stale                     ├─ dependency-review-action          │
│  ├─ first-interaction         └─ attest-build-provenance          │
│  └─ add-to-project                                                 │
│                                                                     │
│  🚀 DEPLOYMENT                📊 UTILITIES                          │
│  ├─ deploy-pages              ├─ create-release                    │
│  ├─ upload-pages-artifact     ├─ toolkit                           │
│  └─ configure-pages           └─ runner-images                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# 📦 NHÓM 1: CORE ACTIONS

## 1.1. `actions/checkout` ⭐⭐⭐

**Repository:** https://github.com/actions/checkout

**Mục đích:** Checkout source code từ repository về runner

```yaml
- uses: actions/checkout@v4
  with:
    # Repository cần checkout (default: current repo)
    repository: owner/repo
    
    # Branch, tag, hoặc SHA (default: current ref)
    ref: main
    
    # Personal Access Token (default: github.token)
    token: ${{ secrets.PAT }}
    
    # Clone depth (default: 1 - shallow clone)
    fetch-depth: 0  # Full history
    
    # Checkout submodules
    submodules: recursive
    
    # Path để checkout vào
    path: my-repo
    
    # Clean working directory trước khi checkout
    clean: true
    
    # Persist credentials cho các git operations sau
    persist-credentials: true
```

**Các Inputs quan trọng:**

| Input | Default | Mô tả |
|-------|---------|-------|
| `repository` | `${{ github.repository }}` | Repository để checkout |
| `ref` | Current ref | Branch/tag/SHA |
| `token` | `${{ github.token }}` | Token để authenticate |
| `fetch-depth` | `1` | Clone depth (0 = full) |
| `submodules` | `false` | `true`/`recursive` |
| `path` | `.` | Thư mục checkout |
| `clean` | `true` | Clean trước khi checkout |
| `lfs` | `false` | Git LFS support |
| `sparse-checkout` | - | Sparse checkout patterns |

**Use Cases:**

```yaml
# 1. Basic checkout
- uses: actions/checkout@v4

# 2. Checkout specific branch
- uses: actions/checkout@v4
  with:
    ref: develop

# 3. Checkout full history (for git log, changelog)
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

# 4. Checkout another repository
- uses: actions/checkout@v4
  with:
    repository: owner/other-repo
    token: ${{ secrets.PAT }}

# 5. Checkout multiple repositories
- uses: actions/checkout@v4
  with:
    path: main-repo
- uses: actions/checkout@v4
  with:
    repository: owner/other-repo
    path: other-repo

# 6. Sparse checkout (chỉ lấy một số folders)
- uses: actions/checkout@v4
  with:
    sparse-checkout: |
      src
      tests
```

---

## 1.2. `actions/cache` ⭐⭐⭐

**Repository:** https://github.com/actions/cache

**Mục đích:** Cache dependencies và build outputs để tăng tốc workflows

```yaml
- uses: actions/cache@v4
  with:
    # Path(s) cần cache
    path: |
      ~/.npm
      node_modules
    
    # Key để identify cache
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    
    # Fallback keys nếu primary key không match
    restore-keys: |
      ${{ runner.os }}-node-
```

**Các Inputs quan trọng:**

| Input | Required | Mô tả |
|-------|----------|-------|
| `path` | ✅ | Paths để cache (multi-line) |
| `key` | ✅ | Cache key (unique identifier) |
| `restore-keys` | ❌ | Fallback keys (ordered) |
| `enableCrossOsArchive` | ❌ | Cache cross-OS |
| `fail-on-cache-miss` | ❌ | Fail nếu không có cache |
| `save-always` | ❌ | Save cache kể cả khi job fails |

**Outputs:**

| Output | Mô tả |
|--------|-------|
| `cache-hit` | `true` nếu exact match |

**Cache Key Patterns:**

```yaml
# Static key (ít thay đổi)
key: my-cache-v1

# Dynamic key với hash
key: node-${{ hashFiles('package-lock.json') }}

# Key với OS
key: ${{ runner.os }}-build-${{ hashFiles('**/*.go') }}

# Key với ngày (invalidate hàng ngày)
key: cache-${{ steps.date.outputs.date }}
```

**Ví dụ theo ngôn ngữ:**

```yaml
# Node.js / npm
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('**/package-lock.json') }}

# Node.js / yarn
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/yarn
      node_modules
    key: yarn-${{ hashFiles('**/yarn.lock') }}

# Python / pip
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: pip-${{ hashFiles('**/requirements.txt') }}

# Java / Maven
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: maven-${{ hashFiles('**/pom.xml') }}

# Java / Gradle
- uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: gradle-${{ hashFiles('**/*.gradle*') }}

# Go
- uses: actions/cache@v4
  with:
    path: ~/go/pkg/mod
    key: go-${{ hashFiles('**/go.sum') }}

# Rust / Cargo
- uses: actions/cache@v4
  with:
    path: |
      ~/.cargo/bin
      ~/.cargo/registry
      target
    key: cargo-${{ hashFiles('**/Cargo.lock') }}
```

---

## 1.3. `actions/upload-artifact` ⭐⭐⭐

**Repository:** https://github.com/actions/upload-artifact

**Mục đích:** Upload files/folders làm artifacts để share giữa jobs hoặc download sau

```yaml
- uses: actions/upload-artifact@v4
  with:
    # Tên artifact
    name: my-artifact
    
    # Path(s) cần upload
    path: |
      dist/
      build/*.zip
    
    # Số ngày giữ artifact (default: 90)
    retention-days: 7
    
    # Compression level (0-9, default: 6)
    compression-level: 9
    
    # Overwrite nếu artifact đã tồn tại
    overwrite: true
    
    # Include hidden files
    include-hidden-files: false
```

**Các Inputs quan trọng:**

| Input | Required | Default | Mô tả |
|-------|----------|---------|-------|
| `name` | ✅ | - | Tên artifact |
| `path` | ✅ | - | Paths để upload |
| `retention-days` | ❌ | 90 | Số ngày giữ |
| `compression-level` | ❌ | 6 | Mức nén (0-9) |
| `if-no-files-found` | ❌ | `warn` | `warn`/`error`/`ignore` |
| `overwrite` | ❌ | `false` | Overwrite existing |

**Outputs:**

| Output | Mô tả |
|--------|-------|
| `artifact-id` | ID của artifact |
| `artifact-url` | URL để download |

**Use Cases:**

```yaml
# 1. Upload build output
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/

# 2. Upload test results
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      test-results/
      coverage/

# 3. Upload với wildcard
- uses: actions/upload-artifact@v4
  with:
    name: logs
    path: "**/*.log"

# 4. Error nếu không có files
- uses: actions/upload-artifact@v4
  with:
    name: required-files
    path: critical/*.bin
    if-no-files-found: error
```

---

## 1.4. `actions/download-artifact` ⭐⭐⭐

**Repository:** https://github.com/actions/download-artifact

**Mục đích:** Download artifacts đã được upload trong cùng workflow

```yaml
- uses: actions/download-artifact@v4
  with:
    # Tên artifact cần download (bỏ trống = tất cả)
    name: my-artifact
    
    # Path để download vào
    path: ./downloaded
    
    # Pattern để match nhiều artifacts
    pattern: build-*
    
    # Merge tất cả vào 1 folder
    merge-multiple: true
    
    # Workflow run ID (để download từ workflow khác)
    run-id: ${{ github.event.workflow_run.id }}
```

**Các Inputs quan trọng:**

| Input | Required | Mô tả |
|-------|----------|-------|
| `name` | ❌ | Tên artifact (bỏ trống = tất cả) |
| `path` | ❌ | Thư mục destination |
| `pattern` | ❌ | Glob pattern match |
| `merge-multiple` | ❌ | Merge nhiều artifacts |
| `github-token` | ❌ | Token để download cross-workflow |
| `run-id` | ❌ | Workflow run ID |

**Use Cases:**

```yaml
# 1. Download artifact cụ thể
- uses: actions/download-artifact@v4
  with:
    name: build-output

# 2. Download tất cả artifacts
- uses: actions/download-artifact@v4

# 3. Download với pattern
- uses: actions/download-artifact@v4
  with:
    pattern: test-results-*
    path: all-results
    merge-multiple: true

# 4. Download từ workflow khác (sử dụng với workflow_run)
- uses: actions/download-artifact@v4
  with:
    name: build
    github-token: ${{ secrets.GITHUB_TOKEN }}
    run-id: ${{ github.event.workflow_run.id }}
```

**Pattern: Build rồi Deploy**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - run: ./deploy.sh dist/
```

---

## 1.5. `actions/github-script` ⭐⭐

**Repository:** https://github.com/actions/github-script

**Mục đích:** Chạy JavaScript code với access đến GitHub API qua Octokit

```yaml
- uses: actions/github-script@v7
  with:
    # Script JavaScript để chạy
    script: |
      const { data: pullRequest } = await github.rest.pulls.get({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.issue.number
      });
      console.log(pullRequest.title);
    
    # Token để authenticate
    github-token: ${{ secrets.GITHUB_TOKEN }}
    
    # Debug mode
    debug: true
    
    # Nhận kết quả
    result-encoding: string
```

**Các Objects có sẵn:**

| Object | Mô tả |
|--------|-------|
| `github` | Octokit client (authenticated) |
| `context` | GitHub context (event, repo, etc.) |
| `core` | @actions/core utilities |
| `glob` | @actions/glob |
| `io` | @actions/io |
| `exec` | @actions/exec |
| `fetch` | node-fetch |

**Use Cases:**

```yaml
# 1. Comment on PR
- uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        body: '👋 Thanks for opening this PR!'
      })

# 2. Add label
- uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.addLabels({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        labels: ['needs-review']
      })

# 3. Get and use result
- uses: actions/github-script@v7
  id: get-pr
  with:
    script: |
      const { data } = await github.rest.pulls.get({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.issue.number
      });
      return data.title;
    result-encoding: string
- run: echo "PR Title: ${{ steps.get-pr.outputs.result }}"

# 4. Close stale issues
- uses: actions/github-script@v7
  with:
    script: |
      const issues = await github.rest.issues.listForRepo({
        owner: context.repo.owner,
        repo: context.repo.repo,
        state: 'open',
        labels: 'stale'
      });
      for (const issue of issues.data) {
        await github.rest.issues.update({
          owner: context.repo.owner,
          repo: context.repo.repo,
          issue_number: issue.number,
          state: 'closed'
        });
      }

# 5. Set output
- uses: actions/github-script@v7
  id: set-result
  with:
    script: |
      core.setOutput('my-output', 'hello world');
```

---

# 🛠️ NHÓM 2: SETUP ACTIONS

## 2.1. `actions/setup-node` ⭐⭐⭐

**Repository:** https://github.com/actions/setup-node

**Mục đích:** Setup Node.js environment

```yaml
- uses: actions/setup-node@v4
  with:
    # Node.js version
    node-version: '20'
    
    # Node.js version từ file
    node-version-file: '.nvmrc'
    
    # Registry URL
    registry-url: 'https://registry.npmjs.org'
    
    # Package manager cache
    cache: 'npm'  # npm, yarn, pnpm
    
    # Cache dependency path
    cache-dependency-path: '**/package-lock.json'
```

**Các Inputs quan trọng:**

| Input | Default | Mô tả |
|-------|---------|-------|
| `node-version` | - | Version cụ thể (`18`, `20.x`, `lts/*`) |
| `node-version-file` | - | File chứa version (`.nvmrc`, `.node-version`) |
| `registry-url` | - | Registry cho publishing |
| `scope` | - | Scope cho scoped packages |
| `cache` | - | `npm`, `yarn`, `pnpm` |
| `cache-dependency-path` | - | Path đến lockfile |
| `check-latest` | `false` | Check latest version |

**Version Syntax:**

```yaml
# Exact version
node-version: '18.17.0'

# Major version (latest minor/patch)
node-version: '20'

# Major.minor (latest patch)
node-version: '18.17'

# LTS versions
node-version: 'lts/*'      # Latest LTS
node-version: 'lts/iron'   # Specific LTS codename

# From file
node-version-file: '.nvmrc'
node-version-file: 'package.json'  # Reads engines.node
```

**Use Cases:**

```yaml
# 1. Basic setup với cache
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
- run: npm ci
- run: npm test

# 2. Matrix testing
strategy:
  matrix:
    node: [18, 20, 22]
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node }}

# 3. Publish to npm
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    registry-url: 'https://registry.npmjs.org'
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

# 4. Publish to GitHub Packages
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    registry-url: 'https://npm.pkg.github.com'
    scope: '@myorg'
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 2.2. `actions/setup-python` ⭐⭐⭐

**Repository:** https://github.com/actions/setup-python

**Mục đích:** Setup Python environment

```yaml
- uses: actions/setup-python@v5
  with:
    # Python version
    python-version: '3.12'
    
    # Version từ file
    python-version-file: '.python-version'
    
    # Package manager cache
    cache: 'pip'  # pip, pipenv, poetry
    
    # Cache dependency path
    cache-dependency-path: '**/requirements*.txt'
    
    # Architecture
    architecture: 'x64'
    
    # Check latest version
    check-latest: true
    
    # Allow pre-release versions
    allow-prereleases: false
```

**Các Inputs quan trọng:**

| Input | Default | Mô tả |
|-------|---------|-------|
| `python-version` | - | Version (`3.12`, `3.x`, `pypy3.9`) |
| `python-version-file` | - | File chứa version |
| `cache` | - | `pip`, `pipenv`, `poetry` |
| `architecture` | `x64` | `x64`, `x86`, `arm64` |
| `check-latest` | `false` | Check latest version |
| `allow-prereleases` | `false` | Allow alpha/beta |

**Outputs:**

| Output | Mô tả |
|--------|-------|
| `python-version` | Installed version |
| `python-path` | Path to Python |
| `cache-hit` | Cache hit status |

**Use Cases:**

```yaml
# 1. Basic setup
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'
- run: pip install -r requirements.txt

# 2. Matrix testing
strategy:
  matrix:
    python: ['3.10', '3.11', '3.12']
steps:
  - uses: actions/setup-python@v5
    with:
      python-version: ${{ matrix.python }}

# 3. Poetry
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'poetry'
- run: |
    pip install poetry
    poetry install

# 4. PyPy
- uses: actions/setup-python@v5
  with:
    python-version: 'pypy3.10'

# 5. Multiple Python versions
- uses: actions/setup-python@v5
  with:
    python-version: |
      3.11
      3.12
# Cả 2 versions sẽ được cài, 3.12 là default
```

---

## 2.3. `actions/setup-java` ⭐⭐⭐

**Repository:** https://github.com/actions/setup-java

**Mục đích:** Setup Java JDK environment

```yaml
- uses: actions/setup-java@v4
  with:
    # Java version
    java-version: '21'
    
    # Distribution
    distribution: 'temurin'
    
    # Build tool cache
    cache: 'maven'  # maven, gradle, sbt
    
    # Package type
    java-package: 'jdk'  # jdk, jre, jdk+fx
    
    # Architecture
    architecture: 'x64'
    
    # Check latest
    check-latest: false
    
    # Maven/Gradle settings
    settings-path: ${{ github.workspace }}
```

**Các Inputs quan trọng:**

| Input | Required | Mô tả |
|-------|----------|-------|
| `java-version` | ✅ | Version (`17`, `21`, `17-ea`) |
| `distribution` | ✅ | JDK distribution |
| `cache` | ❌ | `maven`, `gradle`, `sbt` |
| `java-package` | ❌ | `jdk`, `jre`, `jdk+fx` |
| `architecture` | ❌ | `x64`, `x86`, `arm64` |

**Supported Distributions:**

| Distribution | Vendor | Mô tả |
|--------------|--------|-------|
| `temurin` | Eclipse Adoptium | ⭐ Recommended |
| `zulu` | Azul | Popular choice |
| `liberica` | BellSoft | Full/lite options |
| `microsoft` | Microsoft | Azure optimized |
| `corretto` | Amazon | AWS optimized |
| `semeru` | IBM | OpenJ9 JVM |
| `oracle` | Oracle | Oracle JDK |
| `dragonwell` | Alibaba | Alibaba Cloud |
| `sapmachine` | SAP | SAP optimized |

**Use Cases:**

```yaml
# 1. Maven project
- uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'
    cache: 'maven'
- run: mvn clean install

# 2. Gradle project
- uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'gradle'
- run: ./gradlew build

# 3. Matrix testing multiple versions
strategy:
  matrix:
    java: [11, 17, 21]
    distribution: [temurin, zulu]
steps:
  - uses: actions/setup-java@v4
    with:
      java-version: ${{ matrix.java }}
      distribution: ${{ matrix.distribution }}

# 4. Publish to Maven Central
- uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'
    server-id: ossrh
    server-username: MAVEN_USERNAME
    server-password: MAVEN_PASSWORD
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
- run: mvn deploy
  env:
    MAVEN_USERNAME: ${{ secrets.OSSRH_USERNAME }}
    MAVEN_PASSWORD: ${{ secrets.OSSRH_PASSWORD }}
```

---

## 2.4. `actions/setup-go` ⭐⭐

**Repository:** https://github.com/actions/setup-go

**Mục đích:** Setup Go environment

```yaml
- uses: actions/setup-go@v5
  with:
    # Go version
    go-version: '1.22'
    
    # Version từ file
    go-version-file: 'go.mod'
    
    # Enable module cache
    cache: true
    
    # Cache dependency path
    cache-dependency-path: '**/go.sum'
    
    # Check latest
    check-latest: false
```

**Use Cases:**

```yaml
# 1. Basic setup
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true
- run: go build ./...
- run: go test ./...

# 2. Version from go.mod
- uses: actions/setup-go@v5
  with:
    go-version-file: 'go.mod'
    cache: true

# 3. Matrix testing
strategy:
  matrix:
    go: ['1.21', '1.22']
steps:
  - uses: actions/setup-go@v5
    with:
      go-version: ${{ matrix.go }}
```

---

## 2.5. `actions/setup-dotnet` ⭐⭐

**Repository:** https://github.com/actions/setup-dotnet

**Mục đích:** Setup .NET SDK environment

```yaml
- uses: actions/setup-dotnet@v4
  with:
    # .NET SDK version
    dotnet-version: '8.x'
    
    # Version từ global.json
    global-json-file: './global.json'
    
    # Enable NuGet cache
    cache: true
    
    # NuGet source
    source-url: 'https://nuget.pkg.github.com/owner/index.json'
```

**Use Cases:**

```yaml
# 1. Basic setup
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '8.x'
    cache: true
- run: dotnet build
- run: dotnet test

# 2. Multiple versions
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: |
      6.0.x
      7.0.x
      8.0.x

# 3. Publish NuGet package
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '8.x'
- run: dotnet pack
- run: dotnet nuget push *.nupkg --api-key ${{ secrets.NUGET_KEY }}
```

---

# 🏷️ NHÓM 3: ISSUE & PR MANAGEMENT

## 3.1. `actions/labeler` ⭐⭐

**Repository:** https://github.com/actions/labeler

**Mục đích:** Tự động add labels vào PRs dựa trên files changed

```yaml
- uses: actions/labeler@v5
  with:
    # Path đến config file
    configuration-path: .github/labeler.yml
    
    # Sync labels (remove labels không match)
    sync-labels: false
    
    # Token
    repo-token: ${{ secrets.GITHUB_TOKEN }}
```

**Config file `.github/labeler.yml`:**

```yaml
# Label "documentation" cho các file docs
documentation:
  - changed-files:
      - any-glob-to-any-file: 'docs/**'

# Label "frontend" cho React files
frontend:
  - changed-files:
      - any-glob-to-any-file:
          - 'src/components/**'
          - '**/*.tsx'
          - '**/*.css'

# Label "backend" cho API files
backend:
  - changed-files:
      - any-glob-to-any-file:
          - 'api/**'
          - 'server/**'

# Label "tests" cho test files
tests:
  - changed-files:
      - any-glob-to-any-file:
          - '**/*.test.ts'
          - '**/*.spec.ts'
          - 'tests/**'

# Label "dependencies" cho package files
dependencies:
  - changed-files:
      - any-glob-to-any-file:
          - 'package.json'
          - 'package-lock.json'
          - 'requirements.txt'

# Label "ci" cho workflow files
ci:
  - changed-files:
      - any-glob-to-any-file: '.github/**'
```

**Workflow:**

```yaml
name: Labeler
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/labeler@v5
        with:
          sync-labels: true
```

---

## 3.2. `actions/stale` ⭐⭐

**Repository:** https://github.com/actions/stale

**Mục đích:** Đánh dấu và đóng issues/PRs không hoạt động

```yaml
- uses: actions/stale@v9
  with:
    # Số ngày để coi là stale
    days-before-stale: 60
    
    # Số ngày sau stale để close
    days-before-close: 7
    
    # Label khi stale
    stale-issue-label: 'stale'
    stale-pr-label: 'stale'
    
    # Message khi stale
    stale-issue-message: 'This issue is stale.'
    stale-pr-message: 'This PR is stale.'
    
    # Message khi close
    close-issue-message: 'Closing due to inactivity.'
    close-pr-message: 'Closing due to inactivity.'
    
    # Labels để exempt
    exempt-issue-labels: 'pinned,security,bug'
    exempt-pr-labels: 'pinned,security'
    
    # Limit operations per run
    operations-per-run: 100
```

**Full Workflow:**

```yaml
name: Stale Issues
on:
  schedule:
    - cron: '0 0 * * *'  # Daily
  workflow_dispatch:

jobs:
  stale:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
      - uses: actions/stale@v9
        with:
          days-before-stale: 60
          days-before-close: 7
          stale-issue-label: 'stale'
          stale-pr-label: 'stale'
          stale-issue-message: |
            This issue has been automatically marked as stale because 
            it has not had recent activity. It will be closed in 7 days
            if no further activity occurs.
          exempt-issue-labels: 'pinned,security,bug,enhancement'
          exempt-pr-labels: 'pinned,security,work-in-progress'
```

---

## 3.3. `actions/first-interaction` ⭐

**Repository:** https://github.com/actions/first-interaction

**Mục đích:** Chào mừng first-time contributors

```yaml
- uses: actions/first-interaction@v1
  with:
    repo-token: ${{ secrets.GITHUB_TOKEN }}
    
    # Message cho first issue
    issue-message: |
      👋 Thanks for opening your first issue! 
      We'll review it soon.
    
    # Message cho first PR
    pr-message: |
      🎉 Thanks for your first contribution!
      A maintainer will review your PR shortly.
```

**Workflow:**

```yaml
name: First Interaction
on:
  issues:
    types: [opened]
  pull_request_target:
    types: [opened]

jobs:
  welcome:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
      - uses: actions/first-interaction@v1
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          issue-message: |
            👋 Hi @${{ github.actor }}, thanks for opening your first issue!

            Please make sure you've:
            - [ ] Searched existing issues
            - [ ] Provided reproduction steps
            - [ ] Included environment details

          pr-message: |
            🎉 Hi @${{ github.actor }}, thanks for your first PR!

            A maintainer will review it soon. In the meantime:
            - [ ] Make sure all tests pass
            - [ ] Update documentation if needed
```

---

## 3.4. `actions/add-to-project` ⭐

**Repository:** https://github.com/actions/add-to-project

**Mục đích:** Tự động add issues/PRs vào GitHub Projects

```yaml
- uses: actions/add-to-project@v1
  with:
    # Project URL hoặc number
    project-url: https://github.com/orgs/myorg/projects/1
    
    # Token với project permissions
    github-token: ${{ secrets.PROJECT_TOKEN }}
    
    # Label filter
    labeled: bug, priority-high
    
    # Label filter logic
    label-operator: OR  # AND, OR, NOT
```

---

# 🚀 NHÓM 4: DEPLOYMENT (GitHub Pages)

## 4.1. `actions/configure-pages` ⭐⭐

**Repository:** https://github.com/actions/configure-pages

**Mục đích:** Configure GitHub Pages deployment

```yaml
- uses: actions/configure-pages@v4
  with:
    # Enable Jekyll
    enablement: true
    
    # Static site generator
    static_site_generator: next
    
    # Token
    token: ${{ secrets.GITHUB_TOKEN }}
```

---

## 4.2. `actions/upload-pages-artifact` ⭐⭐

**Repository:** https://github.com/actions/upload-pages-artifact

**Mục đích:** Upload static files cho GitHub Pages

```yaml
- uses: actions/upload-pages-artifact@v3
  with:
    # Path đến static files
    path: './dist'
    
    # Retention days
    retention-days: 1
```

---

## 4.3. `actions/deploy-pages` ⭐⭐

**Repository:** https://github.com/actions/deploy-pages

**Mục đích:** Deploy artifact đã upload lên GitHub Pages

```yaml
- uses: actions/deploy-pages@v4
  with:
    # Token với pages permissions
    token: ${{ secrets.GITHUB_TOKEN }}
    
    # Timeout (minutes)
    timeout: 600000
    
    # Error count before fail
    error_count: 10
```

**Full GitHub Pages Workflow:**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - uses: actions/configure-pages@v4
      
      - uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

---

# 🔒 NHÓM 5: SECURITY ACTIONS

## 5.1. `github/codeql-action` ⭐⭐⭐

**Repository:** https://github.com/github/codeql-action

**Mục đích:** Code scanning với CodeQL

```yaml
# Initialize CodeQL
- uses: github/codeql-action/init@v3
  with:
    languages: javascript, python
    queries: security-extended

# Auto-build (optional)
- uses: github/codeql-action/autobuild@v3

# Analyze
- uses: github/codeql-action/analyze@v3
  with:
    category: "/language:javascript"
```

**Supported Languages:**

| Language | Identifier |
|----------|------------|
| C/C++ | `cpp` |
| C# | `csharp` |
| Go | `go` |
| Java/Kotlin | `java` |
| JavaScript/TypeScript | `javascript` |
| Python | `python` |
| Ruby | `ruby` |
| Swift | `swift` |

**Full Workflow:**

```yaml
name: CodeQL Analysis

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'  # Weekly

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      actions: read
      contents: read
    
    strategy:
      matrix:
        language: [javascript, python]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-and-quality
      
      - uses: github/codeql-action/autobuild@v3
      
      - uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

---

## 5.2. `actions/dependency-review-action` ⭐⭐

**Repository:** https://github.com/actions/dependency-review-action

**Mục đích:** Review dependencies trong PRs cho vulnerabilities

```yaml
- uses: actions/dependency-review-action@v4
  with:
    # Fail on severity
    fail-on-severity: high
    
    # Allow specific licenses
    allow-licenses: MIT, Apache-2.0, BSD-3-Clause
    
    # Deny specific licenses
    deny-licenses: GPL-3.0
    
    # Allow specific advisories
    allow-ghsas: GHSA-xxxx-xxxx-xxxx
```

**Workflow:**

```yaml
name: Dependency Review

on:
  pull_request:
    branches: [main]

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: moderate
          comment-summary-in-pr: always
```

---

## 5.3. `actions/attest-build-provenance` ⭐

**Repository:** https://github.com/actions/attest-build-provenance

**Mục đích:** Tạo build provenance attestation (SLSA)

```yaml
- uses: actions/attest-build-provenance@v1
  with:
    subject-path: 'dist/my-app.tar.gz'
    subject-name: 'my-app'
```

---

# 📊 NHÓM 6: UTILITIES

## 6.1. `actions/create-release` ⭐⭐

**Repository:** https://github.com/actions/create-release

**Mục đích:** Tạo GitHub Releases

> ⚠️ **Note:** Action này đã archived. GitHub khuyến nghị dùng `softprops/action-gh-release` hoặc GitHub CLI.

**Alternative với GitHub CLI:**

```yaml
- name: Create Release
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    gh release create v1.0.0 \
      --title "Release v1.0.0" \
      --notes "Release notes here" \
      --target main \
      dist/*.zip
```

---

## 6.2. `softprops/action-gh-release` (Recommended Alternative)

**Repository:** https://github.com/softprops/action-gh-release

```yaml
- uses: softprops/action-gh-release@v2
  with:
    tag_name: v1.0.0
    name: Release v1.0.0
    body: |
      ## Changelog
      - Feature 1
      - Bug fix 1
    files: |
      dist/*.zip
      dist/*.tar.gz
    draft: false
    prerelease: false
    generate_release_notes: true
```

---

# 🔧 NHÓM 7: ACTION DEVELOPMENT

## 7.1. `actions/toolkit`

**Repository:** https://github.com/actions/toolkit

**Mục đích:** SDK để develop custom GitHub Actions

**Packages:**

| Package | Mô tả |
|---------|-------|
| `@actions/core` | Core utilities (inputs, outputs, logging) |
| `@actions/github` | Octokit client |
| `@actions/exec` | Execute commands |
| `@actions/glob` | Glob patterns |
| `@actions/io` | File system operations |
| `@actions/tool-cache` | Tool downloading/caching |
| `@actions/artifact` | Artifact upload/download |
| `@actions/cache` | Cache operations |
| `@actions/http-client` | HTTP client |

---

# 📋 Quick Reference Table

| Action | Phổ biến | Chức năng chính |
|--------|----------|-----------------|
| `actions/checkout` | ⭐⭐⭐ | Checkout code |
| `actions/cache` | ⭐⭐⭐ | Cache dependencies |
| `actions/upload-artifact` | ⭐⭐⭐ | Upload artifacts |
| `actions/download-artifact` | ⭐⭐⭐ | Download artifacts |
| `actions/setup-node` | ⭐⭐⭐ | Setup Node.js |
| `actions/setup-python` | ⭐⭐⭐ | Setup Python |
| `actions/setup-java` | ⭐⭐⭐ | Setup Java |
| `actions/setup-go` | ⭐⭐ | Setup Go |
| `actions/setup-dotnet` | ⭐⭐ | Setup .NET |
| `actions/github-script` | ⭐⭐ | Run JS with GitHub API |
| `actions/labeler` | ⭐⭐ | Auto-label PRs |
| `actions/stale` | ⭐⭐ | Mark stale issues |
| `actions/deploy-pages` | ⭐⭐ | Deploy to Pages |
| `github/codeql-action` | ⭐⭐⭐ | Security scanning |
| `actions/dependency-review-action` | ⭐⭐ | Dependency security |

---

# 🔗 Tài Liệu Tham Khảo

- [GitHub Actions Organization](https://github.com/actions)
- [GitHub Marketplace - Actions](https://github.com/marketplace?type=actions)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Actions Toolkit](https://github.com/actions/toolkit)
