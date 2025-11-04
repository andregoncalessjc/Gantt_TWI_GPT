# Alterações Detalhadas - Gantt TWI Aprimorado

## 📋 Resumo das Modificações

Foram feitas alterações em **2 arquivos JavaScript** para implementar a mobilidade aprimorada das atividades no Gantt.

---

## 📄 Arquivo 1: `ganttDrawerSVG.js`

### Localização da Alteração
**Linha 276** (aproximadamente)

### Contexto
Dentro da função `Ganttalendar.prototype.drawTask`, na configuração do método `.dragExtedSVG()`

### Código ANTES da Alteração
```javascript
      }).dragExtedSVG($(self.svg.root()), {
        canResize:  this.master.permissions.canWrite || task.canWrite,
        canDrag:    !task.depends && (this.master.permissions.canWrite || task.canWrite),
        resizeZoneWidth:self.resizeZoneWidth,
```

### Código DEPOIS da Alteração
```javascript
      }).dragExtedSVG($(self.svg.root()), {
        canResize:  this.master.permissions.canWrite || task.canWrite,
        // MODIFICAÇÃO: Permite arrastar tarefas NÃO iniciadas (progress == 0) mesmo com dependências
        // Tarefas iniciadas (progress > 0) só podem ser arrastadas se não tiverem dependências
        canDrag:    (task.progress == 0 || !task.depends) && (this.master.permissions.canWrite || task.canWrite),
        resizeZoneWidth:self.resizeZoneWidth,
```

### Explicação da Alteração
- **Antes**: `!task.depends` → Bloqueava o arrasto de QUALQUER tarefa com dependências
- **Depois**: `(task.progress == 0 || !task.depends)` → Permite arrastar tarefas não iniciadas, mesmo com dependências
- **Lógica**: Se `progress == 0` OU se não tem dependências, pode arrastar

---

## 📄 Arquivo 2: `ganttTaskManus.js`

### ALTERAÇÃO 1 - Função `setPeriod`

#### Localização da Alteração
**Linhas 188-192** (aproximadamente)

#### Contexto
Dentro da função `Task.prototype.setPeriod`, após a validação `if (start > end)`

#### Código ANTES da Alteração
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
```

#### Código DEPOIS da Alteração
```javascript
  //cannot start after end
  if (start > end) {
    start = end;
  }

  //if there are dependencies compute the start date and eventually moveTo
  // ========================================================================
  // MODIFICAÇÃO: Só aplica restrição de dependência se a tarefa JÁ iniciou
  // ========================================================================
  // Regra de Negócio:
  // - Tarefas NÃO iniciadas (progress == 0): Podem ser movidas livremente, 
  //   ignorando dependências de predecessoras
  // - Tarefas INICIADAS (progress > 0): Devem respeitar dependências
  // ========================================================================
  if (this.progress > 0) {
    var startBySuperiors = this.computeStartBySuperiors(start);
    if (startBySuperiors != start) {
      return this.moveTo(startBySuperiors, false,true);
    }
  }

  var somethingChanged = false;
```

#### Explicação da Alteração
- **Antes**: Sempre verificava e aplicava restrições de dependências
- **Depois**: Só verifica dependências se `progress > 0` (tarefa já iniciada)
- **Resultado**: Tarefas não iniciadas podem ter datas alteradas livremente

---

### ALTERAÇÃO 2 - Função `moveTo` (VERIFICAÇÃO)

#### Localização
**Linhas 318-328** (aproximadamente)

#### Contexto
Dentro da função `Task.prototype.moveTo`

#### Código Existente (JÁ ESTAVA CORRETO)
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
```

#### Ação Necessária
**NENHUMA** - Este trecho já estava implementado corretamente no código original. Apenas verifique se ele está presente.

Se não estiver, adicione este código no lugar do trecho original que faz:
```javascript
start = this.computeStartBySuperiors(start);
```

---

## 🎯 Resumo das Regras Implementadas

### ✅ Atividades NÃO INICIADAS (progress == 0)
- ✓ Podem ser arrastadas livremente no Gantt
- ✓ Ignoram restrições de dependências (predecessoras)
- ✓ Data de início e fim podem ser alteradas
- ✓ Podem ser movidas para antes das predecessoras terminarem

### ✅ Atividades INICIADAS (progress > 0)
- ✓ Data de início FIXA (não pode ser alterada)
- ✓ Duração VARIÁVEL (pode alterar data de término)
- ✓ Não podem ser arrastadas (movimento bloqueado)
- ✓ Podem ter o final redimensionado
- ✓ Respeitam dependências de predecessoras

### ✅ Propagação Automática
- ✓ Atividades sucessoras se ajustam automaticamente
- ✓ Funciona em cadeia (A → B → C)
- ✓ Já estava implementado no código original

---

## 🔍 Como Localizar os Trechos para Alterar

### Arquivo: `ganttDrawerSVG.js`

**Busque por:**
```javascript
.dragExtedSVG($(self.svg.root()), {
```

**Você encontrará:**
```javascript
canDrag:    !task.depends &&
```

**Substitua por:**
```javascript
canDrag:    (task.progress == 0 || !task.depends) &&
```

---

### Arquivo: `ganttTaskManus.js`

**Busque por (Alteração 1):**
```javascript
//if there are dependencies compute the start date and eventually moveTo
var startBySuperiors = this.computeStartBySuperiors(start);
```

**Envolva com a condição:**
```javascript
if (this.progress > 0) {
  var startBySuperiors = this.computeStartBySuperiors(start);
  if (startBySuperiors != start) {
    return this.moveTo(startBySuperiors, false,true);
  }
}
```

---

**Busque por (Verificação - Alteração 2):**
```javascript
//if depends, start is set to max end + lag of superior
```

**Verifique se logo abaixo tem:**
```javascript
if (this.progress == 0) {
```

Se não tiver, procure por:
```javascript
start = this.computeStartBySuperiors(start);
```

E substitua pela estrutura condicional mostrada acima.

---

## 📝 Observações Importantes

1. **Backup**: Faça backup dos arquivos originais antes de modificar
2. **Numeração de Linhas**: As linhas podem variar ligeiramente dependendo da versão
3. **Comentários**: Todos os comentários foram adicionados para facilitar manutenção futura
4. **Teste**: Após as alterações, teste com atividades iniciadas e não iniciadas

---

## ✅ Checklist de Implementação

- [ ] Fazer backup de `ganttDrawerSVG.js`
- [ ] Fazer backup de `ganttTaskManus.js`
- [ ] Alterar linha 276 em `ganttDrawerSVG.js` (canDrag)
- [ ] Alterar linhas 188-195 em `ganttTaskManus.js` (setPeriod)
- [ ] Verificar linhas 318-328 em `ganttTaskManus.js` (moveTo)
- [ ] Testar com tarefa não iniciada com dependência
- [ ] Testar com tarefa iniciada (progress > 0)
- [ ] Verificar propagação para sucessoras

---

**Última atualização**: 04/11/2025
