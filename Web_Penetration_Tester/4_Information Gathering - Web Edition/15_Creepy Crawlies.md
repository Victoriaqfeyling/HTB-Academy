# Creepy Crawlies

El mundo del web crawling es amplio y complejo, pero no es necesario recorrerlo manualmente. Existen múltiples herramientas que automatizan el proceso de crawling, haciéndolo más rápido y eficiente, permitiéndonos concentrarnos en el análisis de los datos extraídos.

---

# Popular Web Crawlers

## 🕷 Burp Suite Spider

Burp Suite incluye un crawler activo llamado **Spider**.

Características:

* Mapeo automático de aplicaciones web.
* Descubrimiento de contenido oculto.
* Integración directa con herramientas de testing.

---

## 🛡 OWASP ZAP (Zed Attack Proxy)

ZAP es una herramienta gratuita y open-source.

Características:

* Modo automático y manual.
* Spider integrado.
* Identificación de vulnerabilidades comunes.

---

## 🐍 Scrapy (Framework en Python)

Scrapy es un framework potente y flexible para crear crawlers personalizados.

Características:

* Extracción estructurada de datos.
* Manejo de escenarios complejos.
* Automatización de procesamiento.

Ideal para tareas de reconnaissance personalizadas.

---

## 🌐 Apache Nutch

Crawler open-source escalable escrito en Java.

Características:

* Diseñado para crawls masivos.
* Alta extensibilidad.
* Requiere mayor conocimiento técnico.

Más orientado a proyectos de gran escala.

---

# Ética y Responsabilidad

Siempre se debe:

* Obtener permiso antes de realizar crawling intensivo.
* Evitar sobrecargar el servidor.
* Respetar límites de velocidad y políticas del sitio.

---

# Scrapy en Acción

En este laboratorio utilizaremos **Scrapy** junto con un spider personalizado llamado **ReconSpider** para realizar reconnaissance sobre `inlanefreight.com`.

---

# Instalación de Scrapy

Si no está instalado:

```bash
pip3 install scrapy
```

En debian:
```bash
pipx install scrapy
```
Recomendación manual:
```bash
python3 -m venv recon-env
source recon-env/bin/activate
pip install scrapy
```

Esto instalará Scrapy y sus dependencias.

---

# Descargando ReconSpider

```bash
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
```

---

# Ejecutando el Spider

```bash
python3 ReconSpider.py http://inlanefreight.com
```

Reemplazar el dominio por el objetivo deseado.

El spider recorrerá el sitio y recolectará información estructurada.

---

# results.json

Tras la ejecución, se genera un archivo `results.json` con los datos extraídos.

Ejemplo de estructura:

```json
{
    "emails": ["lily.floid@inlanefreight.com"],
    "links": ["https://www.inlanefreight.com/index.php/offices/"],
    "external_files": ["https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf"],
    "js_files": ["https://www.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js"],
    "form_fields": [],
    "images": ["https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs.png"],
    "videos": [],
    "audio": [],
    "comments": ["<!-- #masthead -->"]
}
```

---

# Significado de cada clave

| JSON Key       | Descripción                                     |
| -------------- | ----------------------------------------------- |
| emails         | Direcciones de correo encontradas en el dominio |
| links          | URLs internas encontradas                       |
| external_files | Archivos externos como PDFs                     |
| js_files       | Archivos JavaScript utilizados                  |
| form_fields    | Campos de formularios detectados                |
| images         | URLs de imágenes                                |
| videos         | URLs de videos                                  |
| audio          | URLs de audio                                   |
| comments       | Comentarios HTML encontrados                    |

---

# Valor en Reconnaissance

Analizando este JSON podemos:

* Identificar correos para OSINT.
* Detectar archivos interesantes (PDFs, backups).
* Enumerar librerías JS vulnerables.
* Encontrar endpoints ocultos.
* Extraer comentarios potencialmente sensibles.

---

# Conclusión

Las herramientas de crawling automatizado permiten:

* Acelerar la fase de reconocimiento.
* Extraer datos estructurados.
* Descubrir recursos ocultos.
* Priorizar vectores de ataque.

El verdadero valor no está solo en recolectar datos, sino en analizarlos y correlacionarlos con otros hallazgos como fingerprinting, robots.txt y .well-known endpoints.




---


# Preguntas


#### Después de rastrear inlanefreight.com, identifique la ubicación donde se almacenarán los informes futuros. Responda con el dominio completo, por ejemplo, files.inlanefreight.com.

`Pista`: Quizás haya un comentario al respecto.
