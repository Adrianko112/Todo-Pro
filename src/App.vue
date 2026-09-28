<script setup>
import { ref, computed, watch } from 'vue'
import TaskItem from './components/TaskItem.vue'

// Al primo caricamento controllo se ci sono task già salvate nel browser.
// localStorage salva solo stringhe, quindi devo usare JSON.parse per
// riconvertire la stringa salvata in un array di oggetti utilizzabile.
const saved = localStorage.getItem('todo-pro-tasks')

// Stato principale dell'app: uso ref() per renderlo reattivo, così ogni
// volta che modifico l'array Vue aggiorna da solo l'interfaccia.
const tasks = ref(
  saved
    ? JSON.parse(saved)
    : [
        { id: 1, text: 'Imparare Vue.js', completed: false },
        { id: 2, text: 'Costruire ToDo Pro', completed: false },
      ]
)

// Testo del campo di input, collegato con v-model
const newTaskText = ref('')

// Categoria scelta per la nuova task che sto per aggiungere
const newTaskCategory = ref('personale')

// Metto tutte le categorie disponibili in un unico oggetto: se in futuro
// voglio aggiungerne una nuova, la aggiungo solo qui e basta
const categories = {
  lavoro: { label: 'Lavoro', color: '#3b82f6' },
  personale: { label: 'Personale', color: '#10b981' },
  urgente: { label: 'Urgente', color: '#f43f5e' },
}

function addTask() {
  const text = newTaskText.value.trim()
  if (!text) return // non aggiungo task vuote

  tasks.value.push({
    id: Date.now(), // uso il timestamp come id univoco, semplice ma funziona
    text,
    completed: false,
    category: newTaskCategory.value,
  })

  newTaskText.value = '' // svuoto il campo dopo l'aggiunta
}

function toggleComplete(id) {
  const task = tasks.value.find(t => t.id === id)
  if (task) task.completed = !task.completed
}

function deleteTask(id) {
  tasks.value = tasks.value.filter(t => t.id !== id)
}

// Ogni volta che 'tasks' cambia, salvo automaticamente in localStorage.
// { deep: true } è necessario perché tasks è un array di oggetti: senza
// questa opzione Vue non si accorgerebbe se cambio solo task.completed,
// ma solo se sostituisco l'intero array.
watch(
  tasks,
  (newTasks) => {
    localStorage.setItem('todo-pro-tasks', JSON.stringify(newTasks))
  },
  { deep: true }
)

// Filtro attivo: può essere 'all', 'active' oppure 'completed'
const currentFilter = ref('all')

// Elenco dei filtri, così nel template posso generare i bottoni con v-for
// invece di scriverli tre volte a mano
const filters = [
  { key: 'all', label: 'Tutte' },
  { key: 'active', label: 'Attive' },
  { key: 'completed', label: 'Completate' },
]

// computed = valore derivato che si ricalcola da solo quando cambiano
// 'tasks' o 'currentFilter', e resta in cache nel frattempo (più
// efficiente di richiamare una funzione ad ogni render)
const filteredTasks = computed(() => {
  if (currentFilter.value === 'active') {
    return tasks.value.filter(t => !t.completed)
  }
  if (currentFilter.value === 'completed') {
    return tasks.value.filter(t => t.completed)
  }
  return tasks.value
})

const remainingCount = computed(
  () => tasks.value.filter(t => !t.completed).length
)

const completedCount = computed(() => tasks.value.length - remainingCount.value)

// Numero di task per ogni filtro, da mostrare come badge sui bottoni
const filterCounts = computed(() => ({
  all: tasks.value.length,
  active: remainingCount.value,
  completed: completedCount.value,
}))

// Percentuale di completamento per la barra di avanzamento
const progress = computed(() =>
  tasks.value.length === 0
    ? 0
    : Math.round((completedCount.value / tasks.value.length) * 100)
)

