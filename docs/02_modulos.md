# 02 — Módulos

## `ganttMaster.js` (núcleo)
- Mantém **estado global** (tarefas, links, permissões).
- Expõe flags/configurações de calendário e baseline.
- Define **horário de trabalho** via `setHoursOn(startWorkingHour, endWorkingHour, dateFormat, resolution)`
  - `Date.startWorkingHour`, `Date.endWorkingHour`, `Date.useMillis`, `Date.workingPeriodResolution`.

### Campos relevantes
- `calendarHolidays: string[]`
- `calendarWorkdayInHoliday: string[]`
- `calendarDontWorkInSaturday: boolean`
- `calendarDontWorkInSunday: boolean`
- `baselines: { [taskId]: { taskId, start, end, status, progress } }`
- `showBaselines: boolean`
- `baselineMillis: number`

## `ganttTask.js` (tarefas)
- Modelo de **tarefa**: datas, duração, percentuais, hierarquia (pais/filhos), dependências.
- Métodos para **cálculo de datas** considerando dias úteis.

## `ganttDrawerSVG.js` (renderização SVG)
- Desenha **barras de tarefas** e **links**.
- Se `showBaselines` ativo e houver entrada em `baselines[task.id]`, chama `_createBaselineSVG(task, baseline)` para **barras de baseline** (opacidade reduzida, tooltip com `start`, `end`, `progress`, `status` e data da captura `baselineMillis`).

## `ganttGridEditor.js` (grid/editor)
- Cria e sincroniza o **grid** (colunas editáveis) com o Gantt.
- Eventos de edição disparam **re-renders** de trechos do gráfico.

## `ganttZoom.js` (zoom/calendário visual)
- Define **níveis de zoom** (1D, 3D, 1W, 1M, 1Q, etc.).
- Usa `isHoliday(date)` para marcar células com classe **`holy`** (finais de semana e feriados) e destacar bordas (mudança de mês/semana).

## `ganttUtilities.js` (utilidades)
- Funções de **cálculo de período** (ex.: ajustar fim por duração em dias úteis).
- Helpers de layout e grid.

## `libs/i18nJs.js`
- Textos/labels e **`function isHoliday(date)`**:
  - Retorna `true` para domingos/sábados (conforme flags `calendarDontWorkInSaturday/Sunday`) e para datas presentes em `ge.calendarHolidays`.
  - Retorna `false` para exceções listadas em `ge.calendarWorkdayInHoliday`.
