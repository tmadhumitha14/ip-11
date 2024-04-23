# ip-11
<?php
// Define MySQL database connection parameters
$servername = "localhost";
$username = "root";
$password = "YES";
$database = "ip";

// Create connection
$conn = new mysqli($servername, $username, $password, $database);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Define variables and initialize with empty values
$name = $email = $cv = "";
$nameErr = $emailErr = $cvErr = "";
$successMessage = "";

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Validate name
    if (empty($_POST["name"])) {
        $nameErr = "Name is required";
    } else {
        $name = test_input($_POST["name"]);
    }

    // Validate email
    if (empty($_POST["email"])) {
        $emailErr = "Email is required";
    } else {
        $email = test_input($_POST["email"]);
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            $emailErr = "Invalid email format";
        }
    }

    // Validate CV
    if (empty($_FILES["cv"]["name"])) {
        $cvErr = "CV is required";
    } else {
        $cv = $_FILES["cv"]["name"];
        $cv_temp = $_FILES["cv"]["tmp_name"];
        $cv_dest = "uploads/" . $cv;
        move_uploaded_file($cv_temp, $cv_dest);
    }

    // If no errors, insert into database
    if (empty($nameErr) && empty($emailErr) && empty($cvErr)) {
        $sql = "INSERT INTO job_applications (name, email, cv) VALUES ('$name', '$email', '$cv')";
        if ($conn->query($sql) === TRUE) {
            $successMessage = "Application submitted successfully!";
        } else {
            $successMessage = "Error: " . $sql . "<br>" . $conn->error;
        }
    }
}

function test_input($data)
{
    $data = trim($data);
    $data = stripslashes($data);
    $data = htmlspecialchars($data);
    return $data;
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Job Application Form</title>
</head>
<body>

<h2>Job Application Form</h2>
<p><span class="error">* required field</span></p>
<form method="post" enctype="multipart/form-data" action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]); ?>">
    Name: <input type="text" name="name" value="<?php echo $name; ?>">
    <span class="error">* <?php echo $nameErr; ?></span>
    <br><br>
    Email: <input type="text" name="email" value="<?php echo $email; ?>">
    <span class="error">* <?php echo $emailErr; ?></span>
    <br><br>
    CV: <input type="file" name="cv">
    <span class="error">* <?php echo $cvErr; ?></span>
    <br><br>
    <input type="submit" name="submit" value="Submit">
</form>

<?php echo $successMessage; ?>

</body>
</html>

<?php
// Close connection
$conn->close();
?>
