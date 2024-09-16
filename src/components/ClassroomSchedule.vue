<template>
  <div class="container mx-auto p-4">
    <h1 class="text-5xl font-bold mb-6 justify-center">教室排課表</h1>

    <div class="flex flex-col md:flex-row">
      <div class="w-full md:w-1/6 mb-4 md:mb-0">
        <!-- Moved club filter to the top left -->
        <div class="club-filter-container mb-4">
          <ClubFilter :clubs="uniqueClubs" :modelValue="selectedClub" @update:modelValue="updateSelectedClub" />
        </div>

        <div class="flex md:flex-col space-x-2 md:space-x-0 md:space-y-2">
          <button @click="currentView = 'schedule'" :class="['w-full p-3 border rounded transition-colors duration-200 club-button',
            currentView === 'schedule' ? 'bg-blue-500 text-white' : 'bg-gray-200 hover:bg-gray-300']">
            排課區
          </button>
          <button @click="currentView = 'courses'" :class="['w-full p-3 border rounded transition-colors duration-200 club-button',
            currentView === 'courses' ? 'bg-blue-500 text-white' : 'bg-gray-200 hover:bg-gray-300']">
            課程區
          </button>
        </div>
      </div>

      <div class="w-full md:w-5/6 md:pl-6">
        <div class="flex justify-between mb-4">
          <div class="w-1/2">
            <input v-model="filterText" placeholder="搜尋已排課課程"
              class="p-3 border rounded w-full focus:outline-none focus:ring-2 focus:ring-blue-500" />
          </div>
          <div>
            <button v-if="currentView === 'schedule'" @click="showAddScheduleModal = true"
              class="p-3 border rounded bg-green-500 text-white hover:bg-green-600 transition-colors duration-200 club-button">
              新增排課
            </button>
            <button v-if="currentView === 'courses'" @click="showAddCourseModal = true"
              class="p-3 border rounded bg-green-500 text-white hover:bg-green-600 transition-colors duration-200 club-button">
              新增課程
            </button>
          </div>
        </div>

        <div class="overflow-x-auto bg-white shadow-md rounded-lg">
          <!-- 排課區視圖 -->
          <div class="overflow-x-auto shadow-lg rounded-lg" v-if="currentView === 'schedule'">
            <table class="min-w-full bg-white">
              <thead class="bg-gray-100">
                <tr>
                  <th
                    class="sticky left-0 z-10 bg-gray-100 py-3 px-4 border-b text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
                    教室名稱
                  </th>
                  <th v-for="day in days" :key="day"
                    class="py-3 px-4 border-b text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
                    {{ day }}
                  </th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200">
                <tr v-for="classroom in uniqueClassrooms" :key="classroom" class="hover:bg-gray-50">
                  <td class="sticky left-0 z-10 bg-white py-4 px-4 border-b font-medium text-gray-900">
                    {{ classroom }}
                  </td>
                  <td v-for="day in days" :key="day" class="py-4 px-4 border-b">
                    <draggable 
                      :list="getFilteredClassesForDay(classroom, day)" 
                      :group="{ name: 'courses', pull: 'clone', put: true }"
                      item-key="id"
                      @change="handleDragChange($event, classroom, day)"
                    >
                      <template #item="{ element }">
                        <div 
                          class="mb-2 p-2 rounded-lg shadow-sm relative group transition-all duration-300 ease-in-out hover:shadow-md"
                          :style="{ backgroundColor: element.color + '85' }"
                        >
                          <div class="course-name" >{{ element.courseName }}</div>
                          <div class="course-time">{{ element.startTime }} - {{ element.endTime }}</div>
                          <button @click="confirmDelete(element.id, element.courseName)"
                            class="absolute top-1 right-1 bg-red-500 text-white rounded-full w-6 h-6 flex items-center justify-center text-xs opacity-0 group-hover:opacity-100 transition-opacity duration-200 hover:bg-red-600">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24"
                              stroke="currentColor">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M6 18L18 6M6 6l12 12" />
                            </svg>
                          </button>
                        </div>
                      </template>
                    </draggable>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- 課程區視圖 -->
          <table class="min-w-full" v-else>
            <thead>
              <tr>
                <th class="py-2 px-4 border-b text-center">社團名稱</th>
                <th class="py-2 px-4 border-b text-center">課程名稱</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="course in filteredLessons" :key="course.id">
                <td class="py-2 px-4 border-b text-center truncate">{{ course.clubName }}</td>
                <td class="py-2 px-4 border-b text-center truncate">{{ course.name }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- 新增排課模態框 -->
    <div v-if="showAddScheduleModal" class="fixed inset-0 flex items-center justify-center bg-gray-500 bg-opacity-50">
      <div class="bg-white p-6 rounded shadow-lg max-w-lg w-full">
        <h2 class="text-2xl font-bold mb-6 text-center">新增排課</h2>
        <form @submit.prevent="checkSchedule" novalidate class="space-y-4">
          <!-- 統一樣式容器 -->
          <div>
            <label class="block text-sm font-medium mb-2" for="courseSelect">選擇課程</label>
            <SearchableDropdown :courses="courses" @select="handleCourseSelect" class="w-full" />
          </div>
          <div>
            <label class="block text-sm font-medium mb-2" for="classrooms">選擇教室</label>
            <select v-model="selectClassroom" id="classrooms"
              class="w-full p-3 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500" required>
              <option value="" disabled>請選擇</option>
              <option v-for="classroom in uniqueClassrooms" :key="classroom" :value="classroom">
                {{ classroom }}
              </option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium mb-2" for="dayOfWeek">選擇星期</label>
            <select v-model="selectedDay" id="dayOfWeek"
              class="w-full p-3 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500" required>
              <option value="" disabled>請選擇</option>
              <option v-for="day in days" :key="day" :value="day">{{ day }}</option>
            </select>
          </div>
          <div class="flex space-x-4">
            <div class="w-1/2">
              <label class="block text-sm font-medium mb-2">課程開始時間</label>
              <TimePicker v-model="selectedStartTime" format="HH:mm" />
            </div>
            <div class="w-1/2">
              <label class="block text-sm font-medium mb-2">課程結束時間</label>
              <TimePicker v-model="selectedEndTime" format="HH:mm" />
            </div>
          </div>

          <!-- 操作按鈕 -->
          <div class="flex justify-end space-x-2 mt-6">
            <button type="button" @click="showAddScheduleModal = false"
              class="p-3 border rounded bg-gray-200 hover:bg-gray-300 transition-colors duration-200">
              取消
            </button>
            <button type="submit"
              class="p-3 border rounded bg-blue-500 text-white hover:bg-blue-600 transition-colors duration-200">
              確定
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- 新增課程模態框 -->
    <div v-if="showAddCourseModal" class="fixed inset-0 flex items-center justify-center bg-gray-500 bg-opacity-50">
      <div class="bg-white p-4 rounded shadow-lg">
        <h2 class="text-xl font-bold mb-4">新增課程</h2>
        <form @submit.prevent="addCourse">
          <div class="mb-4">
            <label class="block mb-2" for="courseName">教室名稱</label>
            <input v-model="newCourse" id="courseName" type="text" class="p-2 border rounded w-full" />
          </div>
          <div class="mb-4">
            <label class="block mb-2" for="courseName">課程名稱</label>
            <input v-model="newCourse" id="courseName" type="text" class="p-2 border rounded w-full" />
          </div>
          <div class="mb-4">
            <label class="block mb-2" for="clubId">選擇社團</label>
            <select v-model="selectedClub" id="clubId" class="p-2 border rounded w-full">
              <option value="" disabled selected>請選擇社團</option>
              <option v-for="club in uniqueClubs" :key="club" :value="club">{{ club }}</option>
            </select>
          </div>
          <div class="flex justify-end space-x-2">
            <button type="button" @click="showAddCourseModal = false" class="p-2 border rounded bg-gray-200">
              取消
            </button>
            <button type="submit" class="p-2 border rounded bg-blue-500 text-white">
              確定
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- 刪除確認對話框 -->
    <div v-if="showDeleteConfirmDialog"
      class="fixed inset-0 flex items-center justify-center bg-gray-500 bg-opacity-50">
      <div class="bg-white p-4 rounded shadow-lg">
        <h2 class="text-xl font-bold mb-4">確認刪除</h2>
        <p>確定要刪除課程 "{{ courseToDelete.name }}" 嗎？</p>
        <div class="flex justify-end space-x-2 mt-4">
          <button @click="showDeleteConfirmDialog = false" class="p-2 border rounded bg-gray-200">
            取消
          </button>
          <button @click="deleteCourse" class="p-2 border rounded bg-red-500 text-white">
            確定刪除
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed, watch} from 'vue';
import SearchableDropdown from './SearchableDropdown.vue';
import ClubFilter from './ClubFilter.vue';
import TimePicker from 'vue3-timepicker';
import draggable from 'vuedraggable'


