<template>
    <section class="flex items-center justify-between mb-10">
        <h1 class="text-4xl font-extrabold">
            Summary
        </h1>
        <div>
            <USelectMenu :options="transactionViewOptions" v-model="selectedView" />
        </div>
    </section>

    <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10">
        <Trend title="Income"  :amount="incomeTotal" :last-amount="prevIncomeTotal" :loading="pending" />
        <Trend title="Expense" :amount="expenseTotal" :last-amount="prevExpenseTotal" :loading="pending" />
        <Trend title="Investments" :amount="investmentsTotal" :last-amount="prevInvestmentsTotal" :loading="pending" />
        <Trend title="Saving" :amount="savingTotal" :last-amount="prevSavingTotal" :loading="pending" />
    </section>

    <section class="flex justify-between mb-10">
        <div>
            <h2 class="text-2xl font-extrabold">Transactions</h2>
            <div class="text-gray-500 dark:text-gray-400">
                You have {{ incomeCount }} incomes and {{ expenseCount }} expense this period
            </div>
        </div>
        <div>
            <TransactionModal v-model="isOpen" @saved="refresh()"/>
            <UButton 
            icon="i-heroicons-plus-circle"
             color="white"
              class="text-gray-600 dark:text-white" variant="solid" label="Add" @click="isOpen = true "/>
        </div>
    </section>

    <section v-if="!pending">
        <div v-for="(transactionsOnDay, date) in transactionsGroupedByDate" :key="date" class="mb-10">
            <DailyTransactionsSummary :date="date" :transactions="transactionsOnDay" />
            <Transactions v-for="transaction in transactionsOnDay" :key="transaction.id" :transaction="transaction" @deleted="refresh()" @edited="refresh()" />
        </div>
    </section>
    <section v-else>
        <USkeleton class="h-8 w-full mb-2" v-for="i in 4" :key="i"/>
    </section>

</template>
<script setup>

import { transactionViewOptions } from '~/constants';

const user = useSupabaseUser()
const selectedView = ref(user.value.user_metadata?.transaction_view ?? transactionViewOptions[1])
const isOpen = ref(false)
const { current, previous} = useSelectedTimePeriod(selectedView);


const {pending,refresh,transactions: {
    incomeTotal,
    incomeCount,
    investmentsTotal,
    expenseTotal,
    expenseCount,
    savingTotal,
    grouped: {
        byDate: transactionsGroupedByDate
    }
}} = useFetchTransactions(current)


const { transactions: {
    incomeTotal: prevIncomeTotal,
    investmentsTotal: prevInvestmentsTotal,
    expenseTotal: prevExpenseTotal,
    savingTotal: prevSavingTotal,
}} = useFetchTransactions(previous)


await refresh();

</script>