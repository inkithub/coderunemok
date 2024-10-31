<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CodeRunner</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        /* Base Styling */
        body {
            margin: 0;
            padding: 0;
            font-family: "Arial", sans-serif;
            background-color: #1c1c1c;
            color: #e0e0e0;
        }

        /* Dark and Light Theme */
        .light-theme {
            background-color: #f5f5f5;
            color: #333;
        }
        .light-theme .codeContainer,
        .light-theme iframe {
            background-color: #fff;
            color: #333;
        }

        /* Intro Screen */
        #intro {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #009688;
            display: flex;
            justify-content: center;
            align-items: center;
            opacity: 0;
            animation: fadeIn 2s forwards;
            z-index: 10;
        }
        #intro h1 { color: white; font-size: 48px; }

        /* Floating Action Buttons */
        .fab {
            position: fixed;
            bottom: 20px;
            background-color: #009688;
            color: white;
            border: none;
            border-radius: 50%;
            width: 56px;
            height: 56px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
            cursor: pointer;
            transition: transform 0.2s;
        }
        .fab:hover { transform: scale(1.1); }
        .fab:active { transform: scale(0.9); }
        #settingsButton { right: 90px; }
        #downloadButton { right: 20px; }

        /* Modal Styling */
        #settingsModal {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: #333;
            color: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
            z-index: 100;
        }
        .settings-option { margin-bottom: 15px; }
        .close-button {
            margin-top: 10px;
            padding: 5px 10px;
            background-color: #009688;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
            transition: background-color 0.3s;
        }
        .close-button:hover { background-color: #00796b; }

        /* Container and Textarea Styling */
        #wrapper { padding: 20px; display: grid; gap: 20px; }
        .codeContainer {
            padding: 20px;
            background: #2c2c2c;
            color: #f5f5f5;
            border-radius: 10px;
        }
        .codeLabel { font-weight: bold; margin-bottom: 10px; }

        /* Animation */
        @keyframes fadeIn { 0% { opacity: 0; } 100% { opacity: 1; } }

        /* Loading Animation */
        #loading {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            display: none;
            z-index: 20;
        }
        .loader {
            width: 50px;
            height: 50px;
            background-color: #009688;
            border-radius: 50%;
            animation: pulse 1s infinite;
        }
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }

        /* Line Counter Styles */
        .lineCounter {
            position: absolute;
            bottom: 10px;
            right: 20px;
            color: #009688;
            font-size: 14px;
        }

        /* Error Message Styles */
        #errorMessage {
            display: none;
            color: red;
            font-weight: bold;
            position: fixed;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 30;
        }
    </style>
</head>
<body>

<!-- Intro Screen -->
<div id="intro"><h1>Welcome to CodeRunner!</h1></div>

<div id="wrapper" style="display: none;">
    <div class="codeContainer" id="HTMLContainer">
        <div class="codeLabel">HTML</div>
        <textarea id="htmlCode" style="width: 100%; height: 200px;">Write your HTML code here...</textarea>
        <div class="lineCounter" id="htmlLineCounter">Lines: 1</div>
    </div>
    
    <div class="codeContainer" id="CSSContainer">
        <div class="codeLabel">CSS</div>
        <textarea id="cssCode" style="width: 100%; height: 200px;">Write your CSS code here...</textarea>
        <div class="lineCounter" id="cssLineCounter">Lines: 1</div>
    </div>
    
    <div class="codeContainer" id="JSContainer">
        <div class="codeLabel">JavaScript</div>
        <textarea id="jsCode" style="width: 100%; height: 200px;">JS functionality to be added later...</textarea>
        <div class="lineCounter" id="jsLineCounter">Lines: 1</div>
    </div>

    <div class="codeContainer" id="ResultContainer">
        <div class="codeLabel">Result</div>
        <iframe id="resultFrame" style="width: 100%; height: 200px;"></iframe>
    </div>
</div>

<!-- Loading Animation -->
<div id="loading"><div class="loader"></div></div>

<!-- Error Message -->
<div id="errorMessage"></div>

<!-- Settings Modal -->
<div id="settingsModal">
    <h3>Settings</h3>
    <div class="settings-option">
        <label><input type="checkbox" id="toggleTheme"> Light Theme</label>
    </div>
    <div class="settings-option">
        <label for="fontSize">Font Size:</label>
        <select id="fontSize">
            <option value="12px">12px</option>
            <option value="14px" selected>14px</option>
            <option value="16px">16px</option>
            <option value="18px">18px</option>
        </select>
    </div>
    <button class="close-button" id="closeSettingsButton">Close</button>
</div>

<!-- Floating Action Buttons -->
<button class="fab" id="downloadButton">Save</button>
<button class="fab" id="settingsButton">⚙️</button>

<script>
    $(document).ready(function() {
        // Intro animation
        setTimeout(function() {
            $("#intro").fadeOut(1000, function() {
                $("#wrapper").fadeIn(1000);
            });
        }, 2000);

        // Floating Action Buttons Functionality
        $("#downloadButton").click(downloadCode);
        $("#settingsButton").click(openSettings);
        $("#closeSettingsButton").click(closeSettings); // Attach close function to button

        // Initialize line counters
        updateLineCount("htmlCode", "htmlLineCounter");
        updateLineCount("cssCode", "cssLineCounter");
        updateLineCount("jsCode", "jsLineCounter");

        // Attach keyup event for line counting and auto-running code
        $("#htmlCode, #cssCode, #jsCode").on('input', function() {
            updateLineCount(this.id, this.id + "Counter");
            runCode();  // Auto-run code on input
        });

        // Run Code
        function runCode() {
            $("#loading").show();
            $("#errorMessage").hide();
            try {
                var htmlContent = $("#htmlCode").val();
                var cssContent = $("#cssCode").val();
                var jsContent = $("#jsCode").val();
                var fullContent = `<style>${cssContent}</style>${htmlContent}<script>${jsContent}<\/script>`;
                $("#resultFrame").contents().find("html").html(fullContent);
            } catch (error) {
                $("#errorMessage").text("Error in running code: " + error.message).show();
            } finally {
                $("#loading").hide();
            }
        }

        // Download Code
        function downloadCode() {
            var htmlContent = $("#htmlCode").val();
            var cssContent = $("#cssCode").val();
            var jsContent = $("#jsCode").val();
            var blob = new Blob([htmlContent + "\n" + cssContent + "\n" + jsContent + "\n<!-- Watermark: Created with CodeRunner -->"], { type: "text/plain" });
            var url = URL.createObjectURL(blob);
            var a = document.createElement("a");
            a.href = url;
            a.download = "code.txt";
            a.click();
            URL.revokeObjectURL(url);
        }

        // Open and Close Settings Modal
        function openSettings() { $("#settingsModal").fadeIn(); }
        function closeSettings() { $("#settingsModal").fadeOut(); }

        // Toggle Theme
        $("#toggleTheme").change(toggleTheme);
        function toggleTheme() {
            if ($("#toggleTheme").is(":checked")) {
                $("body").addClass("light-theme");
            } else {
                $("body").removeClass("light-theme");
            }
        }

        // Change Font Size
        $("#fontSize").change(changeFontSize);
        function changeFontSize() {
            var fontSize = $("#fontSize").val();
            $("textarea").css("font-size", fontSize);
        }

        // Update Line Count
        function updateLineCount(textareaId, counterId) {
            var lineCount = $("#" + textareaId).val().split('\n').length;
            $("#" + counterId).text("Lines: " + lineCount);
        }
    });
</script>
</body>
</html>
