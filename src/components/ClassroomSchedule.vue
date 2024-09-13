<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">教室排課表</h1>

    <div class="flex">
      <div class="w-1/6 flex flex-col space-y-2">
        <button @click="currentView = 'schedule'"
          :class="['w-32 p-2 border rounded', currentView === 'schedule' ? 'bg-blue-500 text-white' : 'bg-gray-200']">
          排課區
        </button>
        <button @click="currentView = 'courses'"
          :class="['w-32 p-2 border rounded', currentView === 'courses' ? 'bg-blue-500 text-white' : 'bg-gray-200']">
          課程區
        </button>
      </div>

      <div class="w-5/6 flex flex-col">
        <div class="flex justify-between items-center mb-4">
          <div></div>
          <button v-if="currentView === 'schedule'" @click="showAddScheduleModal = true"
            class="p-2 border rounded bg-green-500 text-white">
            新增排課
          </button>
        </div>

        <div class="flex justify-between items-center mb-4">
          <div></div>
          <button v-if="currentView === 'courses'" @click="showAddCourseModal = true"
            class="p-2 border rounded bg-green-500 text-white">
            新增課程
          </button>
        </div>

        <div class="mb-4">
          <div class="flex space-x-2">
            <button @click="selectedClub = null"
              :class="['p-2 border rounded', selectedClub === null ? 'bg-blue-500 text-white' : 'bg-gray-200']">
              全部
            </button>
            <button v-for="club in uniqueClubs" :key="club" @click="selectedClub = club"
              :class="['p-2 border rounded', selectedClub === club ? 'bg-blue-500 text-white' : 'bg-gray-200']">
              {{ club }}
            </button>
          </div>
        </div>

        <div class="mb-4">
          <input v-model="filterText" placeholder="搜尋課程" class="p-2 border rounded" />
        </div>

        <div class="overflow-x-auto">
          <!-- 排課區視圖 -->
          <table class="min-w-full bg-white border border-gray-300" v-if="currentView === 'schedule'">
            <thead>
              <tr>
                <th class="py-2 px-4 border-b">教室名稱</th>
                <th v-for="day in days" :key="day" class="py-2 px-4 border-b">{{ day }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="classroom in uniqueClassrooms" :key="classroom">
                <td class="py-2 px-4 border-b font-medium">{{ classroom }}</td>
                <td v-for="day in days" :key="day" class="py-2 px-4 border-b">
                  <div v-for="classInfo in getFilteredClassesForDay(classroom, day)" :key="classInfo.startTime"
                    class="mb-1 text-center relative group" :style="{ backgroundColor: classInfo.color }">
                    <div class="font-medium">{{ classInfo.courseName }}</div>
                    <div class="text-sm">({{ classInfo.startTime }} - {{ classInfo.endTime }})</div>
                    <button @click="confirmDelete(classInfo.id, classInfo.courseName)"
                      class="absolute top-0 right-0 bg-red-500 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs opacity-0 group-hover:opacity-100 transition-opacity duration-200">
                      X
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>

          <!-- 課程區視圖 -->
          <table class="min-w-full bg-white border border-gray-300" v-else>
            <thead>
              <tr>
                <th class="py-2 px-4 border-b text-center">社團名稱</th>
                <th class="py-2 px-4 border-b text-center">課程名稱</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="course in filteredCourses" :key="course.id">
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
      <div class="bg-white p-4 rounded shadow-lg">
        <h2 class="text-xl font-bold mb-4">新增排課</h2>
        <form @submit.prevent="checkSchedule">
          <div class="mb-4">
            <label class="block mb-2" for="courseSelect">選擇課程</label>
            <SearchableDropdown
              :courses="courses"
              @select="handleCourseSelect"
            />
          </div>
          <div class="mb-4">
            <label class="block mb-2" for="dayOfWeek">選擇星期</label>
            <select v-model="selectedDay" id="dayOfWeek" class="p-2 border rounded w-full">
              <option v-for="day in days" :key="day" :value="day">{{ day }}</option>
            </select>
          </div>
          <div class="mb-4">
            <label class="block mb-2">選擇時間</label>
            <TimeRangePicker @select-time-range="handleTimeRangeSelect" />
          </div>
          <div class="flex justify-end space-x-2">
            <button type="button" @click="showAddScheduleModal = false" class="p-2 border rounded bg-gray-200">
              取消
            </button>
            <button type="submit" class="p-2 border rounded bg-blue-500 text-white">
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
import { ref, onMounted, computed, watch } from 'vue';
import SearchableDropdown from './SearchableDropdown.vue';
export default {
  name: 'ClassroomSchedule',
  components: {
    SearchableDropdown,
  },
  setup() {
    const scheduleData = ref([]);
    const courseData = ref([]);
    const days = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
    const filterText = ref('');
    const selectedClub = ref(null);
    const currentView = ref('schedule'); // 控制當前視圖
    const showAddCourseModal = ref(false);
    const newCourse = ref('');
    const showDeleteConfirmDialog = ref(false);
    const courseToDelete = ref({ id: null, name: '' });
    const showAddScheduleModal = ref(false);
    const selectedCourse = ref('');
    const selectedDay = ref('');
    const startTime = ref('');
    const endTime = ref('');
    const courses = ref([]); // Initialize with course data
    const searchTerm = ref('');
    const isDropdownOpen = ref(false);
 
    // 获取课程数据
    const fetchCourses = async () => {
      try {
        const response = await fetch(`${apiUrl}/lessons`);
        courses.value = await response.json();
      } catch (error) {
        console.error('Error fetching courses:', error);
      }
    };

    // 过滤课程
    const filteredLessons = computed(() => {
      return courses.value.filter(course =>
        course.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      );
    });

    // 选择课程
    const selectCourse = (course) => {
      selectedCourse.value = course;
      searchTerm.value = '';
      isDropdownOpen.value = false;
    };

    // 关闭下拉菜单
    const closeDropdown = () => {
      setTimeout(() => isDropdownOpen.value = false, 200);
    };

    // 组件挂载时获取课程数据
    onMounted(fetchCourses);
    const confirmDelete = (id, name) => {
      courseToDelete.value = { id, name };
      showDeleteConfirmDialog.value = true;
    };

    const apiUrl = `${process.env.VUE_APP_API_URL}`;
    const checkSchedule = async () => {
      const isOverlapping = await checkTimeOverlap();
      if (isOverlapping) {
        const confirmed = window.confirm(`此时间(${selectedDay.value} ${startTime.value} - ${endTime.value}) 已经有课程。确认要排课吗？`);
        if (confirmed) {
          addSchedule();
        }
      } else {
        addSchedule();
      }
    };

    const checkTimeOverlap = async () => {
      try {
        const response = await fetch(`${apiUrl}/classRoomSchedule?dayOfWeek=${selectedDay.value}&startTime=${startTime.value}&endTime=${endTime.value}`);
        const existingSchedules = await response.json();
        return existingSchedules.length > 0;
      } catch (error) {
        console.error('Error checking schedule overlap:', error);
        return false;
      }
    };

    const addSchedule = async () => {
      try {
        const scheduleData = {
          dayOfWeek: selectedDay.value,
          startTime: startTime.value,
          endTime: endTime.value,
          classRoomId: 41, // Set the class room ID
          lessonId: selectedCourse.value
        };
        await fetch(`${apiUrl}/classRoomSchedule`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(scheduleData)
        });
        showAddScheduleModal.value = false;
        // Refresh data or handle success
      } catch (error) {
        console.error('Error adding schedule:', error);
      }
    };

    const deleteCourse = async () => {
      try {
        await fetch(`${apiUrl}/${courseToDelete.value.id}`, {
          method: 'DELETE'
        });
        showDeleteConfirmDialog.value = false;
        fetchData(); // Refresh the data after deletion
      } catch (error) {
        console.error('Error deleting course:', error);
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
        const response = await fetch(apiUrl + '/classrooms');
        scheduleData.value = await response.json();
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    const fetchCourseData = async () => {
      try {
        const response = await fetch(apiUrl + '/lessons');
        courseData.value = await response.json();
      } catch (error) {
        console.error('Error Fetching data:', error)
      }
    }

    onMounted(fetchData);

    // Call fetchCourses when switching to 'courses' view
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

    // 新增：計算唯一課程
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
      console.log(newCourse.value)
      console.log(selectedClub.value)
      if (!newCourse.value || !selectedClub.value) {
        console.error('Course name and Club ID are required');
        return;
      }

      try {
        const newCourseForJson = {
          clubName: selectedClub.value, // 使用選取的 clubId
          name: newCourse.value
        };
        await fetch('http://localhost:8091/lesson', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(newCourseForJson)
        });
        newCourse.value = '';
        selectedClub.value = ''; // 清空選取的社團
        showAddCourseModal.value = false;
        fetchCourseData(); // 重新擷取課程資料
      } catch (error) {
        console.error('Error adding course:', error);
      }
    };

    const handleCourseSelect = (course) => {
    selectedCourse.value = course.id;
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
      selectedCourse,
      selectedDay,
      startTime,
      endTime,
      courses,
      checkSchedule,
      selectCourse,
      closeDropdown,
      filteredLessons,
      handleCourseSelect
    };
  }
};
</script>

<style scoped>
.container {
  max-width: 100%;
  overflow-x: auto;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  text-align: left;
  padding: 8px;
  border: 1px solid #ddd;
}

th {
  background-color: #f2f2f2;
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

tr:hover {
  background-color: #f5f5f5;
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
</style>