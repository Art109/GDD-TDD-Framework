---
name: revisar-tdd
description: Analisa e critica um Technical Design Document (TDD) de jogo ou software: rastreabilidade com o GDD, arquitetura, riscos, performance, decisões e prontidão para protótipo.
---

# Revisar TDD

Revisão técnica de um TDD. Pergunta central: **um programador consegue começar a implementar com isto, sem adivinhar, e a arquitetura aguenta evoluir do protótipo para o jogo real?**

**Referências:** Modelo de TDD — https://claude.ai/artifact/WeAqiznSvqvXFGYzH25cBH; guias de arquitetura [Godot](https://claude.ai/artifact/LgdBwNCGaE5RTLXgrHBzJD) e [Unity](https://claude.ai/artifact/5zGYdcdfMFWVWF4CaAYZZQ) (cópias em `docs/` do repositório GDD-TDD-Framework). Um TDD em outro formato é revisado pelo conteúdo, não pelos nomes das seções.

## Entrada
- O TDD inteiro. Se existir GDD correspondente, leia também (para rastreabilidade). Se o TDD segue um dos guias, leia o guia da engine.
- Estágio: `protótipo` (padrão), `vertical slice` ou `produção`. Num protótipo, exija clareza e pontos de extensão; não exija CI/CD, observabilidade ou plano de escala.
- Engine/stack, plataforma-alvo, tamanho do time, se informados.

## Processo

1. **Completude** – Presente / Fraca / Ausente:
   - Sempre: Resumo e decisões principais, Stack com versões fixadas, Plataformas, Objetivos e não-objetivos técnicos, Restrições, **Metas de qualidade** (3, em ordem, com cenário verificável), Adaptação do guia (sistemas, `GameContext`, Wirings, `GameEvents`, sub-cenas, taxonomia de estado, desvios), Rastreabilidade GDD → TDD, Visão de blocos (diagrama) e 2+ cenários de execução, Sistemas do core loop, Riscos técnicos com spike, ADRs, Pendências do GDD.
   - Slice+: Todos os sistemas do slice, Componentes e Dados, Conceitos transversais (save e versão, input, pausa, erros, logs, texto), Orçamentos de performance completos, Dívida técnica aceita, Testes e ferramentas com build e distribuição, Glossário técnico.
   - Produção+: Build/CI, Rede/backend, Segurança e anti-cheat, Telemetria, Localização, Migração de save entre versões.
   - Projeto com código existente: Estado atual e plano de migração.
2. **Rastreabilidade GDD → TDD** – cada mecânica/sistema do GDD tem dono técnico? Há sistema técnico sem origem no GDD (gold plating)? Termos do código seguem o glossário do GDD? Monte tabela mecânica → sistema/módulo.
3. **Aderência ao guia** – cada regra do guia está aplicada ou tem desvio registrado com motivo e gatilho de revisão? Desvio sem motivo é achado. Confira em especial: sem autoload/singleton, Resource/ScriptableObject só com dados, Main só com ciclo de vida, `GameEvents` só com 3+ ouvintes, componentes que não recebem o `GameContext`, exigências dos componentes visíveis no editor.
4. **Arquitetura** – responsabilidades claras e sem sobreposição; acoplamento (quem depende de quem, ciclos); fonte única da verdade para cada estado; padrões adequados ao tamanho (não impor ECS/microserviços num protótipo pequeno sem motivo); pontos de extensão para o que o GDD indica que vai crescer. A visão de blocos bate com as seções de sistemas? Os cenários de execução cobrem os fluxos que cruzam vários sistemas (troca de level, sub-cena, save, morte no meio de transição)?
5. **Decisões** – escolhas importantes têm ADR com contexto, alternativas e consequências (inclusive negativas)? Status válido (proposta, aceita, obsoleta, substituída)? ADR aceita foi reescrita em vez de substituída?
6. **Atributos de qualidade** – as metas de qualidade têm cenário verificável e ordem clara? Performance com orçamentos numéricos e máquina de referência (não "deve rodar bem"); confiabilidade (save corrompido, crash, falha de carga); manutenibilidade (organização, convenções, debug tools); testabilidade; segurança quando há online/dinheiro.
7. **Riscos técnicos** – tecnologia imatura ou desconhecida pelo time, dependências de terceiros e lock-in, partes de alta incerteza (IA, física, rede, procedural). Para cada risco: probabilidade, impacto, mitigação ou *spike*, contingência.
8. **Implementabilidade** – interfaces e dados descritos o bastante para dividir em tarefas? Todo parâmetro do GDD mora num `Data`? Ordem de implementação sugerida? Critérios de pronto por sistema?
9. **Caminho de evolução** – o que é descartável no protótipo e o que precisa ser fundação? Os atalhos estão na tabela de dívida técnica com quando pagar, ou escondidos?

## Saída (relatório)

```
# Revisão do TDD – <nome> (estágio: <...>)
Veredito: APROVADO | PRECISA DE REVISÃO | REVISÃO GRANDE | NÃO AVALIADO

## Resumo
## Completude (tabela)
## Rastreabilidade GDD → TDD (tabela; lacunas em destaque)
## Aderência ao guia (regras aplicadas, desvios registrados, desvios sem motivo)
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
