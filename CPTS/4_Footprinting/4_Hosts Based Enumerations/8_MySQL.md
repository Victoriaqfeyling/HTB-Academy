# MySQL 

---

## 1. ¿Qué es MySQL?

**MySQL** es un **sistema de gestión de bases de datos relacional** (RDBMS) **open-source**, desarrollado y mantenido por **Oracle**.

Una **base de datos** es, en términos simples, una colección **estructurada** de datos, organizada para:

* Guardar información de forma ordenada
* Consultarla rápidamente
* Modificarla cuando sea necesario

En un RDBMS como MySQL, la información se almacena principalmente en **tablas**, que tienen:

* **Columnas** (campos / atributos)
* **Filas** (registros)
* **Tipos de dato** (texto, número, fecha, etc.)

MySQL está diseñado para manejar grandes volúmenes de datos de manera eficiente.

---

## 2. Modelo Cliente–Servidor

MySQL funciona con el modelo **cliente-servidor**:

* **MySQL Server**: el servicio que corre en el host y gestiona la base de datos.
* **MySQL Client(s)**: herramientas o programas que se conectan al servidor para ejecutar consultas.

### 2.1 ¿Qué hace el servidor?

El servidor MySQL:

* Escucha conexiones en un puerto TCP (normalmente 3306)
* Autentica usuarios
* Aplica permisos
* Ejecuta consultas SQL
* Devuelve resultados

### 2.2 ¿Qué hace un cliente?

Un cliente MySQL permite:

* Insertar datos
* Borrar datos
* Modificar datos
* Consultar datos

Todo esto se hace usando el lenguaje **SQL**.

---

## 3. Ejemplo práctico: WordPress

Un ejemplo clásico de uso de MySQL es **WordPress**.

WordPress guarda en una base de datos:

* Publicaciones
* Usuarios
* Contraseñas (en general hasheadas)
* Configuraciones

En instalaciones seguras, la base de datos solo debería ser accesible desde:

* **localhost**

Sin embargo, en infraestructuras distribuidas, puede haber:

* Servidor web en una máquina
* Base de datos en otra máquina

Y ahí se vuelve relevante la superficie de ataque: si MySQL queda accesible desde red interna o Internet.

---

## 4. ¿Dónde se guardan las bases?

Los datos suelen almacenarse en:

* Directorios internos (por ejemplo `/var/lib/mysql`)
* Archivos de respaldo/exportación con extensión `.sql`

Ejemplo mencionado:

* `wordpress.sql`

⚠️ Importante: un archivo `.sql` suele contener **SQL plano** (CREATE TABLE, INSERT, etc.). Si un atacante accede a un dump, puede recuperar datos sensibles.

---

## 5. LAMP / LEMP

MySQL es muy usado en sitios web dinámicos y suele verse en stacks como:

* **LAMP**: Linux + Apache + MySQL + PHP
* **LEMP**: Linux + Nginx + MySQL + PHP

En hosting web, MySQL actúa como el “repositorio central” de datos que consumen scripts PHP.

Ejemplos de datos típicos almacenados:

* Usuarios
* Emails
* Permisos
* Contraseñas
* Contenido de formularios
* Valores internos y configuraciones

Nota: MySQL puede almacenar contraseñas en texto plano, pero lo normal es que la app (PHP) las **hashee** antes (one-way encryption).

---

## 6. Comandos SQL y su impacto

Las consultas SQL son instrucciones que el motor ejecuta para:

* Mostrar datos (`SELECT`)
* Insertar (`INSERT`)
* Actualizar (`UPDATE`)
* Borrar (`DELETE`)
* Administrar estructura (`CREATE`, `ALTER`, `DROP`)
* Administrar usuarios y permisos

En aplicaciones web, un error SQL puede revelar información.

Esto es relevante porque:

* Errores SQL detallados confirman interacción con DB
* Mensajes de error pueden filtrar estructura, tablas, columnas
* Esto se explota con **SQL Injection**

---

## 7. MySQL vs MariaDB

**MariaDB** es un fork de MySQL.

Surgió cuando MySQL AB fue adquirida por Oracle y el desarrollador principal impulsó un proyecto abierto basado en el mismo código.

En muchos sistemas Linux, el comando `mysql` puede conectarte a un servidor MariaDB o MySQL sin que lo notes por fuera (hasta ver el banner).

