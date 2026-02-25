# 📡 SNMP 

---

# 1️⃣ ¿Qué es SNMP y para qué sirve?

**SNMP (Simple Network Management Protocol)** es un protocolo diseñado para **monitorizar y administrar dispositivos de red**.

En palabras simples:

👉 Permite preguntarle cosas a un dispositivo.
👉 Permite cambiar configuraciones remotamente.

Imaginá que tenés un router en una empresa. Con SNMP podrías:

* Ver cuánta CPU está usando.
* Ver el tráfico de red.
* Saber si una interfaz está caída.
* Cambiar ciertos parámetros de configuración.

Por eso no es solo un protocolo de "consulta" — también puede **modificar valores**.

---

# 2️⃣ ¿Qué dispositivos suelen tener SNMP?

Muchísimos dispositivos de infraestructura lo usan:

* Routers
* Switches
* Firewalls
* Servidores
* Impresoras
* Dispositivos IoT
* Equipamiento industrial

En entornos corporativos es extremadamente común.

---

# 3️⃣ Cómo funciona SNMP (modelo simple)

SNMP funciona con un modelo:

## 🖥 Agente ↔ Cliente

### 🔹 Agente SNMP

Es el servicio que corre dentro del dispositivo (router, switch, etc).

Es quien "tiene la información".

### 🔹 Cliente SNMP

Es el sistema que consulta o modifica esa información.

Es quien hace las preguntas.

---

# 4️⃣ Puertos que usa SNMP

SNMP usa UDP.

* **UDP 161 → Consultas normales (GET, SET, WALK)**
* **UDP 162 → Traps (alertas automáticas)**

---

# 5️⃣ ¿Qué significa que SNMP puede modificar cosas?

No solo se limita a enviar información.

También puede enviar **comandos de control**.

Ejemplo:

* El cliente puede cambiar el nombre del dispositivo.
* Puede modificar ciertos parámetros.
* Puede activar o desactivar funciones.

Esto ocurre mediante comandos enviados al **puerto UDP 161**.

Desde el punto de vista de seguridad:

⚠ Si está mal configurado, puede permitir cambios no autorizados.

---

# 6️⃣ ¿Qué son los SNMP Traps? (UDP 162)

En la comunicación clásica:

Cliente → Pregunta
Servidor → Responde

Pero SNMP agrega algo más:

## 🔔 Traps

Un **trap** es un mensaje que el dispositivo envía automáticamente cuando ocurre un evento.

Ejemplos:

* Se cae una interfaz de red
* Se reinicia el dispositivo
* Se supera un umbral de CPU

El dispositivo envía esa alerta sin que nadie la haya pedido.

Esto viaja por **UDP 162**.

---

# 7️⃣ ¿Cómo sabe SNMP qué información pedir?

Acá aparece algo clave:

## 📌 OID (Object Identifier)

Cada dato que puede consultarse en un dispositivo tiene un identificador único llamado **OID**.

Un OID es como una dirección.

Ejemplo conceptual:

* Un OID para la CPU
* Un OID para el uptime
* Un OID para el nombre del sistema

Si el cliente quiere saber el uptime, debe pedir el OID correspondiente.

Sin OIDs, SNMP no puede funcionar.

---

# 8️⃣ ¿Por qué es importante en pentesting?

Porque muchas veces:

* Está habilitado innecesariamente
* Usa versiones antiguas (v1 o v2c)
* Usa comunidades por defecto como "public"

Si se puede consultar libremente, puede revelar:

* Información del sistema operativo
* Interfaces de red
* Usuarios
* Información interna de la infraestructura

Por eso es una fase clave en enumeración.

---

# 9️⃣ Versiones de SNMP

* **SNMPv1** → Antigua, sin seguridad real (también usa comunidades)
* **SNMPv2c** → Similar a v1, usa comunidades
* **SNMPv3** → Introduce autenticación y cifrado

En la práctica, muchas redes todavía usan v1 o v2c.

---


---

# 🌳 SNMP – MIB y OID

---

# 1️⃣ El Problema que Resuelve la MIB

Imaginemos esta situación:

Tenés:

* Un router Cisco
* Un switch HP
* Un firewall Fortinet

Todos usan SNMP.

Pero… ¿cómo hace un cliente SNMP para entenderse con todos si cada fabricante diseña sus dispositivos de forma diferente?

