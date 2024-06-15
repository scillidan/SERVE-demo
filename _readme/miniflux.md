## Host miniflux on Windows10

### Install requirements

Download `miniflux-windows-amd64` from [releases](https://github.com/miniflux/v2/releases), rename it to `miniflux.exe`.

```sh
scoop install postgresql14 postgres
scoop install mprocs gsudo
```

### Start

```sh
initdb --locale=C --username=miniflux_user --pgdata=...\miniflux_db
postgres -D ...\miniflux_db
```

Keep the command-line window. Open `postgresql14\current\pgAdmin 4\runtime\pgAdmin4.exe`.

Right-click `Servers` → `Register/Server`:

- General
  - Name `miniflux_srv`
- Connection
  - host name `localhost`
  - Maintenance database `miniflux_db`
  - Username `miniflux_user`

Right-click `Servers/miniflux_srv/Databases` → Create → Database:
    
- General/Database → `miniflux_db`
- Definition/Encoding → `None`

Create `miniflux.conf`, set `ADMIN_USERNAME` and `ADMIN_PASSWORD` for login:

```
DATABASE_URL=user=miniflux_user password=secret dbname=miniflux_db sslmode=disable
RUN_MIGRATIONS=1
POLLING_FREQUENCY=60
CREATE_ADMIN=1
ADMIN_USERNAME=...
ADMIN_PASSWORD=...
DEBUG=on
WORKER_POOL_SIZE=10
PORT=8070
````

Create a new command-line window, test connect and login on `localhost:8070`:

```sh
miniflux -config-file ...\miniflux.conf
```

Finally, create `srv_miniflux.cmd`:

```cmd
mprocs ^
	gsudo "...\postgres.exe -D ...\var_miniflux" ^
	"timeout 5 && ...\miniflux.exe -config-file ...\miniflux.conf"
```

### How to backup data

Create `bk_miniflux.cmd`:

```sh
pg_dump -U miniflux_user -h 127.0.0.1 -p 5432 -F t miniflux_db > ...\miniflux.tar
```

Test it.

### How autorun at startup

I don't want to use `Windows Task Scheduler`. And I don't try [NSSM](https://nssm.cc/). So I just need that autorun `.cmd` at startup and hide window of the program.

Create `srv_miniflux.vbs`:

```
Set WshShell = CreateObject("WScript.Shell")
  WshShell.Run chr(34) & "...\srv_miniflux.cmd" & Chr(34), 0
Set WshShell = Nothing
```

And `bk_miniflux.vbs`:

```
Set WshShell = CreateObject("WScript.Shell")
  WshShell.Run chr(34) & "...\bk_miniflux.cmd" & Chr(34), 0
Set WshShell = Nothing
```

Create shortcut of `.vbs`. Put the `.lnk` into `C:\Users\YourName\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`.

### Third-party reader

1. `scoop install fluent-reader`
2. `localhost:8070` → Settings → API Keys → Create a new API key → `fluent-reader` → Copy the Token
3. Fluent Reader → Setting → Select a service Service → Miniflux :
    - Endpoint `http://127.0.0.1:8070`
    - Type `API Key`
    - Password `the Token`

### Some feed tools or feeds

- [hnrss](https://github.com/hnrss/hnrss)
- [GitHub Trending RSS](https://github.com/mshibanami/GitHubTrendingRSS)
- [F-Droid_Newapps_RSS](https://github.com/yzqzss/f-Droid_Newapps_RSS/)
- [github-search-rss](https://github.com/azu/github-search-rss)
- [RSSHub-Radar](https://github.com/DIYgod/RSSHub-Radar)

### Reference

- [initdb](https://www.postgresql.org/docs/current/app-initdb.html)
- [Server Dialog](https://www.pgadmin.org/docs/pgadmin4/development/server_dialog.html)
- [Configuration Parameters](https://miniflux.app/docs/configuration.html)

## Host miniflux on Ubuntu 22.04.4 Arm

↪ https://miniflux.app/docs/debian.html

```sh
cd /etc/apt/sources.list.d/miniflux.list
sudo touch miniflux.list
echo "deb [trusted=yes] https://repo.miniflux.app/apt/ * *" | sudo tee /etc/apt/sources.list.d/miniflux.list > /dev/null
sudo apt update
sudo apt install miniflux
```

↪ https://stackoverflow.com/questions/65222869/how-do-i-solve-this-problem-to-use-psql-psql-error-fatal-role-postgres-d  
↪ https://kb.objectrocket.com/postgresql/how-to-completely-uninstall-postgresql-757  
↪ https://www.postgresql.org/download/linux/ubuntu/

```sh
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt install postgresql-16
```

```sh
sudo -u postgres psql
```

↪ https://ntmv.net/posts/proper-way-to-install-miniflux/

```sh
CREATE ROLE miniflux_user LOGIN PASSWORD 'miniflux_password'
CREATE USER miniflux_user WITH ENCRYPTED PASSWORD 'miniflux_password'
CREATE DATABASE miniflux_db
GRANT ALL PRIVILEGES ON miniflux_db.* TO 'miniflux_user'@'localhost' IDENTIFIED BY 'miniflux_password'
ALTER USER miniflux_user WITH SUPERUSER
\q
```

```sh
sudo vim /etc/miniflux.conf
```

```
RUN_MIGRATIONS=1
DATABASE_URL=user=miniflux_user password=miniflux_password dbname=miniflux_db sslmode=disable
LISTEN_ADDR=/run/miniflux/miniflux.sock
PORT=8070
```

```sh
miniflux -c /etc/miniflux.conf -migrate
miniflux -c /etc/miniflux.conf -create-admin
```

```sh
miniflux -c /etc/miniflux.conf
```

If `postgresql` not run:

```sh
sudo systemctl start postgresql
```

```sh
sudo systemctl enable --now miniflux
sudo systemctl enable --now postgresql
```

### Reader Themes

↪ https://immmmm.com/hi-miniflux/  
↪ https://github.com/rootknight/Miniflux-Theme-Reeder
