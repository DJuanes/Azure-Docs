## Integración y entrega continuas en Azure Data Factory





#### Información general

En Azure Data Factory, la integración y la entrega continuas (CI/CD) implican el traslado de canalizaciones de Data Factory de un entorno a otro. Puede usar la integración de Data Factory UX con plantillas de Azure Resource Manager para operaciones de CI/CD.En Data Factory UX puede generar una plantilla de Resource Manager desde el menú desplegable **ARM Template**. Al seleccionar **Export ARM Template** (Exportar plantilla de ARM), el portal genera la plantilla de Resource Manager de la factoría de datos y un archivo de configuración que incluye todas las cadenas de conexión y otros parámetros. Después, cree un archivo de configuración para cada entorno (desarrollo, prueba o producción). El archivo de plantilla de Resource Manager principal sigue siendo el mismo para todos los entornos.





#### Ciclo de vida de CI/CD

Aquí se muestra una descripción general de ejemplo del ciclo de vida de CI/CD en una factoría de datos de Azure configurada con Git de Azure Repos:

------

1. Se crea una factoría de datos de desarrollo y se configura con GIT de Azure Repos.
2. A medida que los desarrolladores realizan cambios en sus feature branches, depuran las ejecuciones de la canalización con los cambios más recientes.
3. Una vez que los desarrolladores estén satisfechos con los cambios, crean un pull request desde su feature branch al master branch para que sean revisados por otros miembros.
4. Después de que el pull request se haya aprobado y los cambios se hayan mergeados en el master branch, se pueden publicar en la factoría de desarrollo.
5. Cuando el equipo está listo para implementar los cambios en la factoría de pruebas y después en la de producción, exporta la plantilla de Resource Manager desde el master branch.
6. La plantilla de Resource Manager exportada se implementa con los diferentes archivos de parámetros en la factoría de pruebas y en la de producción.

------





#### Creación de una plantilla de Resource Manager para cada entorno

------

1. En la lista **ARM Template**, seleccione **Export ARM Template** (Exportar plantilla de ARM) para exportar la plantilla de Resource Manager de la factoría de datos en el entorno de desarrollo.
2. En las factorías de datos de prueba y de producción, seleccione **Import ARM Template** (Importar plantilla de ARM). Seleccione **Build your own template in the editor** para abrir el editor de plantillas de Resource Manager.
3. Seleccione **Load file** y después la plantilla de Resource Manager generada.
4. En la sección de configuración, escriba los valores de configuración, como las credenciales del servicio vinculado. Cuando haya terminado, seleccione **Purchase** para implementar la plantilla de Resource Manager.

------





#### Automatización de la integración continua mediante versiones de Azure Pipelines

Aquí se ofrece una guía para configurar una versión de Azure Pipelines, que automatiza la implementación de una factoría de datos en varios entornos:

![](Imagenes/continuous-integration-image1.png)

###### Requisitos

- Una suscripción a Azure vinculada a Visual Studio Team Foundation Server o Azure Repos que use el Azure Resource Manager service endpoint.
- Una data factory configurada con la integración de GIT de Azure Repos.
- Un Azure key vault que contenga los secretos para cada entorno.



###### Configuración de una versión de Azure Pipelines

------

