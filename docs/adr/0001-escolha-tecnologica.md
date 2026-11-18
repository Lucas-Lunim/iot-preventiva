# ADR 0001 · Registrar Alerta e OS na mesma transação

## Status

Aceito como decisão da modelagem do fluxo IoT Preventiva.

## Contexto

Após receber um lote válido de temperatura e vibração, o sistema verifica a persistência do desvio. Se a condição for crítica, o técnico deve encontrar a ordem de serviço e o alerta que a originou. Uma falha durante a gravação pode deixar registros incompletos. Um timeout pode levar o gateway a reenviar o lote.

## Decisão

Usar uma transação do banco de dados relacional para gravar as leituras processadas, o Alerta e a Ordem de Serviço (OS) referentes à ocorrência crítica. Criar a OS com o status “Aguardando aceite”. Deduplicar reenvios por identificador de lote no escopo de cada gateway e devolver o resultado anterior quando o mesmo lote já estiver registrado.

O desenho de resiliência do laboratório propõe acrescentar um evento de notificação na mesma transação, em uma tabela de outbox. O envio de e-mail será feito por um processo em segundo plano após a confirmação da transação.

## Consequências positivas

- Alerta e OS são gravados juntos e permanecem rastreáveis a partir do lote processado.
- O reenvio de um lote já gravado pode retornar seu resultado sem abrir outra OS.
- Na extensão proposta, o técnico pode avaliar a OS no painel mesmo quando o e-mail estiver indisponível.

## Consequências negativas

- A persistência depende da disponibilidade do banco de dados.
- O gateway precisa guardar lotes temporariamente e manter o identificador durante reenvios.
- A extensão com outbox exige monitorar eventos pendentes, retentativas e falhas terminais. A entrega por e-mail é eventual.
