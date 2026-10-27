---
name: gerar-tdd
description: Cria o TDD de um jogo a partir do Modelo de TDD, do GDD e do guia de arquitetura da engine (Godot ou Unity), com rastreabilidade, adaptação do guia, riscos e ADRs.
---

# Gerar TDD

Cria o TDD que diz como **este** jogo aplica o guia de arquitetura aos sistemas do GDD. O TDD não repete as regras do guia: registra como o jogo as aplica e onde se desvia.

**Documentos (docs do Claude; ler com as ferramentas de docs):**
- Modelo de TDD — https://claude.ai/artifact/WeAqiznSvqvXFGYzH25cBH (cópia no repositório: `docs/modelo-tdd.md`)
- Arquitetura Modular para Godot — https://claude.ai/artifact/LgdBwNCGaE5RTLXgrHBzJD (cópia no repositório: `docs/arquitetura-godot.md`)
- Arquitetura Modular para Unity — https://claude.ai/artifact/5zGYdcdfMFWVWF4CaAYZZQ (cópia no repositório: `docs/arquitetura-unity.md`)
- Mapa do kit: `claude/kit-gdd-tdd.md` no Project.

## Entrada
- O GDD do jogo (obrigatório). Sem GDD, sugira `gerar-gdd` primeiro.
- Engine: Godot ou Unity. Se não estiver no GDD nem na conversa, pergunte.
- Estágio (o mesmo do GDD, salvo indicação).
- Código existente, se houver: leia-o para a seção Estado atual e migração.

## Processo
1. **Ler** o modelo de TDD, o GDD inteiro e o guia da engine inteiro.
2. **Rastreabilidade primeiro:** liste cada sistema e mecânica das Regras dos sistemas e do Escopo "dentro" do GDD. Cada um vira uma linha GDD → sistema técnico → componentes → dados → marco. Mecânica sem sistema = lacuna explícita. Não invente sistema que o GDD não pede.
3. **Adaptar o guia:** responda cada item do roteiro "Como adaptar a um projeto" do guia (2D/3D, sistemas e campos do `GameContext`, `Wiring` por domínio, `GameEvents` só com 3+ ouvintes, sub-cenas, taxonomia de estado de cada dado, save, versões). Qualquer desvio de regra do guia entra na tabela de desvios com motivo e gatilho de revisão; sem motivo, não há desvio.
4. **Sistemas:** uma subseção por sistema, com responsabilidade (e o que não faz), onde vive, interface, estado, casos-limite técnicos e "pronto quando" derivado dos critérios de aceitação do GDD.
5. **Dados:** todo parâmetro de balanceamento do GDD mora num `Data` (Resource ou ScriptableObject só com campos).
6. **Orçamentos:** números por métrica e máquina de referência; no protótipo, pelo menos FPS e plataforma-alvo.
7. **Riscos:** cada risco alto com spike (teste descartável, prazo curto) e contingência. Comece pelos riscos técnicos citados no GDD.
8. **ADRs:** uma por decisão difícil de reverter, surpreendente sem contexto ou com troca real (contexto, decisão, alternativas, consequências). Decisões técnicas do GDD migram para cá.
9. **Pendências do GDD:** perguntas de design que travam decisões técnicas, com o que cada uma trava. Ofereça levá-las também para as Perguntas em aberto do GDD.
10. **Criar o documento** como doc novo do Claude, título `TDD — <Nome> (<Estágio> v0.1)`, na ordem do modelo, seção por seção. Apague as linhas de orientação e os `<campos>`; a seção de migração sai em projeto novo.

## Perguntas
Pergunte só o que muda a arquitetura e não está no GDD nem no código (ex.: "há save no protótipo?", "o mundo fica visível atrás da batalha?"), em uma rodada no formato do grilling, com recomendação. Fatos do código ou da engine você descobre sozinho.

## Saída
- O TDD como doc do Claude, aberto para o usuário.
- Uma linha com as lacunas de rastreabilidade e as pendências, e o próximo passo: `revisar-tdd`, depois `teste-do-construtor` e `plano-prototipo`.

## Regras
- Nunca altere o modelo nem os guias. Se a adaptação revelar uma regra do guia que não funciona, anote como observação (`observador-skills`) e diga ao usuário; mudar o guia é decisão dele.
- Se um documento não puder ser lido, diga qual e peça o link.
