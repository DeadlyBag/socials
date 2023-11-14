<template>
    <div class="password-reset">
      <div class="background-box"></div>
      <div class="password-reset-card">
        <h2 class="login-title">Forgot Password</h2>
        <div class="card">
          <div class="card-header bg-primary text-white">
            <h4>Reset Password</h4>
          </div>
          <div class="card-body">
            <form @submit.prevent="resetPassword">
              <div class="form-group">
                <label for="email" class="form-label">Email</label>
                <input
                  id="email"
                  type="email"
                  placeholder="Email"
                  v-model="email"
                  class="form-control"
                >
              </div>
              <button type="submit" class="btn btn-primary">Submit</button>
              <router-link to="/login" class="card-link">Back to Login</router-link>
            </form>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
/*

FIX HIER, FINDET RESET PASSWORD API ENDPOINT NICHT 404 FEHLER IDK WHY


*/
  export default {
    data() {
      return {
        email: "",
      };
    },
    methods: {
      async resetPassword() {
        // Clear any previous error messages
        try {
          // Send a POST request to your server endpoint for password reset requests
          const response = await fetch("https://6791q0wp-8080.euw.devtunnels.ms/api/users/reset-password-request", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({ email: this.email }),
          });
  
          if (response.ok) {
            // Password reset request succeeded
            // You can display a success message or navigate to a confirmation page
            console.log("Password reset email sent");
          } 
        } catch (error) {
          // Handle network or other errors
          console.error("Error initiating password reset:", error);
        }
      },
    },
  };
  </script>
  
  
  <style scoped>
  .password-reset {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    position: relative;
  }
  
  .background-box {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: #f0f0f0;
    opacity: 0.5;
    z-index: -1;
  }
  
  .password-reset-card {
    width: 400px;
    margin: auto;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
  }
  
  .card {
    border-radius: 0;
  }
  
  .btn {
    border-radius: 25px;
    margin-top: 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    padding: 10px 20px;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  
  .btn:hover {
    background-color: #0056b3;
  }
  
  .form-control {
    border-radius: 25px;
    margin-bottom: 15px;
    padding: 10px;
  }
  
  .form-group {
    margin-bottom: 20px;
  }
  
  .form-label {
    margin-bottom: 5px;
  }
  
  .card-link {
    text-decoration: none;
    color: #007bff;
    margin-left: 10px;
  }
  
  .card-link:hover {
    text-decoration: underline;
  }
  
  .card-link:last-child {
    margin-top: 10px;
  }
  </style>
  