---

# 8. Configuración por Defecto

La configuración de bases de datos es un tema enorme (existe el rol de **DBA** dedicado a esto).

La recomendación práctica es montar un laboratorio local:

* Instalar MySQL/MariaDB
* Probar configuraciones
* Ver cómo cambian comportamientos

---

## 8.1 Instalación y configuración por defecto (salida completa)

```bash
sudo apt install mysql-server -y
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

Salida:

```ini
[client]
port		= 3306
socket		= /var/run/mysqld/mysqld.sock

[mysqld_safe]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
skip-host-cache
skip-name-resolve
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
explicit_defaults_for_timestamp

symbolic-links=0

!includedir /etc/mysql/conf.d/
```

### 8.2 Lectura rápida de parámetros importantes

* **port = 3306**: puerto típico MySQL.
* **user = mysql**: usuario del sistema con el que corre el servicio.
* **datadir = /var/lib/mysql**: dónde se guardan los datos.
* **socket = ...mysqld.sock**: socket local (para conexiones locales sin TCP).
* **skip-name-resolve**: evita resolver DNS; usa IPs directamente (mejora performance / reduce dependencia de DNS).

---

# 9. Dangerous Settings (Configuraciones Peligrosas)

Hay muchas opciones que pueden quedar mal configuradas. Las más relevantes desde seguridad:

| Setting            | Descripción                                    | Riesgo principal                                                                                 |
| ------------------ | ---------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `user`             | Define con qué usuario corre el servicio MySQL | Si corre como un usuario incorrecto o con permisos excesivos, aumenta el impacto ante compromiso |
| `password`         | Password del usuario MySQL                     | Puede quedar en texto plano en archivos de config o scripts                                      |
| `admin_address`    | IP donde escucha conexiones administrativas    | Si expone una interfaz administrativa a redes no confiables, amplía superficie de ataque         |
| `debug`            | Configuración de debug activa                  | Puede mostrar información sensible (rutas, queries, estados)                                     |
| `sql_warnings`     | Controla warnings en inserts                   | Puede filtrar información extra ante errores                                                     |
| `secure_file_priv` | Limita import/export de archivos               | Si está mal configurado, puede permitir leer/escribir archivos desde el servidor                 |

### 9.1 Por qué `user`, `password`, `admin_address` son críticos

Porque muchas veces quedan **en texto plano**, y si el atacante logra:

* Leer archivos (LFI, path traversal)
* Obtener shell

podría recuperar credenciales del servidor DB.

Con credenciales válidas, un atacante puede:

* Ver bases completas
* Exfiltrar datos personales
* Modificar registros (impacto de integridad)

### 9.2 Debug y mensajes de error

`debug` y `sql_warnings` pueden ser útiles para el administrador, pero peligrosos si:

* Se exponen en una app web
* La app devuelve errores completos

Esto puede facilitar:

* Enumeración de tablas/columnas
* Confirmación de SQLi
* Escalada a técnicas más avanzadas (SQL Injection)

---

# 10. Footprinting del Servicio MySQL

Normalmente MySQL escucha en:

* **TCP/3306**

Exponer MySQL a redes externas **no es buena práctica**.

Aun así, es común encontrarlo expuesto porque:

* Configuración temporal olvidada
* Workarounds técnicos
* Errores de firewall

---

## 10.1 Escaneo con Nmap (salida completa)

```bash
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*
```

Salida:

```text
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 00:53 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute:
|   Accounts:
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds, average tps: 9002.0
|_mysql-databases: ERROR: Script execution failed (use -d to debug)
|_mysql-dump-hashes: ERROR: Script execution failed (use -d to debug)
| mysql-empty-password:
|_  root account has empty password
| mysql-enum:
|   Valid usernames:
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds, average tps: 10.0
| mysql-info:
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, LongPassword, DontAllowDatabaseTableColumn, Support41Auth, IgnoreSigpipes, SwitchToSSLAfterHandshake, FoundRows, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsCompression, ODBCClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YTSgMfqvx\x0F\x7F\x16\&\x1EAeK>0
|_  Auth Plugin Name: caching_sha2_password
|_mysql-users: ERROR: Script execution failed (use -d to debug)
|_mysql-variables: ERROR: Script execution failed (use -d to debug)
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.21 seconds
```

### 10.2 Nota importante sobre falsos positivos

Nmap y sus scripts pueden dar **falsos positivos**.

Este caso es un ejemplo: Nmap sugiere que `root` tiene password vacío, pero sabemos que **no es cierto**.

Por eso siempre hay que validar manualmente.

---

# 11. Interacción Manual con MySQL

## 11.1 Prueba sin contraseña (fallo esperado)

```bash
mysql -u root -h 10.129.14.132
```

Salida:

```text
ERROR 1045 (28000): Access denied for user 'root'@'10.129.14.1' (using password: NO)
```

Esto confirma:

* El servidor existe
* Requiere autenticación
* No acepta login sin password para root

---

## 11.2 Login con password válido (ejemplo)

```bash
mysql -u root -pP4SSw0rd -h 10.129.14.128
```

Salida:

```text
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)                                                         
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.                                     
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.                           
      
