# Checklist da auditoria · IoT Preventiva

| Verificação | Evidência ou ação |
| --- | --- |
| Consistência dos nomes | Gateway IoT, API de Monitoramento, Banco de Dados, Worker de Notificações e Serviço de E-mail têm os mesmos nomes no C4 L2 e na sequência. O Painel Web aparece no C4 L2 porque é o canal de aceite da OS e fica fora do recorte de ingestão e envio de e-mail. |
| Sintaxe e renderização | Copiar o conteúdo dentro de cada bloco Mermaid para o [Mermaid Live Editor](https://mermaid.live/) ou abrir os arquivos Markdown após o commit no GitHub. |
| Interfaces e injeção de dependência | C4 L3 e classes mostram ServicoIngestao, IRegistroOcorrencias e RepositorioOcorrenciasSql. A implementação é configurada na API e injetada no construtor do caso de uso. |
| Rastreabilidade no Git | Adicionar a pasta `docs` com `architecture/` e `adr/` ao repositório da equipe. Confirmar a renderização nos arquivos do commit. |

## Cenários para mostrar na apresentação

1. Lote normal. As leituras são gravadas e a API confirma o recebimento.
2. Desvio crítico confirmado. Alerta e OS são registrados juntos. A OS aguarda aceite humano.
3. Banco passa de 3 s. O gateway reenvia o lote com o mesmo ID. Se a primeira gravação tiver sido confirmada, o resultado anterior é reaproveitado.
4. E-mail indisponível. O worker agenda nova tentativa. A OS já aparece no painel.

## Evidência do processo

Registre quais artefatos serviram de entrada, os riscos encontrados e como conferiu os resultados. O laboratório pode ser feito pelo navegador. Só registre uso de integração MCP se o agente tiver de fato chamado uma ferramenta conectada. A renderização manual no editor constitui a etapa de validação visual.

## Fontes

- [Sprint 1 · Monitoramento IoT de Manutenção Preventiva](https://chatgpt.com/library?file_id=libfile_3879ab4d23dc819184b1a5db705f4bd2&account_id=personal)
- [Sintaxe C4 em Mermaid](https://mermaid.js.org/syntax/c4.html)
- [Diagrama de sequência em Mermaid](https://mermaid.js.org/syntax/sequenceDiagram.html)
- [Architecture Decision Records · Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
