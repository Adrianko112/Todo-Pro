# ToDo Pro

Il mio primo progetto in Vue.js — una todo list con Vue puro (niente framework CSS o backend), per esercitarmi sui concetti fondamentali del framework.

## Come avviarlo
```bash
cd todo-pro
npm install
npm run dev
```

## Funzionalità
- Aggiungere, completare ed eliminare task
- Categorie colorate (Lavoro / Personale / Urgente)
- Filtri (tutte / attive / completate)
- Dark mode
- Dati persistenti tramite localStorage
- Animazioni di entrata/uscita sulle task

## Concetti Vue che ho usato
- `ref()` e `reactive` → stato reattivo
- `computed()` → valori derivati (filtri, contatore)
- `watch()` → side effect su cambiamento di stato (salvataggio localStorage)
- Props e `defineProps` → passaggio dati genitore → figlio
- Emit e `defineEmits` → comunicazione figlio → genitore ("props giù, eventi su")
- `v-model`, `v-for`, `v-if`, `v-bind`/`:class`/`:style` → direttive di base
- `<TransitionGroup>` → animazioni su liste dinamiche

## Struttura
- `src/App.vue` → contiene tutto lo stato principale (single source of truth)
- `src/components/TaskItem.vue` → componente per la singola task, riceve dati via props e comunica con emit


## Link Github Pages 
- Il progetto è online al seguente link: https://adrianko112.github.io/Todo-Pro/
- Quando apporto modifiche devo rilanciare nel terminale aperto nella cartella del progetto 
```bash
npm run deploy
```
- per le modifiche, attendere 1-2 minuti