MySQL [(none)]> show databases;                                                                          
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.006 sec)


MySQL [(none)]> select version();
+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
1 row in set (0.001 sec)


MySQL [(none)]> use mysql;
MySQL [mysql]> show tables;
+------------------------------------------------------+
| Tables_in_mysql                                      |
+------------------------------------------------------+
| columns_priv                                         |
| component                                            |
| db                                                   |
| default_roles                                        |
| engine_cost                                          |
| func                                                 |
| general_log                                          |
| global_grants                                        |
| gtid_executed                                        |
| help_category                                        |
| help_keyword                                         |
| help_relation                                        |
| help_topic                                           |
| innodb_index_stats                                   |
| innodb_table_stats                                   |
| password_history                                     |
...SNIP...
| user                                                 |
+------------------------------------------------------+
37 rows in set (0.002 sec)
```

### 11.3 Qué significan estas bases

* `mysql`: base interna con usuarios, permisos y metadata.
* `sys`: schema con vistas y métricas para administración.
* `information_schema`: metadata estándar ANSI/ISO.
* `performance_schema`: métricas de performance.

---

# 12. Exploración del schema `sys` (salida completa)

```sql
use sys;
show tables;
```

Salida:

```text
+-----------------------------------------------+
| Tables_in_sys                                 |
+-----------------------------------------------+
| host_summary                                  |
| host_summary_by_file_io                       |
| host_summary_by_file_io_type                  |
| host_summary_by_stages                        |
| host_summary_by_statement_latency             |
| host_summary_by_statement_type                |
| innodb_buffer_stats_by_schema                 |
| innodb_buffer_stats_by_table                  |
| innodb_lock_waits                             |
| io_by_thread_by_latency                       |
...SNIP...
| x$waits_global_by_latency                     |
+-----------------------------------------------+
```

Luego:

```sql
select host, unique_users from host_summary;
```

Salida:

```text
+-------------+--------------+                   
| host        | unique_users |                   
+-------------+--------------+                   
| 10.129.14.1 |            1 |                   
| localhost   |            2 |                   
+-------------+--------------+                   
2 rows in set (0,01 sec)
```

---

# 13. Comandos esenciales para trabajar con MySQL

| Comando                                              | Descripción                                                              |
| ---------------------------------------------------- | ------------------------------------------------------------------------ |
| `mysql -u <user> -p<password> -h <IP address>`       | Conecta al servidor. **No debe haber espacio** entre `-p` y el password. |
| `show databases;`                                    | Lista todas las bases.                                                   |
| `use <database>;`                                    | Selecciona una base específica.                                          |
| `show tables;`                                       | Lista tablas de la base seleccionada.                                    |
| `show columns from <table>;`                         | Muestra columnas de una tabla.                                           |
| `select * from <table>;`                             | Muestra todos los registros.                                             |
| `select * from <table> where <column> = "<string>";` | Busca un valor específico en una columna.                                |

---

## 14. Conclusión

En footprinting de MySQL, lo importante es:

* Detectar exposición en red (3306)
* Identificar versión y plugin de auth
* No confiar ciegamente en scripts (validar manualmente)
* Entender qué bases son críticas (`mysql`, `sys`)
* Con credenciales, explorar de forma controlada

Para consolidar aprendizaje, conviene montar un lab propio (MySQL/MariaDB) y practicar:

* Usuarios y permisos
* Hardening
* Logs y configuraciones


