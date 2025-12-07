# Backend Code Flow & How It Works

A beginner-friendly guide to understanding the Python FastAPI backend code.

## 📚 Table of Contents

1. [What is FastAPI?](#what-is-fastapi)
2. [Project Structure](#project-structure)
3. [How FastAPI Works](#how-fastapi-works)
4. [Code Flow Explained](#code-flow-explained)
5. [File-by-File Breakdown](#file-by-file-breakdown)
6. [Request Flow Examples](#request-flow-examples)
7. [Key Concepts](#key-concepts)

---

## 🚀 What is FastAPI?

**FastAPI** is a modern Python web framework for building APIs (Application Programming Interfaces). Think of it as a way to create a "server" that your frontend can talk to.

### Simple Analogy

Imagine a restaurant:
- **Frontend (Next.js)** = The customer ordering food
- **Backend (FastAPI)** = The kitchen that prepares the food
- **API Endpoints** = The menu items you can order
- **JSON File** = The pantry where ingredients are stored

When you click "Add Task" in the frontend, it's like ordering food. The frontend sends a request to the backend, the backend processes it, saves it to the JSON file, and sends back a response.

---

## 📁 Project Structure

```
backend/
├── main.py           # The main FastAPI application (entry point)
├── database.py       # Functions that read/write the JSON file
└── requirements.txt  # Python packages needed
```

**Simple explanation:**
- `main.py` = The "restaurant" (handles all orders/requests)
- `database.py` = The "kitchen staff" (does the actual work with data)
- `requirements.txt` = Shopping list of ingredients (packages)

---

## 🔄 How FastAPI Works

### Basic Concept

FastAPI uses **decorators** (special Python syntax) to define routes. A route is like a door - when someone knocks (sends a request), FastAPI opens the door and runs the function.

```python
@app.get("/api/todos")  # This is a "decorator" - it tells FastAPI: "When someone visits /api/todos, run this function"
async def get_todos():
    return todos
```

### HTTP Methods

- **GET** = "Give me data" (like reading a book)
- **POST** = "Create something new" (like writing a new page)
- **PUT** = "Update something" (like editing a page)
- **DELETE** = "Remove something" (like tearing out a page)

---

## 📖 Code Flow Explained

### The Big Picture

```
1. Frontend sends request → 2. FastAPI receives it → 3. Calls database function → 4. Returns response → 5. Frontend updates UI
```

### Step-by-Step Example: Adding a Task

1. **User clicks "Add Task"** in the frontend
2. **Frontend sends POST request** to `http://localhost:8000/api/todos`
3. **FastAPI receives request** at the `POST /api/todos` endpoint
4. **FastAPI validates data** (checks if title is provided, etc.)
5. **FastAPI calls `database.py`** function to save the task
6. **Database function** reads JSON file, adds new task, writes back
7. **FastAPI returns** the new task as JSON
8. **Frontend receives response** and updates the UI

---

## 📄 File-by-File Breakdown

### 1. `main.py` - The Main Application

This is the **entry point** of your backend. It's like the "reception desk" of a hotel.

#### What it does:

1. **Creates the FastAPI app**
   ```python
   app = FastAPI(title="Advanced To-Do API")
   ```
   - This creates your API application
   - Think of it as opening your restaurant for business

2. **Sets up CORS (Cross-Origin Resource Sharing)**
   ```python
   app.add_middleware(CORSMiddleware, allow_origins=["*"])
   ```
   - **What is CORS?** It's like a bouncer at a club
   - By default, browsers block requests from different origins (different ports/domains)
   - This tells the browser: "It's okay, let requests from the frontend through"
   - `allow_origins=["*"]` means "allow requests from anywhere" (for development)

3. **Defines Data Models (Pydantic)**
   ```python
   class TodoBase(BaseModel):
       title: str
       priority: str
   ```
   - **What is Pydantic?** It's like a form validator
   - It ensures data coming in matches what we expect
   - If someone sends invalid data, FastAPI automatically rejects it
   - Example: If `title` is required but missing, FastAPI returns an error

4. **Creates API Endpoints**
   - Each endpoint is a function decorated with `@app.get()`, `@app.post()`, etc.
   - These are like different "services" your API offers

#### Key Endpoints:

**GET `/api/todos`** - Get all tasks
```python
@app.get("/api/todos")
async def get_todos():
    todos = get_all_todos()  # Calls database function
    return todos              # Returns JSON response
```

**POST `/api/todos`** - Create a new task
```python
@app.post("/api/todos")
async def create_todo(todo_data: TodoCreate):
    # Validate and clean the data
    # Call database function to save
    # Return the new task
```

**PUT `/api/todos/{id}`** - Update a task
```python
@app.put("/api/todos/{todo_id}")
async def update_todo(todo_id: int, todo_update: TodoUpdate):
    # Find the task by ID
    # Update only the fields provided
    # Save and return updated task
```

**DELETE `/api/todos/{id}`** - Delete a task
```python
@app.delete("/api/todos/{todo_id}")
async def delete_todo(todo_id: int):
    # Find and remove the task
    # Return success message
```

### 2. `database.py` - Data Operations

This file handles **all interactions with the JSON file**. Think of it as the "file clerk" who reads and writes files.

#### Key Functions:

**`load_todos()`** - Read from JSON file
```python
def load_todos() -> List[Dict[str, Any]]:
    # Opens todos.json file
    # Reads the content
    # Converts JSON string to Python list/dict
    # Returns the data
```

**What it does:**
1. Checks if `data/todos.json` exists
2. Opens and reads the file
3. Parses JSON (converts string to Python objects)
4. Returns list of todos
5. If file doesn't exist, returns empty list `[]`

**`save_todos(todos)`** - Write to JSON file
```python
def save_todos(todos: List[Dict[str, Any]]) -> None:
    # Takes Python list/dict
    # Converts to JSON string
    # Writes to file
```

**What it does:**
1. Ensures `data/` folder exists (creates if needed)
2. Converts Python objects to JSON string
3. Writes to `data/todos.json`
4. Saves with nice formatting (indented)

**`get_next_id(todos)`** - Generate new ID
```python
def get_next_id(todos: List[Dict[str, Any]]) -> int:
    # Finds the highest ID in the list
    # Returns that ID + 1
```

**Example:**
- If todos have IDs: [1, 2, 5]
- Returns: 6 (5 + 1)

**`create_todo(todo_data)`** - Create and save a new todo
```python
def create_todo(todo_data: Dict[str, Any]) -> Dict[str, Any]:
    todos = load_todos()           # 1. Read existing todos
    new_id = get_next_id(todos)    # 2. Generate ID
    new_todo = {**todo_data, "id": new_id}  # 3. Create todo object
    todos.append(new_todo)         # 4. Add to list
    save_todos(todos)              # 5. Save to file
    return new_todo                # 6. Return the new todo
```

**`update_todo(id, updates)`** - Update existing todo
```python
def update_todo(todo_id: int, updates: Dict[str, Any]) -> Optional[Dict[str, Any]]:
    todos = load_todos()                    # 1. Read all todos
    # Find the todo with matching ID
    # Update only the fields provided in 'updates'
    # Save back to file
    # Return updated todo
```

**`delete_todo(id)`** - Remove a todo
```python
def delete_todo(todo_id: int) -> bool:
    todos = load_todos()                    # 1. Read all todos
    todos = [t for t in todos if t["id"] != todo_id]  # 2. Filter out the one to delete
    save_todos(todos)                      # 3. Save updated list
    return True                            # 4. Return success
```

---

## 🔍 Request Flow Examples

### Example 1: Getting All Todos

```
Frontend Request:
  GET http://localhost:8000/api/todos

Backend Flow:
  1. FastAPI receives request at @app.get("/api/todos")
  2. Calls get_todos() function
  3. Which calls get_all_todos() from database.py
  4. get_all_todos() calls load_todos()
  5. load_todos() reads data/todos.json
  6. Returns list of todos
  7. FastAPI converts to JSON
  8. Sends response to frontend

Frontend Response:
  [
    {"id": 1, "title": "Buy groceries", "completed": false},
    {"id": 2, "title": "Finish homework", "completed": true}
  ]
```

### Example 2: Creating a New Todo

```
Frontend Request:
  POST http://localhost:8000/api/todos
  Body: {
    "title": "New task",
    "priority": "high",
    "category": "Work"
  }

Backend Flow:
  1. FastAPI receives POST request
  2. Validates data using TodoCreate model (checks title exists, etc.)
  3. Calls create_todo() function
  4. Which calls db_create_todo() from database.py
  5. db_create_todo() calls:
     - load_todos() to read existing todos
     - get_next_id() to generate new ID
     - Creates new todo object
     - Appends to list
     - save_todos() to write to file
  6. Returns the new todo with ID

Frontend Response:
  {
    "id": 3,
    "title": "New task",
    "priority": "high",
    "category": "Work",
    "completed": false
  }
```

### Example 3: Updating a Todo

```
Frontend Request:
  PUT http://localhost:8000/api/todos/1
  Body: {
    "completed": true
  }

Backend Flow:
  1. FastAPI receives PUT request with ID=1
  2. Validates todo_id is a number
  3. Calls update_todo(1, {"completed": true})
  4. Which calls db_update_todo() from database.py
  5. db_update_todo() calls:
     - load_todos() to read all todos
     - Finds todo with id=1
     - Updates only the "completed" field (keeps other fields)
     - save_todos() to write back
  6. Returns updated todo

Frontend Response:
  {
    "id": 1,
    "title": "Buy groceries",
    "completed": true,  // ← Updated!
    ...
  }
```

### Example 4: Deleting a Todo

```
Frontend Request:
  DELETE http://localhost:8000/api/todos/1

Backend Flow:
  1. FastAPI receives DELETE request with ID=1
  2. Validates todo_id
  3. Calls delete_todo(1)
  4. Which calls db_delete_todo() from database.py
  5. db_delete_todo() calls:
     - load_todos() to read all todos
     - Filters out todo with id=1
     - save_todos() to write updated list
  6. Returns {"success": true}

Frontend Response:
  {"success": true}
```

---

## 🎓 Key Concepts

### 1. Async/Await

You'll see `async def` in the code. This means the function can "wait" for slow operations (like reading files) without blocking.

```python
async def get_todos():
    todos = get_all_todos()  # This might take time
    return todos
```

**Simple explanation:** It's like ordering food and waiting at your table instead of standing at the counter.

### 2. Pydantic Models

These are like "templates" that define what data should look like.

```python
class TodoCreate(BaseModel):
    title: str  # Must be a string
    priority: str = "medium"  # Optional, defaults to "medium"
```

**Benefits:**
- Automatic validation (rejects invalid data)
- Automatic documentation (FastAPI generates API docs)
- Type hints (helps catch errors)

### 3. Error Handling

FastAPI automatically handles errors:

```python
try:
    # Do something
except Exception as e:
    raise HTTPException(status_code=500, detail="Error message")
```

**HTTP Status Codes:**
- `200` = Success
- `201` = Created (new resource)
- `400` = Bad Request (invalid data)
- `404` = Not Found (resource doesn't exist)
- `500` = Server Error (something went wrong)

### 4. JSON Serialization

FastAPI automatically converts Python objects to JSON:

```python
# Python dict
todo = {"id": 1, "title": "Task"}

# FastAPI automatically converts to JSON
return todo  # Frontend receives: {"id": 1, "title": "Task"}
```

### 5. Path Parameters

`{todo_id}` in the URL becomes a function parameter:

```python
@app.put("/api/todos/{todo_id}")
async def update_todo(todo_id: int):  # FastAPI extracts {todo_id} from URL
    # todo_id is automatically converted to int
```

**Example:**
- URL: `/api/todos/5`
- `todo_id` = 5

### 6. Request Body

Data sent in POST/PUT requests:

```python
async def create_todo(todo_data: TodoCreate):
    # FastAPI automatically parses JSON body
    # Validates it matches TodoCreate model
    # Passes it as todo_data parameter
```

---

## 🔧 How to Read the Code

### When you see `@app.get("/api/todos")`:

1. `@app` = The FastAPI application
2. `.get` = HTTP method (GET request)
3. `("/api/todos")` = The URL path
4. The function below = What happens when someone visits that URL

### When you see `async def`:

- `async` = This function can wait for slow operations
- `def` = It's a function definition
- The function name = What it does

### When you see `->`:

```python
def get_todos() -> List[Dict[str, Any]]:
```

- This is a **type hint**
- It says "this function returns a list of dictionaries"
- Helps with code clarity (not required, but good practice)

### When you see `Optional[...]`:

```python
def update_todo(...) -> Optional[Dict[str, Any]]:
```

- Means "returns either a dictionary OR None"
- Used when something might not be found

---

## 🎯 Common Patterns

### Pattern 1: Read-Modify-Write

Most database operations follow this pattern:

```python
# 1. Read
todos = load_todos()

# 2. Modify
todos.append(new_todo)

# 3. Write
save_todos(todos)
```

### Pattern 2: Find-Update-Save

For updates:

```python
# 1. Read all
todos = load_todos()

# 2. Find the one to update
for i, todo in enumerate(todos):
    if todo["id"] == todo_id:
        # 3. Update it
        todos[i] = {**todo, **updates}
        break

# 4. Save
save_todos(todos)
```

### Pattern 3: Validate-Process-Respond

All endpoints follow this:

```python
# 1. Validate input (FastAPI does this automatically)
# 2. Process (call database functions)
# 3. Respond (return data or error)
```

---

## 🐛 Debugging Tips

### 1. Check if backend is running

Visit: http://localhost:8000/docs

If you see API documentation, it's running!

### 2. Check JSON file

Look at `data/todos.json` - it should update when you create/update todos.

### 3. Check terminal output

FastAPI shows all requests in the terminal:
```
INFO:     127.0.0.1:52341 - "GET /api/todos HTTP/1.1" 200 OK
```

### 4. Use the interactive docs

Visit http://localhost:8000/docs and try the endpoints directly!

---

## 📝 Summary

**The Backend in 3 Steps:**

1. **FastAPI (`main.py`)** receives HTTP requests
2. **Validates** the data using Pydantic models
3. **Database functions (`database.py`)** read/write the JSON file
4. **FastAPI** sends back a JSON response

**The Flow:**
```
Request → Validate → Process → Save → Respond
```

**Key Files:**
- `main.py` = Handles requests (the "receptionist")
- `database.py` = Handles data (the "file clerk")

**That's it!** FastAPI makes it simple - you define endpoints, and it handles the rest automatically.

---

## 🚀 Next Steps

1. **Try the interactive docs**: http://localhost:8000/docs
2. **Read the code**: Start with `main.py`, then `database.py`
3. **Add a new endpoint**: Copy an existing one and modify it
4. **Experiment**: Change things and see what happens!

**Remember:** The best way to learn is by doing. Don't be afraid to break things - you can always fix them! 💪

