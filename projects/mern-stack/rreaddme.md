# MERN Stack Implementation: To-Do Application

## Overview

This documentation walks through the implementation of a full-stack **To-Do application** using the **MERN stack**. It covers backend setup, database configuration, API testing, and frontend development, resulting in a complete web application deployed on an AWS EC2 instance.

### What is the MERN Stack?

The MERN stack consists of four core technologies:

| Component | Role |
|---|---|
| **M**ongoDB | A NoSQL database that stores data in JSON-like documents, offering flexibility and scalability for web applications. |
| **E**xpress.js | A web framework for Node.js that provides a robust set of features for building server-side applications and APIs. |
| **R**eact.js | A front-end JavaScript library (developed by Facebook) used to build dynamic, single-page user interfaces without full page reloads. |
| **N**ode.js | A JavaScript runtime built on Chrome's V8 engine, enabling server-side execution of JavaScript for full-stack development in a single language. |

Together, these four technologies form a complete framework for building dynamic web applications.

### Project Scope

We will build a simple **To-Do application** that allows users to:
- Create tasks
- View (retrieve) tasks
- Delete tasks

- **Frontend:** React.js
- **Backend:** Node.js + Express.js
- **Database:** MongoDB

The project is divided into two major parts:
1. **Backend configuration** (server, database, API routes)
2. **Frontend configuration** (React UI)

---

## Prerequisites

