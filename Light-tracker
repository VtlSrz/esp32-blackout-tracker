#include <WiFi.h>
#include <WebServer.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>
#include <ESPmDNS.h>
#include <Preferences.h>

Preferences preferences;

// Web server on port 80
WebServer server(80);

// Variables
String ssid = "Light-Tracker"; 
String password = "slavaukraini";
bool isConnected = false;
unsigned long lastConnectionAttempt = 0;
const unsigned long connectionTimeout = 30000; // 30 seconds timeout

// Telegram Bot Configuration
String telegramBotToken = "";
String telegramChatID = "";
const char* telegramServer = "api.telegram.org";

// Light control settings
String customTextOn = "Light ON";
String customTextOff = "Light OFF";
bool includeDate = true;
bool includeTime = true;
bool includeVoltage = false;
bool includePercentage = false;
bool includeTimeCount = false;
bool autoLightControl = false;

// GPIO Pins
const int BATTERY_ADC_PIN = 2;    // GPIO2 for battery voltage reading
const int POWER_DETECT_PIN = 3;   // GPIO3 for detecting 5V charging power
const int LED_PIN = 8;            // GPIO8 for status LED (optional)

// Battery monitoring
float batteryVoltage = 0.0;
int batteryPercentage = 0;
bool isCharging = false;
const float BATTERY_MIN_VOLTAGE = 3.3;  // Minimum voltage for Li-ion battery
const float BATTERY_MAX_VOLTAGE = 4.2;  // Maximum voltage for Li-ion battery
const float VOLTAGE_DIVIDER_RATIO = 2.0; // Adjust based on your voltage divider

// Time counting
unsigned long lastLightOnTime = 0;
unsigned long lastLightOffTime = 0;
unsigned long hoursWithLight = 0;
unsigned long hoursWithoutLight = 0;
bool lightCurrentlyOn = false;

// mDNS hostname
const char* mdnsHostname = "light-tracker";

// Debouncing for power detection
unsigned long lastPowerChangeTime = 0;
const unsigned long DEBOUNCE_DELAY = 1000; // 1 second debounce
bool lastPowerState = false;

