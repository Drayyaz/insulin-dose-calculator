# insulin-dose-calculator
calculation of insulin dose automatically
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Insulin Dose Calculator</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f9fc;
            color: #333;
        }
        
        h1, h2 {
            color: #2c3e50;
            text-align: center;
        }
        
        .container {
            background: white;
            border-radius: 10px;
            padding: 25px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        
        .toggle-buttons {
            display: flex;
            justify-content: center;
            margin-bottom: 20px;
        }
        
        .toggle-btn {
            padding: 10px 20px;
            border: none;
            background: #e0e0e0;
            cursor: pointer;
            font-weight: bold;
        }
        
        .toggle-btn.active {
            background: #3498db;
            color: white;
        }
        
        .calculator-section {
            display: none;
        }
        
        .calculator-section.active {
            display: block;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
        }
        
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }
        
        button {
            background: #3498db;
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            width: 100%;
            margin-top: 10px;
            transition: background 0.3s;
        }
        
        button:hover {
            background: #2980b9;
        }
        
        .result {
            margin-top: 20px;
            padding: 15px;
            background: #e8f4fc;
            border-radius: 5px;
            border-left: 4px solid #3498db;
        }
        
        @media (max-width: 600px) {
            body {
                padding: 10px;
            }
            
            .container {
                padding: 15px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Insulin Dose Calculator</h1>
        
        <div class="toggle-buttons">
            <button class="toggle-btn active" onclick="showSection('opd')">Outpatient</button>
            <button class="toggle-btn" onclick="showSection('ipd')">Inpatient</button>
        </div>
        
        <!-- Outpatient Calculator -->
        <div id="opd-calculator" class="calculator-section active">
            <h2>Outpatient Mode</h2>
            
            <div class="form-group">
                <label for="weight">Weight (kg):</label>
                <input type="number" id="weight" step="0.1" min="30" max="200">
            </div>
            
            <div class="form-group">
                <label for="diabetesType">Diabetes Type:</label>
                <select id="diabetesType">
                    <option value="type1">Type 1</option>
                    <option value="type2">Type 2</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="currentBG">Current Blood Glucose (mg/dL):</label>
                <input type="number" id="currentBG" min="50" max="500">
            </div>
            
            <div class="form-group">
                <label for="targetBG">Target Blood Glucose (mg/dL):</label>
                <input type="number" id="targetBG" value="120" min="70" max="200">
            </div>
            
            <div class="form-group">
                <label for="carbs">Carbohydrates in Meal (grams):</label>
                <input type="number" id="carbs" min="0" max="200">
            </div>
            
            <button onclick="calculateOPD()">Calculate Dose</button>
            
            <div id="opd-result" class="result" style="display: none;">
                <h3>Recommended Insulin Dose</h3>
                <p>Total Daily Dose (TDD): <span id="tdd"></span> units</p>
                <p>Basal Insulin: <span id="basal"></span> units</p>
                <p>Bolus Insulin: <span id="bolus"></span> units</p>
                <p>Carb Ratio (CIR): 1 unit per <span id="cir"></span>g carbs</p>
                <p>Insulin Sensitivity Factor (ISF): 1 unit lowers BG by <span id="isf"></span> mg/dL</p>
                <p><strong>Total Pre-meal Dose: <span id="totalDose"></span> units</strong></p>
            </div>
        </div>
        
        <!-- Inpatient Calculator -->
        <div id="ipd-calculator" class="calculator-section">
            <h2>Inpatient Sliding Scale</h2>
            
            <div class="form-group">
                <label for="ipdBG">Current Blood Glucose (mg/dL):</label>
                <input type="number" id="ipdBG" min="50" max="500">
            </div>
            
            <div class="form-group">
                <label for="mealTime">Time of Next Dose:</label>
                <select id="mealTime">
                    <option value="prebreakfast">Pre-breakfast</option>
                    <option value="prelunch">Pre-lunch</option>
                    <option value="predinner">Pre-dinner</option>
                    <option value="bedtime">Bedtime</option>
                </select>
            </div>
            
            <button onclick="calculateIPD()">Calculate Sliding Scale Dose</button>
            
            <div id="ipd-result" class="result" style="display: none;">
                <h3>Sliding Scale Insulin Dose (8-hourly)</h3>
                <p>Current Blood Glucose: <span id="currentIPDBG"></span> mg/dL</p>
                <p>Recommended Dose: <span id="ssDose"></span> units</p>
                <p>Next Dose Time: <span id="nextDoseTime"></span></p>
                <p style="color: #666;">Calculation: BG ÷ 18 = <span id="ssCalc"></span></p>
                <p style="color: #e74c3c; font-weight: bold;">Monitor BG before next dose and adjust as needed.</p>
            </div>
        </div>
    </div>

    <script>
        // Toggle between OPD and IPD calculators
        function showSection(section) {
            document.querySelectorAll('.toggle-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelectorAll('.calculator-section').forEach(sec => {
                sec.classList.remove('active');
            });
            
            event.target.classList.add('active');
            document.getElementById(section + '-calculator').classList.add('active');
        }

        // Outpatient calculation
        function calculateOPD() {
            const weight = parseFloat(document.getElementById('weight').value);
            const diabetesType = document.getElementById('diabetesType').value;
            const currentBG = parseFloat(document.getElementById('currentBG').value);
            const targetBG = parseFloat(document.getElementById('targetBG').value);
            const carbs = parseFloat(document.getElementById('carbs').value);

            // Validate inputs
            if (!weight || !currentBG || !carbs) {
                alert("Please fill all required fields");
                return;
            }

            // Calculate TDD
            const tddMultiplier = diabetesType === 'type1' ? 0.5 : 0.8;
            const TDD = Math.round(weight * tddMultiplier);

            // Split into basal/bolus
            const basal = Math.round(TDD * 0.5);
            const bolusTotal = TDD - basal;

            // Calculate ratios
            const CIR = Math.round(500 / TDD);
            const ISF = Math.round(1800 / TDD);

            // Calculate dose components
            const carbDose = Math.round((carbs / CIR) * 10) / 10;
            const correctionDose = Math.round(((currentBG - targetBG) / ISF) * 10) / 10;
            let totalDose = Math.round((carbDose + correctionDose) * 10) / 10;
            
            // Ensure dose isn't negative
            totalDose = totalDose < 0 ? 0 : totalDose;

            // Display results
            document.getElementById('tdd').textContent = TDD;
            document.getElementById('basal').textContent = basal;
            document.getElementById('bolus').textContent = bolusTotal;
            document.getElementById('cir').textContent = CIR;
            document.getElementById('isf').textContent = ISF;
            document.getElementById('totalDose').textContent = totalDose;
            document.getElementById('opd-result').style.display = 'block';
        }

        // Inpatient sliding scale calculation
        function calculateIPD() {
            const ipdBG = parseFloat(document.getElementById('ipdBG').value);
            const mealTime = document.getElementById('mealTime').value;

            if (!ipdBG) {
                alert("Please enter current blood glucose");
                return;
            }

            // Calculate sliding scale dose (BG/18)
            const rawDose = ipdBG / 18;
            const roundedDose = Math.round(rawDose * 2) / 2; // Round to nearest 0.5

            // Calculate next dose time
            const doseTimes = {
                'prebreakfast': '08:00 AM',
                'prelunch': '12:00 PM',
                'predinner': '06:00 PM',
                'bedtime': '10:00 PM'
            };
            const nextDose = doseTimes[mealTime];

            // Display results
            document.getElementById('currentIPDBG').textContent = ipdBG;
            document.getElementById('ssDose').textContent = roundedDose;
            document.getElementById('ssCalc').textContent = `${ipdBG} ÷ 18 = ${rawDose.toFixed(1)}`;
            document.getElementById('nextDoseTime').textContent = nextDose;
            document.getElementById('ipd-result').style.display = 'block';
        }
    </script>
</body>
</html>
