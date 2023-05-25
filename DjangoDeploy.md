## Django Deploy (Nginx)
### Author: Marcelo Caçapava Lopes Silva.
### SO: CENTOS, ROCKYLINUX E REDHAT.

<hr>
* Primeiro passo: Vamos instalar nosso servidor de Web:

```bash
#˜ sudo dnf update nginx
```

* Segundo Passo: Considerando que voce possua seu projeto django, vamos instalar em sua virtual env o Gunicorn e realizar o collectstatic nas imagens:

```bash
#˜ django-admin.py startproject myproject .
#˜ source venv/bin/activate
#˜ pip install gunicorn
#˜ ./manage.py collectstatic

```

* Terceiro Passo: Vamos configurar o Gunicorn como serviço:
```bash
#˜ sudo nano /etc/systemd/system/gunicorn.service
```

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=marcelomcls
Group=nginx
WorkingDirectory=/home/marcelomcls/myproject
ExecStart=/home/marcelomcls/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/marcelomcls/myproject/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```

** Vamos habilitar e subir o serviço:

```bash
#˜ sudo systemctl start gunicorn
#˜ sudo systemctl enable gunicorn
```

* Quarto Passo: Vamos configurar o NGINX:

```bash
#˜ sudo nano /etc/nginx/nginx.conf
```

```nginx
server {
    listen 80;
    server_name seu_dominio_ou_ip;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/marcelomcls/myproject;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/home/marcelomcls/myproject/myproject.sock;
    }
}
```

* Quinto Passo: Vamos ajustar os grupos de usuarios, lembrando que meu projeto esta na conta de usuario com nome de usuário marcelomcls:

```bash
#˜ sudo usermod -a -G user marcelomcls nginx
#˜ chmod 710 /home/marcelomcls

#˜ sudo nginx -t
#˜ sudo systemctl start nginx
#˜ sudo systemctl enable nginx
```
