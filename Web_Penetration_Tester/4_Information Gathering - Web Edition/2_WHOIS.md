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

---

## Conclusión

WHOIS es una técnica **pasiva, sigilosa y de bajo riesgo**, ideal para iniciar el reconocimiento web. Aunque por sí sola no suele revelar vulnerabilidades explotables, proporciona **contexto crítico** que potencia el resto de las técnicas de Information Gathering.

Comprender WHOIS permite construir una base sólida sobre la cual avanzar hacia métodos más avanzados de reconocimiento web.
