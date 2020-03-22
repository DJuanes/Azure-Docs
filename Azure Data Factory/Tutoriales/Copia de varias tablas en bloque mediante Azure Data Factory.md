

## Copia de varias tablas en bloque mediante Azure Data Factory





A grandes rasgos, este tutorial incluye los pasos siguientes:

- Creación de una factoría de datos.
- Creación de los servicios vinculados de Azure SQL Database, Azure Synapse Analytics y Azure Storage.
- Creación de los conjuntos de datos de Azure SQL Database y Azure Synapse Analytics.
- Creación de una canalización para buscar las tablas que se deben copiar y otra canalización para realizar la operación de copia real.
- Inicio de la ejecución de una canalización.
- Supervisión de las ejecuciones de canalización y actividad.



#### Prerrequisitos

Consulte la sección [Prerrequisitos](Prerrequisitos.md).



#### Creación de la factoría de datos

Para crear una factoría de datos consulte [Creación de una factoría de datos](Creaci%C3%B3n%20de%20una%20factor%C3%ADa%20de%20datos.md).



#### Creación de servicios vinculados

En este paso, creará un servicio vinculado para vincular su base de datos de Azure SQL con la factoría de datos:

------

1. Seleccione **Connections** (Conexiones) en la parte inferior de la ventana y haga clic en **+ New** (+ Nuevo) en la barra de herramientas.

2. En la ventana **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure SQL Database** y haga clic en **Continue** (Continuar).

3. En la ventana **New Linked Service (Azure SQL Database)** [Nuevo servicio vinculado (Azure SQL Database)], realice los siguientes pasos:
   - Escriba **AzureSqlDatabaseLinkedService** en **Name** (Nombre).
   - Seleccione el servidor de Azure SQL Server en **Server name** (Nombre del servidor).
   - Seleccione su base de datos de Azure SQL en **Database name** (Nombre de la base de datos).
   - Escriba el **name of the user** (nombre del usuario) para conectarse a Azure SQL Database.
   - Escriba la **password** (contraseña) del usuario.
   - Para probar la conexión a Azure SQL Database con la información indicada, haga clic en **Test connection** (Prueba de conexión).
   - Haga clic en **Create** (Crear) para guardar el servicio vinculado.

------



En este paso, creará un servicio vinculado para vincular Azure Synapse Analytics con la factoría de datos:

------

1. En la pestaña **Connections** (Conexiones), haga clic en **+ New** (+ Nuevo) en la barra de herramientas de nuevo.

2. En la ventana **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure Synapse Analytics** y haga clic en **Continue** (Continuar).

3. En la ventana **New Linked Service (Azure Synapse Analytics (formerly SQL DW))** (Nuevo servicio vinculado [Azure Synapse Analytics (Anteriormente SQL DW)]), siga estos pasos:
   - Escriba **AzureSqlDWLinkedService** en **Name** (Nombre).
   - Seleccione el servidor de Azure SQL Server en **Server name** (Nombre del servidor).
   - Seleccione su base de datos de Azure SQL en **Database name** (Nombre de la base de datos).
   - En **User name** (nombre del usuario), escriba un nombre de usuario para conectarse a Azure SQL Database.
   - En **Password** (contraseña), escriba la contraseña del usuario.
   - Para probar la conexión a Azure SQL Database con la información indicada, haga clic en **Test connection** (Prueba de conexión).
   - Haga clic en **Create** (Crear).

------



En este paso, creará un servicio vinculado para vincular Azure Blob Storage con la factoría de datos:

------

1. En la pestaña **Connections** (Conexiones), haga clic en **+ New** (+ Nuevo) en la barra de herramientas de nuevo.

2. En la ventana **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure Blob Storage** y haga clic en **Continue** (Continuar).

3. En la ventana **New Linked Service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], realice los siguientes pasos:
   - Escriba **AzureStorageLinkedService** en **Name** (Nombre).
   - Seleccione su cuenta para **Storage account name** (Nombre de la cuenta de almacenamiento).
   - Haga clic en **Create** (Crear).

------



#### Creación de conjuntos de datos

En este procedimiento creará los conjuntos de datos de origen y recepción.

El conjunto de datos de entrada **AzureSqlDatabaseDataset** hace referencia a **AzureSqlDatabaseLinkedService**. El servicio vinculado especifica la cadena de conexión para conectarse a la base de datos. El conjunto de datos especifica el nombre de la base de datos y la tabla que contienen los datos de origen.

El conjunto de datos de salida **AzureSqlDWDataset** hace referencia a **AzureSqlDWLinkedService**. El servicio vinculado especifica la cadena de conexión para conectarse a Azure Synapse Analytics. El conjunto de datos especifica la base de datos y la tabla donde se copian los datos.

