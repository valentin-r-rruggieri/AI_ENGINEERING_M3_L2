# M3L2 - Marcos de Orquestacion: LangChain

<<<<<<< HEAD
## La idea central de M3L2

En M3L1 construimos todo desde cero:

- el loop Thought -> Action -> Observation
- las tools como funciones Python comunes
- el trace con prints manuales
- el max_steps con un for manual

En M3L2 vemos que LangChain ya tiene todo eso construido como componentes reutilizables.

El cambio no es de concepto. Es de nivel de abstraccion.

```
M3L1: entender como funciona un agente por dentro
M3L2: usar un framework que hace el trabajo repetitivo por nosotros
```

---
=======
## Para quien es esta carpeta

Esta carpeta contiene ejercicios extra para practicar LangChain de forma progresiva.

El recorrido empieza desde cero con seis notebooks nuevos (`E00` a `E05`) y despues continua con ejercicios de profundizacion (`E06` a `E15`).

Cada ejercicio tiene dos versiones:

- `Starter`: notebook para completar.
- `Resolution`: notebook completo para comparar la solucion.
>>>>>>> 8f97693 (mas y mas repaso..)

## Requisitos

- Python 3.10+
- OpenAI API key

Instalacion recomendada:

```bash
pip install langchain langchain-openai langchain-community langchain-core requests chromadb faiss-cpu
```

Cada notebook pide la API key con `getpass` y la guarda en la variable de entorno de esa sesion:

```python
import os
import getpass

if not os.getenv("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass("Ingresa tu OpenAI API key: ")
```

Esto esta pensado para Google Colab: no hace falta subir `.env`, `conexion.py` ni archivos de configuracion compartidos.

## Orden recomendado

```text
E00 -> E01 -> E02 -> E03 -> E04 -> E05
 |      |      |      |      |      |
 chat   LCEL   memoria tools  RAG    RAG + memoria

E06 -> E07 -> E08 -> E09 -> E10 -> E11 -> E12 -> E13 -> E14 -> E15
profundizacion y practica adicional
```

## Ejercicios nuevos de inicio

| Ejercicio | Carpeta | Tema | Que practicas |
|---|---|---|---|
| E00 | `E00_chat_basico_prompt_template/` | Chat basico + PromptTemplate | Primer invoke, template con variables, respuesta del modelo |
| E01 | `E01_lcel_cadenas_langchain/` | LCEL y chains | Composicion `prompt | llm | parser` |
| E02 | `E02_chat_con_memoria_langchain/` | Chat con memoria | Historial conversacional por `session_id` |
| E03 | `E03_tools_langchain_api_dolar/` | Tools + API del dolar | `@tool`, `bind_tools`, `tool_calls`, DolarAPI |
| E04 | `E04_rag_basico_langchain/` | RAG basico | Loader, splitter, embeddings, Chroma, retriever, prompt RAG |
| E05 | `E05_rag_chat_con_memoria/` | RAG chat con memoria | Retriever + historial conversacional |

### E00 - Chat basico y PromptTemplate

Empieza con la llamada mas simple al modelo y muestra por que un prompt como objeto es mas mantenible que un string suelto.

Archivos:

- `E00_chat_basico_prompt_template/M3L2_E00_Starter.ipynb`
- `E00_chat_basico_prompt_template/M3L2_E00_Resolution.ipynb`

### E01 - LCEL y cadenas

Muestra como conectar componentes con el operador `|`.

La idea central:

```python
cadena = prompt | llm | parser
```

Archivos:

- `E01_lcel_cadenas_langchain/M3L2_E01_Starter.ipynb`
- `E01_lcel_cadenas_langchain/M3L2_E01_Resolution.ipynb`

### E02 - Chat con memoria

Introduce el problema de que un LLM no recuerda por si solo. El notebook agrega historial conversacional con `RunnableWithMessageHistory`.

Archivos:

- `E02_chat_con_memoria_langchain/M3L2_E02_Starter.ipynb`
- `E02_chat_con_memoria_langchain/M3L2_E02_Resolution.ipynb`

### E03 - Tools con API del dolar

Explica que una tool es una funcion con contrato claro. El alumno ve `tool_calls` y consulta DolarAPI desde una herramienta.

Archivos:

- `E03_tools_langchain_api_dolar/M3L2_E03_Starter.ipynb`
- `E03_tools_langchain_api_dolar/M3L2_E03_Resolution.ipynb`

### E04 - RAG basico

Construye un flujo RAG completo con un documento creado dentro del notebook:

```text
Documento -> Splitter -> Chunks -> Embeddings -> Chroma -> Retriever -> Prompt -> Modelo
```

Archivos:

- `E04_rag_basico_langchain/M3L2_E04_Starter.ipynb`
- `E04_rag_basico_langchain/M3L2_E04_Resolution.ipynb`

### E05 - RAG chat con memoria

Combina retrieval documental con historial conversacional para responder preguntas de seguimiento.

Archivos:

- `E05_rag_chat_con_memoria/M3L2_E05_Starter.ipynb`
- `E05_rag_chat_con_memoria/M3L2_E05_Resolution.ipynb`

## Ejercicios de profundizacion

| Ejercicio | Carpeta | Tema |
|---|---|---|
| E06 | `E06_manual_to_langchain/` | Del agente manual a LangChain |
| E07 | `E07_prompt_template/` | PromptTemplate con mas foco en estructura |
| E08 | `E08_lcel_chain/` | LCEL aplicado a un flujo guiado |
| E09 | `E09_rag_mini/` | RAG mini como pipeline completo |
| E10 | `E10_llm_wrapper/` | ChatOpenAI como wrapper del modelo |
| E11 | `E11_output_parser/` | StrOutputParser y salida como string |
| E12 | `E12_embeddings/` | Embeddings y similitud |
| E13 | `E13_faiss/` | FAISS como indice vectorial |
| E14 | `E14_retriever/` | Retriever como interfaz estandar |
| E15 | `E15_refactor_chaos/` | Refactorizar un script caotico |

## Como trabajar

1. Abrir el `Starter` del ejercicio.
2. Leer las celdas teoricas antes de tocar codigo.
3. Completar los TODOs en orden.
4. Ejecutar los checks del final.
5. Comparar con `Resolution`.

## Notas

- Los notebooks `E00` a `E05` son el inicio recomendado.
- Los ejercicios `E06` a `E15` quedan como practica adicional y profundizacion.
- No hay notebooks `01_...` en la raiz: todo el material esta organizado por carpeta `E00`, `E01`, etc.
- No se usa `conexion.py`: cada notebook es autonomo para poder ejecutarse en Colab.
