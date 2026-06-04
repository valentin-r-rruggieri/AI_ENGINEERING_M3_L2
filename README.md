# M3L2 - Marcos de Orquestacion: LangChain

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

## Requisitos

- Python 3.10+
- OpenAI API key (para E00 en adelante, excepto E01 que no necesita)
- Packages: `langchain`, `langchain-openai`, `langchain-community`, `langchain-core`
- Para E07 y E08: `faiss-cpu` (`pip install faiss-cpu`)

```bash
pip install langchain langchain-openai langchain-community langchain-core faiss-cpu
```

---

## Mapa de ejercicios

```
E00  ->  E01  ->  E04  ->  E02  ->  E06  ->  E07  ->  E08  ->  E03
bridge  prompt  llm_wrap  lcel    embed   faiss  retriev  RAG
                + E05                                         completo

E10: homework - refactorizacion libre
```

---

## Descripcion detallada de cada ejercicio

---

### E00 - Del agente manual (M3L1) a LangChain

**Archivo**: `E00_manual_to_langchain/`

**Tiempo estimado**: 15-20 minutos

**Necesita API key**: si

**Que practica**:

Toma el agente clima+calculo de M3L1 E03 y lo implementa dos veces:
una vez como lo haciamos en M3L1 (manual), y otra con LangChain.

El alumno ve que los conceptos son los mismos, solo cambia quien los implementa.

**Relacion con la lecture**:

- Lecture M3L1 Seccion 10-12: patron ReAct (Thought/Action/Observation)
- Lecture M3L2 Seccion 5: que permite hacer LangChain
- Lecture M3L2 Seccion 14: agents y tools

**Tabla de mapeo M3L1 -> LangChain**:

| M3L1 manual | LangChain |
|---|---|
| `def weather_tool(city)` | `@tool def weather_tool(city: str) -> str` |
| `choose_action(state)` hardcodeado | LLM decide con tool binding |
| `for step in range(max_steps)` | `max_iterations=5` en AgentExecutor |
| `trace = []` + prints | `verbose=True` |
| `[Thought]/[Action]/[Obs]` a mano | Mensajes internos del AgentExecutor |
| `state = {}` manual | Historial del AgentExecutor |

**Conocimiento que se aprende**:

- El decorator `@tool` lee el docstring y genera el schema automaticamente
- `create_tool_calling_agent` es el LLM que decide que tool usar
- `AgentExecutor` es el loop ReAct implementado por el framework
- `verbose=True` reemplaza todos los prints del trace manual
- La diferencia entre un pipeline hardcodeado y un agente que decide dinamicamente

**Por que empezar aqui**: sin este ejercicio, los demas parecen componentes sueltos.
Este es el contexto que conecta todo.

---

### E01 - PromptTemplate: del string al componente

**Archivo**: `E01_prompt_template/`

**Tiempo estimado**: 8 minutos

**Necesita API key**: NO (solo construye y formatea el template, no llama al modelo)

**Que practica**:

Reemplazar la concatenacion manual de strings por `ChatPromptTemplate`.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 3.4: sintomas de deuda tecnica (prompts duplicados)
- Lecture M3L2 Seccion 10: PromptTemplates, ventajas y formato recomendado
- Lecture M3L2 Seccion 10.4: prompt RAG recomendado

**Conexion con M3L1**:

En M3L1 construiamos el prompt del agente asi:

```python
prompt = f"Thought: {pensamiento}\nAction: {herramienta}({args})"
```

Ese string estaba mezclado con la logica del agente y era dificil de cambiar.
`ChatPromptTemplate` convierte el prompt en un objeto separado, testeable y reutilizable.

**Conocimiento que se aprende**:

- `ChatPromptTemplate.from_messages([("system", "..."), ("human", "...")])` 
- Variables explicitas con `{variable}` en lugar de f-strings
- `.format_messages(var=valor)` para ver el prompt antes de enviarlo al modelo
- `.input_variables` para inspeccionar que espera el template
- Por que el prompt como objeto facilita el debugging

---

### E02 - LCEL Chain: componer con `|`

**Archivo**: `E02_lcel_chain/`

**Tiempo estimado**: 10 minutos

**Necesita API key**: si

**Que practica**:

