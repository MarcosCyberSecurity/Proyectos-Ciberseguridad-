# 🔍 Laboratorio 1: Análisis de Procesos en Tiempo Real y Auditoría de Sockets en Linux

## 🎯 Objetivo
El objetivo de este laboratorio práctico es auditar las conexiones de red activas en un sistema Linux para identificar anomalías y simular un análisis forense de procesos (*Threat Hunting*). Se documenta el procedimiento técnico para rastrear un Identificador de Proceso (PID) sospechoso, localizar su ruta física en el disco y validar su integridad estructural para descartar vectores de persistencia o camuflaje de malware.

## 🧰 Herramientas y Comandos Utilizados
* **`ss -abnop`**: Para el análisis avanzado y auditoría de sockets (red, interfaces y procesos vinculados).
* **`ls -l /proc/[PID]/exe`**: Para localizar la ruta física real del binario en ejecución inspeccionando el enlace simbólico del kernel.
* **`file`**: Para analizar las cabeceras de los archivos y verificar la firma estructural interna del ejecutable.
* **Fuentes OSINT / CTI**: Metodologías de Cyber Threat Intelligence para enriquecimiento y análisis de reputación de infraestructura.

---

## 🚀 Proceso de Ejecución e Investigación

### 1. Auditoría de Conexiones de Red y Sockets Activos
Se analizó un proceso manteniendo conexiones de red activas en el sistema. Utilizando herramientas avanzadas de análisis de sockets, se rastreó el identificador del proceso (PID) hasta localizar la ruta física del archivo binario ejecutable en el disco y validar su tipo de estructura interna con ESTOS COMANDOS:

```shell
ss -abnop
sudo ls -l /proc/1509/exe
sudo file /usr/lib/mate-panel/wnck-applet
```

![[Pasted image 20260812164415.png]]

Al evaluar la salida, el análisis revela dos tipos de comportamiento críticos para la auditoría:
* **Sockets Unix Locales (`u_str`, `u_dgr`):** Se identifican procesos legítimos del sistema operando internamente, destacando `wnck-applet` bajo el **PID 1509**, junto a servicios esenciales como `systemd` (PID 1132) y `dbus-daemon` (PID 1387).
  
* **Conexiones e Interfaces de Red (`udp`, `tcp`):** Se detectan conexiones salientes activas hacia servidores externos (IP `9.9.9.9` mediante los puertos efímeros `49605` y `60901`). Asimismo, se observa un servicio local esperando tráfico de manera inusual en el puerto **TCP 2222** (`*:2222`), lo cual enciende una alerta de seguridad por ser un puerto alternativo comúnmente explotado para *backdoors* o sesiones SSH ocultas.

### 2. Enriquecimiento con Threat Intelligence (Investigación de IoCs)
Como parte del flujo de trabajo de Inteligencia de Amenazas, extraemos los Indicadores de Compromiso (IoCs) detectados en la auditoría de sockets para su posterior análisis y correlación:

* **IP Destino:** `9.9.9.9`
* **Puerto Destino:** `2222`

**Procedimiento de CTI aplicado:** 

1. **Verificación de Reputación:** En un escenario de operaciones reales, contrastamos la dirección IP y el puerto sospechoso en plataformas de Threat Intel (como *VirusTotal*) para verificar si están vinculados a servidores de Comando y Control (C2), nodos de botnets o si pertenecen a una infraestructura legítima (en este caso particular, corresponde al servicio DNS público de Quad9).
   
2. **Contextualización de Amenazas:** Cruzamos el puerto inusual `2222` con bases de datos de vulnerabilidades conocidas para correlacionar qué familias de malware o troyanos de persistencia suelen utilizar dicho puerto para evadir las defensas perimetrales convencionales.

### 3. Rastreo de la Ruta Física del Proceso (Lógica Forense)
Para verificar el origen del proceso vinculado al puerto sospechoso, investigamos su directorio virtual en el sistema de archivos del kernel:
```bash
sudo ls -l /proc/1509/exe
```
> **💡 Nota de Aprendizaje:** Es común que analistas junior se confundan al ver la palabra `exe` en la ruta `/proc/[PID]/exe`, asociándola erróneamente con ejecutables de Windows. En la arquitectura Linux, `exe` es simplemente el nombre del enlace simbólico genérico que utiliza el kernel para apuntar al binario nativo real en el disco.

### 4. Validación Estructural e Integridad del Binario
Finalmente, auditamos la naturaleza interna del archivo ejecutable para determinar si se trata de un binario legítimo o un script malicioso camuflado:
```bash
sudo file /usr/lib/mate-panel/wnck-applet
```

**Criterios de Evaluación del comando `file` para Incident Response:**
* **Binario Nativo Legítimo:** Si el comando devuelve un formato **ELF** (`ELF 64-bit LSB shared object` o `executable`), confirmamos que es un binario nativo compilado legítimamente para Linux.
* **Scripts Ocultos:** Si la salida muestra `Python script` o `ASCII text executable` en una ruta crítica donde el sistema espera un binario, se clasifica como una anomalía de suplantación.
* **Patrón de Malware / Troyano:** Un indicador de compromiso (IoC) definitivo ocurre si el comando `file` confirma la estructura de un binario ejecutable ELF, pero su ruta física reside en directorios temporales o volátiles como `/tmp/...` o `/dev/shm/`. Este comportamiento expone un patrón clásico de malware intentando evadir mecanismos de persistencia tradicionales.

---

## 🛡️ Defensa Correcta y Mitigación (Playbook del SOC)
En caso de detectar puertos abiertos sospechosos (como el `2222` analizado), el flujo de respuesta ante incidentes dicta:
1. Usar el comando `file` de inmediato sobre el enlace de su PID para examinar la cabecera.
2. Validar que el ejecutable resida exclusivamente en rutas seguras de lectura del sistema (como `/usr/lib/` o `/usr/sbin/`).
3. Si el binario corre desde directorios temporales, se procede al aislamiento de la máquina de la red, el volcado de memoria del proceso para análisis en sandbox, y la terminación inmediata (*kill*) del proceso comprometido mediante su PID.










