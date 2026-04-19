# Tarea 7 - Parte 1: AWS CLI + EC2 + Security Group

> ⚠️ Nota:
> Los comandos presentados en este documento utilizan variables de entorno (por ejemplo: `$INSTANCE_ID`, `$SG_ID`, `$SUBNET_ID`) para mejorar la reutilización y buenas prácticas en AWS CLI.
>
> En las capturas de evidencia se observan valores reales generados por AWS durante la ejecución (IDs de recursos), los cuales corresponden correctamente a dichas variables.

---

## Paso 1. Inicio del laboratorio AWS Academy

Se inició el laboratorio en AWS Academy y se verificó que la sesión quedara activa correctamente, mostrando el indicador de AWS en color verde antes de comenzar la recreación de infraestructura mediante CLI.

**Comandos ejecutados:**  
N/A

**Captura:** `01-start-lab-aws-verde.png`

---

## Paso 2. Apertura de CloudShell

Se abrió AWS CloudShell desde la consola web de AWS para ejecutar los comandos de la CLI directamente en el entorno del laboratorio.

**Comandos ejecutados:**  
N/A

**Captura:** `02-cloudshell-abierto.png`

---

## Paso 3. Validación de AWS CLI

Se comprobó que AWS CLI estaba disponible en CloudShell mediante el comando `aws --version`. Además, se validó la identidad de la sesión con `aws sts get-caller-identity` y se revisó la región de trabajo con `aws configure list`, dejando configurada la región `us-east-1`.

**Comandos ejecutados:**
```bash
aws --version
aws sts get-caller-identity
aws configure list
```

**Captura:** `03-validacion-aws-cli.png`

---

## Paso 4. Obtención de la IP pública del administrador

Se obtuvo la dirección IP pública del equipo administrador con el comando `curl -s ifconfig.me`, para usarla posteriormente en la regla SSH del grupo de seguridad.

**Comandos ejecutados:**
```bash
MY_IP=$(curl -s ifconfig.me)
echo $MY_IP
```

**Captura:** `04-ip-administrador-cli.png`

---

## Paso 5. Identificación de la VPC y subnet por defecto

Se obtuvo mediante AWS CLI la VPC por defecto del entorno del laboratorio utilizando el comando `describe-vpcs`. Posteriormente, se identificó una subnet por defecto asociada a dicha VPC mediante el comando `describe-subnets`, la cual será utilizada para desplegar la instancia EC2.

**Comandos ejecutados:**
```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text)

echo $VPC_ID

SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" "Name=default-for-az,Values=true" \
  --query "Subnets[0].SubnetId" \
  --output text)

echo $SUBNET_ID
```

**Captura:** `05-vpc-y-subnet-por-defecto.png`

---

## Paso 6. Creación del grupo de seguridad

Se creó mediante AWS CLI un grupo de seguridad denominado `actividad1-sg-cli` dentro de la VPC por defecto. Este grupo de seguridad será utilizado para definir las reglas de acceso a la instancia EC2.

**Comandos ejecutados:**
```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name actividad1-sg-cli \
  --description "SG CLI actividad 1 EC2" \
  --vpc-id $VPC_ID \
  --query "GroupId" \
  --output text)

echo $SG_ID
```

**Captura:** `06-security-group-creado-cli.png`

---

## Paso 7. Configuración de reglas del grupo de seguridad

En este paso se configuraron las reglas de entrada del grupo de seguridad creado previamente para la instancia EC2, cumpliendo con los requisitos de la actividad: acceso SSH solo desde IPs autorizadas y acceso público a los puertos HTTP y HTTPS.

**Comandos ejecutados:**
```bash
echo $SG_ID

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=100.54.9.131/32,Description='SSH admin local'}]"

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=83.138.41.161/32,Description='SSH admin remoto'}]"

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,IpRanges="[{CidrIp=0.0.0.0/0,Description='HTTP publico'}]"

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=443,ToPort=443,IpRanges="[{CidrIp=0.0.0.0/0,Description='HTTPS publico'}]"

aws ec2 describe-security-groups \
  --group-ids $SG_ID \
  --query "SecurityGroups[0].IpPermissions" \
  --output table
```

**Capturas:**
- `07-1reglas-security-group-cli.png`
- `07-2reglas-security-group-cli.png`
- `07-3reglas-security-group-cli.png`
- `07-4reglas-security-group-cli.png`

