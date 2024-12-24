<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Senate Exam Scheduler for H.L.</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1 {
            text-align: center;
        }
        .scheduler {
            display: flex;
            justify-content: space-between;
            overflow-x: auto;
        }
        .day-section {
            margin-right: 20px;
            width: 200px;
            border: 1px solid #ddd;
            border-radius: 5px;
            padding: 10px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }
        .day-title {
            font-weight: bold;
            text-align: center;
            margin-bottom: 10px;
        }
        .timeslot-container {
            display: flex;
            flex-direction: column;
        }
        .timeslot {
            padding: 10px;
            margin: 2px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            cursor: pointer;
            background-color: #f9f9f9;
        }
        .timeslot:hover {
            background-color: #f0f0f0;
        }
        .busy {
            background-color: #f44336;
            color: white;
        }
        .disabled {
            background-color: #ccc;
            color: #666;
            cursor: not-allowed;
        }
        .button-container {
            margin-top: 20px;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }
        .button-row {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }
        .button {
            padding: 10px 15px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .complete {
            background-color: #4CAF50;
            color: white;
        }
        .instructions {
            font-size: 16px;
            font-weight: bold;
            color: #333;
        }
    </style>
</head>
<body>
    <h1>Senate Exam Scheduler for H.L.</h1>
    <div id="scheduler" class="scheduler">
        <!-- Days and times will be generated here -->
    </div>
    <div class="button-container">
        <div class="button-row">
            <div class="instructions">
                Done removing the time? Click here:
            </div>
            <button class="button complete" id="completeButton">Complete!</button>
        </div>
    </div>

    <script type="module">
        // Import Firebase SDKs
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
        import { getDatabase, ref, push } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

        // Your Firebase configuration
        const firebaseConfig = {
            apiKey: "AIzaSyCMavtnBIXE7BMx5kHOrP2ULeCuLSr87mY",
            authDomain: "senate-scheduler-hl.firebaseapp.com",
            projectId: "senate-scheduler-hl",
            storageBucket: "senate-scheduler-hl.firebasestorage.app",
            messagingSenderId: "435338750351",
            appId: "1:435338750351:web:3d64809041cc6076da03fa",
            measurementId: "G-Q78FRMZQQX"
        };

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        const scheduler = document.getElementById('scheduler');
        const completeButton = document.getElementById('completeButton');
        const dates = ['2025-02-03', '2025-02-04', '2025-02-05', '2025-02-06', '2025-02-07'];
        const times = ['08:00', '08:30', '09:00', '09:30', '10:00', '10:30', '11:00', '11:30', '12:00'];
        const selections = {}; // Store busy times

        // Initialize selections and disable rules
        const disabledTimes = {
            '2025-02-03': ['09:00', '09:30'],
            '2025-02-04': ['11:00'],
            '2025-02-07': ['10:00', '10:30', '11:00', '11:30', '12:00']
        };

        dates.forEach(date => {
            selections[date] = {};
            times.forEach(time => {
                selections[date][time] = false; // Initially free
            });
        });

        // Generate schedule
        function generateSchedule() {
            scheduler.innerHTML = '';
            dates.forEach(date => {
                const daySection = document.createElement('div');
                daySection.classList.add('day-section');

                const title = document.createElement('div');
                title.classList.add('day-title');
                title.textContent = date;
                daySection.appendChild(title);

                const timeslotContainer = document.createElement('div');
                timeslotContainer.classList.add('timeslot-container');

                times.forEach(time => {
                    const slot = document.createElement('div');
                    slot.classList.add('timeslot');
                    slot.textContent = time;

                    if (disabledTimes[date] && disabledTimes[date].includes(time)) {
                        slot.classList.add('disabled');
                    } else {
                        slot.onclick = () => toggleBusyTime(date, time, slot);
                    }

                    timeslotContainer.appendChild(slot);
                });

                daySection.appendChild(timeslotContainer);
                scheduler.appendChild(daySection);
            });
        }

        // Toggle busy time selection
        function toggleBusyTime(date, time, slot) {
            const isBusy = selections[date][time];
            selections[date][time] = !isBusy;
            slot.classList.toggle('busy', !isBusy);
        }

        // Handle "Complete!" button click
        completeButton.addEventListener('click', () => {
            const response = {};
            dates.forEach(date => {
                response[date] = {};
                times.forEach(time => {
                    if (selections[date][time]) {
                        response[date][time] = "Busy";
                    } else if (!disabledTimes[date] || !disabledTimes[date].includes(time)) {
                        response[date][time] = "Available";
                    } else {
                        response[date][time] = "Unavailable";
                    }
                });
            });

            // Push to Firebase Realtime Database
            const responsesRef = ref(db, 'professorResponses');
            push(responsesRef, response);

            // Show alert to professor
            alert("Thank you for your time! I will make the google invite once everyone set up their schedules♪");
        });

        // Initialize schedule
        generateSchedule();
    </script>
</body>
</html>
