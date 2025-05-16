---
lab:
  title: Supervisión de una aplicación de IA generativa
---

# Supervisión de una aplicación de IA generativa

Este ejercicio dura aproximadamente **30** minutos.

> **Nota**: En este ejercicio se presupone cierta familiaridad con Fundición de IA de Azure, por lo que algunas instrucciones son intencionadamente menos detalladas para fomentar la exploración más activa y el aprendizaje práctico.

## Introducción

En este ejercicio, habilitarás la supervisión de una aplicación de finalización de chat y verás su rendimiento en Azure Monitor. Interactúas con el modelo implementado para generar datos, ver los datos generados a través del panel de aplicaciones de Insights para IA generativa y configurar alertas para ayudar a optimizar la implementación del modelo.

## 1. Configuración del entorno

Para completar las tareas de este ejercicio, necesitas:

- Un centro de Fundición de IA de Azure,
- Un proyecto de Fundición de IA de Azure,
- Un modelo implementado (como GPT-4o),
- Un recurso de Application Insights conectado.

### A Creación de un proyecto y un centro de Fundición de IA

Para configurar rápidamente un centro y un proyecto, se proporcionan instrucciones sencillas para usar la interfaz de usuario del portal de la Fundición de IA de Azure.

1. Ve al portal de la Fundición de IA de Azure: Abre [https://ai.azure.com](https://ai.azure.com).
1. Inicie sesión con sus credenciales de Azure.
1. Crear un proyecto:
    1. Ve a **Todos los centros y proyectos**.
    1. Selecciona **+ Nuevo proyecto**.
    1. Escribe un **nombre de proyecto**.
    1. Cuando se te solicite, **crea un nuevo centro**.
    1. Personalización del centro:
        1. Elige la **suscripción**, el **grupo de recursos** la **ubicación**, etc.
        1. Conecta un **nuevo recurso de Servicios de Azure AI** (omite la búsqueda por IA).
    1. Revise y seleccione **Crear**.
1. **Espera a que se complete la implementación** (aproximadamente de 1 a 2 minutos).

### B. Implementación de un modelo

Para generar datos que puedas supervisar, primero debes implementar un modelo e interactuar con él. En las instrucciones se te pide que implementes un modelo GPT-4o, pero **puedes usar cualquier modelo** de la colección de Azure OpenAI Service que tienes disponible.

1. En el menú de la izquierda, en la sección **Mis recursos**, selecciona la página **Modelos y puntos de conexión**.
1. Implementa un **modelo base** y elige **gpt-4o**.
1. **Personaliza los detalles de implementación**.
1. Establece la **capacidad** en **5000 tokens por minuto (TPM).**

El centro y el proyecto están listos, con todos los recursos de Azure necesarios aprovisionados de forma automática.

### C. Conectar Application Insights

Conecta Application Insights al proyecto en Fundición de IA de Azure para iniciar los datos recopilados para la supervisión.

1. Abre el proyecto en el portal de la Fundición de IA de Azure.
1. En el menú de la izquierda, selecciona la página **Seguimiento**.
1. **Crea un nuevo** recurso de Application Insights para conectarse a la aplicación.
1. Escribe un **nombre del recurso de Application Insights**.

Application Insights ahora está conectado al proyecto y los datos comenzarán a recopilarse para su análisis.

## 2. Interacción con un modelo implementado

Interactuarás con el modelo implementado de manera programática, mediante la configuración de una conexión al proyecto de Fundición de IA de Azure con el uso de Azure Cloud Shell. Esto te permitirá enviar un mensaje al modelo y generar datos de supervisión.

### A Conexión con un modelo mediante Cloud Shell

Empieza por recuperar la información necesaria para autenticarse para interactuar con el modelo. A continuación, accederás a Azure Cloud Shell y actualizarás la configuración para enviar las indicaciones proporcionadas al propio modelo implementado.

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Detalles del proyecto**, anota la **Cadena de conexión del proyecto**.
1. **Guarda** la cadena en un Bloc de notas. Usarás esta cadena de conexión para conectarte al proyecto en una aplicación cliente.
1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente).
1. En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.
1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.
1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica**.

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe y ejecuta el siguiente comando:

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Este comando clona el repositorio de GitHub que contiene los archivos de código de este ejercicio.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-foundry/Files/07
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que necesitas:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Escribe el siguiente comando para abrir el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código:

    1. Reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el portal de la Fundición de IA de Azure).
    1. Reemplaza el marcador de posición **your_model_deployment** por el nombre que asignaste a la implementación del modelo gpt-4o (de manera predeterminada `gpt-4o`).

1. *Después* de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **haz clic con el botón derecho y luego en Guardar** para **guardar los cambios**.

### B. Envío de indicaciones al modelo implementado

Ahora ejecutarás varios scripts que envían indicaciones diferentes al modelo implementado. Estas interacciones generan datos que puedes observar más adelante en Azure Monitor.

1. Ejecuta el siguiente comando para **ver el primer script** que se ha proporcionado:

    ```
   code start-prompt.py
    ```

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para **ejecutar el script**:

    ```
   python start-prompt.py
    ```

    El modelo generará una respuesta, que se capturará con Application Insights para su posterior análisis. Vamos a variar nuestras indicaciones para explorar sus efectos.

