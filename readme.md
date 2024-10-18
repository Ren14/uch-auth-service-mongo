# Pre requisites
### Running Docker on MacOS without Docker Desktop
https://dev.to/mochafreddo/running-docker-on-macos-without-docker-desktop-64o

### Install Mongo with Docker
https://www.mongodb.com/docs/manual/tutorial/install-mongodb-community-with-docker/

# Create the application

### Step 1: Set up your Node.js environment

1. **Create a new directory** for your project and navigate into it:
   ```bash
   mkdir auth-api
   cd auth-api
   ```

2. **Initialize a new Node.js project**:
   ```bash
   npm init -y
   ```

3. **Install the required packages**:
   ```bash
   npm install express mongoose bcryptjs jsonwebtoken dotenv
   ```

### Step 2: Create your project structure

Create the following files in your project directory:

```
auth-api/
│
├── .env
├── app.js
├── models/
│   └── User.js
└── routes/
    └── auth.js
```

### Step 3: Set up the User model

In `models/User.js`, define the User schema:

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
});

module.exports = mongoose.model('User', userSchema);
```

### Step 4: Create the authentication routes

In `routes/auth.js`, handle user registration and login:

```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const router = express.Router();

// Register
router.post('/register', async (req, res) => {
    const { email, password } = req.body;
    try {
        const hashedPassword = await bcrypt.hash(password, 10);
        const user = new User({ email, password: hashedPassword });
        await user.save();
        res.status(201).send('User registered successfully');
    } catch (error) {
        res.status(400).send('Error registering user');
    }
});

// Login
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    try {
        const user = await User.findOne({ email });
        if (!user) return res.status(400).send('User not found');

        const validPassword = await bcrypt.compare(password, user.password);
        if (!validPassword) return res.status(400).send('Invalid password');

        const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
        res.json({ token });
    } catch (error) {
        res.status(500).send('Server error');
    }
});

module.exports = router;
```

### Step 5: Set up the Express app

In `app.js`, set up the Express application and connect to MongoDB:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const authRoutes = require('./routes/auth');
require('dotenv').config();

const app = express();
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('MongoDB connected'))
    .catch(err => console.error(err));

// Use Routes
app.use('/api/v1', authRoutes);

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

### Step 6: Create the `.env` file

In the `.env` file, add your MongoDB connection string and JWT secret:

```
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
```

### Step 7: Test the API

1. Start your server:
   ```bash
   node app.js
   ```

2. Use Postman or any API client to test the endpoints:
    - **Register a user**: POST `http://localhost:5000/api/v1/register` with a JSON body `{"email": "test@example.com", "password": "password123"}`.
    - **Login a user**: POST `http://localhost:5000/api/v1/login` with the same JSON body.


3. CURL Examples
   4. Register
   ```curl
   curl --location 'http://localhost:5000/api/v1/register' \
   --header 'Content-Type: application/json' \
   --data-raw '{
   "email": "test@example.com",
   "password": "password123"
   }'
   ```
   5. Login
   ```curl
   curl --location 'http://localhost:4500/api/v1/login' \
   --header 'Content-Type: application/json' \
   --data-raw '{
   "email": "test@example.com",
   "password": "password123"
   }'
   ```
  

This basic setup provides you with a working user authentication API using Node.js, Express, MongoDB, bcrypt, and JWT. You can expand on this by adding features like email verification, password reset, and user roles as needed.

# How to find documents on DataGrip
https://www.jetbrains.com/help/datagrip/sql-for-mongodb.html