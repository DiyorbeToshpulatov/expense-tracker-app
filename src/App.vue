<template>
  <Header></Header>
  <div class="container">
    <Balance :total="total"></Balance>
    <IncomeExpenses :income="+income" :expenses="+expenses"></IncomeExpenses>
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="hadnleTransactionDeleted"
    ></TransactionList>
    <AddTransaction
      @transactionSubmitted="handleTransactionSubmitted"
    ></AddTransaction>
  </div>
</template>

<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";

import { useToast } from "vue-toastification";

import { ref, computed, onMounted } from "vue";

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

//Get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Add transaction

const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqeId(),
    text: transactionData.text,
    amount: transactionData.amount,
  });

  saveTransactionsToLocalStorage();

  toast.success("Transaction added");
};

// Generate uniqe ID
const generateUniqeId = () => {
  return Math.floor(Math.random() * 100000);
};

// Delete transaction
const hadnleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );

  saveTransactionsToLocalStorage();

  toast.success("Transaction Deleted");
};

// Save to localstorage

const saveTransactionsToLocalStorage = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};
</script>