1. En Azure DevOps, abra el proyecto configurado con su data factory.
2. En el lado izquierdo de la página, seleccione **Pipelines** y después **Releases**.
3. Seleccione **New pipeline** o, si tiene pipelines existentes, seleccione **New** y, luego **New release pipeline**.
4. Seleccione la plantilla **Empty job**.
5. En el cuadro **Stage name**, escriba el nombre del entorno.
6. Seleccione **Add artifact** y después el mismo repositorio configurado con la factoría de datos. Seleccione **adf_publish** para el **Default branch**. En **Default version**, seleccione **Latest from default branch**.
7. Añada una tarea de implementación de Azure Resource Manager:
   - En la vista de stage, seleccione **View stage tasks**.
   - Cree una nueva tarea. Busque **Azure Resource Group Deployment** y, después, seleccione **Add**.
   - En la tarea de implementación, seleccione la suscripción, el grupo de recursos y la ubicación de la factoría de datos de destino. Proporcione las credenciales si es necesario.
   - En la lista **Action**, seleccione **Create or update resource group** (Crear o actualizar grupo de recursos).
   - Seleccione el botón de puntos suspensivos ( ... ) situado junto al cuadro **Template**. Busque la plantilla de Azure Resource Manager creada mediante **Import ARM Template** (Importar plantilla de ARM) en la sección Creación de una plantilla de Resource Manager para cada entorno de este artículo. Busque este archivo en la carpeta del branch adf_publish.
   - Seleccione … junto al cuadro **Template parameters** para elegir el archivo de parámetros. El archivo que elija dependerá de si ha creado una copia o si usa el archivo predeterminado, ARMTemplateParametersForFactory.json.
   - Seleccione … junto al cuadro **Override template parameters** y escriba la información de la factoría de datos de destino. En el caso de las credenciales que proceden de Azure Key Vault, escriba el nombre del secreto entre comillas dobles. Por ejemplo, si el nombre del secreto es cred1, escriba **"$(cred1)"** para este valor.
   - Seleccione **Incremental** para el **Deployment mode**.
8. Guarde la canalización de versión.
9. Para desencadenar un release, seleccione **Create release**.

------



###### Obtención de secretos de Azure Key Vault

Hay dos formas de administrar los secretos:

------

- Agregue los secretos al archivo de parámetros.

  Cree una copia del archivo de parámetros que se ha cargado en el branch publish. Establezca los valores de los parámetros que quiera obtener de Key Vault con este formato:

  ```json
  {
      "parameters": {
          "azureSqlReportingDbPassword": {
              "reference": {
                  "keyVault": {
                      "id": "/subscriptions/<subId>/resourceGroups/<resourcegroupId> /providers/Microsoft.KeyVault/vaults/<vault-name> "
                  },
                  "secretName": " < secret - name > "
              }
          }
      }
  }
  ```

  El archivo de parámetros también debe estar en el branch publish.

- Agregue una tarea de Azure Key Vault antes de la tarea de implementación de Azure Resource Manager que se ha descrito en la sección anterior:

  - En la pestaña **Tasks**, cree una tarea. Busque **Azure Key Vault** y agréguelo.
  - En la tarea de Key Vault, seleccione la suscripción en la que haya creado el almacén de claves. Proporcione credenciales si es necesario y, después, seleccione el almacén de claves.

------



###### Actualización de desencadenadores activos

Puede producirse un error en la implementación si intenta actualizar desencadenadores activos. Para actualizar desencadenadores activos, debe detenerlos manualmente e iniciarlos después de la implementación. Puede hacerlo mediante una tarea de Azure PowerShell:

------

1. En la pestaña **Tasks** de la versión, agregue una tarea **Azure PowerShell**. Elija la versión de la tarea 4.*.

2. Seleccione la suscripción en la que se encuentra la factoría.

3. Seleccione **Script File Path** como tipo de script. Esto requiere guardar el script de PowerShell en el repositorio. El siguiente script de PowerShell sirve para detener desencadenadores:

   ```powershell
   $triggersADF = Get-AzDataFactoryV2Trigger -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
   
   $triggersADF | ForEach-Object { Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.name -Force }
   ```

Puede completar pasos similares (con la función *Start-AzDataFactoryV2Trigger*) para reiniciar los desencadenadores después de la implementación.

------



###### Script pre- y post-deployment

Se puede usar este script de ejemplo para detener los desencadenadores antes de la implementación y reiniciarlos más adelante. El script también incluye código para eliminar recursos que se han quitado. Guarde el script en un repositorio de Git de Azure DevOps y haga referencia a él mediante una tarea de Azure PowerShell de la versión 4.*.

Al ejecutar un script pre-deployment, tendrá que especificar una variante de los siguientes parámetros en el campo **Script Arguments**:

```powershell
-armTemplate "$(System.DefaultWorkingDirectory)/<your-arm-template-location>" -ResourceGroupName <your-resource-group-name> -DataFactoryName <your-data-factory-name> -predeployment $true -deleteDeployment $false
```

Al ejecutar un script post-deployment, tendrá que especificar una variante de los siguientes parámetros en el campo **Script Arguments**:

```powershell
-armTemplate "$(System.DefaultWorkingDirectory)/<your-arm-template-location>" -ResourceGroupName <your-resource-group-name> -DataFactoryName <your-data-factory-name> -predeployment $false -deleteDeployment $true
```

Este es el script que se puede usar para el pre- y post-deployment. Contabiliza los recursos eliminados y las referencias a recursos:

```powershell
param
(
    [parameter(Mandatory = $false)] [String] $armTemplate,
    [parameter(Mandatory = $false)] [String] $ResourceGroupName,
    [parameter(Mandatory = $false)] [String] $DataFactoryName,
    [parameter(Mandatory = $false)] [Bool] $predeployment=$true,
    [parameter(Mandatory = $false)] [Bool] $deleteDeployment=$false
)

function getPipelineDependencies {
    param([System.Object] $activity)
    if ($activity.Pipeline) {
        return @($activity.Pipeline.ReferenceName)
    } elseif ($activity.Activities) {
        $result = @()
        return $activity.Activities | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
    } elseif ($activity.ifFalseActivities -or $activity.ifTrueActivities) {
        $result = @()
        $activity.ifFalseActivities | Where-Object {$_ -ne $null} | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        $activity.ifTrueActivities | Where-Object {$_ -ne $null} | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
    } elseif ($activity.defaultActivities) {
        $result = @()
        $activity.defaultActivities | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        if ($activity.cases) {
            $activity.cases | ForEach-Object{ $_.activities } | ForEach-Object{$result += getPipelineDependencies -activity $_ }
        }
        return $result
    }
}

function pipelineSortUtil {
    param([Microsoft.Azure.Commands.DataFactoryV2.Models.PSPipeline]$pipeline,
    [Hashtable] $pipelineNameResourceDict,
    [Hashtable] $visited,
    [System.Collections.Stack] $sortedList)
    if ($visited[$pipeline.Name] -eq $true) {
        return;
    }
    $visited[$pipeline.Name] = $true;
    $pipeline.Activities | ForEach-Object{ getPipelineDependencies -activity $_ -pipelineNameResourceDict $pipelineNameResourceDict}  | ForEach-Object{
        pipelineSortUtil -pipeline $pipelineNameResourceDict[$_] -pipelineNameResourceDict $pipelineNameResourceDict -visited $visited -sortedList $sortedList
    }
    $sortedList.Push($pipeline)

}

function Get-SortedPipelines {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $pipelines = Get-AzDataFactoryV2Pipeline -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $ppDict = @{}
    $visited = @{}
    $stack = new-object System.Collections.Stack
    $pipelines | ForEach-Object{ $ppDict[$_.Name] = $_ }
    $pipelines | ForEach-Object{ pipelineSortUtil -pipeline $_ -pipelineNameResourceDict $ppDict -visited $visited -sortedList $stack }
    $sortedList = new-object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSPipeline]
    
    while ($stack.Count -gt 0) {
        $sortedList.Add($stack.Pop())
    }
    $sortedList
}

function triggerSortUtil {
    param([Microsoft.Azure.Commands.DataFactoryV2.Models.PSTrigger]$trigger,
    [Hashtable] $triggerNameResourceDict,
    [Hashtable] $visited,
    [System.Collections.Stack] $sortedList)
    if ($visited[$trigger.Name] -eq $true) {
        return;
    }
    $visited[$trigger.Name] = $true;
    $trigger.Properties.DependsOn | Where-Object {$_ -and $_.ReferenceTrigger} | ForEach-Object{
        triggerSortUtil -trigger $triggerNameResourceDict[$_.ReferenceTrigger.ReferenceName] -triggerNameResourceDict $triggerNameResourceDict -visited $visited -sortedList $sortedList
    }
    $sortedList.Push($trigger)
}

function Get-SortedTriggers {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $triggers = Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
    $triggerDict = @{}
    $visited = @{}
    $stack = new-object System.Collections.Stack
    $triggers | ForEach-Object{ $triggerDict[$_.Name] = $_ }
    $triggers | ForEach-Object{ triggerSortUtil -trigger $_ -triggerNameResourceDict $triggerDict -visited $visited -sortedList $stack }
    $sortedList = new-object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSTrigger]
    
    while ($stack.Count -gt 0) {
        $sortedList.Add($stack.Pop())
    }
    $sortedList
}

function Get-SortedLinkedServices {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $linkedServices = Get-AzDataFactoryV2LinkedService -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
    $LinkedServiceHasDependencies = @('HDInsightLinkedService', 'HDInsightOnDemandLinkedService', 'AzureBatchLinkedService')
    $Akv = 'AzureKeyVaultLinkedService'
    $HighOrderList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]
    $RegularList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]
    $AkvList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]

    $linkedServices | ForEach-Object {
        if ($_.Properties.GetType().Name -in $LinkedServiceHasDependencies) {
            $HighOrderList.Add($_)
        }
        elseif ($_.Properties.GetType().Name -eq $Akv) {
            $AkvList.Add($_)
        }
        else {
            $RegularList.Add($_)
        }
    }

    $SortedList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]($HighOrderList.Count + $RegularList.Count + $AkvList.Count)
    $SortedList.AddRange($HighOrderList)
    $SortedList.AddRange($RegularList)
    $SortedList.AddRange($AkvList)
    $SortedList
}

$templateJson = Get-Content $armTemplate | ConvertFrom-Json
$resources = $templateJson.resources

#Triggers 
Write-Host "Getting triggers"
$triggersADF = Get-SortedTriggers -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
$triggersTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/triggers" }
$triggerNames = $triggersTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
$activeTriggerNames = $triggersTemplate | Where-Object { $_.properties.runtimeState -eq "Started" -and ($_.properties.pipelines.Count -gt 0 -or $_.properties.pipeline.pipelineReference -ne $null)} | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
$deletedtriggers = $triggersADF | Where-Object { $triggerNames -notcontains $_.Name }
$triggerstostop = $triggerNames | where { ($triggersADF | Select-Object name).name -contains $_ }

if ($predeployment -eq $true) {
    #Stop all triggers
    Write-Host "Stopping deployed triggers"
    $triggerstostop | ForEach-Object { 
        Write-host "Disabling trigger " $_
        Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_ -Force 
    }
}
else {
    #Deleted resources
    #pipelines
    Write-Host "Getting pipelines"
    $pipelinesADF = Get-SortedPipelines -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $pipelinesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/pipelines" }
    $pipelinesNames = $pipelinesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedpipelines = $pipelinesADF | Where-Object { $pipelinesNames -notcontains $_.Name }
    #dataflows
    $dataflowsADF = Get-AzDataFactoryV2DataFlow -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $dataflowsTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/dataflows" }
    $dataflowsNames = $dataflowsTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40) }
    $deleteddataflow = $dataflowsADF | Where-Object { $dataflowsNames -notcontains $_.Name }
    #datasets
    Write-Host "Getting datasets"
    $datasetsADF = Get-AzDataFactoryV2Dataset -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $datasetsTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/datasets" }
    $datasetsNames = $datasetsTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40) }
    $deleteddataset = $datasetsADF | Where-Object { $datasetsNames -notcontains $_.Name }
    #linkedservices
    Write-Host "Getting linked services"
    $linkedservicesADF = Get-SortedLinkedServices -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $linkedservicesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/linkedservices" }
    $linkedservicesNames = $linkedservicesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedlinkedservices = $linkedservicesADF | Where-Object { $linkedservicesNames -notcontains $_.Name }
    #Integrationruntimes
    Write-Host "Getting integration runtimes"
    $integrationruntimesADF = Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $integrationruntimesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/integrationruntimes" }
    $integrationruntimesNames = $integrationruntimesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedintegrationruntimes = $integrationruntimesADF | Where-Object { $integrationruntimesNames -notcontains $_.Name }

    #Delete resources
    Write-Host "Deleting triggers"
    $deletedtriggers | ForEach-Object { 
        Write-Host "Deleting trigger "  $_.Name
        $trig = Get-AzDataFactoryV2Trigger -name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
        if ($trig.RuntimeState -eq "Started") {
            Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force 
        }
        Remove-AzDataFactoryV2Trigger -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting pipelines"
    $deletedpipelines | ForEach-Object { 
        Write-Host "Deleting pipeline " $_.Name
        Remove-AzDataFactoryV2Pipeline -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting dataflows"
    $deleteddataflow | ForEach-Object { 
        Write-Host "Deleting dataflow " $_.Name
        Remove-AzDataFactoryV2DataFlow -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting datasets"
    $deleteddataset | ForEach-Object { 
        Write-Host "Deleting dataset " $_.Name
        Remove-AzDataFactoryV2Dataset -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting linked services"
    $deletedlinkedservices | ForEach-Object { 
        Write-Host "Deleting Linked Service " $_.Name
        Remove-AzDataFactoryV2LinkedService -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting integration runtimes"
    $deletedintegrationruntimes | ForEach-Object { 
        Write-Host "Deleting integration runtime " $_.Name
        Remove-AzDataFactoryV2IntegrationRuntime -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }

    if ($deleteDeployment -eq $true) {
        Write-Host "Deleting ARM deployment ... under resource group: " $ResourceGroupName
        $deployments = Get-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName
        $deploymentsToConsider = $deployments | Where { $_.DeploymentName -like "ArmTemplate_master*" -or $_.DeploymentName -like "ArmTemplateForFactory*" } | Sort-Object -Property Timestamp -Descending
        $deploymentName = $deploymentsToConsider[0].DeploymentName

       Write-Host "Deployment to be deleted: " $deploymentName
        $deploymentOperations = Get-AzResourceGroupDeploymentOperation -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
        $deploymentsToDelete = $deploymentOperations | Where { $_.properties.targetResource.id -like "*Microsoft.Resources/deployments*" }

        $deploymentsToDelete | ForEach-Object { 
            Write-host "Deleting inner deployment: " $_.properties.targetResource.id
            Remove-AzResourceGroupDeployment -Id $_.properties.targetResource.id
        }
        Write-Host "Deleting deployment: " $deploymentName
        Remove-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName -Name $deploymentName
    }

    #Start active triggers - after cleanup efforts
    Write-Host "Starting active triggers"
    $activeTriggerNames | ForEach-Object { 
        Write-host "Enabling trigger " $_
        Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_ -Force 
    }
}
```





