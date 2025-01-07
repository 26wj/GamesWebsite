# Planner-<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Zaki's Planner</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      background: linear-gradient(135deg, #f4f4f9, #dfe6f4);
      color: #333;
    }

    header {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 1em;
      background: #6200ea;
      color: white;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      position: relative;
    }

    .task-count {
      position: absolute;
      right: 20px;
      font-size: 1em;
      color: red;
    }

    main {
      display: flex;
      padding: 1em;
      gap: 1em;
    }

    #calendar, #tasks {
      flex: 1;
      background: white;
      padding: 1em;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
      overflow: hidden;
    }

    #calendar-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 0.5em;
    }

    .day {
      padding: 0.5em;
      background: #e0e0e0;
      text-align: center;
      border-radius: 4px;
      cursor: pointer;
      transition: all 0.3s ease;
    }

    .day:hover {
      background: #9575cd;
      color: white;
      transform: scale(1.1);
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }

    .active {
      background: #6200ea;
      color: white;
    }

    #task-header {
      font-size: 1.2em;
      font-weight: bold;
      margin-bottom: 1em;
    }

    #task-list {
      list-style: none;
      padding: 0;
    }

    #task-list li {
      padding: 0.5em;
      margin: 0.5em 0;
      border-radius: 6px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      animation: fadeIn 0.3s ease;
    }

    #task-list li button {
      background: #ff5252;
      color: white;
      border: none;
      border-radius: 4px;
      padding: 0.3em 0.7em;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    #task-list li button:hover {
      background: #d32f2f;
    }

    #add-task-btn {
      background: #03dac5;
      color: white;
      border: none;
      padding: 0.5em 1em;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 1em;
      transition: all 0.3s ease;
    }

    #add-task-btn:hover {
      background: #018786;
    }

    #search-bar {
      margin-bottom: 1em;
      padding: 0.5em;
      width: 100%;
      border-radius: 6px;
      border: 1px solid #ccc;
    }

    .low-priority {
      background-color: #81c784;
    }

    .medium-priority {
      background-color: #ffeb3b;
    }

    .high-priority {
      background-color: #f44336;
    }

    /* New Styles for Task Form */
    #task-form select {
      padding: 0.5em;
      border-radius: 6px;
      border: 1px solid #ccc;
      margin-bottom: 1em;
    }

    #task-form input[type="text"] {
      padding: 0.5em;
      border-radius: 6px;
      border: 1px solid #ccc;
      width: 100%;
      margin-bottom: 1em;
    }

    @keyframes fadeIn {
      from {
        opacity: 0;
        transform: translateY(-10px);
      }
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }

    /* Animation for task form */
    #task-form {
      transform: translateY(-20px);
      opacity: 0;
      transition: opacity 0.5s, transform 0.5s ease-in-out;
    }

    #task-form.visible {
      transform: translateY(0);
      opacity: 1;
    }
  </style>
