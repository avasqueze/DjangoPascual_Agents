# DjangoPascual: introducción a LangChain

El material de clase construye un asistente que consulta las facultades y los departamentos de este proyecto. Explica modelos, herramientas, agentes y memoria mediante un notebook en español.

Como introducción, abre la [presentación interactiva sobre agentes de IA](clase_langchain/agentes_ia.html) en tu navegador. Es un HTML independiente que funciona sin conexión.

**Comienza en [clase_langchain/README.md](clase_langchain/README.md)**. Allí están las instrucciones para instalar uv, crear el entorno, configurar la clave del modelo y seleccionar el kernel del notebook.

Desde la raíz del repositorio, entra en la carpeta de clase antes de preparar el entorno:

```bash
cd clase_langchain
uv sync --locked
```

Después configura `.env` y abre `intro_langchain_academica.ipynb` siguiendo el README de esa carpeta. Puedes elegir **OpenAI** u **OpenRouter con modelos gratuitos** mediante `MODEL_PROVIDER`. La práctica usa datos locales y se ejecuta sin iniciar el servidor Django.

La [guía docente](clase_langchain/guia_docente.md) contiene la secuencia de explicación y los resultados de referencia. La documentación de la aplicación web está en [README_Django.md](README_Django.md).
