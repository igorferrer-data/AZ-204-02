# AZ-204
##  Como Fazer o Deploy de uma API na Nuvem

Criação de um projeto no Visual Studio: Um novo projeto ASP.NET Core Web API é criado. O foco não é no código da API em si, mas sim na automatização do processo de implantação.

Criação de um projeto no Azure DevOps: Um novo projeto é criado no Azure DevOps, configurado como privado.

Criação de um repositório Git: Um repositório Git vazio é criado dentro do projeto Azure DevOps. O projeto do Visual Studio é clonado localmente usando o comando git clone.

Criação de recursos no Azure: Um novo grupo de recursos (DIO-API-DEMO) e um Azure Container Registry (acrapidemohsouza001) são criados no portal do Azure.

Criação do pipeline YAML: Um pipeline YAML é criado no Azure DevOps que faz o seguinte:
- trigger: Define um gatilho para executar o pipeline quando houver alterações no ramo main do repositório Git.
- pool: Define um pool de agentes Ubuntu para executar o pipeline.
- variables: Define variáveis para o nome da solução (solution), a plataforma de build (buildplatform), e a configuração de build (buildconfiguration).
- steps: Define as etapas do pipeline:
  - task: UseDotNet@2: Instala o SDK do .NET Core.
  - script: Executa comandos para restaurar a solução, construir a solução e executar testes.
  - task: Docker@2: Cria e publica a imagem Docker no Azure Container Registry.

Configuração de Webhooks: Um webhook é configurado no Azure Container Registry para acionar uma nova implantação no Web App sempre que uma nova imagem for publicada.

Criação de Web App: Um Web App no Azure é criado com as devidas configurações, incluindo a ligação ao ACR. A autenticação básica é ativada no Web App para permitir que o ACR publique a imagem.

Execução do Pipeline: O pipeline YAML é executado manualmente, demostrando a automatização do processo.

pipeline.yml:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  solution: '*.sln'
  buildplatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '8.x'
- script: |
    dotnet restore $(solution)
    dotnet build $(solution) --configuration $(buildConfiguration)
    dotnet test $(solution) --configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage"
  displayName: 'Build Solution'
- task: Docker@2
  inputs:
    containerRegistry: 'acrapidemohsouza001'
    repository: 'api-dio-test'
    command: 'buildAndPush'
    Dockerfile: '/APItempoDIO/Dockerfile'
    context: '/APItempoDIO'
    tags: '$(Build.BuildId)'
```
