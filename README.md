# Projeto Wordpress-na-AWS

## Vamos para o console AWS

## 1. Criando uma VPC
1. Acesse o console da AWS e abra o serviço VPC.
2. No menu lateral, clique em Suas VPCs e depois em Criar VPC.
3. Defina os seguintes parâmetros:
    * Nome: `vpc-wordpress`
    * CIDR: `10.0.0.0/16`
4. Clique em Criar VPC.

## 2. Criando as Sub-redes
1. No menu lateral do VPC, vá até Sub-redes e clique em Criar Sub-rede 
2. Configure os seguintes parâmetros:
    * ID da VPC: Selecione a VPC recém-criada (``vpc-wordpress``).
    * Nome da sub-rede : `subnet-publica-1`.
    * Zona de disponibilidade : `us-east-1a`.
    * Bloco CIDR IPv4 da Sub-rede: `10.0.1.0/24.`
3. Clique em Criar Sub-rede.
4. Repita o processo para criar as demais sub-redes:

|Nome da Sub-rede |   Tipo  | Zona de Disponibilizade | Bloco CIDR IPV4 |
|-----------------| --------| ------------------------|-----------------| 
|subnet-publica-2 | Publica |     us-east-1b          |   10.0.2.0/24   |
|subnet-privada-1 | Privada |     us-east-1a          |   10.0.3.0/24   |
|subnet-privada-2 | Privada |     us-east-1b          |   10.0.4.0/24   |

## 3. Criando um Gateway de Internet
1. No serviço VPC, acesse Gateways da Internet.
2. Clique em Criar Gateway da Internet.
3. Defina os seguintes parâmetros:
    * Nome: `igw-wordpress`.
4. Clique em Criar Gateway da Internet.
5. Após criar, insira o Internet Gateway e clique em Ações → Associar a uma VPC.
6. Selecione a VPC (`vpc-wordpress`) e confirme.

## 4. Criando um Gateway NAT
1. No serviço VPC, acesse Gateway NAT.
2. Clique em Criar Gateway NAT.
3. Defina os seguintes parâmetros:
    * Nome: `natgw-wordpress`.
    * Sub-rede: `subnet-publica-1`.
    * Tipo de conectividade: `Público`
    * ID de alocação do IP elástico: `Alocar IP elástico`.
4. Clique em Criar Gateway NAT.

## 5. Criando Tabela de Rotas
### Criando uma tabela de rota pública 
1. No serviço VPC, acesse tabela de rotas.
2. Clique em Criar tabela de rotas.
3. Defina as seguintes parâmetros:
    * Nome: `rt-publica`.
    * VPC: `vpc-wordpress`.
4. Clique em Criar tabela de rotas.
5. Em rotas clique em editar rotas.
6. Adicionar rota:
    * Destino: `0.0.0.0/0`.
    * Alvo: Gateway da Internet (`igw-wordpress`).
7. Clique em Salvar alteração.
8. Clique em Associação de sub-rede.
9. Editar associação de sub-rede.
10. Selecione a `subnet-publica-1` e `subnet-publica-2`
11. Clique em Salvar associação.

### Criando uma tabela de rota privada
1. Clique em Criar tabela de rotas.
2. Defina as seguintes parâmetros:
    * Nome: `rt-privada`.
    * VPC: `vpc-wordpress`.
3. Clique em Criar tabela de rotas.
4. Em rotas clique em editar rotas.
5. Adicionar rota:
    * Destino: `0.0.0.0/0`.
    * Alvo: Gateway NAT (`natgw-wordpress`).
6. Clique em Salvar alteração.
7. Clique em Associação de sub-rede.
8. Editar associação de sub-rede.
10. Selecione a `subnet-privada-1` e `subnet-privada-2`
11. Clique em Salvar associação.

## 6. Criando Grupo de Segurança
1. No serviço VPC, acesse Grupo de Segurança.
2. Clique em Criar grupo de segurança.
3. Crie o primeiro grupo de segurança:
    * Nome: `ec2-privada`.
    * Descrição: `ec2-privada`.
    * VPC: `vpc-wordpress`.
4. Adicionar regra:
    * Tipo: `SSH`
    * Origem: `0.0.0.0/0`
    * Tipo: `NFS`
    * Origem: `efs`
    * Tipo: `MYSQL/Aurora`
    * Origem: `rds-mysql`
    * Tipo: `HTTP`
    * Origem: `lb-classic-wordpress`
