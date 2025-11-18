# Laravel-Breeze-Railpack-Starter

> **App Laravel base** pronta per lanciare nuovi progetti su [Dokploy](https://dokploy.com) usando build Railpack, Breeze (auth scaffolding) e un database MySQL interno già collaudato.

## Stack & Features

- Laravel 12 + Breeze (Blade, Tailwind, Vite)
- Railpack build pipeline già funzionante (npx/npm + composer integrati)
- Configurazione `.env` verificata contro un database Dokploy (`PROVIAMO2`)
- Script NPM/Composer standard per sviluppo locale
- Adminer opzionale tramite Docker Compose nella stessa rete Dokploy

## Requisiti

- PHP 8.2+
- Composer 2+
- Node 22 (Railpack usa `node@22.21.1`)
- NPM 10+

## Setup locale

```bash
git clone https://github.com/<tuo-account>/Laravel-Breeze-Railpack-Starter.git
cd Laravel-Breeze-Railpack-Starter
cp .env.example .env
composer install
npm install
php artisan key:generate
php artisan migrate --seed   # opzionale
npm run dev                  # oppure npm run build
```

## Variabili ambiente (Dokploy)

Queste variabili sono già testate in produzione. Copiale nella sezione **Environment Variables** dell'app su Dokploy prima del deploy:

```
APP_NAME=Laravel
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:8zJ0obVsSwhzRkRRAPAh8TZXiTUbo2saT4Xs2GoEGoQ=
APP_URL=laravel-prova-clean-fnckoj-1084cb-164-68-111-35.traefik.me

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=info

DB_CONNECTION=mysql
DB_HOST=erer-fvl1fy
DB_PORT=3306
DB_DATABASE=testiamo_db
DB_USERNAME=testiamo_user
DB_PASSWORD=medi9810

SESSION_DRIVER=file
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null

BROADCAST_CONNECTION=log
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
CACHE_STORE=file

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=log
MAIL_SCHEME=null
MAIL_HOST=127.0.0.1
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_FROM_ADDRESS=hello@example.com
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

VITE_APP_NAME="${APP_NAME}"
```

> **Nota:** `DB_HOST` deve essere l'**Internal Host** mostrato nella scheda del database dentro Dokploy (es. `erer-fvl1fy`). Funziona solo se app e DB sono nello stesso environment.

## MySQL su Dokploy

1. Crea un servizio MySQL (es. `PROVIAMO2`) nello stesso project/environment dell'app.
2. Copia da Dokploy → *Internal Credentials* i valori `Internal Host`, `database`, `user`, `password`.
3. Incollali nelle variabili ambiente sopra.
4. Esegui un deploy Railpack (nessun `railpack.json` custom richiesto, quello già incluso usa i comandi default).

Per verificare la connessione:

```bash
php artisan migrate --force
php artisan tinker --execute="App\\Models\\User::count();"
```

## Adminer nello stesso environment

Per esplorare il DB da browser puoi avviare Adminer come servizio Compose collegato alla rete Dokploy. Sostituisci `<Internal Host>` col valore reale del DB e `<dokploy-network-name>` con il nome della rete visualizzato nella tab **Network** dell'ambiente (ad esempio `dokploy_default`).

```yaml
version: "3.8"

services:
  adminer:
    image: adminer:4.8.1
    restart: unless-stopped
    ports:
      - 8080:8080
    environment:
      ADMINER_DEFAULT_SERVER: <Internal Host>
    networks:
      - dokploy-network

networks:
  dokploy-network:
    external: true
    name: <dokploy-network-name>
```

> Adminer deve stare **nello stesso environment** dell'app Laravel e del DB, altrimenti il nome host interno non sarà risolvibile.

Una volta deployato, apri il dominio assegnato ad Adminer e usa le stesse credenziali del DB (`testiamo_user / medi9810 / testiamo_db`).

## Deploy su Dokploy (Railpack)

1. Collega il repo GitHub `Laravel-Breeze-Railpack-Starter`.
2. Build type: **Railpack**. Nessun override necessario in `railpack.json` (lascia `"commands": ["..."]`).
3. Configura le variabili ambiente di cui sopra.
4. Associa il servizio MySQL nello stesso environment.
5. Deploy. Railpack eseguirà automaticamente:
   - `composer install --no-scripts`
   - `npm ci`
   - `npm run build`
   - cache/config Laravel (config, events, route, view)

## Troubleshooting rapido

| Problema | Causa comune | Soluzione |
| --- | --- | --- |
| `php_network_getaddresses` da Adminer | Adminer non è sulla rete Dokploy dell'app | Aggiungi `networks` come nell'esempio sopra o crea Adminer come "Application" Docker nello stesso environment |
| `SQLSTATE[HY000] [2002]` | Host/porta errati nel `.env` | Usa l'Internal Host mostrato sul DB Dokploy |
| Build fallisce su `rimraf` / `.vite` | Comandi custom in `railpack.json` | Usa il file incluso (solo `"..."`) per lasciare fare tutto a Railpack |

## Licenza

MIT (come Laravel). Guarda `LICENSE`.
