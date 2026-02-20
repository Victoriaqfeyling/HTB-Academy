# Automating Recon

El reconocimiento manual puede ser efectivo, pero también consume tiempo y es propenso a errores humanos. La automatización del reconocimiento web permite mejorar significativamente la eficiencia, la precisión y la capacidad de escalar el análisis sobre múltiples objetivos.

---

# ¿Por qué automatizar el Reconocimiento?

La automatización ofrece múltiples ventajas:

## ⚡ Eficiencia

Las herramientas automatizadas pueden ejecutar tareas repetitivas mucho más rápido que un humano.

---

## 📈 Escalabilidad

Permite analizar:

* Múltiples dominios
* Subdominios
* Rangos IP
* Infraestructuras completas

---

## 🎯 Consistencia

Las herramientas siguen reglas predefinidas:

* Resultados reproducibles
* Menor margen de error humano

---

## 🔍 Cobertura Integral

La automatización puede incluir:

* Enumeración DNS
* Descubrimiento de subdominios
* Crawling web
* Escaneo de puertos
* Análisis de headers

---

## 🔗 Integración

Muchos frameworks permiten integrarse con:

* Escáneres de vulnerabilidades
* Herramientas de explotación
* Sistemas de reporte

---

# Frameworks de Reconocimiento

Estos frameworks buscan ofrecer una suite completa de herramientas.

---

## 🔎 FinalRecon

Herramienta escrita en Python con estructura modular.

[FinalRecon](https://github.com/thewhiteh4t/FinalRecon)

Permite:

* Análisis de headers
* Información SSL
* Whois lookup
* Crawling
* Enumeración DNS
* Subdominios
* Directory brute force
* Integración con Wayback Machine

---

## 🧠 Recon-ng

Framework modular en Python.

[Recon-ng](https://github.com/lanmaster53/recon-ng)

Incluye:

* Enumeración DNS
* Subdominios
* Escaneo de puertos
* Crawling
* Módulos de explotación

---

## 📧 theHarvester

Enfocado en OSINT.

[theHarvester](https://github.com/laramies/theHarvester)

Recolecta:

* Emails
* Subdominios
* Hosts
* Empleados
* Banners

---

## 🕸 SpiderFoot

Herramienta de automatización OSINT.

[SpiderFoot](https://github.com/smicallef/spiderfoot)

Integra múltiples fuentes de datos para:

* IPs
* Dominios
* Correos
* Perfiles sociales

---

## 🗂 OSINT Framework

Colección organizada de herramientas OSINT clasificadas por tipo de fuente.

[OSINT Framework](https://osintframework.com/)

---

# FinalRecon en Detalle

FinalRecon ofrece múltiples módulos:

## 📑 Header Information

* Revela servidor
* Tecnologías
* Posibles configuraciones inseguras

## 📜 Whois Lookup

* Datos de registro del dominio
* Contactos
* Servidores DNS

## 🔐 SSL Information

* Validez del certificado
* Emisor
* Configuración TLS

## 🕷 Crawler

Extrae:

* HTML
* CSS
* JavaScript
* Enlaces internos y externos
* robots.txt
* sitemap.xml
* Enlaces en JavaScript
* Datos históricos (Wayback)

## 🌐 DNS Enumeration

Consulta más de 40 tipos de registros DNS, incluyendo DMARC.

## 🧬 Subdomain Enumeration

Utiliza fuentes como:

* crt.sh
* ThreatMiner
* CertSpotter
* VirusTotal
* Shodan

## 📂 Directory Enumeration

Soporta wordlists personalizadas y extensiones de archivo.

---

# Instalación de FinalRecon

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```

Cómo ya no se puede utilizar `pip3` de esa forma, hacemos lo siguiente luego de `cd FinalRecon`
```bash
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

Probamos el programa:
```bash
python3 ./finalrecon.py --help
```

<img width="913" height="590" alt="image" src="https://github.com/user-attachments/assets/628bb9dd-8e2f-4194-a9c9-1430199fcde8" />


# Opciones Principales

| Opción     | Argumento | Descripción               |
| ---------- | --------- | ------------------------- |
| -h, --help |           | Mostrar ayuda             |
| --url      | URL       | Especificar objetivo      |
| --headers  |           | Obtener headers           |
| --sslinfo  |           | Información SSL           |
| --whois    |           | Whois lookup              |
| --crawl    |           | Ejecutar crawler          |
| --dns      |           | Enumeración DNS           |
| --sub      |           | Enumerar subdominios      |
| --dir      |           | Buscar directorios        |
| --wayback  |           | Obtener URLs históricas   |
| --ps       |           | Escaneo rápido de puertos |
| --full     |           | Reconocimiento completo   |

---

# Ejemplo de Uso

```bash
./finalrecon.py --headers --whois --url http://inlanefreight.com
```

<img width="1025" height="828" alt="image" src="https://github.com/user-attachments/assets/f29a0758-9db7-4bbd-b649-62f1e3c6aedd" />

Salida relevante:

* Dirección IP del objetivo
* Información del servidor (Apache/2.4.41 Ubuntu)
* Endpoints WordPress detectados
* Información de registro del dominio (Amazon Registrar)
* Servidores DNS (AWS)

---

# Flujo Profesional de Automating Recon

1. Ejecutar módulos básicos (headers, whois).
2. Enumerar DNS y subdominios.
3. Ejecutar crawling automático.
4. Revisar Wayback URLs.
5. Realizar directory brute force.
6. Correlacionar resultados con OSINT.

---

# Conclusión

La automatización en reconnaissance:

* Acelera la recolección de información.
* Reduce errores humanos.
* Amplía cobertura.
* Permite escalar análisis.

Sin embargo, la herramienta no reemplaza el análisis humano. El verdadero valor está en interpretar y correlacionar los datos obtenidos para identificar vectores reales de ataque.