5. Salvar regras.
6. Clique em Criar grupo de segurança.

1. Crie o segundo grupo de segurança:
    * Nome: `rds-mysql`.
    * Descrição: `rds-mysql`.
    * VPC: `vpc-wordpress`.
3. Adicionar regra:
    * Tipo: `MYSQL/Aurora`
    * Origem: `ec2-privada`
4. Salvar regras.
5. Clique em Criar grupo de segurança.

1. Crie o terceiro grupo de segurança:
    * Nome: `efs`.
    * Descrição: `efs`.
    * VPC: `vpc-wordpress`.
2. Adicionar regra:
    * Tipo: `NFS`
    * Origem: `ec2-privada`
3. Salvar regras.
4. Clique em Criar grupo de segurança.

1. Crie o quarto grupo de segurança:
    * Nome: `lb-classic-wordpress`.
    * Descrição: `lb-classic-wordpress`.
    * VPC: `vpc-wordpress`.
2. Adicionar regra:
    * Tipo: `HTTP`
    * Origem: `0.0.0.0/0`
2. Clique em Criar grupo de segurança.


## 7. Criando Elastic File System (EFS)
1. Clique em criar sustema de arquivos
2. Configure os seguintes parâmetros:
    * Nome: `efs-wordpress`.
    * VPC: `vpc-wordpress`.
3. Clique em personalizar.
4.Vá em acesso á rede e faça as seguintes alterações:
    * VPC: `vpc-wordpress`.
Em destino de montagem:
    * Zona de disponibilidade: `us-east-1a`.
    * ID da sub-rede: `subnet-privada-1`.
    * Grupo de segurança: `efs`.
    * Zona de disponibilidade: `us-east-1b`.
    * ID da sub-rede: `subnet-privada-2`.
    * Grupo de segurança: `efs`.
6. Clique em criar.

## 8. Criando Banco de dados
1. Acesse o Aurora and RDS
2. Clique em banco de dados e depois em criar banco de dados
3. Faça as seguintes alterações:
    * Opções de mecanismo: `MySQL`.
    * Modelos: `Nível gratuito`.
    * Nome banco: `wordpress-db`.
    * usuário: `admin`.
    * Gerenciamento de credenciais: `Autogerenciada`.
    * Crie uma senha forte.
    * Configuração da instância: `db.t3.micro`.
    * VPC: `vpc-wordpress`.
    * Grupo de segurança da VPC: `rds-mysql`.
    * Configuração adicional/nome do banco de dados inicial: `wordpress`
4. Ciar banco de dados.

## 9. Criando IntânciaS (EC2)
1. Acesse EC2 e depois intâncias
2. Clique em executar intâncias.
3. Configure os seguintes parametros:
    * Nome: `Nome de sua escolha`.
    * Imagem do Sistema Operacional AMI: `Amazon Linux`(ou outra de sua escolha).
    * Adicione um par de chaves, caso não tiver, clique em criar par de chaves.
    * VPC: `vpc-wordpress`.
    * Sub-rede: `subnet-privada-1`.
    * Grupos de segurança/Selecionar um existente: `ec2-privada`.
4. Em detalhes avançados, faça as seguintes alterações:
    * Perfil de intância IAM: `EC2-SSM-Role`.
5. Em dados do usuario, anexe o seguinte arquivo de extensão `.sh`.

```
#!/bin/bash
dnf update -y
dnf install -y docker nfs-utils
systemctl start docker
systemctl enable docker

curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

EFS_ID="Substitua pelo ID real"
RDS_HOST="Substitua pelo endpoint real" 
RDS_USER="SEU NOME DE USUARIO"
RDS_PASSWORD="SUA SENHA"
RDS_NAME="SEU NOME DO BANCO"

mkdir -p /mnt/efs
echo "${EFS_ID}:/ /mnt/efs nfs4 defaults,_netdev 0 0" >> /etc/fstab
mount -a
mkdir -p /mnt/efs/wordpress
chown -R 1000:1000 /mnt/efs/wordpress
chmod -R 777 /mnt/efs/wordpress

mkdir -p /opt/wordpress
cd /opt/wordpress

cat > docker-compose.yml <<EOF
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "80:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: $RDS_HOST
      WORDPRESS_DB_USER: $RDS_USER
      WORDPRESS_DB_PASSWORD: $RDS_PASSWORD
      WORDPRESS_DB_NAME: $RDS_NAME
    volumes:
      - /mnt/efs/wordpress:/var/www/html/wp-content
EOF

docker-compose up -d
```
6. Clique em executar a intância.
7. Selecione a sua intância recém criada e clique em conectar.
8. Em gerenciador de sessões, clique em conectar.
9. No terminal, coloque os seguintes comandos:

