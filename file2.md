Great 👍 — here comes **Part 2 of 3**: the Groups, Admin Dashboard, and Layout UI.
After you paste Part 1 into your project, create these new files under `src/features/` and `src/features/layout/`.

---

### 📁 `src/features/groups/GroupsService.js`

```js
import axios from '../../utils/axiosConfig';

const getAllGroups = () => axios.get('/groups/').then(r=>r.data);
const getMyGroups = () => axios.get('/groups/my-groups/').then(r=>r.data);
const getAdminGroups = () => axios.get('/groups/my-admin-groups/').then(r=>r.data);
const createGroup = (data) => axios.post('/groups/', data).then(r=>r.data);
const joinGroup = (id) => axios.post(`/groups/${id}/join/`).then(r=>r.data);
const leaveGroup = (id) => axios.post(`/groups/${id}/leave/`).then(r=>r.data);

export default {getAllGroups,getMyGroups,getAdminGroups,createGroup,joinGroup,leaveGroup};
```

---

### 📁 `src/features/groups/GroupsSlice.js`

```js
import {createSlice,createAsyncThunk} from '@reduxjs/toolkit';
import GroupsService from './GroupsService';

export const fetchGroups=createAsyncThunk('groups/fetch',GroupsService.getAllGroups);
export const fetchMyGroups=createAsyncThunk('groups/my',GroupsService.getMyGroups);
export const fetchAdminGroups=createAsyncThunk('groups/admin',GroupsService.getAdminGroups);
export const createGroup=createAsyncThunk('groups/create',GroupsService.createGroup);
export const joinGroup=createAsyncThunk('groups/join',GroupsService.joinGroup);
export const leaveGroup=createAsyncThunk('groups/leave',GroupsService.leaveGroup);

const slice=createSlice({
  name:'groups',
  initialState:{groups:[],myGroups:[],adminGroups:[],loading:false},
  reducers:{},
  extraReducers:(b)=>{
    b.addCase(fetchGroups.fulfilled,(s,a)=>{s.groups=a.payload})
     .addCase(fetchMyGroups.fulfilled,(s,a)=>{s.myGroups=a.payload})
     .addCase(fetchAdminGroups.fulfilled,(s,a)=>{s.adminGroups=a.payload});
  }
});
export default slice.reducer;
```

---

### 📁 `src/features/groups/ExploreGroups.jsx`

```jsx
import React,{useEffect}from'react';
import{useDispatch,useSelector}from'react-redux';
import{fetchGroups,joinGroup}from'./GroupsSlice';
import{Grid,Card,CardContent,Typography,Button,Snackbar,Alert}from'@mui/material';

export default function ExploreGroups(){
 const dispatch=useDispatch();
 const{groups}=useSelector(s=>s.groups);
 const[open,setOpen]=React.useState(false);
 useEffect(()=>{dispatch(fetchGroups())},[dispatch]);
 const handleJoin=async(id)=>{
   try{await dispatch(joinGroup(id)).unwrap();setOpen(true);}catch{}
 };
 return(<>
  <Grid container spacing={2} p={3}>
   {groups.map(g=>(
     <Grid item xs={12} sm={6} md={4} key={g.id}>
      <Card>
       <CardContent>
        <Typography variant="h6">{g.name}</Typography>
        <Typography variant="body2" sx={{mb:2}}>{g.description}</Typography>
        <Button variant="contained" onClick={()=>handleJoin(g.id)}>Join</Button>
       </CardContent>
      </Card>
     </Grid>
   ))}
  </Grid>
  <Snackbar open={open} autoHideDuration={3000} onClose={()=>setOpen(false)}>
   <Alert severity="success" sx={{width:'100%'}}>Joined group!</Alert>
  </Snackbar>
 </>);
}
```

---

### 📁 `src/features/groups/MyGroups.jsx`

```jsx
import React,{useEffect}from'react';
import{useDispatch,useSelector}from'react-redux';
import{fetchMyGroups,leaveGroup}from'./GroupsSlice';
import{Grid,Card,CardContent,Typography,Button,Snackbar,Alert}from'@mui/material';

export default function MyGroups(){
 const dispatch=useDispatch();
 const{myGroups}=useSelector(s=>s.groups);
 const[open,setOpen]=React.useState(false);
 useEffect(()=>{dispatch(fetchMyGroups())},[dispatch]);
 const handleLeave=async(id)=>{
  try{await dispatch(leaveGroup(id)).unwrap();setOpen(true);}catch{}
 };
 return(<>
  <Grid container spacing={2} p={3}>
   {myGroups.map(g=>(
    <Grid item xs={12} sm={6} md={4} key={g.id}>
     <Card><CardContent>
      <Typography variant="h6">{g.name}</Typography>
      <Typography variant="body2" sx={{mb:2}}>{g.description}</Typography>
      <Button variant="outlined" onClick={()=>handleLeave(g.id)}>Leave</Button>
     </CardContent></Card>
    </Grid>
   ))}
  </Grid>
  <Snackbar open={open} autoHideDuration={3000} onClose={()=>setOpen(false)}>
   <Alert severity="info" sx={{width:'100%'}}>Left group</Alert>
  </Snackbar>
 </>);
}
```

