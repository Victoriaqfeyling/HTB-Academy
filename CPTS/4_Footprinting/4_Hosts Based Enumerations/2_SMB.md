# SMB y Samba (Footprinting / Enumeración)


---

## 1) ¿Qué es SMB?

**Server Message Block (SMB)** es un protocolo **cliente-servidor** que regula el acceso a:

* **Archivos** y **directorios** compartidos.
* **Recursos de red**: impresoras, routers, interfaces o servicios publicados.
* **Intercambio de información** entre procesos en red (según implementación).

Históricamente, [SMB](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb/f210069c-7086-4dc2-885e-861d837df688) se popularizó en entornos Microsoft (p.ej., LAN Manager / LAN Server) y se volvió el estándar de facto en la familia **Windows**, manteniendo **compatibilidad hacia atrás** (equipos nuevos pueden comunicarse con versiones antiguas).

Para sistemas Linux/Unix existe el proyecto **Samba**, que habilita comunicación **cross‑platform** vía SMB.

---

## 2) Modelo de comunicación: cliente ↔ servidor

SMB permite que un cliente se comunique con otros participantes de la misma red para acceder a archivos o servicios compartidos. Para que esto funcione:

* El equipo remoto debe tener implementado el protocolo.
* Debe existir una aplicación servidor SMB que reciba, procese y responda solicitudes.

Antes de intercambiar datos, ambas partes **negocian y establecen la conexión** (intercambian mensajes de sesión/capabilities).

### SMB sobre TCP

En redes IP, SMB utiliza **TCP**, por lo que hereda el **three‑way handshake** (SYN → SYN/ACK → ACK). Luego, TCP gobierna la entrega ordenada de datos y retransmisiones.