1. **Abre y revisa el script**, donde la solicitud indica al modelo que **solo responda con una oración y una lista**:

    ```
   code short-prompt.py
    ```

1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos:

    ```
   python short-prompt.py
    ```

1. El siguiente script tiene un objetivo similar, pero incluye las instrucciones para la salida en el **mensaje del sistema** en lugar del mensaje de usuario:

    ```
   code system-prompt.py
    ```

1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos:

    ```
   python system-prompt.py
    ```

1. Por último, vamos a intentar desencadenar un error mediante la ejecución de una indicación con **demasiados tokens**:

    ```
   code error-prompt.py
    ```

1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos: ¡Ten en cuenta que es muy **probable que experimentes un error!**

    ```
   python error-prompt.py
    ```

Ahora que has interactuado con el modelo, puedes revisar los datos en Azure Monitor.

> **Nota**: Los datos de supervisión pueden tardar unos minutos en mostrarse en Azure Monitor.

## 4. Visualización de datos de supervisión en Azure Monitor

Para ver los datos recopilados de las interacciones del modelo, tendrás acceso al panel que se vincula a un libro de Azure Monitor.

### A Ve a Azure Monitor en el portal de la Fundición de IA de Azure

1. Ve a la pestaña del explorador con el **portal de la Fundición de IA de Azure** abierto.
1. Usa el menú de la izquierda y selecciona **Seguimiento**.
1. Selecciona el vínculo en la parte superior, que indica **Consulta el panel de aplicaciones de Insights para IA generativa**. El vínculo abrirá Azure Monitor en una nueva pestaña.
1. Revisa la **información general** que proporciona datos resumidos de las interacciones con el modelo implementado.

## 5. Interpretación de las métricas de supervisión en Azure Monitor

Ahora es el momento de profundizar en los datos y empezar a interpretar lo que te dice.

### A Revisión del uso del token

Céntrate primero en la sección de **uso de tokens** y revisa las siguientes métricas:

- **Tokens de indicación**: El número total de tokens usados en la entrada (las indicaciones que enviaste) en todas las llamadas del modelo.

> Piensa en esto como el *coste de hacer una pregunta* al modelo.

- **Tokens de finalización**: El número de tokens que devolvió el modelo como salida, básicamente la longitud de las respuestas.

> Los tokens de finalización generados suelen representar la mayor parte del uso y el coste del token, especialmente para respuestas largas o detalladas.

- **Total de tokens**: Los tokens de solicitud totales combinados y los tokens de finalización.

> Métrica más importante para la facturación y el rendimiento, ya que impulsa la latencia y el coste.

- **Total de llamadas**: El número de solicitudes de inferencia independientes, que es la cantidad de veces que se llamó al modelo.

> Resulta útil para analizar el rendimiento y comprender el coste medio por llamada.

### B. Comparación de las solicitudes individuales

Desplázate hacia abajo para buscar los **intervalos de IA generativa**, que se visualizan como una tabla en la que cada solicitud se representa como una nueva fila de datos. Revisa y compara el contenido de las siguientes columnas:

- **Estado**: Indica si una llamada de modelo se realizó correctamente o no.

> Úsalo para identificar solicitudes problemáticas o errores de configuración. Es probable que se produzca un error en la última solicitud porque la misma era demasiado larga.

- **Duración**: Muestra cuánto tiempo tardó el modelo en responder, en milisegundos.

> Compara entre filas para explorar qué patrones de solicitud dan lugar a tiempos de procesamiento más largos.

- **Entrada**: Muestra el mensaje de usuario que se envió al modelo.

> Usa esta columna para evaluar qué formulaciones de solicitud son eficaces o problemáticas.

- **Sistema**: Muestra el mensaje del sistema usado en la solicitud (si hay alguna).

> Compara las entradas para evaluar el impacto del uso o el cambio de mensajes del sistema.

- **Salida**: Contiene la respuesta del modelo.

> Úsalo para evaluar el nivel de detalle, la relevancia y la coherencia. Especialmente en relación con los recuentos de tokens y la duración.

## 6. (OPCIONAL) Creación de una alerta

Si tienes tiempo adicional, intenta configurar una alerta para que recibas una notificación cuando la latencia del modelo supere un umbral determinado. Se trata de un ejercicio diseñado para desafiarte, lo que significa que las instrucciones son intencionadamente menos detalladas.

- En Azure Monitor, crea una **nueva regla de alertas** para el proyecto y el modelo de Fundición de IA de Azure.
- Elige una métrica como **Duración de la solicitud (ms)** y define un umbral (por ejemplo, mayor que 4000 ms).
- Crea un **nuevo grupo de acción** para definir cómo se te notificará.

Las alertas te ayudan a prepararte para la producción mediante el establecimiento de una supervisión proactiva. Las alertas que configures dependerán de las prioridades del proyecto y de cómo el equipo haya decidido medir y mitigar los riesgos.

## Dónde encontrar otros laboratorios

Puedes explorar laboratorios y ejercicios adicionales en el [Portal de Aprendizaje de la Fundición de IA de Azure](https://ai.azure.com) o consultar la **sección de laboratorio** del curso para ver otras actividades disponibles.
