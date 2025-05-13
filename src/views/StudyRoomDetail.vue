<template>
  <div class="study-room-detail">
    <div v-if="loading" class="loading">
      <p>加载中...</p>
    </div>
    
    <div v-else-if="!room" class="error">
      <p>无法找到该自习室信息</p>
      <button @click="$router.push('/study-rooms')">返回自习室列表</button>
    </div>
    
    <div v-else class="room-container">
      <div class="room-header">
        <button class="back-btn" @click="$router.push('/study-rooms')">
          返回列表
        </button>
        <h1>{{ room.name }}</h1>
        <div class="room-meta">
          <p><strong>位置：</strong>{{ room.location }}</p>
          <p><strong>开放时间：</strong>{{ room.openTime }} - {{ room.closeTime }}</p>
          <p><strong>座位数：</strong>{{ room.availableSeats }}/{{ room.totalSeats }}</p>
        </div>
      </div>
      
      <!-- 预约时间选择 -->
      <div class="reservation-controls">
        <div class="date-selector">
          <h3>选择日期</h3>
          <div class="date-options">
            <button 
              v-for="(date, index) in availableDates" 
              :key="index"
              :class="{ active: selectedDate === date.value }"
              @click="selectDate(date.value)">
              {{ date.label }}
            </button>
          </div>
        </div>
        
        <div class="time-selector">
          <h3>选择时间段</h3>
          <div class="time-slots">
            <button 
              v-for="(slot, index) in timeSlots" 
              :key="index"
              :class="{ active: selectedTimeSlot === slot.value, disabled: !slot.available }"
              :disabled="!slot.available"
              @click="selectTimeSlot(slot.value)">
              {{ slot.label }}
            </button>
          </div>
        </div>
      </div>
      
      <!-- 座位布局 -->
      <div class="seat-layout">
        <h3>座位布局</h3>
        <div class="layout-container">
          <div class="legend">
            <div class="legend-item">
              <div class="seat-demo available"></div>
              <span>空闲</span>
            </div>
            <div class="legend-item">
              <div class="seat-demo occupied"></div>
              <span>已占用</span>
            </div>
            <div class="legend-item">
              <div class="seat-demo selected"></div>
              <span>已选择</span>
            </div>
            <div class="legend-item">
              <div class="seat-demo maintenance"></div>
              <span>维护中</span>
            </div>
          </div>
          
          <div class="room-layout">
            <div class="room-entrance">门</div>
            
            <div class="seats-grid">
              <div 
                v-for="seat in seats" 
                :key="seat.id" 
                class="seat" 
                :class="getSeatClass(seat)"
                @click="selectSeat(seat)">
                {{ seat.seatNumber }}
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- 预约表单 -->
      <div class="reservation-form" v-if="selectedSeat">
        <h3>预约信息</h3>
        <div class="selected-info">
          <p><strong>自习室：</strong>{{ room.name }}</p>
          <p><strong>座位号：</strong>{{ selectedSeat.seatNumber }}</p>
          <p><strong>日期：</strong>{{ getFormattedDate(selectedDate) }}</p>
          <p><strong>时间段：</strong>{{ getTimeSlotLabel(selectedTimeSlot) }}</p>
        </div>
        
        <div class="reservation-actions">
          <button class="reserve-btn" @click="makeReservation">确认预约</button>
          <button class="cancel-btn" @click="cancelSelection">取消选择</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StudyRoomDetail',
  data() {
    return {
      loading: true,
      roomId: null,
      room: null,
      seats: [],
      selectedSeat: null,
      selectedDate: 'today',
      selectedTimeSlot: 'morning',
      availableDates: [
        { label: '今天', value: 'today' },
        { label: '明天', value: 'tomorrow' },
        { label: '后天', value: 'dayAfterTomorrow' }
      ],
      timeSlots: [
        { label: '上午 (8:00-12:00)', value: 'morning', available: true },
        { label: '下午 (13:00-17:00)', value: 'afternoon', available: true },
        { label: '晚上 (18:00-22:00)', value: 'evening', available: true }
      ]
    }
  },
  created() {
    this.roomId = parseInt(this.$route.params.id);
    this.fetchRoomData();
  },
  methods: {
    fetchRoomData() {
      // 模拟从API获取数据
      setTimeout(() => {
        // 模拟自习室数据
        const mockRooms = [
          {
            id: 1,
            name: '图书馆一楼自习室',
            location: '图书馆',
            openTime: '08:00',
            closeTime: '22:00',
            totalSeats: 100,
            availableSeats: 45
          },
          {
            id: 2,
            name: '图书馆二楼自习室',
            location: '图书馆',
            openTime: '08:00',
            closeTime: '22:00',
            totalSeats: 80,
            availableSeats: 20
          },
          {
            id: 3,
            name: '教学楼A自习室A101',
            location: '教学楼A',
            openTime: '08:00',
            closeTime: '18:00',
            totalSeats: 50,
            availableSeats: 15
          },
          {
            id: 4,
            name: '教学楼B自习室B201',
            location: '教学楼B',
            openTime: '08:00',
            closeTime: '22:00',
            totalSeats: 60,
            availableSeats: 30
          },
          {
            id: 5,
            name: '教学楼C自习室C301',
            location: '教学楼C',
            openTime: '13:00',
            closeTime: '22:00',
            totalSeats: 40,
            availableSeats: 5
          }
        ];
        
        this.room = mockRooms.find(room => room.id === this.roomId);
        
        if (this.room) {
          // 生成座位数据
          this.generateSeats();
        }
        
        this.loading = false;
      }, 800); // 模拟加载延迟
    },
    
    generateSeats() {
      // 生成模拟座位数据
      const rows = Math.ceil(Math.sqrt(this.room.totalSeats));
      const cols = Math.ceil(this.room.totalSeats / rows);
      
      this.seats = [];
      let seatCounter = 1;
      
      for (let i = 0; i < rows; i++) {
        for (let j = 0; j < cols; j++) {
          if (seatCounter <= this.room.totalSeats) {
            // 随机生成座位状态，确保可用座位数与自习室数据一致
            let isMaintenance = Math.random() < 0.05; // 5%的概率是维护中
            let isOccupied = false;
            
            if (!isMaintenance) {
              // 剩余座位数低于总座位数的座位标记为已占用
              isOccupied = seatCounter > this.room.availableSeats;
            }
            
            this.seats.push({
              id: seatCounter,
              seatNumber: seatCounter.toString().padStart(2, '0'),
              row: i + 1,
              col: j + 1,
              status: isMaintenance ? 'maintenance' : (isOccupied ? 'occupied' : 'available')
            });
            
            seatCounter++;
          }
        }
      }
      
      // 随机打乱座位顺序，模拟现实中的不规则排列
      this.seats.sort(() => Math.random() - 0.5);
    },
    
    selectDate(date) {
      this.selectedDate = date;
      // 在实际应用中，这里可能需要重新获取该日期的座位可用情况
    },
    
    selectTimeSlot(timeSlot) {
      this.selectedTimeSlot = timeSlot;
      // 在实际应用中，这里可能需要重新获取该时间段的座位可用情况
    },
    
    selectSeat(seat) {
      if (seat.status !== 'available') {
        return; // 如果座位不可用，不允许选择
      }
      
      if (this.selectedSeat && this.selectedSeat.id === seat.id) {
        this.selectedSeat = null; // 取消选择
      } else {
        this.selectedSeat = seat; // 选择新座位
      }
    },
    
    getSeatClass(seat) {
      if (this.selectedSeat && this.selectedSeat.id === seat.id) {
        return 'selected';
      }
      return seat.status;
    },
    
    getFormattedDate(dateValue) {
      const today = new Date();
      let date = new Date(today);
      
      switch (dateValue) {
        case 'today':
          break;
        case 'tomorrow':
          date.setDate(today.getDate() + 1);
          break;
        case 'dayAfterTomorrow':
          date.setDate(today.getDate() + 2);
          break;
      }
      
      return `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')}`;
    },
    
    getTimeSlotLabel(timeSlotValue) {
      const slot = this.timeSlots.find(slot => slot.value === timeSlotValue);
      return slot ? slot.label : '';
    },
    
    makeReservation() {
      if (!this.selectedSeat || !this.selectedDate || !this.selectedTimeSlot) {
        alert('请选择座位和时间');
        return;
      }
      
      // 模拟预约请求
      const reservation = {
        id: Date.now(),
        roomId: this.room.id,
        roomName: this.room.name,
        seatId: this.selectedSeat.id,
        seatNumber: this.selectedSeat.seatNumber,
        date: this.getFormattedDate(this.selectedDate),
        timeSlot: this.selectedTimeSlot,
        timeSlotLabel: this.getTimeSlotLabel(this.selectedTimeSlot),
        status: 'pending',
        createdAt: new Date().toISOString()
      };
      
      // 获取已有的预约
      let reservations = JSON.parse(localStorage.getItem('reservations') || '[]');
      
      // 检查是否有冲突的预约
      const hasConflict = reservations.some(r => 
        r.date === reservation.date && 
        r.timeSlot === reservation.timeSlot
      );
      
      if (hasConflict) {
        alert('您已经在该时间段预约了其他座位，一个时间段只能预约一个座位');
        return;
      }
      
      // 添加新预约
      reservations.push(reservation);
      
      // 保存到本地存储
      localStorage.setItem('reservations', JSON.stringify(reservations));
      
      // 更新座位状态
      this.selectedSeat.status = 'occupied';
      this.selectedSeat = null;
      
      alert('预约成功！');
      this.$router.push('/reservations');
    },
    
    cancelSelection() {
      this.selectedSeat = null;
    }
  }
}
</script>

