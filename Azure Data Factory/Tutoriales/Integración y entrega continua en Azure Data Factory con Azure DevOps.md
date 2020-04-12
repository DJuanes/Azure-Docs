## Integración y entrega continua (CI / CD) en Azure Data Factory con Azure DevOps





Con la integración continua de Azure Data Factory (ADF), usted ayuda a su equipo a colaborar y desarrollar soluciones de transformación de datos dentro del mismo espacio de trabajo de data factory y a mantener sus esfuerzos de desarrollo combinados en un repositorio central de código. La entrega continua ayuda a construir e implementar su solución ADF para fines de testing y puesta en producción. Básicamente, el proceso de CI / CD ayuda a establecer una buena práctica de desarrollo de software y tiene como objetivo construir una relación saludable entre el desarrollo, el aseguramiento de la calidad y otros equipos de apoyo.



#### Caso de uso de la canalización ADF para implementar

Tenemos una canalización en Azure Data Factory (en el ambiente de desarrollo) sincronizada en un repositorio Git de Azure DevOps.

Queremos testear la canalización en un entorno separado (testing) y luego implementarla en producción.

​														Desarrollo > Testing > Produccion

En mi ADF tengo una solución de canalización de plantilla "Copiar múltiples contenedores de archivos entre File Stores" que usé para copiar archivos / carpetas de un contenedor de blobs a otro.
En mi grupo de recursos de desarrollo de Azure, creé tres artefactos (que pueden variar según sus propias necesidades de desarrollo):

- Key Vault, para almacenar valores secretos.
- Cuenta de almacenamiento, para mantener los archivos en contenedores de blobs.
- Data Factory, lo que vamos a testear e implementar.



#### Creación de una canalización de integración y entrega continua (CI / CD)

Paso 1: Integrar el código de la canalización ADF al control de código (Azure Git)
Para la integración continua (CI) de su canalización de ADF, deberá asegurarse de que, junto con su rama de desarrollo (o feature), tendrá que publicar su rama master desde su IU de Azure Data Factory. Esto creará una rama adf_publish adicional de su repositorio de código en GitHub.
																dev > master > adf_publish
La publicación del código ADF desde la rama master a la rama adf_publish creó automáticamente dos archivos:
**Archivo de plantilla** (ARMTemplateForFactory.json): plantillas que contienen todos los metadatos de la data factory (canalizaciones, conjuntos de datos, etc.).
**Archivo de configuración** (ARMTemplateParametersForFactory.json): contiene parámetros que serán diferentes para cada entorno (desarrollo, testing, producción, etc.).

Paso 2: crear una canalización de implementación en Azure DevOps
2.a Acceda a Azure DevOps: si no tiene una cuenta DevOps, puede comenzar de forma gratuita: https://azure.microsoft.com/es-es/services/devops/ .

