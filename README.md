# Laboratorio Docker: Apache + SSH + Wireshark

Este repositorio contiene un laboratorio educativo para demostrar cómo capturar y analizar tráfico HTTP y SSH utilizando **Docker**, **tcpdump** y **Wireshark**.

---

## 🚀 Objetivo

- Montar un entorno con **Apache** (con autenticación básica) y **SSH**.  
- Generar tráfico desde navegador, `curl` y conexión SSH.  
- Capturarlo con `tcpdump` y analizarlo en **Wireshark**.  
- Comparar credenciales en claro (HTTP sin TLS) vs tráfico cifrado (SSH).  

---

## 📦 Requisitos

- [Docker](https://docs.docker.com/get-docker/)  
- [Docker Compose](https://docs.docker.com/compose/install/)  
- [Wireshark](https://www.wireshark.org/download.html)
- Estructura de archivos:

```
├── apache
│   ├── .htpasswd
│   ├── htdocs
│   │   └── index.html
│   └── httpd.conf   
├── docker-compose.yml
```

---

## ⚙️ Configuración del contenedor Apache en caso de tener que configurar AUTH

El contenedor Apache utiliza autenticación básica, por lo que es necesario generar un archivo `.htpasswd` antes de iniciar:

1. Instala utilidades de Apache dentro del contenedor:  

   `apt update && apt install -y apache2-utils`
   
2. Genera el usuario y contraseña (ejemplo admin:admin123):
   
   `htpasswd -c ./apache/.htpasswd admin`

3. Verifica el archivo:
   
   `cat ./apache/.htpasswd`

 ## ▶️ Despliegue

 1. Clonar este repositorio:

   `https://github.com/Xinjis/Laboratorio-Docker-Apache-SSH-Wireshark.git`
   `cd Laboratorio-Apache-SSH-Wireshark.git`

 2. Levantar los contenedores:

   `docker compose up -d`

 3. Acceso a servicios:
 - Apache: http://localhost
 - SSH: ssh admin@localhost -p 2222

## 📡 Captura de tráfico

Dentro del contenedor:

```
docker exec -it apache_server bash
tcpdump -i eth0 -w /tmp/captura.pcap
```
Copiar el archivo al host:

`docker cp apache_server:/tmp/captura.pcap .`

Este se debe abrir con Wireshark para el análisis.

## ⚠️ Nota

- Este laboratorio es educativo.
- No uses credenciales reales.
- En producción siempre utiliza HTTPS/TLS.

