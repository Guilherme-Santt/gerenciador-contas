<script setup>
defineProps({
  expenses: {
    type: Array,
    required: true
  }
})

const formatCurrency = (val) => {
  return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(val)
}
</script>

<template>
  <div class="fixed-expenses-container">
    <div class="section-header">
      <div>
        <h2>Contas Fixas Cadastradas</h2>
        <p class="subtitle">Despesas mensais recorrentes planejadas.</p>
      </div>
      <button class="btn-primary">+ Nova Conta</button>
    </div>

    <div class="cards-grid">
      <div v-for="item in expenses" :key="item.id" class="expense-card">
        <div class="card-header">
          <span class="expense-name">{{ item.name }}</span>
          <span class="status-badge" :class="item.status">
            {{ item.status === 'paid' ? 'Pago' : 'Pendente' }}
          </span>
        </div>

        <div class="card-body">
          <div class="amount">{{ formatCurrency(item.amount) }}</div>
          <div class="due-date">Vence dia <strong>{{ item.dueDate }}</strong></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.section-header h2 {
  font-size: 1.25rem;
  font-weight: 600;
}

.subtitle {
  color: var(--text-muted);
  font-size: 0.875rem;
}

.btn-primary {
  background-color: var(--primary);
  color: white;
  padding: 0.5rem 1rem;
  border-radius: var(--radius);
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.2s;
}

.btn-primary:hover {
  background-color: var(--primary-hover);
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
}

.expense-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  box-shadow: var(--shadow-sm);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.expense-name {
  font-weight: 600;
  color: var(--text-main);
}

.status-badge {
  font-size: 0.75rem;
  padding: 0.2rem 0.5rem;
  border-radius: 999px;
  font-weight: 500;
}

.status-badge.paid {
  background-color: #dcfce7;
  color: #15803d;
}

.status-badge.pending {
  background-color: #fef3c7;
  color: #b45309;
}

.amount {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text-main);
}

.due-date {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-top: 0.25rem;
}
</style>