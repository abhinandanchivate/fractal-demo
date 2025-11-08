Perfect 👏 — here’s **Part 3 (final)**: all session, document, and comment functionality.
When you add this section to your `src/features/` folder beside the files from Parts 1 & 2, the app will be complete and runnable.

---

### 📁 `src/features/sessions/SessionsService.js`

```js
import axios from '../../utils/axiosConfig';
export const getSessions = (groupId, status) =>
  axios
    .get(`/sessions/`, { params: { group: groupId, status } })
    .then((r) => r.data);
export const createSession = (data) =>
  axios.post('/sessions/', data).then((r) => r.data);
export const deleteSession = (id) =>
  axios.delete(`/sessions/${id}/`).then((r) => r.data);
export default { getSessions, createSession, deleteSession };
```

---

### 📁 `src/features/sessions/SessionsSlice.js`

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import SessionsService from './SessionsService';

export const fetchSessions = createAsyncThunk(
  'sessions/fetch',
  ({ groupId, status }) => SessionsService.getSessions(groupId, status)
);
export const addSession = createAsyncThunk(
  'sessions/add',
  (data) => SessionsService.createSession(data)
);
export const removeSession = createAsyncThunk(
  'sessions/delete',
  (id) => SessionsService.deleteSession(id)
);

const slice = createSlice({
  name: 'sessions',
  initialState: { sessions: [], loading: false },
  reducers: {},
  extraReducers: (b) => {
    b.addCase(fetchSessions.fulfilled, (s, a) => {
      s.sessions = a.payload;
    });
  },
});
export default slice.reducer;
```

---

### 📁 `src/features/sessions/SessionsTable.jsx`

```jsx
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchSessions, removeSession } from './SessionsSlice';
import {
  Table, TableBody, TableCell, TableHead,
  TableRow, Button, Typography, Box
} from '@mui/material';

