# stupidAssignment
SUBMIT.cgi AND RETRIEVEAPPTS.cgi ARE IN /srv/www/cgi-bin/
INDEX.html AND TEACHERSCHEDULES.TXTARE IN ~AILAN/PUBLIC_HTML/

**Submit.cgi
//This is what locks the file, then checks if the appt is already scheduled.

#!/bin/bash

read x
echo $x > ~ailan/public_html/cgilog
teacher=`echo $x | cut -d'&' -f1 | cut -d'=' -f2`
student=`echo $x | cut -d'&' -f2 | cut -d'=' -f2 | sed 's/+/ /g'`
studentname=`echo $student | sed 's/ //g'`
day=`echo $x | cut -d'&' -f3 | cut -d'=' -f2`
scheduledTime=`echo $x | cut -d'&' -f4 | cut -d'=' -f2 | sed 's/%3A/:/'`

ref=`echo $teacher$day$scheduledTime`
appointment=`echo $teacher$day$scheduledTime$studentname`

list=`cat ~ailan/public_html/teacherschedules.txt`

lockfile ~ailan/public_html/teacherschedules
if grep -q $ref ~ailan/public_html/teacherschedules
then
  echo "content-type:  text/html

<html><head><style>body {background-color: red;}</style><link rel="stylesheet" href="assets/css/main.css"><title>Failure</title></head>
<body><h1>Dr. $teacher has an appointment scheduled at that time already</h1><script src="assets/js/main.js"></script></body>
</html>"
else
  echo $appointment >> ~ailan/public_html/teacherschedules
  echo "content-type:  text/html

<html><head><style> body {background-color: green;}</style><link rel="stylesheet" href="assets/css/main.css" /><title>Success!</title></head>
<body><p>$appointment  $list</p><h1>$student has successfully scheduled an appointment with Dr. $teacher for $day at $scheduledTime</h1><h3>Please print this page out as confirmation for your appointment.</h3><script src="assets/js/main.js"></script></body>
<html>"
fi
rm -f ~ailan/public_html/teacherschedules.lock

**retrieveappt.cgi
//This is only so I can bring up the appointments to view on the site, so I don't actually have to be at the host

#!/bin/bash

appts=`cat ~ailan/public_html/teacherschedules`

echo "content-type:  text/html

<html><head><title>Appointments</title></head>
<body<p>$appts</p></body
</html>"

**INDEX.HTML
//This is really straight forward, free css template with minimal JS, but everything of importance is here. It doesn't validate if the drop down boxes or text boxes actually have text, but that would not be hard to implement, I didn't because this is the bare bones requirements.
<!DOCTYPE html>
<html>
<head>
<link rel="stylesheet" href="assets/css/main.css">
 
<script>
        function getContactInfo() {  
            var e, val;
            
            e = document.getElementById("Teachers").value;
            var x = document.getElementById("schedule-form");
            x.style.display = "block";
            
            switch (e) {
                case "Pinet":
                    document.getElementById("contactInfo").innerHTML = "Phone: 625-3059<br>E-mail: pinet-w@mssu.edu"
                    break;
                case "Oakes":
                    document.getElementById("contactInfo").innerHTML = "Phone: 625-9683<br>E-mail: oakes-j@mssu.edu"
                    break;
                case "Collins":
                    document.getElementById("contactInfo").innerHTML = "Phone: 625-9661<br>E-mail: collins-j@mssu.edu"
                    break;
                case "Herr":
                    document.getElementById("contactInfo").innerHTML = "Phone: 625-9663<br>E-mail: herr-d@mssu.edu"
                    break;
                case "Mysterio":
                    document.getElementById("contactInfo").innerHTML = "Phone: 555-1234<br>E-mail: mysterio@mssu.edu"
                    break;
                default:
                    document.getElementById("contactInfo").innerHTML = "Phone:<br>E-mail:"
                    x.style.display = "block";
            }
        }
</script>
</head>
<body>
 
<h2>Select an instructor from the dropdown list</h2>
 
<form id="schedule-form" onChange="getContactInfo()" method="post" action="http://72.24.41.69/cgi-bin/submit.cgi">
        <select name="teachers" id="Teachers">
        <option value="" selected="selected">Please select a teacher</option>
                <option value="Pinet">Bill Pinet</option>
                <option value="Oakes">Jack Oakes</option>
                <option value="Collins">James Collins</option>
                <option value="Herr">Dennis Herr</option>
        <option value="Mysterio">Mysterio</option>
        </select>
    <br>
    
    <p id="contactInfo">Phone:<br>E-mail:</p>
 
    <input type="text" name="name" id="name" placeholder="Enter your name">
    <br>
    <select name="days" id="weekDays">
        <option value="">Please select a day of the week</option>
        <option value="Monday">Monday</option>
        <option value="Tuesday">Tuesday</option>
        <option value="Wednesday">Wednesday</option>
        <option value="Thursday">Thursday</option>
        <option value="Friday">Friday</option>
    </select>
    <br>
    <select name="chosenTime" id="timesAvail">
        <option value="">Please select a time of the work day</option>
        <option value="09:00">9:00</option>
        <option value="09:30">9:30</option>
        <option value="10:00">10:00</option>
        <option value="10:30">10:30</option>
        <option value="11:00">11:00</option>
        <option value="01:30">1:30</option>
        <option value="02:00">2:00</option>
        <option value="02:30">2:30</option>
    </select>
    <br>
    <input type="Submit" value="Schedule">
</form>
<br><br>
<form id="retrieve-appointments" method="post" action="http://72.24.41.69/cgi-bin/retrieveappts.cgi">
    <input type="Submit" value="View Appointments">
</form>
 
<script src="assets/js/main.js"></script>
</body>
</html>

**teachersschedules.txt
//This is also straight forward. Data entered from index.html goes to submit.cgi where it forms a string that is saved in the schedules. It is very straight forward and the order of the data entered is important.

Here are some Scheduled Appointments formatted as:

TeacherDayTimeStudent

PinetMonday09:00Charlie
PinetMonday09:30Zack
CollinsMonday09:00Zack
CollinsWednesday01:30Ailan
MysterioFriday02:30Charlie
PinetWednesday10:00Harold
