# Configuración de Proxy Inverso con Nginx

Este es un ejemplo básico de cómo configurar un proxy inverso utilizando Nginx para dirigir el tráfico desde un dominio específico a un servidor backend.

## Requisitos Previos

Asegúrate de tener instalado Nginx en tu servidor antes de comenzar.

```bash
sudo apt update
sudo apt install nginx
```




## Configuración

**1. Editar el archivo de configuración de Nginx:**

Abre el archivo de configuración de Nginx. Por lo general, se encuentra en /etc/nginx/nginx.conf o en /etc/nginx/sites-available/default.

**2. Agrega la configuración del proxy inverso:**

Agrega la siguiente configuración dentro del bloque server en tu archivo de configuración:

![image](https://github.com/mohaelmes/nginx/assets/158450254/37198e11-81ea-4e24-85d3-53ed82734019)

- **listen**: El puerto en el que Nginx estará a la escucha de las solicitudes entrantes. En este contexto, se utiliza el puerto 80 para el tráfico HTTP estándar. Puedes cambiar este puerto según tus necesidades o requerimientos de configuración.

- **server_name**: El nombre del dominio que este servidor Nginx estará manejando. Esto le indica a Nginx qué solicitudes de dominio debe manejar. Por ejemplo, si deseas dirigir el tráfico desde el dominio "example.com", establecerías `server_name example.com;`.

- **location**: Define cómo Nginx manejará las solicitudes que coincidan con la ubicación especificada en la URL. En este caso, se utiliza para especificar que todas las solicitudes bajo la raíz (`/`) serán redirigidas al servidor backend definido en `proxy_pass`.

- **proxy_pass**: La URL del servidor backend al que se redirigirán las solicitudes. En la configuración del proxy inverso, esta es la dirección del servidor al que deseas enviar las solicitudes recibidas por Nginx para su procesamiento adicional.

- **proxy_set_header**: Esta directiva de Nginx configura las cabeceras que se enviarán al servidor backend. Esto es importante para mantener la información correcta de la solicitud original, como la dirección IP del cliente (`X-Real-IP`), la dirección IP del proxy (`X-Forwarded-For`), entre otras.

Lo modificamos a nuestra manera: 
```bash
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  mohamedelmes.com;

        location / {
            root "C:/Users/amine/OneDrive/Escritorio/nginx"; 
            index index.html;
            try_files $uri $uri/ /index.html;
        }

        location /proxy/ {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

**3. Verifica la sintaxis y reinicia Nginx:**

Verifica que la sintaxis de la configuración sea correcta:
```bash
sudo nginx -t
```

Si todo está bien, reinicia Nginx para aplicar los cambios:
```bash
sudo systemctl restart nginx
```

Con esto, Nginx debería estar actuando como un proxy inverso y dirigiendo todas las solicitudes entrantes desde example.com al servidor backend que se ejecuta en http://localhost:8000.
