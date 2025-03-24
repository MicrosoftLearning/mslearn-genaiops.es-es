---
lab:
  title: Comparación de modelos de lenguaje desde el catálogo de modelo
---

## Comparación de modelos de lenguaje desde el catálogo de modelo

Cuando haya definido el caso de uso, puedes usar el catálogo de modelos para explorar si un modelo de IA resuelve el problema. Puedes usar el catálogo de modelos para seleccionar los modelos que se van a implementar, que puedes comparar para explorar qué modelo se adapta mejor tus necesidades.

En este ejercicio, compararás dos modelos de lenguaje mediante el catálogo de modelos en el Portal de la Fundición de IA de Azure.

Este ejercicio dura aproximadamente **25** minutos.

## Escenario

Imagina que quieres crear una aplicación para ayudar a los estudiantes a aprender a codificar en Python. En la aplicación, quieres un tutor automatizado que pueda ayudar a los estudiantes a escribir y evaluar código. En un ejercicio, los estudiantes deben crear el código de Python necesario para trazar un gráfico circular, en función de la siguiente imagen de ejemplo:

![Gráfico circular que muestra las puntuaciones obtenidas en un examen con secciones para matemáticas (34,9 %), física (28,6 %), química (20,6 %) e inglés (15,9 %)](./images/demo.png)

Debes seleccionar un modelo de lenguaje que acepte imágenes como entrada y pueda generar código preciso. Los modelos disponibles que cumplen esos criterios son GPT-4 Turbo, GPT-4o y GPT-4o mini.

Empecemos por implementar los recursos necesarios para trabajar con estos modelos en el Portal de la Fundición de IA de Azure.

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

## Comparación de modelos

Sabes que hay tres modelos que aceptan imágenes como entrada y cuya infraestructura de inferencia está totalmente administrada por Azure. Ahora, los debes comparar para decidir cuál es el ideal para nuestro caso de uso.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure.
1. Si se te solicita, selecciona el proyecto de IA que creaste anteriormente.
1. Ve a la página **Catálogo de modelos** con el menú de la izquierda.
1. Selecciona **Comparar modelos** (busca el botón situado junto a los filtros en el panel de búsqueda).
1. Quita los modelos seleccionados.
1. Uno por uno, agrega los tres modelos que deseas comparar: **gpt-4**, **gpt-4o** y **gpt-4o-mini**. Para **gpt-4**, asegúrate de que la versión seleccionada sea **turbo-2024-04-09**, ya que es la única versión que acepta imágenes como entrada.
1. Cambia el eje X a **Precisión**.
1. Asegúrate de que el eje Y esté establecido en **Coste**.

Revisa el trazado e intenta responder a las siguientes preguntas:

- *¿Qué modelo es más preciso?*
- *¿Qué modelo es más barato de usar?*

La precisión de las métricas de referencia se calcula en función de los conjuntos de datos genéricos disponibles públicamente. En el trazado ya podemos descartar uno de los modelos, ya que tiene el coste más elevado por token, pero no la precisión más alta. Antes de tomar una decisión, vamos a explorar la calidad de las salidas de los dos modelos restantes específicos del caso de uso.

## Configuración del entorno de desarrollo local

Para experimentar e iterar rápidamente, usarás un cuaderno con código de Python en Visual Studio (VS) Code. Vamos a preparar VS Code para su uso para la generación de ideas local.

1. Abre VS Code y **clona** el siguiente repositorio de Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Almacena el clon en una unidad local y abre la carpeta después de la clonación.
1. En el Explorador de VS Code (panel izquierdo), abre el cuaderno **02-Compare-models.ipynb** en la carpeta **Files/02**.
1. Ejecuta todas las celdas del cuaderno.

## Limpieza

Si has terminado de explorar Servicios de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com?azure-portal=true) en una nueva pestaña del explorador) y visualiza el contenido del grupo de recursos donde implementaste los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
