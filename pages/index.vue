<script setup>
import { transactionViewOptions } from "~/constants.js";
const selectedView = ref(transactionViewOptions[1]);
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
  >
    <TrendSection color="green" title="Income" :amount="4000" :loading="true" />
    <TrendSection
      color="green"
      title="Income"
      :amount="4000"
      :loading="false"
    />
    <TrendSection color="red" title="Income" :amount="4000" :loading="false" />
    <TrendSection
      color="green"
      title="Income"
      :amount="4000"
      :loading="false"
    />
  </section>
</template>
