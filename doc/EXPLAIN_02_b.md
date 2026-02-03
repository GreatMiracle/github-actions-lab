# Context Variables trong GitHub Actions - Hướng Dẫn Chi Tiết

## 📌 Tổng Quan

**Context Variables** là các biến đặc biệt trong GitHub Actions cho phép bạn **truy cập thông tin** về workflow run, môi trường, repository, và nhiều thông tin khác. Chúng được truy cập thông qua cú pháp `${{ context.property }}`.

GitHub Actions cung cấp **10 loại context chính**:

| STT | Context | Mô tả |
|-----|---------|-------|
| 1 | `github` | Thông tin về workflow run và event trigger |
| 2 | `env` | Các biến môi trường |
| 3 | `vars` | Các biến cấu hình (configuration variables) |
| 4 | `job` | Thông tin về job hiện tại |
| 5 | `jobs` | Outputs của các jobs (chỉ trong reusable workflows) |
| 6 | `steps` | Outputs của các steps đã chạy |
| 7 | `runner` | Thông tin về runner đang thực thi |
| 8 | `secrets` | Các secrets đã cấu hình |
| 9 | `strategy` | Thông tin về matrix strategy |
| 10 | `matrix` | Các giá trị matrix hiện tại |
| 11 | `needs` | Outputs của các jobs phụ thuộc |
| 12 | `inputs` | Inputs cho reusable workflows hoặc workflow_dispatch |

---

## 🔷 1. `github` Context

### 📋 Mô tả
Context `github` chứa thông tin về **workflow run** và **event** đã trigger workflow. Đây là context được sử dụng **nhiều nhất**.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `github.action` | `string` | ID của action đang chạy, hoặc ID của step nếu là `run` step | `__run`, `actions/checkout` |
| `github.action_path` | `string` | Đường dẫn đến action (chỉ cho composite actions) | `/home/runner/work/_actions/...` |
| `github.action_ref` | `string` | Ref của action đang chạy | `v4`, `main`, `a1b2c3d` |
| `github.action_repository` | `string` | Repository của action | `actions/checkout` |
| `github.action_status` | `string` | Trạng thái của composite action hiện tại | `success`, `failure` |
| `github.actor` | `string` | Username của người trigger workflow | `octocat`, `my-username` |
| `github.actor_id` | `string` | ID của actor | `583231` |
| `github.api_url` | `string` | URL của GitHub API | `https://api.github.com` |
| `github.base_ref` | `string` | Base branch của PR (chỉ cho PR events) | `main`, `develop` |
| `github.event` | `object` | Full webhook event payload | `{ "action": "opened", ... }` |
| `github.event_name` | `string` | Tên của event đã trigger | `push`, `pull_request`, `schedule` |
| `github.event_path` | `string` | Đường dẫn đến file event payload | `/github/workflow/event.json` |
| `github.graphql_url` | `string` | URL của GitHub GraphQL API | `https://api.github.com/graphql` |
| `github.head_ref` | `string` | Head branch của PR (chỉ cho PR events) | `feature/my-feature` |
| `github.job` | `string` | ID của job hiện tại | `build`, `test` |
| `github.path` | `string` | Đường dẫn đến file chứa PATH updates | `/home/runner/work/_temp/_...` |
| `github.ref` | `string` | Full ref của branch hoặc tag | `refs/heads/main`, `refs/tags/v1.0` |
| `github.ref_name` | `string` | Tên ngắn của branch hoặc tag | `main`, `v1.0.0` |
| `github.ref_protected` | `boolean` | Branch có được protect không | `true`, `false` |
| `github.ref_type` | `string` | Loại ref | `branch`, `tag` |
| `github.repository` | `string` | Tên đầy đủ của repository | `owner/repo-name` |
| `github.repository_id` | `string` | ID của repository | `123456789` |
| `github.repository_owner` | `string` | Owner của repository | `octocat` |
| `github.repository_owner_id` | `string` | ID của repository owner | `583231` |
| `github.repositoryUrl` | `string` | Git URL của repository | `git://github.com/owner/repo.git` |
| `github.retention_days` | `string` | Số ngày lưu artifacts | `90` |
| `github.run_attempt` | `string` | Số lần thử chạy (retry count) | `1`, `2`, `3` |
| `github.run_id` | `string` | ID duy nhất của workflow run | `1658821493` |
| `github.run_number` | `string` | Số thứ tự của workflow run | `1`, `42`, `100` |
| `github.secret_source` | `string` | Nguồn của secrets | `Actions`, `Dependabot`, `None` |
| `github.server_url` | `string` | URL của GitHub server | `https://github.com` |
| `github.sha` | `string` | Commit SHA đã trigger workflow | `ffac537e6cbb...` |
| `github.token` | `string` | GITHUB_TOKEN tự động tạo | `ghs_xxxx...` |
| `github.triggering_actor` | `string` | User thực sự trigger (khác actor khi re-run) | `octocat` |
| `github.workflow` | `string` | Tên của workflow | `CI Pipeline` |
| `github.workflow_ref` | `string` | Ref path đầy đủ của workflow | `owner/repo/.github/workflows/ci.yml@refs/heads/main` |
| `github.workflow_sha` | `string` | Commit SHA của workflow file | `ffac537e6cbb...` |
| `github.workspace` | `string` | Default working directory | `/home/runner/work/repo/repo` |