```
sudo su
cd /
docker ps
cd mnt/efs/wordpress
ls
cat /etc/fstab | grep efs
mount | grep efs
echo "teste EFS" > /mnt/efs/wordpress/teste.txt
cat /mnt/efs/wordpress/teste.txt
sudo dnf install mariadb105
mysql --version
mysql -h <endpoint-do-rds> -u admin -p
SHOW DATABASES;
```

## 10. Criando Load Balancers
1. Em EC2, no painel lateral, procure por load balancers.
2. Clique em criar load balancer.
3. Em Classic Load Balancer, clique em criar.
4. Faça as seguintes alterações:
    * Nome: `clb-wordpress`.
    * VPC: `vpc-wordpress`.
    * Zona de disponibilidade: `subnet-publica-1` e `subnet-publica-2`.
    * Grupo de segurança: `lb-classic-wordpress`.
    * Caminho de ping: `/wp-admin/install.php`.
5.Configurações avançadas de verificação de integridade:
    * Tempo limite resposta: `5 segundos`
    * Limite não íntegro: `2`
    * Intervalo: `30 segundos`
    * Limite íntegro: `2`
    * Adicionar sua intância.
6. Criar load balancer
7. Depois abra seu navegador e coloque o caminho e logo em seguida irá aparecer a tela do Wordpress.

## 11. Criando Auto Scaling
### Criando a AMI personalizada
1. Vá para o EC2 > Instâncias.
2.Selecione a instância que está com o ambiente configurado.
3.No menu “Ações”, clique em:
    * Imagem e modelos > Criar imagem (AMI).
4. Preencha os campos:
    * Nome da imagem: `wordpress-efs-rds-2025`
    * Semelhante à instância: mantenha marcada.
    * Sem reinicializar: deixe desmarcado para garantir consistência.
5. Clique em Criar imagem.

### Criar Launch Template
1. Vá para o painel do EC2.
2. No menu lateral, clique em Modelos de execução (Launch Templates).
3. Clique em Criar modelo de execução.
4. Preencha:
    * Nome: `wordpress-template`
    * Descrição: algo como Template para instância WordPress com EFS e RDS
    * AMI: selecione a AMI que você criou (`wordpress-efs-rds-2025`)
    * Tipo de instância: ex: `t2.micro` (ou o tipo que usou)
    * Par de chaves: selecione a que usou na instância atual
3. Rede:
    * Deixe sem preferir zona específica, ou escolha se quiser fixar
4. Grupos de segurança:
    * Selecione o grupo de segurança da EC2 privada, que tem acesso ao RDS e ao EFS
5. Sistema de arquivos EFS:
    * Já estará incluso se estava montado e no fstab, então não precisa referenciar aqui
    * Outros campos: deixe o restante como padrão
6. Clique em Criar modelo de execução.

### Criando o Auto Scaling Group (ASG)
1. Vá para o console EC2 > Grupos de Auto Scaling
2. Clique em Criar grupo de Auto Scaling
   
3. Etapa 1: Escolher modelo de execução
    * Nome: `wordpress-asg`
    * Modelo de execução: selecione o Launch Template que você criou (`wordpress-template`)
    * Versão do template: pode deixar como Última versão (Latest)
Clique em Avançar

4. Etapa 2: Rede
    * VPC: selecione a sua VPC criada anteriormente
    * Zonas de disponibilidade e sub-redes:
    * Selecione duas sub-redes privadas diferentes (ex: `subnet-privada-1, subnet-privada-2`). Isso garante alta disponibilidade.
Clique em Avançar

5. Etapa 3: Balanceamento de Carga
  * Anexar ao Load Balancer existente:
    * Tipo: Classic Load Balancer
    * Selecione o Load Balancer que você já criou
    * Grupo de destino: não se aplica a CLB, ignore
    * Verificações de integridade: `Verificação de integridade do ELB`
Clique em Avançar

* Etapa 4: Configuração do grupo
    * Capacidade desejada: 3
    * Capacidade mínima: 3
    * Capacidade máxima: 5
Clique em Avançar
