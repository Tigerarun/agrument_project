# Chatbot Application - Quick Start Guide

## 🚀 Get Started in 5 Minutes

This guide will help you set up and run the Chatbot application quickly.

## Prerequisites

Before you begin, ensure you have:
- ✅ PHP 7.4+ installed
- ✅ MySQL/MariaDB server running
- ✅ Web server (Apache/Nginx) or PHP built-in server
- ✅ Modern web browser

## Step 1: Database Setup

### 1.1 Create Database
```sql
CREATE DATABASE youtube;
USE youtube;
```

### 1.2 Import Schema
```bash
# Navigate to the Chatbot directory
cd /path/to/Chatbot

# Import main database structure
mysql -u root -p youtube < database.sql

# Import authentication structure (if using separate auth db)
mysql -u root -p youtube < authentication.sql
```

### 1.3 Create Missing Tables
If the `userinfotable` or `invalid` tables don't exist, create them:

```sql
-- User authentication table
CREATE TABLE `userinfotable` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);

-- Invalid responses table
CREATE TABLE `invalid` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `messages` text NOT NULL,
  PRIMARY KEY (`id`)
);
```

## Step 2: Configure Database Connection

Edit `database.inc.php`:
```php
<?php
$con = mysqli_connect('localhost', 'your_username', 'your_password', 'youtube');

// Test connection
if (!$con) {
    die("Connection failed: " . mysqli_connect_error());
}
?>
```

**Replace:**
- `your_username` with your MySQL username (usually 'root')
- `your_password` with your MySQL password

## Step 3: File Permissions

Ensure proper file permissions:
```bash
# Make files readable by web server
chmod 644 *.php *.css *.sql
chmod 755 image/
chmod 644 image/*.png
```

## Step 4: Start the Application

### Option A: Using PHP Built-in Server (Development)
```bash
# Navigate to Chatbot directory
cd /path/to/Chatbot

# Start PHP server
php -S localhost:8000

# Open browser to: http://localhost:8000
```

### Option B: Using Apache/Nginx (Production)
1. Copy Chatbot folder to your web server document root
2. Configure virtual host (optional)
3. Access via: `http://your-domain.com/Chatbot/`

## Step 5: Test the Application

### 5.1 Register a New User
1. Open the application in your browser
2. Click "Register" button
3. Fill in:
   - Username: `testuser`
   - Password: `password123`
   - Confirm Password: `password123`
4. Click "Sign Up"

### 5.2 Login and Chat
1. Login with your new credentials
2. Try these sample messages:
   - `Hello` → Should respond: "Hello, how are you."
   - `What is your name` → Should respond: "My name is Vishal Bot"
   - `Bye` → Should respond: "Sad to see you are going. Have a nice day"

### 5.3 Test Admin Portal
1. Go back to main page
2. Click "Login as Admin"
3. Navigate to different admin sections:
   - **Chats**: View all conversations
   - **Dataset**: Manage bot responses
   - **Invalid**: See reported issues

## Common Quick Fixes

### Database Connection Issues
```bash
# Check MySQL service
sudo systemctl status mysql
# or
sudo service mysql status

# Start MySQL if stopped
sudo systemctl start mysql
```

### Permission Denied Errors
```bash
# Fix ownership (adjust user:group for your system)
sudo chown -R www-data:www-data /path/to/Chatbot
# or
sudo chown -R apache:apache /path/to/Chatbot
```

### PHP Errors
```bash
# Check PHP error log
tail -f /var/log/php_errors.log

# Enable error display for debugging (add to top of PHP files)
ini_set('display_errors', 1);
error_reporting(E_ALL);
```

## Default Dataset

The application comes with these default responses:

| Question Patterns | Bot Response |
|-------------------|--------------|
| `HI\|\|Hello\|\|Hola` | "Hello, how are you." |
| `How are you` | "Good to see you again!" |
| `what is your name\|\|whats your name` | "My name is Vishal Bot" |
| `what should I call you` | "You can call me Vishal Bot" |
| `Where are your from` | "I m from India" |
| `Bye\|\|See you later\|\|Have a Good Day` | "Sad to see you are going. Have a nice day" |

## Adding Your First Custom Response

1. Login as admin
2. Go to "Dataset" section
3. Click "Insert/Add"
4. Add a new response:
   ```
   ID: 7
   Question: what courses||courses available||programs
   Reply: We offer Computer Science, IT, and Engineering programs.
   ```
5. Test by asking "what courses" in the chat

## Project Structure Overview

```
Chatbot/
├── index.php              # 🏠 Main login page
├── homepage.php           # 💬 Chat interface  
├── register.php           # 📝 User registration
├── get_bot_message.php    # 🤖 Core bot API
├── admin.php              # 🔐 Admin login
├── adminlogin.php         # 📊 Admin dashboard
├── qna.php                # 📚 Dataset management
├── insert.php             # ➕ Add responses
├── update.php             # ✏️ Edit responses
├── delete.php             # 🗑️ Remove responses
├── invalid.php            # ⚠️ Invalid reports
├── invalidans.php         # 📝 Report invalid
├── database.inc.php       # 🔧 DB config
├── database.sql           # 🗄️ Main schema
├── *.css                  # 🎨 Styling files
└── image/                 # 🖼️ Avatar images
```

## Next Steps

Once you have the basic application running:

1. **Enhance Security**: Implement password hashing and prepared statements
2. **Add More Responses**: Expand the chatbot's knowledge base
3. **Customize UI**: Modify CSS for your branding
4. **Deploy to Cloud**: Consider AWS, GCP, or Heroku for production
5. **Add Features**: Implement more advanced NLP or integrate with AI services

## Troubleshooting Checklist

If something isn't working:

- [ ] ✅ Database server is running
- [ ] ✅ Database credentials are correct in `database.inc.php`
- [ ] ✅ All required tables exist
- [ ] ✅ File permissions are correct
- [ ] ✅ PHP extensions (mysqli) are enabled
- [ ] ✅ Web server is serving PHP files
- [ ] ✅ No syntax errors in PHP files
- [ ] ✅ JavaScript is enabled in browser
- [ ] ✅ jQuery CDN is accessible

## Need Help?

Check the comprehensive documentation:
- 📖 [API Documentation](API_DOCUMENTATION.md)
- 🔧 [Function Reference](FUNCTION_REFERENCE.md)
- 🏗️ [Database Schema Details](API_DOCUMENTATION.md#database-schema)

---

**🎉 Congratulations!** You now have a working chatbot application. Start chatting and explore the admin features!

**Estimated Setup Time**: 5-10 minutes
**Difficulty Level**: Beginner-friendly