// HTML Page
const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta charset="UTF-8">
    <title>ESP32-C3 Light Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            min-height: 100vh;
        }
        .container {
            max-width: 900px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
        }
        .status {
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 20px;
            text-align: center;
            font-weight: bold;
        }
        .connected {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        .disconnected {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        .ap {
            background: #cce5ff;
            color: #004085;
            border: 1px solid #b8daff;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            color: #555;
            font-weight: bold;
        }
        input[type="text"],
        input[type="password"],
        input[type="number"] {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            box-sizing: border-box;
            transition: border 0.3s;
        }
        input:focus {
            border-color: #667eea;
            outline: none;
        }
        button {
            padding: 14px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            margin: 5px;
        }
        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            width: 100%;
        }
        .btn-light {
            background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
            color: white;
            flex: 1;
        }
        .btn-dark {
            background: linear-gradient(135deg, #f44336 0%, #d32f2f 100%);
            color: white;
            flex: 1;
        }
        .btn-warning {
            background: linear-gradient(135deg, #ff9800 0%, #f57c00 100%);
            color: white;
            margin-top: 10px;
        }
        .btn-info {
            background: linear-gradient(135deg, #17a2b8 0%, #138496 100%);
            color: white;
        }
        .btn-charging {
            background: linear-gradient(135deg, #ffc107 0%, #ff9800 100%);
            color: black;
        }
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }
        .info {
            margin-top: 20px;
            padding: 15px;
            background: #e7f3ff;
            border-radius: 8px;
            font-size: 14px;
            color: #004085;
        }
        .current-info {
            margin-top: 10px;
            font-size: 14px;
            color: #666;
        }
        .password-container {
            position: relative;
        }
        .toggle-btn {
            position: absolute;
            right: 10px;
            top: 12px;
            background: none;
            border: none;
            color: #667eea;
            cursor: pointer;
            font-size: 14px;
            width: auto;
            padding: 0;
        }
        .toggle-btn:hover {
            transform: none;
            box-shadow: none;
            color: #764ba2;
        }
        .button-row {
            display: flex;
            gap: 10px;
            margin: 20px 0;
        }
        .section {
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 1px solid #eee;
        }
        .section-title {
            font-size: 18px;
            color: #333;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 2px solid #667eea;
        }
        .telegram-config {
            background: #f9f9f9;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        .telegram-status {
            padding: 10px;
            border-radius: 5px;
            text-align: center;
            margin: 10px 0;
            display: none;
        }
        .telegram-success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        .telegram-error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        .telegram-info {
            background: #d1ecf1;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }
        .timestamp {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
            font-style: italic;
        }
        .credentials-display {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            margin-top: 15px;
            border: 1px solid #dee2e6;
        }
        .credential-item {
            margin: 5px 0;
            font-family: monospace;
        }
        .mdns-info {
            background: #e8f5e8;
            color: #2e7d32;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 15px;
            text-align: center;
            border: 1px solid #c8e6c9;
        }
        .reset-btn {
            background: linear-gradient(135deg, #dc3545 0%, #c82333 100%);
            color: white;
            margin-top: 20px;
        }
        .checkbox-group {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            margin: 15px 0;
            padding: 10px;
            background: #f8f9fa;
            border-radius: 8px;
        }
        .checkbox-item {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .checkbox-saving {
            animation: checkboxSave 0.5s ease;
        }
        @keyframes checkboxSave {
            0% { background-color: #d4edda; }
            100% { background-color: transparent; }
        }
        .custom-text-row {
            display: flex;
            gap: 15px;
            margin: 15px 0;
        }
        .custom-text-group {
            flex: 1;
        }
        .preview-box {
            background: #f0f0f0;
            padding: 12px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px dashed #ccc;
            font-family: monospace;
            font-size: 14px;
            min-height: 60px;
        }
        .preview-label {
            font-weight: bold;
            color: #666;
            margin-bottom: 5px;
        }
        .timestamp-options {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
        }
        .battery-info {
            background: #fff3cd;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px solid #ffeaa7;
        }
        .battery-display {
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
        }
        .battery-item {
            text-align: center;
            padding: 10px;
            background: white;
            border-radius: 5px;
            flex: 1;
            margin: 0 5px;
        }
        .battery-value {
            font-size: 24px;
            font-weight: bold;
            color: #333;
        }
        .battery-label {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
        }
        .time-count-info {
            background: #d1ecf1;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px solid #bee5eb;
        }
        .time-count-display {
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
        }
        .time-count-item {
            text-align: center;
            padding: 10px;
            background: white;
            border-radius: 5px;
            flex: 1;
            margin: 0 5px;
        }
        .time-count-value {
            font-size: 20px;
            font-weight: bold;
            color: #333;
        }
        .time-count-label {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
        }
        .status-indicator {
            display: inline-block;
            width: 12px;
            height: 12px;
            border-radius: 50%;
            margin-right: 5px;
        }
        .status-on {
            background-color: #4CAF50;
        }
        .status-off {
            background-color: #f44336;
        }
        .status-charging {
            background-color: #ffc107;
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }
        .grid-2 {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        .grid-item {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
        }
        .auto-control-info {
            background: #e8f5e9;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px solid #c8e6c9;
        }
        .power-status {
            font-size: 14px;
            padding: 8px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: center;
        }
        .power-on {
            background: #c8e6c9;
            color: #2e7d32;
        }
        .power-off {
            background: #ffcdd2;
            color: #c62828;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ESP32-C3 Light Tracker</h1>
        
        <div class="mdns-info">
            <strong>Access this device via:</strong><br>
            http://<span id="mdnsHostname">light-tracker.local</span> or http://<span id="ipAddress">192.168.4.1</span>
        </div>
        
        <div class="status" id="status"></div>
        
        <div class="section">
            <div class="section-title">Device Status</div>
            <div class="grid-2">
                <div class="grid-item">
                    <h3>Light Status</h3>
                    <div id="lightStateDisplay">Loading...</div>
                    <div id="autoControlDisplay" style="margin-top: 10px;"></div>
                    <div class="time-count-display">
                        <div class="time-count-item">
                            <div class="time-count-value" id="hoursWithLight">0</div>
                            <div class="time-count-label">Hours with Light</div>
                        </div>
                        <div class="time-count-item">
                            <div class="time-count-value" id="hoursWithoutLight">0</div>
                            <div class="time-count-label">Hours without Light</div>
                        </div>
                    </div>
                </div>
                <div class="grid-item">
                    <h3>Battery Status</h3>
                    <div id="chargingStatus" style="text-align: center; margin-bottom: 10px;"></div>
                    <div class="battery-display">
                        <div class="battery-item">
                            <div class="battery-value" id="batteryVoltageDisplay">0.00V</div>
                            <div class="battery-label">Voltage</div>
                        </div>
                        <div class="battery-item">
                            <div class="battery-value" id="batteryPercentageDisplay">0%</div>
                            <div class="battery-label">Percentage</div>
                        </div>
                    </div>
                    <div class="power-status" id="powerStatus"></div>
                    <button type="button" id="refreshBatteryBtn" class="btn-info" style="width: 100%; margin-top: 10px;">Refresh Battery Status</button>
                </div>
            </div>
        </div>
        
        <div class="section">
            <div class="section-title">WiFi Configuration</div>
            <form action="/config" method="POST">
                <div class="form-group">
                    <label for="ssid">WiFi SSID:</label>
                    <input type="text" id="ssid" name="ssid" required placeholder="Enter your WiFi SSID">
                </div>
                
                <div class="form-group">
                    <label for="password">WiFi Password:</label>
                    <div class="password-container">
                        <input type="password" id="password" name="password" placeholder="Enter your WiFi password">
                        <button type="button" class="toggle-btn" id="togglePassword">👁️</button>
                    </div>
                </div>
                
                <button type="submit" id="submitBtn" class="btn-primary">Save & Connect to WiFi</button>
            </form>
            
            <div id="currentCredentials" class="credentials-display" style="display: none;">
                <strong>Current WiFi Credentials:</strong><br>
                <div class="credential-item" id="currentSsidDisplay"></div>
                <div class="credential-item" id="currentPasswordDisplay"></div>
            </div>
        </div>
        
        <div class="section">
            <div class="section-title">Telegram Configuration</div>
            <div class="telegram-config">
                <div class="form-group">
                    <label for="botToken">Bot Token:</label>
                    <input type="text" id="botToken" name="botToken" placeholder="Enter your Telegram Bot Token">
                </div>
                <div class="form-group">
                    <label for="chatID">Chat ID:</label>
                    <input type="text" id="chatID" name="chatID" placeholder="Enter your Chat ID">
                </div>
                <button type="button" id="saveTelegramBtn" class="btn-primary">Save Telegram Settings</button>
                
                <div id="currentTelegram" class="credentials-display" style="display: none;">
                    <strong>Current Telegram Settings:</strong><br>
                    <div class="credential-item" id="currentBotTokenDisplay"></div>
                    <div class="credential-item" id="currentChatIdDisplay"></div>
                </div>
            </div>
            <div id="telegramStatus" class="telegram-status"></div>
        </div>
        
        <div class="section">
            <div class="section-title">Message Configuration</div>
            
            <div class="custom-text-row">
                <div class="custom-text-group">
                    <label for="customTextOn">Custom Text for Light ON:</label>
                    <input type="text" id="customTextOn" name="customTextOn" placeholder="Enter custom text for Light ON">
                </div>
                <div class="custom-text-group">
                    <label for="customTextOff">Custom Text for Light OFF:</label>
                    <input type="text" id="customTextOff" name="customTextOff" placeholder="Enter custom text for Light OFF">
                </div>
            </div>
            <button type="button" id="saveCustomTextBtn" class="btn-primary">Save Custom Text</button>
            
            <div class="timestamp-options">
                <label>Message Options:</label>
                <div class="checkbox-group" id="messageOptionsCheckboxes">
                    <div class="checkbox-item">
                        <input type="checkbox" id="includeDate" name="includeDate">
                        <label for="includeDate">Include Date</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="includeTime" name="includeTime">
                        <label for="includeTime">Include Time</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="includeVoltage" name="includeVoltage">
                        <label for="includeVoltage">Include Battery Voltage</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="includePercentage" name="includePercentage">
                        <label for="includePercentage">Include Battery %</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="includeTimeCount" name="includeTimeCount">
                        <label for="includeTimeCount">Include Time Count (годин)</label>
                    </div>
                </div>
                <button type="button" id="saveMessageOptionsBtn" class="btn-primary">Save All Message Options</button>
            </div>
            
            <div class="auto-control-info">
                <label>Auto Light Control:</label>
                <div class="checkbox-item">
                    <input type="checkbox" id="autoLightControl" name="autoLightControl">
                    <label for="autoLightControl">Enable auto light control based on power detection</label>
                </div>
                <div class="power-status" id="autoControlStatus">
                    Auto control disabled
                </div>
            </div>
            
            <div class="preview-box">
                <div class="preview-label">Message Preview:</div>
                <div id="messagePreview">Configure settings to see preview</div>
            </div>
        </div>
        
        <div class="section">
            <div class="section-title">Light Control</div>
            <div class="button-row">
                <button type="button" id="lightOnBtn" class="btn-light">
                    💡 Light ON
                </button>
                <button type="button" id="lightOffBtn" class="btn-dark">
                    🌙 Light OFF
                </button>
            </div>
            <div id="lightStatus"></div>
            <div id="lastAction" class="timestamp"></div>
        </div>
        
        <div class="current-info" id="currentInfo"></div>
        
        <div class="section">
            <div class="section-title">Device Management</div>
            <button type="button" id="showCredentialsBtn" class="btn-warning">Show All Credentials</button>
            <button type="button" id="resetBtn" class="reset-btn">Reset All Settings</button>
            <button type="button" id="resetTimeCountBtn" class="btn-info">Reset Time Counters</button>
        </div>
        
        <div class="info">
            <strong>Instructions:</strong><br>
            1. Access via: <strong>light-tracker.local</strong> or IP address<br>
            2. Configure WiFi and Telegram settings<br>
            3. Set custom text and message options<br>
            4. Enable auto control for automatic light based on charging<br>
            5. Monitor battery status and time counters<br>
            6. Use Light ON/OFF buttons or let auto-control work<br>
            7. WiFi AP remains active at all times
        </div>
    </div>
    
    <script>
        function handleCheckboxChange(checkboxId) {
            const checkbox = document.getElementById(checkboxId);
            const parent = checkbox.parentElement;
            parent.classList.add('checkbox-saving');
            setTimeout(() => {
                parent.classList.remove('checkbox-saving');
            }, 500);
            
            updateMessagePreview();
        }
        
        function handleAutoControlChange() {
            const checkbox = document.getElementById('autoLightControl');
            const autoControl = checkbox.checked;
            
            checkbox.disabled = true;
            
            fetch('/auto-control', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: 'autoControl=' + autoControl
            })
            .then(response => response.text())
            .then(data => {
                showStatus('Auto control setting saved!', 'success');
                checkbox.disabled = false;
                updateStatus();
            })
            .catch(error => {
                showStatus('Error saving auto control setting', 'error');
                checkbox.disabled = false;
                console.error('Error:', error);
            });
        }
        
        function updateMessagePreview() {
            const previewDiv = document.getElementById('messagePreview');
            const date = new Date();
            const dateStr = date.toISOString().split('T')[0];
            const timeStr = date.toTimeString().split(' ')[0];
            
            const customTextOn = document.getElementById('customTextOn').value || 'Light ON';
            const customTextOff = document.getElementById('customTextOff').value || 'Light OFF';
            const batteryVoltage = document.getElementById('batteryVoltageDisplay').textContent.replace('V', '') || '0.00';
            const batteryPercentage = document.getElementById('batteryPercentageDisplay').textContent.replace('%', '') || '0';
            const hoursWithLight = document.getElementById('hoursWithLight').textContent || '0';
            const hoursWithoutLight = document.getElementById('hoursWithoutLight').textContent || '0';
            
            const includeDate = document.getElementById('includeDate').checked;
            const includeTime = document.getElementById('includeTime').checked;
            const includeVoltage = document.getElementById('includeVoltage').checked;
            const includePercentage = document.getElementById('includePercentage').checked;
            const includeTimeCount = document.getElementById('includeTimeCount').checked;
            
            let onMessage = customTextOn;
            let offMessage = customTextOff;
            
            if(includeDate || includeTime) {
                let timestampParts = [];
                if(includeDate) timestampParts.push(dateStr);
                if(includeTime) timestampParts.push(timeStr);
                
                const timestamp = timestampParts.join(' ');
                onMessage += ' - ' + timestamp;
                offMessage += ' - ' + timestamp;
            }
            
            if(includeVoltage || includePercentage) {
                let batteryParts = [];
                if(includeVoltage) batteryParts.push(batteryVoltage + 'V');
                if(includePercentage) batteryParts.push(batteryPercentage + '%');
                
                const batteryInfo = batteryParts.join(' | ');
                onMessage += ' [' + batteryInfo + ']';
                offMessage += ' [' + batteryInfo + ']';
            }
            
            if(includeTimeCount) {
                onMessage += ' (годин зі світлом: ' + hoursWithLight + ')';
                offMessage += ' (годин без світла: ' + hoursWithoutLight + ')';
            }
            
            previewDiv.innerHTML = `
                <strong>Light ON:</strong> ${onMessage}<br>
                <strong>Light OFF:</strong> ${offMessage}
            `;
        }
        
        function updateStatus() {
            fetch('/status')
                .then(response => response.json())
                .then(data => {
                    const statusDiv = document.getElementById('status');
                    const infoDiv = document.getElementById('currentInfo');
                    const ipSpan = document.getElementById('ipAddress');
                    
                    statusDiv.className = 'status ' + data.status;
                    statusDiv.textContent = data.message;
                    
                    ipSpan.textContent = data.ip;
                    
                    infoDiv.innerHTML = `
                        ${data.currentSSID ? 'Connected to: ' + data.currentSSID + '<br>' : ''}
                        ${data.ip ? 'Local IP: ' + data.ip : ''}
                    `;
                    
                    if(data.currentSSID && data.currentSSID !== 'Not set') {
                        document.getElementById('ssid').value = data.currentSSID;
                    }
                    
                    if(data.botToken) {
                        document.getElementById('botToken').value = data.botToken;
                    }
                    if(data.chatID) {
                        document.getElementById('chatID').value = data.chatID;
                    }
                    
                    if(data.customTextOn) {
                        document.getElementById('customTextOn').value = data.customTextOn;
                    }
                    if(data.customTextOff) {
                        document.getElementById('customTextOff').value = data.customTextOff;
                    }
                    
                    document.getElementById('includeDate').checked = data.includeDate;
                    document.getElementById('includeTime').checked = data.includeTime;
                    document.getElementById('includeVoltage').checked = data.includeVoltage;
                    document.getElementById('includePercentage').checked = data.includePercentage;
                    document.getElementById('includeTimeCount').checked = data.includeTimeCount;
                    document.getElementById('autoLightControl').checked = data.autoLightControl;
                    
                    document.getElementById('batteryVoltageDisplay').textContent = data.batteryVoltage + 'V';
                    document.getElementById('batteryPercentageDisplay').textContent = data.batteryPercentage + '%';
                    
                    const chargingStatusDiv = document.getElementById('chargingStatus');
                    if(data.isCharging) {
                        chargingStatusDiv.innerHTML = '<span class="status-indicator status-charging"></span> Charging';
                    } else {
                        chargingStatusDiv.innerHTML = '<span class="status-indicator status-off"></span> Not Charging';
                    }
                    
                    const powerStatusDiv = document.getElementById('powerStatus');
                    if(data.powerConnected) {
                        powerStatusDiv.className = 'power-status power-on';
                        powerStatusDiv.textContent = '🔌 Power Connected';
                    } else {
                        powerStatusDiv.className = 'power-status power-off';
                        powerStatusDiv.textContent = '🔋 Battery Power';
                    }
                    
                    const lightStateDiv = document.getElementById('lightStateDisplay');
                    if(data.lightCurrentlyOn) {
                        lightStateDiv.innerHTML = '<span class="status-indicator status-on"></span> Light is currently ON';
                    } else {
                        lightStateDiv.innerHTML = '<span class="status-indicator status-off"></span> Light is currently OFF';
                    }
                    
                    const autoControlDiv = document.getElementById('autoControlStatus');
                    if(data.autoLightControl) {
                        if(data.powerConnected) {
                            autoControlDiv.className = 'power-status power-on';
                            autoControlDiv.textContent = 'Auto: Light ON (Power Connected)';
                        } else {
                            autoControlDiv.className = 'power-status power-off';
                            autoControlDiv.textContent = 'Auto: Light OFF (Battery Power)';
                        }
                    } else {
                        autoControlDiv.className = 'power-status';
                        autoControlDiv.textContent = 'Auto control disabled';
                    }
                    
                    document.getElementById('hoursWithLight').textContent = data.hoursWithLight;
                    document.getElementById('hoursWithoutLight').textContent = data.hoursWithoutLight;
                    
                    updateCredentialDisplays(data);
                    updateMessagePreview();
                })
                .catch(error => {
                    console.error('Error:', error);
                });
        }
        
        function updateCredentialDisplays(data) {
            const wifiDisplay = document.getElementById('currentCredentials');
            const ssidDisplay = document.getElementById('currentSsidDisplay');
            const passDisplay = document.getElementById('currentPasswordDisplay');
            
            if(data.currentSSID && data.currentSSID !== 'Not set' && data.currentPassword) {
                wifiDisplay.style.display = 'block';
                ssidDisplay.textContent = 'SSID: ' + data.currentSSID;
                passDisplay.textContent = 'Password: ' + data.currentPassword;
            } else {
                wifiDisplay.style.display = 'none';
            }
            
            const telegramDisplay = document.getElementById('currentTelegram');
            const tokenDisplay = document.getElementById('currentBotTokenDisplay');
            const chatDisplay = document.getElementById('currentChatIdDisplay');
            
            if(data.botToken && data.chatID) {
                telegramDisplay.style.display = 'block';
                tokenDisplay.textContent = 'Bot Token: ' + data.botToken;
                chatDisplay.textContent = 'Chat ID: ' + data.chatID;
            } else {
                telegramDisplay.style.display = 'none';
            }
        }
        
        document.getElementById('togglePassword').addEventListener('click', function() {
            const passwordField = document.getElementById('password');
            if(passwordField.type === 'password') {
                passwordField.type = 'text';
                this.textContent = '🙈';
            } else {
                passwordField.type = 'password';
                this.textContent = '👁️';
            }
        });
        
        document.getElementById('refreshBatteryBtn').addEventListener('click', function() {
            const button = this;
            button.textContent = 'Refreshing...';
            button.disabled = true;
            
            fetch('/refresh-battery')
                .then(response => response.text())
                .then(data => {
                    showStatus('Battery status refreshed!', 'success');
                    button.textContent = 'Refresh Battery Status';
                    button.disabled = false;
                    updateStatus();
                })
                .catch(error => {
                    showStatus('Error refreshing battery', 'error');
                    button.textContent = 'Refresh Battery Status';
                    button.disabled = false;
                    console.error('Error:', error);
                });
        });
        
        document.getElementById('resetTimeCountBtn').addEventListener('click', function() {
            if(confirm('Are you sure you want to reset time counters?')) {
                const button = this;
                button.textContent = 'Resetting...';
                button.disabled = true;
                
                fetch('/reset-time-count')
                    .then(response => response.text())
                    .then(data => {
                        showStatus('Time counters reset!', 'success');
                        button.textContent = 'Reset Time Counters';
                        button.disabled = false;
                        updateStatus();
                    })
                    .catch(error => {
                        showStatus('Error resetting time counters', 'error');
                        button.textContent = 'Reset Time Counters';
                        button.disabled = false;
                        console.error('Error:', error);
                    });
            }
        });
        
        document.getElementById('showCredentialsBtn').addEventListener('click', function() {
            updateStatus();
            showStatus('Credentials displayed below', 'info');
        });
        
        document.getElementById('resetBtn').addEventListener('click', function() {
            if(confirm('Are you sure you want to reset ALL settings?\nThis will erase WiFi, Telegram, and custom settings.')) {
                const button = this;
                button.textContent = 'Resetting...';
                button.disabled = true;
                
                fetch('/reset', {
                    method: 'POST'
                })
                .then(response => response.text())
                .then(data => {
                    showStatus('All settings reset successfully!', 'success');
                    setTimeout(() => {
                        location.reload();
                    }, 2000);
                })
                .catch(error => {
                    showStatus('Error resetting settings', 'error');
                    button.textContent = 'Reset All Settings';
                    button.disabled = false;
                });
            }
        });
        
        document.getElementById('saveTelegramBtn').addEventListener('click', function() {
            const botToken = document.getElementById('botToken').value;
            const chatID = document.getElementById('chatID').value;
            
            if(!botToken || !chatID) {
                showTelegramStatus('Please enter both Bot Token and Chat ID', 'error');
                return;
            }
            
            const button = this;
            button.textContent = 'Saving...';
            button.disabled = true;
            
            fetch('/telegram-config', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: 'botToken=' + encodeURIComponent(botToken) + '&chatID=' + encodeURIComponent(chatID)
            })
            .then(response => response.text())
            .then(data => {
                showTelegramStatus('Telegram settings saved successfully!', 'success');
                button.textContent = 'Save Telegram Settings';
                button.disabled = false;
                updateStatus();
            })
            .catch(error => {
                showTelegramStatus('Error saving Telegram settings', 'error');
                button.textContent = 'Save Telegram Settings';
                button.disabled = false;
                console.error('Error:', error);
            });
        });
        
        document.getElementById('saveMessageOptionsBtn').addEventListener('click', function() {
            const includeDate = document.getElementById('includeDate').checked;
            const includeTime = document.getElementById('includeTime').checked;
            const includeVoltage = document.getElementById('includeVoltage').checked;
            const includePercentage = document.getElementById('includePercentage').checked;
            const includeTimeCount = document.getElementById('includeTimeCount').checked;
            
            const button = this;
            button.textContent = 'Saving...';
            button.disabled = true;
            
            fetch('/message-options', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: 'includeDate=' + includeDate + 
                      '&includeTime=' + includeTime + 
                      '&includeVoltage=' + includeVoltage + 
                      '&includePercentage=' + includePercentage + 
                      '&includeTimeCount=' + includeTimeCount
            })
            .then(response => response.text())
            .then(data => {
                showStatus('Message options saved!', 'success');
                button.textContent = 'Save All Message Options';
                button.disabled = false;
                
                const checkboxes = document.querySelectorAll('#messageOptionsCheckboxes .checkbox-item');
                checkboxes.forEach(item => {
                    item.classList.add('checkbox-saving');
                    setTimeout(() => {
                        item.classList.remove('checkbox-saving');
                    }, 500);
                });
                
                updateStatus();
            })
            .catch(error => {
                showStatus('Error saving settings', 'error');
                button.textContent = 'Save All Message Options';
                button.disabled = false;
                console.error('Error:', error);
            });
        });
        
        document.getElementById('saveCustomTextBtn').addEventListener('click', function() {
            const customTextOn = document.getElementById('customTextOn').value;
            const customTextOff = document.getElementById('customTextOff').value;
            
            const button = this;
            button.textContent = 'Saving...';
            button.disabled = true;
            
            fetch('/custom-text', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: 'customTextOn=' + encodeURIComponent(customTextOn) + '&customTextOff=' + encodeURIComponent(customTextOff)
            })
            .then(response => response.text())
            .then(data => {
                showStatus('Custom text saved!', 'success');
                button.textContent = 'Save Custom Text';
                button.disabled = false;
                updateStatus();
            })
            .catch(error => {
                showStatus('Error saving custom text', 'error');
                button.textContent = 'Save Custom Text';
                button.disabled = false;
                console.error('Error:', error);
            });
        });
        
        document.getElementById('lightOnBtn').addEventListener('click', function() {
            sendLightCommand('on');
        });
        
        document.getElementById('lightOffBtn').addEventListener('click', function() {
            sendLightCommand('off');
        });
        
        function sendLightCommand(command) {
            const lightOnBtn = document.getElementById('lightOnBtn');
            const lightOffBtn = document.getElementById('lightOffBtn');
            const statusDiv = document.getElementById('lightStatus');
            const lastActionDiv = document.getElementById('lastAction');
            
            lightOnBtn.disabled = true;
            lightOffBtn.disabled = true;
            
            if(command === 'on') {
                lightOnBtn.textContent = 'Sending...';
                statusDiv.innerHTML = '<div class="telegram-info">Sending Light ON message...</div>';
            } else {
                lightOffBtn.textContent = 'Sending...';
                statusDiv.innerHTML = '<div class="telegram-info">Sending Light OFF message...</div>';
            }
            
            fetch('/light-control', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: 'command=' + command
            })
            .then(response => response.json())
            .then(data => {
                if(data.success) {
                    statusDiv.innerHTML = '<div class="telegram-success">' + data.message + '</div>';
                    lastActionDiv.textContent = 'Last action: ' + data.timestamp;
                } else {
                    statusDiv.innerHTML = '<div class="telegram-error">' + data.message + '</div>';
                }
                
                updateStatus();
                
                lightOnBtn.disabled = false;
                lightOffBtn.disabled = false;
                lightOnBtn.textContent = '💡 Light ON';
                lightOffBtn.textContent = '🌙 Light OFF';
            })
            .catch(error => {
                statusDiv.innerHTML = '<div class="telegram-error">Error sending command</div>';
                console.error('Error:', error);
                
                lightOnBtn.disabled = false;
                lightOffBtn.disabled = false;
                lightOnBtn.textContent = '💡 Light ON';
                lightOffBtn.textContent = '🌙 Light OFF';
            });
        }
        
        function showTelegramStatus(message, type) {
            const statusDiv = document.getElementById('telegramStatus');
            statusDiv.className = 'telegram-status telegram-' + type;
            statusDiv.textContent = message;
            statusDiv.style.display = 'block';
            
            setTimeout(() => {
                statusDiv.style.display = 'none';
            }, 5000);
        }
        
        function showStatus(message, type) {
            const statusDiv = document.getElementById('status');
            const originalClass = statusDiv.className;
            const originalText = statusDiv.textContent;
            
            statusDiv.className = 'status telegram-' + type;
            statusDiv.textContent = message;
            
            setTimeout(() => {
                statusDiv.className = originalClass;
                statusDiv.textContent = originalText;
            }, 3000);
        }
        
        document.getElementById('includeDate').addEventListener('change', () => {
            handleCheckboxChange('includeDate');
            updateMessagePreview();
        });
        document.getElementById('includeTime').addEventListener('change', () => {
            handleCheckboxChange('includeTime');
            updateMessagePreview();
        });
        document.getElementById('includeVoltage').addEventListener('change', () => {
            handleCheckboxChange('includeVoltage');
            updateMessagePreview();
        });
        document.getElementById('includePercentage').addEventListener('change', () => {
            handleCheckboxChange('includePercentage');
            updateMessagePreview();
        });
        document.getElementById('includeTimeCount').addEventListener('change', () => {
            handleCheckboxChange('includeTimeCount');
            updateMessagePreview();
        });
        document.getElementById('autoLightControl').addEventListener('change', handleAutoControlChange);
        
        document.getElementById('customTextOn').addEventListener('input', updateMessagePreview);
        document.getElementById('customTextOff').addEventListener('input', updateMessagePreview);
        
        setInterval(updateStatus, 5000);
        
        updateStatus();
        
        document.querySelector('form').addEventListener('submit', function(e) {
            const button = document.getElementById('submitBtn');
            button.textContent = 'Saving & Connecting...';
            button.disabled = true;
            
            setTimeout(() => {
                button.textContent = 'Save & Connect to WiFi';
                button.disabled = false;
            }, 5000);
        });
    </script>
</body>
</html>
)rawliteral";

// Function declarations
void handleRoot();
void handleConfig();
void handleStatus();
void handleTelegramConfig();
void handleMessageOptions();
void handleAutoControl();
void handleCustomText();
void handleLightControl();
void handleRefreshBattery();
void handleResetTimeCount();
void handleReset();
void handleNotFound();
void connectToWiFi();
void startAP();
void setupTime();
bool sendTelegramMessage(String message);
String getCurrentDateTime();
String getCurrentDate();
String getCurrentTime();
String formatMessage(String customText, bool includeDate, bool includeTime, 
                     bool includeVoltage, bool includePercentage, bool includeTimeCount, 
                     bool isOnCommand);
float readBatteryVoltage();
int calculateBatteryPercentage(float voltage);
void updateTimeCounters();
void checkPowerStatus();
void handleAutoLightControl();
bool isPowerConnected();
void triggerAutoLightOn();
void triggerAutoLightOff();

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println("\n\nESP32-C3 Light Tracker with Battery Monitoring");
  Serial.println("=============================================");
  
  // Initialize GPIO pins
  pinMode(BATTERY_ADC_PIN, INPUT);
  pinMode(POWER_DETECT_PIN, INPUT_PULLDOWN);
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  
  // Initialize preferences
  preferences.begin("light-tracker", false);
  
  // Load saved credentials
  ssid = preferences.getString("ssid", "");
  password = preferences.getString("password", "");
  telegramBotToken = preferences.getString("botToken", "");
  telegramChatID = preferences.getString("chatID", "");
  
  // Load light control settings
  customTextOn = preferences.getString("customTextOn", "Light ON");
  customTextOff = preferences.getString("customTextOff", "Light OFF");
  includeDate = preferences.getBool("includeDate", true);
  includeTime = preferences.getBool("includeTime", true);
  includeVoltage = preferences.getBool("includeVoltage", false);
  includePercentage = preferences.getBool("includePercentage", false);
  includeTimeCount = preferences.getBool("includeTimeCount", false);
  autoLightControl = preferences.getBool("autoLightControl", false);
  
  // Load time counters
  lastLightOnTime = preferences.getULong("lastLightOnTime", 0);
  lastLightOffTime = preferences.getULong("lastLightOffTime", 0);
  hoursWithLight = preferences.getULong("hoursWithLight", 0);
  hoursWithoutLight = preferences.getULong("hoursWithoutLight", 0);
  lightCurrentlyOn = preferences.getBool("lightCurrentlyOn", false);
  
  // Read initial battery status
  batteryVoltage = readBatteryVoltage();
  batteryPercentage = calculateBatteryPercentage(batteryVoltage);
  
  // Check initial power status
  isCharging = isPowerConnected();
  
  // Print loaded settings
  Serial.println("Loaded settings:");
  Serial.printf("WiFi SSID: %s\n", ssid.length() > 0 ? ssid.c_str() : "Not set");
  Serial.printf("Include Date: %s\n", includeDate ? "Yes" : "No");
  Serial.printf("Include Time: %s\n", includeTime ? "Yes" : "No");
  Serial.printf("Include Voltage: %s\n", includeVoltage ? "Yes" : "No");
  Serial.printf("Include Percentage: %s\n", includePercentage ? "Yes" : "No");
  Serial.printf("Include Time Count: %s\n", includeTimeCount ? "Yes" : "No");
  Serial.printf("Auto Light Control: %s\n", autoLightControl ? "Yes" : "No");
  Serial.printf("Light Currently On: %s\n", lightCurrentlyOn ? "Yes" : "No");
  
  Serial.println("Battery Status:");
  Serial.printf("Voltage: %.2fV\n", batteryVoltage);
  Serial.printf("Percentage: %d%%\n", batteryPercentage);
  Serial.printf("Charging: %s\n", isCharging ? "Yes" : "No");
  
  // Start Access Point immediately
  startAP();
  
  // Setup mDNS
  if (!MDNS.begin(mdnsHostname)) {
    Serial.println("Error setting up mDNS responder!");
  } else {
    Serial.println("mDNS responder started");
    Serial.printf("Access via: http://%s.local\n", mdnsHostname);
    
    // Add service to mDNS
    MDNS.addService("http", "tcp", 80);
  }
  
  // Setup web server routes
  server.on("/", HTTP_GET, handleRoot);
  server.on("/config", HTTP_POST, handleConfig);
  server.on("/telegram-config", HTTP_POST, handleTelegramConfig);
  server.on("/message-options", HTTP_POST, handleMessageOptions);
  server.on("/auto-control", HTTP_POST, handleAutoControl);
  server.on("/custom-text", HTTP_POST, handleCustomText);
  server.on("/light-control", HTTP_POST, handleLightControl);
  server.on("/refresh-battery", HTTP_GET, handleRefreshBattery);
  server.on("/reset-time-count", HTTP_POST, handleResetTimeCount);
  server.on("/status", HTTP_GET, handleStatus);
  server.on("/reset", HTTP_POST, handleReset);
  server.onNotFound(handleNotFound);
  
  server.begin();
  Serial.println("HTTP server started");
  
  // Setup time (NTP)
  setupTime();
  
  // Update time counters on startup
  updateTimeCounters();
  
  // Try to connect to WiFi if credentials exist
  if (ssid.length() > 0) {
    Serial.println("Attempting to connect to stored WiFi...");
    connectToWiFi();
  }
}

void loop() {
  server.handleClient();
  
  
  // Handle WiFi connection attempts
  if (!isConnected && ssid.length() > 0 && password.length() > 0) {
    if (millis() - lastConnectionAttempt > 10000) {
      Serial.println("Attempting to reconnect to WiFi...");
      connectToWiFi();
      lastConnectionAttempt = millis();
    }
  }
  
  // Update time counters every minute
  static unsigned long lastCounterUpdate = 0;
  if (millis() - lastCounterUpdate > 60000) {
    updateTimeCounters();
    lastCounterUpdate = millis();
  }
  
  // Check battery status every 30 seconds
  static unsigned long lastBatteryCheck = 0;
  if (millis() - lastBatteryCheck > 30000) {
    batteryVoltage = readBatteryVoltage();
    batteryPercentage = calculateBatteryPercentage(batteryVoltage);
    lastBatteryCheck = millis();
  }
  
  // Check power status with debouncing
  checkPowerStatus();
  
  // Handle auto light control if enabled
  if (autoLightControl) {
    handleAutoLightControl();
  }
  
  // Monitor WiFi connection status
  static bool wasConnected = false;
  if (WiFi.status() == WL_CONNECTED) {
    if (!wasConnected) {
      wasConnected = true;
      isConnected = true;
      Serial.print("\nConnected to WiFi! IP address: ");
      Serial.println(WiFi.localIP());
      Serial.printf("Access via: http://%s.local\n", mdnsHostname);
    }
  } else {
    if (wasConnected) {
      wasConnected = false;
      isConnected = false;
      Serial.println("\nDisconnected from WiFi");
    }
  }
}

float readBatteryVoltage() {
  int adcValue = analogRead(BATTERY_ADC_PIN);
  float voltage = (adcValue / 4095.0) * 3.3 * VOLTAGE_DIVIDER_RATIO;
  
  static float smoothedVoltage = voltage;
  smoothedVoltage = (smoothedVoltage * 0.9) + (voltage * 0.1);
  
  return smoothedVoltage;
}

int calculateBatteryPercentage(float voltage) {
  if (voltage >= BATTERY_MAX_VOLTAGE) return 100;
  if (voltage <= BATTERY_MIN_VOLTAGE) return 0;
  
  int percentage = (int)(((voltage - BATTERY_MIN_VOLTAGE) / 
                         (BATTERY_MAX_VOLTAGE - BATTERY_MIN_VOLTAGE)) * 100);
  
  if (percentage > 100) percentage = 100;
  if (percentage < 0) percentage = 0;
  
  return percentage;
}

bool isPowerConnected() {
  return digitalRead(POWER_DETECT_PIN) == HIGH;
}

void checkPowerStatus() {
  static bool lastPowerReading = false;
  bool currentPowerReading = isPowerConnected();
  
  if (currentPowerReading != lastPowerReading) {
    if (millis() - lastPowerChangeTime > DEBOUNCE_DELAY) {
      isCharging = currentPowerReading;
      lastPowerChangeTime = millis();
      lastPowerReading = currentPowerReading;
      
      Serial.printf("Power status changed: %s\n", 
                   isCharging ? "Charging (5V connected)" : "Battery power");
    }
  } else {
    lastPowerChangeTime = millis();
  }
}

void handleAutoLightControl() {
  static bool lastAutoState = false;
  static unsigned long lastAutoAction = 0;
  
  if (isCharging != lastAutoState) {
    if (millis() - lastAutoAction > 5000) {
      if (isCharging) {
        if (!lightCurrentlyOn) {
          Serial.println("Auto control: Power connected, turning light ON");
          triggerAutoLightOn();
        }
      } else {
        if (lightCurrentlyOn) {
          Serial.println("Auto control: Power disconnected, turning light OFF");
          triggerAutoLightOff();
        }
      }
      lastAutoState = isCharging;
      lastAutoAction = millis();
    }
  }
}

void triggerAutoLightOn() {
  lightCurrentlyOn = true;
  lastLightOnTime = millis();
  
  preferences.putBool("lightCurrentlyOn", true);
  preferences.putULong("lastLightOnTime", lastLightOnTime);
  
  if (telegramBotToken.length() > 0 && telegramChatID.length() > 0 && WiFi.status() == WL_CONNECTED) {
    String message = formatMessage(customTextOn + " (Auto)", includeDate, includeTime, 
                                  includeVoltage, includePercentage, includeTimeCount, true);
    sendTelegramMessage(message);
  }
}

void triggerAutoLightOff() {
  lightCurrentlyOn = false;
  lastLightOffTime = millis();
  
  preferences.putBool("lightCurrentlyOn", false);
  preferences.putULong("lastLightOffTime", lastLightOffTime);
  
  if (telegramBotToken.length() > 0 && telegramChatID.length() > 0 && WiFi.status() == WL_CONNECTED) {
    String message = formatMessage(customTextOff + " (Auto)", includeDate, includeTime, 
                                  includeVoltage, includePercentage, includeTimeCount, false);
    sendTelegramMessage(message);
  }
}

void updateTimeCounters() {
  unsigned long currentTime = millis();
  
  if (lightCurrentlyOn) {
    if (lastLightOnTime > 0) {
      unsigned long elapsed = currentTime - lastLightOnTime;
      hoursWithLight += elapsed / 3600000;
      
      preferences.putULong("hoursWithLight", hoursWithLight);
    }
    lastLightOnTime = currentTime;
    preferences.putULong("lastLightOnTime", lastLightOnTime);
  } else {
    if (lastLightOffTime > 0) {
      unsigned long elapsed = currentTime - lastLightOffTime;
      hoursWithoutLight += elapsed / 3600000;
      
      preferences.putULong("hoursWithoutLight", hoursWithoutLight);
    }
    lastLightOffTime = currentTime;
    preferences.putULong("lastLightOffTime", lastLightOffTime);
  }
}

void setupTime() {
  configTime(0, 0, "pool.ntp.org", "time.nist.gov");
  Serial.println("Waiting for time synchronization...");
  
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    Serial.println("Failed to obtain time");
    return;
  }
  
  Serial.println("Time synchronized");
  char timeString[64];
  strftime(timeString, sizeof(timeString), "%Y-%m-%d %H:%M:%S", &timeinfo);
  Serial.printf("Current time: %s\n", timeString);
}

void startAP() {
  String mac = WiFi.macAddress();
  mac.replace(":", "");
  String apSSID = "LightTracker-" + mac.substring(mac.length() - 4);
  
  WiFi.softAP(apSSID.c_str());
  
  Serial.print("Access Point Started: ");
  Serial.println(apSSID);
  Serial.print("AP IP address: ");
  Serial.println(WiFi.softAPIP());
}

void connectToWiFi() {
  if (ssid.length() == 0 || password.length() == 0) {
    Serial.println("No WiFi credentials to connect");
    return;
  }
  
  Serial.printf("Connecting to: %s\n", ssid.c_str());
  
  if (WiFi.status() == WL_CONNECTED) {
    WiFi.disconnect();
    delay(100);
  }
  
  WiFi.mode(WIFI_MODE_APSTA);
  
  WiFi.begin(ssid.c_str(), password.c_str());
  
  unsigned long startTime = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startTime < connectionTimeout) {
    delay(500);
    Serial.print(".");
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi connected successfully!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    Serial.printf("Access via: http://%s.local\n", mdnsHostname);
    isConnected = true;
    
    setupTime();
  } else {
    Serial.println("\nFailed to connect to WiFi");
    isConnected = false;
    startAP();
  }
}

String getCurrentDate() {
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    return "Date not set";
  }
  
  char dateString[32];
  strftime(dateString, sizeof(dateString), "%Y-%m-%d", &timeinfo);
  return String(dateString);
}

