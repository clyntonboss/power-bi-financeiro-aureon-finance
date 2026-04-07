# Documentação das Medidas
## Projeto Financeiro — Aureon Finance

Este documento lista todas as medidas criadas no modelo Power BI, suas regras de negócio, dependências e retornos esperados.
<br>

[← Voltar ao Projeto](https://github.com/clyntonboss/power-bi-financeiro-aureon-finance)
<br>

Índice das Medidas:
- 💼 [Transações](#-medidas-de-transações)
- 💰 [Receita](#-medidas-de-receita)
- 💸 [Despesas](#-medidas-de-despesas)
- 💱 [Empréstimos](#-medidas-de-empréstimos)
- 💵 [Investimentos](#-medidas-de-investimentos)
- 💬 [Títulos](#-medidas-de-títulos)

---

## 💼 Medidas de Transações
[← Topo](#documentação-das-medidas)
<br>

```DAX
transacoes = 

-- Medida:
--      transacoes
--
-- Descrição:
--      Calcula o total geral de todas as transações registradas na tabela financeira, somando o valor absoluto de cada registro sem filtragem por tipo de transação.
--
-- Tabela origem:
--      fato_transacoes
--
-- Regra de negócio:
--      - Soma a coluna [valor] de todos os registros.
--      - Permite avaliar o total financeiro global dentro do contexto de filtros aplicado no relatório.
--
-- Dependência:
--      fato_transacoes[valor]
--
-- Retorno:
--      Número decimal representando o total de todas as transações no contexto filtrado.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja registros.

VAR _Resultado =
    SUM(
        fato_transacoes[valor]
    )

RETURN
    _Resultado
```

## 💰 Medidas de Receita
[← Topo](#documentação-das-medidas)
<br>

```DAX
transacao_receitas = 

-- Medida:
--      transacao_receita
--
-- Descrição:
--      Calcula o total de receitas registradas na tabela financeira, considerando apenas os registros onde a transação é "Receitas".
--
-- Tabela origem:
--      fato_transacoes
--
-- Regra de negócio:
--      - Soma a coluna [valor] apenas para transações classificadas como "Receitas".
--      - Permite avaliar o total de receitas dentro do contexto de filtros aplicado no relatório.
--
-- Dependência:
--      Medida: transacoes
--      dimensao_tipos[tipo]
--
-- Retorno:
--      Número decimal representando o total de receitas no contexto filtrado.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja registros.

VAR _Resultado =
    CALCULATE(
        [transacoes],
        dimensao_tipos[tipo] = "Receitas"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
maior_menor_receita = 

-- Medida:
--      maior_menor_receita
--
-- Descrição:
--      Destaca o valor da receita do mês que apresenta o maior ou o menor total, considerando o contexto de filtros aplicado no relatório.
--
-- Tabela origem:
--      dimensao_calendario
--      Tabela de medidas [transacao_receitas]
--
-- Regra de negócio:
--      - Calcula a menor e a maior receita entre os meses selecionados.
--      - Retorna o valor da receita apenas se for o menor ou o maior do período.
--
-- Dependência:
--      dimensao_calendario[mes_abreviado]
--      dimensao_calendario[mes_numero]
--      Medida: [transacao_receitas]
--
-- Retorno:
--      Número decimal representando a receita do mês com maior ou menor valor dentro do contexto filtrado.

VAR _MenorReceita =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacao_receitas]
    )

VAR _MaiorReceita =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacao_receitas]
    )

VAR _Resultado =
    IF(
        [transacao_receitas] = _MaiorReceita ||
        [transacao_receitas] = _MenorReceita,
        [transacao_receitas]
    )

RETURN
    _Resultado
```
<br>

```DAX
meta_maxima_receita = 

-- Medida:
--      meta_maxima_receita
--
-- Descrição:
--      Calcula a proporção da receita atual em relação à meta máxima definida, considerando o mesmo período do ano anterior e multiplicando por 1,40 para ajustar o objetivo de crescimento máximo.
--
-- Tabela origem:
--      dimensao_calendario
--      Tabela de medidas [transacao_receitas]
--
-- Regra de negócio:
--      - Compara a receita do período atual com a receita do mesmo período do ano anterior.
--      - Ajusta a referência do ano anterior multiplicando por 1,40 (meta máxima de crescimento de 40%).
--      - Retorna o resultado da divisão, representando a % da meta máxima atingida.
--
-- Dependência:
--      dimensao_calendario[data]
--      Medida: [transacao_receitas]
--
-- Retorno:
--      Número decimal representando a proporção da meta máxima atingida.
--      Ex.: 1 = 100% da meta máxima, 1,2 = 120% da meta máxima.
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() caso não haja dados.

VAR _ReceitaAtual =
    [transacao_receitas]

VAR _ReceitaAnoAnteriorMetaMax =
    CALCULATE(
        [transacao_receitas],
        SAMEPERIODLASTYEAR('dimensao_calendario'[Data])
    ) * 1.4

VAR _Resultado =
    DIVIDE(
        _ReceitaAtual,
        _ReceitaAnoAnteriorMetaMax
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
meta_atingida_receita = 

-- Medida:
--      meta_atingida_receita
--
-- Descrição:
--      Calcula a proporção da receita atual em relação à meta definida, considerando o mesmo período do ano anterior e multiplicando por 1,25 para ajustar o objetivo de crescimento.
--
-- Tabela origem:
--      dimensao_calendario
--      Tabela de medidas [transacao_receitas]
--
-- Regra de negócio:
--      - Compara a receita do período atual com a receita do mesmo período do ano anterior.
--      - Ajusta a referência do ano anterior multiplicando por 1,25 (meta de crescimento de 25%).
--      - Retorna o resultado da divisão, representando a % da meta atingida.
--
-- Dependência:
--      dimensao_calendario[data]
--      Medida: [transacao_receitas]
--
-- Retorno:
--      Número decimal representando a proporção da meta atingida.
--      Ex.: 1 = 100% da meta, 0,8 = 80% da meta.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja dados.

VAR _ReceitaAtual =
    [transacao_receitas]

VAR _ReceitaAnoAnteriorMeta =
    CALCULATE(
        [transacao_receitas],
        SAMEPERIODLASTYEAR('dimensao_calendario'[Data])
    ) * 1.25

VAR _Resultado =
    DIVIDE(
        _ReceitaAtual,
        _ReceitaAnoAnteriorMeta
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## 💸 Medidas de Despesas
[← Topo](#documentação-das-medidas)
<br>

```DAX
transacao_despesas =

-- Medida:
--      transacao_despesas
--
-- Descrição:
--      Calcula o total de despesas registradas na tabela financeira, considerando apenas os registros onde a transação é "Despesas".
--
-- Tabela origem:
--      fato_transacoes
--
-- Regra de negócio:
--      - Soma a coluna [valor] apenas para transações classificadas como "Despesas".
--      - Permite avaliar o total de despesas dentro do contexto de filtros aplicado no relatório.
--
-- Dependência:
--      Medida: transacoes
--      dimensao_tipos[tipo]
--
-- Retorno:
--      Número decimal representando o total de despesas no contexto filtrado.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja registros.

VAR _Resultado =
    CALCULATE(
        [transacoes],
        dimensao_tipos[tipo] = "Despesas"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
maior_menor_despesa = 

-- Medida:
--      maior_menor_despesa
--
-- Descrição:
--      Destaca o valor da despesa do mês que apresenta o maior ou o menor total, considerando o contexto de filtros aplicado no relatório.
--
-- Tabela origem:
--      dimensao_calendario
--      Tabela de medidas [transacao_despesas]
--
-- Regra de negócio:
--      - Calcula a menor e a maior despesa entre os meses selecionados.
--      - Retorna o valor da despesa apenas se for o menor ou o maior do período.
--
-- Dependência:
--      dimensao_calendario[mes_abreviado]
--      dimensao_calendario[mes_numero]
--      Medida: [transacao_despesas]
--
-- Retorno:
--      Número decimal representando a despesa do mês com maior ou menor valor dentro do contexto filtrado.

VAR _MenorDespesa =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacao_despesas]
    )

VAR _MaiorDespesa =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacao_despesas]
    )

VAR _Resultado =
    IF(
        [transacao_despesas] = _MaiorDespesa ||
        [transacao_despesas] = _MenorDespesa,
        [transacao_despesas]
    )

RETURN
    _Resultado
```
<br>

```DAX
percentual_despesas = 

-- Medida:
--      percentual_despesas
--
-- Descrição:
--      Calcula o percentual das despesas em relação à receita total, considerando o contexto de filtros aplicado no relatório.
--
-- Tabela origem:
--      Medidas: [transacao_despesas], [transacao_receita]
--
-- Regra de negócio:
--      - Divide o total de despesas pelo total de receita.
--      - Retorna a proporção das despesas em relação à receita.
--
-- Dependência:
--      Medida: [transacao_despesas]
--      Medida: [transacao_receita]
--
-- Retorno:
--      Número decimal representando o percentual de despesas sobre a receita.
--      Ex.: 0,25 = 25% das receitas são despesas.
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() caso a receita seja zero ou não haja dados.

VAR _Receita =
    [transacao_receitas]

VAR _Despesas =
    [transacao_despesas]

VAR _Resultado =
    DIVIDE(
        _Despesas,
        _Receita
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## 💱 Medidas de Empréstimos
[← Topo](#documentação-das-medidas)
<br>

```DAX
transacao_emprestimos = 

-- Medida:
--      transacao_emprestimos
--
-- Descrição:
--      Calcula o total de empréstimos registrados na tabela financeira, considerando apenas os registros onde a transação é "Empréstimos".
--
-- Tabela origem:
--      fato_transacoes
--
-- Regra de negócio:
--      - Soma a coluna [valor] apenas para transações classificadas como "Empréstimos".
--      - Permite avaliar o total de empréstimos dentro do contexto de filtros aplicado no relatório.
--
-- Dependência:
--      Medida: transacoes
--      dimensao_tipos[tipo]
--
-- Retorno:
--      Número decimal representando o total de empréstimos no contexto filtrado.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja registros.

VAR _Resultado =
    CALCULATE(
        [transacoes],
        dimensao_tipos[tipo] = "Empréstimos"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## 💵 Medidas de Investimentos
[← Topo](#documentação-das-medidas)
<br>

```DAX
transacao_investimentos = 

-- Medida:
--      transacao_investimentos
--
-- Descrição:
--      Calcula o total de investimentos registrados na tabela financeira, considerando apenas os registros onde a transação é "Investimentos".
--
-- Tabela origem:
--      fato_transacoes
--
-- Regra de negócio:
--      - Soma a coluna [valor] apenas para transações classificadas como "Investimentos".
--      - Permite avaliar o total de investimentos dentro do contexto de filtros aplicado no relatório.
--
-- Dependência:
--      Medida: transacoes
--      dimensao_tipos[tipo]
--
-- Retorno:
--      Número decimal representando o total de investimentos no contexto filtrado.
--
-- Observação:
--      COALESCE ou +0 é utilizado para evitar retorno BLANK() caso não haja registros.

VAR _Resultado =
    CALCULATE(
        [transacoes],
        dimensao_tipos[tipo] = "Investimentos"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## 💬 Medidas de Títulos
[← Topo](#documentação-das-medidas)
<br>

```DAX
titulo_principais_despesas = 
0

-- Medida:
--      titulo_principais_despesas
--
-- Descrição:
--      Placeholder para título ou valor de destaque das principais despesas.
--      Atualmente retorna zero e deve ser substituída por uma lógica que represente o título ou destaque das despesas.
--
-- Tabela origem:
--      Nenhuma
--
-- Regra de negócio:
--      - Retorna valor fixo 0, funcionando apenas como placeholder.
--
-- Dependência:
--      Nenhuma
--
-- Retorno:
--      Número inteiro 0
--
-- Observação:
--      Medida inicial, deve ser atualizada com lógica real conforme necessidade.

"""" & SELECTEDVALUE(dimensao_detalhes[detalhe]) & """"
```
<br>

[← Voltar ao Projeto](https://github.com/clyntonboss/power-bi-financeiro-aureon-finance)
