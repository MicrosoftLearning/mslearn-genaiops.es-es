---
lab:
  title: Exploración de la ingeniería de indicaciones con Prompty
  description: Aprende a usar Prompty para probar y mejorar rápidamente los distintos mensajes con el modelo de lenguaje y asegúrate de que se construyan y orquesten para obtener mejores resultados.
---

## Exploración de la ingeniería de indicaciones con Prompty

Este ejercicio dura aproximadamente **45** minutos.

> **Nota**: en este ejercicio se presupone que tienes algún conocimiento de Fundición de IA de Azure, por lo que algunas instrucciones son intencionadamente menos detalladas para fomentar la exploración más activa y el aprendizaje práctico.

## Introducción

Durante la generación de ideas, puede que quieras probar y mejorar rápidamente las distintas indicaciones con el modelo de lenguaje. Hay varias maneras de abordar la ingeniería de indicaciones, a través del área de juegos en el Portal de la Fundición de IA de Azure o mediante Prompty para obtener un enfoque más orientado al código.

En este ejercicio, explorarás la ingeniería de indicaciones con Prompty en Azure Cloud Shell mediante un modelo implementado a través de Fundición de IA de Azure.

## Configuración del entorno

Para completar las tareas de este ejercicio, necesitas:

- Un centro de Fundición de IA de Azure,
- Un proyecto de Fundición de IA de Azure,
- Un modelo implementado (como GPT-4o).

### Creación de un centro y un proyecto de Azure AI

> **Nota**: Si ya tiene un proyecto de Azure AI, puede omitir este procedimiento y usar el proyecto existente.

Puede crear un proyecto de Azure AI manualmente mediante el Portal de la Fundición de IA de Azure, así como implementar el modelo usado en el ejercicio. Sin embargo, también puedes automatizar este proceso mediante el uso de una aplicación de plantilla con [Azure Developer CLI (azd)](https://aka.ms/azd).

1. En un explorador web, abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

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

1. A continuación, escribe el siguiente comando para ejecutar la plantilla de inicio. Aprovisionará un centro de IA con recursos dependientes, un proyecto de IA, los servicios de IA y un punto de conexión en línea.

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copia estos valores, ya que se usarán más adelante.

### Configurar el entorno virtual en Cloud Shell

Para experimentar e iterar rápidamente, usarás un conjunto de scripts de Python en Cloud Shell.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ir a la carpeta con los archivos de código usados en este ejercicio:

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. Escribe los siguientes comandos para activar un entorno virtual e instalar las bibliotecas que necesitas:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. Escribe el siguiente comando para abrir el archivo de configuración que se ha proporcionado:

    ```powershell
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace los marcadores de posición **ENDPOINTNAME** y **APIKEY** por el punto de conexión y los valores de clave que copió anteriormente.
1. *Después* de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o **clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Optimizar las indicaciones del sistema

En implementaciones a gran escala, es fundamental minimizar la longitud de las indicaciones del sistema y, al mismo tiempo, mantener la funcionalidad de la IA generativa. Las indicaciones cortas pueden dar lugar a tiempos de respuesta más rápidos, ya que el modelo de inteligencia artificial procesa menos tokens y también usa menos recursos de cálculo.

1. Escriba el siguiente comando para abrir el archivo de aplicación que se ha proporcionado:

    ```powershell
   code optimize-prompt.py
    ```

    Revise el código y observe que el script ejecuta el archivo de plantilla `start.prompty` que ya tiene una indicación del sistema predefinida.

1. Ejecute `code start.prompty` para revisar la indicación del sistema. Vea cómo podría acortarla, pero manteniendo su intención clara y eficaz. Por ejemplo:

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   Quite las palabras redundantes y céntrese en las instrucciones esenciales. Guarde la indicación optimizada en el archivo.

### Probar y validar la optimización

Es importante probar los cambios en la indicación para tener la seguridad de que se reduce el uso del token sin perder calidad.

1. Ejecute `code token-count.py` para abrir y revisar la aplicación de contador de tokens proporcionada en el ejercicio. Si ha usado una indicación optimizada distinta de la que se proporcionó en el ejemplo anterior, también puede usarla en esta aplicación.

1. Ejecute el script con `python token-count.py` y observe la diferencia en el recuento de tokens. Asegúrese de que la indicación optimizada sigue generando respuestas de alta calidad.

## Analizar las interacciones de usuario

Comprender cómo interactúan los usuarios con la aplicación ayuda a identificar patrones que aumentan el uso de los tokens.

1. Revise un conjunto de datos de ejemplo de indicaciones del usuario:

    - **"Resume la trama de *Guerra y paz*".**
    - **"¿Cuáles son algunos datos curiosos sobre los gatos?"**
    - **"Escribe un plan de negocio detallado para una startup que use inteligencia artificial para optimizar las cadenas de suministro".**
    - **"Traduce al francés "Hola, ¿cómo estás?".**
    - **"Explica el entrelazamiento cuántico a un niño de 10 años".**
    - **"Dame 10 ideas creativas para una historia corta de ciencia ficción".**

    Para cada uno, identifique si es probable que la IA proporcione una **respuesta corta**, **media** o **larga/compleja**.

1. Revise las clasificaciones. ¿Qué patrones observa? Considere:

    - ¿Afecta el **nivel de abstracción** (por ejemplo, creativo frente a realista) a la longitud?
    - ¿Las **indicaciones abiertas** tienden a ser más largas?
    - ¿Cómo influye la **complejidad de las instrucciones** (por ejemplo, "explícalo como si tuviera 10 años") en la respuesta?

1. Introduzca el siguiente comando para ejecutar la aplicación **optimize-prompt**:

    ```
   python optimize-prompt.py
    ```

1. Use algunos de los ejemplos proporcionados anteriormente para comprobar el análisis.
1. Ahora use la siguiente indicación de formato largo y revise su salida:

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. Vuelva a escribir esta indicación para:

    - Limitar el ámbito
    - Establecer expectativas de brevedad
    - Usar un formato o una estructura para guiar la respuesta

1. Compare las respuestas para comprobar que ha obtenido una respuesta más concisa.

> **NOTA**: Puede usar `token-count.py` para comparar el uso de tokens en ambas respuestas.
<br>
<details>
<summary><b>Ejemplo de una indicación reescrita:</b></summary><br>
<p>"Proporciona un resumen de 5 hitos clave en la historia de la inteligencia artificial usando viñetas".</p>
</details>

## [**OPCIONAL**] Aplicar las optimizaciones en un escenario real

1. Imagine que está creando un bot de chat de atención al cliente que debe proporcionar respuestas rápidas y precisas.
1. Integre la plantilla y la indicación del sistema optimizada en el código del bot de chat (*puede usar`optimize-prompt.py` como punto de partida*).
1. Pruebe el bot de chat con varias consultas de usuario para asegurarse de que responde de forma eficaz.

## Conclusión

La optimización de las indicaciones es una capacidad clave para reducir los costos y mejorar el rendimiento de las aplicaciones de IA generativa. Acortar las indicaciones, usar plantillas y analizar las interacciones de los usuarios permiten crear soluciones más eficaces y escalables.

## Limpieza

Si has terminado de explorar Servicios de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com?azure-portal=true) en una nueva pestaña del explorador) y visualiza el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
