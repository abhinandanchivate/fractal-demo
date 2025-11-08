Excellent ✅ — perfect choice.
We’ll go safely and clearly in **3 structured parts** so you can copy them straight into your project folder.

---

### 🧩 PART 1 — Core & Auth

This includes:

* Folder setup (`src/app`, `src/routes`, `src/theme`, `src/utils`)
* Redux store
* Axios config + localStorage helpers
* React Router setup
* Light MUI theme
* Auth slice + service
* Login / Register pages

---

#### 📁 `src/app/store.js`

```js
import { configureStore } from '@reduxjs/toolkit';
import authReducer from '../features/auth/AuthSlice';
import groupsReducer from '../features/groups/GroupsSlice';
import sessionsReducer from '../features/sessions/SessionsSlice';
import documentsReducer from '../features/documents/DocumentsSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    groups: groupsReducer,
    sessions: sessionsReducer,
    documents: documentsReducer,
  },
});

export default store;
```

---

#### 📁 `src/utils/storage.js`

```js
export const saveToken = (token) => {
  localStorage.setItem('access_token', token);
};

export const getToken = () => localStorage.getItem('access_token');

export const clearToken = () => {
  localStorage.removeItem('access_token');
};
```

---

#### 📁 `src/utils/axiosConfig.js`

```js
import axios from 'axios';
import { getToken } from './storage';

const instance = axios.create({
  baseURL: 'http://127.0.0.1:8000/api',
});

instance.interceptors.request.use((config) => {
  const token = getToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

export default instance;
```

---

#### 📁 `src/theme/muiTheme.js`

```js
import { createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    mode: 'light',
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#ff9800',
    },
  },
});

export default theme;
```

---

#### 📁 `src/routes/PrivateRoute.jsx`

```jsx
import { Navigate, Outlet } from 'react-router-dom';
import { useSelector } from 'react-redux';

export default function PrivateRoute() {
  const { token } = useSelector((s) => s.auth);
  return token ? <Outlet /> : <Navigate to="/login" replace />;
}
```

---

#### 📁 `src/routes/AppRouter.jsx`

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { ThemeProvider } from '@mui/material/styles';
import { CssBaseline } from '@mui/material';
import theme from '../theme/muiTheme';
import Navbar from '../features/layout/Navbar';
import Footer from '../features/layout/Footer';
import LandingPage from '../features/layout/LandingPage';
import LoginPage from '../features/auth/LoginPage';
import RegisterPage from '../features/auth/RegisterPage';
import ExploreGroups from '../features/groups/ExploreGroups';
import MyGroups from '../features/groups/MyGroups';
import AdminGroups from '../features/groups/AdminGroups';
import AdminGroupView from '../features/groups/AdminGroupView';
import PrivateRoute from './PrivateRoute';

export default function AppRouter() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <BrowserRouter>
        <Navbar />
        <Routes>
          <Route path="/" element={<LandingPage />} />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />
          <Route element={<PrivateRoute />}>
            <Route path="/explore" element={<ExploreGroups />} />
            <Route path="/my-groups" element={<MyGroups />} />
            <Route path="/admin-groups" element={<AdminGroups />} />
            <Route path="/admin-groups/:id" element={<AdminGroupView />} />
          </Route>
        </Routes>
        <Footer />
      </BrowserRouter>
    </ThemeProvider>
  );
}
```

---

#### 📁 `src/features/auth/AuthService.js`

```js
import axios from '../../utils/axiosConfig';

const register = (data) => axios.post('/auth/register/', data).then((res) => res.data);
const login = (data) => axios.post('/auth/login/', data).then((res) => res.data);

export default { register, login };
```

---

#### 📁 `src/features/auth/AuthSlice.js`

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import AuthService from './AuthService';
import { saveToken, getToken, clearToken } from '../../utils/storage';

export const register = createAsyncThunk('auth/register', async (data) => {
  await AuthService.register(data);
});

export const login = createAsyncThunk('auth/login', async (data) => {
  const res = await AuthService.login(data);
  saveToken(res.access);
  return res.access;
});

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    token: getToken(),
  },
  reducers: {
    logout: (state) => {
      clearToken();
      state.token = null;
    },
  },
  extraReducers: (builder) => {
    builder.addCase(login.fulfilled, (state, action) => {
      state.token = action.payload;
    });
  },
});

export const { logout } = authSlice.actions;
export default authSlice.reducer;
```

---

#### 📁 `src/features/auth/LoginPage.jsx`

```jsx
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { login } from './AuthSlice';
import { useNavigate, Link } from 'react-router-dom';
import { Container, TextField, Button, Typography, Box, Snackbar, Alert } from '@mui/material';

export default function LoginPage() {
  const [form, setForm] = useState({ username: '', password: '' });
  const [open, setOpen] = useState(false);
  const dispatch = useDispatch();
  const navigate = useNavigate();

  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await dispatch(login(form)).unwrap();
      navigate('/explore');
    } catch {
      setOpen(true);
    }
  };

  return (
    <Container maxWidth="xs" sx={{ mt: 8 }}>
      <Typography variant="h5" gutterBottom>Login</Typography>
      <Box component="form" onSubmit={handleSubmit}>
        <TextField fullWidth label="Username" name="username" margin="normal" onChange={handleChange} />
        <TextField fullWidth label="Password" name="password" type="password" margin="normal" onChange={handleChange} />
        <Button type="submit" variant="contained" fullWidth sx={{ mt: 2 }}>Login</Button>
      </Box>
      <Typography sx={{ mt: 2 }}>
        Don’t have an account? <Link to="/register">Register</Link>
      </Typography>
      <Snackbar open={open} onClose={() => setOpen(false)} autoHideDuration={3000}>
        <Alert severity="error" sx={{ width: '100%' }}>Invalid credentials</Alert>
      </Snackbar>
    </Container>
  );
}
```

---

#### 📁 `src/features/auth/RegisterPage.jsx`

```jsx
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { register } from './AuthSlice';
import { useNavigate, Link } from 'react-router-dom';
import { Container, TextField, Button, Typography, Box, Snackbar, Alert } from '@mui/material';

export default function RegisterPage() {
  const [form, setForm] = useState({ username: '', email: '', password: '' });
  const [open, setOpen] = useState(false);
  const dispatch = useDispatch();
  const navigate = useNavigate();

  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await dispatch(register(form)).unwrap();
      navigate('/login');
    } catch {
      setOpen(true);
    }
  };

  return (
    <Container maxWidth="xs" sx={{ mt: 8 }}>
      <Typography variant="h5" gutterBottom>Register</Typography>
      <Box component="form" onSubmit={handleSubmit}>
        <TextField fullWidth label="Username" name="username" margin="normal" onChange={handleChange} />
        <TextField fullWidth label="Email" name="email" type="email" margin="normal" onChange={handleChange} />
        <TextField fullWidth label="Password" name="password" type="password" margin="normal" onChange={handleChange} />
        <Button type="submit" variant="contained" fullWidth sx={{ mt: 2 }}>Register</Button>
      </Box>
      <Typography sx={{ mt: 2 }}>
        Already have an account? <Link to="/login">Login</Link>
      </Typography>
      <Snackbar open={open} onClose={() => setOpen(false)} autoHideDuration={3000}>
        <Alert severity="error" sx={{ width: '100%' }}>Registration failed</Alert>
      </Snackbar>
    </Container>
  );
}
```

---

That’s **Part 1** ✅ — base infrastructure + authentication ready.
Next, we’ll add **Part 2 → Groups + Admin + Layout** (Explore, MyGroups, AdminGroups, Navbar/Footer, Landing Page).

Would you like me to post **Part 2** now?
