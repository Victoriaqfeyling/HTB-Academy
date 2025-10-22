# Hyper-V Administrators 

---

## Grupo **Hyper-V Administrators**


* Es un grupo de seguridad de Windows que otorga acceso completo a las funcionalidades de Hyper‑V en la máquina/servidor.

**Implicación importante:**

* Si los Controladores de Dominio (Domain Controllers) están virtualizados en Hyper‑V, los miembros de `Hyper‑V Administrators` pueden controlar esas máquinas virtuales. Con suficiente acceso a una VM que alojase un Domain Controller, un administrador de virtualización podría clonar la VM, acceder al disco virtual y obtener el fichero `NTDS.dit` (la base de datos de Active Directory), lo que permitiría extraer hashes NTLM de todas las cuentas del dominio.

---

## Comportamiento de `vmms.exe` y restauración de permisos


* Al borrar una máquina virtual, `vmms.exe` (el servicio de Virtual Machine Management Service de Hyper‑V) intenta restaurar los permisos originales del archivo `.vhdx` correspondiente.
* Lo hace ejecutándose como `NT AUTHORITY\SYSTEM` y *sin* suplantar (impersonate) al usuario.

**Consecuencia práctica:**

* Si eliminamos el `.vhdx` y luego creamos un *enlace duro* (native NT hard link) con ese mismo nombre apuntando a un archivo protegido por SYSTEM, cuando `vmms.exe` intente restaurar permisos lo hará en el archivo apuntado *como SYSTEM*. Así podemos forzar que el archivo protegido pase a tener permisos que nos permitan operar sobre él.

**Términos clave:**

* *vmms.exe*: servicio de administración de Hyper‑V.
* *NT AUTHORITY\SYSTEM*: la cuenta con más privilegios locales en Windows (similar a "root").
* *Hard link (enlace duro)*: una referencia a un archivo a nivel de sistema de ficheros NTFS que hace que dos nombres diferentes referencien el mismo contenido en disco.

---

## CVE mencionados y alternativa

**CVE-2018-0952 y CVE-2019-0841**

* El texto indica que, si el sistema operativo es vulnerable a alguna de estas vulnerabilidades, es posible aprovecharlas para lograr privilegios `SYSTEM` tras realizar el ataque con el enlace duro.

**Si no hay vulnerabilidad:**

* Como alternativa, podemos aprovechar una *aplicación instalada en el servidor* que haya instalado un servicio que se ejecute como `SYSTEM`, pero que sea *startable* (iniciable) por usuarios sin privilegios. Es decir, un servicio cuyo binario sea controlable tras la manipulación de permisos y que el servicio pueda ser arrancado por usuarios no privilegiados.

---

## Archivo objetivo 

El texto da un ejemplo concreto: el servicio de mantenimiento de Firefox.

**Ruta objetivo:**

```
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

**Por qué es relevante:**

* Mozilla instala un servicio denominado *Mozilla Maintenance Service* que ejecuta `maintenanceservice.exe`. Si conseguimos tomar control de ese ejecutable (permisos y propiedad), y el servicio corre como `SYSTEM` pero puede ser iniciado por un usuario local sin privilegios, podemos reemplazar el ejecutable por uno malicioso y, al iniciar el servicio, obtener ejecución como `SYSTEM`.

---

## Tomar la propiedad del archivo — `takeown`

**Comando mostrado en el texto:**

```
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

**Explicación de la sintaxis**

* `takeown`: utilidad de Windows para tomar la propiedad de un archivo o directorio.
* `/F <ruta>`: especifica la ruta del archivo o carpeta sobre la que se desea tomar la propiedad.



* El comando cambia el dueño del fichero al usuario que ejecuta `takeown` (o a un grupo que represente al usuario), de forma que después se pueden modificar los permisos (ACL) si el usuario tiene suficientes privilegios para hacerlo.

**Nota importante:** en el flujo descrito, antes del `takeown` se ejecuta el script de PowerShell que, gracias al exploit del enlace duro, nos habrá concedido control total del fichero; una vez tengamos control, `takeown` nos asegura la propiedad para poder sobrescribir el archivo.

---

## Iniciar el servicio y ejecución como SYSTEM — `sc.exe start`

**Comando:**

```
C:\htb> sc.exe start MozillaMaintenance
```

**Explicación**

* `sc.exe`: herramienta de línea de comandos para administrar servicios en Windows.
* `start <NombreServicio>`: orden para arrancar el servicio cuyo nombre interno es `<NombreServicio>`.

#

* Tras reemplazar `maintenanceservice.exe` por una versión maliciosa (por ejemplo, un ejecutable que abra una shell), iniciar el servicio con `sc.exe start MozillaMaintenance` hará que el servicio se ejecute bajo la cuenta `SYSTEM` (según cómo esté configurado el servicio). Si el servicio realmente corre como `SYSTEM`, el binario malicioso se ejecutará con esos privilegios y nos dará ejecución como `SYSTEM`.

---

## Mitigación 

El texto finaliza diciendo que **esta vector fue mitigado por las actualizaciones de seguridad de marzo de 2020**, que cambiaron el comportamiento relacionado con hard links. Es decir, en sistemas parcheados después de esa actualización, el ataque explicado podría no funcionar porque Windows ya no permite la elevación a través de ese mecanismo tal como se describía.

---

## Resumen

1. Miembros de `Hyper‑V Administrators` pueden acceder a VMs y a discos virtuales (riesgo para controladores de dominio).
2. `vmms.exe` restaura permisos como `SYSTEM`; creando un hard link que apunte a un archivo protegido podemos forzar cambios de permiso.
3. Si existe una vulnerabilidad (CVE citadas) o un servicio `SYSTEM` startable por usuarios sin privilegios, podemos reemplazar el ejecutable objetivo (ej. `maintenanceservice.exe`), tomar propiedad (`takeown /F <ruta>`) y arrancar el servicio (`sc.exe start <NombreServicio>`) para obtener ejecución como `SYSTEM`.
4. El vector fue mitigado por parches de marzo de 2020.

---
