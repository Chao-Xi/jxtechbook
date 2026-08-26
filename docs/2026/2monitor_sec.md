# 2026 Monitoring & Logging / Security & Compliance

## 1 Monitoring & Logging

### Prometheus & Grafana (Metrics, Alerts, Visualization)

Prometheus Basics

```
global:
scrape_interval: 15s

scrape_configs:
	- job_name: 'my-app'
		static_configs:
			- targets: ['localhost:9090']
```


**Query Metrics (PromQL)**

- up → Check if a target is up
- `http_requests_total` → Total HTTP requests
- `rate(http_requests_total[5m])`` → Requests per second in the last 5 min
- `sum by (status) (rate(http_requests_total[1m])) → Request count by status

**Grafana Basics**

Data Sources:

- Prometheus → http://localhost:9090
- Elasticsearch → http://localhost:9200

Create Dashboard & Alerts

- Add Panel → Select metric
- Set Alert Conditions → Thresholds, No Data, Query Errors

### ELK Stack (Elasticsearch, Logstash, Kibana)

Elasticsearch Commands

Basic Index Operations

```
curl -X GET "localhost:9200/_cat/indices?v"
curl -X PUT "localhost:9200/my-index"
curl -X DELETE "localhost:9200/my-index"
```


Search & Query

```
curl -X GET "localhost:9200/my-index/_search?pretty"

curl -X POST "localhost:9200/my-index/_doc/1" -H "Content-Type:application/json" -d '{"name": "test"}'
```

Logstash Configuration

**logstash.conf**

```
input {
	file {
		path => "/var/log/syslog"
		start_position => "beginning"
	}
}

output {
	elasticsearch {
	hosts => ["localhost:9200"]	
	}
}
```

**Kibana Basics**



Useful Queries


- message: "error" → Search for logs with "error"
- status:[400 TO 500] → Find logs with HTTP errors

**Datadog**

Agent Installation & Setup

Install Agent (Linux)


```
DD_API_KEY=your_api_key -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
```

Log Monitoring: /etc/datadog-agent/datadog.


```
logs_enabled: true
```

Metric Queries


- avg:system.cpu.user{*} → CPU usage

- top(avg:system.disk.used{*}, 5, 'mean') → Top 5 disk users


### New Relic

**For Linux Servers**

```
curl -Ls https://download.newrelic.com/install/newrelic-cli/scripts/install.sh |
newrelic install
```

**Query Logs & Metrics**

NRQL Queries (New Relic Query Language)

```
SELECT average(cpuPercent) FROM SystemSample SINCE 30 minutes ago

SELECT count(*) FROM Transaction WHERE appName = 'my-app'
```

## 2 Security & Compliance


### 2-1 SonarQube (Code Analysis)


**Start SonarQube Server**

`./sonar.sh start`

**Run analysis with Maven**

`mvn sonar:sonar`

Run analysis with SonarScanner CLI

`sonar-scanner -Dsonar.projectKey=<project-key> -Dsonar.sources=.`


#### 1. SonarQube Integration

**Jenkins Integration**

```
groovy
	pipeline {
		agent any
		environment {
			SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
		}
	stages {
		stage('Checkout') {
			steps {
				git 'https://github.com/your-repo.git'
		}
      }
	stage('SonarQube Analysis') {
		steps {
			script {
				withSonarQubeEnv('SonarQubeServer') {
				sh 'mvn sonar:sonar'
			}
		}
      }
	}
 }
}
```

**GitLab CI/CD Integration**

```
stages:
	- code_analysis

sonarqube_scan:
	stage: code_analysis
	image: maven:3.8.7-openjdk-17

  	script:
		- mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN
    variables:
		SONAR_HOST_URL: "http://sonarqube-server:9000"
		SONAR_TOKEN: "your-sonarqube-token"
```


**GitHub Actions Integration**

```
name: SonarQube Analysis

on:
	push:
		branches:
			- main

jobs:
	sonar_scan:
	runs-on: ubuntu-latest

steps:
	- name: Checkout Code
	  uses: actions/checkout@v4

- name: Set up JDK
	uses: actions/setup-java@v3
	with:
		distribution: 'temurin'
		java-version: '17'

- name: Run SonarQube Scan
  run: mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN
  env:
	SONAR_HOST_URL: "http://sonarqube-server:9000"
	SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```


**ArgoCD Integration (PreSync Hook)**

```
apiVersion: batch/v1
kind: Job
metadata:
	name: sonarqube-analysis
	annotations:
		argocd.argoproj.io/hook: PreSync
spec:
	template:
		spec:
			containers:
				- name: sonar-scanner
				  image: maven:3.8.7-openjdk-17
				  command: ["mvn", "sonar:sonar"]
				  env:
					- name: SONAR_HOST_URL
					  value: "http://sonarqube-server:9000"
					- name: SONAR_TOKEN
					  valueFrom:
						secretKeyRef:
							name: sonar-secret
							key: sonar-token
				 restartPolicy: Never
```


### 2-2. Trivy (Container Vulnerability Scanning)

Basic Commands

**Scan a Docker image**

`trivy image <image-name>`

**Scan a Kubernetes cluster**

`trivy k8s cluster`

**Generate a JSON report**

`trivy image --format json -o report.json <image-name>`


**Jenkins Integration**


