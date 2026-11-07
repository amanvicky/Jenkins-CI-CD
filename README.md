# Jenkins CI/CD — Pipelines as Code

Pipeline-as-code exercises from building CI/CD around the vProfile Java app: a Maven WAR project taken from checkout to a running ECS service, with quality gates that can actually stop the build.

Everything here is real Jenkinsfile code — the `tools` blocks name real configured tools (`MAVEN3`, `OracleJDK8`, `sonar4.7`), the Sonar stage sets real report paths, the Nexus uploader posts a real artifact.

## The pipeline shape

```
fetch code → build (mvn) → unit tests → checkstyle → sonar scan
          → quality gate (blocks!) → archive to Nexus → docker build
          → push to ECR → deploy to ECS → Slack notify
```

## The files, in order of a full run

| File | Stage | What's in it |
|---|---|---|
| `PipelineCodeDemo.txt` | skeleton | Declarative pipeline: `agent any`, `tools`, fetch → build → archive `**/target/*.war` → unit test |
| `JenkinsfileBuildTrigger.txt` + `Build+Triggers+Remotely.pdf` | triggers | Remote build triggering (token URL), the deck behind it |
| `PAAC_Checkstyle.txt` | static analysis | `mvn checkstyle:checkstyle` stage |
| `PAAC_Sonar_Analysis.txt` | static analysis | Sonar scanner under `withSonarQubeEnv`, feeding `surefire`, `jacoco` and checkstyle reports in |
| `PAAC_Sonar_QualityGates.txt` | gate | `waitForQualityGate abortPipeline: true` inside a 1-hour `timeout` |
| `PAAC_CI_Sonar_Nexus.txt` | full CI | The whole chain end-to-end, finishing with `nexusArtifactUploader` posting `vprofile-v2.war` to a Nexus 3 repo |
| `PAAC_CI_SlackNotification.txt` | notify | Stage-wise Slack notifications |
| `PAAC_CI_Docker_ECR.txt` | image | Build the app image, push to ECR |
| `PAAC_CICD_Docker_ECR_ECS.txt` | full CD | The complete version: build → ECR → ECS deploy |
| `CI-Jenkins.pdf` | reference | The full course deck |

## The three details worth reading

**The quality gate actually aborts.** Not a warning stage — the pipeline fails if Sonar's gate fails:

```groovy
stage("Quality Gate") {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

**The Docker stage builds from a multistage Dockerfile.** The CD pipeline's image build points at `./Docker-files/app/multistage/` — the build/runtime split lives in the Dockerfile, and the pipeline just builds and tags it:

```groovy
environment {
    registryCredential = 'ecr:us-east-2:awscreds'
    appRegistry        = "951401132355.dkr.ecr.us-east-2.amazonaws.com/vprofileappimg"
    cluster            = "vprofile"
    service            = "vprofileappsvc"
}
...
dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
```

**Every push gets two tags.** `$BUILD_NUMBER` for traceability, `latest` for convenience — then the ECS service picks it up (`cluster: vprofile`, `service: vprofileappsvc`).

## What each stage is for

Checkstyle catches style drift before Sonar sees it. Sonar adds the deeper scan (bugs, smells, coverage via the JaCoCo report). The gate is what turns those from "reports nobody reads" into a decision. Nexus archives the WAR so any build is re-deployable without rebuilding. Slack closes the loop so nobody has to watch the blue ball.

## The app end of the pipeline

The same vProfile application is what [kubernetes](https://github.com/amanvicky/kubernetes) deploys as objects (pod, replication controller, NodePort/LoadBalancer services) — CI/CD on one repo, orchestration in the other.
