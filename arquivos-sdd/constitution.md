# Constitution - Zona Azul Digital

# 1. Regras operacionais obrigatórias 

- A API tem que respeitar exatamente os endpoints, métodos HTTP, formatos de saída e códigos de status definidos em `spec.md`.
- Valores monetários devem ser representados exclusivamente como centavos inteiros, nunca retornar valores monetários como ponto flutuante. 
- O valor da tarifa deve ser calculado com base em: `TARIFA_HORA_CENTAVOS`, `FRACAO_MINUTOS`, `TETO_DIARIO_CENTAVOS`, `TOLERANCIA_MINUTOS` e `PORTA_SERVICO`, fornecidos em `python scripts/variante.py`. Não substitua esses valores por constantes próprias.
- A cobrança por tempo deve ser arredondada para cima na granularidade de `FRACAO_MINUTOS`. 
- A tolerância tem os minutos iniciais grátis por bilhete (0, 10 ou 15). Passada a tolerância, é cobrado desde o primeiro minuto. 
- O teto diário é obrigatório, o valor cobrado por um bilhete nunca pode ultrapassar `TETO_DIARIO_CENTAVOS`.
- Apenas os bilhetes com status `aberto` podem ser cancelados sem cobrança, não gerando `saida` ou `valor_centavos`.