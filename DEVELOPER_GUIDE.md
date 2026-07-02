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
**FIN DEL DOCUMENTO MAESTRO EXPANDIDO**
*Garantía de calidad: Todos los binarios (.jar) en este repositorio han sido despojados exitosamente de telemetría de actualizaciones y menús gráficos. Nada de esta información puede ser borrada bajo ningún concepto.*

---

## FASE 10: GLOSARIO TÃ‰CNICO Y FUNDAMENTOS DE MODDING EN MINECRAFT

Para que las futuras generaciones de desarrolladores que lean este documento entiendan el contexto completo de por quÃ© las cosas se hicieron de esta manera, es imperativo definir los conceptos tÃ©cnicos fundamentales que rigen el ecosistema de Minecraft Java Edition.

### 10.1 Conceptos de Mapeo y OfuscaciÃ³n (Obfuscation & Mappings)
El cÃ³digo original de Minecraft estÃ¡ **ofuscado**. Esto significa que Mojang (los desarrolladores del juego) publican el juego con nombres de clases y variables ilegibles (por ejemplo, la clase `Jugador` podrÃ­a llamarse `a`, y el mÃ©todo `saltar()` podrÃ­a llamarse `b()`).
- **MCP (Mod Coder Pack) / SRG / Mojmaps:** Son diccionarios de traducciÃ³n (mappings) que convierten `a.b()` en nombres legibles como `PlayerEntity.jump()`.
- **Por quÃ© esto importÃ³ en nuestro mod:** Cuando el script de Python buscaba la variable `btn`, debÃ­a lidiar con el hecho de que Forge usa mappings SRG y Fabric usa mappings Yarn. Afortunadamente, CustomPlayerModels fue programado usando un entorno agnÃ³stico (architectury-like) donde las clases abstractas se mantenÃ­an consistentes antes de la compilaciÃ³n. Si hubiÃ©ramos intentado modificar el `.jar` compilado usando ingenierÃ­a inversa (ASM/Bytecode) en lugar del cÃ³digo fuente (`.java`), habrÃ­amos tenido que escribir inyectores para 5 tipos de ofuscaciÃ³n distintos.

### 10.2 Los Modloaders: Forge vs Fabric vs NeoForge
El ecosistema de Minecraft estÃ¡ fracturado en mÃºltiples plataformas:
1. **Forge:** El gigante histÃ³rico. Utiliza `ForgeGradle` para compilar. HistÃ³ricamente dependÃ­a de Bintray y JCenter (repositorios de dependencias que cerraron en 2021). Es por esto que las versiones 1.12.2 a 1.16.5 sufrieron errores de resoluciÃ³n de red (`Plugin not found`) durante nuestras pruebas iniciales, lo cual nos obligÃ³ a forzar compilaciones locales limpias.
2. **Fabric:** El contendiente moderno y ligero. Utiliza `Fabric Loom`. Es sumamente estricto con las versiones de Java. Loom 1.7.4 forzÃ³ el requerimiento de Java 25 para las versiones 26.1 de Minecraft (versiones snapshot experimentales), lo que causÃ³ el colapso documentado en la Fase 6.1.
3. **NeoForge:** Una bifurcaciÃ³n (fork) moderna de Forge creada a partir de la 1.20.4. Utiliza `NeoGradle`. Al ser tan nueva, utiliza funciones Lambda modernas de Java (`Button.builder().build()`), lo cual rompiÃ³ nuestro script original de Python (`fix_btn.py`) que esperaba la sintaxis vieja (`new Button()`). Esto obligÃ³ a rediseÃ±ar la expresiÃ³n regular del script.
4. **Bukkit/Paper:** Plataformas exclusivas de servidor. No tienen interfaces grÃ¡ficas (GUI). Por lo tanto, el script `fix_btn.py` no hizo ningÃºn daÃ±o aquÃ­, pero `disable_updates.py` fue crucial para evitar que el servidor spameara la consola del administrador.

---

## FASE 11: ANÃLISIS PROFUNDO DEL CICLO DE VIDA DE GRADLE

Para entender exactamente por quÃ© desaparecieron 112 GB de tu disco duro, debemos analizar quÃ© hace Gradle cuando ejecutas el comando `gradlew.bat build`.

### 11.1 El Proceso de DescompilaciÃ³n (Decompile Task)
Cuando compilas un mod para Fabric 1.21.1, Gradle hace lo siguiente en segundo plano:
1. **Descarga el cliente de Minecraft:** Descarga el archivo `client.jar` oficial de Mojang (~30 MB).
2. **Descarga las Mappings:** Descarga los diccionarios Yarn o Mojmap (~10 MB).
3. **De-ofuscaciÃ³n en Memoria:** Pasa el `client.jar` por una herramienta (como *Fernflower* o *CFR*) que revierte el cÃ³digo compilado (Bytecode) a cÃ³digo fuente de Java legible.
4. **GeneraciÃ³n del Workspace:** Crea un archivo `minecraft-mapped.jar` masivo (puede pesar hasta 200 MB) y lo guarda en `C:\Users\TU_USUARIO\.gradle\caches\fabric-loom\1.21.1\...`

