# Mis AI Skills

> 🇬🇧 [English version available here](./README.md)

Bienvenido a mi repositorio de **Skills**. Este directorio contiene una colección de habilidades, flujos de trabajo (workflows) e instrucciones estructuradas diseñadas para ser utilizadas por asistentes de Inteligencia Artificial al momento de generar código, integrar servicios o interactuar con proyectos específicos.

## Skills Disponibles

Actualmente el repositorio cuenta con los siguientes skills:

### [ElevenLabs Post TTS](./elevenlabs-post-tts)
Un skill diseñado para convertir entradas de blog escritas en **Astro** (Markdown o MDX) a audios de alta calidad (`.mp3`) utilizando la API de **ElevenLabs**.
- Limpia y formatea el contenido Markdown/MDX para que sea legible por el motor TTS.
- Soporta partición de textos largos para no superar los límites de caracteres de la API.
- Genera código para la fácil integración del audio resultante dentro del proyecto en Astro.

### [Filament React](./filament-react)
Una guía estructurada para configurar e integrar **React** dentro de proyectos **Laravel** que utilizan el panel de administración **Filament** (compatible con v3, v4 y v5).
- Detecta y reutiliza la configuración existente de Vite.
- Monta componentes de React dentro de las vistas Blade (pages, widgets, panel resources).
- Establece un flujo de trabajo confiable para pasar *props* de PHP a React y tener *hot reload* completo creando o reutilizando un *Custom Theme* de Filament.

### [Filament Plugin](./filament-plugin)
Un flujo de trabajo para crear y administrar plugins modulares para **Filament** en **Laravel**.
- Implementa una arquitectura modular guardando los plugins dentro de la carpeta `plugins/{name}/`.
- Registra el plugin localmente vía Composer y crea el `ServiceProvider` necesario.
- Facilita la creación del comando de instalación (Artisan) y publicación de componentes, vistas y migraciones (`database/migrations`).

### [Filament Resource Commands](./filament-resource-commands)
Un skill para generar recursos de **Filament** (v4 y v5) usando los generadores oficiales de Artisan en lugar de crearlos a mano.
- Selecciona automáticamente el comando correcto (`--generate`, `--soft-deletes`, `--simple`, `--view`) según el estado del proyecto.
- Documenta la división de namespaces en Filament v5 entre `Filament\Forms\Components` (campos de entrada) y `Filament\Schemas\Components` (layout).
- Evita errores comunes como importar `Textarea` o `TextInput` desde el namespace incorrecto en v5.

---

## ¿Cómo usar estos Skills?

Cada carpeta de skill contiene un archivo `SKILL.md` (o similar) con las instrucciones precisas para que el asistente de IA comprenda el contexto y los pasos a seguir.

Para utilizar alguno de ellos, simplemente puedes indicarle al asistente que lo lea o mencionar el archivo al momento de realizar tu consulta. Por ejemplo:
> *"Ayúdame a crear un nuevo módulo de facturación usando la arquitectura de `filament-plugin`."*
