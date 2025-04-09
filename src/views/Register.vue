<template>
  <div class="register">
    <h2>注册</h2>
    <form @submit.prevent="register">
      <div class="form-group">
        <label for="studentId">学号：</label>
        <input type="text" id="studentId" v-model="form.studentId" required>
      </div>
      <div class="form-group">
        <label for="name">姓名：</label>
        <input type="text" id="name" v-model="form.name" required>
      </div>
      <div class="form-group">
        <label for="password">密码：</label>
        <input type="password" id="password" v-model="form.password" required>
      </div>
      <div class="form-group">
        <label for="email">邮箱：</label>
        <input type="email" id="email" v-model="form.email" required>
      </div>
      <div class="form-group">
        <label for="department">院系：</label>
        <input type="text" id="department" v-model="form.department" required>
      </div>
      <div class="form-group">
        <label for="major">专业：</label>
        <input type="text" id="major" v-model="form.major" required>
      </div>
      <button type="submit">注册</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'RegisterView',
  data() {
    return {
      form: {
        studentId: '',
        name: '',
        password: '',
        email: '',
        department: '',
        major: ''
      }
    }
  },
  methods: {
    async register() {
      try {
        const response = await axios.post('/api/register', this.form);
        console.log(response.data);
        alert('注册成功！');
        this.$router.push('/login');
      } catch (error) {
        console.error('注册失败', error);
        alert('注册失败，请重试。');
      }
    }
  }
}
</script>

<style scoped>
.register {
  max-width: 400px;
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