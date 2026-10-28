# Guia de Sincronização, Atualizações e Backup (Runbook)

Este documento é o seu manual de segurança operacional. Siga rigorosamente estas instruções sempre que você decidir sincronizar o seu fork com as novidades lançadas pelo autor original do Securo.

---

## ⚠️ Regra Zero: Nunca atualize sem fazer Backup primeiro!

Atualizações de software podem introduzir novas migrações de banco de dados irreversíveis. Se algo quebrar durante o processo, o seu backup garantirá que nenhum dado financeiro seja perdido.

---

## 1. Passo 1: Backup do Banco de Dados PostgreSQL

### Backup Principal: Na VPS (Easypanel)
No Easypanel, execute o backup do banco `securo-db`:
1. **Via Painel do Easypanel:** Acesse o banco de dados `securo-db` > vá na aba **Backups** e clique em **Create Backup** (ou baixe o arquivo de dump para o seu PC/Google Drive).
2. **Via Terminal SSH da VPS (ou Console do Easypanel):**
   ```bash
   # Gera o dump SQL com a data e hora atual
   docker exec -t securo-db pg_dump -U securo securo_db > /root/backup_securo_pre_update_$(date +%Y%m%d_%H%M%S).sql
   ```
Guarde esse arquivo em um local seguro (ex: Google Drive).

---

## 2. Passo 2: Sincronizando com o Repositório Original (Upstream)

Abra o terminal na pasta local do projeto e execute os passos na ordem:

```bash
# 1. Garanta que o repositório local está limpo (sem alterações soltas)
git status

# 2. Mude para a branch main
git checkout main

# 3. Puxe as novidades do autor original
git fetch upstream
git merge upstream/main

# 4. Envie as novidades do autor para o seu fork no GitHub (mantendo seu fork sincronizado)
git push origin main
```

Neste momento, a sua branch `main` está 100% atualizada com a versão mais recente do criador do projeto, sem nenhum conflito.

---

## 3. Passo 3: Mesclando as Novidades na sua Branch de Produção (`custom-prod`)

Agora vamos trazer as novidades para a sua versão com customizações:

```bash
# 1. Mude para a sua branch de produção
git checkout custom-prod

# 2. Crie uma branch de teste temporária para a atualização (MUITO RECOMENDADO)
git checkout -b update/sync-upstream

# 3. Mescle a main atualizada nesta branch de teste
git merge main
```

### O que pode acontecer neste momento?

#### Cenário A: Fast-forward / Merge Automático (Sem Conflitos)
Se não houver alterações em arquivos que você mexeu, o Git finalizará o merge sozinho sem perguntas.

#### Cenário B: Conflito de Arquivos
Se houver arquivos com conflito:
1. Abra o VS Code / Antigravity e peça:  
   *"Estou fazendo o merge da atualização do autor e tive conflitos. Leia o [CUSTOMIZATIONS.md](file:///d:/Projetos%20code/SECURO/CUSTOMIZATIONS.md) para preservar nossas alterações e resolva os conflitos para mim."*
2. A IA resolverá os conflitos mantendo os arquivos e extensões customizadas intactos.
3. Finalize o merge:
   ```bash
   git add .
   git commit -m "chore: resolve conflitos de sincronizacao com upstream"
   ```

---

## 4. Passo 4: Verificando as Migrações do Alembic

Se o autor adicionou novas tabelas ou colunas:

1. Verifique se o Alembic reclama de múltiplos heads:
   ```bash
   # Dentro do contêiner do backend ou localmente:
   alembic heads
   ```
2. Se aparecer mais de 1 head (bifurcação entre a migração oficial e a nossa customizada):
   * Abra a nossa migração em `backend/alembic/versions/` (listada no `CUSTOMIZATIONS.md`).
   * Altere a variável `down_revision = 'hash_anterior'` para apontar para o hash da **última migração oficial do autor**.
   * Rode novamente: `alembic heads`. Deverá aparecer apenas um único head.

---

## 5. Passo 5: Envio e Deploy Automático na VPS

Como o seu ambiente local é leve (apenas para edição de código):

1. Envie a branch atualizada para a `custom-prod`:
   ```bash
   git checkout custom-prod
   git merge update/sync-upstream
   git push origin custom-prod
   git branch -d update/sync-upstream
   ```
2. O **Easypanel na sua VPS** detectará o novo commit automaticamente:
   * Ele fará o build dos novos contêineres no servidor.
   * Executará as migrações do Alembic.
   * Atualizará a aplicação em produção sem você precisar tocar em nada.
3. Abra a URL do seu Securo no navegador e teste se tudo subiu com sucesso.

---

## 6. Procedimento de Emergência (Rollback)

Se algo der muito errado após o deploy na VPS:

### 1. Voltar o código para o commit anterior:
```bash
git checkout custom-prod
# Voltar 1 commit ou resetar para a tag/commit anterior estável
git reset --hard HEAD~1
git push origin custom-prod --force
```

### 2. Restaurar o banco de dados pelo Backup:
No Easypanel / VPS:
```bash
# Restaura o dump que você gerou no Passo 1
docker exec -i securo-db psql -U securo -d securo_db < /root/backup_securo_pre_update_XXXXX.sql
```
O sistema voltará exatamente ao estado anterior à tentativa de atualização.
