# ExpressJs2

# **Task: Build a Todo List API with User Authentication**

***Objective***

Create a todo list API where users can register, login, and manage their own todos.

**Requirements**

1. Setup & Installation

```bash
npm install express bcryptjs jsonwebtoken dotenv
```

2. Project Structure

```
todo-api/
├── models/
│   ├── User.js
│   └── Todo.js
├── middleware/
│   └── auth.js
├── .env
└── server.js
```

**Core Features**

**Phase 1: User Authentication**

|Endpoint| Method| Description|
|----|----|----|
|/api/auth/register|POST| Create new account|
|/api/auth/login| POST| Login and get token|
|/api/auth/profile| GET| Get user info (protected)|

**User Model**

```javascript
// models/User.js
{
  name: String,
  email: String,      // unique
  password: String,    // hashed with bcrypt
  createdAt: Date
}
```

**Phase 2: Todo Management (All routes protected)**

|Endpoint| Method| Description|
|----|----|----|
|/api/todos| GET| Get user's todos|
|/api/todos| POST| Create new todo|
|/api/todos/:id| PUT| Update todo|
|/api/todos/:id| DELETE| Delete todo|
|/api/todos/:id/complete| PATCH| Mark as complete/incomplete|

**Todo Model**

```javascript
// models/Todo.js
{
  title: String,
  description: String,
  completed: Boolean,
  userId: ObjectId,    // reference to User
  dueDate: Date,
  priority: String |    // 'low', 'medium', 'high'
}
```

**Step-by-Step Implementation**

**Step 1: Environment Setup (.env)**

```env
PORT=3000
JWT_SECRET=your_super_secret_key_here
```

**Step 2: Authentication Middleware**

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

module.exports = (req, res, next) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Access denied' });
  }
  
  try {
    const verified = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = verified.userId;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

**Step 3: Register Route**

```javascript
app.post('/api/auth/register', async (req, res) => {
  try {
    // Check if user exists
    const existingUser = await User.findOne({ email: req.body.email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email already exists' });
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(req.body.password, 10);
    
    // Create user
    const user = new User({
      name: req.body.name,
      email: req.body.email,
      password: hashedPassword
    });
    
    await user.save();
    
    // Create token
    const token = jwt.sign(
      { userId: user._id },
      process.env.JWT_SECRET
    );
    
    res.status(201).json({ token, user: { name: user.name, email: user.email } });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});
```

**Step 4: Todo Routes (Protected)**

```javascript
// Get user's todos
app.get('/api/todos', auth, async (req, res) => {
  const todos = await Todo.find({ userId: req.userId });
  res.json(todos);
});

// Create todo
app.post('/api/todos', auth, async (req, res) => {
  const todo = new Todo({
    title: req.body.title,
    description: req.body.description,
    dueDate: req.body.dueDate,
    priority: req.body.priority || 'medium',
    userId: req.userId
  });
  
  await todo.save();
  res.status(201).json(todo);
});

// Update todo
app.put('/api/todos/:id', auth, async (req, res) => {
  const todo = await Todo.findOneAndUpdate(
    { _id: req.params.id, userId: req.userId },
    req.body,
    { new: true }
  );
  
  if (!todo) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  
  res.json(todo);
});
```

**Practice Tasks**

**Task 1: User Authentication**
- Register a new user (POST to /api/auth/register)
- Login with that user (POST to /api/auth/login)
- Save the token you receive
- Try to access profile without token (should fail)
- Access profile with token (should work)

**Task 2: Todo CRUD**

- Create 3 different todos
-  Get all your todos
- Update one todo's title
- Mark one todo as completed
- Delete a todo

**Task 3: Filtering & Sorting**
Add query parameters to GET /api/todos:

- ?completed=true - show only completed todos
- ?priority=high - filter by priority
- ?sort=dueDate - sort by due date
- ?limit=5 - limit results

**Bonus Challenges**

- Password Reset Flow
   - Add "forgot password" functionality
   - Implement reset token that expires in 1 hour
- Todo Categories
   - Add categories/tags to todos
   - Filter by multiple categories
- Todo Sharing
   - Allow users to share todos with other users
   - Add "shared with" array to Todo model
- Rate Limiting
   - Limit login attempts (5 per minute)
   - Prevent brute force attacks

**Testing with Postman**

***Register Request:***

```json
POST http://localhost:3000/api/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

***Login Request:***

```json
POST http://localhost:3000/api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "secret123"
}
```

***Create Todo (with token):***

```json
POST http://localhost:3000/api/todos
Authorization: Bearer YOUR_TOKEN_HERE
Content-Type: application/json

{
  "title": "Learn Express",
  "description": "Complete the todo API project",
  "dueDate": "2024-12-31",
  "priority": "high"
}
```

**Validation Rules**

***Add these validations:***

- Email must be valid format
- Password minimum 6 characters
- Todo title required, max 100 chars
- Due date must be future date
- Priority must be 'low', 'medium', or 'high'

***Expected Learning Outcomes***

- User authentication with JWT
- Password hashing with bcrypt
- Protected routes
- Relationship between User and Todo
- Environment variables
- Error handling patterns
- Request validation


**Submission Checklist**

[] Users can register and login
[] Tokens expire after 24 hours
[] Users only see their own todos
[] All CRUD operations work
[] Input validation implemented
[] Error messages are helpful
[] Passwords are hashed