2.b Cree un nuevo proyecto DevOps: al hacer clic en el botón [+ Crear proyecto] comenzará a crear su solución de compilación / puesta en producción.
2.c Crear una nueva canalización release
En un ciclo normal de desarrollo de software, crearía una canalización de compilación en mi proyecto Azure DevOps, luego archivaría los archivos compilados y los usaría para futuras operaciones de despliegue. Como el equipo de Microsoft indicó en su página de documentación de CI / CD para Azure Data Factory, la canalización de despliegue se basa en el repositorio de control de origen, por lo que se omite la canalización de compilación; sin embargo, es una cuestión de elección y siempre puede cambiarlo en su solución.
Seleccione [Pipelines] > [Release] y luego haga clic en el botón [New pipeline] para crear una nueva canalización de PAT / PAP para su solución ADF.
2.d Crear grupos de variables para su solución
Este paso es opcional, ya que puede agregar variables directamente a sus canalizaciones de PAT / PAP. Pero en aras de reutilizar algunos de los valores genéricos, creé tres grupos en mi proyecto DevOps con la misma variable (Entorno) para cada uno de estos grupos con los valores ("dev", "tst", "prd").
2.e Agregar artefacto
En la sección Artefacto, configuro los siguientes atributos:
-Servicio de conexión de mi cuenta de GitHub
-Repositorio de GitHub para mi Azure Data Factory
-Rama predeterminada (adf_branch) como fuente para el procesamiento de mi versión
-Alias (Desarrollo) ya que es el comienzo de todo
2.f Agregar nueva etapa
Hago clic en el botón [+ Add a stage], selecciono la plantilla "Empty Job" y lo llamo "Testing". Luego agrego dos agent jobs de la lista de tareas:
-Azure Key Vault, para leer valores secretos de Azure y pasarlos para mis parámetros de plantilla ADF ARM
-Azure Resource Group Deployment, para implementar mi plantilla ADF ARM
Para la tarea (Azure Key Vault), configuré el nombre de "Key vault" en este valor: azu-caes - $ (Environment) -keyvault, que admitirá los Key Vault de los tres entornos según el valor del grupo de variable $(Environment) {dev, tst, prd}.
Para la tarea (Azure Resource Group Deployment), configuro los siguientes atributos:
-Acción: crear o actualizar un grupo de recursos
-Grupo de recursos: azu-caes - $ (Medio ambiente) -rg
-Ubicación de la plantilla: artefacto vinculado
-Plantilla: en mi caso, la ubicación del archivo de plantilla ART vinculada es
$(System.DefaultWorkingDirectory)/Development/azu-eus2-dev-datafactory/ARMTemplateForFactory.json
-Parámetros de plantilla: en mi caso, la ubicación del archivo de parámetros de la plantilla ARM vinculada es
$(System.DefaultWorkingDirectory)/Development/azu-eus2-dev-datafactory/ARMTemplateParametersForFactory.json
-Anular parámetros de plantilla: algunos de mis parámetros de canalización de ADF los dejo en blanco o con el valor predeterminado, sin embargo, configuré específicamente para anular los siguientes parámetros:

   + factoryName: $(factoryName)
   + blob_storage_ls_connectionString: $(blob-storage-ls-connectionString)
   + AzureKeyVault_ls_properties_typeProperties_baseUrl: $(AzureKeyVault_ls_properties_typeProperties_baseUrl)
     -Modo de implementación: incremental
     2.g Agregar variables de canalización
     Para admitir la referencia de variables en mi tarea de implementación de plantilla ARM, agrego las siguientes variables dentro del alcance "Release":
     -factoryName
     -AzureKeyVault_ls_properties_typeProperties_baseUrl
     2.h Clonar etapa testing a una nueva etapa de producción:
     Al hacer clic en el botón "Clonar" en la etapa "Prueba", creo una nueva etapa de despliegue y la llamo "Producción". Mantendré ambas tareas dentro de esta nueva etapa sin cambios, donde la variable $(Entorno) debería hacer todo el truco de cambiar entre dos etapas.
     2.i Enlace de grupos de variables a etapas:
     Un último paso para crear mi canal de despliegue de Azure Data Factory es que necesito vincular mis grupos de Variables a las etapas de despliegue correspondientes. Voy a la sección Variables en mi canalización y selecciono Grupos variables, haciendo clic en el botón [Link variable group] elijo vincular un grupo variable a una etapa para que mis secciones de Prueba y Producción se vean de la siguiente manera:
     Después de guardar mi canalización de despliegue de ADF, mantengo mis dedos cruzados :-) ¡Es hora de probarlo!
     Paso 3: crear y probar una nueva versión
     Hago clic en el botón [+ Create a release] y comienzo a monitorear mi proceso de versión de ADF.
     Después de que se terminó con éxito, fui a verificar mi Data Factory de producción (azu-eus2-prd-datafactory) y pude encontrar mi canalización implementada que se originó en el entorno de desarrollo. Y probé con éxito la canalización de producción ejecutándola.
     Una cosa a tener en cuenta es que todas las conexiones de datos a la cuenta Key Vault y Blob Storage de producción se han implementado correctamente, aunque a través de la canalización de ADF (sin intervenciones manuales).
     Paso 4: Probar el proceso de CI / CD para mi Azure Data Factory
     Ahora es el momento de probar un ciclo completo de commiteo de cambio de código de data factory e implementarlo tanto en entornos de prueba como de producción.
     1) Agrego una nueva tarea de actividad "Esperar 5 segundos" a mi canalización de ADF y la guardo en mi rama de desarrollo de la instancia de desarrollo de data factory (azu-eus2-dev-datafactory).
     2) Creo un pull request y mergeo este cambio en la rama master de mi repositorio de GitHub
     3) Luego publico mi nuevo cambio de código ADF de la rama master a la rama adf_publish.
     4) Y esto activa automáticamente mi canalización de despliegue de ADF para implementar este cambio de código en los ambientes de testing y luego en producción.
     No olvide habilitar "Continues deployment trigger" en su artefacto de canalización de despliegue y seleccione la rama de código final 'adf-Publish' como filtro. De lo contrario, todos los commit de todas las ramas podría desencadenar el proceso de despliegue / implementación.
     5) Y después de que la nueva versión se haya completado con éxito, puedo verificar que la nueva tarea de actividad "Wait" aparezca en la canalización de la data factory de Producción:

**Resumen:**
1) Me siento un poco agotado después de escribir una publicación de blog tan larga. Espero que lo encuentre útil en sus propias canalizaciones de DevOps para sus soluciones de Azure Data Factory.
2) Después de ver un ciclo completo de CI / CD de mi Azure Data Factory donde se cambió el código, se integró en el repositorio principal de código fuente y se implementó automáticamente, ¡puedo decir con confianza que mi fábrica de datos se ha lanzado con éxito! :-)













