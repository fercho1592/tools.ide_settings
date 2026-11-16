Para adaptar este concepto a **GitHub Copilot**, utilizamos su arquitectura nativa de personalización:

1. **Instrucciones globales:** Se configuran en `.github/copilot-instructions.md` (reemplaza a `CLAUDE.md`).
2. **Instrucciones por ruta (*Path-specific*):** Se colocan en `.github/instructions/*.instructions.md` utilizando el bloque `applyTo` con globs (reemplazando el concepto de *skills* aisladas por carpetas).
3. **Comandos personalizados o *Prompt Files*:** Se ubican en `.github/prompts/*.prompt.md` (lo que equivale a los comandos `/sdd-*`).

A continuación, tienes la estructura base del archivo maestro adaptada para desplegarse de forma nativa con **GitHub Copilot** en un proyecto nuevo:

---

# GitHub Copilot SDD & Modular Framework - Master Setup Template

This master document contains the instructions for initializing and deploying the Spec-Driven Development (SDD) framework and layer-scoped rules for GitHub Copilot (`.github/copilot-instructions.md`, path-specific instructions, and prompt files).

[GitHub Copilot SDD & Modular Framework - Master Setup Template](./init_ia_enviroment.md)

### ¿Cómo usar esto en tus nuevos proyectos?

1. Guardas este contenido en un archivo llamado `copilot-sdd-setup.md` en tu repositorio de plantillas.
2. Al abrir un nuevo proyecto con Copilot Chat, le adjuntas el archivo y le pides que ejecute el **Protocolo de Inicialización** para que despliegue automáticamente las carpetas `.github/instructions/` y `.github/prompts/` adaptadas a la estructura de ese repositorio.