### 📖 Các giá trị `github.event_name` có thể có

| Giá trị | Mô tả | Khi nào trigger |
|---------|-------|-----------------|
| `push` | Push commits | Khi push code lên repository |
| `pull_request` | Pull Request events | Khi tạo, update, close PR |
| `pull_request_target` | PR events (chạy trong context của base) | PR từ fork cần write access |
| `pull_request_review` | PR review events | Khi submit review |
| `pull_request_review_comment` | PR review comment | Khi comment trong review |
| `schedule` | Scheduled trigger | Theo cron schedule |
| `workflow_dispatch` | Manual trigger | Khi trigger thủ công từ UI/API |
| `repository_dispatch` | External trigger | Khi nhận webhook từ bên ngoài |
| `create` | Create event | Khi tạo branch hoặc tag |
| `delete` | Delete event | Khi xóa branch hoặc tag |
| `release` | Release events | Khi publish, create release |
| `issues` | Issue events | Khi tạo, update, close issue |
| `issue_comment` | Issue comment | Khi comment trong issue/PR |
| `discussion` | Discussion events | Khi tạo discussion |
| `discussion_comment` | Discussion comment | Khi comment trong discussion |
| `fork` | Fork event | Khi ai đó fork repo |
| `gollum` | Wiki events | Khi update wiki page |
| `label` | Label events | Khi tạo, edit, delete label |
| `milestone` | Milestone events | Khi tạo, update milestone |
| `page_build` | GitHub Pages build | Khi Pages được build |
| `project` | Project events | Khi update project board |
| `project_card` | Project card events | Khi update project card |
| `project_column` | Project column events | Khi update project column |
| `public` | Public event | Khi repo chuyển thành public |
| `registry_package` | Package events | Khi publish package |
| `watch` | Watch event | Khi ai đó star repo |
| `workflow_call` | Reusable workflow | Khi được gọi từ workflow khác |
| `workflow_run` | Workflow run events | Khi workflow khác hoàn thành |
| `check_run` | Check run events | Khi check run được tạo/update |
| `check_suite` | Check suite events | Khi check suite được tạo/update |
| `deployment` | Deployment events | Khi tạo deployment |
| `deployment_status` | Deployment status | Khi update deployment status |
| `member` | Collaborator events | Khi thêm collaborator |
| `membership` | Team membership | Khi thay đổi team membership |
| `merge_group` | Merge queue events | Khi PR vào merge queue |
| `organization` | Org events | Khi thay đổi organization |
| `org_block` | Org block events | Khi block user từ org |
| `status` | Commit status | Khi commit status thay đổi |
| `team` | Team events | Khi tạo, edit team |
| `team_add` | Team add events | Khi thêm repo vào team |
| `branch_protection_rule` | Branch protection | Khi thay đổi protection rules |

### 📖 Các giá trị `github.ref_type` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `branch` | Ref là một branch |
| `tag` | Ref là một tag |

### 📖 Các giá trị `github.secret_source` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `Actions` | Secrets từ GitHub Actions |
| `Dependabot` | Secrets từ Dependabot |
| `None` | Không có secrets source |

### 💡 Ví dụ sử dụng

```yaml
steps:
  - name: Show GitHub Context
    run: |
      echo "Event: ${{ github.event_name }}"
      echo "Ref: ${{ github.ref }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Repository: ${{ github.repository }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Run Number: ${{ github.run_number }}"
```

