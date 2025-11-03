# 04 — Calendário: dias úteis, finais de semana e feriados

## Flags e listas (em `GanttMaster`)
- `calendarHolidays: string[]` — datas no formato **YYYY-MM-DD** (não‑trabalhadas).
- `calendarWorkdayInHoliday: string[]` — exceções (tratar um dia como **trabalhado** mesmo sendo final de semana/feriado).
- `calendarDontWorkInSaturday: boolean`
- `calendarDontWorkInSunday: boolean`

### Exemplo de configuração (em `gantt.html` / `getDemoProject()`)
```js
{
  "calendarHolidays": ["2025-11-10"],
  "calendarWorkdayInHoliday": [],
  "calendarDontWorkInSaturday": true,
  "calendarDontWorkInSunday": true
}
```

## Função `isHoliday(date)` (em `libs/i18nJs.js`)
Regra aplicada pelo sistema para **marcação visual** (classe `holy`) e para cálculos:

1. Se a data estiver em `ge.calendarWorkdayInHoliday` ⇒ **não** é feriado (retorna `false`).
2. Se a data estiver em `ge.calendarHolidays` ⇒ **é feriado** (retorna `true`).
3. Se sábado (`getDay() == 6`) ⇒ retorna `ge.calendarDontWorkInSaturday` (padrão **true**).
4. Se domingo (`getDay() == 0`) ⇒ retorna `ge.calendarDontWorkInSunday` (padrão **true**).
5. Outros dias ⇒ `false` (dia útil).

## Marcação visual no zoom (em `ganttZoom.js`)
- Cada célula de dia avalia `isHoliday(start)`.
- Se `true`, aplica classe **`holy`** nas células de **cabeçalho** e **corpo** (dando destaque de final de semana/feriado).

## Horário de trabalho (em `ganttMaster.js`)
Use `setHoursOn(startWorkingHour, endWorkingHour, dateFormat, resolution)` para definir período de trabalho diário e resolução dos cálculos.

### Exemplo
```js
// 08:00 às 17:00, formato dd/MM/yyyy, resolução por milissegundos (>=1000)
ge.setHoursOn(8*60*60*1000, 17*60*60*1000, "dd/MM/yyyy", 1000);
```

## Teste/validação
- `test_holidays.html` demonstra a checagem de feriados/finais de semana apresentando o resultado por data.

## Recomendações
- Centralize feriados em serviço/config do back‑end.
- Use `calendarWorkdayInHoliday` para exceções (ex.: **sábado útil**).
- Evite datas em outro timezone — use sempre `YYYY-MM-DD` (UTC) para listas.