String getCurrentTime() {
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    return "Time not set";
  }
  
  char timeString[32];
  strftime(timeString, sizeof(timeString), "%H:%M:%S", &timeinfo);
  return String(timeString);
}

String getCurrentDateTime() {
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    return "Time not set";
  }
  
  char timeString[64];
  strftime(timeString, sizeof(timeString), "%Y-%m-%d %H:%M:%S", &timeinfo);
  return String(timeString);
}

String formatMessage(String customText, bool includeDate, bool includeTime, 
                     bool includeVoltage, bool includePercentage, bool includeTimeCount, 
                     bool isOnCommand) {
  String message = customText;
  
  if (includeDate || includeTime) {
    message += " - ";
    if (includeDate) {
      message += getCurrentDate();
      if (includeTime) message += " ";
    }
    if (includeTime) {
      message += getCurrentTime();
    }
  }
  
  if (includeVoltage || includePercentage) {
    message += " [";
    if (includeVoltage) {
      message += String(batteryVoltage, 2) + "V";
      if (includePercentage) message += " | ";
    }
    if (includePercentage) {
      message += String(batteryPercentage) + "%";
    }
    message += "]";
  }
  
  if (includeTimeCount) {
    if (isOnCommand) {
      message += " (годин зі світлом: " + String(hoursWithLight) + ")";
    } else {
      message += " (годин без світла: " + String(hoursWithoutLight) + ")";
    }
  }
  
  return message;
}

