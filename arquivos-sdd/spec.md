# Spec - Requisitos, Casos de Uso e Critérios de aceite

## Contrato (obrigatório, exato)
- Base URL: http://localhost:{PORTA_SERVICO}

# UC1 — Abrir bilhete
POST /bilhetes — body {"placa": "ABC1D23"} (7 caracteres alfanuméricos, maiúsculos) → 201 {"id": 1, "placa": "ABC1D23", "entrada": "<ISO-8601 com fuso -03:00>", "status": "aberto"}

- O body aceita entrada opcional (ISO-8601 com fuso): quando presente, o bilhete abre naquele instante em vez de "agora". É o gancho de testabilidade da correção — sem ele, testar fração/teto exigiria esperar tempo real. Formato inválido → 422 {"erro": "entrada_invalida"}.

## Regras
- `placa` é obrigatória
- Toda placa deve possuir 7 caracteres alfanuméricos, maiúsculos, na sequência de: 3 letras, 1 número, 1 letra, 2 números.
- A `entrada` deve respeitar o fuso definido <ISO-8601 -03:00>.
- Após o bilhete ser aberto, deve possuir o status `aberto`.
- Uma bilhete gerado deve iniciar a partir do momento atual.

## Sucesso 
- status: `201`.

## Critérios de Aceite 
- Para uma placa válida que não possui bilhete previamente aberto, retornar status `201`.
- Para placa válida que já possui bilhete previamente aberto e tenta fazer nova abertura, retornar status: `422`.
- Para uma nova abertura de bilhete, o id deve ser gerado automaticamente.

# UC2 — Encerrar bilhete
POST /bilhetes/{id}/encerramento → 200:

{"id": 1, "placa": "ABC1D23", "entrada": "...", "saida": "...",
 "minutos": 95, "valor_centavos": 1250}

## Regras
- Cobra-se por fração de FRACAO_MINUTOS minutos, arredondando para cima (fração exata cobra 1 fração; 1 minuto a mais já cobra a fração seguinte);
- hora cheia = TARIFA_HORA_CENTAVOS; valor da fração = tarifa ÷ (60 ÷ FRACAO_MINUTOS);
- aplica-se o teto diário: valor_centavos nunca supera TETO_DIARIO_CENTAVOS;
- valor sempre em centavos, inteiro — a API nunca retorna ponto flutuante.[^por-que-centavos]
- Bilhete inexistente gera status: `409`.
- Apenas bilhete aberto pode ser encerrado. 
- A duração deve ser indicada em minutos inteiros, nunca fracionados. Caso seja um valor fracionado, sempre arredondar para cima. 
- A duração em minutos é calculada entre `entrada` e `saida`.
- A duração deve ser representada em minutos inteiros conforme o instante de entrada e saída.
- A cobrança deve usar as regras da seção 7.
- O valor deve ser inteiro em centavos.
- O valor final nunca pode superar `TETO_DIARIO_CENTAVOS`.

## Critérios de Aceite 
- Encerrar um bilhete aberto retorna `200`.
- A resposta contém `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`.
- O status deixa de ser `aberto`.
- Encerrar novamente o mesmo bilhete retorna `409` com `{"erro":"bilhete_ja_encerrado"}`.
- Bilhete inexistente retorna `404` com `{"erro":"bilhete_nao_encontrado"}`.
- O valor nunca excede o teto.

# UC3 — Listar ativos
GET /bilhetes/ativos → 200 com array dos bilhetes abertos, mais recentes primeiro.

## Critérios de aceite
- Bilhetes encerrados não aparecem.
- Bilhetes cancelados não aparecem.
- Bilhetes abertos aparecem.
- A ordenação é decrescente por abertura/recência.
- Sem bilhetes ativos, retorna `[]`.

# UC4 — Relatório diário
GET /relatorios/diario?data=AAAA-MM-DD → 200:

{"data": "2026-10-05", "total_bilhetes": 12,
 "faturamento_centavos": 8400, "tempo_medio_minutos": 47}

- `tempo_medio_minutos` considera apenas bilhetes encerrados no dia, arredondando 0,5 para cima.

# UC5 — Cancelar bilhete
POST /bilhetes/{id}/cancelamento → 200 com status: "cancelado". Só bilhetes abertos podem ser cancelados — sem cobrança (não gera saida nem valor_centavos).

# UC6 — Histórico por placa
GET /bilhetes?placa=ABC1D23 → 200 com array de todos os bilhetes da placa (qualquer status), mais recentes primeiro. Placa que nunca estacionou → array vazio.

# UC7 — Tolerância gratuita
Os primeiros TOLERANCIA_MINUTOS de um bilhete são grátis: duração ≤ tolerância → valor_centavos: 0. Passou da tolerância (mesmo por 1 minuto) → cobra integral desde o primeiro minuto — a tolerância não é descontada.

# UC8 — Uma vaga por placa
POST /bilhetes para placa que já tem bilhete aberto → 409 {"erro": "bilhete_em_aberto"}. Após encerrar ou cancelar, a placa volta a poder abrir.