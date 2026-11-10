# CLAUDE.md — artur

> Arquivo de memória deste repositório. Leia no início de cada sessão. Atualize quando descobrir algo relevante para sessões futuras.

## O que este repositório faz

Repositório pessoal do Artur na V8 Capital, para análises e protótipos. Não é repo da org: o
remote é `github.com/arturignarra-v8/artur`, nunca `github.com/v8capital`. Hoje tem só o
notebook de exploração `teste.ipynb`.

## Fluxo principal

Não há fluxo de produção. O código aqui roda à mão, em notebook ou script, e nada da V8 importa
deste repo.

## Módulos e responsabilidades

- `teste.ipynb`: exploração (pandas).
- `.claude/rules/v8-capital/`: rules do padrão da V8, copiadas do `toolkit`. Não edite aqui.

## Decisões de arquitetura

- Repo pessoal separado da org: rascunho e análise do Artur ficam fora de
  `C:\Github\v8capital`. A pasta `C:\Github\artur` nunca vai para o GitHub da V8.
- Segue o padrão da V8 (rules, `.gitignore`, `.gitattributes` com nbstripout) para que o
  código que sair daqui já esteja no formato dos repos da org.

## Dependências externas e quirks

- nbstripout: o filtro está configurado só neste repo (`git config` local), não no git global.
  Num clone novo, rode `python -m nbstripout --install` antes de commitar notebook.

## Pontos de atenção

- **Código que vai para a V8 passa por Pull Request, sempre, sem exceção.** Nada de commit ou
  push direto na `main` nem em branch de repo em `C:\Github\v8capital` sem Pull Request. Todo
  Pull Request tem como revisores Alexandre Lanutti (`alexandrelanutti-v8`) e Marcio Machado (`marciomachado-v8`);
  nunca sem os dois. Isso vale mesmo quando o `git.md` do padrão diz para trabalhar na branch
  em que a pessoa está.
- Levar algo daqui para um repo da V8 é copiar para o repo de destino, numa branch, e abrir o
  Pull Request lá. Nunca adicione `github.com/v8capital` como remote deste repo.
