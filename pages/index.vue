<script setup>
import { transactionViewOptions } from "~/constants.js";
const selectedView = ref(transactionViewOptions[1]);

const supabase = useSupabaseClient();

const transactions = ref([]);
const isOpen = ref(false);
const isLoading = ref(false);

const fetchTransactions = async () => {
  isLoading.value = true;

  try {
    const { data } = await useAsyncData("transactions", async () => {
      const { data, error } = await supabase.from("transactions").select();

      if (error) throw error;

      return data;
    });
    transactions.value = data.value;
  } catch (error) {
    console.log(error);
  } finally {
    isLoading.value = false;
  }
};
await fetchTransactions();

const transactionsGroupedByDate = computed(() => {
  let grouped = {};

  for (const transaction of transactions.value) {
    const date = new Date(transaction.created_at).toISOString().split("T")[0];

    if (!grouped[date]) {
      grouped[date] = [];
    }

    grouped[date].push(transaction);
  }

  return grouped;
});

const income = computed(() => transactions.value.filter((t) => t.type === "Income"));
const expense = computed(() => transactions.value.filter((t) => t.type === "Expense"));
const incomeCount = computed(() => income.value.length);
const expenseCount = computed(() => expense.value.length);
const incomeTotal = computed(() =>
  income.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);
const expenseTotal = computed(() =>
  expense.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);
</script>

<template>
  <section class="flex items-center justify-between mb-10">
    <h1 class="text-4xl font-extrabold">Summary</h1>
    <div>
      <USelect v-model="selectedView" :options="transactionViewOptions" />
    </div>
  </section>

  <section
    class="grid gid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10"
    v-if="!isLoading"
  >
    <TrendSection
      color="green"
      title="Income"
      :amount="incomeTotal"
      :last-amount="3000"
      :loading="isLoading"
    />
    <TrendSection
      color="green"
      title="Expense"
      :amount="expenseTotal"
      :last-amount="6000"
      :loading="isLoading"
    />
    <TrendSection
      color="red"
      title="Investments"
      :amount="4000"
      :last-amount="2000"
      :loading="isLoading"
    />
    <TrendSection
      color="green"
      title="Savings"
      :last-amount="8000"
      :amount="4000"
      :loading="isLoading"
    />
  </section>

  <section class="flex justify-between mb-10">
    <div>
      <h2 class="text-2xl font-extrabold">Transactions</h2>
      <div class="text-gray-500 dark:text-gray-400">
        You have {{ incomeCount }} incomes and {{ expenseCount }} expenses this period
      </div>
    </div>
    <div>
      <TransactionModal v-model="isOpen" />
      <UButton
        icon="i-heroicons-plus-circle"
        color="white"
        variant="solid"
        label="Add"
        @click="isOpen = true"
      />
    </div>
  </section>

  <section v-if="!isLoading">
    <div
      v-for="(transactionsOnDay, date) in transactionsGroupedByDate"
      :key="date"
      class="mb-10"
    >
      <DailyTransactionSummary :date="date" :transactions="transactionsOnDay" />
      <TransactionView
        v-for="transaction in transactionsOnDay"
        :key="transaction.id"
        :transaction="transaction"
        @transactionDeleted="fetchTransactions"
      />
    </div>
  </section>
  <section v-else><USkeleton class="h-8 w-full mb-2" v-for="i in 4" :key="i" /></section>
</template>