Conectar `ChatPromptTemplate`, `ChatOpenAI` y `StrOutputParser` usando el operador `|` de LCEL.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 6: scripting tactico vs ingenieria estructurada
- Lecture M3L2 Seccion 12: LCEL, que es y por que importa
- Lecture M3L2 Seccion 12.4: beneficios de LCEL

**Lo que LCEL reemplaza**:

```python
# Sin LCEL: 4 pasos imperatives
messages = prompt.format_messages(question=q)
ai_msg   = llm.invoke(messages)
text     = ai_msg.content
result   = parser.invoke(text)

# Con LCEL: 1 linea declarativa
chain = prompt | llm | parser
result = chain.invoke({"question": q})
```

**Conocimiento que se aprende**:

- El operador `|` conecta componentes: el output de uno es el input del siguiente
- `chain.invoke(dict)` ejecuta todos los pasos en secuencia
- Cambiar el modelo = cambiar el objeto `llm`, el resto no cambia
- LCEL hace visible el flujo de datos en una sola linea
- Diferencia entre llamada directa `openai` y el wrapper `ChatOpenAI`

---

### E03 - RAG mini: el pipeline completo

**Archivo**: `E03_rag_mini/`

**Tiempo estimado**: 12-15 minutos

**Necesita API key**: si (+ faiss-cpu)

**Que practica**:

Construir el pipeline RAG completo: ingestion (FAISS) + consulta (LCEL chain).
Compara el script legacy que manda todo el contexto vs el pipeline que solo manda docs relevantes.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 15: de prototipo RAG a pipeline production-grade
- Lecture M3L2 Seccion 16: refactor paso a paso (los 6 pasos del refactor)
- Lecture M3L2 Seccion 17: arquitectura recomendada (flujo ingestion + consulta)
- Lecture M3L2 Seccion 18: debugging y trazabilidad

**Las dos fases del pipeline RAG**:

```
INGESTION (una sola vez):
  Textos -> OpenAIEmbeddings -> FAISS -> Retriever

CONSULTA (por cada pregunta):
  Pregunta -> Retriever -> Docs -> PromptTemplate -> LLM -> Respuesta
```

**Conocimiento que se aprende**:

- `FAISS.from_texts(textos, embeddings)`: ingestion en una linea
- `vectorstore.as_retriever(search_kwargs={"k": 2})`: interfaz estandar
- `RunnablePassthrough()`: pasar el input original sin modificarlo
- La RAG chain con LCEL: `{"context": retriever | format_docs, "question": RunnablePassthrough()} | prompt | llm | parser`
- Debugging modular: inspeccionar cada componente por separado
- Por que separar ingestion de consulta (Lecture Seccion 19.1)

---

### E04 - ChatOpenAI: el modelo como objeto

**Archivo**: `E04_llm_wrapper/`

**Tiempo estimado**: 6 minutos

**Necesita API key**: si

**Que practica**:

Entender que `ChatOpenAI` no es solo "la forma LangChain de llamar a OpenAI".
Es un objeto configurable que encapsula el modelo y expone la interfaz estandar `.invoke()`.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 9: LLMs y ChatModels
- Lecture M3L2 Seccion 9.3: buenas practicas del wrapper

**Conexion con M3L1**:

En M3L1, el modelo estaba hardcodeado dentro del agente:
```python
response = openai.chat.completions.create(model="gpt-4o-mini", ...)
```

Con `ChatOpenAI`, el modelo es un objeto que se configura una sola vez
y se pasa como parametro a chains, agentes, etc.

**Conocimiento que se aprende**:

- `ChatOpenAI(model="gpt-4o-mini", temperature=0)` como objeto configurable
- `llm.invoke("texto")` devuelve un `AIMessage`, no un string
- `.content` para extraer el texto del `AIMessage`
- Por que el wrapper permite cambiar de modelo sin tocar el pipeline
- La diferencia entre `temperature=0` (determinista) y `temperature=1` (creativo)

---

### E05 - StrOutputParser: extraer el texto automaticamente

**Archivo**: `E05_output_parser/`

**Tiempo estimado**: 5 minutos

**Necesita API key**: si

**Que practica**:

Entender por que `StrOutputParser` existe: convierte el `AIMessage` en string automaticamente,
evitando el `.content` manual en todos los lugares.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 8: componentes principales (OutputParsers en la lista)
- Lecture M3L2 Seccion 12.3: ejemplo con parser

**Conexion con M3L1**:

