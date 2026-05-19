## 2026 DevOps Gitlab 


### 1. 基础语法：`script`、`before_script`、`after_script`

**问题：** “解释一下 `.gitlab-ci.yml` 中 `before_script`、`script` 和 `after_script` 的执行顺序。如果一个 Job 在执行 `script` 中途失败，`after_script` 还会运行吗？”

**参考答案：**

- **执行顺序：** `before_script` → `script` → `after_script`
- **失败行为：** 
  - 如果 `script` 中的任何命令返回非零退出码，Job 会标记为 `failed`。
  - **但 `after_script` 仍然会执行**（**用于清理资源、发送失败通知、上传日志等**）。
- **注意：** `after_script` 中的错误**不会**改变 Job 的最终状态（即使 `after_script` 也失败了，Job 仍然是 `failed`）。


**示例：**
```yaml
job:
  before_script:
    - echo "准备环境"
  script:
    - make test
    - make build  # 假设这里失败
  after_script:
    - cleanup.sh  # 仍然会运行
```



### 2. `rules` vs `only/except`（旧语法）

**问题：** “GitLab CI 推荐使用 `rules` 而不是 `only/except`。

<mark>**请用 `rules` 实现：只在 `main` 分支且不是 `push` 事件（而是 Merge Request）时运行该 Job**。</mark>

**参考答案：**

```yaml
job:
  rules:
    - if: $CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "merge_request_event"
      when: on_success
    - when: never  # 其他情况都不运行
```

**解释重要变量：**

- **`$CI_COMMIT_BRANCH`：分支名**
- **`$CI_PIPELINE_SOURCE`：触发源（`push`、`merge_request_event`、`schedule`、`web` 等）**
- **`$CI_MERGE_REQUEST_IID`：MR 内部 ID（仅在 MR pipeline 中存在）**

**为什么淘汰 `only/except`？**  

`rules` 更灵活，支持 `variables`、允许多条件组合、且不依赖隐式逻辑。


### 3. Cache 与 Artifact 的区别

**问题：** “`cache` 和 `artifacts` 都用于保存文件，你在什么场景下使用哪一个？说出 3 个关键区别。”

**参考答案：**

| 维度 | Cache | Artifacts |
|------|-------|------------|
| **主要用途** | 加速依赖安装（如 `node_modules/`、`vendor/`） | 保存构建产物（如二进制文件、测试报告、Docker 镜像 tarball） |
| **生命周期** | **跨 Pipeline 尽力保留（可被 LRU 淘汰**） | 每个 Job 结束后必须上传到 GitLab 对象存储 |
| **默认行为** | 不自动传递给下游 Job | 后续 Job 通过 `dependencies` 或 `needs` 拉取 |
| **下载开销** | **仅在缓存未命中时下载** | 每次 Job 开始前下载（除非指定 `dependencies: []`） |

**最佳实践示例：**

```yaml
job-dependencies:
  cache:
    key: ${CI_JOB_NAME}-${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  artifacts:
    paths:
      - dist/
      - coverage/
    expire_in: 1 week  # 产物保留 7 天
    reports:
      junit: test.xml  # 让 GitLab 测试面板展示
```


### 4. Pipeline 类型：Merge Request Pipeline vs Branch Pipeline

**问题：** “你在 MR 中看到一个 Job 运行了两次（一次针对源分支，一次针对目标分支）。请问这是为什么？如何避免？”

**参考答案：**

**原因：**

- GitLab 默认在 MR 中同时触发两种 Pipeline：

  1. **Branch Pipeline**：运行在源分支（如 `feature/abc`）的最新 commit 上。
  2. **Merge Request Pipeline**：运行在模拟合并后的代码上（`$CI_MERGE_REQUEST_SOURCE_BRANCH_SHA` 合并到 `$CI_MERGE_REQUEST_TARGET_BRANCH_SHA`）。

**解决方案：** 使用 `rules` 明确只运行一种 Pipeline。

```yaml
job:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: on_success
    - when: never
```

或者更常见的做法是：

```yaml
workflow:
  rules:
    - if: $CI_MERGE_REQUEST_IID
      when: always
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: always
    - when: never
```

**注意：** Merge Request Pipeline 中可以使用 `$CI_MERGE_REQUEST_TARGET_BRANCH_NAME` 做更精细的控制。



### 5. 中级故障排查：`Job stuck` 与 `no active runner`

**问题：** “Pipeline 创建后，一个 Job 一直处于 `pending` 状态，日志显示 `This job is stuck because you don't have any active runners that can run this job.` 你如何解决？”

**参考答案：**

**排查步骤：**

