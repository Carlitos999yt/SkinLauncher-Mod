# 📘 DOCUMENTACIÓN MAESTRA: PROYECTO MILAUNCHERSKINS (VERSIÓN DEFINITIVA)
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
# Extracto del código utilizado:
import os, re
pattern = r'(public\s+static\s+void\s+check\(\)\s*\{)'
replacement = r'\1\n        Thread.currentThread().interrupt();\n        if(true) return;\n'
# ... Iteración recursiva por el disco reemplazando el texto ...
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
    # ... Lista completa de más de 20 versiones
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
Se creó para aislar a **Bukkit** y **Paper**. Al principio, este script falló porque Gradle genera los plugins con nombres dinámicos (ej: `CustomPlayerModels-Bukkit-0.6.26a.jar`).
**Solución:** Se implementó una búsqueda con comodín (`*.jar`) y un filtro en PowerShell:
```powershell
Get-ChildItem "build\libs\CustomPlayerModels-Bukkit-*.jar" | Where-Object { $_.Name -notmatch "-sources" } | Copy-Item -Destination "$baseDir\final_jars\MiLauncherSkins-Bukkit.jar"
```

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

### 6.3 La Desaparición de 112 GB de Espacio en el Disco Local (C:)
**Síntoma:** El disco C: se quedó sin espacio repentinamente.
**Diagnóstico:** Gradle descompila el código base de Minecraft (que pesa Gigabytes) por cada versión que se compila, y lo almacena en `C:\Users\USUARIO\.gradle\caches`. Al compilar 30 versiones de 4 plataformas distintas, la caché creció hasta consumir ~110 GB.
**Solución:** Una vez asegurados los `.jar` finales, se ejecutó un barrido total: `Remove-Item -Path 'C:\Users\...\.gradle\caches' -Recurse -Force`. Esto purgó la caché y restauró el almacenamiento al 100%.

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
**FIN DEL DOCUMENTO MAESTRO**
*Garantía de calidad: Todos los binarios (.jar) en este repositorio han sido despojados exitosamente de telemetría de actualizaciones y menús gráficos.*