---

## Paso 8. Identificación de la AMI para EC2

Se obtuvo mediante AWS CLI una imagen de Amazon Linux 2023 compatible con arquitectura `x86_64`, utilizando el comando `describe-images`. Se seleccionó la versión más reciente disponible para lanzar la instancia EC2 conforme a los requisitos de la actividad.

**Comandos ejecutados:**
```bash
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-2023.*-x86_64" "Name=architecture,Values=x86_64" "Name=state,Values=available" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text)

echo $AMI_ID

aws ec2 describe-images \
  --image-ids $AMI_ID \
  --query "Images[0].[ImageId,Name,CreationDate]" \
  --output table
```

**Captura:** `08-ami-amazon-linux-cli.png`

---

## Paso 9. Creación de la instancia EC2

Se creó una instancia EC2 mediante AWS CLI utilizando una imagen de Amazon Linux 2023, tipo `t3.small`, dentro de la VPC por defecto. Se asoció el grupo de seguridad previamente configurado, se asignó una IP pública y se configuró un volumen raíz de 10 GiB tipo `gp2`. Finalmente, se etiquetó la instancia con el nombre `actividad1`.

**Comandos ejecutados:**
```bash
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.small \
  --security-group-ids $SG_ID \
  --subnet-id $SUBNET_ID \
  --associate-public-ip-address \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":10,"VolumeType":"gp2","DeleteOnTermination":true}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=actividad1}]' \
  --query "Instances[0].InstanceId" \
  --output text)

echo $INSTANCE_ID
```

**Captura:** `09-ec2-creada-cli.png`

---

## Paso 10. Verificación de la instancia EC2

Se esperó mediante AWS CLI a que la instancia alcanzara el estado `running` utilizando el comando `wait instance-running`. Posteriormente, se consultó su información para obtener la IP pública y confirmar que la instancia fue desplegada correctamente.

**Comandos ejecutados:**
```bash
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "Instancia en running"

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]" \
  --output table
```

**Captura:** `10-ec2-running-ip-publica-cli.png`

---

## Paso 11. Verificación en consola AWS

Se validó en la consola web de AWS que la instancia EC2 creada mediante CLI se encuentra en estado `running`, confirmando su correcta creación y configuración.

**Comandos ejecutados:**  
N/A

**Captura:** `11-verificacion-ec2-consola.png`

---

## Paso 12. Detención de la instancia

Se detuvo la instancia EC2 al finalizar la sesión para evitar consumo innecesario de recursos en AWS Academy, manteniendo la infraestructura disponible para continuar en futuras sesiones.

**Comandos ejecutados:**
```bash
aws ec2 stop-instances --instance-ids $INSTANCE_ID

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name]" \
  --output table
```

**Captura:** `12-ec2-detenida.png`

---

## Paso 13. Inicio de la instancia para continuar la configuración

Se configuró nuevamente la variable de entorno correspondiente al identificador de la instancia EC2. Posteriormente, se ejecutó el comando para iniciar la instancia mediante AWS CLI y se verificó su estado `running`, junto con la obtención de su dirección IP pública para continuar con la configuración del servidor.

**Comandos ejecutados:**
```bash
INSTANCE_ID=i-0a1145a208156da57

aws ec2 start-instances --instance-ids $INSTANCE_ID

aws ec2 wait instance-running --instance-ids $INSTANCE_ID

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]" \
  --output table
```

**Captura:** `13-ec2-iniciada-y-ip-cli.png`

---

## Paso 13.1. Recarga de variables actuales de trabajo

Antes de continuar con la recreación de la infraestructura, se validaron nuevamente los valores actuales de la sesión para evitar errores por cambios de IP, reinicio del laboratorio o pérdida de variables de entorno en CloudShell.

**Comandos ejecutados:**
```bash
MY_IP=$(curl -s ifconfig.me)
echo $MY_IP

aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=actividad1" "Name=instance-state-name,Values=running,stopped,pending,stopping" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress]" \
  --output table

INSTANCE_ID=i-0a1145a208156da57
echo $INSTANCE_ID

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,SecurityGroups[0].GroupId]" \
  --output table
```

**Captura:** `13-1-recarga-variables-actuales-cli.png`

---

## Paso 14. Creación y asociación de volumen EBS adicional