<style scoped>
.study-room-detail {
  max-width: 1000px;
  margin: 0 auto;
}

.loading, .error {
  text-align: center;
  padding: 50px 0;
}

.room-container {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 30px;
}

.room-header {
  margin-bottom: 30px;
  position: relative;
}

.back-btn {
  position: absolute;
  left: 0;
  top: 0;
  padding: 8px 15px;
  background-color: #f8f9fa;
  color: #333;
}

.room-header h1 {
  text-align: center;
  margin-bottom: 15px;
}

.room-meta {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
}

.room-meta p {
  margin: 5px 10px;
}

.reservation-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 30px;
}

.date-selector, .time-selector {
  flex: 1;
  min-width: 250px;
}

.date-options, .time-slots {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.date-options button, .time-slots button {
  flex: 1;
  min-width: 80px;
  padding: 10px;
  background-color: #f8f9fa;
  border: 1px solid #ddd;
  border-radius: 4px;
  color: #333;
}

button.active {
  background-color: #42b983;
  color: white;
  border-color: #42b983;
}

button.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.seat-layout {
  margin-bottom: 30px;
}

.layout-container {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
}

.legend {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  margin: 0 15px;
}

.seat-demo {
  width: 20px;
  height: 20px;
  margin-right: 8px;
  border-radius: 4px;
}

.seat-demo.available {
  background-color: #42b983;
}

.seat-demo.occupied {
  background-color: #e74c3c;
}

.seat-demo.selected {
  background-color: #3498db;
}

.seat-demo.maintenance {
  background-color: #95a5a6;
}

.room-layout {
  position: relative;
  padding: 30px;
}

.room-entrance {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  background-color: #ffffff;
  padding: 5px 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.seats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(40px, 1fr));
  gap: 15px;
  justify-content: center;
}

