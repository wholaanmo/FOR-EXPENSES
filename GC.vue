<template>
    <navigation/>
    <div class="group-management-container">
      <div class="card">
        <div class="card-header">
          <h1 class="title">Group Management</h1>
          <div class="tabs">
            <button 
              @click="activeTab = 'create'" 
              :class="{ 'tab-button': true, 'active': activeTab === 'create' }">
              <i class="fas fa-plus-circle"></i> Create Group
            </button>
            <button 
              @click="activeTab = 'join'" 
              :class="{ 'tab-button': true, 'active': activeTab === 'join' }">
              <i class="fas fa-user-plus"></i> Join Group
            </button>
          </div>
        </div>
  
        <div class="card-body">
          <transition name="fade" mode="out-in">
            <div v-if="activeTab === 'create'" class="create-group">
              <h2 class="section-title">Create New Group</h2>
              <form @submit.prevent="createGroup" class="group-form">
                <div class="form-group">
                  <label class="form-label">Group Name</label>
                  <input 
                    v-model="groupName" 
                    type="text" 
                    class="form-input"
                    placeholder="Enter your group name"
                    required 
                  />
                </div>
                <button type="submit" class="submit-button">
                  <i class="fas fa-users"></i> Create Group
                </button>
              </form>
  
              <div v-if="joinCode" class="success-box">
                <div class="success-icon">
                  <i class="fas fa-check-circle"></i>
                </div>
                <div class="success-content">
                  <h3>Group Created Successfully!</h3>
                  <p>Your group join code:</p>
                  <div class="code-display">{{ joinCode }}</div>
                  <p class="instruction">Share this code with others to join your group</p>
                </div>
              </div>
            </div>
  
            <div v-else class="join-group">
              <h2 class="section-title">Join Existing Group</h2>
              <form @submit.prevent="joinGroup" class="group-form">
                <div class="form-group">
                  <label class="form-label">Enter Join Code</label>
                  <input 
                    v-model="joinCodeInput" 
                    type="text" 
                    class="form-input"
                    placeholder="Enter the 6-digit code"
                    required 
                  />
                </div>
                <button type="submit" class="submit-button">
                  <i class="fas fa-sign-in-alt"></i> Join Group
                </button>
              </form>
            </div>
          </transition>
  
          <div v-if="error" class="error-message">
            <i class="fas fa-exclamation-circle"></i> {{ error }}
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import Navigation from "./navigation.vue"; 
  
  export default {
    components: { Navigation },
    data() {
      return {
        activeTab: 'create',
        groupName: '',
        joinCode: '',
        joinCodeInput: '',
        error: ''
      };
    },
    methods: {
      async createGroup() {
        try {
          const response = await this.$axios.post('/groups/create', {
            name: this.groupName
          });
          this.joinCode = response.data.joinCode;
          this.error = '';
          // Redirect to group page after creation
          this.$router.push('/group');
        } catch (err) {
          this.error = err.response?.data?.message || 'Failed to create group';
        }
      },
      async joinGroup() {
        try {
          await this.$axios.post('/groups/join', {
            joinCode: this.joinCodeInput
          });
          this.error = '';
          // Redirect to group page after joining
          this.$router.push('/group');
        } catch (err) {
          this.error = err.response?.data?.message || 'Failed to join group';
        }
      }
    },
    async beforeRouteEnter(to, from, next) {
      try {
        // Check if user already has groups
        const response = await this.$axios.get('/groups/my-groups');
        if (response.data.data.length > 0) {
          next('/group'); // Redirect to group page if already in a group
        } else {
          next(); // Show create/join page
        }
      } catch (err) {
        next(); // Show create/join page if error occurs
      }
    }
  };
  </script>
  
  <style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap');
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css');
  
  .group-management-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: #f6f8d5;;
    padding: 20px;
    font-family: 'Poppins', sans-serif;
  }
  
  .card {
    width: 100%;
    max-width: 600px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
  
  .card-header {
    padding: 30px;
    background: linear-gradient(135deg, #a8e6cf 0%, #81c784 100%);
    color: white;
    text-align: center;
  }
  
  .title {
    margin: 0 0 20px;
    font-size: 28px;
    font-weight: 600;
  }
  
  .tabs {
    display: flex;
    border-radius: 8px;
    overflow: hidden;
    background: rgba(255, 255, 255, 0.2);
  }
  
  .tab-button {
    flex: 1;
    padding: 12px;
    background: transparent;
    border: none;
    color: white;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }
  
  .tab-button.active {
    background: rgba(255, 255, 255, 0.3);
    font-weight: 600;
  }
  
  .tab-button:hover {
    background: rgba(255, 255, 255, 0.25);
  }
  
  .card-body {
    padding: 30px;
  }
  
  .section-title {
    margin-top: 0;
    margin-bottom: 25px;
    color: #2c3e50;
    font-size: 22px;
    font-weight: 600;
    text-align: center;
  }
  
  .group-form {
    margin-bottom: 25px;
  }
  
  .form-group {
    margin-bottom: 20px;
  }
  
  .form-label {
    display: block;
    margin-bottom: 8px;
    color: #2c3e50;
    font-size: 14px;
    font-weight: 500;
  }
  
  .form-input {
    width: 100%;
    padding: 12px 15px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 16px;
    transition: border-color 0.3s;
  }
  
  .form-input:focus {
    outline: none;
    border-color: #42b983;
    box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.2);
  }
  
  .submit-button {
    width: 100%;
    padding: 14px;
    background: linear-gradient(135deg, #a8e6cf 0%, #81c784 100%);
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }
  
  .submit-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(66, 185, 131, 0.3);
  }
  
  .success-box {
    display: flex;
    padding: 20px;
    background: #f0fff4;
    border-radius: 8px;
    border: 1px solid #c6f6d5;
    margin-top: 25px;
  }
  
  .success-icon {
    margin-right: 15px;
    color: #48bb78;
    font-size: 32px;
  }
  
  .success-content {
    flex: 1;
  }
  
  .success-content h3 {
    margin: 0 0 10px;
    color: #2f855a;
  }
  
  .code-display {
    display: inline-block;
    padding: 8px 15px;
    background: white;
    border: 1px dashed #48bb78;
    border-radius: 6px;
    font-size: 20px;
    font-weight: 600;
    letter-spacing: 2px;
    color: #2f855a;
    margin: 10px 0;
  }
  
  .instruction {
    color: #718096;
    font-size: 14px;
    margin: 5px 0 0;
  }
  
  .error-message {
    padding: 12px 15px;
    background: #fff5f5;
    border: 1px solid #fed7d7;
    border-radius: 8px;
    color: #e53e3e;
    font-size: 14px;
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s ease;
  }
  
  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }
  
  @media (max-width: 640px) {
    .card-header {
      padding: 20px;
    }
    
    .card-body {
      padding: 20px;
    }
    
    .title {
      font-size: 24px;
    }
    
    .tab-button {
      font-size: 14px;
      padding: 10px;
    }
  }
  </style>
