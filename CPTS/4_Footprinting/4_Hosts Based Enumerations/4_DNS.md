# DNS 

---

# 1. Introducción a DNS

El **Domain Name System (DNS)** es un componente esencial de Internet. Permite traducir nombres de dominio (por ejemplo, `www.hackthebox.com`) en direcciones IP que los sistemas pueden utilizar para comunicarse.

DNS no posee una base de datos centralizada. La información está distribuida en miles de servidores alrededor del mundo.

Podemos imaginarlo como una biblioteca global con múltiples guías telefónicas distribuidas.

---

# 2. Tipos de Servidores DNS

| Tipo de Servidor | Descripción |
|------------------|------------|
| **DNS Root Server** | Son la capa más alta de la jerarquía DNS. No resuelven directamente dominios completos, sino que indican qué servidores son responsables de cada Top-Level Domain (TLD) como `.com`, `.net` o `.org`. Existen 13 conjuntos de root servers a nivel mundial, coordinados por [ICANN](https://www.icann.org/), distribuidos globalmente para garantizar redundancia y disponibilidad. |
| **Authoritative Nameserver** | Tiene autoridad sobre una zona específica (por ejemplo, `empresa.com`). Contiene los registros oficiales (A, MX, NS, etc.) y sus respuestas son definitivas. Si este servidor responde, la información se considera válida y confiable para esa zona. |
| **Non-authoritative Nameserver** | No posee la zona original, sino que responde basándose en información obtenida previamente de otros servidores mediante consultas recursivas o iterativas. Sus respuestas no son la fuente original, pero pueden ser correctas si están actualizadas. |
| **Caching DNS Server** | Almacena temporalmente respuestas DNS obtenidas de otros servidores para acelerar futuras consultas. El tiempo que conserva esa información está determinado por el valor TTL (Time To Live) definido en los registros. Reduce latencia y carga en servidores autoritativos. |
| **Forwarding Server** | No resuelve consultas directamente, sino que reenvía todas las peticiones DNS a otro servidor específico (por ejemplo, el DNS del ISP o un servidor interno). Se utiliza comúnmente en redes corporativas para centralizar el control del tráfico DNS. |
| **Resolver** | Es el componente que inicia la consulta DNS desde el sistema del usuario o router. Puede ser parte del sistema operativo o un servicio local. Su función es contactar a los servidores DNS adecuados hasta obtener la respuesta final que permita traducir un dominio en una dirección IP. |


<img width="948" height="605" alt="image" src="https://github.com/user-attachments/assets/7493524a-ea23-48e6-a9ff-6162849bdf58" />

---

# 3. Seguridad en DNS


---

## 1. DNS y la Falta de Cifrado

El protocolo DNS tradicional fue diseñado en una época en la que la seguridad y la privacidad no eran prioridades centrales en Internet. Por defecto, las consultas DNS viajan en texto plano a través de la red (normalmente por el puerto 53 UDP/TCP).

Esto implica que:

* Cualquier dispositivo dentro de la misma red local (WLAN) puede interceptar las consultas.
* El proveedor de Internet (ISP) puede registrar y analizar todos los dominios que visitamos.
* Un atacante con capacidad de "Man-in-the-Middle" puede espiar o incluso manipular respuestas DNS.

En otras palabras, aunque el contenido de una página web esté protegido con HTTPS, la consulta DNS previa puede revelar qué dominio estamos intentando visitar.

---

## 2. Riesgos Asociados

La falta de cifrado en DNS puede provocar:

* Pérdida de privacidad (historial de navegación expuesto).
* Ataques de DNS Spoofing o DNS Poisoning.
* Redirección maliciosa hacia servidores controlados por atacantes.
* Enumeración pasiva de infraestructura interna en redes corporativas.

Por esta razón, el DNS tradicional representa un punto crítico desde el punto de vista de seguridad.

---

## 3. Mecanismos Modernos de Cifrado DNS

Para mitigar estos riesgos, se desarrollaron soluciones que encapsulan o cifran las consultas DNS:

### DNS over TLS (DoT)

* Utiliza el protocolo TLS para cifrar el tráfico DNS.
* Opera normalmente sobre el puerto 853.
* Establece un canal seguro entre el cliente y el servidor DNS.
* Ofrece confidencialidad y protección contra manipulación.

### DNS over HTTPS (DoH)

* Envía consultas DNS a través de HTTPS.
* Utiliza el puerto 443.
* Se mezcla con el tráfico web normal, lo que dificulta su bloqueo o inspección.
* Es ampliamente utilizado por navegadores modernos.

### DNSCrypt

* Protocolo independiente que cifra y autentica el tráfico DNS.
* Protege contra ataques de tipo "Man-in-the-Middle".
* No depende directamente de TLS, aunque cumple un propósito similar.

Estas tecnologías mejoran significativamente la privacidad del usuario y reducen la posibilidad de manipulación del tráfico DNS.

---

## 4. DNS Como Fuente de Información Estratégica

DNS no solo traduce nombres de dominio a direcciones IP. También almacena información clave sobre la infraestructura asociada a un dominio.

Una simple consulta DNS puede revelar:

* Qué servidores gestionan el correo electrónico (registros MX).
* Cuáles son los nameservers responsables (registros NS).
* Qué subdominios existen (registros A, AAAA, CNAME).
* Información de validación y políticas de correo (registros TXT como SPF o DMARC).

Desde la perspectiva de pentesting y reconocimiento (recon), DNS es una fuente extremadamente valiosa de información pasiva.

Por ejemplo:

* Identificar el servidor de correo puede revelar proveedores externos.
* Los nameservers pueden indicar si la infraestructura está alojada en AWS, Azure u otro proveedor.
* Los registros TXT pueden exponer integraciones con servicios externos.

En consecuencia, DNS no es solo un sistema de resolución de nombres, sino también un mapa indirecto de la arquitectura tecnológica de una organización.





---

# 4. Jerarquía de Dominio

Estructura jerárquica como marca la imagen:

```
Root (.)
 ├── TLD (.com, .net, .org, .io, etc.)
 │     └── Segundo Nivel (inlanefreight.com)
 │            ├── www.inlanefreight.com
 │            ├── dev.inlanefreight.com
 │            └── mail.inlanefreight.com
 │                   └── WS01.dev.inlanefreight.com
```

---

# 5. Tipos de Registros DNS

| Registro | Descripción                                       |
| -------- | ------------------------------------------------- |
| A        | Devuelve dirección IPv4.                          |
| AAAA     | Devuelve dirección IPv6.                          |
| MX       | Servidores de correo responsables.                |
| NS       | Nameservers del dominio.                          |
| TXT      | Información adicional (SPF, DMARC, validaciones). |
| CNAME    | Alias hacia otro dominio.                         |
| PTR      | Resolución inversa (IP → dominio).                |
| SOA      | Información administrativa de la zona.            |

---

# 6. Consulta SOA

```bash
CyberWolfSec@htb[/htb]$ dig soa www.inlanefreight.com

; <<>> DiG 9.16.27-Debian <<>> soa www.inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15876
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; AUTHORITY SECTION:
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
```

El correo administrativo se interpreta como:

```
awsdns-hostmaster@amazon.com
```

---



# 7. DNS – Configuración por Defecto en BIND9

---

# 1. Introducción

Los servidores DNS pueden configurarse de múltiples formas dependiendo del entorno (corporativo, ISP, laboratorio, nube, etc.). Sin embargo, la mayoría de implementaciones en sistemas Linux utilizan **[BIND9 (Berkeley Internet Name Domain)](https://www.isc.org/bind/)** como servidor DNS.

Desde el punto de vista administrativo, entender la configuración por defecto es fundamental para:

* Comprender cómo se resuelven las consultas.
* Identificar posibles malas configuraciones.
* Detectar vulnerabilidades explotables (como zone transfers no restringidos).

Todos los servidores DNS trabajan principalmente con **tres tipos de archivos de configuración**:

1. Archivos de configuración local.
2. Archivos de zona (zone files).
3. Archivos de resolución inversa.

---

# 2. Archivos de Configuración Local

En BIND9, el archivo principal es:

```
/etc/bind/named.conf
```

Este archivo generalmente incluye otros archivos secundarios que organizan la configuración:

* `named.conf.local`
* `named.conf.options`
* `named.conf.log`

## División lógica de named.conf

La configuración se divide en dos grandes bloques:

### 1️⃣ Opciones Globales

Afectan el comportamiento general del servidor DNS.

Ejemplos típicos:

* Control de recursión.
* Interfaces donde escucha el servidor.
* Permisos de consulta.
* Configuración de logging.

Estas opciones impactan a todas las zonas configuradas.

---

### 2️⃣ Declaraciones de Zona

Definen qué dominios administra el servidor.

Cada zona especifica:

* Tipo (master, slave, forward).
* Archivo donde se almacenan los registros.
* Restricciones de transferencia.

Si una opción se define tanto globalmente como dentro de una zona, **la configuración específica de la zona tiene prioridad**.

---

# 3. Ejemplo de Configuración Local

```bash
root@bind9:~# cat /etc/bind/named.conf.local

//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

---

# 4. Análisis Detallado del Ejemplo

### zone "domain.com"

Define que el servidor es responsable de la zona `domain.com`.

---

### type master

Indica que este servidor es el **servidor primario (master)** para esa zona.

* Aquí se realizan las modificaciones.
* Es la fuente original de los datos.
* Los servidores secundarios (slaves) sincronizan desde este.

---

### file "/etc/bind/db.domain.com"

Especifica el archivo donde están almacenados los registros DNS de esa zona.

Este archivo contiene:

* SOA
* NS
* A
* MX
* CNAME
* TXT

Es el "libro telefónico" del dominio.

---

### allow-update { key rndc-key; };

Permite actualizaciones dinámicas autenticadas mediante una clave (`rndc-key`).

Esto es importante en entornos donde:

* Servidores DHCP actualizan automáticamente registros DNS.
* Se utilizan entornos Active Directory.

Si esta directiva está mal configurada, podría permitir modificaciones no autorizadas en la zona.

---

# 5. Relación con los RFC

BIND9 implementa estándares definidos en múltiples RFC relacionados con DNS.

La configuración permite adaptar el servidor a:

* Arquitecturas distribuidas.
* Balanceo de carga.
* Alta disponibilidad.
* Delegación de subzonas.

---



# DNS – Zone Files, Reverse Lookup y Configuraciones Peligrosas

---

# 1. ¿Qué es un Zone File?

Un **zone file** es un archivo de texto que describe completamente una zona DNS utilizando el formato estándar de [**BIND**](https://wiki.debian.org/BIND9?action=show&redirect=Bind9).

En términos simples, una zona es un punto de delegación dentro del árbol jerárquico DNS. Es decir, representa la porción del espacio de nombres sobre la cual un servidor tiene autoridad.

El formato BIND es el estándar de facto en la industria y es utilizado por la mayoría de los servidores DNS.

Un zone file debe contener obligatoriamente:

* Exactamente **un registro SOA (Start of Authority)**.
* Al menos **un registro NS (Name Server)**.

El registro SOA generalmente aparece al comienzo del archivo y define información crítica sobre la administración y sincronización de la zona.

⚠️ Un error de sintaxis en un zone file puede provocar que toda la zona sea considerada inválida. En ese caso, el servidor DNS responderá con errores `SERVFAIL`, como si la zona no existiera.

---

# 2. Zone File – Resolución Directa (Forward Lookup)

Aquí se definen todos los registros directos:

```
Dominio / Hostname → Dirección IP
```

En términos prácticos, este archivo es el "directorio telefónico" que el servidor consulta para saber qué IP corresponde a cada dominio.

## Ejemplo Completo

```bash
root@bind9:~# cat /etc/bind/db.domain.com

;
; BIND forward data file
;
$ORIGIN domain.com
$TTL 86400

@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh (6h)
                    3600       ; retry (1h)
                    604800     ; expire (1w)
                    86400 )    ; minimum TTL (1d)

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

      IN     MX     10     mx.domain.com.
      IN     MX     20     mx2.domain.com.

             IN     A       10.129.14.5

server1      IN     A       10.129.14.5
server2      IN     A       10.129.14.7
ns1          IN     A       10.129.14.2
ns2          IN     A       10.129.14.3

ftp          IN     CNAME   server1
mx           IN     CNAME   server1
mx2          IN     CNAME   server2
www          IN     CNAME   server2
```

---

# 3. Análisis del Registro SOA

El SOA contiene parámetros clave:

* **Serial**: número de versión de la zona. Si aumenta, los servidores secundarios sincronizan.
* **Refresh**: cada cuánto el slave consulta al master.
* **Retry**: tiempo de reintento si falla la consulta.
* **Expire**: cuándo el slave deja de confiar en los datos.
* **Minimum TTL**: tiempo mínimo de cacheo.

Este registro controla el mecanismo de replicación entre servidores master y slave.

---

# 4. Reverse Lookup Zone (Resolución Inversa)

Para que una IP pueda resolverse a un FQDN (Fully Qualified Domain Name), se necesita un archivo de resolución inversa.

Aquí la relación es:

```
Dirección IP → Dominio (PTR record)
```

## Ejemplo Completo

```bash
root@bind9:~# cat /etc/bind/db.10.129.14

;
; BIND reverse data file
;
$ORIGIN 14.129.10.in-addr.arpa
$TTL 86400

@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501
                    21600
                    3600
                    604800
                    86400 )

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

