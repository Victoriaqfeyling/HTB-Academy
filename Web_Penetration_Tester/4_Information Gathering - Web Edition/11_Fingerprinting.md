# Fingerprinting

Fingerprinting se enfoca en extraer detalles técnicos sobre las tecnologías que impulsan un sitio web o aplicación web. De manera similar a cómo una huella digital identifica de forma única a una persona, las firmas digitales de servidores web, sistemas operativos y componentes de software pueden revelar información crítica sobre la infraestructura de un objetivo y sus posibles debilidades de seguridad.

Este conocimiento permite a un atacante adaptar sus ataques y explotar vulnerabilidades específicas de las tecnologías identificadas.

---

## ¿Por qué el Fingerprinting es clave en Web Recon?

### Targeted Attacks

Conocer las tecnologías específicas en uso permite enfocar los esfuerzos en exploits y vulnerabilidades que afectan directamente a esos sistemas. Esto incrementa significativamente las probabilidades de comprometer el objetivo.

### Identifying Misconfigurations

El fingerprinting puede exponer:

* Software desactualizado
* Configuraciones por defecto
* Headers inseguros
* Componentes mal configurados

Muchas de estas debilidades no son evidentes mediante otras técnicas de reconocimiento.

### Prioritising Targets

Cuando existen múltiples objetivos potenciales, el fingerprinting ayuda a priorizar aquellos que:

* Son más propensos a ser vulnerables
* Ejecutan software obsoleto
* Pueden contener información de mayor valor

### Building a Comprehensive Profile

Al combinar datos de fingerprinting con otros hallazgos de reconocimiento (DNS, CT logs, subdominios, OSINT), se construye una visión integral de la infraestructura del objetivo y su postura de seguridad.

---

# Fingerprinting Techniques

Existen varias técnicas utilizadas para identificar servidores web y tecnologías:

## Banner Grabbing

Consiste en analizar los banners que presentan los servidores web y otros servicios. Estos banners suelen revelar:

* Software del servidor
* Número de versión
* Sistema operativo

---

## Analysing HTTP Headers

Los headers HTTP transmitidos en cada request y response contienen gran cantidad de información.

* `Server`: suele indicar el software del servidor web.
* `X-Powered-By`: puede revelar lenguajes de scripting o frameworks.

---

## Probing for Specific Responses

Enviar solicitudes especialmente diseñadas puede provocar respuestas únicas que revelen tecnologías o versiones específicas.

Ejemplo:

* Mensajes de error característicos
* Comportamientos particulares ante métodos HTTP no comunes

---

## Analysing Page Content

El contenido HTML puede proporcionar pistas sobre tecnologías subyacentes:

* Comentarios en el código
* Estructura específica del CMS
* Archivos como `license.txt`
* Scripts cargadosn y otros elementos.

Por ejemplo puede haber un encabezado de copyright que indique el software específico que se utiliza

---

# Herramientas de Fingerprinting

| Tool       | Description                                                             | Features                                                              |
| ---------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Wappalyzer | Extensión de navegador y servicio online para perfilar tecnologías web. | Identifica CMS, frameworks, herramientas de analítica y más.          |
| BuiltWith  | Perfilador tecnológico web con reportes detallados.                     | Ofrece planes gratuitos y pagos con distintos niveles de detalle.     |
| WhatWeb    | Herramienta CLI para fingerprinting web.                                | Usa una amplia base de firmas para identificar tecnologías.           |
| Nmap       | Escáner de red versátil.                                                | Permite fingerprinting de servicios y sistema operativo mediante NSE. |
| Netcraft   | Servicio de seguridad web.                                              | Reportes detallados sobre tecnología, hosting y postura de seguridad. |
| wafw00f    | Herramienta CLI para identificar WAFs.                                  | Detecta presencia y tipo de Web Application Firewall.                 |

---

# Fingerprinting inlanefreight.com

Aplicaremos técnicas manuales y automatizadas para identificar el stack tecnológico de `inlanefreight.com`.

---

## Banner Grabbing

```bash
curl -I inlanefreight.com
```

Resultado:

```
HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:07:44 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://inlanefreight.com/
Content-Type: text/html; charset=iso-8859-1
```

Observación:

* Servidor: Apache 2.4.41
* Sistema base: Ubuntu
* Redirección a HTTPS

---

```bash
curl -I https://inlanefreight.com
```

```
HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:12:12 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Redirect-By: WordPress
Location: https://www.inlanefreight.com/
Content-Type: text/html; charset=UTF-8
```

Observación:

