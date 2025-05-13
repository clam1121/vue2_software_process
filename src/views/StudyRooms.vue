<template>
  <div class="study-rooms">
    <h1>自习室列表</h1>
    
    <!-- 搜索筛选区域 -->
    <div class="search-section">
      <div class="search-filters">
        <div class="filter-group">
          <label for="location">地点</label>
          <select id="location" v-model="filters.location">
            <option value="">全部地点</option>
            <option value="图书馆">图书馆</option>
            <option value="教学楼A">教学楼A</option>
            <option value="教学楼B">教学楼B</option>
            <option value="教学楼C">教学楼C</option>
          </select>
        </div>
        
        <div class="filter-group">
          <label for="time">开放时段</label>
          <select id="time" v-model="filters.timeSlot">
            <option value="">全天</option>
            <option value="morning">上午 (8:00-12:00)</option>
            <option value="afternoon">下午 (13:00-18:00)</option>
            <option value="evening">晚上 (19:00-22:00)</option>
          </select>
        </div>
        
        <div class="filter-group">
          <label for="availableSeats">剩余座位</label>
          <select id="availableSeats" v-model="filters.availableSeats">
            <option value="">不限</option>
            <option value="1">至少1个</option>
            <option value="5">至少5个</option>
            <option value="10">至少10个</option>
            <option value="20">至少20个</option>
          </select>
        </div>

        <button class="search-button" @click="searchRooms">查询</button>
        <button class="save-button" @click="saveSearch">收藏此查询条件</button>
      </div>
    </div>
    
    <!-- 自习室列表 -->
    <div class="room-list">
      <div v-for="room in filteredRooms" :key="room.id" class="room-card">
        <div class="room-info">
          <h3>{{ room.name }}</h3>
          <p class="room-location">{{ room.location }}</p>
          <p class="room-time">开放时间：{{ room.openTime }} - {{ room.closeTime }}</p>
          <div class="room-stats">
            <span class="seats-info">
              座位：{{ room.availableSeats }}/{{ room.totalSeats }}
              <span class="seat-percentage" :class="getSeatStatusClass(room)">
                ({{ calculatePercentage(room.availableSeats, room.totalSeats) }}%)
              </span>
            </span>
          </div>
        </div>
        <div class="room-actions">
          <button @click="viewRoomDetail(room.id)">查看详情</button>
          <button @click="toggleFavorite(room)" class="favorite-btn" :class="{ 'is-favorite': isFavorite(room.id) }">
            {{ isFavorite(room.id) ? '取消收藏' : '收藏' }}
          </button>
        </div>
      </div>
      
      <div v-if="filteredRooms.length === 0" class="no-results">
        没有找到符合条件的自习室，请尝试调整筛选条件
      </div>
    </div>
    
    <!-- 我的收藏 -->
    <div class="favorites-section" v-if="favorites.length > 0">
      <h2>我的收藏</h2>
      <div class="favorite-items">
        <div v-for="favorite in favorites" :key="favorite.id" class="favorite-item">
          <span>{{ favorite.name }}</span>
          <button @click="viewRoomDetail(favorite.id)">查看</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StudyRooms',
  data() {
    return {
      // 筛选条件
      filters: {
        location: '',
        timeSlot: '',
        availableSeats: ''
      },
      // 模拟自习室数据
      rooms: [
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
      ],
      // 收藏的自习室
      favorites: [],
      savedSearches: []
    }
  },
  computed: {
    filteredRooms() {
      let result = [...this.rooms];
      
      // 按地点筛选
      if (this.filters.location) {
        result = result.filter(room => room.location === this.filters.location);
      }
      
      // 按时间段筛选
      if (this.filters.timeSlot) {
        const timeRanges = {
          morning: { start: '08:00', end: '12:00' },
          afternoon: { start: '13:00', end: '18:00' },
          evening: { start: '19:00', end: '22:00' }
        };
        
        const selectedRange = timeRanges[this.filters.timeSlot];
        if (selectedRange) {
          result = result.filter(room => {
            return room.openTime <= selectedRange.start && room.closeTime >= selectedRange.end;
          });
        }
      }
      
      // 按可用座位数筛选
      if (this.filters.availableSeats) {
        const minAvailable = parseInt(this.filters.availableSeats);
        result = result.filter(room => room.availableSeats >= minAvailable);
      }
      
      return result;
    }
  },
  created() {
    // 从localStorage加载收藏
    const savedFavorites = localStorage.getItem('favorites');
    if (savedFavorites) {
      this.favorites = JSON.parse(savedFavorites);
    }
    
    // 加载保存的搜索条件
    const savedSearches = localStorage.getItem('savedSearches');
    if (savedSearches) {
      this.savedSearches = JSON.parse(savedSearches);
    }
  },
  methods: {
    searchRooms() {
      // 这个函数是为了未来可能从API动态加载数据时使用的
      // 目前使用的是计算属性过滤，所以这个函数暂时不执行实际操作
      console.log('搜索条件：', this.filters);
    },
    saveSearch() {
      // 保存当前的搜索条件
      const search = { ...this.filters, id: Date.now() };
      this.savedSearches.push(search);
      localStorage.setItem('savedSearches', JSON.stringify(this.savedSearches));
      alert('已保存搜索条件');
    },
    viewRoomDetail(roomId) {
      this.$router.push(`/study-room/${roomId}`);
    },
    toggleFavorite(room) {
      const index = this.favorites.findIndex(f => f.id === room.id);
      
      if (index > -1) {
        // 取消收藏
        this.favorites.splice(index, 1);
      } else {
        // 添加收藏
        this.favorites.push({
          id: room.id,
          name: room.name
        });
      }
      
      // 保存到localStorage
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
    },
    isFavorite(roomId) {
      return this.favorites.some(f => f.id === roomId);
    },
    calculatePercentage(available, total) {
      return Math.round((available / total) * 100);
    },
    getSeatStatusClass(room) {
      const percentage = this.calculatePercentage(room.availableSeats, room.totalSeats);
      
      if (percentage < 20) {
        return 'status-critical';
      } else if (percentage < 50) {
        return 'status-warning';
      } else {
        return 'status-good';
      }
    }
  }
}
</script>

