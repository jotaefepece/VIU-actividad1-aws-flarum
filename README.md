# Actividad 1 - Despliegue de Flarum en AWS

## Descripción

Esta actividad consiste en el despliegue completo de una aplicación web basada en **Flarum** utilizando 
servicios de **AWS**, siguiendo una arquitectura modular y documentando cada paso del proceso.

El objetivo es comprender la implementación de infraestructura en la nube, configuración de servicios 
y despliegue de aplicaciones modernas.

---

## Arquitectura general

La solución está compuesta por los siguientes componentes:

*   **Instancia EC2:** Amazon Linux 2023.
*   **Volumen EBS adicional:** Almacenamiento persistente.
*   **Servidor web:** Apache.
*   **Lenguaje/Entorno:** PHP 8+ y Composer.
*   **Aplicación:** Flarum.
*   **Base de datos:** RDS (Pendiente en tareas posteriores).
*   **Almacenamiento adicional:** S3 (Tarea posterior).

---

## Estructura del proyecto

```text
.
├── docs
│   ├── 01-ec2.md
│   ├── 02-security-group.md
│   ├── 03-elastic-ip.md
│   ├── 04-rds.md
│   ├── 05-s3.md
│   ├── 06-costes.md
│   ├── 07-cli.md
│   └── 08-conclusiones.md
├── images
│   ├── ec2
│   ├── rds
│   └── s3
└── README.md
```

---

## Desarrollo por tareas

A continuación se documenta cada una de las tareas realizadas:

### 🔹 [Tarea 1 - EC2 y despliegue de Flarum](docs/01-ec2.md)
*   Creación de instancia EC2.
*   Configuración de acceso SSH.
*   Configuración de volumen EBS.
*   Instalación de Apache, PHP y Composer.
*   Despliegue de Flarum.

 **Estado:** Completada  
 **Nota:** Instalación final pendiente de base de datos (ver Tarea 4).

### 🔹 [Tarea 2 - Security Groups](docs/02-security-group.md)
*   Acceso SSH restringido por IP.
*   Apertura del puerto 80 (HTTP).

 **Estado:** Pendiente

### 🔹 [Tarea 3 - Elastic IP](docs/03-elastic-ip.md)
*   Asignación de IP elástica.
*   Asociación a la instancia EC2.

 **Estado:** Pendiente

### 🔹 [Tarea 4 - Base de datos RDS](docs/04-rds.md)
*   Creación de instancia RDS (MySQL).
*   Configuración de acceso.
*   Integración con Flarum.

 **Estado:** Pendiente

### 🔹 [Tarea 5 - Almacenamiento S3](docs/05-s3.md)
*   Creación de bucket S3.
*   Configuración de acceso.
*   Uso para almacenamiento de archivos.

 **Estado:** Pendiente

### 🔹 [Tarea 6 - Costes](docs/06-costes.md)
*   Estimación de costes.
*   Uso de recursos AWS.

 **Estado:** Pendiente

### 🔹 [Tarea 7 - AWS CLI](docs/07-cli.md)
*   Uso de AWS CLI.
*   Automatización de tareas.

 **Estado:** Pendiente

### 🔹 [Tarea 8 - Conclusiones](docs/08-conclusiones.md)
*   Reflexión sobre el aprendizaje.
*   Evaluación del despliegue.

 **Estado:** ⏳ Pendiente

---

## Consideraciones importantes

*   El volumen EBS requiere montaje manual o configuración en `/etc/fstab` para persistencia tras reinicio.
*   Flarum requiere permisos adecuados para el usuario del servidor web (`apache`).
*   La instalación completa depende de la configuración de base de datos en RDS.

---

## Estado actual del proyecto

- [x] Infraestructura base desplegada.
- [x] Servidor web operativo.
- [x] Aplicación Flarum instalada.
- [ ] Configuración de acceso público (Security Groups).
- [ ] Base de datos (RDS).
- [ ] Integración completa de la aplicación.

---

## Objetivos de aprendizaje

*   Comprender el uso de EC2 y EBS.
*   Configurar servidores web en Linux.
*   Desplegar aplicaciones PHP modernas.
*   Integrar servicios gestionados de AWS.
*   Documentar procesos técnicos de forma estructurada.

---

## Autores

| Nombre          | GitHub                                         |
|-----------------|------------------------------------------------|
| Eduardo Criollo | [@eacriollo](https://github.com/eacriollo)     |
| José Poblete    | [@jotaefepece](https://github.com/jotaefepece) |
| Hubert Ferrer   | [@hferrer08](https://github.com/hferrer08)     |
