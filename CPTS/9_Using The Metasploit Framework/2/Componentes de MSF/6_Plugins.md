# Plugins en Metasploit Framework

## Introducción a los Plugins

Los plugins en Metasploit son componentes de software desarrollados por terceros que extienden la funcionalidad del framework. Estos módulos pueden provenir de dos fuentes principales:

1. **Productos comerciales** con versiones Community gratuitas (aunque con funcionalidad limitada)
2. **Proyectos individuales** desarrollados por investigadores de seguridad y miembros de la comunidad

La característica distintiva de los plugins es que han sido autorizados explícitamente por sus creadores para integrarse dentro del ecosistema de Metasploit, diferenciándose así de simple código externo que podría ejecutarse de forma independiente.

## Ventajas de Utilizar Plugins

### Integración Centralizada

Antes de la implementación de plugins, los pentesters debían:
- Alternar entre múltiples aplicaciones
- Exportar e importar resultados manualmente entre herramientas
- Configurar parámetros repetidamente en cada software
- Consolidar información de forma manual

Con los plugins, todo este flujo se simplifica drásticamente. La información se documenta automáticamente en la base de datos activa de `msfconsole`, permitiendo que hosts, servicios y vulnerabilidades estén disponibles de inmediato para consulta.

### Capacidades Técnicas

Los plugins operan directamente con la API de Metasploit Framework, lo que les permite:

- **Manipular el framework completo**: Acceso profundo a todas las funcionalidades internas
- **Automatizar tareas repetitivas**: Ejecutar secuencias complejas de comandos sin intervención manual
- **Agregar comandos nuevos**: Extender la línea de comandos de `msfconsole` con funcionalidad personalizada
- **Integración con bases de datos**: Sincronización automática de resultados y hallazgos

## Uso de Plugins

### Ubicación y Verificación de Plugins Instalados

Los plugins se almacenan en el directorio predeterminado del framework:

```bash
/usr/share/metasploit-framework/plugins
```

Para verificar qué plugins están disponibles en el sistema:

```bash
ls /usr/share/metasploit-framework/plugins
```

**Salida esperada:**
```
aggregator.rb      beholder.rb        event_tester.rb  komand.rb     msfd.rb    nexpose.rb   request.rb  session_notifier.rb  sounds.rb  token_adduser.rb  wmap.rb
alias.rb           db_credcollect.rb  ffautoregen.rb   lab.rb        msgrpc.rb  openvas.rb   rssfeed.rb  session_tagger.rb    sqlmap.rb  token_hunter.rb
auto_add_route.rb  db_tracker.rb      ips_filter.rb    libnotify.rb  nessus.rb  pcap_log.rb  sample.rb   socket_logger.rb     thread.rb  wiki.rb
```

Cada archivo `.rb` (Ruby) representa un plugin independiente.

### Cargando un Plugin

Para activar un plugin dentro de `msfconsole`, utilizamos el comando `load`:

```
msf6 > load nessus
```

**Respuesta del sistema:**
```
[*] Nessus Bridge for Metasploit
[*] Type nessus_help for a command listing
[*] Successfully loaded Plugin: Nessus
```

El mensaje de confirmación incluye:
- **Identificación del plugin**: Nombre y propósito
- **Comando de ayuda**: Cómo acceder a la documentación
- **Estado de carga**: Confirmación de activación exitosa

### Explorando Comandos del Plugin

Cada plugin extiende `msfconsole` con comandos específicos. Para Nessus:

```
msf6 > nessus_help
```

**Salida (parcial):**
```
Command                     Help Text
-------                     ---------
Generic Commands            
-----------------           -----------------
nessus_connect              Connect to a Nessus server
nessus_logout               Logout from the Nessus server
nessus_login                Login into the connected Nessus server with a different username

nessus_user_del             Delete a Nessus User
nessus_user_passwd          Change Nessus Users Password
                            
Policy Commands             
-----------------           -----------------
nessus_policy_list          List all polciies
nessus_policy_del           Delete a policy
```

Los comandos se organizan por categorías funcionales (Generic Commands, Policy Commands, etc.), facilitando la navegación y comprensión del plugin.

### Manejo de Errores

Si intentamos cargar un plugin inexistente o mal instalado:

