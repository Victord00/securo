# Diretrizes do Agente de Atualizações e Customizações (Securo Fork)

Este documento instrui o assistente de IA (Antigravity) sobre como operar neste repositório. O objetivo primordial é manter o fork sustentável, de fácil manutenção e atualizável a longo prazo em relação ao repositório original (`upstream`).

---

## 1. Estratégia de Branches e Versionamento

* **Branch `main` (ou `master`):**
  * Deve permanecer **100% idêntica ao repositório do autor original (`upstream`)**.
  * **NUNCA** faça commits diretos na `main`.
  * Apenas realize pull de atualizações do `upstream` para a `main`.
* **Branch `custom-prod`:**
  * É a branch de produção personalizada onde residem todas as customizações estáveis.
  * O Easypanel (servidor VPS) realiza o deploy a partir desta branch.
* **Branches de Feature (`feature/*` ou `fix/*`):**
  * Qualquer alteração de código solicitada pelo usuário deve ser feita em uma branch temporária (ex: `feature/novo-relatorio`).
  * Somente após validação e teste local, mescle na `custom-prod`.

---

## 2. Princípio da Extensão (Open-Closed Principle)

Para minimizar conflitos de merge durante atualizações futuras do autor:

* **Adicionar novos arquivos em vez de alterar arquivos nativos:**
  * Backend: Crie novos módulos ou serviços em pastas próprias ou com prefixos identificáveis (ex: `backend/app/services/custom_*.py` ou `backend/app/api/custom_*.py`).
  * Frontend: Crie novos componentes em diretórios separados (ex: `frontend/src/components/custom/`).
* **Não reescrever rotas existentes da API:**
  * Em vez de editar as funções em arquivos de rotas originais, crie um novo router FastAPI dedicado (ex: `custom_reports.py`) e apenas registre-o no `main.py`.
* **Frontend: Pattern Wrapper / Decorator:**
  * Se for necessário adicionar novos botões, filtros ou painéis a uma tela nativa, evite editar profundamente o componente original. Crie um componente wrapper que renderiza o componente nativo e adiciona as novas seções.

---

## 3. Banco de Dados e Migrações (Alembic)

* **NUNCA alterar migrações existentes:**
  * Arquivos em `backend/alembic/versions/` criados pelo autor original são imutáveis.
* **Criação de Tabelas Auxiliares:**
  * Prefira criar tabelas novas (com chaves estrangeiras apontando para as tabelas nativas) em vez de adicionar colunas em tabelas originais do Securo. Isso previne quebras caso o autor altere colunas nativas no futuro.
* **Resolução de Head Conflicts no Alembic:**
  * Se o autor lançar novas migrações e ocorrer bifurcação no Alembic, identifique a última migração do autor e ajuste o campo `down_revision` da nossa migração customizada para apontar para o novo head oficial, restabelecendo a cadeia linear.

---

## 4. Gestão de Dependências e Docker

* **Dependências Extras:**
  * Se uma nova biblioteca Python for necessária, documente-a no `pyproject.toml` ou `requirements.txt` com um comentário explicativo: `# [CUSTOM] Dependência adicionada para funcionalidade X`.
  * Faça o mesmo para dependências npm no `package.json`.
* **Ambiente Docker:**
  * Se precisarmos de pacotes de sistema ou ferramentas extras no contêiner, crie arquivos de sobreposição (ex: `Dockerfile.custom` ou `docker-compose.override.yml`), preservando os arquivos originais.

---

## 5. Manutenção Obrigatória do `CUSTOMIZATIONS.md`

Toda vez que o agente adicionar ou modificar qualquer arquivo no projeto:
* O arquivo `CUSTOMIZATIONS.md` na raiz do projeto **DEVE ser atualizado**.
* Deve-se registrar:
  1. Data e nome da funcionalidade.
  2. Arquivos novos criados.
  3. Arquivos nativos que precisaram de ponto de acoplamento (ex: inclusão do router no `main.py`).
  4. Migrações criadas no Alembic.
