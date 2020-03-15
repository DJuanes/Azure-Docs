

## Copia de datos de una base de datos de SQL Server local en Azure Blob Storage con la herramienta Copy Data





En este tutorial, realizará los siguientes pasos:

- Creación de una factoría de datos.
- Uso de la herramienta Copy Data para crear una canalización.
- Supervisión de las ejecuciones de canalización y actividad.





#### Prerrequisitos

###### Suscripción de Azure

Si no tiene una suscripción a Azure, [puede crear una cuenta gratuita](https://azure.microsoft.com/free/).

###### Roles de Azure

La cuenta debe tener un rol *Contributor* o *Owner* asignado o ser *administrator* de la suscripción de Azure.
Para ver los permisos que tiene en la suscripción, vaya a Azure Portal. En la esquina superior derecha, seleccione su nombre de usuario y luego seleccione **Permissions**.

###### SQL Server 2014, 2016 y 2017

En este tutorial se usa una base de datos de SQL Server local como almacén de datos de origen. Tiene que crear una tabla denominada **emp** e insertar un par de entradas de ejemplo:

```sql
CREATE TABLE dbo.emp
 (
     ID int IDENTITY(1,1) NOT NULL,
     FirstName varchar(50),
     LastName varchar(50)
 )
 GO

 INSERT INTO emp (FirstName, LastName) VALUES ('John', 'Doe')
 INSERT INTO emp (FirstName, LastName) VALUES ('Jane', 'Doe')
 GO
```

###### Cuenta de almacenamiento de Azure

La canalización que crea en este tutorial copia los datos de la base de datos de SQL Server local (origen) en Blob Storage (receptor).
Para obtener el nombre y la clave de la cuenta de almacenamiento, siga estos pasos:

------

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**.
2. Vaya a Azure Portal (https://portal.azure.com/).
3. Seleccione **All services** en el panel izquierdo. Use la palabra clave **Storage** para filtrar el resultado y, luego, seleccione **Storage accounts**.
4. En la lista de cuentas de almacenamiento, filtre por su cuenta de almacenamiento, si fuera necesario. Después, seleccione su cuenta de almacenamiento.
5. En la ventana **Storage account**, seleccione **Access keys**.
6. En los cuadros **Storage account name** y **key1**, copie los valores y péguelos en un editor de texto, para su uso posterior en el tutorial.

------



En esta sección se crea un contenedor de blobs denominado **adftutorial** en la instancia de Blob Storage:

------

1. En la ventana **Storage account**, vaya a **Overview** y, después, seleccione **Blobs**.
2. En la ventana **Blob service**, seleccione **Container**.
3. En la ventana **New container** en **Name**, escriba **adftutorial**. Después, seleccione **OK**.
4. En la lista de contenedores, seleccione **adftutorial**.
5. Mantenga abierta la ventana **container** de **adftutorial**. Úselo para comprobar la salida al final de este tutorial.

------





#### Creación de una factoría de datos

Para crear una factoría de datos consulte [Creación de una factoría de datos](Creaci%C3%B3n%20de%20una%20factor%C3%ADa%20de%20datos.md)





#### Uso de la herramienta Copy Data para crear una canalización

------

1. En la página **Let's get started** (Introducción), seleccione **Copy Data** (Copiar datos).
2. En la página **Properties** (Propiedades), en **Task name** (Nombre de la tarea), especifique **CopyFromOnPremSqlToAzureBlobPipeline**. Luego, seleccione **Next**.
3. En la página **Source data store** (Almacén de datos de origen), haga clic en **Create new connection** (Crear nueva conexión).
4. En **New Linked Service** (Nuevo servicio vinculado), busque **SQL Server** y, a continuación, seleccione  **Continue** (Continuar).
5. En el cuadro de diálogo **New Linked Service (SQL Server)** (Nuevo servicio vinculado), en **Name** (Nombre), escriba **SqlServerLinkedService**. Seleccione **+New** (+Nuevo) en la opción **Connect via integration runtime** (Conectar mediante IR). Tiene que crear un entorno de ejecución de integración, descargarlo en la máquina y registrarlo en Data Factory.
6. En el cuadro de diálogo **Integration Runtime Setup** (Configuración de Integration Runtime), seleccione **Self-Hosted** (Autohospedado). Luego, seleccione **Siguiente**.
7. En el cuadro de diálogo **Integration Runtime Setup** (Configuración de Integration Runtime), en **Name** (Nombre) escriba **TutorialIntegrationRuntime**. Luego, seleccione **Next** (Siguiente).
8. En el cuadro de diálogo **Integration Runtime Setup** (Configuración de Integration Runtime), seleccione **Click here to launch the express setup for this computer** (Haga clic aquí para iniciar la configuración rápida en este equipo). Esta acción instala el entorno de ejecución de integración en la máquina y la registra en Data Factory.
9. Ejecute la aplicación descargada. Verá el estado de la configuración rápida en la ventana.
10. En el cuadro de diálogo **New Linked Service (SQL Server)** (Nuevo servicio vinculado), confirme que **TutorialIntegrationRuntime** está seleccionado en el campo de Integration Runtime. A continuación, siga estos pasos:
    - En **Name** (Nombre), escriba **SqlServerLinkedService**.
    - Escriba el nombre de la instancia de SQL Server local en **Server name** (Nombre del servidor).
    - Escriba el nombre de la base de datos local en **Database name** (Nombre de la base de datos).
    - Seleccione la autenticación adecuada en **Authentication type** (Tipo de autenticación).
    - Escriba el nombre de usuario con acceso a la instancia de SQL Server local en **User name** (Nombre de usuario).
    - Escriba la **password** (contraseña) del usuario.
    - Pruebe la conexión y seleccione **Finish** (Finalizar).
11. En la página **Source data store** (Almacén de datos de origen), seleccione **Next** (Siguiente).
12. En la página **Select tables from which to copy the data or use a custom query** (Seleccionar tablas de donde copiar los datos o usar una consulta personalizada), seleccione la tabla **[dbo].[emp]** de la lista y luego seleccione **Next** (Siguiente).
13. En la página **Destination data store** (Almacén de datos de destino), seleccione **Create new connection** (Crear nueva conexión).
14. En **New Linked Service** (Nuevo servicio vinculado), busque y seleccione **Azure Blob** y, después, seleccione **Continue** (Continuar).
15. En el cuadro de diálogo **New Linked Service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], realice los siguientes pasos:
    - En **Name** (Nombre), escriba **AzureStorageLinkedService**.
    - En **Connect via integration runtime** (Conectar mediante IR), seleccione **TutorialIntegrationRuntime**
    - Seleccione la cuenta de almacenamiento en la lista desplegable **Storage account name** (Nombre de la cuenta de almacenamiento).
    - Seleccione **Finish** (Finalizar).
16. En el cuadro de diálogo **Destination data store** (Almacén de datos de destino), asegúrese de que está seleccionado **Azure Blob Storage**. Luego, seleccione **Next** (Siguiente).
17. En el cuadro de diálogo **Choose the output file or folder** (Elegir el archivo o la carpeta de salida), en **Folder path** (Ruta de acceso a la carpeta) escriba **adftutorial/fromonprem**. Si no existe la carpeta de salida Data Factory la crea automáticamente.
18. En el cuadro de diálogo **File format settings** (Configuración de formato de archivo), seleccione **Next** (Siguiente).
19. En el cuadro de diálogo **Settings** (Configuración), seleccione **Next** (Siguiente).
20. En el cuadro de diálogo **Summary** (Resumen), revise los valores de configuración y seleccione **Next** (Siguiente).
21. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización o la tarea que ha creado.
22. En la pestaña **Monitor** (Supervisión), puede ver el estado de la canalización que ha creado. Puede usar los vínculos de la columna **Action** (Acción) para ver las ejecuciones de la canalización y volver a ejecutarla.
23. Seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) de la columna **Actions** (Acciones) para ver las ejecuciones de actividad asociadas a las de la canalización. Para ver detalles acerca de la operación de copia, seleccione el vínculo **Details** (Detalles) (icono de gafas) en la columna **Actions** (Acciones). Para volver a la vista **Pipeline Runs** (Ejecuciones de canalización) seleccione **Pipelines Runs** (Ejecuciones de canalizaciones) en la parte superior.
24. Confirme que ve un archivo de salida en la carpeta **fromonprem** del contenedor **adftutorial**.
25. Seleccione la pestaña **Edit** (Editar) de la izquierda para cambiar al modo de edición. Seleccione **Code** (Código) para ver el código JSON asociado con la entrada abierta en el editor.

------