Integración y entrega continua (CI / CD) en Azure Data Factory con Azure DevOps

Con la integración continua de Azure Data Factory (ADF), usted ayuda a su equipo a colaborar y desarrollar soluciones de transformación de datos dentro del mismo espacio de trabajo de data factory y a mantener sus esfuerzos combinados de desarrollo en un repositorio central de código. La entrega continua ayuda a construir e implementar su solución ADF para fines de prueba y puesta en producción. Básicamente, el proceso de CI / CD ayuda a establecer una buena práctica de desarrollo de software y tiene como objetivo construir una relación saludable entre el desarrollo, el aseguramiento de la calidad y otros equipos de apoyo.

**Mi caso de uso de la canalización ADF para implementar**
Entonces, aquí está mi caso de uso: tengo una canalización en mi Azure Data Factory sincronizada en GitHub (mi espacio de trabajo de Desarrollo) y quiero poder probarlo en un entorno separado y luego implementarlo en un entorno de Producción.
													Development > Testing > Production

En mi ADF tengo una solución de canalización de plantilla "Copiar múltiples contenedores de archivos entre File Stores" que usé para copiar archivos / carpetas de un contenedor de blobs a otro.
En mi grupo de recursos de desarrollo de Azure, creé tres artefactos (que pueden variar según sus propias necesidades de desarrollo):
-Key Vault, para almacenar valores secretos.
-Cuenta de almacenamiento, para mantener mis archivos de prueba en contenedores de blobs
-Data Factory, lo que voy a testear e implementar

**Creando una canalización de integración y entrega continua (CI / CD) para mi ADF**
Paso 1: Integrar el código de la canalización ADF al control de código (GitHub)
Para la integración continua (CI) de su canalización de ADF, deberá asegurarse de que, junto con su rama de desarrollo (o feature), tendrá que publicar su rama master desde su IU de Azure Data Factory. Esto creará una rama adf_publish adicional de su repositorio de código en GitHub.
													development > master > adf_publish
La publicación de mi código ADF desde el maestro a la rama adf_publish creó automáticamente dos archivos:
<b>Archivo de plantilla</b> (ARMTemplateForFactory.json): plantillas que contienen todos los metadatos de la data factory (canalizaciones, conjuntos de datos, etc.).
<b>Archivo de configuración</b> (ARMTemplateParametersForFactory.json): contiene parámetros que serán diferentes para cada entorno (desarrollo, testing, producción, etc.).

Paso 2: crear una canalización de implementación en Azure DevOps
2.a Acceda a DevOps: si no tiene una cuenta DevOps, puede comenzar de forma gratuita: https://azure.microsoft.com/en-ca/services/devops/ o solicite a su equipo de DevOps que le proporcione acceso al entorno de compilación / puesta en producción de su organización.
2.b Cree un nuevo proyecto DevOps: al hacer clic en el botón [+ Crear proyecto] comenzará a crear su solución de compilación / puesta en producción.
2.c Crear una nueva canalización release
En un ciclo normal de desarrollo de software, crearía una canalización de compilación en mi proyecto Azure DevOps, luego archivaría los archivos compilados y los usaría para futuras operaciones de despliegue. Como el equipo de Microsoft indicó en su página de documentación de CI / CD para Azure Data Factory, la canalización de despliegue se basa en el repositorio de control de origen, por lo que se omite la canalización de compilación; sin embargo, es una cuestión de elección y siempre puede cambiarlo en su solución.
Seleccione [Pipelines] > [Release] y luego haga clic en el botón [New pipeline] para crear una nueva canalización de PAT / PAP para su solución ADF.
2.d Crear grupos de variables para su solución
Este paso es opcional, ya que puede agregar variables directamente a sus canalizaciones de PAT / PAP. Pero en aras de reutilizar algunos de los valores genéricos, creé tres grupos en mi proyecto DevOps con la misma variable (Entorno) para cada uno de estos grupos con los valores ("dev", "tst", "prd").
2.e Agregar artefacto
En la sección Artefacto, configuro los siguientes atributos:
-Servicio de conexión de mi cuenta de GitHub
-Repositorio de GitHub para mi Azure Data Factory
-Rama predeterminada (adf_branch) como fuente para el procesamiento de mi versión
-Alias (Desarrollo) ya que es el comienzo de todo
2.f Agregar nueva etapa
Hago clic en el botón [+ Add a stage], selecciono la plantilla "Empty Job" y lo llamo "Testing". Luego agrego dos agent jobs de la lista de tareas:
-Azure Key Vault, para leer valores secretos de Azure y pasarlos para mis parámetros de plantilla ADF ARM
-Azure Resource Group Deployment, para implementar mi plantilla ADF ARM
Para la tarea (Azure Key Vault), configuré el nombre de "Key vault" en este valor: azu-caes - $ (Environment) -keyvault, que admitirá los Key Vault de los tres entornos según el valor del grupo de variable $(Environment) {dev, tst, prd}.
Para la tarea (Azure Resource Group Deployment), configuro los siguientes atributos:
-Acción: crear o actualizar un grupo de recursos
-Grupo de recursos: azu-caes - $ (Medio ambiente) -rg
-Ubicación de la plantilla: artefacto vinculado
-Plantilla: en mi caso, la ubicación del archivo de plantilla ART vinculada es
$(System.DefaultWorkingDirectory)/Development/azu-eus2-dev-datafactory/ARMTemplateForFactory.json
-Parámetros de plantilla: en mi caso, la ubicación del archivo de parámetros de la plantilla ARM vinculada es
$(System.DefaultWorkingDirectory)/Development/azu-eus2-dev-datafactory/ARMTemplateParametersForFactory.json
-Anular parámetros de plantilla: algunos de mis parámetros de canalización de ADF los dejo en blanco o con el valor predeterminado, sin embargo, configuré específicamente para anular los siguientes parámetros:
   + factoryName: $(factoryName)
   + blob_storage_ls_connectionString: $(blob-storage-ls-connectionString)
   + AzureKeyVault_ls_properties_typeProperties_baseUrl: $(AzureKeyVault_ls_properties_typeProperties_baseUrl)
