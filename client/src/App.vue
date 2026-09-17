<script setup>
import { ref } from 'vue'
import FixedExpenses from './components/FixedExpenses.vue'

const currentTab = ref('fixed-expenses')

const mockData = ref({
  fixedExpenses: [
    { id: 1, name: 'Aluguel', amount: 1800.00, dueDate: 10, status: 'pending' },
    { id: 2, name: 'Internet', amount: 120.00, dueDate: 15, status: 'paid' },
    { id: 3, name: 'Energia (CPFL)', amount: 195.50, dueDate: 22, status: 'pending' }
  ],
  transactions: [
    { id: 101, date: '2026-09-02', description: 'Supermercado', amount: -245.80 },
    { id: 102, date: '2026-09-10', description: 'Pagamento Aluguel', amount: -1800.00 },
    { id: 103, date: '2026-09-15', description: 'Pagamento Internet', amount: -120.00 }
  ]
})
</script>

<template>
  <div class="app-layout">
    <header class="header">
      <div class="header-brand">
        <h1>Financeiro</h1>
        <span class="badge">Protótipo</span>
      </div>

      <nav class="nav">
        <button 
          class="nav-btn"
          :class="{ active: currentTab === 'fixed-expenses' }"
          @click="currentTab = 'fixed-expenses'"
        >
          Contas Fixas
        </button>
        <button 
          class="nav-btn"
          :class="{ active: currentTab === 'import' }"
          @click="currentTab = 'import'"
        >
          Conciliação / Extrato
        </button>
      </nav>
    </header>

    <main class="main-content">
      <FixedExpenses 
        v-if="currentTab === 'fixed-expenses'" 
        :expenses="mockData.fixedExpenses" 
      />

      <section v-if="currentTab === 'import'">
        <h2>Conciliação de Extrato</h2>
        <p class="subtitle">Ainda vamos criar a interface do extrato aqui.</p>
      </section>
    </main>
  </div>
</template>

<style scoped>
.app-layout {
  max-width: 1000px;
  margin: 2rem auto;
  padding: 0 1.5rem;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.header-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.header-brand h1 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text-main);
}

.badge {
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  background-color: #e0e7ff;
  color: var(--primary);
  border-radius: 999px;
  font-weight: 600;
}

.nav {
  display: flex;
  gap: 0.5rem;
  background-color: #f1f5f9;
  padding: 0.25rem;
  border-radius: var(--radius);
}

.nav-btn {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--text-muted);
  background: transparent;
  border-radius: calc(var(--radius) - 2px);
  transition: all 0.2s ease;
}

.nav-btn.active {
  background-color: var(--surface);
  color: var(--text-main);
  box-shadow: var(--shadow-sm);
}

.main-content {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.5rem;
  box-shadow: var(--shadow-sm);
}

.subtitle {
  color: var(--text-muted);
  font-size: 0.875rem;
}
</style>