Se creó un volumen EBS adicional de 5 GiB en la misma zona de disponibilidad de la instancia EC2 y se adjuntó correctamente al dispositivo `/dev/sdb`, cumpliendo el requisito de almacenamiento adicional solicitado en la actividad.

**Comandos ejecutados:**
```bash
AZ=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].Placement.AvailabilityZone" \
  --output text)

echo $AZ

VOLUME_ID=$(aws ec2 create-volume \
  --availability-zone $AZ \
  --size 5 \
  --volume-type gp2 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=actividad1-ebs-cli}]' \
  --query "VolumeId" \
  --output text)

echo $VOLUME_ID

aws ec2 wait volume-available --volume-ids $VOLUME_ID

aws ec2 attach-volume \
  --volume-id $VOLUME_ID \
  --instance-id $INSTANCE_ID \
  --device /dev/sdb

aws ec2 describe-volumes \
  --volume-ids $VOLUME_ID \
  --query "Volumes[0].[VolumeId,Size,State,Attachments[0].Device]" \
  --output table
```

**Captura:** `14-ebs-creado-y-adjuntado-cli.png`

---

## Paso 15. Asociación de Elastic IP

Se reservó y asoció una Elastic IP a la instancia EC2 mediante AWS CLI, asegurando que la dirección IP pública permanezca fija y no cambie tras reinicios, cumpliendo el requisito de la actividad.

**Comandos ejecutados:**
```bash
ALLOC_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --query "AllocationId" \
  --output text)

echo $ALLOC_ID

aws ec2 associate-address \
  --instance-id $INSTANCE_ID \
  --allocation-id $ALLOC_ID

aws ec2 describe-addresses \
  --allocation-ids $ALLOC_ID \
  --query "Addresses[0].[PublicIp,AllocationId,InstanceId]" \
  --output table
```

**Captura:** `15-elastic-ip-asociada-cli.png`

---

## Paso 15.1. Verificación del key pair de la instancia inicial

Se verificó que la instancia EC2 creada inicialmente no tenía un key pair asociado, lo que impedía la conexión SSH.

**Comandos ejecutados:**
```bash
aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,KeyName]" \
  --output table
```

**Captura:** `15-1-verificacion-keypair-cli.png`

---

## Paso 15.2. Listado de key pairs disponibles

Se listaron los key pairs disponibles en la cuenta para identificar uno válido que permitiera recrear la instancia con acceso SSH.

**Comandos ejecutados:**
```bash
aws ec2 describe-key-pairs \
  --query "KeyPairs[*].KeyName" \
  --output table
```

**Captura:** `15-2-listado-keypairs-cli.png`

---

## Paso 15.3. Recreación de instancia EC2 con Key Pair

Se recreó la instancia EC2 utilizando un key pair válido, permitiendo el acceso SSH y corrigiendo la limitación de la instancia anterior que no tenía clave asociada.

**Comandos ejecutados:**
```bash
SUBNET_ID=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].SubnetId" \
  --output text)

SG_ID=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" \
  --output text)

NEW_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.small \
  --key-name vockey \
  --security-group-ids $SG_ID \
  --subnet-id $SUBNET_ID \
  --associate-public-ip-address \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":10,"VolumeType":"gp2","DeleteOnTermination":true}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=actividad1-cli}]' \
  --query "Instances[0].InstanceId" \
  --output text)

echo $NEW_INSTANCE_ID

aws ec2 wait instance-running --instance-ids $NEW_INSTANCE_ID

aws ec2 describe-instances \
  --instance-ids $NEW_INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]" \
  --output table
```

**Captura:** `15-3-ec2-creada-con-keypair-cli.png`

---

## Paso 15.4. Verificación del key pair en la nueva instancia

Se verificó que la nueva instancia EC2 fue creada correctamente con un key pair asociado, condición necesaria para habilitar el acceso SSH.

**Comandos ejecutados:**
```bash
aws ec2 describe-instances \
  --instance-ids $NEW_INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,KeyName,State.Name,PublicIpAddress]" \
  --output table
```

**Captura:** `15-4-verificacion-keypair-nueva-ec2-cli.png`

---

## Paso 15.5. Reasociación de Elastic IP a la nueva instancia

Se reasoció la Elastic IP previamente reservada hacia la nueva instancia EC2 creada con key pair, garantizando conectividad estable para el acceso remoto y web.

