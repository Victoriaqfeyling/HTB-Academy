# Well-Known URIs

El estándar **.well-known**, definido en el [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615), establece un directorio estandarizado dentro del dominio raíz de un sitio web. Este directorio, accesible normalmente a través de la ruta:

```
https://example.com/.well-known/
```

centraliza metadatos críticos del sitio, incluyendo archivos de configuración e información relacionada con servicios, protocolos y mecanismos de seguridad.

---

# ¿Por qué existe .well-known?

El objetivo principal es proporcionar una ubicación predecible y estandarizada para que:

* Navegadores
* Aplicaciones
* Clientes OAuth
* Herramientas de seguridad

puedan descubrir automáticamente archivos de configuración importantes.

Por ejemplo, si un cliente quiere acceder a la política de seguridad del sitio:

```
https://example.com/.well-known/security.txt
```

Este enfoque elimina la necesidad de "adivinar" rutas.

---

# Registro Oficial (IANA)

La **Internet Assigned Numbers Authority (IANA)** mantiene un [registro](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) oficial de URIs .well-known.

Cada sufijo tiene un propósito específico definido por estándares o especificaciones.

Algunos ejemplos relevantes:

| URI Suffix           | Descripción                                            | Estado      | Referencia                 |
| -------------------- | ------------------------------------------------------ | ----------- | -------------------------- |
| security.txt         | Información de contacto para reportar vulnerabilidades | Permanente  | RFC 9116                   |
| change-password      | URL estándar para cambio de contraseña                 | Provisional | [W3C Spec](https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri)                   |
| openid-configuration | Configuración de OpenID Connect   (oAuth 2.0)                      | Permanente  | [OpenID Connect Discovery](http://openid.net/specs/openid-connect-discovery-1_0.html)    |
| assetlinks.json      | Verificación de propiedad de activos digitales         | Permanente  | [Google Digital Asset Links](https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md)  |
| mta-sts.txt          | Política de MTA-STS para seguridad de email            | Permanente  | RFC 8461                   |

Cada entrada define cómo debe implementarse el recurso.

---

# .well-known en Web Recon

Desde la perspectiva de reconnaissance ofensivo, los endpoints .well-known pueden revelar:

* Configuración de autenticación
* Políticas de seguridad
* Infraestructura de identidad
* Información criptográfica
* Integraciones externas

Uno de los endpoints más valiosos es:

```
/.well-known/openid-configuration
```

---

# OpenID Connect Discovery

El endpoint:

```
https://example.com/.well-known/openid-configuration
```

forma parte del protocolo **OpenID Connect Discovery**, una capa de identidad construida sobre OAuth 2.0.

Cuando se consulta, devuelve un documento JSON con metadatos del proveedor de identidad.

Ejemplo:

```json
{
  "issuer": "https://example.com",
  "authorization_endpoint": "https://example.com/oauth2/authorize",
  "token_endpoint": "https://example.com/oauth2/token",
  "userinfo_endpoint": "https://example.com/oauth2/userinfo",
  "jwks_uri": "https://example.com/oauth2/jwks",
  "response_types_supported": ["code", "token", "id_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email"]
}
```

---

# Información Extraíble

A partir de este endpoint se pueden identificar múltiples vectores de análisis.

## 🔎 Descubrimiento de Endpoints

* Authorization Endpoint → URL para solicitudes de autorización.
* Token Endpoint → Emisión de tokens.
* Userinfo Endpoint → Información del usuario autenticado.
* JWKS URI → Conjunto de claves públicas usadas para firmar tokens.

---

## 🔐 Análisis Criptográfico

El campo:

```
"id_token_signing_alg_values_supported": ["RS256"]
```

permite:

* Identificar algoritmos soportados.
* Detectar posibles configuraciones débiles.
* Evaluar riesgo de ataques como algorithm confusion.

---

## 🎯 Scopes y Response Types

El campo:

```
"scopes_supported"
```

indica qué permisos pueden solicitarse.

Esto ayuda a:

* Mapear capacidades del sistema.
* Identificar posibles abusos de privilegios.

---

# Oportunidades en Recon

Explorar el directorio `.well-known/` puede revelar:

* security.txt → Contactos internos y estructura organizacional.
* change-password → Endpoints directos para manipulación de credenciales.
* mta-sts.txt → Configuración de seguridad de correo.
* assetlinks.json → Relación con aplicaciones móviles.

Estos recursos permiten:

* Mapear infraestructura.
* Identificar integraciones externas.
* Descubrir servicios asociados.
* Comprender la arquitectura de autenticación.

---

# Enfoque Metodológico

Durante un pentest, una práctica recomendada es:

1. Enumerar automáticamente `.well-known/`.
2. Consultar el registro IANA para identificar posibles endpoints.
3. Analizar respuestas JSON en busca de:

   * Endpoints adicionales.
   * Claves públicas.
   * Configuraciones expuestas.
4. Correlacionar con hallazgos de fingerprinting y crawling.

---

# Conclusión

El estándar **.well-known** proporciona acceso estructurado a metadatos críticos del sitio.

En web reconnaissance representa:

* Un punto de enumeración altamente informativo.
* Una fuente de descubrimiento de endpoints.
* Una ventana directa a configuraciones de autenticación y seguridad.

Cuando se combina con crawling, fingerprinting y análisis de headers, permite construir un mapa detallado del ecosistema de seguridad del objetivo.
