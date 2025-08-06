# Chatbot Application - Function Reference

## Table of Contents
1. [PHP Functions](#php-functions)
2. [JavaScript Functions](#javascript-functions)
3. [Database Functions](#database-functions)
4. [CSS Classes and Components](#css-classes-and-components)
5. [Configuration Functions](#configuration-functions)

## PHP Functions

### Authentication Functions

#### `login_user($username, $password)`
**File**: `index.php`
**Purpose**: Authenticate user credentials
**Parameters**:
- `$username` (string): User's username
- `$password` (string): User's password (plain text)

**Return**: Boolean (redirects on success, shows alert on failure)

**SQL Query**:
```sql
SELECT * FROM userinfotable WHERE username='$username' AND password='$password'
```

**Implementation**:
```php
if(isset($_POST['login'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];
    
    $query = "select * from userinfotable WHERE username='$username' AND password='$password'";
    $query_run = mysqli_query($con, $query);
    
    if(mysqli_num_rows($query_run) > 0) {
        // Valid credentials
        $_SESSION['username'] = $username;
        header('location:homepage.php');
    } else {
        // Invalid credentials
        echo '<script type="text/javascript"> alert("Invalid credentials") </script>';
    }
}
```

#### `register_user($username, $password, $cpassword)`
**File**: `register.php`
**Purpose**: Register new user account
**Parameters**:
- `$username` (string): Desired username
- `$password` (string): User's password
- `$cpassword` (string): Password confirmation

**Validation Steps**:
1. Check password match
2. Check username availability
3. Insert new user record

**Implementation**:
```php
if(isset($_POST['submit_btn'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];
    $cpassword = $_POST['cpassword'];
    
    if($password == $cpassword) {
        $query = "select * from userinfotable WHERE username='$username'";
        $query_run = mysqli_query($con, $query);
        
        if(mysqli_num_rows($query_run) > 0) {
            // Username exists
            echo '<script type="text/javascript"> alert("User already exists.. try another username") </script>';
        } else {
            // Register new user
            $query = "insert into userinfotable values('','$username','$password')";
            $query_run = mysqli_query($con, $query);
            
            if($query_run) {
                echo '<script type="text/javascript"> alert("User Registered.. Go to login page to login") </script>';
            } else {
                echo '<script type="text/javascript"> alert("'.mysqli_error($con).'") </script>';
            }
        }
    } else {
        echo '<script type="text/javascript"> alert("Password and confirm password does not match!") </script>';
    }
}
```

### Chatbot Core Functions

#### `get_bot_response($txt)`
**File**: `get_bot_message.php`
**Purpose**: Process user message and return bot response
**Parameters**:
- `$txt` (string): User's input message

**Algorithm**:
1. Sanitize input with `mysqli_real_escape_string()`
2. Search for matching patterns in chatbot_hints
3. Store conversation in message table
4. Return response

**Implementation**:
```php
$txt = mysqli_real_escape_string($con, $_POST['txt']);
$sql = "select reply from chatbot_hints where question like '%$txt%'";
$res = mysqli_query($con, $sql);

if(mysqli_num_rows($res) > 0) {
    $row = mysqli_fetch_assoc($res);
    $html = $row['reply'];
} else {
    $html = "Sorry not be able to understand you";
}

// Store user message
$added_on = date('Y-m-d h:i:s');
mysqli_query($con, "insert into message(message,added_on,type) values('$txt','$added_on','user')");

// Store bot response
$added_on = date('Y-m-d h:i:s');
mysqli_query($con, "insert into message(message,added_on,type) values('$html','$added_on','bot')");

echo $html;
```

### Admin Management Functions

#### `display_chat_history()`
**File**: `adminlogin.php`
**Purpose**: Retrieve and display all chat messages
**Return**: HTML table with message data

**SQL Query**:
```sql
SELECT id, message, added_on, type FROM message
```

**Implementation**:
```php
$conn = mysqli_connect("localhost", "root", "", "youtube");
$sql = "SELECT id,message,added_on,type from message";
$result = $conn->query($sql);

if($result->num_rows > 0) {
    while ($row = $result->fetch_assoc()) {
        echo "<tr>
                <td>".$row["id"]."</td>
                <td>".$row["message"]."</td>
                <td>".$row["added_on"]."</td>
                <td>".$row['type']."</td>
              </tr>";
    }
}
```

#### `display_qna_dataset()`
**File**: `qna.php`
**Purpose**: Retrieve and display chatbot question-answer pairs
**Return**: HTML table with edit/delete options

**Implementation**:
```php
$sql = "SELECT id,question,reply from chatbot_hints";
$result = $conn->query($sql);

if($result->num_rows > 0) {
    while ($row = $result->fetch_assoc()) {
        echo "<tr>
                <td>".$row["id"]."</td>
                <td>".$row["question"]."</td>
                <td>".$row["reply"]."</td>
                <td><a href='update.php?rn=$row[id]&ques=$row[question]&rep=$row[reply]'>Edit/Update</td>
                <td><a href='delete.php?rn=$row[id]' onclick='return checkdelete()'>Delete</td>
              </tr>";
    }
}
```

#### `insert_response($id, $question, $reply)`
**File**: `insert.php`
**Purpose**: Add new question-answer pair to chatbot dataset
**Parameters**:
- `$id` (int): Unique identifier
- `$question` (string): Question pattern(s)
- `$reply` (string): Bot's response

**Implementation**:
```php
if(isset($_POST['submit'])) {
    $id = $_POST['id'];
    $question = $_POST['question'];
    $reply = $_POST['reply'];
    
    $query = "insert into chatbot_hints values('$id','$question','$reply')";
    $query_run = mysqli_query($conn, $query);
    
    if($query_run) {
        echo '<script type="text/javascript"> alert("Success!") </script>';
    } else {
        echo '<script type="text/javascript"> alert("'.mysqli_error($conn).'") </script>';
    }
}
```

#### `update_response($id, $question, $reply)`
**File**: `update.php`
**Purpose**: Modify existing question-answer pair
**Parameters**:
- `$id` (int): Record identifier
- `$question` (string): Updated question pattern
- `$reply` (string): Updated response

**Implementation**:
```php
if($_GET['submit']) {
    $id = $_GET['id'];
    $question = $_GET['question'];
    $reply = $_GET['reply'];
    
    $query = "UPDATE CHATBOT_HINTS SET id='$id',question='$question',reply='$reply' WHERE id='$id'";
    $data = mysqli_query($conn, $query);
    
    if($data) {
        echo "<script>alert('Record Updated')</script>";
    } else {
        echo "Failed To Update Record";
    }
}
```

#### `delete_response($id)`
**File**: `delete.php`
**Purpose**: Remove question-answer pair from dataset
**Parameters**:
- `$id` (int): Record identifier to delete

**Implementation**:
```php
$id = $_GET['rn'];
$query = "DELETE FROM CHATBOT_HINTS WHERE ID ='$id'";
$data = mysqli_query($conn, $query);

if($data) {
    echo "<script>alert('Record Deleted from database')</script>";
} else {
    echo "<font color='red'>Failed to delete from db!!";
}
```

### Invalid Response Functions

#### `report_invalid($messages)`
**File**: `invalidans.php`
**Purpose**: Report invalid bot responses
**Parameters**:
- `$messages` (string): Description of invalid response

**Implementation**:
```php
if(isset($_POST['submit'])) {
    $messages = $_POST['messages'];
    
    $query = "insert into invalid values('','$messages')";
    $query_run = mysqli_query($conn, $query);
    
    if($query_run) {
        echo '<script type="text/javascript"> alert("Success!") </script>';
    } else {
        echo '<script type="text/javascript"> alert("'.mysqli_error($conn).'") </script>';
    }
}
```

#### `display_invalid_reports()`
**File**: `invalid.php`
**Purpose**: Display all invalid response reports
**Return**: HTML table with reported issues

**Implementation**:
```php
$sql = "SELECT id,messages from invalid";
$result = $conn->query($sql);

if($result->num_rows > 0) {
    while ($row = $result->fetch_assoc()) {
        echo "<tr>
                <td>".$row["id"]."</td>
                <td>".$row["messages"]."</td>
              </tr>";
    }
}
```

## JavaScript Functions

### Chat Interface Functions

#### `getCurrentTime()`
**File**: `homepage.php`
**Purpose**: Get formatted current time for message timestamps
**Return**: String in format "HH:MM AM/PM"

**Implementation**:
```javascript
function getCurrentTime() {
    var now = new Date();
    var hh = now.getHours();
    var min = now.getMinutes();
    var ampm = (hh >= 12) ? 'PM' : 'AM';
    hh = hh % 12;
    hh = hh ? hh : 12;
    hh = hh < 10 ? '0' + hh : hh;
    min = min < 10 ? '0' + min : min;
    var time = hh + ":" + min + " " + ampm;
    return time;
}
```

#### `send_msg()`
**File**: `homepage.php`
**Purpose**: Send user message and get bot response via AJAX
**Dependencies**: jQuery, `getCurrentTime()`

**Workflow**:
1. Get user input from `#input-me`
2. Create and display user message HTML
3. Send AJAX request to `get_bot_message.php`
4. Display bot response with "Invalid Answer?" option
5. Scroll to bottom of chat

**Implementation**:
```javascript
function send_msg() {
    jQuery('.start_chat').hide();
    var txt = jQuery('#input-me').val();
    
    // Create user message HTML
    var html = '<li class="messages-me clearfix">' +
               '<span class="message-img">' +
               '<img src="image/user_avatar.png" class="avatar-sm rounded-circle">' +
               '</span>' +
               '<div class="message-body clearfix">' +
               '<div class="message-header">' +
               '<strong class="messages-title">Me</strong>' +
               '<small class="time-messages text-muted">' +
               '<span class="minutes">' + getCurrentTime() + '</span>' +
               '</small></div>' +
               '<p class="messages-p">' + txt + '</p>' +
               '</div></li>';
    
    jQuery('.messages-list').append(html);
    jQuery('#input-me').val('');
    
    if(txt) {
        jQuery.ajax({
            url: 'get_bot_message.php',
            type: 'post',
            data: 'txt=' + txt,
            success: function(result) {
                var html = '<li class="messages-you clearfix">' +
                          '<span class="message-img">' +
                          '<img src="image/bot_avatar.png" class="avatar-sm rounded-circle">' +
                          '</span>' +
                          '<div class="message-body clearfix">' +
                          '<div class="message-header">' +
                          '<strong class="messages-title">Chatbot</strong>' +
                          '<small class="time-messages text-muted">' +
                          '<span class="minutes">' + getCurrentTime() + '</span>' +
                          '</small></div>' +
                          '<p class="messages-p">' + result + '</p>' +
                          '</div></li>' +
                          '<a href="invalidans.php" id="invalid_btn">' +
                          '<i>Invalid Answer ?</i></a>';
                
                jQuery('.messages-list').append(html);
                jQuery('.messages-box').scrollTop(jQuery('.messages-box')[0].scrollHeight);
            }
        });
    }
}
```

### Admin Functions

#### `checkdelete()`
**File**: `qna.php`
**Purpose**: Confirm deletion of dataset entries
**Return**: Boolean confirmation result

**Implementation**:
```javascript
function checkdelete() {
    return confirm('Are you Sure you want to delete this record ?');
}
```

## Database Functions

### Connection Functions

#### `database_connect()`
**File**: `database.inc.php`
**Purpose**: Establish MySQL database connection
**Configuration**: Host, username, password, database name

**Implementation**:
```php
<?php
$con = mysqli_connect('localhost', 'root', '', 'youtube');
?>
```

**Enhanced Version with Error Handling**:
```php
<?php
function database_connect() {
    $host = 'localhost';
    $username = 'root';
    $password = '';
    $database = 'youtube';
    
    $con = mysqli_connect($host, $username, $password, $database);
    
    if (!$con) {
        die("Connection failed: " . mysqli_connect_error());
    }
    
    return $con;
}

$con = database_connect();
?>
```

### Query Wrapper Functions

#### `safe_query($connection, $query, $params = [])`
**Purpose**: Execute prepared statements safely
**Parameters**:
- `$connection` (mysqli): Database connection
- `$query` (string): SQL query with placeholders
- `$params` (array): Parameter values

**Implementation**:
```php
function safe_query($connection, $query, $params = []) {
    $stmt = $connection->prepare($query);
    
    if (!$stmt) {
        die("Prepare failed: " . $connection->error);
    }
    
    if (!empty($params)) {
        $types = str_repeat('s', count($params)); // Assume all strings
        $stmt->bind_param($types, ...$params);
    }
    
    $stmt->execute();
    return $stmt->get_result();
}

// Usage example
$result = safe_query($con, "SELECT * FROM userinfotable WHERE username = ?", [$username]);
```

#### `insert_message($connection, $message, $type)`
**Purpose**: Insert message into conversation history
**Parameters**:
- `$connection` (mysqli): Database connection
- `$message` (string): Message content
- `$type` (string): 'user' or 'bot'

**Implementation**:
```php
function insert_message($connection, $message, $type) {
    $added_on = date('Y-m-d H:i:s');
    $query = "INSERT INTO message(message, added_on, type) VALUES (?, ?, ?)";
    
    $stmt = $connection->prepare($query);
    $stmt->bind_param("sss", $message, $added_on, $type);
    
    return $stmt->execute();
}
```

## CSS Classes and Components

### Layout Classes

#### `.main-wrapper`
**Purpose**: Main container for login/register forms
**Properties**:
- Centers content
- Sets background styling
- Controls form layout

#### `.myform`
**Purpose**: Form styling container
**Properties**:
- Form-specific styling
- Input alignment
- Button spacing

#### `.inner_container`
**Purpose**: Inner form content wrapper
**Properties**:
- Padding and margins
- Input field organization

### Message Interface Classes

#### `.messages-box`
**Purpose**: Chat message container
**Properties**:
```css
.messages-box {
    height: 400px;
    overflow-y: auto;
    padding: 15px;
}
```

#### `.messages-list`
**Purpose**: Unordered list for messages
**Properties**:
- List styling removal
- Message item spacing

#### `.messages-me`
**Purpose**: User message styling
**Properties**:
```css
.messages-me {
    text-align: right;
    margin-bottom: 15px;
}

.messages-me .message-img {
    float: right;
    margin-left: 10px;
}
```

#### `.messages-you`
**Purpose**: Bot message styling
**Properties**:
```css
.messages-you {
    text-align: left;
    margin-bottom: 15px;
}

.messages-you .message-img {
    float: left;
    margin-right: 10px;
}
```

#### `.avatar-sm`
**Purpose**: Avatar image sizing
**Properties**:
```css
.avatar-sm {
    width: 32px;
    height: 32px;
    border-radius: 50%;
}
```

### Navigation Classes

#### `header`
**Purpose**: Main navigation header
**Properties**:
```css
header {
    background: linear-gradient(blue, black, blue);
    padding: 10px 0;
    color: white;
}
```

#### `nav ul li`
**Purpose**: Navigation menu items
**Properties**:
```css
nav ul li {
    display: inline-block;
    margin: 0 15px;
}

nav ul li a {
    color: white;
    text-decoration: none;
    padding: 8px 12px;
    border-radius: 4px;
    transition: background-color 0.3s;
}

nav ul li a:hover {
    background-color: rgba(255, 255, 255, 0.2);
}
```

### Button Classes

#### `#login_btn`, `#register_btn`, `#admin_btn`
**Purpose**: Primary action buttons
**Properties**:
```css
#login_btn, #register_btn, #admin_btn {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    margin: 5px;
}

#login_btn:hover, #register_btn:hover, #admin_btn:hover {
    background-color: #0056b3;
}
```

### Input Classes

#### `.inputvalues`
**Purpose**: Form input field styling
**Properties**:
```css
.inputvalues {
    width: 100%;
    padding: 12px;
    margin: 8px 0;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 16px;
}

.inputvalues:focus {
    border-color: #007bff;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}
```

## Configuration Functions

### Session Management

#### `start_session()`
**Purpose**: Initialize PHP session
**Implementation**:
```php
session_start();

// Check if user is logged in
function is_logged_in() {
    return isset($_SESSION['username']);
}

// Require login for protected pages
function require_login() {
    if (!is_logged_in()) {
        header('location: index.php');
        exit();
    }
}

// Logout function
function logout_user() {
    session_destroy();
    header('location: index.php');
    exit();
}
```

### Database Configuration

#### `get_db_config()`
**Purpose**: Return database configuration array
**Implementation**:
```php
function get_db_config() {
    return [
        'host' => 'localhost',
        'username' => 'root',
        'password' => '',
        'database' => 'youtube',
        'charset' => 'utf8mb4'
    ];
}

function connect_database() {
    $config = get_db_config();
    
    $con = mysqli_connect(
        $config['host'],
        $config['username'], 
        $config['password'],
        $config['database']
    );
    
    if (!$con) {
        error_log("Database connection failed: " . mysqli_connect_error());
        die("Database connection failed");
    }
    
    mysqli_set_charset($con, $config['charset']);
    return $con;
}
```

### Error Handling Functions

#### `handle_error($error_message, $context = [])`
**Purpose**: Centralized error handling and logging
**Implementation**:
```php
function handle_error($error_message, $context = []) {
    $log_message = date('Y-m-d H:i:s') . " - ERROR: " . $error_message;
    
    if (!empty($context)) {
        $log_message .= " - Context: " . json_encode($context);
    }
    
    error_log($log_message);
    
    // Display user-friendly error in production
    if (defined('PRODUCTION') && PRODUCTION) {
        echo "An error occurred. Please try again later.";
    } else {
        echo "Error: " . $error_message;
    }
}

// Database error handler
function handle_db_error($connection, $query = '') {
    $error = mysqli_error($connection);
    handle_error("Database error: " . $error, ['query' => $query]);
}
```

### Validation Functions

#### `validate_input($data, $type = 'text')`
**Purpose**: Input validation and sanitization
**Parameters**:
- `$data` (string): Input data to validate
- `$type` (string): Validation type (text, email, password, etc.)

**Implementation**:
```php
function validate_input($data, $type = 'text') {
    // Basic sanitization
    $data = trim($data);
    $data = stripslashes($data);
    $data = htmlspecialchars($data);
    
    switch($type) {
        case 'username':
            // Allow alphanumeric and underscore, 3-20 characters
            if (!preg_match('/^[a-zA-Z0-9_]{3,20}$/', $data)) {
                return false;
            }
            break;
            
        case 'password':
            // Minimum 6 characters
            if (strlen($data) < 6) {
                return false;
            }
            break;
            
        case 'email':
            if (!filter_var($data, FILTER_VALIDATE_EMAIL)) {
                return false;
            }
            break;
    }
    
    return $data;
}

// Usage example
$username = validate_input($_POST['username'], 'username');
if ($username === false) {
    handle_error("Invalid username format");
    exit();
}
```

---

## Function Usage Examples

### Complete User Registration Flow
```php
// 1. Validate inputs
$username = validate_input($_POST['username'], 'username');
$password = validate_input($_POST['password'], 'password');
$cpassword = validate_input($_POST['cpassword'], 'password');

// 2. Check password match
if ($password !== $cpassword) {
    handle_error("Passwords do not match");
    exit();
}

// 3. Check username availability
$result = safe_query($con, "SELECT id FROM userinfotable WHERE username = ?", [$username]);
if ($result->num_rows > 0) {
    handle_error("Username already exists");
    exit();
}

// 4. Hash password (security improvement)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// 5. Insert new user
$insert_result = safe_query($con, 
    "INSERT INTO userinfotable (username, password) VALUES (?, ?)", 
    [$username, $hashed_password]
);

if ($insert_result) {
    echo "Registration successful";
} else {
    handle_error("Registration failed");
}
```

### Complete Chat Message Flow
```php
// 1. Get and validate user input
$user_message = validate_input($_POST['txt'], 'text');
if (empty($user_message)) {
    echo "Please enter a message";
    exit();
}

// 2. Search for bot response
$result = safe_query($con, 
    "SELECT reply FROM chatbot_hints WHERE question LIKE ?", 
    ['%' . $user_message . '%']
);

$bot_response = ($result->num_rows > 0) 
    ? $result->fetch_assoc()['reply']
    : "Sorry not be able to understand you";

// 3. Store conversation
insert_message($con, $user_message, 'user');
insert_message($con, $bot_response, 'bot');

// 4. Return response
echo $bot_response;
```

---

**Last Updated**: 2024-12-19
**Version**: 1.0.0