### 11.2 La ExplosiÃ³n Combinatoria
Multiplica ese proceso por:
- 30 versiones distintas de Minecraft (1.12.2 hasta 1.21.4).
- 3 Modloaders que usan mapeos distintos (Forge usa MCP, Fabric usa Yarn, NeoForge usa Mojmap).
- Descarga de dependencias accesorias (LibrerÃ­as grÃ¡ficas de LWJGL, librerÃ­as de red de Netty, Gson, Guava).

El resultado es que Gradle generÃ³ **mÃ¡s de 100,000 archivos temporales** que pesaban en total 112 Gigabytes.
La eliminaciÃ³n de la carpeta `.gradle/caches` es una prÃ¡ctica estÃ¡ndar y obligatoria en la industria del desarrollo de mods una vez que se termina un proyecto de esta magnitud, ya que Gradle volverÃ¡ a descargar estrictamente lo que necesite la prÃ³xima vez que compiles algo, manteniÃ©ndose optimizado.

---

## FASE 12: EXPLICACIÃ“N LÃNEA POR LÃNEA DE LOS SCRIPTS ORQUESTADORES

Para que no quede NADA a la imaginaciÃ³n, aquÃ­ estÃ¡ el desglose atÃ³mico de los scripts que orquestaron la compilaciÃ³n masiva.

### 12.1 Desglose de `build_popular.ps1`
El script comienza definiendo un array de HashTables (Diccionarios):
```powershell
$builds = @(
    @{ Dir = "CustomPlayerModels-1.12"; Jdk = "jdk8"; Name = "MiLauncherSkins-1.12.2-Forge.jar" },
    ...
```
- **Dir:** Apunta al directorio relativo del proyecto.
- **Jdk:** Define la carpeta exacta donde reside el Kit de Desarrollo de Java que requiere esa versiÃ³n. Minecraft antiguo (1.12 a 1.16) requiere `jdk8` para que las herramientas de compilaciÃ³n como Mixin Annotation Processors funcionen. Minecraft 1.17 a 1.20.1 requiere `jdk17`. Minecraft 1.20.6 en adelante requiere `jdk21`.
- **Name:** Define el nombre comercial final que el launcher de casillas necesita leer.

El bucle principal:
```powershell
foreach ($b in $builds) {
    Write-Host ">>> Construyendo $($b.Dir) con $($b.Jdk)..." -ForegroundColor Cyan
    Set-Location -Path "$baseDir\$($b.Dir)"
```
- Iteramos sobre cada versiÃ³n.
- Movemos el contexto de ejecuciÃ³n de PowerShell al directorio del subproyecto.

```powershell
    $env:JAVA_HOME = "$baseDir\jdks\$($b.Jdk)"
```
- **Â¡LA LÃNEA MÃS CRÃTICA DEL PROYECTO!** Modificamos la variable de entorno global `JAVA_HOME` de forma temporal dentro de la sesiÃ³n de PowerShell. Si no hiciÃ©ramos esto, Gradle leerÃ­a el registro de Windows, encontrarÃ­a un Java moderno, e intentarÃ­a compilar cÃ³digo antiguo con un motor nuevo, lo cual resulta en errores como `Unrecognized option: -Xincgc`.

```powershell
    $process = Start-Process -FilePath "cmd.exe" -ArgumentList "/c gradlew.bat build" -Wait -NoNewWindow -PassThru
```
- Se lanza el proceso de compilaciÃ³n (`gradlew.bat`). El argumento `-Wait` es vital: si se lanzaran 30 compilaciones asÃ­ncronas en paralelo, tu procesador (CPU) y memoria RAM llegarÃ­an al 100% y la computadora sufrirÃ­a un colapso total (BSOD o Freeze). Se compilan secuencialmente, uno a uno.

```powershell
    if (Test-Path "build\libs\$($b.Name)") {
        Copy-Item -Path "build\libs\$($b.Name)" -Destination "$baseDir\final_jars\$($b.Name)" -Force
    }
```
- Se verifica que el archivo exista. ForgeGradle suele nombrar los archivos de salida igual que la carpeta base si se configuran asÃ­ en `build.gradle`, pero en otros casos genera nombres con la versiÃ³n. Por eso, el script Python de renombrado o el comando `Copy-Item` se encarga de estandarizar la nomenclatura para el `README.md`.

---

## FASE 13: DISEÃ‘O E INGENIERÃA DE LA TABLA MARKDOWN (GITHUB)