---

## 🔷 2. `env` Context

### 📋 Mô tả
Context `env` chứa các **biến môi trường** được set trong workflow, job, hoặc step.

### 📊 Cách truy cập

| Cú pháp | Mô tả |
|---------|-------|
| `${{ env.MY_VAR }}` | Truy cập trong expressions |
| `$MY_VAR` | Truy cập trong shell scripts |
| `${MY_VAR}` | Truy cập trong shell scripts (explicit) |

### 📖 Các cấp độ định nghĩa

| Cấp độ | Scope | Ví dụ |
|--------|-------|-------|
| Workflow level | Tất cả jobs và steps | Đặt ở đầu workflow file |
| Job level | Tất cả steps trong job | Đặt trong `jobs.<job_id>.env` |
| Step level | Chỉ step đó | Đặt trong `steps[*].env` |

### 💡 Ví dụ sử dụng

```yaml
env:
  WORKFLOW_VAR: "workflow-level"

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: "job-level"
    steps:
      - name: Step with env
        env:
          STEP_VAR: "step-level"
        run: |
          echo "Workflow var: ${{ env.WORKFLOW_VAR }}"
          echo "Job var: $JOB_VAR"
          echo "Step var: ${STEP_VAR}"
```

---

## 🔷 3. `vars` Context

### 📋 Mô tả
Context `vars` chứa các **configuration variables** được định nghĩa ở cấp organization, repository, hoặc environment.

### 📊 Đặc điểm

| Đặc điểm | Mô tả |
|----------|-------|
| Không mã hóa | Khác với secrets, vars không được mã hóa |
| Visible trong logs | Giá trị sẽ hiện trong workflow logs |
| Multiple scopes | Có thể set ở org, repo, hoặc environment level |

### 📖 Thứ tự ưu tiên (Override)

| Ưu tiên | Level | Mô tả |
|---------|-------|-------|
| 1 (cao nhất) | Environment | Override repo và org |
| 2 | Repository | Override org |
| 3 (thấp nhất) | Organization | Default value |

### 💡 Ví dụ sử dụng

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Use configuration variables
        run: |
          echo "API URL: ${{ vars.API_URL }}"
          echo "Environment: ${{ vars.ENVIRONMENT }}"
          echo "Feature Flag: ${{ vars.ENABLE_FEATURE_X }}"
```

---

## 🔷 4. `job` Context

### 📋 Mô tả
Context `job` chứa thông tin về **job đang thực thi**.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `job.container` | `object` | Container info của job | `{ "id": "abc123", ... }` |
| `job.container.id` | `string` | ID của container | `abc123def456` |
| `job.container.network` | `string` | ID của network | `net123` |
| `job.services` | `object` | Service containers của job | `{ "redis": {...} }` |
| `job.services.<service_id>.id` | `string` | ID của service container | `redis123` |
| `job.services.<service_id>.network` | `string` | Network của service | `net123` |
| `job.services.<service_id>.ports` | `object` | Port mappings | `{ "6379": "32768" }` |
| `job.status` | `string` | Trạng thái hiện tại của job | `success`, `failure`, `cancelled` |

### 📖 Các giá trị `job.status` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `success` | Job thành công (hoặc đang thành công đến thời điểm hiện tại) |
| `failure` | Có step nào đó failed |
| `cancelled` | Job bị hủy |

### 💡 Ví dụ sử dụng

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      redis:
        image: redis
        ports:
          - 6379:6379
    steps:
      - name: Show job info
        run: |
          echo "Job status: ${{ job.status }}"
          echo "Redis port: ${{ job.services.redis.ports['6379'] }}"
```

---

## 🔷 5. `jobs` Context

### 📋 Mô tả
Context `jobs` chỉ available trong **reusable workflows** và chứa outputs của các jobs.

### 📊 Cách sử dụng

| Property | Mô tả |
|----------|-------|
| `jobs.<job_id>.result` | Kết quả của job |
| `jobs.<job_id>.outputs` | Outputs của job |
| `jobs.<job_id>.outputs.<output_name>` | Giá trị output cụ thể |

### 📖 Các giá trị `jobs.<job_id>.result` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `success` | Job hoàn thành thành công |
| `failure` | Job thất bại |
| `cancelled` | Job bị hủy |
| `skipped` | Job bị skip |

### 💡 Ví dụ sử dụng (Reusable Workflow)

