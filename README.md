# React CRUD Guide: Tenant Management App (Beginner Friendly)

This guide walks you through building a **small React application** with full **CRUD (Create, Read, Update, Delete)** operations against a Tenant API.

We will use:
- **Create React App** for bootstrapping
- **Axios** for API requests
- **React Router** for navigation
- Basic CSS (optional Bootstrap / Material UI)
- **Jest + React Testing Library** for tests

---

## 1) Initial Setup

### 1.1 Create a new React app

```bash
npx create-react-app tenant-crud-app
cd tenant-crud-app
```

### 1.2 Install dependencies

```bash
npm install axios react-router-dom
```

Optional UI libraries:

```bash
# Bootstrap option
npm install bootstrap

# OR Material UI option
npm install @mui/material @emotion/react @emotion/styled
```

### 1.3 Start the app

```bash
npm start
```

---

## 2) Recommended Project Structure

Use a simple, scalable structure:

```text
src/
  api/
    tenantService.js
  components/
    TenantList.jsx
    TenantForm.jsx
    AddTenant.jsx
    EditTenant.jsx
    Navbar.jsx
  pages/
    Home.jsx
  styles/
    app.css
    form.css
  App.jsx
  index.js
```

### Why this structure?
- `api/`: all backend calls in one place
- `components/`: reusable UI pieces
- `pages/`: route-level screens
- `styles/`: keeps CSS organized

---

## 3) API Integration with Axios

You shared this endpoint style:

- GET tenants: `https://localhost:7168/api/Tenants?page=1&pageSize=20`
- POST tenant: `https://localhost:7168/api/Tenants`
- GET by id: `/api/Tenants/{id}`
- PUT by id: `/api/Tenants/{id}`
- DELETE by id: `/api/Tenants/{id}`

### 3.1 Create `src/api/tenantService.js`

```js
import axios from 'axios';

const API_BASE_URL = 'https://localhost:7168/api/Tenants';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
    Accept: '*/*',
  },
});

export const getTenants = async (page = 1, pageSize = 20) => {
  const response = await api.get(`?page=${page}&pageSize=${pageSize}`);
  return response.data;
};

export const getTenantById = async (id) => {
  const response = await api.get(`/${id}`);
  return response.data;
};

export const createTenant = async (tenantPayload) => {
  const response = await api.post('', tenantPayload);
  return response.data;
};

export const updateTenant = async (id, tenantPayload) => {
  const response = await api.put(`/${id}`, tenantPayload);
  return response.data;
};

export const deleteTenant = async (id) => {
  const response = await api.delete(`/${id}`);
  return response.data;
};
```

### 3.2 CRUD request examples

```js
// GET list
const data = await getTenants(1, 20);

// POST create
await createTenant({
  name: 'SAFruits',
  address: 'NRT',
  phone: '79999999',
  email: 'sa@mail.cm',
});

// PUT update
await updateTenant('11111111-1111-1111-1111-111111111111', {
  name: 'Kindred Fruit Traders Updated',
  address: 'Guntur, AP, India',
  phone: '+91-90000-00000',
  email: 'ops@kindredfruit.example',
  isActive: true,
});

// DELETE
await deleteTenant('11111111-1111-1111-1111-111111111111');
```

> Note: if your backend uses HTTPS with a local certificate, you may need to trust that certificate locally.

---

## 4) Components (Functional + Hooks)

### 4.1 `TenantList.jsx` (Read + Delete)

```jsx
import React, { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import { deleteTenant, getTenants } from '../api/tenantService';

const TenantList = () => {
  const [tenants, setTenants] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');

  const loadTenants = async () => {
    try {
      setLoading(true);
      const data = await getTenants(1, 20);
      setTenants(Array.isArray(data) ? data : data.items || []);
      setError('');
    } catch (err) {
      setError('Failed to load tenants.');
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  const handleDelete = async (id) => {
    const ok = window.confirm('Are you sure you want to delete this tenant?');
    if (!ok) return;

    try {
      await deleteTenant(id);
      await loadTenants();
    } catch (err) {
      alert('Delete failed.');
      console.error(err);
    }
  };

  useEffect(() => {
    loadTenants();
  }, []);

  if (loading) return <p>Loading tenants...</p>;
  if (error) return <p>{error}</p>;

  return (
    <div>
      <h2>Tenant List</h2>
      <Link to="/add">Add Tenant</Link>
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Phone</th>
            <th>Address</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {tenants.map((tenant) => (
            <tr key={tenant.id}>
              <td>{tenant.name}</td>
              <td>{tenant.email}</td>
              <td>{tenant.phone}</td>
              <td>{tenant.address}</td>
              <td>
                <Link to={`/edit/${tenant.id}`}>Edit</Link>
                <button onClick={() => handleDelete(tenant.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default TenantList;
```

