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
