# Guia de Instalação e Fluxo de Trabalho (Securo Fork)

Este documento contém o passo a passo completo para configurar o seu ambiente de desenvolvimento local, versionamento no GitHub e o deploy contínuo na sua VPS com Easypanel.

---

## 1. Passo Inicial no GitHub (Criar o Fork)

1. Acesse o repositório original: [https://github.com/securo-finance/securo](https://github.com/securo-finance/securo)
2. No canto superior direito, clique em **Fork**.
3. Escolha a sua conta pessoal do GitHub como destino.
4. **IMPORTANTE**: Certifique-se de que a opção *"Copy the `main` branch only"* esteja **desmarcada** (caso queira ver outras tags/branches) ou mantenha marcada se desejar apenas a branch principal.
5. Copie a URL do seu fork criado (ex: `https://github.com/SEU_USUARIO/securo.git`).

---

## 2. Clonar o Repositório e Configurar os Remotes

Abra o terminal na pasta do projeto (`d:\Projetos code\SECURO`) e execute os comandos abaixo:

```bash
# 1. Clona o seu fork para a pasta local
git clone https://github.com/SEU_USUARIO/securo.git .

# 2. Conecta o repositório original do autor como "upstream"
git remote add upstream https://github.com/securo-finance/securo.git

# 3. Confirma se os dois remotes estão configurados corretamente
git remote -v
```

Você deverá ver algo assim:
* `origin`: aponta para o seu repositório pessoal (`SEU_USUARIO/securo.git`)
* `upstream`: aponta para o repositório oficial (`securo-finance/securo.git`)

---

## 3. Criação da Branch de Produção (`custom-prod`)

Para que você nunca suje a branch `main`:

```bash
# Cria e muda para a branch onde ficarão as suas customizações
git checkout -b custom-prod

# Envia a nova branch para o seu GitHub
git push -u origin custom-prod
```

> **Regra de Ouro:** A branch `main` nunca recebe código direto. Ela é usada exclusivamente para puxar novidades do autor (`upstream`).

---

## 4. Seu Computador Local: Apenas para Edição de Código (Zero Docker Local!)

Como a aplicação vai rodar direto na sua VPS:
* **Você NÃO precisa instalar o Docker Desktop no seu Windows.**
* Seu computador fica leve, consumindo zero memória com máquinas virtuais ou WSL.
* **O que você precisa no PC:** Apenas o **Git** e o **Antigravity / VS Code** para editarmos os arquivos.
* O build do Docker, o PostgreSQL e a execução acontecerão 100% dentro da sua VPS através do Easypanel.

---

## 5. Configurando o Deploy na sua VPS com Easypanel

Agora vamos colocar a sua instância online 24/7 na sua VPS:

1. Acesse o seu painel do **Easypanel**.
2. Crie um novo projeto ou entre em um existente (ex: crie um projeto chamado `securo`).
3. Clique em **+ New** > **App** (ou crie a partir de um GitHub repository).
4. Configure a integração com o GitHub:
   * **Repository**: Escolha o seu fork (`SEU_USUARIO/securo`).
   * **Branch**: Selecione obrigatoriamente a branch **`custom-prod`**.
   * **Build Type**: Dockerfile ou Docker Compose (conforme a estrutura do repositório).
5. **Banco de Dados (PostgreSQL)**:
   * No Easypanel, dentro do mesmo projeto, clique em **+ New** > **Database** > **Postgres**.
   * Dê o nome de `securo-db`.
   * Vincule a URL de conexão do Postgres nas variáveis de ambiente (`DATABASE_URL`) do aplicativo Securo.
6. **Domínio e SSL**:
   * Na aba **Domains** do app Securo no Easypanel, adicione o subdomínio desejado (ex: `securo.seudominio.com`).
   * O Easypanel gera o certificado HTTPS automaticamente via Let's Encrypt.
7. **Deploy Automático**:
   * Habilite a opção de **Auto Deploy** (Webhook do GitHub).
   * A partir deste momento, sempre que fizermos um commit e dermos `git push origin custom-prod`, o Easypanel compila e atualiza o seu sistema online sozinho.

---

## 6. Fluxo Diário de Trabalho

Quando você quiser criar uma nova funcionalidade ou ajustar algo no sistema comigo:

1. Criamos uma branch de feature:
   ```bash
   git checkout custom-prod
   git checkout -b feature/nome-da-funcionalidade
   ```
2. Desenvolvemos e testamos localmente no navegador.
3. Documentamos a alteração no `CUSTOMIZATIONS.md`.
4. Fazemos o merge na `custom-prod`:
   ```bash
   git checkout custom-prod
   git merge feature/nome-da-funcionalidade
   git push origin custom-prod
   ```
5. A VPS detecta o push e atualiza a sua versão de produção automaticamente!
