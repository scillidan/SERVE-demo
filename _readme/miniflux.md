# Host miniflux (on Windows10)

## Install requirements

Download `miniflux-windows-amd64` from [releases](https://github.com/miniflux/v2/releases), rename it to `miniflux.exe`.

```sh
scoop install postgresql14 postgres
scoop install mprocs gsudo
```

## Start

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

## How to backup data

Create `bk_miniflux.cmd`:

```sh
pg_dump -U miniflux_user -h 127.0.0.1 -p 5432 -F t miniflux_db > ...\miniflux.tar
```

Test it.

## How autorun at startup

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

## Third-party reader

1. `scoop install fluent-reader`
2. `localhost:8070` → Settings → API Keys → Create a new API key → `fluent-reader` → Copy the Token
3. Fluent Reader → Setting → Select a service Service → Miniflux :
    - Endpoint `http://127.0.0.1:8070`
    - Type `API Key`
    - Password `the Token`

## Some feed tools or feeds

- [hnrss](https://github.com/hnrss/hnrss)
- [GitHub Trending RSS](https://github.com/mshibanami/GitHubTrendingRSS)
- [F-Droid_Newapps_RSS](https://github.com/yzqzss/f-Droid_Newapps_RSS/)
- [github-search-rss](https://github.com/azu/github-search-rss)
- [RSSHub-Radar](https://github.com/DIYgod/RSSHub-Radar)

## Reference

- [initdb](https://www.postgresql.org/docs/current/app-initdb.html)
- [Server Dialog](https://www.pgadmin.org/docs/pgadmin4/development/server_dialog.html)
- [Configuration Parameters](https://miniflux.app/docs/configuration.html)