import 'vue3-timepicker/dist/VueTimepicker.css'

export default {
  name: 'ClassroomSchedule',
  components: {
    SearchableDropdown,
    TimePicker,
    ClubFilter,
    draggable
  },
  setup() {
    const scheduleData = ref([]);
    const courseData = ref([]);
    const days = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
    const daysMap = new Map([
      ['星期一', 'monday'],
      ['星期二', 'tuesday'],
      ['星期三', 'wednesday'],
      ['星期四', 'thursday'],
      ['星期五', 'friday'],
      ['星期六', 'saturday']
    ]);
    const filterText = ref('');
    const selectedClub = ref(null);
    const currentView = ref('schedule');
    const showAddCourseModal = ref(false);
    const newCourse = ref('');
    const showDeleteConfirmDialog = ref(false);
    const courseToDelete = ref({ id: null, name: '' });
    const showAddScheduleModal = ref(false);
    const selectedDay = ref('');
    const startTime = ref('');
    const endTime = ref('');
    const courses = ref([]);
    const searchTerm = ref('');
    const apiUrl = process.env.VUE_APP_API_URL;
    const selectedStartTime = ref('');
    const selectedEndTime = ref('');
    const selectClassroom = ref('');
    const selectedCourse = ref(null);
    const updateSelectedClub = (newValue) => {
      selectedClub.value = newValue
    }


    const fetchCourses = async () => {
      try {
        const response = await fetch(`${apiUrl}/lessons`);
        courses.value = await response.json();
      } catch (error) {
        console.error('Error fetching courses:', error);
      }
    };

    const handleCourseSelect = (course) => {
      selectedCourse.value = course;
      console.log("Selected course:", selectedCourse.value); // 用於調試
    };
    const filteredLessons = computed(() => {
      return courses.value.filter(course =>
        course.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      );
    });

    onMounted(fetchCourses);

    const confirmDelete = (id, name) => {
      courseToDelete.value = { id, name };
      showDeleteConfirmDialog.value = true;
    };

    const checkSchedule = async () => {
      const existingSchedules = await getExistingSchedules();
      if (existingSchedules.length > 0) {
        let targetDay = daysMap.get(selectedDay.value)
        const scheduleDetails = existingSchedules
          .map(schedule => `${schedule[targetDay.toLowerCase()]} ${schedule.startTime} - ${schedule.endTime}`)
          .join('\n');

        const confirmed = window.confirm(
          `${selectedDay.value} 裡已有課程:\n${scheduleDetails}\n確認要排課嗎？`
        );
        if (confirmed) {
          addSchedule();
        }
      } else {
        addSchedule();
      }
    };

    const deleteCourse = async () => {
      try {
        await fetch(`${apiUrl}/classroomSchedule/${courseToDelete.value.id}`, {
          method: 'DELETE'
        });
        showDeleteConfirmDialog.value = false;
        fetchData();
      } catch (error) {
        console.error('Error deleting course:', error);
      }
    };

    const getExistingSchedules = async () => {
      try {
        let targetDay = daysMap.get(selectedDay.value)
        let existingSchedules = scheduleData.value
          .filter(schedule => schedule[targetDay.toLowerCase()] != null)
          .filter(schedule => schedule.classRoomName === selectClassroom.value)
          .filter(schedule => {
            let baseDate = new Date(1970, 0, 1);
            let targetStartDate = new Date(baseDate);
            let [hours, minutes] = selectedStartTime.value.split(':').map(Number);
            targetStartDate.setHours(hours);
            targetStartDate.setMinutes(minutes);

            let targetEndDate = new Date(baseDate);
            let [targetEndHours, targetEndMinutes] = selectedEndTime.value.split(':').map(Number);
            targetEndDate.setHours(targetEndHours);
            targetEndDate.setMinutes(targetEndMinutes);

            let scheduleStartTime = schedule.startTime
            let startDate = new Date(baseDate);
            let [startHour, startMinutes] = scheduleStartTime.split(':').map(Number);
            startDate.setHours(startHour);
            startDate.setMinutes(startMinutes);

            let endTime = schedule.endTime
            let endDate = new Date(baseDate);
            let [endHour, endMinutes] = endTime.split(':').map(Number);
            endDate.setHours(endHour);
            endDate.setMinutes(endMinutes);

            return (targetStartDate >= startDate && targetStartDate <= endDate)
              || (targetEndDate >= startDate && targetEndDate <= endDate)
          });

        console.log('Existing Schedules:', existingSchedules);
        return existingSchedules;
      } catch (error) {
        console.error('Error checking schedule overlap:', error);
        return false;
      }
    };

    const addSchedule = async () => {
      console.log(selectClassroom.value)
      try {
        const scheduleData = {
          dayOfWeek: daysMap.get(selectedDay.value).toUpperCase(),
          startTime: `${selectedStartTime.value}:00`,
          endTime: `${selectedEndTime.value}:00`,
          classRoomName: selectClassroom.value,
          lessonId: selectedCourse.value.id  // 使用課程名稱
        };
        await fetch(`${apiUrl}/classRoomSchedule`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(scheduleData)
        });
        showAddScheduleModal.value = false;
        fetchData();
      } catch (error) {
        console.error('Error adding schedule:', error);
        fetchData();
      }
    };

    const clubColors = {
      '桌球隊': '#FFADAD',
      '課後活動': '#CAFFBF',
      '弦樂團': '#FDFFB6',
      '籃球隊': '#FFD6A5',
      '足球隊': '#A0C4FF',
      '扯鈴隊': '#BDB2FF',
      '弦個練': '#9BF6FF',
      '舞蹈團': '#FFC6FF',
    };

    const fetchData = async () => {
      try {
        const response = await fetch(`${apiUrl}/classrooms`);
        scheduleData.value = await response.json();
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    const fetchCourseData = async () => {
      try {
        const response = await fetch(`${apiUrl}/lessons`);
        courseData.value = await response.json();
      } catch (error) {
        console.error('Error Fetching data:', error)
      }
    }

    onMounted(fetchData);

    watch(currentView, (newView) => {
      if (newView === 'courses') {
        fetchCourseData();
      }
    });

    const uniqueClassrooms = computed(() =>
      [...new Set(scheduleData.value.map(item => item.classRoomName))]
    );

    const filteredCourses = computed(() => {
      return courseData.value.filter(course => {
        const matchesClub = selectedClub.value === null || course.clubName === selectedClub.value;
        const matchesFilter = course.name.includes(filterText.value);
        return matchesClub && matchesFilter;
      });
    });

    const uniqueCourses = computed(() => {
      const courses = new Set();
      days.forEach(day => {
        const dayKey = getDayKey(day);
        scheduleData.value.forEach(item => {
          if (item[dayKey]) courses.add(item[dayKey]);
        });
      });
      return [...courses];
    });

    const getDayKey = (day) => {
      const dayMap = {
        '星期一': 'monday',
        '星期二': 'tuesday',
        '星期三': 'wednesday',
        '星期四': 'thursday',
        '星期五': 'friday',
        '星期六': 'saturday'
      };
      return dayMap[day];
    };

    const getFilteredClassesForDay = (classroom, day) => {
      const dayKey = getDayKey(day);
      const classes = scheduleData.value.filter(item =>
        item.classRoomName === classroom &&
        (selectedClub.value === null || item.clubName === selectedClub.value)
      );

      return classes
        .map(item => ({
          ...item,
          courseName: item[dayKey],
          color: clubColors[item.clubName] || '#FFFFFF'
        }))
        .filter(item => item.courseName && item.courseName.includes(filterText.value));
    };

    const addCourse = async () => {
      if (!newCourse.value || !selectedClub.value) {
        console.error('Course name and Club ID are required');
        return;
      }

      try {
        const newCourseForJson = {
          clubName: selectedClub.value,
          name: newCourse.value
        };
        await fetch(`${apiUrl}/lesson`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(newCourseForJson)
        });
        newCourse.value = '';
        selectedClub.value = '';
        showAddCourseModal.value = false;
        fetchCourseData();
      } catch (error) {
        console.error('Error adding course:', error);
      }
    };

    const uniqueClubs = computed(() =>
      [...new Set(scheduleData.value.map(item => item.clubName))]
    );

    return {
      days,
      uniqueClassrooms,
      uniqueClubs,
      uniqueCourses,
      filteredCourses,
      getFilteredClassesForDay,
      filterText,
      selectedClub,
      currentView,
      addCourse,
      courseToDelete,
      deleteCourse,
      confirmDelete,
      showDeleteConfirmDialog,
      showAddCourseModal,
      newCourse,
      showAddScheduleModal,
      selectedDay,
      startTime,
      endTime,
      courses,
      checkSchedule,
      filteredLessons,
      selectedStartTime,
      selectedEndTime,
      selectClassroom,
      handleCourseSelect,
      updateSelectedClub
    };
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200');

.club-button {
  padding: 0.75rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.25rem;
  font-weight: 600;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.club-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, #3b82f6, #60a5fa);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: -1;
}

.club-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.club-button:hover::before {
  opacity: 0.1;
}

.club-button.active {
  background-color: #3b82f6;
  color: white;
  transform: translateY(-3px);
  box-shadow: 0 4px 6px rgba(59, 130, 246, 0.5);
}

.club-button.active::before {
  opacity: 1;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.7);
  }
  70% {
    box-shadow: 0 0 0 10px rgba(59, 130, 246, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(59, 130, 246, 0);
  }
}

.club-button.active {
  animation: pulse 1.5s infinite;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  .club-filter-container {
    margin-bottom: 1rem;
  }
}

/* Additional styles from your original CSS */
.container {
  max-width: 100%;
  overflow-x: auto;
}

table {
  border-collapse: separate;
  border-spacing: 0;
  width: 100%;
  padding: 0.75rem 1.5rem;
}

th,
td {
  text-align: left;
  padding: 12px;
  border: 1px solid #e2e8f0;
}

th {
  background-color: #f8fafc;
  font-weight: bold;
  text-transform: uppercase;
  font-size: 0.75rem;
  letter-spacing: 0.05em;
}

tr:nth-child(even) {
  background-color: #f9fafb;
}

tr:hover {
  background-color: #f1f5f9;
}

.text-center {
  text-align: center;
}

.font-medium {
  font-weight: 500;
}

.mb-1 {
  margin-bottom: 0.25rem;
}

/* Custom TimePicker styles */
::v-deep .vue3-timepicker input {
  width: 100%;
  padding: 0.75rem;
  border-radius: 0.375rem;
  border: 1px solid #e2e8f0;
  transition: all 0.2s;
}

::v-deep .vue3-timepicker input:focus {
  outline: none;
  box-shadow: 0 0 0 2px #3b82f6;
}

.group:hover {
  transform: translateY(-2px);
}

.course-card {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 12px;
  transition: all 0.3s ease;
}

.course-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.course-name {
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  font-size: 1rem;
  color: #2d3748;
  margin-bottom: 4px;
  letter-spacing: 0.5px;
}

.course-time {
  font-family: 'Poppins', sans-serif;
  font-weight: 400;
  font-size: 0.875rem;
  color: #4a5568;
}

</style>