# Proyecto Final — Sistemas Inteligentes  
## Automatización logística con N8n y reimplementación programática con LangChain

**Curso:** Módulo Sistemas Inteligentes  
**Docente:** Luis Fernando Castillo  
**Período académico:** 2026-1  
**Integrantes:** Vanessa Restrepo Obando - Daniel Felipe Mejia

Este proyecto desarrolla un caso de uso de logística automatizada en dos niveles acumulativos.  
En el **Nivel 1**, se implementa un flujo visual en **n8n** para procesar solicitudes de pedidos, consultar stock, registrar pedidos y generar facturas.  
En el **Nivel 2**, se reimplementa el mismo caso de uso de forma programática usando **LangChain**, **Gemini**, **Telegram Bot** y herramientas sobre un archivo Excel con las hojas **Stock**, **Pedidos** y **Facturas**. 

---

## Caso de uso

El sistema recibe mensajes de usuario relacionados con pedidos de productos logísticos.  
A partir del mensaje, el sistema identifica la intención, extrae el producto y la cantidad solicitada, consulta el inventario disponible, registra el pedido, actualiza el stock y, cuando aplica, genera una factura. 

Ejemplos de interacción:

- `Quiero pedir PROD-003 cantidad 2`
- `Consulta stock de PROD-005`
- `Consultar estado de mi pedido`

Este caso fue elegido porque representa un proceso real y concreto de automatización operativa, con evidencia verificable en datos estructurados y respuestas observables en tiempo real.

---

## Estructura del repositorio

```text
proyecto-sistemas-inteligentes/
├── README.md
├── nivel1_n8n/
│   └── flujo_n8n.json
├── nivel2_langchain/
│   └── nivel2_gemini_telegram_stock_pedidos.ipynb
├── datos/
│   └── Registros.xlsx
└── evidencias/
    └── enlaces.txt
```

### Descripción de archivos

- `nivel1_n8n/logistic_automatization.json`: exportación del workflow funcional de n8n para el Nivel 1. 
- `nivel2_langchain/nivel2_gemini_telegram_stock_pedidos.ipynb`: notebook con la solución programática en LangChain.
- `datos/Registros.xlsx`: archivo de datos con las hojas `Stock`, `Pedidos` y `Facturas`.
- `evidencias/enlaces.txt`: enlaces al video de demostración, presentación y otros recursos entregables. 

---

## Nivel 1 — N8n

El Nivel 1 implementa un flujo visual funcional en **n8n** que automatiza el procesamiento del caso de uso.  
La entrega incluye el flujo exportado en JSON, evidencia en video y documentación en la plantilla PPTX oficial del proyecto. 

### Funcionalidad implementada

- Recepción del evento o solicitud inicial.
- Procesamiento del mensaje o entrada del usuario.
- Consulta de disponibilidad en inventario.
- Registro del pedido.
- Generación de salida o respuesta final.
- Exportación del flujo en JSON. 

### Entregables asociados

- JSON del workflow.
- Video de creación y ejecución real del flujo.
- Diapositivas 04, 05 y 06 de la plantilla PPTX. 

---

## Nivel 2 — LangChain

El Nivel 2 reimplementa el mismo caso de uso del Nivel 1 usando **LangChain** en Python, manteniendo la funcionalidad principal del sistema. La solución usa **Gemini** como LLM, un **ChatPromptTemplate** para clasificar intención y extraer entidades, y herramientas programáticas para consultar stock, crear pedidos y revisar estados. 

### Arquitectura general

Flujo conceptual:

`Telegram -> Handler Python -> ChatPromptTemplate -> Gemini -> Tools de negocio -> Respuesta al usuario` 

### Componentes principales

- **LLM:** `gemini-2.5-flash` con temperatura 0. 
- **Prompt:** `ChatPromptTemplate` con instrucciones para clasificar intención y extraer `product_id`, `cantidad` y `order_id`. 
- **Tools:** funciones para consultar stock, crear pedidos y consultar estado. 
- **Canal de entrada/salida:** bot de Telegram con `python-telegram-bot`. 
- **Persistencia:** archivo `Registros.xlsx` con hojas `Stock`, `Pedidos` y `Facturas`. 

### Operaciones implementadas

- Crear pedido.
- Validar disponibilidad de stock.
- Registrar pedido en Excel.
- Actualizar inventario.
- Generar factura cuando el pedido es despachado.
- Consultar estado del último pedido del usuario. 

---

## Archivo de datos

El proyecto usa el archivo `Registros.xlsx` como almacenamiento local de la lógica transaccional.  
Este archivo contiene las siguientes hojas: 

### Hoja `Stock`
Columnas principales:

- `producto_id`
- `descripcion_producto`
- `stock`
- `precio_unitario` 