-Modo de implementación: incremental
2.g Agregar variables de canalización
Para admitir la referencia de variables en mi tarea de implementación de plantilla ARM, agrego las siguientes variables dentro del alcance "Release":
-factoryName
-AzureKeyVault_ls_properties_typeProperties_baseUrl
2.h Clonar etapa testing a una nueva etapa de producción:
Al hacer clic en el botón "Clonar" en la etapa "Prueba", creo una nueva etapa de despliegue y la llamo "Producción". Mantendré ambas tareas dentro de esta nueva etapa sin cambios, donde la variable $(Entorno) debería hacer todo el truco de cambiar entre dos etapas.
2.i Enlace de grupos de variables a etapas:
Un último paso para crear mi canal de despliegue de Azure Data Factory es que necesito vincular mis grupos de Variables a las etapas de despliegue correspondientes. Voy a la sección Variables en mi canalización y selecciono Grupos variables, haciendo clic en el botón [Link variable group] elijo vincular un grupo variable a una etapa para que mis secciones de Prueba y Producción se vean de la siguiente manera:
Después de guardar mi canalización de despliegue de ADF, mantengo mis dedos cruzados :-) ¡Es hora de probarlo!
Paso 3: crear y probar una nueva versión
Hago clic en el botón [+ Create a release] y comienzo a monitorear mi proceso de versión de ADF.
Después de que se terminó con éxito, fui a verificar mi Data Factory de producción (azu-eus2-prd-datafactory) y pude encontrar mi canalización implementada que se originó en el entorno de desarrollo. Y probé con éxito la canalización de producción ejecutándola.
Una cosa a tener en cuenta es que todas las conexiones de datos a la cuenta Key Vault y Blob Storage de producción se han implementado correctamente, aunque a través de la canalización de ADF (sin intervenciones manuales).
Paso 4: Probar el proceso de CI / CD para mi Azure Data Factory
Ahora es el momento de probar un ciclo completo de commiteo de cambio de código de data factory e implementarlo tanto en entornos de prueba como de producción.
1) Agrego una nueva tarea de actividad "Esperar 5 segundos" a mi canalización de ADF y la guardo en mi rama de desarrollo de la instancia de desarrollo de data factory (azu-eus2-dev-datafactory).
2) Creo un pull request y mergeo este cambio en la rama master de mi repositorio de GitHub
3) Luego publico mi nuevo cambio de código ADF de la rama master a la rama adf_publish.
4) Y esto activa automáticamente mi canalización de despliegue de ADF para implementar este cambio de código en los ambientes de testing y luego en producción.
No olvide habilitar "Continues deployment trigger" en su artefacto de canalización de despliegue y seleccione la rama de código final 'adf-Publish' como filtro. De lo contrario, todos los commit de todas las ramas podría desencadenar el proceso de despliegue / implementación.
5) Y después de que la nueva versión se haya completado con éxito, puedo verificar que la nueva tarea de actividad "Wait" aparezca en la canalización de la data factory de Producción:

**Resumen:**
1) Me siento un poco agotado después de escribir una publicación de blog tan larga. Espero que lo encuentre útil en sus propias canalizaciones de DevOps para sus soluciones de Azure Data Factory.
2) Después de ver un ciclo completo de CI / CD de mi Azure Data Factory donde se cambió el código, se integró en el repositorio principal de código fuente y se implementó automáticamente, ¡puedo decir con confianza que mi fábrica de datos se ha lanzado con éxito! :-)

