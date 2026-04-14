# Tarea 7 - Parte 1: AWS CLI + EC2 + Security Group

> ⚠️ Nota:
> Los comandos presentados en este documento utilizan variables de entorno (por ejemplo: $INSTANCE_ID, $SG_ID, $SUBNET_ID) para mejorar la reutilización y buenas prácticas en AWS CLI.
> 
> En las capturas de evidencia se observan valores reales generados por AWS durante la ejecución (IDs de recursos), los cuales corresponden correctamente a dichas variables.

## Paso 1. Inicio del laboratorio AWS Academy

Se inició el laboratorio en AWS Academy y se verificó que la sesión quedara activa correctamente, mostrando el indicador de AWS en color verde antes de comenzar la recreación de infraestructura mediante CLI.

**Comandos ejecutados:**  
N/A

---

## Paso 2. Apertura de CloudShell

Se abrió AWS CloudShell desde la consola web de AWS para ejecutar los comandos de la CLI directamente en el entorno del laboratorio.

**Comandos ejecutados:**  
N/A

---

## Paso 3. Validación de AWS CLI

Se comprobó que AWS CLI estaba disponible en CloudShell mediante el comando `aws --version`. Además, se validó la identidad de la sesión con `aws sts get-caller-identity` y se revisó la región de trabajo con `aws configure list`, dejando configurada la región `us-east-1`.

**Comandos ejecutados:**
```bash
aws --version
aws sts get-caller-identity
aws configure list
```

---

## Paso 4. Obtención de la IP pública del administrador

Se obtuvo la dirección IP pública del equipo administrador con el comando `curl -s ifconfig.me`, para usarla posteriormente en la regla SSH del grupo de seguridad.

**Comandos ejecutados:**
```bash
MY_IP=$(curl -s ifconfig.me)
echo $MY_IP
```

---

## Paso 5. Identificación de la VPC y subnet por defecto

Se obtuvo mediante AWS CLI la VPC por defecto del entorno del laboratorio utilizando el comando `describe-vpcs`. Posteriormente, se identificó una subnet por defecto asociada a dicha VPC mediante el comando `describe-subnets`, la cual será utilizada para desplegar la instancia EC2.

**Comandos ejecutados:**
```bash
VPC_ID=$(aws ec2 describe-vpcs   --filters "Name=isDefault,Values=true"   --query "Vpcs[0].VpcId"   --output text)

echo $VPC_ID

SUBNET_ID=$(aws ec2 describe-subnets   --filters "Name=vpc-id,Values=$VPC_ID" "Name=default-for-az,Values=true"   --query "Subnets[0].SubnetId"   --output text)

echo $SUBNET_ID
```

---

## Paso 6. Creación del grupo de seguridad

Se creó mediante AWS CLI un grupo de seguridad denominado `actividad1-sg-cli` dentro de la VPC por defecto. Este grupo de seguridad será utilizado para definir las reglas de acceso a la instancia EC2.

**Comandos ejecutados:**
```bash
SG_ID=$(aws ec2 create-security-group   --group-name actividad1-sg-cli   --description "SG CLI actividad 1 EC2"   --vpc-id $VPC_ID   --query "GroupId"   --output text)

echo $SG_ID
```

---

## Paso 7. Configuración de reglas del grupo de seguridad

En este paso se configuraron las reglas de entrada del grupo de seguridad creado previamente para la instancia EC2, cumpliendo con los requisitos de la actividad: acceso SSH solo desde IPs autorizadas y acceso público a los puertos HTTP y HTTPS.

### 7.1 Objetivo del paso

Se buscó dejar el grupo de seguridad preparado para:
- Permitir acceso SSH desde la IP pública del administrador.
- Permitir acceso SSH desde la IP remota indicada en el enunciado.
- Permitir acceso público a los puertos 80 y 443.
- Verificar finalmente todas las reglas configuradas.

### 7.2 Comandos ejecutados
```bash
echo $SG_ID

aws ec2 authorize-security-group-ingress   --group-id $SG_ID   --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=100.54.9.131/32,Description='SSH admin local'}]"

aws ec2 authorize-security-group-ingress   --group-id $SG_ID   --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges="[{CidrIp=83.138.41.161/32,Description='SSH admin remoto'}]"

aws ec2 authorize-security-group-ingress   --group-id $SG_ID   --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,IpRanges="[{CidrIp=0.0.0.0/0,Description='HTTP publico'}]"

aws ec2 authorize-security-group-ingress   --group-id $SG_ID   --ip-permissions IpProtocol=tcp,FromPort=443,ToPort=443,IpRanges="[{CidrIp=0.0.0.0/0,Description='HTTPS publico'}]"

aws ec2 describe-security-groups   --group-ids $SG_ID   --query "SecurityGroups[0].IpPermissions"   --output table
```

