# Local Production Simulation Guide (FrankenPHP)

This guide documents how to run a local production-mimicking environment for ChurchCRM using **FrankenPHP (Caddy)**. 

Running the local production configuration allows you to test and practice the routing, Caddy configuration, and performance characteristics of your live deployment (like Oracle Cloud Infrastructure) before deploying.

---

## 🎭 Dev vs. Local Production Environments

| Feature | Development Stack (`dev`) | Production Mimic Stack (`local-prod`) |
| :--- | :--- | :--- |
| **Web Server** | Apache (`httpd`) | **FrankenPHP** (Caddy + PHP hybrid) |
| **PHP Extensions** | Includes Xdebug & Development modules | Stripped-down, production-optimized |
| **Helper Tools** | Adminer & Mailpit included | **None** (Matches live VPS deployment) |
| **Code Changes** | Auto-reload (Development) | Served over minimal Caddy rules (Production) |

---

## 🚀 Quick Boot (1-Line Command)

Because the production FrankenPHP image is a secure, minimal runtime, it lacks Node/NPM and Composer build utilities. 

To boot the production simulation, run this command in **PowerShell** from the root of the project:

```powershell
npm run docker:prod:boot
```

### What this command does automatically:
1. Copies the `docker/Config.php` configuration template into place.
2. Spins up the development container (`webserver-dev`) to download packages and compile all frontend assets (Webpack) and backend models (Propel).
3. Fully stops the dev containers.
4. Spins up the production FrankenPHP stack (`local-prod`) on port 80, serving the compiled assets.

---

## 🔗 Local URLs

Once the boot process completes, access your production-simulated environment:

* **ChurchCRM Web Application**: [http://localhost](http://localhost)
  * **Default Username**: `admin`
  * **Default Password**: `changeme`

> [!NOTE]
> Database administration tools (Adminer) and mail capture dashboards (Mailpit) are not active in the production profile to match the security baseline of a live environment.

---

## 🛑 Shutting Down Production

To manage the local production services, execute these commands in **PowerShell**:

### Stop Services (Pause Stack)
Stops the running containers without removing their networks or resource profiles:
```powershell
npm run docker:prod:stop
```

### Tear Down Services (Full Shutdown)
Stops and removes the containers and networks, keeping database volume records safe:
```powershell
npm run docker:prod:down
```

### Hard Reset (Wipe Database and Rebuild)
To wipe the database tables, clean volumes, and do a fresh build:
```powershell
docker compose -f docker/docker-compose.yaml --profile local-prod down -v
npm run docker:prod:boot
```

---

## 🛠️ Step-by-Step Manual Boot (Alternative)

If you prefer to perform each step manually:

1. **Verify configuration and compile assets:**
   ```powershell
   npm run docker:dev:boot
   ```
2. **Stop the development containers:**
   ```powershell
   npm run docker:dev:down
   ```
3. **Start the local production stack:**
   ```powershell
   npm run docker:prod:start
   ```