👉 Para resolver ese problema se creó la **MIB (Management Information Base)**.

La MIB es el "diccionario universal" que permite que SNMP funcione de manera estándar entre distintos fabricantes.

---

# 2️⃣ ¿Qué es exactamente una MIB?

Una **MIB** es:

* Un archivo de texto
* Escrito en formato **ASN.1 (Abstract Syntax Notation One)**
* Con estructura jerárquica tipo árbol
* Que describe TODOS los objetos que pueden consultarse vía SNMP

⚠ Importante:

La MIB **NO contiene datos reales**.

No guarda métricas.
No guarda valores.

Solo describe:

* Qué se puede consultar
* Dónde está
* Qué tipo de dato devuelve
* Si puede modificarse o no

---

# 3️⃣ ¿Qué información contiene una MIB?

Cada objeto dentro de la MIB define:

* 📌 Un OID (dirección única)
* 🏷 Un nombre simbólico
* 🧾 Tipo de dato (Integer, String, Counter, etc.)
* 🔐 Permisos (read-only o read-write)
* 📖 Descripción textual

Es decir, responde a la pregunta:

> "¿Dónde está la información y cómo debo interpretarla?"

---

# 4️⃣ La Estructura en Forma de Árbol

La MIB organiza los objetos en una estructura jerárquica tipo árbol.

Visualmente sería algo así:

```
1
└── 3
    └── 6
        └── 1
            └── 2
                └── 1
                    └── 1
```

Cada nivel representa un nodo.

Cada nodo tiene un número.

La combinación completa forma un **OID**.

---

# 5️⃣ ¿Qué es un OID?

Un **OID (Object Identifier)** es la dirección exacta de un objeto dentro del árbol SNMP.

Ejemplo real:

```
1.3.6.1.2.1.1.1.0
```

Ese número representa una ruta dentro del árbol.

Es como una ruta de carpetas en un sistema operativo:

```
/home/juan/documentos/archivo.txt
```

Pero en formato numérico.

---

# 6️⃣ Cómo Leer un OID

Tomemos este ejemplo:

```
1.3.6.1.2.1.1.1.0
```

Cada número representa un nivel en la jerarquía.

Cuanto más largo el OID:

👉 Más específico es el objeto.

Muchos nodos intermedios no contienen datos reales.
Solo sirven como organización.

Los datos reales suelen estar en los nodos finales.

---

# 7️⃣ ¿Por Qué el OID es Tan Importante?

Cuando el cliente SNMP quiere información:

No dice:

"Dame la CPU"

Dice:

"Dame el valor del OID X"

El agente busca ese OID y devuelve el valor correspondiente.

Sin OID:

❌ No hay forma de saber qué dato pedir.

---

# 8️⃣ Relación entre MIB y OID

La relación es simple:

* La MIB define el mapa
* El OID es la dirección dentro del mapa
* El agente tiene los datos reales

Ejemplo mental:

MIB → Manual del edificio
OID → Número de departamento
Agente → Persona que vive ahí
Cliente → Quien toca el timbre

---

# 9️⃣ ASN.1 – ¿Por Qué es Relevante?

Las MIB están escritas en **ASN.1**, que es un estándar para definir estructuras de datos.

No necesitás dominar ASN.1 para hacer pentesting.

Pero es importante entender que:

* Es un formato estructurado
* Es estándar
* Permite que distintos fabricantes sean compatibles

---

# 🔟 Repositorios Útiles para Consultar OIDs

Podés buscar OIDs y su significado en:

