<script setup>
import { onMounted, ref } from 'vue'
import db from './firebase/config.js'
import { collection, query, getDocs, setDoc, doc, onSnapshot, addDoc, deleteDoc} from 'firebase/firestore'


const expenses = ref([]);
const date = ref(null);
const isEditing = ref(false)
const expenseId = ref(null);

const getExpense = async ()=>{
  onSnapshot(collection(db, 'expenses'), (querySnap) =>{
    const tmp = []
    querySnap.forEach((doc)=>{
      const expense = {
        id: doc.id,
        data: doc.data()
      }
      tmp.push(expense)
    })
    expenses.value = tmp
  })
}

const addExpense = async () => {
  if (!description.value || !category.value || !amount.value) {
    alert('Insert all Values!!');
    return;
  }

  if (isEditing.value === false) {
    const data = {
      description: description.value,
      category: category.value,
      amount: amount.value,
      date: new Date()
    };
    const docRef = await addDoc(collection(db, 'expenses'), data);
    console.log('Expense added successfully');
    description.value = '';
    category.value = '';
    amount.value = null;
  } else {
    updateExpense();
  }

  description.value = null;
  category.value = null;
  amount.value = null;
  expenseId.value = null;
}

const formatDateTime = (timestamp) => {
  if (!timestamp || !timestamp.seconds) {
    return "Invalid Date";
  }

  const milliseconds =
    timestamp.seconds * 1000 + timestamp.nanoseconds / 1000000;
  const date = new Date(milliseconds);
  return date.toLocaleString();
};

const onEdit = async (data, id) => {
  description.value = data.description;
  category.value = data.category;
  amount.value = data.amount;
  expenseId.value = id; 
  isEditing.value = true;
  date.value = data.date
}

const updateExpense = async () => {
  if (!expenseId.value) {
    console.error('Expense ID is not set');
    return;
  }

  const data = {
    description: description.value,
    category: category.value,
    amount: amount.value,
    date: date.value,
    dateEdited: new Date(),
    isEdited: true
  };

  try {
    await setDoc(doc(db, 'expenses', expenseId.value), data);
    console.log('Expense updated successfully');
  } catch (error) {
    console.error('Error updating expense:', error);
  }
}



const deleteExpense = async (id) =>{
    if(confirm('Delete Expense?')){
        await deleteDoc(doc(db, 'expenses', id))
    }
}

onMounted(()=>{
  getExpense()
})


</script>

<template>
  <div class="adjust justify-center items-center h-full bg-blue-300 shadow-lg rounded-lg p-2">
    <h1 class="title text-4xl font-bold text-center">Expense Tracker</h1>
    <div class="container flex flex-col justify-center mx-auto md:flex-row left">
      <div class="left lg:w-1/3 m-3 p-3 border border-slate-500 rounded h-1/3 bg-gray-200">
        <form @submit.prevent="addExpense">
          <label for="description" class="my-5">Description: </label><br>
          <input type="text"  class="border border-slate-500 rounded text-black mb-5 p-2" id="description" placeholder="description"><br>
          <label for="category" class="my-5">Category: </label><br>
          <input type="text"  class="border border-slate-500 rounded text-black mb-5 p-2" id="category" placeholder="category"><br>
          <label for="amount" class="my-5">Amount: : </label><br>
          <input type="number"  class="border border-slate-500 rounded text-black mb-5 p-2" id="amount" placeholder="₱:"><br>
          <button type="submit"class="border-2 border-black py-2 rounded text-black hover:opacity-80 hover:border-white hover:text-black hover:bg-white transition ease-linear duration-100">
            <svg xmlns="http://www.w3.org/2000/svg" width="60" height="16" fill="currentColor" class="bi bi-send" viewBox="0 0 16 16">
              <path d="M15.854.146a.5.5 0 0 1 .11.54l-5.819 14.547a.75.75 0 0 1-1.329.124l-3.178-4.995L.643 7.184a.75.75 0 0 1 .124-1.33L15.314.037a.5.5 0 0 1 .54.11ZM6.636 10.07l2.761 4.338L14.13 2.576zm6.787-8.201L1.591 6.602l4.339 2.76z"/>
            </svg>
          </button>
        </form>
      </div>
      <div class="center ml-0 m-3 w-max border border-slate-500 rounded p-3 overflow-auto max-h-[400px]">
        <table class="scrolldown">
          <thead>
            <tr>
              <th class="px-3 py-2 border text-lg">Description</th>
              <th class="px-3 py-2 border text-lg">Category</th>
              <th class="px-3 py-2 border text-lg">Amount</th>
              <th class="px-3 py-2 border text-lg">Date</th>
              <th class="px-3 py-2 border text-lg">Edit</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="expense in expenses">
              <td class="px-3 py-2 border nowrap">{{ expense.data.description }}</td>
              <td class="px-3 py-2 border nowrap">{{ expense.data.category }}</td>
              <td class="px-3 py-2 border nowrap">{{ "₱" + parseFloat(expense.data.amount).toFixed(2) }}</td>
              <td class="px-3 py-2 border nowrap">{{ formatDateTime(expense.data.date) }}</td>
              <td class="px-3 py-2 border text-center flex gap-px align-center">
                <button @click="onEdit(expense.data, expense.id)" class="mx-1 mb-3 mt-3">
                  <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 text-blue-500">
                    <path stroke-linecap="round" stroke-linejoin="round" d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10"/>
                  </svg>
                </button>
                <button @click="deleteExpense(expense.id)" class="mx-1 mb-3 mt-3">
                  <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"  stroke="currentColor" class="w-6 h-6 text-red-500">
                    <path stroke-linecap="round" stroke-linejoin="round" d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"/>
                  </svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>



