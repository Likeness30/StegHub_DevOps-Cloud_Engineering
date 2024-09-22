
## MERN WEB STACK IMPLEMENTATION ON AWS

### Introduction

The MERN stack is a popular choice for building dynamic web applications. It leverages a combination of JavaScript technologies:

__MongoDB__: A NoSQL document database for flexible data storage.
__Express.js__: A lightweight web framework for Node.js that simplifies building APIs and web applications.
__React.js__: A powerful library for creating interactive user interfaces.
__Node.js__: A JavaScript runtime environment that allows execution of server-side JavaScript code.
This guide provides a comprehensive overview of setting up and utilizing each component of the MERN stack to develop robust web applications.

## Step 0: Prerequisites
__1.__ Step 0: Prerequisites
EC2 Instance Setup

Launch an EC2 instance (t3.small, Ubuntu 24.04 LTS) in the us-east-1 region using the AWS console.
The choice of the instance type is based on:
- __Memory:__ t3.small offers more memory than t2.micro.
- __Burst Capability:__  t3 instances provide more flexible CPU burst performance.
- __Performance:__  t3.small typically offers better performance than t2.micro.

__2.__Security Group Setup
- Configure inbound rules in the security group:
- HTTP (port 80): From anywhere.
- HTTPS (port 443): From anywhere.
- SSH (port 22): From your IP address.
- Custom TCP (port 5000, 3000): From anywhere (for Node.js and React development).

__3__ The private ssh key permission was changed for the private key file and then used to connect to the instance by running
```
chmod 400 my-ec2-key.pem
```
```
ssh -i "my-ec2-key.pem" ubuntu@yourip
```
Where __username=ubuntu__ and __public ip address=yourip__

## Step 1 - Backend Configuration

__1.__ __Update and upgrade the server’s package index__

```bash
sudo apt update && sudo apt upgrade -y
```
![alt text](images/apt.update1.PNG "apt update")
![alt text](images/apt.upgrade1.PNG "apt upgrade")

__2.__ __Install Node.js
- Add Node.js repository:
```
curl fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

__3.__ __Install node.js with the command below__.

```
sudo apt-get install nodejs -y
```

![alt text](images/ "nginx landing page")

__Note:__ the above command installs both node.js and npm. NPM is a package manager for Node just as apt is a package manager for Ubuntu. It is used to install Node modules and packages and to manage dependency conflicts.

__4.__ __Verify the Node installation with the command below__.

```
node -v        // It shows the node version

npm -v        // It shows the npm version
```
![alt text](images/nod-v.PNG "Node version")
![alt text](images/npm-v.PNG "npm version")


### Application Code Setup

__1.__ __Create a new directory for the TO-DO project and switch to it. Then initialize the project directory.__
```
mkdir Todo && ls && cd Todo

npm init
```
![alt text](images/todo.1.PNG "Todo directory")

This is to initialize the project directory and in the process, creates a new file called package.json. This file will contain information about your application and the dependencies it needs to run.
Follow the prompts after running the command. You can press “Enter” several times to accept default values, then accept to write out the package.json file by typing yes.


### Install ExpressJs

Express is a framework for Node.js. It simplifies development and abstracts a lot of low level details. For example, express helps to define routes of your application based on HTTP methods and URLs.

__1.__ __Install Express using npm__

```
npm install express
```
![alt text](images/express.install.PNG "Express installation")

__2.__ __Create a file index.js and run ls to confirm the file__

```
touch index.js && ls
```

__3.__ __Install dotenv module__
```
npm install dotenv
```

![alt text](images/dotenv.1.PNG "Installation of dotenv")

__4.__ __Open index.js file__
```bash
vim index.js
```
Type the code below into it

```bash
const express = require('express');
require('dotenv').config();

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

![alt text](images/port5000.PNG "index.js running on port 5000")

__Note:__ Port 5000 have been specified to be used in the code. This was required later on the browser.

