# Registro de Customizações do Sistema (Securo Fork)

Este arquivo serve como mapa de memória para o assistente de IA (Antigravity) e para os desenvolvedores. Toda vez que uma alteração, nova tela, serviço ou migração for criada neste fork, ela deve ser registrada aqui.

---

## 📌 Convenções Adotadas

* **Extensão de Código:** Arquivos próprios recebem prefixo `custom_` ou ficam em pastas `custom/`.
* **Branch de Produção:** `custom-prod`.
* **Registro de Dependências:** Novas bibliotecas no `pyproject.toml` ou `package.json` recebem a tag `# [CUSTOM]`.

---

## 📝 Histórico de Customizações

*(Nenhuma customização adicionada ainda. As novas funcionalidades serão listadas abaixo com data, arquivos afetados e pontos de integração).*

### Exemplo de Formato para Novos Registros:
```markdown
### [YYYY-MM-DD] - Nome da Funcionalidade
* **Objetivo:** Descrição breve do que foi adicionado ou modificado.
* **Arquivos Criados:**
  * `backend/app/api/custom_reports.py`
  * `frontend/src/components/custom/CustomChart.tsx`
* **Pontos de Acoplamento:**
  * `backend/app/main.py`: Inclusão do roteador `custom_reports.router`.
* **Migrações de Banco:**
  * `backend/alembic/versions/20261028_custom_tabela_x.py` (down_revision: `<hash_do_autor>`)
```