- An AWS EC2 instance (Ubuntu) with SSH access
- A terminal/SSH client
- A [Postman](https://www.postman.com/) account (for API testing)
- A [MongoDB Atlas](https://www.mongodb.com/atlas) account (referred to as "MLab" in the walkthrough, but the modern equivalent is MongoDB Atlas)

---

## Part 1: Backend Configuration

### Step 1: Connect to the AWS Server

Log in to your EC2 instance via SSH using your key pair, username, and public IP address:

```bash
ssh -i <your-key.pem> <username>@<public-ip-address>
```

![Screenshot: Terminal showing successful SSH login to the AWS instance](screenshots/01-ssh-login.png)

### Step 2: Update and Upgrade Ubuntu

Update the package list to ensure you're working with the latest packages:

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 3: Install Node.js

Node.js is required to run the backend server.

```bash
sudo apt install nodejs -y
```

Verify the installation:

```bash
node -v
```

![Screenshot: Terminal output showing installed Node.js version](screenshots/02-node-version.png)

### Step 4: Install NPM

NPM (Node Package Manager) is used to manage project dependencies.

```bash
sudo apt install npm -y
```

Verify the installation:

```bash
npm -v
```

![Screenshot: Terminal output showing installed NPM version](screenshots/03-npm-version.png)

### Step 5: Create the Project Directory

Create a directory for the project and initialize it with NPM:

```bash
mkdir To-Do
cd To-Do
npm init
```

Press **Enter** to accept the default settings (or type a custom project name).

![Screenshot: package.json file generated after npm init](screenshots/04-package-json.png)

### Step 6: Install Express.js

Express.js provides the server-side framework for the application.

```bash
npm install express
```

### Step 7: Create the Entry File

Create the main server file:

```bash
touch index.js
```

### Step 8: Install the dotenv Module

The `dotenv` module manages environment configuration in a single, centralized file.

```bash
npm install dotenv
```

### Step 9: Configure the Express Server

Open `index.js` and add the server configuration to listen on port `5000`:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 5000;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use((req, res, next) => {
  res.send('Welcome to Express');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

Start the server:

```bash
node index.js
```

You should see: `Server running on port 5000`

![Screenshot: Terminal showing "Server running on port 5000" message](screenshots/05-server-running.png)

### Step 10: Open Port 5000 in AWS Security Group

To access the server from the browser, open port 5000 in your EC2 instance's security group:

1. Go to the **AWS Management Console** → **EC2** → **Security Groups**.
2. Select the inbound rules for your instance.
3. Click **Add new rule**.
4. Set **Port** to `5000`, **Source** to `Anywhere (0.0.0.0/0)`.
5. Click **Save rules**.

![Screenshot: AWS Security Group inbound rules showing port 5000 opened](screenshots/06-security-group-port5000.png)

Test it by visiting `http://<public-ip-address>:5000` in your browser. You should see "Welcome to Express."

![Screenshot: Browser showing "Welcome to Express" message](screenshots/07-welcome-express-browser.png)

---

### Step 11: Create Routes

Routes handle the core functionality of the app: creating, retrieving, and deleting tasks.

```bash
mkdir routes
cd routes
touch api.js
```

Open `api.js` and add the route logic:

```javascript
const express = require('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {
  Todo.find({}, (err, data) => {
    if (err) return next(err);
    res.json(data);
  });
});

router.post('/todos', (req, res, next) => {
  if (req.body.action === "") return res.json({ error: "Input field required" });
  const todo = new Todo({ action: req.body.action });
  todo.save((err, data) => {
    if (err) return next(err);
    res.json(data);
  });
});

router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({ _id: req.params.id }, (err, data) => {
    if (err) return next(err);
    res.json(data);
  });
});

module.exports = router;
```

![Screenshot: routes/api.js file with route definitions](screenshots/08-api-js-routes.png)

### Step 12: Install Mongoose

Since MongoDB is a NoSQL database, we use **Mongoose** to define schemas (blueprints) for our data.

```bash
npm install mongoose
```

### Step 13: Create the Data Model

```bash
mkdir models
cd models
touch todo.js
```

Add the schema definition in `todo.js`:

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

const Todo = mongoose.model('todo', TodoSchema);
module.exports = Todo;
```

![Screenshot: models/todo.js file with schema definition](screenshots/09-todo-model.png)

### Step 14: Update Routes to Use the Model

Return to `routes/api.js` and ensure it references the `Todo` model created above (see Step 11's updated code, which already imports it).

---

### Step 15: Set Up MongoDB Atlas (Database)

1. Go to [MongoDB Atlas](https://www.mongodb.com/atlas) and create an account.
2. Deploy a new cluster, selecting the **free tier**.
3. Name the cluster (e.g., `to-do-db`).
4. Choose **AWS** as the cloud provider and select a region closest to you.
5. Click **Create Cluster**.

![Screenshot: MongoDB Atlas cluster creation page](screenshots/10-atlas-cluster-creation.png)

#### Create Database Credentials

1. Create a database username and password.
2. Click **Create Database User**.

![Screenshot: MongoDB Atlas database user creation form](screenshots/11-atlas-db-user.png)

#### Configure Network Access

1. Go to **Network Access**.
2. Click **Add IP Address**.
3. In the IP address field, enter `0.0.0.0/0` (this allows access from anywhere).
4. Set the temporary access duration (e.g., one week) if prompted.
5. Click **Confirm**/**Save**.

![Screenshot: MongoDB Atlas Network Access settings showing "Allow access from anywhere"](screenshots/12-atlas-network-access.png)

#### Create a Database and Collection

1. Return to the cluster page and click **Browse Collections**.
2. Click **Create Database**.
3. Specify a database name and a collection name.
4. Click **Create**.

![Screenshot: MongoDB Atlas collections view showing newly created database](screenshots/13-atlas-create-database.png)

#### Get the Connection String

1. Go back to the cluster and click **Connect**.
2. Select **Drivers**, and choose **Node.js** as the driver.
3. Copy the provided connection string.

![Screenshot: MongoDB Atlas connection string dialog](screenshots/14-atlas-connection-string.png)

---

### Step 16: Configure Environment Variables

Create a `.env` file in the project root:

```bash
touch .env
```

Open the file and add the MongoDB connection string, replacing `<password>` with your database user's password:

```
DB = 'mongodb+srv://<username>:<password>@<cluster-url>/<database>?retryWrites=true&w=majority'
```

![Screenshot: .env file with the MongoDB connection string](screenshots/15-env-file.png)

> **Note:** Storing sensitive configuration (like database credentials) in environment variables rather than hardcoding them is a security best practice.

### Step 17: Update index.js to Use the Environment Configuration

Update `index.js` to connect to MongoDB using Mongoose and the `.env` configuration:

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const dotenv = require('dotenv');

dotenv.config();

const app = express();
const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB)
  .then(() => console.log('Database connected successfully'))
  .catch(err => console.log(err));

mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use(bodyParser.json());

app.use('/api', require('./routes/api'));

app.use((err, req, res, next) => {
  if (err.status) {
    res.status(err.status).json({ message: err.message }).end();
  } else {
    res.status(500).json({ message: err.message }).end();
  }
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

Start the server:

```bash
node index.js
```

You should now see: `Database connected successfully`

![Screenshot: Terminal showing "Database connected successfully" message](screenshots/16-database-connected.png)

> **Troubleshooting: `MongoParseError: options usenewurlparser, useunifiedtopology are not supported`**
>
> If you're using a newer version of the MongoDB Node.js driver (which Mongoose relies on), you may see this error when starting the server. The `useNewUrlParser` and `useUnifiedTopology` options were required in older driver versions but are now default behavior — recent driver versions reject them instead of silently ignoring them.
>
> **Fix:** Remove both options from the `mongoose.connect()` call so it looks like this:
>
> ```javascript
> mongoose.connect(process.env.DB)
>   .then(() => console.log('Database connected successfully'))
>   .catch(err => console.log(err));
> ```
>
> Save `index.js` and run `node index.js` again — the server should now connect successfully.

---

## Testing the Backend with Postman

Before building the frontend, it's essential to verify the backend works correctly. We'll test three endpoints using **Postman**:

- **GET** `/api/todos` — retrieve all tasks
- **POST** `/api/todos` — create a new task
- **DELETE** `/api/todos/:id` — remove a specific task

### Step 1: Install Postman and Create an Account

Download [Postman](https://www.postman.com/downloads/) and sign in.

![Screenshot: Postman dashboard after login](screenshots/17-postman-dashboard.png)

### Step 2: Test the GET Request

1. Click the **+** icon to create a new HTTP request.
2. Set the method to `GET`.
3. Enter the URL: `http://<public-ip-address>:5000/api/todos`
4. Click **Send**.

> If the server isn't running, start it with `node index.js` before sending the request.

Since the database is initially empty, you should receive an empty array `[]` as the response.

![Screenshot: Postman GET request showing an empty array response](screenshots/18-postman-get-empty.png)

### Step 3: Test the POST Request

1. Create a new request, set the method to `POST`.
2. Enter the same URL: `http://<public-ip-address>:5000/api/todos`
3. Under **Headers**, add `Content-Type: application/json`.
4. Under **Body**, select **raw** and enter JSON data:

```json
{
  "action": "My first task"
}
```

5. Click **Send**.

![Screenshot: Postman POST request with JSON body and successful response](screenshots/19-postman-post-request.png)

Repeat this to add a second task, e.g.:

```json
{
  "action": "Learning the MERN stack with Stegtech Technology Hub"
}
```

### Step 4: Verify with GET Again

Send the GET request again — you should now see both tasks returned.

![Screenshot: Postman GET request showing multiple tasks in the response](screenshots/20-postman-get-multiple.png)

### Step 5: Test the DELETE Request

1. Create a new request, set the method to `DELETE`.
2. Copy the `_id` of the task you want to remove from a previous GET response.
3. Enter the URL: `http://<public-ip-address>:5000/api/todos/<task-id>`
4. Click **Send**.

![Screenshot: Postman DELETE request with task ID in the URL](screenshots/21-postman-delete-request.png)

Send a GET request again to confirm the task was removed.

![Screenshot: Postman GET request showing the task list after deletion](screenshots/22-postman-get-after-delete.png)

> **Tip:** Attempting a DELETE request without specifying an ID will return an error, since the API needs to know exactly which task to remove.

---

## Part 2: Frontend Configuration

### Step 1: Create the React Application

From the project root directory, scaffold a new React app called `client`:

```bash
npx create-react-app client
```

This installs all necessary React dependencies in one step.

![Screenshot: Terminal output after create-react-app finishes installing](screenshots/23-create-react-app.png)

### Step 2: Install Supporting Dependencies

From the **project root** (not inside `client`), install `concurrently` and `nodemon`:

```bash
npm install concurrently --save-dev
npm install nodemon --save-dev
```

- **concurrently** — runs multiple commands (frontend + backend) simultaneously from a single terminal.
- **nodemon** — automatically restarts the server whenever backend code changes.

### Step 3: Update package.json Scripts

Open the root `package.json` and replace the `scripts` section:

```json
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```

![Screenshot: Root package.json showing updated scripts section](screenshots/24-package-json-scripts.png)

### Step 4: Configure the React Proxy

Navigate into the `client` folder and open its `package.json`. Add a `proxy` setting:

```json
"proxy": "http://localhost:5000"
```

**Why this matters:** Since the backend runs on port `5000` and the React frontend runs on port `3000`, the proxy setting allows the frontend to make API calls (e.g., `/api/todos`) without specifying the full backend URL each time. This simplifies development when running both servers on the same machine.

![Screenshot: client/package.json showing the proxy configuration](screenshots/25-client-proxy-config.png)

### Step 5: Run Both Servers Concurrently

From the project root:

```bash
npm run dev
```

You should see confirmation that the backend is connected, followed by the React development server starting.

![Screenshot: Terminal showing both backend and frontend running via npm run dev](screenshots/26-npm-run-dev.png)

### Step 6: Open Port 3000 in AWS Security Group

Just like port 5000, open port 3000 in your EC2 security group:

1. Go to **Security Groups** → **Inbound Rules**.
2. Add a new rule for port `3000`, source `Anywhere`.
3. Click **Save**.

![Screenshot: AWS Security Group inbound rules showing port 3000 opened](screenshots/27-security-group-port3000.png)

Visit `http://<public-ip-address>:3000` in your browser to see the default React welcome page.

![Screenshot: Browser showing the default React app landing page](screenshots/28-default-react-page.png)

---

### Step 7: Create React Components

Inside `client/src`, create a `components` directory with three files:

```bash
cd client/src
mkdir components
cd components
touch Input.js ListTodo.js Todo.js
```

![Screenshot: File listing showing Input.js, ListTodo.js, and Todo.js inside components/](screenshots/29-components-directory.png)

#### Input.js

Handles the input field and submission for new tasks:

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Input = () => {
  const [action, setAction] = useState('');

  const handleChange = (event) => {
    setAction(event.target.value);
  };

  const addTodo = () => {
    const task = { action };

    if (task.action && task.action.length > 0) {
      axios.post('/api/todos', task)
        .then((res) => {
          if (res.data) {
            window.location.reload();
          }
        })
        .catch((err) => console.log(err));
    }
  };

  return (
    <div>
      <input type="text" onChange={handleChange} value={action} />
      <button onClick={addTodo}>Add</button>
    </div>
  );
};

export default Input;
```

#### Step 8: Install Axios

Axios is a promise-based HTTP client used to make requests from React to the backend API.

```bash
cd client
npm install axios
```

#### ListTodo.js

Renders the list of tasks and provides deletion functionality:

```javascript
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {todos && todos.length > 0 ? (
        todos.map((todo) => (
          <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
            {todo.action}
          </li>
        ))
      ) : (
        <li>No todo(s) left</li>
      )}
    </ul>
  );
};

export default ListTodo;
```

#### Todo.js

The main component that fetches tasks and combines `Input` and `ListTodo`:

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import Input from './Input';
import ListTodo from './ListTodo';

const Todo = () => {
  const [todos, setTodos] = useState([]);

  const getTodos = () => {
    axios.get('/api/todos')
      .then((res) => setTodos(res.data))
      .catch((err) => console.log(err));
  };

  const deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then((res) => {
        if (res.data) getTodos();
      })
      .catch((err) => console.log(err));
  };

  useEffect(() => {
    getTodos();
  }, []);

  return (
    <div>
      <h1>My Todo(s)</h1>
      <Input />
      <ListTodo todos={todos} deleteTodo={deleteTodo} />
    </div>
  );
};

export default Todo;
```

![Screenshot: Todo.js, Input.js, and ListTodo.js files open in the editor](screenshots/30-components-code.png)

### Step 9: Update App.js

Replace the contents of `client/src/App.js`:

```javascript
import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
};

export default App;
```

### Step 10: Update Styling

Replace the contents of `client/src/App.css` and `client/src/index.css` with your preferred styling for the to-do list layout (input field, button, and list items).

![Screenshot: App.css and index.css with final styling applied](screenshots/31-css-styling.png)

---

## Running and Testing the Complete Application

From the project root, start both servers:

```bash
npm run dev
```

Open your browser and navigate to `http://<public-ip-address>:3000`.

![Screenshot: Fully functional To-Do app in the browser with an empty task list](screenshots/32-todo-app-empty.png)

### Add a Task

Type a task into the input field (e.g., "My first task") and click **Add**. The task should appear in the list immediately.

![Screenshot: To-Do app showing a newly added task in the list](screenshots/33-todo-app-task-added.png)

### Add Multiple Tasks

Continue adding tasks — each one should persist and display in the list.

![Screenshot: To-Do app showing multiple tasks listed](screenshots/34-todo-app-multiple-tasks.png)

### Delete a Task

Click on a task to delete it. The list should refresh and no longer display the removed task.

![Screenshot: To-Do app after deleting a task, showing the updated list](screenshots/35-todo-app-task-deleted.png)

### Verify via Postman

Send a `GET` request to `http://<public-ip-address>:5000/api/todos` to confirm the database reflects the current state of tasks shown in the UI.

![Screenshot: Postman GET request confirming database matches the UI state](screenshots/36-postman-final-verification.png)

---

## Summary

By following this guide, you have:

1. Set up a Node.js/Express.js backend server on an AWS EC2 instance.
2. Created RESTful API routes for creating, reading, and deleting tasks.
3. Modeled application data using Mongoose and connected it to a MongoDB Atlas database.
4. Tested all API endpoints using Postman.
5. Built a React frontend that communicates with the backend via a proxy configuration.
6. Deployed and tested a fully functional To-Do application using the complete MERN stack.

## Next Steps

The next stack to explore is the **MEAN stack** (MongoDB, Express.js, Angular, Node.js), which replaces React with Angular as the frontend framework.

---

## Appendix: Common Commands Reference

| Task | Command |
|---|---|
| Update packages | `sudo apt update && sudo apt upgrade -y` |
| Install Node.js | `sudo apt install nodejs -y` |
| Install NPM | `sudo apt install npm -y` |
| Initialize project | `npm init` |
| Install Express | `npm install express` |
| Install dotenv | `npm install dotenv` |
| Install Mongoose | `npm install mongoose` |
| Install dev dependencies | `npm install concurrently nodemon --save-dev` |
| Create React app | `npx create-react-app client` |
| Install Axios | `npm install axios` |
| Run both servers | `npm run dev` |
| Start server only | `node index.js` |

## Appendix: Ports Used

| Port | Purpose |
|---|---|
| 5000 | Backend (Express/Node.js API) |
| 3000 | Frontend (React development server) |