```yaml
# Trong reusable workflow
on:
  workflow_call:
    outputs:
      build_result:
        description: "Build result"
        value: ${{ jobs.build.outputs.result }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.build.outputs.result }}
    steps:
      - id: build
        run: echo "result=success" >> $GITHUB_OUTPUT
```

---

## 🔷 6. `steps` Context

### 📋 Mô tả
Context `steps` chứa thông tin về các **steps đã chạy** trong job hiện tại.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `steps.<step_id>.outputs` | `object` | Outputs của step | `{ "version": "1.0.0" }` |
| `steps.<step_id>.outputs.<output_name>` | `string` | Output cụ thể | `1.0.0` |
| `steps.<step_id>.conclusion` | `string` | Kết quả sau khi `continue-on-error` | `success`, `failure` |
| `steps.<step_id>.outcome` | `string` | Kết quả trước khi `continue-on-error` | `success`, `failure` |

### 📖 Các giá trị `conclusion` và `outcome` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `success` | Step thành công |
| `failure` | Step thất bại |
| `cancelled` | Step bị hủy |
| `skipped` | Step bị skip (do `if` condition) |

### 📖 Sự khác biệt `conclusion` vs `outcome`

| Property | Khi `continue-on-error: true` và step failed |
|----------|----------------------------------------------|
| `outcome` | `failure` (kết quả thực tế) |
| `conclusion` | `success` (kết quả sau khi áp dụng continue-on-error) |

### 💡 Ví dụ sử dụng

```yaml
steps:
  - id: get-version
    run: echo "version=1.2.3" >> $GITHUB_OUTPUT
  
  - name: Use previous step output
    run: |
      echo "Version: ${{ steps.get-version.outputs.version }}"
      echo "Outcome: ${{ steps.get-version.outcome }}"
```

---

## 🔷 7. `runner` Context

### 📋 Mô tả
Context `runner` chứa thông tin về **runner** đang thực thi job.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `runner.name` | `string` | Tên của runner | `GitHub Actions 2` |
| `runner.os` | `string` | Hệ điều hành | `Linux`, `Windows`, `macOS` |
| `runner.arch` | `string` | Kiến trúc CPU | `X86`, `X64`, `ARM`, `ARM64` |
| `runner.temp` | `string` | Thư mục temp | `/home/runner/work/_temp` |
| `runner.tool_cache` | `string` | Thư mục tool cache | `/opt/hostedtoolcache` |
| `runner.debug` | `string` | Debug mode đang bật? | `1` (true) hoặc empty |
| `runner.environment` | `string` | Loại runner environment | `github-hosted`, `self-hosted` |

### 📖 Các giá trị `runner.os` có thể có

| Giá trị | Mô tả | Runner type |
|---------|-------|-------------|
| `Linux` | Ubuntu | `ubuntu-latest`, `ubuntu-22.04`, ... |
| `Windows` | Windows Server | `windows-latest`, `windows-2022`, ... |
| `macOS` | macOS | `macos-latest`, `macos-14`, ... |

### 📖 Các giá trị `runner.arch` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `X86` | 32-bit Intel/AMD |
| `X64` | 64-bit Intel/AMD |
| `ARM` | 32-bit ARM |
| `ARM64` | 64-bit ARM (Apple Silicon, etc.) |

### 📖 Các giá trị `runner.environment` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `github-hosted` | GitHub-hosted runner |
| `self-hosted` | Self-hosted runner |

### 💡 Ví dụ sử dụng

```yaml
steps:
  - name: Show runner info
    run: |
      echo "Runner Name: ${{ runner.name }}"
      echo "OS: ${{ runner.os }}"
      echo "Architecture: ${{ runner.arch }}"
      echo "Temp directory: ${{ runner.temp }}"
      
  - name: OS-specific command
    run: |
      if [ "${{ runner.os }}" == "Linux" ]; then
        echo "Running on Linux"
      elif [ "${{ runner.os }}" == "Windows" ]; then
        echo "Running on Windows"
      fi
    shell: bash
```

---

## 🔷 8. `secrets` Context

### 📋 Mô tả
Context `secrets` chứa các **secrets** được cấu hình cho repository hoặc organization.

### 📊 Đặc điểm

