<?php
// Conexão com o banco de dados
include 'db_connect.php';

// Inicia a sessão
session_start();

// Variável para armazenar mensagens de erro
$errorMessage = "";

// Verifica se o formulário foi enviado
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Capturar os dados do formulário
    $email = $_POST['email'] ?? '';
    $password = $_POST['password'] ?? '';

    // Valida se os campos não estão vazios
    if (empty($email) || empty($password)) {
        $errorMessage = "Por favor, preencha todos os campos.";
    } else {
        // Consulta o banco de dados para verificar o e-mail
        $sql = "SELECT * FROM users WHERE email = ?";  
        $stmt = $conn->prepare($sql);
        $stmt->bind_param("s", $email);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($result->num_rows > 0) {
            // Usuário encontrado no banco de dado
            $user = $result->fetch_assoc();
            if (password_verify($password, $user['password'])) {
                
                // Armazena o ID e o e-mail do usuário na sessão
                $_SESSION['user_id'] = $user['id'];  
                $_SESSION['user_email'] = $user['email'];  

                // Verifica o role para saber se o usuário é admin
                if ($user['role'] == 'admin') {

                    // Se for admin, redireciona para o painel de administração
                    $_SESSION['role'] = 'admin';  
                    header("Location: admin_dashboard.php");  
                    exit();  
                } else {

                    // Se for usuário normal, redireciona para a página principal
                    $_SESSION['role'] = 'user';  
                    header("Location: index.php");  
                    exit();  
                }
            } else {
                $errorMessage = "Senha incorreta!";
            }
        } else {
            $errorMessage = "Usuário não encontrado!";
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
        <title>Login</title>
        <!-- CSS do Bootstrap -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
        <link rel="stylesheet" href="style.css">
    </head>

    <body>

        <header class="bg-dark text-white p-3">
            <div class="container d-flex justify-content-between align-items-center">
                <div class="logo-container">
                    <img src="https://i.ibb.co/VDhBwwW/rb-54432.png" alt="Store Logo" class="logo-image">
                    <span class="logo-text">Coffee Shop</span>
                </div>
            </div>
        </header>

        <section class="hero bg-light py-5 d-flex justify-content-center align-items-center" style="min-height: calc(100vh - 100px);">
            <div class="login-container p-4 bg-white rounded shadow" style="max-width: 400px; width: 90%;">
                <h2 class="text-center mb-4">Login</h2>
                <p class="text-center">Preencha seus dados para entrar!</p>
                <form action="login.php" method="POST">
                    <div class="mb-3">
                        <label for="email" class="form-label">E-mail</label>
                        <input type="email" id="email" name="email" class="form-control" placeholder="Digite seu e-mail" required>
                    </div>
                    <div class="mb-3">
                        <label for="password" class="form-label">Senha</label>
                        <input type="password" id="password" name="password" class="form-control" placeholder="Digite sua senha" required>
                    </div>
                    <?php if (!empty($errorMessage)): ?>
                        <div class="error-message"><?php echo $errorMessage; ?></div>
                    <?php endif; ?>
                    <button type="submit" class="btn btn-success w-100 mt-3">Entrar</button>
                </form>
                <div class="register-link mt-3 text-center">
                    <a href="register.php" class="text-decoration-none">Novo por aqui? Cadastre-se</a>
                </div>
            </div>
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
    </body>

</html>