5    IN     PTR    server1.domain.com.
7    IN     PTR    server2.domain.com.
```

En este caso:

* 10.129.14.5 → server1.domain.com
* 10.129.14.7 → server2.domain.com

Los registros PTR son fundamentales para:

* Validaciones de correo electrónico.
* Análisis forense.
* Identificación de infraestructura.

---



# 5 DNS – Configuraciones Peligrosas en BIND 

---

# 1. Introducción

Los servidores DNS, especialmente cuando utilizan **BIND9**, pueden volverse altamente vulnerables si ciertas directivas no están correctamente restringidas.

DNS es un servicio crítico. Un error de configuración no solo puede provocar fallos operativos, sino también:

* Exposición masiva de infraestructura.
* Participación involuntaria en ataques DDoS.
* Filtración de información interna.
* Superficie ampliada para ataques dirigidos.

Muchas vulnerabilidades en entornos reales no provienen de fallos del software, sino de **malas configuraciones**.

- [Lista de vulnerabilidades dirigidas al servidor BIND9](https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64)
- [SecurityTrails ataques más comunes](https://web.archive.org/web/20250329174745/https://securitytrails.com/blog/most-popular-types-dns-attacks)

---

# 2. Por qué ocurren las malas configuraciones

DNS puede volverse complejo rápidamente:

* Múltiples zonas.
* Servidores master y slave.
* Integración con DHCP.
* Entornos híbridos (on-prem + cloud).
* Necesidades de recursión interna.

Cuando surgen problemas operativos, los administradores muchas veces "abren" configuraciones para que el sistema funcione, priorizando disponibilidad sobre seguridad.

Esto suele derivar en directivas permisivas como:

* `allow-transfer { any; };`
* `allow-recursion { any; };`
* `allow-query { any; };`

Lo que transforma un servidor DNS interno en un objetivo expuesto públicamente.

---

# 3. Directivas Críticas y Sus Riesgos

## allow-query

Define qué hosts pueden enviar consultas al servidor.

### Riesgo

Si está configurado como:

```
allow-query { any; };
```

Cualquier persona en Internet puede:

* Enumerar registros.
* Consultar subdominios.
* Extraer información estratégica.

Aunque parezca inofensivo, combinado con otras configuraciones débiles puede facilitar reconocimiento masivo.

---

## allow-recursion

Define qué hosts pueden realizar consultas recursivas.

### ¿Qué es recursión?

El servidor no solo responde por sus zonas autoritativas, sino que consulta otros servidores en nombre del cliente.

### Riesgo

Si está habilitado públicamente:

```
allow-recursion { any; };
```

El servidor se convierte en un **open resolver**.

Consecuencias:

* Puede ser utilizado en ataques de amplificación DNS (DDoS).
* Puede ser abusado para ocultar origen real de consultas maliciosas.
* Genera alto consumo de recursos.

Los ataques de amplificación DNS explotan la diferencia entre el tamaño de la consulta y el tamaño de la respuesta.

---

## allow-transfer

Controla qué hosts pueden realizar transferencias de zona (AXFR).

### Riesgo Crítico

Si está configurado como:

```
allow-transfer { any; };
```

Un atacante puede ejecutar:

```
dig axfr dominio.com @servidor_dns
```

Y obtener:

* Todos los subdominios.
* Hosts internos.
* IP privadas.
* Servidores de correo.
* Controladores de dominio.

Esto equivale a entregar el mapa completo de la infraestructura.

En términos de pentesting, una transferencia de zona exitosa puede reducir drásticamente el tiempo de reconocimiento.

---

## zone-statistics

Recopila estadísticas de las zonas configuradas.

### Riesgo

Puede revelar:

* Estructura de consultas.
* Volumen de tráfico.
* Información indirecta sobre uso interno.

Aunque no es tan crítico como allow-transfer, puede facilitar análisis avanzados de comportamiento.

---

# 4. Riesgo Operativo Real

En entornos productivos es común encontrar:

* Servidores DNS internos accesibles desde Internet.
* Recursión habilitada globalmente.
* Transferencias permitidas a subredes completas.
* Actualizaciones dinámicas mal autenticadas.

Cuando la funcionalidad tiene prioridad sobre la seguridad, pueden generarse:

* Filtraciones de infraestructura interna.
* Enumeración completa de subdominios.
* Exposición de IP privadas.
* Identificación de sistemas críticos (VPN, DC, mail).
* Participación involuntaria en ataques DDoS.

---

# 5. Impacto desde el Punto de Vista Ofensivo

Durante una auditoría o pentest, estas configuraciones permiten:

* Reconocimiento pasivo altamente efectivo.
* Descubrimiento de activos no documentados.
* Identificación de entornos internos separados.
* Mapeo completo de la arquitectura tecnológica.

En muchos casos, una mala configuración DNS expone más información que un escaneo activo de red.





---


# DNS Footprinting – Análisis Técnico Detallado y Enumeración Avanzada

---

# 1. Introducción al Footprinting DNS

El footprinting en DNS consiste en extraer la mayor cantidad de información posible únicamente mediante consultas DNS legítimas. A diferencia de un escaneo de red tradicional, aquí no estamos explotando vulnerabilidades directamente, sino aprovechando cómo el servicio responde a nuestras solicitudes.

DNS es una de las fuentes más ricas de información en la fase de reconocimiento.

A través de consultas específicas podemos descubrir:

* Nameservers
* Versión del servidor
* Registros TXT sensibles
* Servidores de correo
* Subdominios
* Infraestructura interna
* Posibles configuraciones inseguras

---

# 2. Enumeración de Nameservers (NS Query)

El primer paso lógico es identificar qué servidores tienen autoridad sobre el dominio.

```bash
CyberWolfSec@htb[/htb]$ dig ns inlanefreight.htb @10.129.14.128