#### Usar parámetros personalizados con la plantilla de Resource Manager

Es posible que desee reemplazar la plantilla de parametrización predeterminada en estos escenarios:

- Se usa CI/CD automatizada y se quieren cambiar algunas propiedades durante la implementación de Resource Manager, pero las propiedades no están parametrizadas de forma predeterminada.
- La fábrica es tan grande que la plantilla de Resource Manager predeterminada no es válida porque contiene más parámetros que el número máximo permitido (256).

En estas condiciones, para reemplazar la plantilla de parametrización predeterminada, cree un archivo denominado arm-template-parameters-definition.json en la carpeta especificada como carpeta raíz para la integración de Git.



###### Sintaxis de un archivo de parámetros personalizados

Puede usar las instrucciones siguientes durante la creación del archivo de parámetros personalizado. El archivo consta de una sección para cada tipo de entidad: trigger, pipeline, linked service, dataset, integration runtime, etc.

------

- Escriba la ruta de acceso de la propiedad en el tipo de entidad correspondiente.
- El establecimiento de un nombre de propiedad en * indica que quiere parametrizar todas las propiedades que incluye.
- El establecimiento del valor de una propiedad como una cadena indica que quiere parametrizar la propiedad. Use el formato <action>:<name>:<stype>.
  - <action> puede ser uno de estos caracteres:
    - = significa que el valor actual debe conservarse como el valor predeterminado para el parámetro.
    - -significa que no se debe conservar el valor predeterminado para el parámetro.
    - | es un caso especial para los secretos de Azure Key Vault para una cadena de conexión o claves.
  - <name> es el nombre del parámetro. Si está en blanco, toma el nombre de la propiedad. Si el valor empieza por un carácter -, el nombre está abreviado.
  - <stype> es el tipo del parámetro. Si <stype> está en blanco, el tipo predeterminado es string. Los valores admitidos son: string, bool, number, object y securestring.
  - Al especificar una matriz en el archivo de definición indica que la propiedad coincidente en la plantilla es una matriz. Data Factory recorre en iteración todos los objetos de la matriz mediante la definición especificada en el objeto del entorno de ejecución de integración de la matriz. El segundo objeto, una cadena, se convierte en el nombre de la propiedad, que se utiliza como el nombre del parámetro para cada iteración.
