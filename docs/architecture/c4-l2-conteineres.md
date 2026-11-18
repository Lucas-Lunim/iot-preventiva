# Auditoria C4 Nível 2 · IoT Preventiva

## Passo 1

1. A gravação depende do banco. Uma falha na única instância, caso a implantação tenha apenas uma, interrompe o registro de leituras, alertas e ordens. O gateway deve reter e reenviar o lote com a mesma identificação. A topologia de implantação continua a confirmar.
2. Enviar e-mail dentro da requisição estenderia a espera do gateway e faria a disponibilidade da ingestão depender do provedor. Na proposta, um worker consome uma fila persistente e reprocessa falhas fora da requisição.
3. A interrupção após a gravação pode deixar o gateway sem confirmação. Uma chave única de lote e a recuperação do resultado já registrado impedem que um reenvio repita o registro crítico.

A fronteira abaixo é uma proposta lógica para o laboratório. A fila de notificações é implementada como uma tabela de outbox no Banco de Dados. O Painel Web consulta os registros por meio da API e é o ponto de aceite da OS pelo técnico.

```mermaid
C4Container
    title IoT Preventiva - Contêineres da arquitetura proposta
    Person(tecnico, "Técnico de Manutenção", "Avalia alertas e aceita ordens de serviço.")
    System_Ext(gateway, "Gateway IoT", "Consolida e envia lotes de temperatura e vibração.")
    System_Boundary(iot, "IoT Preventiva") {
        Container(painel, "Painel Web", "Aplicação web", "Exibe alertas e ordens de serviço para avaliação.")
        Container(api, "API de Monitoramento", "API REST", "Valida lotes, avalia limiares e registra ocorrências.")
        ContainerDb(db, "Banco de Dados", "Banco relacional", "Guarda leituras, alertas, ordens de serviço e a fila persistente de notificações.")
        Container(worker, "Worker de Notificações", "Processo em segundo plano", "Consome a fila persistente e agenda retentativas de e-mail.")
    }
    System_Ext(email, "Serviço de E-mail", "Envia notificações de ocorrências críticas.")
    Rel(tecnico, painel, "Avalia alertas e aceita OS")
    Rel(painel, api, "Consulta alertas e atualiza OS", "HTTPS")
    Rel(gateway, api, "Envia lote identificado", "HTTPS")
    Rel(api, db, "Grava leituras e ocorrências", "SQL")
    Rel(worker, db, "Consome fila e registra resultado", "SQL")
    Rel(worker, email, "Solicita envio", "HTTPS")
    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```