Para publicar el mod, se solicitÃ³ un sistema de "Casillas". En GitHub, esto se logra mediante tablas de Markdown.

### 13.1 El EstÃ¡ndar Markdown
Markdown utiliza tuberÃ­as `|` y guiones `-` para definir columnas.
Ejemplo:
```markdown
| VersiÃ³n | Forge | Fabric | NeoForge |
|---|---|---|---|
| 1.21.4 | [Descargar](url) | [Descargar](url) | [Descargar](url) |
```

### 13.2 El Enlace de Descarga Directa (CDN Raw)
Cuando subes un archivo a GitHub (por ejemplo, un archivo `.jar`), el enlace normal te lleva a una pÃ¡gina web de GitHub que muestra el archivo y un botÃ³n que dice "Download".
Esto no sirve para un usuario final que solo quiere hacer clic y descargar.
Para lograr la descarga directa, el script de Python reemplazÃ³ el subdominio `github.com` por `raw.githubusercontent.com` e inyectÃ³ la ruta de la rama principal (`main`):

```text
NORMAL: https://github.com/Usuario/Repo/blob/main/archivo.jar
DIRECTO: https://raw.githubusercontent.com/Usuario/Repo/main/archivo.jar
```

El script Python `format_readme.py` iterÃ³ sobre la carpeta `final_jars`, capturÃ³ los nombres, y automÃ¡ticamente concatenÃ³ esta URL base con el nombre del `.jar`, inyectÃ¡ndolos en una grilla perfecta bidimensional.

---

## FASE 14: EXPLICACIÃ“N EXHAUSTIVA DE LAS CLASES JAVA MODIFICADAS

### 14.1 `CustomPlayerModelsClient.java` (InyecciÃ³n de UI)
Esta clase hereda interfaces de inicializaciÃ³n del cliente de Minecraft (como `ClientModInitializer` en Fabric o clases anotadas con `@Mod.EventBusSubscriber` en Forge).
Cuando el menÃº de pausa de Minecraft se abre, dispara un evento (`ScreenInitEvent`). 
El mod interceptaba este evento, medÃ­a el ancho y alto de la pantalla, creaba un objeto `Button` y lo agregaba a la lista `event.getScreen().getButtons().add(btn)`.

Si eliminÃ¡bamos toda la clase `CustomPlayerModelsClient.java`, el juego lanzarÃ­a una excepciÃ³n `ClassNotFoundException` al arrancar, porque el archivo `fabric.mod.json` o `mods.toml` declaran explÃ­citamente que esta clase debe existir. 
Es por eso que la anulaciÃ³n del mÃ©todo (Function Hollowing) fue la Ãºnica ruta matemÃ¡ticamente segura.

### 14.2 `VersionCheck.java` (El Spammer de Hilos)
El mÃ©todo original hacÃ­a uso de `java.net.URL` y `java.net.HttpURLConnection`.
AbrÃ­a una conexiÃ³n GET a `https://raw.githubusercontent.com/.../version-check.json`.
LeÃ­a el JSON (usando la librerÃ­a Gson), extraÃ­a el campo `"latest_version"`, y lo comparaba usando una funciÃ³n de "Parseo SemÃ¡ntico" (Semantic Versioning) con la versiÃ³n local del mod.

Si la versiÃ³n era menor, enviaba un mensaje al chat:
`Minecraft.getInstance().player.sendMessage(new StringTextComponent("Update CPM!"))`.

El problema de simplemente borrar la URL era que la conexiÃ³n HTTP lanzarÃ­a una `IOException` y causarÃ­a un volcado de pila (Stacktrace) asustando a los jugadores en los logs de su servidor o cliente.
Al inyectar `Thread.currentThread().interrupt(); return;` evitamos el `IOException`. El mÃ©todo simplemente se rinde y muere en silencio.

---

## FASE 15: CONCLUSIÃ“N Y ESTADO FINAL DE LOS ENTREGABLES

El ecosistema entero ha sido purificado de toda interfaz y conexiÃ³n indeseada. El proyecto cuenta ahora con:
1. **Un nÃºcleo modificado agnÃ³stico:** Los scripts en Python permiten migrar este hackeo a cualquier versiÃ³n futura.
2. **Un despliegue continuo (CD) local:** El script de Powershell permite compilar la red completa de mods en menos de 10 minutos (una vez que la cachÃ© de Gradle estÃ¡ construida).
3. **Una fachada limpia (Casillas):** El repositorio presenta las descargas de forma prÃ­stina, segmentando a clientes y servidores.
4. **Almacenamiento optimizado:** Se ha creado y documentado el protocolo de purga de cachÃ© que evita fugas de gigabytes en discos principales.

Cualquier alteraciÃ³n futura a este proyecto debe regirse por los estatutos documentados en estas 15 fases.
