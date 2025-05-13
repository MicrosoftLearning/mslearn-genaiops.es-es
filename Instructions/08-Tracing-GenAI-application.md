---
lab:
  title: Análisis y depuración de la aplicación de IA generativa con seguimiento
---

# Análisis y depuración de la aplicación de IA generativa con seguimiento

Este ejercicio dura aproximadamente **30** minutos.

> **Nota**: en este ejercicio se presupone que tienes algún conocimiento de Fundición de IA de Azure, por lo que algunas instrucciones son intencionadamente menos detalladas para fomentar la exploración más activa y el aprendizaje práctico.

## Introducción

En este ejercicio, ejecutarás un asistente de IA generativa de varios pasos que recomienda excursiones de senderismo y sugiere equipo de excursionismo. Usarás las características de seguimiento del SDK de inferencia de Azure AI para analizar cómo se ejecuta la aplicación e identificar los puntos de decisión clave tomados por el modelo y la lógica circundante.

Interactuarás con un modelo implementado para simular el recorrido real del usuario, realizar un seguimiento de cada fase de la aplicación desde la entrada de usuario a la respuesta del modelo al procesamiento posterior y verás los datos de seguimiento en Fundición de IA de Azure. Esto te ayudará a comprender cómo el seguimiento mejora la observabilidad, simplifica la depuración y apoya la optimización del rendimiento de las aplicaciones de IA generativa.

## Configuración del entorno

Para completar las tareas de este ejercicio, necesitas:

- Un centro de Fundición de IA de Azure,
- Un proyecto de Fundición de IA de Azure,
- Un modelo implementado (como GPT-4o),
- Un recurso de Application Insights conectado.

### Creación de un centro y un proyecto de Fundición de IA de Azure

Para configurar rápidamente un centro y un proyecto, se proporcionan instrucciones sencillas para usar la interfaz de usuario del Portal de la Fundición de IA de Azure.

1. Ve al Portal de la Fundición de IA de Azure: abre [https://ai.azure.com](https://ai.azure.com).
1. Inicie sesión con sus credenciales de Azure.
1. Crear un proyecto:
    1. Ve a **Todos los centros y proyectos**.
    1. Selecciona **+ Nuevo proyecto**.
    1. Escribe un **nombre de proyecto**.
    1. Cuando se te solicite, **crea un nuevo centro**.
    1. Personalización del centro:
        1. Selecciona **Suscripción**, **Grupo de recursos**, **Ubicación**, etc.
        1. Conecta un nuevo recurso de **Servicio de Azure AI** (omite Búsqueda de AI).
    1. Revise y seleccione **Crear**.
1. **Espera unos minutos a que se complete la implementación** (~ 1-2 minutos).

### Implementación de un modelo

Para generar datos que puedas supervisar, primero deberás implementar un modelo e interactuar con él. En las instrucciones se te pide que implementes un modelo GPT-4o, pero **puedes usar cualquier modelo** de la colección de Azure OpenAI Service que esté disponible.

1. En el menú de la izquierda, en la sección **Mis recursos**, selecciona la página **Modelos y puntos de conexión**.
1. Implementa un **modelo base** y elige **gpt-4o**.
1. **Personaliza los detalles de implementación**.
1. Establece la **capacidad** en **5000 tokens por minuto (TPM).**

El centro y el proyecto están listos, todos los recursos de Azure necesarios se han aprovisionado automáticamente.

### Conectar Application Insights

Conecta Application Insights al proyecto en Fundición de IA de Azure para empezar a recopilar datos para su análisis.

1. Abre el proyecto en el Portal de la Fundición de IA de Azure.
1. Usa el menú de la izquierda y selecciona la página **Seguimiento**.
1. **Crea un nuevo** recurso de Application Insights para conectarse a la aplicación.
1. Escribe el **nombre del recurso de Application Insights**.

Application Insights ahora está conectado al proyecto y los datos comenzarán a recopilarse para su análisis.

## Ejecución de una aplicación de IA generativa con Cloud Shell

Te conectarás al proyecto de Fundición de IA de Azure desde Azure Cloud Shell e interactuarás mediante programación con un modelo implementado como parte de una aplicación de IA generativa.

### Interacción con un modelo implementado

Empieza por recuperar la información necesaria para autenticarte para interactuar con el modelo implementado. A continuación, accederás a Azure Cloud Shell y actualizarás el código de la aplicación de IA generativa.

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Detalles del proyecto**, anota la **Cadena de conexión del proyecto**.
1. **Guarda** la cadena en un Bloc de notas. Usarás esta cadena de conexión para conectarte al proyecto en una aplicación cliente.
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

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-genaiops/Files/08
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

    1. Reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure).
    1. Reemplaza el marcador de posición **your_model_deployment** por el nombre que asignaste a la implementación del modelo gpt-4o (de manera predeterminada `gpt-4o`).

