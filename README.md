# Jenkins CI/CD — Pipeline Notes

Pipeline-as-code notes from building CI/CD with Jenkins: every stage from checkout to production deploy.

## Contents

| File | Covers |
|---|---|
| `PipelineCodeDemo.txt` | Declarative pipeline skeleton |
| `JenkinsfileBuildTrigger.txt` | Build triggers incl. remote/webhook |
| `PAAC_Checkstyle.txt` | Static analysis stage |
| `PAAC_Sonar_Analysis.txt`, `PAAC_Sonar_QualityGates.txt` | SonarQube scan + quality gates |
| `PAAC_CI_Sonar_Nexus.txt` | Artifacts to Nexus after quality gate |
| `PAAC_CI_SlackNotification.txt` | Slack notifications per stage |
| `PAAC_CI_Docker_ECR.txt`, `PAAC_CICD_Docker_ECR_ECS.txt` | Build image → ECR → deploy to ECS |
| `CI-Jenkins.pdf` | Full course deck |

## Pipeline shape

```
checkout → build → checkstyle → sonar (gate) → nexus → docker build
        → push ECR → deploy ECS → slack notify
```