En este tutorial, las tablas de origen y destino SQL no están codificadas en las definiciones de los conjuntos de datos. En su lugar, la actividad ForEach (Para cada uno) pasa el nombre de la tabla en tiempo de ejecución a la actividad de copia.



En este paso, creará un conjunto de datos de origen de la instancia de SQL Database:

------

1. Haga clic en **+ (plus)** [+ (más)] en el panel izquierdo y en **Dataset** (Conjunto de datos).

2. En la ventana **New Dataset** (Nuevo conjunto de datos), seleccione **Azure SQL Database** y luego seleccione **Continue** (Continuar).

3. En la ventana **Set properties** (Establecer propiedades), en **Name** (Nombre), escriba **AzureSqlDatabaseDataset**. En **Linked service** (Servicio vinculado), seleccione **AzureSqlDatabaseLinkedService**. A continuación, haga clic en **OK** (Aceptar).

4. Cambie a la pestaña **Connection** (Conexión) y seleccione cualquier tabla para **Table** (Tabla). Esta tabla es ficticia. Especifique una consulta en el conjunto de datos de origen al crear una canalización. Como alternativa, puede hacer clic en la casilla **Edit** (Editar) y escribir **dbo.dummyName** como nombre de la tabla.

------



En este paso, creará un conjunto de datos de destino para Azure Synapse Analytics:

------

1. Haga clic en **+ (plus)** [+ (más)] en el panel izquierdo y en **Dataset** (Conjunto de datos).

2. En la ventana **New Dataset** (Nuevo conjunto de datos), seleccione **Azure Synapse Analytics (formerly SQL DW)** (Azure Synapse Analytics [anteriormente SQL DW]) y haga clic en **Continue** (Continuar).

3. En la ventana **Set properties** (Establecer propiedades), en **Name** (Nombre), escriba **AzureSqlDWDataset**. En **Linked service** (Servicio vinculado), seleccione **AzureSqlDWLinkedService**. A continuación, haga clic en **OK** (Aceptar).

4. Vaya a la pestaña **Parameters** (Parámetros), haga clic en **+ New** (+ Nuevo) y escriba **DWTableName** como nombre del parámetro.

5. Cambie a la pestaña **Connection** (Conexión),
   - En **Table** (Tabla), active la opción **Edit** (Editar). Escriba **dbo** en el cuadro de entrada del nombre de la primera tabla. A continuación, seleccione el segundo cuadro de entrada y haga clic en el vínculo **Add dynamic content** (Agregar contenido dinámico) debajo.
   - En la página **Add Dynamic Content** (Agregar contenido dinámico), haga clic en **DWTAbleName** en **Parameters** (Parámetros) y se rellenará automáticamente el cuadro de texto de expresiones de la parte superior `@dataset().DWTableName`, finalmente, haga clic en **Finish** (Finalizar). La propiedad **tableName** se establece en el valor que se pasa como argumento del parámetro **DWTableName**. La actividad ForEach recorre en iteración una lista de tablas y las pasa una a una a la actividad de copia.

------



#### Creación de canalizaciones

En este procedimiento, creará dos canalizaciones:

- **GetTableListAndTriggerCopyData**: busca la tabla del sistema de Azure SQL Database para obtener la lista de tablas que se copiará y desencadena la canalización **IterateAndCopySQLTables** para realizar la copia de datos real.
- **IterateAndCopySQLTables**: toma una lista de tablas como parámetro. Para cada tabla de la lista, copia datos de la tabla de Azure SQL Database a Azure Synapse Analytics mediante la copia almacenada provisionalmente y PolyBase.



En este paso, creará la canalización **IterateAndCopySQLTables**:

------

1. En el panel izquierdo, haga clic en **+ (plus)** [+ (más)] y en **Pipeline** (Canalización).

2. En la pestaña **General**, especifique **IterateAndCopySQLTables** como nombre.

3. Cambie a la pestaña **Parameters** (Parámetros) y realice las siguientes acciones:

   - Haga clic en **+ New** (+ Nuevo).
   - Escriba **tableList** en el parámetro **Name** (Nombre).
   - Seleccione **Array** para **Type** (tipo).