__5.__ __Start the server to see if it works. Open your terminal in the same directory as your index.js file. Run__

```bash
node index.js
```

Port 5000 has been opened in ec2 security group.

__Access the server with the public IP followed by the port__

```bash
http://yourip:5000
```

## Routes

There are three actions that the ToDo application needs to be able to do:
- Create a new task
- Display list of all task
- Delete a completed task

Each task was associated with some particular endpoint and used different standard __HTTP__ request methods: __POST__, __GET__, __DELETE__.

For each task, routes were created which defined various endpoints that the ToDo app depends on.

__1.__ __Create a folder routes, switch to routes directory and create a file api.js. Open the file__

```bash
mkdir routes && cd routes && touch api.js
```

__Copy__ the code below into the file

```
const express = require('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

});

module.exports = router;
```

## Models

A model is at the heart of JavaScript based applications and it is what makes it interactive.

Models was used to define the database schema. This is important in order be able to define the fields stored in each Mongodb document.

In essence, the schema is a blueprint of how the database is constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties.
To create a schema and a model, mongoose  was installed, which is a Node.js package that makes working with mongodb easier.

__1.__ __Change the directory back to Todo folder and install mongoose__
```
npm install mongoose
```

![alt text](images/mongose.install.PNG "mongose installation")

__2.__ __Create a new folder models, switch to models directory, create a file todo.js inside models. Open the file__

```
mkdir models && cd models && touch todo.js
```

Past the code below into the file

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

// Create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
![alt text](images/todo.PNG "todo.js script")

The routes was updated from the file api.js in the ‘routes’ directory to make use of the new model.

__3.__ __In Routes directory, open api.js and delete the code inside with :%d__.

```
vim api.js
```

Paste the new code below into it
```
const express = require('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {
  // This will return all the data, exposing only the id and action field to the client
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});

router.post('/todos', (req, res, next) => {
  if (req.body.action) {
    Todo.create(req.body)
      .then(data => res.json(data))
      .catch(next);
  } else {
    res.json({
      error: "The input field is empty"
    });
  }
});

router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({"_id": req.params.id})
    .then(data => res.json(data))
    .catch(next);
});

module.exports = router;
```

![alt text](images/api.jss.PNG "api.js update")


## MongoDB Database

__mLab__ provides MongoDB database as a service solution (DBaaS). MongoDB has two cloud database management system components: mLab and Atlas, Both were formerly cloud databases managed by MongoDB (MongoDB acquired mLab in 2018, with certain differences). In November, MongoDB merged the two cloud databases and as such, __mLab.com__ redirects to the __MongoDB Atlas website__.

__1.__ __Create a MongoDB database and collection inside mLab__

- MongoDB Cluster Overview

- MongoDB Setup
- Create a MongoDB Cluster using MongoDB Atlas

- Go to MongoDB Atlas and create a free-tier cluster in the us-east-1 region.

- Whitelist IP Addresses

- Allow access from anywhere (for testing purposes).

- Create a Database and Collection

- Create a todo_db database and todos collection.

- Connect MongoDB with Node.js

__2.__ __Create a file in your Todo directory and name it .env, open the file__
```
touch .env && vim .env
```
Add connection string below to access the database

```
DB = ‘mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority’
```
![alt text](images/db.acess.PNG "my db")
![alt text](images/db.con.PNG "my db")
![alt text](images/db.connect.PNG "My db")


__3.__ __Update the index.js to reflect the use of .env so that Node.js can connect to the database__.

```
vim index.js
```

Delete existing content in the file, and update it with the entire code below:

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log(`Database connected successfully`))
  .catch(err => console.log(err));

// Since mongoose promise is deprecated, we override it with Node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
  console.log(err);
  next();
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