1. **检查 Runner 标签 (Tags)：**  
   
   Job 可能要求特定 tag（如 `tags: ["docker"]`），但 Runner 没有该 tag。  
   → 要么移除 Job 中的 `tags:`，要么给 Runner 打上相同 tag。

2. **检查 Runner 状态：**  
   在 GitLab UI 的 `Settings` → `CI/CD` → `Runners` 中查看：
   - Runner 是否 `active`（勾选框）
   - Runner 是否被项目/群组/实例级别锁定

3. **检查 Runner Executor 类型：**  
   
   Job 可能需要 Docker 环境（`image: node:18`），但 Runner 是 `shell` executor 且未安装 Docker。  
   → 注册 Runner 时使用 `--executor docker` 或 `--executor kubernetes`。

4. **配额限制（SaaS 特有）：**  
   
   如果是 GitLab.com 免费版，月度 Pipeline 分钟数用完 → 升级或等待下月。

**快速验证：** 在本地用 `gitlab-runner exec` 模拟运行该 Job 看是否缺依赖。



### <mark>6. `needs` 与 `dependencies` 的区别</mark>  

**问题：** “`needs` 和 `dependencies` 都涉及 Job 之间的关系。什么场景用 `needs`，什么场景用 `dependencies`？”

**参考答案：**

- **`needs: [job_a, job_b]`**：  
  
**控制**执行顺序**。声明本 Job 不依赖 Stage 顺序，可以在 Stage A 的某个 Job 完成后立即开始（跳过整 stage 等待）。  
  → 用于**加速 Pipeline**，例如：Lint 和 Test 并行后，Deploy 只需要 Test 完成，不需要 Lint。**

- **`dependencies: [job_a]`**：  

**控制**哪些 artifacts 被下载**。默认 Job 会下载所有上游 Job 的 artifacts，使用 `dependencies` 可以限制只下载特定 Job 的 artifacts，节省网络带宽和磁盘空间。**

**示例：**

```yaml
stages: [build, test, deploy]

build:
  stage: build
  artifacts:
    paths: [binary/
  script: make build

test:
  stage: test
  needs: [build]  # 不等待同一 stage 的其他 job，build 完成即开始
  dependencies: [build]  # 只下载 build 的 artifact
  script: test.sh

deploy:
  stage: deploy
  dependencies: []  # 不下载任何 artifact
  script: deploy.sh
```

****常见误区：** 新手容易混淆，认为 `needs` 自动带来 `dependencies`。实际上，如果 `needs` 了一个 Job 但不加 `dependencies`，仍然会下载该 Job 的 artifacts（除非该 Job 没生成 artifacts）**。



### 附加：中级面试可能会问的实用命令

**问题：** “你在本地写了一个 `.gitlab-ci.yml`，想快速验证语法是否正确，不 push 到仓库怎么办？”

**参考答案：**

1. **使用 GitLab API（推荐）：**

```bash
   curl --request POST --header "Content-Type: application/json" \
     --data '{"content": "{ \"image\": \"ruby:2.7\", \"script\": \"echo hello\" }" }' \
     "https://gitlab.com/api/v4/projects/<project_id>/ci/lint"
```

2. **使用 GitLab CI Lint 界面：**  
   项目 → `CI/CD` → `CI Lint`（粘贴 YAML 并点击 Validate）

3. **本地工具（社区方案）：**
   
```bash
# 安装 gitlab-ci-local (第三方)
npm install -g gitlab-ci-local
gitlab-ci-local --dry-run
```

**注意：** 官方目前没有离线 CLI 校验工具，上述 API 和 UI 是最可靠的方式。



### 17 架构设计：多团队 (Multi-tenant) Pipeline 设计

**问题：** “你们有 20 个微服务团队，每个团队都想自定义 Pipeline 阶段，但你作为 Platform 团队要保证安全与效率。你会如何设计 GitLab CI 的架构？”

**参考答案：**

核心思路是 **“Pipeline as Code + Parent-Child Pipelines + 模板治理”**。

1.  **分层模板：**
    -   **Base Template (Platform 维护)：** 定义安全扫描、Artifact 上传、K8s 部署的基础 Job。这些 Job 使用 `rules` 控制触发，且只允许 `protected` 分支。
    -   **Child Templates (团队维护)：** 团队只能扩展 `pre-build` 或 `post-deploy` 阶段，但不能跳过 Security 步骤。
2.  **使用 `include` 策略：**
    -   `include: project` 指向 Platform 仓库。
    -   `include: ref` 固定为某个 tag (e.g., `v1.0.0`)，确保版本一致性，避免团队使用 `main` 分支导致不兼容。
