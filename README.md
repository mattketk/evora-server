# Evora

Andor wrapper for Evora and Flask server.

## Installation

`evora-server` requires the proprietary Andor libraries to be installed in `/usr/local/lib`. The library can be used for debugging without the Andor libraries, but they are necessary to run the actual camera.

To install `evora-server`, clone the repository and run

```console
pip install .
```

or to install in editable mode

```console
pip install -e .
```

## Running the server

To run the server, from a terminal at the root of the project, execute

```console
python app.py
```

which is equivalent to

```console
flask --debug run --port 3000
```

### Running in debug mode

To run the server in debug mode, with the dummy module mocking the camera, edit `debug.py` and set `DEBUGGING = True`.

## Deploying for production

The recommended way to deploy `evora-server` is behind a [gunicorn](https://gunicorn.org) web server. To run the Flask webapp from `gunicorn`, execute

```console
gunicorn -w 4 'app:app'
```

which will spin a web server with four workers. This command should be wrapped in a daemon or systemd service to be executed in the background.

### Configuring nginx

In addition to `gunicorn`, a reverse proxy is needed to run the Evora client and server in the same web server. In Ubuntu, install `nginx` (alternatively you can use `Apache`) with

```console
sudo apt update
sudo apt install nginx
```

and adjust the firewall to open the desired ports. Then start `nginx` with

```console
sudo systemd enable --now nginx
```

We'll now add a new site for Evora. Create a new file `/etc/nginx/sites-available/evora.conf` with

```console
sudo vim /etc/nginx/sites-available/evora.conf
```

and include the configuration

```nginx
server {
    listen 9900;
    server_name localhost;
    access_log  /usr/local/var/log/nginx/evora.log;

    location /api/ {
        proxy_pass http://127.0.0.1:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```

This configuration creates a server running on port `9900` and adds a reverse proxy to where `gunicorn` is running the Flask webapp. After this, restart `nginx` with

```console
sudo systemctl restart nginx
```

and test that it works by navigating to [http://localhost:9900/api/getTemperature](http://localhost:9900/api/getTemperature).