</head>
<body>
  <header>
    <h1>Zaki's Planner</h1>
    <div class="task-count" id="undone-tasks">0 undone tasks</div>
  </header>
  <main>
    <section id="calendar">
      <h2>Calendar</h2>
      <div id="calendar-grid"></div>
    </section>
    <section id="tasks">
      <input type="text" id="search-bar" placeholder="Search tasks..." />
      <div id="task-header">Select a day to view your plan</div>
      <ul id="task-list"></ul>
      <button id="add-task-btn" style="display: none;">+ Add Task</button>

      <!-- New Task Form -->
      <div id="task-form" style="display: none;">
        <input type="text" id="task-name" placeholder="Task name" />
        <select id="priority-select">
          <option value="low">Low Priority</option>
          <option value="medium">Medium Priority</option>
          <option value="high">High Priority</option>
        </select>
        <select id="person-select">
          <option value="self">Self</option>
          <option value="person1">Person 1</option>
          <option value="person2">Person 2</option>
        </select>
        <button id="save-task-btn">Save Task</button>
      </div>
      <button id="export-to-excel">Export Data to Excel</button>
    </section>
  </main>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const calendarGrid = document.getElementById('calendar-grid');
      const taskHeader = document.getElementById('task-header');
      const taskList = document.getElementById('task-list');
      const addTaskBtn = document.getElementById('add-task-btn');
      const taskForm = document.getElementById('task-form');
      const saveTaskBtn = document.getElementById('save-task-btn');
      const searchBar = document.getElementById('search-bar');
      const undoneTasksCount = document.getElementById('undone-tasks');

      let currentFileData = {}; // Stores task data for Excel export

      // Retrieve tasks from localStorage (or use sample data)
      const getTasksForDay = (day) => {
        const tasks = localStorage.getItem(`tasks-${day}`);
        return tasks ? JSON.parse(tasks) : [];
      };

      const saveTasksForDay = (day, tasks) => {
        localStorage.setItem(`tasks-${day}`, JSON.stringify(tasks));
      };

      // Populate the calendar
      const daysInMonth = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
      for (let day = 1; day <= daysInMonth; day++) {
        const dayCell = document.createElement('div');
        dayCell.textContent = day;
        dayCell.classList.add('day');
        dayCell.dataset.day = day;

        dayCell.addEventListener('click', () => {
          const selectedDay = dayCell.dataset.day;
          taskHeader.textContent = `Tasks for Day ${selectedDay}`;
          const tasks = getTasksForDay(selectedDay);
          updateTaskList(selectedDay, tasks);
          addTaskBtn.style.display = 'block';
          addTaskBtn.dataset.day = selectedDay;

          const undoneTasks = tasks.filter(task => !task.completed).length;
          undoneTasksCount.textContent = `${undoneTasks} undone tasks`;

          document.querySelectorAll('.day').forEach(dayElement => {
            dayElement.classList.remove('active');
          });
          dayCell.classList.add('active');
        });

        calendarGrid.appendChild(dayCell);
      }

      const updateTaskList = (day, tasks) => {
        taskList.innerHTML = '';
        tasks.forEach((task, index) => {
          const taskItem = document.createElement('li');
          taskItem.textContent = task.name;
          taskItem.classList.add(task.priority);

          if (task.completed) {
            taskItem.style.textDecoration = 'line-through';
          }

          const deleteBtn = document.createElement('button');
          deleteBtn.textContent = 'Delete';
          deleteBtn.addEventListener('click', () => {
            tasks.splice(index, 1);
            saveTasksForDay(day, tasks);
            updateTaskList(day, tasks);
          });

          const completeBtn = document.createElement('button');
          completeBtn.textContent = 'Complete';
          completeBtn.addEventListener('click', () => {
            task.completed = !task.completed;
            saveTasksForDay(day, tasks);
            updateTaskList(day, tasks);
          });

          taskItem.appendChild(deleteBtn);
          taskItem.appendChild(completeBtn);
          taskList.appendChild(taskItem);
        });
      };

      addTaskBtn.addEventListener('click', () => {
        const selectedDay = addTaskBtn.dataset.day;
        taskForm.classList.add('visible');
        taskForm.style.display = 'block';
        taskForm.dataset.day = selectedDay;
      });

      saveTaskBtn.addEventListener('click', () => {
        const selectedDay = taskForm.dataset.day;
        const tasks = getTasksForDay(selectedDay);

        const newTaskName = document.getElementById('task-name').value;
        const newTaskPriority = document.getElementById('priority-select').value;
        const newTaskPerson = document.getElementById('person-select').value;

        if (newTaskName && newTaskPriority) {
          const newTask = {
            name: newTaskName,
            priority: newTaskPriority,
            completed: false,
            person: newTaskPerson
          };

          tasks.push(newTask);
          saveTasksForDay(selectedDay, tasks);
          updateTaskList(selectedDay, tasks);

          // Clear and hide the form
          taskForm.classList.remove('visible');
          taskForm.style.display = 'none';
          document.getElementById('task-name').value = '';
        }
      });

      // Export to Excel
      const exportToExcel = () => {
        const wb = XLSX.utils.book_new();
        const sheetData = [];

        // Iterate over all days
        for (let day = 1; day <= daysInMonth; day++) {
          const tasks = getTasksForDay(day);
          if (tasks.length > 0) {
            tasks.forEach(task => {
              sheetData.push([
                `Day ${day}`,
                task.name,
                task.priority,
                task.completed ? 'Completed' : 'Pending',
                task.person
              ]);
            });
          }
        }

        const ws = XLSX.utils.aoa_to_sheet(sheetData);
        XLSX.utils.book_append_sheet(wb, ws, 'Planner Data');
        XLSX.writeFile(wb, 'planner_data.xlsx');
      };

      document.getElementById('export-to-excel').addEventListener('click', exportToExcel);
    });
  </script>
</body>
</html>
