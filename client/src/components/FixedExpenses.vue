<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
    monthlyData: {
        type: Object,
        required: true
    },
    categories: {
        type: Array,
        required: true
    }
})

const emit = defineEmits(['save-item'])

const availableYears = [2025, 2026, 2027]
const selectedYear = ref(2026)

const monthsList = [
    { num: '01', label: 'Jan' },
    { num: '02', label: 'Fev' },
    { num: '03', label: 'Mar' },
    { num: '04', label: 'Abr' },
    { num: '05', label: 'Mai' },
    { num: '06', label: 'Jun' },
    { num: '07', label: 'Jul' },
    { num: '08', label: 'Ago' },
    { num: '09', label: 'Set' },
    { num: '10', label: 'Out' },
    { num: '11', label: 'Nov' },
    { num: '12', label: 'Dez' }
]

const selectedMonthNum = ref('09')

const currentPeriodKey = computed(() => `${selectedYear.value}-${selectedMonthNum.value}`)

const currentMonthData = computed(() => {
    return props.monthlyData[currentPeriodKey.value] || { incomes: [], expenses: [] }
})

const totalIncomes = computed(() => {
    return currentMonthData.value.incomes.reduce((acc, item) => acc + item.amount, 0)
})

const totalExpenses = computed(() => {
    return currentMonthData.value.expenses.reduce((acc, item) => acc + item.amount, 0)
})

const balanceDay5 = computed(() => {
    const incomes = currentMonthData.value.incomes
        .filter(item => item.day <= 5)
        .reduce((acc, item) => acc + item.amount, 0)

    const expenses = currentMonthData.value.expenses
        .filter(item => item.dueDate <= 5)
        .reduce((acc, item) => acc + item.amount, 0)

    return incomes - expenses
    })

const balanceDay20 = computed(() => {
    const incomes = currentMonthData.value.incomes
        .filter(item => item.day > 5 && item.day <= 20)
        .reduce((acc, item) => acc + item.amount, 0)

    const expenses = currentMonthData.value.expenses
        .filter(item => item.dueDate > 5 && item.dueDate <= 20)
        .reduce((acc, item) => acc + item.amount, 0)

    return incomes - expenses
})

const totalBalance = computed(() => totalIncomes.value - totalExpenses.value)

const formatCurrency = (val) => {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(val)
}

const showModal = ref(false)
const modalMode = ref('add')
const itemType = ref('income')

const formData = ref({
    id: null,
    name: '',
    amount: 0,
    day: 1,
    dueDate: 1,
    category: '',
    applyAllMonths: false
})

const openAddModal = (type) => {
    modalMode.value = 'add'
    itemType.value = type
    formData.value = {
        id: null,
        name: '',
        amount: 0,
        day: 5,
        dueDate: 10,
        category: props.categories[0] || 'Outros',
        applyAllMonths: false
    }

    showModal.value = true
}

const openEditModal = (type, item) => {
    modalMode.value = 'edit'
    itemType.value = type
    formData.value = {
        id: item.id,
        name: item.name,
        amount: item.amount,
        day: item.day || 1,
        dueDate: item.dueDate || 1,
        category: item.category || props.categories[0] || 'Outros',
        applyAllMonths: false
    }

    showModal.value = true
}

const submitForm = () => {
    emit('save-item', {
        type: itemType.value,
        mode: modalMode.value,
        targetMonthKey: currentPeriodKey.value,
        itemData: { ...formData.value }
    })

    showModal.value = false
}
</script>

