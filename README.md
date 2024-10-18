# Desafio

## Provider (AWS):
`provider "aws" {
  region = "us-east-1"
}`

Define o provedor da AWS e especifica a região como us-east-1.

## Variáveis (projeto e candidato):
`variable "projeto" {
  description = "Nome do projeto"
  type        = string
  default     = "VExpenses"
}`

`variable "candidato" {
  description = "Nome do candidato"
  type        = string
  default     = "SeuNome"
}`

Define variáveis para o nome do projeto e o nome do candidato, que são usadas para nomear os recursos criados.

## TLS Private Key:
`resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}`

Gera uma chave privada RSA de 2048 bits, que será usada para acessar a instância EC2.

## AWS Key Pair:
`resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${var.projeto}-${var.candidato}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}`

Cria um par de chaves na AWS utilizando a chave pública gerada no recurso tls_private_key. Essa chave será usada para autenticar via SSH na instância EC2.

## VPC (Virtual Private Cloud):
`resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true`
  
 `tags = {
    Name = "${var.projeto}-${var.candidato}-vpc"
  }
}`

Cria uma VPC com o bloco CIDR 10.0.0.0/16, habilitando suporte a DNS e nomes de host. A VPC é a rede virtual na qual todos os outros recursos estarão contidos.

## Subnet:
`resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"`

  `tags = {
    Name = "${var.projeto}-${var.candidato}-subnet"
  }
}`

Cria uma subnet dentro da VPC com o bloco CIDR 10.0.1.0/24, alocada na zona de disponibilidade us-east-1a. A subnet é a divisão de rede onde a instância EC2 será provisionada.

## Internet Gateway:
`resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id`

  `tags = {
    Name = "${var.projeto}-${var.candidato}-igw"
  }
}`

Cria um Internet Gateway, permitindo que a VPC tenha comunicação com a Internet.

## Route Table e Association:
`resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id`

  `route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }`

 ` tags = {
    Name = "${var.projeto}-${var.candidato}-route_table"
  }
}`

`resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id`

  `tags = {
    Name = "${var.projeto}-${var.candidato}-route_table_association"
  }
}`

Cria uma tabela de rotas e adiciona uma rota padrão (0.0.0.0/0) para redirecionar o tráfego para o Internet Gateway.
Associa a tabela de rotas com a subnet, permitindo que a subnet tenha acesso à Internet via a rota criada.

# Security Group:
`resource "aws_security_group" "main_sg" {
  name        = "${var.projeto}-${var.candidato}-sg"
  description = "Permitir SSH de qualquer lugar e todo o tráfego de saída"
  vpc_id      = aws_vpc.main_vpc.id`

  ## Regras de entrada
  `ingress {
    description      = "Allow SSH from anywhere"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }`

  ## Regras de saída
  `egress {
    description      = "Allow all outbound traffic"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }`

  `tags = {
    Name = "${var.projeto}-${var.candidato}-sg"
  }
}`

Cria um Security Group para a VPC, definindo as regras de segurança:
Entrada: Permite acesso SSH (porta 22) de qualquer endereço IP (0.0.0.0/0 e IPv6 ::/0).
Saída: Permite todo o tráfego de saída (sem restrições).

## Amazon Machine Image (AMI):
`data "aws_ami" "debian12" {
  most_recent = true`

  `filter {
    name   = "name"
    values = ["debian-12-amd64-*"]
  }`

  `filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["679593333241"]
}`

Busca a AMI mais recente da distribuição Debian 12, filtrando por debian-12-amd64-* e com suporte a virtualização HVM.

## Instância EC2:
`resource "aws_instance" "debian_ec2" {
  ami             = data.aws_ami.debian12.id
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.main_subnet.id
  key_name        = aws_key_pair.ec2_key_pair.key_name
  security_groups = [aws_security_group.main_sg.name]`

  `associate_public_ip_address = true`

  `root_block_device {
    volume_size           = 20
    volume_type           = "gp2"
    delete_on_termination = true
  }`

 ` user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get upgrade -y
              EOF`

  `tags = {
    Name = "${var.projeto}-${var.candidato}-ec2"
  }
}`

Cria uma instância EC2 utilizando a AMI Debian 12 com o tipo de instância t2.micro.
A instância é criada na subnet definida, com o IP público associado, segurança fornecida pelo Security Group e o par de chaves gerado previamente.
O disco de root tem tamanho de 20 GB e tipo gp2 (SSD padrão).
Inclui um script de inicialização (user_data) para atualizar o sistema Debian automaticamente após a criação.

## Outputs:
`output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}`

`output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}`

private_key: Exibe a chave privada (sensível) para acessar a instância EC2.
ec2_public_ip: Exibe o endereço IP público da instância EC2 criada.

# Observações:

## Restrições de SSH:
Grupos de Segurança: Restringir o acesso SSH a IPs específicos é crucial. Utilizar grupos de segurança para permitir o acesso apenas de IPs confiáveis.
SSH Keys: Considerar utilizar chaves SSH públicas para autenticar o acesso, em vez de senhas. Isso oferece uma camada adicional de segurança.
Fail2ban: Implementar um sistema como o Fail2ban para bloquear automaticamente IPs que tentam fazer login por SSH várias vezes sem sucesso.

## Proteção contra Ataques:
MFA: Configurar a autenticação de dois fatores (MFA) para a conta da AWS que está sendo utilizada para maior segurança.
IAM Roles: Utilizar papéis IAM (Identity and Access Management) para conceder permissões mínimas necessárias à instância EC2, evitando a necessidade de chaves de acesso root.
Firewall: Considerar a implementação de um firewall adicional, como o AWS WAF, para proteger a aplicação contra ataques web.

## Automação de Atualizações: 
O script user_data realiza uma atualização completa do sistema ao iniciar, o que é uma prática útil para garantir que o sistema esteja atualizado, porém pode aumentar o tempo de inicialização da instância.


# Descrição Técnica das Alterações Feitas no código
***Restrição de Acesso SSH***: A regra de segurança foi ajustada para restringir o acesso SSH a um IP confiável definido na variável trusted_ip. Isso impede que qualquer pessoa fora dessa rede tente acessar a instância via SSH.

***Desabilitar Autenticação via Senha:*** No script de user_data, foi adicionada uma linha para desativar o uso de senhas no SSH. Assim, o acesso à instância será feito exclusivamente por chaves SSH, aumentando a segurança.

***Instalação Automatizada do Nginx:*** O script também foi modificado para instalar e iniciar o Nginx automaticamente na inicialização da instância. O serviço também é configurado para iniciar automaticamente em reinicializações.

## ***Outras Melhorias***
A inclusão da automação da instalação do Nginx reduz a necessidade de intervenção manual após o provisionamento da instância.
Melhor controle sobre o acesso SSH ajuda a evitar ataques de força bruta.
