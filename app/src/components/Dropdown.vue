<template>
    <div class="relative inline-block text-left">
      <div>
        <button @click.stop="toggleDropdown" type="button" class="inline-flex justify-between w-full px-2 py-2 bg-blue-700 text-sm font-medium text-white hover:bg-blue-900 border border-gray-300 rounded-full shadow-sm focus:outline-none" id="menu-button">
          <span v-if="false">{{ selectedLabel || 'Select an option' }}</span>
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M5.23 7.21a.75.75 0 011.06.02L10 10.94l3.71-3.71a.75.75 0 111.06 1.06l-4 4a.75.75 0 01-1.06 0l-4-4a.75.75 0 01.02-1.06z" clip-rule="evenodd" />
          </svg>
        </button>
      </div>
  
      <!-- Dropdown menu -->
      <div v-if="isOpen" class="dropdown-options text-white origin-top-right absolute right-0 mt-0 rounded-md shadow-lg bg-blue-700 ring-1 ring-black ring-opacity-5 focus:outline-none z-10" role="menu">
        <div class="dropdown-back"></div>
        <div
            v-for="(item, index) in list"
            :key="index"
            @click="selectOption(item)"
            class="hover:bg-blue-900"
          >
            {{ item.text }}
        </div>
        <slot></slot>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'Dropdown',
    props: {
      modelValue: {
        type: [String, Number],
        required: true
      },
      list: {
        type: Array,
        required: true,
        default: () => []
      }
    },
    data() {
      return {
        isOpen: false
      };
    },
    mounted() {
      window.addEventListener('click',() => {
        this.hideDropdown()
      })
    },
    computed: {
      selectedLabel() {
        const selected = this.list.find(item => item.key === this.modelValue);
        return selected ? selected.value : '';
      }
    },
    methods: {
      hideDropdown() {
        this.isOpen = false
      },
      toggleDropdown() {
        this.isOpen = !this.isOpen;
      },
      selectOption(item) {
        this.$emit('update:modelValue', item.key);
        this.isOpen = false;
      }
    }
  };
  </script>
  
  <style>
  .dropdown-options{
    min-width: 140px;
    overflow: hidden;
  }
  .dropdown-options > div{
    padding: 0.75rem 1rem;
  }
  .dropdown-back{
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    background-color: rgba(0,0,0,0.2);
  }
  </style>