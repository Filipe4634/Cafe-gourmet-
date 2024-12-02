<?php
// Inicia a sessão
session_start();

// Verifica se o usuário é administrador
if ($_SESSION['role'] !== 'admin') {
    header('Location: login.php');
    exit();
}

// Conexão com o banco de dados
include 'db_connect.php';

// Verifica se o ID do pedido foi passado
if (!isset($_GET['id']) || empty($_GET['id'])) {
    die("ID do pedido não fornecido!");
}

$order_id = $_GET['id'];

// Consulta os detalhes do pedido
$sql = "SELECT * FROM orders WHERE order_id = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("i", $order_id);
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows === 0) {
    die("Pedido não encontrado!");
}

$order = $result->fetch_assoc();
?>

<!DOCTYPE html>
<html lang="pt-br">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Detalhes do Pedido</title>
        <!-- CSS do Bootstrap -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
        <link rel="stylesheet" href="style.css">
    </head>

    <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
            <div class="container-fluid">
                <a class="navbar-brand dash-title" href="admin_dashboard.php">Painel de administração</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
            </div>
        </nav>

        <div class="container my-5">
            <h1 class="text-center">Detalhes do Pedido</h1>
            <div class="card my-4">
                <div class="card-header bg-info text-white">
                    Pedido ID: <?php echo htmlspecialchars($order['order_id']); ?>
                </div>
                <div class="card-body">
                    <p><strong>Usuário:</strong> <?php echo htmlspecialchars($order['user_id']); ?></p>
                    <p><strong>Data do Pedido:</strong> <?php echo htmlspecialchars($order['order_date']); ?></p>
                    <p><strong>Status do Pagamento:</strong> 
                        <span class="badge <?php echo $order['payment_status'] === 'pendente' ? 'bg-warning' : 'bg-success'; ?>">
                            <?php echo htmlspecialchars($order['payment_status']); ?>
                        </span>
                    </p>
                    <p><strong>Itens do Pedido:</strong></p>
                    <ul>
                        <?php
                        $items = json_decode($order['items'], true);
                        foreach ($items as $item) {
                            echo "<li>" . htmlspecialchars($item['name']) . " - R$" . number_format($item['price'], 2, ',', '.') . "</li>";
                        }
                        ?>
                    </ul>
                    <p><strong>Total:</strong> R$<?php echo number_format($order['total'], 2, ',', '.'); ?></p>
                </div>
            </div>
            <a href="admin_dashboard.php" class="btn btn-secondary">Voltar</a>
        </div>

        <footer class="bg-dark text-white text-center py-3">
            <div class="container">
                <p>&copy; 2024 Projeto Transdisciplinar - Coffee Shop</p>
            </div>
        </footer>

        <!-- JS do Bootstrap -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    </body>
</html>
