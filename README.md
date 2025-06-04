# Roteiro sobre configuração e uso de um pipeline CI/CD

Este repositório apresenta um roteiro prático para configurar e utilizar um pipeline de **Integração Contínua e Delivery Contínuo (CI/CD)**. O objetivo é simular práticas reais de DevOps no contexto de desenvolvimento web, utilizando o GitHub Actions.

O GitHub Actions permite automatizar fluxos de trabalho (workflows) como testes, builds e deploys. Neste tutorial, aplicaremos esses princípios em um cenário semelhante ao de produção.

**Contexto**:  Considere que você faz parte da equipe de desenvolvimento web de uma ONG voltada ao cuidado animal, responsável pelo Adote Fácil, um sistema web para adoção de animais Sua equipe é distribuída geograficamente e é composta por profissionais da área de projeto Web em que cada um está responsável por diferentes partes do sistema. Você percebe que a adoção de metodologias de integração e entrega contínuas neste projeto irá promover diversas melhorias, como: Automatizar a função de um dos contribuintes de analisar os Pull Requests e realizar os Merges, disponibilizando-o para outras funções, elevar o padrão do codigo produzido, incentivar a criação e atualização constante de documentação, permitir a coleta de avaliações dos usuários de forma mais ágil, entre outras vantagens. Cientes desses potenciais ganhos, vocês desenvolveram um guia para a instalação de um ambiente servidor focado nessas práticas de CI/CD..

## Tarefa #1: Configurar o GitHub Actions
#### Passo 1

Crie um Token pessoal no GitHub (i) e;  faça um Fork (ii) do projeto adote-facil. 

**(i)** Para criar um Token, clique no ícone do seu perfil, vá em **Settings →**

