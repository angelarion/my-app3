# my-app3

Crear un formulario con los siguientes datos  Nombre, Apellido, Teléfono Dirección y en la misma pagina  tener los botones de agregar editar y eliminar  también agregar un datagrid para  ver los que se va agregando y eliminando, todo esto en docker 
ChatGPT
Para cumplir con tu requerimiento, vamos a configurar un proyecto que incluye un formulario HTML para gestionar datos de usuarios, PHP para manejar las operaciones CRUD y MySQL para almacenar los datos. Además, utilizaremos Docker para contenerizar la aplicación. A continuación, te guiaré paso a paso para crear este proyecto.

Estructura del Proyecto
Formulario HTML y JS: Para capturar y mostrar los datos.
PHP: Para manejar las operaciones CRUD.
MySQL: Para la base de datos.
Docker: Para contenerizar la aplicación.
Paso 1: Crear el Formulario HTML y JS
Crea un archivo index.html:

html
Copiar código
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestión de Usuarios</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            padding: 20px;
            background-color: #f7f7f7;
        }
        form {
            max-width: 600px;
            margin: auto;
            padding: 20px;
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        label {
            display: block;
            margin-bottom: 8px;
        }
        input, button {
            width: 100%;
            padding: 10px;
            margin-bottom: 20px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            padding: 15px 20px;
        }
        button:hover {
            background-color: #45a049;
        }
        table {
            width: 100%;
            margin-top: 20px;
            border-collapse: collapse;
        }
        table, th, td {
            border: 1px solid #ddd;
            padding: 8px;
        }
        th {
            background-color: #f2f2f2;
        }
    </style>
</head>
<body>
    <h1>Gestión de Usuarios</h1>
    <form id="userForm">
        <input type="hidden" id="userId" name="userId">
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" name="nombre" required>
        
        <label for="apellido">Apellido:</label>
        <input type="text" id="apellido" name="apellido" required>
        
        <label for="telefono">Teléfono:</label>
        <input type="tel" id="telefono" name="telefono" required>
        
        <label for="direccion">Dirección:</label>
        <input type="text" id="direccion" name="direccion" required>
        
        <button type="button" onclick="addUser()">Agregar</button>
        <button type="button" onclick="editUser()">Editar</button>
        <button type="button" onclick="deleteUser()">Eliminar</button>
    </form>
    
    <h2>Lista de Usuarios</h2>
    <table id="userTable">
        <thead>
            <tr>
                <th>ID</th>
                <th>Nombre</th>
                <th>Apellido</th>
                <th>Teléfono</th>
                <th>Dirección</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody>
            <!-- Datos serán insertados aquí -->
        </tbody>
    </table>

    <script>
        document.addEventListener("DOMContentLoaded", function() {
            loadUsers();
        });

        function loadUsers() {
            fetch('actions.php?action=read')
                .then(response => response.json())
                .then(data => {
                    const tbody = document.querySelector('#userTable tbody');
                    tbody.innerHTML = '';
                    data.forEach(user => {
                        const row = document.createElement('tr');
                        row.innerHTML = `
                            <td>${user.id}</td>
                            <td>${user.nombre}</td>
                            <td>${user.apellido}</td>
                            <td>${user.telefono}</td>
                            <td>${user.direccion}</td>
                            <td>
                                <button onclick="fillForm(${user.id}, '${user.nombre}', '${user.apellido}', '${user.telefono}', '${user.direccion}')">Seleccionar</button>
                            </td>
                        `;
                        tbody.appendChild(row);
                    });
                });
        }

        function fillForm(id, nombre, apellido, telefono, direccion) {
            document.getElementById('userId').value = id;
            document.getElementById('nombre').value = nombre;
            document.getElementById('apellido').value = apellido;
            document.getElementById('telefono').value = telefono;
            document.getElementById('direccion').value = direccion;
        }

        function addUser() {
            const formData = new FormData(document.getElementById('userForm'));
            formData.append('action', 'create');
            fetch('actions.php', {
                method: 'POST',
                body: formData
            }).then(response => response.text())
            .then(data => {
                alert(data);
                loadUsers();
                document.getElementById('userForm').reset();
            });
        }

        function editUser() {
            const formData = new FormData(document.getElementById('userForm'));
            formData.append('action', 'update');
            fetch('actions.php', {
                method: 'POST',
                body: formData
            }).then(response => response.text())
            .then(data => {
                alert(data);
                loadUsers();
                document.getElementById('userForm').reset();
            });
        }

        function deleteUser() {
            const formData = new FormData(document.getElementById('userForm'));
            formData.append('action', 'delete');
            fetch('actions.php', {
                method: 'POST',
                body: formData
            }).then(response => response.text())
            .then(data => {
                alert(data);
                loadUsers();
                document.getElementById('userForm').reset();
            });
        }
    </script>
</body>
</html>
Paso 2: Crear el Script PHP para Manejar las Operaciones CRUD
Crea un archivo llamado actions.php para manejar las operaciones CRUD:

php
Copiar código
<?php
$servername = "mysql";
$username = "serviceuser";
$password = "servicepassword";
$dbname = "servicedb";

// Crear conexión
$conn = new mysqli($servername, $username, $password, $dbname);

// Verificar conexión
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$action = $_POST['action'] ?? $_GET['action'];

switch($action) {
    case 'create':
        $nombre = $_POST['nombre'];
        $apellido = $_POST['apellido'];
        $telefono = $_POST['telefono'];
        $direccion = $_POST['direccion'];
        $stmt = $conn->prepare("INSERT INTO users (nombre, apellido, telefono, direccion) VALUES (?, ?, ?, ?)");
        $stmt->bind_param("ssss", $nombre, $apellido, $telefono, $direccion);
        if ($stmt->execute()) {
            echo "Usuario agregado exitosamente";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
        break;

    case 'read':
        $result = $conn->query("SELECT * FROM users");
        $users = [];
        while($row = $result->fetch_assoc()) {
            $users[] = $row;
        }
        echo json_encode($users);
        break;

    case 'update':
        $id = $_POST['userId'];
        $nombre = $_POST['nombre'];
        $apellido = $_POST['apellido'];
        $telefono = $_POST['telefono'];
        $direccion = $_POST['direccion'];
        $stmt = $conn->prepare("UPDATE users SET nombre=?, apellido=?, telefono=?, direccion=? WHERE id=?");
        $stmt->bind_param("ssssi", $nombre, $apellido, $telefono, $direccion, $id);
        if ($stmt->execute()) {
            echo "Usuario actualizado exitosamente";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
        break;

    case 'delete':
        $id = $_POST['userId'];
        $stmt = $conn->prepare("DELETE FROM users WHERE id=?");
        $stmt->bind_param("i", $id);
        if ($stmt->execute()) {
            echo "Usuario eliminado exitosamente";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
        break;

    default:
        echo "Acción no válida";
}

$conn->close();
?>
Paso 3: Crear el Script SQL para la Base de Datos
Crea un archivo init.sql para inicializar la base de datos y la tabla users:

sql
Copiar código
CREATE DATABASE IF NOT EXISTS servicedb;

USE servicedb;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(255) NOT NULL,
    apellido VARCHAR(255) NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    direccion VARCHAR(255