* La redirección es gestionada por WordPress.

---

```bash
curl -I https://www.inlanefreight.com
```

```
HTTP/1.1 200 OK
Date: Fri, 31 May 2024 12:12:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8
```

Observación:

* Presencia de `wp-json` → Indicador claro de WordPress.

---

# Wafw00f

Antes de continuar con técnicas más agresivas, es importante detectar la presencia de un WAF.

Instalación:

```bash
pip3 install git+https://github.com/EnableSecurity/wafw00f
```

Ejecución:

```bash
wafw00f inlanefreight.com
```

Resultado:

```
                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://inlanefreight.com
[+] The site https://inlanefreight.com is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2
```

Conclusión:

* El sitio está protegido por Wordfence WAF.
* Puede filtrar o bloquear intentos de reconocimiento más agresivos.

---

# Nikto

Nikto es un escáner de servidores web que también realiza fingerprinting.

Instalación (si no está presente):

```bash
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl
```

Ejecución enfocada en identificación de software:

```bash
nikto -h inlanefreight.com -Tuning b


- Nikto v2.5.0
---------------------------------------------------------------------------
+ Multiple IPs found: 134.209.24.248, 2a03:b0c0:1:e0::32c:b001
+ Target IP:          134.209.24.248
+ Target Hostname:    www.inlanefreight.com
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=inlanefreight.com
                   Altnames: inlanefreight.com, www.inlanefreight.com
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2024-05-31 13:35:54 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: Link header found with value: ARRAY(0x558e78790248). See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link
+ /: The site uses TLS and the Strict-Transport-Security HTTP header is not defined. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /index.php?: Uncommon header 'x-redirect-by' found, with contents: WordPress.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /: The Content-Encoding header is set to "deflate" which may mean that the server is vulnerable to the BREACH attack. See: http://breachattack.com/
+ Apache/2.4.41 appears to be outdated (current is at least 2.4.59). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /license.txt: License file found may identify site software.
+ /: A Wordpress installation was found.
+ /wp-login.php?action=register: Cookie wordpress_test_cookie created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /wp-login.php:X-Frame-Options header is deprecated and has been replaced with the Content-Security-Policy HTTP header with the frame-ancestors directive instead. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /wp-login.php: Wordpress login found.
+ 1316 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2024-05-31 13:47:27 (GMT0) (693 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Hallazgos relevantes:

- `IPs`: El sitio web se resuelve en direcciones IPv4 (`134.209.24.248`) e IPv6 (`2a03:b0c0:1:e0::32c:b001`).
- `Server Technology`: El sitio web funciona en `Apache/2.4.41 (Ubuntu)`.
- `WordPress Presence`: El análisis identificó una instalación de `WordPress`, incluida la página de inicio de sesión (`/wp-login.php`). Esto sugiere que el sitio podría ser un objetivo potencial de exploits comunes relacionados con `WordPress`.
- `Information Disclosure`: La presencia de el archivo `license.txt` podría revelar detalles adicionales sobre los componentes de software del sitio web.
`Headers`: Se encontraron varios encabezados no estándar o inseguros, incluido un encabezado faltante y un encabezado potencialmente inseguro:`Strict-Transport-Security` y `x-redirect-by`.

---

# Conclusión

El fingerprinting permite:

* Identificar con precisión el stack tecnológico del objetivo.
* Detectar configuraciones inseguras.
* Priorizar vectores de ataque.
* Reducir la superficie de incertidumbre antes de la explotación.

En una metodología profesional de pentesting, esta fase es crítica para transformar un reconocimiento genérico en un ataque dirigido y eficiente.

---


# Preguntas

vHosts necesarios para estas preguntas:
- `app.inlanefreight.local`
- `dev.inlanefreight.local`

  Agregamos los vHosts al archivo /etc/hosts

#### Determinar la versión de Apache que se ejecuta en app.inlanefreight.local en el sistema de destino. (Formato: 0.0.0)

<img width="1230" height="333" alt="image" src="https://github.com/user-attachments/assets/19438ea6-c14d-4a82-98d0-d2c6458102e9" />


#### ¿Qué CMS se utiliza en app.inlanefreight.local en el sistema de destino? Responda solo con el nombre, p. ej., WordPress.

<img width="1911" height="211" alt="image" src="https://github.com/user-attachments/assets/a9a3853c-48d6-48ed-ac97-153f95c88d9b" />

#### ¿En qué sistema operativo se ejecuta el servidor web dev.inlanefreight.local en el sistema de destino? Responda solo con el nombre, p. ej., Debian.

