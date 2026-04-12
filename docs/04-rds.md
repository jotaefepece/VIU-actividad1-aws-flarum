# Tarea 4 - Base de Datos RDS (MySQL)

## Objetivo

Desplegar una base de datos en **Amazon RDS** (MySQL 5.x) para almacenar 
la información de la aplicación Flarum, garantizando seguridad, conectividad y 
persistencia de datos.

---

## Creación de la base de datos

Se accede al servicio **RDS** desde la consola de AWS.

Evidencia
 
![Inicio en RDS](../images/rds/01-inicio-en-rds.png)

Se ingresa a la opción de creación de base de datos.

Evidencia
 
![Crear Base de Datos](../images/rds/02-en-sección-crear-base-de-datos.png)

Se selecciona el motor **MySQL** requerido por la actividad.

Evidencia
 
![Selección MySQL](../images/rds/03-selección-motor-MySQL.png)

Se configuran las credenciales de acceso para el usuario administrador.

Evidencia
 
![Configuración de credenciales](../images/rds/04-identificación-db-passgametix2026.png)

Finalmente, se crea la base de datos correctamente.

Evidencia
  
![DB creada](../images/rds/05-base-de-dato-creada.png)

---

## Preparación de la instancia EC2

Se instala el cliente **MariaDB** para permitir la conexión desde la instancia EC2 hacia el endpoint de RDS.

Evidencia
 
![Instalación cliente MariaDB](../images/rds/06-instalación-cliente-mariadb.png)

```bash
sudo dnf install mariadb105 -y
```

---

## Configuración de seguridad

Se accede a la edición de las reglas de entrada del **Security Group** de la base de datos.

Evidencia
 
![Editar reglas de entrada](../images/rds/07-ingreso-a-editar-reglas-de-entrada.png)

Se habilita el puerto **3306** para MySQL.

Evidencia
 
![Habilitar puerto 3306](../images/rds/08-permitiendo-acceso-para-MySQL-Aurora.png)

Se permite el acceso desde la instancia EC2 (seleccionando su Security Group previo).

Evidencia
 
![Acceso desde EC2](../images/rds/09-permitiendo-acceso-para-MySQL-seguridad-EC2-previo.png)

---

## Conexión a la base de datos

Se realiza la conexión desde la instancia EC2 usando el cliente MySQL y el endpoint proporcionado por RDS.

Evidencia
 
![Conexión por consola](../images/rds/10-ingreso-a-mysql-por-consola.png)

---

## Creación de base de datos y usuario

Se crea la base de datos y el usuario específico con los permisos necesarios para el funcionamiento de Flarum.

Evidencia
 
![Creación DB y Usuario](../images/rds/11-creando-usuario-y-db-flarum.png)

---

## Integración con Flarum

Se ingresan los datos de conexión (Host, Database, User, Password) en el instalador web de Flarum, logrando desplegar la aplicación correctamente.

Evidencia
 
![Flarum conectado](../images/rds/12-luego-de-ingresar-parametros-de-config-se-abre-el-foro-flarum.png)

- Aplicación conectada exitosamente a RDS.

---

## Verificación del funcionamiento

Se realiza un post de prueba dentro del foro para validar la escritura en la base de datos.

Evidencia
 
![Post de prueba](../images/rds/13-foro-funcionando-posteo-integrantes-grupo.png)

---

## Acceso adicional para administración

Se añade una regla de entrada adicional para permitir el acceso directo a la base de datos desde la IP del administrador (gestión remota).

Evidencia
 
![Acceso administrador](../images/rds/14-agregando-regla-de-entrada-Mi-IP-a-Mysql.png)

---

## Configuración de backups

Se configuran las copias de seguridad automáticas con un periodo de retención de **10 días**.

Evidencia
  
![Configuración de backups](../images/rds/15-configurando-backups-10-días.png)

---

## Configuración de mantenimiento

Se define la ventana de mantenimiento programado para asegurar la estabilidad del servicio.

Evidencia
 
![Ventana de mantenimiento](../images/rds/16-configurando-mantenimiento-programado.png)

---

## Resumen de configuración

Se visualiza el resumen de todas las configuraciones aplicadas a la instancia de base de datos antes de finalizar.

Evidencia
 
![Resumen RDS](../images/rds/16-resumen-modificaciones-db.png)

---

## Resultado final

La aplicación Flarum se encuentra completamente operativa y conectada a la base de datos RDS, con todas las políticas de seguridad y mantenimiento activas.

Evidencia
 
![Resultado final ampliado](../images/rds/17-cierre-presentación-ampliada-foro-funcionando.png)

---

## Conclusión

Se ha implementado correctamente la base de datos en Amazon RDS cumpliendo todos los requisitos de la actividad:
- [x] Motor MySQL 5.x.
- [x] Acceso restringido mediante Security Groups.
- [x] Conectividad bidireccional con EC2.
- [x] Configuración de backups automáticos (10 días).
- [x] Ventana de mantenimiento definida.
- [x] Integración completa con el despliegue de Flarum.

---

## Volver al índice general

Acceder al README principal de la actividad1 desde aquí:

🔙 [Volver al README](../README.md)