;; ANSWER SECTION:
inlanefreight.htb. 604800 IN NS ns.inlanefreight.htb.
ns.inlanefreight.htb. 604800 IN A 10.129.34.136
```

## Análisis

* El servidor autoritativo es `ns.inlanefreight.htb`.
* Su IP es `10.129.34.136`.

Esto es importante porque:

* Podemos consultar directamente ese nameserver.
* Puede estar configurado diferente al servidor inicial.
* Puede permitir transferencias de zona.

---

# 3. Version Enumeration (CHAOS Query)

Algunos servidores DNS exponen su versión mediante consultas CHAOS.

```bash
CyberWolfSec@htb[/htb]$ dig CH TXT version.bind 10.129.120.85

;; ANSWER SECTION:
version.bind. 0 CH TXT "9.10.6-P1"
```

## Análisis

* El servidor está corriendo BIND versión 9.10.6-P1.

Esto nos permite:

1. Buscar CVEs específicos para esa versión.
2. Consultar CVE Details.
3. Verificar si existen exploits públicos.

⚠️ No todos los servidores permiten esta consulta.

---

# 4. Consulta ANY

La consulta ANY intenta recuperar todos los registros disponibles que el servidor esté dispuesto a revelar.

```bash
CyberWolfSec@htb[/htb]$ dig any inlanefreight.htb @10.129.14.128

