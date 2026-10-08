# Constitution - Zona Azul Digital

# 1. Regras operacionais obrigatórias 

- A API tem que respeitar exatamente os endpoints, métodos HTTP, formatos de saída e códigos de status definidos em `spec.md`.
- Valores monetários devem ser representados exclusivamente como centavos inteiros, nunca retornar valores monetários como ponto flutuante. 
- O valor da tarifa deve ser calculado com base em: `TARIFA_HORA_CENTAVOS`, `FRACAO_MINUTOS`, `TETO_DIARIO_CENTAVOS`, `TOLERANCIA_MINUTOS` e `PORTA_SERVICO`, fornecidos em `python scripts/variante.py`. Não substitua esses valores por constantes próprias.
- A cobrança por tempo deve ser arredondada para cima na granularidade de `FRACAO_MINUTOS`. 
- A tolerância tem os minutos iniciais grátis por bilhete (0, 10 ou 15). Passada a tolerância, é cobrado desde o primeiro minuto. 
- O teto diário é obrigatório, o valor cobrado por um bilhete nunca pode ultrapassar `TETO_DIARIO_CENTAVOS`.
- Apenas os bilhetes com status `aberto` podem ser cancelados sem cobrança, não gerando `saida` ou `valor_centavos`.
- O `tempo_medio_minutos` considera apenas bilhetes encerrados no dia, arredondando 0,5 para cima.
- Os primeiros `TOLERANCIA_MINUTOS` de um bilhete são grátis. Quando suração é menor ou igual a tolerância, `valor_centavos`: 0. Passou da tolerância (mesmo por 1 minuto): cobra integral desde o primeiro minuto. A tolerância não é descontada.
- Não é possível abrir um bilhete para uma placa que já possui bilhete `aberto`. Após encerrrar ou cancelar, a placa volta a poder abrir.
- A placa deve possuir 7 caracteres alfanuméricos, maiúsculos, na sequência de: 3 letras, 1 número, 1 letra, 2 números.
- Datas e horários devem preservar o fuso informado e as respostas de abertura devem utilizar ISO-8601 com fuso `-03:00`.
- A API não deve inventar comportamentos para situações que possuem erro explicitamente definido no contrato.
- Erros devem possuir o código HTTP e o corpo JSON exatamente conforme `spec.md`.

# 3. Contrato HTTP

| Método | Rota | Sucesso | Erros |
|---|---|---|---|
| `[POST]` | `/bilhetes ` | `201` | `422`, `409`|
| `[GET]` | `/bilhetes/{id}/encerramento` | `200` | 
| `[GET]` | `/bilhetes/ativos` | `200` | 
| `[GET]` | `/relatorios/diario?data=AAAA-MM-DD` | `200` |
| `[GET]` | `/bilhetes?placa=ABC1D23` | `200` | 
| `[DELETE]` | `/...` | `[STATUS]` | `[STATUS]` |

Os métodos, rotas, campos e códigos HTTP definidos no contrato
devem ser preservados.

# 4. Princípios obrigatórios
- Não inventar campos, rotas ou status.
- Em nenhuma hipótese alterar regras definidas no contrato.
- Não produzir efeitos colaterais em operações rejeitadas.
- Todas as regras críticas devem ser verificáveis por testes.
- Toda a aplicação roda dentro de um container, nada depende de instalação na máquina host.

# 5. Stack e Linguagem 
- Linguagem: Python 3.12
- Framework HTTP: FastAPI
- Servidor ASGI: Uvicorn
- Validação: Pydantic v2 (vem com FastAPI)
- Testes: pytest + httpx (TestClient)

# 6. Encapsulamento de componentes 
- config - Centraliza os parâmetros da variante
- schemas - Valida DTOs (placa, ISO-8601, data), não conhece: regras de valor, persistência
- router - HTTP ⇄ domínio + mapeia erro → status code, não conhece: como se calcula valor
- service: Todas as regras (fração, teto, tolerância, 1-vaga), não conhece: HTTP / JSON
- repository - Guarda e busca bilhetes em memória, não conehce: regras de negócio


