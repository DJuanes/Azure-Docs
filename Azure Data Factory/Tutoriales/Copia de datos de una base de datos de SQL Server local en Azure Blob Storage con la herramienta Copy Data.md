

## Copia de datos de una base de datos de SQL Server local en Azure Blob Storage con la herramienta Copy Data





En este tutorial, realizará los siguientes pasos:

- Creación de una factoría de datos.
- Uso de la herramienta Copy Data para crear una canalización.
- Supervisión de las ejecuciones de canalización y actividad.



#### Prerrequisitos

Consulte la sección [Prerrequisitos](Prerrequisitos.md).



#### Creación de la factoría de datos

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



