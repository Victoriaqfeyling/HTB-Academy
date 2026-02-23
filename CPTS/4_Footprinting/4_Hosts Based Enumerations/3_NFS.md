# NFS (Network File System) – Guía Completa de Enumeración y Explotación

---

# 1. Introducción a NFS

**Network File System (NFS)** es un sistema de archivos en red desarrollado por Sun Microsystems cuyo objetivo es permitir el acceso a sistemas de archivos remotos como si fueran locales.

A diferencia de SMB (usado principalmente en entornos Windows), NFS se utiliza principalmente entre sistemas **Linux y Unix**.

⚠️ Un cliente NFS no puede comunicarse directamente con un servidor SMB.

[NFS](https://en.wikipedia.org/wiki/Network_File_System) es un estándar de Internet que define los procedimientos de un sistema de archivos distribuido.

---

# 2. Versiones de NFS

| Versión                | Características                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------- |
| **NFSv2**              | Versión antigua, soportada por muchos sistemas. Operaba completamente sobre UDP.                               |
| **NFSv3**              | Soporta tamaños de archivo variables y mejor reporte de errores. No totalmente compatible con NFSv2.           |
| **NFSv4**              | Incluye Kerberos, funciona a través de firewalls, usa puerto 2049, soporta ACLs, es stateful, mayor seguridad. |
| **NFSv4.1 ([RFC 8881](https://datatracker.ietf.org/doc/html/rfc8881))** | Soporta despliegues en clúster, acceso paralelo (pNFS), multipathing (session trunking).                       |

### Diferencia crítica entre v3 y v4

* **NFSv3** autentica el equipo cliente.
* **NFSv4** requiere autenticación del usuario (similar a SMB moderno).

Una ventaja importante de NFSv4 es que utiliza únicamente el puerto:

```
2049/TCP o UDP
```

Lo que facilita su paso a través de firewalls.

---

# 3. Arquitectura Interna

NFS se basa en:

* [**ONC-RPC (Open Network Computing Remote Procedure Call)**](https://en.wikipedia.org/wiki/Sun_RPC)
* También conocido como SUN-RPC
* Usa representación de datos independiente del sistema ([XDR](https://en.wikipedia.org/wiki/External_Data_Representation))

Puertos importantes:

| Puerto | Servicio             |
| ------ | -------------------- |
| 111    | rpcbind / portmapper |
| 2049   | NFS                  |

⚠️ NFS por sí mismo no implementa autenticación ni autorización.

La autenticación depende de RPC.
La autorización depende de la información del sistema de archivos.

Generalmente se usa autenticación basada en:

* UID
* GID
* Membresías de grupo

### Problema importante

El cliente y el servidor pueden tener mapeos diferentes de UID/GID. El servidor no realiza validaciones adicionales.

En NFS (especialmente en versiones antiguas como NFSv2 y NFSv3), el servidor confía en los UID/GID que el cliente le envía durante las operaciones. Esto significa que la autorización se basa únicamente en esos identificadores numéricos y no en una validación fuerte de identidad.

Si en el sistema cliente existe un usuario con un UID específico (por ejemplo, UID 0 correspondiente a root) y el servidor no aplica mecanismos como `root_squash`, este podría interpretar esa identidad como legítima. Además, si los mapeos de UID/GID no coinciden entre ambos sistemas, pueden producirse accesos no deseados o comportamientos inesperados en los permisos.

Por eso NFS solo debería utilizarse en redes confiables o combinarse con mecanismos de autenticación más robustos como Kerberos en NFSv4.

---

# 4. Configuración por Defecto

El archivo principal de configuración es:

```
/etc/exports
```

[Manual](https://manpages.ubuntu.com/manpages/trusty/man5/exports.5.html)

Contenido por defecto:

```bash
CyberWolfSec@htb[/htb]$ cat /etc/exports

# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

Formato general:

```
/directorio  host(opciones)
```

---

# 5. Opciones Comunes

| Opción | Descripción |
|--------|------------|
| **rw** | Permite lectura y escritura sobre el recurso exportado. Los clientes pueden crear, modificar y eliminar archivos dentro del share NFS. |
| **ro** | Permite únicamente acceso de lectura. Los clientes pueden listar y visualizar archivos, pero no modificarlos ni eliminarlos. |
| **sync** | Obliga al servidor a confirmar la escritura en disco antes de responder al cliente. Es más seguro e íntegro, pero puede afectar el rendimiento. |
| **async** | Permite que el servidor responda antes de que los datos se escriban físicamente en disco. Mejora el rendimiento, pero puede implicar riesgo de pérdida de datos ante fallos. |
| **secure** | Solo permite conexiones desde puertos privilegiados (<1024). Tradicionalmente estos puertos solo pueden ser utilizados por procesos ejecutados como root, lo que agrega una capa básica de control. |
| **insecure** | Permite conexiones desde puertos superiores a 1024. Esto puede ser riesgoso, ya que usuarios no privilegiados podrían iniciar conexiones al servicio NFS. |
| **no_subtree_check** | Desactiva la verificación de subdirectorios cuando se exporta un subdirectorio específico. Mejora el rendimiento y evita problemas de validación, pero reduce ciertos controles de consistencia. |
| **root_squash** | Mapea al usuario root remoto (UID 0) a un usuario anónimo (generalmente `nobody`). Evita que root desde el cliente tenga privilegios de root en el servidor. Es una medida de seguridad clave. |

---

# 6. Creación de un Export

Ejemplo práctico:

```bash
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server
root@nfs:~# exportfs

/mnt/nfs      	10.129.14.0/24
```

Se comparte `/mnt/nfs` a toda la red `10.129.14.0/24`.

---

# 7. Configuraciones Peligrosas

| Opción | Riesgo |
|--------|--------|
| **rw** | Permite lectura y escritura sobre el recurso exportado. Si el acceso no está correctamente restringido por IP o red, un atacante podría modificar, eliminar o subir archivos maliciosos dentro del share. |
| **insecure** | Permite conexiones desde puertos mayores a 1024. Como estos puertos pueden ser utilizados por usuarios no privilegiados, un atacante podría interactuar con el servicio NFS sin necesidad de privilegios elevados en su sistema local. |
| **nohide** | Si existe otro sistema de archivos montado dentro de un directorio exportado, esta opción permite que también sea visible automáticamente. Esto puede exponer recursos adicionales que el administrador no tenía intención de publicar. |
| **no_root_squash** | Desactiva la protección que convierte al root remoto en usuario anónimo. Si está habilitado, el usuario root desde el cliente mantendrá UID 0 en el servidor, lo que puede permitir la creación de archivos SUID o la modificación de archivos críticos, facilitando escaladas de privilegios. |

⚠️ `no_root_squash` es extremadamente peligroso.

Permite que root remoto cree archivos con UID 0 reales.

---

# 8. Footprinting del Servicio

Puertos clave:

* 111 (rpcbind)
* 2049 (nfs)

---

## Escaneo básico con Nmap

```bash
CyberWolfSec@htb[/htb]$ sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:12 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00018s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      45837/tcp   mountd
|   100021  1,3,4      44629/tcp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)
```

Esto revela:

* Versiones RPC
* Servicios asociados (mountd, nlockmgr, nfs_acl)

---

## Uso de Scripts NSE para NFS

```bash
CyberWolfSec@htb[/htb]$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| nfs-showmount:
|_  /mnt/nfs 10.129.14.0/24
```

Se puede:

* Listar archivos
* Ver permisos
* Identificar claves privadas

---

# 9. Enumeración con showmount

```bash
CyberWolfSec@htb[/htb]$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

---

# 10. Montaje Manual del Share

Una vez que hayamos detectado dicho servicio NFS, podemos montarlo en nuestra máquina local. Para ello, podemos crear una nueva carpeta vacía donde se montará el recurso compartido NFS. Una vez montado, podemos navegar por ella y ver su contenido como en nuestro sistema local.

```bash
CyberWolfSec@htb[/htb]$ mkdir target-NFS
CyberWolfSec@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
CyberWolfSec@htb[/htb]$ cd target-NFS
CyberWolfSec@htb[/htb]$ tree .

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

---

# 11. Análisis de Permisos

Al montar el recurso NFS podemos ver los permisos, propietarios y grupos asociados a cada archivo.  
Si identificamos los nombres de usuario, grupos, UID y GID, podemos replicarlos en nuestro sistema local para que coincidan con los del servidor.  

De esta forma, si la configuración lo permite, podremos acceder o modificar archivos respetando los mismos identificadores que espera el servidor NFS.

## Con nombres

```bash
CyberWolfSec@htb[/htb]$ ls -l mnt/nfs/

-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
```

## Con UID/GID

```bash
CyberWolfSec@htb[/htb]$ ls -n mnt/nfs/

-rw-r--r-- 1 0 0 1872 Sep 19 17:27 id_rsa
```

Si `root_squash` está activo, root no podrá modificar ciertos archivos.

---

# 12. Escalada de Privilegios vía NFS

NFS también puede utilizarse como vector de escalada de privilegios.  
Si tenemos acceso por SSH y existe un recurso NFS mal configurado (por ejemplo, con `no_root_squash`), podemos subir un binario al share y asignarle el bit SUID correspondiente a un usuario privilegiado.  

Luego, al ejecutar ese binario desde el sistema comprometido, heredaremos los permisos del propietario, lo que puede permitir acceso a archivos restringidos o incluso privilegios elevados.

Una vez finalizado el análisis o la explotación, es recomendable desmontar el recurso NFS para limpiar el entorno:

```bash
sudo umount ./target-NFS

Escenario típico:

1. Existe `no_root_squash`
2. Podemos montar el share
3. Creamos binario SUID como root
4. Ejecutamos desde el sistema remoto

Ejemplo conceptual:

```bash
cp /bin/bash ./bash-root
chmod +s bash-root
```

Si el servidor respeta UID 0 remoto → obtenemos shell root.

---

# 13. Desmontar el Share

```bash
CyberWolfSec@htb[/htb]$ cd ..
CyberWolfSec@htb[/htb]$ sudo umount ./target-NFS
```

---

# 14. Conclusiones Técnicas

NFS es:

* Extremadamente potente
* Simple de configurar
* Muy peligroso si se configura mal

Puntos críticos en pentesting:

✔ Enumerar puertos 111 y 2049
✔ Usar scripts nfs* de Nmap
✔ Revisar exportaciones
✔ Analizar UID/GID
✔ Verificar root_squash
✔ Buscar claves privadas
✔ Evaluar no_root_squash

NFS mal configurado puede equivaler a acceso root remoto sin autenticación real.

---



## Preguntas


#### Enumere el servicio NFS y envíe el contenido de flag.txt en el recurso compartido "nfs" como respuesta.

Enviamos una traza `ICMP` al host para verificar que esté activo:
<img width="836" height="211" alt="image" src="https://github.com/user-attachments/assets/acccb1db-05cd-4b1a-a1c5-eca7c980613a" />

Realizamos un escaneo `TCP SYN` al host para descubrir sus puertos abiertos:
```bash
nmap -Pn -n --reason -sS <ip>
```
<img width="841" height="331" alt="image" src="https://github.com/user-attachments/assets/f07531d2-982d-443f-ab8f-571daa6e668a" />

Vemos que el servidor tiene los puertos 111 y 2049 que son los correspondientes al servicio `NFS`.

Realizamos un escaneo con nmap de versiones con los scripts NSE correspondientes a `NFS`:
```bash
find / -type f -name "nfs*" 2>/dev/null  |grep scripts
nmap --script=nfs* -sV -p111,2049 10.129.5.154
```

<img width="1170" height="496" alt="image" src="https://github.com/user-attachments/assets/d18c349b-950e-43c6-941e-fc06980f079a" />

Enumeramos manualmente con `showmount`:
```bash
showmount -e <ip>
```

Obtenemos los shares:
<img width="421" height="132" alt="image" src="https://github.com/user-attachments/assets/e9b3c9c6-969f-45e4-ab09-3c5ebecb8579" />

Creamos la carpeta para el montaje:
```bash
mkdir /tmp/target-NFS
```

Montamos en la carpeta:
```bash
mount -t nfs 10.129.5.154:/ /tmp/target-NFS/ -o nolock
```

<img width="805" height="728" alt="image" src="https://github.com/user-attachments/assets/6470a7b3-5185-4b22-82c7-4e0871fe3f17" />

Leemos las flags:
```bash
find . -type f -name "flag.txt" | xargs cat
```
<img width="753" height="128" alt="image" src="https://github.com/user-attachments/assets/55f6cd7c-49c8-4bf8-891a-ede36e27e4b3" />



#### Enumere el servicio NFS y envíe el contenido de flag.txt en el recurso compartido "nfsshare" como respuesta.

Se resolvió en el ejercicio anterior
