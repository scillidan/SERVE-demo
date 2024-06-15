## Localhost

```sh
python310 -m venv venv
venv\Scripts\activate.bat
pip install .
python changedetection.py
```

Open `Localhost:5000`

## PM2

```sh
pm2 start changedetection.py --name changedetection --interpreter "...\changedetection.io\venv\Scripts\python.exe" --cwd "...\changedetection.io"
```