# Desafio

## Provider (AWS):
Define o provedor da AWS e especifica a região como us-east-1.

## Variáveis (projeto e candidato):
Define variáveis para o nome do projeto e o nome do candidato, que são usadas para nomear os recursos criados.

## TLS Private Key:
Gera uma chave privada RSA de 2048 bits, que será usada para acessar a instância EC2.

## AWS Key Pair:
Cria um par de chaves na AWS utilizando a chave pública gerada no recurso tls_private_key. Essa chave será usada para autenticar via SSH na instância EC2.

## VPC (Virtual Private Cloud):
Cria uma VPC com o bloco CIDR 10.0.0.0/16, habilitando suporte a DNS e nomes de host. A VPC é a rede virtual na qual todos os outros recursos estarão contidos.

## Subnet:
Cria uma subnet dentro da VPC com o bloco CIDR 10.0.1.0/24, alocada na zona de disponibilidade us-east-1a. A subnet é a divisão de rede onde a instância EC2 será provisionada.

## Internet Gateway:
Cria um Internet Gateway, permitindo que a VPC tenha comunicação com a Internet.

## Route Table e Association:
Cria uma tabela de rotas e adiciona uma rota padrão (0.0.0.0/0) para redirecionar o tráfego para o Internet Gateway.
Associa a tabela de rotas com a subnet, permitindo que a subnet tenha acesso à Internet via a rota criada.

## Security Group:
Cria um Security Group para a VPC, definindo as regras de segurança:
Entrada: Permite acesso SSH (porta 22) de qualquer endereço IP (0.0.0.0/0 e IPv6 ::/0).
Saída: Permite todo o tráfego de saída (sem restrições).

## Amazon Machine Image (AMI):
Busca a AMI mais recente da distribuição Debian 12, filtrando por debian-12-amd64-* e com suporte a virtualização HVM.

## Instância EC2:
Cria uma instância EC2 utilizando a AMI Debian 12 com o tipo de instância t2.micro.
A instância é criada na subnet definida, com o IP público associado, segurança fornecida pelo Security Group e o par de chaves gerado previamente.
O disco de root tem tamanho de 20 GB e tipo gp2 (SSD padrão).
Inclui um script de inicialização (user_data) para atualizar o sistema Debian automaticamente após a criação.

## Outputs:
private_key: Exibe a chave privada (sensível) para acessar a instância EC2.
ec2_public_ip: Exibe o endereço IP público da instância EC2 criada.
