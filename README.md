ammu-store/
 ├── server.js
 ├── users.json
 ├── public/
 │   ├── index.html
 │   ├── login.html
 │   ├── dashboard.html
 │   ├── style.css
 │   ├── script.js
 │   ├── game1.html
 │   ├── game2.html
 const express = require("express");
const fs = require("fs");
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());
app.use(express.static("public"));

let users = JSON.parse(fs.readFileSync("users.json"));

// Register
app.post("/register", (req, res) => {
    const { username, password } = req.body;
    users.push({ username, password, paid: false });
    fs.writeFileSync("users.json", JSON.stringify(users));
    res.send("User registered");
});

// Login
app.post("/login", (req, res) => {
    const { username, password } = req.body;
    const user = users.find(u => u.username === username && u.password === password);

    if (user) {
        res.json(user);
    } else {
        res.send("Invalid login");
    }
});

// Fake Payment (2$ course unlock)
app.post("/pay", (req, res) => {
    const { username } = req.body;
    users = users.map(u => {
        if (u.username === username) u.paid = true;
        return u;
    });

    fs.writeFileSync("users.json", JSON.stringify(users));
    res.send("Payment Success");
});

app.listen(3000, () => console.log("Server running on port 3000"));
[]
<!DOCTYPE html>
<html>
<head>
  <title>Ammu Store</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

<h1>Welcome to Ammu Store 🚀</h1>

<a href="login.html">Login</a>

<h2>Courses</h2>
<ul>
  <li>Free Editing Course - Available</li>
  <li>Paid Editing Course ($2)</li>
</ul>

<h2>Puzzle Games</h2>
<a href="game1.html">Game 1</a><br>
<a href="game2.html">Game 2</a>

</body>
</html>

<!DOCTYPE html>
<html>
<body>

<h2>Login</h2>

<input id="username" placeholder="Username"><br>
<input id="password" placeholder="Password"><br>

<button onclick="login()">Login</button>

<script>
function login() {
    fetch("/login", {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({
            username: document.getElementById("username").value,
            password: document.getElementById("password").value
        })
    })
    .then(res => res.json())
    .then(data => {
        localStorage.setItem("user", JSON.stringify(data));
        window.location.href = "dashboard.html";
    });
}
</script>

</body>
</html>
<!DOCTYPE html>
<html>
<body>

<h1>Dashboard</h1>

<div id="content"></div>

<button onclick="buyCourse()">Buy Paid Course ($2)</button>

<script>
let user = JSON.parse(localStorage.getItem("user"));

if(user.paid){
    document.getElementById("content").innerHTML =
    "<h2>Paid Course Unlocked 🎉</h2>";
}else{
    document.getElementById("content").innerHTML =
    "<h2>Free Course Only</h2>";
}

function buyCourse(){
    fetch("/pay", {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({username: user.username})
    })
    .then(() => alert("Payment Success"));
}
</script>

</body>
</html>
<!DOCTYPE html>
<html>
<body>

<h2>Guess Number (1-10)</h2>
<input id="guess">
<button onclick="check()">Check</button>

<p id="result"></p>

<script>
let num = Math.floor(Math.random()*10)+1;

function check(){
    let g = document.getElementById("guess").value;
    if(g == num){
        document.getElementById("result").innerText = "Correct 🎉";
    } else {
        document.getElementById("result").innerText = "Try again ❌";
    }
}
</script>

</body>
</html>
<!DOCTYPE html>
<html>
<body>

<h2>Memory Game</h2>

<button onclick="start()">Start</button>
<p id="show"></p>

<input id="answer" placeholder="Repeat number">
<button onclick="check()">Check</button>

<script>
let num;

function start(){
    num = Math.floor(Math.random()*1000);
    document.getElementById("show").innerText = num;
    setTimeout(() => {
        document.getElementById("show").innerText = "";
    }, 2000);
}

function check(){
    let ans = document.getElementById("answer").value;
    alert(ans == num ? "Correct" : "Wrong");
}
</script>

</body>
</html>
body {
  font-family: Arial;
  text-align: center;
  background: #111;
  color: white;
}

a {
  color: cyan;
}
