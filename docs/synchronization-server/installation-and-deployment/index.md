# Installation and Deployment

This page provides detailed instructions for installing and deploying the Kotatsu Sync Server. The server requires a MySQL database for storing user data, favorites, and reading history. For detailed configuration options after installation, see Configuration Options.

## Prerequisites
Before installing the Kotatsu Sync Server, ensure you have:

* MySQL database (version 5.7 or higher)
* For Docker deployment:
    * Docker installed (version 19.03 or higher)
* For Systemd deployment:
    * JDK 17 or higher
    * Gradle 8.3 or higher

## Database Setup
The first step in deploying the sync server is setting up the MySQL database schema:

1. Download the database.sql script from the repository
2. Run the following command to set up the schema:

``` bash
mysql -h hostname -u user database < ./database.sql
```

Replace `hostname`, `user`, and `database` with your MySQL server hostname, username, and database name.

## Deployment Methods
### Docker Deployment
Docker deployment is the recommended method for deploying the Kotatsu Sync Server. The 
[Dockerfile](https://github.com/KotatsuApp/kotatsu-syncserver/blob/master/Dockerfile) uses a multi-stage build process:

1. Build the Docker image:
```
docker build github.com/KotatsuApp/kotatsu-syncserver.git -t kotatsuapp/syncserver
```

2. Run the Docker container:
```
docker run -d -p 8081:8080 \
    -e DATABASE_HOST=your_mysql_db_host \
    -e DATABASE_USER=your_mysql_db_user \
    -e DATABASE_PASSWORD=your_mysql_db_password \
    -e DATABASE_NAME=your_mysql_db_name \
    -e DATABASE_PORT=your_mysql_db_port \
    -e JWT_SECRET=your_secret \
    -e ALLOW_NEW_REGISTER=true \
    --restart always \
    --name kotatsu-sync kotatsuapp/syncserver
```

After initial setup, if you want to prevent others from using your server instance, register your accounts first, then set `ALLOW_NEW_REGISTER` to `false` and restart the server.

### Docker Compose Deployment
For users who want to deploy both the server and database together, Docker Compose provides a convenient solution:

1. Use the provided docker-compose.yaml file or create one with the following content:
``` docker
services:
  db:
    image: mysql:8.4
    environment:
      MYSQL_USER: your_user
      MYSQL_PASSWORD: your_password
      MYSQL_DATABASE: kotatsu_db
      MYSQL_ALLOW_EMPTY_PASSWORD: true
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    ports:
      - "3306:3306"
  server:
    container_name: kotatsu-sync
    build: ./
    depends_on:
      - db
    links:
      - db
    ports:
      - "8081:8080"
    environment:
      DATABASE_HOST: db
      DATABASE_USER: your_user
      DATABASE_PASSWORD: your_password
      DATABASE_NAME: kotatsu_db
      DATABASE_PORT: 3306
      JWT_SECRET: your_secret
      ALLOW_NEW_REGISTER: true
    restart: always
```

2. Run the following command in the directory containing docker-compose.yaml:
```
docker-compose up -d
```

### Systemd Deployment
For users who prefer running the server directly on their Linux system:

1. Clone the repository and build the server:
```
git clone https://github.com/KotatsuApp/kotatsu-syncserver.git
cd kotatsu-syncserver && ./gradlew shadowJar
```

2. Edit the `kotatsu-sync.service` file, replacing the placeholder values with your actual configuration

3. Deploy the service:

``` bash
cp kotatsu-sync.service /etc/systemd/system
systemctl daemon-reload
systemctl enable kotatsu-sync
systemctl start kotatsu-sync
```

## Configuration Options
The server supports the following configuration options, which can be set as environment variables:

| Variable           | Desciption                    | Default    | Required         |
| ------------------ | ----------------------------- | ---------- | ---------------- |
| PORT               | Server listening port         | 8080       | :material-close: |
| JWT_SECRET         | Secret for signing JWT tokens | -          | :material-check: |
| DATABASE_NAME      | MySQL database name           | kotatsu_db | :material-close: |
| DATABASE_HOST      | MySQL server hostname         | localhost  | :material-close: |
| DATABASE_PORT      | MySQL server port             | 3306       | :material-close: |
| DATABASE_USER      | MySQL username                | -          | :material-check: |
| DATABASE_PASSWORD  | MySQL password                | -          | :material-check: |
| ALLOW_NEW_REGISTER | Allow new user registrations  | true       | :material-close: |

These options can be set differently depending on your deployment method:

* Docker: Use `-e` flags
* Docker Compose: Set in the `environment` section
* Systemd: Define in the Environment directives of the service file

## Post-Installation Steps
After deployment, complete these steps:

1. Verify the server is running by accessing the root endpoint (e.g., `http://your-server:8081/`)
2. Configure the Kotatsu app to use your server:
    * Go to `Options -> Settings -> Services -> Synchronization settings -> Server address`
    * Enter your server's URL
3. Register an account through the app
4. Enable synchronization in the app and select which data to sync (favorites, history, or both)

## Troubleshooting
Common issues and solutions:

* Server doesn't start: Verify all required environment variables are set correctly
* Database connection failures: Ensure your MySQL server is running and accessible
* Authentication issues: Check that `JWT_SECRET` is properly configured
* Registration fails: Confirm `ALLOW_NEW_REGISTER` is set to true

For further assistance, contact the Kotatsu App community in the 
[Telegram group](https://t.me/kotatsuapp) or create an issue on GitHub.