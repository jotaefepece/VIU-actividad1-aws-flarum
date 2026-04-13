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

### Estimación AWS
https://calculator.aws/#/estimate?id=31aa893fdf99d1d92628b9765fa3b0e40a0db5f5

