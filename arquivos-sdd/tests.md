# Testes

## Princípio: toda regra numérica deve ser testada com 3 valores próximos

Para cada limite definido pelo sistema, devem ser testados três pontos:

* **o valor exato do limite**;
* **um valor imediatamente abaixo**;
* **um valor imediatamente acima**.

## 1. Fração de cobrança (`FRACAO_MINUTOS`)

A cobrança deve utilizar arredondamento **para cima**. O enunciado determina que:

A fração exata cobra 1 fração; 1 minuto a mais já cobra a fração seguinte.

Portanto, devem ser testados os seguintes cenários:

* Duração **exatamente igual a 1 fração** - cobra **1 fração**.
* Duração de **1 fração + 1 minuto** - cobra **2 frações**.
* Duração de **1 minuto**, quando ainda está abaixo de uma fração - cobra **1 fração inteira**.
* Duração correspondente a um **múltiplo exato de frações** - não adiciona uma fração extra.
* Duração de **0 minutos** (entrada e saída no mesmo instante) - verificar se o sistema cobra alguma coisa ou retorna **zero**.

## 2. Tolerância (`TOLERANCIA_MINUTOS`)

A regra é explícita:

* **`≤ TOLERANCIA_MINUTOS`** - cobrança igual a **0**.
* Ultrapassou a tolerância, **mesmo por apenas 1 minuto** - a cobrança começa integralmente desde o primeiro minuto.

Testar:

* Duração **exatamente igual à tolerância** - `valor_centavos = 0`.
* Duração de **tolerância + 1 minuto** - cobra como se a tolerância não existisse, considerando o período integral desde o início.
* Quando `TOLERANCIA_MINUTOS = 0` - não existe período gratuito.

### Pegadinha importante

A tolerância **não deve ser descontada da duração**.

Por exemplo, não deve ser aplicada uma fórmula equivalente a:

```text
minutos_cobrados = minutos - tolerancia
```

É necessário incluir um teste que comprove que, ao ultrapassar a tolerância, a cobrança é feita **integralmente desde o primeiro minuto**.

## 3. Teto diário (`TETO_DIARIO_CENTAVOS`)

Devem ser testados os limites do teto:

* Duração longa cujo valor calculado **ultrapassa o teto** - resultado deve ser **exatamente o valor do teto**.
* Duração cujo valor calculado é **exatamente igual ao teto** - permanece no teto.
* Duração cujo valor fica **1 centavo abaixo do teto** - não deve sofrer limitação.

### Interação com a tolerância

Também deve ser testado o cenário em que:

* o período ultrapassa a tolerância;
* a cobrança é calculada;
* o valor calculado ultrapassa o teto.

É necessário documentar **a ordem de aplicação das regras**, ou seja, qual regra é aplicada primeiro e qual prevalece no resultado final.