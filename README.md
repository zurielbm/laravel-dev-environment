# Laravel Sail Dev Environment (with Valet/PhpMonitor Proxy)

## 🚀 Overview

This project uses [Laravel Sail](https://laravel.com/docs/sail) for the local development environment.  
Since I already use [**Laravel Valet**](https://laravel.com/docs/valet) with [**PhpMonitor**](https://phpmon.app) ([YT Tutorial](https://youtu.be/fO3hVhkvm3w?si=t0-63IJ3cUwGr6_u)) to proxy domains locally, I configured Sail’s `docker-compose.yml` to avoid conflicts and renamed it to `docker-compose-dev.yml` to clearly separate it from a future production Docker setup.

---

## 📦 Installation & Setup

### 1. Install Laravel Sail

After creating or cloning the Laravel project:

```bash
composer require laravel/sail --dev
php artisan sail:install
```

This publishes the default `docker-compose.yml` into your project.

---

### 2. Rename Docker Compose for Development

To avoid confusion with a production file, the default compose file was renamed:

```bash
mv docker-compose.yml docker-compose-dev.yml
```

---

### 3. Alias for Sail

Since Sail defaults to looking for `docker-compose.yml`, an alias was added to point to the renamed file.  
Add this to your shell configuration (`~/.zshrc` or `~/.bashrc`):

```bash
alias sail="COMPOSE_FILE=docker-compose-dev.yml ./vendor/bin/sail"
```

Reload your shell:

```bash
source ~/.zshrc   # or ~/.bashrc
```

Now you can run Sail commands normally:

```bash
sail up -d
sail artisan migrate
sail pnpm install
sail pnpm run dev
sail pnpm run build
```

---

## ⚙️ Environment Variables

The `.env` file was updated to integrate properly with Valet/PhpMonitor proxying:

```dotenv
###########################
### PROXY REQUIREMENTS ####
### FOR DEV ENVIRONMENT ###
###########################

# Vite port mapping (LAN/dev proxy)
VITE_PORT=5000

# App URL proxied through Valet (PhpMonitor)
APP_URL=http://custom-domain.test

# Expose the Laravel app on port 8000 for proxy forwarding
APP_PORT=8000
```

This ensures:

- Laravel’s `APP_URL` matches the Valet-served domain (`custom-domain.test`)
- Vite runs on a fixed LAN port (`5000`)
- The container exposes the app on port `8000`, which Valet/PhpMonitor forwards internally

---

## 🐳 Docker Compose Dev Config

Key adjustments inside `docker-compose-dev.yml`:

- **DNS/host mapping** for Valet proxy:
  ```yaml
  extra_hosts:
    - "custom-domain.test:127.0.0.1"
  ```
- **Port mappings** to support proxy & Vite:
  ```yaml
  ports:
    - "${APP_PORT:-80}:80"
    - "${VITE_PORT:-5173}:${VITE_PORT:-5173}"
  ```

This allows PhpMonitor/Valet to catch traffic on `custom-domain.test` and route it into the container.

---

## 🖥️ Development Workflow

1. Start containers:

   ```bash
   sail up -d
   ```

2. Run migrations, seeders, etc.:

   ```bash
   sail artisan migrate
   ```

3. Start frontend dev server with hot reload:

   ```bash
   sail pnpm run dev
   ```

4. Visit the app at:
   ```
   http://custom-domain.test
   ```

---

## 🌐 Notes

- `docker-compose-dev.yml` is **for development only**.
- A separate `docker-compose-prod.yml` will be created for production deployment.
- PhpMonitor + Valet handle the domain proxying locally so the app can be accessed at `http://custom-domain.test` instead of `http://localhost:8000`.

```shell
alias sail="COMPOSE_FILE=docker-compose-dev.yml ./vendor/bin/sail"
```
