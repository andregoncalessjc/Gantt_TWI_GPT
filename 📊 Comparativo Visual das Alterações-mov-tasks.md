# 📊 Comparativo Visual das Alterações

## 🎯 Visão Geral

Este documento mostra **lado a lado** as alterações realizadas nos arquivos do Gantt.

---

## 📄 ALTERAÇÃO 1: `ganttDrawerSVG.js` (Linha ~276)

### 🔴 CÓDIGO ORIGINAL

```javascript
      }).mousedown(function () {
        var task = self.master.getTask($(this).attr("taskid"));
        task.rowElement.click();
      }).dragExtedSVG($(self.svg.root()), {
        canResize:  this.master.permissions.canWrite || task.canWrite,
        canDrag:    !task.depends && (this.master.permissions.canWrite || task.canWrite),
        resizeZoneWidth:self.resizeZoneWidth,
        startDrag:  function (e) {
          $(".ganttSVGBox .focused").removeClass("focused");
        },
```

### 🟢 CÓDIGO MODIFICADO

```javascript
      }).mousedown(function () {
        var task = self.master.getTask($(this).attr("taskid"));
        task.rowElement.click();
      }).dragExtedSVG($(self.svg.root()), {
        canResize:  this.master.permissions.canWrite || task.canWrite,
        
        // ========================================================================
        // MODIFICAÇÃO: Permite arrastar tarefas NÃO iniciadas mesmo com dependências
        // ========================================================================
        // Lógica:
        // - Se task.progress == 0 (não iniciada): PODE arrastar (ignora dependências)
        // - Se !task.depends (sem dependências): PODE arrastar
        // - Caso contrário: NÃO pode arrastar
        // ========================================================================
        canDrag:    (task.progress == 0 || !task.depends) && (this.master.permissions.canWrite || task.canWrite),
        
        resizeZoneWidth:self.resizeZoneWidth,
        startDrag:  function (e) {
          $(".ganttSVGBox .focused").removeClass("focused");
        },
```

### 📝 O QUE MUDOU?

| Aspecto | Antes | Depois |
|---------|-------|--------|
| **Condição** | `!task.depends` | `(task.progress == 0 \|\| !task.depends)` |
| **Tarefas sem dependência** | ✅ Pode arrastar | ✅ Pode arrastar |
| **Tarefas COM dependência e NÃO iniciadas** | ❌ NÃO pode arrastar | ✅ **PODE arrastar** |
| **Tarefas COM dependência e INICIADAS** | ❌ NÃO pode arrastar | ❌ NÃO pode arrastar |

### 🎯 RESULTADO

- **Tarefas não iniciadas** (progress = 0) agora podem ser arrastadas livremente, **mesmo tendo predecessoras**
- **Tarefas iniciadas** (progress > 0) continuam bloqueadas se tiverem dependências

---

## 📄 ALTERAÇÃO 2: `ganttTaskManus.js` - Função `setPeriod` (Linhas ~188-195)

### 🔴 CÓDIGO ORIGINAL

```javascript
  //cannot start after end
  if (start > end) {
    start = end;
  }

  //if there are dependencies compute the start date and eventually moveTo
  var startBySuperiors = this.computeStartBySuperiors(start);
  if (startBySuperiors != start) {
    return this.moveTo(startBySuperiors, false,true);
  }

  var somethingChanged = false;

  if (this.start != start || this.start != wantedStartMillis) {
    this.start = start;
    somethingChanged = true;
  }
```

### 🟢 CÓDIGO MODIFICADO

```javascript
  //cannot start after end
  if (start > end) {
    start = end;
  }

  // ========================================================================
  // MODIFICAÇÃO: Só aplica restrição de dependência se a tarefa JÁ iniciou
  // ========================================================================
  // Regra de Negócio:
  // - Tarefas NÃO iniciadas (progress == 0): Podem ter datas alteradas 
  //   livremente, ignorando dependências de predecessoras
  // - Tarefas INICIADAS (progress > 0): Devem respeitar dependências e
  //   manter data de início fixa (data real de início)
  // ========================================================================
  //if there are dependencies compute the start date and eventually moveTo
  if (this.progress > 0) {
    // Tarefa JÁ INICIADA: Aplica restrições de dependência
    var startBySuperiors = this.computeStartBySuperiors(start);
    if (startBySuperiors != start) {
      return this.moveTo(startBySuperiors, false,true);
    }
  }
  // Se progress == 0 (NÃO iniciada): Pula a verificação de dependências

  var somethingChanged = false;

  if (this.start != start || this.start != wantedStartMillis) {
    this.start = start;
    somethingChanged = true;
  }
```

### 📝 O QUE MUDOU?

| Aspecto | Antes | Depois |
|---------|-------|--------|
| **Verificação de dependências** | Sempre executada | Só se `progress > 0` |
| **Tarefas NÃO iniciadas** | ❌ Bloqueadas por predecessoras | ✅ **Livres para mover** |
| **Tarefas INICIADAS** | ✅ Bloqueadas por predecessoras | ✅ Bloqueadas por predecessoras |
| **Alteração de data pela tabela** | ❌ Limitada | ✅ **Livre se não iniciada** |

### 🎯 RESULTADO

- **Tarefas não iniciadas** podem ter suas datas alteradas pela tabela sem restrições de dependências
- **Tarefas iniciadas** continuam respeitando as dependências (comportamento original)

---

## 📄 VERIFICAÇÃO: `ganttTaskManus.js` - Função `moveTo` (Linhas ~318-328)

