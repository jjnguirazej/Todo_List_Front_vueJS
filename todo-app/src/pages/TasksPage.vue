<template>
    <main style="min-height: 50vh; margin-top: 2rem;">
        <div class="container">
            <div class="row">
                <div class="col-md-8 offset-md-2">
                    <!-- Add new Task -->
                    <NewTask @added="handleAddedTask" />
                    <!-- List of uncompleted tasks -->

                    <Tasks :tasks="uncompletedTasks" 

                            @updated="handleUpdatedTask"

                            @completed ="handleCompletedTasks"

                            @removed ="handleRemovedTasks"

                            />

                    <!-- Show toggle button -->

                    <div class="text-center my-3" v-show="showToggleCompletedBtn">
                        <button class="btn btn-sm btn-secondary"
                            @click="$event => showCompletedTasks = !showCompletedTasks">
                            <span v-if="!showCompletedTasks">show completed</span>
                            <span v-else>hide  completed</span>
                        </button>
                    </div>

                    <!-- List of completed tasks -->
                    <Tasks :tasks="completedTasks" 
                        :show="completedTaskIsVisible && showCompletedTasks"
                        @updated="handleUpdatedTask"
                        @completed ="handleCompletedTasks"
                        @removed ="handleRemovedTasks"
                        
                        />


                </div>
            </div>
        </div>
    </main>
</template>

<script setup>

import { computed, onMounted, ref } from 'vue';

import { storeToRefs } from 'pinia';

import { allTasks, createTask, updateTask, completeTask, removeTask } from "../components/http/task-api";

import { useTaskStore } from "../stores/task"

import Tasks from "../components/tasks/Tasks.vue"

import NewTask from '../components/tasks/NewTask.vue';

const store = useTaskStore();

const { completedTasks, uncompletedTasks } = storeToRefs(store)

const tasks = ref([])
onMounted(async ()=>{

   await store.fetchAllTasks()

})

const showToggleCompletedBtn = computed(
    () => uncompletedTasks.value.length > 0 && completedTasks.value.length > 0
)

const completedTaskIsVisible = computed(
    () => uncompletedTasks.value.length == 0 || completedTasks.value.length > 0
)

const showCompletedTasks = ref(false)

const handleAddedTask = async (NewTask) => {

    const { data: createdTask} = await createTask(NewTask)

    tasks.value.unshift(createdTask.data)
}

const handleUpdatedTask = async(task) => {

    const { data: updatedTask } = await updateTask(task.id,{

        name: task.name
    })

    const currentTask = tasks.value.find(item => item.id == task.id) 
    currentTask.name = updatedTask.data.name
}

const handleCompletedTasks = async(task) => {

const { data: updatedTask } = await completeTask(task.id,{

    is_completed: task.is_completed
})

const currentTask = tasks.value.find(item => item.id == task.id) 
currentTask.is_completed = updatedTask.data.is_completed
}

const handleRemovedTasks = async(task) => {

    await removeTask(task.id)
    
    const index = tasks.value.findIndex(item => item.id == task.id)

    task.value.splice(index,1)
}




</script>