bool sendTelegramMessage(String message) {
  if (telegramBotToken.length() == 0 || telegramChatID.length() == 0) {
    Serial.println("Telegram bot token or chat ID not configured!");
    return false;
  }
  
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("Not connected to WiFi. Cannot send Telegram message.");
    return false;
  }
  
  WiFiClientSecure client;
  client.setInsecure();
  
  if (!client.connect(telegramServer, 443)) {
    Serial.println("Connection to Telegram failed");
    return false;
  }
  
  message.replace(" ", "%20");
  message.replace("\n", "%0A");
  message.replace("годин", "godyn");
  
  String url = "/bot" + telegramBotToken + "/sendMessage?chat_id=" + telegramChatID + "&text=" + message;
  
  String request = String("GET ") + url + " HTTP/1.1\r\n" +
                   "Host: " + telegramServer + "\r\n" +
                   "Connection: close\r\n\r\n";
  
  client.print(request);
  
  unsigned long timeout = millis();
  while (client.available() == 0) {
    if (millis() - timeout > 5000) {
      Serial.println(">>> Client Timeout !");
      client.stop();
      return false;
    }
  }
  
  String response = "";
  while (client.available()) {
    response += client.readStringUntil('\r');
  }
  
  client.stop();
  
  if (response.indexOf("\"ok\":true") != -1) {
    Serial.println("Telegram message sent successfully!");
    return true;
  } else {
    Serial.println("Failed to send Telegram message.");
    return false;
  }
}

