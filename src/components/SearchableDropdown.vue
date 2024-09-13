<template>
    <div class="relative" ref="dropdownRef">
        <input type="text" class="w-full p-2 border rounded" placeholder="搜尋課程" v-model="searchTerm"
            @focus="isOpen = true" />
        <ul v-if="isOpen && filteredCourses.length > 0"
            class="absolute z-10 w-full mt-1 bg-white border border-gray-300 rounded shadow-lg max-h-60 overflow-y-auto">
            <li v-for="course in filteredCourses" :key="course" class="p-2 hover:bg-gray-100 cursor-pointer"
                @click="handleSelect(course)">
                {{ course.name }}
            </li>
        </ul>
        <ul v-if="isOpen && filteredCourses.length === 0"
            class="absolute z-10 w-full mt-1 bg-white border border-gray-300 rounded shadow-lg max-h-60 overflow-y-auto">
            <li class="p-2 text-gray-500">
                沒有找到課程
            </li>
        </ul>
    </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue';

export default {
    name: 'SearchableDropdown',
    props: {
        courses: {
            type: Array,
            required: true
        }
    },
    emits: ['select'],
    setup(props, { emit }) {
        const searchTerm = ref('');
        const isOpen = ref(false);
        const dropdownRef = ref(null);
        const selectCourse = (course) => {
            emit('select', course);  // 發出整個課程對象
        };
        const filteredCourses = computed(() => {
            return props.courses.filter(course =>
                course.name.toLowerCase().includes(searchTerm.value.toLowerCase())
            );
        });

        const handleSelect = (course) => {
            searchTerm.value = course.name;
            isOpen.value = false;
            emit('select', course); // 發出 select 事件，並傳遞整個 course 對象
        };

        const handleClickOutside = (event) => {
            if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
                isOpen.value = false;
            }
        };

        onMounted(() => {
            document.addEventListener('click', handleClickOutside);
        });

        onUnmounted(() => {
            document.removeEventListener('click', handleClickOutside);
        });

        return {
            searchTerm,
            isOpen,
            filteredCourses,
            handleSelect,
            dropdownRef,
            selectCourse
        };
    }
};
</script>