**Comandos ejecutados:**
```bash
aws ec2 describe-addresses \
  --allocation-ids $ALLOC_ID \
  --query "Addresses[0].[PublicIp,AllocationId,InstanceId,AssociationId]" \
  --output table

aws ec2 associate-address \
  --instance-id $NEW_INSTANCE_ID \
  --allocation-id $ALLOC_ID \
  --allow-reassociation

aws ec2 describe-addresses \
  --allocation-ids $ALLOC_ID \
  --query "Addresses[0].[PublicIp,AllocationId,InstanceId]" \
  --output table
```

**Captura:** `15-5-reasociacion-elastic-ip-nueva-ec2-cli.png`

---

## Paso 15.6. Actualización de variable de trabajo

Se actualizó la variable `INSTANCE_ID` para continuar la configuración utilizando la nueva instancia EC2 creada correctamente con key pair.

**Comandos ejecutados:**
```bash
INSTANCE_ID=$NEW_INSTANCE_ID
echo $INSTANCE_ID

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress,KeyName]" \
  --output table
```

**Captura:** `15-6-actualizacion-instance-id-cli.png`

---

## Paso 16.1. Actualización de regla SSH en el grupo de seguridad

Se actualizó la regla de acceso SSH del grupo de seguridad para permitir conexión desde la IP pública actual del administrador, corrigiendo el error de timeout en el puerto 22.

**Comandos ejecutados:**
```bash
MY_IP=$(curl -s ifconfig.me)
echo $MY_IP

SG_ID=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" \
  --output text)

echo $SG_ID

aws ec2 describe-security-groups \
  --group-ids $SG_ID \
  --query "SecurityGroups[0].IpPermissions[?FromPort==`22`]" \
  --output table

aws ec2 revoke-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=100.54.9.131/32}]"

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=$MY_IP/32,Description='SSH admin local actual'}]"

aws ec2 describe-security-groups \
  --group-ids $SG_ID \
  --query "SecurityGroups[0].IpPermissions[?FromPort==`22`]" \
  --output table
```

**Captura:** `16-1-actualizacion-regla-ssh-cli.png`

---

## Paso 16.2. Reintento de conexión SSH

Tras actualizar la regla SSH del grupo de seguridad, se reintentó la conexión remota a la instancia EC2 utilizando la Elastic IP y el archivo de clave privada.

**Comandos ejecutados:**
```bash
chmod 400 labsuser.pem

ssh -i labsuser.pem ec2-user@54.162.200.212
```

**Captura:** `16-2-reintento-conexion-ssh-cli.png`

---

## Paso 17. Verificación de discos en la instancia

Se verificaron los discos visibles dentro del sistema operativo para identificar correctamente el volumen raíz y el volumen EBS adicional adjuntado a la instancia.

**Comandos ejecutados:**
```bash
lsblk

sudo fdisk -l
```

**Captura:** `17-verificacion-discos-ec2.png`

---

## Paso 18. Reasociación del volumen EBS a la nueva instancia

Se desasoció el volumen EBS de 5 GiB de la instancia anterior y se asoció a la nueva instancia creada mediante AWS CLI.

**Comandos ejecutados:**
```bash
aws ec2 detach-volume \
  --volume-id vol-0c6dff2abf40a7fbb

aws ec2 describe-volumes \
  --volume-ids vol-0c6dff2abf40a7fbb \
  --query "Volumes[0].State"

aws ec2 attach-volume \
  --volume-id vol-0c6dff2abf40a7fbb \
  --instance-id i-05304b7db11058ae5 \
  --device /dev/sdb

aws ec2 describe-volumes \
  --volume-ids vol-0c6dff2abf40a7fbb \
  --query "Volumes[0].Attachments"
```

**Captura:** `18-mover-volumen-ebs.png`

---

## Paso 19. Verificación del volumen EBS en la nueva instancia

Se verificó desde la nueva instancia EC2 que el volumen EBS reasociado ya se encuentra visible en el sistema operativo para continuar con su preparación.

**Comandos ejecutados:**
```bash
ssh -i labsuser.pem ec2-user@54.162.200.212

lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
```

**Captura:** `19-verificacion-volumen-ebs-en-ec2.png`

---

## Paso 20. Formateo del volumen EBS adicional

Se formateó el volumen EBS adicional de 5 GiB para poder utilizarlo como almacenamiento persistente dentro de la instancia.

