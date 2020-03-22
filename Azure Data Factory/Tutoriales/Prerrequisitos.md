

## Prerrequisitos





Para poder realizar los tutoriales, primero debe realizar los siguientes pasos:

###### Suscripción de Azure

Si no tiene una suscripción a Azure, [puede crear una cuenta gratuita](https://azure.microsoft.com/free/).



###### Roles de Azure

La cuenta debe tener un rol *Contributor* o *Owner* asignado o ser *administrator* de la suscripción de Azure.
Para ver los permisos que tiene en la suscripción, vaya a Azure Portal. En la esquina superior derecha, seleccione su nombre de usuario y luego seleccione **Permissions** (Permisos).



###### Cuenta de Azure Storage

Cree una cuenta de Azure Storage (Blob Storage) de uso general. Siga estos pasos para crear un contenedor de blobs:

------

1. En la página **Storage account** (cuenta de almacenamiento), seleccione **Overview** > **Containers** (Información general > Contenedores).

2. En la barra de herramientas de la página <*Account name*> - **Containers** (<*Nombre de la cuenta*> - Contenedores), seleccione **Container** (Contenedor).

3. En el cuadro de diálogo **New container** (Nuevo contenedor), escriba **adftutorial** para el nombre y seleccione **OK** (Aceptar).

4. Abra un editor de texto y cree un archivo denominado **emp.txt** con el siguiente contenido:

   ```
   John, Doe
   Jane, Doe
   ```

5. Guarde el archivo en la carpeta **C:\ADFv2QuickStartPSH**. A continuación, vuelva a Azure Portal.

6. En la página <*Account name*> - **Containers** (<*Nombre de la cuenta*> - Contenedores) donde lo dejó, seleccione **adftutorial** en la lista actualizada de contenedores.

   - Si ha cerrado la ventana o ha pasado a otra página; inicie sesión de nuevo en [Azure Portal](https://portal.azure.com/).
   - En el menú de Azure Portal, seleccione **All services** (Todos los servicios) y, a continuación, seleccione **Storage** > **Storage accounts** (Almacenamiento > Cuentas de almacenamiento).
   - Seleccione la cuenta de almacenamiento y, después, seleccione **Containers** > **adftutorial** (Contenedores > adftutorial).

7. En la barra de herramientas de la página del contenedor **adftutorial**, seleccione **Upload** (Cargar).

8. En la página **Upload blob** (Cargar blob), seleccione **Files** (Archivos) y, a continuación, busque y seleccione el archivo **emp.txt**.

9. Expanda el título **Advanced** (Avanzado).

10. En el cuadro **Upload to folder** (Cargar en carpeta), escriba **input**.

11. Seleccione el botón **Upload** (Cargar). Debería ver el archivo **emp.txt** y el estado de la carga en la lista.

12. Seleccione el icono **Close** (**X**) (Cerrar).

------



Para obtener el nombre y la clave de la cuenta de almacenamiento, siga estos pasos:

------

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**.
2. Vaya a Azure Portal (https://portal.azure.com/).
3. Seleccione **All services** en el panel izquierdo. Use la palabra clave **Storage** para filtrar el resultado y, luego, seleccione **Storage accounts**.
4. En la lista de cuentas de almacenamiento, filtre por su cuenta de almacenamiento, si fuera necesario. Después, seleccione su cuenta de almacenamiento.
5. En la ventana **Storage account**, seleccione **Access keys**.
6. En los cuadros **Storage account name** y **key1**, copie los valores y péguelos en un editor de texto, para su uso posterior.

------



###### SQL Server 2014, 2016 y 2017

Para los tutoriales en los que se usa una base de datos de SQL Server local como almacén de datos de origen, tiene que crear una tabla denominada **emp** e insertar un par de entradas de ejemplo:

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

 

###### Azure SQL Database / Azure SQL Data Warehouse

En algunos tutoriales se usa Azure SQL Database como origen, con los datos de ejemplo de Adventure Works LT, y Azure SQL Data Warehouse como el destino de la canalización.

