<template>
  <div>
    <div class="form">
      <div class="form__element">
        Задачи
        <input v-model="taskName" type="text" placeholder="Добавьте задачи" />
        <button type="button" @click="addTask">Добавить</button>
        <div class="form__element__list">
          <span
            class="form__element__list-elem"
            v-for="task in data.tasks"
            :key="task.id"
          >
            <b>id:</b> {{ task.id }} <b>name:</b> {{ task.name }}
            <button @click="deleteTask(task.id)">Удалить</button>
          </span>
        </div>
      </div>
      <div class="form__element">
        Даты
        <input type="text" placeholder="Добавьте даты" v-model="dateName" />
        <button type="button" @click="addDate">Добавить</button>
        <div class="form__element__list">
          <span
            class="form__element__list-elem"
            v-for="date in data.dates"
            :key="date.id"
          >
            <b>id:</b> {{ date.id }} <b>name:</b> {{ date.name }}
            <button @click="deleteDate(date.id)">Удалить</button>
          </span>
        </div>
      </div>
      <div class="form__element">
        Статус
        <input type="text" placeholder="Добавьте статус" v-model="statusName" />
        <button type="button" @click="addStatus">Добавить</button>
        <div class="form__element__list">
          <span
            class="form__element__list-elem"
            v-for="status in data.status"
            :key="status.id"
          >
            <b>id:</b> {{ status.id }} <b>name:</b> {{ status.name }}
            <button @click="deleteStatus(status.id)">Удалить</button>
          </span>
        </div>
      </div>
      <div class="form__element">
        События
        <div class="form__element__list">
          <span
            class="form__element__list-elem"
            v-for="event in data.events"
            :key="event.id"
          >
            <b>id:</b> {{ event.id }} <b>status:</b> {{ event.status.name }}
            <b>date:</b> {{ event.date.name }} <b>task:</b>
            {{ event.task.name }}
            <button @click="deleteEvent(event.id, event.status.id, event.date.id, event.task.id)">Удалить</button>
          </span>
        </div>
      </div>
    </div>

    <table>
      <thead>
        <tr>
          <td>Задачи/Даты</td>
          <td v-for="date in data.dates" :key="date.id">
            {{ date.name }}
          </td>
        </tr>
      </thead>

      <tbody>
        <tr v-for="task in data.tasks" :key="task.id">
          <input v-model="task.name" type="text" />
          <td v-for="date in data.dates" :key="date.id">
            <select
              @change="
                (event) => setEvent(event.target.value, task.id, date.id)
              "
            >
              <option value="">Выберите статус задачи</option>
              <option
                v-for="option in data.status"
                :value="option.id"
                :key="option.id"
              >
                {{ option.name }}
              </option>
            </select>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";

const taskName = ref("");
const dateName = ref("");
const statusName = ref("");

const data = reactive({
  tasks: [],
  dates: [],
  status: [],
  events: [],
});

const getId = (length = 10) => {
  return Math.random().toString(36).substring(2, length);
};

const addTask = () => {
  if (!taskName.value) {
    console.log("Введите задачу");
    return;
  }
  data.tasks.push({
    id: getId(),
    name: taskName.value,
  });

  taskName.value = "";
};

const deleteTask = (id) => {
  const index = data.tasks.findIndex((t) => t.id == id);
  data.tasks.splice(index, 1);

  data.events = data.events.map((e) => {
    if (e.task.id == id) {
      e.task = {};
    }
    return e;
  });
};

const addDate = () => {
  if (!dateName.value) {
    console.log("Введите дату");
    return;
  }
  data.dates.push({
    id: getId(),
    name: dateName.value,
  });

  dateName.value = "";
};

const deleteDate = (id) => {
  const index = data.dates.findIndex((d) => d.id == id);
  data.dates.splice(index, 1);

  data.events = data.events.map((e) => {
    if (e.date.id == id) {
      e.date = {};
    }
    return e;
  });
};

const addStatus = () => {
  if (!statusName.value) {
    console.log("Введите статус");
    return;
  }
  data.status.push({
    id: getId(),
    name: statusName.value,
  });

  statusName.value = "";
};

const deleteStatus = (id) => {
  const index = data.status.findIndex((s) => s.id == id);
  data.status.splice(index, 1);

  data.events = data.events.map((e) => {
    if (e.status.id == id) {
      e.date = {};
    }
    return e;
  });
};

const currEvent = reactive({
  index: "",
  elem: "",
});

const setEvent = (statusId, taskId, dateId) => {
  getCurrEvent(taskId, dateId);

  const date = data.dates.find((d) => d.id == dateId);
  const task = data.tasks.find((t) => t.id == taskId);
  const status = data.status.find((s) => s.id == statusId)

  if (isHasEvent()) {
    const { elem, index } = currEvent;

    if (!statusId) {
      data.events.splice(index, 1);
    } else {
      data.events[index] = {
        id: elem.id,
        date: { id: dateId, name: date.name },
        task: { id: taskId, name: task.name },
        status: { id: statusId, name: status.name },
      };
    }
    clearCurrEvent();
    return;
  }

  data.events.push({
    id: getId(),
    date: { id: dateId, name: date.name },
    task: { id: taskId, name: task.name },
    status: { id: statusId, name: status.name },
  });
};

const isHasEvent = () => {
  return currEvent.elem ? true : false;
};

const clearCurrEvent = () => {
  currEvent.elem = "";
  currEvent.index = "";
};

const getCurrEvent = (taskId, dateId) => {
  const event = data.events.find((e) => {
    return e.date.id == dateId && e.task.id == taskId;
  });

  if (!event) return;

  currEvent.elem = event;
  currEvent.index = data.events.indexOf(event);
};

const deleteEvent = (eventId, statusId, dateId, taskId) => {
  setEvent(statusId, taskId, dateId);

  const index = data.events.findIndex((e) => e.id == eventId);
  data.events.splice(index, 1);
};
</script>

<style>
.form {
  display: flex;
  justify-content: flex-start;
}
.form__element {
  text-align: center;
  min-width: 200px;
  margin: 0 20px;
  padding: 10px;
  border: 1px solid #000;
}
.form__element__list {
  display: flex;
  flex-direction: column;
  padding: 10px 0;
}
</style>
