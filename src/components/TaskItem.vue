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
  <div class="task-item" :class="{ completed: task.completed }">
    <!-- Pallino colorato in base alla categoria. Uso ?. (optional
         chaining) perché se una task non ha una categoria valida
         (es. task vecchie) non voglio che l'app vada in errore -->
    <span
      class="category-dot"
      :style="{ background: categories[task.category]?.color || '#999' }"
      :title="categories[task.category]?.label"
    ></span>

    <input
      type="checkbox"
      :checked="task.completed"
      @change="emit('toggle-complete', task.id)"
    />

    <span class="task-text">{{ task.text }}</span>

    <button class="delete-btn" @click="emit('delete-task', task.id)">
      ✕
    </button>
  </div>
</template>

<style scoped>
.task-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: var(--card-bg, #fff);
  border-radius: 8px;
  margin-bottom: 8px;
  transition: opacity 0.2s ease;
}

.task-item.completed .task-text {
  text-decoration: line-through;
  opacity: 0.5;
}

.task-text {
  flex: 1;
  text-align: left;
}

.category-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
}

input[type="checkbox"] {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.delete-btn {
  background: transparent;
  border: none;
  color: #e74c3c;
  cursor: pointer;
  font-size: 16px;
  padding: 4px 8px;
  border-radius: 4px;
}

.delete-btn:hover {
  background: rgba(231, 76, 60, 0.1);
}
</style>
