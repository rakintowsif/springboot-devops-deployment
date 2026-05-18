# Traditional Spring Boot Deployment

This folder documents a traditional Ubuntu-based deployment for the `eSim-Backend` Spring Boot project without Docker. It includes the deployment procedure, CI/CD workflow sample, systemd service unit, and Nginx configuration.

## Project overview

- Application: `Esim`
- Repository path on server: `/var/web/html_new/dev_esim/eSim-Backend`
- Branch: `dev`
- Build tool: Maven
- Runtime: Java 17+ (OpenJDK)
- Service: `javamaven.service`

## Server prerequisites

1. Ubuntu 20.04 / 22.04 server.
2. Install required packages:
   ```bash
   sudo apt update
   sudo apt install -y openjdk-17-jdk maven git nginx
   ```
3. Ensure the GitHub Actions deploy key or SSH key is available on the GitHub repository secrets.
4. Verify the application user has permission to the deployment folder.

## Deployment directory layout

```text
/var/web/html_new/dev_esim/eSim-Backend
├── Esim
│   ├── pom.xml
│   ├── src/
│   └── target/
└── javamaven.service
```

## Traditional deployment procedure

1. Clone the repository on the Ubuntu server:
   ```bash
   sudo mkdir -p /var/web/html_new/dev_esim
   sudo chown -R $USER:$USER /var/web/html_new/dev_esim
   git clone <repo-url> /var/web/html_new/dev_esim/eSim-Backend
   ```
2. Switch to the `dev` branch and pull the latest changes:
   ```bash
   cd /var/web/html_new/dev_esim/eSim-Backend
   git checkout dev
   git pull origin dev
   ```
3. Build the Spring Boot application:
   ```bash
   cd /var/web/html_new/dev_esim/eSim-Backend/Esim
   mvn -B package --file pom.xml
   ```
4. Move the built JAR into the application folder:
   ```bash
   mv target/Esim-0.0.1-SNAPSHOT.jar /var/web/html_new/dev_esim/eSim-Backend/Esim/Esim-0.0.1-SNAPSHOT.jar
   ```
5. Reload the systemd configuration and restart the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable javamaven.service
   sudo systemctl restart javamaven.service
   ```

## Systemd service

Use the sample `javamaven.service` file in this folder and place it at `/etc/systemd/system/javamaven.service`.

## Nginx reverse-proxy configuration

A sample Nginx server block is provided in `nginx-eSim-backend.conf`. Place it in `/etc/nginx/sites-available/` and create a symlink in `/etc/nginx/sites-enabled/`.

## CI/CD workflow sample

A sample GitHub Actions workflow is provided in `cicd-dev.yml`. Copy it to `.github/workflows/dev-traditional-deployment.yml` when you are ready to enable pipeline deployment.

## Permissions and ownership

Set ownership and permissions carefully on the deployment folder:

```bash
sudo chown -R nginx:esim /var/web/html_new/dev_esim/eSim-Backend
sudo chmod -R 770 /var/web/html_new/dev_esim/eSim-Backend
sudo chmod -R g+s /var/web/html_new/dev_esim/eSim-Backend
```

## Notes

- The service file runs the JAR from the final deployment path.
- If the application listens on port `8080`, Nginx forwards requests to `http://127.0.0.1:8080`.
- Keep `dev` branch deployments separate from `main` or `staging` for safer promotion.
