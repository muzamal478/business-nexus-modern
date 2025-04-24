## Week 1: Setup + Authentication + Basic Layouts

### Folder Structure
```
business-nexus/
├── public/
│   ├── favicon.ico
│   └── mock-data.json
├── src/
│   ├── assets/
│   │   └── logo.png
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.jsx
│   │   │   ├── Card.jsx
│   │   │   └── Input.jsx
│   │   ├── layout/
│   │   │   ├── DashboardLayout.jsx
│   │   │   ├── Navbar.jsx
│   │   │   └── Sidebar.jsx
│   ├── context/
│   │   └── AuthContext.jsx
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── Login.jsx
│   │   │   └── Register.jsx
│   │   ├── dashboard/
│   │   │   ├── InvestorDashboard.jsx
│   │   │   └── EntrepreneurDashboard.jsx
│   │   ├── profile/
│   │   │   ├── InvestorProfile.jsx
│   │   │   └── EntrepreneurProfile.jsx
│   ├── services/
│   │   └── api.js
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── db.json
├── package.json
├── vite.config.js
└── README.md
```

### Step-by-Step Setup
1. **Initialize Vite Project**:
   ```bash
   npm create vite@latest business-nexus-modern -- --template react
   cd business-nexus
   npm install
   ```

2. **Install Dependencies**:
   ```bash
   npm install tailwindcss@latest postcss@latest autoprefixer@latest axios react-router-dom
   npx tailwindcss init -p
   ```

3. **Configure Tailwind** (`tailwind.config.js`):
   ```javascript
   /** @type {import('tailwindcss').Config} */
   export default {
     content: ["./index.html", "./src/**/*.{js,jsx}"],
     theme: { extend: {} },
     plugins: [],
   };
   ```

4. **Tailwind CSS** (`src/index.css`):
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

5. **Setup JSON Server**:
   - Create `db.json` in the root:
   ```json
   {
     "users": [
       { "id": 1, "email": "investor@example.com", "password": "pass123", "role": "investor", "name": "John Investor" },
       { "id": 2, "email": "entrepreneur@example.com", "password": "pass123", "role": "entrepreneur", "name": "Jane Entrepreneur" }
     ],
     "entrepreneurs": [
       { "id": 1, "name": "Jane Entrepreneur", "startup": "TechTrend", "pitch": "Innovative AI solutions", "bio": "Passionate founder", "funding": 500000, "pitchDeck": "deck.pdf" }
     ],
     "investors": [
       { "id": 1, "name": "John Investor", "bio": "Experienced VC", "interests": ["AI", "Fintech"], "portfolio": ["CompanyA", "CompanyB"] }
     ],
     "requests": [
       { "id": 1, "investorId": 1, "entrepreneurId": 1, "status": "pending" }
     ]
   }
   ```
   - Run JSON Server: `npx json-server --watch db.json --port 3001`

6. **Git Setup**:
   ```bash
   git init
   git add .
   git commit -m "Initial project setup"
   git remote add origin https://github.com/your-username/business-nexus.git
   git push -u origin main
   ```

### Code: Week 1

#### `src/App.jsx`
```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Login from "./pages/auth/Login";
import Register from "./pages/auth/Register";
import InvestorDashboard from "./pages/dashboard/InvestorDashboard";
import EntrepreneurDashboard from "./pages/dashboard/EntrepreneurDashboard";
import InvestorProfile from "./pages/profile/InvestorProfile";
import EntrepreneurProfile from "./pages/profile/EntrepreneurProfile";
import DashboardLayout from "./components/layout/DashboardLayout";
import { AuthProvider } from "./context/AuthContext";

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />
          <Route element={<DashboardLayout />}>
            <Route path="/dashboard/investor" element={<InvestorDashboard />} />
            <Route path="/dashboard/entrepreneur" element={<EntrepreneurDashboard />} />
            <Route path="/profile/investor/:id" element={<InvestorProfile />} />
            <Route path="/profile/entrepreneur/:id" element={<EntrepreneurProfile />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```

