---
inclusion: fileMatch
fileMatchPattern: "**/{docker-compose,Dockerfile,.gitlab-ci,.env}*"
---

# Infrastructure & Deployment

## Docker Compose Services

| Service | Base | Purpose |
|---|---|---|
| `app` | PHP-FPM | Laravel API via Nginx |
| `queue-worker` | PHP CLI | `php artisan queue:work` |
| `scheduler` | PHP CLI | `php artisan schedule:run` (cron) |
| `mysql` | MySQL 8 | Primary database |
| `redis` | Redis | Cache + queue broker |

## Queues

| Queue | Purpose | Jobs |
|---|---|---|
| `otp` | High-priority OTP delivery | SendOtpEmail |
| `emails` | Transactional emails | SendPasswordResetEmail, SendWelcomeEmail |
| `dev_alerts` | Error notifications | SendDevAlert |
| `default` | General background | LogAuditEntry, ExportAuditCsv |

## Deployment

```
GitLab Push → CI (Pint + PHPUnit + Enlightn) → Docker Build → ECR Push → ECS Deploy
```

- Region: `ap-southeast-1`
- Registry: AWS ECR
- Config: `.env` bind mount + AWS Secrets Manager
- CI/CD: `.gitlab-ci.yml`