```
msf6 > load Plugin_That_Does_Not_Exist
```

**Error resultante:**
```
[-] Failed to load plugin from /usr/share/metasploit-framework/plugins/Plugin_That_Does_Not_Exist.rb: cannot load such file -- /usr/share/metasploit-framework/plugins/Plugin_That_Does_Not_Exist.rb
```

Este mensaje indica:
- **Ruta de búsqueda**: Dónde Metasploit intentó localizar el plugin
- **Causa del fallo**: Archivo no encontrado o inaccesible

## Instalación de Plugins Personalizados

### Plugins Pre-instalados vs. Personalizados

Los plugins populares se incluyen automáticamente en las actualizaciones de distribuciones como Parrot OS. Sin embargo, para plugins de terceros no incluidos en los repositorios oficiales, debemos realizar una instalación manual.

### Proceso de Instalación Manual

**Ejemplo práctico**: Instalando los plugins de [DarkOperator](https://github.com/darkoperator/Metasploit-Plugins)

1. **Clonar el repositorio del plugin:**

```bash
git clone https://github.com/darkoperator/Metasploit-Plugins
```

2. **Verificar el contenido descargado:**

```bash
ls Metasploit-Plugins
```

**Contenido:**
```
aggregator.rb      ips_filter.rb  pcap_log.rb          sqlmap.rb
alias.rb           komand.rb      pentest.rb           thread.rb
auto_add_route.rb  lab.rb         request.rb           token_adduser.rb
beholder.rb        libnotify.rb   rssfeed.rb           token_hunter.rb
db_credcollect.rb  msfd.rb        sample.rb            twitt.rb
db_tracker.db      msgrpc.rb      session_notifier.rb  wiki.rb
event_tester.rb    nessus.rb      session_tagger.rb    wmap.rb
ffautoregen.rb     nexpose.rb     socket_logger.rb
growl.rb           openvas.rb     sounds.rb
```

3. **Copiar el plugin al directorio de Metasploit:**

```bash
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```

**Consideraciones importantes:**
- Se requieren privilegios de superusuario (`sudo`)
- Los permisos del archivo deben ser apropiados para que Metasploit pueda leerlo
- El archivo debe tener extensión `.rb`

4. **Iniciar `msfconsole` y cargar el plugin:**

```bash
msfconsole -q
```

```
msf6 > load pentest
```

**Salida del plugin pentest:**
```
       ___         _          _     ___ _           _
      | _ \___ _ _| |_ ___ __| |_  | _ \ |_  _ __ _(_)_ _
      |  _/ -_) ' \  _/ -_|_-<  _| |  _/ | || / _` | | ' \ 
      |_| \___|_||_\__\___/__/\__| |_| |_|\_,_\__, |_|_||_|
                                              |___/
      
Version 1.6
Pentest Plugin loaded.
by Carlos Perez (carlos_perez[at]darkoperator.com)
[*] Successfully loaded plugin: pentest
```

### Verificación de la Instalación

Una vez cargado, el menú de ayuda de `msfconsole` se extiende automáticamente:

```
msf6 > help
```

**Nuevas categorías de comandos agregadas por pentest:**

```
Tradecraft Commands
===================
    Command          Description
    -------          -----------
    check_footprint  Checks the possible footprint of a post module on a target system.

auto_exploit Commands
=====================
    Command           Description
    -------           -----------
    show_client_side  Show matched client side exploits from data imported from vuln scanners.
    vuln_exploit      Runs exploits based on data imported from vuln scanners.

Discovery Commands
==================
    Command                 Description
    -------                 -----------
    discover_db             Run discovery modules against current hosts in the database.
    network_discover        Performs a port-scan and enumeration of services found for non pivot networks.
    pivot_network_discover  Performs enumeration of networks available to a specified Meterpreter session.
    show_session_networks   Enumerate the networks one could pivot thru Meterpreter in the active sessions.

Project Commands
================
    Command       Description
    -------       -----------
    project       Command for managing projects.

Postauto Commands
=================
    Command             Description
    -------             -----------
    app_creds           Run application password collection modules against specified sessions.
    get_lhost           List local IP addresses that can be used for LHOST.
    multi_cmd           Run shell command against several sessions
    multi_meter_cmd     Run a Meterpreter Console Command against specified sessions.
    multi_meter_cmd_rc  Run resource file with Meterpreter Console Commands against specified sessions.
    multi_post          Run a post module against specified sessions.
    multi_post_rc       Run resource file with post modules and options against specified sessions.
    sys_creds           Run system password collection modules against specified sessions.
```

Cada categoría proporciona funcionalidad específica orientada a diferentes fases del pentesting.

## Plugins Populares

La comunidad de Metasploit ha desarrollado numerosos plugins especializados. Algunos de los más utilizados incluyen:

| Plugin | Estado | Descripción |
|--------|--------|-------------|
| [**nMap**](https://nmap.org/) | Pre-instalado | Integración con el escáner de puertos Nmap |
| [**NexPose**](https://sectools.org/tool/nexpose/) | Pre-instalado | Conexión con el escáner de vulnerabilidades Nexpose |
| [**Nessus**](https://www.tenable.com/products/nessus) | Pre-instalado | Integración con Nessus para escaneo de vulnerabilidades |
| [**Mimikatz**](http://blog.gentilkiwi.com/mimikatz) | Pre-instalado V.1 | Extracción de credenciales en memoria (Windows) |
| [**Stdapi**](https://www.rubydoc.info/github/rapid7/metasploit-framework/Rex/Post/Meterpreter/Extensions/Stdapi/Stdapi) | Pre-instalado | API estándar de Meterpreter |
| [**Railgun**](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-Railgun-for-Windows-post-exploitation) | Disponible | Permite llamadas directas a la API de Windows |
| [**Priv**](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/post/meterpreter/extensions/priv/priv.rb) | Disponible | Escalación de privilegios |
| [**Incognito**](https://www.offsec.com/metasploit-unleashed/fun-incognito/) | Pre-instalado | Manipulación de tokens de Windows |
| [**Darkoperator's**](https://github.com/darkoperator/Metasploit-Plugins) | Requiere instalación | Suite de automatización para pentesting |

Cada plugin debe estudiarse individualmente para maximizar su utilidad durante las evaluaciones de seguridad.

## Mixins: Fundamento Técnico de los Plugins

### Conceptos de Ruby y Programación Orientada a Objetos

Metasploit Framework está escrito en Ruby, un lenguaje de programación orientado a objetos. Esta característica arquitectónica es fundamental para la flexibilidad del framework.

### ¿Qué son los Mixins?

Los **Mixins** son clases que actúan como métodos utilizables por otras clases, sin necesidad de ser la clase padre. Esta relación no es herencia tradicional, sino **inclusión**.

**Diferencia clave:**
- **Herencia**: Una clase hija hereda todas las características de la clase padre
- **Mixin**: Una clase incluye métodos específicos de un módulo sin establecer relación de parentesco

### Casos de Uso de Mixins

Los Mixins se implementan cuando:

1. **Necesitamos proporcionar características opcionales**: Una clase puede incluir funcionalidades que no todos los objetos requieren
2. **Queremos reutilizar una característica en múltiples clases**: Un mismo comportamiento se necesita en clases sin relación de herencia

### Implementación en Ruby

En Ruby, los Mixins se implementan usando módulos con la palabra clave `include`:

```ruby
module MiMixin
  def metodo_compartido
    # código
  end
end

class MiClase
  include MiMixin
end
```

### Relevancia para Usuarios de Metasploit

Para usuarios que están iniciando con Metasploit, **no es necesario dominar los Mixins** para realizar evaluaciones de seguridad efectivas. Este concepto se menciona para:

- Comprender la arquitectura subyacente del framework
- Apreciar el nivel de personalización posible
- Prepararse para desarrollo avanzado de módulos personalizados

Para profundizar en Mixins, se puede consultar documentación adicional sobre Ruby y programación orientada a objetos.

## Referencias

- [Metasploit-Plugins de DarkOperator](https://github.com/darkoperator/Metasploit-Plugins)
- [Using Mixins](https://en.wikibooks.org/wiki/Metasploit/UsingMixins)
---

**Nota**: La familiarización con los plugins disponibles y sus capacidades es esencial para maximizar la eficiencia durante evaluaciones de seguridad. Se recomienda explorar la documentación específica de cada plugin antes de su implementación en entornos de producción.
