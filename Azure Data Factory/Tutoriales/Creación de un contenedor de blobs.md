

## Creación de un contenedor de blobs





Cree una cuenta de Azure Storage (Blob Storage) de uso general. Siga estos pasos para crear un contenedor de blobs:

------

1. En la página de la cuenta de almacenamiento, seleccione **Overview** > **Containers** (Información general > Contenedores).

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

