<script setup lang="ts">
const props = defineProps({
  name: { type: String, default: "modal" },
  show: { type: Boolean, default: false },
})
</script>
<template>
  <transition name="modal">
    <div v-if="props.show">
      <div
        @click="$emit('onClose')"
        class="modal-mask absolute bg-black opacity-70 inset-0 z-0 h-full top-0 left-0"
      ></div>
      <div
        class="modal-body w-full absolute px-3 py-5 rounded-t-2xl shadow-lg bg-white flex flex-col"
      >
        <div class="modal-content flex-1">
          <slot></slot>
        </div>
        <div class="modal-footer">
          <button
            @click="$emit('onSubmit')"
            class="w-full text-center text-white bg-green-500 py-5 rounded-2xl text-xl"
          >
            บันทึกข้อมูล
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>
<style lang="scss" scoped>
.modal {
  &-body {
    min-height: 50vh;
    bottom: 0;
    transition: all 0.3s ease;
  }
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.1s ease;
}

.modal-enter-active,
.modal-leave-active {
  .modal-body {
    bottom: -50vh;
  }
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

// /* -- slideUp -- */
// @-webkit-keyframes modal-enter {
//   from {
//     -webkit-transform: translate3d(0, 100px, 0);
//     transform: translate3d(0, 100px, 0);
//   }
// }

// @keyframes modal-enter {
//   from {
//     -webkit-transform: translate3d(0, 100px, 0);
//     transform: translate3d(0, 100px, 0);
//   }
// }

// .modal-enter-active {
//   -webkit-animation: modal-enter both cubic-bezier(0.4, 0, 0, 1.5);
//   animation: modal-enter both cubic-bezier(0.4, 0, 0, 1.5);
// }

// @-webkit-keyframes modal-leave {
//   to {
//     -webkit-transform: translate3d(0, 100px, 0);
//     transform: translate3d(0, 100px, 0);
//   }
// }

// @keyframes modal-leave {
//   to {
//     -webkit-transform: translate3d(0, 100px, 0);
//     transform: translate3d(0, 100px, 0);
//   }
// }

// .modal-leave-active {
//   -webkit-animation: modal-leave both;
//   animation: modal-leave both;
// }
</style>
