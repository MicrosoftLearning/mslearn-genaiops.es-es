---
lab:
  title: Orquestación de un sistema RAG
  description: Aprende a implementar sistemas de generación aumentada de recuperación (RAG) en las aplicaciones para mejorar la precisión y relevancia de las respuestas generadas.
---

## Orquestación de un sistema RAG

Los sistemas de generación aumentada de recuperación (RAG) combinan la eficacia de los modelos de lenguaje grande con mecanismos de recuperación eficaces para mejorar la precisión y la relevancia de las respuestas generadas. Al aprovechar LangChain para orquestación y Fundición de IA de Azure para funcionalidades de IA, podemos crear una canalización sólida que recupere información relevante de un conjunto de datos y genere respuestas coherentes. En este ejercicio, recorrerás los pasos necesarios para configurar el entorno, procesar previamente datos, crear incrustaciones y crear un índice, lo que te permitirá implementar un sistema RAG de forma eficaz.

Este ejercicio dura aproximadamente **30** minutos.

## Escenario

Imagina que quieres crear una aplicación que proporcione recomendaciones sobre hoteles en Londres. En la aplicación, quieres un agente que no solo pueda recomendar hoteles, sino que responda a las preguntas que los usuarios puedan tener sobre ellos.

Has seleccionado un modelo GPT-4 para proporcionar respuestas generativas. Ahora quieres reunir un sistema RAG que proporcione datos de fundamentación al modelo en función de las reseñas de otros usuarios, lo que guiará el comportamiento del chat para proporcionar recomendaciones personalizadas.

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
    cd ~/mslearn-genaiops/Files/04/
     ```

1. Escribe los siguientes comandos para activar un entorno virtual e instalar las bibliotecas que necesitas:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. Escribe el siguiente comando para abrir el archivo de configuración que se ha proporcionado:

    ```powershell
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza los marcadores de posición **your_azure_openai_service_endpoint** y **your_azure_openai_service_api_key** por los valores de punto de conexión y clave que copiaste anteriormente.
1. *Después* de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o **clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Implementación de RAG

Ahora ejecutarás un script que ingiere y preprocesa datos, crea incrustaciones y crea un índice y un almacén de vectores, lo que le permite implementar un sistema RAG de forma eficaz.

1. Ejecuta el siguiente comando para **ver el script** que se ha proporcionado:

    ```powershell
   code RAG.py
    ```

1. Revisa el script y observa que usa un archivo .csv con reseñas de hoteles como datos de base. Para ver el contenido de este archivo, ejecuta el comando `download app_hotel_reviews.csv` y abre el archivo.
1. **Ejecuta el script** escribiendo el siguiente comando en la línea de comandos:

    ```
   python RAG.py
    ```

1. Una vez que se ejecuta la aplicación, puedes empezar a formular preguntas como `Where can I stay in London?` y, a continuación, realizar un seguimiento con consultas más específicas.

## Conclusión

En este ejercicio has creado un sistema RAG típico con sus componentes principales. Mediante el uso de tus propios documentos para informar a las respuestas de un modelo, proporcionas datos de base utilizados por el LLM cuando formula una respuesta. Para una solución empresarial, esto significa que puedes restringir la inteligencia artificial generativa al contenido empresarial.

## Limpieza

Si has terminado de explorar Servicios de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com?azure-portal=true) en una nueva pestaña del explorador) y visualiza el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