4. En el cuadro de herramientas **Activities** (Actividades), expanda **Iteration & Conditions** (Iteraciones y condiciones), arrastre la actividad **ForEach** y colóquela en la superficie del diseñador.

   - En la pestaña **General** de la parte inferior escriba **IterateSQLTables** para **Name** (Nombre).
   - Cambie a la pestaña **Settings** (Configuración), haga clic en el cuadro de entrada **Items** (Elementos) y, finalmente, haga clic en el vínculo **Add dynamic content** (Agregar contenido dinámico) que aparece a continuación.
   - En la página **Add dynamic content** (Agregar contenido dinámico), haga clic en la **tableList** bajo **Parameters** (Parámetros), que rellenará automáticamente el cuadro de texto de expresiones de la parte superior como `@pipeline().parameter.tableList`. Haga clic en **Finish** (Finalizar).
   - Cambie a la pestaña **Activities** (Actividades), haga clic en **pencil icon** (icono de lápiz) para agregar una actividad secundaria a la actividad **ForEach**.

5. En el cuadro de herramientas **Activities** (Actividades), expanda **Move & Transfer** (Mover y transferir), arrastre la actividad **Copy data** (Copiar datos) y colóquela en la superficie del diseñador. **IterateAndCopySQLTable** es el nombre de la canalización y **IterateSQLTables** es el nombre de la actividad ForEach. El diseñador está en el ámbito de la actividad. Para volver al editor de canalización desde el editor de la actividad ForEach, puede hacer clic en el vínculo del menú de la ruta de navegación.

6. Cambie a la pestaña **Source** (Origen) y realice los pasos siguientes:

   - Seleccione **AzureSqlDatabaseDataset** en **Source Dataset** (Conjunto de datos de origen).

   - Seleccione la opción **Query** (Consulta) en **Use query** (Usar consulta).

   - Haga clic en el cuadro de entrada **Query** (Consulta) -> seleccione el vínculo **Add dynamic content** (Agregar contenido dinámico) que aparece a continuación -> escriba la siguiente expresión para **Query** (Consulta) -> seleccione **Finish** (Finalizar).

     ```sql
     SELECT * FROM [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]
     ```

7. Cambie a la pestaña **Sink** (Receptor) y realice los pasos siguientes:

   - Seleccione **AzureSqlDWDataset** en **Sink Dataset** (Conjunto de datos receptor).

   - Haga clic en el cuadro de entrada VALUE del parámetro DWTableName -> seleccione **Add dynamic content** (Agregar contenido dinámico) que aparece a continuación, escriba la expresión `[@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]` como script, -> seleccione **Finish** (Finalizar).

   - Como método de copia, seleccione **PolyBase**.

   - Desactive la opción **Use type default** (Usar tipo predeterminado).

   - Haga clic en el cuadro de entrada **Pre-copy Script** (Script previo a la copia) -> seleccione el vínculo **Add dynamic content** (Agregar contenido dinámico) que aparece a continuación -> escriba la siguiente expresión como script -> seleccione **Finish** (Finalizar).

     ```sql
     TRUNCATE TABLE [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]
     ```

8. Cambie a la pestaña **Settings** (Configuración) y realice los pasos siguientes:

   - Active la casilla de **Enable Staging** (Habilitar almacenamiento provisional).
   - Seleccione **AzureStorageLinkedService** como **Store Account Linked Service** (Servicio vinculado a la cuenta de almacenamiento).

9. Para comprobar la configuración de la canalización, haga clic en **Validate** (Validar) en la barra de herramientas. Para cerrar **Pipeline Validation Report** (Informe de comprobación de la canalización), haga clic en **>>** .

------



En este paso, creará la canalización **GetTableListAndTriggerCopyData**:

------

1. En el panel izquierdo, haga clic en **+ (plus)** [+ (más)] y en **Pipeline** (Canalización).

2. En la pestaña **General**, cambie el nombre de la canalización a **GetTableListAndTriggerCopyData**.

3. En el cuadro de herramientas **Activities** (Actividades), expanda **General**, arrastre la actividad **Lookup** (Búsqueda) a la superficie del diseñador y realice los pasos siguientes:

   - Escriba **LookupTableList** en **Name** (Nombre).
   - Escriba **Retrieve the table list from Azure SQL Database** (Recuperar la lista de tablas de Azure SQL Database) en **Description** (Descripción).

4. Cambie a la pestaña **Settings** (Configuración) y realice los pasos siguientes:

   - Seleccione **AzureSqlDatabaseDataset** en **Source Dataset** (Conjunto de datos de origen).

   - Seleccione **Query** (Consulta) en **Use query** (Usar consulta).

   - Escriba la siguiente consulta SQL en el campo **Query** (Consulta).

     ```sql
     SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE = 'BASE TABLE' and TABLE_SCHEMA = 'SalesLT' and TABLE_NAME <> 'ProductModel'
     ```

   - Desactive la casilla del campo **First row only** (Solo la primera fila).

