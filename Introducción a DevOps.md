

## ¿Qué es DevOps?

------

DevOps es la unión de personas, procesos y productos para permitir la entrega continua de valor a nuestros usuarios finales.

Esto se logra trabajando juntos con un conjunto compartido de prácticas y herramientas.

- Planificación ágil: creamos un backlog de trabajo que todos los miembros del equipo y de la gerencia puedan ver. Priorizamos los elementos para que sepamos en qué necesitamos trabajar primero. El backlog puede incluir historias de usuarios, bugs y cualquier otra información que nos ayude.
- Integración continua (CI): automatizamos cómo construimos y probamos nuestro código. Lo ejecutamos cada vez que un miembro del equipo comitee cambios en el control de versiones.
- Entrega continua (CD): CD es cómo probamos, configuramos e implementamos desde una compilación a un entorno de QA o de producción.
- Monitoreo: utilizamos la telemetría para obtener información sobre el rendimiento y los patrones de uso de una aplicación. Podemos usar esa información para mejorar a medida que iteramos.

#### ¿Qué hace que un equipo sea considerado de élite?

DevOps ayuda a las empresas a experimentar con formas de aumentar la adopción y la satisfacción del cliente. Puede conducir a un mejor desempeño organizacional y, a menudo, a una mayor rentabilidad y participación en el mercado.

Se utiliza métricas para crear cuatro categorías para comparar a los equipos de élite con los de bajo rendimiento:

- Implementar con más frecuencia: prácticas como el monitoreo, las pruebas continuas, la gestión de cambios de base de datos y la integración de la seguridad al principio del proceso de desarrollo de software ayudan a los equipos de élite a desplegar con mayor frecuencia y con mayor previsibilidad y seguridad.
- Reducir el tiempo de entrega desde el commit hasta la implementación: el tiempo de entrega (lead time) es el tiempo que tarda un feature en llegar al cliente. Al trabajar en requerimientos más pequeños, automatizar procesos manuales e implementar con mayor frecuencia, los equipos de élite pueden lograr en horas o días lo que una vez tomó semanas o incluso meses.
- Reducir la tasa de fallos de cambios: un nuevo feature que falla en producción o que causa que otros desarrollos se rompan puede crear una oportunidad perdida entre usted y sus usuarios. A medida que los equipos de alto rendimiento maduran, reducen su tasa de fallas.
- Recuperarse de incidentes más rápidamente: actuar según las métricas ayuda a los equipos de élite a recuperarse más rápidamente mientras se despliega con más frecuencia.

La forma de implementar la infraestructura en la nube también es importante. La nube mejora el rendimiento de la entrega de software, y los equipos que adoptan características esenciales de la nube tienen más probabilidades de convertirse en equipos de élite.

DevOps es una razón clave por la que muchos equipos de élite pueden ofrecer valor a los clientes, en forma de nuevos features y mejoras, más rápidamente que sus competidores.

#### Lo que DevOps no es

Al considerar qué es DevOps, también es importante asegurarse de que entendemos lo que no es. DevOps no es:

- Una metodología
- Una pieza específica de software
- Una solución rápida para los desafíos de una organización
- Solo un título de equipo o trabajo



## ¿Qué es Azure DevOps?

------

Azure DevOps es un conjunto de servicios que abarca todo el ciclo de vida de DevOps. Comienza con la planificación y se extiende hasta la implementación y la supervisión.

Azure DevOps proporciona varias herramientas que puede usar para una mejor colaboración en equipo. También tiene herramientas para procesos de compilación automatizados, pruebas, control de versiones y gestión de paquetes.

![img](C:\Users\Diego\Documents\GitHub\Azure-Docs\Imagenes\Azure Boards.png)

Azure Boards: es una herramienta ágil que nos ayuda a planificar, monitorear y discutir nuestro trabajo.

![img](Imagenes/Azure Repos.png)

Azure Repos: proporciona repositorios Git públicos y privados ilimitados, alojados en la nube.

![img](Imagenes/Azure Pipelines.png)

Azure Pipelines: nos permitirá construir, probar e implementar con CI / CD que funciona con cualquier lenguaje, plataforma y nube.

![img](Imagenes/Azure Test Plans.png)

Azure Test Plans: son herramientas de prueba manuales y exploratorias.

![img](Imagenes/Azure Artifacts.png)

Azure Artifacts: nos permite crear, alojar y compartir paquetes.



## Ejercicio: crear una organización de Azure DevOps

------

#### Crear una cuenta de Azure DevOps

1. Vaya a **dev.azure.com**.
2. Seleccione el botón **Start free**.
3. Inicie sesión con su cuenta de Microsoft. O si no tiene una cuenta de Microsoft, seleccione **Create One!** y completa los pasos.
4. Revise los Términos de servicio, la Declaración de privacidad y el Código de conducta, y seleccione **Continue** si los acepta.

#### Crear una organización

1. Si nunca ha creado una organización de Azure DevOps, verá una ventana con el botón **Create new organization**. Si es así, verá un enlace que dice **New organization**. Selecciona la opción que ves.
2. En la ventana de notificaciones de Privacidad y Términos de servicio de Azure DevOps, seleccione **Continue**.
3. Cree una organización junto al campo dev.azure.com/. Si se le indica que el nombre ya está en uso, simplemente agregue algunos números al final para que sea único.
4. Elija una ubicación cerca de usted donde se alojarán sus proyectos.
5. Selecciona **Continue**.