---
name: refinar-projeto
description: Refina GDD/TDD a partir de uma revisão: verifica consistência entre documentos, prioriza achados, reescreve seções, registra decisões e changelog, e separa protótipo de evolução.
---

# Refinar projeto

Transforma crítica em documento melhor. Usada depois de `revisar-gdd` / `revisar-tdd`, ou direto quando o usuário pede para "melhorar", "refinar" ou "evoluir" um GDD/TDD.

## Entrada
- Os documentos do projeto (GDD, TDD e quaisquer outros: escopo, ADRs, backlog).
- Um relatório de revisão, se houver. Se não houver, faça antes uma revisão rápida (passos principais de `revisar-gdd`/`revisar-tdd`) e mostre os achados.
- Estágio-alvo: para onde o documento deve ir (ex.: de rascunho → protótipo jogável).

## Processo

1. **Consistência entre documentos** (sempre que houver mais de um):
   - Mesmos nomes, números e valores em todos (vida, dano, preços, limites, nomes de sistemas).
   - Toda mecânica do GDD tem contrapartida no TDD e vice-versa.
   - Dependências bidirecionais: se A depende de B, B sabe disso.
   - Referências obsoletas a sistemas renomeados/removidos.
   - Liste conflitos com os trechos de cada lado; **não escolha o vencedor sozinho**.
2. **Triagem** – agrupe os achados em: *Aplicar agora* (claro, sem decisão de design), *Precisa de decisão* (vire pergunta com 2–3 opções e trade-offs), *Adiar* (vai para "Evolução futura").
3. **Decisões** – pergunte ao usuário só o que está em *Precisa de decisão*, em lote, com uma recomendação marcada. Se o usuário não estiver presente, aplique a recomendação e marque como `[DECISÃO PROVISÓRIA]`.
4. **Reescrita** – edite seção por seção:
   - Substitua vago por específico (números, critérios testáveis, exemplos).
   - Corte o que não serve a nenhum pilar ou ao escopo atual; mova para "Evolução futura" em vez de apagar ideias.
   - Mantenha a estrutura e a voz do documento; não reescreva o que não tinha problema.
5. **Separação protótipo × evolução** – cada documento termina com:
   - **Escopo do protótipo**: o mínimo para validar o core loop e os riscos maiores.
   - **Evolução futura**: o que fica para depois, com o gatilho que justificaria implementar.
   - **Hipóteses a validar** no protótipo e como medir cada uma.
6. **Registro** – no fim de cada documento, mantenha:
   - `Changelog` (versão, data, o que mudou, por quê, ID do achado de origem).
   - `Registro de decisões` (decisão, alternativas, motivo) – ou ADRs separados para o TDD.
7. **Verificação** – releia o resultado: os achados Críticos e Maiores foram resolvidos ou viraram perguntas explícitas? Algum número novo conflita com outro documento?

## Saída
- Os documentos atualizados, no mesmo formato e lugar de origem (arquivo editado no próprio formato; doc atualizado no mesmo doc).
- Um resumo curto: achados resolvidos, decisões tomadas (e quais são provisórias), itens adiados, perguntas ainda abertas.

## Regras
- Não invente requisitos novos de design sem marcar como sugestão.
- Nunca apague conteúdo do usuário sem registrar no changelog.
- Uma rodada de refino resolve os bloqueadores; não tente perfeição. Sugira nova revisão quando as mudanças forem grandes.
