# Daily-routine-planner
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Daily Routine Planner</title>
<style>
  body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background: #f9f9f9; }
  h1 { text-align: center; color: #333; }
  #newTaskForm { display: flex; gap: 10px; margin-bottom: 20px; }
  input, select { padding: 10px; font-size: 16px; flex: 1; }
  button { padding: 10px 20px; background: #007bff; color: white; border: none; cursor: pointer; }
  button:hover { background: #0056b3; }
  ul { list-style: none; padding: 0; }
  li { background: white; padding: 15px; margin-bottom: 10px; border-radius: 5px; display: flex; justify-content: space-between; align-items: center; }
  .task-info { flex-grow: 1; }
  .task-time { font-weight: bold; margin-right: 10px; }
  .actions button { margin-left: 5px; background: #dc3545; }
  .actions button.edit { background: #28a745; }
</style>
</head>
<body>

<h1>My Daily Routine</h1>

<form id="newTaskForm">
  <input type="text" id="taskName" placeholder="Task name" required />
  <input type="time" id="taskTime" required />
  <button type="submit">Add Task</button>
</form>

<ul id="taskList"></ul>

<script>
  let tasks = JSON.parse(localStorage.getItem('dailyRoutineTasks')) || [];

  function saveTasks() {
    localStorage.setItem('dailyRoutineTasks', JSON.stringify(tasks));
  }

  function renderTasks() {
    const taskList = document.getElementById('taskList');
    taskList.innerHTML = '';
    tasks.sort((a,b) => a.time.localeCompare(b.time));
    tasks.forEach((task, index) => {
      const li = document.createElement('li');
      li.innerHTML = `
        <div class="task-info">
          <span class="task-time">${task.time}</span>
          <span>${task.name}</span>
        </div>
        <div class="actions">
          <button class="edit" onclick="editTask(${index})">Edit</button>
          <button onclick="deleteTask(${index})">Delete</button>
          <button onclick="setReminder(${index})">🔔</button>
        </div>
      `;
      taskList.appendChild(li);
    });
  }

  function addTask(name, time) {
    tasks.push({ name, time });
    saveTasks();
    renderTasks();
  }

  function editTask(index) {
    const newName = prompt("Edit task name:", tasks[index].name);
    if (!newName) return;
    const newTime = prompt("Edit task time (HH:MM):", tasks[index].time);
    if (!newTime) return;
    tasks[index] = { name: newName, time: newTime };
    saveTasks();
    renderTasks();
  }

  function deleteTask(index) {
    if (confirm('Delete this task?')) {
      tasks.splice(index, 1);
      saveTasks();
      renderTasks();
    }
  }

  function setReminder(index) {
    if (!('Notification' in window)) {
      alert('This browser does not support notifications.');
      return;
    }
    if (Notification.permission !== 'granted') {
      Notification.requestPermission().then(permission => {
        if (permission === 'granted') {
          scheduleNotification(index);
        }
      });
    } else {
      scheduleNotification(index);
    }
  }

  function scheduleNotification(index) {
    const task = tasks[index];
    const [hours, minutes] = task.time.split(':').map(Number);
    const now = new Date();
    const taskTime = new Date();
    taskTime.setHours(hours, minutes, 0, 0);

    // If task time has passed today, schedule for tomorrow
    if (taskTime <= now) taskTime.setDate(taskTime.getDate() + 1);

    const delay = taskTime.getTime() - now.getTime();

    setTimeout(() => {
      new Notification('Reminder', {
        body: `Time for: ${task.name}`,
        icon: 'https://img.icons8.com/ios-filled/50/000000/alarm.png'
      });
    }, delay);

    alert(`Reminder set for ${task.name} at ${task.time}`);
  }

  document.getElementById('newTaskForm').addEventListener('submit', (e) => {
    e.preventDefault();
    const name = document.getElementById('taskName').value.trim();
    const time = document.getElementById('taskTime').value;
    if (name && time) {
      addTask(name, time);
      e.target.reset();
    }
  });

  renderTasks();
</script>

</body>
</html>