void handleRoot() {
  server.send(200, "text/html", index_html);
}

void handleConfig() {
  if (server.hasArg("ssid") && server.hasArg("password")) {
    String newSSID = server.arg("ssid");
    String newPassword = server.arg("password");
    
    preferences.putString("ssid", newSSID);
    preferences.putString("password", newPassword);
    
    ssid = newSSID;
    password = newPassword;
    
    Serial.println("New WiFi credentials saved:");
    Serial.printf("SSID: %s\n", ssid.c_str());
    Serial.printf("Password: %s\n", password.c_str());
    
    connectToWiFi();
    
    String response = "<html><body style='font-family: Arial; text-align: center; margin-top: 50px;'>";
    response += "<h2>WiFi Credentials Saved!</h2>";
    response += "<p>SSID: " + ssid + "</p>";
    response += "<p>Password: " + password + "</p>";
    if (isConnected) {
      response += "<p style='color: green;'>Connected to WiFi!</p>";
      response += "<p>IP Address: " + WiFi.localIP().toString() + "</p>";
    } else {
      response += "<p style='color: orange;'>Attempting to connect...<br>AP remains active for configuration.</p>";
    }
    response += "<p><a href='/'>Go Back</a></p>";
    response += "</body></html>";
    
    server.send(200, "text/html", response);
  } else {
    server.send(400, "text/plain", "Missing SSID or Password");
  }
}

