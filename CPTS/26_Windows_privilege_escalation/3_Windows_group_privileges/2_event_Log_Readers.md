# Event Log Readers 


## Contexto general

Cuando está activado el [*auditing* (registro) de la creación de procesos](https://learn.microsoft.com/es-es/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/audit-process-creation) y la captura de la línea de comando correspondiente, Windows guarda esa información en el registro de seguridad como el [**evento ID 4688: "A new process has been created"**](https://learn.microsoft.com/es-es/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4688). Esto permite a los defensores ver qué binarios se ejecutan y con qué parámetros — por ejemplo, si el parámetro contiene una contraseña. Estos eventos pueden exportarse a un SIEM o a una herramienta de búsqueda (por ejemplo ElasticSearch) para detectar actividad sospechosa.

El texto menciona ejemplos de comandos que atacantes suelen ejecutar tras obtener acceso (por ejemplo: `tasklist`, `ver`, `ipconfig`, `systeminfo`), comandos para reconocimiento (`dir`, `net view`, `ping`, `net use`, `type`) y comandos usados para movimiento lateral o ejecución de tareas (`at`, `reg`, `wmic`, `wusa`).

`Nota`: [Estudio sobre comandos mas utilizados por atacantes](https://blogs.jpcert.or.jp/en/2016/01/windows-commands-abused-by-attackers.html)

También se menciona que, además de monitorizar, una organización puede restringir la ejecución de comandos mediante reglas afinadas de AppLocker.

---

## Permisos — grupo *Event Log Readers*

El grupo [**Event Log Readers**](https://learn.microsoft.com/es-es/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255(v=ws.11)?redirectedfrom=MSDN#event-log-readers) permite a sus miembros **leer** los registros de eventos del equipo local. Esto es útil si un administrador quiere que usuarios con ciertos permisos (p. ej. power users o desarrolladores) puedan revisar logs sin darles privilegios administrativos completos.

### Ejemplo de comprobación de pertenencia al grupo

En el texto aparece el comando de Windows `net localgroup` para listar los miembros de un grupo local.

```text
C:\htb> net localgroup "Event Log Readers"

Alias name     Event Log Readers
Comment        Members of this group can read event logs from local machine

Members

-------------------------------------------------------------------------------
logger
The command completed successfully.
```

**Explicación de sintaxis:**

* `net localgroup "Event Log Readers"`

  * `net localgroup` es la subcomando de `net` para gestionar grupos locales.
  * La cadena entre comillas especifica el nombre del grupo local (aquí contiene un espacio, por eso se usan comillas).
  * La salida lista el alias, comentario y los miembros actuales.

---

## Riesgo: contraseñas en parámetros de línea de comando

Microsoft publica referencia de comandos integrados y muchos aceptan contraseñas como parámetros. Si se registra la línea de comando (process command line auditing habilitado), **esa contraseña quedará en los eventos**. El texto destaca esto como un riesgo de seguridad importante.

---

## Consultar eventos con `wevtutil`

El texto muestra que podemos consultar eventos desde la línea de comandos con [`wevtutil`](https://learn.microsoft.com/es-es/windows-server/administration/windows-commands/wevtutil). Ejemplo en el texto:

```powershell
PS C:\htb> wevtutil qe Security /rd:true /f:text | Select-String "/user"

        Process Command Line:   net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

**Despiece y explicación de cada parte:**

* `wevtutil` — utilidad de línea de comandos para interactuar con los registros de eventos de Windows.
* `qe` — abreviatura de "query events" (consultar eventos).
* `Security` — nombre del registro al que consultamos (registro de seguridad).
* `/rd:true` — indica lectura en orden inverso (reverse direction: recientes primero). En `wevtutil` este modificador invierte el orden de lectura.
* `/f:text` — formato de salida: texto legible.
* `|` — tubería de PowerShell/cmd: toma la salida del comando anterior y la pasa al siguiente.
* `Select-String "/user"` — comando de PowerShell que filtra líneas que contienen la cadena `/user`.

La salida muestra la línea de comando registrada: `net use T: \\fs01\backups /user:tim MyStr0ngP@ssword` — observamos que la contraseña aparece como parte de la línea de comando.

### Pasar credenciales a `wevtutil`

El texto indica que `wevtutil` admite parámetros `/u` y `/p` para especificar credenciales alternativas.

```text
C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

**Explicación de parámetros adicionales:**

* `/r:share01` — especifica el equipo remoto (remote) al que queremos consultar.
* `/u:julie.clay` — usuario remoto con el que autenticarnos.
* `/p:Welcome1` — contraseña asociada al usuario remoto. (Observa el riesgo: aquí la contraseña aparece en el comando local si se ejecuta manualmente.)
* `findstr "/user"` — en este ejemplo usan `findstr` (equivalente a `grep` en Windows) para filtrar la salida.

---

## Consultar eventos con `Get-WinEvent` (PowerShell)

El texto muestra cómo usar [`Get-WinEvent`](https://learn.microsoft.com/es-es/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.5&viewFallbackFrom=powershell-7.1) para filtrar eventos de creación de procesos (ID 4688) y extraer la propiedad que contiene la línea de comando.

```powershell
PS C:\htb> Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}

CommandLine
-----------
net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

**Explicación paso a paso:**

* `Get-WinEvent -LogName security` — obtiene todos los eventos del registro `Security`.
* `| where { ... }` — filtra objetos de evento con condiciones dentro del bloque `{ }`.

  * `$_` es la variable automática que representa el objeto actual en el pipeline.
  * `$.ID -eq 4688` — selecciona sólo eventos con ID igual a 4688 (creación de procesos).
  * `-and` — operador lógico AND.
  * `$_.Properties[8].Value -like '*/user*'` — comprueba si la propiedad en el índice 8 (que en este contexto contiene la línea de comando) contiene la cadena `/user`.

    * `Properties` es una colección; el índice exacto puede variar con versiones/idiomas, pero en el texto usan `[8]` porque ahí está el valor de la línea de comando.
    * `-like '*/user*'` usa comodines `*` para buscar subcadenas que incluyan `/user`.
* `| Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}` — proyecta el resultado creando una columna llamada `CommandLine` cuyo contenido es la propiedad `Properties[8].Value` (la línea de comando capturada).

**Salida:** se muestra la línea de comando completa capturada por el evento.

### Nota importante sobre permisos (texto original)

El texto subraya que **buscar en el registro de seguridad con `Get-WinEvent` requiere acceso administrativo** o que se cambie el permiso en la clave de registro: `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. La membresía sólo en el grupo *Event Log Readers* **no** es suficiente para ejecutar esta operación en el registro de seguridad.

---

## Ejecución con otras credenciales en `Get-WinEvent`

El texto menciona que el cmdlet `Get-WinEvent` puede ejecutarse como otro usuario mediante el parámetro `-Credential` (esto permitiría proporcionar credenciales con más privilegios para leer el log de seguridad si la cuenta lo permite).

---

## Otros logs: *PowerShell Operational*

El texto finaliza indicando que el *PowerShell Operational log* puede contener información sensible (por ejemplo si está habilitado *script block logging* o *module logging*), y que ese registro **es accesible por usuarios sin privilegios**. Esto lo distingue del registro de seguridad, ya que puede ofrecer otra vía para encontrar comandos o credenciales si está habilitado el tipo de logging que captura el contenido de los scripts.

---

