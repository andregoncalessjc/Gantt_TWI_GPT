# Documentação Técnica — ___gantt_twi

> **Linguagem:** PT‑BR • **Público:** Desenvolvedores • **Escopo:** arquitetura, APIs internas, calendário (dias úteis/feriados), baseline, renderização e extensões.

## Estrutura dos documentos
- [01_arquitetura.md](01_arquitetura.md) — visão geral, fluxo e arquivos principais
- [02_modulos.md](02_modulos.md) — `ganttMaster.js`, `ganttTask.js`, `ganttDrawerSVG.js`, `ganttGridEditor.js`, `ganttZoom.js`, `ganttUtilities.js`
- [03_baseline.md](03_baseline.md) — conceito, estrutura de dados, salvar/carregar e desenhar baseline
- [04_calendario.md](04_calendario.md) — dias úteis, finais de semana, feriados e função `isHoliday`
- [05_renderizacao.md](05_renderizacao.md) — como o Gantt é desenhado (SVG), classes CSS e performance
- [06_api_interna.md](06_api_interna.md) — métodos de inicialização, utilitários, eventos e hooks
- [07_extensoes.md](07_extensoes.md) — pontos de extensão, roadmap, boas práticas

## Requisitos & dependências (alto nível)
- jQuery 3.1+ e jQuery UI 1.12+ (vide `gantt.html`)
- Bibliotecas locais: `libs/date.js`, `libs/i18nJs.js`, `libs/jquery/dateField`, `libs/jquery/valueSlider/`
- CSS: `platform.css`, `gantt.css`, `ganttPrint.css`

## Demo e dados de exemplo
- `gantt.html` inicializa o componente e carrega um projeto de exemplo via `getDemoProject()`.
- `project_data.json` contém um dataset de exemplo.
- `test_holidays.html` demonstra a validação de feriados/finais de semana (`isHoliday`).

