<?php
class DbConnect {

    private $conn;

    function __construct() {
    }

    function connect() {
        $this->conn = new mysqli("localhost", "root", "Luckyaun2106", "studentdb");
        if (mysqli_connect_errno()) {
            echo "Failed to connect to MySQL: " . mysqli_connect_error();
        }
        return $this->conn;
    }

}


class DbHandler {

    private $conn;

    function __construct() {
        $db = new DbConnect();
        $this->conn = $db->connect();
    }

    public function createStudent($student_id, $name, $major) {
        $stmt = $this->conn->prepare("INSERT INTO students(id, name, major) VALUES(?,?,?)");
        $stmt->bind_param("sss", $student_id, $name, $major);
        $result = $stmt->execute();
        $stmt->close();

        if ($result) {
            return $student_id;
          } else {
            return NULL;
        }
    }
public function createLab($lid, $lname, $lscore) {
        $stmt = $this->conn->prepare("INSERT INTO labs(lid, lname, lscore) VALUES(?,?,?)");
        $stmt->bind_param("sss", $lid, $lname, $lscore);
        $result = $stmt->execute();
        $stmt->close();

        if ($result) {
            return $lid;
          } else {
            return NULL;
        }
    }


    public function getStudent($student_id) {
        $stmt = $this->conn->prepare("SELECT * FROM students WHERE id = ?");
        $stmt->bind_param("s", $student_id);
        if ($stmt->execute()) {
            $res = array();
            $student = $stmt->get_result();
            $stmt->close();
            return $student;
        } else {
            return NULL;
        }
    }
    public function getLab($lid) {
        $stmt = $this->conn->prepare("SELECT * FROM labs WHERE lid = ?");
        $stmt->bind_param("s", $lid);
        if ($stmt->execute()) {
            $res = array();
            $lab = $stmt->get_result();
            $stmt->close();
            return $lab;
        } else {
            return NULL;
        }
    }
 public function getAllStudent() {
        $stmt = $this->conn->prepare("SELECT * FROM students");
        $stmt->execute();
        $students = $stmt->get_result();
        $stmt->close();
        return $students;
    }
public function getAllLab() {
        $stmt = $this->conn->prepare("SELECT * FROM labs");
        $stmt->execute();
        $labs = $stmt->get_result();
        $stmt->close();
        return $labs;
    }
public function countStudent() {
        $stmt = $this->conn->prepare("SELECT COUNT(id)  FROM students");
        $stmt->execute();
        $students = $stmt->get_result();
        $stmt->close();
        return $students;
    }
 public function countLabs() {
        $stmt = $this->conn->prepare("SELECT COUNT(lid) FROM labs");
        $stmt->execute();
        $labs = $stmt->get_result();
        $stmt->close();
        return $labs;
}

    public function updateStudent($student_id, $name, $major) {
        $stmt = $this->conn->prepare("UPDATE students set name = ?, major = ? WHERE id = ?");
        $stmt->bind_param("sss", $name, $major, $student_id);
        $stmt->execute();
        $num_affected_rows = $stmt->affected_rows;
        $stmt->close();
        return $num_affected_rows > 0;
    }
public function updateLab($lid, $lname, $lscore) {
        $stmt = $this->conn->prepare("UPDATE labs set lname = ?, lscore = ? WHERE lid = ?");
        $stmt->bind_param("sss", $lname, $lscore, $lid);
        $stmt->execute();
        $num_affected_rows = $stmt->affected_rows;
        $stmt->close();
        return $num_affected_rows > 0;
    }


    public function deleteStudent($student_id) {
        $stmt = $this->conn->prepare("DELETE FROM students WHERE id = ?");
        $stmt->bind_param("s", $student_id);
        $stmt->execute();
        $num_affected_rows = $stmt->affected_rows;
        $stmt->close();
        return $num_affected_rows > 0;
    }
public function deleteLab($lid) {
        $stmt = $this->conn->prepare("DELETE FROM labs WHERE lid = ?");
        $stmt->bind_param("s", $lid);
        $stmt->execute();
        $num_affected_rows = $stmt->affected_rows;
        $stmt->close();
        return $num_affected_rows > 0;
    }

}
require 'vendor/autoload.php';
$app = new \Slim\Slim();

$app->get('/students/count', function() {
            $response = array();
            $db = new DbHandler();

            $result = $db->countStudent();



            echoRespnse(200, $result->fetch_assoc());
        });
$app->get('/labs/count', function() {
            $response = array();
            $db = new DbHandler();

            $result = $db->countLabs();



            echoRespnse(200, $result->fetch_assoc());
        });

$app->get('/students', function() {
            $response = array();
            $db = new DbHandler();

            $result = $db->getAllStudent();

            $response["error"] = false;
            $response["students"] = array();

            while ($student = $result->fetch_assoc()) {
                $tmp = array();
                $tmp["id"] = $student["id"];
                $tmp["name"] = $student["name"];
                $tmp["major"] = $student["major"];
                $tmp["createdAt"] = $student["created_at"];
                array_push($response["students"], $tmp);
            }
 echoRespnse(200, $response);
        });
$app->get('/labs', function() {
            $response = array();
            $db = new DbHandler();

            $result = $db->getAllLab();

            $response["error"] = false;
            $response["labs"] = array();

            while ($lab = $result->fetch_assoc()) {
                $tmp = array();
                $tmp["lid"] = $lab["lid"];
                $tmp["lname"] = $lab["lname"];
                $tmp["lscore"] = $lab["lscore"];
                $tmp["createdAt"] = $lab["created_at"];
                array_push($response["labs"], $tmp);
            }

            echoRespnse(200, $response);
        });


