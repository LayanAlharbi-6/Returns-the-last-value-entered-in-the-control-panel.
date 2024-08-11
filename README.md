# Returns-the-last-value-entered-in-the-control-panel.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تحكم بالروبوت</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            flex-direction: column;
        }
        #controls {
            display: grid;
            grid-template-columns: 100px 100px 100px;
            grid-template-rows: 100px 100px 100px;
            gap: 10px;
            justify-items: center;
            align-items: center;
            grid-template-areas:
                "left forward right"
                "left stop right"
                "left backward right";
        }
        .control-button {
            width: 90px;
            height: 90px;
            font-size: 16px;
            border: none;
            color: white;
            background-color: #007BFF;
            cursor: pointer;
            border-radius: 5px;
            text-align: center;
        }
        .control-button.forward {
            grid-area: forward;
            background-color: #28a745; /* Green */
        }
        .control-button.backward {
            grid-area: backward;
            background-color: #dc3545; /* Red */
        }
        .control-button.left {
            grid-area: left;
            background-color: #ffc107; /* Yellow */
        }
        .control-button.right {
            grid-area: right;
            background-color: #17a2b8; /* Teal */
        }
        .control-button.stop {
            grid-area: stop;
            background-color: #6c757d; /* Gray */
        }
    </style>
</head>
<body>
    <h1>تحكم بالروبوت</h1>
    <div id="controls">
        <button class="control-button left" onclick="move('left')">Left</button>
        <button class="control-button forward" onclick="move('forward')">Forward</button>
        <button class="control-button right" onclick="move('right')">Right</button>
        <button class="control-button stop" onclick="move('stop')">Stop</button>
        <button class="control-button backward" onclick="move('backward')">Backward</button>
    </div>
    <script>
        function move(direction) {
            fetch('http://localhost/server/index.php', { // تأكد من مسار الملف الصحيح
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ direction: direction })
            })
            .then(response => response.json())
            .then(data => {
                console.log('Success:', data);
                alert("Last Direction: " + data.last_direction); // عرض آخر قيمة  
            })
            .catch((error) => {
                console.error('Error:', error);
            });
        }
    </script>
    
</body>
</html>

________________________________________________
## كود php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: POST, GET, OPTIONS");
header("Access-Control-Allow-Headers: Content-Type");

// إعداد الاتصال بقاعدة البيانات 
$servername = "localhost";
$username = "LA";
$password = "LA12345678la";
$dbname = "robot1";

// إنشاء الاتصال
$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$data = json_decode(file_get_contents('php://input'), true);
$direction = isset($data['direction']) ? $data['direction'] : '';

if ($direction) {
    $stmt = $conn->prepare('INSERT INTO `directions_robots` (`direction`) VALUES (?)');
    $stmt->bind_param('s', $direction);
    $stmt->execute();
    $stmt->close();
}

//  آخر قيمة تم إدخالها في قاعدة البيانات
$sql = "SELECT `direction` FROM `directions_robots` ORDER BY `id` DESC LIMIT 1";
$result = $conn->query($sql);

$last_direction = '';
if ($result->num_rows > 0) {
    $row = $result->fetch_assoc();
    $last_direction = $row['direction'];
}

// إغلاق الاتصال
$conn->close();

header('Content-Type: application/json');
echo json_encode(['status' => 'success', 'last_direction' => $last_direction]);
?>