void handleTelegramConfig() {
  if (server.hasArg("botToken") && server.hasArg("chatID")) {
    telegramBotToken = server.arg("botToken");
    telegramChatID = server.arg("chatID");
    
    preferences.putString("botToken", telegramBotToken);
    preferences.putString("chatID", telegramChatID);
    
    Serial.println("Telegram credentials saved:");
    Serial.printf("Bot Token: %s\n", telegramBotToken.c_str());
    Serial.printf("Chat ID: %s\n", telegramChatID.c_str());
    
    server.send(200, "text/plain", "Telegram settings saved successfully!");
  } else {
    server.send(400, "text/plain", "Missing Bot Token or Chat ID");
  }
}

void handleMessageOptions() {
  if (server.hasArg("includeDate") && server.hasArg("includeTime") && 
      server.hasArg("includeVoltage") && server.hasArg("includePercentage") && 
      server.hasArg("includeTimeCount")) {
    
    includeDate = (server.arg("includeDate") == "true");
    includeTime = (server.arg("includeTime") == "true");
    includeVoltage = (server.arg("includeVoltage") == "true");
    includePercentage = (server.arg("includePercentage") == "true");
    includeTimeCount = (server.arg("includeTimeCount") == "true");
    
    preferences.putBool("includeDate", includeDate);
    preferences.putBool("includeTime", includeTime);
    preferences.putBool("includeVoltage", includeVoltage);
    preferences.putBool("includePercentage", includePercentage);
    preferences.putBool("includeTimeCount", includeTimeCount);
    
    Serial.println("Message options saved:");
    Serial.printf("Include Date: %s\n", includeDate ? "Yes" : "No");
    Serial.printf("Include Time: %s\n", includeTime ? "Yes" : "No");
    Serial.printf("Include Voltage: %s\n", includeVoltage ? "Yes" : "No");
    Serial.printf("Include Percentage: %s\n", includePercentage ? "Yes" : "No");
    Serial.printf("Include Time Count: %s\n", includeTimeCount ? "Yes" : "No");
    
    server.send(200, "text/plain", "Message options saved successfully!");
  } else {
    server.send(400, "text/plain", "Missing message options parameters");
  }
}

