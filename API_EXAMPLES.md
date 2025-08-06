# Chatbot Application - API Examples & Code Samples

## Table of Contents
1. [Authentication Examples](#authentication-examples)
2. [Chat API Examples](#chat-api-examples)
3. [Admin API Examples](#admin-api-examples)
4. [JavaScript Integration Examples](#javascript-integration-examples)
5. [cURL Command Examples](#curl-command-examples)
6. [PHP Integration Examples](#php-integration-examples)
7. [Error Handling Examples](#error-handling-examples)
8. [Security Implementation Examples](#security-implementation-examples)

## Authentication Examples

### User Registration

#### Basic Registration Form
```html
<!DOCTYPE html>
<html>
<head>
    <title>Register New User</title>
</head>
<body>
    <form action="register.php" method="POST">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required 
               pattern="[a-zA-Z0-9_]{3,20}" title="3-20 characters, letters, numbers, underscore only">
        
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required minlength="6">
        
        <label for="cpassword">Confirm Password:</label>
        <input type="password" id="cpassword" name="cpassword" required>
        
        <button type="submit" name="submit_btn">Register</button>
    </form>

    <script>
        // Client-side password validation
        document.getElementById('cpassword').addEventListener('input', function() {
            const password = document.getElementById('password').value;
            const confirmPassword = this.value;
            
            if (password !== confirmPassword) {
                this.setCustomValidity('Passwords do not match');
            } else {
                this.setCustomValidity('');
            }
        });
    </script>
</body>
</html>
```

#### AJAX Registration
```javascript
async function registerUser(username, password, confirmPassword) {
    try {
        const formData = new FormData();
        formData.append('username', username);
        formData.append('password', password);
        formData.append('cpassword', confirmPassword);
        formData.append('submit_btn', '1');

        const response = await fetch('register.php', {
            method: 'POST',
            body: formData
        });

        const result = await response.text();
        
        // Check for success/error messages in response
        if (result.includes('User Registered')) {
            return { success: true, message: 'Registration successful' };
        } else if (result.includes('User already exists')) {
            return { success: false, message: 'Username already exists' };
        } else if (result.includes('does not match')) {
            return { success: false, message: 'Passwords do not match' };
        } else {
            return { success: false, message: 'Registration failed' };
        }
    } catch (error) {
        return { success: false, message: 'Network error: ' + error.message };
    }
}

// Usage
registerUser('john_doe', 'mypassword123', 'mypassword123')
    .then(result => {
        if (result.success) {
            alert('Registration successful! Please login.');
            window.location.href = 'index.php';
        } else {
            alert('Registration failed: ' + result.message);
        }
    });
```

### User Login

#### Basic Login Form
```html
<form action="index.php" method="POST" id="loginForm">
    <div class="form-group">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>
    </div>
    
    <div class="form-group">
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
    </div>
    
    <button type="submit" name="login">Login</button>
    <a href="register.php">Don't have an account? Register</a>
</form>
```

#### JavaScript Login with Session Handling
```javascript
class AuthManager {
    static async login(username, password) {
        const formData = new FormData();
        formData.append('username', username);
        formData.append('password', password);
        formData.append('login', '1');

        try {
            const response = await fetch('index.php', {
                method: 'POST',
                body: formData
            });

            // Check if redirected to homepage (successful login)
            if (response.url.includes('homepage.php')) {
                return { success: true, message: 'Login successful' };
            } else {
                const text = await response.text();
                if (text.includes('Invalid credentials')) {
                    return { success: false, message: 'Invalid username or password' };
                }
                return { success: false, message: 'Login failed' };
            }
        } catch (error) {
            return { success: false, message: 'Network error: ' + error.message };
        }
    }

    static async logout() {
        const formData = new FormData();
        formData.append('logout', '1');

        try {
            await fetch('homepage.php', {
                method: 'POST',
                body: formData
            });
            window.location.href = 'index.php';
        } catch (error) {
            console.error('Logout failed:', error);
        }
    }

    static isLoggedIn() {
        // Check if we're on a protected page
        return !window.location.pathname.includes('index.php');
    }
}

// Usage
document.getElementById('loginForm').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    
    const result = await AuthManager.login(username, password);
    
    if (result.success) {
        window.location.href = 'homepage.php';
    } else {
        alert(result.message);
    }
});
```

## Chat API Examples

### Basic Chat Integration

#### Simple Chat Message Sender
```javascript
class ChatBot {
    constructor() {
        this.messageContainer = document.querySelector('.messages-list');
        this.inputField = document.querySelector('#input-me');
        this.sendButton = document.querySelector('input[value="Send"]');
        
        this.setupEventListeners();
    }

    setupEventListeners() {
        this.sendButton.addEventListener('click', () => this.sendMessage());
        this.inputField.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.sendMessage();
            }
        });
    }

    async sendMessage() {
        const message = this.inputField.value.trim();
        if (!message) return;

        // Display user message
        this.displayMessage(message, 'user');
        this.inputField.value = '';

        // Get bot response
        try {
            const botResponse = await this.getBotResponse(message);
            this.displayMessage(botResponse, 'bot');
        } catch (error) {
            this.displayMessage('Sorry, I encountered an error. Please try again.', 'bot');
            console.error('Chat error:', error);
        }
    }

    async getBotResponse(message) {
        const formData = new FormData();
        formData.append('txt', message);

        const response = await fetch('get_bot_message.php', {
            method: 'POST',
            body: formData
        });

        if (!response.ok) {
            throw new Error('Failed to get bot response');
        }

        return await response.text();
    }

    displayMessage(message, type) {
        const timestamp = this.getCurrentTime();
        const avatar = type === 'user' ? 'user_avatar.png' : 'bot_avatar.png';
        const alignment = type === 'user' ? 'messages-me' : 'messages-you';
        const sender = type === 'user' ? 'Me' : 'Chatbot';

        const messageHTML = `
            <li class="${alignment} clearfix">
                <span class="message-img">
                    <img src="image/${avatar}" class="avatar-sm rounded-circle">
                </span>
                <div class="message-body clearfix">
                    <div class="message-header">
                        <strong class="messages-title">${sender}</strong>
                        <small class="time-messages text-muted">
                            <span class="minutes">${timestamp}</span>
                        </small>
                    </div>
                    <p class="messages-p">${message}</p>
                </div>
            </li>
        `;

        this.messageContainer.insertAdjacentHTML('beforeend', messageHTML);
        this.scrollToBottom();
    }

    getCurrentTime() {
        const now = new Date();
        let hours = now.getHours();
        const minutes = now.getMinutes();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        
        hours = hours % 12;
        hours = hours ? hours : 12;
        
        const formattedTime = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')} ${ampm}`;
        return formattedTime;
    }

    scrollToBottom() {
        const messagesBox = document.querySelector('.messages-box');
        messagesBox.scrollTop = messagesBox.scrollHeight;
    }
}

// Initialize chat
document.addEventListener('DOMContentLoaded', () => {
    new ChatBot();
});
```

#### Advanced Chat with Features
```javascript
class AdvancedChatBot extends ChatBot {
    constructor() {
        super();
        this.isTyping = false;
        this.messageHistory = [];
        this.setupAdvancedFeatures();
    }

    setupAdvancedFeatures() {
        this.addTypingIndicator();
        this.addMessageValidation();
        this.addOfflineDetection();
    }

    async sendMessage() {
        const message = this.inputField.value.trim();
        if (!message || this.isTyping) return;

        // Validate message
        if (!this.validateMessage(message)) {
            this.displaySystemMessage('Please enter a valid message (no empty or very long messages).');
            return;
        }

        // Add to history
        this.messageHistory.push({ message, type: 'user', timestamp: new Date() });

        // Display user message
        this.displayMessage(message, 'user');
        this.inputField.value = '';

        // Show typing indicator
        this.showTypingIndicator();

        try {
            const botResponse = await this.getBotResponse(message);
            this.hideTypingIndicator();
            this.displayMessage(botResponse, 'bot');
            
            // Add bot response to history
            this.messageHistory.push({ message: botResponse, type: 'bot', timestamp: new Date() });
            
            // Add invalid response option
            this.addInvalidResponseOption();
            
        } catch (error) {
            this.hideTypingIndicator();
            this.displayMessage('Sorry, I encountered an error. Please try again.', 'bot');
            console.error('Chat error:', error);
        }
    }

    validateMessage(message) {
        return message.length > 0 && message.length <= 500;
    }

    showTypingIndicator() {
        this.isTyping = true;
        const typingHTML = `
            <li class="messages-you clearfix typing-indicator">
                <span class="message-img">
                    <img src="image/bot_avatar.png" class="avatar-sm rounded-circle">
                </span>
                <div class="message-body clearfix">
                    <div class="typing-dots">
                        <span></span><span></span><span></span>
                    </div>
                </div>
            </li>
        `;
        this.messageContainer.insertAdjacentHTML('beforeend', typingHTML);
        this.scrollToBottom();
    }

    hideTypingIndicator() {
        this.isTyping = false;
        const typingIndicator = document.querySelector('.typing-indicator');
        if (typingIndicator) {
            typingIndicator.remove();
        }
    }

    displaySystemMessage(message) {
        const messageHTML = `
            <li class="system-message">
                <div class="system-message-content">${message}</div>
            </li>
        `;
        this.messageContainer.insertAdjacentHTML('beforeend', messageHTML);
        this.scrollToBottom();
    }

    addInvalidResponseOption() {
        const lastMessage = this.messageContainer.lastElementChild;
        if (lastMessage && lastMessage.classList.contains('messages-you')) {
            const invalidButton = `
                <div class="invalid-response-option">
                    <a href="#" onclick="reportInvalidResponse()" class="invalid-link">
                        <small>⚠️ Invalid Answer?</small>
                    </a>
                </div>
            `;
            lastMessage.insertAdjacentHTML('beforeend', invalidButton);
        }
    }

    exportChatHistory() {
        const chatData = {
            timestamp: new Date().toISOString(),
            messages: this.messageHistory
        };
        
        const blob = new Blob([JSON.stringify(chatData, null, 2)], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        
        const a = document.createElement('a');
        a.href = url;
        a.download = `chat_history_${Date.now()}.json`;
        a.click();
        
        URL.revokeObjectURL(url);
    }

    addOfflineDetection() {
        window.addEventListener('online', () => {
            this.displaySystemMessage('✅ Connection restored');
        });

        window.addEventListener('offline', () => {
            this.displaySystemMessage('⚠️ Connection lost. Messages will be sent when connection is restored.');
        });
    }
}

// Global function for invalid response reporting
async function reportInvalidResponse() {
    const lastBotMessage = document.querySelector('.messages-you:last-of-type .messages-p');
    const message = lastBotMessage ? lastBotMessage.textContent : 'Unknown response';
    
    const formData = new FormData();
    formData.append('messages', `Invalid response: ${message}`);
    formData.append('submit', '1');

    try {
        const response = await fetch('invalidans.php', {
            method: 'POST',
            body: formData
        });

        if (response.ok) {
            alert('Thank you for reporting this issue. We will review it.');
        } else {
            alert('Failed to report issue. Please try again.');
        }
    } catch (error) {
        alert('Network error. Please try again later.');
    }
}
```

## Admin API Examples

### Dataset Management

#### Add New Response
```javascript
async function addBotResponse(id, question, reply) {
    const formData = new FormData();
    formData.append('id', id);
    formData.append('question', question);
    formData.append('reply', reply);
    formData.append('submit', '1');

    try {
        const response = await fetch('insert.php', {
            method: 'POST',
            body: formData
        });

        const result = await response.text();
        
        if (result.includes('Success!')) {
            return { success: true, message: 'Response added successfully' };
        } else {
            return { success: false, message: 'Failed to add response' };
        }
    } catch (error) {
        return { success: false, message: 'Network error: ' + error.message };
    }
}

// Usage example
addBotResponse(7, 'hello||hi||hey', 'Hello! How can I help you today?')
    .then(result => {
        if (result.success) {
            console.log('✅ Response added');
            location.reload(); // Refresh the dataset view
        } else {
            console.error('❌ Failed to add response:', result.message);
        }
    });
```

#### Update Existing Response
```javascript
async function updateBotResponse(id, question, reply) {
    const url = `update.php?submit=1&id=${encodeURIComponent(id)}&question=${encodeURIComponent(question)}&reply=${encodeURIComponent(reply)}`;

    try {
        const response = await fetch(url, { method: 'GET' });
        const result = await response.text();
        
        if (result.includes('Record Updated')) {
            return { success: true, message: 'Response updated successfully' };
        } else {
            return { success: false, message: 'Failed to update response' };
        }
    } catch (error) {
        return { success: false, message: 'Network error: ' + error.message };
    }
}

// Usage example
updateBotResponse(3, 'what is your name||bot name||your name', 'I am Aresa Bot, your friendly assistant!')
    .then(result => {
        if (result.success) {
            alert('✅ Response updated successfully');
            location.reload();
        } else {
            alert('❌ Update failed: ' + result.message);
        }
    });
```

#### Delete Response
```javascript
async function deleteBotResponse(id) {
    if (!confirm('Are you sure you want to delete this response?')) {
        return { success: false, message: 'Deletion cancelled by user' };
    }

    try {
        const response = await fetch(`delete.php?rn=${encodeURIComponent(id)}`, {
            method: 'GET'
        });

        const result = await response.text();
        
        if (result.includes('Record Deleted')) {
            return { success: true, message: 'Response deleted successfully' };
        } else {
            return { success: false, message: 'Failed to delete response' };
        }
    } catch (error) {
        return { success: false, message: 'Network error: ' + error.message };
    }
}

// Usage example
deleteBotResponse(5)
    .then(result => {
        if (result.success) {
            alert('✅ Response deleted');
            location.reload();
        } else {
            alert('❌ Deletion failed: ' + result.message);
        }
    });
```

### Chat History Management

#### Fetch Chat History
```javascript
async function fetchChatHistory() {
    try {
        const response = await fetch('adminlogin.php');
        const html = await response.text();
        
        // Parse the HTML to extract chat data
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        const rows = doc.querySelectorAll('table tr');
        
        const chatHistory = [];
        
        // Skip header row
        for (let i = 1; i < rows.length; i++) {
            const cells = rows[i].querySelectorAll('td');
            if (cells.length >= 4) {
                chatHistory.push({
                    id: cells[0].textContent.trim(),
                    message: cells[1].textContent.trim(),
                    addedOn: cells[2].textContent.trim(),
                    type: cells[3].textContent.trim()
                });
            }
        }
        
        return chatHistory;
    } catch (error) {
        console.error('Failed to fetch chat history:', error);
        return [];
    }
}

// Usage example
fetchChatHistory().then(history => {
    console.log('Chat history:', history);
    
    // Analyze chat data
    const userMessages = history.filter(msg => msg.type === 'user');
    const botMessages = history.filter(msg => msg.type === 'bot');
    
    console.log(`Total conversations: ${userMessages.length}`);
    console.log(`Bot responses: ${botMessages.length}`);
});
```

#### Generate Chat Analytics
```javascript
class ChatAnalytics {
    static async generateReport() {
        const history = await fetchChatHistory();
        
        const analytics = {
            totalMessages: history.length,
            userMessages: history.filter(msg => msg.type === 'user').length,
            botMessages: history.filter(msg => msg.type === 'bot').length,
            messagesPerDay: this.getMessagesPerDay(history),
            popularQuestions: this.getPopularQuestions(history),
            responseRate: 0
        };
        
        analytics.responseRate = analytics.userMessages > 0 
            ? (analytics.botMessages / analytics.userMessages * 100).toFixed(2) 
            : 0;
            
        return analytics;
    }

    static getMessagesPerDay(history) {
        const dayCount = {};
        
        history.forEach(msg => {
            const date = new Date(msg.addedOn).toDateString();
            dayCount[date] = (dayCount[date] || 0) + 1;
        });
        
        return dayCount;
    }

    static getPopularQuestions(history) {
        const questionCount = {};
        
        history
            .filter(msg => msg.type === 'user')
            .forEach(msg => {
                const question = msg.message.toLowerCase();
                questionCount[question] = (questionCount[question] || 0) + 1;
            });
        
        return Object.entries(questionCount)
            .sort(([,a], [,b]) => b - a)
            .slice(0, 10)
            .map(([question, count]) => ({ question, count }));
    }
}

// Usage
ChatAnalytics.generateReport().then(report => {
    console.log('📊 Chat Analytics Report:', report);
    
    // Display in UI
    document.getElementById('total-messages').textContent = report.totalMessages;
    document.getElementById('response-rate').textContent = report.responseRate + '%';
    
    // Create charts or tables with the data
});
```

## JavaScript Integration Examples

### Real-time Chat Updates

#### WebSocket-like Polling for Updates
```javascript
class RealTimeChat {
    constructor() {
        this.lastMessageId = 0;
        this.pollingInterval = null;
        this.isActive = true;
        
        this.startPolling();
        this.setupVisibilityChange();
    }

    startPolling() {
        this.pollingInterval = setInterval(() => {
            if (this.isActive) {
                this.checkForNewMessages();
            }
        }, 2000); // Poll every 2 seconds
    }

    async checkForNewMessages() {
        try {
            const response = await fetch(`get_new_messages.php?since=${this.lastMessageId}`);
            const messages = await response.json();
            
            messages.forEach(msg => {
                this.displayMessage(msg.message, msg.type);
                this.lastMessageId = Math.max(this.lastMessageId, msg.id);
            });
        } catch (error) {
            console.error('Failed to check for new messages:', error);
        }
    }

    setupVisibilityChange() {
        document.addEventListener('visibilitychange', () => {
            this.isActive = !document.hidden;
        });
    }

    destroy() {
        if (this.pollingInterval) {
            clearInterval(this.pollingInterval);
        }
    }
}

// Note: This requires a new endpoint 'get_new_messages.php'
```

#### Auto-save Draft Messages
```javascript
class DraftManager {
    constructor(inputSelector) {
        this.input = document.querySelector(inputSelector);
        this.draftKey = 'chatbot_draft';
        
        this.loadDraft();
        this.setupAutoSave();
    }

    loadDraft() {
        const draft = localStorage.getItem(this.draftKey);
        if (draft && this.input) {
            this.input.value = draft;
        }
    }

    setupAutoSave() {
        if (!this.input) return;
        
        this.input.addEventListener('input', () => {
            const value = this.input.value;
            if (value) {
                localStorage.setItem(this.draftKey, value);
            } else {
                localStorage.removeItem(this.draftKey);
            }
        });

        // Clear draft when message is sent
        this.input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                setTimeout(() => {
                    localStorage.removeItem(this.draftKey);
                }, 100);
            }
        });
    }
}

// Initialize draft manager
new DraftManager('#input-me');
```

## cURL Command Examples

### User Registration
```bash
# Register a new user
curl -X POST http://localhost/chatbot/register.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=testuser&password=mypassword&cpassword=mypassword&submit_btn=1"

# Expected response includes: "User Registered.. Go to login page to login"
```

### Get Bot Response
```bash
# Send a message to the bot
curl -X POST http://localhost/chatbot/get_bot_message.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "txt=hello"

# Expected response: "Hello, how are you."
```

### Admin Operations
```bash
# Add new bot response
curl -X POST http://localhost/chatbot/insert.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "id=10&question=weather||forecast&reply=I cannot provide weather information yet.&submit=1"

# Update existing response
curl -X GET "http://localhost/chatbot/update.php?submit=1&id=3&question=what%20is%20your%20name&reply=I%20am%20your%20assistant"

# Delete response
curl -X GET "http://localhost/chatbot/delete.php?rn=5"

# Report invalid response
curl -X POST http://localhost/chatbot/invalidans.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "messages=Bot gave wrong answer to my question&submit=1"
```

### Testing with Session Management
```bash
# Create a session file for cookie storage
COOKIE_JAR="/tmp/chatbot_cookies.txt"

# Step 1: Login (this will set session cookies)
curl -X POST http://localhost/chatbot/index.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=testuser&password=mypassword&login=1" \
  -c "$COOKIE_JAR" \
  -L  # Follow redirects

# Step 2: Access protected page using session
curl -X GET http://localhost/chatbot/homepage.php \
  -b "$COOKIE_JAR"

# Step 3: Send chat message with session
curl -X POST http://localhost/chatbot/get_bot_message.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "txt=hello" \
  -b "$COOKIE_JAR"

# Clean up
rm "$COOKIE_JAR"
```

## PHP Integration Examples

### Custom Bot Response Handler
```php
<?php
// custom_bot_handler.php
class CustomBotHandler {
    private $connection;
    
    public function __construct($dbConnection) {
        $this->connection = $dbConnection;
    }
    
    public function processMessage($userMessage) {
        // Sanitize input
        $userMessage = $this->sanitizeInput($userMessage);
        
        // Try to find a response
        $response = $this->findResponse($userMessage);
        
        // If no response found, try AI/NLP service
        if ($response === null) {
            $response = $this->getAIResponse($userMessage);
        }
        
        // Store conversation
        $this->storeMessage($userMessage, 'user');
        $this->storeMessage($response, 'bot');
        
        return $response;
    }
    
    private function sanitizeInput($input) {
        $input = trim($input);
        $input = stripslashes($input);
        $input = htmlspecialchars($input);
        return mysqli_real_escape_string($this->connection, $input);
    }
    
    private function findResponse($message) {
        $sql = "SELECT reply FROM chatbot_hints WHERE question LIKE ?";
        $stmt = $this->connection->prepare($sql);
        $searchTerm = "%$message%";
        $stmt->bind_param("s", $searchTerm);
        $stmt->execute();
        $result = $stmt->get_result();
        
        if ($result->num_rows > 0) {
            $row = $result->fetch_assoc();
            return $row['reply'];
        }
        
        return null;
    }
    
    private function getAIResponse($message) {
        // Integrate with external AI service (OpenAI, Dialogflow, etc.)
        // For now, return default response
        return "I'm still learning. Could you please rephrase your question?";
    }
    
    private function storeMessage($message, $type) {
        $sql = "INSERT INTO message (message, added_on, type) VALUES (?, NOW(), ?)";
        $stmt = $this->connection->prepare($sql);
        $stmt->bind_param("ss", $message, $type);
        return $stmt->execute();
    }
    
    public function getConversationHistory($limit = 50) {
        $sql = "SELECT * FROM message ORDER BY added_on DESC LIMIT ?";
        $stmt = $this->connection->prepare($sql);
        $stmt->bind_param("i", $limit);
        $stmt->execute();
        return $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
    }
    
    public function getPopularQuestions($limit = 10) {
        $sql = "SELECT message, COUNT(*) as frequency 
                FROM message 
                WHERE type = 'user' 
                GROUP BY message 
                ORDER BY frequency DESC 
                LIMIT ?";
        $stmt = $this->connection->prepare($sql);
        $stmt->bind_param("i", $limit);
        $stmt->execute();
        return $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
    }
}

// Usage
include 'database.inc.php';
$botHandler = new CustomBotHandler($con);

if ($_POST['txt']) {
    $response = $botHandler->processMessage($_POST['txt']);
    echo $response;
}
?>
```

### Database Migration Helper
```php
<?php
// migrate_database.php
class DatabaseMigrator {
    private $connection;
    
    public function __construct($dbConnection) {
        $this->connection = $dbConnection;
    }
    
    public function createMissingTables() {
        $tables = [
            'userinfotable' => "
                CREATE TABLE IF NOT EXISTS `userinfotable` (
                    `id` int(11) NOT NULL AUTO_INCREMENT,
                    `username` varchar(255) NOT NULL UNIQUE,
                    `password` varchar(255) NOT NULL,
                    `email` varchar(255) DEFAULT NULL,
                    `created_at` timestamp DEFAULT CURRENT_TIMESTAMP,
                    PRIMARY KEY (`id`)
                ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
            ",
            'invalid' => "
                CREATE TABLE IF NOT EXISTS `invalid` (
                    `id` int(11) NOT NULL AUTO_INCREMENT,
                    `messages` text NOT NULL,
                    `reported_at` timestamp DEFAULT CURRENT_TIMESTAMP,
                    `status` enum('pending','reviewed','resolved') DEFAULT 'pending',
                    PRIMARY KEY (`id`)
                ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
            ",
            'admin_logs' => "
                CREATE TABLE IF NOT EXISTS `admin_logs` (
                    `id` int(11) NOT NULL AUTO_INCREMENT,
                    `action` varchar(100) NOT NULL,
                    `details` text,
                    `admin_user` varchar(100),
                    `timestamp` timestamp DEFAULT CURRENT_TIMESTAMP,
                    PRIMARY KEY (`id`)
                ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
            "
        ];
        
        foreach ($tables as $name => $sql) {
            if ($this->connection->query($sql)) {
                echo "✅ Table '$name' created or already exists\n";
            } else {
                echo "❌ Failed to create table '$name': " . $this->connection->error . "\n";
            }
        }
    }
    
    public function addIndexes() {
        $indexes = [
            "CREATE INDEX IF NOT EXISTS idx_username ON userinfotable(username)",
            "CREATE INDEX IF NOT EXISTS idx_question ON chatbot_hints(question)",
            "CREATE INDEX IF NOT EXISTS idx_message_type ON message(type)",
            "CREATE INDEX IF NOT EXISTS idx_message_date ON message(added_on)"
        ];
        
        foreach ($indexes as $sql) {
            if ($this->connection->query($sql)) {
                echo "✅ Index created\n";
            } else {
                echo "❌ Failed to create index: " . $this->connection->error . "\n";
            }
        }
    }
    
    public function seedDefaultData() {
        // Check if data already exists
        $result = $this->connection->query("SELECT COUNT(*) as count FROM chatbot_hints");
        $row = $result->fetch_assoc();
        
        if ($row['count'] > 0) {
            echo "📊 Data already exists, skipping seed\n";
            return;
        }
        
        $defaultResponses = [
            ['HI||Hello||Hola||Hey', 'Hello! How can I help you today?'],
            ['How are you||how r u', 'I\'m doing great! Thanks for asking.'],
            ['what is your name||whats your name||bot name', 'I\'m Aresa Bot, your virtual assistant.'],
            ['help||assistance||support', 'I\'m here to help! You can ask me about our services, courses, or general information.'],
            ['thank you||thanks||thx', 'You\'re welcome! Is there anything else I can help you with?'],
            ['bye||goodbye||see you', 'Goodbye! Have a wonderful day!']
        ];
        
        $sql = "INSERT INTO chatbot_hints (question, reply) VALUES (?, ?)";
        $stmt = $this->connection->prepare($sql);
        
        foreach ($defaultResponses as [$question, $reply]) {
            $stmt->bind_param("ss", $question, $reply);
            if ($stmt->execute()) {
                echo "✅ Added: $question\n";
            } else {
                echo "❌ Failed to add: $question\n";
            }
        }
    }
}

// Run migration
include 'database.inc.php';
$migrator = new DatabaseMigrator($con);

echo "🚀 Starting database migration...\n";
$migrator->createMissingTables();
$migrator->addIndexes();
$migrator->seedDefaultData();
echo "✅ Migration completed!\n";
?>
```

## Error Handling Examples

### Comprehensive Error Handler
```php
<?php
// error_handler.php
class ErrorHandler {
    private static $logFile = 'chatbot_errors.log';
    
    public static function setup() {
        set_error_handler([self::class, 'handleError']);
        set_exception_handler([self::class, 'handleException']);
        register_shutdown_function([self::class, 'handleFatalError']);
    }
    
    public static function handleError($severity, $message, $file, $line) {
        $error = [
            'type' => 'Error',
            'severity' => $severity,
            'message' => $message,
            'file' => $file,
            'line' => $line,
            'timestamp' => date('Y-m-d H:i:s'),
            'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'Unknown',
            'ip' => $_SERVER['REMOTE_ADDR'] ?? 'Unknown'
        ];
        
        self::logError($error);
        
        // Don't display errors in production
        if (defined('PRODUCTION') && PRODUCTION) {
            return true;
        }
        
        return false; // Let PHP handle the error display
    }
    
    public static function handleException($exception) {
        $error = [
            'type' => 'Exception',
            'message' => $exception->getMessage(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
            'timestamp' => date('Y-m-d H:i:s')
        ];
        
        self::logError($error);
        
        if (defined('PRODUCTION') && PRODUCTION) {
            echo "An error occurred. Please try again later.";
        } else {
            echo "<h3>Exception:</h3><pre>" . print_r($error, true) . "</pre>";
        }
    }
    
    public static function handleFatalError() {
        $error = error_get_last();
        
        if ($error && in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR])) {
            self::logError([
                'type' => 'Fatal Error',
                'message' => $error['message'],
                'file' => $error['file'],
                'line' => $error['line'],
                'timestamp' => date('Y-m-d H:i:s')
            ]);
        }
    }
    
    private static function logError($error) {
        $logEntry = json_encode($error) . "\n";
        file_put_contents(self::$logFile, $logEntry, FILE_APPEND | LOCK_EX);
    }
    
    public static function getRecentErrors($count = 50) {
        if (!file_exists(self::$logFile)) {
            return [];
        }
        
        $lines = file(self::$logFile, FILE_IGNORE_NEW_LINES);
        $recentLines = array_slice($lines, -$count);
        
        return array_map('json_decode', $recentLines);
    }
}

// Initialize error handling
ErrorHandler::setup();
?>
```

### Database Error Recovery
```javascript
class DatabaseErrorRecovery {
    static async retryOperation(operation, maxRetries = 3, delay = 1000) {
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                return await operation();
            } catch (error) {
                console.warn(`Attempt ${attempt} failed:`, error.message);
                
                if (attempt === maxRetries) {
                    throw new Error(`Operation failed after ${maxRetries} attempts: ${error.message}`);
                }
                
                // Wait before retrying
                await new Promise(resolve => setTimeout(resolve, delay * attempt));
            }
        }
    }
    
    static async sendMessageWithRetry(message) {
        return this.retryOperation(async () => {
            const formData = new FormData();
            formData.append('txt', message);
            
            const response = await fetch('get_bot_message.php', {
                method: 'POST',
                body: formData,
                timeout: 10000 // 10 second timeout
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            const text = await response.text();
            
            // Check for common error patterns
            if (text.includes('database connection') || text.includes('mysqli_connect')) {
                throw new Error('Database connection error');
            }
            
            return text;
        });
    }
}

// Usage example
DatabaseErrorRecovery.sendMessageWithRetry('hello')
    .then(response => {
        console.log('Bot response:', response);
        displayMessage(response, 'bot');
    })
    .catch(error => {
        console.error('Failed to get response:', error);
        displayMessage('Sorry, I\'m having trouble responding right now. Please try again in a moment.', 'bot');
    });
```

## Security Implementation Examples

### Input Validation and Sanitization
```php
<?php
// security.php
class SecurityHelper {
    
    public static function validateAndSanitizeInput($input, $type = 'text', $maxLength = 255) {
        // Basic sanitization
        $input = trim($input);
        $input = stripslashes($input);
        
        // Length check
        if (strlen($input) > $maxLength) {
            throw new InvalidArgumentException("Input too long. Maximum $maxLength characters allowed.");
        }
        
        switch ($type) {
            case 'username':
                if (!preg_match('/^[a-zA-Z0-9_]{3,20}$/', $input)) {
                    throw new InvalidArgumentException("Username must be 3-20 characters, alphanumeric and underscore only.");
                }
                break;
                
            case 'password':
                if (strlen($input) < 8) {
                    throw new InvalidArgumentException("Password must be at least 8 characters long.");
                }
                // Check for at least one number and one letter
                if (!preg_match('/(?=.*[a-zA-Z])(?=.*\d)/', $input)) {
                    throw new InvalidArgumentException("Password must contain at least one letter and one number.");
                }
                break;
                
            case 'email':
                if (!filter_var($input, FILTER_VALIDATE_EMAIL)) {
                    throw new InvalidArgumentException("Invalid email format.");
                }
                break;
                
            case 'message':
                // Remove potential XSS
                $input = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
                // Basic profanity filter (you can expand this)
                $badWords = ['spam', 'abuse']; // Add more as needed
                foreach ($badWords as $word) {
                    if (stripos($input, $word) !== false) {
                        throw new InvalidArgumentException("Message contains inappropriate content.");
                    }
                }
                break;
        }
        
        return $input;
    }
    
    public static function hashPassword($password) {
        return password_hash($password, PASSWORD_DEFAULT);
    }
    
    public static function verifyPassword($password, $hash) {
        return password_verify($password, $hash);
    }
    
    public static function generateCSRFToken() {
        if (!isset($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }
        return $_SESSION['csrf_token'];
    }
    
    public static function verifyCSRFToken($token) {
        return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
    }
    
    public static function sanitizeForDatabase($connection, $input) {
        return mysqli_real_escape_string($connection, $input);
    }
    
    public static function rateLimit($identifier, $maxRequests = 60, $timeWindow = 3600) {
        $key = "rate_limit_$identifier";
        $current = $_SESSION[$key] ?? ['count' => 0, 'timestamp' => time()];
        
        // Reset if time window has passed
        if (time() - $current['timestamp'] > $timeWindow) {
            $current = ['count' => 0, 'timestamp' => time()];
        }
        
        $current['count']++;
        $_SESSION[$key] = $current;
        
        if ($current['count'] > $maxRequests) {
            throw new Exception("Rate limit exceeded. Please try again later.");
        }
        
        return true;
    }
}

// Secure registration example
try {
    session_start();
    
    if ($_POST['submit_btn']) {
        // CSRF protection
        if (!SecurityHelper::verifyCSRFToken($_POST['csrf_token'])) {
            throw new Exception("Invalid security token.");
        }
        
        // Rate limiting
        $clientIP = $_SERVER['REMOTE_ADDR'];
        SecurityHelper::rateLimit("register_$clientIP", 5, 3600); // 5 registrations per hour
        
        // Validate inputs
        $username = SecurityHelper::validateAndSanitizeInput($_POST['username'], 'username');
        $password = SecurityHelper::validateAndSanitizeInput($_POST['password'], 'password');
        $cpassword = SecurityHelper::validateAndSanitizeInput($_POST['cpassword'], 'password');
        
        if ($password !== $cpassword) {
            throw new Exception("Passwords do not match.");
        }
        
        // Hash password
        $hashedPassword = SecurityHelper::hashPassword($password);
        
        // Use prepared statement
        $stmt = $con->prepare("SELECT id FROM userinfotable WHERE username = ?");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        
        if ($stmt->get_result()->num_rows > 0) {
            throw new Exception("Username already exists.");
        }
        
        // Insert new user
        $stmt = $con->prepare("INSERT INTO userinfotable (username, password) VALUES (?, ?)");
        $stmt->bind_param("ss", $username, $hashedPassword);
        
        if ($stmt->execute()) {
            echo "Registration successful!";
        } else {
            throw new Exception("Registration failed. Please try again.");
        }
    }
} catch (Exception $e) {
    error_log("Registration error: " . $e->getMessage());
    echo "Error: " . $e->getMessage();
}
?>

<!-- Add CSRF token to forms -->
<form method="POST">
    <input type="hidden" name="csrf_token" value="<?php echo SecurityHelper::generateCSRFToken(); ?>">
    <!-- other form fields -->
</form>
```

---

These examples provide a comprehensive foundation for working with the Chatbot application's APIs. Each example includes error handling, security considerations, and best practices for production use.

**Remember to:**
- Always validate and sanitize user inputs
- Use prepared statements for database queries
- Implement proper error handling
- Add rate limiting for API endpoints
- Use HTTPS in production
- Regularly update dependencies and security patches

**Last Updated**: 2024-12-19
**Version**: 1.0.0