- Una definición no puede ser específica de una instancia de recurso. Cualquier definición se aplica a todos los recursos de ese tipo.
- De forma predeterminada, todas las cadenas seguras, como los secretos de Key Vault, las cadenas de conexión, las claves y los tokens, están parametrizadas.

------



###### Plantilla de parametrización de ejemplo

```json
{
    "Microsoft.DataFactory/factories/pipelines": {
        "properties": {
            "activities": [{
                "typeProperties": {
                    "waitTimeInSeconds": "-::number",
                    "headers": "=::object"
                }
            }]
        }
    },
    "Microsoft.DataFactory/factories/integrationRuntimes": {
        "properties": {
            "typeProperties": {
                "*": "="
            }
        }
    },
    "Microsoft.DataFactory/factories/triggers": {
        "properties": {
            "typeProperties": {
                "recurrence": {
                    "*": "=",
                    "interval": "=:triggerSuffix:number",
                    "frequency": "=:-freq"
                },
                "maxConcurrency": "="
            }
        }
    },
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "connectionString": "|:-connectionString:secureString",
                    "secretAccessKey": "|"
                }
            }
        },
        "AzureDataLakeStore": {
            "properties": {
                "typeProperties": {
                    "dataLakeStoreUri": "="
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/datasets": {
        "properties": {
            "typeProperties": {
                "*": "="
            }
        }
    }
}
```