5. Arrastre la actividad **Execute Pipeline** (Ejecutar canalización) del cuadro de herramientas de actividades y colóquela en la superficie del diseñador; después, establezca el nombre en **TriggerCopy**.

6. Para **Connect** (conectar) la actividad **Lookup** (Búsqueda) a la actividad **Execute Pipeline** (Ejecutar canalización), arrastre el **cuadro verde** vinculado a la actividad Lookup a la izquierda de la actividad Execute Pipeline.

7. Cambie a la pestaña **Settings** (Configuración) de la actividad **Execute Pipeline** (Ejecutar canalización) y realice los pasos siguientes:

   - Seleccione **IterateAndCopySQLTables** en **Invoked pipeline** (Canalización invocada).
   - Expanda la sección **Advanced** (Avanzado) y desactive la casilla **Wait on completion** (Esperar a que se complete).
   - Haga clic en **+ New** (+ Nuevo) en la sección **Parameters** (Parámetros).
   - Escriba **tableList** en el parámetro **Name** (Nombre).
   - Haga clic en el cuadro de entrada VALOR -> seleccione el vínculo **Add dynamic content** (Agregar contenido dinámico) que aparece a continuación -> escriba `@activity('LookupTableList').output.value` como valor del nombre de tabla -> seleccione **Finish** (Finalizar). Estamos configurando la lista de resultados de la actividad de búsqueda como entrada de la segunda canalización. La lista de resultados contiene la lista de tablas cuyos datos deben copiarse en el destino.

8. Para comprobar la canalización, haga clic en **Validate** (Comprobar) en la barra de herramientas. Para cerrar **Pipeline Validation Report** (Informe de comprobación de la canalización), haga clic en **>>** .

9. Para publicar entidades (conjuntos de datos, canalizaciones, etc.) en el servicio Data Factory, haga clic en **Publish all** (Publicar todo) en la parte superior de la ventana. Espere hasta que la publicación se realice correctamente.

------



#### Desencadenamiento de una ejecución de la canalización

------

1. Vaya a la canalización **GetTableListAndTriggerCopyData**, haga clic en **Add Trigger** (Agregar desencadenador) y haga clic en **Trigger Now** (Desencadenar ahora).

2. Confirme la ejecución en la página **Pipeline run** (Ejecución de canalización) y, a continuación, seleccione **Finish** (Finalizar).

------



#### Supervisión de la ejecución de la canalización

------

1. Vaya a la pestaña **Monitor** (Supervisar). Haga clic en **Refresh** (Actualizar) hasta que vea las ejecuciones de las canalizaciones. Continúe actualizando la lista hasta que vea el estado **Succeeded** (Correcto).

2. Para ver las ejecuciones de actividad asociadas a la canalización **GetTableListAndTriggerCopyData**, haga clic en el vínculo de nombre de canalización. Debería ver dos ejecuciones de actividad.

3. Para ver la salida de la actividad **Lookup** (Búsqueda), haga clic en el vínculo **Output** (Salida) junto a la actividad en la columna **ACTIVITY NAME** (NOMBRE DE ACTIVIDAD). La ventana **Output** (Salida) se puede maximizar y restaurar. Después de la revisión, haga clic en la **X** para cerrar la ventana **Output** (Salida).

   ```json
   {
       "count": 9,
       "value": [
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "Customer"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "ProductDescription"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "Product"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "ProductModelProductDescription"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "ProductCategory"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "Address"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "CustomerAddress"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "SalesOrderDetail"
           },
           {
               "TABLE_SCHEMA": "SalesLT",
               "TABLE_NAME": "SalesOrderHeader"
           }
       ],
       "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)",
       "effectiveIntegrationRuntimes": [
           {
               "name": "DefaultIntegrationRuntime",
               "type": "Managed",
               "location": "East US",
               "billedDuration": 0,
               "nodes": null
           }
       ]
   }
   ```

4. Para volver a la vista **Pipeline Runs** (Ejecuciones de canalización), haga clic en el vínculo **All Pipeline runs** (Todas las ejecuciones de canalización) en la parte superior del menú de la ruta de navegación. Haga clic en **IterateAndCopySQLTables**, bajo la columna **PIPELINE NAME** (NOMBRE DE CANALIZACIÓN), para ver las ejecuciones de actividad de la canalización. Tenga en cuenta que hay una ejecución de la actividad **Copy** (Copiar) para cada tabla de la salida de la actividad **Lookup** (Búsqueda).

5. Confirme que los datos se copiaron en la instancia de Azure Synapse Analytics de destino.

------