// Data di oggi in italiano, es. "domenica 28 settembre"
const today = new Date().toLocaleDateString('it-IT', {
  weekday: 'long',
  day: 'numeric',
  month: 'long',
})

// Dark mode: stessa logica delle task, leggo la preferenza salvata
// all'avvio e la riscrivo ogni volta che cambia
const isDarkMode = ref(localStorage.getItem('todo-pro-dark') === 'true')

function toggleDarkMode() {
  isDarkMode.value = !isDarkMode.value
  localStorage.setItem('todo-pro-dark', isDarkMode.value)
}
</script>

<template>
  <!-- La classe .dark sta sul contenitore più esterno, così il tema
       cambia su tutta la pagina e non solo sulla card -->
  <div class="page" :class="{ dark: isDarkMode }">
    <div class="blob blob-1"></div>
    <div class="blob blob-2"></div>

    <main class="app">
      <header class="header">
        <div>
          <p class="date">{{ today }}</p>
          <h1>ToDo <span class="accent">Pro</span></h1>
        </div>
        <button
          class="theme-toggle"
          :aria-label="isDarkMode ? 'Attiva tema chiaro' : 'Attiva tema scuro'"
          @click="toggleDarkMode"
        >
          {{ isDarkMode ? '☀️' : '🌙' }}
        </button>
      </header>

      <section class="progress-card">
        <div class="progress-info">
          <span>
            <strong>{{ completedCount }}</strong> di {{ tasks.length }} completate
          </span>
          <span class="progress-percent">{{ progress }}%</span>
        </div>
        <div class="progress-track">
          <!-- :style con una variabile reattiva: la larghezza della barra
               segue in automatico la percentuale -->
          <div class="progress-fill" :style="{ width: progress + '%' }"></div>
        </div>
      </section>

      <form class="add-form" @submit.prevent="addTask">
        <div class="input-row">
          <input
            v-model="newTaskText"
            type="text"
            placeholder="Cosa devi fare oggi?"
          />
          <button type="submit" class="add-btn" :disabled="!newTaskText.trim()">
            <span class="plus">+</span>
            <span class="add-label">Aggiungi</span>
          </button>
        </div>

        <!-- v-for funziona anche sugli oggetti, non solo sugli array:
             (cat, key) mi dà sia il valore che la chiave.
             Uso dei "chip" cliccabili al posto della <select> -->
        <div class="category-picker">
          <button
            v-for="(cat, key) in categories"
            :key="key"
            type="button"
            class="chip"
            :class="{ selected: newTaskCategory === key }"
            :style="{ '--chip-color': cat.color }"
            @click="newTaskCategory = key"
          >
            <span class="chip-dot"></span>
            {{ cat.label }}
          </button>
        </div>
      </form>

      <nav class="filters">
        <button
          v-for="f in filters"
          :key="f.key"
          :class="{ active: currentFilter === f.key }"
          @click="currentFilter = f.key"
        >
          {{ f.label }}
          <span class="badge">{{ filterCounts[f.key] }}</span>
        </button>
      </nav>

      <!-- Uso TransitionGroup invece di un semplice div per animare
           l'entrata/uscita delle task. "name=task" fa sì che Vue cerchi
           automaticamente le classi CSS task-enter-*, task-leave-*, task-move -->
      <TransitionGroup name="task" tag="div" class="task-list">
        <TaskItem
          v-for="task in filteredTasks"
          :key="task.id"
          :task="task"
          :categories="categories"
          @toggle-complete="toggleComplete"
          @delete-task="deleteTask"
        />

        <!-- serve una key anche qui perché TransitionGroup la richiede
             su ogni figlio diretto -->
        <div v-if="filteredTasks.length === 0" key="empty" class="empty">
          <div class="empty-icon">🎉</div>
          <p>Nessuna task qui.</p>
          <small>Goditi il momento, oppure aggiungine una nuova.</small>
        </div>
      </TransitionGroup>

      <footer class="footer" v-if="tasks.length > 0">
        {{ remainingCount }}
        {{ remainingCount === 1 ? 'task rimasta' : 'task rimaste' }}
      </footer>
    </main>
  </div>
