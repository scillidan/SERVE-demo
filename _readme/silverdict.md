## Localhost

```sh
cd client
yarn install
yarn build
mv build ../server/
```

```sh
cd ../server
python310 -m venv venv
venv\Scripts\activate.bat
pip install -r requirements.txt
pip install lxml
python server.py
```

Open `localhost:2628`.

Setting → Sources → add `yourpath` then `Enter` → Wait for import.

In addition, some cache saved on `C:\Users\yourname\.silverdict` and `C:\Users\yourname\.cache\SilverDict`.

## PM2

```sh
pm2 start server.py --name silverdict --interpreter "...\SilverDict\server\venv\Scripts\python.exe" --cwd "...\SilverDict\server" 
```