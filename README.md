task-manager/
├─ backend/
│  ├─ package.json
│  ├─ .env.example
│  ├─ server.js
│  ├─ models/
│  │  └─ Task.js
│  └─ routes/
│     └─ tasks.js
└─ frontend/
   ├─ package.json
   ├─ public/
   │  └─ index.html
   └─ src/
      ├─ index.js
      ├─ App.js
      ├─ api.js
      ├─ components/
      │  ├─ TaskForm.js
      │  ├─ TaskItem.js
      │  └─ TaskList.js
      └─ styles.css
Backend
backend/package.json
json
Copy code
{
  "name": "task-manager-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.0.0",
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}
backend/.env.example
ini
Copy code
PORT=5000
MONGO_URI=mongodb://localhost:27017/task_manager_db
Rename to .env and put your real MongoDB URI.

backend/models/Task.js
js
Copy code
const mongoose = require('mongoose');

const TaskSchema = new mongoose.Schema({
  title: { type: String, required: true, trim: true },
  description: { type: String, trim: true, default: '' },
  completed: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Task', TaskSchema);
backend/routes/tasks.js
js
Copy code
const express = require('express');
const router = express.Router();
const Task = require('../models/Task');

// GET /api/tasks - list
router.get('/', async (req, res) => {
  try {
    const tasks = await Task.find().sort({ createdAt: -1 });
    res.json(tasks);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// POST /api/tasks - create
router.post('/', async (req, res) => {
  try {
    const { title, description } = req.body;
    if (!title) return res.status(400).json({ message: 'Title required' });
    const task = new Task({ title, description });
    const saved = await task.save();
    res.status(201).json(saved);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// PUT /api/tasks/:id - update
router.put('/:id', async (req, res) => {
  try {
    const { title, description, completed } = req.body;
    const task = await Task.findByIdAndUpdate(
      req.params.id,
      { title, description, completed },
      { new: true }
    );
    if (!task) return res.status(404).json({ message: 'Task not found' });
    res.json(task);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// DELETE /api/tasks/:id - delete
router.delete('/:id', async (req, res) => {
  try {
    const removed = await Task.findByIdAndDelete(req.params.id);
    if (!removed) return res.status(404).json({ message: 'Task not found' });
    res.json({ message: 'Deleted' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

module.exports = router;
backend/server.js
js
Copy code
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const tasksRouter = require('./routes/tasks');
app.use('/api/tasks', tasksRouter);

const PORT = process.env.PORT || 5000;
const MONGO = process.env.MONGO_URI;

mongoose.connect(MONGO, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => {
    console.log('MongoDB connected');
    app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
  })
  .catch(err => {
    console.error('MongoDB connection error:', err);
    process.exit(1);
  });
Frontend (React)
Use create-react-app structure — below are the essential files.

frontend/package.json
json
Copy code
{
  "name": "task-manager-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "PORT=3000 react-scripts start",
    "build": "react-scripts build"
  }
}
frontend/public/index.html
html
Copy code
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Task Manager</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
frontend/src/index.js
js
Copy code
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';
import './styles.css';

createRoot(document.getElementById('root')).render(<App />);
frontend/src/api.js
js
Copy code
const BASE = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

export async function fetchTasks() {
  const res = await fetch(`${BASE}/tasks`);
  return res.json();
}

export async function createTask(data) {
  const res = await fetch(`${BASE}/tasks`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return res.json();
}

export async function updateTask(id, data) {
  const res = await fetch(`${BASE}/tasks/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return res.json();
}

export async function deleteTask(id) {
  const res = await fetch(`${BASE}/tasks/${id}`, { method: 'DELETE' });
  return res.json();
}
frontend/src/App.js
js
Copy code
import React, { useEffect, useState } from 'react';
import { fetchTasks, createTask, updateTask, deleteTask } from './api';
import TaskForm from './components/TaskForm';
import TaskList from './components/TaskList';

export default function App() {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);

  async function load() {
    setLoading(true);
    const data = await fetchTasks();
    setTasks(data);
    setLoading(false);
  }

  useEffect(() => { load(); }, []);

  async function handleCreate(task) {
    const created = await createTask(task);
    setTasks(prev => [created, ...prev]);
  }

  async function handleUpdate(id, fields) {
    const updated = await updateTask(id, fields);
    setTasks(prev => prev.map(t => (t._id === id ? updated : t)));
  }

  async function handleDelete(id) {
    await deleteTask(id);
    setTasks(prev => prev.filter(t => t._id !== id));
  }

  return (
    <div className="container">
      <h1>Task Manager</h1>
      <TaskForm onCreate={handleCreate} />
      {loading ? <p>Loading...</p> : 
        <TaskList tasks={tasks} onUpdate={handleUpdate} onDelete={handleDelete} />
      }
    </div>
  );
}
frontend/src/components/TaskForm.js
js
Copy code
import React, { useState } from 'react';

export default function TaskForm({ onCreate }) {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');

  function submit(e) {
    e.preventDefault();
    if (!title.trim()) return;
    onCreate({ title: title.trim(), description: description.trim() });
    setTitle('');
    setDescription('');
  }

  return (
    <form className="task-form" onSubmit={submit}>
      <input
        placeholder="Task title"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <input
        placeholder="Optional description"
        value={description}
        onChange={e => setDescription(e.target.value)}
      />
      <button type="submit">Add</button>
    </form>
  );
}
frontend/src/components/TaskItem.js
js
Copy code
import React, { useState } from 'react';

export default function TaskItem({ task, onToggle, onDelete, onEdit }) {
  const [editing, setEditing] = useState(false);
  const [title, setTitle] = useState(task.title);
  const [desc, setDesc] = useState(task.description || '');

  function save() {
    onEdit(task._id, { title, description: desc, completed: task.completed });
    setEditing(false);
  }

  return (
    <div className={`task-item ${task.completed ? 'completed' : ''}`}>
      <input
        type="checkbox"
        checked={task.completed}
        onChange={() => onToggle(task._id, { completed: !task.completed })}
      />
      {editing ? (
        <>
          <input value={title} onChange={e => setTitle(e.target.value)} />
          <input value={desc} onChange={e => setDesc(e.target.value)} />
          <button onClick={save}>Save</button>
          <button onClick={() => setEditing(false)}>Cancel</button>
        </>
      ) : (
        <>
          <div className="task-main" onDoubleClick={() => setEditing(true)}>
            <div className="task-title">{task.title}</div>
            {task.description && <div className="task-desc">{task.description}</div>}
          </div>
          <div className="actions">
            <button onClick={() => setEditing(true)}>Edit</button>
            <button onClick={() => onDelete(task._id)}>Delete</button>
          </div>
        </>
      )}
    </div>
  );
}
frontend/src/components/TaskList.js
js
Copy code
import React from 'react';
import TaskItem from './TaskItem';

export default function TaskList({ tasks, onUpdate, onDelete }) {
  return (
    <div className="task-list">
      {tasks.length === 0 && <p>No tasks yet.</p>}
      {tasks.map(task => (
        <TaskItem
          key={task._id}
          task={task}
          onToggle={(id, data) => onUpdate(id, { ...task, ...data })}
          onDelete={onDelete}
          onEdit={(id, fields) => onUpdate(id, fields)}
        />
      ))}
    </div>
  );
}
frontend/src/styles.css
css
Copy code
body { font-family: Arial, Helvetica, sans-serif; background:#f7f7f8; margin:0; padding:20px; }
.container { max-width:800px; margin:0 auto; background:#fff; padding:20px; border-radius:8px; box-shadow:0 4px 20px rgba(0,0,0,.05); }
h1 { margin-top:0; }
.task-form { display:flex; gap:8px; margin-bottom:16px; }
.task-form input { flex:1; padding:8px; border:1px solid #ddd; border-radius:4px; }
.task-form button { padding:8px 12px; border:none; background:#2d8cf0; color:#fff; border-radius:4px; cursor:pointer; }
.task-item { display:flex; align-items:flex-start; gap:10px; padding:10px; border-bottom:1px solid #eee; }
.task-item.completed .task-title { text-decoration:line-through; color:#888; }
.task-main { flex:1; cursor:default; }
.task-title { font-weight:600; }
.task-desc { font-size:0.9rem; color:#666; margin-top:4px; }
.actions button { margin-left:8px; }