### ℹ️ CÓDIGO JÁ EXISTENTE (Não precisa alterar)

```javascript
  //if depends, start is set to max end + lag of superior
  // REGRA CUSTOM (FASE 2): Se a tarefa NÃO iniciou (progress == 0), ignora dependências.
  if (this.progress == 0) {
    // Permite movimento livre, ignorando a restrição de dependência
    // O 'start' permanece o 'start' desejado (wantedStartMillis)
  } else {
    // Se a tarefa JÁ iniciou, mantém a checagem de dependência (comportamento padrão)
    start = this.computeStartBySuperiors(start);
  }

  var end = computeEndByDuration(start, this.duration);
```

### ⚠️ AÇÃO NECESSÁRIA

**APENAS VERIFIQUE** se este código já está presente no seu arquivo. Se estiver, **não precisa fazer nada**.

Se **NÃO** estiver, procure por este trecho:

```javascript
start = this.computeStartBySuperiors(start);
var end = computeEndByDuration(start, this.duration);
```

E substitua por:

```javascript
// REGRA CUSTOM: Se a tarefa NÃO iniciou (progress == 0), ignora dependências
if (this.progress == 0) {
  // Permite movimento livre, ignorando a restrição de dependência
} else {
  // Se a tarefa JÁ iniciou, mantém a checagem de dependência
  start = this.computeStartBySuperiors(start);
}

var end = computeEndByDuration(start, this.duration);
```

---

## 🔍 Como Encontrar os Trechos para Alterar

### Método 1: Busca por Texto (Recomendado)

#### Para `ganttDrawerSVG.js`:
1. Abra o arquivo no editor
2. Pressione `Ctrl+F` (ou `Cmd+F` no Mac)
3. Busque por: `canDrag:`
4. Você encontrará: `canDrag:    !task.depends &&`
5. Modifique para: `canDrag:    (task.progress == 0 || !task.depends) &&`

#### Para `ganttTaskManus.js` (Alteração 1):
1. Abra o arquivo no editor
2. Pressione `Ctrl+F`
3. Busque por: `//if there are dependencies compute the start date`
4. Logo abaixo você verá: `var startBySuperiors = this.computeStartBySuperiors(start);`
5. Envolva este bloco com: `if (this.progress > 0) { ... }`

#### Para `ganttTaskManus.js` (Verificação):
1. Pressione `Ctrl+F`
2. Busque por: `REGRA CUSTOM (FASE 2)`
3. Se encontrar → **Não precisa fazer nada**
4. Se NÃO encontrar → Busque por `start = this.computeStartBySuperiors(start);` e aplique a modificação

---

## 📊 Tabela Resumo das Alterações

| Arquivo | Linha | Buscar por | Substituir/Adicionar |
|---------|-------|------------|----------------------|
| `ganttDrawerSVG.js` | ~276 | `canDrag:    !task.depends &&` | `canDrag:    (task.progress == 0 \|\| !task.depends) &&` |
| `ganttTaskManus.js` | ~188 | `var startBySuperiors = this.computeStartBySuperiors(start);` | Envolver com `if (this.progress > 0) { ... }` |
| `ganttTaskManus.js` | ~318 | `start = this.computeStartBySuperiors(start);` | Verificar se já tem condicional `if (this.progress == 0)` |

---

## ✅ Checklist de Implementação

```
[ ] 1. Fazer backup dos arquivos originais
[ ] 2. Abrir ganttDrawerSVG.js
[ ] 3. Localizar linha com "canDrag:"
[ ] 4. Modificar condição conforme indicado
[ ] 5. Adicionar comentários explicativos
[ ] 6. Salvar ganttDrawerSVG.js
[ ] 7. Abrir ganttTaskManus.js
[ ] 8. Localizar função setPeriod
[ ] 9. Adicionar condicional if (this.progress > 0)
[ ] 10. Adicionar comentários explicativos
[ ] 11. Verificar função moveTo
[ ] 12. Salvar ganttTaskManus.js
[ ] 13. Abrir gantt-versao-01.html no navegador
[ ] 14. Testar com tarefa não iniciada com dependência
[ ] 15. Testar com tarefa iniciada
[ ] 16. Verificar propagação para sucessoras
```

---

## 🎓 Entendendo a Lógica

### Por que `task.progress == 0`?

- `progress = 0` significa que a tarefa **ainda não começou**
- Se não começou, a data de início é **planejada**, não **real**
- Datas planejadas podem ser ajustadas livremente
- Quando `progress > 0`, a tarefa **já iniciou**, então a data de início é **real** e deve ser fixa

### Por que usar `||` (OU)?

```javascript
(task.progress == 0 || !task.depends)
```

Esta condição significa:
- **PODE arrastar** SE a tarefa não iniciou (progress == 0)
- **OU** SE a tarefa não tem dependências (!task.depends)
- **Resultado**: Máxima flexibilidade para planejamento

### Fluxo de Decisão

```
Usuário tenta arrastar uma tarefa
    ↓
Verifica: task.progress == 0?
    ↓
SIM (não iniciada)
    → PERMITE arrastar (ignora dependências)
    ↓
NÃO (já iniciada)
    → Verifica: !task.depends?
        ↓
        SIM → PERMITE arrastar
        ↓
        NÃO → BLOQUEIA arrasto
```

---

**Documento criado em**: 04/11/2025  
**Versão**: 1.0  
**Código bem comentado e organizado** ✅