* [https://oidref.com](https://oidref.com)
* [https://www.alvestrand.no/objectid/](https://www.alvestrand.no/objectid/)
* [https://oid-info.com](https://oid-info.com)

Estos sitios te permiten traducir un OID numérico en algo entendible.

---

# 1️⃣1️⃣ ¿Por Qué Esto es Clave en Pentesting?

Cuando hacés un:

```
snmpwalk -v2c -c public 10.10.10.10
```

El resultado que ves son OIDs con valores.

Si entendés la estructura:

✔ Podés identificar qué información es sensible
✔ Podés detectar usuarios
✔ Podés identificar interfaces
✔ Podés obtener datos del sistema operativo
✔ Podés descubrir información interna de red

Si no entendés OIDs y MIBs, el output parece ruido.

---


---

# 🔐 SNMP – Versiones y Community Strings

---

# 1️⃣ ¿Por qué existen distintas versiones de SNMP?

SNMP nació en una época donde:

* Las redes eran pequeñas
* No existía tanta exposición a Internet
* La seguridad no era una prioridad como hoy

Con el tiempo, se descubrieron muchos problemas de seguridad.

En lugar de eliminar el protocolo, se fueron creando nuevas versiones para intentar corregir esos problemas.

Por eso hoy existen:

* SNMPv1
* SNMPv2c
* SNMPv3

Cada una mejora (en teoría) la anterior.

---

# 2️⃣ 🔴 SNMPv1 – La versión original

Es la primera versión del protocolo.

Características principales:

* Permite consultar información (GET)
* Permite modificar valores (SET)
* Permite recibir traps

⚠ Problemas graves de seguridad:

❌ No tiene autenticación real
❌ No tiene cifrado
❌ Todo viaja en texto plano

Eso significa que:

Si alguien captura el tráfico de red (por ejemplo con Wireshark), puede leer absolutamente todo.

Incluso puede ver la "contraseña" utilizada.

---

# 3️⃣ ¿Qué significa que viaja en texto plano?

Texto plano significa:

* No está cifrado
* No está protegido
* Se puede leer directamente

Es como mandar una contraseña por WhatsApp sin cifrado.

Cualquiera que intercepte el tráfico puede verla.

---

# 4️⃣ 🟠 SNMPv2c – Community-Based

SNMPv2 tuvo varias variantes.

La única que se mantiene en uso hoy es:

👉 SNMPv2c

La "c" significa **community-based**.

Esto significa que introduce el concepto de:

## Community String

Pero atención:

Aunque mejora algunas funciones técnicas del protocolo...

⚠ En términos de seguridad es prácticamente igual a v1.

¿Por qué?

Porque la community string también viaja en texto plano.

---

# 5️⃣ Entonces… ¿SNMPv2c es seguro?

No.

Es más práctico y eficiente que v1.

Pero desde el punto de vista de seguridad:

Es casi lo mismo.

Si alguien intercepta el tráfico:

Puede ver la community string.

Y si tiene la community correcta:

Puede enumerar completamente el dispositivo.

---

# 6️⃣ 🟢 SNMPv3 – La versión segura

SNMPv3 fue creado para solucionar los problemas de seguridad.

Introduce:

✔ Autenticación basada en usuario y contraseña
✔ Cifrado del tráfico
✔ Integridad de los mensajes

Esto significa que:

* No cualquiera puede consultar
* No cualquiera puede modificar
* El tráfico no puede leerse fácilmente

Ahora sí existe protección real.

---

# 7️⃣ ¿Entonces por qué no todos usan SNMPv3?

Porque:

* Es más complejo de configurar
* Requiere más parámetros
* Migrar redes grandes puede ser costoso
* Muchos dispositivos legacy no lo soportan completamente

Resultado práctico:

Muchas organizaciones siguen usando SNMPv2c.

Y ahí es donde aparece el problema.

---

# 8️⃣ Community Strings – Qué son realmente

Una **community string** es como una contraseña simple.

Funciona como un "token" que el cliente envía al agente.

Ejemplo clásico:

```
snmpwalk -v2c -c public 10.10.10.10
```

En ese ejemplo:

"public" es la community string.

Si es correcta:

El dispositivo responde.

Si es incorrecta:

No responde o devuelve error.

---

# 9️⃣ Tipos comunes de community strings

Las más comunes (por mala práctica):

* public
* private
* public123
* nombre_del_host

Muchos dispositivos vienen por defecto con:

* public → solo lectura
* private → lectura y escritura

Si no se cambian…

Quedan expuestos.

---

# 🔟 ¿Por qué son peligrosas?

Porque:

* Viajan en texto plano
* Pueden interceptarse
* Muchas veces no se cambian
* Se reutilizan en múltiples dispositivos

Si un atacante obtiene una community válida:

Puede:

✔ Enumerar usuarios
✔ Obtener interfaces
✔ Ver rutas internas
✔ Obtener versión del sistema
✔ Modificar configuraciones (si tiene permisos)

---

# 1️⃣1️⃣ Escenario de Ataque Real

Imaginemos:

Una empresa usa SNMPv2c con community "public".

Un atacante en la misma red ejecuta:

```
snmpwalk -v2c -c public 192.168.1.1
```

Si responde:

Acaba de exponer:

* Información del dispositivo
* Arquitectura interna
* Información de red

Todo sin credenciales fuertes.

---

---

# ⚙️ SNMP – Configuración del Demonio y Configuraciones Peligrosas

---

# 1️⃣ ¿Qué es el demonio SNMP?

En sistemas Linux, el servicio SNMP que responde a las consultas se llama:

```
snmpd
```

Es un **demonio (daemon)**, es decir:

👉 Un proceso que corre en segundo plano esperando consultas.

Este servicio es el que:

* Escucha en UDP 161
* Responde consultas SNMP
* Aplica reglas de acceso
* Define qué puede verse y qué puede modificarse

---

# 2️⃣ Archivo de configuración principal

El comportamiento del servicio se define en:

```bash
/etc/snmp/snmpd.conf
```

Este archivo controla:

* Qué IP escucha
* Qué OIDs son visibles
* Qué comunidades existen
* Qué usuarios pueden autenticarse
* Qué permisos tiene cada uno

---

# 3️⃣ Ejemplo real de configuración

Comando para ver solo líneas activas:

```bash
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```

Salida:

```
sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly
```

---

# 4️⃣ Analizando línea por línea

### sysLocation

Define la ubicación física del dispositivo.

⚠ Puede revelar información sensible.

---

### sysContact

Persona responsable del sistema.

⚠ Puede revelar emails internos.

---

### agentaddress

```
agentaddress 127.0.0.1,[::1]
```

Define en qué IP escucha el demonio.

En este caso:

Solo localhost.

Si estuviera configurado como:

```
agentaddress udp:161
```

Escucharía en todas las interfaces.

---

### view systemonly

```
view systemonly included .1.3.6.1.2.1.1
```

Define qué parte del árbol OID puede verse.

Aquí solo permite consultar:

* Información del sistema

Esto limita el alcance.

---

### rocommunity public default -V systemonly

Significa:

* Community: public
* Permiso: read-only
* Desde: cualquier IP
* Vista: systemonly

Traducción práctica:

Cualquiera que conozca la community "public" puede consultar los OIDs definidos en la vista.

---

### rouser authPrivUser authpriv -V systemonly

Define un usuario SNMPv3 con:

* Autenticación
* Cifrado
* Acceso limitado a systemonly

Esto ya es una configuración más segura.

---

# 5️⃣ ¿Qué controla realmente snmpd.conf?

El archivo determina:

✔ Qué parte del árbol OID es accesible
✔ Quién puede leer
✔ Quién puede escribir
✔ Desde qué IP se permite acceso
✔ Si requiere autenticación

Es literalmente la política de seguridad de SNMP.

---

# 6️⃣ Configuraciones peligrosas

Algunas directivas pueden ser extremadamente riesgosas.

---

## 🔴 rwuser noauth

Significa:

* Usuario con permisos de escritura
* Sin autenticación
* Acceso al árbol completo

Impacto:

Un atacante podría modificar configuraciones sin autenticarse.

---

## 🔴 rwcommunity <string> <IP>

Ejemplo:

```
rwcommunity private 0.0.0.0/0
```

Significa:

* Community con permisos de lectura y escritura
* Desde cualquier IP
* Acceso completo al árbol OID

Impacto:

Acceso total si la community es conocida.

---

## 🔴 rwcommunity6

Misma lógica pero para IPv6.

Muchas veces se protege IPv4 pero se olvidan reglas en IPv6.

---

# 7️⃣ ¿Por qué esto es grave?

Con permisos de escritura (rw):

Un atacante podría:

✔ Modificar configuraciones de red
✔ Cambiar parámetros del sistema
✔ Activar o desactivar servicios
✔ Alterar rutas

En ciertos escenarios:

Puede facilitar ejecución remota indirecta.

---

# 8️⃣ Error común en administradores

Muchos piensan:

"SNMP solo es monitoreo"

Pero en realidad:

SNMP puede modificar el sistema si está mal configurado.

Y si además:

* Usa v2c
* Usa community débil
* Está expuesto a Internet

Se convierte en una superficie de ataque seria.

---

# 9️⃣ Recomendación práctica

Para entender bien SNMP:

Lo ideal es:

✔ Instalar una VM
✔ Configurar snmpd manualmente
✔ Probar distintas comunidades
✔ Hacer snmpwalk
✔ Cambiar permisos

Se recomienda investigar las [páginas del manual](https://www.net-snmp.org/docs/man/snmpd.conf.html) 

Nada reemplaza verlo funcionando.

---

# 🔎 Conclusión

El archivo:

```
/etc/snmp/snmpd.conf
```

Es el corazón de la seguridad SNMP.

Si está mal configurado:

Puede permitir:

* Enumeración masiva
* Exposición de información interna
* Modificación remota
* Escalada de impacto

Por eso, en pentesting, cuando vemos SNMP abierto:

Siempre debemos preguntarnos:

👉 ¿Qué configuración está detrás?
👉 ¿Es solo lectura?
👉 ¿Tiene escritura?
👉 ¿Está restringido por IP?
👉 ¿Es v2c o v3?

Responder esas preguntas define el impacto real.


---

# 🔎 SNMP – Footprinting y Enumeración Práctica

---

# 1️⃣ ¿Qué significa “footprinting” en SNMP?

Footprinting es la fase donde recolectamos información sin explotar directamente el sistema.

En el caso de SNMP, esto es especialmente potente porque:

👉 SNMP está diseñado para exponer información del sistema.

Si está mal configurado, nos permite obtener:

* Sistema operativo
* Versión del kernel
* Usuarios
* Procesos
* Servicios instalados
* Paquetes del sistema
* Información de red

Y todo esto sin explotación activa.

---

# 2️⃣ Herramientas principales para SNMP

Las tres herramientas más utilizadas en pentesting son:

* **snmpwalk** → Enumeración completa del árbol OID
* **onesixtyone** → Fuerza bruta de community strings
* **braa** → Enumeración masiva y rápida de OIDs

Cada una cumple un rol distinto en la metodología.

---

# 3️⃣ Enumeración con snmpwalk

Comando básico:

```bash
snmpwalk -v2c -c public 10.129.14.128
```

Parámetros:

* `-v2c` → Versión del protocolo
* `-c public` → Community string
* IP → Objetivo

Si la community es válida y SNMP está abierto, veremos algo como esto:

```
iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (5134) 0:00:51.34
iso.3.6.1.2.1.1.4.0 = STRING: "mrb3n@inlanefreight.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "htb"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
...
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/boot/vmlinuz-5.11.0-34-generic root=UUID=9a6a5c52-f92a-42ea-8ddf-940d7e0f4223 ro quiet splash"
...
iso.3.6.1.2.1.25.6.3.1.2.1235 = STRING: "proftpd-basic_1.3.6c-2_amd64"
iso.3.6.1.2.1.25.6.3.1.2.1243 = STRING: "python3_3.8.2-0ubuntu2_amd64"
```

---

# 4️⃣ ¿Qué información estamos obteniendo realmente?

Del primer bloque ya obtenemos:

✔ Sistema operativo
✔ Versión exacta del kernel
✔ Arquitectura
✔ Nombre del host
✔ Email interno
✔ Ubicación física

Luego vemos:

```
BOOT_IMAGE=/boot/vmlinuz...
```

Eso nos da información de arranque.

Y más abajo vemos paquetes instalados:

```
proftpd-basic_1.3.6c-2
python3_3.8.2
```

👉 Esto es oro en pentesting.

Ya tenemos:

* OS fingerprint
* Versiones específicas
* Software potencialmente vulnerable

Sin explotar nada.

---

# 5️⃣ ¿Qué pasa si no conocemos la community string?

Ahí entra:

## onsixtyone

Instalación:

```bash
sudo apt install onesixtyone
```

Uso:

```bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

Salida ejemplo:

```
Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic #41~20.04.2-Ubuntu SMP Fri Sep 24 09:06:38 UTC 2021 x86_64
```

Aquí descubrimos que la community válida es:

```
public
```

---

# 6️⃣ Wordlists útiles

Repositorio recomendado:

[https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)

# 🔑 SNMP – Patrones de Community Strings y Creación de Wordlists Personalizadas

---

# 1️⃣ Un error común en redes grandes

Cuando SNMP está implementado en redes pequeñas, muchas veces se deja la community por defecto:

* public
* private

Pero en redes empresariales grandes (50, 100 o más servidores), la realidad cambia.

Los administradores suelen:

* Cambiar la community por algo "más seguro"
* Asociarla al hostname
* Agregar números
* Agregar símbolos

Ejemplos reales típicos:

* servidor01
* srv-backup
* fw-core!
* db01_snmp
* router123
* switch-core$

A primera vista parece más complejo.

Pero aparece un problema importante.

---

# 2️⃣ El problema de la consistencia

En entornos grandes, los administradores necesitan mantener orden.

Si hay 100 servidores, no pueden inventar una community totalmente distinta y aleatoria para cada uno.

Entonces tienden a usar patrones repetitivos:

* hostname
* hostname + año
* hostname + número incremental
* prefijo común + hostname
* sufijo fijo para todos

Ejemplo en red real:

* snmp_srv01
* snmp_srv02
* snmp_srv03

O:

* core-router!
* access-router!
* backup-router!

Esto genera una oportunidad ofensiva:

👉 Si descubrimos el patrón en un host, podemos inferir el resto.

---

# 3️⃣ Comunidades vinculadas a IP específicas

A veces la configuración restringe la community a ciertas IPs:

```
rwcommunity core-router 192.168.1.10
```

Pero si el servicio está mal filtrado o expuesto:

Podemos intentar descubrir la community igualmente.

Y si sabemos cómo se nombran internamente los hosts:

Podemos generar wordlists inteligentes.

---

# 4️⃣ Ataque basado en patrones

En lugar de usar solo listas genéricas como:

* public
* private
* admin

Podemos construir listas basadas en:

* Naming convention de la empresa
* Dominios internos
* Hostnames descubiertos por DNS
* Subdominios
* Información filtrada en banners

Por ejemplo, si detectamos hostname:

```
htb-core-01
```

Podríamos probar:

* htb-core-01
* htb-core-01!
* htb-core-01_snmp
* core-01
* core01
* core01!

Este enfoque es mucho más efectivo que brute force ciego.

---

# 5️⃣ Creación de wordlists personalizadas con Crunch

Herramienta útil:

[https://secf00tprint.github.io/blog/passwords/crunch/advanced/en](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en)

Crunch permite generar listas personalizadas basadas en reglas.

Ejemplos básicos:

Sintaxix básica:
```bash
crunch <min_length> <max_length>
```

Generar combinaciones numéricas:

```bash
crunch 4 4 0123456789 -o numbers.txt
```

Generar combinaciones basadas en patrón:

```bash
crunch 8 12 -t core@@%% -o custom.txt
```

Donde:

* @ → letra minúscula
* , → letra mayúscula
* % → número

Podríamos generar algo como:

* coreab12
* corexy45

`Nota`: En este caso, el patrón define realmente 8 caracteres.

También podemos usar diccionarios base y agregar reglas.



---

# 6️⃣ Estrategia inteligente en pentesting

El enfoque profesional no es:

❌ Probar millones de combinaciones aleatorias.

Sino:

✔ Analizar naming conventions
✔ Identificar patrones
✔ Generar wordlists dirigidas
✔ Reducir ruido y tiempo

En redes grandes, la previsibilidad humana es una debilidad.

---


---

# 7️⃣ Enumeración masiva con braa

Una vez que conocemos la community, podemos usar [braa](https://github.com/mteg/braa):

```bash
sudo apt install braa
```

Sintaxis general:

```bash
braa <community>@<IP>:.1.3.6.*
```

Ejemplo real:

```bash
braa public@10.129.14.128:.1.3.6.*
```

Salida:

```
10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64
10.129.14.128:20ms:.1.3.6.1.2.1.1.4.0:mrb3n@inlanefreight.htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.5.0:htb
```

Braa es más rápido que snmpwalk y genera menos ruido.

Es ideal cuando queremos consultas específicas.

---

# 8️⃣ Diferencias prácticas entre herramientas

snmpwalk:

* Muy completo
* Mucho output
* Más ruido

onesixtyone:

* Descubre communities
* Paso previo necesario

braa:

* Rápido
* Selectivo
* Ideal para automatización

---

# 9️⃣ Escenario completo de ataque

1. Detectamos UDP 161 abierto con Nmap.
2. Probamos community "public".
3. Si no funciona, usamos onesixtyone.
4. Encontramos community válida.
5. Enumeramos con snmpwalk o braa.
6. Identificamos software vulnerable.

Todo sin explotación directa.

---

# 🔟 Conclusión

SNMP es uno de los servicios más subestimados en enumeración.

Un SNMP mal configurado puede revelar:

✔ Arquitectura completa
✔ Usuarios internos
✔ Software instalado
✔ Información del sistema
✔ Detalles de red

Y lo más importante:

Todo esto puede obtenerse sin explotación activa.

Por eso, en infraestructura, SNMP debe revisarse siempre.

Y en pentesting, debe enumerarse siempre.



---



### Preguntas


#### Enumere el servicio SNMP y obtenga la dirección de correo electrónico del administrador. Envíela como respuesta.


Envíamos una traza `ICMP` al host para verificar si se encuentra activo:

<img width="641" height="167" alt="image" src="https://github.com/user-attachments/assets/13052c81-9dac-4b5c-b6e8-ceec7e854b6a" />


Hacemos un escaneo UDP sobre el puerto 161 para verificar que el servicio se encuentra disponible:
<img width="631" height="156" alt="image" src="https://github.com/user-attachments/assets/918be748-bfa9-413b-93d8-1c524c91d39e" />


Descubrimos es servicio SNMP en el puerto UDP 161, realizamos un escaneo de versiones y lanzamos el script=banner:

<img width="886" height="296" alt="image" src="https://github.com/user-attachments/assets/07da4d2c-d2e7-493b-8c07-76c7f8804561" />


Vemos `SNMPv1` activo y la comunity string `public`.

`SNMPv3` está instalado, pero `SNMPv1/v2c` sigue abierto, lo cual anula completamente la seguridad.

---

Buscamos todos los scripts NSE disponibles en el sistema:
```bash
find / -type f -name snmp* 2>/dev/null |grep scripts
```
<img width="661" height="323" alt="image" src="https://github.com/user-attachments/assets/116d69fe-ce24-4adb-b6c0-b92c696a50bd" />



Realizamos un escaneo de versiones en nmap lanzando un conjunto de scripts predeterminados:
```bash
nmap -Pn -n --reason -sU -sVC -p161 <ip>
```
Vemos que la enumeración es extensa:
<img width="906" height="887" alt="image" src="https://github.com/user-attachments/assets/6a6c26f9-dbc4-4daf-ae84-f048095d4c72" />


---

Enumeramos con `snmpwalk`, probamos ingresar por el protocolo `snmp v1`:
```bash
snmpwalk -v1 -c public <ip>
```
<img width="948" height="874" alt="image" src="https://github.com/user-attachments/assets/9604cfd6-06f6-4d92-9547-221008f6991e" />


`Nota`: También podríamos conectarnos por el protocolo `v2c`:
```bash
snmpwalk -v2c -c public 10.129.8.142
```
<img width="938" height="882" alt="image" src="https://github.com/user-attachments/assets/0fd43604-c9f6-4f06-86f1-a9692bafe823" />


Obtenemos el mail del administrador: `devadmin@inlanefreight.htb`

---


#### ¿Cuál es la versión personalizada del servidor SNMP?




Al realizar la enumeración anterior, pudimos observar la versión personalizada:

<img width="947" height="881" alt="image" src="https://github.com/user-attachments/assets/a7b3efe4-5379-4322-9f2c-33a0949d9a1a" />


Versión personalizada:  `InFreight SNMP v0.91`


Para experimentar, ya que conocemos su `OID`, lo consultamos con `braa`, su sintaxis es la siguiente:
```bash
braa <community>@<IP>:<OID>
```

La cadena es:
```
iso.3.6.1.2.1.1.6.0 = STRING: "InFreight SNMP v0.91"
```
`iso` no es una parte "especial". Es simplemente el nombre simbólico del nodo raíz, por lo tanto:
```
iso = 1
```

Por lo que el `OID` es:
```
1.3.6.1.2.1.1.6.0
```

Entonces usamos el siguiente comando:
```bash
braa public@10.129.8.142:.1.3.6.1.2.1.1.6.0
```
<img width="669" height="93" alt="image" src="https://github.com/user-attachments/assets/6755b792-ae26-4e59-8469-d20f55b04ba7" />


#### Enumere el script personalizado que se está ejecutando en el sistema y envíe su salida como respuesta.

Cuando utilizamos el siguiente comando, pudimos ver el script en la enumeración:
```bash
snmpwalk -v2c -c public 10.129.8.142
```


<img width="1865" height="955" alt="image" src="https://github.com/user-attachments/assets/bba67da3-1693-4f9f-bb93-e1bcc4bea640" />


`Script`: `/usr/share/flag.sh`

`Salida del script`: `HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}`
