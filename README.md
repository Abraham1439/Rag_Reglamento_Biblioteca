# Asistente RAG Biblioteca Duoc UC

Proyecto de la Evaluación Parcial N°1 (ISY0101 - Ingeniería de Soluciones con IA): un agente conversacional que responde consultas de estudiantes sobre el **Reglamento de Bibliotecas Duoc UC** (sede Plaza Norte), usando una arquitectura RAG (Retrieval-Augmented Generation).

Fuente oficial del reglamento: https://bibliotecas.duoc.cl/reglamento

## Alcance

El asistente permite consultar normas sobre préstamos, renovaciones, atrasos, sanciones, salas de estudio y uso de lentes de realidad virtual. Recupera fragmentos de los documentos locales y los utiliza como contexto para generar una respuesta en español con sus fuentes.

La interacción se realiza desde un notebook. El proyecto no consulta cuentas de estudiantes ni realiza préstamos, reservas o pagos.

## Estructura del proyecto

```
Rag_Reglamento_Biblioteca/
├── data/                       # Reglamento dividido en documentos temáticos
│   ├── reglamento_general.txt
│   ├── prestamos_renovaciones.txt
│   ├── morosos_sanciones.txt
│   ├── normas_disciplinarias.txt
│   ├── salas_estudio.txt
│   └── uso_lentes_vr.txt
├── notebook/                   # Notebook ejecutable en Google Colab / Jupyter
│   └── asistente_biblioteca_duoc.ipynb
├── .env.example
├── .gitignore
├── README.md
└── requirements.txt
```

El notebook lee los seis documentos existentes en `data/`. Detecta esa carpeta al ejecutarse desde la raíz del proyecto o desde `notebook/`; no genera otra carpeta ni sobrescribe los TXT.

## Cómo funciona el pipeline RAG (en el Notebook)

Todo el pipeline RAG se encuentra implementado de forma modular y autocontenida dentro del archivo `notebook/asistente_biblioteca_duoc.ipynb`:

1. **Instalación y Configuración**: Carga de dependencias y configuración segura de llaves API (Groq para el LLM y Mistral para los Embeddings).
2. **Carga de documentos y fragmentación (chunking)**: Lectura de los 6 documentos temáticos y división en fragmentos, priorizando la separación por artículos. Los textos extensos pueden dividirse en fragmentos menores.
3. **Construcción del Índice Vectorial (FAISS)**: Vectorización con el modelo de embeddings `mistral-embed` y almacenamiento en el índice vectorial FAISS.
4. **Formulación de Prompts (Prompt Engineering)**: Implementación de instrucciones del sistema, mensajes de derivación y técnicas Zero-Shot, Few-Shot y Chain-of-Thought.
5. **Agente RAG y filtro de recuperación**: El agente calcula una puntuación a partir de la distancia del fragmento más cercano mediante `1 / (1 + distancia)`. Si esa puntuación es inferior a `0.45`, devuelve un mensaje que orienta al usuario a contactar a un encargado de biblioteca. Si alcanza o supera el umbral, envía el contexto recuperado al modelo. Las instrucciones del prompt le exigen reconocer cuándo ese contexto no contiene información suficiente para responder. El score no representa un porcentaje de confianza ni garantiza que la respuesta sea correcta.
6. **Evaluación End-to-End**: Ejecución de consultas dentro y fuera de alcance, mostrando respuesta, fuentes y criterio esperado para revisión manual. No se asigna aprobación automática: se debe comprobar el contenido y las citas.

El agente devuelve `respuesta`, `score` y `fuentes`. Cuando no hay información suficiente, orienta al usuario para contactar a un encargado; no realiza una transferencia automática ni devuelve una bandera de derivación.

## Requisitos previos

- Para ejecución local: Python con `pip` y el módulo `venv` disponibles.
- Git, si vas a clonar el repositorio. También puedes descargarlo y descomprimirlo como ZIP.
- Conexión a Internet para instalar dependencias y consultar las API.
- Una clave API de Groq para generar respuestas y otra de Mistral para generar embeddings (representaciones numéricas del texto).

