---
lab:
  title: Optimización del modelo mediante un conjunto de datos sintético
---

## Optimización del modelo mediante un conjunto de datos sintético

La optimización de una aplicación de IA generativa implica aprovechar los conjuntos de datos para mejorar el rendimiento y la confiabilidad del modelo. Mediante el uso de datos sintéticos, los desarrolladores pueden simular una amplia gama de escenarios y casos perimetrales que podrían no estar presentes en datos reales. Además, la evaluación de las salidas del modelo es fundamental para obtener aplicaciones de IA confiables y de alta calidad. Todo el proceso de optimización y evaluación se puede administrar de forma eficaz mediante el SDK de evaluación de Azure AI, que proporciona herramientas y marcos sólidos para simplificar estas tareas.

## Escenario

Imagina que quieres crear una aplicación de guía inteligente con tecnología de IA para mejorar las experiencias de los visitantes en un museo. La aplicación tiene como objetivo responder a preguntas sobre figuras históricas. Para evaluar las respuestas de la aplicación, debes crear un conjunto de datos sintético completo de preguntas y respuestas que abarque varios aspectos de estas personalidades y su trabajo.

Has seleccionado un modelo GPT-4 para proporcionar respuestas generativas. Ahora quieres reunir un simulador que genere interacciones contextualmente relevantes, evaluando el rendimiento de la IA en diferentes escenarios.

Vamos a empezar implementando los recursos necesarios para crear esta aplicación.

## Creación de un centro y un proyecto de Azure AI

Puedes crear manualmente un centro y un proyecto de Azure AI a través del Portal de la Fundición de IA de Azure, así como implementar los modelos usados en el ejercicio. Sin embargo, también puedes automatizar este proceso mediante el uso de una aplicación de plantilla con [Azure Developer CLI (azd)](https://aka.ms/azd).

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copia estos valores, ya que se usarán más adelante.

## Configuración del entorno de desarrollo local

Para experimentar e iterar rápidamente, usarás un cuaderno con código de Python en Visual Studio (VS) Code. Vamos a preparar VS Code para su uso para la generación de ideas local.

1. Abre VS Code y **clona** el siguiente repositorio de Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Almacena el clon en una unidad local y abre la carpeta después de la clonación.
1. En el Explorador de VS Code (panel izquierdo), abra el cuaderno **06-Optimize-your-model.ipynb** en la carpeta **Files/06**.
1. Ejecuta todas las celdas del cuaderno.

## Limpieza

Si has terminado de explorar Servicios de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com?azure-portal=true) en una nueva pestaña del explorador) y visualiza el contenido del grupo de recursos donde implementaste los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