.seat {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  font-weight: bold;
  color: white;
  transition: transform 0.2s;
}

.seat:hover {
  transform: scale(1.1);
}

.seat.available {
  background-color: #42b983;
}

.seat.occupied {
  background-color: #e74c3c;
  cursor: not-allowed;
}

.seat.selected {
  background-color: #3498db;
}

.seat.maintenance {
  background-color: #95a5a6;
  cursor: not-allowed;
}

.reservation-form {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
}

.selected-info {
  margin-bottom: 20px;
}

.selected-info p {
  margin: 10px 0;
}

.reservation-actions {
  display: flex;
  gap: 15px;
}

.reserve-btn, .cancel-btn {
  flex: 1;
  padding: 12px;
}

.reserve-btn {
  background-color: #42b983;
}

.cancel-btn {
  background-color: #e74c3c;
}

.reserve-btn:hover {
  background-color: #3aa876;
}

.cancel-btn:hover {
  background-color: #c0392b;
}

@media (max-width: 768px) {
  .room-meta, .reservation-controls {
    flex-direction: column;
  }
  
  .date-selector, .time-selector {
    width: 100%;
  }
  
  .seat {
    width: 35px;
    height: 35px;
    font-size: 10px;
  }
  
  .seats-grid {
    gap: 10px;
  }
}
</style> 