export default function SessionsTable({ groupId, status }) {
  const dispatch = useDispatch();
  const { sessions } = useSelector((s) => s.sessions);
  useEffect(() => {
    dispatch(fetchSessions({ groupId, status }));
  }, [dispatch, groupId, status]);

  const handleDelete = (id) => dispatch(removeSession(id));

  return (
    <Box sx={{ mt: 3 }}>
      <Typography variant="h6" sx={{ mb: 1 }}>
        {status === 'active' ? 'Active Sessions' : 'Completed Sessions'}
      </Typography>
      <Table size="small">
        <TableHead>
          <TableRow>
            <TableCell>Title</TableCell>
            <TableCell>Description</TableCell>
            <TableCell>Start</TableCell>
            <TableCell>End</TableCell>
            <TableCell>Action</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {sessions.map((s) => (
            <TableRow key={s.id}>
              <TableCell>{s.title}</TableCell>
              <TableCell>{s.description}</TableCell>
              <TableCell>{s.start_time}</TableCell>
              <TableCell>{s.end_time}</TableCell>
              <TableCell>
                <Button color="error" onClick={() => handleDelete(s.id)}>
                  Delete
                </Button>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </Box>
  );
}
```

---

### 📁 `src/features/documents/DocumentsService.js`

```js
import axios from '../../utils/axiosConfig';
const getDocuments = () => axios.get('/documents/').then((r) => r.data);
const uploadDocument = (formData) =>
  axios.post('/documents/', formData, {
    headers: { 'Content-Type': 'multipart/form-data' },
  }).then((r) => r.data);
const approveDocument = (id) =>
  axios.post(`/documents/${id}/approve/`).then((r) => r.data);
const deleteDocument = (id) =>
  axios.delete(`/documents/${id}/`).then((r) => r.data);
export default { getDocuments, uploadDocument, approveDocument, deleteDocument };
```

---

### 📁 `src/features/documents/DocumentsSlice.js`

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import DocumentsService from './DocumentsService';

export const fetchDocuments = createAsyncThunk('docs/fetch', DocumentsService.getDocuments);
export const uploadDocument = createAsyncThunk('docs/upload', DocumentsService.uploadDocument);
export const approveDocument = createAsyncThunk('docs/approve', DocumentsService.approveDocument);
export const deleteDocument = createAsyncThunk('docs/delete', DocumentsService.deleteDocument);

const slice = createSlice({
  name: 'documents',
  initialState: { documents: [] },
  reducers: {},
  extraReducers: (b) => {
    b.addCase(fetchDocuments.fulfilled, (s, a) => { s.documents = a.payload; });
  },
});
export default slice.reducer;
```

---

### 📁 `src/features/documents/DocumentsTable.jsx`

```jsx
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchDocuments, approveDocument, deleteDocument } from './DocumentsSlice';
import {
  Table, TableBody, TableCell, TableHead,
  TableRow, Button, Typography, Box
} from '@mui/material';
import DocumentCommentsDialog from './DocumentCommentsDialog';

export default function DocumentsTable({ admin }) {
  const dispatch = useDispatch();
  const { documents } = useSelector((s) => s.documents);
  const [open, setOpen] = React.useState(false);
  const [docId, setDocId] = React.useState(null);

  useEffect(() => { dispatch(fetchDocuments()); }, [dispatch]);

  const handleApprove = (id) => dispatch(approveDocument(id));
  const handleDelete = (id) => dispatch(deleteDocument(id));
  const handleComments = (id) => { setDocId(id); setOpen(true); };

  return (
    <Box sx={{ mt: 4 }}>
      <Typography variant="h6">Documents</Typography>
      <Table size="small">
        <TableHead>
          <TableRow>
            <TableCell>Title</TableCell>
            <TableCell>Uploaded By</TableCell>
            <TableCell>Approved</TableCell>
            <TableCell>Actions</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {documents.map((d) => (
            <TableRow key={d.id}>
              <TableCell>{d.title}</TableCell>
              <TableCell>{d.uploaded_by?.username}</TableCell>
              <TableCell>{d.approved ? 'Yes' : 'No'}</TableCell>
              <TableCell>
                <Button onClick={() => handleComments(d.id)}>Comments</Button>
                {admin && !d.approved && (
                  <Button onClick={() => handleApprove(d.id)}>Approve</Button>
                )}
                {admin && (
                  <Button color="error" onClick={() => handleDelete(d.id)}>Delete</Button>
                )}
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
      {open && <DocumentCommentsDialog open={open} onClose={() => setOpen(false)} docId={docId} />}
    </Box>
  );
}
```

---

### 📁 `src/features/documents/DocumentCommentsDialog.jsx`

```jsx
import React, { useEffect, useState } from 'react';
import {
  Dialog, DialogTitle, DialogContent, DialogActions,
  TextField, Button, List, ListItem, ListItemText
} from '@mui/material';
import axios from '../../utils/axiosConfig';

export default function DocumentCommentsDialog({ open, onClose, docId }) {
  const [comments, setComments] = useState([]);
  const [text, setText] = useState('');

  const loadComments = async () => {
    const res = await axios.get(`/document-comments/?document=${docId}`);
    setComments(res.data);
  };
  useEffect(() => { if (open) loadComments(); }, [open]);

  const handleAdd = async () => {
    await axios.post('/document-comments/', { document: docId, comment: text });
    setText(''); loadComments();
  };
  const handleDelete = async (id) => {
    await axios.delete(`/document-comments/${id}/`);
    loadComments();
  };

  return (
    <Dialog open={open} onClose={onClose} fullWidth maxWidth="sm">
      <DialogTitle>Comments</DialogTitle>
      <DialogContent>
        <List>
          {comments.map((c) => (
            <ListItem
              key={c.id}
              secondaryAction={
                <Button color="error" onClick={() => handleDelete(c.id)}>Delete</Button>
              }
            >
              <ListItemText
                primary={c.user?.username}
                secondary={c.comment}
              />
            </ListItem>
          ))}
        </List>
        <TextField
          fullWidth
          label="Add comment"
          value={text}
          onChange={(e) => setText(e.target.value)}
          sx={{ mt: 2 }}
        />
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose}>Close</Button>
        <Button onClick={handleAdd} variant="contained">Post</Button>
      </DialogActions>
    </Dialog>
  );
}
```

---

### 📁 `src/features/groups/AdminGroupView.jsx`

```jsx
import React, { useState } from 'react';
import { Box, TextField, Button, Typography } from '@mui/material';
import { useDispatch } from 'react-redux';
import { addSession } from '../sessions/SessionsSlice';
import SessionsTable from '../sessions/SessionsTable';
import DocumentsTable from '../documents/DocumentsTable';

export default function AdminGroupView() {
  const dispatch = useDispatch();
  const [form, setForm] = useState({ title: '', description: '', start_time: '', end_time: '' });
  const groupId = window.location.pathname.split('/').pop();

  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });
  const handleSubmit = async (e) => {
    e.preventDefault();
    await dispatch(addSession({ ...form, group: groupId }));
  };

  return (
    <Box p={3}>
      <Typography variant="h6" gutterBottom>Create Session</Typography>
      <Box component="form" onSubmit={handleSubmit}>
        <TextField label="Title" name="title" fullWidth margin="dense" onChange={handleChange} />
        <TextField label="Description" name="description" fullWidth margin="dense" onChange={handleChange} />
        <TextField label="Start Time" name="start_time" type="datetime-local" fullWidth margin="dense" onChange={handleChange} />
        <TextField label="End Time" name="end_time" type="datetime-local" fullWidth margin="dense" onChange={handleChange} />
        <Button type="submit" variant="contained" sx={{ mt: 1 }}>Create</Button>
      </Box>

      <SessionsTable groupId={groupId} status="active" />
      <SessionsTable groupId={groupId} status="completed" />
      <DocumentsTable admin />
    </Box>
  );
}
```

---

### ✅ Final Steps

1. Create `src/index.js`:

   ```js
   import React from 'react';
   import ReactDOM from 'react-dom/client';
   import { Provider } from 'react-redux';
   import AppRouter from './routes/AppRouter';
   import { store } from './app/store';

   const root = ReactDOM.createRoot(document.getElementById('root'));
   root.render(
     <Provider store={store}>
       <AppRouter />
     </Provider>
   );
   ```

2. Run it:

   ```bash
   npm start
   ```

3. Test with your backend at `http://127.0.0.1:8000/api/`.

---

🎉 You now have a **complete, runnable Virtual Study Group frontend** built with React 18 + Redux Toolkit + MUI (Light Theme), fully mapped to your backend endpoints.

Would you like me to also generate a **Postman collection (`VirtualStudyGroup_API.postman_collection.json`)** for all `/api/auth`, `/groups`, `/sessions`, `/documents`, and `/document-comments` endpoints to simplify backend testing?
