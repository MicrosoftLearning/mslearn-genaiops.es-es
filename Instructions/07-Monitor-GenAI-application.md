---
lab:
  title: Supervisión de una aplicación de IA generativa
  description: Aprende a supervisar las interacciones con el modelo implementado y obtén información sobre cómo optimizar su uso con la aplicación de IA generativa.
---

# Supervisión de una aplicación de IA generativa

Este ejercicio dura aproximadamente **30** minutos.

> **Nota**: en este ejercicio se presupone que tienes algún conocimiento de Fundición de IA de Azure, por lo que algunas instrucciones son intencionadamente menos detalladas para fomentar la exploración más activa y el aprendizaje práctico.

## Introducción

En este ejercicio, habilitarás la supervisión de una aplicación de finalización de chat y verás su rendimiento en Azure Monitor. Interactuarás con el modelo implementado para generar datos, verás los datos generados a través del panel de información para aplicaciones de IA generativa y configurarás alertas para ayudar a optimizar la implementación del modelo.

## Configuración del entorno

Para completar las tareas de este ejercicio, necesitas:

- Un proyecto de Fundición de IA de Azure,
- Un modelo implementado (como GPT-4o),
- Un recurso de Application Insights conectado.

### Implementación de un modelo en un proyecto de Fundición de IA de Azure

Para configurar rápidamente un proyecto de Fundición de IA de Azure, a continuación se proporcionan instrucciones sencillas para usar la interfaz de usuario del Portal de la Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure.
1. En la página principal, en la sección **Explorar modelos y funcionalidades**, busca el modelo `gpt-4o`, que usaremos en nuestro proyecto.
1. En los resultados de la búsqueda, selecciona el modelo **gpt-4o** para ver sus detalles y, a continuación, en la parte superior de la página del modelo, selecciona **Usar este modelo**.
1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.
1. Selecciona **Personalizar** y especifica la siguiente configuración para el proyecto:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *selecciona cualquier ubicación compatible con los servicios de IA***\*

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que se cree el proyecto, incluida la implementación del modelo gpt-4 que seleccionaste.
1. En el panel de navegación de la izquierda, seleccione **Información general** para ver la página principal del proyecto.
1. En el área **Puntos de conexión y claves**, asegúrese de que la biblioteca **Fundición de IA de Azure** esté seleccionada y vea el **punto de conexión del proyecto de Fundición de IA de Azure**.
1. **Guarde** el punto de conexión en un cuaderno. Usarás este punto de conexión para conectarte al proyecto en una aplicación cliente.

### Conectar Application Insights

Conecta Application Insights al proyecto en Fundición de IA de Azure para empezar a recopilar los datos para la supervisión.

1. Usa el menú de la izquierda y selecciona la página **Seguimiento**.
1. **Crea un nuevo** recurso de Application Insights para conectarse a la aplicación.
1. Escribe un nombre de recurso de Application Insights y selecciona **Crear**.

Application Insights ahora está conectado al proyecto y los datos comenzarán a recopilarse para su análisis.

## Interacción con un modelo implementado

Interactuarás con el modelo implementado mediante programación configurando una conexión al proyecto de Fundición de IA de Azure con Azure Cloud Shell. Esto te permitirá enviar una indicación al modelo y generar datos de supervisión.

### Conexión con un modelo mediante Cloud Shell

Empieza por recuperar la información necesaria para autenticarte para interactuar con el modelo. A continuación, accederás a Azure Cloud Shell y actualizarás la configuración para enviar las indicaciones proporcionadas a tu propio modelo implementado.

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente).
1. En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.
1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.
1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica**.

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe y ejecuta los comandos siguientes:

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Este comando clona el repositorio de GitHub que contiene los archivos de código de este ejercicio.

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que necesitas:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects opentelemetry-instrumentation-openai-v2 azure-monitor-opentelemetry
    ```

1. Escribe el siguiente comando para abrir el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código:

    1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** por el punto de conexión de tu proyecto (copiado de la página **Información general** del proyecto del portal de la Fundición de IA de Azure).
    1. Reemplaza el marcador de posición **your_model_deployment** por el nombre que asignaste a la implementación del modelo GPT-4o (de manera predeterminada `gpt-4o`).

1. *Después* de reemplazar el marcador de posición, en el editor de código, usa el comando **CTRL+S** o **clic con el botón derecho > Guardar** para **guardar los cambios** y, a continuación, usa el comando **CTRL+Q** o **clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Envío de indicaciones al modelo implementado

Ahora ejecutarás varios scripts que envían indicaciones diferentes al modelo implementado. Estas interacciones generan datos que podrás observar más adelante en Azure Monitor.

1. Ejecuta el siguiente comando para **ver el primer script** que se ha proporcionado:

    ```
   code start-prompt.py
    ```

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
   az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.
1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python start-prompt.py
    ```

    El modelo generará una respuesta, que se capturará con Application Insights para su posterior análisis. Vamos a variar nuestras indicaciones para explorar sus efectos.

