<template>
  <navigation/>
 
   <header class="container">
     <div class="content-wrapper">
 
       <form @submit.prevent="handleSubmit" class="expense-form"> <!-- CLASS IS NEWWWWWWWWWWW-->
         <input type="hidden" v-model="action" />
         <input type="hidden" v-if="editId" v-model="editId" />


           <!-- CLASS IS NEWWWWWWWWWWW-->
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
      <div v-if="successMessage" ref="successMessage" class="success-message" :class="{ hide: hideMessage }"> {{ successMessage }} </div>

       <div class="expenses-section"> <!-- NEWWWWWWWWWWW-->
        <h3>Your Expenses</h3> <!-- NEWWWWWWWWWWW-->
         <div class="expenses-table"> <!-- NEWWWWWWWWWWW-->
          <table>
            <thead>
              <tr>
                <th>Expense Type</th>
                <th>Item Name</th>
                <th>Item Price</th>
                <th>Date</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="expense in expenses" :key="expense.id">
                <td>{{ expense.expense_type }}</td>
                <td>{{ expense.item_name }}</td>
                <td>₱{{ expense.item_price.toFixed(2) }}</td>
                <td>{{ formatDate(expense.created_at) }}</td>
                <td class="actions">
                  <button @click="editExpense(expense)" class="edit-btn">Edit</button>
                  <button @click="deleteExpense(expense.id)" class="delete-btn">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

       </div>
     </div>
     <div class="total">
      Total: <strong>₱{{ totalAmount.toFixed(2) }}</strong> (≈ {{ formatUsd(convertPhpToUsd(totalAmount)) }} USD)
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
       action: 'add',
       hideMessage: false,
       successTimeout: null,
       usdExchangeRate: 56.50
     };
   },
   computed: {
    totalAmount() {
      return this.expenses?.reduce((sum, expense) => 
        sum + this.parseCurrency(expense.item_price), 0) || 0;
    },
    totalInUsd() {
      return (this.totalAmount / this.usdExchangeRate).toFixed(2);
    }
  },
  async mounted() {
    try {
      const response = await axios.get('https://api.exchangerate-api.com/v4/latest/USD');
      this.usdExchangeRate = response.data.rates.PHP;
      await this.fetchExpenses();
    } catch (error) {
      console.error("Initialization error:", error);
      await this.fetchExpenses().catch(e => console.error("Expense fetch failed:", e));
    }
  },
  methods: {
    parseCurrency(value) {
      if (!value) return 0;
      const numericValue = String(value).replace(/[^\d.]/g, '');
      return parseFloat(numericValue) || 0;
    },
    convertPhpToUsd(phpAmount) {
      return this.parseCurrency(phpAmount) / this.usdExchangeRate; // Use dynamic rate
    },
    formatUsd(value) {
      return '$' + parseFloat(value).toFixed(2);
    },
    formatPHP(value) {
      const amount = this.parseCurrency(value);
      return '₱' + amount.toLocaleString('en-PH', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      });
    },

  showSuccessMessage(message) {
    // Clear any existing timeout
    if (this.successTimeout) {
      clearTimeout(this.successTimeout);
    }
    
    // Reset hide state and set message
    this.hideMessage = false;
    this.successMessage = message;
    
    // Set timeout to hide after 2.5s
    this.successTimeout = setTimeout(() => {
      this.hideMessage = true;
      
      // Clear message after fade out completes
      setTimeout(() => {
        this.successMessage = '';
      }, 500);
    }, 2500);
  },

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
        this.showSuccessMessage('Failed to load expenses. Please try again.');
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
         axios.put(`/api/expenses/${this.editId}`, expenseData, config)
         .then(() => {
           this.showSuccessMessage('Expense updated successfully!');
           this.fetchExpenses();
           this.resetForm();
         })
         .catch(error => {
           console.error("Error updating expense:", error);
           this.showSuccessMessage('Failed to update expense.'); 
         });
       } else {

         // Add new expense
         axios.post('/api/expenses', expenseData, config)
      .then(response => {
        console.log('Full response:', response); //NEWWWWWWWWWWWWWWW
        if (response.data.success === 1) {
          this.showSuccessMessage('Expense added successfully!');
          this.fetchExpenses();
          this.resetForm();
        } else {
          this.showSuccessMessage('Failed to add expense: ' + (response.data.message || 'Unknown error'));
        }
      })
      .catch(error => {
        console.error("Full error:", error.response); // Add this
        this.showSuccessMessage('Failed to add expense: ' + 
          (error.response?.data?.message || error.message || 'Unknown error')); 
    });
  }
},

