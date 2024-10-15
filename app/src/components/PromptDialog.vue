<template>
    <transition name="fade" >
      <div v-if="visible" class="fixed inset-0 bg-gray-800 bg-opacity-50 flex items-center justify-center z-20">
        <div class="bg-gray-900 m-2 p-6 rounded-lg shadow-lg w-96 transform transition-transform duration-300" :class="{'scale-100': visible, 'scale-90': !visible}">
          <p class="text-lg font-medium mb-4"><slot>Test</slot></p>
          <input
            v-model="inputValue"
            type="text"
            class="w-full px-4 py-2 border rounded-lg mb-4"
            placeholder="Type here..."
            ref="textinput"
          />
          <div class="flex justify-end space-x-2">
            <button
              @click="handleClose"
              class="px-4 py-2 bg-gray-300 text-gray-700 rounded-lg hover:bg-gray-400 transition"
            >
              Close
            </button>
            <button
              @click="handleButtonClick"
              class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
            >
              Submit
            </button>
          </div>
        </div>
      </div>
    </transition>
  </template>
  
  <script>

  export default {
    props: {
      modelValue: {
        type: String,
      },
      visible: {
        type: Boolean,
        default: false,
      },
    },
    data() {
      return {
        inputValue: "",
      };
    },
    mounted() {
      this.inputValue = this.modelValue
    },
    methods: {
      handleButtonClick() {
        this.$emit("change", this.inputValue);
        this.$emit("update:modelValue", this.inputValue);
        this.inputValue = ""; // Optionally clear the input
        this.handleClose();
      },
      handleClose() {
        this.$emit("close");
      },
    },
    watch: {
      modelValue(newValue) {
        this.inputValue = newValue
      },
      visible(newValue) {
        if (newValue) {
          this.$nextTick(()=>{
            this.$refs.textinput.focus()
          })
        }
      }
    }
  };
  </script>
  
  <style scoped>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s ease, transform 0.3s ease;
  }
  
  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
    transform: scale(0.9);
  }
  </style>
  