# Local Docker Development Setup (Windows Host)

This guide documents the developer workflow and commands required to set up and run the ChurchCRM development environment on a Windows host using Docker Desktop when local system constraints (e.g., missing local PHP/Composer runtimes) require compiling resources inside the container.

---

## Architecture Constraints & Strategies

1. **No Host PHP**: The standard testing setup runs builds on the host before launching minimal containers. Because there is no host-level PHP installation, we use the **Docker-in-Container Development Workflow** (`dev` profile) to build and run the entire stack (including compiling PHP/Node dependencies) internally.
2. **NPM Prepare Lifecycle Scripts**: The `package.json` includes `prepare` script lifecycle commands targeting POSIX/Bash shells. When run on Windows, these fail due to redirect and conditional differences. We bypass them using `--ignore-scripts`.
3. **Image Extensions**: We permanently baked PHP's `bcmath` extension (required by `composer.json`) into `docker/Dockerfile.churchcrm-apache-php8` so it is automatically included upon build/start.

---

## Step-by-Step Launch Procedure

Run the following commands sequentially from the root of your project directory in **PowerShell**:

### Step 1: Copy Configuration Templates
```powershell
Copy-Item docker/Config.php src/Include/Config.php
```

### Step 2: Install Node Dependencies (Bypass Hooks)
```powershell
npm ci --ignore-scripts
```

### Step 3: Clean Up Existing Docker Containers (Optional)
Ensure the Docker namespace and network are clear of any conflicting resources:
```powershell
docker compose -f docker/docker-compose.yaml --profile dev down -v
```

### Step 4: Launch and Build Dev Container Stack
```powershell
docker compose -f docker/docker-compose.yaml --profile dev up -d --build
```

### Step 5: Run Compilation inside the Container
Trigger the dependency manager (Composer) and asset bundlers (Webpack) inside the running container context:
```powershell
docker compose -f docker/docker-compose.yaml --profile dev exec -w /home/ChurchCRM webserver-dev bash -c "source /root/.nvm/nvm.sh && npm run build"
```

---

## Local URLs and Service Mappings

Once started, the following services are mapped to your local machine:

* **ChurchCRM Web Application**: [http://localhost](http://localhost) (or [http://127.0.0.1](http://127.0.0.1))
* **Adminer (Database Administration UI)**: [http://localhost:8088](http://localhost:8088)
  * *System*: MySQL
  * *Server*: `database`
  * *Username*: `churchcrm`
  * *Password*: `changeme`
  * *Database*: `churchcrm`
* **Mailpit (Fake SMTP Web Dashboard)**: [http://localhost:8025](http://localhost:8025)

### Default Application Credentials (Pre-seeded)
Since the database container is automatically initialized with the Cypress test database dump (`cypress/data/seed.sql`), the app is pre-seeded. You can log in immediately:
* **URL**: [http://localhost/](http://localhost/)
* **Username**: `admin`
* **Password**: `changeme`
