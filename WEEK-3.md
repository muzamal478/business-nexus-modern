## Week 3: Communication Chat + Polish + Demo Prep

### Folder Structure Update
```
business-nexus/
├── src/
│   ├── components/
│   │   ├── chat/
│   │   │   ├── ChatWindow.jsx
│   │   │   └── Message.jsx
│   ├── pages/
│   │   ├── chat/
│   │   │   └── Chat.jsx
│   ├── context/
│   │   └── ChatContext.jsx
```

### Update `db.json` (Add Chat Data)
```json
{
  "users": [...],
  "entrepreneurs": [...],
  "investors": [...],
  "requests": [...],
  "messages": [
    { "id": 1, "senderId": 1, "receiverId": 2, "content": "Interested in your startup!", "timestamp": "2025-04-24T10:00:00Z" },
    { "id": 2, "senderId": 2, "receiverId": 1, "content": "Thanks! Let's discuss.", "timestamp": "2025-04-24T10:05:00Z" }
  ]
}
```

### Code: Week 3

#### `src/context/ChatContext.jsx`
```jsx
import { createContext, useState, useContext } from "react";
import api from "../services/api";

const ChatContext = createContext();

export const ChatProvider = ({ children }) => {
  const [messages, setMessages] = useState([]);

  const fetchMessages = async (userId, otherUserId) => {
    const response = await api.get("/messages", {
      params: { senderId: userId, receiverId: otherUserId },
    });
    setMessages(response.data);
  };

  const sendMessage = async (senderId, receiverId, content) => {
    const message = {
      senderId,
      receiverId,
      content,
      timestamp: new Date().toISOString(),
    };
    const response = await api.post("/messages", message);
    setMessages([...messages, response.data]);
  };

  return (
    <ChatContext.Provider value={{ messages, fetchMessages, sendMessage }}>
      {children}
    </ChatContext.Provider>
  );
};

export const useChat = () => useContext(ChatContext);
```

#### `src/components/chat/ChatWindow.jsx`
```jsx
import { useState, useEffect } from "react";
import { useChat } from "../../context/ChatContext";
import { useAuth } from "../../context/AuthContext";
import Message from "./Message";
import Input from "../common/Input";
import Button from "../common/Button";

function ChatWindow({ otherUserId }) {
  const { user } = useAuth();
  const { messages, fetchMessages, sendMessage } = useChat();
  const [content, setContent] = useState("");

  useEffect(() => {
    fetchMessages(user.id, otherUserId);
  }, [user.id, otherUserId]);

  const handleSend = (e) => {
    e.preventDefault();
    if (content.trim()) {
      sendMessage(user.id, otherUserId, content);
      setContent("");
    }
  };

  return (
    <div className="flex flex-col h-[500px] bg-white shadow rounded-lg">
      <div className="flex-1 p-4 overflow-y-auto">
        {messages.map((msg) => (
          <Message
            key={msg.id}
            content={msg.content}
            isSender={msg.senderId === user.id}
            timestamp={msg.timestamp}
          />
        ))}
      </div>
      <form onSubmit={handleSend} className="p-4 border-t">
        <div className="flex gap

System: I'm sorry, but I notice the response was cut off. Let me complete the `ChatWindow.jsx` file and provide the remaining code and instructions to finalize the Week 3 tasks, ensuring everything is professional, consistent, and ready for copy-paste. I'll also include deployment steps, demo prep, and a summary to meet your requirements by covering all aspects of the Business Nexus project.

---

### Completing `src/components/chat/ChatWindow.jsx`
```jsx
import { useState, useEffect } from "react";
import { useChat } from "../../context/ChatContext";
import { useAuth } from "../../context/AuthContext";
import Message from "./Message";
import Input from "../common/Input";
import Button from "../common/Button";

function ChatWindow({ otherUserId }) {
  const { user } = useAuth();
  const { messages, fetchMessages, sendMessage } = useChat();
  const [content, setContent] = useState("");

  useEffect(() => {
    fetchMessages(user.id, otherUserId);
  }, [user.id, otherUserId]);

  const handleSend = (e) => {
    e.preventDefault();
    if (content.trim()) {
      sendMessage(user.id, otherUserId, content);
      setContent("");
    }
  };

  return (
    <div className="flex flex-col h-[500px] bg-white shadow rounded-lg">
      <div className="flex-1 p-4 overflow-y-auto">
        {messages.map((msg) => (
          <Message
            key={msg.id}
            content={msg.content}
            isSender={msg.senderId === user.id}
            timestamp={msg.timestamp}
          />
        ))}
      </div>
      <form onSubmit={handleSend} className="p-4 border-t flex gap-2">
        <Input
          type="text"
          value={content}
          onChange={(e) => setContent(e.target.value)}
          placeholder="Type a message..."
          className="flex-1"
        />
        <Button type="submit">Send</Button>
      </form>
    </div>
  );
}

