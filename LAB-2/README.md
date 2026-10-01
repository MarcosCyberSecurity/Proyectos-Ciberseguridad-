# 🌐 Lab 2: servidor NGINX - Versión español


# 🌐 Laboratorio 2: Despliegue de Servidor Web Nginx y Análisis de Logs de Auditoría (Baseline)

## 🎯 Objetivo

El objetivo de esta práctica es desplegar localmente un servidor web Nginx para comprender la estructura de los logs de acceso (`access.log`) y errores (`error.log`). El análisis de logs es fundamental en ciberseguridad para establecer una línea base (*baseline*) de tráfico legítimo y aprender a identificar anomalías, escaneos de vulnerabilidades o intrusiones.

## 🧰 Herramientas y Comandos Utilizados

* **`Nginx`**: Servidor web ligero utilizado para simular el entorno de producción.
* **`tail -f /var/log/nginx/access.log`**: Para monitorear y analizar las peticiones entrantes HTTP en tiempo real.
* **`ls -l /var/log/nginx/`**: Para auditar los permisos de almacenamiento y la ubicación de las carpetas de registros de auditoría.

---

## 🚀 Proceso de Ejecución e Investigación

### 1. Inicialización del Servicio Web

Arrancamos el servidor Nginx de forma local en nuestra máquina utilizando privilegios elevados:

```bash
sudo /usr/sbin/nginx/
```

### 2. Auditoría del Directorio de Logs

Accedemos y verificamos las carpetas y archivos donde el servidor registra la actividad, asegurando que tanto `access.log` como `error.log` estén recopilando datos correctamente:
```bash
ls -l /var/log/nginx/
```

### 3. Simulación de Tráfico Web Legítimo

Nos conectamos al servidor web de forma local utilizando la IP de Loopback `127.0.0.1` en el puerto estándar `80`. La interfaz fotorrealista confirma que el servidor responde de manera correcta:

![Captura del Laboratorio 2](img/Captura%20de%20pantalla%202026-10-01%20180718.png)


### 4. Análisis e Interpretación de los Logs de Acceso en Tiempo Real
Para validar qué ocurre por detrás cada vez que un usuario interactúa con la web, ejecutamos el comando de seguimiento interactivo:

```bash
sudo tail -f /var/log/nginx/access.log
```

**Análisis técnico de las líneas capturadas en el log:**

* **Códigos de Estado HTTP 200 / 304:** Las peticiones del cliente con estado `GET / HTTP/1.1` demuestran que el servidor devolvió la página de inicio con éxito (`200 OK`) o sirvió el recurso desde la caché del navegador sin cambios (`304 Not Modified`).
  
* **Códigos de Estado HTTP 404:** Se registran peticiones fallidas como `GET /OO HTTP/1.1 404`. En un entorno real de Blue Team, un pico inusual de códigos 404 consecutivos nos alertaría sobre un posible escaneo de directorios automatizado (ej. Dirbuster, Gobuster).
  
* **Identificación del Cliente (User-Agent):** El log almacena con precisión el origen de la petición: `Mozilla/5.0 (X11; Ubuntu; Linux x86_64...)`, dato crucial en el análisis forense para identificar la huella digital (*fingerprinting*) de un cliente o atacante.

---

## 🛡️ Conclusiones para Ciberseguridad

El monitoreo de logs en servidores web es el pilar para alimentar sistemas SIEM (como Splunk o Wazuh). Comprender cómo se registra una petición legítima en Nginx nos capacita para crear reglas de detección capaces de identificar comportamientos maliciosos, tales como ataques de Inyección SQL (SQLi), Cross-Site Scripting (XSS) o ataques de fuerza bruta.