---

### 📁 `src/features/groups/AdminGroups.jsx`

```jsx
import React,{useEffect,useState}from'react';
import{useDispatch,useSelector}from'react-redux';
import{fetchAdminGroups,createGroup}from'./GroupsSlice';
import{Grid,Card,CardContent,Typography,Button,TextField,Box,Snackbar,Alert}from'@mui/material';
import{useNavigate}from'react-router-dom';

export default function AdminGroups(){
 const dispatch=useDispatch();const navigate=useNavigate();
 const{adminGroups}=useSelector(s=>s.groups);
 const[form,setForm]=useState({name:'',description:''});
 const[open,setOpen]=useState(false);
 useEffect(()=>{dispatch(fetchAdminGroups())},[dispatch]);
 const handleSubmit=async(e)=>{e.preventDefault();await dispatch(createGroup(form));setOpen(true);dispatch(fetchAdminGroups());};
 return(<Box p={3}>
  <Typography variant="h6" gutterBottom>Create Group</Typography>
  <Box component="form" onSubmit={handleSubmit} sx={{mb:3}}>
   <TextField label="Name" name="name" fullWidth margin="dense" onChange={e=>setForm({...form,[e.target.name]:e.target.value})}/>
   <TextField label="Description" name="description" fullWidth margin="dense" onChange={e=>setForm({...form,[e.target.name]:e.target.value})}/>
   <Button type="submit" variant="contained" sx={{mt:1}}>Create</Button>
  </Box>
  <Typography variant="h6" gutterBottom>My Admin Groups</Typography>
  <Grid container spacing={2}>
   {adminGroups.map(g=>(
    <Grid item xs={12} sm={6} md={4} key={g.id}>
     <Card><CardContent>
      <Typography variant="h6">{g.name}</Typography>
      <Typography variant="body2">{g.description}</Typography>
      <Button variant="outlined" sx={{mt:1}} onClick={()=>navigate(`/admin-groups/${g.id}`)}>View Group</Button>
     </CardContent></Card>
    </Grid>
   ))}
  </Grid>
  <Snackbar open={open} autoHideDuration={3000} onClose={()=>setOpen(false)}>
   <Alert severity="success">Group Created</Alert>
  </Snackbar>
 </Box>);
}
```

---

### 📁 `src/features/layout/Navbar.jsx`

```jsx
import React from'react';
import{AppBar,Toolbar,Typography,Button}from'@mui/material';
import{Link,useNavigate}from'react-router-dom';
import{useSelector,useDispatch}from'react-redux';
import{logout}from'../auth/AuthSlice';

export default function Navbar(){
 const{token}=useSelector(s=>s.auth);
 const dispatch=useDispatch();const navigate=useNavigate();
 const handleLogout=()=>{dispatch(logout());navigate('/');};
 return(
 <AppBar position="sticky">
  <Toolbar>
   <Typography variant="h6" sx={{flexGrow:1}}>StudyGroups</Typography>
   {!token?(
    <>
     <Button color="inherit" component={Link} to="/register">Register</Button>
     <Button color="inherit" component={Link} to="/login">Login</Button>
    </>
   ):(
    <>
     <Button color="inherit" component={Link} to="/explore">Explore</Button>
     <Button color="inherit" component={Link} to="/my-groups">My Groups</Button>
     <Button color="inherit" component={Link} to="/admin-groups">Admin Groups</Button>
     <Button color="inherit" onClick={handleLogout}>Logout</Button>
    </>
   )}
  </Toolbar>
 </AppBar>);
}
```

---

### 📁 `src/features/layout/Footer.jsx`

```jsx
import React from'react';
import{AppBar,Toolbar,Typography}from'@mui/material';

export default function Footer(){
 return(
  <AppBar position="sticky" color="primary" sx={{top:'auto',bottom:0}}>
   <Toolbar>
    <Typography variant="body2" sx={{flexGrow:1,textAlign:'center'}}>
     © {new Date().getFullYear()} Virtual Study Group | Team LearnLytica
    </Typography>
   </Toolbar>
  </AppBar>
 );
}
```

---

### 📁 `src/features/layout/LandingPage.jsx`

```jsx
import React from'react';
import{Container,Typography,Button}from'@mui/material';
import{Link}from'react-router-dom';

export default function LandingPage(){
 return(
 <Container sx={{textAlign:'center',py:10}}>
  <Typography variant="h3" gutterBottom>Welcome to Virtual Study Groups</Typography>
  <Typography variant="h6" mb={3}>
   Collaborate, Learn, and Share Knowledge with Peers
  </Typography>
  <Button component={Link} to="/register" variant="contained" sx={{mx:1}}>Get Started</Button>
  <Button component={Link} to="/login" variant="outlined" sx={{mx:1}}>Already a Member? Login</Button>
 </Container>);
}
```

---

✅ That completes **Part 2 — Groups + Admin + Layout**.

Next (Part 3 → Sessions + Documents + Comments + Admin Group View) will finalize the app with full data tables, document upload & approval, and the discussion dialog.

Would you like me to proceed with **Part 3 (final)** now?
