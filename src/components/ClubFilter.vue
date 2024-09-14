<template>
    <div class="club-filter-container my-8">
      <div class="flex flex-wrap justify-center gap-3">
        <button @click="$emit('update:modelValue', null)" 
                :class="['club-button', modelValue === null ? 'active' : '']">
          全部
        </button>
        <button v-for="club in clubs" 
                :key="club" 
                @click="$emit('update:modelValue', club)" 
                :class="['club-button', modelValue === club ? 'active' : '']">
          {{ club }}
        </button>
      </div>
    </div>
  </template>
  
  <script>
  import { defineComponent } from 'vue'
  
  export default defineComponent({
    name: 'ClubFilter',
    props: {
      clubs: {
        type: Array,
        required: true
      },
      modelValue: {
        type: String,
        default: null
      }
    },
    emits: ['update:modelValue']
  })
  </script>
  
  <style scoped>
  .club-filter-container {
    perspective: 1000px;
  }
  
  .club-button {
    padding: 0.75rem 1.5rem;
    border: 2px solid #e2e8f0;
    border-radius: 9999px;
    font-weight: 600;
    transition: all 0.3s ease;
    transform-style: preserve-3d;
    position: relative;
    overflow: hidden;
  }
  
  .club-button:before {
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
    transform: translateY(-3px) rotateX(5deg);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
  
  .club-button:hover:before {
    opacity: 0.1;
  }
  
  .club-button.active {
    background-color: #3b82f6;
    color: white;
    transform: translateY(-3px);
    box-shadow: 0 4px 6px rgba(59, 130, 246, 0.5);
  }
  
  .club-button.active:before {
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
  </style>