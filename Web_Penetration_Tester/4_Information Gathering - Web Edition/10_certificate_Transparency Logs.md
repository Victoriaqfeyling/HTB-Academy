# Certificate Transparency Logs

## Introducción

En el ecosistema de Internet, la **confianza** es un pilar fundamental. Uno de los mecanismos clave que sostienen esta confianza es el uso de **SSL/TLS**, que permite cifrar la comunicación entre un navegador y un servidor web.

En el centro de SSL/TLS se encuentran los **certificados digitales**, que validan la identidad de un sitio web. Sin embargo, el proceso de emisión de certificados no es infalible: pueden existir certificados mal emitidos, fraudulentos o directamente maliciosos que permitan suplantar sitios web legítimos, interceptar datos confidenciales o propagar malware

Para mitigar este riesgo surge **Certificate Transparency (CT)**.

---

## ¿Qué son los Certificate Transparency Logs?

Los **Certificate Transparency Logs** son **registros públicos, inmutables (append-only)** que almacenan información sobre todos los certificados SSL/TLS emitidos por las Autoridades Certificadoras (CAs).

Cada vez que una CA emite un certificado:

* Debe enviarlo a **múltiples CT logs**
* Estos logs son mantenidos por **organizaciones independientes**
* Cualquiera puede inspeccionarlos

Pueden pensarse como un **registro global de certificados** emitidos en Internet.

---

## ¿Por qué existen los CT Logs?

Los CT logs cumplen varios objetivos críticos:

### Detección temprana de certificados fraudulentos

Permiten identificar rápidamente **certificados no autorizados** o mal emitidos para un dominio legítimo. Esto posibilita:

* Revocar certificados maliciosos
* Reducir ataques de impersonación o MITM

---

### Responsabilidad de las Certificate Authorities

Al ser públicos:

* Las CAs quedan expuestas si emiten certificados incorrectos
* Se fomenta el cumplimiento de estándares
* Se refuerza la confianza en el ecosistema PKI

---

### Fortalecimiento del Web PKI

Los CT logs introducen **auditoría pública** sobre la emisión de certificados, fortaleciendo la infraestructura de confianza que sostiene HTTPS.

---

# Registros de Transparencia de Certificados (Certificate Transparency Logs)

## ¿Cómo funcionan los Certificate Transparency Logs?

Los **Certificate Transparency (CT) Logs** son un mecanismo diseñado para aumentar la seguridad y la confianza en el ecosistema de certificados SSL/TLS. Su funcionamiento combina técnicas criptográficas con un modelo de auditoría pública.

A continuación se detalla el proceso paso a paso:

---

### 1. Emisión del Certificado

Cuando el propietario de un sitio web solicita un certificado SSL/TLS a una Autoridad Certificadora (CA):

1. La CA realiza un proceso de validación (due diligence) para verificar:

   * La identidad del solicitante.
   * La propiedad o control del dominio.
2. Una vez validado, la CA genera un **pre-certificado**.

El *pre-certificado* es una versión preliminar del certificado final que será emitido.

---

### 2. Envío a los CT Logs

La CA envía ese pre-certificado a múltiples servidores de CT Logs.

Características importantes:

* Cada CT Log es operado por una organización diferente.
* Esto garantiza **redundancia y descentralización**.
* Los logs son **append-only** (solo se pueden agregar entradas).

  * No se pueden modificar.
  * No se pueden eliminar.

Esto asegura la integridad histórica de los certificados emitidos.

---

### 3. Signed Certificate Timestamp (SCT)

Cuando un CT Log recibe el pre-certificado:

1. Genera un **Signed Certificate Timestamp (SCT)**.
2. El SCT es una prueba criptográfica que demuestra que:

   * El certificado fue recibido.
   * Fue registrado en un momento específico.

Luego:

* El SCT se incorpora al certificado final.
* El certificado final es entregado al propietario del sitio web.

El SCT funciona como evidencia pública de que el certificado fue correctamente registrado.

---

### 4. Verificación por el Navegador

Cuando un usuario visita un sitio web:

