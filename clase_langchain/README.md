# Introducción a LangChain con datos académicos

Construiremos un asistente que consulta las facultades y los departamentos de DjangoPascual. Seguiremos este recorrido: **datos → modelo → herramientas → agente → memoria**.

El notebook explica cada paso y contiene el código completo. Utiliza una copia local de los datos de ejemplo; puedes ejecutar esta carpeta por separado, sin iniciar Django ni crear una base de datos. La aplicación original conserva su propio entorno y su documentación en `README_Django.md`, en la raíz del repositorio.

Para presentar los conceptos antes del notebook, abre [agentes_ia.html](agentes_ia.html) en tu navegador. Incluye cuatro secciones gráficas: anatomía del agente, ciclo de ejecución, memoria y conexión con LangChain. Pulsa los nodos para explorar cada diagrama y avanza por la ejecución para ver los mensajes acumulados. Funciona sin conexión ni instalación; usa las flechas del teclado para cambiar de sección.

## 1. Qué necesitas

- Visual Studio Code con las extensiones **Python** y **Jupyter**, ambas de Microsoft.
- **uv**, que crea el entorno virtual e instala Python y las dependencias.
- Una clave de la API de **OpenAI** o de **OpenRouter**, según la opción que elijas.
- Conexión a Internet para instalar paquetes y consultar el modelo.
- Conocimientos básicos de funciones, listas y diccionarios de Python.

