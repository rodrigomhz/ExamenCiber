# MEMORIA COMPLETA DE AUDITORÍA DE SEGURIDAD
## Contexto del Examen
En este ejercicio de auditoría de seguridad, se nos proporcionó una máquina virtual objetivo con sistema Ubuntu que debíamos analizar, identificando vulnerabilidades y intentando su explotación. La máquina atacante utilizada fue Kali Linux, con ambas máquinas conectadas en la red NAT 10.0.2.0/24 para facilitar la comunicación y las pruebas.
---

![Maquina](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Ubuntu.png)

![Red2](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Red2.png)

## 1 Configuración Inicial
Lo primero que realizamos fue introducir la máquina víctima en nuestra red NAT 10.0.2.0

![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/Red1.png)

A partir de ahí, verificamos la IP de la máquina Kali y procedimos a detectar la IP de la víctima.


![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/IPA.png)

Justificación:

  - Mantener ambas máquinas (atacante y víctima) en la misma red NAT permite la comunicación directa

  - Facilita el escaneo y la explotación al estar en el mismo segmento de red


## 2 Descubrimiento de la Máquina Víctima

### 2.1 Primer intento: arp-scan

Primero robamos un arp-scan para descubrir la máquina, pero no dio información fiable para identificarla como víctima

![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ARP.png)

### 2.2 Segundo intento: Nmap escaneando toda la red

Comando ejecutado:

![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/NMAP1.png)

````
nmap -sS -sV -O 10.0.2.5/24 -T5
````
Este escaneo completo revelA la IP 10.0.2.20, que coincidE con una máquina Linux con múltiples servicios levantados.

## 3 Enumeración de Servicios

Parece ser la Máquina en **10.0.2.20** la asociada al examen:

  Tiene servicios activos como **FTP, SSH, Apache, MySQL, y Jetty**. 
  Esta parece ser una máquina basada en Linux, y los servicios disponibles pueden ser explotados dependiendo de las vulnerabilidades conocidas.

## 4 Escaneo de vulnerabilidades con Nmap

Se intentó el clásico comando:
````
nmap -sS -sV -O --script vuln 10.0.2.20 -T5
````

Problemas detectados:
  - El escaneo tardó demasiado
  - No devolvió resultados de vulnerabilidades útiles
  - No detectó la vulnerabilidad de ProFTPd ni nada crítico

Conclusión:
Nmap no fue útil para la fase de vulnerabilidades debido al tiempo y la falta de resultados críticos.

## 5 Análisis en paralelo con Nessus

Dado que Nmap no daba resultados, arrancamos Nessus para realizar el analisis

![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/nessus1.png)

![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/nessus2.png)


Se creó un escaneo dirigido a la IP 10.0.2.20.

## 5 Pruebas Manuales de Acceso

Debido a que las herramientas de Nessus y de nmap estaban dando problemas, fuimos probando diferentes cosas de forma manual, bajo el conocimiento de los puertos abiertos y sus servicios correspondientes.

 ### 5.1 FTP
  
  Se intentó:
  
   - Acceso anónimo

     ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/FTP1.png)
  
   - root/root

     ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/FTP2.png)
  
  Todos fallaron
  
### 5.2 HTTP

  Se accedió al servicio web para reconocimiento inicial

  ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/http1.png)

  Usuario Identificado:

  ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/http2.png)
  
  Se encontró el usuario "chewbacca" durante la enumeración

  Intentamos usarlo en:

   - SSH
     
  ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssh.png)
    
   - Explotación de Metasploit
       Ataque de Fuerza Bruta SSH con Metasploit
       ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/sshlogin1.png)
  
     ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/sshlogin2.png)

     Configuración del módulo:

      - Se utilizó el usuario "chewbacca" identificado previamente
      
      - Se empleó lista de contraseñas por defecto (rokyou)
      
      Resultado:
     
      ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssh3.png)
      
      - Fallo completo en el ataque de fuerza bruta
      
      - Errores de conexión: "connection closed by remote host"
      
      - Posible protección contra fuerza bruta en el servicio SSH
      
      - El mensaje "Connection reset by peer" sugiere que el servidor cortó la conexión

Ninguno funcionó.

## 6 Vulnerabilidades Nessus

  ### Vulnerabilidad SSL Identificada

  ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/ssl.png)

  ### Drupal
  ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/drupal.png)

  Módulo: exploit/unix/webapp/drupal_drupageddon2
  Payload: php/meterpreter/reverse_tcp
  
  Configuración:
  
  - LHOST: 10.0.2.5
  
  - LPORT: 4444

  Resultado:

  - El objetivo apareció como vulnerable
  
  - Exploit completado pero no se creó sesión
  
  - Posibles causas: Protecciones, versión incorrecta, o filtros de salida

    ### SPIP

    Módulo: exploit/multi/http/spip_connect_exec
    
    Configuración:
    
    RHOSTS: 10.0.2.20
    
    Resultado:
    
    No se pudo determinar la versión de SPIP
    
    Exploit abortado por imposibilidad de verificar explotabilidad
    
    Se sugirió usar set ForceExploit true para override

    ### Samba

    ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/samba.png)
    ![Red1](https://github.com/rodrigomhz/ExamenCiber/blob/main/Imagenes/samba2.png)

Si el Exploit Funciona:
Conexión reversa establecida

Sesión de Meterpreter obtenida

Acceso al sistema Linux objetivo

Capacidad de ejecutar comandos con privilegios del servicio Samba

Problemas Potenciales (Basado en Patrones Anteriores):
Timeout en la conexión

Share de solo lectura imposibilitando la escritura

Servicio Samba parcheado contra esta vulnerabilidad

Filtros de red bloqueando la conexión reversa








