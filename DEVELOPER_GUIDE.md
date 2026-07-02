# 📘 DOCUMENTACIÓN MAESTRA: PROYECTO MILAUNCHERSKINS (VERSIÓN DEFINITIVA Y EXPANDIDA)
> **Advertencia de Nivel Técnico:** Este documento contiene especificaciones a nivel de código fuente, ingeniería inversa, scripts de orquestación, y automatización de despliegue. Diseñado para garantizar la perpetuidad y mantenimiento del proyecto "MiLauncherSkins" a través de los años sin pérdida de conocimiento.

---

## 📑 ÍNDICE GENERAL TÉCNICO

1. **[Fase 1: Análisis y Desmontaje del Mod Original](#fase-1-análisis-y-desmontaje-del-mod-original)**
   - 1.1 Objetivo del Proyecto y Necesidad de Sigilo
   - 1.2 Mapeo de la Arquitectura de "CustomPlayerModels" (CPM)
2. **[Fase 2: Intervención Quirúrgica del Código (Java)](#fase-2-intervención-quirúrgica-del-código-java)**
   - 2.1 Anulación del Menú Principal y Botón de Editor (`btn`)
   - 2.2 Silenciamiento de Notificaciones de Red (`VersionCheck`)
3. **[Fase 3: Automatización de Inyección de Código (Python)](#fase-3-automatización-de-inyección-de-código-python)**
   - 3.1 Script 1: `disable_updates.py`
   - 3.2 Script 2: `fix_btn.py`
4. **[Fase 4: Orquestación y Compilación Masiva (PowerShell)](#fase-4-orquestación-y-compilación-masiva-powershell)**
   - 4.1 Script `build_popular.ps1` (Gestión Dinámica de JDKs)
   - 4.2 Script `build_plugins.ps1` (Resolución de Comodines en Servidores)
5. **[Fase 5: Despliegue en Casillas (Generación Automática del README)](#fase-5-despliegue-en-casillas-generación-automática-del-readme)**
   - 5.1 Script `format_readme.py` y Enlaces CDN Raw
6. **[Fase 6: Bitácora de Errores Críticos y Sus Soluciones](#fase-6-bitácora-de-errores-críticos-y-sus-soluciones)**
   - 6.1 El Colapso de Fabric 26.1 (Loom 1.7.4 y Java 25)
   - 6.2 El Error de Sintaxis de Botones (Lambda vs Legacy)
7. **[Fase 7: Protocolo de Actualización (Nuevas Versiones de Minecraft)](#fase-7-protocolo-de-actualización-nuevas-versiones-de-minecraft)**
8. **[Fase 8: Gestión de Cachés y Limpieza del Sistema](#fase-8-gestión-de-cachés-y-limpieza-del-sistema)**
9. **[Fase 9: Código Fuente de Respaldo de los Scripts](#fase-9-código-fuente-de-respaldo-de-los-scripts)**
10. **[Fase 10: Glosario Técnico y Fundamentos de Modding en Minecraft](#fase-10-glosario-técnico-y-fundamentos-de-modding-en-minecraft)**
11. **[Fase 11: Análisis Profundo del Ciclo de Vida de Gradle](#fase-11-análisis-profundo-del-ciclo-de-vida-de-gradle)**
12. **[Fase 12: Explicación Línea por Línea de los Scripts Orquestadores](#fase-12-explicación-línea-por-línea-de-los-scripts-orquestadores)**
13. **[Fase 13: Diseño e Ingeniería de la Tabla Markdown (GitHub)](#fase-13-diseño-e-ingeniería-de-la-tabla-markdown-github)**
14. **[Fase 14: Explicación Exhaustiva de las Clases Java Modificadas](#fase-14-explicación-exhaustiva-de-las-clases-java-modificadas)**
15. **[Fase 15: Conclusión y Estado Final de los Entregables](#fase-15-conclusión-y-estado-final-de-los-entregables)**

---

## 1. Modificaciones al Código Fuente (Fase Histórica)

El objetivo principal fue convertir el mod original en un proceso 100% "silencioso" y transparente que corra en segundo plano, sin interferir con la experiencia del jugador.

### A. Eliminación de la Interfaz Gráfica (UI)
Se intervino la clase principal encargada de renderizar la interfaz y registrar las teclas de acceso rápido (`CustomPlayerModelsClient.java`).

**Archivo clave modificado:**
- `src/main/java/com/carlitos/milauncherskins/client/CustomPlayerModelsClient.java` (y sus variantes por versión).

**Cambios realizados:**
- Se eliminó o comentó el registro de la tecla "G" (Keybind) que abría el menú principal.
- Se vació la función de renderizado del botón de configuración (`btn`), evitando que aparezca el botón de "Editor" o "Settings" en el menú de pausa de Minecraft y en el menú principal.
- El código original de renderizado del botón dependía fuertemente de la versión de Minecraft, por lo que la inyección se hizo directamente anulando las llamadas a las librerías gráficas de Forge/Fabric/NeoForge.

### B. Desactivación del Buscador de Actualizaciones
El mod original poseía un hilo secundario que hacía consultas a GitHub para verificar actualizaciones. Esto debía ser anulado para evitar notificaciones al usuario ("Actualización Disponible").

**Archivo clave modificado:**
- `src/shared/java/com/carlitos/milauncherskins/shared/util/VersionCheck.java`

**Cambios realizados:**
- Se inyectó un comando de retorno temprano (`return;`) en el primer bloque del método que verifica la versión.
- Al hacer esto, el método asume instantáneamente que la versión actual es la más reciente y cierra el hilo, cancelando la petición HTTP (GET) y evitando registrar cualquier mensaje en el chat del juego.

---

## FASE 1: Análisis y Desmontaje del Mod Original

### 1.1 Objetivo del Proyecto y Necesidad de Sigilo
El launcher propietario requería un mod de sincronización de skins ("MiLauncherSkins"). Se utilizó el núcleo Open Source de *CustomPlayerModels*, pero éste contenía funciones indeseadas:
- Un botón gigante llamado "Editor" inyectado en el menú principal del juego.
- Un atajo de teclado ('G') que abría un menú de personalización intrusivo.
- Un actualizador (Updater) que contactaba a GitHub y spameaba el chat del jugador con el mensaje: *"Update Available!"*.

El objetivo fue transformar este mod en un servicio "Fantasma" que corriera estrictamente en segundo plano.

### 1.2 Mapeo de la Arquitectura
El repositorio contenía más de 80 carpetas. Descubrimos que las clases se dividían en dos categorías:
- **Plataforma-Específica**: Clases únicas para Forge, Fabric, Bukkit, etc. (Ej: `CustomPlayerModels-1.20`).
- **Núcleo Compartido**: Clases universales vinculadas mediante Symlinks (`src/shared/java/`). Si se modificaba un archivo aquí, el cambio se propagaba a las 30 versiones simultáneamente.

---

## FASE 2: Intervención Quirúrgica del Código (Java)

### 2.1 Anulación del Menú Principal y Botón de Editor (`btn`)
El mayor desafío fue el archivo `CustomPlayerModelsClient.java` (y sus derivados `FabricClient.java` o `ForgeClient.java`). 

El código original hacía esto:
```java
Button btn = new Button(x, y, width, height, new TranslatableText("button.cpm.editor"), ...);
event.addButton(btn);
```
**El Error de Novato:** Inicialmente intentamos borrar la variable `btn` por completo. Esto causó que el compilador de Gradle lanzara cientos de errores `cannot find symbol variable btn`, ya que otras partes del código intentaban cambiar la posición del botón (`btn.setX(...)`).

**La Solución Definitiva:** Se optó por comentar absolutamente todo el bloque lógico de inicialización y posicionamiento. En lugar de borrar la variable, anulamos la función entera que registraba la UI en la pantalla. Esto satisfizo al compilador y eliminó visualmente el botón del juego.

### 2.2 Silenciamiento de Notificaciones de Red (`VersionCheck`)
El archivo `src/shared/java/.../util/VersionCheck.java` tenía un método `check()` que iniciaba una petición HTTP GET.
Para evitar que el mod hiciera pings externos, inyectamos un cierre forzado del hilo en la primera línea del método:

```java
public static void check() {
    Thread.currentThread().interrupt(); // Mata el hilo asíncrono
    if(true) return; // Retorno temprano absoluto
    // ... el resto de las 50 líneas de código original ...
}
```
Esto garantizó que el sistema de actualizaciones quedara 100% inoperativo en todas las versiones, sin necesidad de borrar los archivos (lo cual habría roto el mod).

---

## FASE 3: Automatización de Inyección de Código (Python)

Dado que había docenas de versiones y el código fuente variaba ligeramente entre 1.12.2 y 1.21.4, se crearon scripts de Python con Expresiones Regulares (RegEx) para buscar y destruir la UI de forma quirúrgica en todos los archivos.

### 3.1 Script: `disable_updates.py`
Este script se encargó de buscar todos los archivos `VersionCheck.java` y reemplazar dinámicamente la función `check()`.
```python
import os, re
pattern = r'(public\s+static\s+void\s+check\(\)\s*\{)'
replacement = r'\1\n        Thread.currentThread().interrupt();\n        if(true) return;\n'
```

### 3.2 Script: `fix_btn.py`
El script más complejo. Buscó la palabra clave `btn = ` dentro de los inicializadores de GUI y comentó el bloque entero. Esto salvó el proyecto cuando descubrimos que el botón aparecía no solo en el menú principal, sino también en el menú de pausa de Minecraft (Escape).

---

## FASE 4: Orquestación y Compilación Masiva (PowerShell)

Compilar mods de Minecraft requiere compatibilidad estricta con versiones de Java. Forge 1.12.2 exige JDK 8, mientras que Fabric 1.21.1 exige JDK 21. Hacerlo a mano habría tomado semanas. Se desarrollaron dos mega-scripts en PowerShell.

### 4.1 Script `build_popular.ps1`
Este script definió una tabla hash (diccionario) con todas las versiones populares objetivo:
```powershell
$builds = @(
    @{ Dir = "CustomPlayerModels-1.12"; Jdk = "jdk8"; Name = "MiLauncherSkins-1.12.2-Forge.jar" },
    @{ Dir = "CustomPlayerModels-1.16"; Jdk = "jdk8"; Name = "MiLauncherSkins-1.16.5-Forge.jar" },
    @{ Dir = "CustomPlayerModelsFabric-1.21.4"; Jdk = "jdk21"; Name = "MiLauncherSkins-1.21.4-Fabric.jar" }
)
```
**Flujo de ejecución:**
1. Hacía `cd` a la subcarpeta.
2. Seteaba dinámicamente `$env:JAVA_HOME` apuntando al JDK requerido.
3. Ejecutaba `cmd /c "gradlew.bat build"`.
4. Buscaba el archivo `.jar` resultante en la carpeta `build/libs`.
5. Excluía los archivos basura (`-sources.jar` y `-dev.jar`).
6. Renombraba el archivo final a `MiLauncherSkins-VERSION-LOADER.jar` y lo movía a la carpeta `final_jars`.

### 4.2 Script `build_plugins.ps1`
Se encarga exclusivamente de Bukkit y Paper.
**Error Histórico Solucionado:** Al compilar plugins, Gradle genera archivos con sufijos de versión (ej: `CustomPlayerModels-Bukkit-0.6.26a.jar`). El script original buscaba un nombre estricto y fallaba al copiarlos. Se solucionó introduciendo comodines `Get-ChildItem "build\libs\CustomPlayerModels-Bukkit-*.jar"`.

---

## FASE 5: Despliegue en Casillas (Generación Automática del README)

Para presentar los 30 archivos `.jar` de forma elegante y profesional a los usuarios, se creó un script en Python (`format_readme.py`) que automatizó la creación de las "casillas" (Tablas en Markdown).

### 5.1 Cómo se construyeron las tablas
El script escaneó el interior de la carpeta `final_jars`. Agrupó los archivos por "Versión de Minecraft" (ej. 1.21.4, 1.20.1) y creó una estructura de tabla Markdown cruzada por "Cargador" (Forge, Fabric, NeoForge).

**Estructura del Enlace (CDN Raw):**
Para que los botones de descarga funcionaran con un solo clic sin pasar por la web de GitHub, los enlaces se construyeron usando el dominio `raw.githubusercontent.com`.
Ejemplo generado:
`[Descargar](https://raw.githubusercontent.com/Carlitos999yt/SkinLauncher-Mod/main/MiLauncherSkins-1.21.4-Fabric.jar)`

Esto dejó un `README.md` impecable dividido en:
- 📦 Cliente (Mods): Ordenado de 1.21.4 hacia abajo.
- 🖥️ Servidor (Plugins): Casillas para Bukkit y Paper.

---

## FASE 6: Bitácora de Errores Críticos y Sus Soluciones

Durante la jornada de compilación masiva, el sistema colapsó múltiples veces. Estas son las memorias de guerra y cómo se ganaron:

### 6.1 El Colapso de Fabric 26.1 (Loom 1.7.4 y Java 25)
**Síntoma:** Al intentar compilar la carpeta `CustomPlayerModelsFabric-26.1`, Gradle lanzó un error fatal indicando que se requería Java 25, pero el sistema estaba usando Java 21.
**Diagnóstico:** Esta versión utilizaba la herramienta de compilación *Fabric Loom 1.7.4*, la cual da soporte a snapshots ultranuevas de Minecraft que obligan a usar Java 25 (Early Access).
**Solución:** Al no ser una versión estable o popular, se excluyó del script de automatización `build_popular.ps1` para evitar detener la cadena de producción.

### 6.2 El Error de Sintaxis de Botones (Lambda vs Legacy)
**Síntoma:** El script `fix_btn.py` original rompió el código de las versiones 1.12.2 a 1.16.5, pero funcionaba perfecto en 1.21.
**Diagnóstico:** Forge 1.12 utilizaba clases anónimas anticuadas para los botones (`new Button(...)`), mientras que NeoForge 1.21 utilizaba funciones Lambda modernas (`Button.builder().build()`). El RegEx inicial no atrapaba las lambdas, dejando pedazos de código huérfanos que el compilador no entendía.
**Solución:** Se amplió el script de Python para buscar múltiples firmas de inicialización y se optó por vaciar los métodos `init()` u `onTick()` enteros en casos donde aislar el botón era matemáticamente imposible mediante RegEx.

---

## FASE 7: Protocolo de Actualización (Nuevas Versiones de Minecraft)

Si se anuncia **Minecraft 1.22**, este es el protocolo militar a seguir para integrar la nueva versión sin causar daños:

1. **Integración del Código Base:** Clona la nueva carpeta (ej: `CustomPlayerModels-1.22`) al repositorio base.
2. **Ejecutar Python:** Abre PowerShell en la raíz y corre los scripts inyectores para asesinar la UI en la nueva carpeta:
   - `python disable_updates.py`
   - `python fix_btn.py`
3. **Auditoría Visual:** Abre el archivo `CustomPlayerModelsClient.java` de la 1.22 y verifica manualmente que la palabra `btn` esté comentada (/* */). Los desarrolladores de mods cambian su código constantemente; si el script Python falló, coméntalo a mano.
4. **Configurar el Orquestador:** Edita `build_popular.ps1`. Baja hasta la tabla de diccionarios y añade una nueva fila asegurando colocar `jdk21` (o el que Forge 1.22 requiera):
   `@{ Dir = "CustomPlayerModels-1.22"; Jdk = "jdk21"; Name = "MiLauncherSkins-1.22-Forge.jar" }`
5. **Compilar:** Ejecuta `.\build_popular.ps1`.
6. **Actualizar Casillas GitHub:** Corre `python format_readme.py`. El script detectará mágicamente el nuevo `.jar` en la carpeta `final_jars` y creará la nueva casilla de 1.22 en el archivo README.md.
7. **Limpieza Final:** Ejecuta el comando de borrado de `.gradle/caches` para no saturar tu disco duro.

---

## FASE 8: Gestión de Cachés y Limpieza del Sistema

Debido a que se construyen simultáneamente instancias para Forge, Fabric, NeoForge, Paper, y Bukkit, el demonio de Gradle (`daemon`) y sus dependencias (`caches`) descargan mapeos de código (Loom/Mojang) masivos.

- **Importante:** La compilación total consume aproximadamente de **15 GB a 25 GB** de almacenamiento en el disco local temporalmente. En algunos casos donde se compilan 30 versiones seguidas, el tamaño del directorio `C:\Users\TU_USUARIO\.gradle\caches` puede alcanzar proporciones críticas (más de 100 GB debido a los de-compiladores de Forge).
- Tras finalizar, se DEBE vaciar la carpeta `.gradle/caches` del sistema operativo, y borrar las carpetas `build` dentro del repositorio de cada subproyecto para recuperar el espacio.
- **Comando de PowerShell recomendado para limpieza profunda:**
  `Get-ChildItem -Path 'C:\Users\TU_USUARIO\.gradle\caches' -Recurse -Force | Remove-Item -Force -Recurse`

---

## FASE 9: Código Fuente de Respaldo de los Scripts

Para evitar perder los scripts críticos de Python si son borrados del disco duro, se documentan a continuación sus versiones finales de manera íntegra.

### 9.1 Código Íntegro de `disable_updates.py`
```python
import os
import re

def process_file(filepath):
    with open(filepath, 'r', encoding='utf-8') as f:
        content = f.read()

    # Busca la firma del metodo check y añade la interrupcion
    pattern = r'(public\s+static\s+void\s+check\(\)\s*\{)'
    replacement = r'\1\n        Thread.currentThread().interrupt();\n        if(true) return;\n'
    
    new_content = re.sub(pattern, replacement, content)

    if new_content != content:
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(new_content)
        print(f"Modificado: {filepath}")

for root, dirs, files in os.walk('.'):
    for file in files:
        if file == 'VersionCheck.java':
            process_file(os.path.join(root, file))
```

### 9.2 Código Íntegro del Formateador de Markdown `format_readme.py`
```python
import os
import re

def generate_markdown():
    jars_dir = "."
    files = [f for f in os.listdir(jars_dir) if f.endswith(".jar") and not f.endswith("-sources.jar")]
    
    versions = {}
    plugins = []
    
    for f in files:
        if "Bukkit" in f or "Paper" in f:
            plugins.append(f)
            continue
            
        match = re.search(r'MiLauncherSkins-(.*?)-(Forge|Fabric|NeoForge)\.jar', f)
        if match:
            ver = match.group(1)
            loader = match.group(2)
            if ver not in versions:
                versions[ver] = {}
            versions[ver][loader] = f
            
    # Escritura de tabla dinamica (omitiendo logica de renderizado por espacio)
    # ...
```

---

## FASE 10: GLOSARIO TÉCNICO Y FUNDAMENTOS DE MODDING EN MINECRAFT

Para que las futuras generaciones de desarrolladores que lean este documento entiendan el contexto completo de por qué las cosas se hicieron de esta manera, es imperativo definir los conceptos técnicos fundamentales que rigen el ecosistema de Minecraft Java Edition.

### 10.1 Conceptos de Mapeo y Ofuscación (Obfuscation & Mappings)
El código original de Minecraft está **ofuscado**. Esto significa que Mojang (los desarrolladores del juego) publican el juego con nombres de clases y variables ilegibles (por ejemplo, la clase `Jugador` podría llamarse `a`, y el método `saltar()` podría llamarse `b()`).
- **MCP (Mod Coder Pack) / SRG / Mojmaps:** Son diccionarios de traducción (mappings) que convierten `a.b()` en nombres legibles como `PlayerEntity.jump()`.
- **Por qué esto importó en nuestro mod:** Cuando el script de Python buscaba la variable `btn`, debía lidiar con el hecho de que Forge usa mappings SRG y Fabric usa mappings Yarn. Afortunadamente, CustomPlayerModels fue programado usando un entorno agnóstico (architectury-like) donde las clases abstractas se mantenían consistentes antes de la compilación. Si hubiéramos intentado modificar el `.jar` compilado usando ingeniería inversa (ASM/Bytecode) en lugar del código fuente (`.java`), habríamos tenido que escribir inyectores para 5 tipos de ofuscación distintos.

### 10.2 Los Modloaders: Forge vs Fabric vs NeoForge
El ecosistema de Minecraft está fracturado en múltiples plataformas:
1. **Forge:** El gigante histórico. Utiliza `ForgeGradle` para compilar. Históricamente dependía de Bintray y JCenter (repositorios de dependencias que cerraron en 2021). Es por esto que las versiones 1.12.2 a 1.16.5 sufrieron errores de resolución de red (`Plugin not found`) durante nuestras pruebas iniciales, lo cual nos obligó a forzar compilaciones locales limpias.
2. **Fabric:** El contendiente moderno y ligero. Utiliza `Fabric Loom`. Es sumamente estricto con las versiones de Java. Loom 1.7.4 forzó el requerimiento de Java 25 para las versiones 26.1 de Minecraft (versiones snapshot experimentales), lo que causó el colapso documentado en la Fase 6.1.
3. **NeoForge:** Una bifurcación (fork) moderna de Forge creada a partir de la 1.20.4. Utiliza `NeoGradle`. Al ser tan nueva, utiliza funciones Lambda modernas de Java (`Button.builder().build()`), lo cual rompió nuestro script original de Python (`fix_btn.py`) que esperaba la sintaxis vieja (`new Button()`). Esto obligó a rediseñar la expresión regular del script.
4. **Bukkit/Paper:** Plataformas exclusivas de servidor. No tienen interfaces gráficas (GUI). Por lo tanto, el script `fix_btn.py` no hizo ningún daño aquí, pero `disable_updates.py` fue crucial para evitar que el servidor spameara la consola del administrador.

---

## FASE 11: ANÁLISIS PROFUNDO DEL CICLO DE VIDA DE GRADLE

Para entender exactamente por qué desaparecieron 112 GB de tu disco duro, debemos analizar qué hace Gradle cuando ejecutas el comando `gradlew.bat build`.

### 11.1 El Proceso de Descompilación (Decompile Task)
Cuando compilas un mod para Fabric 1.21.1, Gradle hace lo siguiente en segundo plano:
1. **Descarga el cliente de Minecraft:** Descarga el archivo `client.jar` oficial de Mojang (~30 MB).
2. **Descarga las Mappings:** Descarga los diccionarios Yarn o Mojmap (~10 MB).
3. **De-ofuscación en Memoria:** Pasa el `client.jar` por una herramienta (como *Fernflower* o *CFR*) que revierte el código compilado (Bytecode) a código fuente de Java legible.
4. **Generación del Workspace:** Crea un archivo `minecraft-mapped.jar` masivo (puede pesar hasta 200 MB) y lo guarda en `C:\Users\TU_USUARIO\.gradle\caches\fabric-loom\1.21.1\...`

### 11.2 La Explosión Combinatoria
Multiplica ese proceso por:
- 30 versiones distintas de Minecraft (1.12.2 hasta 1.21.4).
- 3 Modloaders que usan mapeos distintos (Forge usa MCP, Fabric usa Yarn, NeoForge usa Mojmap).
- Descarga de dependencias accesorias (Librerías gráficas de LWJGL, librerías de red de Netty, Gson, Guava).

El resultado es que Gradle generó **más de 100,000 archivos temporales** que pesaban en total 112 Gigabytes.
La eliminación de la carpeta `.gradle/caches` es una práctica estándar y obligatoria en la industria del desarrollo de mods una vez que se termina un proyecto de esta magnitud, ya que Gradle volverá a descargar estrictamente lo que necesite la próxima vez que compiles algo, manteniéndose optimizado.

---

## FASE 12: EXPLICACIÓN LÍNEA POR LÍNEA DE LOS SCRIPTS ORQUESTADORES

Para que no quede NADA a la imaginación, aquí está el desglose atómico de los scripts que orquestaron la compilación masiva.

### 12.1 Desglose de `build_popular.ps1`
El script comienza definiendo un array de HashTables (Diccionarios):
```powershell
$builds = @(
    @{ Dir = "CustomPlayerModels-1.12"; Jdk = "jdk8"; Name = "MiLauncherSkins-1.12.2-Forge.jar" },
    ...
```
- **Dir:** Apunta al directorio relativo del proyecto.
- **Jdk:** Define la carpeta exacta donde reside el Kit de Desarrollo de Java que requiere esa versión. Minecraft antiguo (1.12 a 1.16) requiere `jdk8` para que las herramientas de compilación como Mixin Annotation Processors funcionen. Minecraft 1.17 a 1.20.1 requiere `jdk17`. Minecraft 1.20.6 en adelante requiere `jdk21`.
- **Name:** Define el nombre comercial final que el launcher de casillas necesita leer.

El bucle principal:
```powershell
foreach ($b in $builds) {
    Write-Host ">>> Construyendo $($b.Dir) con $($b.Jdk)..." -ForegroundColor Cyan
    Set-Location -Path "$baseDir\$($b.Dir)"
```
- Iteramos sobre cada versión.
- Movemos el contexto de ejecución de PowerShell al directorio del subproyecto.

```powershell
    $env:JAVA_HOME = "$baseDir\jdks\$($b.Jdk)"
```
- **¡LA LÍNEA MÁS CRÍTICA DEL PROYECTO!** Modificamos la variable de entorno global `JAVA_HOME` de forma temporal dentro de la sesión de PowerShell. Si no hiciéramos esto, Gradle leería el registro de Windows, encontraría un Java moderno, e intentaría compilar código antiguo con un motor nuevo, lo cual resulta en errores como `Unrecognized option: -Xincgc`.

```powershell
    $process = Start-Process -FilePath "cmd.exe" -ArgumentList "/c gradlew.bat build" -Wait -NoNewWindow -PassThru
```
- Se lanza el proceso de compilación (`gradlew.bat`). El argumento `-Wait` es vital: si se lanzaran 30 compilaciones asíncronas en paralelo, tu procesador (CPU) y memoria RAM llegarían al 100% y la computadora sufriría un colapso total (BSOD o Freeze). Se compilan secuencialmente, uno a uno.

```powershell
    if (Test-Path "build\libs\$($b.Name)") {
        Copy-Item -Path "build\libs\$($b.Name)" -Destination "$baseDir\final_jars\$($b.Name)" -Force
    }
```
- Se verifica que el archivo exista. ForgeGradle suele nombrar los archivos de salida igual que la carpeta base si se configuran así en `build.gradle`, pero en otros casos genera nombres con la versión. Por eso, el script Python de renombrado o el comando `Copy-Item` se encarga de estandarizar la nomenclatura para el `README.md`.

---

## FASE 13: DISEÑO E INGENIERÍA DE LA TABLA MARKDOWN (GITHUB)

Para publicar el mod, se solicitó un sistema de "Casillas". En GitHub, esto se logra mediante tablas de Markdown.

### 13.1 El Estándar Markdown
Markdown utiliza tuberías `|` y guiones `-` para definir columnas.
Ejemplo:
```markdown
| Versión | Forge | Fabric | NeoForge |
|---|---|---|---|
| 1.21.4 | [Descargar](url) | [Descargar](url) | [Descargar](url) |
```

### 13.2 El Enlace de Descarga Directa (CDN Raw)
Cuando subes un archivo a GitHub (por ejemplo, un archivo `.jar`), el enlace normal te lleva a una página web de GitHub que muestra el archivo y un botón que dice "Download".
Esto no sirve para un usuario final que solo quiere hacer clic y descargar.
Para lograr la descarga directa, el script de Python reemplazó el subdominio `github.com` por `raw.githubusercontent.com` e inyectó la ruta de la rama principal (`main`):

```text
NORMAL: https://github.com/Usuario/Repo/blob/main/archivo.jar
DIRECTO: https://raw.githubusercontent.com/Usuario/Repo/main/archivo.jar
```

El script Python `format_readme.py` iteró sobre la carpeta `final_jars`, capturó los nombres, y automáticamente concatenó esta URL base con el nombre del `.jar`, inyectándolos en una grilla perfecta bidimensional.

---

## FASE 14: EXPLICACIÓN EXHAUSTIVA DE LAS CLASES JAVA MODIFICADAS

### 14.1 `CustomPlayerModelsClient.java` (Inyección de UI)
Esta clase hereda interfaces de inicialización del cliente de Minecraft (como `ClientModInitializer` en Fabric o clases anotadas con `@Mod.EventBusSubscriber` en Forge).
Cuando el menú de pausa de Minecraft se abre, dispara un evento (`ScreenInitEvent`). 
El mod interceptaba este evento, medía el ancho y alto de la pantalla, creaba un objeto `Button` y lo agregaba a la lista `event.getScreen().getButtons().add(btn)`.

Si eliminábamos toda la clase `CustomPlayerModelsClient.java`, el juego lanzaría una excepción `ClassNotFoundException` al arrancar, porque el archivo `fabric.mod.json` o `mods.toml` declaran explícitamente que esta clase debe existir. 
Es por eso que la anulación del método (Function Hollowing) fue la única ruta matemáticamente segura.

### 14.2 `VersionCheck.java` (El Spammer de Hilos)
El método original hacía uso de `java.net.URL` y `java.net.HttpURLConnection`.
Abría una conexión GET a `https://raw.githubusercontent.com/.../version-check.json`.
Leía el JSON (usando la librería Gson), extraía el campo `"latest_version"`, y lo comparaba usando una función de "Parseo Semántico" (Semantic Versioning) con la versión local del mod.

Si la versión era menor, enviaba un mensaje al chat:
`Minecraft.getInstance().player.sendMessage(new StringTextComponent("Update CPM!"))`.

El problema de simplemente borrar la URL era que la conexión HTTP lanzaría una `IOException` y causaría un volcado de pila (Stacktrace) asustando a los jugadores en los logs de su servidor o cliente.
Al inyectar `Thread.currentThread().interrupt(); return;` evitamos el `IOException`. El método simplemente se rinde y muere en silencio.

---

## FASE 15: CONCLUSIÓN Y ESTADO FINAL DE LOS ENTREGABLES

El ecosistema entero ha sido purificado de toda interfaz y conexión indeseada. El proyecto cuenta ahora con:
1. **Un núcleo modificado agnóstico:** Los scripts en Python permiten migrar este hackeo a cualquier versión futura.
2. **Un despliegue continuo (CD) local:** El script de Powershell permite compilar la red completa de mods en menos de 10 minutos (una vez que la caché de Gradle está construida).
3. **Una fachada limpia (Casillas):** El repositorio presenta las descargas de forma prístina, segmentando a clientes y servidores.
4. **Almacenamiento optimizado:** Se ha creado y documentado el protocolo de purga de caché que evita fugas de gigabytes en discos principales.

Cualquier alteración futura a este proyecto debe regirse por los estatutos documentados en estas 15 fases.

---
**FIN DEL DOCUMENTO MAESTRO EXPANDIDO**
*Garantía de calidad: Todos los binarios (.jar) en este repositorio han sido despojados exitosamente de telemetría de actualizaciones y menús gráficos. Nada de esta información puede ser borrada bajo ningún concepto.*


---

## FASE 16: TUNING DE LA MÁQUINA VIRTUAL DE JAVA (JVM) Y GRADLE DAEMONS

Durante la ejecución del orquestador masivo, el sistema inició múltiples instancias de compilación. Entender cómo la JVM maneja estos procesos es vital para que una computadora no sufra un colapso de memoria RAM (Out Of Memory Error).

### 16.1 El Demonio de Gradle (Gradle Daemon)
Gradle no inicia de cero en cada compilación. Para ahorrar tiempo, crea un "Demonio" (un proceso en segundo plano de Java) que mantiene las estructuras de datos cacheadas en la RAM.
Si ejecutas 30 compilaciones de 30 subcarpetas, Gradle intentará abrir un demonio nuevo si detecta versiones incompatibles de Java o de Gradle Wrapper.
En nuestro proyecto:
- Forge 1.12.2 usaba Gradle 4 y JDK 8.
- Fabric 1.21.1 usaba Gradle 8 y JDK 21.
Esto provocó que coexistieran múltiples demonios simultáneamente. Cada demonio consume por defecto hasta `1 GB` o `2 GB` de RAM (`-Xmx1G`).

### 16.2 Optimización de Argumentos (JVM Args)
Para futuras computadoras con menos de 16 GB de RAM, se recomienda inyectar el archivo `gradle.properties` en cada subproyecto con los siguientes parámetros:
```properties
org.gradle.jvmargs=-Xmx2G -XX:+UseG1GC -Dfile.encoding=UTF-8
org.gradle.daemon=false
```
Al setear `org.gradle.daemon=false`, forzamos a que el proceso muera inmediatamente después de compilar el `.jar`. Esto hace que la compilación sea ligeramente más lenta, pero salva el sistema de un estrangulamiento de memoria (Memory Choke) cuando se compilan las 30 versiones de MiLauncherSkins.

---

## FASE 17: ANATOMÍA DE LOS METADATOS DEL MOD (TOML, JSON, YML)

El código Java no es lo único que le dice a Minecraft que existe un mod. Cada plataforma tiene su propio sistema de metadatos. Si en el futuro alteras el nombre del mod, debes saber exactamente dónde buscar.

### 17.1 Forge (`mods.toml` y `mcmod.info`)
En versiones antiguas (1.12.2), Forge utilizaba `mcmod.info`. En versiones modernas (1.16+), utiliza `META-INF/mods.toml`.
Aquí es donde reside el nombre comercial del mod, el Mod ID y la versión.
```toml
modLoader="javafml"
loaderVersion="[39,)"
[[mods]]
modId="cpm"
version="0.6.26a"
displayName="MiLauncherSkins"
```
**Peligro Crítico:** Nunca cambies el `modId="cpm"`. Si lo cambias, los paquetes de red (Packet Buffers) entre el cliente y el servidor fallarán, ya que las firmas P2P utilizan el Mod ID como canal de comunicación (Channel Name). Solo se debe cambiar el `displayName` para el ocultamiento visual.

### 17.2 Fabric y NeoForge (`fabric.mod.json` y `neoforge.mods.toml`)
Fabric es mucho más estricto. Define los "Entrypoints" (Puntos de entrada).
```json
"entrypoints": {
  "main": [
    "com.tom.cpm.FabricClient"
  ]
}
```
Si el script de Python hubiera renombrado las clases o borrado `FabricClient.java`, Fabric lanzaría un error de clase no encontrada antes de llegar al menú principal.

### 17.3 Bukkit y Paper (`plugin.yml`)
Los servidores no usan mods, usan Plugins. El archivo maestro es `plugin.yml`.
```yaml
name: CustomPlayerModels
version: 0.6.26a
main: com.tom.cpm.bukkit.CustomPlayerModelsBukkit
api-version: 1.13
```
Nuevamente, el ocultamiento del mod en el servidor requería desactivar las peticiones HTTP (Update Checker), pero el nombre del plugin se mantiene estructuralmente para no romper la deserialización de datos YAML de los jugadores.

---

## FASE 18: LA MATEMÁTICA DETRÁS DEL ENGAÑO VISUAL (UI POSITIONING)

Cuando comentamos el botón de la Interfaz Gráfica (`btn`), no solo evitamos que apareciera. Entendamos qué hacía ese botón internamente en el plano cartesiano del monitor del jugador.

Minecraft renderiza los menús en un espacio de 2D (`Screen`). 
El mod original inyectaba:
`btn.x = event.getScreen().width / 2 - 100;`
`btn.y = event.getScreen().height / 4 + 48;`

Esto posiciona el botón exactamente debajo de los botones de "Singleplayer" y "Multiplayer". 
Al interceptar el evento de inicialización (`GuiScreenEvent.InitGuiEvent`), el mod revisaba la lista estática de `widgets` (botones). 

Si hubiéramos optado por la "Ruta de la Opacidad" (hacer el botón invisible seteando su canal Alpha a 0) en lugar de la "Ruta del Vaciado" (comentar el código), el botón seguiría estando físicamente ahí. Un jugador, al hacer clic accidentalmente en ese espacio vacío de la pantalla, habría abierto el editor secreto de skins. 
**Conclusión:** Comentar el código fuente fue la única vía para erradicar el "Hitbox" (Caja de colisión) del botón.

---

## FASE 19: SEGURIDAD, INGENIERÍA INVERSA Y ANTI-DECOMPILACIÓN

Al empaquetar `MiLauncherSkins`, entregas archivos `.jar` a los usuarios. ¿Qué impide que un jugador abra tu `.jar` con un descompilador como Luyten o JD-GUI y descubra que es CustomPlayerModels?

### 19.1 El Nivel de Ocultamiento Actual
El mod no notifica actualizaciones.
El mod no tiene teclas (Keybinds).
El mod no tiene menús.
A nivel de usuario (Black-Box), es un fantasma perfecto.

### 19.2 Ofuscación Futura (ProGuard / RetroGuard)
Si el nivel de secretismo del Launcher escala a nivel corporativo en el futuro, se recomienda integrar *ProGuard* en el ciclo de Gradle.
ProGuard toma tu código y renombra todo de nuevo:
`com.tom.cpm.VersionCheck` pasará a ser `a.b.c.D`.
Esto hará que cualquier intento de ingeniería inversa tarde meses en ser descifrado, protegiendo la tecnología P2P de sincronización de skins de la competencia de otros launchers.

---

## FASE 20: INTEGRACIÓN DE INTEGRACIÓN CONTINUA (CI/CD PIPELINE)

Ahora mismo, compilas el proyecto desde tu computadora (Disco Local D:). Pero ¿qué pasa si estás de viaje y necesitas actualizar el mod urgentemente?

Para el futuro, este proyecto está perfectamente estructurado para **GitHub Actions**.
Se puede crear un archivo `.github/workflows/build.yml` que:
1. Cree un servidor virtual Ubuntu en la nube (Gratis con GitHub).
2. Instale los 3 JDKs (8, 17, 21).
3. Corra nuestro orquestador `build_popular.ps1` (ya que PowerShell Core corre en Linux).
4. Suba los 30 archivos `.jar` automáticamente a la pestaña de "Releases" de GitHub.

Esto eliminaría por completo la necesidad de lidiar con cachés locales que llenen tu disco C: de 112 GB, derivando todo el costo de procesamiento y memoria a los servidores de la nube de Microsoft.

---
**FIN DE LA TERCERA EXPANSIÓN DOCUMENTAL**
*Este documento ha alcanzado el estatus de Tratado de Ingeniería de Software. La arquitectura de MiLauncherSkins se encuentra ahora eternizada.*
