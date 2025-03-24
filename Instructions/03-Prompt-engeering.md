---
lab:
  title: Exploración de la ingeniería de indicaciones con Prompty
---

## Exploración de la ingeniería de indicaciones con Prompty

Durante la generación de ideas, puede que quieras probar y mejorar rápidamente las distintas indicaciones con el modelo de lenguaje. Hay varias maneras de abordar la ingeniería de indicaciones, a través del área de juegos en el Portal de la Fundición de IA de Azure o mediante Prompty para obtener un enfoque más orientado al código.

En este ejercicio, explorarás la ingeniería de indicaciones con Prompty en Visual Studio Code mediante un modelo implementado a través de Fundición de IA de Azure.

Este ejercicio dura aproximadamente **40** minutos.

## Escenario

Imagina que quieres crear una aplicación para ayudar a los estudiantes a aprender a codificar en Python. En la aplicación, quieres un tutor automatizado que pueda ayudar a los estudiantes a escribir y evaluar código. Sin embargo, no quieres que la aplicación de chat proporcione únicamente todas las respuestas. Quieres que los estudiantes reciban sugerencias personalizadas que les animen a pensar en cómo continuar.

Has seleccionado un modelo GPT-4 con el que empezar a experimentar. Ahora quieres aplicar la ingeniería de indicaciones para guiar el comportamiento del chat para que sea un tutor que genera sugerencias personalizadas.

Empecemos implementando los recursos necesarios para trabajar con este modelo en el Portal de la Fundición de IA de Azure.

## Creación de un centro y un proyecto de Azure AI

> **Nota**: si ya tienes un centro y un proyecto de Azure AI, puedes omitir este procedimiento y usar el proyecto existente.

Puedes crear manualmente un centro y un proyecto de Azure AI a través del Portal de la Fundición de IA de Azure, así como implementar el modelo usado en el ejercicio. Sin embargo, también puedes automatizar este proceso mediante el uso de una aplicación de plantilla con [Azure Developer CLI (azd)](https://aka.ms/azd).

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
   
## Configuración del entorno de desarrollo local

Para experimentar e iterar rápidamente, usarás Prompty en Visual Studio (VS) Code. Vamos a preparar VS Code para su uso para la generación de ideas local.

1. Abre VS Code y **clona** el siguiente repositorio de Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Almacena el clon en una unidad local y abre la carpeta después de la clonación.
1. En el panel de extensiones de VS Code, busca e instala la extensión **Prompty**.
1. En el Explorador de VS Code (panel izquierdo), haz clic con el botón derecho en la carpeta **Files/03**.
1. Selecciona **New Prompty** en el menú desplegable.
1. Abre el archivo recién creado denominado **basic.prompty**.
1. Ejecuta el archivo Prompty seleccionando el botón **play** situado en la esquina superior derecha (o presiona F5).
1. Cuando se te solicite, selecciona **Allow**.
1. Selecciona la cuenta de Azure e inicia sesión.
1. Vuelve a VS Code, donde se abrirá un panel **Output** con un mensaje de error. El mensaje de error debe indicarte que no se ha especificado el modelo implementado o que no se encuentra.

Para corregir el error, debes configurar un modelo para que Prompty lo use.

## Actualización de los metadatos de la indicación

Para ejecutar el archivo Prompty, debes especificar el modelo de lenguaje que se va a usar para generar la respuesta. Los metadatos se definen en *frontmatter* del archivo Prompty. Vamos a actualizar los metadatos con la configuración del modelo y otra información.

1. Abre el panel de terminal de Visual Studio Code.
1. Copia el archivo **basic.prompty** (en la misma carpeta) y cambia el nombre de la copia a `chat-1.prompty`.
1. Abre **chat-1.prompty** y actualiza los campos siguientes para cambiar información básica:

    - **Nombre:**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **Descripción:**

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **Modelo implementado**:

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. A continuación, agrega el siguiente marcador de posición para la clave de API en el parámetro **azure_deployment**.

    - **Clave del punto de conexión**:

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. Guarda el archivo Prompty actualizado.

El archivo Prompty ahora tiene todos los parámetros necesarios, pero algunos parámetros usan marcadores de posición para obtener los valores necesarios. Los marcadores de posición se almacenan en el archivo **.env** de la misma carpeta.

## Actualización de la configuración del modelo

Para especificar qué modelo usa Prompty, debes proporcionar la información del modelo en el archivo .env.

1. Abre el archivo **.env** de la carpeta **Files/03**.
1. Actualiza cada uno de los marcadores de posición con los valores que copiaste anteriormente de la salida de la implementación del modelo en Azure Portal:

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. Guarda el archivo .env.
1. Vuelve a ejecutar el archivo **chat-1.prompty**.

Ahora deberías obtener una respuesta generada por IA, aunque no relacionada con tu escenario, ya que solo usaste la entrada de ejemplo. Vamos a actualizar la plantilla para que sea un profesor ayudante de IA.

## Edición de la sección de ejemplo

En la sección de ejemplo se especifican las entradas de Prompty y se proporcionan valores predeterminados que se usarán si no se proporcionan entradas.

1. Edita los campos de los parámetros siguientes:

    - **firstName**: elige cualquier otro nombre.
    - **context**: quita esta sección completa.
    - **question**: reemplaza el texto proporcionado por:

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    La sección de **ejemplo** debería tener este aspecto:
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. Ejecuta el archivo Prompty actualizado y revisa la salida.

