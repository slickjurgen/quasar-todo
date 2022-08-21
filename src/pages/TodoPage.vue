<template>
  <q-page class="bg-grey-3 column">
    <div class="row q-pa-sm bg-primary">
      <q-input 
        v-model="newTask" 
        @keyup.enter="addTask"
        class="col"
        square
        filled 
        bg-color="white"
        placeholder="Add task"
        dense>
        <template v-slot:append>
          <q-btn
            @click="addTask" 
            round 
            dense 
            flat 
            icon="add" />
        </template>
      </q-input>
    </div>
    <q-list 
      class="bg-white"
      separator
      bordered>
      <!--
        Rendering a <label> tag (notice tag="label")
        so QCheckboxes will respond to clicks on QItems to
        change Toggle state.
      -->

      <q-item
        v-for="(task, index) in tasks"
        :key="task.title" 
        @click="markTaskDone(index)"
        clickable
        :class="{ 'done bg-blue-1' : task.done }"
        v-ripple>
        <q-item-section avatar>
          <q-checkbox 
            v-model="task.done"  
            class="no-pointer-events"
            color="primary" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{ task.title }}</q-item-label>
        </q-item-section>
        <q-item-section
          v-if="task.done"
          side>
          <q-btn 
            @click.stop="deleteTask(index)"
            flat 
            round 
            dense
            color="primary" 
            icon="delete" />
        </q-item-section>
      </q-item>
    </q-list>
    <div
      v-if="!tasks.length" 
      class="no-tasks absolute-center">
      <q-icon
        name="check"
        size="100px"
        color="primary" 
      />
      <div class="text-h5 text-primary text-center">
        No tasks
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { v4 as uuidv4 } from 'uuid'

export default defineComponent({
  data() {
    return {
      newTask: '',
      tasks: [
      //  {
      //    title: 'Get bananas',
      //    done: false
      //  },        
      //  {
      //    title: 'Eat bananas',
      //    done: false
      //  },        
      //  {
      //    title: 'Poo bananas',
      //    done: false
      //  }
      ]
    } 
  },

  created() {
    // fetch on init
    this.fetchData()
  },
  methods: {
    fetchData() {
      fetch('http://localhost:10245/graphql/todo', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: JSON.stringify({
          query: "{ allTask{_id, title, done} }",
          //variables: {
          //  now: new Date().toISOString(),
          //},
        }),
      })
      //  .then((res) => res.json())
      //  .then((result) => console.log(result));

        .then(function(response) {
          return response.json()
        })
        .then(data => {
          console.log(data);
          //data.data.allTask.forEach(function (task, index) {
          for (let task of data.data.allTask) {
            console.log(task);
            this.tasks.push({
              id: task._id,
              title: task.title,
              done: task.done?task.done:false
            });
          }
          //});
        })
        //.then(data =>  this.tasks.push({
        //    title: data.data.allTask[0].title,
        //    done: false
        //}))

    },
    saveData(task) {
      console.log("trying to save task...");
      fetch('http://localhost:10245/data/mutate/todo', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: '{"mutations":[{"createOrReplace":{"_id":"' + task.id + 
          '", "_type":"task", "title":"' + task.title + 
          '", "done":' + task.done + '}}]}',
      })        
        .then(function(response) {
          return response.json()
        })
        .then(data => console.log(data))
    },
    deleteData(task) {
      console.log("trying to remove task...");
      fetch('http://localhost:10245/data/mutate/todo', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: '{"mutations":[{"delete":{"_id":"' + task.id + '"}}]}',
      })        
        .then(function(response) {
          return response.json()
        })
        .then(data => console.log(data))
    },
    markTaskDone(index) {
      this.tasks[index].done = !this.tasks[index].done
      this.saveData(this.tasks[index])
    },
    deleteTask(index) {
      this.$q.dialog({
        title: 'Confirm',
        message: 'Really delete?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        this.deleteData(this.tasks[index])
        this.tasks.splice(index, 1)
        this.$q.notify('Task deleted')
      })
    },
    addTask() {
      let task = {id: uuidv4(), title: this.newTask, done: false}
      this.tasks.push(task)
// save to db
      this.saveData(task)
      this.newTask = ''
    }
  }

})
</script>

<style lang="scss">
  .done {
    .q-item__label {
      text-decoration: line-through;
      color: #bbb;
    } 
  }
  .no-tasks {
    opacity: 0.5;
  }
</style>
