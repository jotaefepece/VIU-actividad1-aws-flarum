#  Tarea 3: Asociación de Elastic IP en EC2

## Objetivo

Asignar una dirección IP elástica a la instancia EC2 para evitar que la 
IP pública cambie tras reinicios, garantizando un acceso estable al servidor web.

---

## Creación de la Elastic IP

Se accede al servicio EC2 → sección **Elastic IPs**.

Evidencias:

![Elastic IPs](../images/ec2/51-dentro-de-direcciones-IP-elasticas.png)

---

## Asignación de nueva IP elástica

Se selecciona la opción por defecto de grupo fronterizo asignado.

Evidencias:

![Asignación Elastic IP](../images/ec2/52-asignando-direcciones-grupo-fronterizo.png)

- [x] Se mantiene la configuración por defecto.

---

Evidencias:

![Elastic IP creada](../images/ec2/53-muestra-asignación-ip-elástica-ok.png)

- [x] Dirección IP elástica creada correctamente.

---

## Asociación a la instancia EC2

Se asocia la IP a la instancia en ejecución:

Evidencias:

![Asociando IP](../images/ec2/54-asignando-a-instancia-en-ejecución.png)

---

Evidencias:

![IP asociada](../images/ec2/55-instancia-asociada-a-dirección-IP-elástica.png)

- [x] IP elástica vinculada correctamente.
- [x] La IP pública anterior deja de estar disponible.

---

## Validación de acceso

Se accede a la aplicación mediante la nueva IP:

Evidencias:

![Flarum con Elastic IP](../images/ec2/56-vista-de-flarum-desde-IP-elástica.png)

- [x] Aplicación Flarum accesible correctamente.
- [x] Servidor web funcionando sin cambios adicionales.

---

Evidencias:

![Configuración IP](../images/ec2/57-en-configuración-elástica-muestra-IP-elástica.png)

- [x] Confirmación desde interfaz de acceso a AWS, la IP activa.

---

## Persistencia tras reinicio

Se asegura que los servicios se mantengan tras reinicio.

Evidencias:

![Enable HTTPD](../images/ec2/58-dejando-servicio-http-para-recargar-desde-inicio.png)

```bash
sudo systemctl enable httpd
```
- [x] Apache configurado para iniciar automáticamente.

Evidencias:

![Reinicio de instancia](../images/ec2/59-reinicio-instancia-para-prueba-IP.png)

Se reinicia la instancia desde AWS para validar comportamiento.

Evidencias:

![Post-reinicio](../images/ec2/60-recarga-de-sitio-sin-problemas-con-IP-elástica.png)

- [x] La IP elástica se mantiene.
- [x] El servicio web sigue operativo.
- [x] No se requieren reconfiguraciones adicionales.

---

## Simulación de dominio

Se configura resolución local mediante archivo `/etc/hosts`.

Evidencias:

![Configuración dominio](../images/ec2/61-simulando-dominio-con-IP-elástica.png)

```text
<IP_ELASTICA> actividad1-08masw.universidadviu.com
```

Evidencias:

![Dominio operativo](../images/ec2/62-flarum-funcionando-con-dominio-simulado-ip-elástica.png)

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

🔙 [Volver al README](../README.md)
