# Web Archives (Wayback Machine)

En el dinámico mundo digital, los sitios web cambian constantemente: páginas que desaparecen, secciones que se modifican, tecnologías que se reemplazan. Sin embargo, gracias a la **Wayback Machine** del Internet Archive, es posible retroceder en el tiempo y explorar cómo eran los sitios web en el pasado.

---

# ¿Qué es la Wayback Machine?

[Wayback Machine](https://web.archive.org/)

<img width="1764" height="821" alt="image" src="https://github.com/user-attachments/assets/7efb680f-44cd-47a6-9593-3eb1f62a022f" />


La **Wayback Machine** es un archivo digital de la World Wide Web y otros recursos de Internet.

Fue creada por la organización sin fines de lucro **Internet Archive** y archiva sitios web desde 1996.

Permite a los usuarios:

* Visualizar versiones antiguas de un sitio web.
* Consultar capturas históricas (snapshots).
* Analizar cambios en diseño, contenido y funcionalidad.

Cada versión archivada se denomina **capture** o **snapshot**.

---

# ¿Cómo funciona la Wayback Machine?

Opera de forma similar a un motor de búsqueda, pero en lugar de solo indexar contenido, almacena copias completas de las páginas.

Su funcionamiento puede dividirse en tres etapas:

---

<img width="1258" height="745" alt="image" src="https://github.com/user-attachments/assets/ee50e818-c0a6-4995-b2c8-be3af387cb72" />


## 1️⃣ Crawling

La Wayback Machine utiliza bots automatizados que:

* Navegan sistemáticamente la web.
* Siguen enlaces.
* Descargan copias completas de las páginas encontradas.

---

## 2️⃣ Archiving

Las páginas descargadas se almacenan junto con:

* HTML
* CSS
* JavaScript
* Imágenes
* Recursos asociados

Cada captura queda asociada a una **fecha y hora específica**, creando una instantánea histórica.

La frecuencia de archivado depende de:

* Popularidad del sitio.
* Frecuencia de actualización.
* Recursos disponibles del Internet Archive.

Algunos sitios se archivan varias veces por día; otros solo unas pocas veces al año.

---

## 3️⃣ Accessing

Los usuarios pueden:

1. Introducir una URL en la interfaz.
2. Seleccionar una fecha.
3. Visualizar cómo era el sitio en ese momento.

También es posible:

* Buscar términos dentro del contenido archivado.
* Descargar contenido para análisis offline.

---

# Limitaciones

* No todos los sitios están archivados.
* No todas las páginas de un sitio se capturan.
* Algunos propietarios solicitan exclusión del archivo.
* Puede haber recursos faltantes en ciertas capturas.

---

# Importancia en Web Reconnaissance

La Wayback Machine es una fuente extremadamente valiosa durante la fase de reconocimiento.

---

## 🔎 Descubrir Activos Ocultos

Permite encontrar:

* Directorios antiguos
* Subdominios olvidados
* Archivos eliminados
* Paneles administrativos antiguos

Estos recursos pueden no estar disponibles actualmente, pero podrían seguir existiendo en el servidor.

---

## 🔄 Analizar Cambios y Evolución

Comparando snapshots históricos se pueden detectar:

* Cambios en estructura
* Tecnologías utilizadas anteriormente
* Versiones antiguas vulnerables
* Eliminación de funcionalidades

Esto puede revelar patrones interesantes o errores de configuración.

---

## 🧠 Fuente de OSINT

El contenido archivado puede revelar:

* Empleados antiguos
* Correos electrónicos
* Estrategias de marketing
* Tecnologías usadas históricamente

---

## 🕵 Reconocimiento Pasivo

Acceder a snapshots archivados:

* No interactúa directamente con el servidor objetivo.
* No genera logs en la infraestructura actual del target.
* Es menos detectable.

---

# Ejemplo: Hack The Box en el Pasado

<img width="1237" height="583" alt="image" src="https://github.com/user-attachments/assets/894e9472-1c5a-40ab-b8f4-fc8d92d57ef7" />


Si buscamos versiones antiguas de Hack The Box en la Wayback Machine y seleccionamos la captura más temprana disponible (por ejemplo 2017-06-10), podemos observar:

* Diseño inicial de la plataforma.
* Versión beta (0.8.7).
* Estructura original del sitio.
* Cambios significativos respecto a la versión actual.

Este tipo de análisis puede ser útil para:

* Identificar tecnologías usadas en el pasado.
* Detectar endpoints que ya no son visibles.
* Analizar evolución de la superficie de ataque.

---

# Metodología Recomendada

Durante un pentest:

1. Consultar la Wayback Machine para el dominio objetivo.
2. Revisar capturas más antiguas y más recientes.
3. Buscar rutas interesantes (admin, backup, api, dev).
4. Comparar cambios estructurales.
5. Correlacionar con resultados de crawling y Google Dorking.

---




# Preguntas


#### ¿Cuántos laboratorios de pruebas de penetración tenía HackTheBox el 8 de agosto de 2018? Responda con un número entero, por ejemplo, 1234

<img width="1593" height="894" alt="image" src="https://github.com/user-attachments/assets/d0dfa118-1e4e-497d-80d7-2a0bf0e79f1e" />

<img width="1593" height="894" alt="image" src="https://github.com/user-attachments/assets/c92af997-0dc9-4f54-9514-30911cf28d5f" />



#### ¿Cuántos miembros tenía HackTheBox el 10 de junio de 2017? Responde con un número entero, p. ej., 1234.


<img width="1593" height="894" alt="image" src="https://github.com/user-attachments/assets/200de3a6-c126-4122-b88b-6776bd4adb1a" />

<img width="1660" height="1011" alt="image" src="https://github.com/user-attachments/assets/2dc904c5-acc8-41ba-a4ce-ff545744b3a3" />


#### En marzo de 2002, ¿a qué sitio web redirigía el dominio facebook.com? Responda con el dominio completo, por ejemplo, http://www.facebook.com/

<img width="1660" height="1011" alt="image" src="https://github.com/user-attachments/assets/af902412-145d-4ee4-badf-7f23375a25f2" />


<img width="1767" height="1011" alt="image" src="https://github.com/user-attachments/assets/32940a61-ed28-4e3d-90e9-37f4b8c2d26c" />


#### Según el sitio web paypal.com en octubre de 1999, ¿qué se podía usar para transferir dinero a cualquier persona? Responda con el nombre del producto, por ejemplo, "Mi Dispositivo". Quite el símbolo ™ de su respuesta.

<img width="1767" height="1011" alt="image" src="https://github.com/user-attachments/assets/e89c7d08-c80a-4749-b48c-668620f9f7c3" />

<img width="1767" height="1011" alt="image" src="https://github.com/user-attachments/assets/2705e218-8cfc-4c11-b132-b0ea8b43438f" />


#### Volviendo a noviembre de 1998 en google.com, ¿qué dirección albergaba el "Prototipo de Motor de Búsqueda de Google" no alfa de Google? Responda con la dirección completa, por ejemplo, http://google.com

<img width="1767" height="1011" alt="image" src="https://github.com/user-attachments/assets/81058ed0-829e-4d13-b468-d6b807d91ea6" />

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/deff2250-40b0-4a70-8282-63f6c45e49af" />


#### Volviendo a marzo de 2000 en www.iana.org, ¿cuándo exactamente se actualizó el sitio por última vez? Responda con la fecha en el pie de página, por ejemplo, 11 de marzo de 1999

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/ed494e60-3448-4abd-865e-1a17c57eb255" />

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/d1b588ff-6ca4-4ddd-80b8-ebb952f218e2" />


#### Según la instantánea de wikipedia.com tomada el 9 de febrero de 2003, ¿en cuántos artículos estaban trabajando ya en la versión en inglés? Responda con el número que indican sin comas, por ejemplo, 100000, no 100000

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/2f03c8be-db46-4b1a-be9f-db7d32719032" />

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/82a45873-3190-44bf-a372-f7e4bc1abdea" />
