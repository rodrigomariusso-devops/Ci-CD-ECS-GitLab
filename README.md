# CI/CD Pipeline para Deploy no AWS ECS

Este repositório contém uma estrutura de CI/CD usando o GitLab CI para construir e enviar imagens Docker para o Amazon Elastic Container Registry (ECR), e um exemplo de aplicação simples que será implantada no Amazon Elastic Container Service (ECS).

## Estrutura do Projeto

O projeto é dividido em dois grupos:

1. **automation/pipeline-base**: Contém a configuração base do pipeline.
   - **`.gitlab-ci.yml`**: Arquivo de configuração do GitLab CI.
  
2. **homolog/deploy-ecr**: Contém a aplicação simples que será construída e enviada para o ECR.

### Aplicação Simples

A aplicação inclui os seguintes arquivos:

- **index.html**: Página principal da aplicação.
- **css/styles.css**: Estilo da aplicação.
- **js/script.js**: Comportamento da aplicação.
- **Dockerfile**: Arquivo de configuração do Docker para construir a imagem da aplicação.

## Configuração do Pipeline

### Pipeline Base e Pipeline do Projeto

A estrutura de CI/CD foi dividida em duas partes principais: o **pipeline base** e o **pipeline do projeto**. 

1. **Pipeline Base (`automation/pipeline-base`)**:
   - O pipeline base contém a configuração padrão que pode ser reutilizada por outros projetos. Isso inclui os estágios de build e push da imagem Docker, além de variáveis comuns necessárias para autenticação com o AWS ECR.
   - Ao centralizar essa configuração, garantimos consistência e reduzimos a duplicação de código entre projetos. Assim, qualquer alteração ou melhoria na configuração do pipeline pode ser aplicada em todos os projetos que utilizam essa base de forma rápida e eficiente.

2. **Pipeline do Projeto (`homolog/deploy-ecr`)**:
   - O pipeline do projeto utiliza o pipeline base como ponto de partida. Ele herda as definições e jobs do pipeline base, permitindo que o projeto se concentre nas suas necessidades específicas.
   - Isso significa que, ao adicionar novas funcionalidades ou ajustar a lógica de deploy, o projeto pode se beneficiar automaticamente de qualquer atualização que for feita no pipeline base, facilitando a manutenção e a escalabilidade.

### Resumo da Configuração do Pipeline

#### Arquivo `.gitlab-ci.yml`

Este arquivo define dois estágios principais: `build` e `push`. Abaixo está um resumo de cada etapa:

1. **Estágios**
   - **build**: Compila a imagem Docker.
   - **push**: Envia a imagem para o ECR.

2. **Variáveis**
   - **AWS_REGION**: A região da AWS onde o ECR está localizado (ex: `us-east-2`).
   - **ECR_REPOSITORY**: O repositório ECR onde as imagens serão armazenadas.

3. **Jobs**
   - **build_image**: 
     - Este job constrói a imagem Docker usando o Dockerfile especificado.
     - Executa apenas em branches `master`.

   - **push_to_ecr**: 
     - Este job autentica com o ECR, faz o tag e envia a imagem Docker para o ECR.
     - Também executa apenas em branches `master`.

### Exemplo de `.gitlab-ci.yml`

```yaml
# Estágios padrão para build e envio de imagem
stages:
  - build
  - push

# Variáveis comuns para acesso ao ECR
variables:
  AWS_REGION: "us-east-2"
  ECR_REPOSITORY: "598723879413.dkr.ecr.$AWS_REGION.amazonaws.com"

# Job de build da imagem Docker
build_image:
  stage: build
  script:
    - echo "Building Docker image..."
    - docker build -t $IMAGE_NAME -f $DOCKERFILE .
  only:
    - master
  tags:
    - hom

# Job para enviar a imagem ao AWS ECR
push_to_ecr:
  stage: push
  script:
    - echo "Authenticating with AWS ECR..."
    - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY

    - echo "Tagging and pushing image to ECR..."
    - docker tag $IMAGE_NAME:latest $ECR_REPOSITORY/$IMAGE_NAME:latest
    - docker push $ECR_REPOSITORY/$IMAGE_NAME:latest

    - docker tag $IMAGE_NAME:latest $ECR_REPOSITORY/$IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $ECR_REPOSITORY/$IMAGE_NAME:$CI_COMMIT_SHA
  only:
    - master
  tags:
    - hom
```

## Como Usar

1. **Clone o Repositório**:
   Clone este repositório em sua máquina local.

   ```bash
   git clone <URL_DO_REPOSITORIO>
   ```

2. **Configure o GitLab CI**:
   - Certifique-se de que o GitLab CI esteja habilitado em seu projeto.
   - Adicione as variáveis necessárias no GitLab para `IMAGE_NAME`, `DOCKERFILE`, e credenciais da AWS para autenticação no ECR.

3. **Construa e Envie a Imagem**:
   - Faça push para a branch `master` e o pipeline do GitLab iniciará automaticamente, construindo e enviando a imagem para o ECR.
