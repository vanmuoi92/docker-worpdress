# WordPress Dockerize

## Prerequisites

- Docker engine
- Docker compose
- Environment summary
  
```yaml
    project:
        - project-name: "WordPress"
        - wordpress-version: latest
    services:
        - nginx: 1.18
        - php: 7.4
        - mysql: 8.0
```

## Installation

**Required before installation**

```bash
# clone git dockerize
cd `<working_dir>`
git clone https://gitlab.com/coffeemugs/docker/wordpress.git .
```

### Initialize project

#### Project have Wordpress Core

```bash
# clone your project
git clone --branch develop <your-git-url> src

# setup and restore database
bin/init
bin/mysql < src/<your-db-dump-file.sql>

# setting up the wp-config.php file
bin/config

# setup the correct domain
bin/setupdomain `<your_domain>`
```

#### Project just have wp-content

```bash
# cleaning & download WordPress Core
bin/removeall
rm -rf src/
bin/download
bin/setup

# clone project
git clone --branch develop <your-git-url> <git_folder_name>

# restore database
bin/mysql < <path_to_sql_file>

# move WordPress Core
rsync -azvh <git_folder_name>/wp-content src/
rm -rf <git_folder_name>

# update domain in database
bin/setupdomain "<old_domain:required>"
```
   
### Setup default WordPress project

```bash
# donwload WordPress Core
bin/download

# setup the correct domain
bin/setup `<your_domain:optional>`
```

### Other tips

- ERR_CERT_INVALID in Chrome on MacOS: Just make sure the page is selected (click anywhere on the background), and type `thisisunsafe`

## Custom CLI Commands

- `bin/bash`: Drop into the bash prompt of your Docker container. The `phpfpm` container should be mainly used to access the filesystem within Docker.
- `bin/cli`: Run any CLI command without going into the bash prompt. Ex. `bin/cli ls`
- `bin/clinotty`: Run any CLI command with no TTY. Ex. `bin/clinotty php -v`
- `bin/cpfromcontainer`: Copy folders or files from container to host. Ex. `bin/cpfromcontainer wp-content`
- `bin/cptocontainer`: Copy folders or files from host to container. Ex. `bin/cptocontainer --all`
- `bin/down`: docker-compose down all containters
- `bin/download`: Download WordPress core and create `wp-config.php` with WP-CLI. Ex. `bin/download` to install latest version or enter specifical version `bin/download 5.9.3`
- `bin/fixowns`: This will fix filesystem ownerships within the container.
- `bin/init`: Initialize exist project (Setup database, install WP-CLI, create config file).
- `bin/initenv`: Initialize environment files.
- `bin/mysql`: Run the MySQL CLI with database config from `env/db.env`. Ex. `bin/mysql -e "SELECT * FROM wp_options"` or`bin/mysql < wordpress.sql`
- `bin/mysqldump`: Backup the WordPress database. Ex. `bin/mysqldump > wordpress.sql`
- `bin/remove`: Remove all containers.
- `bin/removeall`: Remove all containers, networks, volumes, and images.
- `bin/removevolumes`: Remove all volumes.
- `bin/restart`: Stop and then start all containers.
- `bin/root`: Run any CLI command as root without going into the bash prompt. Ex `bin/root apt-get install nano`
- `bin/rootnotty`: Run any CLI command as root with no TTY. Ex `bin/rootnotty chown -R app: /var/www/html`
- `bin/setup`: Run the WordPress setup process to install WordPress from the source code, with optional domain name. Defaults to `wordpress.test`. Ex. `bin/setup wordpress.test`
- `bin/setupdb`: Create database and user base on  file `db.env`
- `bin/setupdomain`: Setup WordPress domain name on local. Ex. `bin/setupdomain wordpress.test`
- `bin/start`: Start all containers, good practice to use this instead of `docker-compose up -d`, as it may contain additional helpers.
- `bin/status`: Check the container status.
- `bin/stop`: Stop container. Ex. `bin/stop phpfpm`
- `bin/stopall`: Stop all containers.
- `bin/xdebug`: Disable or enable Xdebug. Accepts params `disable` (default) or `enable`. Ex. `bin/xdebug enable`
- `bin/wpcli`: Install WP-CLI on container. Ex. `bin/wpcli` or run WP-CLI command. Ex. `bin/wpcli config list`
- `bin/wpcli-upgrade`: Updates WP-CLI to the latest release.

## Misc Info

### Database

The hostname of each service is the name of the service within the `docker-compose.yml` file. So for example, MySQL's hostname is `db` (not `localhost`) when accessing it from within a Docker container.

Change your database infomation in `env/db.env`

To connect to the MySQL CLI tool of the Docker instance, run:

```bash
bin/mysql
```

> ERROR 1045 (28000): Access denied for user 'wordpress'@'192.168.80.3' (using password: YES). Run `bin/setupdb` before run it:
>
> ```bash
> bin/setupdb
> bin/mysql
> ```
>
You can use the `bin/mysql` script to import a database, for example a file stored in your local host directory at `wordpress.sql`:

```bash
bin/mysql < wordpress.sql
```

You also can use `bin/mysqldump` to export the database. The file will appear in your local host directory at `wordpress.sql`:

```bash
bin/mysqldump > wordpress.sql
```

> Getting an "Access denied, you need (at least one of) the SUPER privilege(s) for this operation." message when running one of the above lines? Try running it as root with:
>
> ```bash
> bin/clinotty mysql -hdb -uroot -proot <your_wp_db> < src/backup.sql
> ```
>

### Email / Mailcatcher

View emails sent locally through Mailcatcher by visiting <http://{yourdomain}:1080>. Set the mailserver host to mailcatcher and port to 1080.

### phpMyAdmin

View MySQL by visiting <http://{yourdomain}:7070>.

### Xdebug & VS Code

Install and enable the PHP Debug extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).

Otherwise, this project now automatically sets up Xdebug support with VS Code. If you wish to set this up manually, please see the [`.vscode/launch.json`](https://github.com/markshust/docker-magento/blame/master/compose/.vscode/launch.json) file.

Open any *.php file, click on Run & Debug tab -> create a launch.json file

![Run & Debug tab](https://imgur.com/dZIesiZ.jpg)

Config launch.json file, change `"${workspaceFolder}/src"` by your source code folder

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}/src"
            }
        },
        {
            "name": "Launch currently open script",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "port": 0,
            "runtimeArgs": [
                "-dxdebug.start_with_request=yes"
            ],
            "env": {
                "XDEBUG_MODE": "debug,develop",
                "XDEBUG_CONFIG": "client_port=${port}"
            }
        },
        {
            "name": "Launch Built-in web server",
            "type": "php",
            "request": "launch",
            "runtimeArgs": [
                "-dxdebug.mode=debug",
                "-dxdebug.start_with_request=yes",
                "-S",
                "localhost:0"
            ],
            "program": "",
            "cwd": "${workspaceRoot}",
            "port": 9003,
            "serverReadyAction": {
                "pattern": "Development Server \\(http://localhost:([0-9]+)\\) started",
                "uriFormat": "http://localhost:%s",
                "action": "openExternally"
            }
        }
    ]
}
```

### PHP Version

Change PHP version in `docker-compose.yml` file

```yml
phpfpm:
    build: &php
        context: images/php/<version>
        args: *args
    links:
        - db
    volumes: *appvolumes
    restart: on-failure
```
