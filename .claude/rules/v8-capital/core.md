# V8 Core

## Referências

### Repositórios

Antes de implementar, procure aqui. Os outros repos da V8 costumam estar na mesma pasta que
este (`../<repo>`), mas nela pode haver repos que não são da org: da V8 são os que têm remote
em `github.com/v8capital`. Se não estiver clonado, o código está em
`github.com/v8capital/<repo>`.

| Precisa de | Use |
|---|---|
| Dia útil, feriado | `Calendar` de `v8_utilities.anbima_calendar`. Nunca `bdate_range`, `busday_count` ou `weekday()`: não conhecem feriado nacional nem de B3 |
| E-mail, Teams, Power BI | Em código novo, `v8_microsoft`. Em código que já usa `v8_utilities.email_manipulator` ou `v8_utilities.sharepoint`, não migre sem pedido: a migração tem plano e pendências (em 09/10/2026). `microsoft_azure` está arquivado. SharePoint ainda não existe no `v8_microsoft` (spec 004 do repo `microsoft`): pergunte antes de implementar |
| Cor, fonte, logo, componente visual | repo `design_system`: valores vêm de `v8_design_system/tokens/dist` e `assets`, nunca escritos à mão. Não é pacote instalável: leia os arquivos pelo caminho do repo clonado, sem `pip install` |
| Precificação de renda fixa e dado de mercado | `v8_pricing` |
| Dado da B3, do Banco Central, da ANBIMA, da CVM | `v8_b3`, `v8_bacen`, `v8_anbima`, `v8_cvm` |

Módulo `v8_*` fora desta tabela pode ser rascunho ou estar em revisão (em 09/10/2026). Em código que já o
usa, siga o que está lá. Para criar dependência nova nele, pergunte antes.

## Estrutura

### Pacote V8

O código costuma ficar numa pasta `v8_<repo>` na raiz, que faz o papel do `src/`. O prefixo
evita conflito com pacotes externos e entre repos, e é por esse nome que os outros repos
importam o pacote instalado. Hífen no nome do repo vira underscore. `src/` também vale quando
fizer sentido. Repo montado no Airflow usa `v8_<repo>/` na raiz: o `PYTHONPATH` do compose
do `dataflow` aponta para a raiz de cada repo, e `src/` exigiria mudar o compose.

### Pastas

`tests/` com pytest, `scripts/` para pontos de entrada, `notebooks/` para exploração.

- `drafts/`: rascunho e script de teste. Código de produção nunca importa de lá. Quando o
  rascunho vira código de verdade, vai para o pacote.
- `data/`: saída de teste e arquivo baixado à mão, sempre no `.gitignore`. Se a rotina precisa
  do arquivo para rodar em outra máquina ou no Airflow, ele vai para o file server.
- Versão antiga de código não vai para pasta (`old/`): fica no histórico do git.

## Efeito colateral

### Produção e envio

Antes de rodar o que grava em produção (file server, banco, DAG) ou envia e-mail, mensagem ou
ordem, diga o que vai acontecer e espere confirmação.

### Quem consome

Antes de mudar o que outros consomem (nome, pasta ou colunas de arquivo gerado, tabela,
assinatura de função pública), busque esse nome nos outros repos da org (ver
"Repositórios"), inclusive nas DAGs do `dataflow` (`dags/`). Repo não clonado fica fora dessa
busca; para ele, use `gh search code "<nome>" --owner v8capital`. Ao terminar, liste o que
achou, com arquivo e linha, e o que precisa mudar junto. Se não achou nada, diga onde
procurou.

## Claude

### CLAUDE.md

Não crie `CLAUDE.md` em subpastas. O `CLAUDE.md` do repo só muda com pedido e fala só do
repo.

### Rules do padrão

`.claude/rules/v8-capital/` vem do padrão da V8 e é atualizada pelo toolkit. Não edite aqui.
Se a pessoa pedir para mudar um comportamento que vem dessas rules, diga que a mudança vai no
padrão; se for algo só deste repo, escreva numa rule fora dessa pasta.
