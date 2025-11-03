# 03 — Baseline (concepção, dados e uso)

## Conceito
**Baseline** é uma “foto” do plano em um instante no tempo. Serve para comparar o cronograma **planejado** vs **atual**.

## Estruturas no `GanttMaster`
- `baselines: { [taskId]: { taskId, start, end, status, progress } }`
- `baselineMillis: number` (timestamp da captura)
- `showBaselines: boolean` (exibição on/off)

Quando `showBaselines = true`, `ganttDrawerSVG` cria uma barra adicional por tarefa (via `_createBaselineSVG`) atrás da barra atual, com **tooltip** contendo:
- Nome da tarefa
- Data de captura (a partir de `baselineMillis`)
- `Start`, `End`, `Duration`
- `Status`, `Progress`

## Como salvar (capturar) uma baseline
Exemplo (JS) — captura a baseline da **situação atual**:

```js
// Supondo 'ge' como instância de GanttMaster
function salvarBaseline(label) {
  const map = {};
  const now = Date.now();

  ge.tasks.forEach(t => {
    if (t.id == null) return;
    map[t.id] = {
      taskId: t.id,
      start: t.start,   // millis
      end:   t.end,     // millis
      status: t.status || "PLANNED",
      progress: t.progress || 0
    };
  });

  ge.baselines = map;
  ge.baselineMillis = now;
  ge.showBaselines = true;   // liga a visualização
  ge.redraw();               // força redesenho (se necessário)
  console.log("Baseline salva:", label || new Date(now).toISOString());
}
```

> **Persistência:** salve `ge.baselines` + `ge.baselineMillis` no seu back‑end ou `localStorage` e recarregue junto com o projeto.

## Como carregar uma baseline persistida
```js
function carregarBaseline(baselinePersistida) {
  ge.baselines = baselinePersistida.map;
  ge.baselineMillis = baselinePersistida.millis;
  ge.showBaselines = true;
  ge.redraw();
}
```

## Como apresentar na tela
- Garanta que `ge.baselines[task.id]` exista para as tarefas desejadas.
- Defina `ge.showBaselines = true`.
- O `ganttDrawerSVG` chamará `_createBaselineSVG(task, baseline)` para cada tarefa com baseline.

### Estilo visual
- Barra de baseline com **opacidade** (~0.5) e **altura reduzida** (≈ `taskHeight / 1.5`).
- Cores podem ser personalizadas via CSS/SVG (classe específica da baseline).

## Boas práticas
- Nomeie a baseline (`baselineMillis` + rótulo).
- Capture baseline **após** aprovação do plano.
- Não edite a baseline diretamente; capture outra versão se necessário (versionamento).

