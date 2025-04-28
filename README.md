import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';

const app = express();


app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect('')
.then(() => console.log('MongoDB Connected'))
.catch(err => console.error(err));

// Student Schema
const studentSchema = new mongoose.Schema({
    name: String,
    age: Number,
    department: String,
});

const Student = mongoose.model('Student', studentSchema);

// Routes

// Create
app.post('/students', async (req, res) => {
    const student = new Student(req.body);
    await student.save();
    res.send(student);
});

// Read
app.get('/students', async (req, res) => {
    const students = await Student.find();
    res.send(students);
});

// Update
app.put('/students/:id', async (req, res) => {
    const student = await Student.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.send(student);
});

// Delete
app.delete('/students/:id', async (req, res) => {
    await Student.findByIdAndDelete(req.params.id);
    res.send({ message: 'Student deleted' });
});

// Start server
app.listen(5001, () => {
    console.log('Server running on http://localhost:5001');
});


// frontend/src/App.js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

function App() {
  const [students, setStudents] = useState([]);
  const [formData, setFormData] = useState({ name: '', age: '', department: '' });
  const [editingId, setEditingId] = useState(null);

  useEffect(() => {
    fetchStudents();
  }, []);

  const fetchStudents = async () => {
    const res = await axios.get('http://localhost:5001/students');
    setStudents(res.data);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (editingId) {
      await axios.put(`http://localhost:5001/students/${editingId}`, formData);
      setEditingId(null);
    } else {
      await axios.post('http://localhost:5001/students', formData);
    }
    setFormData({ name: '', age: '', department: '' });
    fetchStudents();
  };

  const handleEdit = (student) => {
    setFormData({ name: student.name, age: student.age, department: student.department });
    setEditingId(student._id);
  };

  const handleDelete = async (id) => {
    await axios.delete(`http://localhost:5001/students/${id}`);
    fetchStudents();
  };

  return (
    <div style={{ maxWidth: '600px', margin: '50px auto' }}>
      <h2>Student Form</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Name"
          value={formData.name}
          onChange={e => setFormData({ ...formData, name: e.target.value })}
          required
        />
        <input
          type="number"
          placeholder="Age"
          value={formData.age}
          onChange={e => setFormData({ ...formData, age: e.target.value })}
          required
        />
        <input
          type="text"
          placeholder="Department"
          value={formData.department}
          onChange={e => setFormData({ ...formData, department: e.target.value })}
          required
        />
        <button type="submit">Save
        </button>
      </form>

      <h2>Students List</h2>
      <ul>
        {students.map(student => (
          <li key={student._id}>
            <strong>{student.name}</strong> (Age: {student.age}) - Dept: {student.department}
            <br />
            <button onClick={() => handleEdit(student)} style={{ marginRight: '10px', marginTop: '5px' }}>Edit</button>
            <button onClick={() => handleDelete(student._id)} style={{ marginTop: '5px' }}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