$app->get('/students/:id', function($student_id) {
            $response = array();
            $db = new DbHandler();

            $result = $db->getStudent($student_id);
            $response["students"] = array();

            if ($result != NULL) {
                $response["error"] = false;

            while ($student = $result->fetch_assoc()) {
                $tmp = array();
                $tmp["id"] = $student["id"];
                $tmp["name"] = $student["name"];
                $tmp["major"] = $student["major"];
                $tmp["createdAt"] = $student["created_at"];
                array_push($response["students"], $tmp);
            }
 echoRespnse(200, $response);
            } else {
                $response["error"] = true;
                $response["message"] = "The requested resource doesn't exists";
                echoRespnse(404, $response);
            }
        });
$app->get('/labs/:id', function($lid) {
            $response = array();
            $db = new DbHandler();

            $result = $db->getLab($lid);
            $response["labs"] = array();

            if ($result != NULL) {
                $response["error"] = false;

            while ($lab = $result->fetch_assoc()) {
                $tmp = array();
                $tmp["lid"] = $lab["lid"];
                $tmp["lname"] = $lab["lname"];
                $tmp["lscore"] = $lab["lscore"];
                $tmp["createdAt"] = $lab["created_at"];
                array_push($response["labs"], $tmp);
            }
                echoRespnse(200, $response);
            } else {
                $response["error"] = true;
                $response["message"] = "The requested resource doesn't exists";
                echoRespnse(404, $response);
            }
        });

$app->post('/students', function() use ($app) {

            verifyRequiredParams(array('id','name','major'));

            $response = array();
            $id = $app->request->post('id');
            $name = $app->request->post('name');
            $major = $app->request->post('major');

  $db = new DbHandler();

            $student_id = $db->createStudent($id, $name, $major);

            if ($student_id != NULL) {
                $response["error"] = false;
                $response["message"] = "Student created successfully";
                $response["student_id"] = $student_id;
                echoRespnse(201, $response);
            } else {
                $response["error"] = true;
                $response["message"] = "Failed to create student. Please try again";
                echoRespnse(200, $response);
            }
        });
$app->post('/labs', function() use ($app) {

            verifyRequiredParams(array('lid','lname','lscore'));

            $response = array();
            $lid = $app->request->post('lid');
            $lname = $app->request->post('lname');
            $lscore = $app->request->post('lscore');

            $db = new DbHandler();

            $lid = $db->createLab($lid, $lname, $lscore);

            if ($lid != NULL) {
                $response["error"] = false;
                $response["message"] = "Lab created successfully";
                $response["lid"] = $lid;
                echoRespnse(201, $response);
            } else {
                $response["error"] = true;
                $response["message"] = "Failed to create student. Please try again";
                echoRespnse(200, $response);
            }
        });
$app->put('/students/:id', function($student_id) use($app) {
            verifyRequiredParams(array('name','major'));

            $name = $app->request->post('name');
            $major = $app->request->post('major');

            $db = new DbHandler();
            $response = array();

            $result = $db->updateStudent($student_id, $name, $major);
            if ($result) {
                $response["error"] = false;
                $response["message"] = "Sutdent updated successfully";
            } else {
                $response["error"] = true;
                $response["message"] = "Sutdent failed to update. Please try again!";
            }
            echoRespnse(200, $response);
        });
$app->put('/labs/:id', function($lid) use($app) {
            verifyRequiredParams(array('lname','lscore'));

            $lname = $app->request->post('lname');
            $lscore = $app->request->post('lscore');

            $db = new DbHandler();
            $response = array();

            $result = $db->updateLab($lid, $lname, $lscore);
            if ($result) {
                $response["error"] = false;
                $response["message"] = "Lab updated successfully";
            } else {
                $response["error"] = true;
                $response["message"] = "Lab failed to update. Please try again!";
            }
            echoRespnse(200, $response);
        });

$app->delete('/students/:id', function($student_id) use($app) {

            $db = new DbHandler();
            $response = array();
            $result = $db->deleteStudent($student_id);
            if ($result) {
                $response["error"] = false;
                $response["message"] = "Student deleted succesfully";
            } else {
                $response["error"] = true;
                $response["message"] = "Student failed to delete. Please try again!";
            }
            echoRespnse(200, $response);
        });
$app->delete('/labs/:id', function($lid) use($app) {

            $db = new DbHandler();
            $response = array();
            $result = $db->deleteLab($lid);
            if ($result) {
                $response["error"] = false;
                $response["message"] = "Lab deleted succesfully";
            } else {
                $response["error"] = true;
                $response["message"] = "Lab failed to delete. Please try again!";
            }
            echoRespnse(200, $response);
        });
function verifyRequiredParams($required_fields) {
    $error = false;
    $error_fields = "";
    $request_params = array();
    $request_params = $_REQUEST;

    if ($_SERVER['REQUEST_METHOD'] == 'PUT') {
        $app = \Slim\Slim::getInstance();
        parse_str($app->request()->getBody(), $request_params);
    }
    foreach ($required_fields as $field) {
        if (!isset($request_params[$field]) || strlen(trim($request_params[$field])) <= 0) {
            $error = true;
            $error_fields .= $field . ', ';
        }
    }

    if ($error) {

        $response = array();
        $app = \Slim\Slim::getInstance();
        $response["error"] = true;
        $response["message"] = 'Required field(s) ' . substr($error_fields, 0, -2) . ' is missing or empty';
        echoRespnse(400, $response);
        $app->stop();
    }
}


function echoRespnse($status_code, $response) {
    $app = \Slim\Slim::getInstance();

    $app->status($status_code);

    $app->contentType('application/json');

    echo json_encode($response);
}

$app->run();
?>
