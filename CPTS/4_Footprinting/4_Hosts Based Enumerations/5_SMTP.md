# SMTP (Simple Mail Transfer Protocol)

## ¿Qué es SMTP?

SMTP (**Simple Mail Transfer Protocol**) es el protocolo estándar para **enviar correos electrónicos** a través de redes IP. Es importante entender que SMTP está orientado principalmente al **envío y el traspaso** (transferencia/relay) de mensajes:

* Puede usarse entre un **cliente de correo** (por ejemplo Thunderbird/Outlook/Gmail como interfaz) y el **servidor de salida**.
* También puede usarse **entre dos servidores SMTP** para ir “saltando” el mensaje hasta llegar al servidor final del destinatario.

Aunque conceptualmente se suele hablar de “cliente-servidor”, en el mundo real un **servidor SMTP puede actuar como cliente** cuando se conecta a otro servidor SMTP para entregarle un email.

En el ecosistema de correo, SMTP suele convivir con:

* **IMAP** y/o **POP3**, que se usan para **recibir/descargar** correos desde el buzón del usuario.
* SMTP por sí solo no sirve para “leer” el correo del mailbox; su función principal es **enviar**.

---

## Puertos típicos y cifrado (25, 587, 465)

Por defecto, los servidores SMTP escuchan en **TCP/25**. Sin embargo, es común ver:

* **TCP/587 (Submission)**: pensado para que usuarios/servidores **autenticados** envíen correo. Normalmente se inicia en texto plano y luego se ejecuta **STARTTLS** para “elevar” la conexión a **TLS** (cifrada). Esto protege credenciales y contenido durante el tránsito.
* **TCP/465 (SMTPS implícito)**: en algunos entornos se utiliza para SMTP **directamente cifrado** desde el inicio (TLS implícito), en lugar de STARTTLS.

### ¿Por qué importa esto en pentesting?

Porque SMTP **sin cifrado** transmite comandos, datos y potencialmente credenciales **en texto plano**. Si el servidor no fuerza TLS o permite AUTH sin STARTTLS, se abre la puerta a:

* **Captura de credenciales** en redes internas (sniffing).
* **Intercepción** del contenido del mensaje.
* Debilidad operativa: “se puede”, pero no significa que “deba” estar permitido.

---

## Flujo de envío: de tu cliente al servidor final

Cuando enviás un mail desde un cliente (MUA), el flujo típico simplificado es:

1. El cliente se conecta al servidor de envío.
2. Se autentica (si corresponde) y entrega el mensaje.
3. El servidor lo procesa y lo reenvía (MTA) hacia el servidor del destinatario.
4. El servidor de destino lo deposita en el buzón del usuario (MDA).
5. El usuario lo lee por POP3/IMAP.

En términos de roles:

* **MUA (Mail User Agent):** el cliente (Thunderbird, Outlook, etc.).
* **MSA (Mail Submission Agent):** recibe correo desde usuarios autenticados (a veces llamado *relay* o *submission* en este contexto). Valida origen/forma del envío.
* **MTA (Mail Transfer Agent):** el motor que “transporta” el correo entre servidores (Postfix/Sendmail/Exim, etc.). Verifica tamaño, filtros, colas, etc.
* **MDA (Mail Delivery Agent):** el que deposita el correo en el mailbox del usuario.

La cadena conceptual:

`Cliente (MUA) ➞ Submission Agent (MSA) ➞ Open Relay (MTA) ➞ Mail Delivery Agent (MDA) ➞ Mailbox (POP3/IMAP)`

> Nota: “Open Relay” en el diagrama representa un **escenario inseguro** (relay abierto), no algo que deba existir siempre.

---

## Problemas clásicos de SMTP (y por qué se abusa)

SMTP tiene dos desventajas típicas que se prestan al abuso:

### 1) Confirmación de entrega poco confiable

SMTP **no devuelve** una confirmación de entrega “usable” por defecto. Si falla, suele responder con:

* Un error genérico
* En general en inglés
* A veces con headers del mensaje devuelto

Esto complica el “seguimiento” si no hay mecanismos adicionales.

### 2) El remitente no es confiable (spoofing)

En SMTP, el remitente puede declararse con `MAIL FROM:` y eso **no prueba identidad** por sí solo. Históricamente esto habilitó:

* **Spoofing** de remitentes.
* Abuso de servidores mal configurados como **open relays** para enviar spam masivo.

Hoy se mitiga con técnicas como:

