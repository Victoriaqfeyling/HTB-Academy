
# DNS Zone Transfers

## Introducción

Si bien el **brute-force de subdominios** puede ser muy efectivo, existe una técnica **potencialmente más eficiente y menos ruidosa** para descubrir subdominios: los **DNS Zone Transfers**.

Este mecanismo, pensado para la **replicación legítima** de registros DNS entre servidores, puede convertirse en una **fuente masiva de información** cuando está mal configurado.

---

## ¿Qué es una Zone Transfer?

Una **DNS Zone Transfer** es una copia completa de **todos los registros DNS** de una zona (un dominio y sus subdominios) desde un servidor DNS hacia otro.

Su objetivo legítimo es:

* Mantener **consistencia** entre servidores primarios y secundarios
* Asegurar **redundancia** y **alta disponibilidad**

Sin embargo, si no se aplican controles de acceso adecuados, **cualquier tercero no autorizado** puede descargar el archivo completo de zona.

Esto puede revelar:

* Todos los subdominios
* Direcciones IP asociadas
* Registros de correo, servicios y configuraciones internas

---

## Cómo funciona una Zone Transfer

El proceso típico consta de los siguientes pasos:

1. **Solicitud de transferencia (AXFR)**
   El servidor secundario envía una solicitud de transferencia completa (*AXFR*) al servidor primario.

2. **Transferencia del registro SOA**
   El servidor primario responde con el registro **SOA (Start of Authority)**, que contiene información clave como el número de serie de la zona, lo que ayuda al servidor secundario a determinar si los datos de su zona están actualizados.

3. **Transmisión de registros DNS**
   Se transfieren todos los registros de la zona: A, AAAA, MX, NS, CNAME, TXT, SRV, etc.

4. **Finalización de la transferencia**
   El servidor primario indica que la transferencia fue completada.

5. **Acknowledgement (ACK)**
   El servidor secundario confirma la recepción correcta de la información.

<img width="975" height="608" alt="image" src="https://github.com/user-attachments/assets/dd1a8eb4-10e2-4547-b71e-27a3e2d6cb71" />


---

## La vulnerabilidad de Zone Transfer

En los primeros años de Internet, era común permitir **transferencias de zona abiertas**, ya que simplificaban la administración.

Este enfoque introdujo una vulnerabilidad crítica:

* Cualquier cliente podía solicitar una copia completa de la zona
* No existían restricciones de origen

Como resultado, un atacante podía obtener un **mapa completo de la infraestructura DNS** de la organización.

---

## Información expuesta en una Zone Transfer

Una transferencia de zona no autorizada puede revelar:

### Subdominios

* Subdominios no públicos
* Entornos de desarrollo y staging
* Paneles administrativos
* Servicios internos

---

### Direcciones IP

* IPs internas o externas
* Posibles objetivos para escaneos adicionales

---

### Registros DNS críticos

* Name Servers (NS)
* Servidores de correo (MX)
* Servicios específicos (SRV)
* Registros TXT con información sensible

Toda esta información reduce drásticamente la **incertidumbre del atacante**.

---

## Mitigación

Hoy en día, la mayoría de los servidores DNS están configurados correctamente para:

* Permitir transferencias **solo a servidores secundarios autorizados**
* Bloquear solicitudes AXFR desde clientes no confiables

Sin embargo, errores de configuración o prácticas obsoletas aún pueden provocar exposiciones.

Por este motivo, **probar una zone transfer (con autorización)** sigue siendo una técnica válida durante el reconocimiento.

---

## Explotación de Zone Transfers con dig

El comando `dig` permite solicitar una transferencia de zona completa.

### Comando de ejemplo

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

Este comando:

* Solicita una transferencia AXFR
* Especifica el name server objetivo
* Indica el dominio a transferir

---

## Ejemplo práctico: zonetransfer.me

El dominio `zonetransfer.me` está diseñado específicamente para **demostrar los riesgos** de las zone transfers.

### Fragmento del output

```text
; <<>> DiG 9.18.12-1~bpo11+1-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
...
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
...
;; Query time: 10 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Mon May 27 18:31:35 BST 2024
;; XFR size: 50 records (messages 1, bytes 2085)
```

<img width="1913" height="718" alt="image" src="https://github.com/user-attachments/assets/27dcdbea-7a49-4ba9-8ebb-71ffd3c35494" />


Este resultado expone:

* Subdominios
* Registros de correo
* Direcciones IP
* Servicios internos

---

## Consideraciones de OPSEC

* Las solicitudes AXFR pueden ser **registradas y alertadas**
* Algunos servidores responden parcialmente o bloquean el intento
* Repetir intentos innecesarios aumenta la detección

Buenas prácticas:

* Probar transferencias solo una vez
* Validar previamente los name servers
* Priorizar técnicas pasivas antes de AXFR

---

## Conclusión

Las **DNS Zone Transfers** representan una de las vulnerabilidades históricas más críticas en DNS.

Aunque hoy son menos comunes, cuando están presentes permiten obtener:

* Un inventario completo de subdominios
* Información estructural de alto valor
* Un punto de partida sólido para fases posteriores del pentest

Por su **alto impacto y bajo esfuerzo**, verificar la posibilidad de una zone transfer debería formar parte de cualquier estrategia seria de Web Reconnaissance.

---

# PREGUNTAS

### Tras realizar una transferencia de zona para el dominio inlanefreight.htb en el sistema de destino, ¿cuántos registros DNS se recuperan del servidor de nombres del sistema de destino? Indique su respuesta como un número entero, por ejemplo, 123.

Al encontrarnos dentro de una red interna no podemos utilizar el comando `dig` utilizando el resolver de nuestro ISP, ya que necesitamos resolver a una ip privada dentro de la intranet. Por lo tanto, necesitamos encontrar el servidor dns para este dominio.

Luego de agregar la ip objetivo al ´etc/hosts´ escaneamos con nmap el dominio:

<img width="891" height="339" alt="image" src="https://github.com/user-attachments/assets/7f64b2bd-0d70-4fb8-913b-cea7e5a2d7f4" />

Descubrimos que el mismo dominio es el servidor dns.

Realizamos la transferfrencia de zona utilizando el siguiente comando:
```bash
dig AXFR @10.129.17.180 inlanefreight.htb
```
<img width="1662" height="769" alt="image" src="https://github.com/user-attachments/assets/19529079-84a3-449a-9988-211c6d5785f9" />

### Dentro del registro de zona transferido anteriormente, busque la dirección IP de ftp.admin.inlanefreight.htb. Responda solo con la dirección IP, p. ej., 127.0.0.1.

<img width="766" height="78" alt="image" src="https://github.com/user-attachments/assets/d62d1f86-b0d4-4f2c-adb8-347f87956d76" />


### Dentro del mismo registro de zona, identifique la dirección IP más grande asignada dentro del rango de IP 10.10.200. Responda con la dirección IP completa, p. ej., 10.10.200.1.
<img width="801" height="54" alt="image" src="https://github.com/user-attachments/assets/9b393cbd-ca57-4ea8-8629-61f6b17c51c7" />