</template>

<style scoped>
/* Tutti i colori sono variabili CSS: il tema scuro si limita a
   ridefinirle, senza dover riscrivere ogni regola. Le variabili si
   ereditano anche dentro TaskItem, quindi il componente figlio le può usare. */
.page {
  --bg: #f4f2ff;
  --card: rgba(255, 255, 255, 0.78);
  --surface: #ffffff;
  --text: #1e1b3a;
  --muted: #7a7894;
  --border: rgba(30, 27, 58, 0.08);
  --primary: #6d5dfc;
  --primary-2: #b05dfc;
  --shadow: 0 20px 50px -20px rgba(76, 60, 200, 0.35);
  --task-shadow: 0 2px 8px rgba(30, 27, 58, 0.05);

  position: relative;
  min-height: 100vh;
  padding: 48px 16px;
  background: var(--bg);
  color: var(--text);
  overflow: hidden;
  transition: background 0.4s ease, color 0.4s ease;
}

.page.dark {
  --bg: #0f0e1a;
  --card: rgba(26, 24, 44, 0.72);
  --surface: #1f1d35;
  --text: #ecebff;
  --muted: #8d8aad;
  --border: rgba(255, 255, 255, 0.07);
  --shadow: 0 20px 60px -20px rgba(0, 0, 0, 0.7);
  --task-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  color-scheme: dark;
}

/* Macchie di colore sfocate sullo sfondo, puramente decorative */
.blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.55;
  pointer-events: none;
}

.blob-1 {
  width: 420px;
  height: 420px;
  background: #a594ff;
  top: -120px;
  left: -100px;
}

.blob-2 {
  width: 380px;
  height: 380px;
  background: #ff9ad5;
  bottom: -120px;
  right: -80px;
}

.dark .blob {
  opacity: 0.25;
}

.app {
  position: relative;
  max-width: 540px;
  margin: 0 auto;
  padding: 32px 28px;
  background: var(--card);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid var(--border);
  border-radius: 28px;
  box-shadow: var(--shadow);
}

/* ---------- Header ---------- */
.header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 24px;
}

.date {
  margin: 0 0 4px;
  font-size: 13px;
  font-weight: 600;
  text-transform: capitalize;
  color: var(--muted);
}

h1 {
  margin: 0;
  font-size: 34px;
  font-weight: 800;
  letter-spacing: -1px;
}

.accent {
  background: linear-gradient(135deg, var(--primary), var(--primary-2));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

.theme-toggle {
  width: 44px;
  height: 44px;
  border-radius: 14px;
  border: 1px solid var(--border);
  background: var(--surface);
  font-size: 18px;
  cursor: pointer;
  box-shadow: var(--task-shadow);
  transition: transform 0.3s ease;
}

.theme-toggle:hover {
  transform: rotate(20deg) scale(1.08);
}

/* ---------- Barra di avanzamento ---------- */
.progress-card {
  padding: 16px 18px;
  margin-bottom: 20px;
  border-radius: 18px;
  background: linear-gradient(135deg, var(--primary), var(--primary-2));
  color: white;
}

.progress-info {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 10px;
  font-size: 14px;
}

.progress-info strong {
  font-size: 18px;
}

.progress-percent {
  font-size: 22px;
  font-weight: 800;
}

.progress-track {
  height: 8px;
  border-radius: 99px;
  background: rgba(255, 255, 255, 0.25);
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 99px;
  background: white;
  transition: width 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

/* ---------- Form ---------- */
.add-form {
  margin-bottom: 20px;
}

.input-row {
  display: flex;
  gap: 8px;
  padding: 6px;
  border-radius: 18px;
  background: var(--surface);
  border: 1px solid var(--border);
  box-shadow: var(--task-shadow);
  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}

.input-row:focus-within {
  border-color: var(--primary);
  box-shadow: 0 0 0 4px rgba(109, 93, 252, 0.15);
}

.input-row input {
  flex: 1;
  min-width: 0;
  padding: 10px 12px;
  border: none;
  outline: none;
  background: transparent;
  font-size: 15px;
}

.input-row input::placeholder {
  color: var(--muted);
}

.add-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 18px;
  border: none;
  border-radius: 13px;
  background: linear-gradient(135deg, var(--primary), var(--primary-2));
  color: white;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.15s ease, opacity 0.2s ease, box-shadow 0.2s ease;
}

.add-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px -6px rgba(109, 93, 252, 0.6);
}

