<template>
  <div class="profile">
    <h1>个人中心</h1>
    
    <div v-if="loading" class="loading">
      <p>加载中...</p>
    </div>
    
    <div v-else class="profile-container">
      <!-- 基本信息 -->
      <div class="profile-section">
        <h2>基本信息</h2>
        <div class="section-content">
          <div class="form-group">
            <label for="studentId">学号</label>
            <input type="text" id="studentId" v-model="userInfo.studentId" disabled>
          </div>
          <div class="form-group">
            <label for="name">姓名</label>
            <input type="text" id="name" v-model="userInfo.name" :disabled="!isEditing">
          </div>
          <div class="form-group">
            <label for="email">邮箱</label>
            <input type="email" id="email" v-model="userInfo.email" :disabled="!isEditing">
          </div>
          <div class="form-group">
            <label for="phone">手机号</label>
            <input type="text" id="phone" v-model="userInfo.phone" :disabled="!isEditing">
          </div>
          <div class="form-group">
            <label for="department">院系</label>
            <input type="text" id="department" v-model="userInfo.department" disabled>
          </div>
          <div class="form-group">
            <label for="major">专业</label>
            <input type="text" id="major" v-model="userInfo.major" disabled>
          </div>
          <div class="form-actions">
            <button v-if="!isEditing" class="edit-btn" @click="startEditing">修改信息</button>
            <template v-else>
              <button class="save-btn" @click="saveChanges">保存</button>
              <button class="cancel-btn" @click="cancelEditing">取消</button>
            </template>
          </div>
        </div>
      </div>
      
      <!-- 安全设置 -->
      <div class="profile-section">
        <h2>安全设置</h2>
        <div class="section-content">
          <div class="security-item">
            <div class="security-info">
              <strong>登录密码</strong>
              <p>用于登录账号</p>
            </div>
            <button class="action-btn" @click="showPasswordForm = !showPasswordForm">
              {{ showPasswordForm ? '取消' : '修改' }}
            </button>
          </div>
          
          <div v-if="showPasswordForm" class="password-form">
            <div class="form-group">
              <label for="currentPassword">当前密码</label>
              <input type="password" id="currentPassword" v-model="passwordForm.currentPassword">
            </div>
            <div class="form-group">
              <label for="newPassword">新密码</label>
              <input type="password" id="newPassword" v-model="passwordForm.newPassword">
            </div>
            <div class="form-group">
              <label for="confirmPassword">确认新密码</label>
              <input type="password" id="confirmPassword" v-model="passwordForm.confirmPassword">
              <div class="error-msg" v-if="passwordError">{{ passwordError }}</div>
            </div>
            <div class="form-actions">
              <button class="save-btn" @click="changePassword">更新密码</button>
            </div>
          </div>
        </div>
      </div>
      
      <!-- 通知设置 -->
      <div class="profile-section">
        <h2>通知设置</h2>
        <div class="section-content">
          <div class="notification-settings">
            <div class="notification-item">
              <div class="notification-info">
                <strong>预约提醒</strong>
                <p>预约时间前15分钟发送提醒</p>
              </div>
              <div class="toggle-switch">
                <input type="checkbox" id="reservationReminder" v-model="settings.reservationReminder">
                <label for="reservationReminder"></label>
              </div>
            </div>
            
            <div class="notification-item">
              <div class="notification-info">
                <strong>即将签到提醒</strong>
                <p>签到即将过期时发送提醒</p>
              </div>
              <div class="toggle-switch">
                <input type="checkbox" id="checkinReminder" v-model="settings.checkinReminder">
                <label for="checkinReminder"></label>
              </div>
            </div>
            
            <div class="notification-item">
              <div class="notification-info">
                <strong>信用变更通知</strong>
                <p>信用分发生变化时通知</p>
              </div>
              <div class="toggle-switch">
                <input type="checkbox" id="creditNotification" v-model="settings.creditNotification">
                <label for="creditNotification"></label>
              </div>
            </div>
          </div>
          
          <div class="form-actions">
            <button class="save-btn" @click="saveSettings">保存设置</button>
          </div>
        </div>
      </div>
      
      <!-- 账户操作 -->
      <div class="profile-section">
        <h2>账户操作</h2>
        <div class="section-content">
          <div class="account-actions">
            <button class="danger-btn" @click="showLogoutConfirm = true">退出登录</button>
            <button class="danger-btn" @click="showDeleteConfirm = true">申请注销账户</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 退出登录确认弹窗 -->
    <div class="modal" v-if="showLogoutConfirm">
      <div class="modal-content">
        <h3>确认退出登录</h3>
        <p>确定要退出当前账号吗？</p>
        <div class="modal-actions">
          <button class="cancel-btn" @click="showLogoutConfirm = false">取消</button>
          <button class="confirm-btn" @click="logout">确认退出</button>
        </div>
      </div>
    </div>
    
    <!-- 注销账户确认弹窗 -->
    <div class="modal" v-if="showDeleteConfirm">
      <div class="modal-content">
        <h3>确认注销账户</h3>
        <p>注销账户将清除您的所有数据，且无法恢复。请谨慎操作！</p>
        <div class="modal-actions">
          <button class="cancel-btn" @click="showDeleteConfirm = false">取消</button>
          <button class="danger-btn" @click="deleteAccount">确认注销</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      loading: true,
      isEditing: false,
      userInfo: {
        studentId: '',
        name: '',
        email: '',
        phone: '',
        department: '',
        major: ''
      },
      userInfoBackup: null,
      showPasswordForm: false,
      passwordForm: {
        currentPassword: '',
        newPassword: '',
        confirmPassword: ''
      },
      passwordError: '',
      settings: {
        reservationReminder: true,
        checkinReminder: true,
        creditNotification: true
      },
      showLogoutConfirm: false,
      showDeleteConfirm: false
    }
  },
  created() {
    this.loadUserInfo();
    this.loadSettings();
  },
  methods: {
    loadUserInfo() {
      // 模拟从API获取用户信息
      setTimeout(() => {
        const userInfoData = localStorage.getItem('userInfo');
        if (userInfoData) {
          const userData = JSON.parse(userInfoData);
          this.userInfo = {
            studentId: userData.studentId || '未设置',
            name: userData.name || '未设置',
            email: userData.email || '未设置',
            phone: userData.phone || '未设置',
            department: userData.department || '未设置',
            major: userData.major || '未设置'
          };
        }
        this.loading = false;
      }, 500);
    },
    loadSettings() {
      // 从localStorage加载设置
      const settingsData = localStorage.getItem('userSettings');
      if (settingsData) {
        this.settings = JSON.parse(settingsData);
      }
    },
    startEditing() {
      // 备份当前信息，以便取消时恢复
      this.userInfoBackup = { ...this.userInfo };
      this.isEditing = true;
    },
    cancelEditing() {
      // 恢复备份信息
      if (this.userInfoBackup) {
        this.userInfo = { ...this.userInfoBackup };
      }
      this.isEditing = false;
    },
    saveChanges() {
      // 模拟保存到API
      setTimeout(() => {
        // 更新localStorage中的用户信息
        const userInfoData = localStorage.getItem('userInfo');
        if (userInfoData) {
          const userData = JSON.parse(userInfoData);
          const updatedUserData = {
            ...userData,
            name: this.userInfo.name,
            email: this.userInfo.email,
            phone: this.userInfo.phone
          };
          localStorage.setItem('userInfo', JSON.stringify(updatedUserData));
        }
        
        // 更新备份
        this.userInfoBackup = { ...this.userInfo };
        this.isEditing = false;
        
        alert('个人信息已更新');
      }, 500);
    },
    changePassword() {
      // 验证密码
      if (!this.passwordForm.currentPassword) {
        this.passwordError = '请输入当前密码';
        return;
      }
      
      if (!this.passwordForm.newPassword) {
        this.passwordError = '请输入新密码';
        return;
      }
      
      if (this.passwordForm.newPassword.length < 6) {
        this.passwordError = '新密码长度不能小于6位';
        return;
      }
      
      if (this.passwordForm.newPassword !== this.passwordForm.confirmPassword) {
        this.passwordError = '两次输入的密码不一致';
        return;
      }
      
      // 模拟密码修改
      setTimeout(() => {
        // 实际应用中需要调用API验证当前密码并修改新密码
        
        // 重置表单和错误信息
        this.passwordForm = {
          currentPassword: '',
          newPassword: '',
          confirmPassword: ''
        };
        this.passwordError = '';
        this.showPasswordForm = false;
        
        alert('密码修改成功');
      }, 500);
    },
    saveSettings() {
      // 保存设置到localStorage
      localStorage.setItem('userSettings', JSON.stringify(this.settings));
      alert('设置已保存');
    },
    logout() {
      // 清除用户登录信息
      localStorage.removeItem('userInfo');
      
      // 重定向到登录页面
      this.$router.push('/login');
    },
    deleteAccount() {
      // 模拟账户注销
      setTimeout(() => {
        // 清除所有与用户相关的数据
        localStorage.removeItem('userInfo');
        localStorage.removeItem('userSettings');
        localStorage.removeItem('reservations');
        localStorage.removeItem('favorites');
        localStorage.removeItem('savedSearches');
        
        alert('账户已注销');
        this.$router.push('/login');
      }, 500);
    }
  }
}
</script>