#### `src/context/AuthContext.jsx`
```jsx
import { createContext, useState, useContext } from "react";
import { useNavigate } from "react-router-dom";
import api from "../services/api";

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();

  const login = async (email, password, role) => {
    const response = await api.get("/users", { params: { email, password, role } });
    if (response.data.length > 0) {
      setUser(response.data[0]);
      navigate(`/dashboard/${role}`);
    } else {
      alert("Invalid credentials");
    }
  };

  const register = async (email, password, role, name) => {
    const response = await api.post("/users", { email, password, role, name });
    setUser(response.data);
    navigate(`/dashboard/${role}`);
  };

  const logout = () => {
    setUser(null);
    navigate("/login");
  };

  return (
    <AuthContext.Provider value={{ user, login, register, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

#### `src/services/api.js`
```javascript
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:3001",
});

export default api;
```

#### `src/components/layout/DashboardLayout.jsx`
```jsx
import { Outlet } from "react-router-dom";
import Navbar from "./Navbar";
import Sidebar from "./Sidebar";

function DashboardLayout() {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Navbar />
        <main className="p-6 flex-1 overflow-auto">
          <Outlet />
        </main>
      </div>
    </div>
  );
}

export default DashboardLayout;
```

#### `src/components/layout/Navbar.jsx`
```jsx
import { useAuth } from "../../context/AuthContext";

function Navbar() {
  const { user, logout } = useAuth();

  return (
    <nav className="bg-blue-600 text-white p-4 flex justify-between">
      <div>Business Nexus</div>
      <div className="flex items-center gap-4">
        <span>{user?.name}</span>
        <button onClick={logout} className="bg-red-500 px-4 py-2 rounded">
          Logout
        </button>
      </div>
    </nav>
  );
}

export default Navbar;
```

#### `src/components/layout/Sidebar.jsx`
```jsx
import { NavLink } from "react-router-dom";
import { useAuth } from "../../context/AuthContext";

function Sidebar() {
  const { user } = useAuth();
  const role = user?.role;

  return (
    <aside className="bg-gray-800 text-white w-64 p-4">
      <div className="mb-6">Menu</div>
      <nav>
        <NavLink to={`/dashboard/${role}`} className="block py-2 hover:bg-gray-700">
          Dashboard
        </NavLink>
        <NavLink to={`/profile/${role}/${user?.id}`} className="block py-2 hover:bg-gray-700">
          Profile
        </NavLink>
      </nav>
    </aside>
  );
}

export default Sidebar;
```

#### `src/components/common/Button.jsx`
```jsx
function Button({ children, onClick, className }) {
  return (
    <button
      onClick={onClick}
      className={`bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 ${className}`}
    >
      {children}
    </button>
  );
}

export default Button;
```

#### `src/components/common/Input.jsx`
```jsx
function Input({ label, type, value, onChange, name, required }) {
  return (
    <div className="mb-4">
      <label className="block text-sm mb-1">{label}</label>
      <input
        type={type}
        value={value}
        onChange={onChange}
        name={name}
        required={required}
        className="w-full p-2 border rounded"
      />
    </div>
  );
}

export default Input;
```

#### `src/components/common/Card.jsx`
```jsx
function Card({ children, className }) {
  return (
    <div className={`bg-white shadow rounded-lg p-4 ${className}`}>
      {children}
    </div>
  );
}

export default Card;
```

#### `src/pages/auth/Login.jsx`
```jsx
import { useState } from "react";
import { useAuth } from "../../context/AuthContext";
import Input from "../../components/common/Input";
import Button from "../../components/common/Button";
import { Link } from "react-router-dom";