Esta es una explicación de cómo se construye la plantilla anterior, desglosada por tipo de recurso:

**Pipelines**

- Cualquier propiedad de la ruta de acceso activities/typeProperties/waitTimeInSeconds está parametrizada. Cualquier actividad en una canalización que tiene una propiedad de nivel de código denominada waitTimeInSeconds está parametrizada como un número, con un nombre predeterminado. Sin embargo, no tendrá un valor predeterminado en la plantilla de Resource Manager. Será una entrada obligatoria durante la implementación de Resource Manager.
- De forma similar, una propiedad denominada headers (por ejemplo, en una actividad Web) está parametrizada con el tipo object (JObject). Tiene un valor predeterminado, que es el mismo que el de la factoría de origen.

**IntegrationRuntimes**

- Todas las propiedades bajo la ruta de acceso typeProperties están parametrizadas con sus valores predeterminados correspondientes. Por ejemplo, hay dos propiedades en las propiedades de tipo IntegrationRuntimes: computeProperties y ssisProperties. Ambos tipos de propiedades se crean con sus respectivos valores y tipos (Object) predeterminados.

**Triggers**

- En typeProperties, hay dos propiedades parametrizadas. La primera de ellas es maxConcurrency, que tiene especificado un valor predeterminado y es de tipo string. Tiene el nombre de parámetro predeterminado <entityName>_properties_typeProperties_maxConcurrency._
- La propiedad recurrence también está parametrizada. En ella, se especifica que todas las propiedades de ese nivel están parametrizadas como cadenas, con valores predeterminados y los nombres de parámetro. Una excepción es la propiedad interval, que está parametrizada como el tipo number. El sufijo del nombre del parámetro es <entityName>_properties_typeProperties_recurrence_triggerSuffix. De forma similar, la propiedad freq es una cadena y está parametrizada como una cadena, pero sin un valor predeterminado.