En M3L1 la respuesta final era un string que construiamos a mano.
Con LangChain, el parser es el ultimo componente de la chain y garantiza
que el resultado siempre sea del tipo esperado.

**Conocimiento que se aprende**:

- `StrOutputParser()` como componente independiente
- `llm | parser`: composicion minima para obtener string directamente
- Por que los parsers son componentes separados (testeables, intercambiables)
- Tabla de parsers disponibles: Str, Json, Pydantic, CommaSeparatedList

---

### E06 - OpenAIEmbeddings: convertir texto en vectores

**Archivo**: `E06_embeddings/`

**Tiempo estimado**: 7 minutos

**Necesita API key**: si (no necesita faiss-cpu)

**Que practica**:

Entender que es un embedding y por que la similitud vectorial es el fundamento del RAG.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 13.1: que es un Index
- Lecture M3L2 Seccion 13.3: por que aislar retrieval
- Fundamento matematico del retrieval por similitud

**Por que este ejercicio existe**:

Sin entender que es un embedding, el FAISS de E07 parece magia.
Este ejercicio muestra que es solo matematica: vectores y distancias.

**Conocimiento que se aprende**:

- `OpenAIEmbeddings().embed_query(texto)` devuelve una lista de ~1536 numeros
- Textos con significado similar tienen vectores similares (cercanos)
- Textos con significado diferente tienen vectores lejanos
- Similitud del coseno: mide que tan "cercanos" son dos vectores
- `embed_documents([t1, t2, ...])` para embeder multiples textos

---

### E07 - FAISS: guardar vectores y buscar por similitud

**Archivo**: `E07_faiss/`

**Tiempo estimado**: 6 minutos

**Necesita API key**: si + faiss-cpu

**Que practica**:

Usar FAISS para almacenar vectores y hacer busquedas por similitud eficientes.
Reemplaza la busqueda naive "manda todo el contexto" del script legacy.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 13.1: que es un Index
- Lecture M3L2 Seccion 13.4: ejemplo con FAISS
- Lecture M3L2 Seccion 19.5: manejar retrieval vacio

**Por que FAISS importa**:

```
Sin FAISS (script legacy):          Con FAISS:
context = join(TODOS_LOS_DOCS)      k_docs = faiss.search(query, k=2)
                                    context = format(k_docs)

Manda 500 docs aunque               Manda solo los 2 mas relevantes
solo 1 sea relevante
```

**Conocimiento que se aprende**:

- `FAISS.from_texts(textos, embeddings)`: crea el indice en memoria
- `vectorstore.similarity_search(query, k=N)`: devuelve los N docs mas similares
- `doc.page_content`: el texto del documento recuperado
- Por que k=2 es mejor que mandar todo el contexto
- FAISS es intercambiable con Chroma, Pinecone, etc.

---

### E08 - Retriever: la interfaz estandar de busqueda

**Archivo**: `E08_retriever/`

**Tiempo estimado**: 6 minutos

**Necesita API key**: si + faiss-cpu

**Que practica**:

Entender por que `as_retriever()` existe: encapsula el vector store detras
de una interfaz estandar que se conecta directamente con LCEL.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 13.2: que es un Retriever
- Lecture M3L2 Seccion 13.3: por que aislar retrieval
- Lecture M3L2 Seccion 19.4: testear retriever por separado

**Conexion con M3L1**:

En M3L1 una "tool de busqueda" era una funcion Python normal.
El retriever de LangChain es la misma idea, pero con una interfaz
que permite conectarlo directamente con `|` en una LCEL chain.

**La diferencia clave**:

```python
# similarity_search directo: especifico de FAISS
docs = vectorstore.similarity_search(query, k=2)

# as_retriever: interfaz estandar
retriever = vectorstore.as_retriever(search_kwargs={"k": 2})
docs = retriever.invoke(query)  # misma interfaz para FAISS, Chroma, Pinecone
```

**Conocimiento que se aprende**:

- `vectorstore.as_retriever(search_kwargs={"k": 2})` crea el retriever
- `retriever.invoke(query)` devuelve `List[Document]`
- Por que la interfaz estandar permite intercambiar el vector store sin tocar el pipeline
- El parametro `k`: tradeoff entre contexto y tokens
- Como el retriever se conecta en la RAG chain de E03

---

### E10 - Refactorizar el Caos (homework)

**Archivo**: `E10_refactor_chaos/`

