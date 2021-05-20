# Introdução
-----

Esta solução visa auto taggear recursos com e notificar sobre recursos não taggeados corretamente. Todo novo recurso criado é registrado no CloudTrail, que gera um evento no formato JSON com as informações do recurso criado, o CloudWatch EventBridge recebe este JSON e aciona 2 funções Lambdas, uma delas cria uma Tag automatica no recurso com o nome do usuário criador, a outra função checa se o recurso criado recebeu as Tags devidas e em caso negativo envia uma notificação através do SES para o email configurado informando as Tags faltantes.
Obs. Solução funciona apenas para Instancia EC2, Bucket S3 e Instancia RDS.

![Auto TAG Notify](./autotag.png)

## Deploy da Solução

A solução deve ser criada em cada região que deverá ser monitorada.

### Pré Requisitos
1 - Cloud Trail configurado em cada uma das regiões que receberão a solução;

2 - SES autorizado a enviar email pelo dominio que enviará o email.
É necessário configurar o SES apenas em uma região; (As outras regioes conseguem usar o SES em outras regiões)

### Deploy utilizando cloudformation

1 - Criar um bucket no S3 com o nome como por exemplo, **"auto-tag-resources"** e uma pasta com o nome **"auto-tag-resources"**;

2 - Na pasta criada dentro do bucket, fazer o upload dos arquivos **"auto-tag-resources.zip"** e **"auto-tag-notify-resources.zip"** contidos neste repositório;

3 - Baixar para sua maquina o arquivo do CloudFormation **"auto-tag-resources.json"** e realizar o deploy com o comando abaixo:

```
aws cloudformation create-stack --stack-name auto-tag-resources --template-body file://auto-tag-resources.json --capabilities CAPABILITY_NAMED_IAM  --parameters ParameterKey=pSupportingFilesBucket,ParameterValue=auto-tag-resources ParameterKey=pSupportingFilesPrefix,ParameterValue=auto-tag-resources/ --profile  <AWS_Profile> --region <AWS_Region>
```
4 - Acompanhar pelo console da AWS a conclusão do deploy da Stack pelo CloudFormation;

5 - Na função Lambda **"auto-tag-resources-notify"** configurar as variaves de ambiente conforme o modelo abaixo:

```
EMAIL_ADDRESS	["user1@email.com","user2@email.com","user3@email.com"]
EMAIL_SENDER	tags_notify@email.com
TAGS_TO_CHECK	["squad","tribo","produto"]
```

Onde: <br>
    **"EMAIL_ADDRESS"** são os emails que receberão a notificação das Tags faltantes;<br>
    **"EMAIL_SENDER"** é o remetente das mesnsagens de email;<br>
    **"TAGS_TO_CHECK"** é a lista de Tags que serão checadas.<br>
