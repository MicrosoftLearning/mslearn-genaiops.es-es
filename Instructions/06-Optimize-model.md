---
lab:
  title: Optimización del modelo mediante un conjunto de datos sintético
  description: Obtén información sobre cómo crear conjuntos de datos sintéticos y usarlos para mejorar el rendimiento y la confiabilidad del modelo.
---

## Optimización del modelo mediante un conjunto de datos sintético

La optimización de una aplicación de IA generativa implica aprovechar los conjuntos de datos para mejorar el rendimiento y la confiabilidad del modelo. Mediante el uso de datos sintéticos, los desarrolladores pueden simular una amplia gama de escenarios y casos perimetrales que podrían no estar presentes en datos reales. Además, la evaluación de las salidas del modelo es fundamental para obtener aplicaciones de IA confiables y de alta calidad. Todo el proceso de optimización y evaluación se puede administrar de forma eficaz mediante el SDK de evaluación de Azure AI, que proporciona herramientas y marcos sólidos para simplificar estas tareas.

Este ejercicio durará aproximadamente **30** minutos\*.

> \* Esta estimación no incluye la tarea opcional al final del ejercicio.
## Escenario

Imagina que quieres crear una aplicación de guía inteligente con tecnología de IA para mejorar las experiencias de los visitantes en un museo. La aplicación tiene como objetivo responder a preguntas sobre figuras históricas. Para evaluar las respuestas de la aplicación, debes crear un conjunto de datos sintético completo de preguntas y respuestas que abarque varios aspectos de estas personalidades y su trabajo.

Has seleccionado un modelo GPT-4 para proporcionar respuestas generativas. Ahora quieres reunir un simulador que genere interacciones contextualmente relevantes, evaluando el rendimiento de la IA en diferentes escenarios.

Vamos a empezar implementando los recursos necesarios para crear esta aplicación.

## Creación de un centro y un proyecto de Azure AI

