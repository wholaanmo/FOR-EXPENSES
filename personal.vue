<template>
  <navigation/>
 
   <header class="container">
     <div class="content-wrapper">
       <div v-if="successMessage" class="success-message">{{ successMessage }}</div>
 
       <form @submit.prevent="handleSubmit">
         <input type="hidden" v-model="action" />
         <input type="hidden" v-if="editId" v-model="editId" />
 
         <div class="form-group">
           <label>EXPENSE TYPE:</label>
           <select v-model="expenseType" required @change="checkExpenseType">
             <option value="Food">Food</option>
             <option value="Bill">Bill</option>
             <option value="Transportation">Transportation</option>
             <option value="Entertainment">Entertainment</option>
             <option value="Healthcare">Healthcare</option>
             <option value="Personalcare">Personal care</option>
             <option value="Shopping">Shopping</option> 
             <option value="Other">Other</option>
            </select>
         </div>
 
         <div v-if="expenseType === 'Other'" class="form-group">
           <label>Custom Expense Type:</label>
           <input type="text" v-model="customExpenseType" placeholder="Enter custom expense type" />
         </div>
 
         <div class="form-group">
           <label>ITEM NAME:</label>
           <input type="text" v-model="itemName" placeholder="Enter item name" required />
         </div>
 
         <div class="form-group">
           <label>ITEM PRICE:</label>
           <input type="number" v-model.number="itemPrice" placeholder="Enter item price" required step="0.01" />
         </div>
 
         <button class="btn" type="submit">{{ editId ? 'Save Changes' : 'Add Expense' }}</button>
        <!--<button class="btn1" type="submit">{{ editId ? 'Save Changes' : 'Add Budget' }}</button> -->
       </form>
 
       <div>
         <div class="h3">Your Expenses</div>
         <ul>
           <li v-for="expense in expenses" :key="expense.id">
            {{ expense.item_name }} - ₱{{ expense.item_price.toFixed(2) }}
           </li>
         </ul>
       </div>
     </div>
     <div class="total">
       Total: ₱{{ (totalAmount * 50).toFixed(2) }} (≈ ${{ totalAmount.toFixed(2) }})
     </div>
    </header>
 </template>
 
 <script>
 import Navigation from "./navigation.vue";
 import axios from 'axios';
 
 export default {
   components: { Navigation },
   data() {
     return {
       expenseType: '',
       customExpenseType: '',
       itemName: '',
       itemPrice: '',
       successMessage: '',
       editId: null,
       expenses: [],
       personalBudgetId: null, //TEMPORARYYYYYYYYYYYYYYYYYY
       action: 'add'
     };
   },
   computed: {
     totalAmount() {
      return this.expenses?.reduce((sum, expense) => sum + (expense.item_price || 0), 0) || 0; //NEWWWWWWWWWWWWWWWWWWWW
     }
   },
   methods: {
     fetchExpenses() {
       axios.get('/api/expenses', {
         headers: { Authorization: `Bearer ${localStorage.getItem('jsontoken')}` }
       })
       .then(response => {
        this.expenses = response.data?.data || []; // NEWWWWWWWWWWWWWWWWWW
       })
       .catch(error => {
        console.error("Error fetching expenses:", error);
        this.expenses = []; // Reset to empty array on error
        this.successMessage = 'Failed to load expenses. Please try again.';
        });
    },
     handleSubmit() {
       const expenseData = {
         item_price: this.itemPrice,
         expense_type: this.expenseType === 'Other' ? this.customExpenseType : this.expenseType,
         item_name: this.itemName,
         personal_budget_id: this.personalBudgetId //TEMPORARYYYYYYYYYYYYYYYYYYYYYYY
       };
       const config = {
         headers: {
           Authorization: `Bearer ${localStorage.getItem('jsontoken')}`
         }
       };
       
       if (this.editId) {
         // Edit existing expense
         axios.put('/api/expenses', { ...expenseData, id: this.editId }, config)
         .then(() => {
           this.successMessage = 'Expense updated successfully!';
           this.fetchExpenses();
           this.resetForm();
         })
         .catch(error => {
           console.error("Error updating expense:", error);
           this.successMessage = 'Failed to update expense.';
         });
       } else {

         // Add new expense
         axios.post('/api/expenses', expenseData, config)
      .then(response => {
        console.log('Full response:', response); //NEWWWWWWWWWWWWWWW
        if (response.data.success === 1) {
          this.successMessage = 'Expense added successfully!';
          this.fetchExpenses();
          this.resetForm();
        } else {
          this.successMessage = 'Failed to add expense: ' + (response.data.message || 'Unknown error');
        }
      })
      .catch(error => {
        console.error("Full error:", error.response); // Add this
        this.successMessage = 'Failed to add expense: ' + 
            (error.response?.data?.message || error.message || 'Unknown error');
    });
  }
},
     editExpense(expense) {
       this.editId = expense.id;
       this.expenseType = expense.expense_type;
       this.itemName = expense.item_name;
       this.itemPrice = expense.item_price;
     },
     deleteExpense(id) {
       if (confirm('Are you sure you want to delete this expense?')) {
         axios.delete(`/api/expenses/${id}`, {
           headers: { Authorization: `Bearer ${localStorage.getItem('jsontoken')}` }
         })
         .then(() => {
           this.successMessage = 'Expense deleted successfully!';
           this.fetchExpenses();
         });
       }
     },
     resetForm() {
       this.expenseType = '';
       this.customExpenseType = '';
       this.itemName = '';
       this.itemPrice = '';
       this.editId = null;
     }
   },
   mounted() {
     this.fetchExpenses();
   }
 };
 </script>

 
 
 <style scoped>
 header {
  background-color: #2a4935;
  z-index: 99;
  width: 100%;
  position: sticky;
  transition: .5s ease all;
  color: #f6f8d5;
}

 .container {
    background-color: #9dd7af;
    max-width: 1000px;
    max-height: 75vh;
    width: 90%;
    margin: 20px auto 20px auto; /* Top, auto-sides, bottom */
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 74vh;
    overflow-y: auto;
    overflow-x: hidden;
    justify-content: flex-start;
    padding-top: 20px;
    z-index: 1;
    position: relative; /* important: NOT absolute */
    box-sizing: border-box;
}

 
 .content-wrapper {
     width: 100%; /* Ensures the form takes the full width */
     overflow-y: visible;
     display: flex;
     flex-wrap: wrap;
     flex-direction: column;
     margin-top: 30px;
     align-items: center; /* Centers form inputs */
 }
 
 .success-message {
     color: black;
     padding: 10px;
     margin-bottom: 20px;
     border-radius: 5px;
     text-align: center;
 }
 
 table {
     width: 75%;
     border-collapse: collapse;
     margin-top: 20px;
 }
 
 table, th, td {
     border: 1px solid black;
     padding: 7px;
     text-align: left;
     color: black;
 }
 
 th {
     background-color: rgba(255, 255, 255, 0.15);
     color: black;
 }

 .h3{
  font-size: 24px;
  font-weight: bold;
  margin-top: 30px;
  color: black;
 }
 
 .total {
     font-size: 20px;
     font-weight: bold;
     color: red;
     padding: 20px;
     background-color: white;
     box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
     text-align: center;
     width: 1100px;
     position: relative; /* Keeps it visible at the bottom */
     bottom: 0; /* Stick to the bottom */

 
 }
 
 .form-group {
     margin-bottom: 20px;
     margin-top: 20px;
 }
 
 label {
     display: block;
     margin-bottom: 5px;
     font-size: 17px;
     color: black;
 }
 
 input[type="text"],
 input[type="number"],
 select {
     width: 800px;
     padding: 10px;
     font-size: 16px;
     border-radius: 10px;
     border: 3px solid #ddd;
     border-color: black;
     box-sizing: border-box;
     min-height: 35px;
 }
 
 .btn {
     padding: 12px 50px;
     background-color: white;
     box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
     color: black;
     border: none;
     border-radius: 12px;
     cursor: pointer;
     font-size: 15px;
     transition: background-color 0.3s, color 0.3s; /* Smooth effect */
}