.add-btn:active:not(:disabled) {
  transform: scale(0.97);
}

.add-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.plus {
  font-size: 20px;
  line-height: 1;
}

.category-picker {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 12px;
}

/* --chip-color arriva dal template con :style, così ogni chip usa
   il colore della sua categoria con un'unica regola CSS */
.chip {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border-radius: 99px;
  border: 1px solid var(--border);
  background: var(--surface);
  font-size: 13px;
  font-weight: 600;
  color: var(--muted);
  cursor: pointer;
  transition: all 0.2s ease;
}

.chip-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--chip-color);
}

.chip:hover {
  border-color: var(--chip-color);
}

.chip.selected {
  color: var(--chip-color);
  border-color: var(--chip-color);
  background: color-mix(in srgb, var(--chip-color) 12%, transparent);
}

/* ---------- Filtri ---------- */
.filters {
  display: flex;
  gap: 4px;
  padding: 4px;
  margin-bottom: 16px;
  border-radius: 14px;
  background: var(--border);
}

.filters button {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 8px 10px;
  border: none;
  border-radius: 11px;
  background: transparent;
  font-size: 13px;
  font-weight: 600;
  color: var(--muted);
  cursor: pointer;
  transition: all 0.2s ease;
}

.filters button.active {
  background: var(--surface);
  color: var(--text);
  box-shadow: var(--task-shadow);
}

.badge {
  min-width: 20px;
  padding: 1px 6px;
  border-radius: 99px;
  font-size: 11px;
  background: var(--border);
}

.filters button.active .badge {
  background: var(--primary);
  color: white;
}

/* ---------- Lista ---------- */
.task-list {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.empty {
  text-align: center;
  padding: 36px 0 24px;
  color: var(--muted);
}

.empty-icon {
  font-size: 40px;
  margin-bottom: 8px;
}

.empty p {
  margin: 0 0 4px;
  font-weight: 700;
  color: var(--text);
}

.footer {
  margin-top: 20px;
  text-align: center;
  font-size: 13px;
  color: var(--muted);
}

@media (max-width: 480px) {
  .page {
    padding: 20px 12px;
  }

  .app {
    padding: 24px 18px;
    border-radius: 22px;
  }

  .add-label {
    display: none;
  }
}

/* Animazioni con TransitionGroup: Vue cerca da solo classi con questi
   nomi in base al valore di "name" passato sopra.
   - enter = quando un elemento entra (nuova task)
   - leave = quando un elemento esce (task eliminata)
   - move  = quando un elemento cambia posizione senza entrare/uscire */

.task-enter-active,
.task-leave-active {
  transition: all 0.35s cubic-bezier(0.22, 1, 0.36, 1);
}

.task-enter-from {
  opacity: 0;
  transform: translateY(-10px) scale(0.97);
}

.task-leave-to {
  opacity: 0;
  transform: translateX(40px);
}

/* position: absolute toglie l'elemento dal flusso della pagina mentre
   esce, così le altre task scorrono al suo posto senza scatti */
.task-leave-active {
  position: absolute;
  width: 100%;
}

.task-move {
  transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1);
}
</style>
