# FTP y TFTP (Footprinting / Enumeración)

---

## 1) FTP (File Transfer Protocol)

### ¿Qué es FTP?

[**FTP**](https://datatracker.ietf.org/doc/html/rfc959) es uno de los protocolos más antiguos de Internet para **transferencia de archivos**. Opera en la **capa de aplicación** del stack TCP/IP, al igual que **HTTP** o **POP**. Puede utilizarse mediante:

* Clientes dedicados (CLI/GUI).
* Herramientas integradas en sistemas operativos.
* Aplicaciones que lo consumen como “backend” para subir/bajar archivos.

### Canales en FTP: control vs datos

Una conexión FTP **abre dos canales**:

1. **Canal de control (TCP/21)**

* Se usa para **comandos** del cliente y **códigos de estado** del servidor.
* Es donde viajan acciones como `USER`, `PASS`, `LIST`, `RETR`, `STOR`, etc.

2. **Canal de datos (típicamente TCP/20 en modo activo)**

* Se usa **exclusivamente** para la **transferencia** (listados, descargas, subidas).
* El protocolo controla errores durante la transmisión.
* Si una transferencia se corta, suele poder **reanudarse** tras reestablecer contacto (según cliente/servidor/escenario).

> Idea clave para footprinting: **ver banner/códigos por 21** y confirmar **cómo se abre el canal de datos** (activo/pasivo) suele explicar fallos “raros” detrás de firewalls.

---

## 2) FTP Activo vs FTP Pasivo

### FTP Activo

* Cliente conecta al **control** (TCP/21).
* Cliente le indica al servidor **qué puerto del cliente** usar para el canal de datos.
* Problema típico: si el **cliente está detrás de firewall**, el servidor no puede “volver” por conexiones entrantes → la transferencia se rompe.

### FTP Pasivo (PASV)

* Se creó para evitar el problema anterior.
* El **servidor anuncia un puerto** y el **cliente inicia** la conexión del canal de datos.
* Como el cliente inicia el flujo, el firewall suele permitirlo.

> En práctica de pentest, si ves mensajes como “Consider using PASV”, probá cambiar a modo pasivo. En entornos reales, FTP detrás de NAT/firewall suele depender de PASV.

---

## 3) Comandos y códigos de estado

FTP define [**muchos comandos**](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/) y [**códigos de estado**](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes), pero **no todos** están implementados igual en todos los servidores.

* El cliente puede pedir: subir/bajar archivos, listar/crear/borrar directorios, renombrar, etc.
* El servidor responde con **status codes** (ej.: `220`, `230`, `150`, `226`, etc.) indicando si se ejecutó y el resultado.

**Ejemplos vistos en interacción:**

* `220` → banner / servicio listo.
* `230` → login exitoso.
* `150` → abriendo conexión de datos (listado/transferencia).
* `226` → transferencia/listado completado.

---

## 4) Credenciales, texto plano y Anonymous FTP

### Credenciales

Normalmente se requieren credenciales válidas para usar FTP.

### FTP es clear-text

FTP (clásico) es un protocolo **en texto plano**: usuario/contraseña y comandos pueden ser **sniffeables** si se dan condiciones de red que lo permitan.

### Anonymous FTP

Algunos servidores permiten **acceso anónimo**:

* Permite subir/bajar archivos **sin contraseña real**.
* Por riesgo, suele estar **limitado** (permisos/restricciones/chroot).

> Para footprinting, incluso si no podés descargar, el simple **listado (ls)** puede filtrar información útil (nombres de proyectos, clientes, plantillas, estructura interna, usuarios/roles, etc.).

---

## 5) TFTP (Trivial File Transfer Protocol)

### ¿Qué es TFTP?

**TFTP** es una versión más simple que FTP para transferencias.

* **No ofrece autenticación** de usuarios.
* Carece de funciones “avanzadas” típicas de FTP.
* Usa **UDP** (no TCP) → es **no confiable** y se apoya en recuperación a nivel aplicación.

### Implicancia práctica de seguridad

* No hay login con password.
* El acceso se controla básicamente con permisos de archivos/OS.
* En la práctica, opera en directorios/archivos que estén **globalmente compartidos** (lectura/escritura amplia).
* Por falta de seguridad, suele restringirse a **redes locales y protegidas**.

### Comandos típicos de TFTP

| Comando   | Descripción                                                     |
| --------- | --------------------------------------------------------------- |
| `connect` | Define host remoto (y opcionalmente puerto) para transferencias |
| `get`     | Descarga archivo(s) desde el remoto al local                    |
| `put`     | Sube archivo(s) del local al remoto                             |
| `quit`    | Sale del cliente                                                |
| `status`  | Muestra estado: modo (ascii/binary), conexión, timeouts, etc.   |
| `verbose` | Activa/desactiva salida detallada                               |

**Limitación importante:** a diferencia de FTP, TFTP **no** tiene funcionalidad de **listado de directorios**.

---

## 6) Configuración por defecto: vsFTPd

Uno de los servidores FTP más usados en Linux es [**vsFTPd**](https://security.appspot.com/vsftpd.html).

* Archivo principal: `/etc/vsftpd.conf`
* Se recomienda montarlo en una VM y revisar opciones reales.

### Instalación (Debian/Ubuntu)

```bash
sudo apt install vsftpd
```

> vsFTPd es “uno de varios” servidores FTP. Se usa mucho para demostrar configuraciones de forma simple. No todas las opciones están en el archivo por defecto: el resto está en el [**man page**](http://vsftpd.beasts.org/vsftpd_conf.html).

### Opciones típicas (ejemplo de `vsftpd.conf`)

> Ejemplo mostrado filtrando comentarios: `cat /etc/vsftpd.conf | grep -v "#"`

<img width="606" height="256" alt="image" src="https://github.com/user-attachments/assets/3fa1b54f-39f9-43f1-8740-3e2faa176455" />


| Setting                                   | Descripción                                  |
| ----------------------------------------- | -------------------------------------------- |
| `listen=NO`                               | ¿Corre desde inetd o como daemon standalone? |
| `listen_ipv6=YES`                         | ¿Escucha en IPv6?                            |
| `anonymous_enable=NO`                     | ¿Habilita acceso anónimo?                    |
| `local_enable=YES`                        | ¿Permite login de usuarios locales?          |
| `dirmessage_enable=YES`                   | ¿Muestra mensajes al entrar a directorios?   |
| `use_localtime=YES`                       | ¿Usa hora local?                             |
| `xferlog_enable=YES`                      | ¿Loguea subidas/descargas?                   |
| `connect_from_port_20=YES`                | ¿Conexión de datos desde puerto 20 (activo)? |
| `secure_chroot_dir=/var/run/vsftpd/empty` | Directorio vacío para chroot seguro          |
| `pam_service_name=vsftpd`                 | Servicio PAM usado por vsftpd                |
| `rsa_cert_file=...`                       | Ubicación del cert RSA (SSL/TLS)             |
| `rsa_private_key_file=...`                | Ubicación de la private key                  |
| `ssl_enable=NO`                           | ¿Habilita SSL/TLS?                           |

### Archivo a vigilar: `/etc/ftpusers`

Este archivo se usa para **negar acceso** a usuarios específicos.
Ejemplo:

<img width="696" height="286" alt="image" src="https://github.com/user-attachments/assets/e6c32437-f7df-4952-9411-bb28ce0d7e85" />


Aunque existan como usuarios del sistema, **no podrán** loguearse por FTP si están listados ahí.

---

## 7) Settings peligrosos / misconfig típicas

### Anonymous login habilitado + escritura

En vsFTPd, habilitar anónimo y permitir escritura puede ser crítico.
[Opciones](http://vsftpd.beasts.org/vsftpd_conf.html) típicas (cuando existen):

| Setting                        | Descripción                                                                                     |
| ------------------------------ | ----------------------------------------------------------------------------------------------- |
| `anonymous_enable=YES`         | Permite login anónimo                                                                           |
| `anon_upload_enable=YES`       | Permite subir archivos como anónimo                                                             |
| `anon_mkdir_write_enable=YES`  | Permite crear directorios como anónimo                                                          |
| `no_anon_password=YES`         | No pide password a anónimo                                                                      |
| `anon_root=/home/username/ftp` | Directorio root para anónimo                                                                    |
| `write_enable=YES`             | Habilita comandos de escritura: `STOR`, `DELE`, `RNFR/RNTO`, `MKD`, `RMD`, `APPE`, `SITE`, etc. |

**Riesgo típico:**

* Acceso a información sensible.
* Subida de archivos “maliciosos” o de pivote (ej.: webshell si hay sincronización con webroot).
* Abuso de logs/paths para ejecución remota en escenarios concretos.

> Ojo: a veces en infra interna se habilita para “compartir rápido”, pero si esa interfaz termina expuesta o puenteada, se vuelve un punto de entrada.

---

## 8) Interacción práctica con `ftp` (Anonymous)

### Login anónimo

```bash
ftp <IP>
```

Ejemplo de flujo:

* El servidor responde con `220` y un **banner**.
* Ingresás `anonymous` como usuario.
* Si permite, responde `230 Login successful`.

### Enumeración inicial (listado)

Dentro del prompt `ftp>`:

```ftp
ls
```

<img width="696" height="268" alt="image" src="https://github.com/user-attachments/assets/c68a68ce-032a-4f70-814b-2a349a2dd3e6" />

El listado puede mostrar:

* Archivos sueltos (`.pptx`, `.txt`, etc.).
* Directorios por área (Clients, Documents, Employees…).
* Permisos (útil para inferir si hay escritura habilitada).

```
Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.


ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

### Ver estado (overview del cliente/sesión)

```ftp
status
```


Sirve para ver:

* Modo actual (binario/ascii).
* Si usa comandos PORT.
* Flags de cliente.


```
ftp> status

Connected to 10.129.14.136.
No proxy connection.
Connecting using address family: any.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Case: off; CR stripping: on
Quote control characters: on
Ntrans: off
Nmap: off
Hash mark printing: off; Use of PORT cmds: on
Tick counter printing: off
```

### Más detalle: `debug` y `trace`

```ftp
debug
trace
```

Luego, por ejemplo:

```ftp
ls
```

Vas a ver los comandos reales enviados (ej.: `PORT`, `LIST`) y respuestas del servidor.


```
ftp> debug

Debugging on (debug=1).


ftp> trace

Packet tracing on.


ftp> ls

---> PORT 10,10,14,4,188,195
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```



---

`Nota`: Otras configuraciones:

| Setting               | Descripción                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| dirmessage_enable=YES | ¿Mostrar un mensaje cuando el usuario ingresa por primera vez a un directorio nuevo? |
| chown_uploads=YES     | ¿Cambiar la propiedad de los archivos cargados anónimamente?               |
| chown_username=username | Usuario al que se le asignará la propiedad de los archivos subidos de forma anónima. |
| local_enable=YES      | ¿Permitir que los usuarios locales del sistema inicien sesión vía FTP?     |
| chroot_local_user=YES | ¿Restringir (chroot) a los usuarios locales a su directorio de inicio?     |
| chroot_list_enable=YES | ¿Usar una lista de usuarios locales que serán forzados a permanecer en su directorio (chroot)? |

## 9) Settings útiles que impactan enumeración

### `hide_ids=YES`

Si está activo, el listado muestra user/group como `ftp` en lugar de UID/GID reales.

* **Pro:** evita filtrar usuarios locales.
* **Contra para el atacante:** perdés señales sobre qué usuarios/grupos reales están asociados a archivos.

```
ftp> ls

---> TYPE A
200 Switching to ASCII mode.
ftp: setsockopt (ignored): Permission denied
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

### `ls_recurse_enable=YES`

Permite listados recursivos:

```ftp
ls -R
```

Útil para ver rápidamente:

* Estructura completa.
* Subdirectorios relevantes.
* Nombres de archivos con valor (contratos, plantillas, notas internas, etc.).

```
ftp> ls -R

---> PORT 10,10,14,4,222,149
200 PORT command successful. Consider using PASV.
---> LIST -R
150 Here comes the directory listing.
.:
-rw-rw-r--    1 ftp      ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp      ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp      ftp            0 Sep 15 14:57 testupload.txt

./Clients:
drwx------    2 ftp      ftp          4096 Sep 16 18:04 HackTheBox
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:00 Inlanefreight

./Clients/HackTheBox:
-rw-r--r--    1 ftp      ftp         34872 Sep 16 18:04 appointments.xlsx
-rw-r--r--    1 ftp      ftp        498123 Sep 16 18:04 contract.docx
-rw-r--r--    1 ftp      ftp        478237 Sep 16 18:04 contract.pdf
-rw-r--r--    1 ftp      ftp           348 Sep 16 18:04 meetings.txt

./Clients/Inlanefreight:
-rw-r--r--    1 ftp      ftp         14211 Sep 16 18:00 appointments.xlsx
-rw-r--r--    1 ftp      ftp         37882 Sep 16 17:58 contract.docx
-rw-r--r--    1 ftp      ftp            89 Sep 16 17:58 meetings.txt
-rw-r--r--    1 ftp      ftp        483293 Sep 16 17:59 proposal.pptx

./Documents:
-rw-r--r--    1 ftp      ftp         23211 Sep 16 18:05 appointments-template.xlsx
-rw-r--r--    1 ftp      ftp         32521 Sep 16 18:05 contract-template.docx
-rw-r--r--    1 ftp      ftp        453312 Sep 16 18:05 contract-template.pdf

./Employees:
226 Directory send OK.
```
---

## 10) Transferencias: descargar y subir

### Impacto de Descarga y Subida de Archivos en FTP

La descarga y subida de archivos es la funcionalidad principal de un servidor FTP. Sin embargo, desde la perspectiva de un pentester, estas capacidades pueden tener implicancias de seguridad mucho más profundas.

### 1. Descarga de archivos (Information Disclosure)

Cuando el servidor permite listar y descargar archivos:

- Podemos acceder a documentos internos (contratos, plantillas, notas, backups, etc.).
- Es posible encontrar credenciales almacenadas en texto plano.
- Pueden aparecer configuraciones sensibles, archivos `.env`, scripts o respaldos.
- La estructura de directorios puede revelar información organizacional (clientes, empleados, proyectos).

Incluso si no se permite la descarga, el simple listado de archivos ya constituye una fuga de información valiosa.

---

### 2. Subida de archivos (File Upload Abuse)

Si el servidor permite la subida de archivos (`STOR`) y además:

- El FTP está vinculado a un servidor web.
- Existe sincronización automática con el webroot.
- O hay vulnerabilidades como LFI (Local File Inclusion).

Entonces puede producirse un escenario de explotación más grave.

#### Escenario típico con LFI:

1. Se sube un archivo malicioso (por ejemplo, un script PHP).
2. Se utiliza una vulnerabilidad LFI para incluir ese archivo.
3. El servidor interpreta y ejecuta el código.
4. Esto puede derivar en ejecución remota de comandos (RCE).

---

### 3. Abuso de logs de FTP

Otra técnica interesante consiste en aprovechar los logs del servidor FTP.

En ciertos escenarios:

- El servidor registra comandos o nombres de usuario en archivos de log.
- Si existe una vulnerabilidad LFI en la aplicación web.
- Y el atacante logra inyectar código dentro del log (por ejemplo, como nombre de usuario).

Entonces:

1. Se fuerza al servidor a escribir código malicioso en el archivo de log.
2. Se utiliza la vulnerabilidad LFI para incluir el log.
3. El código se ejecuta en el contexto del servidor web.

Esto puede conducir a **Remote Command Execution (RCE)**.

---

### 4. Importancia durante la fase de Enumeración

Durante la fase de enumeración debemos:

- Verificar permisos de lectura y escritura.
- Identificar si el FTP está expuesto a Internet.
- Determinar si permite acceso anónimo.
- Analizar si está integrado con otros servicios (como un servidor web).

El FTP no debe analizarse de forma aislada.  
Debe evaluarse en conjunto con todos los servicios detectados, ya que su combinación con vulnerabilidades como LFI puede escalar el impacto significativamente.

---

## Conclusión

Un servidor FTP mal configurado puede pasar de ser un simple repositorio de archivos a convertirse en un vector de:

- Fuga de información.
- Escalada de privilegios.
- Ejecución remota de comandos (RCE).

Por ello, cualquier capacidad de lectura o escritura detectada durante el footprinting debe ser evaluada cuidadosamente dentro del contexto completo de la infraestructura.

### Descargar un archivo (RETR)

Ejemplo con espacios escapados:

```ftp
get Important\ Notes.txt
```

Resultado típico:

* `150 Opening BINARY mode data connection ...`
* `226 Transfer complete.`

```
ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxrwx    1 ftp      ftp             0 Sep 16 17:24 Calendar.pptx
drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees
-rwxrwxrwx    1 ftp      ftp            41 Sep 18 15:58 Important Notes.txt
226 Directory send OK.


ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)


ftp> exit

221 Goodbye.
```




### Descargar “todo” con `wget` (mirror)

Para obtener todo lo accesible (útil en jerarquías grandes, pero ruidoso):

```bash
wget -m --no-passive ftp://anonymous:anonymous@<IP>
```

Notas:

* `-m` crea un “mirror” y guarda estructura.
* Suele crear un directorio con nombre del host/IP destino.
* **Riesgo operacional:** puede disparar alertas por volumen/behavior.

```
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/                                         
           => ‘10.129.14.136/.listing’                                                                     
Connecting to 10.129.14.136:21... connected.                                                               
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.                                                                 
12.12.1.136/.listing           [ <=>                                  ]     466  --.-KB/s    in 0s       
                                                                                                         
2021-09-19 14:45:58 (65,8 MB/s) - ‘10.129.14.136/.listing’ saved [466]                                     
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/Calendar.pptx   
           => ‘10.129.14.136/Calendar.pptx’                                       
==> CWD not required.                                                           
==> SIZE Calendar.pptx ... done.                                                                                                                            
==> PORT ... done.    ==> RETR Calendar.pptx ... done.       

...SNIP...

2021-09-19 14:45:58 (48,3 MB/s) - ‘10.129.14.136/Employees/.listing’ saved [119]

FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```



Una vez descargados todos los archivos, wgetse creará un directorio con la dirección IP de nuestro objetivo. Todos los archivos descargados se almacenan allí, y luego podemos inspeccionarlos localmente.

```
tree .

.
└── 10.129.14.136
    ├── Calendar.pptx
    ├── Clients
    │   └── Inlanefreight
    │       ├── appointments.xlsx
    │       ├── contract.docx
    │       ├── meetings.txt
    │       └── proposal.pptx
    ├── Documents
    │   ├── appointments-template.xlsx
    │   ├── contract-template.docx
    │   └── contract-template.pdf
    ├── Employees
    └── Important Notes.txt

5 directories, 9 files
```

### Subir un archivo (STOR)

## Verificación de Permisos de Subida en FTP

Luego de identificar un servidor FTP, es importante comprobar si tenemos permisos para subir archivos. En muchos entornos, especialmente en servidores web, el FTP se utiliza como mecanismo de sincronización para que los desarrolladores suban rápidamente contenido al servidor.

Con frecuencia, los administradores asumen que estos servicios no son accesibles desde el exterior, lo que lleva a descuidar su hardening. Esta falsa sensación de seguridad puede provocar configuraciones incorrectas, como permisos excesivos o autenticación débil.

Si el servidor FTP permite la subida de archivos y está conectado a un servidor web, el riesgo aumenta considerablemente. En ese caso, un atacante podría subir un archivo malicioso (por ejemplo, una webshell) y ejecutarlo desde el navegador, obteniendo ejecución remota de comandos (RCE).

Además, esto podría permitir el establecimiento de una reverse shell, facilitando la ejecución de comandos internos y potencialmente la escalada de privilegios.

Por ello, durante la fase de enumeración, no solo debemos identificar el servicio FTP, sino también evaluar cuidadosamente los permisos de escritura y su integración con otros servicios.

Crear archivo local:

```bash
touch testupload.txt
```

Luego en `ftp>`:

```ftp
put testupload.txt
```

```
ftp> put testupload.txt 

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.


ftp> ls

---> TYPE A
200 Switching to ASCII mode.
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

Si se permite escritura, verás `150 Ok to send data` y `226 Transfer complete`.

**Por qué importa en pentest:**

* Si FTP está vinculado a un servidor web o a un proceso de sincronización, una subida puede convertirse en **ejecución** (ej.: acceso directo a webshell) y derivar en **reverse shell** / ejecución de comandos.

---

## 11) Footprinting con Nmap + NSE


El uso de escáneres de red es una técnica práctica y ampliamente utilizada durante la fase de footprinting. Estas herramientas permiten identificar servicios activos en un host, incluso cuando no se encuentran expuestos en sus puertos estándar.

Una de las herramientas más utilizadas para este propósito es **Nmap**. Además de detectar puertos abiertos y versiones de servicios, Nmap incorpora el [**Nmap Scripting Engine (NSE)**](https://nmap.org/book/nse.html), un conjunto de scripts diseñados para interactuar con servicios específicos y obtener información adicional.

Estos scripts pueden realizar tareas como:

- Identificar configuraciones inseguras.
- Detectar acceso anónimo en servicios como FTP.
- Extraer información del sistema remoto.
- Verificar vulnerabilidades conocidas.

La base de datos de scripts NSE puede actualizarse fácilmente mediante el comando correspondiente, asegurando que se utilicen las últimas versiones disponibles durante la enumeración.

### Actualizar base de scripts NSE

```bash
sudo nmap --script-updatedb
```

### Encontrar scripts FTP en el sistema

```bash
find / -type f -name ftp* 2>/dev/null | grep scripts
```

Ejemplos típicos:

* `ftp-anon.nse` (chequea anónimo + lista contenido)
* `ftp-syst.nse` (usa `STAT`/`SYST` para info del server)
* `ftp-brute.nse` (fuerza bruta — usar sólo si está permitido)
* Scripts específicos de vulnerabilidades/backdoors (dependen de versiones/targets).

### Escaneo recomendado (servicio estándar 21)

```bash
sudo nmap -sV -p21 -sC -A <IP>
```

Qué aporta:

* `-sV`: versión del servicio.
* `-sC`: scripts por defecto (incluye checks útiles como `ftp-anon`, `ftp-syst` si aplica).
* `-A`: agresivo (detecciones extra).

```
sudo nmap -sV -p21 -sC -A 10.129.14.136

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

**Interpretación típica de salida:**

* Confirma `21/tcp open ftp`.
* Indica servidor (ej.: `vsftpd`).
* [`ftp-anon`](https://nmap.org/nsedoc/scripts/ftp-anon.html) puede listar contenido y marcar si hay **writeable**.
* [`ftp-syst`](https://nmap.org/nsedoc/scripts/ftp-syst.html) puede devolver status del server (incluye si control/datos son texto plano).

### Traza de scripts (qué comandos viajan)

```bash
sudo nmap -sV -p21 -sC -A <IP> --script-trace
```

Permite ver:

* Conexiones paralelas.
* Puertos origen efímeros.
* Banner recibido.
* Comandos enviados por NSE.


```
nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:54 CEST                                                                                                                                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.14.136:21]                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [10.129.14.136:21]             
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 24 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 32 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #1 [10.129.14.136:21] (timeout: 7000ms) EID 42
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #2 [10.129.14.136:21] (timeout: 9000ms) EID 50
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #3 [10.129.14.136:21] (timeout: 7000ms) EID 58
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #4 [10.129.14.136:21] (timeout: 11000ms) EID 66
NSE: TCP 10.10.14.4:54226 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54228 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54230 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54232 > 10.129.14.136:21 | CONNECT
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 58 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

## Análisis del Script Trace en Nmap

El historial del escaneo muestra que se están ejecutando múltiples escaneos en paralelo contra el servicio, cada uno con distintos tiempos de espera (timeouts). Esto es normal cuando Nmap utiliza scripts NSE, ya que puede abrir varias conexiones simultáneamente para obtener información más rápidamente.

En el caso observado, la máquina local utiliza puertos efímeros distintos (por ejemplo, 54226, 54228, 54230, 54232) para iniciar conexiones hacia el puerto 21 del servidor FTP. Cada script NSE establece su propia conexión mediante el comando `CONNECT`.

A partir de la primera respuesta del servidor, podemos identificar el banner FTP, lo que confirma que el servicio está activo y responde correctamente a las solicitudes enviadas por los scripts.

Si es necesario realizar pruebas manuales o validar comportamientos específicos, también es posible interactuar directamente con el servicio utilizando herramientas como `netcat` o `telnet`, lo que permite observar las respuestas del servidor sin intermediarios.


---

## 12) Interacción manual (netcat / telnet / TLS)

### Conectar a FTP por texto plano

```bash
nc -nv <IP> 21
```

```bash
telnet <IP> 21
```

Esto sirve para:

* Ver banner.
* Probar comandos manuales.
* Diagnosticar comportamientos de red.

### FTP con TLS/SSL (STARTTLS)

Cuando hay cifrado, necesitás un cliente que lo soporte. Ejemplo con OpenSSL:

```bash
openssl s_client -connect <IP>:21 -starttls ftp
```


**Valor del certificado en footprinting:**

* Puede revelar **hostname (CN/SAN)**.
* Puede incluir **email** de contacto.
* Puede indicar **organización/unidad** y **ubicación** (si el cert está bien rellenado).
* Si es self-signed, igual puede dar pistas internas.


```
CONNECTED(00000003)                                                                                      
Can't use SSL_get_servername                        
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1

depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify return:1
---                                                 
Certificate chain
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
 
 i:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
---
 
Server certificate

-----BEGIN CERTIFICATE-----

MIIENTCCAx2gAwIBAgIUD+SlFZAWzX5yLs2q3ZcfdsRQqMYwDQYJKoZIhvcNAQEL
...SNIP...
```

---

## 13) Checklist rápido

1. **Descubrir servicio**

   * `nmap -sV -p21 -sC -A <IP>`

2. **Revisar banner + soporte anónimo**

   * `ftp <IP>` → mirar `220` y probar `anonymous`

3. **Enumerar estructura**

   * `ls` / `ls -R` (si habilitado)
   * Tomar nota de nombres de archivos/directorios

4. **Validar permisos**

   * Intentar `get <archivo>`
   * Intentar `put <archivo>` (si está permitido y dentro de reglas del engagement)

5. **Profundizar**

   * `status`, `debug`, `trace`
   * `--script-trace` en Nmap si querés ver tráfico

6. **Si hay TLS**

   * `openssl s_client -starttls ftp` y extraer pistas del cert

---


## Preguntas


#### ¿Qué versión del servidor FTP se ejecuta en el sistema de destino? Envíe el banner completo como respuesta

Realizamos banner grabbing con cualquiera de los dos comandos:

```bash
nc -nv <ip> 21
```
```bash
telnet <ip> 21
<generar error con un caracter>
quit
```

<img width="486" height="227" alt="image" src="https://github.com/user-attachments/assets/e55b9581-fc83-4290-adbf-c5e767617f37" />

Obtenemos el banner:
```
InFreight FTP v1.1
```

#### Enumere el servidor FTP y busque el archivo flag.txt. Envíe su contenido como respuesta.

Lanzamos el script `ftp-anon` y observamos que es posible la conexión como `anonymous` y lista el archivo `flag.txt`:

<img width="667" height="223" alt="image" src="https://github.com/user-attachments/assets/e3b01ac1-df1b-43ed-8602-b3c4b49a2e12" />

Nos logueamos como `anonymous`:
<img width="753" height="288" alt="image" src="https://github.com/user-attachments/assets/87464e2d-dff5-45d0-99e5-870bb3fa6fcd" />

Nos descargamos el archivo:
<img width="741" height="215" alt="image" src="https://github.com/user-attachments/assets/8ba349b3-69b7-47ef-995d-c154fa637a72" />