1. El navegador recibe el certificado SSL/TLS.
2. Verifica los SCTs incluidos.
3. Comprueba que esos SCTs existan y sean válidos en los CT Logs públicos.

Si:

* ✅ Los SCTs son válidos → Se establece la conexión segura.
* ❌ No son válidos → El navegador puede mostrar una advertencia.

Esto evita que certificados emitidos de forma fraudulenta pasen desapercibidos.

---

### 5. Monitoreo y Auditoría

Los CT Logs son monitoreados constantemente por:

* Investigadores de seguridad.
* Propietarios de dominios.
* Vendors de navegadores.

Se buscan anomalías como:

* Certificados emitidos para dominios que el solicitante no controla.
* Certificados que violan estándares de la industria.
* Emisiones sospechosas.

Si se detecta un problema:

* Se reporta a la CA correspondiente.
* Puede iniciarse una investigación.
* El certificado puede ser revocado.

Este modelo introduce **transparencia pública en la emisión de certificados**, algo que antes no existía.

---

# La Estructura del Árbol de Merkle

Para garantizar que los CT Logs sean íntegros y resistentes a manipulaciones, utilizan una estructura criptográfica llamada **Árbol de Merkle (Merkle Tree)**.

## ¿Qué es un Árbol de Merkle?

Es una estructura en forma de árbol donde:

* Cada nodo hoja representa el hash de un certificado.
* Cada nodo intermedio es el hash de la concatenación de sus nodos hijos.
* La raíz del árbol se llama **Merkle Root**.

La Merkle Root es un único hash que representa el estado completo del log.

---

## Ejemplo Conceptual


<img width="1257" height="595" alt="image" src="https://github.com/user-attachments/assets/f55c9237-61b6-4f75-8bb7-b4152770616e" />  



Imaginemos que tenemos cuatro certificados:

* Cert 1
* Cert 2
* Cert 3
* Cert 4

### Nivel 1 – Hojas

Cada certificado se convierte en un hash:

* H(Cert 1)
* H(Cert 2)
* H(Cert 3)
* H(Cert 4)

### Nivel 2 – Nodos Intermedios

Se combinan de a pares:

* Hash 1 = H(H(Cert 1) + H(Cert 2))
* Hash 2 = H(H(Cert 3) + H(Cert 4))

### Nivel 3 – Raíz

* Root Hash = H(Hash 1 + Hash 2)

Esta raíz representa el estado completo del log.

---

## Verificación Eficiente (Merkle Path)

Una ventaja clave del Árbol de Merkle es que permite verificar la inclusión de un certificado sin descargar todo el log.

Por ejemplo, para verificar **Cert 2** (blog.inlanefreight.com), necesitamos:

1. El hash de Cert 2.
2. El hash 1 que verifica también su par en el árbol (Cert 1).
3. Root hash.

Con estos valores podemos:

* Reconstruir Hash 1.
* Luego reconstruir la Root Hash.
* Compararla con la Root pública del log.

Si coincide → El certificado está efectivamente incluido.

---

## Propiedad de Inmutabilidad

Si:

* Se modifica un solo bit de un certificado.
* Se altera cualquier nodo intermedio.

Entonces:

* Cambia su hash.
* Cambia el hash superior.
* Cambia la Root Hash.

Esto hace que cualquier manipulación sea detectable inmediatamente.

---

# Importancia en Seguridad Ofensiva (Pentesting / OSINT)

Desde el punto de vista de un analista de seguridad:

* Los CT Logs permiten descubrir subdominios expuestos.
* Permiten identificar entornos olvidados (dev, staging, test).
* Revelan infraestructura histórica.
* Ayudan en reconocimiento pasivo sin interacción directa con el target.

Son una fuente extremadamente valiosa en la fase de **Information Gathering**.

---

# Conclusión

Los Certificate Transparency Logs:

* Introducen transparencia pública en la emisión de certificados.
* Detectan emisiones fraudulentas.
* Permiten auditoría continua.
* Utilizan Árboles de Merkle para garantizar integridad criptográfica.

Gracias a esta arquitectura, cualquier intento de manipulación o emisión indebida puede ser detectado, fortaleciendo la seguridad del ecosistema SSL/TLS y aumentando la confianza en Internet.







