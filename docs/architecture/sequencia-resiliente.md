# Sequência resiliente · Ingestão de leituras IoT

## Passo 2

O gateway envia um lote identificado com leituras de temperatura e vibração. A API valida as leituras e verifica se o desvio ultrapassou o limite pelo período exigido. Para um desvio crítico confirmado, o Banco de Dados grava leituras, Alerta, OS em “Aguardando aceite” e evento de notificação na mesma transação. Lotes válidos sem desvio crítico registram somente as leituras.

```mermaid
sequenceDiagram
    participant Gateway as Gateway IoT
    participant API as API de Monitoramento
    participant DB as Banco de Dados
    participant Worker as Worker de Notificações
    participant Email as Serviço de E-mail

    Gateway->>API: Enviar lote com identificador estável
    API->>API: Validar lote e avaliar persistência do desvio
    API->>DB: Persistir lote e, se crítico, Alerta, OS e evento pendente
    alt Banco confirma dentro de 3 s
        DB-->>API: Commit confirmado ou resultado anterior para o mesmo lote
        API-->>Gateway: Confirmar recebimento
    else Banco não confirma em 3 s
        API-->>Gateway: Falha temporária, resultado ainda incerto
        Gateway->>Gateway: Reter lote e agendar reenvio com espera progressiva
        Note over Gateway,DB: Reenvio com o mesmo ID recupera resultado anterior ou grava uma vez
    end

    Note over DB,Worker: Fluxo independente, somente para evento já confirmado no banco
    opt Evento crítico pendente e elegível
        Worker->>DB: Reservar evento na fila persistente
        DB-->>Worker: Entregar evento com identificador estável
        Worker->>Email: Solicitar envio com ID do evento
        alt E-mail aceito pelo provedor
            Email-->>Worker: Confirmação de aceite
            Worker->>DB: Marcar evento como aceito
        else Serviço de E-mail indisponível
            Email-->>Worker: Falha temporária
            alt Há tentativas restantes
                Worker->>DB: Agendar nova tentativa com espera progressiva
            else Limite atingido
                Worker->>DB: Marcar falha terminal para revisão
            end
        end
    end
```

- **Timeout de 3 s.** É o limite didático de espera da API pelo banco. Um timeout deixa o resultado incerto. A chave do lote é única no escopo do gateway, com verificação de conteúdo. O gateway reenvia com a mesma chave. Se a primeira transação confirmou, a API devolve o resultado existente.
- **E-mail indisponível.** O worker lê a fila persistente independente da ingestão. O cenário propõe uma tentativa inicial e até três retentativas, com atrasos de 30 s, 2 min e 10 min. A falha final fica registrada para revisão. A OS permanece disponível no painel para aceite.
- **Entrega e duplicidade.** O worker reserva o evento de forma exclusiva por um prazo, de modo que uma interrupção permita retomada. O identificador do evento deve ser usado para deduplicação quando o provedor oferecer suporte. Se o provedor receber o envio e a confirmação se perder, a ausência de deduplicação pode gerar e-mail duplicado.