### Hoja `Pedidos`
Columnas principales:

- `order_id`
- `cliente`
- `chat_id`
- `producto_id`
- `descripcion_producto`
- `cantidad`
- `estado`
- `stock`
- `fecha_pedido`
- `fecha_despacho`
- `total` 

### Hoja `Facturas`
Columnas principales:

- `factura_id`
- `order_id`
- `cliente`
- `monto_total`
- `fecha_factura`
- `estado_factura` 

---

## Requisitos

Para ejecutar el notebook del Nivel 2 se requieren estas librerías:

```bash
pip install -U langchain langchain-core langchain-google-genai python-telegram-bot pandas==2.2.2 openpyxl pydantic==2.12.3
```

Estas dependencias aparecen definidas directamente en el notebook entregado. 

---

## Variables de entorno

El sistema requiere dos credenciales:

- `GOOGLE_API_KEY`
- `TELEGRAM_BOT_TOKEN` 

En Google Colab pueden cargarse desde **Secrets**, y en entorno local pueden definirse como variables de entorno del sistema. 

Ejemplo local:

```bash
export GOOGLE_API_KEY="tu_api_key"
export TELEGRAM_BOT_TOKEN="tu_token"
```

---

## Ejecución del notebook

1. Abrir el archivo `nivel2_gemini_telegram_stock_pedidos.ipynb`.
2. Instalar dependencias.
3. Cargar credenciales.
4. Verificar que `Registros.xlsx` esté disponible en la misma carpeta de trabajo.
5. Ejecutar las celdas de carga, herramientas, modelo y pruebas locales.
6. Inicializar el bot de Telegram y dejar activo el polling para pruebas en tiempo real. 

En entorno notebook, el arranque del bot debe hacerse con llamadas asíncronas usando `await`, porque `initialize()`, `start()` y `start_polling()` son corrutinas. 

Ejemplo:

```python
app = build_telegram_app()
await app.initialize()
await app.start()
await app.updater.start_polling()
```

---

## Ejemplos de prueba

### Crear pedido
Entrada:

```text
Quiero pedir PROD-003 cantidad 2
```

Salida esperada:

- Se identifica intención `crear_pedido`.
- Se consulta `Stock`.
- Si hay disponibilidad, el pedido queda registrado como `DESPACHADO`.
- Se actualiza el stock.
- Se genera una factura. 

### Consultar estado
Entrada:

```text
Consultar estado de mi pedido
```

Salida esperada:

- Se recupera el último pedido asociado al usuario.
- Se responde con el `order_id`, producto, cantidad, estado, total y fechas.

---

## Decisiones de diseño

Se utilizó **Gemini** para interpretar mensajes libres en lenguaje natural y extraer una estructura controlada de intención y entidades. Esta decisión permite separar el razonamiento lingüístico del manejo determinístico de reglas de negocio.

Las operaciones críticas del sistema, como validación de stock, creación de pedido y facturación, se implementaron con funciones programáticas sobre datos estructurados. Esto mejora la trazabilidad del proceso y facilita explicar el comportamiento del sistema durante la sustentación.

No se utilizó **RAG** en este caso, porque el problema se basa en datos transaccionales estructurados y no en recuperación semántica de documentos extensos. Esta justificación es consistente con los criterios solicitados en la diapositiva 08. 

---

## Evidencias del proyecto

La evaluación exige que la funcionalidad se evidencie con ejecución real, video y documentación. Por eso este repositorio debe complementarse con: 

- Enlace al video del flujo n8n.
- Enlace al video del sistema LangChain funcionando.
- Plantilla PPTX diligenciada.
- JSON del flujo n8n.
- Notebook o repositorio Git del Nivel 2. 

---

## Uso de IA generativa

Este proyecto utilizó herramientas de IA generativa como apoyo en tareas de asistencia técnica, redacción, depuración y organización de documentación.  
Las partes asistidas incluyeron:

- apoyo en la redacción del README;
- sugerencias para estructurar el caso de uso;
- orientación para depurar errores de ejecución asíncrona del bot;
- apoyo para organizar la documentación del proyecto y la presentación. 

Todo el código entregado fue revisado, adaptado y comprendido por el equipo antes de su inclusión en la versión final del proyecto, en cumplimiento de los lineamientos de integridad académica del curso.

---

## Referencias

- n8n Documentation: https://docs.n8n.io
- LangChain Python Docs: https://python.langchain.com/docs/
- Russell, S. & Norvig, P. (2022). *Artificial Intelligence: A Modern Approach*.
- Zadeh, L. A. (1965). *Fuzzy sets*.
- Jackson, P. (1999). *Introduction to Expert Systems*.
- Goodfellow, I., Bengio, Y. & Courville, A. (2016). *Deep Learning*.
- Géron, A. (2023). *Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow*. 