**LinkedServices**

- Los servicios vinculados son únicos. Dado que los servicios vinculados y los conjuntos de datos tienen una gran variedad de tipos, puede proporcionar una personalización específica de tipo. En este ejemplo, para todos los servicios vinculados de tipo AzureDataLakeStore, se aplicará una plantilla específica. Para todos los demás (a través de *), se aplicará otra plantilla.
- La propiedad connectionString se parametrizará como un valor securestring. No tendrá un valor predeterminado.
- La propiedad secretAccessKey resulta ser AzureKeyVaultSecret. Se parametriza automáticamente como un secreto de Azure Key Vault y se captura desde el almacén de claves configurado. También puede parametrizar el propio almacén de claves.

**Datasets**

- Aunque la personalización específica de tipos está disponible para conjuntos de datos, puede proporcionar configuración sin necesidad de una configuración explícita de nivel *. En el ejemplo anterior, todas las propiedades del conjunto de datos de typeProperties están parametrizadas.



###### Plantilla de parametrización predeterminada

Aquí se muestra la plantilla de parametrización predeterminada actual:

```json
{
    "Microsoft.DataFactory/factories/pipelines": {
    },
    "Microsoft.DataFactory/factories/dataflows": {
    },
    "Microsoft.DataFactory/factories/integrationRuntimes":{
        "properties": {
            "typeProperties": {
                "ssisProperties": {
                    "catalogInfo": {
                        "catalogServerEndpoint": "=",
                        "catalogAdminUserName": "=",
                        "catalogAdminPassword": {
                            "value": "-::secureString"
                        }
                    },
                    "customSetupScriptProperties": {
                        "sasToken": {
                            "value": "-::secureString"
                        }
                    }
                },
                "linkedInfo": {
                    "key": {
                        "value": "-::secureString"
                    },
                    "resourceId": "="
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/triggers": {
        "properties": {
            "pipelines": [{
                    "parameters": {
                        "*": "="
                    }
                },  
                "pipelineReference.referenceName"
            ],
            "pipeline": {
                "parameters": {
                    "*": "="
                }
            },
            "typeProperties": {
                "scope": "="
            }

        }
    },
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "userName": "=",
                    "accessKeyId": "=",
                    "servicePrincipalId": "=",
                    "userId": "=",
                    "clientId": "=",
                    "clusterUserName": "=",
                    "clusterSshUserName": "=",
                    "hostSubscriptionId": "=",
                    "clusterResourceGroup": "=",
                    "subscriptionId": "=",
                    "resourceGroupName": "=",
                    "tenant": "=",
                    "dataLakeStoreUri": "=",
                    "baseUrl": "=",
                    "database": "=",
                    "serviceEndpoint": "=",
                    "batchUri": "=",
            "poolName": "=",
                    "databaseName": "=",
                    "systemNumber": "=",
                    "server": "=",
                    "url":"=",
                    "aadResourceId": "=",
                    "connectionString": "|:-connectionString:secureString"
                }
            }
        },
        "Odbc": {
            "properties": {
                "typeProperties": {
                    "userName": "=",
                    "connectionString": {
                        "secretName": "="
                    }
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/datasets": {
        "*": {
            "properties": {
                "typeProperties": {
                    "folderPath": "=",
                    "fileName": "="
                }
            }
        }}
}
```

En el ejemplo siguiente se muestra cómo agregar un único valor a la plantilla. Solamente se quiere agregar un identificador de clúster interactivo de Azure Databricks. El archivo es el mismo que el anterior, salvo por la adición de existingClusterId en el campo de propiedades de Microsoft.DataFactory/factories/linkedServices:

```json
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "userName": "=",
                    "accessKeyId": "=",
                    "servicePrincipalId": "=",
                    "userId": "=",
                    "clientId": "=",
                    "clusterUserName": "=",
                    "clusterSshUserName": "=",
                    "hostSubscriptionId": "=",
                    "clusterResourceGroup": "=",
                    "subscriptionId": "=",
                    "resourceGroupName": "=",
                    "tenant": "=",
                    "dataLakeStoreUri": "=",
                    "baseUrl": "=",
                    "database": "=",
                    "serviceEndpoint": "=",
                    "batchUri": "=",
            "poolName": "=",
                    "databaseName": "=",
                    "systemNumber": "=",
                    "server": "=",
                    "url":"=",
                    "aadResourceId": "=",
                    "connectionString": "|:-connectionString:secureString",
                    "existingClusterId": "-"
                }
            }
        },
        "Odbc": {
            "properties": {
                "typeProperties": {
                    "userName": "=",
                    "connectionString": {
                        "secretName": "="
                    }
                }
            }
        }
    },
```





#### Hotfix branch

Si implementa una factoría en producción y se da cuenta de que hay un error que se debe corregir de inmediato, es posible que deba implementar un hotfix. Este enfoque se conoce como ingeniería de corrección rápida o QFE.

------

1. En Azure DevOps, vaya a la release que se ha implementado en producción. Busque el último commit que se ha implementado.
2. En el mensaje del commit, obtenga el commit ID del collaboration branch.
3. Cree un hotfix branch a partir de ese commit.
4. Vaya a Azure Data Factory UX y cambie al hotfix branch.
5. Mediante Azure Data Factory UX, corrija el error. Testee los cambios.
6. Una vez comprobada la corrección, seleccione **Export ARM Template** (Exportar plantilla de ARM) para obtener la plantilla de Resource Manager del hotfix.
7. Compruebe manualmente esta compilación en el branch adf_publish.
8. Si ha configurado su release pipeline para que se desencadene automáticamente en función de las inserciones en el repositorio de adf_publish, se iniciará un nuevo release de forma automática. De lo contrario, ponga en cola manualmente el release.
9. Implemente el release hotfix en las factorías de test y producción. Este release contiene la carga de producción anterior más la revisión realizada en el paso 5.
10. Agregue los cambios del hotfix al branch de desarrollo para que las versiones posteriores no incluyan el mismo error.

------





#### Procedimientos recomendados para CI/CD

- **Integración de Git**. Solo tiene que configurar la factoría de datos de desarrollo con la integración de Git. Los cambios en los entornos de test y producción se implementan a través de CI/CD y no se necesita la integración Git.
- **Script de CI/CD de Data Factory**. Antes del paso de implementación de Resource Manager en CI/CD, debe completar ciertas tareas, como detener y reiniciar los triggers y realizar la limpieza. Se recomienda usar scripts de PowerShell antes y después de la implementación.
- **Integration runtimes y uso compartido**. Integration runtimes no cambian a menudo y son similares en todas las fases de CI/CD. Si quiere compartir integration runtimes en todos los stages, considere la posibilidad de usar una factoría ternaria solo para contener los entornos de ejecución de integración compartidos. Puede usar esta factoría compartida en todos los entornos como un tipo de entorno de ejecución de integración vinculado.
- **Key Vault**. Cuando usa servicios vinculados basados en Azure Key Vault, puede aprovecharlos todavía más si mantiene almacenes de claves independientes para diferentes entornos. También puede configurar niveles de permisos independientes para cada almacén de claves. Si sigue este enfoque, se recomienda mantener los mismos nombres de secreto en todas las fases.



#### Características no admitidas

- Por diseño, Data Factory no permite cherry-picking de commits o la publicación selectiva de recursos. Las publicaciones incluirán todos los cambios realizados en la factoría de datos.
  - Las entidades de Data Factory dependen unas de otras. La publicación selectiva de un subconjunto de recursos podría provocar comportamientos y errores inesperados.
  - En casos excepcionales, cuando necesite la publicación selectiva, considere la posibilidad de usar un hotfix.
- No es posible publicar desde branches privados.
- En la actualidad no se pueden hospedar proyectos en Bitbucket.