<template>
    <div class="financial-container">
        <div class="year-selector">
        <button 
            v-for="year in availableYears" 
            :key="year"
            class="year-btn"
            :class="{ active: selectedYear === year }"
            @click="selectedYear = year"
        >
            {{ year }}
        </button>
    </div>

    <div class="month-selector">
        <button 
            v-for="m in monthsList" 
            :key="m.num"
            class="month-btn"
            :class="{ active: selectedMonthNum === m.num }"
            @click="selectedMonthNum = m.num"
        >
            {{ m.label }}
        </button>
    </div>
    
    <div class="summary-cards">
        <div class="summary-card income">
            <span>Total Receitas</span>
            <strong>{{ formatCurrency(totalIncomes) }}</strong>
        </div>
        <div class="summary-card expense">
            <span>Total Despesas</span>
            <strong>{{ formatCurrency(totalExpenses) }}</strong>
        </div>
        <div class="summary-card period" :class="{ negative: balanceDay5 < 0 }">
            <span>Saldo Restante (Dia 05)</span>
            <strong>{{ formatCurrency(balanceDay5) }}</strong>
        </div>
        <div class="summary-card period" :class="{ negative: balanceDay20 < 0 }">
            <span>Saldo Restante (Dia 20)</span>
            <strong>{{ formatCurrency(balanceDay20) }}</strong>
        </div>
        <div class="summary-card balance" :class="{ negative: totalBalance < 0 }">
            <span>Saldo Final do Mês</span>
            <strong>{{ formatCurrency(totalBalance) }}</strong>
        </div>
    </div>

    <section class="section-block">
        <div class="section-header">
            <h3>Receitas (Entradas)</h3>
            <button class="btn-add income" @click="openAddModal('income')">+ Adicionar Entrada</button>
        </div>

        <div class="cards-grid">
            <div v-for="item in currentMonthData.incomes" :key="item.id" class="mini-card income-border">
                <div class="card-top">
                    <span class="category-tag">{{ item.category || 'Geral' }}</span>
                    <button class="btn-edit" @click="openEditModal('income', item)">✎</button>
                </div>

                <div class="card-info">
                    <span class="card-title">{{ item.name }}</span>
                    <span class="card-date">Dia {{ item.day }}</span>
                </div>
                <div class="card-value income-text">{{ formatCurrency(item.amount) }}</div>
            </div>
        </div>
    </section>

    <section class="section-block">
        <div class="section-header">
            <h3>Contas Fixas (Saídas)</h3>
            <button class="btn-add expense" @click="openAddModal('expense')">+ Adicionar Conta</button>
        </div>

        <div class="cards-grid">
            <div v-for="item in currentMonthData.expenses" :key="item.id" class="mini-card expense-border">
            <div class="card-top">
                <span class="category-tag">{{ item.category || 'Geral' }}</span>
                <button class="btn-edit" @click="openEditModal('expense', item)">✎</button>
            </div>

            <div class="card-info">
                <span class="card-title">{{ item.name }}</span>
                <span class="card-status" :class="item.status">
                {{ item.status === 'paid' ? 'Pago' : 'Vence dia ' + item.dueDate }}
                </span>
            </div>
            <div class="card-value expense-text">{{ formatCurrency(item.amount) }}</div>
            </div>
        </div>
    </section>

    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
        <div class="modal-card">
            <h3>{{ modalMode === 'add' ? 'Adicionar' : 'Editar' }} {{ itemType === 'income' ? 'Receita' : 'Despesa' }}</h3>

            <form @submit.prevent="submitForm" class="modal-form">
            <div class="form-group">
                <label>Descrição</label>
                <input type="text" v-model="formData.name" required placeholder="Ex: Aluguel, Salário" />
            </div>

            <div class="form-group">
                <label>Valor (R$)</label>
                <input type="number" step="0.01" v-model.number="formData.amount" required />
            </div>

            <div class="form-group">
                <label>Categoria</label>
                <select v-model="formData.category" required>
                <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
                </select>
            </div>

            <div class="form-group" v-if="itemType === 'income'">
                <label>Dia do Recebimento</label>
                <input type="number" min="1" max="31" v-model.number="formData.day" required />
            </div>

            <div class="form-group" v-else>
                <label>Dia do Vencimento</label>
                <input type="number" min="1" max="31" v-model.number="formData.dueDate" required />
            </div>

            <div class="form-checkbox" v-if="modalMode === 'add'">
                <input type="checkbox" id="allMonths" v-model="formData.applyAllMonths" />
                <label for="allMonths">Aplicar para todos os meses de {{ selectedYear }}</label>
            </div>

            <div class="modal-actions">
                <button type="button" class="btn-cancel" @click="showModal = false">Cancelar</button>
                <button type="submit" class="btn-submit">Salvar</button>
            </div>
            </form>
        </div>
        </div>
    </div>
