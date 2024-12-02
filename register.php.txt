<?php
// Conexão com o banco de dados
include 'db_connect.php';

// Verifica se o formulário foi enviado
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Captura os dados do formulario
    $name = $_POST['name'] ?? '';
    $password = $_POST['password'] ?? '';
    $email = $_POST['email'] ?? '';
    $cpf = $_POST['cpf'] ?? '';
    $postal_code = $_POST['postal_code'] ?? '';
    $address = $_POST['address'] ?? '';
    $number = $_POST['number'] ?? '';
    $district = $_POST['district'] ?? '';
    $city = $_POST['city'] ?? '';
    $state = $_POST['state'] ?? '';

    // Valida os campos obrigatórios
    if (empty($name) || empty($password) || empty($email)) {
        echo "Por favor, preencha os campos obrigatórios (name, Senha e E-mail).";
    } elseif (!preg_match("/^\d{3}\.\d{3}\.\d{3}-\d{2}$/", $cpf)) {
        echo "CPF inválido! Insira no formato: 000.000.000-00.";
    } elseif (!preg_match("/^\d{5}-\d{3}$/", $postal_code)) {
        echo "CEP inválido! Insira no formato: 00000-000.";
    } else {
        // Criptografa a senha antes de armazenar
        $encryptedPassword = password_hash($password, PASSWORD_BCRYPT);

        // Inseri os dados no banco de dados
        $sql = "INSERT INTO users (name, password, email, cpf, postal_code, address, `number`, district, city, state) 
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)";

        $stmt = $conn->prepare($sql);
        $stmt->bind_param("ssssssssss", $name, $encryptedPassword, $email, $cpf, $postal_code, $address, $number, $district, $city, $state);

        if ($stmt->execute()) {
            echo "<script>alert('Cadastro realizado com sucesso!');</script>";
            echo "<script>window.location.href = 'login.php';</script>";
            exit();
        } else {
            echo "Erro ao cadastrar usuário: " . $stmt->error;
        }

        $stmt->close();
    }
}

$conn->close();
?>

<!DOCTYPE html>
<html lang="pt-br">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Cadastro</title>
        <!-- CSS do Bootstrap e ícones -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
        <link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-icons/1.10.3/font/bootstrap-icons.min.css" rel="stylesheet">
        <link rel="stylesheet" href="style.css">
    </head>

    <body>
        <header class="bg-dark text-white p-3">
            <div class="container d-flex justify-content-between align-items-center">
                <div class="logo-container">
                    <img src="https://i.ibb.co/VDhBwwW/rb-54432.png" alt="Store Logo" class="logo-image">
                    <span class="logo-text">Coffee Shop</span>
                </div>
                <nav>
                    <a href="login.php" class="btn btn-secondary ms-2">Voltar</a>
                </nav>
            </div>
        </header>

        <section class="hero bg-light py-5 text-center">
            <div class="container">
                <h1>Cadastro</h1>
                <p>Preencha seus dados para cadastro.</p>
            </div>
        </section>

        <section class="section container py-5">
            <form action="register.php" method="POST" class="row g-3">
                <div class="col-md-6">
                    <label for="name" class="form-label">Nome</label>
                    <input type="text" name="name" class="form-control" id="name" placeholder="Nome" required>
                </div>
                <div class="col-md-6">
                    <label for="password" class="form-label">Senha</label>
                    <input type="password" name="password" class="form-control" id="password" placeholder="Senha" required>
                </div>
                <div class="col-md-6">
                    <label for="email" class="form-label">E-mail</label>
                    <input type="email" name="email" class="form-control" id="email" placeholder="E-mail" required>
                </div>
                <div class="col-md-6">
                    <label for="cpf" class="form-label">CPF</label>
                    <input type="text" name="cpf" class="form-control" id="cpf" placeholder="000.000.000-00" required>
                </div>
                <div class="col-md-6">
                    <label for="postal_code" class="form-label">CEP</label>
                    <input type="text" name="postal_code" class="form-control" id="postal_code" placeholder="00000-000" required>
                </div>
                <div class="col-md-6">
                    <label for="address" class="form-label">Endereço</label>
                    <input type="text" name="address" class="form-control" id="address" placeholder="Endereço" required>
                </div>
                <div class="col-md-6">
                    <label for="number" class="form-label">Número</label>
                    <input type="text" name="number" class="form-control" id="number" placeholder="Número" required>
                </div>
                <div class="col-md-6">
                    <label for="district" class="form-label">Bairro</label>
                    <input type="text" name="district" class="form-control" id="district" placeholder="Bairro" required>
                </div>
                <div class="col-md-6">
                    <label for="city" class="form-label">Cidade</label>
                    <input type="text" name="city" class="form-control" id="city" placeholder="Cidade" required>
                </div>
                <div class="col-md-6">
                    <label for="state" class="form-label">Estado</label>
                    <input type="text" name="state" class="form-control" id="state" placeholder="Estado" required>
                </div>
                <div class="col-12 text-end">
                    <button type="submit" class="btn btn-success">Cadastrar</button>
                </div>
            </form>
        </section>

        <footer class="bg-dark text-white text-center py-3">
            <div class="container">
                <h5>Contato</h5>
                <p><i class="bi bi-telephone"></i> Telefone: (xx) xxxxx-xxxx</p>
                <p><i class="bi bi-whatsapp"></i> WhatsApp: (xx) xxxxx-xxxx</p>
                <p><i class="bi bi-envelope"></i> E-mail: atendimento@loja.com.br</p>
                <hr class="border-top border-light">
            </div>
            &copy; 2024 Projeto Transdisciplinar.
        </footer>

        <!-- JS do Bootstrap -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
        <!-- Biblioteca Inputmask -->
        <script src="https://cdn.jsdelivr.net/npm/inputmask/dist/inputmask.min.js"></script>
        <script src="script.js"></script>

    </body>

</html>
