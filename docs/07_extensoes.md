# 07 — Extensões, Roadmap e Boas Práticas

## Pontos de extensão sugeridos
- **Persistência de baseline** no back‑end com versionamento (rotular, listar, restaurar).
- **Calendário corporativo** centralizado (API REST), com cache local.
- **Caminho crítico** visual (marcação e filtro).
- **% Concluído por esforço** e por duração.
- **Impressão/Exportação** (PDF) com opções de colapso/zoom por período.
- **I18n completo** (valores, formatos, labels do grid).

## Boas práticas
- **Não** altere baseline capturada; crie **nova versão**.
- Mantenha listas de feriados em **UTC** (formato `YYYY-MM-DD`).
- Evite mix de datas em milissegundos e strings sem normalização.
- Agrupe operações de edição para reduzir re-renderizações.

## Diretrizes de contribuição
- Commits pequenos, mensagens claras.
- Pull Requests com descrição do impacto (UI, calendário, baseline).
- Testes manuais: `test_holidays.html` e cenários com baseline on/off.

## Próximos passos (propostos)
1. API REST para baseline (CRUD + rótulos).
2. Tela de configurações de calendário (com importação de feriados).
3. Modo impressão (faixa de datas + colunas selecionáveis).
4. Marcação visual de **desvios vs baseline** (adiantado/atrasado).
