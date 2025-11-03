# 06 — API interna / Inicialização / Hooks

## Bootstrap (exemplo simplificado)
```html
<link rel="stylesheet" href="gantt.css">
<script src="jquery.js"></script>
<script src="jquery-ui.js"></script>
<script src="libs/date.js"></script>
<script src="libs/i18nJs.js"></script>
<script src="ganttUtilities.js"></script>
<script src="ganttTask.js"></script>
<script src="ganttMaster.js"></script>
<script src="ganttDrawerSVG.js"></script>
<script src="ganttGridEditor.js"></script>
<script src="ganttZoom.js"></script>

<div id="workSpace"></div>
<script>
  var ge = new GanttMaster();
  ge.init($("#workSpace")); // cria grid + gráfico
  ge.loadProject(getDemoProject()); // ou dados do back-end
  ge.checkpoint(); // limpa pilha de undo
</script>
```

## Calendário (definição em runtime)
```js
ge.calendarHolidays = ["2025-01-01","2025-04-21"];
ge.calendarWorkdayInHoliday = ["2025-12-21"]; // ex.: sábado útil
ge.calendarDontWorkInSaturday = true;
ge.calendarDontWorkInSunday = true;
ge.redraw();
```

## Baseline (salvar / mostrar)
```js
salvarBaseline("versao_A");
ge.showBaselines = true;
ge.redraw();
```

## Horário de trabalho / resolução
```js
ge.setHoursOn(8*60*60*1000, 17*60*60*1000, "dd/MM/yyyy", 1000);
```

## Eventos e extensão (pontos comuns)
- **After load**: `ge.loadProject(project)` → re-render automático.
- **Edição no grid** (em `ganttGridEditor.js`) → atualiza tarefa e agenda redraw parcial.
- **Zoom** (`ganttZoom.js`) → `ge.gantt.zoomGantt(isPlus)`.

> Observação: o projeto base não expõe um “EventBus” formal; as extensões são feitas por **sobrescrita de funções** ou **wrappers** em torno de métodos padrão.
