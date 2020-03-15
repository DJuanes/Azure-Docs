

## Carga de datos en Azure SQL Data Warehouse mediante Azure Data Factory





#### Prerrequisitos

- Suscripción de Azure: si no tiene una suscripción a Azure, [puede crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Azure SQL Data Warehouse: Este almacén de datos contiene los datos que se copian de SQL Database.
- Azure SQL Database: este tutorial copia los datos de una base de datos de Azure SQL con los datos de ejemplo de Adventure Works LT.
- Cuenta de almacenamiento de Azure: Azure Storage se usa como blob de *almacenamiento provisional* en la operación de copia masiva.





#### Creación una factoría de datos

Para crear una factoría de datos consulte [Creación de una factoría de datos](Creaci%C3%B3n%20de%20una%20factor%C3%ADa%20de%20datos.md).





#### Carga de datos en Azure SQL Data Warehouse

------

1. En la página **Get started** (Introducción), seleccione el icono **Copy Data** (Copiar datos).
2. En la página **Properties** (Propiedades), especifique **CopyFromSQLToSQLDW** en el campo **Task name** (Nombre de la tarea) y seleccione **Next** (Siguiente).

3. En la página **Source data store** (Almacén de datos de origen), realice los pasos siguientes:
   - Haga clic en **+ Create new connection** (+ Crear nueva conexión).
   - Seleccione **Azure SQL Database** en la galería y seleccione **Continue** (Continuar).
   - En la página **New Linked Service** (Nuevo servicio vinculado) seleccione el nombre del servidor y el de la base de datos en la lista desplegable y especifique el nombre de usuario y la contraseña. Haga clic en **Test connection** (Prueba de conexión) para validar la configuración y, después, seleccione **Finish** (Finalizar).
   - Seleccione el servicio vinculado recién creado como origen y, a continuación, haga clic en **Next** (Siguiente).
4. En la página **Select tables from which to copy the data or use a custom query** (Seleccionar tablas de donde copiar los datos o usar una consulta personalizada), escriba **SalesLT** para filtrar las tablas. Elija el cuadro **(Select all)** (Seleccionar todo) para usar todas las tablas para la copia y, a continuación, seleccione **Next** (Siguiente).

5. En la página **Destination data store** (Almacén de datos de destino), realice los pasos siguientes:
   - Haga clic en **+ Create new connection** (+ Crear nueva conexión) para agregar una conexión.
   - Seleccione **Azure SQL Data Warehouse** en la galería y seleccione **Next** (Siguiente).
   - En la página **New Linked Service** (Nuevo servicio vinculado) seleccione el nombre del servidor y el de la base de datos en la lista desplegable y especifique el nombre de usuario y la contraseña. Haga clic en **Test connection** (Prueba de conexión) para validar la configuración y, después, seleccione **Finish** (Finalizar).
   - Seleccione el servicio vinculado recién creado como receptor y, a continuación, haga clic en **Next** (Siguiente).
6. En la página **Table mapping** (Asignación de tabla), revise el contenido y seleccione **Next** (Siguiente). Se muestra una asignación de tabla inteligente. Las tablas de origen se asignan a las tablas de destino en función de los nombres de tabla. Si la tabla de origen no existe en el destino, Azure Data Factory crea una con el mismo nombre de manera predeterminada. También se puede asignar una tabla de origen a una tabla de destino existente.

7. En la página **Schema mapping** (Asignación de esquema), revise el contenido y seleccione **Next** (Siguiente). La asignación inteligente de tabla se basa en el nombre de las columnas. Si permite que Data Factory cree automáticamente las tablas, la conversión de tipos de datos puede producirse cuando haya incompatibilidades entre los almacenes de origen y de destino. Si hay una conversión de tipos de datos no compatibles entre las columnas de origen y de destino, verá un mensaje de error junto a la tabla correspondiente.

8. En la página **Settings** (Configuración), siga estos pasos:
   - En la sección **Staging settings** (Configuración provisional), haga clic en **+ New** (+Nuevo) para crear un almacenamiento provisional. El almacenamiento se usa para almacenar provisionalmente los datos antes de cargarlos en SQL Data Warehouse mediante PolyBase. Una vez que se completa la copia, los datos provisionales en Azure Storage se limpian automáticamente.
   - En la página **New Linked Service** (Nuevo servicio vinculado), seleccione su cuenta de almacenamiento y seleccione **Finish** (Finalizar).
   - En la sección **Advanced settings** (Configuración avanzada), anule la selección de la opción **Use type default** (Usar valor predeterminado de type) y seleccione **Next** (Siguiente).
9. En la página **Summary** (Resumen), revise la configuración y seleccione **Next** (Siguiente)

10. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización (tarea).

11. La columna **Actions** (Acciones) incluye los vínculos para ver los detalles de la ejecución de actividad y volver a ejecutar la canalización.

12. Para ver las ejecuciones de actividad, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). Para volver a la vista de ejecuciones de canalización, seleccione el vínculo **Pipelines** (Canalizaciones) de la parte superior. Seleccione **Refresh** (Actualizar) para actualizar la lista.

13. Para supervisar los detalles de la ejecución de cada actividad de copia, seleccione el vínculo **Details** (Detalles) en **Actions** (Acciones) en la vista de supervisión de la actividad.


------

