# V8 Git

Vale quando a pessoa não deu instrução de git. Se ela pediu algo específico, siga o pedido.

## Antes de editar

Na primeira edição da sessão, rode `git fetch` e `git status` e aja pelo resultado:

- **Branch atrás do remoto:** avise e pergunte se pode rodar `git pull` antes de editar. Espere
  a resposta: editar em cima de versão velha vira conflito no push.
- **Branch que não é a `main`:** diga em qual branch a pessoa está, uma vez.
- **Mudança não commitada:** pode editar. Ela fica fora do commit (ver "O que entra").
- **`fetch` falhou** por rede ou credencial: avise uma vez e siga; não tente configurar
  credencial.

## Commit

### O que entra

- Adicione ao commit só os arquivos que você mudou nesta tarefa, pelo caminho
  (`git add <arquivo>`), não `git add .`. Se algum deles já tinha mudança da pessoa antes de
  você editar, diga que ela vai junto.
- Antes de commitar, diga em português simples o que entra no commit.
- Antes de commitar `.ipynb`, confira que o nbstripout está ativo:
  `git check-attr filter -- <notebook>.ipynb` mostra `filter: nbstripout`, e
  `git config filter.nbstripout.clean` devolve um comando. Se não estiver, avise a pessoa em
  vez de commitar. Não use `nbstripout --status`: ele só lê a configuração local do repo e não
  enxerga a instalação global.

### Mensagem

Todo commit segue o padrão da V8, Conventional Commits: `<tipo>(<escopo>): <descrição>`. Siga
este formato mesmo que os commits recentes do repo usem outro (como `:sparkles: feat:`): o
histórico tem formatos antigos e não é modelo.

- Tipo em inglês e minúsculo, um destes: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`,
  `build`, `ci`, `chore`, `revert`, `style`. Na dúvida: apagar código é `refactor`; mexer em
  dado, configuração ou arquivo do repo (`.gitignore`, `.gitattributes`) é `chore`;
  dependência e empacotamento são `build`.
- Escopo opcional: o módulo ou a pasta principal, em minúsculas.
- Descrição em português, começando em minúscula, com o verbo no presente (`adiciona`,
  `corrige`, `remove`) e sem ponto final. A primeira linha inteira cabe em 72 caracteres.
- Termos que a equipe usa em inglês ficam em inglês: commit, merge, Pull Request, branch,
  push, deploy, DAG, hook. Nome de arquivo, função ou pacote vai como está no código.
- A mensagem é só texto, sem emoji, nem como caractere nem como shortcode.
- Corpo opcional, depois de uma linha em branco, para o porquê.
- Mudança que quebra quem consome (ver "Quem consome", no `core.md`): `!` depois do tipo ou
  do escopo e, no corpo, `BREAKING CHANGE: <o que muda para quem usa>`.

```text
feat(investo): envia o carrego diário pela API de produção
fix(indice): corrige a data-base quando a B3 publica com atraso
chore: adiciona o filtro do nbstripout no .gitattributes
docs: explica no README como abrir um Pull Request
```

```text
feat(carteira)!: grava a posição diária em parquet

BREAKING CHANGE: quem lia posicao_diaria.csv passa a ler posicao_diaria.parquet, na mesma pasta.
```

## Push e histórico

- Push só com confirmação da pessoa naquele momento.
- Commit já enviado fica como está: para corrigir, faça um commit novo. `push --force`,
  `amend` ou `rebase` de commit enviado só com pedido explícito.

## Branch

Trabalhe na branch em que a pessoa está, inclusive na `main`. Branch nova ou worktree só se ela
pedir: trabalho em branch separada só chega na `main` com merge, e esse passo costuma ficar
esquecido. Quando ela pedir uma branch, ao terminar o trabalho lembre que falta o merge (ou o
Pull Request) para a `main`.

## Credenciais

Credencial nova vai em `.env` ou variável de ambiente, nunca em arquivo versionado. Antes de
commitar num repo com `.env`, confira com o git:

- `git check-ignore -q .env` sai com código 0 (o git ignora o arquivo);
- `git ls-files .env` não lista nada (o arquivo não está versionado).

Se uma das duas falhar, avise em vez de commitar. Se encontrar credencial versionada, não copie
o padrão e avise a pessoa.
