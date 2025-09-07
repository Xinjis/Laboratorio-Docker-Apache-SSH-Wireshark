# 📘 DOCUMENTACION_TECNICA.md (manual detallado)

# Manual Técnico: Laboratorio Docker + Wireshark

Este documento detalla paso a paso la creación de un entorno de laboratorio con **Docker** para la captura y análisis de tráfico en protocolos **HTTP** y **SSH**.


## 1. Introducción

Este laboratorio busca mostrar cómo viajan las credenciales en protocolos sin cifrado (HTTP con autenticación básica) frente a protocolos seguros (SSH).  

Se utilizarán:
- **Docker** para desplegar servicios.  
- **tcpdump** para capturar tráfico.  
- **Wireshark** para analizarlo.  


## 2. Arquitectura del laboratorio

- **Contenedor Apache**: puerto `80`, autenticación básica (usuario definido en `.htpasswd`).  
- **Contenedor SSH**: puerto `2222`, acceso con usuario/clave definidos en `docker-compose.yml`. 

```
Host ──> [Docker]
├─ Apache (80)
└─ SSH (2222
```

## 3. Configuración del laboratorio

### 3.1 Docker Compose

Archivo `docker-compose.yml`:

```
yaml
services:
  web:
    image: httpd:2.4
    container_name: apache_server
    ports:
      - "80:80"
    volumes:
      - ./apache/httpd.conf:/usr/local/apache2/conf/httpd.conf:ro
      - ./apache/.htpasswd:/usr/local/apache2/.htpasswd:ro
      - ./apache/htdocs:/usr/local/apache2/htdocs
    networks:
      - labnet

  ssh:
    image: linuxserver/openssh-server
    container_name: ssh_server
    environment:
      - PUID=1000
      - PGID=1000
      - PASSWORD_ACCESS=true
      - USER_NAME=admin
      - USER_PASSWORD=admin123
    ports:
      - "2222:2222"
    volumes:
      - ./ssh/config:/config
    networks:
      - labnet

networks:
  labnet:
    driver: bridge
```

### 3.2 Configuración de Apache en caso de ser encesario
- Generar archivo .htpasswd en caso de ser necesario:
  
```
apt update && apt install -y apache2-utils
htpasswd -c ./apache/.htpasswd admin
```

- Configurar httpd.conf con autenticación básica:
  
```
<Directory "/usr/local/apache2/htdocs">
    AuthType Basic
    AuthName "Contenido Restringido"
    AuthUserFile "/usr/local/apache2/.htpasswd"
    Require valid-user
</Directory>
```

## 4. Generación de tráfico
   
- HTTP:
  
`curl -v http://localhost --user admin:admin123`

- SSH:

`ssh admin@localhost -p 2222`

También se puede acceder vía navegador a http://localhost

## 5. Captura de paquetes

- Dentro del contenedor Apache:

```
docker exec -it apache_server bash
tcpdump -i eth0 -w /tmp/captura.pcap
```

- Detener con Ctrl + C.
- Copiar archivo al host:
  
`docker cp apache_server:/tmp/captura.pcap .`

## 6. Análsis en Wireshark
- HTTP sin TLS
  
`http.response.code == 401`
<img width="2724" height="1604" alt="Pasted Graphic 4" src="https://github.com/user-attachments/assets/51c17baf-937f-4256-a75f-4b0c81b1fe79" />

Ver cabecera WWW-Authenticate.

- Filtrar credenciales:

`http.authorization`
<img width="2702" height="1570" alt="Pasted Graphic 5" src="https://github.com/user-attachments/assets/6559593c-1fb9-4214-a791-5df35f4d445a" />

- Decodificar Base64

`echo -n 'YWRtaW46YWRtaW4xMjM=' | base64 -d`
<img width="996" height="82" alt="Pasted Graphic 6" src="https://github.com/user-attachments/assets/fc26d9fa-f1e5-478d-a4d6-27f6c5b8fb7d" />

El resultado esperado es admin:admin123

## 7. Conclusiones
- HTTP sin TLS expone credenciales → nunca debe usarse en producción.
- SSH protege toda la sesión → la confidencialidad está garantizada.
- La lección clave: TLS/HTTPS no es opcional.

## 8. Consideraciones
	•	Laboratorio con fines educativos.
	•	No usar credenciales reales.
	•	Ideal para prácticas en ciberseguridad y análisis de tráfico.













