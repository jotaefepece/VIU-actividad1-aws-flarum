#  Tarea 3: Asociación de Elastic IP en EC2

## Objetivo

Asignar una dirección IP elástica a la instancia EC2 para evitar que la 
IP pública cambie tras reinicios, garantizando un acceso estable al servidor web.

---

## Creación de la Elastic IP

Se accede al servicio EC2 → sección **Elastic IPs**.

 **Imagen 51**  
![Elastic IPs](../images/ec2/51-dentro-de-direcciones-IP-elasticas.png)

---

## Asignación de nueva IP elástica

Se selecciona la opción **Allocate Elastic IP address**.

 **Imagen 52**  
![Asignación Elastic IP](../images/ec2/52-asignando-direcciones-grupo-fronterizo.png)

- [x] Se mantiene la configuración por defecto.
- [x] Se selecciona el grupo fronterizo correspondiente.

---

 **Imagen 53**  
![Elastic IP creada](../images/ec2/53-muestra-asignación-ip-elástica-ok.png)

- [x] Dirección IP elástica creada correctamente.

---

## Asociación a la instancia EC2

Se asocia la IP a la instancia en ejecución:

 **Imagen 54**  
![Asociando IP](../images/ec2/54-asignando-a-instancia-en-ejecución.png)

---

 **Imagen 55**  
![IP asociada](../images/ec2/55-instancia-asociada-a-dirección-IP-elástica.png)

- [x] IP elástica vinculada correctamente.
- [x] La IP pública anterior deja de estar disponible.

---

## Validación de acceso

Se accede a la aplicación mediante la nueva IP:

 **Imagen 56**  
![Flarum con Elastic IP](../images/ec2/56-vista-de-flarum-desde-IP-elástica.png)

- [x] Aplicación Flarum accesible correctamente.
- [x] Servidor web funcionando sin cambios adicionales.

---

 **Imagen 57**  
![Configuración IP](../images/ec2/57-en-configuración-elástica-muestra-IP-elástica.png)

- [x] Confirmación desde consola AWS de la IP activa.

---

## Persistencia tras reinicio

Se asegura que los servicios se mantengan tras reinicio.

 **Imagen 58**  
![Enable HTTPD](../images/ec2/58-dejando-servicio-http-para-recargar-desde-inicio.png)

```bash
sudo systemctl enable httpd
```
- [x] Apache configurado para iniciar automáticamente.

 **Imagen 59**  
![Reinicio de instancia](../images/ec2/59-reiniciando-instancia.png)

Se reinicia la instancia desde AWS para validar comportamiento.

 **Imagen 60**  
![Post-reinicio](../images/ec2/60-después-de-reinicio-validación-de-IP.png)

- [x] La IP elástica se mantiene.
- [x] El servicio web sigue operativo.
- [x] No se requieren reconfiguraciones adicionales.

---

## Simulación de dominio

Se configura resolución local mediante archivo `/etc/hosts`.

 **Imagen 61**  
![Configuración dominio](../images/ec2/61-configuración-de-dominio-en-archivo-hosts.png)

```text
<IP_ELASTICA> actividad1-08masw.universidadviu.com
```

 **Imagen 62**  
![Dominio operativo](../images/ec2/62-acceso-vía-dominio-post-reinicio.png)

- [x] Dominio resuelto correctamente.
- [x] Acceso funcional mediante nombre de dominio.

---

## Consideraciones importantes

- Una Elastic IP permanece fija incluso tras reinicios.
- La IP pública anterior se pierde al asociar una Elastic IP.
- AWS puede generar costes si la IP no está asociada a una instancia activa.

---

## Conclusión

Se ha configurado correctamente una dirección IP elástica:
- [x] Acceso estable al servidor web.
- [x] Persistencia tras reinicio de la instancia.
- [x] Integración completa con la aplicación desplegada.

---

## Volver al índice general

Acceder al README principal de la actividad1 desde aquí:

👉 [Volver al README](../README.md)
