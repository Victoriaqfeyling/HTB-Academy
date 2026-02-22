#  Metodología de Enumeración



---

## 1. Introducción a la Metodología de Enumeración

Los procesos complejos, como un pentest, requieren una **metodología estandarizada**. Sin una guía clara, es fácil omitir pasos o repetir acciones sin sentido. Muchos pentesters siguen **hábitos personales**, pero eso no constituye una metodología formal.

La enumeración es dinámica, pero necesitamos un marco que permita adaptarnos al entorno sin perder el rumbo. Para eso, HTB propone una **metodología de 6 capas**, dividida en tres niveles generales:

* **Infrastructure-Based Enumeration**
* **Host-Based Enumeration**
* **OS-Based Enumeration**

Estas capas representan "muros" o límites que debemos atravesar para acercarnos al objetivo.

---

## 2. Metodología en 6 Capas (Layers)

La metodología utiliza **capas concéntricas** que representan barreras a superar. Cada capa contiene información específica que debemos identificar.

<img width="1248" height="701" alt="image" src="https://github.com/user-attachments/assets/97464bd2-95df-4037-a125-439cf96c324d" />

---

## 🟦 Capa 1: Internet Presence

**Objetivo:** identificar la presencia pública del objetivo.

Incluye:

* Dominios
* Subdominios
* vHosts
* ASN
* Netblocks
* IPs públicas
* Instancias cloud
* Controles de seguridad (Cloudflare, WAF)

> Es el primer mapa del terreno: todo lo que la organización expone hacia Internet.

---

## 🟩 Capa 2: Gateway

**Objetivo:** comprender las medidas defensivas y la posición del objetivo dentro de la red.

Componentes comunes:

* Firewalls
* DMZ
* IPS/IDS
* EDR
* Proxies
* NAC
* Segmentación de red
* VPN
* Cloudflare / WAF

> Aquí entendemos qué tan protegido está el objetivo y cómo podríamos evitar o analizar esas defensas.

---

## 🟨 Capa 3: Accessible Services

**Objetivo:** enumerar y comprender todos los servicios accesibles.

Datos clave:

* Tipo de servicio
* Funcionalidad
* Configuración
* Puerto
* Versión
* Interfaz expuesta

> Cada servicio está allí por una razón. Comprender esa razón es fundamental para una explotación efectiva.

Esta es la capa principal tratada en este módulo.

---

## 🟧 Capa 4: Processes

**Objetivo:** entender cómo los servicios procesan información.

Analizamos:

* PID
* Datos procesados
* Tareas
* Fuente
* Destino

> La comunicación interna entre procesos revela dependencias, flujos de datos y puntos débiles que no siempre son visibles externamente.

---

## 🟫 Capa 5: Privileges

**Objetivo:** comprender permisos, roles y privilegios.

Incluye:

* Grupos
* Usuarios
* Permisos
* Restricciones
* Variables de entorno

> Muchos fallos críticos provienen de configuraciones incorrectas o privilegios excesivos, especialmente en Active Directory.

---

## 🟥 Capa 6: OS Setup

**Objetivo:** estudiar el sistema operativo y su configuración interna.

Elementos clave:

* Tipo de sistema operativo
* Nivel de parches
* Configuración de red
* Archivos de configuración
* Archivos sensibles
* Variables del entorno del OS

> Esta capa revela la calidad del trabajo del equipo de IT y la postura interna de seguridad.

---

## 3. Metáfora del Laberinto

La metodología puede visualizarse como un **laberinto**. Cada capa es un “muro” con múltiples posibles entradas.

Los cuadrados en el diagrama representan **vulnerabilidades** (gaps).

<img width="1224" height="733" alt="image" src="https://github.com/user-attachments/assets/bdab5cab-80e7-4bf6-9c97-a2c45aa21720" />


Puntos clave:

* No todos los gaps llevan al interior.
* Se debe priorizar qué caminos explorar según el tiempo disponible.
* En un pentest siempre existe la posibilidad de más vulnerabilidades que no descubrimos.

La historia del ataque a [**SolarWinds**](https://www.rpc.senate.gov/policy-papers/the-solarwinds-cyberattack) muestra que un atacante con meses de estudio del entorno puede descubrir vectores que un pentest típico no llega a ver.

---

## 4. Aplicación Práctica: Pentest Externo Black Box

### **Capa 1: Internet Presence**
Encontramos todos los activos posibles.

* Identificamos dominios, subdominios, IPs públicas, ASN y netblocks asociados.
* Buscamos interfaces expuestas (web, VPN, portales, paneles) y huellas de infraestructura.
* El objetivo es definir la superficie real de ataque dentro del alcance permitido.

### **Capa 2: Gateway**
Descubrimos defensas y arquitectura.

* Entendemos cómo “se llega” al objetivo: perímetro, segmentación visible y puntos de entrada.
* Detectamos controles como WAF, proxies, balanceadores, firewalls, CDN y mecanismos anti-bot.
* El objetivo es anticipar restricciones, rutas de acceso y comportamiento esperado del perímetro.

### **Capa 3: Accessible Services**
Analizamos servicios, funciones y configuraciones.

* Enumeramos servicios expuestos (puertos, protocolos, versiones) y su propósito.
* Revisamos configuraciones, endpoints, banners, autenticación y posibles misconfigurations.
* El objetivo es comprender cómo comunicarnos con cada servicio y qué vectores pueden derivar en impacto.

### **Capa 4: Processes**
Comprendemos tarea, origen y destino.

* Con acceso interno o ejecución en el host, observamos procesos y flujos reales de datos.
* Identificamos dependencias entre componentes (quién habla con quién, qué consume qué, y por qué).
* El objetivo es revelar rutas de datos, acoplamientos y puntos débiles que no son visibles desde el exterior.

### **Capa 5: Privileges**
Estudiamos permisos y usuarios.

* Determinamos con qué usuario/grupo corre cada servicio y qué privilegios efectivos posee.
* Buscamos permisos excesivos, delegaciones, credenciales expuestas, y oportunidades de escalada.
* El objetivo es entender qué acciones son posibles (y cuáles no) con los permisos actuales.

### **Capa 6: OS Setup**
Investigamos el sistema operativo y su configuración.

* Recolectamos información del OS: versión, parches, hardening, servicios internos y políticas.
* Identificamos configuraciones inseguras, secretos en archivos/configs, y controles defensivos activos.
* El objetivo es evaluar la postura interna y extraer información sensible útil para avanzar o demostrar impacto.

Cada capa nos acerca más al núcleo de la infraestructura.


---

## 5. Metodología en la Práctica

Una metodología no es una lista de comandos (eso es un **cheat sheet**).
Es un **marco sistemático** para explorar y comprender un objetivo.

El cómo se obtiene cada dato es dinámico: herramientas cambian, tecnologías evolucionan. Lo importante es:

* Seguir la estructura
* Adaptarse al contexto
* Mantener un pensamiento analítico

> La metodología guía. Las herramientas ejecutan.

---

