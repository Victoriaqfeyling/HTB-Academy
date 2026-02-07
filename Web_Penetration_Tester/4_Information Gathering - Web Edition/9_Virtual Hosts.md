# Virtual Hosts (VHosts)

## Introducción

Una vez que **DNS** dirige el tráfico hacia el servidor correcto (IP), entra en juego la configuración del **servidor web** (Apache, Nginx, IIS) para decidir **cómo manejar** cada request entrante.

En un mismo servidor es común alojar **múltiples sitios o aplicaciones**. Para lograrlo, los servidores web utilizan **Virtual Hosting**, una técnica que permite diferenciar qué contenido servir según el dominio/subdominio solicitado.

---

## ¿Cómo funcionan los Virtual Hosts?

La base del virtual hosting es el **HTTP Host header**, un campo incluido en cada request HTTP.

Cuando un navegador solicita un sitio, envía algo así:

```http
GET / HTTP/1.1
Host: www.example.com
```

El servidor web usa ese `Host:` como “selector” para decidir:

* qué configuración de sitio aplicar
* qué `DocumentRoot` usar
* qué contenido devolver

---

## Diferencia clave: Subdominios vs VHosts

Aunque se relacionan, no son lo mismo:

### Subdominios

* Son extensiones del dominio principal (`blog.example.com`).
* Normalmente tienen **registros DNS propios** (A/AAAA/CNAME).
* Pueden apuntar a:

  * la misma IP del dominio principal, o
  * una IP distinta.

Sirven para separar servicios: blog, tienda, mail, etc.

---

### Virtual Hosts (VHosts)

* Son **configuraciones internas** en el servidor web.
* Permiten alojar **múltiples sitios** sobre una misma IP.
* Pueden estar asociadas a:

  * dominios distintos (`example1.com`, `example2.org`)
  * subdominios (`dev.example.com`)

Un VHost **puede existir incluso sin un registro DNS público**.

---

## Acceso a VHosts sin DNS

Si un virtual host no está publicado en DNS, aún puede accederse manualmente modificando el archivo **hosts** del atacante:

```text
<IP>    vhost.ejemplo.com
```

Esto “engaña” al sistema local para que resuelva ese nombre a una IP específica, permitiendo probar el Host header.

---

## VHost Fuzzing

Muchos sitios tienen subdominios **no públicos** (no aparecen en DNS). Estos pueden ser accesibles solo internamente o bajo ciertas condiciones.

El **VHost fuzzing** consiste en:

* tomar una IP conocida
* enviar requests cambiando el `Host:` con múltiples candidatos
* detectar cuándo el servidor devuelve contenido distinto

Esto permite descubrir:

* subdominios ocultos
* paneles internos
* aplicaciones legacy
* ambientes dev/staging

---

## Ejemplo de configuración Apache (Name-Based Virtual Hosting)

```apacheconf
<VirtualHost *:80>
    ServerName www.example1.com
    DocumentRoot /var/www/example1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example2.org
    DocumentRoot /var/www/example2
</VirtualHost>

<VirtualHost *:80>
    ServerName www.another-example.net
    DocumentRoot /var/www/another-example
</VirtualHost>
```

En este caso, **múltiples dominios distintos** comparten la misma IP. El servidor decide qué contenido servir usando el **Host header**.

---

## Cómo el servidor resuelve qué VHost servir

Proceso simplificado:

1. El navegador solicita `http://<IP>` con un `Host: dominio`.
2. El servidor web lee el **Host header**.
3. Compara contra su configuración de virtual hosts.
4. Selecciona el `DocumentRoot` correspondiente.
5. Devuelve el contenido al cliente.

<img width="1252" height="775" alt="image" src="https://github.com/user-attachments/assets/34c9a80b-771b-48eb-8798-310bdeca693d" />


---

## Tipos de Virtual Hosting

### 1) Name-Based Virtual Hosting

* Usa exclusivamente el **Host header**.
* Es el más común.
* No requiere múltiples IPs.

✅ Pros:

* barato y escalable
* fácil de configurar

⚠️ Contras:

* puede tener limitaciones con SSL/TLS (dependiendo de configuración y SNI)

---

### 2) IP-Based Virtual Hosting

* Cada sitio tiene una IP distinta.
* El servidor decide según la IP destino.

