# Multi-cloud service mapping

| Capability | AWS | Azure | GCP | Shared layer |
|---|---|---|---|---|
| Kubernetes | EKS | AKS | GKE | Helm + Argo CD |
| Database | RDS MariaDB | MariaDB workload / Azure DB option | Cloud SQL MySQL | Spring Boot + `DB_URL` |
| Registry | Docker Hub | Docker Hub | Docker Hub / Artifact Registry | OCI images |
| IaC | Terraform AWS | Terraform Azure | Terraform Google | Git |
| CI | Jenkins | Jenkins | Jenkins | One pipeline |
| Security | SonarQube + Trivy evidence | SonarQube + Trivy | SonarQube + IAM | Shift-left |
| CD | Argo CD | Argo CD | Argo CD | GitOps |
| Observability | CloudWatch / K8s metrics | Azure Monitor + Prometheus + Grafana | Cloud Monitoring + Prometheus | Actuator metrics |