1. *Después* de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **haz clic con el botón derecho y luego en Guardar** para **guardar los cambios**.

### Actualización del código de la aplicación de IA generativa

Ahora que tanto el entorno como el archivo .env están configurados, es el momento de preparar el script del asistente de IA para su ejecución. Junto a la conexión con un proyecto de IA y la habilitación de Application Insights, debes:

- Interactuar con el modelo implementado.
- Definir la función para especificar la indicación.
- Definir el flujo principal que llama a todas las funciones.

Agregará estas tres partes a un script inicial.

1. Ejecuta el siguiente comando para **abrir el script** que se ha proporcionado:

    ```
   code start-prompt.py
    ```

    Verás que varias líneas clave se han dejado en blanco o se han marcado con # comentarios vacíos. La tarea consiste en completar el script copiando y pegando las líneas correctas a continuación en las ubicaciones adecuadas.

1. En el script, busca **# Function to call the model and handle tracing**.
1. Pega el siguiente código debajo de este comentario:

    ```
   def call_model(system_prompt, user_prompt, span_name):
        with tracer.start_as_current_span(span_name) as span:
            span.set_attribute("session.id", SESSION_ID)
            span.set_attribute("prompt.user", user_prompt)
            start_time = time.time()
    
            response = chat_client.complete(
                model=model_name,
                messages=[SystemMessage(system_prompt), UserMessage(user_prompt)]
            )
    
            duration = time.time() - start_time
            output = response.choices[0].message.content
            span.set_attribute("response.time", duration)
            span.set_attribute("response.tokens", len(output.split()))
            return output
    ```

1. En el script, busca **# Function to recommend a hike based on user preferences**.
1. Pega el siguiente código debajo de este comentario:

    ```
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. En el script, busca **# ---- Main Flow ----**.
1. Pega el siguiente código debajo de este comentario:

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. **Guarda los cambios** realizados en el script.
1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para **ejecutar el script**:

    ```
   python start-prompt.py
    ```

1. Proporciona una descripción del tipo de caminata que estás buscando, por ejemplo:

    ```
   A one-day hike in the mountains
    ```

    El modelo generará una respuesta, que se capturará con Application Insights. Puedes visualizar los seguimientos en el **Portal de la Fundición de IA de Azure**.

> **Nota**: los datos de supervisión pueden tardar unos minutos en mostrarse en Azure Monitor.

## Visualización de los datos de seguimiento en el Portal de la Fundición de IA de Azure

Después de ejecutar el script, capturaste un seguimiento de la ejecución de la aplicación de IA. Ahora lo explorarás con Application Insights en Fundición de IA de Azure.

> **Nota:** más adelante, volverás a ejecutar el código y volverás a ver los seguimientos en el Portal de la Fundición de IA de Azure. Vamos a explorar primero dónde encontrar los seguimientos para visualizarlos.

### Ve al Portal de la Fundición de IA de Azure

1. **¡No cierres Cloud Shell!** Volverás a esto para actualizar el código y volverlo a ejecutar.
1. Ve a la pestaña del explorador con el **Portal de la Fundición de IA de Azure** abierto.
1. Usa el menú de la izquierda y selecciona **Seguimiento**.
1. *Si* no se muestra ningún dato, **actualiza** la vista.
1. Selecciona el seguimiento **train_guide_session** para abrir una nueva ventana que muestre más detalles.

### Revisión del seguimiento

En esta vista se muestra el seguimiento de una sesión completa del asistente de IA de la guía de senderos.

- **Intervalo de nivel superior**: trail_guide_session Este es el intervalo primario. Representa la ejecución completa del asistente de principio a fin.

- **Intervalos secundarios anidados**: cada línea con sangría representa una operación anidada. Encontrarás lo siguiente:

    - **recommend_hike** que captura la lógica para decidir una caminata.
    - **recommend_model_call** que es el intervalo creado por call_model() dentro de recommend_hike.
    - **chat gpt-4o** que el SDK de inferencia de Azure AI instrumenta automáticamente para mostrar la interacción real de LLM.

1. Puedes hacer clic en cualquier intervalo para ver:

    1. La duración.
    1. Sus atributos, como la indicación del usuario, los tokens usados, el tiempo de respuesta.
    1. Cualquier error o datos personalizados adjuntos con **span.set_attribute(...)**.

## Adición de más funciones al código


1. Escribe el siguiente comando para **volver a abrir el script**:

    ```
   code start-prompt.py
    ```

1. En el script, busca **# Function to generate a trip profile for the recommended hike**.
1. Pega el siguiente código debajo de este comentario:

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. En el script, busca **# Function to match recommended gear with products in the catalog**.
1. Pega el siguiente código debajo de este comentario:

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. En el script, busca **# Run profile function**.
1. Debajo y **alineado con** este comentario, pega el código siguiente:

    ```
           profile = generate_trip_profile(hike)
           if not profile:
           print("Failed to generate trip profile. Please check Application Insights for trace.")
           exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. En el script, busca **# Run match product function**.
