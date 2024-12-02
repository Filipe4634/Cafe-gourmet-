<?php
// Autenticação
include 'session_check.php';

// Captura o ID do usuário da sessão
$user_id = $_SESSION['user_id']; 

// Conecta ao banco de dados
include 'db_connect.php';

// Busca os pedidos do usuário
$sql = "SELECT order_id, order_date, total, payment_status, items FROM orders WHERE user_id = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("d", $user_id);
$stmt->execute();
$result = $stmt->get_result();

// Verifica se o usuário tem pedidos
$orders = [];
if ($result->num_rows > 0) {
    while ($order = $result->fetch_assoc()) {
        $order['items'] = json_decode($order['items'], true);
        $orders[] = $order;
    }
}

$stmt->close();
$conn->close();
?>

<!DOCTYPE html>
<html lang="pt-br">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Meus Pedidos</title>
        <!-- CSS do Bootstrap e ícones -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
        <link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-icons/1.10.3/font/bootstrap-icons.min.css" rel="stylesheet">
        <link rel="stylesheet" href="style.css">
    </head>

    <body>
        <header class="bg-dark text-white p-3">
            <div class="container d-flex justify-content-between align-items-center">
                <!-- Logo Container -->
                <div class="logo-container">
                    <img src="https://i.ibb.co/VDhBwwW/rb-54432.png" alt="Store Logo" class="logo-image">
                    <span class="logo-text">Coffee Shop</span>
                </div>
                <nav>
                    <a href="index.php" class="btn btn-secondary ms-2">Voltar</a>
                </nav>
            </div>
        </header>

        <section class="hero bg-light py-5 text-center">
            <div class="container">
                <h1>Meus Pedidos</h1>
                <p>Acompanhe seus pedidos realizados.</p>
            </div>
        </section>

        <section class="section container py-5" id="ordersContainer">
            <?php if (empty($orders)) : ?>
                <p class="text-center">Você ainda não possui pedidos cadastrados.</p>
            <?php else : ?>
                <?php foreach ($orders as $order) : ?>
                    <div class="card p-3 mb-3">
                        <h5>Pedido #<?php echo $order['order_id']; ?></h5>
                        <p><strong>Data:</strong> <?php echo date('d/m/Y H:i', strtotime($order['order_date'])); ?></p>
                        <p><strong>Status:</strong> <?php echo htmlspecialchars($order['payment_status']); ?></p>
                        <p><strong>Total:</strong> R$ <?php echo number_format($order['total'], 2, ',', '.'); ?></p>
                        <p><strong>Itens:</strong></p>
                        <ul>
                            <?php foreach ($order['items'] as $item) : ?>
                                <li><?php echo htmlspecialchars($item['name']); ?> - R$ <?php echo number_format($item['price'], 2, ',', '.'); ?></li>
                            <?php endforeach; ?>
                        </ul>
                    </div>
                <?php endforeach; ?>
            <?php endif; ?>
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
