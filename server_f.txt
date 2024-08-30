const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const fs = require('fs');
const xlsx = require('xlsx');

const app = express();
app.use(cors());
app.use(bodyParser.json());

const excelFilePath = './users.xlsx';

// Helper function to read users from Excel file
function readUsers() {
    if (fs.existsSync(excelFilePath)) {
        const workbook = xlsx.readFile(excelFilePath);
        const sheet = workbook.Sheets[workbook.SheetNames[0]];
        return xlsx.utils.sheet_to_json(sheet);
    }
    return [];
}

// Helper function to write users to Excel file
function writeUsers(users) {
    const worksheet = xlsx.utils.json_to_sheet(users);
    const workbook = xlsx.utils.book_new();
    xlsx.utils.book_append_sheet(workbook, worksheet, 'Users');
    xlsx.writeFile(workbook, excelFilePath);
}

// Register route
app.post('/register', (req, res) => {
    const { username, password } = req.body;
    const users = readUsers();
    
    if (users.some(user => user.username === username)) {
        return res.status(400).json({ message: 'User already exists!' });
    }

    users.push({ username, password });
    writeUsers(users);
    res.json({ message: 'Registration successful!' });
});

// Login route
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    const users = readUsers();
    
    const user = users.find(user => user.username === username && user.password === password);
    if (user) {
        res.json({ message: 'Login successful!' });
    } else {
        res.status(400).json({ message: 'Invalid username or password!' });
    }
});

// Password reset route
app.post('/reset-password', (req, res) => {
    const { username, newPassword } = req.body;
    const users = readUsers();
    
    const user = users.find(user => user.username === username);
    if (user) {
        user.password = newPassword;
        writeUsers(users);
        res.json({ message: 'Password reset successful!' });
    } else {
        res.status(400).json({ message: 'Username not found!' });
    }
});

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
