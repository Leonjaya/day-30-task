# day-30-task
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://jsonplaceholder.typicode.com/users',
});

export const getUsers = () => api.get('/');
export const getUser = (id) => api.get(`/${id}`);
export const addUser = (user) => api.post('/', user);
export const updateUser = (id, user) => api.put(`/${id}`, user);
export const deleteUser = (id) => api.delete(`/${id}`);

export default api;
import React, { createContext, useState, useEffect } from 'react';
import { getUsers, addUser, updateUser, deleteUser } from '../api';

const UserContext = createContext();

export const UserProvider = ({ children }) => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    const response = await getUsers();
    setUsers(response.data);
  };

  const createUser = async (user) => {
    const response = await addUser(user);
    setUsers([...users, response.data]);
  };

  const editUser = async (id, updatedUser) => {
    const response = await updateUser(id, updatedUser);
    setUsers(users.map(user => user.id === id ? response.data : user));
  };

  const removeUser = async (id) => {
    await deleteUser(id);
    setUsers(users.filter(user => user.id !== id));
  };

  return (
    <UserContext.Provider value={{ users, createUser, editUser, removeUser }}>
      {children}
    </UserContext.Provider>
  );
};

export default UserContext;
import React, { useContext } from 'react';
import UserContext from '../context/UserContext';

const UserList = ({ onEdit }) => {
  const { users, removeUser } = useContext(UserContext);

  return (
    <div>
      <h2>User List</h2>
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>
                <button onClick={() => onEdit(user)}>Edit</button>
                <button onClick={() => removeUser(user.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default UserList;
import React, { useState, useEffect, useContext } from 'react';
import UserContext from '../context/UserContext';

const UserForm = ({ userToEdit, onClear }) => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const { createUser, editUser } = useContext(UserContext);

  useEffect(() => {
    if (userToEdit) {
      setName(userToEdit.name);
      setEmail(userToEdit.email);
    } else {
      setName('');
      setEmail('');
    }
  }, [userToEdit]);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (userToEdit) {
      editUser(userToEdit.id, { name, email });
    } else {
      createUser({ name, email });
    }
    onClear();
  };

  return (
    <div>
      <h2>{userToEdit ? 'Edit User' : 'Add User'}</h2>
      <form onSubmit={handleSubmit}>
        <div>
          <label>Name:</label>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />
        </div>
        <div>
          <label>Email:</label>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
        </div>
        <button type="submit">{userToEdit ? 'Update User' : 'Add User'}</button>
        {userToEdit && <button onClick={onClear}>Cancel</button>}
      </form>
    </div>
  );
};

export default UserForm;
import React, { useState } from 'react';
import UserList from './UserList';
import UserForm from './UserForm';

const Main = () => {
  const [userToEdit, setUserToEdit] = useState(null);

  const handleEdit = (user) => {
    setUserToEdit(user);
  };

  const clearEdit = () => {
    setUserToEdit(null);
  };

  return (
    <div>
      <UserForm userToEdit={userToEdit} onClear={clearEdit} />
      <UserList onEdit={handleEdit} />
    </div>
  );
};

export default Main;
body {
  font-family: Arial, sans-serif;
  padding: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

button {
  padding: 5px 10px;
  margin-right: 5px;
  cursor: pointer;
}

form {
  margin-bottom: 20px;
}

input {
  padding: 5px;
  margin-right: 10px;
}
import React from 'react';
import { UserProvider } from './context/UserContext';
import Main from './components/Main';
import './styles.css';

function App() {
  return (
    <UserProvider>
      <Main />
    </UserProvider>
  );
}

export default App;