Puedes crear manualmente un centro y un proyecto de Azure AI a través del Portal de la Fundición de IA de Azure, así como implementar los modelos usados en el ejercicio. Sin embargo, también puedes automatizar este proceso mediante el uso de una aplicación de plantilla con [Azure Developer CLI (azd)](https://aka.ms/azd).

1. En un explorador web, abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica**.

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para el repositorio de este ejercicio:

    ```powershell
   rm -r mslearn-genaiops -f
   git clone https://github.com/MicrosoftLearning/mslearn-genaiops
    ```

1. Una vez que se ha clonado el repositorio, escribe los siguientes comandos para inicializar la plantilla de inicio. 
   
    ```powershell
   cd ./mslearn-genaiops/Starter
   azd init
    ```

1. Una vez que se te solicite, asigna al nuevo entorno un nombre, ya que se usará como base para proporcionar nombres únicos a todos los recursos aprovisionados.
        
1. A continuación, escribe el siguiente comando para ejecutar la plantilla de inicio. Aprovisionará un centro de IA con recursos dependientes, un proyecto de IA, los servicios de IA y un punto de conexión en línea. También implementarás los modelos GPT-4 Turbo, GPT-4o y GPT-4o mini.

    ```powershell
   azd up  
    ```

1. Cuando se te solicite, elige la suscripción que deseas usar y, a continuación, elige una de las siguientes ubicaciones para el aprovisionamiento de recursos:
   - Este de EE. UU.
   - Este de EE. UU. 2
   - Centro-Norte de EE. UU
   - Centro-sur de EE. UU.
   - Centro de Suecia
   - Oeste de EE. UU.
   - Oeste de EE. UU. 3
    
1. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más.

    > **Nota**: los recursos de Azure OpenAI están restringidos a nivel de inquilino por unas cuotas regionales. Las regiones enumeradas anteriormente incluyen la cuota predeterminada para los tipos de modelo usados en este ejercicio. Elegir aleatoriamente una región reduce el riesgo de que una sola región alcance su límite de cuota. En caso de que se alcance un límite de cuota, es posible que tengas que crear otro grupo de recursos en otra región. Más información sobre la [disponibilidad del modelo por región](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)

    <details>
      <summary><b>Sugerencia de solución de problemas</b>: no hay cuota disponible en una región determinada</summary>
        <p>Si recibes un error de implementación para cualquiera de los modelos debido a que no hay una cuota disponible en la región elegida, prueba a ejecutar los comandos siguientes:</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        Reemplaza <code>new_env_name</code>, <code>new_rg_name</code> y <code>new_location</code> por nuevos valores. La nueva ubicación deberá ser una de las regiones enumeradas al principio del ejercicio, por ejemplo <code>eastus2</code>, <code>northcentralus</code>, etc.
        </ul>
    </details>

1. Una vez estén aprovisionados todos los recursos, usa los siguientes comandos para capturar el punto de conexión y la clave de acceso al recurso de Servicios de IA. Ten en cuenta que debes reemplazar `<rg-env_name>` y `<aoai-xxxxxxxxxx>` por los nombres del grupo de recursos y el recurso de Servicios de IA. Ambos se imprimen en la salida de la implementación.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copia estos valores, ya que se usarán más adelante.

## Configuración del entorno de desarrollo en Cloud Shell

Para experimentar e iterar rápidamente, usarás un conjunto de scripts de Python en Cloud Shell.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ir a la carpeta con los archivos de código usados en este ejercicio:

     ```powershell
    cd ~/mslearn-genaiops/Files/06/
     ```

1. Escribe los siguientes comandos para activar un entorno virtual e instalar las bibliotecas que necesitas:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. Escribe el siguiente comando para abrir el archivo de configuración que se ha proporcionado:

    ```powershell
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza los marcadores de posición **your_azure_openai_service_endpoint** y **your_azure_openai_service_api_key** por los valores de punto de conexión y clave que copiaste anteriormente.
1. *Después* de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o **clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Generación de datos sintéticos

Ahora ejecutarás un script que genera un conjunto de datos sintético y lo usarás para evaluar la calidad del modelo entrenado previamente.

1. Ejecuta el siguiente comando para **editar el script** que se ha proporcionado:

    ```powershell
   code generate_synth_data.py
    ```

1. En el script, busca **# Definir función de devolución de llamada**.
1. Pega el siguiente código debajo de este comentario:

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    Puedes aportar cualquier punto de conexión de aplicación para simularlo especificando una función de devolución de llamada de destino. En este caso, usarás una aplicación que es un LLM con un archivo Prompty `application.prompty`. La función de devolución de llamada anterior procesa cada mensaje generado por el simulador mediante la realización de las siguientes tareas:
    * Recupera el mensaje de usuario más reciente.
    * Carga un flujo de avisos desde application.prompty.
    * Genera una respuesta mediante el flujo de solicitud.
    * Da formato a la respuesta para cumplir el protocolo de chat de OpenAI.
    * Anexa la respuesta del asistente a la lista de mensajes.

    >**Nota**: Para obtener más información sobre el uso de Prompty, consulta la [documentación de Prompty](https://www.prompty.ai/docs).

1. A continuación, busca **# Ejecutar el simulador**.
1. Pega el siguiente código debajo de este comentario:

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   El código anterior inicializará el simulador y lo ejecutará para generar conversaciones sintéticas basadas en un texto extraído previamente de Wikipedia.

1. A continuación, busca **# Evaluar el modelo**.
1. Pega el siguiente código debajo de este comentario:

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    Ahora que tienes un conjunto de datos, puedes evaluar la calidad y eficacia de la aplicación de IA generativa. En el código anterior, usarás la base como métrica de calidad.

1. Guarda los cambios.
1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para **ejecutar el script**:

    ```
   python generate_synth_data.py
    ```

    Una vez finalizado el script, puedes descargar los archivos de salida; para ello, ejecuta `download simulation_output.jsonl` y `download groundedness_eval_output.json` revisa su contenido. Si la métrica de base no está cerca de la versión 3.0, puedes cambiar los parámetros de LLM, como `temperature`, `top_p`, `presence_penalty` o `frequency_penalty` en el archivo `application.prompty` y volver a ejecutar el script para generar un nuevo conjunto de datos para su evaluación. También puedes cambiar `wiki_search_term` para obtener un conjunto de datos sintético basado en un contexto diferente.

## (OPCIONAL) Ajuste del modelo

Si tienes tiempo adicional, puedes usar el conjunto de datos generado para ajustar el modelo en la Fundición de IA de Azure. El ajuste preciso depende de los recursos de infraestructura en la nube, lo que puede tardar una cantidad variable de tiempo en aprovisionar en función de la capacidad del centro de datos y la demanda simultánea.

1. Abre una nueva pestaña del explorador y ve al [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure.
1. En la página principal de la Fundición de IA, selecciona el proyecto creado al principio del ejercicio.
1. Ve a la página **Ajuste** de la sección **Crear y personalizar**, con el menú de la izquierda.
1. Selecciona el botón para agregar un nuevo modelo de ajuste, selecciona el modelo **gpt-4o** y, a continuación, selecciona **Siguiente**.
1. **Ajusta** el modelo mediante la siguiente configuración:
    - **Versión del modelo**: *Selecciona la versión predeterminada*
    - **Método de personalización**: supervisado
    - **Sufijo del modelo**: `ft-travel`
    - **Recurso de IA conectado**: *selecciona la conexión que se ha creado al crear tu centro. Debería estar seleccionada de forma predeterminada.*
    - **Datos de aprendizaje**: carga los archivos

    <details>  
    <summary><b>Sugerencia de solución de problemas: error de permisos</b></summary>
    <p>Si recibes un error de permisos, prueba lo siguiente para solucionar los problemas:</p>
    <ul>
        <li>En Azure Portal, selecciona el recurso Servicios de IA.</li>
        <li>En Administración de recursos, en la pestaña Identidad, confirma que se trata de una identidad administrada asignada por el sistema.</li>
        <li>Ve a la cuenta de almacenamiento asociada. En la página IAM, agrega la asignación de roles <em>Propietario de datos de Storage Blob</em>.</li>
        <li>En <strong>Asignar acceso a</strong>, elige <strong>Identidad administrada</strong>, <strong>+Seleccionar miembros</strong>, selecciona <strong>Todas las identidades administradas asignadas por el sistema</strong> y selecciona tu recurso de Servicios de Azure AI.</li>
        <li>Selecciona Revisar y asignar para guardar la nueva configuración y volver a intentar el paso anterior.</li>
    </ul>
    </details>

    - **Cargar archivo**: selecciona el archivo JSON que descargaste en el paso anterior.
    - **Datos de validación**: ninguno
    - **Parámetros de tarea**: *mantén la configuración predeterminada*
1. El ajuste se iniciará y puede tardar algún tiempo en completarse.

    > **Nota**: el ajuste preciso y la implementación pueden tardar un tiempo significativo (30 minutos o más), por lo que es posible que tengas que volver a comprobarlo periódicamente. Puedes ver más detalles del progreso hasta ahora seleccionando el trabajo de ajuste del modelo y viendo su pestaña **Registros**.

## (OPCIONAL) Implementación del modelo ajustado

Cuando se ha completado correctamente el ajuste preciso, puedes implementar el modelo con ajuste preciso.

1. Selecciona el vínculo de trabajo de ajuste para abrir su página de detalles. A continuación, selecciona la pestaña **Métricas** y explora las métricas de ajuste.
1. Implementa el modelo con ajuste preciso con las siguientes configuraciones:
    - **Nombre de implementación**: *nombre válido para la implementación de modelo*
    - **Tipo de implementación**: estándar
    - **Límite de frecuencia de tokens por minuto (miles)**: 5000 *(o el máximo disponible en la suscripción si es inferior a 5000)
    - **Filtro de contenido**: valor predeterminado
1. Espera a que se complete la implementación para poder probarla, esto puede tardar un tiempo. Comprueba el **estado de aprovisionamiento** hasta que se haya realizado correctamente (es posible que tengas que actualizar el explorador para ver el estado actualizado).
1. Cuando la implementación esté lista, ve al modelo con ajuste preciso y selecciona **Abrir en área de juegos**.

    Ahora que has implementado el modelo ajustado, puedes probarlo en el sitio de prueba de chat como lo harías con cualquier modelo base.

## Conclusión

En este ejercicio has creado un conjunto de datos sintético que simula una conversación entre un usuario y una aplicación de finalización de chat. Con este conjunto de datos, puedes evaluar la calidad de las respuestas de la aplicación y ajustarla para lograr los resultados deseados.

## Limpieza

Si has terminado de explorar Servicios de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com?azure-portal=true) en una nueva pestaña del explorador) y visualiza el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
