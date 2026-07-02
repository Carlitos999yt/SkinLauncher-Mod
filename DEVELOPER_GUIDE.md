# 🛠️ Guía de Desarrollo y Mantenimiento

Esta documentación explica detalladamente los pasos técnicos que se tomaron para modificar el código fuente de CustomPlayerModels, eliminar su interfaz gráfica (UI) y su buscador de actualizaciones automáticas, y cómo recompilar todas las versiones de manera automatizada. 

Esta guía servirá para futuras versiones o mantenimientos del mod.

---

## 1. Modificaciones al Código Fuente

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

## 2. Automatización (Scripts de Python)

Para aplicar las modificaciones de código en las más de 30 sub-versiones (1.12.2 hasta 1.21.4) de manera uniforme sin editar archivo por archivo, se utilizaron scripts de inyección en Python:

1. **`deep_clean.py` / `fix_btn.py`**:
   - Escanean recursivamente el directorio buscando todas las instancias de `CustomPlayerModelsClient.java`.
   - Utilizan expresiones regulares para identificar el bloque de código donde se instancia el botón de la UI (`btn = ...`) y los registros de las teclas.
   - Reemplazan el bloque con código vacío o comentado, asegurando que la compilación sea válida en Java.

2. **`disable_updates.py`**:
   - Busca `VersionCheck.java`.
   - Modifica el bloque inicial inyectando un `Thread.currentThread().interrupt(); return;` antes de que se establezca la conexión URL.

---

## 3. Proceso de Compilación (PowerShell)

El proceso de construcción y empaquetado final (.jar) se automatizó utilizando Gradle y varios scripts en PowerShell. 
Existen tres scripts fundamentales para el despliegue masivo:

1. **`build_popular.ps1`**: 
   - Itera a través de un diccionario configurado con las carpetas de las versiones más relevantes (ej: `1.12.2`, `1.16.5`, `1.18.2`, `1.20.1`, `1.21.1`) y sus respectivos JDKs requeridos (`jdk8`, `jdk17`, `jdk21`).
   - Setea la variable de entorno `$env:JAVA_HOME` para cada versión, previniendo errores de incompatibilidad entre versiones antiguas de Forge/Fabric y versiones nuevas de Java.
   - Ejecuta `gradlew.bat build`.
   - Copia los `.jar` finales resultantes excluyendo los archivos "-sources" y "-dev", y los mueve a la carpeta central `final_jars` renombrándolos al estándar (ej: `MiLauncherSkins-1.20.1-Forge.jar`).

2. **`build_plugins.ps1`**:
   - Realiza un proceso similar para los proyectos `CustomPlayerModels-Bukkit` y `CustomPlayerModels-Paper`.
   - Debido a diferencias en los sufijos generados, utiliza un comodín (`-*.jar`) para capturar el binario correcto y lo copia como `MiLauncherSkins-Bukkit.jar` y `MiLauncherSkins-Paper.jar`.

3. **`update_readme.py` y `format_readme.py`**:
   - Escanean la carpeta `final_jars` finalizada.
   - Construyen dinámicamente tablas en Markdown con los enlaces de descarga directos (Raw CDN) hacia el repositorio de GitHub.
   - Inyectan la tabla en `README.md`.

---

## 4. Limpieza de Memoria Caché (Caché de Gradle)

Debido a que se construyen simultáneamente instancias para Forge, Fabric, NeoForge, Paper, y Bukkit, el demonio de Gradle (`daemon`) y sus dependencias (`caches`) descargan mapeos de código (Loom/Mojang) masivos.

- **Importante:** La compilación total consume aproximadamente de **15 GB a 25 GB** de almacenamiento en el disco local temporalmente.
- Tras finalizar, se debe vaciar la carpeta `.gradle/caches` del sistema y las carpetas `build` dentro del repositorio para recuperar el espacio.

---
**Nota para el futuro:** Si se requiere portar el mod a Minecraft 1.22+, simplemente añade el módulo en el entorno de desarrollo, asegúrate de aplicar `disable_updates.py` y `fix_btn.py` sobre el nuevo código, agrégalo a la lista en `build_popular.ps1` con el JDK apropiado, y ejecuta la compilación automatizada.
