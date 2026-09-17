<script setup>
import { ref } from 'vue'
import FixedExpenses from './components/FixedExpenses.vue'

const currentTab = ref('fixed-expenses')

const categories = ref(['Casa', 'Transporte', 'Alimentação', 'Trabalho', 'Lazer', 'Outros'])

const monthlyData = ref({
  '2026-09': {
    incomes: [
      { id: 1, name: 'Salário', amount: 4500.00, day: 5, category: 'Trabalho' },
      { id: 2, name: 'Freelance', amount: 800.00, day: 20, category: 'Trabalho' }
    ],
    expenses: [
      { id: 101, name: 'Aluguel', amount: 1800.00, dueDate: 10, status: 'pending', category: 'Casa' },
      { id: 102, name: 'Internet', amount: 120.00, dueDate: 15, status: 'paid', category: 'Casa' },
      { id: 103, name: 'Energia (CPFL)', amount: 195.50, dueDate: 22, status: 'pending', category: 'Casa' }
    ]
  },
  '2026-10': {
    incomes: [
      { id: 3, name: 'Salário', amount: 4500.00, day: 5, category: 'Trabalho' }
    ],
    expenses: [
      { id: 104, name: 'Aluguel', amount: 1800.00, dueDate: 10, status: 'pending', category: 'Casa' },
      { id: 105, name: 'Internet', amount: 120.00, dueDate: 15, status: 'pending', category: 'Casa' }
    ]
  }
})

const handleSaveItem = ({ type, mode, targetMonthKey, itemData }) => {
  const collectionKey = type === 'income' ? 'incomes' : 'expenses'
  
  if (mode === 'add') {
    const newItem = { id: Date.now(), ...itemData }

    if (itemData.applyAllMonths) {
      const allMonths = ['01','02','03','04','05','06','07','08','09','10','11','12']
      const year = targetMonthKey.split('-')[0]

      allMonths.forEach(mNum => {
        const key = `${year}-${mNum}`
        if (!monthlyData.value[key]) {
          monthlyData.value[key] = { incomes: [], expenses: [] }
        }
        monthlyData.value[key][collectionKey].push({ ...newItem, id: Date.now() + Math.random() })
      })
    } else {
      if (!monthlyData.value[targetMonthKey]) {
        monthlyData.value[targetMonthKey] = { incomes: [], expenses: [] }
      }
      monthlyData.value[targetMonthKey][collectionKey].push(newItem)
    }
  } else if (mode === 'edit') {
    if (monthlyData.value[targetMonthKey]) {
      const list = monthlyData.value[targetMonthKey][collectionKey]
      const index = list.findIndex(i => i.id === itemData.id)
      if (index !== -1) {
        list[index] = { ...itemData }
      }
    }
  }
}
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
          Planejamento Mensal
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
        :monthly-data="monthlyData" 
        :categories="categories"
        @save-item="handleSaveItem"
      />

      <section v-if="currentTab === 'import'">
        <h2>Conciliação de Extrato</h2>
        <p class="subtitle">Em breve montaremos esta tela.</p>
      </section>
    </main>
  </div>
</template>

<style scoped>
.app-layout {
  width: 100%;
  padding: 1.5rem 2rem;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.header-brand {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.header-brand h1 {
  font-size: 1.25rem;
  font-weight: 700;
}

.badge {
  font-size: 0.7rem;
  padding: 0.2rem 0.4rem;
  background-color: #e0e7ff;
  color: var(--primary);
  border-radius: 999px;
  font-weight: 600;
}

.nav {
  display: flex;
  gap: 0.35rem;
  background-color: #f1f5f9;
  padding: 0.25rem;
  border-radius: var(--radius);
}

.nav-btn {
  padding: 0.4rem 0.85rem;
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--text-muted);
  background: transparent;
  border-radius: calc(var(--radius) - 2px);
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
  font-size: 0.85rem;
}
</style>