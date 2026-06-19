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
[ping -c 3 192.168.179.133]
<img width="1676" height="841" alt="image" src="https://github.com/user-attachments/assets/a7d4b5c3-01ab-4417-978e-20dc9a6ece2b" />