void handleAutoControl() {
  if (server.hasArg("autoControl")) {
    autoLightControl = (server.arg("autoControl") == "true");
    
    preferences.putBool("autoLightControl", autoLightControl);
    
    Serial.printf("Auto light control setting saved: %s\n", 
                 autoLightControl ? "Enabled" : "Disabled");
    
    if (autoLightControl) {
      if (isCharging && !lightCurrentlyOn) {
        triggerAutoLightOn();
      } else if (!isCharging && lightCurrentlyOn) {
        triggerAutoLightOff();
      }
    }
    
    server.send(200, "text/plain", "Auto control setting saved successfully!");
  } else {
    server.send(400, "text/plain", "Missing auto control parameter");
  }
}

void handleCustomText() {
  if (server.hasArg("customTextOn") && server.hasArg("customTextOff")) {
    customTextOn = server.arg("customTextOn");
    customTextOff = server.arg("customTextOff");
    
    preferences.putString("customTextOn", customTextOn);
    preferences.putString("customTextOff", customTextOff);
    
    Serial.println("Custom text settings saved:");
    Serial.printf("Custom Text ON: %s\n", customTextOn.c_str());
    Serial.printf("Custom Text OFF: %s\n", customTextOff.c_str());
    
    server.send(200, "text/plain", "Custom text saved successfully!");
  } else {
    server.send(400, "text/plain", "Missing custom text parameters");
  }
}