### 7.3 Descripción breve de cada comando
- `echo $SG_ID`: muestra en pantalla el identificador del grupo de seguridad creado previamente, para confirmar que la variable fue guardada correctamente.
- `authorize-security-group-ingress` para puerto `22` con `100.54.9.131/32`: permite acceso SSH únicamente desde la IP pública del administrador.
- `authorize-security-group-ingress` para puerto `22` con `83.138.41.161/32`: permite acceso SSH desde la IP remota especificada en el enunciado.
- `authorize-security-group-ingress` para puerto `80` con `0.0.0.0/0`: habilita acceso HTTP público desde cualquier dirección IP.
- `authorize-security-group-ingress` para puerto `443` con `0.0.0.0/0`: habilita acceso HTTPS público desde cualquier dirección IP.
- `describe-security-groups`: consulta y muestra en formato tabla las reglas del grupo de seguridad para verificar que la configuración quedó correcta.

### 7.4 Resultado del paso

Al finalizar, el grupo de seguridad quedó configurado con las reglas necesarias para administración remota segura por SSH y acceso web público por HTTP y HTTPS, manteniendo bloqueado el resto del tráfico entrante no autorizado.

---

## Paso 8. Identificación de la AMI para EC2

Se obtuvo mediante AWS CLI una imagen de Amazon Linux 2023 compatible con arquitectura `x86_64`, utilizando el comando `describe-images`. Se seleccionó la versión más reciente disponible para lanzar la instancia EC2 conforme a los requisitos de la actividad.

**Comandos ejecutados:**
```bash
AMI_ID=$(aws ec2 describe-images   --owners amazon   --filters "Name=name,Values=al2023-ami-2023.*-x86_64" "Name=architecture,Values=x86_64" "Name=state,Values=available"   --query "sort_by(Images, &CreationDate)[-1].ImageId"   --output text)

echo $AMI_ID

aws ec2 describe-images   --image-ids $AMI_ID   --query "Images[0].[ImageId,Name,CreationDate]"   --output table
```

---

## Paso 9. Creación de la instancia EC2

Se creó una instancia EC2 mediante AWS CLI utilizando una imagen de Amazon Linux 2023, tipo `t3.small`, dentro de la VPC por defecto. Se asoció el grupo de seguridad previamente configurado, se asignó una IP pública y se configuró un volumen raíz de 10 GiB tipo `gp2`. Finalmente, se etiquetó la instancia con el nombre `actividad1`.

**Comandos ejecutados:**
```bash
INSTANCE_ID=$(aws ec2 run-instances   --image-id $AMI_ID   --instance-type t3.small   --security-group-ids $SG_ID   --subnet-id $SUBNET_ID   --associate-public-ip-address   --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":10,"VolumeType":"gp2","DeleteOnTermination":true}}]'   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=actividad1}]'   --query "Instances[0].InstanceId"   --output text)

echo $INSTANCE_ID
```

---

## Paso 10. Verificación de la instancia EC2

Se esperó mediante AWS CLI a que la instancia alcanzara el estado `running` utilizando el comando `wait instance-running`. Posteriormente, se consultó su información para obtener la IP pública y confirmar que la instancia fue desplegada correctamente.

**Comandos ejecutados:**
```bash
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "Instancia en running"

aws ec2 describe-instances   --instance-ids $INSTANCE_ID   --query "Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]"   --output table
```

---

## Paso 11. Verificación en consola AWS

Se validó en la consola web de AWS que la instancia EC2 creada mediante CLI se encuentra en estado `running`, confirmando su correcta creación y configuración.

**Comandos ejecutados:**  
N/A

---

## Paso 12. Detención de la instancia

Se detuvo la instancia EC2 al finalizar la sesión para evitar consumo innecesario de recursos en AWS Academy, manteniendo la infraestructura disponible para continuar en futuras sesiones.

**Comandos ejecutados:**
```bash
aws ec2 stop-instances --instance-ids $INSTANCE_ID

aws ec2 describe-instances   --instance-ids $INSTANCE_ID   --query "Reservations[0].Instances[0].[InstanceId,State.Name]"   --output table
```
---

### Conclusión parcial

Se logró recrear mediante AWS CLI una infraestructura básica en AWS, incluyendo red, seguridad e instancia EC2, validando su funcionamiento tanto por consola como por comandos.