[Ejemplos](https://web.archive.org/web/20240815212710/https://winprotocoldoc.blob.core.windows.net/productionwindowsarchives/MS-SMB2/%5BMS-SMB2%5D.pdf#%5B%7B%22num%22%3A920%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C69%2C738%2C0%5D)

---

## 3) Shares, jerarquía y ACL

Un servidor SMB puede exponer partes arbitrarias del filesystem local como **shares** (recursos compartidos). Por eso:

* La **jerarquía** que ve el cliente puede ser **parcialmente independiente** de la estructura real del servidor.
* Los permisos se rigen por **ACL (Access Control Lists)** definidas sobre el **share**.

### ACL en SMB

Las ACL permiten control fino por:

* Usuario
* Grupo
* Tipo de acceso (lectura, escritura, ejecución, control total, etc.)

Importante: estas ACL están **definidas a nivel share** y **no necesariamente** reflejan exactamente los permisos locales del sistema de archivos del servidor.

---

## 4) Samba, CIFS y SMB/CIFS

**Samba** es una implementación alternativa de SMB para sistemas Unix‑like. Implementa [**CIFS (Common Internet File System)**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/934c2faa-54af-4526-ac74-6a24d126724e), que es un **dialecto** de SMB (una variante/implementación específica). Por eso muchas veces se ve como **SMB/CIFS**.

### CIFS y puertos

* CIFS se asocia principalmente a **SMBv1** (considerado legado).
* Cuando SMB se transmite hacia servicios antiguos basados en **NetBIOS**, suelen aparecer conexiones por:

  * **TCP/137**, **UDP/137** (servicio de nombres NetBIOS)
  * **UDP/138** (datagram)
  * **TCP/139** (NetBIOS session)
* En cambio, **SMB directo** (más moderno) opera principalmente sobre **TCP/445**.

### Versiones SMB (resumen)

| Versión       | Soporte típico           | Características                                  |
| ------------- | ------------------------ | ------------------------------------------------ |
| **CIFS**      | Windows NT 4.0           | Comunicación vía interfaz NetBIOS                |
| **SMB 1.0**   | Windows 2000             | Conexión directa vía TCP                         |
| **SMB 2.0**   | Vista / Server 2008      | Mejoras de performance, message signing, caching |
| **SMB 2.1**   | Win 7 / Server 2008 R2   | Mecanismos de locking                            |
| **SMB 3.0**   | Win 8 / Server 2012      | Multichannel, cifrado end‑to‑end, remote storage |
| **SMB 3.0.2** | Win 8.1 / Server 2012 R2 | (mejoras incremental)                            |
| **SMB 3.1.1** | Win 10 / Server 2016     | Integrity checking, cifrado AES‑128              |

> En infra modernas se prefiere **SMB2/SMB3**. SMB1/CIFS es legado y suele ser desaconsejado, aunque puede sobrevivir en entornos puntuales.

---

## 5) Samba y Active Directory

* Con **Samba v3**, el servidor puede ser miembro completo de un dominio **Active Directory**.
* Con **Samba v4**, Samba puede actuar como **Domain Controller (DC)**.

Para esto, Samba utiliza *daemons* (procesos en background):

* **smbd**: provee funcionalidades SMB principales (file sharing, autenticación/servicio, etc.).
* **nmbd**: implementa funciones relacionadas con NetBIOS (nombres, browsing, etc.).
* El servicio SMB coordina estos procesos.

---

## 6) Workgroup, NetBIOS, NBNS y WINS

En una red SMB clásica, cada host suele pertenecer a un **workgroup** (grupo de trabajo). Un workgroup identifica una colección de equipos y recursos.

**NetBIOS** (Network Basic Input/Output System) es una API histórica para networking, desarrollada por `IBM`. En entornos NetBIOS:

* Cuando una máquina “aparece” en la red, necesita registrar un nombre.
* Esto puede hacerse:

  * Por registro directo del hostname (cada host reserva su nombre), o
  * Usando un **NBNS** ([NetBIOS Name Server](https://networkencyclopedia.com/netbios-name-server-nbns/)).
* En Windows, NBNS evolucionó en [**WINS**](https://networkencyclopedia.com/windows-internet-name-service-wins/).

---

## 7) Configuración por defecto de Samba (`smb.conf`)

Samba se configura mediante un archivo de texto, normalmente:

* `/etc/samba/smb.conf`

Ejemplo (filtrando comentarios):

```bash
cat /etc/samba/smb.conf | grep -v "#\|;"
```

```
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

* Sección **[global]**: aplica como base para todos los shares.
* Secciones **[share]**: definen recursos compartidos específicos.

Ejemplo simplificado:

* `[global]` define `workgroup`, `server string`, logs, rol de servidor, PAM, mapeo a guest, etc.
* Shares por defecto relacionados con impresión:

  * `[printers]`
  * `[print$]`

> Importante: los settings globales pueden ser **sobrescritos** por share. Ahí aparecen muchas misconfigs típicas.

Más [opciones de configuración](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)

---

## 8) Opciones comunes en Samba (tabla)

| Setting | Descripción |
|---|---|
| `[sharename]` | Nombre del recurso compartido (share) publicado. |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup/dominio que verán los clientes al consultar. |
| `path = /path/here/` | Ruta local del servidor que se expone como share. |
| `server string = STRING` | Texto descriptivo mostrado al iniciar conexión (identificación). |
| `unix password sync = yes` | Sincroniza la contraseña UNIX con la contraseña SMB. |
| `usershare allow guests = yes` | Permite “usershares” accesibles sin autenticación (según definición). |
| `map to guest = bad user` | Si el usuario no existe, lo mapea al usuario guest. |
| `browseable = yes` | El share aparece en listados de recursos disponibles. |
| `guest ok = yes` | Permite entrar al share como invitado (sin credenciales). |
| `read only = yes` | Restringe a lectura: no crear, modificar ni borrar. |
| `create mask = 0700` | Permisos por defecto en archivos nuevos creados (owner). |


**Diferencia:** `usershare allow guests` habilita que los *usershares* (shares creados por usuarios) puedan ser accesibles sin autenticación, mientras que `guest ok` permite que un share específico acepte conexiones como usuario invitado (guest) sin credenciales.
En Samba hay shares definidos por el administrador en el archivo `smb.conf` (shares “estáticos” o globales) y también existen los llamados `usershares`, que pueden ser creados dinámicamente por usuarios locales del sistema si la configuración lo permite.

Los usershares dependen del parámetro `usershare allow guests`, mientras que los shares definidos en `smb.conf` pueden permitir acceso invitado mediante `guest ok = yes`.

---

## 9) Settings peligrosos (por qué importan)

Algunas opciones son “cómodas” para usuarios internos, pero peligrosas si el servicio queda accesible indebidamente:

| Setting                                        | Riesgo/Impacto                                                                 |
| ---------------------------------------------- | ------------------------------------------------------------------------------ |
| `browseable = yes`                             | Facilita descubrir shares y su estructura                                      |
| `read only = no` / `writable = yes`            | Permite crear/modificar archivos (aumenta impacto)                             |
| `guest ok = yes`                               | Acceso anónimo: más superficie de ataque                                       |
| `create mask = 0777` / `directory mask = 0777` | Permisos excesivos en archivos/dirs nuevos                                     |
| `logon script = script.sh`                     | Script ejecutado al login: puede ser vector de abuso si se controla            |
| `magic script` / `magic output`                | Automatizaciones que pueden ejecutar acciones inesperadas                      |
| `enable privileges = yes`                      | Respeta privilegios asignados a SIDs específicos (riesgo si está mal diseñado) |

### Ejemplo: por qué `browseable = yes` es sensible

Para un administrador, permite que empleados encuentren y naveguen shares fácilmente. Pero **el atacante** también podrá enumerarlos tras obtener acceso (o incluso de forma anónima) y encontrar información sensible o rutas de abuso.

---

## 10) Ejemplo de share inseguro (`[notes]`)

Ejemplo de share con settings peligrosos:

```ini
[notes]
    comment = CheckIT
    path = /mnt/notes/

    browseable = yes
    read only = no
    writable = yes
    guest ok = yes

    enable privileges = yes
    create mask = 0777
    directory mask = 0777
```

Esto suele aparecer en contextos de testing o redes internas pequeñas. El problema es que muchas veces queda “olvidado” y expone:

* **Enumeración completa** de shares.
* Alta probabilidad de **descarga** y **modificación** de contenido.

---

## 11) Reinicio del servicio

Tras modificar `/etc/samba/smb.conf`:

```bash
sudo systemctl restart smbd
```

---

## 12) Enumeración manual con `smbclient`

### Listar shares (-L) con null session (-N)

```bash
smbclient -N -L //<IP>
```

* `-N` usa **null session** (sin usuario/contraseña).
* Te devuelve lista de shares y comentarios.


```
smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```


### Conectarse a un share

```bash
smbclient //<IP>/notes
```

Si permite acceso anónimo, puede decir “Anonymous login successful”.


```
Enter WORKGROUP\<username>'s password: 
Anonymous login successful
Try "help" to get a list of possible commands.


smb: \> help

?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!            


smb: \> ls

  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

### Comandos útiles dentro de `smbclient`

* `help` → lista comandos disponibles.
* `ls` / `dir` → listar contenido.
* `get <archivo>` → descargar.

### Ejecutar comandos locales sin salir

`smbclient` permite ejecutar comandos locales con `!`:

* `!ls`
* `!cat <archivo>`

> Esto es útil para validar rápidamente que el archivo se descargó y para inspección local.

---

## 13) Monitoreo desde el servidor: `smbstatus`


Desde el punto de vista administrativo, el comando:

```bash
smbstatus
```

permite visualizar las sesiones activas en el servidor Samba. Con esta herramienta es posible identificar:

* Qué usuarios están conectados.
* Desde qué host o dirección IP se originan las conexiones.
* A qué recurso compartido (share) accede cada cliente.
* La versión del protocolo SMB utilizada y ciertos parámetros de seguridad.

Esta información es especialmente relevante en redes internas, ya que permite detectar accesos inesperados o actividad sospechosa, incluso dentro de subredes aisladas. En entornos corporativos, donde múltiples usuarios acceden simultáneamente a recursos compartidos, este monitoreo resulta clave para la auditoría y el control de accesos.

---

## Samba en Seguridad a Nivel de Dominio

Cuando Samba forma parte de un entorno con **Active Directory**, el modelo de autenticación cambia significativamente. En este caso, el servidor Samba actúa como miembro de un dominio Windows y delega la validación de credenciales al **Domain Controller (DC)**.

El Domain Controller es el componente central que:

* Mantiene la base de datos de usuarios y grupos.
* Almacena hashes de contraseñas.
* Aplica políticas de autenticación.
* Autoriza el acceso a recursos compartidos.

La información crítica del dominio se gestiona principalmente en la base de datos **NTDS.dit**, que contiene usuarios, grupos y credenciales. Además, el sistema de autenticación se apoya en componentes como el **Security Account Manager (SAM)** para la gestión de cuentas.

Cuando un usuario intenta acceder a un share SMB en un entorno de dominio:

1. El cliente envía sus credenciales.
2. El servidor Samba consulta al Domain Controller.
3. El DC valida la identidad del usuario.
4. Si la autenticación es correcta, el servidor aplica las ACL del share y concede o deniega el acceso.

Esto significa que SMB no es únicamente un protocolo de compartición de archivos, sino también un punto crítico dentro del esquema de autenticación del dominio.

---

## Importancia en Footprinting

En un entorno de dominio, la exposición o mala configuración de SMB puede permitir:

* Enumeración de usuarios del dominio.
* Descubrimiento de shares internos.
* Identificación de políticas débiles.

Por ello, comprender la relación entre Samba, Active Directory y el Domain Controller es fundamental para evaluar correctamente el impacto de un servicio SMB expuesto dentro de una infraestructura corporativa.



```bash
smbstatus
```

```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           

No locked files
```

Muestra:

* Versión de Samba.
* Usuario conectado.
* Host origen.
* Share al que está conectado.
* Dialecto/protocolo (ej.: SMB3_11).
* Estado de cifrado y signing.

---

## 14) Footprinting con Nmap (puertos 139/445)


Volviendo a nuestras herramientas de enumeración, **Nmap** es una de las primeras opciones para analizar un servicio SMB de forma automatizada. Nmap no solo permite detectar puertos abiertos y versiones de servicio, sino que también incorpora scripts del **Nmap Scripting Engine (NSE)** específicamente diseñados para SMB.

Estos scripts pueden ayudarnos a:

* Identificar la versión del servicio (por ejemplo, Samba).
* Detectar configuraciones de seguridad como SMB signing.
* Obtener información del sistema operativo.
* Enumerar información básica del dominio o workgroup.

Sin embargo, este tipo de escaneos puede tardar más tiempo, especialmente cuando se utilizan múltiples scripts o configuraciones agresivas. Además, aunque Nmap es muy útil, no siempre muestra todos los detalles que pueden obtenerse mediante interacción manual con el servicio.

Por esta razón, es recomendable combinar:

* 🔎 Enumeración automatizada (Nmap + NSE)
* 🖐 Enumeración manual (smbclient, rpcclient, etc.)

---

### ¿Por qué escaneamos los puertos 139 y 445?

El servicio SMB puede operar sobre dos puertos principales:

### 🔹 TCP 139 – NetBIOS Session Service

Este puerto se utiliza cuando SMB funciona sobre la capa NetBIOS. Es más común en implementaciones antiguas o configuraciones que mantienen compatibilidad hacia atrás.

### 🔹 TCP 445 – SMB directo sobre TCP

Este es el puerto utilizado por versiones modernas de SMB (SMB2/SMB3). Aquí SMB funciona directamente sobre TCP sin necesidad de NetBIOS.

Escanear ambos puertos es importante porque:

* Algunos sistemas pueden exponer solo uno de ellos.
* Pueden estar habilitadas diferentes versiones del protocolo en cada puerto.
* NetBIOS puede revelar información adicional como nombres de host o workgroup.
* Nos permite detectar configuraciones heredadas o inseguras.

Un escaneo típico sería:

```bash
sudo nmap -sV -sC -p139,445 <IP>
```

Con este comando buscamos:

* Detectar si SMB está activo.
* Identificar la versión del servicio.
* Ejecutar scripts básicos de enumeración.

En el siguiente paso analizaremos qué información concreta puede devolver Nmap sobre nuestro servidor Samba de prueba, donde hemos creado el share `[notes]` para fines de laboratorio.


```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for sharing.inlanefreight.htb (10.129.14.128)
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.35 seconds
```

* Scripts host:

  * `nbstat` (nombres NetBIOS)
  * `smb2-security-mode` (signing habilitado/obligatorio)
  * `smb2-time` (hora del servidor)

> Limitación: a veces Nmap no devuelve demasiado detalle, por lo que conviene complementarlo con herramientas de interacción directa.

---

## 15) Enumeración MS‑RPC con `rpcclient`

### Concepto: RPC

[**Remote Procedure Call (RPC)**](https://www.geeksforgeeks.org/operating-systems/remote-procedure-call-rpc-in-operating-system/) permite invocar funciones remotas (pasando parámetros y recibiendo resultados). En SMB/Windows/Samba es central para enumeración.

[Manual](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)




### ¿Qué es MS-RPC?

MS-RPC (Microsoft Remote Procedure Call) es la implementación de Microsoft del concepto de **Remote Procedure Call (RPC)**. RPC es un mecanismo que permite que un programa ejecute funciones en otro sistema remoto como si fueran llamadas locales.

En lugar de simplemente transferir archivos, RPC permite invocar procedimientos específicos en un servidor remoto, enviando parámetros y recibiendo respuestas estructuradas.

---

### ¿Por qué se usa MS-RPC en SMB?

Aunque SMB se conoce principalmente como un protocolo de compartición de archivos, en entornos Windows cumple muchas más funciones. Para ello, utiliza MS-RPC encapsulado dentro del propio canal SMB (normalmente sobre el puerto 445).

Gracias a MS-RPC, SMB puede:

* Enumerar usuarios y grupos del dominio.
* Consultar información del sistema.
* Gestionar servicios remotos.
* Interactuar con componentes de Active Directory.
* Realizar operaciones administrativas.

Es decir, SMB no solo transporta archivos, sino también llamadas a funciones remotas del sistema.

---

### Relación entre SMB y MS-RPC

En entornos modernos:

* SMB opera sobre TCP 445.
* Dentro de esa conexión, se encapsulan llamadas MS-RPC.
* Esto permite que herramientas como `rpcclient`, `enum4linux` o `CrackMapExec` interactúen con el sistema remoto.

Por eso, cuando enumeramos SMB, en realidad también estamos interactuando con servicios RPC expuestos a través de él.

---

### Resumen

SMB es el canal de transporte.
MS-RPC es el mecanismo que permite ejecutar funciones remotas dentro de ese canal.

Sin MS-RPC, SMB sería solo un protocolo de transferencia de archivos. Con MS-RPC, se convierte en un componente clave de la administración y autenticación en redes Windows.


### Conexión anónima (null session)

```bash
rpcclient -U "" <IP>
```

### Consultas útiles (resumen)

| Query | Descripción |
|-------|-------------|
| srvinfo | Devuelve información general del servidor, como nombre, versión del sistema operativo y tipo de plataforma. |
| enumdomains | Enumera los dominios o workgroups disponibles en el servidor. |
| querydominfo | Muestra información detallada del dominio, incluyendo nombre, rol del servidor y estadísticas de usuarios. |
| netshareenumall | Enumera todos los recursos compartidos (shares) disponibles en el servidor SMB. |
| netsharegetinfo <share> | Obtiene información específica de un recurso compartido, como ruta local, permisos y configuración. |
| enumdomusers | Enumera los usuarios registrados en el dominio o sistema. |
| queryuser <RID> | Devuelve información detallada de un usuario específico a partir de su RID (Relative Identifier). |

### Qué se puede filtrar a anónimos

Con `rpcclient` puede filtrarse:

* Dominio/workgroup
* Shares y paths
* Usuarios (dependiendo de configuración)
* ACLs/SIDs en algunos casos

> Conclusión operativa: si existe acceso anónimo, un único error de permisos/visibilidad puede elevar mucho el riesgo.

```
rpcclient $> srvinfo

        DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
			
rpcclient $> enumdomains

name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]


rpcclient $> querydominfo

Domain:         DEVOPS
Server:         DEVSMB
Comment:        DEVSM
Total Users:    2
Total Groups:   0
Total Aliases:  0
Sequence No:    1632361158
Force Logoff:   -1
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1


rpcclient $> netshareenumall

netname: print$
        remark: Printer Drivers
        path:   C:\var\lib\samba\printers
        password:
netname: home
        remark: INFREIGHT Samba
        path:   C:\home\
        password:
netname: dev
        remark: DEVenv
        path:   C:\home\sambauser\dev\
        password:
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
netname: IPC$
        remark: IPC Service (DEVSM)
        path:   C:\tmp
        password:
		
		
rpcclient $> netsharegetinfo notes

netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
        type:   0x0
        perms:  0
        max_uses:       -1
        num_uses:       1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE 
DACL
        ACL     Num ACEs:       1       revision:       2
        ---
        ACE
                type: ACCESS ALLOWED (0) flags: 0x00 
                Specific bits: 0x1ff
                Permissions: 0x101f01ff: Generic all access SYNCHRONIZE_ACCESS WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS 
                SID: S-1-1-0
```

---

# Enumeración de Usuarios SMB mediante `rpcclient`

## Riesgo del Acceso Anónimo

Los ejemplos anteriores demuestran que un servidor SMB mal configurado puede filtrar información sensible incluso a usuarios anónimos. Cuando un servicio de red permite conexiones sin autenticación (null session), el riesgo no está únicamente en el acceso inicial, sino en la **información que puede ser enumerada sin credenciales válidas**.

Un simple error de configuración puede otorgar:

* Visibilidad sobre usuarios del dominio.
* Información sobre grupos internos.
* Detalles de cuentas y políticas.

Esto incrementa significativamente la superficie de ataque, ya que conocer los nombres de usuarios es el primer paso para realizar ataques como:

* Password spraying.
* Fuerza bruta dirigida.
* Ataques de credenciales débiles.

El factor humano suele ser el eslabón más débil: contraseñas simples, reutilización de claves o falta de políticas estrictas pueden facilitar el compromiso del sistema.

---

# Enumeración de Usuarios con `rpcclient`

Una vez establecida una sesión anónima:

```bash
rpcclient -U "" <IP>
```

Podemos utilizar distintos comandos para extraer información.

---

### 1️⃣ Enumerar usuarios del dominio

```bash
rpcclient $> enumdomusers

user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]
```

Este comando devuelve:

* Nombre de usuario.
* RID (Relative Identifier).

El RID es importante porque identifica de forma única a cada usuario dentro del dominio.

---

### 2️⃣ Obtener información detallada de un usuario

### Usuario cry0l1t3

```bash
rpcclient $> queryuser 0x3e9

        User Name   :   cry0l1t3
        Full Name   :   cry0l1t3
        Home Drive  :   \\devsmb\\cry0l1t3
        Dir Drive   :
        Profile Path:   \\devsmb\\cry0l1t3\\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:50:56 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:50:56 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e9
        group_rid:      0x201
        acb_info :      0x00000014
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...
```

Información relevante que podemos extraer:

* Ruta del perfil y home directory.
* Fecha del último cambio de contraseña.
* Grupo principal (group_rid).
* Contador de intentos fallidos.

---

### Usuario mrb3n

```bash
rpcclient $> queryuser 0x3e8

        User Name   :   mrb3n
        Full Name   :
        Home Drive  :   \\devsmb\\mrb3n
        Dir Drive   :
        Profile Path:   \\devsmb\\mrb3n\\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:47:59 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:47:59 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e8
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...
```

---

### 3️⃣ Enumeración de grupo asociado

Ambos usuarios pertenecen al grupo con RID 0x201.

```bash
rpcclient $> querygroup 0x201

        Group Name:     None
        Description:    Ordinary Users
        Group Attribute:7
        Num Members:2
```

Esto indica:

* El grupo corresponde a usuarios estándar.
* Contiene 2 miembros.

---

### Impacto de Seguridad

Si un atacante puede realizar esta enumeración de manera anónima:

* Obtiene una lista válida de usuarios.
* Reduce el ruido en ataques de fuerza bruta.
* Puede dirigir ataques de password spraying.
* Puede correlacionar información con otras fuentes OSINT.

La enumeración de usuarios es una fase crítica en el reconocimiento interno, ya que transforma un acceso anónimo limitado en una oportunidad real de compromiso si existen credenciales débiles





---


## 16) Enumeración de usuarios por RIDs

Aunque algunos comandos estén restringidos, `queryuser <RID>` suele funcionar si el RID es válido. Por eso se puede:

* **Bruteforcear RIDs** y extraer usuarios/grupos.

Ejemplo:

```bash
for i in $(seq 500 1100); do
  rpcclient -N -U "" <IP> -c "queryuser 0x$(printf '%x\n' $i)" \
  | grep "User Name\|user_rid\|group_rid" && echo "";
done
```


```
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201
		
        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
		
        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

---

## 17) Alternativas: Impacket / SMBMap / CrackMapExec / enum4linux‑ng

### Impacket: `samrdump.py`

Permite enumerar usuarios y metadatos del dominio/workgroup.

- [Impacket](https://github.com/fortra/impacket)
- [samrdump](https://github.com/fortra/impacket/blob/master/examples/samrdump.py)

```
samrdump.py 10.129.14.128

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: 
mrb3n (1000)/UserComment: 
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
mrb3n (1000)/ScriptPath: 
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/UserComment: 
cry0l1t3 (1001)/PrimaryGroupId: 513
cry0l1t3 (1001)/BadPasswordCount: 0
cry0l1t3 (1001)/LogonCount: 0
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
cry0l1t3 (1001)/ScriptPath: 
[*] Received 2 entries.
```

### SMBMap

- [smbmap](https://github.com/ShawnDEvans/smbmap)

```bash
smbmap -H <IP>
```

```
smbmap -H 10.129.14.128

[+] Finding open SMB ports....
[+] User SMB session established on 10.129.14.128...
[+] IP: 10.129.14.128:445       Name: 10.129.14.128                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        home                                                    NO ACCESS       INFREIGHT Samba
        dev                                                     NO ACCESS       DEVenv
        notes                                                   NO ACCESS       CheckIT
        IPC$                                                    NO ACCESS       IPC Service (DEVS
```

Útil para ver shares y permisos según sesión.

### CrackMapExec (CME)

- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)

```bash
crackmapexec smb <IP> --shares -u '' -p ''
```

```
crackmapexec smb 10.129.14.128 --shares -u '' -p ''

SMB         10.129.14.128   445    DEVSMB           [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB         10.129.14.128   445    DEVSMB           [+] \: 
SMB         10.129.14.128   445    DEVSMB           [+] Enumerated shares
SMB         10.129.14.128   445    DEVSMB           Share           Permissions     Remark
SMB         10.129.14.128   445    DEVSMB           -----           -----------     ------
SMB         10.129.14.128   445    DEVSMB           print$                          Printer Drivers
SMB         10.129.14.128   445    DEVSMB           home                            INFREIGHT Samba
SMB         10.129.14.128   445    DEVSMB           dev                             DEVenv
SMB         10.129.14.128   445    DEVSMB           notes           READ,WRITE      CheckIT
SMB         10.129.14.128   445    DEVSMB           IPC$                            IPC Service (DEVSM)
```


Suele reportar:

* Nombre de host
* Dominio
* Signing
* SMBv1 habilitado/deshabilitado
* Shares y permisos (READ/WRITE)

### enum4linux‑ng

Automatiza consultas y devuelve mucha información:

* Nombres NetBIOS, workgroup
* Dialectos SMB soportados
* Null session permitida
* Usuarios, shares, políticas


### Instalación

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
python3 -m venv enum4linux-env
source enum4linux-env/bin/activate
pip3 install -r requirements.txt
```

```bash
./enum4linux-ng.py 10.129.14.128 -A

ENUM4LINUX - next generation

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.14.128
[*] Username ......... ''
[*] Random Username .. 'juzgtcsu'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 =====================================
|    Service Scan on 10.129.14.128    |
 =====================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =====================================================
|    NetBIOS Names and Workgroup for 10.129.14.128    |
 =====================================================
[+] Got domain/workgroup name: DEVOPS
[+] Full NetBIOS names information:
- DEVSMB          <00> -         H <ACTIVE>  Workstation Service
- DEVSMB          <03> -         H <ACTIVE>  Messenger Service
- DEVSMB          <20> -         H <ACTIVE>  File Server Service
- ..__MSBROWSE__. <01> - <GROUP> H <ACTIVE>  Master Browser
- DEVOPS          <00> - <GROUP> H <ACTIVE>  Domain/Workgroup Name
- DEVOPS          <1d> -         H <ACTIVE>  Master Browser
- DEVOPS          <1e> - <GROUP> H <ACTIVE>  Browser Service Elections
- MAC Address = 00-00-00-00-00-00

 ==========================================
|    SMB Dialect Check on 10.129.14.128    |
 ==========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
SMB 1.0: false
SMB 2.02: true
SMB 2.1: true
SMB 3.0: true
SMB1 only: false
Preferred dialect: SMB 3.0
SMB signing required: false

 ==========================================
|    RPC Session Check on 10.129.14.128    |
 ==========================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user session
[+] Server allows session using username 'juzgtcsu', password ''
[H] Rerunning enumeration with user 'juzgtcsu' might give more results

 ====================================================
|    Domain Information via RPC for 10.129.14.128    |
 ====================================================
[+] Domain: DEVOPS
[+] SID: NULL SID
[+] Host is part of a workgroup (not a domain)

 ============================================================
|    Domain Information via SMB session for 10.129.14.128    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DEVSMB
NetBIOS domain name: ''
DNS domain: ''
FQDN: htb

 ================================================
|    OS Information via RPC for 10.129.14.128    |
 ================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 7, Windows Server 2008 R2
OS version: '6.1'
OS release: ''
OS build: '0'
Native OS: not supported
Native LAN manager: not supported
Platform id: '500'
Server type: '0x809a03'
Server type string: Wk Sv PrQ Unx NT SNT DEVSM

 ======================================
|    Users via RPC on 10.129.14.128    |
 ======================================
[*] Enumerating users via 'querydispinfo'
[+] Found 2 users via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 2 users via 'enumdomusers'
[+] After merging user results we have 2 users total:
'1000':
  username: mrb3n
  name: ''
  acb: '0x00000010'
  description: ''
'1001':
  username: cry0l1t3
  name: cry0l1t3
  acb: '0x00000014'
  description: ''

 =======================================
|    Groups via RPC on 10.129.14.128    |
 =======================================
[*] Enumerating local groups
[+] Found 0 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 0 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 0 group(s) via 'enumdomgroups'

 =======================================
|    Shares via RPC on 10.129.14.128    |
 =======================================
[*] Enumerating shares
[+] Found 5 share(s):
IPC$:
  comment: IPC Service (DEVSM)
  type: IPC
dev:
  comment: DEVenv
  type: Disk
home:
  comment: INFREIGHT Samba
  type: Disk
notes:
  comment: CheckIT
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
[*] Testing share IPC$
[-] Could not check share: STATUS_OBJECT_NAME_NOT_FOUND
[*] Testing share dev
[-] Share doesn't exist
[*] Testing share home
[+] Mapping: OK, Listing: OK
[*] Testing share notes
[+] Mapping: OK, Listing: OK
[*] Testing share print$
[+] Mapping: DENIED, Listing: N/A

 ==========================================
|    Policies via RPC for 10.129.14.128    |
 ==========================================
[*] Trying port 445/tcp
[+] Found policy:
domain_password_information:
  pw_history_length: None
  min_pw_length: 5
  min_pw_age: none
  max_pw_age: 49710 days 6 hours 21 minutes
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: false
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: None
domain_logoff_information:
  force_logoff_time: 49710 days 6 hours 21 minutes

 ==========================================
|    Printers via RPC for 10.129.14.128    |
 ==========================================
[+] No printers returned (this is not an error)

Completed after 0.61 seconds
```


> Importante: distintas herramientas pueden devolver resultados distintos (por implementación/edge cases). No conviene depender de una sola: usar más de 2 herramientas y corroborar manualmente lo crítico.

---

## 18) Checklist

1. **Detectar puertos**: `139/445` (y eventualmente 137/138 si hay NetBIOS).
2. **Nmap -sV -sC** para huella inicial.
3. **Listar shares** con `smbclient -N -L`.
4. **Conectarse** a shares interesantes y listar/descargar (`ls`, `get`).
5. **RPC enum** con `rpcclient` (dominio, shares, users).
6. **User enum** por `enumdomusers` o brute-force de RIDs.
7. Complementar con **CME/SMBMap/enum4linux‑ng/Impacket**.

---

---


## Preguntas


#### ¿Qué versión del servidor SMB se ejecuta en el sistema de destino? Envíe el banner completo como respuesta.

Enviamos un paquete `ICMP` de reconocimiento para verificar que el host esté activo:
<img width="1152" height="327" alt="image" src="https://github.com/user-attachments/assets/88990704-5aa6-40d4-9098-4ba38097c7b1" />

Hacemos rápidamente un `TCP SYN Scann` de los top 100 ports:
<img width="1268" height="468" alt="image" src="https://github.com/user-attachments/assets/2921e85c-a08c-4a83-94a0-2e4e51ac7fe4" />


Realizamos un escaneo de versiones y utilizamos `script=banner` sobre los puertos TCP 139 y 445, obviamos por ahora los cripts de reconocimiento `-sC`:
```bash
nmap -sV --script=banner -p139,445 10.129.4.97
```

<img width="1262" height="291" alt="image" src="https://github.com/user-attachments/assets/5b05c0af-04b9-44ea-a3e6-9c23f4dd1dc2" />

Obtenemos `Samba smbd 4`

#### ¿Cuál es el nombre del recurso compartido accesible en el destino?

Realizamos el siguiente comando para visualizar los scripts de nmap:
```bash
find / -type f -name smb* 2>/dev/null |grep scripts
```

<img width="923" height="852" alt="image" src="https://github.com/user-attachments/assets/aece8ad9-c82f-41f9-8a2b-5d17c2263725" />

Realizamos un escaneo con nmap utilizando el script `smb-enum-shares`:
```bash
nmap --script=smb-enum-shares -p139,445 10.129.4.97
```
<img width="1221" height="324" alt="image" src="https://github.com/user-attachments/assets/274cfa54-c884-4c33-abb7-e85c4b829d9f" />



Como no obtuvimos nada significativo al respecto con nmap, procedemos a intentar listar los shares con `smbclient` utilizando una null session:

```bash
smbclient -N -L //10.129.4.97
```
<img width="1189" height="223" alt="image" src="https://github.com/user-attachments/assets/37dc1a0f-5412-4342-a388-cc7e7c366aba" />

Obtenemos el nombre del recurso compartido en el destino.


#### Conéctese al recurso compartido detectado y busque el archivo flag.txt. Envíe el contenido como respuesta.

Nos conectamos al recurso con el comando:
```bash
smbclient //10.129.4.97/sambashare
```

Logramos conectarnos por lo que comenzamos a listar los recursos hasta que obtenemos la flag:

<img width="1174" height="488" alt="image" src="https://github.com/user-attachments/assets/b898b2ec-e6b9-49f7-80c8-8870ba60b4e3" />



#### Descubra a qué dominio pertenece el servidor.

`Pista`: Recuerde que podemos utilizar otros servicios para obtener información sobre acciones específicas.

Realizamos una conexión anónima, es decir una null session mediante `rpcclient`:
```bash
rpcclient -U "" 10.129.4.97
```

<img width="751" height="87" alt="image" src="https://github.com/user-attachments/assets/de2efa57-1536-4e84-b5ca-b55ad394c967" />

Realizamos los comandos `SRVINFO`, `enumdomains` y `querydominfo`:

<img width="990" height="539" alt="image" src="https://github.com/user-attachments/assets/43bd0080-7d49-4565-a489-1ea142471733" />


Y obtenemos el cominio del servidor.


#### Encuentre información adicional sobre el share que encontramos anteriormente y envíe la versión personalizada de este share como respuesta.

Listamos los shares con el comando `netshareenumall` y luego buscamos información específica con el comando `netsharegetinfo sambashare`:
<img width="1495" height="772" alt="image" src="https://github.com/user-attachments/assets/3f8ca6ab-3dc8-467c-b773-af5e8f1e35da" />

Obtenemos la versión personalizada: `InFreight SMB v3.1`


#### ¿Cuál es la ruta completa del sistema de ese recurso compartido específico? (formato: "/directorio/nombres")

`Pista`: Recuerde que los sistemas operativos basados ​​en Linux no tienen una unidad "C:\".

Utilizando el último comando obtenemos la respuesta:
<img width="1495" height="772" alt="image" src="https://github.com/user-attachments/assets/58877b5c-e2dd-4139-99a8-c4cb340fa8f9" />

Respuesta: `/home/sambauser`

---

Adicional:

En este ejercicio no se puede probar la enumeración de usuarios porque no tiene usuarios.

Procedemos a probar las herramientas automatizadas:

## samrdump

```bash
python3 -m venv impacket-env
source impacket-env/bin/activate
pip install impacket
```

```bash
samrdump.py 10.129.4.122
```

<img width="803" height="259" alt="image" src="https://github.com/user-attachments/assets/9ff58aae-b13a-4157-89db-6a7319088c14" />

---

## smbmap

```bash
apt update
apt install smbmap
smbmap -h
```

TODO

## crackmapexec

```bash
apt install -y libxml2-dev libxslt1-dev zlib1g-dev git python3-dev build-essential libssl-dev libffi-dev
python3 -m venv cme-env
source cme-env/bin/activate
git clone https://github.com/byt3bl33d3r/CrackMapExec.git
cd CrackMapExec
pip install .
crackmapexec -h
```

no funciona.

## enum4linux

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
python3 -m venv enum4linux-env
source enum4linux-env/bin/activate
pip3 install -r requirements.txt
```

<img width="901" height="791" alt="image" src="https://github.com/user-attachments/assets/ad39858f-8ddc-45b4-9b85-b562417a1c25" />

<img width="901" height="791" alt="image" src="https://github.com/user-attachments/assets/a9b7d3c7-2230-4b0d-8963-aa083bf16fe2" />

<img width="901" height="737" alt="image" src="https://github.com/user-attachments/assets/06301a61-81c2-4a15-a977-8c40a50131d7" />

<img width="901" height="696" alt="image" src="https://github.com/user-attachments/assets/5147a23e-10e8-4750-8d0f-b2ce43794737" />

<img width="901" height="696" alt="image" src="https://github.com/user-attachments/assets/f8ffd404-c0d1-40dd-a101-eeee717d94c3" />
