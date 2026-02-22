# Staff (OSINT de Personal)

## Objetivo

La búsqueda e identificación de **empleados** en redes sociales y plataformas profesionales (p. ej., **LinkedIn**, **Xing**) es una técnica OSINT que permite inferir información valiosa sobre:

* **Estructura del equipo** (roles, seniority, áreas)
* **Tecnologías usadas** (lenguajes, frameworks, bases de datos)
* **Arquitecturas** (microservicios, REST, CI/CD)
* **Herramientas internas** (Atlassian Suite, repos, sistemas de tickets)
* **Medidas de seguridad** (por el perfil del equipo de seguridad y sus prácticas)

Además, el contenido que un empleado publica (posts, repositorios, charlas, material compartido) suele reflejar:

* En qué está trabajando
* Qué considera relevante
* Qué herramientas domina
* Qué decisiones técnicas (o errores) pueden terminar expuestos

> **Nota**: Todo lo mencionado se basa en **reconocimiento pasivo**. No se realizan acciones intrusivas ni acceso no autorizado.

---

# Fuentes principales

## 1) Redes profesionales

* **LinkedIn**: perfiles, experiencia laboral, skills, proyectos, publicaciones
* **Xing**: equivalente en algunos países/mercados

## 2) Ofertas laborales

Los **job postings** son una fuente especialmente útil para inferir **stack tecnológico** y prácticas internas, ya que detallan habilidades requeridas, herramientas y metodologías de trabajo.

---

# Análisis de Job Post (LinkedIn)


```
Required Skills/Knowledge/Experience:

* 3-10+ years of experience on professional software development projects.

« An active US Government TS/SCI Security Clearance (current SSBI) or eligibility to obtain TS/SCI within nine months.
« Bachelor's degree in computer science/computer engineering with an engineering/math focus or another equivalent field of discipline.
« Experience with one or more object-oriented languages (e.g., Java, C#, C++).
« Experience with one or more scripting languages (e.g., Python, Ruby, PHP, Perl).
« Experience using SQL databases (e.g., PostgreSQL, MySQL, SQL Server, Oracle).
« Experience using ORM frameworks (e.g., SQLAIchemy, Hibernate, Entity Framework).
« Experience using Web frameworks (e.g., Flask, Django, Spring, ASP.NET MVC).
« Proficient with unit testing and test frameworks (e.g., pytest, JUnit, NUnit, xUnit).
« Service-Oriented Architecture (SOA)/microservices & RESTful API design/implementation.
« Familiar and comfortable with Agile Development Processes.
« Familiar and comfortable with Continuous Integration environments.
« Experience with version control systems (e.g., Git, SVN, Mercurial, Perforce).

Desired Skills/Knowledge/ Experience:

« CompTIA Security+ certification (or equivalent).
« Experience with Atlassian suite (Confluence, Jira, Bitbucket).
« Algorithm Development (e.g., Image Processing algorithms).
« Software security.
« Containerization and container orchestration (Docker, Kubernetes, etc.)
« Redis.
« NumPy.
```


Ejemplo de habilidades requeridas (extracto):

* Lenguajes OOP: **Java, C#, C++**
* Lenguajes scripting: **Python, Ruby, PHP, Perl**
* Bases de datos: **PostgreSQL, MySQL, SQL Server, Oracle**
* ORMs: **SQLAlchemy, Hibernate, Entity Framework**
* Frameworks Web: **Flask, Django, Spring, ASP.NET MVC**
* Testing: **pytest, JUnit, NUnit, xUnit**
* Arquitectura: **SOA / microservicios / REST APIs**
* Metodologías: **Agile**
* Integración Continua: **CI**
* Control de versiones: **Git, SVN, Mercurial, Perforce**

Skills deseables:

* **CompTIA Security+**
* **Atlassian Suite (Confluence, Jira, Bitbucket)**
* **Software security**
* **Docker / Kubernetes**
* **Redis**
* **NumPy**

## Qué podemos inferir a partir de esto

A partir de una sola oferta, podemos inferir con bastante precisión:

1. **Stack de desarrollo preferido**

   * Lenguajes dominantes (Java/.NET/Python)
   * Frameworks web usados y probables rutas/estructuras típicas

2. **Backends y datos**

   * DBs utilizadas (Postgres/MySQL/Oracle/etc.)
   * Uso de ORMs → posibilidad de patrones de código comunes

3. **Arquitectura y despliegue**

   * Microservicios + REST → presencia de APIs internas/externas
   * CI/CD → pipelines, artefactos, repositorios, runners

4. **Herramientas de colaboración**

   * Atlassian Suite → potencial exposición de:

     * Confluence (docs internas)
     * Jira (tickets)
     * Bitbucket (repos)

