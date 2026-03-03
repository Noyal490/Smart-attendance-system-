<!DOCTYPE html>
<html>
<head>
<title>Smart Attendance System</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
    font-family: Arial;
    background: linear-gradient(rgba(0,0,0,0.6),rgba(0,0,0,0.6)),
    url("https://images.unsplash.com/photo-1523050854058-8df90110c9f1") no-repeat center center/cover;
    text-align:center;
    color:black;
}

.container{
    background: rgba(255,255,255,0.95);
    width: 500px;
    margin:40px auto;
    padding:20px;
    border-radius:15px;
    box-shadow:0 0 15px gray;
}

input, button{
    padding:8px;
    margin:5px;
}

button{
    background:#007bff;
    color:white;
    border:none;
    border-radius:5px;
    cursor:pointer;
}

button:hover{
    background:#0056b3;
}

.student-box{
    background:#f2f2f2;
    padding:8px;
    margin:5px;
    border-radius:8px;
}

.hidden{
    display:none;
}
</style>
</head>

<body>

<!-- LOGIN PAGE -->
<div class="container" id="loginPage">
<h2>Teacher Login</h2>
<input type="text" id="username" placeholder="Username"><br>
<input type="password" id="password" placeholder="Password"><br>
<button onclick="login()">Login</button>
<p id="loginMsg" style="color:red;"></p>
</div>

<!-- MAIN PAGE -->
<div class="container hidden" id="mainPage">

<h2>Smart Attendance System</h2>

<button onclick="logout()">Logout</button>

<br><br>

<input type="text" id="studentName" placeholder="Enter Student Name">
<button onclick="addStudent()">Add Student</button>

<hr>

<h3>Select Date</h3>
<input type="date" id="attendanceDate">

<hr>

<h3>Students</h3>
<div id="studentList"></div>

<hr>

<h3>Search Student</h3>
<input type="text" id="searchName" placeholder="Student Name">
<button onclick="searchStudent()">Search</button>

<p id="result"></p>

<hr>

<h3>Attendance Chart</h3>
<canvas id="attendanceChart"></canvas>

</div>

<script>

// Load students from Local Storage
let students = JSON.parse(localStorage.getItem("students")) || {};
let chart;

// Auto load data & today's date
window.onload = function(){
    document.getElementById("attendanceDate").valueAsDate = new Date();
    displayStudents();
    updateChart();
}

// LOGIN FUNCTION
function login(){
    let user = document.getElementById("username").value.trim();
    let pass = document.getElementById("password").value.trim();

    if(user === "admin" && pass === "1234"){
        document.getElementById("loginPage").classList.add("hidden");
        document.getElementById("mainPage").classList.remove("hidden");
    }else{
        document.getElementById("loginMsg").innerHTML="Invalid Login!";
    }
}

// LOGOUT FUNCTION
function logout(){
    document.getElementById("mainPage").classList.add("hidden");
    document.getElementById("loginPage").classList.remove("hidden");
}

// ADD STUDENT
function addStudent(){
    let name = document.getElementById("studentName").value.trim();

    if(name==""){
        alert("Enter name");
        return;
    }

    if(students[name]){
        alert("Already exists");
        return;
    }

    students[name] = {};
    localStorage.setItem("students", JSON.stringify(students));

    displayStudents();
    document.getElementById("studentName").value="";
}

// DISPLAY STUDENTS
function displayStudents(){
    let list = document.getElementById("studentList");
    list.innerHTML="";

    for(let name in students){
        let div = document.createElement("div");
        div.className="student-box";

        div.innerHTML = `
        <strong>${name}</strong><br>
        <label>
        <input type="radio" name="${name}" 
        onclick="markAttendance('${name}','Present')"> Present
        </label>

        <label>
        <input type="radio" name="${name}" 
        onclick="markAttendance('${name}','Absent')"> Absent
        </label>
        `;

        list.appendChild(div);
    }
}

// MARK ATTENDANCE
function markAttendance(name,status){

    let date = document.getElementById("attendanceDate").value;

    if(date==""){
        alert("Select Date First");
        return;
    }

    students[name][date] = status;
    localStorage.setItem("students", JSON.stringify(students));

    alert(name+" marked "+status+" on "+date);
    updateChart();
}

// SEARCH STUDENT
function searchStudent(){
    let name = document.getElementById("searchName").value.trim();

    if(!students[name]){
        document.getElementById("result").innerHTML="Student not found";
        return;
    }

    let present=0;
    let total=0;

    for(let date in students[name]){
        total++;
        if(students[name][date]=="Present"){
            present++;
        }
    }

    let percent = total==0 ? 0 : ((present/total)*100).toFixed(2);

    document.getElementById("result").innerHTML=
    "Present: "+present+
    "<br>Total: "+total+
    "<br>Attendance %: "+percent+"%";
}

// UPDATE CHART
function updateChart(){

    let totalPresent=0;
    let totalAbsent=0;

    for(let name in students){
        for(let date in students[name]){
            if(students[name][date]=="Present")
                totalPresent++;
            else
                totalAbsent++;
        }
    }

    if(chart) chart.destroy();

    chart = new Chart(document.getElementById("attendanceChart"),{
        type:"pie",
        data:{
            labels:["Present","Absent"],
            datasets:[{
                data:[totalPresent,totalAbsent],
                backgroundColor:["green","red"]
            }]
        }
    });
}

</script>

</body>
</html>