Puedes elegir OpenAI o los modelos gratuitos de OpenRouter. OpenAI utiliza la cuota de tu cuenta y puede tener costo. La opción gratuita de OpenRouter requiere una cuenta y una clave propias; está sujeta a disponibilidad y límites de uso, según su [documentación](https://openrouter.ai/docs/guides/routing/routers/free-router). La primera parte, que consulta los datos con Python, funciona sin clave.

## 2. Instalar uv

Si ya lo tienes instalado, compruébalo con `uv --version` y continúa al siguiente paso. Los comandos siguientes corresponden a la [guía oficial de instalación](https://docs.astral.sh/uv/getting-started/installation/).

En macOS o Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

En Windows, desde PowerShell:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Cierra y abre la terminal después de instalarlo. Verifica:

```bash
uv --version
```

## 3. Abrir la carpeta y crear el entorno

En VS Code, selecciona **Archivo → Abrir carpeta** y abre **`clase_langchain`**, dentro de `DjangoPascual`. Después abre **Terminal → Nueva terminal**. Debes estar en la carpeta que contiene este README y `pyproject.toml`.

Si tu terminal está en la raíz de DjangoPascual, entra primero con:

```bash
cd clase_langchain
```

Crea el entorno e instala las dependencias:

```bash
uv sync --locked
```

El comando crea `.venv` dentro de `clase_langchain` e instala las versiones de `uv.lock`. El proyecto selecciona Python 3.13 mediante `.python-version`; uv puede descargarlo si hace falta. No necesitas activar el entorno manualmente para usar `uv run` ni para seleccionar su kernel en VS Code.

Comprueba que estás usando ese Python:

```bash
uv run python -c "import sys; print(sys.executable)"
```

La ruta debe terminar en `clase_langchain/.venv/bin/python` en macOS/Linux o en `clase_langchain\.venv\Scripts\python.exe` en Windows.

## 4. Elegir el proveedor y configurar su clave

Copia `.env.example` a un archivo llamado **`.env`** dentro de `clase_langchain`. Puedes hacerlo desde el explorador de VS Code o desde la terminal.

En macOS/Linux:

```bash
cp .env.example .env
```

En PowerShell:

```powershell
Copy-Item .env.example .env
```

Abre `.env` y elige **una** de las siguientes configuraciones. Solo necesitas la clave del proveedor seleccionado.

### Opción A: OpenAI

Configura estos valores en `.env`:

```dotenv
MODEL_PROVIDER=openai
OPENAI_API_KEY=tu_clave_personal
OPENAI_MODEL=gpt-5-nano
```

Usa una clave de tu cuenta de OpenAI con acceso al modelo y cuota disponible. Puedes cambiar `OPENAI_MODEL` por otro modelo de tu cuenta que admita llamadas a herramientas. La conexión utiliza la [API de Chat Completions de OpenAI](https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create).

### Opción B: OpenRouter gratuito

Crea una cuenta en [OpenRouter](https://openrouter.ai) y genera una clave desde su panel de API keys. Configura estos valores en `.env`:

```dotenv
MODEL_PROVIDER=openrouter
OPENROUTER_API_KEY=tu_clave_personal_de_openrouter
OPENROUTER_MODEL=openrouter/free
```

`openrouter/free` selecciona entre modelos gratuitos y filtra los que admiten las herramientas enviadas por el agente. El modelo elegido puede cambiar entre solicitudes. Puedes consultar cómo funciona en la [guía del router gratuito](https://openrouter.ai/docs/guides/routing/routers/free-router).

También puedes indicar un modelo concreto cuyo identificador termine en `:free` y que admita herramientas. Para esta práctica, la configuración de OpenRouter acepta únicamente `openrouter/free` o identificadores terminados en `:free`. Los modelos gratuitos tienen [límites de solicitudes y disponibilidad](https://openrouter.ai/docs/api/reference/limits); no equivalen a acceso ilimitado.

### Cambiar de proveedor

Cambia `MODEL_PROVIDER` a `openai` u `openrouter`, configura la clave correspondiente y **reinicia el kernel**. Después ejecuta las celdas desde el principio. La celda de configuración muestra el proveedor y el modelo seleccionados. Si conservas ambas claves en `.env`, solo utiliza la del proveedor elegido.

El notebook carga el `.env` de esta carpeta. Las dependencias ya incluidas funcionan con ambas opciones mediante sus APIs compatibles. No escribas las claves en las celdas ni compartas `.env`; está excluido mediante `.gitignore`.

## 5. Ejecutar el notebook

1. Abre **`intro_langchain_academica.ipynb`** en VS Code.
2. Pulsa **Seleccionar kernel**, arriba a la derecha.
3. Si hace falta, elige **Seleccionar otro kernel → Entornos de Python**.
4. Selecciona el Python de **`.venv` dentro de `clase_langchain`**.
5. Ejecuta las celdas en orden con **Shift + Enter**. Lee la explicación de cada bloque y revisa su resultado antes de continuar.

Las dependencias incluyen `ipykernel`, que permite ejecutar las celdas. Estas instrucciones siguen la [integración de uv con Jupyter en VS Code](https://docs.astral.sh/uv/guides/integration/jupyter/#using-jupyter-from-vs-code).

La celda de configuración avisa si falta la clave. La parte anterior a esa celda funciona sin llamadas al modelo. Para repetir toda la demostración desde cero, usa **Reiniciar kernel** y vuelve a ejecutar las celdas en orden.

### Alternativa: JupyterLab

Desde la misma carpeta ejecuta:

```bash
uv run jupyter lab
```

Abre la dirección local que indique la terminal y selecciona el notebook. Como JupyterLab se inicia con `uv run`, utiliza el entorno del proyecto.

## 6. Qué resultados comprobar

Los datos proceden del fixture académico del repositorio, convertidos a UTF-8 y reducidos a los campos que utiliza la práctica.

| Facultad | Departamentos | Total |
| --- | --- | --- |
| `10`: Facultad de Ingenieria | `111`: Departamento de Sistemas Digitales; `112`: Departamento de Eléctrica. | 2 |
| `20`: Facultad de Diseño | `211`: Departamento de Diseño gráfico. | 1 |

El modelo interpreta la pregunta y redacta la respuesta; **las herramientas Python consultan los registros y calculan los totales**. El notebook permite ver la herramienta elegida, sus argumentos y el resultado devuelto.

La memoria conserva el historial de una conversación identificada por `thread_id`. Cambiar ese identificador separa la conversación. Reiniciar el kernel o crear otro `InMemorySaver` elimina el historial de esta demostración.

## Problemas frecuentes

| Problema | Qué hacer |
| --- | --- |
| La terminal no reconoce `uv`. | Cierra y vuelve a abrir la terminal o consulta la guía de instalación. |
| `ModuleNotFoundError` o no aparece el kernel. | Ejecuta `uv sync --locked` dentro de `clase_langchain`, comprueba las extensiones Python y Jupyter y selecciona su `.venv` como kernel. |
| No se encuentra `datos_academicos.json`. | Conserva `recursos/` junto al notebook. Abre `clase_langchain` en VS Code y reinicia el kernel. |
| Falta `OPENAI_API_KEY` u `OPENROUTER_API_KEY`. | Revisa `MODEL_PROVIDER` y configura la clave de ese proveedor en `.env`. No necesitas las dos claves. |
| `MODEL_PROVIDER` no es válido. | Usa exactamente `openai` u `openrouter`. Reinicia el kernel y ejecuta las celdas desde el principio. |
| El modelo de OpenRouter no es gratuito o no admite herramientas. | Usa `OPENROUTER_MODEL=openrouter/free` o elige un identificador terminado en `:free` que admita herramientas. |
| OpenRouter responde con error `429` o sin un proveedor disponible. | Revisa los límites de tu cuenta y la disponibilidad del servicio. Evita volver a ejecutar todas las celdas repetidamente; cada llamada del agente también puede generar varias solicitudes. |
| Error de autenticación, cuota o acceso al modelo. | Comprueba la clave, el acceso y la cuota de tu cuenta. Si cambias `.env`, reinicia el kernel y ejecuta las celdas en orden. |
| El asistente no recuerda una consulta. | Comprueba que usas `agente_con_memoria` y el mismo `config` entre turnos. Recrear el agente o su checkpointer comienza otro historial. |
| `uv sync --locked` indica que el bloqueo no coincide. | Conserva `pyproject.toml` y `uv.lock` de esta entrega. Si modificaste las dependencias, ejecuta `uv lock` y después `uv sync --locked`. |

## Alternativa de instalación con pip

Si prefieres pip, instala primero Python 3.12 o 3.13. Desde `clase_langchain`, crea y activa el entorno:

En macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

En Windows PowerShell:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Con el entorno activo:

```bash
python -m pip install -r requirements.txt
```

Después configura `.env` y selecciona ese `.venv` como kernel siguiendo los pasos anteriores. `requirements.txt` fija las dependencias directas; `uv.lock` incluye también las dependencias indirectas del recorrido con uv.

## Archivos y origen

- `intro_langchain_academica.ipynb`: explicación y demostración completa.
- `recursos/datos_academicos.json`: datos locales de la práctica.
- `guia_docente.md`: secuencia de explicación, preguntas y resultados esperados para el profesor.
- `pyproject.toml`, `uv.lock` y `requirements.txt`: configuración del entorno.
- `.env.example`: plantilla de configuración sin credenciales.
- `LICENSE`: aviso del material original adaptado.

Adaptación del módulo 1 de LangChain Academy y del material `intro_langchain_1` del docente. Los datos corresponden a `Academica/fixtures/datos_iniciales.json` de DjangoPascual. Se conserva el aviso MIT del material original.