**Comandos ejecutados:**
```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

**Captura:** `20-formateo-volumen-ebs-correcto.png`

---

## Paso 21. Montaje del volumen EBS

Se montó el volumen EBS adicional en el sistema de archivos para su uso como almacenamiento dentro de la instancia.

**Comandos ejecutados:**
```bash
sudo mkdir /mnt/ebs

sudo mount /dev/nvme1n1 /mnt/ebs

df -h
```

**Captura:** `21-volumen-ebs-montado.png`

---

## Paso 22. Configuración de montaje persistente (fstab)

Se configuró el archivo `fstab` para montar automáticamente el volumen EBS en cada reinicio y se dejó preparada su validación.

**Comandos ejecutados:**
```bash
sudo blkid

sudo nano /etc/fstab

sudo umount /mnt/ebs

sudo mount -a

df -h
```

**Captura:** `22-validacion-fstab.png`

---

## Paso 23. Validación final del volumen EBS

Se verificó que el volumen EBS se monta correctamente y que el sistema puede montarlo automáticamente mediante la configuración en `/etc/fstab`.

**Comandos ejecutados:**
```bash
sudo mount /dev/nvme1n1 /mnt/ebs
df -h

sudo umount /mnt/ebs
sudo mount -a
df -h
```

**Captura:** `23-validacion-final-ebs.png`

---

## Conclusión final

Se logró implementar y administrar una infraestructura EC2 completa utilizando AWS CLI, incluyendo configuración de red, seguridad, dirección IP elástica, almacenamiento persistente con EBS y automatización del montaje mediante `fstab`.

Además, durante el proceso se resolvieron incidencias reales relacionadas con la ausencia de key pair, actualización de reglas SSH, reasociación de volúmenes y validación de persistencia, fortaleciendo la comprensión práctica del entorno AWS.

# Tarea 7 - Parte 2: AWS CLI + RDS MySQL 

### Paso 1. Verificación del entorno CLI

Se verificó que la sesión de AWS CLI del laboratorio estuviera activa correctamente y que la región de trabajo fuera `us-east-1`, ya que AWS Academy limita el uso a regiones de Estados Unidos y la actividad se viene realizando en Norte de Virginia.

### Paso 2. Identificación de la VPC por defecto

Se consultó mediante AWS CLI la VPC por defecto de la cuenta para reutilizar la misma red donde ya se encuentra la infraestructura previa de la actividad.

### Paso 3. Obtención del grupo de seguridad de la EC2

Se identificó por CLI el grupo de seguridad asociado a la instancia EC2 `actividad1`, ya que la actividad exige permitir el acceso a la base de datos desde la máquina que aloja el servidor web Apache.

### Paso 4. Creación del grupo de seguridad para RDS

Se creó un nuevo grupo de seguridad específico para la instancia RDS MySQL, con el fin de controlar de forma separada el acceso a la base de datos.

### Paso 5. Acceso al RDS desde la IP del administrador

Se obtuvo la IP pública actual del entorno de administración mediante `curl ifconfig.me` y se configuró correctamente una regla de entrada al puerto `3306` en el grupo de seguridad de la base de datos RDS, restringiendo el acceso únicamente a dicha IP mediante notación CIDR `/32`.

### Paso 6. Acceso al RDS desde la instancia EC2

Se añadió una regla de entrada al puerto `3306` para permitir la conexión desde la IP privada de la instancia EC2 `actividad1`, cumpliendo con el requisito de conectividad entre el servidor web Apache y la base de datos MySQL.

### Paso 7. Validación del grupo de seguridad del RDS

Se verificó la configuración del grupo de seguridad asociado a la base de datos RDS, confirmando que el puerto `3306` quedó habilitado únicamente para la IP del administrador y para la IP privada de la instancia EC2.

### Paso 8. Consulta de subredes para RDS

Se listaron las subredes disponibles dentro de la VPC por defecto con el objetivo de seleccionar las necesarias para la creación del grupo de subredes de Amazon RDS.

### Paso 9. Creación del DB Subnet Group

Se creó el grupo de subredes para la instancia RDS utilizando subredes pertenecientes a distintas zonas de disponibilidad de la región `us-east-1`, permitiendo el correcto despliegue de la base de datos dentro de la VPC.

### Paso 10. Creación de la instancia RDS MySQL

Se creó la instancia RDS MySQL mediante AWS CLI utilizando la versión `5.7.44-rds.20260212`, correspondiente a una versión válida disponible en Amazon RDS. Se configuró con clase `db.t3.micro`, almacenamiento de `20 GiB gp2`, cifrado en reposo habilitado, sin Multi-AZ, con backups automáticos a las `03:00 UTC` con retención de `10 días` y ventana de mantenimiento a las `04:00 UTC`.

### Paso 11. Espera de aprovisionamiento

Se utilizó el comando `wait` de AWS CLI para esperar a que la instancia RDS completara su creación y alcanzara el estado disponible.

### Paso 12. Validación final del RDS

Se realizó la validación final de la instancia RDS MySQL mediante AWS CLI, comprobando su identificador, estado `available`, motor y versión, endpoint de conexión, puerto `3306`, clase `db.t3.micro`, almacenamiento de `20 GiB`, cifrado en reposo habilitado, ausencia de Multi-AZ y retención de backups de `10 días`.

# Tarea 7 - Parte 3: AWS CLI + Bucket S3

### Paso 1. Verificación del entorno CLI

Se definieron variables en la CLI para reutilizar el nombre del bucket y la región solicitada por la actividad. En este caso, el bucket debe crearse en `us-east-1` y su nombre debe seguir el formato `xxyyzz-actividad1-08masw`.

---

### Paso 2. Crear Bucket S3

Se creó el bucket en la región `us-east-1` mediante AWS CLI y se verificó su existencia utilizando un comando de listado filtrado.

---

### Paso 3. Configuración de cifrado SSE-S3 en bucket

Se configuró el cifrado por defecto del bucket S3 utilizando Server-Side Encryption con claves administradas por Amazon S3 (SSE-S3).

El algoritmo aplicado corresponde a AES256, lo que garantiza que todos los objetos almacenados en el bucket serán cifrados automáticamente.

---

### Paso 4. Creación de archivos PDF y JPG en CloudShell

Se crearon dos archivos de prueba en la consola CloudShell: un documento PDF y una imagen JPG.

Estos archivos simulan los recursos solicitados por la actividad, los cuales posteriormente serán almacenados en el bucket S3 y expuestos públicamente.

---

### Paso 5. Subida de archivos al bucket S3

Se subieron al bucket S3 los archivos de prueba (PDF y JPG) utilizando comandos de AWS CLI.

Posteriormente, se verificó la carga correcta de los objetos mediante el comando:

```bash
aws s3 ls "s3://$BUCKET/"
```

---

### Paso 6. Eliminación del bloqueo de acceso público del bucket S3

Para permitir la descarga pública de los objetos almacenados en el bucket, se eliminó la configuración de bloqueo de acceso público mediante AWS CLI.

```bash
aws s3api delete-public-access-block --bucket "$BUCKET"
aws s3api get-public-access-block --bucket "$BUCKET"
```

El mensaje `NoSuchPublicAccessBlockConfiguration` confirma que el bloqueo fue eliminado correctamente.

---

### Paso 7. Aplicación de política pública de lectura en el bucket S3

Se creó una política de bucket para permitir la descarga pública de los objetos almacenados.

```bash
cat > bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::$BUCKET/*"
    }
  ]
}
EOF
```

Luego se aplicó al bucket:

```bash
aws s3api put-bucket-policy \
  --bucket "$BUCKET" \
  --policy file://bucket-policy.json
```

---

### Paso 8. Validación de acceso público a los objetos del bucket S3

Se generaron las URLs públicas de los archivos almacenados:

```bash
PDF_URL="https://$BUCKET.s3.$REGION.amazonaws.com/ejemplo.pdf"
JPG_URL="https://$BUCKET.s3.$REGION.amazonaws.com/ejemplo.jpg"

echo $PDF_URL
echo $JPG_URL
```

Posteriormente, se validó el acceso público mediante:

```bash
curl -I "$PDF_URL"
curl -I "$JPG_URL"
```

La respuesta `HTTP/1.1 200 OK` confirmó que los archivos son accesibles públicamente.

---

### URLs públicas

- PDF: https://xxyyzz-actividad1-08masw.s3.us-east-1.amazonaws.com/ejemplo.pdf
- Imagen: https://xxyyzz-actividad1-08masw.s3.us-east-1.amazonaws.com/ejemplo.jpg