✅ Pros:

* mejor aislamiento
* útil para protocolos que no usan Host header

⚠️ Contras:

* requiere muchas IPs (costoso)
* menos escalable

---

### 3) Port-Based Virtual Hosting

* Diferentes sitios se exponen en diferentes puertos.

Ejemplo:

* Sitio A → `:80`
* Sitio B → `:8080`

✅ Pros:

* útil cuando hay pocas IPs

⚠️ Contras:

* menos común
* menos “user friendly” (hay que especificar puerto)

---

## Herramientas para descubrir Virtual Hosts

| Herramienta     | Descripción                                          | Features                               |
| --------------- | ---------------------------------------------------- | -------------------------------------- |
| [**gobuster**](https://github.com/OJ/gobuster)    | Brute-force de directorios/archivos y también VHosts | Rápida, múltiples métodos, wordlists   |
| [**feroxbuster**](https://github.com/epi052/feroxbuster) | Similar a gobuster (Rust), conocido por su velocidad y flexibilidad                            | Recursión, wildcard discovery, filtros |
| [**ffuf**](https://github.com/ffuf/ffuf)        | Fuzzer web muy rápido (Host header)                  | Alta personalización y filtrado        |

---

## Gobuster para VHost Discovery

### Preparación

Antes de brute-forzar VHosts necesitás:

1. **IP/URL objetivo**

* obtenida vía DNS, recon o escaneo

2. **Wordlist**

* SecLists o personalizada

---

### Comando general

```bash
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```

* `-u` especifica el objetivo
* `-w` wordlist
* `--append-domain` agrega automáticamente el dominio base

> ℹ️ En versiones nuevas de Gobuster, `--append-domain` es obligatorio para construir correctamente los vhosts completos.

---

### Ejemplo (HTB)

```bash
gobuster vhost -u http://inlanefreight.htb:81 \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
  --append-domain
```

Output ejemplo:

```text
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://inlanefreight.htb:81
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: forum.inlanefreight.htb:81 Status: 200 [Size: 100]
[...]
Progress: 114441 / 114442 (100.00%)
===============================================================
Finished
===============================================================
```


---

## Flags útiles en Gobuster

* `-t` → aumenta threads (más velocidad)
* `-k` → ignora errores de certificado SSL/TLS
* `-o` → guarda output a archivo

---

## Consideraciones de OPSEC

* El VHost discovery puede generar **mucho tráfico**.
* Puede ser detectado por:

  * IDS
  * WAF
  * logs del servidor

Buenas prácticas:

* controlar threads (`-t`)
* usar wordlists acotadas
* filtrar por tamaño/código HTTP

---

## Conclusión

Los **Virtual Hosts** amplían significativamente la superficie de ataque, porque un mismo servidor puede alojar múltiples aplicaciones y entornos.

La combinación de:

* análisis DNS
* modificación de hosts
* fuzzing del Host header

permite descubrir VHosts ocultos que muchas veces contienen **aplicaciones internas o mal configuradas**, aportando hallazgos de alto valor en un pentest web.


---

# PREGUNTAS

#### Ejecute ataques de fuerza bruta contra hosts virtuales en el sistema objetivo. ¿Cuál es el subdominio completo con el prefijo "web"? Responda usando el dominio completo, por ejemplo, "x.inlanefreight.htb".


#### Ejecute ataques de fuerza bruta contra hosts virtuales en el sistema objetivo. ¿Cuál es el subdominio completo con el prefijo "vm"? Responda usando el dominio completo, por ejemplo, "x.inlanefreight.htb".


#### Ejecutar ataques de fuerza bruta contra hosts virtuales en el sistema objetivo. ¿Cuál es el subdominio completo con el prefijo "br"? Responda usando el dominio completo, por ejemplo, "x.inlanefreight.htb".


#### Ejecutar ataques de fuerza bruta contra hosts virtuales en el sistema objetivo. ¿Cuál es el subdominio completo con el prefijo "a"? Responda usando el dominio completo, por ejemplo, "x.inlanefreight.htb".



#### Ejecutar ataques de fuerza bruta contra hosts virtuales en el sistema objetivo. ¿Cuál es el subdominio completo con el prefijo "su"? Responda usando el dominio completo, por ejemplo, "x.inlanefreight.htb".