```
groovy
	pipeline {
	agent any
		stages {
			stage('Checkout') {
			steps {
				git 'https://github.com/your-repo.git'
			}
		}

stage('Trivy Scan') {
	steps {
		sh 'trivy image your-docker-image:latest
        }
	}
	}
}
```

**GitLab CI/CD Integration**


```
stages:
	- security_scan

trivy_scan:
	stage: security_scan
	image: aquasec/trivy
	script:
		- trivy image your-docker-image:latest --format json -o trivy_report.json
artifacts:
	paths:
		- trivy_report.json
```


**GitHub Actions Integration**

```
name: Trivy Scan
on:
	push:
		branches:
			- main

jobs:
	trivy_scan:
		runs-on: ubuntu-latest
		steps:
			- name: Checkout Code
			uses: actions/checkout@v4

			- name: Run Trivy Scan
				run: |
					docker pull your-docker-image:latest
					trivy image your-docker-image:latest --format json --output trivy_report.json

			- name: Upload Trivy Report
				uses: actions/upload-artifact@v4
				with:
					name: trivy-report
					path: trivy_report.json
```



**ArgoCD Integration (PreSync Hook)**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: trivy-scan
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: trivy-scanner
          image: aquasec/trivy:latest
          command:
            - trivy
            - image
            - your-docker-image:latest
      restartPolicy: Never
```

**Kubernetes Integration (Admission Controller)**


```
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: trivy-webhook

webhooks:
  - name: trivy-scan.k8s
    rules:
      - apiGroups:
          - ""
        apiVersions:
          - v1
        operations:
          - CREATE
        resources:
          - pods

    clientConfig:
      service:
        name: trivy-webhook-service
        namespace: security
        path: /validate

    admissionReviewVersions:
      - v1

    sideEffects: None
```

### 2-2 OWASP Dependency-Check (Software Dependency Analysis)

**Basic Commands**

Run a scan on a project

`./dependency-check/bin/dependency-check.sh --scan /path/to/project`

Run a scan using Maven plugin

`mvn org.owasp:dependency-check-maven:check`


**Jekins Integration**

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }
    }
}
```

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                    mvn org.owasp:dependency-check-maven:check \
                        -DfailBuildOnCVSS=7
                '''
            }
        }
    }
}
```

```
Checkout
   ↓
Build / Dependency Analysis
   ↓
OWASP Dependency Check
   ↓
Vulnerability >= threshold?
   ├── Yes → Pipeline FAILED
   └── No  → Pipeline continues
```


**GitLab CI/CD Integration**


```
stages:
  - security_scan

owasp_dependency_check:
  stage: security_scan
  image: maven:3.8.7-openjdk-17

  script:
    - mvn org.owasp:dependency-check-maven:check

  artifacts:
    paths:
      - target/dependency-check-report.html
```


```
stages:
  - security_scan

owasp_dependency_check:
  stage: security_scan
  image: maven:3.8.7-openjdk-17

  script:
    - mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7

  artifacts:
    when: always
    paths:
      - target/dependency-check-report.html
```

| 配置                    | 作用                               |
| --------------------- | -------------------------------- |
| `stages`              | 定义 Pipeline 阶段                   |
| `security_scan`       | Security Scan stage              |
| `image`               | 使用 Maven + JDK 17 的 Docker image |
| `script`              | 执行 OWASP Dependency-Check        |
| `-DfailBuildOnCVSS=7` | CVSS ≥ 7 时让 Job 失败               |
| `artifacts`           | 保存扫描报告                           |
| `when: always`        | 即使扫描失败，也保存报告                     |


**Github**

```
name: OWASP Dependency Check

on:
  push:
    branches:
      - main

jobs:
  owasp_dependency_check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Run OWASP Dependency-Check
        run: mvn org.owasp:dependency-check-maven:check

      - name: Upload OWASP Report
        uses: actions/upload-artifact@v4
        with:
          name: owasp-report
          path: target/dependency-check-report.html
```

```
Workflow
 ├── on          → 什么时候触发
 ├── jobs        → 做什么工作
 │    └── job
 │         ├── runs-on → Runner
 │         └── steps
 │              ├── checkout
 │              ├── security scan
 │              └── upload report 
```

**Argo CD PreSync Hook Job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: owasp-dependency-check
  annotations:
    argocd.argoproj.io/hook: PreSync

spec:
  template:
    spec:
      containers:
        - name: owasp-check
          image: maven:3.8.7-openjdk-17
          command:
            - mvn
            - org.owasp:dependency-check-maven:check

      restartPolicy: Never
```
 
```
apiVersion: batch/v1
kind: Job
metadata:
  name: owasp-dependency-check
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded

spec:
  backoffLimit: 0

  template:
    spec:
      containers:
        - name: owasp-check
          image: maven:3.8.7-openjdk-17
          command:
            - mvn
            - org.owasp:dependency-check-maven:check
            - -DfailBuildOnCVSS=7

      restartPolicy: Never
```


关键点：

* PreSync → 在 Argo CD 正式同步应用资源之前执行。
* command → 执行 OWASP Dependency-Check。
* -DfailBuildOnCVSS=7 → CVSS ≥ 7 时 Job 失败。
* Job 失败 → PreSync Hook 失败 → Argo CD Sync 失败/停止后续同步。
* hook-delete-policy: HookSucceeded → 成功的 Hook Job 执行完后删除。 
  