## Ejecución local: Jupyter o VS Code

### 1. Obtener el proyecto

```bash
git clone https://github.com/Abraham1439/Rag_Reglamento_Biblioteca.git
cd Rag_Reglamento_Biblioteca
```

Si ya tienes el proyecto, abre una terminal en su carpeta raíz, donde se encuentra `requirements.txt`.

### 2. Crear y activar un entorno virtual

El entorno virtual mantiene las dependencias del proyecto separadas de las de otros proyectos. Créalo una sola vez y actívalo cada vez que abras una nueva terminal para trabajar.

**Windows (PowerShell):**

```powershell
py -m venv .venv
.venv\Scripts\activate
```

Si `py` no está disponible, utiliza `python -m venv .venv`.

**Windows (CMD):**

```bat
py -m venv .venv
.venv\Scripts\activate.bat
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

La ruta debe apuntar a la carpeta `.venv` del proyecto.

### 3. Instalar las dependencias

Con el entorno activado, ejecuta:

```bash
pip install -r requirements.txt
```

### 4. Configurar las claves API

Copia el archivo de ejemplo en la raíz del proyecto:

**Windows:**

```bash
cp .env.example .env
```

Si ya tienes un archivo `.env` configurado, edítalo sin volver a copiar el ejemplo. Reemplaza estos valores por tus claves reales:

```dotenv
LLM_API_KEY="tu_clave_groq"
EMBEDDING_API_KEY="tu_clave_mistral"
```

El notebook carga `.env` durante la configuración local. Si las claves no están definidas, las solicita mediante `getpass`, que oculta la entrada. Los valores de ejemplo deben reemplazarse: el notebook los interpreta como claves si los dejas escritos.

`.env` y `.venv/` están excluidos en `.gitignore`. Mantén las claves fuera del código y de las salidas guardadas del notebook.

### 5. Abrir y ejecutar el notebook
**Con VS Code**, abre la carpeta del proyecto, instala las extensiones Python y Jupyter si aún no las tienes y abre el notebook. En el selector de kernel, elige el intérprete de `.venv` (el kernel es el proceso de Python que ejecuta las celdas).

En ambos casos:

1. **Omite la primera celda**, que contiene `%cd /content` y `git clone`: está destinada únicamente a Google Colab.
2. Puedes omitir también la celda `!pip install ...`, ya que instalaste las dependencias en el paso 3.
3. Ejecuta las demás celdas en orden, comenzando por la configuración de credenciales y siguiendo con la carga de documentos, el índice FAISS, los prompts y el agente.
4. Ejecuta las consultas de demostración y los casos de evaluación. Verás las respuestas, las fuentes recuperadas y el score.


## Ejecución en Google Colab

En Colab no necesitas crear un entorno virtual local.

1. Abre Google Colab e importa `notebook/asistente_biblioteca_duoc.ipynb` desde GitHub o sube el archivo.
2. Ejecuta la primera celda: clona el repositorio en `/content/Rag_Reglamento_Biblioteca` y establece esa carpeta como directorio de trabajo. Así quedan disponibles los documentos de `data/`.
3. Ejecuta la celda de instalación de dependencias.
4. En la sección **Secretos** de Colab, agrega `LLM_API_KEY` y `EMBEDDING_API_KEY` y habilita su acceso para el notebook. Si no los configuras, la celda de credenciales solicitará las claves mediante `getpass`.
5. Ejecuta las celdas restantes en orden.

Si el repositorio ya está clonado en esa sesión, omite el comando `git clone` y asegúrate de trabajar en `/content/Rag_Reglamento_Biblioteca`. Al reiniciar el entorno de Colab, puede ser necesario repetir la preparación.

## Configuración

### Proveedores y modelos

Estas variables se leen desde el entorno, desde `.env` en local o desde los secretos de Colab:

| Variable | Función | Valor del ejemplo |
| --- | --- | --- |
| `LLM_API_KEY` | Clave de Groq | Debes reemplazarla |
| `LLM_BASE_URL` | Endpoint del modelo de lenguaje | `https://api.groq.com/openai/v1` |
| `LLM_MODEL` | Modelo principal para respuestas | `groq/compound-mini` |
| `EMBEDDING_API_KEY` | Clave de Mistral | Debes reemplazarla |
| `EMBEDDING_BASE_URL` | Endpoint de embeddings | `https://api.mistral.ai/v1` |
| `EMBEDDING_MODEL` | Modelo de embeddings | `mistral-embed` |