formatDate(dateString) {
      if (!dateString) return 'N/A';
      const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' };
      return new Date(dateString).toLocaleDateString('en-US', options);
    }, // FOR TABLE (NEWWWWWWWWWWWWWWWWW)

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
           this.showSuccessMessage('Expense deleted successfully!');
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
  background-color: #dff0d8;
  border: 1px solid #d6e9c6;
  opacity: 1;
  transition: opacity 0.5s ease-out;
  position: relative;
  z-index: 100;
}

.success-message.hide {
  opacity: 0;
  height: 0;
  padding: 0;
  margin: 0;
  border: none;
  overflow: hidden;
  transition: opacity 0.5s ease-out, height 0.5s 0.5s, padding 0.5s 0.5s, margin 0.5s 0.5s;
} /* NEWWWWWWWWWWWWWWWWWWWWWWWWWW*/ 

 .expense-form {
  margin-bottom: 30px;
}  /*NEWWWWWWWWWWWWWWWWWWWWW*/
 

.expenses-section {
  margin-top: 10px; /* Added top margin */
}

.expenses-section h3 {
  margin-top: 10px;
  margin-bottom: 25px; /* Increased from 15px */
  color: #333;
  font-size: 1.5rem; /* Larger font */
  padding-bottom: 10px;
  border-bottom: 2px solid #eee; /* Added separator */
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

.expenses-table {
  overflow-x: auto;
  margin: 30px 0; /* Added vertical spacing */
}  /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

table {
  width: 100%;
  border-collapse: separate; /* Changed from collapse */
  border-spacing: 0 10px; /* Space between rows */
  margin-bottom: 30px; /* Increased from 20px */
}  /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

th, td {
  padding: 6px 20px; /* Increased padding */
  text-align: center;
  border-bottom: 2px solid #ddd; /* Thicker border */
  color: #333;
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

th {
  background-color: #f8f9fa;
  font-weight: 600;
  font-size: 1rem; /* Larger header text */
  padding: 12px 20px; /* More padding for headers */
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

tr {
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05); /* Subtle shadow */
  margin-bottom: 15px; /* Space between rows */
}

tr:hover {
  background-color: #f5f5f5;
  transform: translateY(-2px); /* Lift effect */
  box-shadow: 0 4px 8px rgba(0,0,0,0.1); /* Stronger shadow on hover */
  transition: all 0.2s ease; /* Smooth transition */
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

.actions {
  display: flex;
  gap: 10px;
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

.edit-btn, .delete-btn {
  padding: 8px 15px;
  font-size: 0.9rem;
  border-radius: 4px;
  cursor: pointer;
  border: none;
  color: white;
  font-weight: 500;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

/* Edit Button Specific Styles */
.edit-btn {
  background-color: #2196F3;
}

/* Delete Button Specific Styles */
.delete-btn {
  background-color: #f44336;
}

/* Hover Effects */
.edit-btn:hover {
  background-color: #1976D2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(33, 150, 243, 0.3);
  
  /* Ripple Effect */
  &::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 5px;
    height: 5px;
    background: rgba(255, 255, 255, 0.5);
    opacity: 0;
    border-radius: 100%;
    transform: scale(1, 1) translate(-50%);
    transform-origin: 50% 50%;
  }
  
  &:hover::after {
    animation: ripple 0.6s ease-out;
  }
}

.delete-btn:hover {
  background-color: #d32f2f;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(244, 67, 54, 0.3);
  
  /* Pulse Effect */
  animation: pulse 0.5s ease-in-out;
}

/* Ripple Animation */
@keyframes ripple {
  0% {
    transform: scale(0, 0);
    opacity: 0.5;
  }
  100% {
    transform: scale(20, 20);
    opacity: 0;
  }
}

/* Pulse Animation */
@keyframes pulse {
  0% {
    transform: translateY(-2px) scale(1);
  }
  50% {
    transform: translateY(-2px) scale(1.05);
  }
  100% {
    transform: translateY(-2px) scale(1);
  }
}

/* Active State */
.edit-btn:active, .delete-btn:active {
  transform: translateY(0);
  box-shadow: 0 1px 3px rgba(0,0,0,0.2);
} /*NEWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW*/

.total {
     font-size: 20px;
     font-weight: bold;
     color: #333;
     padding: 20px;
     background-color: white;
     box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
     text-align: center;
     width: 1100px;
     position: relative; /* Keeps it visible at the bottom */
     bottom: 0; /* Stick to the bottom */
     
}/*NEWWWWWWWWWWWWWWWWWWWWW */

 
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
