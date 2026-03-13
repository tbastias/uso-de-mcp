# Cómo funcionan MCP y las llamadas a herramientas en LLM

## Introducción

Los modelos de lenguaje (LLM) modernos pueden extender sus capacidades utilizando **herramientas externas** (tools), lo que permite acceder a datos, ejecutar código o consultar servicios externos. Esta capacidad es conocida como **tool calling o function calling**, y es la base de muchos sistemas de **IA agentica**.

Para estandarizar cómo se definen y se integran estas herramientas con los modelos, surge **MCP (Model Context Protocol)**.

---

## Una vista simplificada de la invocación de herramientas con LLM

El funcionamiento básico de un LLM no cambia:  
el modelo genera texto **token por token** hasta llegar a un resultado final.

Cuando se utilizan herramientas, el prompt inicial incluye:

- Instrucciones sobre **cómo invocar herramientas**
- Una **lista de herramientas disponibles**
- El **schema de cada herramienta** (por ejemplo en JSON)

El modelo puede entonces generar una salida estructurada que indique que desea llamar a una función o herramienta.

Flujo simplificado:

1. El usuario envía una pregunta.
2. El prompt incluye las herramientas disponibles.
3. El LLM decide si necesita usar una herramienta.
4. Si es necesario, genera una **llamada a función estructurada**.
5. La aplicación ejecuta la herramienta.
6. El resultado se vuelve a enviar al LLM.
7. El LLM usa ese resultado para generar la respuesta final.

Es importante notar que **el modelo no ejecuta la herramienta directamente**; el proceso externo (cliente o backend) interpreta la llamada y ejecuta la acción.

---

## No todos los LLM funcionan bien con tool calling

Para que este mecanismo funcione correctamente, los modelos suelen estar:

- **Fine-tuned** o ajustados para realizar llamadas a funciones.
- Entrenados para generar estructuras correctas (por ejemplo JSON).

Aun así, durante el desarrollo es común que el modelo genere llamadas mal formadas o incorrectas.

---

## Cómo funcionan las herramientas en LLM-as-a-service

Cuando se usan proveedores como APIs de LLM, el proceso normalmente ocurre a través de **peticiones HTTP**.

La aplicación envía un JSON que incluye:

- historial de conversación
- prompt del usuario
- lista de herramientas disponibles

El proveedor procesa la respuesta y devuelve:

- texto normal, o
- una estructura que indica **tool calls**.

Si se devuelve una llamada a herramienta:

1. La aplicación ejecuta la herramienta.
2. El resultado se agrega a la conversación.
3. Se vuelve a consultar al LLM.

Este proceso puede repetirse varias veces.

---

## MCP entra en escena

El problema que surge es **cómo definir herramientas de forma estándar**, especialmente si:

- diferentes desarrolladores crean herramientas
- diferentes aplicaciones usan distintos LLM
- cada proveedor tiene su propio formato

Aquí es donde aparece **MCP (Model Context Protocol)**.

MCP es un **estándar abierto para describir herramientas y exponerlas a los LLM**. Su objetivo es permitir que terceros publiquen herramientas que puedan ser utilizadas por distintos clientes o modelos.

En lugar de implementar cada integración manualmente:

- las herramientas se publican mediante **servidores MCP**
- los clientes recopilan esas herramientas
- luego las envían al LLM como funciones disponibles

Esto permite construir ecosistemas donde los modelos pueden utilizar herramientas desarrolladas por terceros.

---

## Cómo se exponen herramientas MCP a los LLM

Cuando una aplicación utiliza herramientas MCP:

1. El cliente local detecta las herramientas disponibles en servidores MCP.
2. Convierte esas herramientas en el formato que el proveedor del LLM espera.
3. Las incluye en la petición al modelo.

Por ejemplo:

- si se usa un modelo de OpenAI,
- las herramientas MCP pueden enviarse como **function calls**.

De esta forma, el modelo puede invocar herramientas MCP igual que cualquier otra función.

---

## Los LLM no “saben” qué es MCP

Un punto importante es que:

**Los LLM no conocen MCP como protocolo.**

Para el modelo:

- solo existen **herramientas disponibles en el prompt**.

MCP actúa en otra capa del sistema:

- define cómo se **descubren y describen herramientas**
- facilita su integración en aplicaciones cliente
- pero el modelo solo ve funciones que puede llamar.

---

## Idea clave

- **Tool calling**: mecanismo que permite al LLM invocar funciones.
- **MCP**: estándar para **definir y distribuir herramientas** que luego pueden ser usadas mediante tool calling.

En otras palabras:

- tool calling es la **capacidad del modelo**
- MCP es la **infraestructura que organiza las herramientas**.
