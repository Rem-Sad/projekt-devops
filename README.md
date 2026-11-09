# DevOps Dyplom

Aplikacja FastAPI (lista zadań) wdrażana na AWS EC2 przez Terraform, z pipeline'em
GitHub Actions i monitoringiem Prometheus + Grafana + Alertmanager (alerty na Discord).

## Architektura

Developer -> GitHub -> GitHub Actions -> Docker Hub -> SSH deploy -> EC2 (Docker Compose)
                                                                      |- app (FastAPI)
                                                                      |- prometheus + node-exporter
                                                                      |- alertmanager -> Discord
                                                                      '- grafana


## Struktura
| Ścieżka | Zawartość |
|---|---|
| `app/` | kod aplikacji, testy, Dockerfile |
| `terraform/` | infrastruktura AWS (EC2, Security Group, Elastic IP, klucz SSH) |
| `monitoring/` | konfiguracja Prometheus, reguł alertów, Alertmanager, Grafana |
| `.github/workflows/ci-cd.yml` | pipeline CI/CD |
| `docs/` | notatki do obrony |

## Uruchomienie infrastruktury od zera (3 komendy)

cd terraform
cp terraform.tfvars.example terraform.tfvars   # wpisz swój klucz publiczny
terraform init && terraform plan && terraform apply

Ponowne `apply` nic nie zmienia (idempotentność). Wynik: `terraform output public_ip`.

## Sekrety GitHub (Settings -> Secrets -> Actions)
`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `EC2_HOST` (IP z outputu), `EC2_SSH_KEY` (klucz prywatny), `DISCORD_WEBHOOK_URL`.

## CI/CD
- commit do dowolnej gałęzi: testy -> build obrazu -> push do Docker Hub (tagi: `sha-xxxxxxx`, nazwa gałęzi)
- commit do `main`: dodatkowo deploy na EC2 i test `/healthcheck`
- zawsze: powiadomienie o wyniku na Discord

## Monitoring
Prometheus zbiera metryki aplikacji (`/metrics`) i serwera. Alerty: `AppDown`, `HighCPU` (>80% 2 min), `LowDisk` (<15%).
Grafana: `http://<IP>:3000` (admin / hasło z `GRAFANA_PASSWORD`, domyślnie `admin` - do zmiany!).

## Lokalnie

cd app && pip install -r requirements.txt && pytest && uvicorn main:app --reload

