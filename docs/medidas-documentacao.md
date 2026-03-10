# Documentação de Medidas - Projeto Aureon Finance

Este documento lista todas as medidas criadas no modelo Power BI, suas regras de negócio, dependências e retornos esperados.

---

## Medidas de Transações Financeiras

### transacao_receita

```DAX
VAR TotalReceitas =
    CALCULATE(
        SUM(fFinanceiro[valor_absoluto]),
        fFinanceiro[transacao] = "Receitas"
    )
RETURN
    COALESCE(TotalReceitas, 0)
```

Descrição: Calcula o total de receitas registradas na tabela financeira.

### transacao_despesas

```DAX
VAR TotalDespesas =
    CALCULATE(
        SUM(fFinanceiro[valor_absoluto]),
        fFinanceiro[transacao] = "Despesas"
    )
RETURN
    COALESCE(TotalDespesas, 0)
```

Descrição: Calcula o total de despesas registradas na tabela financeira.

### transacao_emprestimos

```DAX
VAR TotalEmprestimos =
    CALCULATE(
        SUM(fFinanceiro[valor_absoluto]),
        fFinanceiro[transacao] = "Empréstimos"
    )
RETURN
    COALESCE(TotalEmprestimos, 0)
```

Descrição: Calcula o total de empréstimos registrados na tabela financeira.

### transacao_investimentos

```DAX
VAR TotalInvestimentos =
    CALCULATE(
        SUM(fFinanceiro[valor_absoluto]),
        fFinanceiro[transacao] = "Investimentos"
    )
RETURN
    COALESCE(TotalInvestimentos, 0)
```

Descrição: Calcula o total de investimentos registrados na tabela financeira.

### transacoes

```DAX
VAR TotalTransacoes =
    SUM(fFinanceiro[valor_absoluto])
RETURN
    COALESCE(TotalTransacoes, 0)
```

Descrição: Calcula o total geral de todas as transações registradas na tabela financeira.

---

## Medidas de Meta e Proporção

### meta_atingida_receita

```DAX
VAR ReceitaAtual = [transacao_receita]
VAR ReceitaAnoAnteriorMeta =
    CALCULATE(
        [transacao_receita],
        SAMEPERIODLASTYEAR('dCalendario'[Data])
    ) * 1.25
VAR Resultado =
    DIVIDE(ReceitaAtual, ReceitaAnoAnteriorMeta)
RETURN
    COALESCE(Resultado, 0)
```

Descrição: Proporção da receita atual em relação à meta de crescimento de 25% em relação ao mesmo período do ano anterior.

### meta_maxima_receita

```DAX
VAR ReceitaAtual = [transacao_receita]
VAR ReceitaAnoAnteriorMetaMax =
    CALCULATE(
        [transacao_receita],
        SAMEPERIODLASTYEAR('dCalendario'[Data])
    ) * 1.4
VAR Resultado =
    DIVIDE(ReceitaAtual, ReceitaAnoAnteriorMetaMax)
RETURN
    COALESCE(Resultado, 0)
```

Descrição: Proporção da receita atual em relação à meta máxima de crescimento de 40% em relação ao mesmo período do ano anterior.

### percentual_despesas

```DAX
VAR Receita = [transacao_receita]
VAR Despesas = [transacao_despesas]
VAR Resultado =
    DIVIDE(Despesas, Receita)
RETURN
    COALESCE(Resultado, 0)
```

Descrição: Percentual das despesas em relação à receita total.

---

## Medidas de Maior e Menor Valores Mensais

### maior_menor_receita

```DAX
VAR _MenorReceita =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacao_receita]
    )
VAR _MaiorReceita =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacao_receita]
    )
VAR Resultado =
    IF(
        [transacao_receita] = _MaiorReceita ||
        [transacao_receita] = _MenorReceita,
        [transacao_receita]
    )
RETURN
    COALESCE(Resultado, 0)
```

Descrição: Destaca o valor da receita do mês que apresenta o maior ou o menor total.

### maior_menor_despesa

```DAX
VAR _MenorDespesa =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacao_despesas]
    )
VAR _MaiorDespesa =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacao_despesas]
    )
VAR Resultado =
    IF(
        [transacao_despesas] = _MaiorDespesa ||
        [transacao_despesas] = _MenorDespesa,
        [transacao_despesas]
    )
RETURN
    COALESCE(Resultado, 0)
```

Descrição: Destaca o valor da despesa do mês que apresenta o maior ou o menor total.

---

## Medidas Placeholder

### titulo_principais_despesas

```DAX
RETURN 0
```

Descrição: Placeholder para título ou valor de destaque das principais despesas. Atualmente retorna zero.
