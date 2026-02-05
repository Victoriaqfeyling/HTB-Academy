# WHOIS

## ¿Qué es WHOIS?

**WHOIS** es un protocolo ampliamente utilizado de **consulta y respuesta**, diseñado para acceder a bases de datos que almacenan información sobre recursos registrados en Internet.

Está principalmente asociado a **nombres de dominio**, aunque también puede utilizarse para obtener información sobre:

* Bloques de direcciones IP
* Sistemas autónomos (AS)

Una forma simple de pensarlo es como una **guía telefónica de Internet**, que permite saber **quién es el dueño** o **responsable** de un determinado activo online.

---

## Uso básico de WHOIS

Durante un reconocimiento web, WHOIS suele ser una de las **primeras técnicas pasivas** que se ejecutan.

Ejemplo de consulta desde consola:

```bash
whois inlanefreight.com
```

Salida típica (fragmento):

```text
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
```



---

## Información que suele contener un registro WHOIS

Un registro WHOIS típico puede incluir los siguientes campos:

* **Domain Name**
  El nombre del dominio (ej: `example.com`).

* **Registrar**
  Empresa donde se registró el dominio (GoDaddy, Namecheap, Amazon Registrar, etc.).

* **Registrant Contact**
  Persona u organización que registró el dominio.

* **Administrative Contact**
  Responsable administrativo del dominio.

* **Technical Contact**
  Responsable de los aspectos técnicos del dominio.

* **Creation Date / Expiration Date**
  Fecha de creación del dominio y fecha de expiración.

* **Name Servers**
  Servidores DNS encargados de resolver el dominio a direcciones IP.

> ⚠️ En muchos casos, parte de esta información puede estar **oculta o anonimizada** mediante servicios de *privacy protection*.

---

## Breve historia de WHOIS

La historia de WHOIS está estrechamente ligada a **Elizabeth Feinler**, una científica informática clave en los inicios de Internet.

Durante la década de 1970, Feinler y su equipo en el **Network Information Center (NIC)** del *Stanford Research Institute* detectaron la necesidad de un sistema que permitiera **registrar y administrar los recursos de red** del ARPANET, precursor de Internet.

La solución fue la creación del **directorio WHOIS**, una base de datos simple pero revolucionaria que almacenaba información sobre:

* Usuarios de red
* Hostnames
* Nombres de dominio

Este sistema sentó las bases para los mecanismos de registro y consulta que aún hoy se utilizan.

---

Formalización y estandarización

A medida que Internet comenzó a expandirse más allá de sus orígenes académicos, se volvió evidente la necesidad de formalizar y estandarizar el protocolo WHOIS.

En 1982, WHOIS fue estandarizado mediante la RFC 812, lo que sentó las bases para un sistema más estructurado, documentado y escalable para la gestión de:

Registros de dominios

Información técnica asociada

Mecanismos de consulta y respuesta

Durante esta etapa, Ken Harrenstien y Vic White, ambos integrantes del Network Information Center (NIC), jugaron un rol clave en la definición formal del protocolo WHOIS y en cómo debía funcionar su interacción cliente-servidor.

Esta estandarización permitió que WHOIS dejara de ser una solución ad-hoc y pasara a convertirse en un componente fundamental de la infraestructura de Internet.

El auge del WHOIS distribuido y los RIR

Con el crecimiento exponencial de Internet durante la década de 1990, el modelo centralizado de WHOIS comenzó a mostrar claras limitaciones:

Falta de escalabilidad

Cuellos de botella administrativos

Dificultades para gestionar recursos a nivel global

Como respuesta, surgieron los Registros Regionales de Internet (RIR), marcando la transición hacia un modelo de WHOIS distribuido.

Figuras clave como Randy Bush y John Postel contribuyeron significativamente al desarrollo de este sistema, que dividió la responsabilidad de gestión de recursos de Internet en regiones geográficas.

Gracias a esta descentralización:

Se mejoró la escalabilidad del sistema

Aumentó la resiliencia operativa

WHOIS pudo acompañar el crecimiento acelerado de Internet

Este modelo distribuido es la base del funcionamiento actual de WHOIS para direcciones IP y sistemas autónomos.

ICANN y la modernización de WHOIS

La creación de la Internet Corporation for Assigned Names and Numbers (ICANN) en 1998 marcó un punto de inflexión en la evolución de WHOIS.

ICANN asumió la responsabilidad de:

La gestión global del DNS

