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
  lavoro: { label: 'Lavoro', color: '#3498db' },
  personale: { label: 'Personale', color: '#42b883' },
  urgente: { label: 'Urgente', color: '#e74c3c' },
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

// Dark mode: stessa logica delle task, leggo la preferenza salvata
// all'avvio e la riscrivo ogni volta che cambia
const isDarkMode = ref(localStorage.getItem('todo-pro-dark') === 'true')

function toggleDarkMode() {
  isDarkMode.value = !isDarkMode.value
  localStorage.setItem('todo-pro-dark', isDarkMode.value)
}
</script>

<template>
  <div class="app" :class="{ dark: isDarkMode }">
    <div class="header">
      <h1>📝 ToDo Pro</h1>
      <button class="theme-toggle" @click="toggleDarkMode">
        {{ isDarkMode ? '☀️' : '🌙' }}
      </button>
    </div>

    <form class="add-form" @submit.prevent="addTask">
      <input
        v-model="newTaskText"
        type="text"
        placeholder="Cosa devi fare?"
      />
      <!-- v-for funziona anche sugli oggetti, non solo sugli array:
           (cat, key) mi dà sia il valore che la chiave -->
      <select v-model="newTaskCategory">
        <option v-for="(cat, key) in categories" :key="key" :value="key">
          {{ cat.label }}
        </option>
      </select>
      <button type="submit">Aggiungi</button>
    </form>

    <div class="filters">
      <button
        :class="{ active: currentFilter === 'all' }"
        @click="currentFilter = 'all'"
      >
        Tutte
      </button>
      <button
        :class="{ active: currentFilter === 'active' }"
        @click="currentFilter = 'active'"
      >
        Attive
      </button>
      <button
        :class="{ active: currentFilter === 'completed' }"
        @click="currentFilter = 'completed'"
      >
        Completate
      </button>
    </div>

    <p class="counter">{{ remainingCount }} task rimaste</p>

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
      <p v-if="filteredTasks.length === 0" key="empty" class="empty">
        Nessuna task qui. 🎉
      </p>
    </TransitionGroup>
  </div>
</template>

<style scoped>
.app {
  max-width: 480px;
  margin: 0 auto;
  padding: 32px 20px;
  border-radius: 16px;
  transition: background 0.2s ease, color 0.2s ease;
}

/* Dark mode: la classe .dark viene aggiunta/tolta dinamicamente dal
   :class="{ dark: isDarkMode }" nel template */
.app.dark {
  background: #1e1e1e;
  color: #eee;
}

.app.dark .task-item {
  background: #2a2a2a;
}

.app.dark .add-form input {
  background: #2a2a2a;
  border-color: #444;
  color: #eee;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

h1 {
  margin: 0;
}

.theme-toggle {
  background: transparent;
  border: 1px solid #ccc;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  cursor: pointer;
  font-size: 16px;
}

.filters {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-bottom: 16px;
}

.filters button {
  padding: 6px 14px;
  border-radius: 20px;
  border: 1px solid #ccc;
  background: transparent;
  cursor: pointer;
  font-size: 13px;
}

.filters button.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.add-form {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}

.add-form input {
  flex: 1;
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 15px;
}

.add-form button {
  padding: 10px 18px;
  border-radius: 8px;
  border: none;
  background: #42b883;
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.add-form button:hover {
  background: #369870;
}

.counter {
  text-align: center;
  color: #888;
  margin-bottom: 16px;
}

.task-list {
  position: relative;
}

.empty {
  text-align: center;
  color: #aaa;
  padding: 24px 0;
}

/* Animazioni con TransitionGroup: Vue cerca da solo classi con questi
   nomi in base al valore di "name" passato sopra.
   - enter = quando un elemento entra (nuova task)
   - leave = quando un elemento esce (task eliminata)
   - move  = quando un elemento cambia posizione senza entrare/uscire */

.task-enter-active,
.task-leave-active {
  transition: all 0.3s ease;
}

.task-enter-from {
  opacity: 0;
  transform: translateX(-20px);
}

.task-leave-to {
  opacity: 0;
  transform: translateX(20px);
}

/* position: absolute toglie l'elemento dal flusso della pagina mentre
   esce, così le altre task scorrono al suo posto senza scatti */
.task-leave-active {
  position: absolute;
  width: 100%;
}

.task-move {
  transition: transform 0.3s ease;
}
</style>
