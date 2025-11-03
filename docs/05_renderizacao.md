# 05 — Renderização (SVG) e estilos

## Como desenha
- O módulo `ganttDrawerSVG.js` cria um **SVG** por tarefa e renderiza:
  - Barra da tarefa (atual).
  - (Se habilitado) Barra de **baseline** via `_createBaselineSVG(task, baseline)`.
  - Handles de link/redimensionamento, rótulos, tooltips (via atributos `data-label`).

## Baseline no SVG
- Quando `master.showBaselines === true` e existe `master.baselines[task.id]`:
  - Cria um **retângulo** posicionado por `baseline.startDate`/`baseline.endDate`.
  - Aplica **opacidade** (~0.5) e **altura** reduzida (`taskHeight / 1.5`).
  - Monta **tooltip** com `Name`, `@baselineMillis`, `Start`, `End`, `Duration`, `Status`, `Progress`.

## Estilos/CSS
- Classes principais em `gantt.css` e `platform.css`.
- Células marcadas com classe `holy` para FDS/feriados (via `ganttZoom.js`).
- Ajuste de cores e opacidades pode ser feito via regras CSS adicionais.

## Performance
- Cada re-render considera **janela visível** (linhas visíveis).
- Evite gatilhos de redesenho global após cada pequena edição; agrupe operações quando possível.