3.  **权限隔离：**
    -   每个服务项目拥有独立的 `CI/CD Settings` -> `Token`。
    -   禁止普通开发者在 `.gitlab-ci.yml` 中写 `docker` 命令或修改 Runner Executor，强制通过 CI Lint 插件检查。

### 8 Monorepo 优化：极速 CI

**问题：** “我们的 Monorepo 有 10GB，`git clone` 就需要 5 分钟。你是 Senior Engineer，如何优化 GitLab Runner 的 checkout 速度？”

**参考答案：**
这个问题考察对 **GitLab Runner (尤其是 Autoscaling)** 和 **Git Strategy** 的深层次理解。

**方案 A：GIT_STRATEGY: clone vs fetch**

-   **默认策略优化：** 对长期存在的 Runner (如 K8s Pod)，设置 `GIT_STRATEGY: fetch`。不要每次都 `clone`，只拉取增量。
-   **致命陷阱：** 在 K8s Autoscaling 中，`fetch` 可能因为 Job 运行在新 Pod (无缓存 repo) 而失败。**解决方案：** 配置 `FF_USE_FASTZIP: true` 和分布式缓存 (S3/GCS)。

**方案 B：Git LFS + Partial Clone (高级)**

```yaml
variables:
  GIT_DEPTH: 20  # 浅克隆
  GIT_FETCH_EXTRA_FLAGS: --filter=blob:none  # Git Partial Clone (Blobless)
  GIT_STRATEGY: clone
```

-   **解释：** `--filter=blob:none` 只下载 commit 树结构，不下载大文件内容。只有在 `git checkout` 需要时才按需下载。配合 GIT_LFS_SKIP_SMUDGE: "1"，可将 clone 时间从 5 分钟降至 30 秒。

**方案 C：Cache 预热**

-   使用 `pre_get_sources_script` 从远端 (NFS/S3) 拉取一个预制的 `.git/objects` 包。

### 9 High-Level Artifact 与 Cache 策略

**问题：** “你会如何设计 CI 来管理 500+ 个 Python/Node 项目的依赖缓存与构建产物？如何避免磁盘爆满？”

**参考答案：**

Senior 必须区分 **Cache** (依赖) 与 **Artifacts** (二进制/报告)。

| 特性 | Cache | Artifacts |
| :--- | :--- | :--- |
| **生命周期** | 尽力交付，可被逐出 | 必须保留，承诺上传 |
| **用途** | `/node_modules`, `site-packages` | Docker 镜像 tarball, 测试报告 |
| **存储** | 通常是 Runner Local 或 S3 缓存 | GitLab 对象存储 (MinIO/GCS) |
| **失效策略** | `key` 规则 (如 `$CI_COMMIT_REF_SLUG`) | `expire_in: 1 hour` |

**优化策略：**

1.  **Cache Key 分层：**
    
    ```yaml
    cache:
      key: $CI_COMMIT_REF_SLUG-$CI_JOB_NAME  # 分支级
      paths: [node_modules/]
    ```
    -   对于 lockfile 变动: `key: ${CI_JOB_NAME}-${CI_COMMIT_SHORT_SHA}-${package-lock.json}`

    
2.  **防止膨胀：** 使用 `cache:policy: pull` 在测试 Job 中只拉取不推送，避免并发写入损坏 cache。
3.  **Artifact 瘦身：** 不要 `artifacts: paths: [dir/]`。使用 `artifacts: reports` (GitLab 原生支持的格式如 JUnit, Covertura) 来存储元数据而非原始文件。

### 10 故障排查：Runner 注册与 CI Lint


**问题：** “用户报错 `Job failed: prepare environment: failed to start process: exec: "docker": executable file not found in $PATH`。你作为 Platform Owner，如何在 5 分钟内解决？”

**参考答案：**

**诊断步骤：**

1.  **查看 Runner 配置 (`config.toml`)：**
    -   错误显示找不到 `docker` 二进制。这意味着 Runner 的 `executor` 是 `shell` 或 `custom`，但 Job 试图运行 `docker build`。
2.  **根本原因：** Runner 注册时使用了错误 executor。
    -   如果用户想要 `docker build`，应该使用 `executor = "docker"` 或 `"kubernetes"`。
    -   如果坚持用 `shell` executor，需要在主机上安装 Docker CLI (极其危险，属于 Docker in Docker 的乱用)。
3.  **解决方案：**
    -   **短期：** 让用户修改 `.gitlab-ci.yml`，将 script 改为 `docker build` -> 这是错误的，会循环依赖。
    -   **正确操作：** Platform 重新注册 Runner。
        ```bash
        gitlab-runner register \
          --executor docker \
          --docker-image alpine:latest \
          --docker-privileged  # 如果要用 DinD
        ```
    -   **高级修复：** 检查 `volumes` 挂载。如果是 K8s，检查 Pod Security Policy 是否允许 `privileged: true`。

