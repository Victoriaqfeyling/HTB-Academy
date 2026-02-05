# Introducción — Web Reconnaissance

## Contexto

El **Web Reconnaissance** (reconocimiento web) es la base de una evaluación de seguridad completa. Esta etapa consiste en recolectar información de forma **sistemática y meticulosa** sobre un sitio web o una aplicación web objetivo.

Puede pensarse como la **fase preparatoria** del pentest: antes de intentar explotar vulnerabilidades, es fundamental entender *qué expone el objetivo, cómo está construido y qué información deja ver hacia el exterior*.

Dentro del **Proceso de Penetration Testing**, el Web Reconnaissance forma parte de la fase de **Information Gathering**, y condiciona directamente la calidad de todas las etapas posteriores.

---

## Proceso de Penetration Testing

De manera general, un pentest suele seguir el siguiente flujo:

1. **Pre-Engagement**
   Definición de alcance, reglas de compromiso y objetivos.

2. **Information Gathering**
   Recolección de información sobre el objetivo (fase donde se ubica el Web Reconnaissance).

3. **Vulnerability Assessment**
   Identificación y análisis de vulnerabilidades potenciales.

4. **Exploitation**
   Explotación controlada de las vulnerabilidades detectadas.

5. **Post-Exploitation**
   Evaluación del impacto real tras la explotación.

6. **Lateral Movement**
   Movimiento lateral dentro de la infraestructura comprometida.

7. **Proof of Concept (PoC)**
   Demostración controlada de los hallazgos.

8. **Post-Engagement**
   Reporte final, conclusiones y recomendaciones.

<img width="1689" height="638" alt="image" src="https://github.com/user-attachments/assets/c72380b9-0360-4384-8afc-0c29e1bd0bdc" />


---

## Objetivos del Web Reconnaissance

El reconocimiento web persigue varios objetivos clave:

### Identificación de activos

Consiste en descubrir todos los componentes **públicamente accesibles** del objetivo, tales como:

* Páginas web
* Subdominios
* Direcciones IP
* Tecnologías utilizadas (servidores web, frameworks, lenguajes, CMS, etc.)

Este paso permite construir una **visión global de la presencia online** del objetivo.

---

### Descubrimiento de información oculta

Busca identificar información sensible expuesta de forma involuntaria, por ejemplo:

* Archivos de backup
* Archivos de configuración
* Documentación interna

Este tipo de hallazgos suele brindar **pistas valiosas** sobre la arquitectura interna y posibles vectores de ataque.

---

### Análisis de la superficie de ataque

En esta etapa se evalúa qué tan amplia y compleja es la **superficie de ataque**, analizando:

* Tecnologías utilizadas
* Configuraciones visibles
* Puntos de entrada potenciales

Cuanto mayor sea la superficie expuesta, mayor será la probabilidad de encontrar vulnerabilidades.

---

### Recolección de inteligencia

Se obtiene información que puede reutilizarse en etapas posteriores, como:

* Direcciones de correo electrónico
* Personal clave de la organización
* Patrones de comportamiento

Esta inteligencia puede utilizarse tanto para explotación técnica como para **ataques de ingeniería social**.

---

## Uso ofensivo y defensivo del reconocimiento

* **Desde la perspectiva del atacante**, la información recolectada permite personalizar los ataques, apuntar a debilidades específicas y evadir controles de seguridad.
* **Desde la perspectiva defensiva**, el reconocimiento sirve para identificar exposiciones innecesarias y corregirlas antes de que puedan ser explotadas.

---

## Tipos de Reconnaissance

El Web Reconnaissance se divide en dos metodologías fundamentales:

* **Reconocimiento Activo**
* **Reconocimiento Pasivo**

Comprender sus diferencias es clave para recolectar información de forma efectiva y controlada.

---

## Reconocimiento Activo

En el **reconocimiento activo**, el atacante interactúa directamente con el sistema objetivo para obtener información. Esta interacción puede ser detectada por mecanismos de seguridad.

Algunas técnicas comunes incluyen:

* **Port Scanning**: identificación de puertos y servicios abiertos. Herramientas: Nmap, Masscan, Unicornscan.
* **Vulnerability Scanning**: búsqueda de vulnerabilidades conocidas.Herramientas: Nessus, OpenVas, Nikto.
* **Network Mapping**: mapeo de la topología de red. Herramientas: Traceroute, Nmap.
* **Banner Grabbing**: obtención de información desde banners de servicios. Herramientas: Netcat, Curl.
* **OS Fingerprinting**: identificación del sistema operativo. Herramientas: Nmap, Xprobe2.
* **Service Enumeration**: detección de versiones de servicios. Herramientas: Nmap.
* **Web Spidering**: rastreo automático del sitio web. Herramientas: Burp Suite Spider, OWASP ZAP Spider, Scrapy.

> ⚠️ El reconocimiento activo suele ser más completo, pero implica un **mayor riesgo de detección**, ya que puede disparar alertas, IDS o firewalls.

---

## Reconocimiento Pasivo

El **reconocimiento pasivo** se basa en recolectar información **sin interactuar directamente** con el objetivo, utilizando únicamente fuentes públicas.

Ejemplos de técnicas pasivas:

* Consultas en motores de búsqueda
* Búsquedas WHOIS
* Análisis de registros DNS
* Revisión de archivos históricos (Wayback Machine)
* Análisis de redes sociales
* Revisión de repositorios de código público

> ✅ Esta metodología es **mucho más sigilosa**, aunque depende exclusivamente de la información ya disponible públicamente.

---

## Conclusión

El Web Reconnaissance es una etapa crítica que define el éxito del resto del pentest. Un reconocimiento bien realizado permite entender el objetivo en profundidad, reducir suposiciones y maximizar la efectividad de las fases posteriores.

En este módulo se comenzará analizando **WHOIS**, una de las técnicas pasivas fundamentales para obtener información sobre dominios, propietarios y la infraestructura digital asociada, sentando así las bases para métodos de reconocimiento más avanzados.
