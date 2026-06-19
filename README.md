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
|Atacante|Kali Linux|192.168.179.129|
|Objetivo|Metasploitable 2|192.168.179.133|
<img width="1701" height="906" alt="image" src="https://github.com/user-attachments/assets/c71ae853-42f1-4a34-86ce-01874c565e62" />
<img width="805" height="433" alt="image" src="https://github.com/user-attachments/assets/fe24f633-a70d-4b37-a9e1-4d2efeb59bf1" />

Verificación de conectividad:
```bash
ping -c 3 192.168.179.133
```
<img width="1676" height="841" alt="image" src="https://github.com/user-attachments/assets/a7d4b5c3-01ab-4417-978e-20dc9a6ece2b" />

# Fase 1: Reconocimiento de Red
El reconocimiento activo consiste en enviar paquetes a la red para descubrir qué hosts están activos, sin aún analizar sus servicios. Se usaron dos herramientas complementarias:

netdiscover trabaja a nivel de capa 2 (ARP), preguntando "¿quién tiene esta IP?" a toda la red. Es muy sigiloso porque usa tráfico que cualquier dispositivo genera normalmente.

nmap -sn realiza un ping sweep a nivel de capa 3 (ICMP), confirmando qué hosts responden sin escanear puertos.

Se ejecutará:
```bash
netdiscover -r 192.168.124.0/24
nmap -sn 192.168.124.0/24 | grep "Nmap scan report for"
```
<img width="1031" height="865" alt="image" src="https://github.com/user-attachments/assets/f204def8-33a8-4f11-96b8-f790564a6596" />

Conclusión: El host objetivo 192.168.124.133 está activo y accesible desde nuestra máquina atacante.

# Fase 2: Escaneo con Nmap / Zenmap
## 2.1 Detección de sistema operativo
```bash
nmap -O 192.168.124.133
```
<img width="1002" height="852" alt="image" src="https://github.com/user-attachments/assets/0bee8360-89e5-4e99-b196-7bbb0b73a867" />

Nmap identificó el objetivo como Linux 2.6.9 – 2.6.33, un kernel antiguo que confirma la naturaleza vulnerable de Metasploitable 2.

## 2.2 Detección de versiones -- top 1000 puertos
```bash
nmap -sV 192.168.124.133
```

<img width="987" height="867" alt="image" src="https://github.com/user-attachments/assets/76e50a5e-73e3-4257-9235-cb9d52111b56" />

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

# Fase 3: Identificación de Vulnerabilidades
# Fase 4: Explotación distcc CVE-2004-2687
# Fase 5: Reverse Shell
# Fase 6: Explotación vsFTPD 2.3.4 CVE-211
# Conclusiones y lecciones aprendidas