### 11. Security: Secrets Management (Vault Integration)

**问题：** “团队在 `.gitlab-ci.yml` 中硬编码 `AWS_ACCESS_KEY`。作为 Platform 团队，你将如何强制实施零信任？”

**参考答案：**

使用 **GitLab JWT + HashiCorp Vault** 动态 Secret。

1.  **禁用 CI/CD 变量中的静态 Secret：** 设置项目设置 -> CI/CD -> 强制保护变量。
2.  **集成 Vault：**
    -   在 Runner 机器上配置 Vault Agent。
    -   在 Job 中，GitLab 自动注入 `CI_JOB_JWT_V2`。
    -   **配置 `secrets` 关键字：**

```yaml
        job:
          secrets:
            DATABASE_PASSWORD:
              vault:
                engine: kv-v2
                path: production/db
                field: password
              file: false  # 作为环境变量注入
```
3.  **原理：** Runner 拿着 JWT token (包含 `project_id`, `ref_type`) 去 Vault 请求。Vault 验证签名 (使用 GitLab 的公钥) 后，返回动态生成的 15 分钟有效期的数据库密码或 AWS STS Token。
4.  **审计：** 通过 GitLab Audit Events 和 Vault 的 Audit Log 双重记录谁在哪个 pipeline 拿了 key。

### 6. Senior 设计题：Multi-Region Deployment

**问题：** “设计一个 Pipeline，将 API 服务部署到 us-east-1 (蓝绿) 和 eu-west-1 (滚动)。需要 manual approval 只在第一区域前触发，且必须使用 GitLab Environment。”

**参考答案：**

利用 **父子 Pipeline (Downstream) + Needs + 环境策略**。

**1. 主文件 (`.gitlab-ci.yml`)**:

```yaml
stages: [build, deploy-us, deploy-eu]

build-image:
  stage: build
  script: docker build -t api:$CI_COMMIT_SHA .
  after_script:
    - docker push api:$CI_COMMIT_SHA

# 动态触发子流水线部署美国区
deploy-us:
  stage: deploy-us
  trigger:
    include: .gitlab/deploy-us.yml
    strategy: depend
  variables:
    REGION: us-east-1
    STRATEGY: blue-green
  environment:
    name: production/us-east-1
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual  # 手动批准才能进入美国区

deploy-eu:
  stage: deploy-eu
  trigger:
    include: .gitlab/deploy-eu.yml
  needs: ["deploy-us"]  # 自动触发，但前提是 us 完全成功
  environment:
    name: production/eu-west-1
  variables:
    STRATEGY: rolling
```

**2. 为什么这是 Senior 做法？**

-   **分离责任：** 使用 `trigger` 而不是 `parallel:matrix`，因为两个区域的部署逻辑完全不同 (蓝绿 vs 滚动)。
-   **Environment 锁定：** 将 `environment:name` 设为动态 (`production/us-east-1`)，GitLab 会自动生成 Deployment 历史视图，并且支持 `gitlab-ci.yml` 中的 `environment:action: stop` 实现回滚。
-   **变量传播：** 子 Pipeline 自动继承 `REGION` 变量，无需手动重复。

### 附加：系统工程师必知的 GitLab 内部组件

如果面试官问“Runner 挂了怎么排查”，你要展示对 **GitLab 架构** 的了解：

1.  **Sidekiq (Job Queue)：** 负责解析 `.gitlab-ci.yml`，生成 Job 放入 Redis。如果 Pipeline 卡在 `pending`，检查 Sidekiq 队列是否阻塞。
2.  **Workhorse：** 负责上传 Artifact。如果 Artifact 上传失败，检查 Workhorse -> Object Storage 的网络。
3.  **Runner Manager (`gitlab-runner` 进程)：** 轮询 GitLab API (`/api/v4/jobs/request`)。如果超时设置 (`request_concurrency`) 不对，会错过 Job。
4.  **GitLab-Runner 缓存机制：** 如果你用 S3 做分布式缓存，检查 `[runners.cache.s3]` 配置。如果 `Authentication failed`，查看 IAM Role 是否绑定了正确的 Policy (s3:PutObject)。

### 总结：Senior 的回答风格

-   **不要说：** “我会写 `docker build -t image .`”
-   **要说：** “针对 Monorepo 场景，我会设置 `GIT_DEPTH: 20` 结合 `--filter=blob:none` 减少网络 IO；同时使用 `cache:key:files` 锁定依赖版本；最后通过 `needs` 将并行度提升 400%，总耗时从 15 分钟降至 3 分钟。”