<style scoped>
.study-rooms {
  max-width: 1000px;
  margin: 0 auto;
}

h1 {
  margin-bottom: 20px;
  text-align: center;
}

.search-section {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.search-filters {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  align-items: flex-end;
}

.filter-group {
  flex: 1;
  min-width: 150px;
}

.filter-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

.search-button, .save-button {
  padding: 10px 20px;
  height: 40px;
}

.save-button {
  background-color: #2c3e50;
}

.save-button:hover {
  background-color: #1a2530;
}

.room-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.room-card {
  background-color: #ffffff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.room-card:hover {
  transform: translateY(-4px);
}

.room-info {
  padding: 15px;
}

.room-location {
  color: #666;
  margin: 5px 0;
}

.room-time {
  color: #666;
  margin-bottom: 10px;
}

.room-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.seats-info {
  display: flex;
  align-items: center;
  gap: 5px;
}

.seat-percentage {
  font-weight: bold;
}

.status-good {
  color: #42b983;
}

.status-warning {
  color: #f39c12;
}

.status-critical {
  color: #e74c3c;
}

.room-actions {
  display: flex;
  border-top: 1px solid #eee;
}

.room-actions button {
  flex: 1;
  padding: 10px;
  border: none;
  background-color: #f8f9fa;
  cursor: pointer;
  transition: background-color 0.2s;
}

.room-actions button:hover {
  background-color: #e9ecef;
}

.favorite-btn {
  color: #666;
}

.favorite-btn.is-favorite {
  color: #e74c3c;
}

.no-results {
  grid-column: 1 / -1;
  text-align: center;
  padding: 40px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.favorites-section {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  margin-top: 30px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h2 {
  margin-bottom: 15px;
}

.favorite-items {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.favorite-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 15px;
  background-color: #f8f9fa;
  border-radius: 4px;
  gap: 10px;
}

.favorite-item button {
  padding: 5px 10px;
  font-size: 12px;
}
</style> 