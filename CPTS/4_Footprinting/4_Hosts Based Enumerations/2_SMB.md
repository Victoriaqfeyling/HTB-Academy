# SMB y Samba (Footprinting / Enumeración)


---

## 1) ¿Qué es SMB?

**Server Message Block (SMB)** es un protocolo **cliente-servidor** que regula el acceso a:

* **Archivos** y **directorios** compartidos.
* **Recursos de red**: impresoras, routers, interfaces o servicios publicados.
* **Intercambio de información** entre procesos en red (según implementación).

Históricamente, SMB se popularizó en entornos Microsoft (p.ej., LAN Manager / LAN Server) y se volvió el estándar de facto en la familia **Windows**, manteniendo **compatibilidad hacia atrás** (equipos nuevos pueden comunicarse con versiones antiguas).

Para sistemas Linux/Unix existe el proyecto **Samba**, que habilita comunicación **cross‑platform** vía SMB.

---

## 2) Modelo de comunicación: cliente ↔ servidor

SMB permite que un cliente se comunique con otros participantes de la misma red para acceder a archivos o servicios compartidos. Para que esto funcione:

* El equipo remoto debe tener implementado el protocolo.
* Debe existir una aplicación servidor SMB que reciba, procese y responda solicitudes.

Antes de intercambiar datos, ambas partes **negocian y establecen la conexión** (intercambian mensajes de sesión/capabilities).

### SMB sobre TCP

En redes IP, SMB utiliza **TCP**, por lo que hereda el **three‑way handshake** (SYN → SYN/ACK → ACK). Luego, TCP gobierna la entrega ordenada de datos y retransmisiones.

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

**Samba** es una implementación alternativa de SMB para sistemas Unix‑like. Implementa **CIFS (Common Internet File System)**, que es un **dialecto** de SMB (una variante/implementación específica). Por eso muchas veces se ve como **SMB/CIFS**.

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

**NetBIOS** (Network Basic Input/Output System) es una API histórica para networking. En entornos NetBIOS:

* Cuando una máquina “aparece” en la red, necesita registrar un nombre.
* Esto puede hacerse:

  * Por registro directo del hostname (cada host reserva su nombre), o
  * Usando un **NBNS** (NetBIOS Name Server).
* En Windows, NBNS evolucionó en **WINS**.

---

## 7) Configuración por defecto de Samba (`smb.conf`)

Samba se configura mediante un archivo de texto, normalmente:

* `/etc/samba/smb.conf`

Ejemplo (filtrando comentarios):

```bash
cat /etc/samba/smb.conf | grep -v "#\|;"
```

### Ejemplo de estructura

* Sección **[global]**: aplica como base para todos los shares.
* Secciones **[share]**: definen recursos compartidos específicos.

Ejemplo simplificado (como en el contenido):

* `[global]` define `workgroup`, `server string`, logs, rol de servidor, PAM, mapeo a guest, etc.
* Shares por defecto relacionados con impresión:

  * `[printers]`
  * `[print$]`

> Importante: los settings globales pueden ser **sobrescritos** por share. Ahí aparecen muchas misconfigs típicas.

---

## 8) Opciones comunes en Samba (tabla)

| Setting                        | Descripción                                           |
| ------------------------------ | ----------------------------------------------------- |
| `[sharename]`                  | Nombre del recurso compartido                         |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup/Dominio visible a clientes                  |
| `path = /path/here/`           | Directorio del servidor que se comparte               |
| `server string = STRING`       | Texto/banner identificatorio del servidor             |
| `unix password sync = yes`     | Sincroniza password UNIX con password SMB             |
| `usershare allow guests = yes` | Permite acceso no autenticado a shares definidos      |
| `map to guest = bad user`      | Acción cuando el login no mapea a usuario UNIX válido |
| `browseable = yes`             | ¿El share aparece en listados?                        |
| `guest ok = yes`               | Permite conexión sin password                         |
| `read only = yes`              | Solo lectura                                          |
| `create mask = 0700`           | Permisos por defecto de archivos creados              |

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

Ejemplo de share con settings peligrosos (como en el contenido):

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

