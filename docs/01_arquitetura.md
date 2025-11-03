# 01 — Arquitetura

## Componentes principais
- **`ganttMaster.js`** — núcleo: estado global (tarefas, links, permissões), calendário e flags.
- **`ganttTask.js`** — modelo/comportamento de tarefas (datas, duração, dependências).
- **`ganttDrawerSVG.js`** — renderização do Gantt em SVG (barras, baseline, links).
- **`ganttGridEditor.js`** — editor em grid (colunas/inputs acoplados às tarefas).
- **`ganttZoom.js`** — níveis de zoom (dia/semana/mês/…); marcação de feriados.
- **`ganttUtilities.js`** — utilidades (datas, layout, operações de cálculo).
- **`libs/i18nJs.js`** — textos/formatos + **`isHoliday(date)`** (lógica de calendário).
- **`gantt.html`** — página de exemplo e bootstrap do componente.

## Fluxo (alto nível)
1. Carrega dependências JS/CSS.
2. Instancia `GanttMaster` e elementos DOM (grid + gantt).
3. Carrega dados (`ge.loadProject(project)`), incluindo calendário.
4. Renderiza grid e gráfico (SVG) conforme zoom atual.
5. Interações: mover/redimensionar tarefas, criar links, editar no grid.
6. Baseline: quando populada e `showBaselines=true`, desenha barras de baseline atrás das atuais.

## Estruturas destacadas (em `GanttMaster`)
- **Calendário:**
  - `calendarHolidays: string[]` — datas `YYYY-MM-DD` não‑trabalhadas.
  - `calendarWorkdayInHoliday: string[]` — exceções (dias não‑úteis que serão considerados úteis).
  - `calendarDontWorkInSaturday: boolean`
  - `calendarDontWorkInSunday: boolean`
- **Baseline:**
  - `baselines: { [taskId: string]: { taskId, start, end, status, progress } }`
  - `showBaselines: boolean`
  - `baselineMillis: number` — timestamp da captura/versão.

## Dados do projeto (exemplo)
Em `gantt.html`, a função `getDemoProject()` define:

```js
{
  "calendarHolidays": ["2025-11-10"],
  "calendarWorkdayInHoliday": [],
  "calendarDontWorkInSaturday": true,
  "calendarDontWorkInSunday": true,
  "tasks": [ /* ... */ ]
}
```

> Observação: o dataset real também pode vir de `project_data.json` ou back‑end.
