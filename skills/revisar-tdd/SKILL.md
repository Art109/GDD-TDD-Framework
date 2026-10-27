---
name: revisar-tdd
description: Analisa e critica um Technical Design Document (TDD) de jogo ou software: rastreabilidade com o GDD, arquitetura, riscos, performance, decisões e prontidão para protótipo.
---

# Revisar TDD

Revisão técnica de um TDD. Pergunta central: **um programador consegue começar a implementar com isto, sem adivinhar, e a arquitetura aguenta evoluir do protótipo para o jogo real?**

## Entrada
- O TDD inteiro. Se existir GDD correspondente, leia também (para rastreabilidade).
- Estágio: `protótipo` (padrão), `vertical slice` ou `produção`. Num protótipo, exija clareza e pontos de extensão; não exija CI/CD, observabilidade ou plano de escala.
- Engine/stack, plataforma-alvo, tamanho do time, se informados.

## Processo

1. **Completude** – Presente / Fraca / Ausente:
   - Sempre: Objetivo e escopo técnico, Engine/stack e versão, Plataformas, Visão geral da arquitetura (diagrama ou lista de módulos), Sistemas principais com responsabilidades, Modelo de dados/estado, Fluxo do core loop em termos técnicos, Fora de escopo.
   - Slice+: Orçamentos de performance (FPS, memória, tempo de carga, tamanho do build), Save/persistência, Pipeline de assets, Input, Áudio, UI, Estratégia de testes, Riscos técnicos.
   - Produção+: Build/CI, Rede/backend, Segurança e anti-cheat, Telemetria/analytics, Localização, Plano de migração e versionamento.
2. **Rastreabilidade GDD → TDD** – cada mecânica/sistema do GDD tem dono técnico? Há sistema técnico sem origem no GDD (gold plating)? Monte tabela mecânica → sistema/módulo.
3. **Arquitetura** – responsabilidades claras e sem sobreposição; acoplamento (quem depende de quem, ciclos); fonte única da verdade para cada estado; padrões adequados ao tamanho (não impor ECS/microserviços num protótipo pequeno sem motivo); pontos de extensão para o que o GDD indica que vai crescer.
4. **Decisões** – escolhas importantes (engine, rede, persistência, padrão de arquitetura) têm justificativa e alternativas consideradas? Sugira registrar como ADR curto (contexto, decisão, consequências).
5. **Atributos de qualidade** – performance (orçamentos numéricos, não "deve rodar bem"), confiabilidade (save corrompido, crash, perda de conexão), manutenibilidade (organização, convenções, debug tools), testabilidade, segurança quando há online/dinheiro.
6. **Riscos técnicos** – tecnologia imatura ou desconhecida pelo time, dependências de terceiros e lock-in, partes de alta incerteza (IA, física, rede, procedural). Para cada risco: probabilidade, impacto, mitigação ou *spike* de protótipo para validar.
7. **Implementabilidade** – interfaces e dados descritos o bastante para dividir em tarefas? Ordem de implementação sugerida? Critérios de pronto por sistema?
8. **Caminho de evolução** – o que é descartável no protótipo e o que precisa ser fundação? Onde a dívida técnica é aceitável e está marcada como tal?

## Saída (relatório)

```
# Revisão do TDD – <nome> (estágio: <...>)
Veredito: APROVADO | PRECISA DE REVISÃO | REVISÃO GRANDE | NÃO AVALIADO

## Resumo
## Completude (tabela)
## Rastreabilidade GDD → TDD (tabela; lacunas em destaque)
## Pontos fortes
## Achados (Crítico / Maior / Menor / Sugestão)
## Riscos técnicos (tabela: risco · prob · impacto · mitigação/spike)
## Decisões que merecem ADR
## Perguntas em aberto
## Próximos passos (máx. 5, priorizados)
```

Cada achado: **[ID] Título** — Onde (seção + trecho) · Problema · Consequência prática · Recomendação concreta.

## Regras
- Cite o documento; nada de checklist genérico colado sem relação com o texto.
- Calibre ao estágio: não penalize um protótipo por não ter infraestrutura de produção, mas aponte se uma escolha atual vai impedir a evolução.
- Prefira a solução mais simples que atende o GDD; complexidade precisa de justificativa.
- Não edite o TDD durante a revisão. Para aplicar, use `refinar-projeto`.
