# Deployment

Cujana.com wird analog zu `mpwg/symiapp.com` als rein statische Cloudflare-Pages-Seite betrieben.
Es gibt keinen Build-Schritt, keine Node-Abhängigkeiten im Repository und keine GitHub Action für das Deployment.

## Zielzustand

- Hosting: Cloudflare Pages
- Pages-Projekt: `cujana-com`
- Repository: `mpwg/cujana.com`
- Production Branch: `main`
- Framework preset: `None`
- Build command: leer lassen
- Build output directory: `/`
- Root directory: `/`
- Preview deployments: alle Branches
- Production deployments: automatisch bei Push auf `main`

Die versionierte Pages-Konfiguration liegt in [`../wrangler.jsonc`](../wrangler.jsonc):

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "cujana-com",
  "pages_build_output_dir": "./",
  "compatibility_date": "2026-04-24",
  "send_metrics": false
}
```

`pages_build_output_dir` zeigt bewusst auf `./`, weil die auszuliefernden Dateien direkt im
Repository-Root liegen.

## Einmalige Einrichtung in Cloudflare

1. Cloudflare Dashboard öffnen.
2. Zu **Workers & Pages** wechseln.
3. **Create application** auswählen.
4. **Pages** und danach **Connect to Git** wählen.
5. GitHub verbinden oder die bestehende GitHub-Integration verwenden.
6. Repository `mpwg/cujana.com` auswählen.
7. Projektname `cujana-com` setzen.
8. Build Settings setzen:

```text
Framework preset: None
Build command: leer lassen
Build output directory: /
Root directory: /
Production branch: main
```

9. Preview deployments für Branches aktiv lassen.
10. Projekt speichern und den ersten Deployment-Lauf starten.

Cloudflare erzeugt danach automatisch:

- Production Deployments bei Pushes auf `main`
- Preview Deployments für andere Branches
- Deployment-Status in GitHub
- Preview-URLs für Pull Requests

## Domain

Die Domain `cujana.com` muss im Pages-Projekt als Custom Domain verbunden sein.
Bei Cloudflare DNS ist für die Apex-Domain folgender Eintrag vorgesehen:

```text
Type: CNAME
Name: @
Target: cujana-com.pages.dev
Proxy status: Proxied
```

HTTPS wird von Cloudflare Pages nach erfolgreicher Domain-Validierung automatisch bereitgestellt.

## Kein Direct Upload

Für dieses Projekt soll die Cloudflare-Pages-Git-Integration verwendet werden, nicht Direct Upload.
Cloudflare weist darauf hin, dass ein Pages-Projekt nach der Wahl von Git Integration bzw. Direct
Upload nicht einfach auf die jeweils andere Betriebsart umgestellt werden kann. Für Cujana ist Git
Integration die passende Variante, weil Pushes auf `main` automatisch veröffentlichen sollen.

## Lokale Prüfung

Die Seite kann ohne Build lokal ausgeliefert werden:

```bash
python3 -m http.server 4173
```

Dann im Browser öffnen:

```text
http://127.0.0.1:4173/
```

Optionaler Playwright-Smoke-Test:

```bash
npx playwright --version
```

Playwright ist nur ein lokales Prüfwerkzeug. `node_modules/`, `package.json` und `package-lock.json`
werden ignoriert und gehören nicht zum statischen Deployment.

## Deployment auslösen

Ein Production Deployment wird durch Push auf `main` ausgelöst:

```bash
git push origin main
```

Für Preview Deployments genügt ein Push auf einen anderen Branch. Cloudflare erstellt dafür eine
Branch-Preview unter dem Pages-Projekt.
