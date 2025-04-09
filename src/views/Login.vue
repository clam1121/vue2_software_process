<template>
  <div class="login">
    <h2>登录</h2>
    <form @submit.prevent="login">
      <div class="form-group">
        <label for="studentId">学号：</label>
        <input type="text" id="studentId" v-model="form.studentId" required>
      </div>
      <div class="form-group">
        <label for="password">密码：</label>
        <input type="password" id="password" v-model="form.password" required>
      </div>
      <button type="submit">登录</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'LoginView',
  data() {
    return {
      form: {
        studentId: '',
        password: ''
      }
    }
  },
  methods: {
    async login() {
      try {
        const response = await axios.post('/api/login', this.form);
        console.log(response.data);
        alert('登录成功！');
        this.$router.push('/');
      } catch (error) {
        console.error('登录失败', error);
        alert('登录失败，请检查您的学号和密码。');
      }
    }
  }
}
</script>

<style scoped>
.login {
  max-width: 300px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
  text-align: left;
}

label {
  display: block;
  margin-bottom: 5px;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #3aa876;
}
</style>