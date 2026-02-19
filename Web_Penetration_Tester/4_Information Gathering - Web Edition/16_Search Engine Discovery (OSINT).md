# Search Engine Discovery (OSINT)

Los motores de búsqueda son nuestras guías en el vasto ecosistema de Internet. Sin embargo, más allá de responder consultas cotidianas, también almacenan una enorme cantidad de información indexada que puede ser extremadamente valiosa durante la fase de reconocimiento web.

Esta práctica se conoce como **Search Engine Discovery** u **OSINT (Open Source Intelligence)**.

Consiste en utilizar motores de búsqueda como herramientas estratégicas para descubrir información sobre:

* Sitios web objetivo
* Organizaciones
* Empleados
* Infraestructura tecnológica
* Documentos públicos

---

# ¿Por qué es importante en Web Recon?

## 🌍 Open Source

La información es públicamente accesible.
No requiere interacción directa con el objetivo.

---

## 📚 Amplitud de Información

Los motores de búsqueda indexan una gran porción de la web, incluyendo:

* Documentos PDF
* Backups expuestos
* Páginas olvidadas
* Subdominios

---

## 🧩 Facilidad de Uso

No requiere herramientas complejas ni explotación activa.

---

## 💰 Costo Cero

Es completamente gratuito.

---

# Aplicaciones en Seguridad

La información obtenida puede utilizarse para:

* Security Assessment → Identificar datos expuestos.
* Competitive Intelligence → Analizar competidores.
* Threat Intelligence → Detectar amenazas emergentes.
* Investigación periodística.

⚠ Limitación: Los motores de búsqueda no indexan todo y algunos datos pueden estar protegidos o no ser accesibles públicamente.

---

# Search Operators

Los **Search Operators** son comandos especiales que permiten realizar búsquedas más precisas.

---

# Tabla de Operadores de Búsqueda

| Operador                | Descripción del Operador                                                          | Ejemplo                                           | Descripción del Ejemplo                                                     |
| ----------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------------------------- |
| site:                   | Limita los resultados a un sitio web o dominio específico.                        | site:example.com                                  | Encuentra todas las páginas públicas accesibles en example.com.             |
| inurl:                  | Busca páginas que contengan un término específico en la URL.                      | inurl:login                                       | Busca páginas de inicio de sesión en cualquier sitio web.                   |
| filetype:               | Busca archivos de un tipo específico.                                             | filetype:pdf                                      | Encuentra documentos PDF descargables.                                      |
| intitle:                | Busca páginas que contengan un término específico en el título.                   | intitle:"confidential report"                     | Busca documentos titulados "confidential report" o variaciones similares.   |
| intext: / inbody:       | Busca un término dentro del contenido de la página.                               | intext:"password reset"                           | Identifica páginas que contienen el término "password reset".               |
| cache:                  | Muestra la versión en caché de una página web (si está disponible).               | cache:example.com                                 | Visualiza la versión almacenada en caché de example.com.                    |
| link:                   | Encuentra páginas que enlazan a una página específica.                            | link:example.com                                  | Identifica sitios web que enlazan a example.com.                            |
| related:                | Encuentra sitios web relacionados con una página específica.                      | related:example.com                               | Descubre sitios similares a example.com.                                    |
| info:                   | Proporciona información básica sobre una página web.                              | info:example.com                                  | Muestra detalles generales como título y descripción.                       |
| define:                 | Proporciona definiciones de una palabra o frase.                                  | define:phishing                                   | Obtiene la definición de "phishing" desde distintas fuentes.                |
| numrange:               | Busca números dentro de un rango específico.                                      | site:example.com numrange:1000-2000               | Encuentra páginas en example.com que contengan números entre 1000 y 2000.   |
| allintext:              | Encuentra páginas que contengan todas las palabras especificadas en el contenido. | allintext:admin password reset                    | Busca páginas que contengan "admin" y "password reset" en el texto.         |
| allinurl:               | Encuentra páginas que contengan todas las palabras especificadas en la URL.       | allinurl:admin panel                              | Busca páginas que contengan "admin" y "panel" en la URL.                    |
| allintitle:             | Encuentra páginas que contengan todas las palabras especificadas en el título.    | allintitle:confidential report 2023               | Busca páginas que contengan "confidential", "report" y "2023" en el título. |
| AND                     | Reduce resultados requiriendo que todos los términos estén presentes.             | site:example.com AND (inurl:admin OR inurl:login) | Encuentra páginas de admin o login específicamente en example.com.          |
| OR                      | Amplía resultados incluyendo cualquiera de los términos.                          | "linux" OR "ubuntu" OR "debian"                   | Busca páginas que mencionen Linux, Ubuntu o Debian.                         |
| NOT                     | Excluye resultados que contengan el término especificado.                         | site:bank.com NOT inurl:login                     | Encuentra páginas en bank.com excluyendo páginas de login.                  |
| * (comodín)             | Representa cualquier palabra o carácter.                                          | site:socialnetwork.com filetype:pdf user* manual  | Busca manuales de usuario (user guide, user handbook) en PDF.               |
| .. (búsqueda por rango) | Encuentra resultados dentro de un rango numérico específico.                      | site:ecommerce.com "price" 100..500               | Busca productos con precios entre 100 y 500.                                |
| " " (comillas)          | Busca una frase exacta.                                                           | "information security policy"                     | Encuentra documentos que contengan exactamente esa frase.                   |
| - (signo menos)         | Excluye términos de los resultados de búsqueda.                                   | site:news.com -inurl:sports                       | Busca noticias en news.com excluyendo contenido deportivo.                  |


---

# Google Dorking

También conocido como **Google Hacking**, consiste en combinar operadores para descubrir información sensible.

---

[Source](https://www.exploit-db.com/google-hacking-database)

---

## 🔐 Encontrar páginas de login

```
site:example.com inurl:login
site:example.com (inurl:login OR inurl:admin)
```

---

## 📂 Identificar archivos expuestos

```
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:docx)
```

---

## ⚙ Descubrir archivos de configuración

```
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf)
```

---

## 🗄 Localizar backups de base de datos

```
site:example.com inurl:backup
site:example.com filetype:sql
```

---

# Valor Estratégico

Google Dorking permite:

* Detectar credenciales expuestas.
* Encontrar archivos sensibles indexados.
* Descubrir endpoints no enlazados.
* Mapear infraestructura indirectamente.

---

# Enfoque Profesional

Durante un pentest, una estrategia común es:

1. Enumerar dominios y subdominios con `site:`.
2. Buscar archivos sensibles con `filetype:`.
3. Buscar endpoints administrativos con `inurl:`.
4. Combinar operadores para reducir ruido.
5. Correlacionar hallazgos con crawling y fingerprinting.

---

# Conclusión

Search Engine Discovery es una técnica poderosa dentro del reconocimiento pasivo.

Permite:

* Obtener información sin interactuar directamente con el objetivo.
* Descubrir datos expuestos accidentalmente.
* Identificar vectores potenciales de ataque.

Cuando se combina con crawling, robots.txt y análisis de .well-known URIs, se convierte en una herramienta esencial para construir un mapa completo de la superficie de ataque.
