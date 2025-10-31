# n8n lokal mit HTTPS (Caddy + mkcert) und PostgreSQL (macOS)

Diese Anleitung beschreibt die lokale Entwicklung von n8n hinter Caddy mit HTTPS (mkcert) und einer PostgreSQL-Datenbank auf macOS.

## Voraussetzungen
- Docker Desktop (macOS)
- Homebrew

## 1) Projekt vorbereiten
```bash
cd /Users/chris/Sites/aam_lehrgang/n8n-https-local
```

## 2) Hosts-Eintrag
Sorge dafür, dass `n8n.localhost` auf `127.0.0.1` zeigt:
```bash
echo '127.0.0.1 n8n.localhost' | sudo tee -a /etc/hosts
```

## 3) mkcert installieren und Zertifikate erzeugen
```bash
brew install mkcert
mkcert -install
mkdir -p certs
mkcert -cert-file certs/n8n.localhost.pem -key-file certs/n8n.localhost-key.pem n8n.localhost
```

## 4) .env anlegen
```bash
cp env.example .env
# Werte wie N8N_PG_PASSWORD, N8N_ENCRYPTION_KEY anpassen
```

Empfehlung für `N8N_ENCRYPTION_KEY` (>= 32 Zeichen):
```bash
LC_ALL=C tr -dc 'A-Za-z0-9' </dev/urandom | head -c 40; echo
```

## 5) Container starten
```bash
docker compose up -d --pull always
```

- Caddy liefert TLS auf `https://n8n.localhost`.
- n8n nutzt PostgreSQL (internes Netzwerk).

## 6) Nutzung
Im Browser öffnen:
```
https://n8n.localhost
```
Beim ersten Start erscheint der integrierte n8n‑Setup‑Wizard (Owner anlegen).
Falls der Browser eine Warnung zeigt, einmal komplett neu starten (`chrome://restart`).

## 7) Nützliche Kommandos
Logs streamen:
```bash
docker compose logs -f caddy n8n postgres
```
Status/Healthchecks:
```bash
docker compose ps
```
Stack stoppen:
```bash
docker compose down
```

## 8) Überblick Konfiguration
- Zertifikate: `certs/n8n.localhost.pem`, `certs/n8n.localhost-key.pem`
- Datenbank: `postgres:16-alpine` mit Volume `postgres-data`
- n8n: `DB_TYPE=postgresdb` mit `DB_POSTGRESDB_*` (aus `N8N_PG_*`)

## 9) Sicherheit (lokal)
- Setze sichere Passwörter in `.env`.
- Teile `certs/` nicht öffentlich.

### Dateien
- `docker-compose.yml` – startet `n8n`, `caddy`, `postgres`
- `Caddyfile` – Reverse-Proxy mit TLS
- `env.example` – Vorlage für `.env`
- `.gitignore` – ignoriert `.env`, Zertifikate und OS/Editor-Artefakte
