# Testes

## Princípio: toda regra numérica deve ser testada com 3 valores próximos

Para cada limite definido pelo sistema, devem ser testados três pontos:

- **o valor exato do limite**;
- **um valor imediatamente abaixo**;
- **um valor imediatamente acima**.

## 1. Fração de cobrança (`FRACAO_MINUTOS`)

A cobrança deve utilizar arredondamento **para cima**. O enunciado determina que:

A fração exata cobra 1 fração; 1 minuto a mais já cobra a fração seguinte.

Portanto, devem ser testados os seguintes cenários:

- Duração **exatamente igual a 1 fração** - cobra **1 fração**.
- Duração de **1 fração + 1 minuto** - cobra **2 frações**.
- Duração de **1 minuto**, quando ainda está abaixo de uma fração - cobra **1 fração inteira**.
- Duração correspondente a um **múltiplo exato de frações** - não adiciona uma fração extra.
- Duração de **0 minutos** (entrada e saída no mesmo instante) - verificar se o sistema cobra alguma coisa ou retorna **zero**.

## 2. Tolerância (`TOLERANCIA_MINUTOS`)

A regra é explícita:

- **`≤ TOLERANCIA_MINUTOS`** - cobrança igual a **0**.
- Ultrapassou a tolerância, **mesmo por apenas 1 minuto** - a cobrança começa integralmente desde o primeiro minuto.

Testar:

- Duração **exatamente igual à tolerância** - `valor_centavos = 0`.
- Duração de **tolerância + 1 minuto** - cobra como se a tolerância não existisse, considerando o período integral desde o início.
- Quando `TOLERANCIA_MINUTOS = 0` - não existe período gratuito.

### Pegadinha importante

A tolerância **não deve ser descontada da duração**.

Por exemplo, não deve ser aplicada uma fórmula equivalente a:

```text
minutos_cobrados = minutos - tolerancia
```

É necessário incluir um teste que comprove que, ao ultrapassar a tolerância, a cobrança é feita **integralmente desde o primeiro minuto**.

## 3. Teto diário (`TETO_DIARIO_CENTAVOS`)

Devem ser testados os limites do teto:

- Duração longa cujo valor calculado **ultrapassa o teto** - resultado deve ser **exatamente o valor do teto**.
- Duração cujo valor calculado é **exatamente igual ao teto** - permanece no teto.
- Duração cujo valor fica **1 centavo abaixo do teto** - não deve sofrer limitação.

### Interação com a tolerância

Também deve ser testado o cenário em que:

- o período ultrapassa a tolerância;
- a cobrança é calculada;
- o valor calculado ultrapassa o teto.

É necessário documentar **a ordem de aplicação das regras**, ou seja, qual regra é aplicada primeiro e qual prevalece no resultado final.

## 4. Interação entre as regras

As regras de tolerância, fração e teto devem ser testadas também em conjunto, pois a ordem de aplicação pode alterar o resultado.

### Tolerância + fração

Testar um caso em que a duração ultrapassa a tolerância em apenas **1 minuto**.

O sistema deve:

- considerar que a tolerância foi ultrapassada;
- cobrar **desde o minuto 0**;
- aplicar a cobrança utilizando as frações.

### Fração + teto

Testar uma duração que gere **muitas frações** e faça o valor calculado ultrapassar o teto.

O resultado final deve ser limitado ao **teto diário**.

## 5. Relatório diário (`tempo_medio_minutos`)

### Arredondamento da média

O arredondamento deve considerar **0,5 como arredondamento para cima**.

Testar:

- uma situação em que a média resulte exatamente em **`.5`** - deve arredondar **para cima**;
- uma situação em que a média resulte em **`.4`** - deve arredondar **para baixo**.

### Definição de "no dia"

É necessário verificar qual data determina a inclusão do bilhete no relatório:

- **data de entrada**; ou
- **data de saída**.

Para isso, criar um bilhete que atravesse a meia-noite, e verificar em qual data esse bilhete aparece no relatório.

### Bilhetes cancelados

Verificar se bilhetes cancelados são considerados em:

- `total_bilhetes`;
- `faturamento`;
- `tempo_medio`.

Para isso, criar um cenário contendo pelo menos:

- um bilhete cancelado;
- um bilhete encerrado;

ambos no mesmo dia.