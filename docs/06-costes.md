## 1. Creación de una nueva estimación en AWS Pricing Calculator

Se accedió a la calculadora oficial de costes de AWS y se creó una nueva estimación. Esta estimación servirá como base para configurar los servicios utilizados en la actividad y comparar sus costes mensuales y anuales entre las regiones de España y Norte de Virginia.

## 2. Selección del servicio Amazon EC2 en la calculadora

Se localizó y seleccionó el servicio Amazon EC2 dentro de AWS Pricing Calculator, con el fin de estimar el coste de la instancia utilizada como servidor web en la actividad.

## 3. Configuración de Amazon EC2 para la región España

Se configuró el servicio Amazon EC2 en AWS Pricing Calculator para la región Europa (España) - Madrid, utilizando una instancia bajo demanda sin reserva de tipo t3.micro. Se mantuvieron las opciones por defecto en los parámetros no especificados, incluyendo el sistema operativo Linux, tenencia compartida y carga de trabajo de uso constante.

## 4. Configuración del modelo de pago bajo demanda en EC2

Se configuró el modelo de pago de la instancia EC2 como bajo demanda, sin utilizar planes de ahorro ni reservas, cumpliendo con el requisito de la actividad que especifica el uso de instancias sin reserva.

## 5. Configuración del tráfico de red en EC2 para la región España

Se configuró la estimación del tráfico de red para la instancia EC2 en la región España, estableciendo 20 GB mensuales de tráfico de entrada desde Internet y 50 GB mensuales de tráfico de salida hacia Internet. Aunque AWS no aplica cargos por tráfico entrante, se incluyó este valor para reflejar completamente el escenario planteado en la actividad.

## 6. Configuración de Amazon EC2 para la región Norte de Virginia

Se creó una segunda configuración del servicio Amazon EC2 en AWS Pricing Calculator para la región US East (N. Virginia), utilizando una instancia bajo demanda sin reserva de tipo t3.micro. Esta configuración replica los mismos parámetros utilizados en la región España, permitiendo posteriormente realizar la comparativa de costes entre ambas regiones.

## 7. Configuración del modelo de pago y tráfico en EC2 para la región Norte de Virginia

Se configuró el modelo de pago bajo demanda para la instancia EC2 en la región US East (N. Virginia), replicando las mismas condiciones establecidas para la región España. Asimismo, se definió un tráfico de red de 20 GB mensuales de entrada y 50 GB mensuales de salida hacia Internet, con el fin de mantener condiciones equivalentes para la comparativa de costes.

## 8. Configuración de Amazon RDS MySQL para la región España

Se añadió a la estimación el servicio Amazon RDS MySQL para la región Europa (España) - Madrid. Se configuró una instancia de base de datos de tipo db.t3.micro con un nodo, almacenamiento de 20 GB y despliegue en modalidad Single-AZ. Asimismo, se utilizó un modelo de precios bajo demanda sin habilitar opciones adicionales como proxy o monitoreo avanzado, manteniendo las configuraciones por defecto no especificadas en la actividad.

## 9. Configuración de Amazon RDS MySQL para la región Norte de Virginia

Se añadió a la estimación el servicio Amazon RDS MySQL para la región US East (N. Virginia), configurando una instancia de tipo db.t3.micro con un nodo, almacenamiento de 20 GB y despliegue Single-AZ. Se mantuvieron las mismas condiciones definidas para la región España con el objetivo de realizar una comparativa de costes consistente entre ambas regiones.

## 10. Configuración de Amazon S3 para la región España

Se configuró el servicio Amazon S3 en la región Europa (España), utilizando la clase de almacenamiento estándar con un volumen de 100 GB mensuales. Además, se estimaron 1000 solicitudes de tipo PUT, COPY, POST y LIST, junto con 50000 solicitudes GET. Se consideró también una transferencia de salida a Internet de 1000 GB mensuales, manteniendo el resto de configuraciones con valores por defecto.

## 11. Configuración de Amazon S3 para la región Norte de Virginia

Se configuró el servicio Amazon S3 en la región US East (N. Virginia), utilizando la clase de almacenamiento estándar con un volumen de 100 GB mensuales. Además, se estimaron 1000 solicitudes de tipo PUT, COPY, POST y LIST, junto con 50000 solicitudes GET. Se consideró también una transferencia de salida a Internet de 1000 GB mensuales, manteniendo el resto de configuraciones con valores por defecto.

## 12. Comparación de costos entre regiones España y Norte de Virginia

Se realizó una comparación de los costos estimados utilizando AWS Pricing Calculator para dos regiones: Europa (España) y US East (N. Virginia). Se consideraron los servicios EC2, RDS y S3 con configuraciones equivalentes en ambas regiones. Se observó el costo mensual total para cada región y se calculó el costo anual multiplicando el valor mensual por 12, con el fin de analizar las diferencias de precio entre ambas ubicaciones.

### Resultados

- Región EE.UU (N. Virginia):
  - Costo mensual: 24.63 USD
  - Costo anual: 295.56 USD

- Región España:
  - Costo mensual: 27.79 USD
  - Costo anual: 333.48 USD

### Tabla comparativa

| Región | Costo mensual | Costo anual |
|--------|--------------|------------|
| EE.UU (N. Virginia) | 24.63 USD | 295.56 USD |
| España | 27.79 USD | 333.48 USD |

## 13. Análisis comparativo de costos entre regiones

A partir de los resultados obtenidos en AWS Pricing Calculator, se realizó un análisis comparativo entre las regiones Europa (España) y US East (N. Virginia). Se observó que la región de Norte de Virginia presenta un menor costo mensual aproximado (24.63 USD) en comparación con la región de España (27.79 USD). Esta diferencia se debe principalmente a variaciones en los precios de los servicios EC2 y RDS entre regiones, mientras que el servicio S3 mantiene un costo similar en ambos casos. En términos anuales, esta diferencia se amplifica, lo que hace que la región de Norte de Virginia sea más conveniente desde el punto de vista económico.

## Conclusión

A partir de la estimación realizada, se evidencia que existen diferencias de costo entre regiones de AWS aun utilizando configuraciones equivalentes. En este caso, la región de Norte de Virginia resulta más económica que la región de España, lo que demuestra la importancia de seleccionar adecuadamente la ubicación de los recursos en la nube para optimizar costos sin afectar la funcionalidad. Esta diferencia se explica porque la región de Norte de Virginia es una de las más utilizadas y con mayor infraestructura dentro de AWS, lo que permite ofrecer precios más competitivos en comparación con otras regiones.

### Estimación AWS Calculator

https://calculator.aws/#/estimate?id=74313b3e9c691b9412c98ae6dd6623cd947ce83d