| Đặc điểm | Mô tả |
|----------|-------|
| Mã hóa | Secrets luôn được mã hóa |
| Masked trong logs | Tự động mask trong workflow logs |
| Case-sensitive | Tên secrets phân biệt hoa/thường |
| Không truyền qua forks | Secrets không available cho PRs từ forks |

### 📊 Built-in Secrets

| Secret | Mô tả |
|--------|-------|
| `secrets.GITHUB_TOKEN` | Token tự động tạo cho workflow |
| `secrets.ACTIONS_STEP_DEBUG` | Nếu set, bật debug logging |
| `secrets.ACTIONS_RUNNER_DEBUG` | Nếu set, bật runner diagnostic logging |

### 📖 Thứ tự ưu tiên (Override)

| Ưu tiên | Level | Mô tả |
|---------|-------|-------|
| 1 (cao nhất) | Environment | Override repo và org |
| 2 | Repository | Override org |
| 3 (thấp nhất) | Organization | Default value |

### 💡 Ví dụ sử dụng

```yaml
steps:
  - name: Deploy with tokens
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
    run: |
      echo "Deploying..."
      # Secrets sẽ tự động được mask trong logs
      
  - name: Push to Docker Hub
    run: |
      echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
```

---

## 🔷 9. `strategy` Context

### 📋 Mô tả
Context `strategy` chứa thông tin về **matrix strategy** của job hiện tại.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `strategy.fail-fast` | `boolean` | Có stop jobs khác khi 1 job fail? | `true`, `false` |
| `strategy.job-index` | `number` | Index của job trong matrix (0-based) | `0`, `1`, `2` |
| `strategy.job-total` | `number` | Tổng số jobs trong matrix | `6`, `9`, `12` |
| `strategy.max-parallel` | `number` | Số jobs tối đa chạy song song | `1`, `2`, `4` |

### 💡 Ví dụ sử dụng

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      max-parallel: 2
      matrix:
        node: [16, 18, 20]
        os: [ubuntu-latest, windows-latest]
    steps:
      - name: Show strategy info
        run: |
          echo "Fail-fast: ${{ strategy.fail-fast }}"
          echo "Job index: ${{ strategy.job-index }}"
          echo "Total jobs: ${{ strategy.job-total }}"
          echo "Max parallel: ${{ strategy.max-parallel }}"
```

---

## 🔷 10. `matrix` Context

### 📋 Mô tả
Context `matrix` chứa các **giá trị matrix** được áp dụng cho job hiện tại.

### 📊 Cách sử dụng

| Property | Mô tả |
|----------|-------|
| `matrix.<property>` | Giá trị của matrix property |
| `matrix.<property>.<nested>` | Giá trị nested (nếu là object) |

### 💡 Ví dụ sử dụng

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
        include:
          - os: ubuntu-latest
            node: 20
            experimental: true
        exclude:
          - os: windows-latest
            node: 16
    steps:
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          
      - name: Show matrix values
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node }}"
          echo "Experimental: ${{ matrix.experimental }}"
```

---

## 🔷 11. `needs` Context

### 📋 Mô tả
Context `needs` chứa thông tin về **outputs** và **kết quả** của các jobs mà job hiện tại phụ thuộc vào.

### 📊 Danh sách Properties

| Property | Kiểu dữ liệu | Mô tả | Ví dụ giá trị |
|----------|--------------|-------|---------------|
| `needs.<job_id>.outputs` | `object` | Outputs của job phụ thuộc | `{ "version": "1.0.0" }` |
| `needs.<job_id>.outputs.<name>` | `string` | Output cụ thể | `1.0.0` |
| `needs.<job_id>.result` | `string` | Kết quả của job | `success`, `failure`, `cancelled`, `skipped` |

### 📖 Các giá trị `needs.<job_id>.result` có thể có

| Giá trị | Mô tả |
|---------|-------|
| `success` | Job hoàn thành thành công |
| `failure` | Job thất bại |
| `cancelled` | Job bị hủy |
| `skipped` | Job bị skip |

### 💡 Ví dụ sử dụng

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    steps:
      - id: get-version
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Show build info
        run: |
          echo "Build result: ${{ needs.build.result }}"
          echo "Version: ${{ needs.build.outputs.version }}"

  deploy:
    needs: [build, test]
    if: needs.build.result == 'success' && needs.test.result == 'success'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy version
        run: echo "Deploying ${{ needs.build.outputs.version }}"