export default ChatWindow;
```

### `src/components/chat/Message.jsx`
```jsx
function Message({ content, isSender, timestamp }) {
  return (
    <div className={`flex ${isSender ? "justify-end" : "justify-start"} mb-2`}>
      <div
        className={`max-w-xs p-2 rounded-lg ${
          isSender ? "bg-blue-500 text-white" : "bg-gray-200"
        }`}
      >
        <p>{content}</p>
        <p className="text-xs text-gray-500 mt-1">
          {new Date(timestamp).toLocaleTimeString()}
        </p>
      </div>
    </div>
  );
}

export default Message;
```

### `src/pages/chat/Chat.jsx`
```jsx
import { useParams } from "react-router-dom";
import ChatWindow from "../../components/chat/ChatWindow";
import Card from "../../components/common/Card";

function Chat() {
  const { userId } = useParams();

  return (
    <Card>
      <h2 className="text-2xl mb-4">Chat</h2>
      <ChatWindow otherUserId={parseInt(userId)} />
    </Card>
  );
}

export default Chat;
```

### Update `src/App.jsx` (Add Chat Route)
```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Login from "./pages/auth/Login";
import Register from "./pages/auth/Register";
import InvestorDashboard from "./pages/dashboard/InvestorDashboard";
import EntrepreneurDashboard from "./pages/dashboard/EntrepreneurDashboard";
import InvestorProfile from "./pages/profile/InvestorProfile";
import EntrepreneurProfile from "./pages/profile/EntrepreneurProfile";
import Chat from "./pages/chat/Chat";
import DashboardLayout from "./components/layout/DashboardLayout";
import { AuthProvider } from "./context/AuthContext";
import { ChatProvider } from "./context/ChatContext";

function App() {
  return (
    <AuthProvider>
      <ChatProvider>
        <BrowserRouter>
          <Routes>
            <Route path="/login" element={<Login />} />
            <Route path="/register" element={<Register />} />
            <Route element={<DashboardLayout />}>
              <Route path="/dashboard/investor" element={<InvestorDashboard />} />
              <Route path="/dashboard/entrepreneur" element={<EntrepreneurDashboard />} />
              <Route path="/profile/investor/:id" element={<InvestorProfile />} />
              <Route path="/profile/entrepreneur/:id" element={<EntrepreneurProfile />} />
              <Route path="/chat/:userId" element={<Chat />} />
            </Route>
          </Routes>
        </BrowserRouter>
      </ChatProvider>
    </AuthProvider>
  );
}

export default App;
```

### Update Dashboards to Include Chat
#### `src/pages/dashboard/InvestorDashboard.jsx`
```jsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import api from "../../services/api";
import Card from "../../components/common/Card";
import Button from "../../components/common/Button";

