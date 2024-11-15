# Configuracion para NGINX | Exponer  los servicios en dominio local

¡Me alegra que ahora puedas acceder! Aquí tienes la configuración y pasos actualizados y simplificados para que todo funcione correctamente.

---

## **Configuración Completa**

### **Archivo `docker-compose.yml`**
Asegúrate de que tu archivo `docker-compose.yml` incluye un servicio para NGINX y las redes están correctamente configuradas:

```yaml
version: "3.9"

networks:
  gitea:
    driver: bridge

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - gitea
    depends_on:
      - server
      - jenkins_service

  server:
    image: gitea/gitea:1.22.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=${GITEA_DB_PASSWD}
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
    depends_on:
      - db

  db:
    image: postgres:14
    container_name: tesis-db-1
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=gitea
    networks:
      - gitea
    volumes:
      - ./postgres:/var/lib/postgresql/data

  jenkins_service:
    image: jenkins/jenkins:lts
    container_name: jenkins_service
    user: root
    ports:
      - "50000:50000"
    networks:
      - gitea
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
```

---

### **Archivo `nginx.conf`**
Incluye una configuración válida para NGINX:

```nginx
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Configuración para Gitea
    server {
        listen 80;
        server_name gitea.myservice.net;

        location / {
            proxy_pass http://server:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    # Configuración para Jenkins
    server {
        listen 80;
        server_name jenkins.myservice.net;

        location / {
            proxy_pass http://jenkins_service:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

Guarda este archivo en el mismo directorio que tu archivo `docker-compose.yml`.

---

### **Pasos Actualizados**

1. **Verificar Dependencias**
   - Asegúrate de que tienes Docker y Docker Compose instalados correctamente.
   - Si estás en Windows con WSL, revisa que Docker Desktop esté funcionando y que las redes estén configuradas.

2. **Actualizar `/etc/hosts`**
   Agrega las siguientes líneas al archivo `/etc/hosts` (o `C:\Windows\System32\drivers\etc\hosts` en Windows):

   ```plaintext
   127.0.0.1 gitea.myservice.net
   127.0.0.1 jenkins.myservice.net
   ```

3. **Levantar los Contenedores**
   En el directorio donde se encuentra tu archivo `docker-compose.yml`, ejecuta:

   ```bash
   docker-compose up -d
   ```

4. **Verificar Logs**
   Revisa los logs de NGINX para asegurarte de que todo está funcionando:

   ```bash
   docker-compose logs nginx
   ```

   Si hay algún error, corrígelo antes de proceder.

5. **Probar Servicios**
   Abre tu navegador y accede a las URLs configuradas:
   - [http://gitea.myservice.net](http://gitea.myservice.net)
   - [http://jenkins.myservice.net](http://jenkins.myservice.net)

6. **Verificar Configuración de NGINX**
   Si necesitas asegurarte de que la configuración de NGINX es válida, entra al contenedor y ejecuta:

   ```bash
   docker exec -it nginx nginx -t
   ```

---

### **Notas Finales**
- Si necesitas agregar más servicios en el futuro, simplemente añade nuevas secciones `server` en el archivo `nginx.conf` y apunta al contenedor correspondiente.
- Asegúrate de reiniciar NGINX después de modificar la configuración:

   ```bash
   docker-compose restart nginx
   ```

Con estos pasos, tus servicios estarán accesibles mediante los dominios locales configurados. 😊