```

---

## 🔷 12. `inputs` Context

### 📋 Mô tả
Context `inputs` chứa các **input values** cho:
- `workflow_dispatch` - Manual trigger inputs
- `workflow_call` - Reusable workflow inputs

### 📊 Đặc điểm

| Đặc điểm | workflow_dispatch | workflow_call |
|----------|-------------------|---------------|
| Định nghĩa | Trong `on.workflow_dispatch.inputs` | Trong `on.workflow_call.inputs` |
| Truy cập | `${{ inputs.<name> }}` | `${{ inputs.<name> }}` |
| Types | string, choice, boolean, number, environment | string, boolean, number |
| Required | Có thể set | Có thể set |
| Default | Có thể set | Có thể set |

### 📖 Các Input Types cho workflow_dispatch

| Type | Mô tả | UI Element |
|------|-------|------------|
| `string` | Free text input | Text box |
| `choice` | Selection từ options | Dropdown |
| `boolean` | True/false | Checkbox |
| `number` | Numeric value | Number input |
| `environment` | GitHub environment | Environment selector |

### 📖 Các Input Types cho workflow_call

| Type | Mô tả |
|------|-------|
| `string` | String value (default) |
| `boolean` | Boolean value |
| `number` | Numeric value |

### 💡 Ví dụ sử dụng - workflow_dispatch

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: true
        type: string
        default: 'latest'
      dry_run:
        description: 'Run in dry-run mode'
        required: false
        type: boolean
        default: false
      replicas:
        description: 'Number of replicas'
        required: false
        type: number
        default: 3

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Environment: ${{ inputs.environment }}"
          echo "Version: ${{ inputs.version }}"
          echo "Dry run: ${{ inputs.dry_run }}"
          echo "Replicas: ${{ inputs.replicas }}"
```

### 💡 Ví dụ sử dụng - workflow_call (Reusable Workflow)

```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: string
      version:
        description: 'Version to deploy'
        required: false
        type: string
        default: 'latest'
    secrets:
      deploy_token:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          TOKEN: ${{ secrets.deploy_token }}
        run: |
          echo "Deploying ${{ inputs.version }} to ${{ inputs.environment }}"
```

---

## 📊 Bảng Tổng Hợp: Context Availability

| Context | Workflow | Job | Step | Notes |
|---------|----------|-----|------|-------|
| `github` | ✅ | ✅ | ✅ | Available everywhere |
| `env` | ✅ | ✅ | ✅ | Depends on where defined |
| `vars` | ✅ | ✅ | ✅ | Configuration variables |
| `job` | ❌ | ✅ | ✅ | Only in jobs |
| `jobs` | ❌ | ❌ | ✅ | Only in reusable workflows |
| `steps` | ❌ | ❌ | ✅ | Only in steps |
| `runner` | ❌ | ❌ | ✅ | Only in steps |
| `secrets` | ❌ | ✅ | ✅ | Not in workflow-level |
| `strategy` | ❌ | ✅ | ✅ | Only in matrix jobs |
| `matrix` | ❌ | ✅ | ✅ | Only in matrix jobs |
| `needs` | ❌ | ✅ | ✅ | Only when `needs` defined |
| `inputs` | ✅ | ✅ | ✅ | Only for dispatch/call |

---

## 💡 Best Practices

### 1. Luôn kiểm tra context availability

```yaml
- name: Conditional based on event
  if: github.event_name == 'push'
  run: echo "This is a push event"
```

### 2. Sử dụng default values cho inputs

```yaml
- name: Use input with default
  run: echo "Value: ${{ inputs.version || 'latest' }}"
```

### 3. Validate matrix values khi cần

```yaml
- name: Validate matrix
  if: matrix.os == 'ubuntu-latest' && matrix.node == 20
  run: echo "Running special tests"
```

### 4. Kết hợp contexts trong conditions

```yaml
if: |
  github.event_name == 'push' && 
  github.ref == 'refs/heads/main' && 
  needs.build.result == 'success'
```

### 5. Sử dụng toJson() để debug

```yaml
- name: Debug contexts
  run: |
    echo "GitHub context:"
    echo '${{ toJson(github) }}'
    echo "Matrix context:"
    echo '${{ toJson(matrix) }}'
```

---

## 🔗 Tài Liệu Tham Khảo

- [GitHub Actions Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
- [Environment Variables](https://docs.github.com/en/actions/learn-github-actions/environment-variables)
- [Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
