<?php
// Autenticação
include 'session_check.php';

// Conexão com o banco de dados
include 'db_connect.php'; 

// Verificar se o usuário está logado
if (!isset($_SESSION['user_id'])) {
    echo "Usuário não autenticado!";
    exit;
}

// Captura o ID do usuário da sessão
$user_id = $_SESSION['user_id'];

// Busca os itens do carrinho do usuário
$sql = "
    SELECT c.product_id, c.quantity, p.name AS product_name, p.price AS product_price 
    FROM cart c
    JOIN products p ON c.product_id = p.id
    WHERE c.user_id = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("i", $user_id);
$stmt->execute();
$result = $stmt->get_result();

// Verificar se há itens no carrinho
if ($result->num_rows === 0) {
    echo "<script>alert('O carrinho está vazio. Adicione produtos antes de finalizar o pedido.');</script>";
    echo "<script>window.location.href = 'index.php';</script>";
    exit;
}


// Calcula o total do pedido
$items = [];
$total = 0;
while ($row = $result->fetch_assoc()) {
    $items[] = [
        'name' => $row['product_name'],
        'price' => $row['product_price'],
        'quantity' => $row['quantity']
    ];
    $total += $row['product_price'] * $row['quantity'];
}

// Inseri o pedido no banco de dados
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Verificar se o método de pagamento foi selecionado
    $payment_method = $_POST['paymentMethod'] ?? '';
    if (empty($payment_method)) {
        echo "Selecione uma forma de pagamento.";
        exit;
    }

    // Converte os itens para JSON
    $items_json = json_encode($items);

    // Inseri o pedido na tabela 'orders'
    $sql = "INSERT INTO orders (user_id, total, items, payment_status, payment_method) VALUES (?, ?, ?, ?, ?)";
    $stmt = $conn->prepare($sql);
    $payment_status = 'pendente'; // Status inicial do pagamento
    $stmt->bind_param("idsss", $user_id, $total, $items_json, $payment_status, $payment_method);

    if ($stmt->execute()) {
        // Limpa o carrinho do usuário
        $sql = "DELETE FROM cart WHERE user_id = ?";
        $stmt = $conn->prepare($sql);
        $stmt->bind_param("i", $user_id);
        $stmt->execute();

        echo "<script>alert('Pedido realizado com sucesso!'); window.location.href = 'orders.php';</script>";
    } else {
        echo "Erro ao finalizar o pedido: " . $stmt->error;
    }

    $stmt->close();
    $conn->close();
}
?>

<!DOCTYPE html>
<html lang="pt-br">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Confirmação de Pagamento</title>
        <!-- CSS do Bootstrap -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
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

        <div class="container mt-5">
            <h1 class="text-center mb-4">Confirmação de Pagamento</h1>
            <div class="row">
                <div class="col-md-6">
                    <h4>Resumo do Pedido</h4>
                    <ul class="list-group mb-3">
                        <?php foreach ($items as $item): ?>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                <?php echo htmlspecialchars($item['name']); ?>
                                <span>R$<?php echo number_format($item['price'], 2, ',', '.'); ?> x <?php echo $item['quantity']; ?></span>
                            </li>
                        <?php endforeach; ?>
                        <li class="list-group-item d-flex justify-content-between">
                            <strong>Total:</strong>
                            <span id="span-weight">R$<?php echo number_format($total, 2, ',', '.'); ?></span>
                        </li>
                    </ul>
                </div>

                <div class="col-md-6">
                    <h4>Escolha a Forma de Pagamento</h4>
                    <form method="POST">
                        <div class="form-check mb-2">
                            <input class="form-check-input" type="radio" name="paymentMethod" value="Cartão de Crédito" id="creditCard" required>
                            <label class="form-check-label" for="creditCard">Cartão de Crédito</label>
                        </div>
                        <div class="form-check mb-2">
                            <input class="form-check-input" type="radio" name="paymentMethod" value="Pix" id="pix">
                            <label class="form-check-label" for="pix">Pix</label>
                        </div>
                        <div class="form-check mb-3">
                            <input class="form-check-input" type="radio" name="paymentMethod" value="Boleto" id="boleto">
                            <label class="form-check-label" for="boleto">Boleto</label>
                        </div>
                        <button type="submit" class="btn btn-success w-100">Confirmar pagamento</button>
                    </form>
                </div>
            </div>
        </div>

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
