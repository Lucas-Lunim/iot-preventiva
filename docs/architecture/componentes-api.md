# Componentes da API de Monitoramento · C4 Nível 3

Recorte da ingestão. O caso de uso recebe `IRegistroOcorrencias` por injeção no construtor. A configuração da API vincula o contrato à implementação `RepositorioOcorrenciasSql`.

```mermaid
C4Component
    title IoT Preventiva - Componentes da API de Monitoramento
    System_Ext(gateway, "Gateway IoT", "Envia lotes de telemetria.")
    Container_Boundary(api, "API de Monitoramento") {
        Component(ingestao, "Endpoint de Ingestão", "REST", "Recebe o lote e converte o resultado em resposta HTTP.")
        Component(servico, "ServicoIngestao", "Caso de uso", "Valida leituras, avalia a janela crítica e coordena o registro.")
        Component(repo, "RepositorioOcorrenciasSql", "Adaptador SQL", "Implementa IRegistroOcorrencias e delimita a transação.")
    }
    ContainerDb(db, "Banco de Dados", "Banco relacional", "Persiste ocorrências e a fila de notificações.")
    Rel(gateway, ingestao, "Envia lote", "HTTPS")
    Rel(ingestao, servico, "Executa processarLote")
    Rel(servico, repo, "Usa IRegistroOcorrencias injetada no construtor")
    Rel(repo, db, "Executa a transação", "SQL")
    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```