### Parámetros RAG

**Actualmente estos parámetros se definen directamente en la celda de configuración del notebook.** Aunque aparecen en `.env.example`, modificarlos allí no cambia el comportamiento del cuaderno.

| Parámetro | Valor | Función |
| --- | --- | --- |
| `DATA_DIR` | Detección automática de `data/` | Carpeta con los documentos |
| `CHUNK_SIZE` | `500` | Tamaño objetivo de los fragmentos, en caracteres |
| `CHUNK_OVERLAP` | `80` | Solapamiento entre fragmentos, en caracteres |
| `TOP_K` | `3` | Cantidad de fragmentos recuperados por consulta |
| `CONFIDENCE_THRESHOLD` | `0.45` | Umbral mínimo del score para consultar al LLM |

Si modificas los documentos o la fragmentación, vuelve a ejecutar la carga, la construcción del índice y la creación del agente. El índice FAISS se mantiene en memoria y se reconstruye al ejecutar la ingesta.

## Ejemplo de uso

Después de ejecutar la celda que crea `agent`, agrega una celda con:

```python
resultado = agent.answer(
    "¿Cuántas horas al día puedo reservar una sala de estudio?",
    technique="zero-shot",
)

print(resultado["respuesta"])
print("Score:", resultado["score"])
print("Fuentes:", resultado["fuentes"])
```

Las técnicas disponibles son `zero-shot`, `few-shot` y `chain-of-thought`.
La técnica seleccionada para el funcionamiento habitual es `zero-shot`, que también es el valor por defecto de `agent.answer()`. Las otras variantes se conservan como alternativas utilizadas en pruebas exploratorias. El notebook entregado no contiene resultados comparativos registrados.

El `score` es una medida de similitud calculada a partir de la distancia del resultado recuperado; **no representa una probabilidad de que la respuesta sea correcta**. Las fuentes corresponden a los archivos recuperados y deben contrastarse con la respuesta.

## Evaluación y limitaciones

- El notebook incluye 8 consultas de evaluación manual dentro y fuera de alcance. Revisa que las respuestas estén respaldadas por los documentos y que las citas correspondan.
- Cuando falta información, el asistente debe indicar que se consulte a un encargado. Esto es una orientación textual, no una transferencia a una persona.
- El contenido consultado es el de los archivos locales: no se sincroniza automáticamente con la página oficial del reglamento.
- La ejecución realiza llamadas a Groq y Mistral y depende de las credenciales, los modelos disponibles y los límites de uso de cada cuenta.

## Uso de Inteligencia Artificial

Se utilizó apoyo de Inteligencia Artificial como (chatgpt astra y Claude 3.5 Sonnet) para estructurar el código del pipeline RAG, redactar la documentación técnica y formatear el cuaderno ejecutable, siguiendo los lineamientos y ejemplos del material del ramo (RA1: conexión a LLMs, prompt engineering, RAG y evaluación). Las decisiones de diseño (elección del caso, división del reglamento en documentos temáticos, definición del filtro de confianza) fueron discutidas y validadas por el equipo. Las conclusiones y reflexiones individuales del informe fueron redactadas por cada integrante sin apoyo de IA, conforme a como se pedia en la pauta.