5. **Seguridad**

   * Si se pide Security+ y “software security”, hay señales de prácticas mínimas de seguridad y roles orientados a AppSec.

---

# Análisis de perfiles de empleados

## LinkedIn – Employee #1 (About)

<img width="961" height="254" alt="image" src="https://github.com/user-attachments/assets/5cb4d1a4-76c7-430d-a36b-e8106a9e0222" />


En descripciones técnicas se suelen encontrar pistas sobre:

* Tecnologías web: **W3C specs, Web Components, React, Svelte, AngularJS**
* Enlaces a repositorios: **GitHub** con proyectos OSS

### ¿Por qué esto es útil en recon?

* Permite inferir el **ecosistema frontend** real
* Un GitHub vinculado puede revelar:

  * Repos públicos
  * Herramientas usadas
  * Estilo de desarrollo
  * Posibles errores operativos
 

Por ejemplo, si buscamos brevemente errores de configuración de seguridad en Django, encontraremos el siguiente [repositorio de Github](https://github.com/boomcamp/django-security) que describe el Top 10 de OWASP para Django. Podemos usarlo para comprender la estructura interna de Django y su funcionamiento. Las mejores prácticas también suelen indicarnos qué buscar, ya que muchos confían ciegamente en ellas e incluso nombran muchos de los archivos como se muestra en las instrucciones.


---

## Riesgo: sobreexposición por “portafolio público”

Publicar proyectos puede ser una ventaja profesional, pero también introduce riesgo cuando se cometen errores, por ejemplo:

* Emails personales expuestos en código o metadata
* Tokens o secrets hardcodeados
* Configuraciones inseguras

### Ejemplo conceptual del curso

<img width="962" height="285" alt="image" src="https://github.com/user-attachments/assets/d247670c-76b6-4f6f-b0b6-40bac0110d88" />

En el repositorio se detecta:

* Email personal en un archivo (metadata / author)
* Un **JWT hardcodeado** en el código

> Esto puede derivar en reutilización de credenciales, pivoteo hacia otros servicios, o abuso de tokens si el mismo patrón se replica en entornos reales.

---

## LinkedIn – Employee #2 (Career)

Los historiales de carrera también aportan señales fuertes:

<img width="1250" height="829" alt="image" src="https://github.com/user-attachments/assets/0ffb409f-3f72-46e0-acee-9297548ab379" />


* Seniority (ej. VP Software Engineer)
* Proyectos (apps móviles CRM, sistemas internos)
* Tecnologías listadas: **Java, React, Elastic, Kafka**, etc.

### ¿Qué inferimos?

* Presencia probable de:

  * **Sistemas de mensajería/eventos** (Kafka)
  * **Búsqueda/observabilidad** (Elastic)
  * **Aplicaciones internas** (CRM, brokers, etc.)

---

# Cómo buscar dentro de LinkedIn (estratégico)

LinkedIn permite filtrar por:

* Conexiones
* Ubicación
* Empresa
* Universidad
* Industria
* Idioma del perfil
* Servicios
* Nombre
* Título

> Cuanto más específicos sean los filtros, menos resultados obtendremos. Hay que definir primero el objetivo de búsqueda.

### Si el objetivo es inferir stack e infraestructura

Priorizar perfiles de:

* Desarrollo (backend/frontend)
* DevOps/SRE
* Seguridad (AppSec, SecOps, SOC)

Porque:

* Los developers revelan **stack y patrones de implementación**
* DevOps/SRE revelan **CI/CD, contenedores, cloud, observabilidad**
* Seguridad revela **controles existentes y madurez** (WAF, SAST, SIEM, hardening, etc.)

---

# Conclusiones

1. Los empleados son una fuente OSINT valiosa para inferir tecnologías y estructura interna.
2. Los job posts revelan stacks completos (lenguajes, DBs, frameworks, herramientas, metodología).
3. Perfiles técnicos con GitHub pueden exponer patrones de desarrollo y errores (emails, tokens, secretos).
4. La búsqueda debe hacerse con un propósito claro (stack, madurez, medidas defensivas) para no generar ruido.

---

# Aplicación en un Pentest (Recon Pasivo)

Durante el reconocimiento, este enfoque ayuda a:

* Preparar hipótesis realistas del stack
* Enfocar la búsqueda de vulnerabilidades (según frameworks/tecnologías)
* Identificar proveedores y herramientas (Atlassian, CI/CD)
* Encontrar potenciales exposiciones públicas de info sensible

> **Resultado esperado**: un mapa inicial de tecnologías, equipos y superficie de ataque, sin interacción directa con los sistemas del objetivo.