</template>

<style scoped>
.year-selector {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
}

.year-btn {
  padding: 0.3rem 0.8rem;
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--text-muted);
  background: transparent;
  border: none;
  border-bottom: 2px solid transparent;
}

.year-btn.active {
  color: var(--primary);
  border-bottom-color: var(--primary);
}

.month-selector {
  display: flex;
  gap: 0.35rem;
  overflow-x: auto;
  padding-bottom: 0.75rem;
  margin-bottom: 1.5rem;
  border-bottom: 1px solid var(--border);
}

.month-btn {
  padding: 0.4rem 0.75rem;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-muted);
  background: var(--bg-main);
  border: 1px solid var(--border);
  border-radius: var(--radius);
}

.month-btn.active {
  background-color: var(--primary);
  color: white;
  border-color: var(--primary);
}

.summary-cards {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.75rem;
  margin-bottom: 1.5rem;
}

.summary-card {
  padding: 0.85rem 1rem;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background-color: var(--bg-main);
  display: flex;
  flex-direction: column;
}

.summary-card span {
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 500;
}

.summary-card strong {
  font-size: 1.1rem;
  margin-top: 0.2rem;
}

.summary-card.income strong { color: var(--success); }
.summary-card.expense strong { color: var(--danger); }
.summary-card.period strong { color: var(--primary); }
.summary-card.period.negative strong { color: var(--danger); }
.summary-card.balance.negative strong { color: var(--danger); }

.section-block {
  margin-bottom: 1.5rem;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.section-header h3 {
  font-size: 0.95rem;
  font-weight: 600;
}

.btn-add {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 0.35rem 0.65rem;
  border-radius: 6px;
  background: transparent;
  border: 1px solid var(--border);
}

.btn-add.income { color: var(--success); border-color: var(--success); }
.btn-add.expense { color: var(--primary); border-color: var(--primary); }

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 0.65rem;
}

.mini-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 0.65rem 0.85rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 0.4rem;
  box-shadow: var(--shadow-sm);
}

.income-border { border-left: 3px solid var(--success); }
.expense-border { border-left: 3px solid var(--primary); }

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.category-tag {
  font-size: 0.65rem;
  font-weight: 600;
  text-transform: uppercase;
  background-color: #f1f5f9;
  color: var(--text-muted);
  padding: 0.15rem 0.4rem;
  border-radius: 4px;
}

.btn-edit {
  background: transparent;
  color: var(--text-muted);
  font-size: 0.85rem;
}

.card-info {
  display: flex;
  flex-direction: column;
}

.card-title {
  font-size: 0.825rem;
  font-weight: 600;
  color: var(--text-main);
}

.card-date, .card-status {
  font-size: 0.7rem;
  color: var(--text-muted);
}

.card-status.paid {
  color: var(--success);
  font-weight: 600;
}

.card-value {
  font-size: 0.95rem;
  font-weight: 700;
}

.income-text { color: var(--success); }
.expense-text { color: var(--text-main); }

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-card {
  background-color: var(--surface);
  border-radius: var(--radius);
  padding: 1.5rem;
  width: 100%;
  max-width: 420px;
  box-shadow: var(--shadow-md);
}

.modal-card h3 {
  font-size: 1.1rem;
  margin-bottom: 1rem;
}

.modal-form {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.form-group label {
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-muted);
}

.form-group input,
.form-group select {
  padding: 0.5rem;
  border: 1px solid var(--border);
  border-radius: 6px;
  font-size: 0.875rem;
}

.form-checkbox {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.8rem;
  color: var(--text-main);
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.btn-cancel {
  padding: 0.4rem 0.85rem;
  background-color: transparent;
  color: var(--text-muted);
  border: 1px solid var(--border);
  border-radius: 6px;
  font-size: 0.8rem;
}

.btn-submit {
  padding: 0.4rem 0.85rem;
  background-color: var(--primary);
  color: white;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 600;
}
</style>