function InvestorDashboard() {
  const [entrepreneurs, setEntrepreneurs] = useState([]);

  useEffect(() => {
    api.get("/entrepreneurs").then((response) => {
      setEntrepreneurs(response.data);
    });
  }, []);

  return (
    <div>
      <h2 className="text-2xl mb-4">Entrepreneurs</h2>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {entrepreneurs.map((entrepreneur) => (
          <Card key={entrepreneur.id}>
            <h3 className="text-lg font-bold">{entrepreneur.name}</h3>
            <p className="text-sm text-gray-600">{entrepreneur.startup}</p>
            <p className="mt-2">{entrepreneur.pitch}</p>
            <div className="flex gap-2 mt-4">
              <Link to={`/profile/entrepreneur/${entrepreneur.id}`}>
                <Button>View Profile</Button>
              </Link>
              <Link to={`/chat/${entrepreneur.id}`}>
                <Button className="bg-green-500 hover:bg-green-600">Message</Button>
              </Link>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

export default InvestorDashboard;
```

#### `src/pages/dashboard/EntrepreneurDashboard.jsx`
```jsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import api from "../../services/api";
import Card from "../../components/common/Card";
import Button from "../../components/common/Button";

function EntrepreneurDashboard() {
  const [requests, setRequests] = useState([]);

  useEffect(() => {
    api.get("/requests").then(async (response) => {
      const requestsWithInvestor = await Promise.all(
        response.data.map(async (req) => {
          const investor = await api.get(`/investors/${req.investorId}`);
          return { ...req, investor: investor.data };
        })
      );
      setRequests(requestsWithInvestor);
    });
  }, []);

  return (
    <div>
      <h2 className="text-2xl mb-4">Collaboration Requests</h2>
      <div className="space-y-4">
        {requests.map((request) => (
          <Card key={request.id}>
            <h3 className="text-lg font-bold">{request.investor.name}</h3>
            <p className="text-sm text-gray-600">{request.investor.bio}</p>
            <p className="mt-2">Status: {request.status}</p>
            <div className="flex gap-2 mt-4">
              <Button>Respond</Button>
              <Link to={`/chat/${request.investorId}`}>
                <Button className="bg-green-500 hover:bg-green-600">Message</Button>
              </Link>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
}

export default EntrepreneurDashboard;
```

### UI Polish
1. **Hover Effects and Transitions**:
   Update `src/index.css` to add global transitions:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;

   button, a {
     @apply transition duration-200 ease-in-out;
   }
   ```

2. **Mobile Responsiveness**:
   - Tailwind’s responsive classes (`sm:`, `md:`, `lg:`) are already used in dashboards and layouts.
   - Test on mobile devices using browser dev tools (`Ctrl+Shift+M`).

3. **Dark Mode (Optional)**:
   Add a toggle in `src/components/layout/Navbar.jsx`:
   ```jsx
   import { useState } from "react";
   import { useAuth } from "../../context/AuthContext";

   function Navbar() {
     const { user, logout } = useAuth();
     const [darkMode, setDarkMode] = useState(false);

     const toggleDarkMode = () => {
       setDarkMode(!darkMode);
       document.documentElement.classList.toggle("dark");
     };

     return (
       <nav className="bg-blue-600 dark:bg-gray-800 text-white p-4 flex justify-between">
         <div>Business Nexus</div>
         <div className="flex items-center gap-4">
           <span>{user?.name}</span>
           <button onClick={toggleDarkMode} className="bg-gray-500 px-4 py-2 rounded">
             {darkMode ? "Light" : "Dark"}
           </button>
           <button onClick={logout} className="bg-red-500 px-4 py-2 rounded">
             Logout
           </button>
         </div>
       </nav>
     );
   }

   export default Navbar;
   ```

   Update `src/index.css` for dark mode:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;

   button, a {
     @apply transition duration-200 ease-in-out;
   }

   .dark {
     @apply bg-gray-900 text-white;
   }

   .dark .bg-white {
     @apply bg-gray-800;
   }

   .dark .text-gray-600 {
     @apply text-gray-300;
   }
   ```

4. **Consistent Styling**:
   - Ensure all components use Tailwind classes consistently.
   - Use `Card`, `Button`, and `Input` components across all pages.

### Testing & Debugging
1. **User Flows**:
   - Test login/register with both roles (`investor@example.com`/`pass123`, `entrepreneur@example.com`/`pass123`).
   - Verify redirects to correct dashboards.
   - Check profile views and chat functionality.
2. **Mobile Testing**:
   - Use Chrome DevTools to simulate mobile devices.
   - Ensure sidebar collapses on mobile (add Tailwind `hidden md:block` to `Sidebar.jsx`):
     ```jsx
     <aside className="hidden md:block bg-gray-800 text-white w-64 p-4">
     ```
3. **Bug Fixes**:
   - Fix any layout issues (e.g., overflow in chat window).
   - Ensure reusable components are clean and modular.

### Deployment
1. **Vercel Deployment**:
   - Install Vercel CLI: `npm install -g vercel`
   - Run: `vercel`
   - Follow prompts to link to your GitHub repo (`business-nexus`).
   - Set environment variables in Vercel dashboard:
     - `VITE_API_URL=http://localhost:3001` (for local JSON Server; replace with hosted API if available).
   - Deploy: `vercel --prod`
   - Get the live URL (e.g., `https://business-nexus.vercel.app`).

2. **Mock Data**:
   - Ensure `db.json` is hosted (e.g., use `my-json-server.typicode.com`):
     - Create a repo with `db.json`, then access at `https://my-json-server.typicode.com/your-username/business-nexus`.
     - Update `src/services/api.js`:
       ```javascript
       import axios from "axios";

       const api = axios.create({
         baseURL: "https://my-json-server.typicode.com/your-username/business-nexus",
       });

       export default api;
       ```

3. **GitHub Push**:
   ```bash
   git add .
   git commit -m "Week 3 complete: Chat feature, UI polish, and deployment"
   git push
   ```

### Demo Preparation
1. **Walkthrough Presentation (5-7 mins)**:
   - Create a PowerPoint/Google Slides deck:
     - Slide 1: Title ("Business Nexus: Networking Platform")
     - Slide 2: Project Overview (Purpose, Features)
     - Slide 3: Tech Stack
     - Slide 4: Demo (Screenshots or Live Demo Link)
     - Slide 5: Challenges & Solutions
     - Slide 6: Future Improvements (e.g., real backend, notifications)
     - Slide 7: Q&A
   - Record a video using Zoom/Screencastify showing:
     - Login/register process.
     - Investor dashboard (view entrepreneurs, message).
     - Entrepreneur dashboard (view requests, message).
     - Chat functionality.
     - Profile pages.
   - Upload to Google Drive/YouTube and share the link.

2. **Demo Video Script**:
   - "Welcome to Business Nexus, a platform connecting entrepreneurs and investors."
   - "Here’s the login page. Let’s log in as an investor..."
   - "This is the investor dashboard, showing entrepreneur profiles with a 'Message' button..."
   - "Switching to the entrepreneur dashboard, we see collaboration requests..."
   - "The chat feature allows real-time communication..."
   - "Finally, here’s a sample profile page for an investor..."

### Week 3 Deliverables
- Real-time-style chat feature with message history and UI.
- Polished UI with hover effects, transitions, and dark mode.
- Fully tested user flows and mobile responsiveness.
- Deployed app on Vercel (`https://business-nexus.vercel.app`).
- Demo video and presentation deck.
- Final GitHub push.

---
