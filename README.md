<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Update Manager</title>
  <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/axios@1.4.0/dist/axios.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/babel-standalone@6.26.0/babel.min.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body>
  <div id="root" class="container mx-auto p-4"></div>
  <script type="text/babel">
    const { useState, useEffect } = React;

    const UpdateManager = () => {
      const [updates, setUpdates] = useState([]);
      const [title, setTitle] = useState('');
      const [date, setDate] = useState('');
      const [description, setDescription] = useState('');
      const [headers, setHeaders] = useState([]);
      const [newHeader, setNewHeader] = useState('');
      const [relatedUpdate, setRelatedUpdate] = useState('');
      const [filter, setFilter] = useState('');
      const [sortBy, setSortBy] = useState('date');
      const [highlight, setHighlight] = useState('');
      const [reminders, setReminders] = useState([]);
      const [newReminder, setNewReminder] = useState('');
      const [todos, setTodos] = useState([]);
      const [newTodo, setNewTodo] = useState('');

      useEffect(() => {
        fetchUpdates();
        fetchReminders();
        fetchTodos();
      }, []);

      const fetchUpdates = async () => {
        try {
          const response = await axios.get('http://localhost:5000/api/updates');
          setUpdates(response.data);
        } catch (error) {
          console.error('Error fetching updates:', error);
        }
      };

      const fetchReminders = async () => {
        try {
          const response = await axios.get('http://localhost:5000/api/reminders');
          setReminders(response.data);
        } catch (error) {
          console.error('Error fetching reminders:', error);
        }
      };

      const fetchTodos = async () => {
        try {
          const response = await axios.get('http://localhost:5000/api/todos');
          setTodos(response.data);
        } catch (error) {
          console.error('Error fetching todos:', error);
        }
      };

      const addUpdate = async () => {
        if (!title || !date || !description) return;
        try {
          const response = await axios.post('http://localhost:5000/api/updates', {
            title,
            date,
            description,
            headers,
            relatedUpdate
          });
          setUpdates([...updates, response.data]);
          setTitle('');
          setDate('');
          setDescription('');
          setHeaders([]);
          setRelatedUpdate('');
        } catch (error) {
          console.error('Error adding update:', error);
        }
      };

      const updateUpdate = async (id, updatedData) => {
        try {
          const response = await axios.put(`http://localhost:5000/api/updates/${id}`, updatedData);
          setUpdates(updates.map(update => update._id === id ? response.data : update));
        } catch (error) {
          console.error('Error updating update:', error);
        }
      };

      const removeUpdate = async (id) => {
        try {
          await axios.delete(`http://localhost:5000/api/updates/${id}`);
          setUpdates(updates.filter(update => update._id !== id));
        } catch (error) {
          console.error('Error removing update:', error);
        }
      };

      const addHeader = () => {
        if (newHeader) {
          setHeaders([...headers, newHeader]);
          setNewHeader('');
        }
      };

      const addReminder = async () => {
        if (!newReminder) return;
        try {
          const response = await axios.post('http://localhost:5000/api/reminders', { text: newReminder, date: new Date() });
          setReminders([...reminders, response.data]);
          setNewReminder('');
        } catch (error) {
          console.error('Error adding reminder:', error);
        }
      };

      const addTodo = async () => {
        if (!newTodo) return;
        try {
          const response = await axios.post('http://localhost:5000/api/todos', { text: newTodo, completed: false });
          setTodos([...todos, response.data]);
          setNewTodo('');
        } catch (error) {
          console.error('Error adding todo:', error);
        }
      };

      const toggleTodo = async (id, completed) => {
        try {
          const response = await axios.put(`http://localhost:5000/api/todos/${id}`, { completed });
          setTodos(todos.map(todo => todo._id === id ? response.data : todo));
        } catch (error) {
          console.error('Error toggling todo:', error);
        }
      };

      const filteredUpdates = updates
        .filter(update => update.title.toLowerCase().includes(filter.toLowerCase()) || update.description.toLowerCase().includes(filter.toLowerCase()))
        .sort((a, b) => sortBy === 'date' ? new Date(b.date) - new Date(a.date) : a.title.localeCompare(b.title));

      return (
        <div className="space-y-6">
          <h1 className="text-3xl font-bold">Update Manager</h1>
          
          {/* Add Update Form */}
          <div className="bg-white p-4 rounded shadow">
            <h2 className="text-xl font-semibold mb-2">Add New Update</h2>
            <input type="text" value={title} onChange={e => setTitle(e.target.value)} placeholder="Title" className="border p-2 mb-2 w-full" />
            <input type="date" value={date} onChange={e => setDate(e.target.value)} className="border p-2 mb-2 w-full" />
            <textarea value={description} onChange={e => setDescription(e.target.value)} placeholder="Description" className="border p-2 mb-2 w-full" />
            <div className="flex mb-2">
              <input type="text" value={newHeader} onChange={e => setNewHeader(e.target.value)} placeholder="Add Header" className="border p-2 flex-grow" />
              <button onClick={addHeader} className="bg-blue-500 text-white p-2 ml-2 rounded">Add Header</button>
            </div
