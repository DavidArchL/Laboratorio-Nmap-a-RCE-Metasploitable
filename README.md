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
[!WARNING]⚠️ Todas las técnicas demostradas en este laboratorio se realizaron en un entorno completamente controlado y aislado, sobre una máquina virtual diseñada específicamente para este propósito. El uso de estas técnicas sobre sistemas sin autorización explícita es ilegal y va en contra de la ética profesional en ciberseguridad. Este documento tiene fines exclusivamente educativos.

