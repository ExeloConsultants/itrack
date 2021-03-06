<template>
  <div :class="{ 'active' : active, 'text-sm' : !active }" class="task-item px-8 py-6 mb-4">
    <p :class="{ 'font-bold' : active }" class="flex-grow">{{ task.name }}</p>
    <div class="separator h-px  md:hidden my-4" />
    <div class="flex items-center justify-between">
      <p class="w-1/3 md:ml-8 md:mr-4">{{ timeSpent }}</p>
      <div class="w-1/3 relative md:px-8">
        <transition-group name="buttons">
          <BtnStart v-if="!active" btnTitle="Start Tracking" @click="startTracking" />
          <BtnPause v-else :active="active" btnTitle="Pause Tracking" @click="pauseTracking" />
        </transition-group>
      </div>
      <div class="w-1/3">
        <BtnDelete class="ml-auto" btnTitle="Delete Task" @click="deleteTaskFromList(task)" />
      </div>
    </div>
  </div>
</template>

<script>
import BtnDelete from './button/BtnDelete.vue';
import BtnPause from './button/BtnPause.vue';
import BtnStart from './button/BtnStart.vue';

import { computed, ref, watch } from 'vue';
import { useStore } from '../store.js';
import { formatTime } from '../utils.js'

export default {
  name: 'Task',
  components: {
    BtnDelete,
    BtnPause,
    BtnStart
  },
  props: {
    task: Object
  },
  emits: ['reduce:total','update:total'],
  setup(props, { emit }) {
    const { removeTask, setActiveTask, setPausedTask } = useStore();

    const active = computed(() => props.task.taskActive);
    const taskTotal = ref(0);
    const timeSpent = computed(() => { return formatTime(taskTotal.value) });

    let increment;

    const deleteTaskFromList = (current) => {
      if (taskTotal.value > 0) {
        if(confirm('Do you really want to remove this task and the time spent on it?')) {
          clearInterval(increment);
          removeTask(current);
          emit('reduce:total', taskTotal.value);
        } else { return }
      } else {
        removeTask(current);
      }
    }

    const startTracking = () => {
      setActiveTask(props.task.id);
    }

    const pauseTracking = () => {
      setPausedTask(props.task.id);
    }

    watch(active, () => {
      if (active.value) {
        increment = setInterval(() => {
          taskTotal.value++;
          emit('update:total');
        }, 1000);
      } else {
        clearInterval(increment);
      }
    })

    return {
      active,
      deleteTaskFromList,
      pauseTracking,
      startTracking,
      timeSpent
    }
  }
}
</script>