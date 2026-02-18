# Robots.txt

Imagina que eres invitado a una gran fiesta en una mansión. Puedes recorrer la casa libremente, pero algunas habitaciones están marcadas como "Privado" y se espera que no entres. De forma similar, el archivo **robots.txt** funciona como una guía de etiqueta para bots en el mundo web, indicando qué áreas pueden explorar y cuáles deberían evitar.

---

# ¿Qué es robots.txt?

Técnicamente, **robots.txt** es un archivo de texto simple ubicado en el directorio raíz de un sitio web.

Ejemplo:

```
https://www.example.com/robots.txt
```

Este archivo sigue el estándar conocido como **Robots Exclusion Standard**, que define cómo deben comportarse los crawlers al visitar un sitio web.

Contiene instrucciones llamadas **directivas**, que indican qué partes del sitio pueden o no pueden ser rastreadas.

---

# ¿Cómo funciona robots.txt?

Las directivas dentro de robots.txt suelen dirigirse a **user-agents**, que identifican distintos tipos de bots.

Ejemplo básico:

```
User-agent: *
Disallow: /private/
```

Esto significa:

* `User-agent: *` → Aplica a todos los bots.
* `Disallow: /private/` → No pueden acceder a URLs que comiencen con `/private/`.

También pueden:

* Permitir acceso a ciertas rutas.
* Definir retrasos entre solicitudes.
* Proporcionar enlaces a sitemaps.

---

# Estructura de robots.txt

El archivo robots.txt es un documento de texto plano ubicado en la raíz del sitio.

Está compuesto por bloques llamados "records", separados por líneas en blanco.

Cada bloque contiene:

## 1️⃣ User-agent

Especifica el bot al que aplican las reglas.

Ejemplos:

```
User-agent: *
User-agent: Googlebot
User-agent: Bingbot
```

---

## 2️⃣ Directivas

Son instrucciones específicas para el user-agent definido.

### Directivas comunes

| Directive   | Description                                                                   | Example                                                                             |
| ----------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Disallow    | Especifica rutas que el bot no debe rastrear.                                 | Disallow: /admin/                                                                   |
| Allow       | Permite explícitamente ciertas rutas incluso si una regla Disallow las cubre. | Allow: /public/                                                                     |
| Crawl-delay | Define el tiempo de espera entre solicitudes.                                 | Crawl-delay: 10                                                                     |
| Sitemap     | Indica la URL del sitemap XML.                                                | Sitemap: [https://www.example.com/sitemap.xml](https://www.example.com/sitemap.xml) |

---

# ¿Por qué respetar robots.txt?

Aunque robots.txt no es técnicamente obligatorio (un bot malicioso podría ignorarlo), los crawlers legítimos sí lo respetan.

Respetarlo es importante por varias razones:

## Evitar sobrecargar el servidor

Limitar el acceso de bots puede prevenir tráfico excesivo que degrade el rendimiento.

## Proteger información sensible

Puede evitar que información privada sea indexada por motores de búsqueda.

## Cumplimiento legal y ético

Ignorar robots.txt podría violar términos de servicio o implicar problemas legales si se accede a datos privados o protegidos.

---

# robots.txt en Web Reconnaissance

Desde la perspectiva ofensiva, robots.txt es una fuente valiosa de inteligencia.

---

## 🔎 Descubrir directorios ocultos

Las rutas en `Disallow` suelen señalar:

* Paneles administrativos
* Directorios privados
* Backups
* Recursos sensibles

Paradójicamente, lo que el administrador quiere ocultar a los buscadores puede convertirse en un punto de interés para un atacante.

---

## 🗺 Mapear la estructura del sitio

El análisis de rutas permitidas y denegadas ayuda a:

* Entender la organización interna.
* Detectar secciones no enlazadas desde el menú principal.

---

## 🪤 Detectar trampas (Honeypots)

Algunos sitios incluyen rutas falsas en robots.txt para atraer bots maliciosos.

Identificar estas rutas puede indicar:

* Nivel de madurez defensiva.
* Presencia de mecanismos de monitoreo.

---

# Ejemplo de robots.txt

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10

Sitemap: https://www.example.com/sitemap.xml
```

### Análisis del ejemplo

* Todos los bots no pueden acceder a `/admin/` y `/private/`.
* Todos los bots pueden acceder a `/public/`.
* Googlebot debe esperar 10 segundos entre solicitudes.
* Se proporciona un sitemap para facilitar el rastreo.

---

# Inferencias desde Recon

A partir de este robots.txt podemos inferir:

* Existe un posible panel administrativo en `/admin/`.
* Hay contenido privado en `/private/`.
* Existe una estructura pública diferenciada (`/public/`).

Este tipo de información puede orientar etapas posteriores como:

* Directory enumeration
* Acceso manual a rutas interesantes
* Análisis de configuración

---

# Conclusión

robots.txt es un archivo simple pero extremadamente informativo.

En reconocimiento web permite:

* Descubrir rutas interesantes.
* Comprender la estructura interna.
* Detectar posibles configuraciones inseguras.
* Obtener pistas sobre recursos sensibles.

Aunque está diseñado como guía para crawlers legítimos, desde la perspectiva de seguridad puede revelar información estratégica clave sobre la superficie de ataque del objetivo.
