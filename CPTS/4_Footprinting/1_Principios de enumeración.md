# 🧭 Footprinting 


---

## 1. Enumeración

En ciberseguridad, **enumeración** es el proceso de recolectar información sobre un objetivo mediante:

* **Métodos activos:** escaneos, conexiones directas, consultas a servicios.
* **Métodos pasivos:** uso de terceros, información pública, sin interactuar con el objetivo.

> **OSINT no es enumeración.**
> OSINT es completamente pasivo y debe ejecutarse por separado.

La enumeración es un **ciclo iterativo**: cada dato descubierto abre nuevas rutas para seguir investigando.

---

## 2. Qué enumerar

* Dominios y subdominios
* Rangos de IP
* Servicios expuestos
* Protocolos utilizados
* Infraestructura y topología
* Proveedores externos (SaaS, cloud, CDNs)

El objetivo no es solo saber "qué puertos están abiertos", sino **entender la función y el contexto de cada servicio** dentro del ecosistema del objetivo.

---

## 3. Entender la infraestructura antes de atacarla

Un error común es atacar servicios visibles (SSH, RDP, WinRM) con **bruteforce** sin comprender primero:

* el rol del servicio,
* las defensas existentes,
* las rutas indirectas posibles.

El bruteforce es:

* ruidoso,
* detectable,
* propenso a activar bloqueos,
* y generalmente inútil si no se entiende el entorno.

> Un pentester profesional no busca entrar "porque sí", sino encontrar **todas las formas posibles de entrar**.

---

## 4. La metáfora del cazador de tesoros

Un pentester no "cava" al azar. Como un explorador:

* estudia mapas,
* entiende el terreno,
* elige herramientas,
* analiza riesgos,
* formula un plan.

Si cava sin orientación:

* desperdicia recursos,
* genera daño,
* queda bloqueado,
* nunca encuentra el objetivo.

---

## 5. Ver lo visible… y lo invisible

Las preguntas fundamentales de la enumeración son:

### Lo que podemos ver

* ¿Qué vemos exactamente?
* ¿Por qué lo vemos?
* ¿Qué mapa mental genera?
* ¿Qué información obtenemos?
* ¿Cómo podemos usarla?

### Lo que no vemos

* ¿Qué falta?
* ¿Por qué no aparece?
* ¿Qué podría estar ocultando el sistema?
* ¿Qué implicancias tiene para el ataque?

Un pentester modela tanto lo expuesto como lo oculto para reconstruir la infraestructura completa.

---

## 6. El verdadero bloqueo no suele ser explotación, sino comprensión

Cuando un pentester no sabe cómo avanzar, rara vez es por falta de herramientas.
Generalmente es por falta de **entendimiento técnico del servicio, protocolo o infraestructura**.

> Nuestra tarea no es explotar máquinas, sino descubrir **cómo podrían explotarse**.

---

## 7. Principios fundamentales de la Enumeración

| Nº    | Principio                                                           | Descripción                                                                                       |
| ----- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **1** | Hay más de lo que parece. Considera todos los puntos de vista.      | Lo visible no es todo. Piensa en servicios internos, defensas, terceros, restricciones, bypasses. |
| **2** | Distingue lo que ves de lo que no ves.                              | Diferencia evidencia real de suposiciones. Lo oculto puede ser clave.                             |
| **3** | Siempre hay forma de obtener más información. Entiende el objetivo. | Desde certificados hasta metadatos, APIs, ASNs o integraciones externas.                          |

---