### 4.2 Reusable `TenantForm.jsx` (Create + Update)

```jsx
import React, { useState } from 'react';

const TenantForm = ({ initialValues, onSubmit, submitText = 'Save' }) => {
  const [formData, setFormData] = useState(
    initialValues || {
      name: '',
      address: '',
      phone: '',
      email: '',
      isActive: true,
    }
  );

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value,
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>Name</label>
      <input name="name" value={formData.name} onChange={handleChange} required />

      <label>Address</label>
      <input name="address" value={formData.address} onChange={handleChange} required />

      <label>Phone</label>
      <input name="phone" value={formData.phone} onChange={handleChange} required />

      <label>Email</label>
      <input name="email" type="email" value={formData.email} onChange={handleChange} required />

      <label>
        <input
          name="isActive"
          type="checkbox"
          checked={formData.isActive}
          onChange={handleChange}
        />
        Is Active
      </label>

      <button type="submit">{submitText}</button>
    </form>
  );
};

export default TenantForm;
```

### 4.3 `AddTenant.jsx`

```jsx
import React from 'react';
import { useNavigate } from 'react-router-dom';
import { createTenant } from '../api/tenantService';
import TenantForm from './TenantForm';

const AddTenant = () => {
  const navigate = useNavigate();

  const handleAdd = async (payload) => {
    try {
      await createTenant(payload);
      navigate('/');
    } catch (err) {
      alert('Failed to create tenant');
      console.error(err);
    }
  };

  return (
    <div>
      <h2>Add Tenant</h2>
      <TenantForm onSubmit={handleAdd} submitText="Create" />
    </div>
  );
};

export default AddTenant;
```

### 4.4 `EditTenant.jsx`

```jsx
import React, { useEffect, useState } from 'react';
import { useNavigate, useParams } from 'react-router-dom';
import { getTenantById, updateTenant } from '../api/tenantService';
import TenantForm from './TenantForm';

const EditTenant = () => {
  const { id } = useParams();
  const navigate = useNavigate();
  const [initialValues, setInitialValues] = useState(null);

  useEffect(() => {
    const loadTenant = async () => {
      try {
        const data = await getTenantById(id);
        setInitialValues(data);
      } catch (err) {
        alert('Failed to load tenant');
        console.error(err);
      }
    };

    loadTenant();
  }, [id]);

  const handleUpdate = async (payload) => {
    try {
      await updateTenant(id, payload);
      navigate('/');
    } catch (err) {
      alert('Failed to update tenant');
      console.error(err);
    }
  };

  if (!initialValues) return <p>Loading tenant details...</p>;

  return (
    <div>
      <h2>Edit Tenant</h2>
      <TenantForm initialValues={initialValues} onSubmit={handleUpdate} submitText="Update" />
    </div>
  );
};

export default EditTenant;
```

---

## 5) Routing Setup (React Router)

### `App.jsx`

```jsx
import React from 'react';
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import TenantList from './components/TenantList';
import AddTenant from './components/AddTenant';
import EditTenant from './components/EditTenant';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<TenantList />} />
        <Route path="/add" element={<AddTenant />} />
        <Route path="/edit/:id" element={<EditTenant />} />
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

---

## 6) User Interface Suggestions

### Option A: Basic CSS

Create `src/styles/app.css`:

```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
  background: #f4f6f8;
}

.container {
  max-width: 900px;
  margin: 24px auto;
  background: #fff;
  padding: 16px;
  border-radius: 8px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 10px;
}

button {
  margin-left: 8px;
}
```

### Option B: Bootstrap

In `src/index.js`:

```js
import 'bootstrap/dist/css/bootstrap.min.css';
```

Then use classes like `container`, `table`, `btn btn-primary`, etc.

### Example list rendering snippet

```jsx
{tenants.length === 0 ? (
  <p>No tenants found.</p>
) : (
  tenants.map((tenant) => <div key={tenant.id}>{tenant.name}</div>)
)}
```

---

## 7) State Management (Local vs Global)

### Local state (enough for small CRUD app)
- Use `useState` for form fields and lists.
- Use `useEffect` for loading data when component mounts.

### When to use global state?
Use Context API / Redux when:
- many components need the same data,
- auth/user session is shared app-wide,
- complex state transitions become hard to manage.

For this small app, **local state is usually enough**.

---

## 8) Testing Basics (Jest + React Testing Library)

Create React App already includes testing setup.

### Example test: list component renders heading

`src/components/TenantList.test.jsx`

```jsx
import { render, screen } from '@testing-library/react';
import TenantList from './TenantList';

