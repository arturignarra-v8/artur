---
paths:
  - "**/*.py"
  - "**/*.ipynb"
---

# V8 Python

Erro de data ou de caminho raramente quebra o processo: muda o número ou grava o arquivo no
lugar errado, e o resultado parece plausível. Boa parte das regras abaixo existe por isso.

## Convenções

### Nomes

Nome de arquivo e de módulo sem espaço, hífen nem acento: arquivo assim não pode ser importado.

### Idioma

Termos de programação em inglês (`get`, `list`, `find`, `load`, `save`, `parse`), termos do
mercado em português (`carteira`, `fundo`, `cota`, `cotista`): `get_carteira`,
`list_fundos`. Não renomeie código existente só para seguir isto.

### Módulos

- Nada roda no import: o módulo só define funções, classes e constantes que não dependem de
  rede, disco ou raiz do file server. O ponto de entrada fica em
  `if __name__ == "__main__":` ou em `scripts/`. Os repos são importados pelo Airflow.
- Antes de copiar uma função ou arquivo de outro lugar, importe. Cópia diverge.

### Exceções

Não engula exceção: deixe subir, ou trate com uma decisão explícita (valor padrão
documentado, pular o item avisando). Registrar e seguir sem decidir também esconde a falha, e
ela vira número errado mais adiante.

## Dados

### Datas e dias úteis

Detalhes do `Calendar`; a regra de usá-lo está no `core.md`. Para contagem de pregão, e não
de dia útil bancário, use `Calendar(use_manual_b3_holidays=True)`.

```python
from v8_utilities.anbima_calendar import Calendar

def calcula(...):
    # dentro da função: o construtor lê o file server e pode baixar da ANBIMA e regravar
    # o arquivo de feriados
    calendar = Calendar()
    calendar.is_workday(data)
    calendar.date_n(data, -5)                      # 5 dias úteis antes
    calendar.count_days(inicio, fim)               # um par de datas
    calendar.count_days_vec(inicios, fins)         # colunas de pares, vetorizado
    calendar.get_last_workday(data)
    calendar.get_end_of_month(data)
```

**Cada função ajusta de um jeito a data que não é dia útil:**

| Função | O que faz com data que não é dia útil |
|---|---|
| `count_days(inicio, fim)` | início **e fim** vão para o próximo dia útil |
| `count_days_vec(inicios, fins)` | início vai para o próximo dia útil; fim **volta** para o anterior |
| `date_n(data, n)` | com `n > 0`, volta para o dia útil anterior antes de somar; com `n < 0`, vai para o próximo; com `n = 0`, devolve a data sem ajuste (para normalizar, use `get_last_workday`) |

**Armadilhas:**

1. **`count_days` e `count_days_vec` não são intercambiáveis.** Quando o fim não é dia útil,
   o resultado difere. De segunda 05/10/2026 a sábado 10/10/2026, `count_days` dá 6 (o fim vai
   para terça 13, porque 12/10 é feriado) e `count_days_vec` dá 5. De sábado a sábado, 1
   contra 0. Ao vetorizar um cálculo que usava `count_days`, ou o contrário, confira se as datas
   finais podem cair fora de dia útil e qual ajuste a regra de negócio quer.
2. **As duas contagens incluem as duas pontas.** `count_days(d, d)` é `1`; dois dias úteis
   seguidos dão `2`. Se a regra de negócio pede quantos dias se passaram, subtraia 1 (ou use
   `count_days_vec(..., inclusive=False)`). Confira qual das duas a fórmula quer.
3. **`date_n_vector` recebe uma única data base** e vários deslocamentos. Não serve para uma
   coluna de datas diferentes.
4. **Data é `datetime` ou `pd.Timestamp`, nunca string**, da entrada até a saída da regra de
   negócio. Comparação entre strings de data falha sem erro.

### Caminhos de arquivo

1. **Nunca escreva caminho de rede nem letra de drive fixos** (`\\192.168.0.22\...`, `Y:\`).
   Em container Linux, um caminho UNC vira o nome literal de uma pasta criada no diretório
   atual: a gravação "funciona" e o dado se perde. Já aconteceu em produção.
2. **Nunca guarde a raiz do file server numa constante de módulo** (`DIR = <raiz> / "X"` fora
   de função). Ela é avaliada no import, e as DAGs do `dataflow` trocam a raiz por variável de
   ambiente para rodar em teste. Com a constante, o teste grava em produção. Resolva o caminho
   dentro da função que o usa.
3. **Mecanismo de caminho:** use o que o arquivo ou o repo já usa (`PathV8` ou
   `file_server`). Não converta de um para o outro sem pedido. Se o repo ainda não tem nenhum,
   pergunte antes de escolher: os dois estão em revisão (em 09/10/2026), mesmo que a docstring de
   `file_server.raiz()` diga que ele é o preferido para código novo.
4. **Prefira `01_PARSED` a `00_RAW`.** RAW é o dado como veio da fonte. Leia RAW só para
   depurar o próprio parsing ou se a pessoa pedir.

Depois de mudar qualquer caminho, confira onde o arquivo realmente caiu.

## Sugestões

### Classes e módulos

Código que outros repos usam (cliente de API, fonte de dado, motor de cálculo) costuma ser
orientado a objeto, como um SDK: quem usa instancia uma classe e chama métodos, sem saber
de autenticação, arquivo ou banco. Os formatos que se repetem na V8:

- **Base e especialização** (Template Method): a classe base cuida do que é comum
  (autenticação, conexão, cache, onde salvar) e cada subclasse só do que é dela. Ex.: uma
  base por administrador e uma subclasse por relatório; uma base de fonte de dado com
  `load()` e uma subclasse por fonte.
- **Porta de entrada única** (Facade): uma classe reúne as partes e é o que os outros
  importam. Ex.: um snapshot de dados de mercado para uma data, que carrega as fontes na
  ordem certa.
- **Objeto do domínio:** ativo, carteira e fundo como classe com atributos nomeados, em
  vez de dict solto. Um construtor alternativo por origem (`from_ticker`, `from_api`).
- **Dependência recebida, não buscada:** o motor de cálculo recebe o ativo e os dados de
  mercado; o cliente recebe a conexão. Assim dá para testar e reaproveitar.
- **Carga explícita:** deixe claro quando a classe acessa rede, disco ou banco (`load()`,
  `load_on_init=`).
- **Erro com nome:** falha vira exceção específica, escolhida pelo que quem chama precisa
  fazer (credencial, sem permissão, não encontrado, tentar de novo).

Fora disso, prefira função: transformação de dado é função pura, que recebe o bruto e
devolve a tabela. Não crie classe só para agrupar funções, nem camadas para um script.

## Notebooks

- Notebook é exploração: código de produção nunca importa de notebook.
- Saída de notebook não vai para o git, porque pode ter posição de fundo. O `nbstripout`
  limpa a saída no commit. Notebook que precisa manter a saída, como um relatório, usa o
  metadado `keep_output`.
