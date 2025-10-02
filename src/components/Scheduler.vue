<template>
  <DayPilotScheduler
      scale="Day"
      :eventHeight="100"
      :cellWidth="100"
      :durationBarVisible="false"
      :eventBorderRadius="10"
      :rowMarginTop="2"
      :rowMarginBottom="2"
      :days="DayPilot.Date.today().daysInMonth()"
      :startDate="DayPilot.Date.today().firstDayOfMonth()"
      :timeHeaders="[ { groupBy: 'Month' }, { groupBy: 'Day', format: 'd' } ]"
      :floatingEvents="false"
      @timeRangeSelected="onTimeRangeSelected"
      :events="events"
      :resources="resources"
      ref="schedulerRef"
  >
    <template #event="{ event }">
      <div class="event-body">
        <div class="event-title">{{ event.text() }}</div>
        <div class="event-important">
          <label><span class="event-item-desc">Important:</span><input type="checkbox" v-model="event.data.important" @change="onImportantChange(event)" :title="'Mark as important'" /></label>
        </div>
        <div class="event-status">
          <span class="event-item-desc">Status:</span>
          <select v-model="event.data.status" @change="onStatusChange(event)" @mousedown.stop>
            <option value="Not Started">Not Started</option>
            <option value="In Progress">In Progress</option>
            <option value="Completed">Completed</option>
          </select>
        </div>
        <div class="event-attachment">
          <span class="event-item-desc">Attachment:</span>
          <label class="file-label" :title="'Attach a file'">
            <input type="file" @change="onFileChange($event, event)" class="file-input" />
            <svg>
              <use xlink:href="/icons/daypilot.svg#plus-circle"></use>
            </svg>
          </label>
          <span v-if="event.data.attachment">
              <a :href="event.data.attachment.url" target="_blank">{{ event.data.attachment.name }}</a>
            </span>
        </div>
      </div>
    </template>

  </DayPilotScheduler>
</template>

<script setup>
import { DayPilot, DayPilotScheduler } from '@daypilot/daypilot-lite-vue';
import { ref, onMounted } from 'vue';

const events = ref([]);
const resources = ref([]);

const onTimeRangeSelected = async (args) => {
  const scheduler = args.control;
  const modal = await DayPilot.Modal.prompt("Create a new event:", "Event 1");
  scheduler.clearSelection();
  if (modal.canceled) { return; }
  scheduler.events.add({
    start: args.start,
    end: args.end,
    id: DayPilot.guid(),
    resource: args.resource,
    text: modal.result,
    data: {
      completed: false,
      status: 'Not Started',
      attachment: null
    }
  });
};

const onImportantChange = (event) => {
  console.log(`Event ${event.text()} important: ${event.data.important}`);
};

const onStatusChange = (event) => {
  console.log(`Event ${event.text()} status changed to: ${event.data.status}`);
};

const onFileChange = (e, event) => {
  const file = e.target.files[0];
  if (file) {
    const url = URL.createObjectURL(file);
    event.data.attachment = {
      name: file.name,
      url: url
    };
    console.log(`File attached to event "${event.text()}": ${file.name}`);
  }
};

const schedulerRef = ref(null);

const loadEvents = () => {
  events.value = [
    {
      id: 1,
      start: DayPilot.Date.today().firstDayOfMonth().addDays(1),
      end: DayPilot.Date.today().firstDayOfMonth().addDays(4),
      text: "Event 1",
      resource: "R1",
      important: false,
      status: 'Not Started',
      attachment: null
    },
    {
      id: 2,
      start: DayPilot.Date.today().firstDayOfMonth().addDays(2),
      end: DayPilot.Date.today().firstDayOfMonth().addDays(7),
      text: "Event 2",
      resource: "R2",
      important: false,
      status: 'In Progress',
      attachment: null
    }
  ];
};

const loadResources = () => {
  resources.value = [
    { name: "Resource 1", id: "R1" },
    { name: "Resource 2", id: "R2" },
    { name: "Resource 3", id: "R3" }
  ];
};

onMounted(() => {
  loadResources();
  loadEvents();
});
</script>

<style>
.scheduler_default_event_inner {
  background: #ffcc6699;
  border: 1px solid rgba(248, 185, 50, 0.75);
}

</style>
<style scoped>

.event-body {
  padding: 5px;
}

.event-title {
  font-size: 14px;
  font-weight: bold;
  color: #333;
  flex-grow: 1;
  margin-right: 5px;
}

.event-actions input[type="checkbox"] {
  margin-right: 10px;
  cursor: pointer;
}

.file-input {
  display: none;
}

.file-label {
  cursor: pointer;
  font-size: 16px;
}

.file-label svg {
  width: 16px;
  height: 16px;
  color: #0075ff;
}

.event-body select {
  padding: 2px;
  border-radius: 15px;
}

.event-important {
  margin-top: 5px;
}

.event-important label {
  display: flex;
  align-items: center;
  gap: 2px;
}

.event-item-desc {
  width: 70px;
}

.event-status {
  margin-top: 5px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.event-attachment {
  margin-top: 5px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.event-attachment label {
  display: flex;
  align-items: center;
  gap: 5px;
}

.event-attachment a {
  color: #1a73e8;
  text-decoration: none;
}

.event-attachment a:hover {
  text-decoration: underline;
}

.important .event-title {
  color: red;
}

</style>