* [**SPF** (Sender Policy Framework)](https://dmarcian.com/what-is-spf/)
* [**DKIM** (DomainKeys Identified Mail)](https://dkim.org/)
* (y normalmente también DMARC, aunque este texto menciona SPF/DKIM)

---

## ESMTP y STARTTLS

Con el tiempo se extendió SMTP con **ESMTP (Extended SMTP)**. En la práctica, cuando la gente dice “SMTP” suele referirse a ESMTP.

ESMTP agrega capacidades extra (extensiones) y típicamente permite:

* Identificar extensiones tras `EHLO`.
* Iniciar **TLS** con `STARTTLS` (después de EHLO).
* Autenticación segura con extensiones como [**AUTH PLAIN**](https://www.samlogic.net/articles/smtp-commands-reference-auth.htm) (siempre que haya TLS, porque PLAIN en texto plano es un desastre).

---

# Configuración por defecto (ejemplo Postfix)

Los servidores SMTP pueden configurarse de muchas formas, pero una idea clave es que el servidor de correo se encarga de **enviar y reenviar** mensajes.

A continuación se muestra un ejemplo (Postfix) leyendo `main.cf` y filtrando comentarios/líneas vacías.

## Output completo: lectura de configuración

```txt
CyberWolfSec@htb[/htb]$ cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"

smtpd_banner = ESMTP Server
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail1.inlanefreight.htb
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtp_generic_maps = hash:/etc/postfix/generic
mydestination = $myhostname, localhost
masquerade_domains = $myhostname
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
recipient_delimiter = +
smtp_bind_address = 0.0.0.0
inet_protocols = ipv4
smtpd_helo_restrictions = reject_invalid_hostname
home_mailbox = /home/postfix
```

### ¿Qué nos dice esto como pentesters?

* `myhostname`: nombre del host SMTP (huella directa del entorno).
* `mynetworks`: rangos considerados “de confianza” para ciertas políticas (crítico para **open relay** si se configura mal).
* `smtpd_helo_restrictions`: políticas de validación al saludo HELO/EHLO.
* `smtpd_banner`: banner (huella/identificación expuesta).

---

# Comandos SMTP más importantes

SMTP es “conversacional”: el cliente envía comandos y el servidor responde con códigos.

| Comando      | Descripción                                                            |
| ------------ | ---------------------------------------------------------------------- |
| `AUTH PLAIN` | Extensión para autenticar al cliente (idealmente bajo TLS).            |
| `HELO`       | Saludo inicial con nombre del cliente (inicia sesión básica).          |
| `EHLO`       | Saludo ESMTP que además lista extensiones soportadas.                  |
| `MAIL FROM`  | Declara el remitente (envelope sender).                                |
| `RCPT TO`    | Declara el destinatario.                                               |
| `DATA`       | Inicia el envío del contenido del mensaje.                             |
| `RSET`       | Aborta la transacción actual sin cerrar la conexión.                   |
| `VRFY`       | Verifica si un usuario/mailbox existe (puede usarse para enumeración). |
| `EXPN`       | Expande listas/alias (según configuración).                            |
| `NOOP`       | Mantiene viva la sesión (evita timeout).                               |
| `QUIT`       | Cierra la sesión.                                                      |

---

# Interacción manual con Telnet (HELO/EHLO)

Para “hablar” SMTP de forma simple, se puede usar `telnet` (TCP crudo). La sesión se inicia con **HELO** o **EHLO**.

## Output completo: Telnet - HELO/EHLO

```txt
CyberWolfSec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

### Lectura del output

* `220`: el servidor está listo.
* `250`: OK.
* Tras `EHLO`, el servidor lista extensiones soportadas (pista para *capabilities* y vectores):

  * `PIPELINING`, `SIZE`, `VRFY` (si aparece), `DSN`, etc.

---

# Enumeración con VRFY (ojo con falsos positivos)

`VRFY` puede ayudar a enumerar usuarios. Sin embargo, **no siempre es confiable**:

* Dependiendo de la configuración, el servidor puede responder **252** (“no puedo verificar, pero acepto”) incluso para usuarios inexistentes.
* Por eso, nunca hay que confiar ciegamente en herramientas automáticas: ejecutan comandos estándar, pero el admin puede haber configurado respuestas “engañosas”.

[Códigos de respuesta SMTP](https://serversmtp.com/smtp-error/)

## Output completo: Telnet - VRFY

```txt
CyberWolfSec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server

VRFY root

252 2.0.0 root


VRFY cry0l1t3

252 2.0.0 cry0l1t3


VRFY testuser

252 2.0.0 testuser


VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

### ¿Qué significa este comportamiento?

Que el servidor está “devolviendo OK” (252) para cualquier cosa. Esto inutiliza `VRFY` como prueba definitiva de existencia.

---

## SMTP a través de un proxy

En algunos escenarios, el tráfico debe pasar por un proxy. Se puede forzar el proxy a abrir el túnel hacia SMTP con algo como:

`CONNECT 10.129.14.128:25 HTTP/1.0`

A partir de ahí, el tráfico SMTP viaja dentro del túnel.

---

# Enviar un email desde consola (DATA)

Desde la consola, el flujo de envío se parece mucho a lo que hace un cliente de correo:

1. `EHLO`
2. `MAIL FROM`
3. `RCPT TO`
4. `DATA` (headers + body)
5. Terminar con `.` en una línea sola
6. `QUIT`

## Output completo: Send an Email

```txt
CyberWolfSec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work.
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```

### Puntos clave del ejemplo

* El servidor indica cuándo pasar a modo contenido con `354`.
* El mensaje termina cuando enviamos una línea con un punto `.`.
* `queued as ...` confirma que el mail quedó en cola.

---

# Headers de email (por qué son oro en OSINT / forense)

El **header** de un email puede incluir muchísima información útil:

* Remitente y destinatario.
* Tiempos (creación, recepción).
* “Hops” o servidores por los que pasó (`Received:`).
* Información sobre cliente, formato, encoding, etc.

Parte de esa info es obligatoria y otra opcional, pero en general el header se transmite como parte del protocolo y **es accesible** por remitente y destinatario aunque no se muestre “a simple vista”.

La estructura está definida por [**RFC 5322**](https://datatracker.ietf.org/doc/html/rfc5322).

---

# Configuraciones peligrosas

## Relay (concepto) vs Open Relay (problema)

Para evitar filtros de spam y mejorar entregabilidad, a veces se usa un **relay server** confiable. En ese caso, el emisor normalmente debe **autenticarse**.

El problema aparece cuando, por mala configuración, el servidor permite que **cualquier IP** use el relay.

### Ejemplo de Open Relay (Postfix)

Si el admin no sabe qué rangos permitir, puede caer en el error de permitir “todo” para que “no se corte el correo”.

## Configuración peligrosa: Open Relay

```txt
mynetworks = 0.0.0.0/0
```

### Impacto

Con esto, el servidor podría:

* Reenviar correos de cualquiera (spam masivo).
* Ser usado como infraestructura de abuso, lo que termina en blocklists.
* Permitir ataques relacionados a spoofing/relay.

---

# Footprinting con Nmap (smtp-commands)

Nmap incluye scripts por defecto que ayudan a enumerar capacidades SMTP. El script `smtp-commands` usa `EHLO` para listar comandos/extensiones soportadas.

## Output completo: Nmap

```txt
CyberWolfSec@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING,
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

### Lectura del resultado

* Servicio detectado: **Postfix smtpd**.
* `smtp-commands` lista extensiones, incluida `VRFY` (si está habilitada).

---

# Detección de Open Relay con Nmap (smtp-open-relay)

Para probar si un SMTP se comporta como open relay, Nmap ofrece el script [**`smtp-open-relay`**](https://nmap.org/nsedoc/scripts/smtp-open-relay.html), que intenta múltiples variantes (16 tests en este ejemplo).

## Output completo: Nmap - Open Relay

```txt
CyberWolfSec@htb[/htb]$ sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-30 02:29 CEST
NSE: Loaded 1 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Initiating ARP Ping Scan at 02:29
Scanning 10.129.14.128 [1 port]
Completed ARP Ping Scan at 02:29, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 02:29
Completed Parallel DNS resolution of 1 host. at 02:29, 0.03s elapsed
Initiating SYN Stealth Scan at 02:29
Scanning 10.129.14.128 [1 port]
Discovered open port 25/tcp on 10.129.14.128
Completed SYN Stealth Scan at 02:29, 0.06s elapsed (1 total ports)
NSE: Script scanning 10.129.14.128.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.07s elapsed
Nmap scan report for 10.129.14.128
Host is up (0.00020s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@ESMTP> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest%nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org"@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@[10.129.14.128]:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@ESMTP:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@[10.129.14.128]>
|_ MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@ESMTP>
MAC Address: 00:00:00:00:00:00 (VMware)

NSE: Script Post-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds
           Raw packets sent: 2 (72B) | Rcvd: 2 (72B)
```

### Interpretación

* **“Server is an open relay (16/16 tests)”**: el servidor aceptó reenviar correo bajo todas las variantes probadas.
* Esto es un hallazgo serio (abuso para spam/phishing) y normalmente termina en **blocklists**.

---

## Checklist rápido para pentesting SMTP (práctico)

* Identificar puertos: 25/465/587.
* Banner y versión (`-sV`, `smtp-commands`).
* Extensiones tras `EHLO` (VRFY, AUTH, STARTTLS).
* Probar enumeración (VRFY/EXPN) sabiendo que puede mentir.
* Probar open relay (`smtp-open-relay`).
* Revisar si permite AUTH sin TLS (mala práctica).
* Documentar evidencias (transcripts telnet + outputs de nmap).


---


# Preguntas

#### Enumere el servicio SMTP y envíe el banner, incluyendo su versión como respuesta.

Enviamos una traza `ICMP` para verificar que el host está activo:

<img width="510" height="146" alt="image" src="https://github.com/user-attachments/assets/35dd205b-2a0c-475f-a081-dc9d720675d2" />

Realizamos un `TCP SYN Scann`:
```bash
nmap -Pn -n --reason -sS <ip>
```
<img width="883" height="386" alt="image" src="https://github.com/user-attachments/assets/3d0306e9-9af2-4163-bbd2-b8a386c4222b" />

Vemos el puerto 25 abierto correspondiente al servicio `SMTP`.

Realizamos banner grabbing de forma manual con netcat:

```bash
nc -nv <ip> 25
```
<img width="532" height="106" alt="image" src="https://github.com/user-attachments/assets/e6eab30b-d1e2-43e6-a731-ea0ae084035d" />

Obtenemos el banner: `InFreight ESMTP v2.11`

También nos conectamos al servicio con `telnet`:

<img width="367" height="215" alt="image" src="https://github.com/user-attachments/assets/dcd1c176-b937-489a-977f-104c70f48c1a" />


#### Enumere el servicio SMTP con más detalle y encuentre el nombre de usuario existente en el sistema. Envíelo como respuesta.


`Pista`: En los sistemas, los nombres de usuario suelen tener el mismo nombre que el empleado. Recomendamos usar la lista de palabras de Footprinting que se proporciona como recurso. Recuerde que algunos servidores SMTP tienen tiempos de respuesta más altos


Al conectarnos, sabemos que tenemos la extensión `VRFY` que podría servirnos para enumerar usuarios.

<img width="400" height="521" alt="image" src="https://github.com/user-attachments/assets/acc739d0-4d59-4748-8902-5532c6c3f5df" />



Nos descargamos la wordlist que nos dan en la academia llamada `footprinting-wordlist.txt`:
```
michael
james
john
robert
david
william
mary
christopher
joseph
richard
daniel
thomas
matthew
jennifer
charles
anthony
patricia
linda
mark
elizabeth
joshua
steven
andrew
kevin
brian
barbara
jessica
jason
susan
timothy
paul
kenneth
lisa
ryan
sarah
karen
jeffrey
donald
ashley
eric
jacob
nicholas
jonathan
ronald
michelle
kimberly
nancy
justin
sandra
amanda
brandon
stephanie
emily
melissa
gary
edward
stephen
scott
george
donna
jose
rebecca
deborah
laura
cynthia
carol
amy
margaret
gregory
sharon
larry
angela
maria
alexander
benjamin
nicole
kathleen
patrick
samantha
tyler
samuel
betty
brenda
pamela
aaron
kelly
robin
heather
rachel
adam
christine
zachary
debra
katherine
dennis
nathan
christina
julie
jordan
kyle
anna
```

Y utilizamos la herramienta `smb-user-enum` para realizar fuerza bruta mediante el método `VRFY` con la wordlist:

```bash
smtp-user-enum -M VRFY -U /home/juan/Descargas/footprinting-wordlist.txt -t 10.129.6.98 -m 1 -w 20
```

Cómo dice en la pista, hay que probar con distintos tiempos de respuesta para encontrar el usuario:

- `m`: 1 Worker Processes
- `w`: Query timeout: 20 s

<img width="1324" height="579" alt="image" src="https://github.com/user-attachments/assets/2d786c0b-2d23-4cd9-9af8-847468a53b2d" />