El desarrollo de políticas relacionadas con WHOIS

La coordinación entre registradores y registros

Vint Cerf, considerado uno de los “padres de Internet”, tuvo un papel destacado en el establecimiento de la ICANN y en la consolidación de este nuevo modelo de gobernanza.

Bajo la supervisión de ICANN se lograron varios avances importantes:

Mayor estandarización de los formatos de datos WHOIS

Mejora en la precisión de la información

Resolución de disputas relacionadas con dominios

Entre estas disputas se incluyen casos de:

Ciberocupación (cybersquatting)

Infracción de marcas registradas

Conflictos por dominios no utilizados

Para abordar estos conflictos, ICANN definió la Política Uniforme de Resolución de Disputas de Nombres de Dominio (UDRP), que establece un marco de resolución mediante arbitraje.

Privacidad y la era del GDPR

En el siglo XXI, la exposición pública de datos WHOIS comenzó a generar crecientes preocupaciones de privacidad.

La disponibilidad abierta de información personal como:

Nombres

Direcciones físicas

Números de teléfono

se convirtió en un problema relevante, especialmente para individuos y pequeñas organizaciones.

Como consecuencia, se popularizaron los servicios de privacidad, que permiten ocultar o anonimizar los datos del registrante real.

La implementación del General Data Protection Regulation (GDPR) en 2018 aceleró este cambio, imponiendo estrictas obligaciones legales a los operadores de WHOIS en materia de protección de datos personales.

Desde entonces, gran parte de la información histórica que antes era accesible públicamente se encuentra limitada o redaccionada.

El presente y el futuro de WHOIS

Actualmente, WHOIS continúa evolucionando para adaptarse a un entorno donde existe una tensión constante entre:

Transparencia (necesaria para la seguridad, investigación y cumplimiento)

Privacidad (derecho fundamental de los individuos)

Uno de los principales esfuerzos para equilibrar ambos aspectos es el desarrollo del Registration Data Access Protocol (RDAP).

RDAP propone:

Acceso más granular a los datos

Mejor control de permisos

Un enfoque más moderno y respetuoso con la privacidad

Este protocolo representa el camino evolutivo natural de WHOIS y será cada vez más relevante en tareas de reconocimiento web y análisis de infraestructura.

Conclusión

La evolución de WHOIS refleja la propia historia de Internet: desde un entorno académico y abierto, hacia una infraestructura global, regulada y consciente de la privacidad.

Para el Web Reconnaissance, entender esta evolución es clave para:

Interpretar correctamente la información disponible

Comprender por qué ciertos datos ya no son públicos

Saber cuándo y cómo complementar WHOIS con otras técnicas pasivas

Este contexto histórico permite utilizar WHOIS de forma más efectiva y realista dentro de un pentest moderno.

---

## ¿Por qué WHOIS es importante para el Web Recon?

Desde la perspectiva del **pentesting**, WHOIS es una fuente de información extremadamente valiosa durante la fase de reconocimiento.

### Identificación de personal clave

Los registros WHOIS pueden revelar:

* Nombres de personas
* Direcciones de correo electrónico
* Teléfonos de contacto

Esta información puede utilizarse para:

* Ingeniería social
* Campañas de phishing dirigidas
* Identificación de roles clave dentro de la organización

---

### Descubrimiento de infraestructura de red

Campos como *Name Servers* o direcciones IP permiten inferir:

* Proveedores utilizados
* Estructura básica de DNS
* Posibles puntos de entrada o configuraciones débiles

Esta información suele cruzarse luego con técnicas como **DNS enumeration** o **fingerprinting de servicios**.

---

### Análisis histórico

Mediante servicios de terceros (por ejemplo, historiales de WHOIS), es posible observar:

* Cambios de propietario
* Modificaciones en datos de contacto
* Evolución de la infraestructura técnica

Este análisis puede revelar **reorganizaciones internas**, **migraciones de infraestructura** o prácticas de seguridad deficientes.
[Whoisfreaks](https://whoisfreaks.com/)

---

## Conclusión

WHOIS es una técnica **pasiva, sigilosa y de bajo riesgo**, ideal para iniciar el reconocimiento web. Aunque por sí sola no suele revelar vulnerabilidades explotables, proporciona **contexto crítico** que potencia el resto de las técnicas de Information Gathering.

Comprender WHOIS permite construir una base sólida sobre la cual avanzar hacia métodos más avanzados de reconocimiento web.
