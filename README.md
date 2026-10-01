# 🎓 Student Registration — Multi-Cloud DevSecOps Platform

A single **multi-cloud DevSecOps project** created by combining the AWS, Azure DevSecOps, and GCP Student Registration projects.

## Cloud footprint

- **AWS:** EKS + RDS MariaDB + VPC/NAT + ALB evidence
- **Azure:** AKS + Azure Monitor + Managed Prometheus + Managed Grafana + Trivy evidence
- **GCP:** GKE + private Cloud SQL MySQL + Artifact Registry + IAM + Cloud Monitoring
- **Shared delivery:** GitHub + Jenkins + SonarQube + Docker + Trivy + Docker Hub + Helm + Argo CD

## Architecture

```text
                           GitHub
                             |
                             v
                    +----------------+
                    |     Jenkins    |
                    | Test/Sonar/    |
                    | Docker/Trivy  |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |   Docker Hub   |
                    | frontend+API   |
                    +---+----+----+--+
                        |    |    |
             +----------+    |    +----------+
             v               v               v
          AWS EKS         Azure AKS       GCP GKE
             |               |               |
          RDS MariaDB     MariaDB/DB      Cloud SQL
             |               |               |
             +-------+-------+-------+-------+
                     |
                  Argo CD
                     |
               Helm GitOps
                     |
          Prometheus / Grafana /
          CloudWatch / Monitoring
```

## CI/CD pipeline

`GitHub → Jenkins → Maven Test → SonarQube → npm Build → Docker Build → Trivy HIGH/CRITICAL Gate → Docker Hub → update AWS/Azure/GCP Helm values → Argo CD → Kubernetes`

The important multi-cloud design decision is that the **application and container images are shared**, while infrastructure and database connection details remain cloud-specific.

## Project structure

```text
multicloud-student-registration/
├── backend/                         # shared Spring Boot API
├── frontend/                        # shared React/Vite application
├── helm/student-registration/       # shared Helm chart
├── deploy/
│   ├── values/aws.yaml
│   ├── values/azure.yaml
│   ├── values/gcp.yaml
│   └── secrets/*-secret.example.yaml
├── argocd/
│   ├── student-registration-aws.yaml
│   ├── student-registration-azure.yaml
│   └── student-registration-gcp.yaml
├── terraform/
│   ├── aws/
│   ├── azure/
│   └── gcp/
├── docs/
│   ├── cloud-mapping.md
│   ├── screenshots.md
│   └── screenshots/
├── Jenkinsfile
└── README.md
```

## Database portability

The backend accepts one environment variable:

`DB_URL`

Examples:
- AWS RDS: `jdbc:mariadb://<rds-endpoint>:3306/student_registration`
- Azure MariaDB: `jdbc:mariadb://<service>:3306/student_registration`
- GCP Cloud SQL: `jdbc:mysql://<private-ip>:3306/student_registration`

The backend includes both MariaDB and MySQL JDBC drivers so the same image can be used in all three clouds.

## Security

- SonarQube static analysis
- Trivy vulnerability and secret scanning
- HIGH/CRITICAL security gate in Jenkins
- Kubernetes Secrets for DB credentials
- Non-root backend runtime
- GitOps deployment through Argo CD
- Cloud IAM/service-account based access where supported
- No Terraform state files are included in this merged package

## Screenshots of tools

The project retains the actual screenshots supplied by the source projects. The gallery includes Terraform, Jenkins, Docker Hub, SonarQube, Trivy, Kubernetes, EKS/AKS/GKE, ALB/Ingress, Argo CD, Azure Monitor, Prometheus, Grafana, CloudWatch and the application.

Open: `docs/screenshots.md`

## Deployment notes

The Terraform directories are source implementations from the three projects and should be reviewed before applying. In particular, verify provider authentication, CIDRs, cluster names, IAM, ingress controller configuration, DNS, quotas and costs.

The Azure Terraform directory provisions the supporting DevOps/Kubernetes VM infrastructure from the original Azure project; the original workflow created AKS separately.

## Interview title

**Multi-Cloud Student Registration Platform — DevSecOps, Kubernetes & GitOps across AWS, Azure and GCP**

## Resume summary

> Designed and implemented a multi-cloud Student Registration platform across AWS EKS, Azure AKS and Google GKE using Terraform, Docker, Jenkins, SonarQube, Trivy, Helm and Argo CD. Built a cloud-neutral React/Spring Boot application with shared CI/CD and GitOps delivery, cloud-specific infrastructure adapters, secure database configuration, Kubernetes health/metrics endpoints, and Prometheus/Grafana/Cloud Monitoring observability.

**Author:** Anurag Patil
