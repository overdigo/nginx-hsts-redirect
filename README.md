# NginX HSTS redirect
Veja como configura o redirecionamento do Nginx para que o site seja aprovado para entrar para a lista HSTS
O segredo do redirect no nginx para ser aceito como HSTS é redirecionar primeiro para HTTPS e depois redirecionar novamente para adicionar ou remover o www

## Redirect para sites WWW
Esse primeiro exemplo é para sites WWW
#### Redirect http com www para https
Nesse caso vai existir apenas 1 redrect direto para o site https://www.exemplo.com.br.

Entra http://www.exemplo.com.br e sai https://www.exemplo.com.br
```sh
server {
    listen 80;
    listen [::]:80;
    server_name www.exemplo.com.br;
    return 301 https://www.exemplo.com.br$request_uri;
}
```
#### Redirect http sem www para https
Nesse caso o url vai ser redirecionado para https://exemplo.com.br

Entra http://exemplo.com.br e sai https://exemplo.com.br
```sh
server {
    listen 80;
    listen [::]:80;
    server_name exemplo.com.br;
    return 301 https://exemplo.com.br$request_uri;
}
```
#### Redirect https sem www para https com www
Nesse redirect o url https sem www vai receber novo redirect para https com www - redirecionando para https://www.exemplo.com.br

Para funcionar corretamente é preciso conter os certificados e o header HSTS

Entra https://exemplo.com.br e sai https://www.exemplo.com.br
```sh
server {
    server_name exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl on;
    return 301 $scheme://www.exemplo.com.br$request_uri;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
}
```
#### Server final, onde vai ser recebido todos outros redirects
Coloque todo código como root, index, log, certificados e header HSTS
Entra https://www.exemplo.com.br e não faz redirect
```sh
server {
    server_name www.exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
    ...
    CODE
    ...
}
```
#### Resultado final para Redirect NginX com HSTS com WWW
```sh
server {
    listen 80;
    listen [::]:80;
    server_name www.exemplo.com.br;
    return 301 https://www.exemplo.com.br$request_uri;
}
server {
    listen 80;
    listen [::]:80;
    server_name exemplo.com.br;
    return 301 https://exemplo.com.br$request_uri;
}
server {
    server_name exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl on;
    return 301 $scheme://www.exemplo.com.br$request_uri;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
}
server {
    server_name www.exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
    ...
    CODE
    ...
}
```
### TESTE
Faça teste em [https://httpstatus.io/](https://httpstatus.io/)

Veja o resultado do teste abaixo:
o link http vai para https depois para https com www
o link http com www vai para https com www
o link com htts sem www vai para https com www
e o link com https e www não faz nenhum redirect

![alt text](https://raw.githubusercontent.com/overdigo/nginx-hsts-redirect/master/nginx-hsts-redirect.jpg "nginx HSTS redirect www")

### Enviando para HSTS

Envie seu site como HSTS em: [https://hstspreload.org/](https://hstspreload.org/)

![alt text](https://raw.githubusercontent.com/overdigo/nginx-hsts-redirect/master/dominio-hsts-teste.jpg "nginx HSTS redirect www")

---

## Redirect para sites não WWW
Esse exemplo é para sites não WWW

```sh
server {
    listen 80;
    listen [::]:80;
    server_name www.exemplo.com.br;
    return 301 https://www.exemplo.com.br$request_uri;
}
server {
    listen 80;
    listen [::]:80;
    server_name exemplo.com.br;
    return 301 https://exemplo.com.br$request_uri;
}
server {
    server_name www.exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl on;
    return 301 $scheme://exemplo.com.br$request_uri;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
}
server {
    server_name exemplo.com.br;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/exemplo.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/exemplo.com.br/privkey.pem;
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
}
```

Publicado e mantido por  [supertutorial](https://www.supertutorial.com.br/)