```
![alt text](images/updated.ind.PNG "My db")


Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

__4.__ __Start your server using the command__
```
node index.js
```
There was a deprecation warning as diplayed in the image above.
To silent this warning, __{ useNewUrlParser: true, useUnifiedTopology: true }__ was removed from the code.
This helps iin automating the process and ensure that it runs  error free, also check for punction error or speelling errors

## Testing Backend Code without Frontend using RESTful API

Postman was used to test the backend code. Post man is used to test API calls and check  how the backend is functioning, the communication between your server and the destination server, any errors would be seen and displayed, 
Install Postman on your device, Postman has a browser interface, you can click on any functionalities to check the status of tour website through issuing of api calls and also checking the template eg, body etc

The endpoints were tested. For the endpoints that require body, JSON was sent back with the necessary fields since it’s what was set up in the code.
In you security group on AWS, go to security group and add a custom TCP path for port 5000, this would later be added to your ip to test your api calls.

__1.__ __Open Postman and Set the header__

```
http://yourip:5000/api/todos
```
![alt text](images/Post.PNG "post request")

### Make a GET requests to the API

This request retrieves all existing records from our To-Do application (backend requests these records from the database and sends us back as a response to GET request).

![alt text](images/Get.PNG "get request")

###  Check Database Collections

![alt text](images/ixv.PNG "Database collection")

### Make another GET requests to the API

![alt text](images/Get.PNG "get request")

## Step 2 - Frontend Creation

It is time to create a user interface for a Web client (browser) to interact with the application via API

__1.__ __In the same root directory as your backend code, which is the Todo directory, run:__

```
npx create-react-app client
```
![alt text](images/client.install.PNG "react installation")

or you can verbose it if is not installing 

```
npx create-react-app client --verbose
```

or you can also yarn it it is another form of installing it

```
yarn create-react-app client 
```
or you can install with vite, its is another way of installing it

```
npm create vite@latest  //I used this to install mine
```
This created a new folder in the Todo directory called client, where all the react code was added.

### Running a React App

Before testing the react app, the following dependencies needs to be installed in the project root directory.

- __Install concurrently__. It is used to run more than one command simultaneously from the same terminal window.
```
npm install concurrently --save-dev
```
![alt text](images/concurent.PNG "concurrent installation")

- __Install nodemon__. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
```
npm install nodemon --save-dev
```
![alt text](images/deamonn.PNG "deamon installation")

- In Todo folder open the package.json file, change the highlighted part of the below screenshot and replace with the code below:
```
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```
![alt text](images/prozy.PNG "Package.json update")

### Configure Proxy In package.json

- Change directory to “client”
```
cd client
```

- Open the package.json file
```
vim package.json
```

Add the key value pair in the package.json file
```
“proxy”: “http://localhost:5000
```

The whole purpose of adding the proxy configuration above is to make it possible to access the application directly from the browser by simply calling the server url like
http://locathost:5000 rather than always including the entire path like http://localhost:5000/api/todos

Ensure you are inside the Todo directory, and simply do:
```
npm run dev
```
![alt text](images/run-dev.PNG "run dev")

Go to the Ec2 console, security group create an inbounding rule for custom TCP port 3000, In the web browser behind your already used ip put :3000

The app opened and started running on localhost:3000

__Note__: In order to access the application from the internet, TCP port 3000 had been opened on EC2.


## Creating React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For the Todo app, there are two stateful components and one stateless component. From Todo directory, run:

```
cd client
```

Move to the “src” directory
```
cd src
```

__2.__ __Inside your src folder, create another folder called “components”__

```
mkdir components
```
Move into the components directory
```
cd components
```

__3.__ __Inside the ‘components’ directory create three files “Input.js”, “ListTodo.js” and “Todo.js”.__
```
touch Input.js ListTodo.js Todo.js
```

#### Open Input.js file
```
vim Input.js
```
Paste in the following:

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ""
  }

  handleChange = (event) => {
    this.setState({ action: event.target.value });
  }

  addTodo = () => {
    const task = { action: this.state.action };

    if (task.action && task.action.length > 0) {
      axios.post('/api/todos', task)
        .then(res => {
          if (res.data) {
            this.props.getTodos();
            this.setState({ action: "" });
          }
        })
        .catch(err => console.log(err));
    } else {
      console.log('Input field required');
    }
  }

  render() {
    let { action } = this.state;
    return (
      <div>
        <input type="text" onChange={this.handleChange} value={action} />
        <button onClick={this.addTodo}>add todo</button>
      </div>
    );
  }
}

export default Input;
```
![alt text](images/input.PNG "Input.js")