jest.mock('../api/tenantService', () => ({
  getTenants: jest.fn().mockResolvedValue([]),
  deleteTenant: jest.fn(),
}));

test('renders tenant list heading', async () => {
  render(<TenantList />);
  expect(await screen.findByText(/Tenant List/i)).toBeInTheDocument();
});
```

### Example test: form submits data

`src/components/TenantForm.test.jsx`

```jsx
import { fireEvent, render, screen } from '@testing-library/react';
import TenantForm from './TenantForm';

test('submits form values', () => {
  const onSubmit = jest.fn();
  render(<TenantForm onSubmit={onSubmit} />);

  fireEvent.change(screen.getByLabelText(/Name/i), { target: { value: 'Demo Tenant' } });
  fireEvent.change(screen.getByLabelText(/Address/i), { target: { value: 'Hyderabad' } });
  fireEvent.change(screen.getByLabelText(/Phone/i), { target: { value: '9999999999' } });
  fireEvent.change(screen.getByLabelText(/Email/i), { target: { value: 'demo@mail.com' } });

  fireEvent.click(screen.getByRole('button', { name: /save/i }));

  expect(onSubmit).toHaveBeenCalled();
});
```

Run tests:

```bash
npm test
```

---

## 9) Interview Preparation (React Fundamentals)

Here are common interview questions + short answers:

1. **What is the difference between state and props?**
   - `props` are inputs passed from parent to child (read-only).
   - `state` is local mutable data managed by a component.

2. **What does `useEffect` do?**
   - It handles side effects like API calls, subscriptions, timers, and syncing external data.

3. **When does `useEffect` run?**
   - No dependency array: every render.
   - Empty array `[]`: once after initial render.
   - With dependencies `[x]`: on mount + when `x` changes.

4. **How is functional component lifecycle represented with hooks?**
   - Mount/update logic in `useEffect`; cleanup via return function.

5. **How do you optimize React performance?**
   - Use memoization (`React.memo`, `useMemo`, `useCallback`) where needed.
   - Avoid unnecessary re-renders.
   - Use list keys correctly.
   - Split large components.

6. **What are controlled components?**
   - Form elements controlled by React state (`value` + `onChange`).

7. **What is lifting state up?**
   - Moving shared state to nearest common parent so multiple children can use/update it.

8. **What is the virtual DOM?**
   - React creates an in-memory representation of the UI and efficiently updates only changed parts.

9. **What is the difference between Context API and Redux?**
   - Context: built-in, lightweight global sharing.
   - Redux: robust predictable state container with tooling/middleware.

10. **Why are keys important in lists?**
    - They help React identify which items changed, improving correctness and performance.

---

## 10) Best Practices

- Keep API logic in service files (not directly in many components).
- Reuse form components for Add/Edit pages.
- Validate inputs on client side before submit.
- Handle loading and error states clearly.
- Keep components small and single-purpose.
- Use environment variables for API URLs (e.g., `.env` with `REACT_APP_API_URL`).
- Avoid hardcoding magic strings everywhere.
- Write tests for critical flows (render, submit, API success/failure).
- Use ESLint/Prettier for consistent formatting.

Example `.env`:

```env
REACT_APP_API_BASE_URL=https://localhost:7168/api/Tenants
```

Then in service:

```js
const API_BASE_URL = process.env.REACT_APP_API_BASE_URL;
```

---

## Quick Start Checklist

1. Create app and install dependencies.
2. Add `tenantService.js` for CRUD API methods.
3. Build `TenantList`, `TenantForm`, `AddTenant`, `EditTenant`.
4. Configure routes in `App.jsx`.
5. Add styling (plain CSS or Bootstrap/MUI).
6. Test key components.
7. Improve with validation, pagination, and better UX.

---

## Bonus Improvements You Can Add Later

- Pagination controls (`page`, `pageSize`)
- Search/filter tenants
- Toast notifications for success/error
- Confirmation modal for delete
- Form validation library (Formik + Yup)
- Protected routes + authentication
- Dockerize frontend for easier deployment

If you want, I can also provide a **ready-to-copy full code skeleton** for all files in this structure.