---

## CT Logs y Web Reconnaissance

Desde el punto de vista del **reconocimiento web**, los CT logs son una **fuente extremadamente valiosa** para la enumeración de subdominios.

A diferencia de:

* brute-forcing
* wordlists
* fuzzing de nombres

los CT logs ofrecen **datos reales e históricos**, no basados en suposiciones.

### Ventajas clave para el pentesting

* Enumeración de subdominios **sin interactuar con el objetivo**
* Descubrimiento de subdominios **no guessables**
* Acceso a subdominios **antiguos o expirados**
* Ideal para descubrir:

  * entornos dev/staging
  * servicios legacy
  * aplicaciones olvidadas

---

## ¿Por qué los CT Logs son tan efectivos?

Los certificados SSL/TLS suelen incluir en su campo **SAN (Subject Alternative Name)**:

* Dominio principal
* Subdominios asociados

Esto hace que cada certificado emitido deje un **rastro histórico** de subdominios, incluso si:

* ya no existen
* no están en DNS
* no responden actualmente

---

## Herramientas para consultar CT Logs

Existen múltiples servicios para consultar CT logs. Dos de los más utilizados son:

| Herramienta | Características                                        | Casos de uso                              | Pros                 | Contras           |
| ----------- | ------------------------------------------------------ | ----------------------------------------- | -------------------- | ----------------- |
| [**crt.sh**](https://crt.sh/)  | Interfaz web simple, búsqueda por dominio, muestra SAN | Enumeración rápida de subdominios         | Gratis, sin registro | Filtros limitados |
| [**Censys**](https://search.censys.io/)  | Motor de búsqueda avanzado, filtros por certificados   | Análisis profundo, correlación de activos | Muy potente, API     | Requiere cuenta   |

---

## crt.sh en la práctica

### Uso vía web

* Acceder a: [https://crt.sh](https://crt.sh)
* Buscar por dominio: `example.com`
* Revisar el campo **Name Value**

---

### Uso vía API (línea de comandos)

crt.sh también permite consultas automatizadas devolviendo resultados en formato JSON.

### Ejemplo: buscar subdominios "dev" de facebook.com

```bash
curl -s "https://crt.sh/?q=facebook.com&output=json" \
| jq -r '.[] | select(.name_value | contains("dev")) | .name_value' \
| sort -u
```

### Explicación del comando

* `curl -s "https://crt.sh/?q=facebook.com&output=json"`
  Descarga los certificados asociados al dominio en formato JSON.

* `jq -r '.[] | select(.name_value | contains("dev")) | .name_value'`
  Filtra certificados cuyo SAN contiene la palabra `dev`.

* `sort -u`
  Ordena y elimina duplicados.

---

### Resultado de ejemplo

```text
*.dev.facebook.com
*.newdev.facebook.com
*.secure.dev.facebook.com
dev.facebook.com
devvm1958.ftw3.facebook.com
facebook-amex-dev.facebook.com
facebook-amex-sign-enc-dev.facebook.com
newdev.facebook.com
secure.dev.facebook.com
```


---

## Limitaciones de los CT Logs

Aunque son extremadamente útiles, tienen algunas limitaciones:

* No garantizan que el subdominio esté activo
* Pueden incluir subdominios ya eliminados
* No revelan puertos, paths ni contenido

Por eso deben combinarse con:

* DNS resolution
* HTTP probing
* VHost discovery

---

## OPSEC y consideraciones éticas

* Las búsquedas en CT logs son **100% pasivas**
* No generan tráfico hacia el objetivo
* Son ideales para la fase inicial de recon

---

## Conclusión

Los **Certificate Transparency Logs** son una de las fuentes más poderosas y subestimadas para el Web Reconnaissance.

Su capacidad para revelar:

* subdominios reales
* información histórica
* activos olvidados

los convierte en una herramienta esencial para cualquier pentester que busque **ampliar la superficie de ataque sin levantar alertas**.

Integrar CT logs con técnicas activas permite construir una visión completa y precisa de la infraestructura del objetivo.
