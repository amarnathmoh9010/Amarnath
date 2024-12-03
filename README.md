//styles/GlobalStyles.js
import { createGlobalStyle } from 'styled-components';

const GlobalStyles = createGlobalStyle`
  body {
    font-family: 'Roboto', sans-serif;
    margin: 0;
    padding: 0;
    background: #f4f4f9;
    color: #333;
  }

  h1, h2, h3, h4, h5, h6 {
    margin: 0;
    padding: 0;
  }

  button {
    cursor: pointer;
    border: none;
    padding: 10px 15px;
    border-radius: 5px;
    background-color: #007BFF;
    color: white;
    font-size: 16px;
    transition: background 0.3s ease;
  }

  button:hover {
    background-color: #0056b3;
  }

  input, textarea {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
`;

export default GlobalStyles;

//components/Modal.jsx
import React from 'react';
import styled from 'styled-components';

const ModalWrapper = styled.div`
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
`;

const ModalContent = styled.div`
  background: white;
  border-radius: 10px;
  padding: 20px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
`;

const Modal = ({ children, onClose }) => (
  <ModalWrapper onClick={onClose}>
    <ModalContent onClick={(e) => e.stopPropagation()}>
      {children}
    </ModalContent>
  </ModalWrapper>
);

export default Modal;
//features/tasks/taskSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  tasks: [],
  filter: 'all', // 'all', 'completed', 'pending', 'overdue'
};

const taskSlice = createSlice({
  name: 'tasks',
  initialState,
  reducers: {
    addTask: (state, action) => {
      state.tasks.push({ id: Date.now(), ...action.payload, completed: false });
    },
    editTask: (state, action) => {
      const index = state.tasks.findIndex(task => task.id === action.payload.id);
      if (index !== -1) state.tasks[index] = { ...state.tasks[index], ...action.payload };
    },
    deleteTask: (state, action) => {
      state.tasks = state.tasks.filter(task => task.id !== action.payload);
    },
    toggleComplete: (state, action) => {
      const task = state.tasks.find(task => task.id === action.payload);
      if (task) task.completed = !task.completed;
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
  },
});

export const { addTask, editTask, deleteTask, toggleComplete, setFilter } = taskSlice.actions;

export default taskSlice.reducer;
//components/TaskDashboard.jsx
import React, { useState } from 'react';
import styled from 'styled-components';
import TaskForm from './TaskForm';
import TaskList from './TaskList';
import TaskFilters from './TaskFilters';

const Wrapper = styled.div`
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
`;

const Header = styled.h1`
  text-align: center;
  margin-bottom: 20px;
`;

const TaskDashboard = () => {
  const [isFormOpen, setFormOpen] = useState(false);

  return (
    <Wrapper>
      <Header>Task Management</Header>
      <TaskFilters />
      <TaskList />
      <button onClick={() => setFormOpen(true)}>Add New Task</button>
      {isFormOpen && (
        <Modal onClose={() => setFormOpen(false)}>
          <TaskForm onClose={() => setFormOpen(false)} />
        </Modal>
      )}
    </Wrapper>
  );
};

export default TaskDashboard;
//components/TaskForm.jsx
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { addTask } from '../features/tasks/taskSlice';

const TaskForm = ({ onClose }) => {
  const [task, setTask] = useState({ title: '', description: '', dueDate: '' });
  const dispatch = useDispatch();

  const handleSubmit = (e) => {
    e.preventDefault();
    if (task.title.trim()) {
      dispatch(addTask(task));
      setTask({ title: '', description: '', dueDate: '' });
      onClose();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <h3>Create Task</h3>
      <input
        type="text"
        placeholder="Task Title"
        value={task.title}
        onChange={(e) => setTask({ ...task, title: e.target.value })}
        required
      />
      <textarea
        placeholder="Task Description"
        value={task.description}
        onChange={(e) => setTask({ ...task, description: e.target.value })}
      ></textarea>
      <input
        type="date"
        value={task.dueDate}
        onChange={(e) => setTask({ ...task, dueDate: e.target.value })}
        required
      />
      <button type="submit">Save Task</button>
      <button type="button" onClick={onClose}>
        Cancel
      </button>
    </form>
  );
};

export default TaskForm;
//App.js
import React from 'react';
import { Provider } from 'react-redux';
import { store } from './store';
import TaskDashboard from './components/TaskDashboard';
import GlobalStyles from './styles/GlobalStyles';

const App = () => (
  <>
    <GlobalStyles />
    <Provider store={store}>
      <TaskDashboard />
    </Provider>
  </>
);

export default App;