.btn:hover {
    background-color: rgb(26, 25, 25); /* Change to any color you want */
    color: white; /* Text color on hover */
}

.btn1 {
    padding: 12px 50px;
    background-color: white;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
    color: black;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    font-size: 15px;
    margin-top: 10px; /* Add vertical spacing instead */
    margin-left: 430px;
    transition: background-color 0.3s, color 0.3s;
}

.btn1:hover {
    background-color: rgb(26, 25, 25); /* Change to any color you want */
    color: white; /* Text color on hover */
}

/* RESPONSIVE DESIGN */

@media screen and (max-width: 1000px) {
    .container {
        margin: 20px 0px 0px 30px;
        padding: 15px;
        height: 500px;

    }

    .content-wrapper {
        margin-top: 20px;
        padding: 0px;
    }

    .form-group {
        width: 100%;
        padding: 0 0 10 0px;
        margin: 20px 0;
    }

    input[type="text"],
    input[type="number"],
    select {
        width: 100%;
        max-width: 400px;
        font-size: 15px;
    }

    .btn,
    .btn1 {
        width: 100%;
        font-size: 15px;
        margin: 10px auto;
        display: block;
    }

    .h3{
      font-size: 20px;
      font-weight: bold;
      margin-top: 30px;
      color: white;
    }

    .total {
        font-size: 17px;
        position: relative;
    }
}

</style>
