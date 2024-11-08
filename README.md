
Q1. Create MongoDB Schema (server/models/Todo.js)
Design Todo schema with fields (title, description, status, etc.)
Create MongoDB model with validation rules
	
const mongoose = require('mongoose');
const TodoSchema = new mongoose.Schema({
    title: {
        type: String,
        required: true
    },
    description: {
        type: String,
        required: false
    },
    completed: {
        type: Boolean,
        default: false
    },
    createdAt: {
        type: Date,
        default: Date.now
    }
});
module.exports = mongoose.model('Todo', TodoSchema);

Q2. Create Server (server/server.js)

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection 
mongoose.connect(   mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority, { 
useNewUrlParser: true, 
useUnifiedTopology: true }) 
.then(() => console.log('MongoDB Atlas connected successfully'))
 .catch((err) => console.error('MongoDB connection error:', err));

// Routes
const todoRoutes = require('./routes/todos');
app.use('/api/todos', todoRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});


Q3: How do you implement a route to retrieve and create todos in a Todo application?

Get all todos: Use the GET method to retrieve all todos from the database, sorting by creation date.

const router = require('express').Router();
const Todo = require('../models/Todo');

// Get all todos
router.get('/', async (req, res) => {
    try {
        const todos = await Todo.find().sort({ createdAt: -1 });
        res.json(todos);
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

Create a new todo: Use the POST method to create a new todo by accepting title and description from the request body and saving it to the database.

// Create todo
router.post('/', async (req, res) => {
    const todo = new Todo({
        title: req.body.title,
        description: req.body.description
    });

    try {
        const newTodo = await todo.save();
        res.status(201).json(newTodo);
    } catch (err) {
        res.status(400).json({ message: err.message });
    }
});


Q4: How do you implement update and delete routes for a Todo application with error handling?

Update a todo: Use the PATCH method to update a todo by id. Check if fields such as title, description, or completed are present in the request, and update accordingly.

// Update todo
router.patch('/:id', async (req, res) => {
    try {
        const todo = await Todo.findById(req.params.id);
        if (req.body.title) todo.title = req.body.title;
        if (req.body.description) todo.description = req.body.description;
        if (req.body.completed !== undefined) todo.completed = req.body.completed;
        const updatedTodo = await todo.save();
        res.json(updatedTodo);
    } catch (err) {
        res.status(400).json({ message: err.message });
    }
});


Delete a todo: Use the DELETE method to remove a todo by id. Return a success message upon deletion.

// Delete todo
router.delete('/:id', async (req, res) => {
    try {
        await Todo.findByIdAndDelete(req.params.id);
        res.json({ message: 'Todo deleted' });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

module.exports = router;





Q5. TodoForm.js (handles the form for adding and updating todos):
import React from 'react';
import { TextField, Button } from '@material-ui/core';
const TodoForm = ({ title, description, setTitle, setDescription, handleSubmit, editing }) => (
    <form onSubmit={handleSubmit}>
        <TextField
            fullWidth
            label="Todo Title"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            margin="normal"
        />
        <TextField
            fullWidth
            label="Description"
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            margin="normal"
        />
        <Button
            type="submit"
            variant="contained"
            color="primary"
            style={{ marginTop: '1rem' }}
        >
            {editing ? 'Update Todo' : 'Add Todo'}
        </Button>
    </form>
);
export default TodoForm;
Q6. TodoItem.js (handles the display and actions for individual todos):
import React from 'react';
import { ListItem, ListItemText, ListItemSecondaryAction, IconButton, Checkbox } from '@material-ui/core';
import { Delete, Edit } from '@material-ui/icons';

const TodoItem = ({ todo, toggleComplete, deleteTodo, startEditing }) => (
    <ListItem>
        <Checkbox
            checked={todo.completed}
            onChange={() => toggleComplete(todo._id, todo.completed)}
        />
        <ListItemText
            primary={todo.title}
            secondary={todo.description}
            style={{
                textDecoration: todo.completed ? 'line-through' : 'none'
            }}
        />
        <ListItemSecondaryAction>
            <IconButton edge="end" onClick={() => startEditing(todo)}>
                <Edit />
            </IconButton>
            <IconButton edge="end" onClick={() => deleteTodo(todo._id)}>
                <Delete />
            </IconButton>
        </ListItemSecondaryAction>
    </ListItem>
);

export default TodoItem;