1. **Abre y revisa el script**, donde la indicación indica al modelo que **responda solo con una frase y una lista**:

    ```
   code short-prompt.py
    ```

1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos:

    ```
   python short-prompt.py
    ```

1. El siguiente script tiene un objetivo similar, pero incluye las instrucciones para la salida en el **mensaje del sistema** en lugar del mensaje del usuario:

    ```
   code system-prompt.py
    ```

1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos:

    ```
   python system-prompt.py
    ```

1. Por último, vamos a intentar desencadenar un error ejecutando una indicación con **demasiados tokens**:

    ```
   code error-prompt.py
    ```

1. **Ejecuta el script** escribiendo el comando siguiente en la línea de comandos. Ten en cuenta que es muy **probable que se produzca un error.**

    ```
   python error-prompt.py
    ```

Ahora que has interactuado con el modelo, puedes revisar los datos en Azure Monitor.

> **Nota**: los datos de supervisión pueden tardar unos minutos en mostrarse en Azure Monitor.

## Visualización de datos de supervisión en Azure Monitor

Para ver los datos recopilados de las interacciones del modelo, tendrás acceso al panel que se vincula a un libro de Azure Monitor.

### Ve a Azure Monitor desde el Portal de la Fundición de IA de Azure

1. Ve a la pestaña del explorador con el **Portal de la Fundición de IA de Azure** abierto.
1. Use el menú de la izquierda y seleccione **Supervisión**.
1. Seleccione el **uso de recursos** y revise los datos resumidos de las interacciones con el modelo implementado.

> **Nota**: También puede seleccionar **Explorador de métricas de Azure Monitor** en la parte inferior de la página Supervisión para obtener una vista completa de todas las métricas disponibles. El vínculo abrirá Azure Monitor en una nueva pestaña.

## Interpretación de las métricas de supervisión

Ahora es el momento de profundizar en los datos y empezar a interpretar lo que dicen.

### Revisión del uso de tokens

Céntrate primero en la sección **uso de tokens** y revisa las métricas siguientes:

- **Solicitudes totales**: número de solicitudes de inferencia independientes, que es la cantidad de veces que se llamó al modelo.

> Resulta útil para analizar el rendimiento y comprender el coste medio por llamada.

- **Número total de tokens**: los tokens de indicación totales combinados y los tokens de finalización.

> Métrica más importante para la facturación y el rendimiento, ya que impulsa la latencia y el coste.

- **Recuento de tokens de indicación**: número total de tokens usados en la entrada (las indicaciones enviadas) en todas las llamadas del modelo.

> Piensa en esto como el *coste de plantear* al modelo una pregunta.

- **Recuento de tokens de finalización**: número de tokens que devolvió el modelo como salida, básicamente la longitud de las respuestas.

> Los tokens de finalización generados suelen representar la mayor parte del uso y el coste de los tokens, especialmente para respuestas largas o detalladas.

### Comparación de las indicaciones individuales

1. Usa el menú de la izquierda y selecciona **Seguimiento**. Expanda cada intervalo de inteligencia artificial de generación **generate_completion** para ver sus intervalos secundarios. Cada indicación se representa como una nueva fila de datos. Revisa y compara el contenido de las columnas siguientes:

- **Entrada**: muestra el mensaje del usuario que se envió al modelo.

> Usa esta columna para evaluar qué formulaciones de indicaciones son eficaces o problemáticas.

- **Salida**: contiene la respuesta del modelo.

> Úsalo para evaluar el nivel de detalle, la relevancia y la coherencia. Especialmente en relación con los recuentos de tokens y la duración.

- **Duración**: muestra cuánto tiempo tardó el modelo en responder, en milisegundos.

> Compara entre filas para explorar qué patrones de indicación dan lugar a tiempos de procesamiento más largos.

- **Correcto**: indica si una llamada al modelo se realizó correctamente o no.

> Úsalo para identificar indicaciones problemáticas o errores de configuración. Es probable que se produzca un error en la última indicación porque era demasiado larga.

## (OPCIONAL) Creación de una alerta

Si tienes más tiempo, intenta configurar una alerta para que se te notifique cuando la latencia del modelo supere un umbral determinado. Se trata de un ejercicio diseñado para representar un desafío, lo que significa que las instrucciones son intencionadamente menos detalladas.

- En Azure Monitor, crea una **nueva regla de alertas** para el proyecto y el modelo de la Fundición de IA de Azure.
- Elige una métrica como **Duración de la solicitud (ms)** y define un umbral (por ejemplo, mayor que 4000 ms).
- Crea un **nuevo grupo de acciones** para definir cómo se te notificará.

Las alertas te ayudan a prepararte para la producción mediante el establecimiento de una supervisión proactiva. Las alertas que configures dependerán de las prioridades del proyecto y de cómo el equipo haya decidido medir y mitigar los riesgos.

## Dónde encontrar otros laboratorios

Puedes explorar laboratorios y ejercicios adicionales en el [Portal de aprendizaje de la Fundición de IA de Azure](https://ai.azure.com) o consultar la **sección de laboratorio** del curso para ver otras actividades disponibles.
