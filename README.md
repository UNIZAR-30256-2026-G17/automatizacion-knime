# Automatización del flujo de datos de Montgomery

Este repositorio contiene la arquitectura de backend diseñada para la extracción, procesamiento, predicción y carga automatizada de los datos de criminalidad del departamento de policía de Montgomery.

## Contenido del repositorio

Actualmente, el repositorio se estructura en torno a la carpeta principal:

* **`/Montgomery`**: Contiene el flujo de trabajo (Workflow) de KNIME configurado para su ejecución autónoma. Este flujo integra toda la lógica de negocio, desde la conexión con la API de origen hasta la escritura final en la base de datos NoSQL.

## Arquitectura del flujo (workflow)

El proceso implementado en KNIME sigue las siguientes etapas técnicas:

1.  **Extracción en vivo**: Se realiza una lectura directa mediante el nodo `CSV Reader` conectado a la API de Montgomery, eliminando la dependencia de descargas manuales o scripts locales.
2.  **Procesamiento y limpieza**: 
    * **Filtrado temporal**: Se conservan únicamente los registros de los últimos 3 años.
    * **Deduplicación**: Eliminación de filas duplicadas utilizando el `incident_id` como referencia.
    * **Estandarización**: Unificación de nombres de ciudades y gestión de valores nulos o erróneos mediante nodos de reemplazo y búsqueda.
3.  **Carga incremental**: El sistema compara los datos procesados con el histórico local (`crimenes_ya_subidos.csv`) para realizar una subida eficiente a **MongoDB Atlas**, enviando únicamente los delitos estrictamente nuevos.
4.  **Modelado predictivo**: Se entrenan tres modelos de **Random Forest** en paralelo para calcular el índice de criminalidad (IC) a escala diaria, mensual y anual. Las predicciones se actualizan diariamente en la nube mediante un modo de escritura de sobrescritura.

## Despliegue y automatización

Aunque este repositorio actúa como almacenamiento del código y versionado, la ejecución operativa se ha trasladado a **KNIME Community Hub**. 

* **Entorno**: El flujo se encuentra alojado en un espacio privado del Hub bajo un plan **Pro**, lo que permite el uso de capacidades de computación en la nube.
* **Programación (schedules)**: Se ha configurado un schedule para ejecutar el flujo completo de forma automática todos los días a las **11:00 de la mañana**. Esto garantiza que la base de datos esté siempre actualizada sin necesidad de intervención manual ni de mantener equipos locales encendidos.

---

> **NOTA DE PRIVACIDAD Y SEGURIDAD**: 
> Por motivos de seguridad y para proteger la integridad de nuestra base de datos en MongoDB Atlas, los campos de credenciales en el nodo **MongoDB Connector** han sido anonimizados. Se han utilizado los marcadores de posición `USUARIO` y `CONTRASEÑA`. Para una ejecución funcional en entornos locales, deberán sustituirse por las credenciales autorizadas del equipo.
