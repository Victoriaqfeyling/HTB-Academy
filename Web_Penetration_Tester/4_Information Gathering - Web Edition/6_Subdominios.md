# Subdomains

## Introducción

Al explorar registros DNS, normalmente comenzamos por el **dominio principal** (por ejemplo, `example.com`). Sin embargo, debajo de este dominio suele existir una **red de subdominios** que amplía considerablemente la superficie de ataque.

Los **subdominios** son extensiones del dominio principal y se utilizan para **organizar, separar o aislar** distintas funcionalidades o servicios. Ejemplos comunes incluyen:

* `blog.example.com`
* `shop.example.com`
* `mail.example.com`

Desde el punto de vista del Web Reconnaissance, estos subdominios suelen ser **mucho más interesantes** que el dominio principal.

---

## ¿Por qué los subdominios son importantes para el Web Recon?

Los subdominios frecuentemente alojan recursos que:

* No están enlazados desde el sitio principal
* No reciben el mismo nivel de hardening
* Quedan olvidados con el paso del tiempo

Esto los convierte en **objetivos ideales durante un pentest web**.

---

## Riesgos comunes asociados a subdominios

### Entornos de desarrollo y staging

Las organizaciones suelen utilizar subdominios para:

* Pruebas de nuevas funcionalidades
* Entornos de staging o preproducción

Ejemplos:

* `dev.example.com`
* `staging.example.com`

Estos entornos a menudo presentan:

* Configuraciones débiles
* Credenciales por defecto
* Información sensible expuesta

---

### Portales de login ocultos

Algunos subdominios alojan:

* Paneles administrativos
* Interfaces internas
* Dashboards de gestión

Estos portales no siempre están pensados para ser accesibles públicamente, pero pueden quedar expuestos.

---

### Aplicaciones legacy

Aplicaciones antiguas o abandonadas pueden residir en subdominios:

* Software desactualizado
* Vulnerabilidades conocidas
* Falta de mantenimiento

Este tipo de activos suele ser especialmente valioso para un atacante.

---

### Exposición de información sensible

Los subdominios pueden revelar:

* Documentación interna
* Archivos de configuración
* Backups
* Datos confidenciales

Todo esto amplía significativamente el impacto potencial de una brecha.

---

## Enumeración de Subdominios

La **enumeración de subdominios** es el proceso de identificar y listar sistemáticamente todos los subdominios asociados a un dominio objetivo.

Desde el punto de vista de DNS:

* Los subdominios suelen resolverse mediante **registros A / AAAA**
* También pueden existir **registros CNAME**, que apuntan a otros dominios o subdominios

---

## Enfoques para la enumeración de subdominios

Existen dos enfoques principales:

* **Enumeración activa**
* **Enumeración pasiva**

Cada uno tiene ventajas y limitaciones.

---

## Enumeración activa de subdominios

La enumeración activa implica **interactuar directamente** con la infraestructura DNS del objetivo.

### Transferencia de zona (Zone Transfer)

Una transferencia de zona mal configurada puede revelar:

* La lista completa de subdominios
* Registros internos de la organización

Sin embargo, hoy en día este tipo de fallas es **poco frecuente** debido a mejores prácticas de seguridad.

---

### Fuerza bruta de subdominios

El método activo más común consiste en **probar nombres de subdominios** contra el dominio objetivo.

Este enfoque utiliza:

* Wordlists de subdominios comunes
* Patrones personalizados

Herramientas habituales:

* `dnsenum`
* `ffuf`
* `gobuster`

Este método:

* Ofrece alto control
* Puede descubrir subdominios no indexados
* Es más **detectable** por mecanismos de defensa

---

## Enumeración pasiva de subdominios

La enumeración pasiva evita interactuar directamente con el objetivo y se apoya en **fuentes externas**.

### Certificate Transparency (CT Logs)

Los certificados SSL/TLS suelen listar subdominios en el campo **Subject Alternative Name (SAN)**.

Los **CT logs** son repositorios públicos que permiten:

* Descubrir subdominios históricos
* Identificar infraestructura expuesta

Este método es extremadamente valioso y sigiloso.

---

### Motores de búsqueda

Mediante operadores avanzados, por ejemplo:

```text
site:example.com
```

es posible identificar subdominios indexados públicamente.

---

### Bases de datos y servicios online

Existen múltiples servicios que agregan información DNS desde diversas fuentes, permitiendo:

* Buscar subdominios
* Analizar cambios históricos
* Evitar interacción directa con el objetivo

---

## Comparación de enfoques

| Método | Ventajas                  | Desventajas              |
| ------ | ------------------------- | ------------------------ |
| Activo | Mayor control y cobertura | Más ruidoso y detectable |
| Pasivo | Sigiloso y rápido         | Puede no ser completo    |

---

## Estrategia recomendada

Una enumeración de subdominios efectiva combina:

* Técnicas **pasivas** para descubrimiento inicial
* Técnicas **activas** para validación y ampliación

Esta combinación maximiza resultados y minimiza detección.

---

## Conclusión

Los subdominios representan una **fuente crítica de superficie de ataque** en cualquier pentest web.

Enumerarlos correctamente permite:

* Descubrir activos ocultos
* Identificar entornos mal configurados
* Ampliar significativamente el alcance del reconocimiento

Un buen pentester rara vez se queda solo con el dominio principal: **los subdominios suelen contar la historia real de la infraestructura**.