;; ANSWER SECTION:
inlanefreight.htb. 604800 IN TXT "v=v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb. 604800 IN NS ns.inlanefreight.htb.
```

## Qué obtenemos aquí

* Registros TXT (SPF, validaciones externas).
* Nameservers.
* Posible información sobre proveedores externos.

El registro SPF puede revelar:

* Servicios de correo utilizados.
* IPs autorizadas.
* Infraestructura externa integrada.

---


# DNS – Zone Transfer (AXFR) y Enumeración de Subdominios (Explicación Avanzada)

---

# 1. ¿Qué es una Zone Transfer?

En DNS, una **zone transfer** es el mecanismo por el cual un servidor DNS replica (copia) los registros de una zona hacia otro servidor.

Esto existe por una razón operativa fundamental:

* Un fallo de DNS puede dejar fuera de servicio sitios web, correo electrónico y múltiples sistemas críticos.
* Por resiliencia, una zona casi nunca depende de un único servidor.
* Se mantiene la misma información en varios servidores (redundancia) para **alta disponibilidad** y, en algunos casos, **balanceo**.

La zona original se mantiene en un servidor llamado:

* **Primary Name Server** (o servidor **master**)

Y los servidores que replican esa zona se llaman:

* **Secondary Name Servers** (o servidores **slave**)

La transferencia de zona completa se denomina:

* **AXFR (Asynchronous Full Transfer Zone)**

Generalmente se realiza sobre **TCP/53**.

---

# 2. Modelo Master/Slave y Sincronización

En un esquema típico:

* El **master** contiene la versión “oficial” de la zona.
* Los **slaves** consultan periódicamente al master para detectar cambios.

¿Cómo detecta cambios un servidor slave?

1. Solicita el registro **SOA** de la zona.
2. Lee el **serial number**.
3. Si el serial del master es mayor, significa que la zona fue actualizada.
4. El slave ejecuta una transferencia (AXFR o IXFR) para sincronizar.

Este proceso se controla con parámetros del SOA, especialmente:

* **refresh**: cada cuánto el slave revisa el SOA del master.
* **retry**: cuánto espera para reintentar si el master no responde.
* **expire**: cuánto tiempo puede servir la zona sin poder refrescar.

En implementaciones BIND, muchas veces se utiliza una clave (por ejemplo `rndc-key` o claves TSIG) para asegurar que:

* El slave solo sincronice con el master legítimo.
* No cualquiera pueda solicitar transferencias.

---

# 3. Riesgo de Seguridad: allow-transfer mal configurado

La transferencia de zona es **necesaria** para operar, pero es un riesgo crítico si la directiva `allow-transfer` está abierta.

Si el administrador:

* Permite transferencias a una subred amplia por “comodidad”
* O utiliza `any` por pruebas

entonces **cualquier atacante** puede descargar el “mapa” completo de la zona.

Esto es un hallazgo de alto impacto porque:

* Reduce drásticamente el esfuerzo de reconocimiento.
* Revela activos internos no documentados.
* Permite planificar ataques dirigidos de forma precisa.

---

# 4. Zone Transfer (AXFR) – Caso Práctico

La transferencia de zona es uno de los hallazgos más críticos en DNS.

```bash
CyberWolfSec@htb[/htb]$ dig axfr inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr inlanefreight.htb @10.129.14.128
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 4 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:51:19 CEST 2021
;; XFR size: 9 records (messages 1, bytes 520)
```

## Análisis Técnico del Output

AXFR devuelve, entre otros:

* **SOA**: información administrativa y serial de la zona.
* **TXT**: validaciones y políticas (SPF, verificación Atlassian, etc.).
* **NS**: nameserver autoritativo.
* **A records**: subdominios con IPs directas.

Esto permite mapear rápidamente:

* Aplicaciones (`app.inlanefreight.htb`).
* Dominios internos (`internal.inlanefreight.htb`).
* Infra de correo (`mail1.inlanefreight.htb`).

✅ Si `allow-transfer` está mal configurado, el servidor literalmente entrega la base completa de registros.

---

# 5. AXFR en Subzonas Internas

Un punto clave es que muchas organizaciones separan zonas “internas” en subdominios.

Ejemplo: `internal.inlanefreight.htb`.

Si el servidor permite transferencias para esa subzona, la fuga puede ser incluso más grave.

```bash
CyberWolfSec@htb[/htb]$ dig axfr internal.inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr internal.inlanefreight.htb @10.129.14.128
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN   TXT     "MS=ms97310371"
internal.inlanefreight.htb. 604800 IN   TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN   TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      10.129.34.136
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:53:11 CEST 2021
;; XFR size: 15 records (messages 1, bytes 664)
```

## Interpretación del Impacto

Este resultado revela activos extremadamente sensibles:

* **dc1 / dc2** → posibles Controladores de Dominio (AD).
* **vpn** → vector directo de acceso remoto.
* **wsus** → infraestructura de actualización (posibles vectores de supply chain interno).
* **ws1 / ws2** → estaciones de trabajo internas.

En un pentest real, un AXFR exitoso de una zona interna normalmente se reporta como:

* **Information Disclosure (Alta / Crítica)**

porque habilita reconocimiento completo sin escaneo activo.

---

# 6. Cuando AXFR falla: Subdomain Brute Forcing

Si no se puede descargar la zona, se puede intentar descubrir subdominios mediante fuerza bruta.

El concepto es simple:

* Tener una lista de nombres comunes.
* Probarlos contra el DNS.
* Guardar los que resuelven.

```bash
CyberWolfSec@htb[/htb]$ for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';;\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
```

## Notas prácticas

* Se filtran líneas de comentarios (`;;`) y SOA.
* Se guardan hits en `subdomains.txt`.
* Mientras más buena la wordlist, mejor la cobertura.

---

# 7. Uso de DNSenum

Herramientas como [**dnsenum**](https://github.com/fwaeytens/dnsenum) automatizan gran parte del proceso.

```bash
CyberWolfSec@htb[/htb]$ dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb

dnsenum VERSION:1.2.6

-----   inlanefreight.htb   -----

Name Servers:
______________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136

Trying Zone Transfers and getting Bind Versions:
_________________________________________________

Trying Zone Transfer for inlanefreight.htb on ns.inlanefreight.htb ...
AXFR record query failed: no nameservers

Brute forcing with /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt:
____________________________________________________________________________________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136
mail1.inlanefreight.htb.                 604800   IN    A        10.129.18.201
app.inlanefreight.htb.                   604800   IN    A        10.129.18.15

...SNIP...
done.
```

## Qué aporta DNSenum

* Enumera NS y registros relevantes.
* Intenta AXFR automáticamente.
* Ejecuta brute force con wordlists.
* Resume resultados en un solo output.

---

# 8. Conclusión (Pentesting)

### Si AXFR funciona

✅ Descubrís infraestructura completa en segundos.

* Subdominios
* Hosts internos
* IP privadas
* Roles (dc, vpn, mail, wsus)

### Si AXFR no funciona

➡️ Se pasa a brute forcing:

* Wordlists (SecLists)
* Herramientas automatizadas (dnsenum)

En ambos casos, DNS sigue siendo una fuente crítica de información para mapear y priorizar vectores de ataque.




---

# 9. Impacto Estratégico del Footprinting DNS

Un servidor DNS mal configurado puede permitir:

* Reconocimiento completo sin escaneo activo.
* Descubrimiento de infraestructura interna.
* Identificación de vectores de ataque.
* Mapeo organizacional.

En muchos casos, la información obtenida vía DNS supera la obtenida con un escaneo de puertos inicial.

---

# Preguntas


#### Interactúe con el DNS de destino utilizando su dirección IP y enumere su FQDN para el dominio "inlanefreight.htb".

Enviamos una traza `ICMP` al host destino para verificar si está activo:
<img width="522" height="149" alt="image" src="https://github.com/user-attachments/assets/0b6ffc22-c0e4-4689-8618-22ae29f69ea1" />

Realizamos un escaneo rápido para confirmar que el host destino es un servidor DNS:
<img width="733" height="172" alt="image" src="https://github.com/user-attachments/assets/007b266a-dcf8-4b52-82e8-070ef6d86786" />

El ejercicio nos pide que averiguemos el `FQDN` del servidor DNS del dominio `inlanefreight.htb`, interactuando con la IP del servidor DNS.

Entonces procedemos a buscar los servidores `NS` del dominio `inlanefreight.htb`, interactuando con uno de sus servidores DNS a través de su IP:

```bash
dig ns inlanefreight.htb @10.129.6.9
```

<img width="794" height="553" alt="image" src="https://github.com/user-attachments/assets/21eb1845-884c-4889-8e22-67cd8d2c8a3b" />



#### Identifique si es posible realizar una transferencia de zona y envíe el registro TXT como respuesta. (Formato: HTB{...})

`Pista`: Las zonas a menudo tienen el nombre de un subdominio.


Realizamos una transferencia de zona para el dominio:
```bash
dig axfr inlanefreight.htb @10.129.6.9
```
<img width="1907" height="448" alt="image" src="https://github.com/user-attachments/assets/56e1643e-d577-4e3f-9cfc-b5578bf0b6a7" />

No obtenemos un registro `TXT` con el formato pedido, pero sí obtenemos una lista de subdominios.

Procedemos a realizar transferencia de zona para cada uno de esos subdominios hasta encontrar el registro `TXT` con el formato solicitado:

```bash
dig axfr internal.inlanefreight.htb @10.129.6.9
```

<img width="1919" height="811" alt="image" src="https://github.com/user-attachments/assets/58fc1d26-20a8-42df-90c0-72be720fad48" />

Lo encontramos en el subdominio `internal.inlanefreight.htb`.


Por curiosidad, también probamos el método de subdomain bruteforce:
```bash
for sub in $(cat /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.6.9 | grep -v ';;\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```
<img width="1883" height="943" alt="image" src="https://github.com/user-attachments/assets/1386aad0-522d-4273-8d76-7b41e2e89758" />



#### ¿Cuál es la dirección IPv4 del nombre de host DC1?

En el ejercicio anterior, realizamos transferencia de zona para el subdominio `internal` que contiene el host `DC1`:
```bash
dig axfr internal.inlanefreight.htb @10.129.6.9
```

<img width="1902" height="538" alt="image" src="https://github.com/user-attachments/assets/1c053af4-68b3-473f-b5a5-2002b682c763" />


#### ¿Cuál es el FQDN del host donde el último octeto termina con "xxx203"?

`Pista`: Recuerde que las diferentes listas de palabras no siempre tienen las mismas entradas.


Debido a que algunos subdominios permitían transferencia de zona y otros no. Comenzamos probando fuerza bruta con `dnsenum` sobre los subdominios que no permitían transferencia de zona. Esos subdominios son:

- app
- dev
- mail
- ns

Utilizamos el siguiente comando para hacer fuerza bruta y probar con `app`:
```bash
dnsenum --dnsserver 10.129.6.9 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt app.inlanefreight.htb 
```
<img width="1682" height="361" alt="image" src="https://github.com/user-attachments/assets/662c737a-ede2-4724-98b5-36f8bf331531" />

Probamos con `dev`:
```bash
dnsenum --dnsserver 10.129.6.9 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt dev.inlanefreight.htb 
```
<img width="1753" height="815" alt="image" src="https://github.com/user-attachments/assets/7104ff02-e1ba-4e5e-97ed-fa3e14cde259" />

Enumera pero no encuentra la respuesta requerida.

Probamos con `mail`:

```bash
dnsenum --dnsserver 10.129.6.9 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt mail.inlanefreight.htb 
```
<img width="1372" height="268" alt="image" src="https://github.com/user-attachments/assets/7a31b8a1-2605-4ee7-900a-e9d0cd511786" />

Probamos con `ns`:
```bash
dnsenum --dnsserver 10.129.6.9 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt ns.inlanefreight.htb 
```
<img width="1338" height="287" alt="image" src="https://github.com/user-attachments/assets/fbde13c2-5fea-40d0-8715-8eed73cbafda" />




Como el único subdominio que podemos enumerar por fuerza bruta es `dev`, y no encontramos el subdominio de `dev` que nos piden, intentamos realizar fuerza bruta con otra wordlist, en este caso `fierce-hostlist.txt`:

```bash
dnsenum --dnsserver 10.129.6.9 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb
```

<img width="1580" height="787" alt="image" src="https://github.com/user-attachments/assets/bb21cd1a-e425-4f8f-85a6-d6401e9eb311" />

El `FQDN` es `win2k.dev.inlanefreight.htb`.