void handleLightControl() {
  if (server.hasArg("command")) {
    String command = server.arg("command");
    String message = "";
    bool success = false;
    bool isOnCommand = (command == "on");
    
    if (isOnCommand) {
      lightCurrentlyOn = true;
      lastLightOnTime = millis();
      preferences.putBool("lightCurrentlyOn", true);
      preferences.putULong("lastLightOnTime", lastLightOnTime);
    } else if (command == "off") {
      lightCurrentlyOn = false;
      lastLightOffTime = millis();
      preferences.putBool("lightCurrentlyOn", false);
      preferences.putULong("lastLightOffTime", lastLightOffTime);
    }
    
    if (isOnCommand) {
      message = formatMessage(customTextOn, includeDate, includeTime, 
                             includeVoltage, includePercentage, includeTimeCount, true);
      success = sendTelegramMessage(message);
    } else if (command == "off") {
      message = formatMessage(customTextOff, includeDate, includeTime, 
                             includeVoltage, includePercentage, includeTimeCount, false);
      success = sendTelegramMessage(message);
    } else {
      message = "Invalid command";
      success = false;
    }
    
    Serial.print("Light control command: ");
    Serial.print(command);
    Serial.print(" - Message: ");
    Serial.print(message);
    Serial.print(" - Success: ");
    Serial.println(success ? "Yes" : "No");
    
    String json = "{";
    json += "\"success\":";
    json += success ? "true" : "false";
    json += ",\"message\":\"";
    json += success ? "Telegram message sent!" : "Failed to send message";
    json += "\",\"timestamp\":\"";
    json += getCurrentDateTime();
    json += "\"}";
    
    server.send(200, "application/json", json);
  } else {
    String json = "{\"success\":false,\"message\":\"No command specified\"}";
    server.send(400, "application/json", json);
  }
}

void handleRefreshBattery() {
  batteryVoltage = readBatteryVoltage();
  batteryPercentage = calculateBatteryPercentage(batteryVoltage);
  
  Serial.println("Battery status refreshed:");
  Serial.printf("Voltage: %.2fV\n", batteryVoltage);
  Serial.printf("Percentage: %d%%\n", batteryPercentage);
  Serial.printf("Charging: %s\n", isCharging ? "Yes" : "No");
  
  server.send(200, "text/plain", "Battery status refreshed!");
}

void handleResetTimeCount() {
  hoursWithLight = 0;
  hoursWithoutLight = 0;
  lastLightOnTime = 0;
  lastLightOffTime = 0;
  
  preferences.putULong("hoursWithLight", 0);
  preferences.putULong("hoursWithoutLight", 0);
  preferences.putULong("lastLightOnTime", 0);
  preferences.putULong("lastLightOffTime", 0);
  
  Serial.println("Time counters reset");
  
  server.send(200, "text/plain", "Time counters have been reset.");
}

void handleStatus() {
  String status;
  String message;
  
  if (WiFi.status() == WL_CONNECTED) {
    status = "connected";
    message = "Connected to WiFi";
  } else if (ssid.length() > 0) {
    status = "disconnected";
    message = "Not connected. AP mode active.";
  } else {
    status = "ap";
    message = "AP Mode - No WiFi configured";
  }
  
  String json = "{";
  json += "\"status\":\"" + status + "\",";
  json += "\"message\":\"" + message + "\",";
  json += "\"currentSSID\":\"" + (ssid.length() > 0 ? ssid : "Not set") + "\",";
  json += "\"currentPassword\":\"" + (password.length() > 0 ? password : "") + "\",";
  json += "\"botToken\":\"" + (telegramBotToken.length() > 0 ? telegramBotToken : "") + "\",";
  json += "\"chatID\":\"" + (telegramChatID.length() > 0 ? telegramChatID : "") + "\",";
  json += "\"customTextOn\":\"" + (customTextOn.length() > 0 ? customTextOn : "") + "\",";
  json += "\"customTextOff\":\"" + (customTextOff.length() > 0 ? customTextOff : "") + "\",";
  json += "\"includeDate\":" + String(includeDate ? "true" : "false") + ",";
  json += "\"includeTime\":" + String(includeTime ? "true" : "false") + ",";
  json += "\"includeVoltage\":" + String(includeVoltage ? "true" : "false") + ",";
  json += "\"includePercentage\":" + String(includePercentage ? "true" : "false") + ",";
  json += "\"includeTimeCount\":" + String(includeTimeCount ? "true" : "false") + ",";
  json += "\"autoLightControl\":" + String(autoLightControl ? "true" : "false") + ",";
  json += "\"batteryVoltage\":" + String(batteryVoltage, 2) + ",";
  json += "\"batteryPercentage\":" + String(batteryPercentage) + ",";
  json += "\"isCharging\":" + String(isCharging ? "true" : "false") + ",";
  json += "\"powerConnected\":" + String(isCharging ? "true" : "false") + ",";
  json += "\"lightCurrentlyOn\":" + String(lightCurrentlyOn ? "true" : "false") + ",";
  json += "\"hoursWithLight\":" + String(hoursWithLight) + ",";
  json += "\"hoursWithoutLight\":" + String(hoursWithoutLight) + ",";
  json += "\"ip\":\"" + (WiFi.status() == WL_CONNECTED ? WiFi.localIP().toString() : WiFi.softAPIP().toString()) + "\"";
  json += "}";
  
  server.send(200, "application/json", json);
}

void handleReset() {
  preferences.clear();
  
  ssid = "";
  password = "";
  telegramBotToken = "";
  telegramChatID = "";
  customTextOn = "Light ON";
  customTextOff = "Light OFF";
  includeDate = true;
  includeTime = true;
  includeVoltage = false;
  includePercentage = false;
  includeTimeCount = false;
  autoLightControl = false;
  batteryVoltage = 0.0;
  batteryPercentage = 0;
  isCharging = false;
  lastLightOnTime = 0;
  lastLightOffTime = 0;
  hoursWithLight = 0;
  hoursWithoutLight = 0;
  lightCurrentlyOn = false;
  isConnected = false;
  
  Serial.println("All settings reset to defaults");
  
  WiFi.disconnect();
  delay(100);
  startAP();
  
  server.send(200, "text/plain", "All settings have been reset. Device will restart configuration.");
}

void handleNotFound() {
  String message = "Page Not Found\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET) ? "GET" : "POST";
  message += "\nTry accessing: http://";
  message += mdnsHostname;
  message += ".local\n";
  
  server.send(404, "text/plain", message);
}
