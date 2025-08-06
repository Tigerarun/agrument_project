# Chatbot Application - Comprehensive API Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Database Schema](#database-schema)
4. [Public APIs](#public-apis)
5. [Frontend Components](#frontend-components)
6. [Admin Portal](#admin-portal)
7. [Installation & Setup](#installation--setup)
8. [Usage Examples](#usage-examples)
9. [Security Considerations](#security-considerations)
10. [Troubleshooting](#troubleshooting)

## Project Overview

The **Aresa Chatbot** is a cloud-based Student Information System that provides automated responses to student queries using artificial intelligence algorithms. The system analyzes user messages and provides appropriate responses from a predefined dataset.

### Key Features
- **User Authentication**: Secure login and registration system
- **Interactive Chat Interface**: Real-time messaging with bot responses
- **Admin Portal**: Management interface for chatbot dataset
- **Invalid Response Reporting**: Users can report incorrect bot responses
- **Message History**: Complete conversation tracking
- **Responsive UI**: Modern web interface with Bootstrap integration

### Technology Stack
- **Backend**: PHP 7.4+
- **Database**: MySQL/MariaDB
- **Frontend**: HTML5, CSS3, JavaScript (jQuery)
- **Framework**: Bootstrap 4.1.1
- **Server**: Apache/Nginx (LAMP/LEMP stack)

## System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (HTML/CSS/JS) │◄──►│   (PHP)         │◄──►│   (MySQL)       │
│                 │    │                 │    │                 │
│ - Login/Register│    │ - Authentication│    │ - User Data     │
│ - Chat Interface│    │ - Chat Logic    │    │ - Messages      │
│ - Admin Portal  │    │ - CRUD Ops      │    │ - Bot Responses │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Database Schema

### Core Tables

#### 1. `userinfotable` - User Authentication
```sql
CREATE TABLE `userinfotable` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

**Purpose**: Stores user credentials for authentication.

#### 2. `chatbot_hints` - Bot Response Dataset
```sql
CREATE TABLE `chatbot_hints` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `question` varchar(100) NOT NULL,
  `reply` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
);
```

**Purpose**: Contains question patterns and corresponding bot responses.

**Data Format**:
- Questions can contain multiple variations separated by `||`
- Example: `'HI||Hello||Hola'` → `'Hello, how are you.'`

#### 3. `message` - Conversation History
```sql
CREATE TABLE `message` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `message` text NOT NULL,
  `added_on` datetime NOT NULL,
  `type` varchar(10) NOT NULL,
  PRIMARY KEY (`id`)
);
```

**Purpose**: Stores complete conversation history.

**Types**:
- `user`: Messages from users
- `bot`: Responses from chatbot

#### 4. `invalid` - Invalid Response Reports
```sql
CREATE TABLE `invalid` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `messages` text NOT NULL,
  PRIMARY KEY (`id`)
);
```

**Purpose**: Tracks user-reported invalid bot responses for admin review.

## Public APIs

### 1. Authentication API

#### User Registration
**Endpoint**: `register.php`
**Method**: POST
**Purpose**: Register new users

**Parameters**:
```php
$_POST['username']   // string: User's chosen username
$_POST['password']   // string: User's password
$_POST['cpassword']  // string: Password confirmation
```

**Response Scenarios**:
```javascript
// Success
alert("User Registered.. Go to login page to login");

// Username exists
alert("User already exists.. try another username");

// Password mismatch
alert("Password and confirm password does not match!");

// Database error
alert("[MySQL Error Message]");
```

**Usage Example**:
```html
<form action="register.php" method="post">
    <input name="username" type="text" placeholder="Username" required/>
    <input name="password" type="password" placeholder="Password" required/>
    <input name="cpassword" type="password" placeholder="Confirm Password" required/>
    <input name="submit_btn" type="submit" value="Sign Up"/>
</form>
```

#### User Login
**Endpoint**: `index.php`
**Method**: POST
**Purpose**: Authenticate existing users

**Parameters**:
```php
$_POST['username']  // string: User's username
$_POST['password']  // string: User's password
```

**Response Logic**:
```php
// Success: Redirect to homepage.php with session
$_SESSION['username'] = $username;
header('location:homepage.php');

// Failure: Show alert
alert("Invalid credentials");
```

**Usage Example**:
```html
<form action="index.php" method="post">
    <input name="username" type="text" placeholder="Username" required/>
    <input name="password" type="password" placeholder="Password" required/>
    <input name="login" type="submit" value="Login"/>
</form>
```

### 2. Chatbot Core API

#### Get Bot Message
**Endpoint**: `get_bot_message.php`
**Method**: POST
**Purpose**: Process user messages and return bot responses

**Parameters**:
```php
$_POST['txt']  // string: User's message/query
```

**Algorithm**:
1. Sanitize input using `mysqli_real_escape_string()`
2. Search for matching patterns in `chatbot_hints` table using `LIKE '%$txt%'`
3. Return first matching reply or default "Sorry not be able to understand you"
4. Store both user message and bot response in `message` table

**SQL Query**:
```sql
SELECT reply FROM chatbot_hints WHERE question LIKE '%$txt%'
```

**Response Format**:
```php
// Direct echo of bot response
echo $html; // Bot's reply message
```

**Usage Example**:
```javascript
// jQuery AJAX call
$.ajax({
    url: 'get_bot_message.php',
    type: 'post',
    data: 'txt=' + userMessage,
    success: function(result) {
        // result contains bot's response
        displayBotMessage(result);
    }
});
```

### 3. Admin Management APIs

#### View Chat History
**Endpoint**: `adminlogin.php`
**Method**: GET
**Purpose**: Display complete conversation history

**Data Retrieved**:
```sql
SELECT id, message, added_on, type FROM message
```

**Display Format**:
```html
<table>
    <tr>
        <th>ID</th>
        <th>Message</th>
        <th>Added On</th>
        <th>Type</th>
    </tr>
    <!-- Dynamic rows for each message -->
</table>
```

#### Manage Bot Dataset
**Endpoint**: `qna.php`
**Method**: GET
**Purpose**: Display and manage chatbot response dataset

**Data Retrieved**:
```sql
SELECT id, question, reply FROM chatbot_hints
```

**Operations Available**:
- **Edit**: `update.php?rn={id}&ques={question}&rep={reply}`
- **Delete**: `delete.php?rn={id}`
- **Add New**: `insert.php`

#### Insert New Response
**Endpoint**: `insert.php`
**Method**: POST
**Purpose**: Add new question-answer pairs

**Parameters**:
```php
$_POST['id']        // int: Unique ID for the entry
$_POST['question']  // string: Question pattern(s)
$_POST['reply']     // string: Bot's response
```

**SQL Operation**:
```sql
INSERT INTO chatbot_hints VALUES('$id', '$question', '$reply')
```

#### Update Response
**Endpoint**: `update.php`
**Method**: GET (initial) / GET (submit)
**Purpose**: Modify existing question-answer pairs

**Parameters**:
```php
// Initial load
$_GET['rn']   // int: Record ID
$_GET['ques'] // string: Current question
$_GET['rep']  // string: Current reply

// Update submission
$_GET['id']       // int: Record ID
$_GET['question'] // string: Updated question
$_GET['reply']    // string: Updated reply
```

**SQL Operation**:
```sql
UPDATE chatbot_hints 
SET id='$id', question='$question', reply='$reply' 
WHERE id='$id'
```

#### Delete Response
**Endpoint**: `delete.php`
**Method**: GET
**Purpose**: Remove question-answer pairs

**Parameters**:
```php
$_GET['rn']  // int: Record ID to delete
```

**SQL Operation**:
```sql
DELETE FROM chatbot_hints WHERE id='$id'
```

### 4. Invalid Response Reporting

#### Report Invalid Answer
**Endpoint**: `invalidans.php`
**Method**: POST
**Purpose**: Allow users to report incorrect bot responses

**Parameters**:
```php
$_POST['messages']  // string: Description of invalid response
```

**SQL Operation**:
```sql
INSERT INTO invalid VALUES('', '$messages')
```

#### View Invalid Reports
**Endpoint**: `invalid.php`
**Method**: GET
**Purpose**: Admin view of invalid response reports

**Data Retrieved**:
```sql
SELECT id, messages FROM invalid
```

## Frontend Components

### 1. Chat Interface (`homepage.php`)

#### Main Chat Container
```html
<div class="card">
    <div class="card-body messages-box">
        <ul class="list-unstyled messages-list">
            <!-- Dynamic message list -->
        </ul>
    </div>
    <div class="card-header">
        <div class="input-group">
            <input id="input-me" type="text" class="form-control" 
                   placeholder="Type your message here..."/>
            <span class="input-group-append">
                <input type="button" class="btn btn-primary" 
                       value="Send" onclick="send_msg()">
            </span>
        </div>
    </div>
</div>
```

#### JavaScript Functions

##### Time Formatting
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

##### Message Sending
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
    
    // Add to message list
    jQuery('.messages-list').append(html);
    jQuery('#input-me').val('');
    
    // Send to bot
    if (txt) {
        jQuery.ajax({
            url: 'get_bot_message.php',
            type: 'post',
            data: 'txt=' + txt,
            success: function(result) {
                // Create bot response HTML
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

### 2. Styling Components

#### CSS Classes (from `style.css`, `log.css`, `navbar.css`)

##### Message Styling
```css
.messages-me {
    /* User messages - right aligned */
    text-align: right;
}

.messages-you {
    /* Bot messages - left aligned */
    text-align: left;
}

.avatar-sm {
    width: 32px;
    height: 32px;
}

.messages-box {
    height: 400px;
    overflow-y: auto;
}
```

##### Navigation Styling
```css
header {
    background: linear-gradient(blue, black, blue);
}

nav ul li {
    display: inline-block;
    margin: 0 10px;
}

nav ul li a {
    color: white;
    text-decoration: none;
}
```

## Admin Portal

### Navigation Structure
```html
<header>
    <h1>AR<span>ESA<span>BOT</span></span></h1>
    <nav>
        <ul>
            <li><a href="adminlogin.php">Chats</a></li>
            <li><a href="qna.php">Dataset</a></li>
            <li><a href="invalid.php">Invalid</a></li>
            <li><a href="index.php">Sign Out</a></li>
        </ul>
    </nav>
</header>
```

### Admin Features

#### 1. Chat Monitoring
- **File**: `adminlogin.php`
- **Purpose**: Monitor all user-bot conversations
- **Features**:
  - View message content
  - See timestamps
  - Identify message types (user/bot)
  - Track conversation flow

#### 2. Dataset Management
- **File**: `qna.php`
- **Purpose**: Manage bot response database
- **Features**:
  - View all question-answer pairs
  - Edit existing responses
  - Delete outdated entries
  - Add new responses

#### 3. Invalid Response Tracking
- **File**: `invalid.php`
- **Purpose**: Review user-reported issues
- **Features**:
  - View reported invalid responses
  - Identify problematic patterns
  - Improve bot accuracy

## Installation & Setup

### Prerequisites
```bash
- PHP 7.4 or higher
- MySQL 5.7 or higher / MariaDB 10.2+
- Apache 2.4+ or Nginx 1.14+
- Web browser with JavaScript enabled
```

### Database Setup
1. **Create Database**:
```sql
CREATE DATABASE youtube;
USE youtube;
```

2. **Import Schema**:
```bash
mysql -u root -p youtube < database.sql
mysql -u root -p youtube < authentication.sql
```

3. **Configure Database Connection**:
```php
// database.inc.php
<?php
$con = mysqli_connect('localhost', 'root', 'your_password', 'youtube');
?>
```

### File Structure
```
Chatbot/
├── index.php              # Main login page
├── homepage.php           # Chat interface
├── register.php           # User registration
├── get_bot_message.php    # Core bot API
├── admin.php              # Admin login
├── adminlogin.php         # Admin dashboard
├── qna.php                # Dataset management
├── insert.php             # Add responses
├── update.php             # Edit responses
├── delete.php             # Remove responses
├── invalid.php            # Invalid reports
├── invalidans.php         # Report invalid
├── database.inc.php       # DB config
├── database.sql           # Main schema
├── authentication.sql     # User schema
├── style.css              # Main styles
├── log.css                # Login styles
├── navbar.css             # Navigation styles
└── image/                 # Avatar images
    ├── user_avatar.png
    ├── bot_avatar.png
    └── admin.png
```

### Web Server Configuration

#### Apache (.htaccess)
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]

# Security headers
Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"
```

#### Nginx
```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/Chatbot;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Usage Examples

### Basic User Workflow

#### 1. User Registration
```html
<!-- Registration Form -->
<form action="register.php" method="post">
    <input name="username" type="text" value="john_doe" required/>
    <input name="password" type="password" value="secure123" required/>
    <input name="cpassword" type="password" value="secure123" required/>
    <input name="submit_btn" type="submit" value="Sign Up"/>
</form>
```

#### 2. User Login
```html
<!-- Login Form -->
<form action="index.php" method="post">
    <input name="username" type="text" value="john_doe" required/>
    <input name="password" type="password" value="secure123" required/>
    <input name="login" type="submit" value="Login"/>
</form>
```

#### 3. Chat Interaction
```javascript
// User sends message
function sendMessage(text) {
    $.ajax({
        url: 'get_bot_message.php',
        type: 'post',
        data: 'txt=' + text,
        success: function(response) {
            console.log('Bot response:', response);
            displayMessage(response, 'bot');
        }
    });
}

// Example usage
sendMessage("Hello"); // Returns: "Hello, how are you."
sendMessage("What is your name"); // Returns: "My name is Vishal Bot"
```

### Admin Workflow

#### 1. Adding New Responses
```html
<!-- Insert Form -->
<form action="insert.php" method="post">
    <input name="id" type="text" value="7"/>
    <input name="question" type="text" value="what courses||courses available||programs"/>
    <input name="reply" type="text" value="We offer Computer Science, IT, and Engineering programs."/>
    <input name="submit" type="submit" value="Add Response"/>
</form>
```

#### 2. Updating Existing Responses
```html
<!-- Update Form -->
<form action="update.php" method="get">
    <input name="id" type="text" value="3"/>
    <input name="question" type="text" value="what is your name||whats your name||bot name"/>
    <input name="reply" type="text" value="My name is Aresa Bot, your virtual assistant."/>
    <input name="submit" type="submit" value="Update"/>
</form>
```

#### 3. Managing Invalid Reports
```php
// View invalid reports
$sql = "SELECT id, messages FROM invalid";
$result = $conn->query($sql);

while ($row = $result->fetch_assoc()) {
    echo "Report ID: " . $row["id"];
    echo "Issue: " . $row["messages"];
    // Admin can then update the corresponding chatbot_hints entry
}
```

### API Integration Examples

#### CURL Examples
```bash
# User registration
curl -X POST http://your-domain.com/register.php \
     -d "username=testuser&password=test123&cpassword=test123&submit_btn=1"

# Get bot response
curl -X POST http://your-domain.com/get_bot_message.php \
     -d "txt=hello"
```

#### JavaScript Fetch API
```javascript
// Register user
async function registerUser(username, password) {
    const formData = new FormData();
    formData.append('username', username);
    formData.append('password', password);
    formData.append('cpassword', password);
    formData.append('submit_btn', '1');

    const response = await fetch('register.php', {
        method: 'POST',
        body: formData
    });
    
    return response.text();
}

// Get bot response
async function getBotResponse(message) {
    const formData = new FormData();
    formData.append('txt', message);

    const response = await fetch('get_bot_message.php', {
        method: 'POST',
        body: formData
    });
    
    return response.text();
}
```

## Security Considerations

### Current Security Measures
1. **Input Sanitization**: Uses `mysqli_real_escape_string()` for SQL inputs
2. **Session Management**: PHP sessions for user authentication
3. **Password Storage**: Plain text (⚠️ **SECURITY RISK**)

### Recommended Security Improvements

#### 1. Password Hashing
```php
// Registration
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Login verification
if (password_verify($password, $hashed_password)) {
    // Valid login
}
```

#### 2. Prepared Statements
```php
// Instead of string concatenation
$stmt = $con->prepare("SELECT * FROM userinfotable WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
```

#### 3. CSRF Protection
```php
// Generate token
$csrf_token = bin2hex(random_bytes(32));
$_SESSION['csrf_token'] = $csrf_token;

// Verify token
if (!hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    die('CSRF token mismatch');
}
```

#### 4. Input Validation
```php
function validateInput($data) {
    $data = trim($data);
    $data = stripslashes($data);
    $data = htmlspecialchars($data);
    return $data;
}
```

#### 5. SQL Injection Prevention
```php
// Use prepared statements for all database queries
$stmt = $con->prepare("SELECT reply FROM chatbot_hints WHERE question LIKE ?");
$search_term = "%$txt%";
$stmt->bind_param("s", $search_term);
$stmt->execute();
```

### Security Checklist
- [ ] Implement password hashing
- [ ] Use prepared statements
- [ ] Add CSRF protection
- [ ] Validate all inputs
- [ ] Implement rate limiting
- [ ] Add HTTPS support
- [ ] Set secure session cookies
- [ ] Implement proper error handling
- [ ] Add logging for security events
- [ ] Regular security updates

## Troubleshooting

### Common Issues

#### 1. Database Connection Errors
**Error**: `connection failed: Access denied`
**Solution**:
```php
// Check database.inc.php configuration
$con = mysqli_connect('localhost', 'correct_username', 'correct_password', 'youtube');

// Test connection
if (!$con) {
    die("Connection failed: " . mysqli_connect_error());
}
```

#### 2. Session Issues
**Error**: User not staying logged in
**Solution**:
```php
// Ensure session_start() is called
session_start();

// Check session variables
if (!isset($_SESSION['username'])) {
    header('location: index.php');
    exit();
}
```

#### 3. AJAX Not Working
**Error**: Bot responses not appearing
**Solution**:
```javascript
// Check jQuery is loaded
if (typeof jQuery === 'undefined') {
    console.error('jQuery is not loaded');
}

// Add error handling
$.ajax({
    url: 'get_bot_message.php',
    type: 'post',
    data: 'txt=' + txt,
    success: function(result) {
        console.log('Success:', result);
    },
    error: function(xhr, status, error) {
        console.error('AJAX Error:', error);
    }
});
```

#### 4. CSS/JavaScript Not Loading
**Error**: Styling or functionality broken
**Solution**:
```html
<!-- Check file paths -->
<link href="style.css" rel="stylesheet">
<script src="//cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>

<!-- Use absolute paths if needed -->
<link href="/chatbot/style.css" rel="stylesheet">
```

#### 5. Invalid Response Reporting
**Error**: "Invalid Answer?" link not working
**Solution**:
```php
// Ensure invalid table exists
CREATE TABLE `invalid` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `messages` text NOT NULL,
  PRIMARY KEY (`id`)
);

// Check form submission
if (isset($_POST['submit'])) {
    $messages = mysqli_real_escape_string($conn, $_POST['messages']);
    $query = "INSERT INTO invalid VALUES('', '$messages')";
    // Execute query
}
```

### Performance Optimization

#### 1. Database Indexing
```sql
-- Add indexes for better performance
CREATE INDEX idx_username ON userinfotable(username);
CREATE INDEX idx_question ON chatbot_hints(question);
CREATE INDEX idx_message_type ON message(type);
```

#### 2. Caching Bot Responses
```php
// Simple in-memory cache
$response_cache = array();

function getCachedResponse($question) {
    global $response_cache;
    
    $cache_key = md5(strtolower($question));
    
    if (isset($response_cache[$cache_key])) {
        return $response_cache[$cache_key];
    }
    
    // Query database
    $sql = "SELECT reply FROM chatbot_hints WHERE question LIKE '%$question%'";
    $result = mysqli_query($con, $sql);
    
    if (mysqli_num_rows($result) > 0) {
        $row = mysqli_fetch_assoc($result);
        $response_cache[$cache_key] = $row['reply'];
        return $row['reply'];
    }
    
    return "Sorry not be able to understand you";
}
```

#### 3. Optimize AJAX Calls
```javascript
// Debounce function to prevent spam
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// Debounced message sending
const debouncedSendMsg = debounce(send_msg, 300);
```

### Monitoring and Logging

#### 1. Error Logging
```php
// Enable error logging
ini_set('log_errors', 1);
ini_set('error_log', '/path/to/error.log');

// Custom logging function
function logError($message, $context = array()) {
    $log_entry = date('Y-m-d H:i:s') . ' - ' . $message;
    if (!empty($context)) {
        $log_entry .= ' - Context: ' . json_encode($context);
    }
    error_log($log_entry);
}

// Usage
logError('Database connection failed', array('host' => 'localhost', 'db' => 'youtube'));
```

#### 2. Usage Analytics
```php
// Track message volume
$sql = "SELECT COUNT(*) as message_count, DATE(added_on) as date 
        FROM message 
        GROUP BY DATE(added_on) 
        ORDER BY date DESC";

// Track popular questions
$sql = "SELECT message, COUNT(*) as frequency 
        FROM message 
        WHERE type = 'user' 
        GROUP BY message 
        ORDER BY frequency DESC 
        LIMIT 10";
```

---

## Conclusion

This comprehensive documentation covers all aspects of the Chatbot application, from basic usage to advanced administration. The system provides a solid foundation for a student information chatbot with room for enhancements in security, performance, and AI capabilities.

For support or contributions, please refer to the project repository or contact the development team.

**Last Updated**: $(date '+%Y-%m-%d')
**Version**: 1.0.0
**Author**: Chatbot Development Team