1. Debajo y **alineado con** este comentario, pega el código siguiente:

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. **Guarda los cambios** realizados en el script.
1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para **ejecutar el script**:

    ```
   python start-prompt.py
    ```

1. Proporciona una descripción del tipo de caminata que estás buscando, por ejemplo:

    ```
   I want to go for a multi-day adventure along the beach
    ```

> **Nota**: los datos de supervisión pueden tardar unos minutos en mostrarse en Azure Monitor.

### Visualización de los nuevos seguimientos en el Portal de la Fundición de IA de Azure

1. Vuelve al Portal de la Fundición de IA de Azure.
1. Debería aparecer un nuevo seguimiento con el mismo nombre **trail_guide_session**. Actualiza la vista si es necesario.
1. Selecciona el nuevo seguimiento para abrir una vista más detallada.
1. Revisa los nuevos intervalos secundarios anidados **trip_profile_generation** y **product_matching**.
1. Selecciona **product_matching** y revisa los metadatos que aparecen.

    En la función product_matching, incluíste **span.set_attribute("matched.count", len(matched)))**. Al establecer el atributo con el par clave-valor **matched.count** y la longitud de la variable coincidente, agregaste esta información al seguimiento de **product_matching**. Puedes encontrar este par clave-valor en **atributos** en los metadatos.

## (OPCIONAL) Seguimiento de un error

Si tienes tiempo adicional, puedes revisar cómo usar seguimientos cuando tengas un error. Se proporciona un script que probablemente produzca un error. Ejecútalo y revisa los seguimientos.

Se trata de un ejercicio diseñado para representar un desafío, lo que significa que las instrucciones son intencionadamente menos detalladas.

1. En Cloud Shell, abre el script **error-prompt.py**. Este script se encuentra en el mismo directorio que el script **start-prompt.py**. Revisa el contenido.
1. Ejecuta el script **error-prompt.py**. Proporciona una respuesta en la línea de comandos cuando se te solicite.
1. *Esperamos* que el mensaje de salida incluya **No se pudo generar el perfil de viaje. Consulta Application Insights para el seguimiento.**.
1. Vaya al seguimiento de **trip_profile_generation** e inspecciona por qué se produjo un error.

<br>
<details>
<summary><b>Obtener la respuesta sobre</b>: ¿Por qué puedes haber experimentado un error...</summary><br>
<p>Si inspeccionas el seguimiento de LLM para la función generate_trip_profile, observarás que la respuesta del asistente incluye acentos pendientes y la palabra json para dar formato a la salida como un bloque de código.

Aunque esto resulta útil para mostrar, provoca problemas en el código porque la salida ya no es JSON válido. Esto produce un error de análisis durante el procesamiento posterior.

Es probable que el error se deba a cómo se indica a LLM que se ajuste a un formato específico para su salida. La inclusión de las instrucciones en la indicación del usuario aparece más eficaz que colocarla en el aviso del sistema.</p>
</details>

## Dónde encontrar otros laboratorios

Puedes explorar laboratorios y ejercicios adicionales en el [Portal de aprendizaje de la Fundición de IA de Azure](https://ai.azure.com) o consultar la **sección de laboratorio** del curso para ver otras actividades disponibles.