![image](https://github.com/user-attachments/assets/b2861fa7-e874-4f81-8ea8-95a183802456)

**Developer settings →**

![image](https://github.com/user-attachments/assets/fd6f423d-5db5-4f39-b63a-c40aeb20e8c5)

**Personal access tokens →**

![image](https://github.com/user-attachments/assets/6b242dbd-69fd-42a8-b69d-b1cbc6cb8352)

**Tokens (classic)** →

![image](https://github.com/user-attachments/assets/44645080-395c-4a8b-a37f-adb8148bb3c1)

**Generate new token (classic)**. 

![image](https://github.com/user-attachments/assets/cafc4d73-2941-40de-a8eb-c111c2beea52)

Dê um nome para ele e marque as opções `repo` e `workflows` para gerar o Token. Gere o token mínimo (7 dias) apenas para este experimento. 

![image](https://github.com/user-attachments/assets/da72310e-d94c-4ea9-9c9f-7053b286fddd)

Copie e guarde esse código (token) para usar quando o GitHub lhe pedir para usar uma senha (password). 

#

**(ii)** Entre no repositório [adote-facil](https://github.com/ArthurEnrique15/adote-facil) e clique no botão **Fork** no canto superior direito na página do projeto no GitHub.

![fork](https://github.com/user-attachments/assets/a57143ff-3f79-4d3d-b827-018a7d91d39d)

#### Passo 2

Clone o repositório para sua máquina local, usando o seguinte comando (onde `<USER>` deve ser substituído pelo seu usuário no GitHub):

```bash
git clone https://github.com/<USER>/adote-facil.git
```

Em seguida, no diretório clonado, copie o código a seguir para um arquivo com o seguinte nome e caminho: `.github/workflows/experimento-ci-cd.yml`. Isto é, crie diretórios `.github` e depois `workflows` e salve o código abaixo no arquivo `experimento-ci-cd.yml`. 

**Linux:** Utilize os comandos os comandos `mv`, `cd`, `ls`, `mkdir` e `touch` no seu terminal ou use a GUI para criar os diretórios e arquivo.

**Windows:**

![Captura de Tela (18)](https://github.com/user-attachments/assets/40147beb-a254-4642-b7c5-0b8e8f493757)

No VSCode ou na sua IDE de preferência, clique com o botaão direito sobre o diretório `workflow` e selecione a opção New File. Então, crie o arquivo experimento-ci-cd.yml

![image](https://github.com/user-attachments/assets/ed77c027-a986-4dbb-bb27-f22e97fab558)

#

**Arquivo YML**
```yml
# Nome do workflow
name: experimento-ci-cd

# Evento que aciona o workflow: toda vez que for criado um Pull Request para a branch main
on:
  pull_request:
    branches:
      - main

# Definição dos jobs (tarefas) que serão executadas
jobs:

  # Primeiro job: Executar testes unitários
  unit-test:
    runs-on: ubuntu-latest  # Define o sistema operacional usado no runner (Ubuntu na versão mais recente)
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4  # Faz o download do repositório no runner

      - name: Instalar dependências do backend
        run: |
          cd backend              # Acessa a pasta do backend
          npm install             # Instala as dependências do Node.js

      - name: Executar testes unitários com Jest
        run: |
          cd backend              # Acessa novamente a pasta do backend
          npm test -- --coverage  # Executa os testes e gera relatório de cobertura

  # Segundo job: Build (construção) das imagens Docker
  build:
    needs: unit-test  # Esse job só será executado após o job 'unit-test' ser concluído com sucesso
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v2  # Habilita a ferramenta Buildx do Docker para builds mais avançados

      - name: Configurar Docker QEMU
        uses: docker/setup-qemu-action@v2  # Permite builds multiplataforma usando emulação (útil em CI)

      - name: Build das imagens Docker
        run: docker compose build  # Executa o build das imagens definidas no docker-compose.yml

  # Terceiro job: Subir os containers temporariamente para testes básicos de integração
  up-containers:
    needs: build  # Esse job depende do job 'build'
    runs-on: ubuntu-latest

    # Definição de variáveis de ambiente necessárias para o backend e banco
    env:
      POSTGRES_DB: adote_facil
      POSTGRES_HOST: adote-facil-postgres
      POSTGRES_USER: ${{ secrets.POSTGRES_USER }}  # Usuário do banco, vindo dos segredos do repositório
      POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}  # Senha do banco
      POSTGRES_PORT: 5432
      POSTGRES_CONTAINER_PORT: 6500

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Criar arquivo .env
        working-directory: ./backend  # Define o diretório de trabalho para esse passo
        run: |
          # Gera o arquivo .env com as variáveis definidas acima
          echo "POSTGRES_DB=${{ env.POSTGRES_DB }}" > .env
          echo "POSTGRES_HOST=${{ env.POSTGRES_HOST }}" >> .env
          echo "POSTGRES_USER=${{ env.POSTGRES_USER }}" >> .env
          echo "POSTGRES_PASSWORD=${{ env.POSTGRES_PASSWORD }}" >> .env
          echo "POSTGRES_PORT=${{ env.POSTGRES_PORT }}" >> .env
          echo "POSTGRES_CONTAINER_PORT=${{ env.POSTGRES_CONTAINER_PORT }}" >> .env

      - name: Subir containers com Docker Compose
        working-directory: ./backend
        run: |
          docker compose up -d     # Sobe os containers em segundo plano
          sleep 10                 # Aguarda alguns segundos para garantir que os serviços subam
          docker compose down      # Encerra os containers após o teste

  # Quarto job: Geração e entrega do artefato do projeto
  delivery:
    needs: build  # Esse job também depende do job 'build'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Gerar arquivo ZIP do projeto completo
        run: zip -r adote-facil-projeto.zip . -x '*.git*' '*.github*' 'node_modules/*'
        # Compacta todos os arquivos do repositório, exceto pastas desnecessárias como .git, .github e node_modules

      - name: Upload do artefato
        uses: actions/upload-artifact@v4
        with:
          name: adote-facil-projeto  # Nome do artefato que aparecerá na aba de artefatos da execução
          path: adote-facil-projeto.zip  # Caminho do arquivo que será enviado
```

Esse arquivo ativa e configura o GHA para toda vez que ocorrer um evento `pull_request` tendo como alvo a branch principal (main) do repositório. Ele realiza três jobs:

- roda os testes (test)
- faz a compilação (build)
- realiza uma entrega (delivery)

#### Passo 3

Entre no diretório criado (use o comando cd no terminal) ".../adote-facil/", realize um `add`, um `commit` e um `git push`, ou seja:

```bash
git add --all
git commit -m "Configurando o GitHub Actions"
git push origin main
```

## Tarefa #2: Configurar GitHub Secrets

Em muitos projetos, utilizamos variáveis sensíveis (como senhas e chaves) que não devem ser expostas no código. Para isso, o GitHub Actions permite o uso de Secrets — variáveis de ambiente criptografadas e seguras. Vamos configurar os Secrets para armazenar os dados de conexão com o banco de dados:

#### Passo 1

Acesse seu repositório adote-facil no GitHub, clique em Settings, depois vá em: **Secrets and variables** → **Actions** → **New repository secret**

![image](https://github.com/user-attachments/assets/6a78aaca-23a4-4047-a88d-59f3283f885b)
![image](https://github.com/user-attachments/assets/348907d6-765a-4dd3-bb1d-a0f5d4ae351a)
![image](https://github.com/user-attachments/assets/9e9bb6dc-6e5c-424f-a129-cb354e2f1b0f)

#### Passo 2

Crie dois segredos (secrets) com os seguintes valores:

1. **POSTGRES_USER**
   - **Name**: `POSTGRES_USER`
   - **Secret**: `Postgres`

2. **POSTGRES_PASSWORD**
   - **Name**: `POSTGRES_PASSWORD`
   - **Secret**: `postgres`

Esses valores simulam um cenário de acesso ao banco de dados. Eles serão utilizados automaticamente no workflow `.github/workflows/experimento-ci-cd.yml`.

## Tarefa #3: Criando um Pull Request (PR) com bug

Este caso simula um cenário em que o código contém um bug, fazendo com que o pipeline CI/CD falhe e rejeite o merge na main.

#### Passo 1

Vamos simular que a função de atualizar o e-mail do usuário está retornando um valor diferente do esperado. Para isso, edite o arquivo /backend/src/services/user/update-user-spec.ts e:

- Comente a linha 92.
- Logo abaixo, insira:
```diff
expect(result).toEqual(Success.create({ ...updatedUser, email: 'email-errado@mail.com' }))
````
![image](https://github.com/user-attachments/assets/e878aec6-e5a4-46ac-9d8a-bcab226fd1db)
![image](https://github.com/user-attachments/assets/8f4e723d-b29b-4f26-b06b-60bf98cc2636)

#### Passo 2

Após modificar o código, você deve criar uma novo branch, realizar um `commit` e um `push`:

```bash
git checkout -b bug
git add --all
git commit -m "Alterando o arquivo update-user-spec.ts"
git push origin bug
```

#### Passo 3

Em seguida, crie um Pull Request (PR) com suas modificações. Para isso, acesse no navegador a seguinte URL, substituindo <USER> pelo seu nome de usuário no GitHub: `https://github.com/<USER>/adote-facil/compare/main...bug`.

Nessa página, você poderá revisar as alterações feitas. Após conferir, clique no botão Create pull request. Na janela que se abrirá, insira uma breve descrição do PR e confirme a criação clicando novamente em Create pull request.

![image](https://github.com/user-attachments/assets/1ad57596-5f2f-4af7-8334-027c528043ad)

Assim que o PR for criado, o pipeline configurado no arquivo experimento-ci-cd.yml será automaticamente iniciada pelo GitHub Actions. Como introduzimos um bug, os testes irão falhar — essa falha será exibida na tela de execução do pipeline. Você pode acompanhar o status da execução acessando a aba Actions do seu repositório.

Em suma, o servidor de CI/CD detectou automaticamente um problema no código enviado, impedindo que ele seja integrado ao branch principal do projeto. 

![image](https://github.com/user-attachments/assets/b2669050-3b83-4f00-9cf4-3410e451ab84)


## Tarefa #4: Criando um Pull Request (PR) com a correção

Vamos restaurar o arquivo ao seu estado original. Para isso, descomente a linha 92 e exclua a linha 93. Assim, quando criarmos um novo PR, os testes serão executados com sucesso, sem apresentar falhas.

Após modificar o código, salve o arquivo e crie um novo branch para consertar o bug, realize um `add`, um `commit` e um `push`:

```bash
git checkout -b fixture
git add --all
git commit -m "Consertando a função Test"
git push origin fixture
```
Insira seu nome de usuário e senha (Token) do GH se for requerido.

Em seguida, crie novamente um Pull Request (PR) com a correção. Para isso, acesse no navegador a seguinte URL, substituindo <USER> pelo seu nome de usuário no GitHub: `https://github.com/<USER>/adote-facil/compare/main...fixture`.

![image](https://github.com/user-attachments/assets/db547d8a-a3ee-4a3c-81ad-69aa76ebac2a)

Nessa página, você poderá revisar as alterações realizadas. Depois, clique no botão Create pull request no canto superior direito da tela. Na janela que abrir, insira uma breve descrição do PR e confirme a criação clicando no botão Create pull request no canto inferior direito. Você pode acompanhar o andamento da pipeline acessando a aba Actions do repositório e selecionando o nome do PR em execução.

![image](https://github.com/user-attachments/assets/3b7ba313-04c5-415f-8097-946cb7f5a5c8)

Após a criação do PR, o GitHub Actions iniciará automaticamente o pipeline, que executará os testes, realizará o build e fará a entrega do artefato gerando um arquivo .zip do projeto. Quando concluído, o arquivo .zip estará disponível para download.

# FIM
