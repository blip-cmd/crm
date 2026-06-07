# Local Docker Development Setup (Windows Host)

This guide documents the developer workflow, setup requirements, and commands to run ChurchCRM locally on a Windows host using Docker Desktop. 

Because the host machine typically lacks local PHP/Composer runtimes, all compilation (dependency installations, Propel model builds, Webpack assets compilation) runs internally inside the container context.

---

## 🚀 Quick Start (1-Line Command)

You can copy the configuration templates, build the Docker images, start all services, and compile all dependencies in one single step. Run this command in **PowerShell** from the root of your project:

```powershell
npm run docker:dev:boot
```

> [!TIP]
> The first execution will download the base images and compile the node/composer modules. Once built, subsequent runs are heavily cached and will boot in seconds.

---

## 🔗 Local URLs & Service Mappings

Once the boot process completes, access these services in your web browser:

* **ChurchCRM Web Application**: [http://localhost](http://localhost)
  * **Default Username**: `admin`
  * **Default Password**: `changeme`
* **Adminer (Database Admin GUI)**: [http://localhost:8088](http://localhost:8088)
  * *System*: `MySQL`
  * *Server*: `database`
  * *Username*: `churchcrm`
  * *Password*: `changeme`
  * *Database*: `churchcrm`
* **Mailpit (Fake SMTP Web Dashboard)**: [http://localhost:8025](http://localhost:8025)

---

## 🛠️ Step-by-Step Manual Setup

If you prefer to run the setup stages individually, execute these commands sequentially in **PowerShell**:

#### 1. Copy Configuration Template
```powershell
Copy-Item docker/Config.php src/Include/Config.php
```

#### 2. Install Node Dependencies on Windows Host (Bypass Unix hooks)
```powershell
npm ci --ignore-scripts
```

#### 3. Spin Up Dev Container Stack (Bake images)
```powershell
docker compose -f docker/docker-compose.yaml --profile dev up -d --build
```

#### 4. Run Application Compilation inside Webserver Container
```powershell
docker compose -f docker/docker-compose.yaml --profile dev exec -w /home/ChurchCRM webserver-dev bash -c "source /root/.nvm/nvm.sh && npm run build"
```

---

## 🛑 Shutting Down the Application

To shut down the local environment, you have two options in **PowerShell**:

### Option 1: Stop Containers (Preserve Container State)
Stops the running containers but keeps them created. This is faster for resuming later:
```powershell
npm run docker:dev:stop
```

### Option 2: Full Tear Down (Remove Containers & Networks)
Stops and removes the containers and network interfaces, but **keeps your database volume/data persistent**:
```powershell
npm run docker:dev:down
```

---

## 🎭 Local Production Simulation (Mimic Live Deployment)

To mimic your live production server (Oracle Cloud Infrastructure configuration using FrankenPHP and Caddy), you can run a local production stack. 

Unlike the development setup, this target runs on a minimal, secure, and production-optimized FrankenPHP base image (without development tools like Xdebug or local package compilers).

### How It Works:
1. **Asset Compilation**: Because the production container is a stripped-down image with no compiler tools (NPM/Composer), we use the development container to initially install packages and compile Webpack/Propel assets to the host directory.
2. **Execution**: The local production stack mounts the compiled assets and serves them via FrankenPHP on standard port 80.

### 1-Line Boot Command
Run this command in **PowerShell** from the root of the project to copy configuration templates, run asset compilation in dev, shut down dev, and spin up the production FrankenPHP stack:
```powershell
npm run docker:prod:boot
```

### Manual Command Sequence
If you prefer running the sequence manually:
```powershell
# 1. Compile assets inside the dev container
npm run docker:dev:boot

# 2. Stop the dev container stack
npm run docker:dev:down

# 3. Spin up the production FrankenPHP stack
npm run docker:prod:start
```

### Shutting Down the Production Mimic
To stop or fully tear down the production stack:
* **Stop**: `npm run docker:prod:stop`
* **Down**: `npm run docker:prod:down`

---

## 🔍 Troubleshooting & Known Gotchas

Here are the solutions to common issues encountered during setup on Windows hosts:

### 1. Biome Binary Mismatch Error
* **Symptom**: `Error: Cannot find module '@biomejs/cli-linux-x64/biome'` during the frontend compilation step.
* **Cause**: Running `npm ci` on a Windows host installs the Windows-compatible Biome binary. The Debian container, however, requires the Linux-specific binary.
* **Solution**: Run this command to install the Linux binary inside the webserver container without changing the host's lockfile:
  ```powershell
  docker compose -f docker/docker-compose.yaml --profile dev exec -w /home/ChurchCRM webserver-dev bash -c "source /root/.nvm/nvm.sh && npm install --no-save @biomejs/cli-linux-x64"
  ```
  Then re-run the build.

### 2. WSL2 DNS Resolution Drop (Could not resolve host: nodejs.org)
* **Symptom**: The build fails when installing NVM/Node with a network resolution timeout or DNS failure.
* **Cause**: WSL2's internal virtual network interface periodically loses DNS synchronization with the Windows host namespace.
* **Solution**: We resolved this by baking `network: host` inside the `build` configuration blocks of the `webserver-dev` and `webserver-test` services in `docker/docker-compose.yaml`. This forces Docker to share the host network space during image building.

### 3. NPM Prepare Hooks Fail on Windows
* **Symptom**: Standard `npm install` fails with Unix-like redirection and command-not-found errors.
* **Cause**: `package.json` contains prepare/lifecycle scripts structured for bash/sh shells.
* **Solution**: Always install with `--ignore-scripts` on Windows host command lines (e.g. `npm ci --ignore-scripts`).

### 4. Hard Resetting the Environment
* **Symptom**: Any inconsistent state, database locks, or corrupted volumes.
* **Solution**: Fully tear down all volumes and rebuild the stack:
  ```powershell
  docker compose -f docker/docker-compose.yaml --profile dev down -v
  npm run docker:dev:boot
  ```
