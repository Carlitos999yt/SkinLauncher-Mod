# MiLauncher Skins Mod

![Skins](https://img.shields.io/badge/Minecraft-Skins-blue) ![Versions](https://img.shields.io/badge/Versions-1.12.2%20|%201.16.5%20|%201.20.1%20|%201.21.1-green)

Este es el repositorio oficial de almacenamiento de **MiLauncher Skins**.

Este mod permite cambiar y personalizar tu skin de Minecraft. 
Está diseñado para funcionar en segundo plano de manera completamente transparente.

**Novedades en esta versión:**
- Interfaz gráfica oculta para mayor inmersión.
- Búsqueda de actualizaciones automáticas desactivada por completo.

Estos mods se encargan de sincronizar las skins personalizadas de todos los jugadores de forma directa (Peer-to-Peer) a través de los paquetes de red del servidor de Minecraft, eliminando por completo la necesidad de un servidor web externo o una base de datos centralizada.

## 🚀 ¿Cómo funciona?

1. **Inyección Dinámica**: MiLauncher descarga automáticamente el `.jar` correspondiente a la versión de Minecraft que el jugador seleccione justo antes de iniciar el juego.
2. **Sincronización P2P**: Una vez dentro de un servidor multijugador, el mod inyecta la textura de la skin del jugador en los paquetes de red enviados al servidor y se distribuye a los clientes cercanos.
3. **Cero Latencia**: Al usar el mismo protocolo de datos de Minecraft, las skins cargan al instante para todos los jugadores cercanos.

## 📥 Descargas Automáticas (CDN)

El launcher utiliza los siguientes enlaces Raw (CDN) para descargar los mods en tiempo real. 
*(No es necesario que los descargues manualmente si usas MiLauncher)*:

- [1.12.2 Forge](https://raw.githubusercontent.com/Carlitos999yt/SkinLauncher-Mod/main/MiLauncherSkins-1.12.2-Forge.jar)
- [1.16.5 Forge](https://raw.githubusercontent.com/Carlitos999yt/SkinLauncher-Mod/main/MiLauncherSkins-1.16.5-Forge.jar)
- [1.20.1 Forge](https://raw.githubusercontent.com/Carlitos999yt/SkinLauncher-Mod/main/MiLauncherSkins-1.20.1-Forge.jar)
- [1.21.1 Forge](https://raw.githubusercontent.com/Carlitos999yt/SkinLauncher-Mod/main/MiLauncherSkins-1.21.1-Forge.jar)

## 🏆 Créditos
Este sistema de red ultra-eficiente está construido sobre el legendario motor de transferencia de red de **[Customizable Player Models (CPM)](https://github.com/tom5454/CustomPlayerModels)** creado por tom5454. Todos los créditos de la ingeniería de red y los payloads P2P van hacia su trabajo original.
