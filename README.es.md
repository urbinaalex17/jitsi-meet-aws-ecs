
<p align="center">
  <img src="https://img.s3wfg.com/web/img/images_uploaded/8/c/dominion_logo_620x350.jpg" />
</p>

# Jitsi Meet en Amazon Elastic Container Service

![](docs/jitsi-docker.png)

[Jitsi](https://jitsi.org/) es un conjunto de projectos de Codigo Abierto que te permite facilmente contruir e instalar soluciones de videoconferencia seguras.

[Jitsi Meet](https://jitsi.org/jitsi-meet/) es una solucion de video conferencia totalmente encriptada y 100% de codigo abierto que puedes usar todo el dia, todos los dias, gratis sin necesidad de una cuenta.

Este repositorio contiene los archivos necesarios para correr un stack de Jitsi Meet en [Docker](https://www.docker.com) usando [Docker Compose](https://docs.docker.com/compose/) asi como tambien las instrucciones para correr el stack en [Amazon ElasticContainer Services](https://aws.amazon.com/es/ecs/)

## Primeros pasos

### Prerequisitos

Debes instalar las siguientes herramientas: 
- [ecs-cli](https://docs.aws.amazon.com/es_es/AmazonECS/latest/developerguide/ECS_CLI_installation.html)
- [docker](https://docs.docker.com/get-docker)
- [docker-compose](https://docs.docker.com/compose/install)

Ya sea que uses una cuenta de servicio de IAM o un usuario de IAM, debes tener los siguientes permisos/roles:
- AmazonECS_FullAccess
- ElasticLoadBalancingFullAccess

### Configurando Elastic Container Service

[Generar una llave](https://docs.aws.amazon.com/es_es/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) en la consola de AWS y [configurar](https://docs.aws.amazon.com/es_es/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) ecs-cli para usarla.

Las siguientes variables de entorno seran configuradas en la shell:

| Nombre | Descripcion | Ejemplo |
| --- | --- | --- |
| PROFILE_NAME | Especifica el nombre del perfil a usar | aws |
| AWS_ACCESS_KEY_ID | Especifica el clave de acceso de AWS a usar | AKIAXKO5CYVQ3XXXXXXX |
| AWS_SECRET_ACCESS_KEY | Especifica la llave secreta de AWS a usar | KHvshFRp8SstYaou936ZtQD0IaaZXXXXXXXXXXXX |
| ECS_CLUSTER | Especifica el nombre del cluster ECS a usar | jitsi-cluster |
| AWS_REGION | Especifica la region  de AWS a usar | eu-west-3 |
| AWS_KEYPAIR | Especifica el nombre de un par de llaves existentes para habilitar acceso SSH a las instancias EC2 | jitsi-keypair | 
| AWS_INSTANCE_TYPE | Especifica el tipo de maquina EC2 para las instancias de contenedores | m5.xlarge |
| ECS_CLUSTER_SIZE | Especifica el numero de instancias a crear y registrar en el cluster | 2 |

Configurar las credenciales en ECS
```bash
export PROFILE_NAME=<replace>
export AWS_ACCESS_KEY_ID=<replace>
export AWS_SECRET_ACCESS_KEY=<replace>
ecs-cli configure profile --profile-name $PROFILE_NAME --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY
```
 
Configurar el cluster ECS 
```bash
export ECS_CLUSTER=jitsi-ecs
export AWS_REGION=eu-west-3
ecs-cli configure --cluster $ECS_CLUSTER --default-launch-type EC2 --region $AWS_REGION --config-name $ECS_CLUSTER
```

Crear el cluster ECS
```bash
export AWS_KEYPAIR=jitsi-ecs
export AWS_INSTANCE_TYPE=m5.xlarge
export ECS_CLUSTER_SIZE=1
ecs-cli up --keypair $AWS_KEYPAIR --capability-iam --size $ECS_CLUSTER_SIZE --instance-type $AWS_INSTANCE_TYPE --launch-type EC2
```

### Definicion de tareas ECS

Despues de tener el cluster ECS arriba y corriendo, la definicion de tarea para jitsi meet debe ser creada usando el archivo [docker-compose.yml](jitsi-meet/docker-compose.yml) localizado en el directorio [jitsi-meet](jitsi-meet).

```bash
cd jitsi-meet
ecs-cli compose --file docker-compose.yml create
```
## TODO
- Container Links
- Create ECS Service
- Update Security Group
- Add SSL Certificate
- Configure Load Balancer
