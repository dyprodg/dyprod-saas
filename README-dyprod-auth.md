# Dyprod SaaS – Auth Boilerplate mit Go + Astro

## ✨ Projektziel

Ein sofort einsatzbereites SaaS-Auth-Template mit moderner JWT-Architektur (Access + Refresh Token), modular aufgebaut mit Go (Lambda Functions), Astro als UI, plus Terraform & GitHub Actions zur Automatisierung.

---

## 📂 Projektstruktur

```bash
dyprod-saas/
├── backend/
│   ├── auth/                # /auth/login, /auth/refresh, /auth/logout
│   │   └── Makefile         # Sub-Makefile für Test/Build
│   ├── hello/               # /hello – abgesicherte Demo-Funktion
│   │   └── Makefile
│   └── shared/              # JWT, DB, Middleware
│
├── frontend/                # Astro Frontend
│   ├── public/
│   └── src/
│       ├── components/
│       └── pages/
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── terraform.tfvars
│   └── modules/
│
├── .github/
│   └── workflows/
│       ├── deploy-auth.yml
│       ├── deploy-hello.yml
│       ├── deploy-frontend.yml
│       └── terraform-plan.yml
│
├── Makefile                 # zentrales Build/Deploy
└── README.md                # Diese Anleitung
```

---

## ⚙️ Features

- 🔐 JWT Auth mit Refresh Tokens (DynamoDB)
- ☁️ AWS Lambda Functions für API Calls
- 🧪 Beispiel-Route `/hello` → "Hello, "
- 🧭 GitHub Actions CI/CD
- 🛠️ Terraform-Infrastruktur
- 🎯 Astro UI optional mit Login-Anbindung

---

## 🔐 Auth Flow

```plaintext
[Login] --POST--> /auth/login 
  ↳ JWT Access (15min) + Refresh (HTTP-only Cookie)

[API-Call] --Bearer Token--> /hello
  ↳ JWT validieren → "Hello Dennis"

[Refresh] --Cookie--> /auth/refresh
  ↳ DynamoDB: Gültig? → Neues Access Token

[Logout] --POST--> /auth/logout
  ↳ Refresh Token aus DB löschen
```

---

## 🧱 Makefile Architektur

### 📄 Haupt-Makefile (`/Makefile`)

```makefile
AUTH_PATH=backend/auth
HELLO_PATH=backend/hello
FRONTEND_PATH=frontend

.PHONY: help auth hello frontend terraform test

help:
	@echo "Verfügbare Targets:"
	@echo "  make auth         -> Auth Function bauen + deployen"
	@echo "  make hello        -> Hello Function bauen + deployen"
	@echo "  make frontend     -> Astro UI lokal starten"
	@echo "  make tf-init      -> Terraform initialisieren"
	@echo "  make tf-apply     -> Terraform apply"
	@echo "  make test-auth    -> Unit-Tests in auth ausführen"

auth:
	cd $(AUTH_PATH) && make deploy

hello:
	cd $(HELLO_PATH) && make deploy

frontend:
	cd $(FRONTEND_PATH) && npm install && npm run dev

tf-init:
	cd terraform && terraform init

tf-apply:
	cd terraform && terraform apply -var-file=terraform.tfvars

test-auth:
	cd $(AUTH_PATH) && make test
```

### 📄 Sub-Makefile Beispiel (`/backend/auth/Makefile`)

```makefile
.PHONY: build test deploy

build:
	GOOS=linux GOARCH=amd64 go build -o bootstrap main.go

zip:
	zip -j function.zip bootstrap

deploy: build zip
	aws lambda update-function-code \
		--function-name dyprod-auth \
		--zip-file fileb://function.zip

test:
	go test ./...
```

---

## 🧑‍💻 Schritt-für-Schritt Setup

```bash
make tf-init
make tf-apply
make auth
make hello
make frontend
```

---

## 🗄️ Datenbank: DynamoDB – Tabelle `refresh_tokens`

```json
{
  "token": "uuid",
  "user_id": "xyz",
  "expires_at": "timestamp"
}
```

---

## 📊 Logging & Monitoring

- `log.Printf()` in jeder Go-Funktion → CloudWatch
- Terraform erstellt automatisch passende LogGroups

---

## ⚖️ GitHub Actions Beispiel: auth

```yaml
name: Deploy Auth
on:
  push:
    paths:
      - 'backend/auth/**'
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy Auth
        run: make auth
```

---

## ✏️ Erweiterungsideen

- Admin Panel mit Astro (Login & Token Handling)
- Benutzerverwaltung / Passwort Reset
- OpenAPI Docs (Swagger)
- Optional: Upstash Redis Rate Limiter

---

## 👋 Kontakt / Maintainer

Dennis Diepolder[www.dennisdiepolder.com](https://www.dennisdiepolder.com)

---

**Kopierbar. Anpassbar. SaaS-ready. Ohne Schnickschnack.**