In oder to make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

Move to the client folder
```
cd ../..
```
__Install Axios__
```
npm install axios
```
![alt text](images/axios.PNG "axios installation)

#### Go to components directory
```
cd src/components
```

#### After that open the ListTodo.js

```
vim ListTodo.js
```
Copy and paste the following code:

```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {
        todos && todos.length > 0 ? (
          todos.map(todo => {
            return (
              <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                {todo.action}
              </li>
            );
          })
        ) : (
          <li>No todo(s) left</li>
        )
      }
    </ul>
  );
}

export default ListTodo;
```

![alt text](images/listtodo.PNG "listtodo)


#### Then in the Todo.js file, write the following code

```
vim Todo.js
```

```
import React, { Component } from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  }

  componentDidMount() {
    this.getTodos();
  }

  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if (res.data) {
          this.setState({
            todos: res.data
          });
        }
      })
      .catch(err => console.log(err));
  }

  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if (res.data) {
          this.getTodos();
        }
      })
      .catch(err => console.log(err));
  }

  render() {
    let { todos } = this.state;
    return (
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos} />
        <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
      </div>
    );
  }
}

export default Todo;
```

__We need to make a little adjustment to our react code. Delete the logo and adjust our App.js to look like this__

### Move to src folder
```
cd ..
```

Ensure to be in the src folder and run:
```
vim App.js
```

#### Copy and paste the following code

```
import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
}

export default App;

```

####  In the src directory, open the App.css

```
vim App.css
```

Paste the following code into it

```
.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 60%;
  margin-left: auto;
  margin-right: auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5rem;
  color: #787a80;
}

input:focus {
  outline: none;
}

button {
  width: 25%;
  height: 45px;
  border: none;
  margin-left: 10px;
  font-size: 25px;
  background: #101113;
  border-radius: 5px;
  color: #787a80;
  cursor: pointer;
}

button:focus {
  outline: none;
}

ul {
  list-style: none;
  text-align: left;
  padding: 15px;
  background: #171a1f;
  border-radius: 5px;
}

li {
  padding: 15px;
  font-size: 1.5rem;
  margin-bottom: 15px;
  background: #282c34;
  border-radius: 5px;
  overflow-wrap: break-word;
  cursor: pointer;
}

@media only screen and (min-width: 300px) {
  .App {
    width: 80%;
  }

  input {
    width: 100%;
  }

  button {
    width: 100%;
    margin-top: 15px;
    margin-left: 0;
  }
}

@media only screen and (min-width: 640px) {
  .App {
    width: 60%;
  }

  input {
    width: 50%;
  }

  button {
    width: 30%;
    margin-left: 10px;
    margin-top: 0;
  }
}

```

![alt text](images/2.PNG)

#### In the src directory, open the index.css

```
vim index.css
```

#### Copy and paste the code below:

```
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282c34;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
}
```

#### Go to the Todo directory
```
cd ../..
```

Run:
```
npm run dev
```

![alt text](images/run-dev.PNG "run dev)


The To-Do app is ready and fully functional with the functionality discussed earlier: Creating a task, deleting a task, and viewing all the tasks.

__The client can now be viewed in the browser__

![alt text](images/brow.PNG "viewed on browser")

__Check them on the MongoDB database__

![alt text](images/db.connect.PNG "Checking db)