function Login() {
  const { login } = useAuth();
  const [form, setForm] = useState({ email: "", password: "", role: "investor" });

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    login(form.email, form.password, form.role);
  };

  return (
    <div className="flex items-center justify-center h-screen bg-gray-100">
      <form onSubmit={handleSubmit} className="bg-white p-6 rounded shadow w-96">
        <h2 className="text-2xl mb-4">Login</h2>
        <Input
          label="Email"
          type="email"
          name="email"
          value={form.email}
          onChange={handleChange}
          required
        />
        <Input
          label="Password"
          type="password"
          name="password"
          value={form.password}
          onChange={handleChange}
          required
        />
        <div className="mb-4">
          <label className="block text-sm mb-1">Role</label>
          <select
            name="role"
            value={form.role}
            onChange={handleChange}
            className="w-full p-2 border rounded"
          >
            <option value="investor">Investor</option>
            <option value="entrepreneur">Entrepreneur</option>
          </select>
        </div>
        <Button type="submit">Login</Button>
        <p className="mt-4">
          Don't have an account? <Link to="/register" className="text-blue-500">Register</Link>
        </p>
      </form>
    </div>
  );
}

export default Login;
```

#### `src/pages/auth/Register.jsx`
```jsx
import { useState } from "react";
import { useAuth } from "../../context/AuthContext";
import Input from "../../components/common/Input";
import Button from "../../components/common/Button";
import { Link } from "react-router-dom";

function Register() {
  const { register } = useAuth();
  const [form, setForm] = useState({ email: "", password: "", role: "investor", name: "" });

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    register(form.email, form.password, form.role, form.name);
  };

  return (
    <div className="flex items-center justify-center h-screen bg-gray-100">
      <form onSubmit={handleSubmit} className="bg-white p-6 rounded shadow w-96">
        <h2 className="text-2xl mb-4">Register</h2>
        <Input
          label="Name"
          type="text"
          name="name"
          value={form.name}
          onChange={handleChange}
          required
        />
        <Input
          label="Email"
          type="email"
          name="email"
          value={form.email}
          onChange={handleChange}
          required
        />
        <Input
          label="Password"
          type="password"
          name="password"
          value={form.password}
          onChange={handleChange}
          required
        />
        <div className="mb-4">
          <label className="block text-sm mb-1">Role</label>
          <select
            name="role"
            value={form.role}
            onChange={handleChange}
            className="w-full p-2 border rounded"
          >
            <option value="investor">Investor</option>
            <option value="entrepreneur">Entrepreneur</option>
          </select>
        </div>
        <Button type="submit">Register</Button>
        <p className="mt-4">
          Already have an account? <Link to="/login" className="text-blue-500">Login</Link>
        </p>
      </form>
    </div>
  );
}

export default Register;
```

#### `src/pages/dashboard/InvestorDashboard.jsx` (Placeholder)
```jsx
function InvestorDashboard() {
  return <div>Investor Dashboard (Week 2)</div>;
}

export default InvestorDashboard;
```

#### `src/pages/dashboard/EntrepreneurDashboard.jsx` (Placeholder)
```jsx
function EntrepreneurDashboard() {
  return <div>Entrepreneur Dashboard (Week 2)</div>;
}

export default EntrepreneurDashboard;
```

#### `src/pages/profile/InvestorProfile.jsx` (Placeholder)
```jsx
function InvestorProfile() {
  return <div>Investor Profile (Week 2)</div>;
}

export default InvestorProfile;
```

#### `src/pages/profile/EntrepreneurProfile.jsx` (Placeholder)
```jsx
function EntrepreneurProfile() {
  return <div>Entrepreneur Profile (Week 2)</div>;
}
export default EntrepreneurProfile;
```

### Week 1 Deliverables
- Project setup with Vite, Tailwind, and React Router.
- Authentication UI (Login/Register) with role-based redirects.
- Shared DashboardLayout with Navbar and Sidebar.
- Component library (Button, Input, Card).
- GitHub repo pushed: `git add . && git commit -m "Week 1 complete" && git push`.

---