**Tiempo estimado**: 20-30 minutos (para hacer en casa)

**Necesita API key**: si + faiss-cpu

**Que practica**:

Dado un script de chatbot de soporte con todos los problemas tipicos
(prompt manual, modelo hardcodeado, sin retrieval real, todo mezclado),
refactorizarlo usando LangChain.

No hay TODOs guiados. El alumno decide que componentes usar.

**Relacion con la lecture**:

- Lecture M3L2 Seccion 3: el problema de los scripts RAG cuando crecen
- Lecture M3L2 Seccion 6: scripting tactico vs ingenieria estructurada
- Lecture M3L2 Seccion 22: checklist de produccion completo

**Conexion con el PPTX de clase**:

Este ejercicio es la version notebook del taller grupal "Refactorizar el Caos"
del hands-on de la clase. Se puede hacer en clase (20 min en grupos) o como tarea.

**Conocimiento que se aprende**:

- Identificar los problemas de un script real
- Decidir que componentes LangChain usar para cada problema
- Separar ingestion de consulta en un caso real
- Aplicar el checklist de produccion de la lecture
- Entender la diferencia entre un chatbot que "funciona" y uno que es mantenible

---

## Resumen de cobertura de la lecture

| Seccion de Lecture M3L2 | Ejercicios que la cubren |
|---|---|
| Sec 3: El problema de los scripts RAG | E01, E03, E10 |
| Sec 5: Por que LangChain | E00 |
| Sec 6: Scripting tactico vs ingenieria estructurada | E02, E10 |
| Sec 9: LLMs y ChatModels | E04 |
| Sec 10: PromptTemplates | E01 |
| Sec 12: LCEL | E02, E03 |
| Sec 13: Indexes y Retrievers | E06, E07, E08 |
| Sec 14: Agents y Tools | E00 |
| Sec 15-16: De prototipo a production-grade | E03 |
| Sec 17: Arquitectura recomendada | E03 |
| Sec 18: Debugging y trazabilidad | E03 |
| Sec 19: Buenas practicas | E03, E04, E08 |
| Sec 22: Checklist de produccion | E10 |

---

## Conceptos de M3L1 y su equivalente en M3L2

| Concepto M3L1 | Equivalente LangChain M3L2 | Notebook |
|---|---|---|
| `def weather_tool(city)` | `@tool def weather_tool(city: str)` | E00 |
| `choose_action(state)` manual | LLM con tool binding | E00 |
| `for step in range(max_steps)` | `AgentExecutor(max_iterations=5)` | E00 |
| `trace = []` + prints manuales | `verbose=True` | E00 |
| f-string como prompt | `ChatPromptTemplate` | E01 |
| `openai.chat.completions.create()` | `ChatOpenAI` | E04 |
| `response.choices[0].message.content` | `StrOutputParser` | E05 |
| Busqueda naive (manda todo) | `FAISS` + `similarity_search` | E07 |
| Tool de busqueda manual | `Retriever` (`as_retriever`) | E08 |
| Pipeline script completo | LCEL RAG chain | E02, E03 |

---

## Progresion recomendada para la clase (40 minutos)

### Camino rapido (conceptos base, sin RAG completo)

```
E00 (15 min) -> E04 (6 min) -> E01 (10 min) -> discusion (9 min)
```

### Camino RAG (mostrar el pipeline completo)

```
E04 (6 min) -> E01 (6 min) -> E07 (6 min) -> E08 (6 min) -> E03 demo (10 min) -> discusion (12 min)
```

### Camino recomendado para un alumno solo

```
E00 -> E01 -> E04 -> E05 -> E02 -> E06 -> E07 -> E08 -> E03 -> E10
```

El orden sigue la progresion de la lecture: primero el puente con M3L1,
despues los componentes individuales, despues el pipeline completo,
finalmente la refactorizacion libre.

---

## Notas para el instructor

- **E00** es el ejercicio mas importante: sin el puente conceptual, los demas parecen herramientas sueltas.
- **E01** no necesita API key: util para mostrar en vivo sin depender de la red.
- **E03** es el ejercicio mas complejo (29 celdas): conviene haberlo ejecutado antes de clase.
- **E10** no tiene TODOs guiados: ideal para homework o taller grupal.
- Los Resolution notebooks tienen el codigo completo: se pueden usar como solucionario o demostracion.
