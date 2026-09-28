<script setup>
// Componente per una singola task. Riceve i dati dal genitore (App.vue)
// tramite props e non li modifica mai direttamente: se deve succedere
// qualcosa, lo comunico al genitore con emit.
defineProps({
  task: {
    type: Object,
    required: true
  },
  categories: {
    type: Object,
    required: true
  }
})

// Eventi che questo componente manda su verso il genitore.
// È il genitore a decidere cosa fare quando li riceve.
const emit = defineEmits(['toggle-complete', 'delete-task'])
</script>

<template>
  <!-- Passo il colore della categoria come variabile CSS (--cat-color):
       così barra laterale, etichetta e checkbox lo usano tutti.
       Uso ?. (optional chaining) perché se una task non ha una categoria
       valida (es. task vecchie) non voglio che l'app vada in errore -->
  <div
    class="task-item"
    :class="{ completed: task.completed }"
    :style="{ '--cat-color': categories[task.category]?.color || '#9ca3af' }"
  >
    <!-- Checkbox personalizzata: la <input> vera resta (per accessibilità
         e tastiera) ma è nascosta, e disegno io il cerchio al suo posto -->
    <label class="check">
      <input
        type="checkbox"
        :checked="task.completed"
        @change="emit('toggle-complete', task.id)"
      />
      <span class="check-circle">
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M5 12.5l4.5 4.5L19 7.5" />
        </svg>
      </span>
    </label>

    <div class="task-body">
      <span class="task-text">{{ task.text }}</span>
      <span v-if="categories[task.category]" class="task-tag">
        {{ categories[task.category].label }}
      </span>
    </div>

    <button
      class="delete-btn"
      aria-label="Elimina task"
      @click="emit('delete-task', task.id)"
    >
      <svg viewBox="0 0 24 24" aria-hidden="true">
        <path d="M4 7h16M10 11v6M14 11v6M6 7l1 12a2 2 0 002 2h6a2 2 0 002-2l1-12M9 7V4h6v3" />
      </svg>
    </button>
  </div>
</template>

<style scoped>
.task-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px 14px 14px 18px;
  background: var(--surface, #fff);
  border: 1px solid var(--border, #eee);
  border-radius: 16px;
  box-shadow: var(--task-shadow);
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease, opacity 0.3s ease;
}

/* Barretta colorata a sinistra con il colore della categoria */
.task-item::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 4px;
  background: var(--cat-color);
}

.task-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 24px -12px rgba(30, 27, 58, 0.3);
}

.task-item.completed {
  opacity: 0.6;
}

/* ---------- Checkbox ---------- */
.check {
  position: relative;
  flex-shrink: 0;
  cursor: pointer;
}

.check input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.check-circle {
  display: grid;
  place-items: center;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: 2px solid var(--cat-color);
  transition: background 0.2s ease, transform 0.2s ease;
}

.check-circle svg {
  width: 14px;
  height: 14px;
  fill: none;
  stroke: white;
  stroke-width: 3;
  stroke-linecap: round;
  stroke-linejoin: round;
  /* Il segno di spunta si "disegna" animando stroke-dashoffset */
  stroke-dasharray: 24;
  stroke-dashoffset: 24;
  transition: stroke-dashoffset 0.3s ease 0.05s;
}

.check:hover .check-circle {
  transform: scale(1.1);
}

.check input:checked + .check-circle {
  background: var(--cat-color);
}

.check input:checked + .check-circle svg {
  stroke-dashoffset: 0;
}

.check input:focus-visible + .check-circle {
  outline: 3px solid color-mix(in srgb, var(--cat-color) 40%, transparent);
  outline-offset: 2px;
}

/* ---------- Testo ---------- */
.task-body {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 4px;
  text-align: left;
}

.task-text {
  font-size: 15px;
  font-weight: 500;
  overflow-wrap: anywhere;
  transition: color 0.3s ease;
}

.task-item.completed .task-text {
  text-decoration: line-through;
  color: var(--muted);
}

.task-tag {
  align-self: flex-start;
  padding: 1px 8px;
  border-radius: 99px;
  font-size: 11px;
  font-weight: 700;
  color: var(--cat-color);
  background: color-mix(in srgb, var(--cat-color) 12%, transparent);
}

/* ---------- Elimina ---------- */
.delete-btn {
  display: grid;
  place-items: center;
  width: 34px;
  height: 34px;
  flex-shrink: 0;
  border: none;
  border-radius: 10px;
  background: transparent;
  color: var(--muted);
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.2s ease, background 0.2s ease, color 0.2s ease;
}

.delete-btn svg {
  width: 18px;
  height: 18px;
  fill: none;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}

/* Il cestino compare solo al passaggio del mouse (o col focus da tastiera) */
.task-item:hover .delete-btn,
.delete-btn:focus-visible {
  opacity: 1;
}

.delete-btn:hover {
  background: rgba(244, 63, 94, 0.12);
  color: #f43f5e;
}

/* Su touch screen non c'è hover, quindi il cestino resta sempre visibile */
@media (hover: none) {
  .delete-btn {
    opacity: 1;
  }
}
</style>
