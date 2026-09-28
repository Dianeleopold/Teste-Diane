# Instruções para o Claude Code

Delegue tarefas médias e grandes a subagentes.
Tarefas simples e rápidas (uma edição pontual, uma pergunta direta) execute você mesmo.
Não use sempre o Fable.
Use o Opus 5.5 para tarefas mais simples.

## Distribuição de modelos
- Fable 5.1: arquitetura, bugs complexos e revisão de código.
- Opus 5.5: edições, testes, documentação e refatoração.
- Haiku 4.5: pesquisas e resumos.
- Especifique o modelo em cada chamada de agente.

## Delegação
- Agrupe tarefas relacionadas em um mesmo subagente; separe apenas o que é independente. Planeje antes de executar.
- Execute subagentes independentes em paralelo.
- Leia o relatório e confira nos arquivos os pontos críticos antes de concluir.