<style scoped>
.profile {
  max-width: 800px;
  margin: 0 auto;
}

h1, h2 {
  margin-bottom: 20px;
}

h1 {
  text-align: center;
}

.loading {
  text-align: center;
  padding: 40px 0;
}

.profile-container {
  margin-bottom: 40px;
}

.profile-section {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 30px;
  overflow: hidden;
}

.profile-section h2 {
  padding: 15px 20px;
  margin: 0;
  background-color: #f8f9fa;
  border-bottom: 1px solid #eee;
}

.section-content {
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

input[type="text"],
input[type="email"],
input[type="password"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

input:disabled {
  background-color: #f8f9fa;
  cursor: not-allowed;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.edit-btn, .save-btn, .cancel-btn, .action-btn, .danger-btn, .confirm-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
}

.edit-btn, .action-btn {
  background-color: #f8f9fa;
  color: #2c3e50;
}

.save-btn, .confirm-btn {
  background-color: #42b983;
  color: white;
}

.cancel-btn {
  background-color: #95a5a6;
  color: white;
}

.danger-btn {
  background-color: #e74c3c;
  color: white;
}

.error-msg {
  color: #e74c3c;
  font-size: 12px;
  margin-top: 5px;
}

/* 安全设置样式 */
.security-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-bottom: 1px solid #eee;
}

.security-item:last-child {
  border-bottom: none;
}

.security-info p {
  margin: 5px 0 0 0;
  color: #666;
  font-size: 14px;
}

.password-form {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

/* 通知设置样式 */
.notification-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-bottom: 1px solid #eee;
}

.notification-item:last-child {
  border-bottom: none;
}

.notification-info p {
  margin: 5px 0 0 0;
  color: #666;
  font-size: 14px;
}

/* 开关样式 */
.toggle-switch {
  position: relative;
  width: 60px;
  height: 30px;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-switch label {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  border-radius: 15px;
  cursor: pointer;
  transition: 0.4s;
}

.toggle-switch label:before {
  position: absolute;
  content: "";
  height: 22px;
  width: 22px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  border-radius: 50%;
  transition: 0.4s;
}

.toggle-switch input:checked + label {
  background-color: #42b983;
}

.toggle-switch input:checked + label:before {
  transform: translateX(30px);
}

/* 账户操作样式 */
.account-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

/* 弹窗样式 */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  width: 90%;
  max-width: 500px;
}

.modal-content h3 {
  margin-top: 0;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

@media (max-width: 768px) {
  .security-item, .notification-item {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .toggle-switch, .action-btn {
    margin-top: 10px;
  }
}
</style> 