### Conectarse a un share

```bash
smbclient //<IP>/notes
```

Si permite acceso anónimo, puede decir “Anonymous login successful”.

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

Desde el punto de vista administrativo:

```bash
smbstatus
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

Escaneo típico:

```bash
sudo nmap <IP> -sV -sC -p139,445
```

Lo común es obtener:

* Servicio y versión (ej.: `Samba smbd`).
* Scripts host:

  * `nbstat` (nombres NetBIOS)
  * `smb2-security-mode` (signing habilitado/obligatorio)
  * `smb2-time` (hora del servidor)

> Limitación: a veces Nmap no devuelve demasiado detalle, por lo que conviene complementarlo con herramientas de interacción directa.

---

## 15) Enumeración MS‑RPC con `rpcclient`

### Concepto: RPC

**Remote Procedure Call (RPC)** permite invocar funciones remotas (pasando parámetros y recibiendo resultados). En SMB/Windows/Samba es central para enumeración.

### Conexión anónima (null session)

```bash
rpcclient -U "" <IP>
```

### Consultas útiles (resumen)

| Query                     | Descripción                       |
| ------------------------- | --------------------------------- |
| `srvinfo`                 | Información del servidor          |
| `enumdomains`             | Enumera dominios disponibles      |
| `querydominfo`            | Info de dominio/servidor/usuarios |
| `netshareenumall`         | Enumera todos los shares          |
| `netsharegetinfo <share>` | Detalle de un share               |
| `enumdomusers`            | Enumera usuarios                  |
| `queryuser <RID>`         | Info de un usuario por RID        |

### Qué se puede filtrar a anónimos

Con `rpcclient` puede filtrarse:

* Dominio/workgroup
* Shares y paths
* Usuarios (dependiendo de configuración)
* ACLs/SIDs en algunos casos

> Conclusión operativa: si existe acceso anónimo, un único error de permisos/visibilidad puede elevar mucho el riesgo.

---

## 16) Enumeración de usuarios por RIDs

Aunque algunos comandos estén restringidos, `queryuser <RID>` suele funcionar si el RID es válido. Por eso se puede:

* **Bruteforcear RIDs** y extraer usuarios/grupos.

Ejemplo (loop Bash, como en el contenido):

```bash
for i in $(seq 500 1100); do
  rpcclient -N -U "" <IP> -c "queryuser 0x$(printf '%x\n' $i)" \
  | grep "User Name\|user_rid\|group_rid" && echo "";
done
```

---

## 17) Alternativas: Impacket / SMBMap / CrackMapExec / enum4linux‑ng

### Impacket: `samrdump.py`

Permite enumerar usuarios y metadatos del dominio/workgroup.

### SMBMap

```bash
smbmap -H <IP>
```

Útil para ver shares y permisos según sesión.

### CrackMapExec (CME)

```bash
crackmapexec smb <IP> --shares -u '' -p ''
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

> Importante: distintas herramientas pueden devolver resultados distintos (por implementación/edge cases). No conviene depender de una sola: corroborar manualmente lo crítico.

---

## 18) Checklist rápido para tu writeup

1. **Detectar puertos**: `139/445` (y eventualmente 137/138 si hay NetBIOS).
2. **Nmap -sV -sC** para huella inicial.
3. **Listar shares** con `smbclient -N -L`.
4. **Conectarse** a shares interesantes y listar/descargar (`ls`, `get`).
5. **RPC enum** con `rpcclient` (dominio, shares, users).
6. **User enum** por `enumdomusers` o brute-force de RIDs.
7. Complementar con **CME/SMBMap/enum4linux‑ng/Impacket**.

---

## 19) Nota final de seguridad

El acceso anónimo a SMB (null session/guest) puede exponer:

* Shares y paths internos.
* Usuarios válidos (insumo para ataques de password spraying / brute force en escenarios permitidos).
* Configuración del host/dominio.

Una vez que un atacante obtiene visibilidad sobre recursos compartidos, un único ajuste mal hecho (p.ej., `guest ok = yes` + `writable = yes`) puede elevar el impacto de forma significativa.
