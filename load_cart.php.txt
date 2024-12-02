<?php
// Inicia a sessão
session_start();

// Conexão com o banco de dados
include 'db_connect.php'; 

// Configura cabeçalho para retorno JSON
header('Content-Type: application/json');

// Verifica se o usuário está logado
if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'Usuário não autenticado']);
    exit;
}

// Captura o ID do usuário logado
$user_id = $_SESSION['user_id'];

// Capturar saídas inesperadas
ob_start();

try {
    // Atualiza consulta para garantir qeu o JOIN com a tabela `products` funcione
    $sql = "
        SELECT 
            cart.id AS cart_id,
            cart.product_id,
            cart.quantity,
            cart.added_at,
            products.name AS product_name,
            products.price AS product_price
        FROM cart
        JOIN products ON cart.product_id = products.id
        WHERE cart.user_id = ?
    ";

    $stmt = $conn->prepare($sql);
    $stmt->bind_param("i", $user_id);
    $stmt->execute();
    $result = $stmt->get_result();

    $cart_items = [];
    while ($row = $result->fetch_assoc()) {
        $cart_items[] = [
            'id' => $row['cart_id'], 
            'product_id' => $row['product_id'], 
            'name' => $row['product_name'], 
            'price' => (float)$row['product_price'], 
            'quantity' => (int)$row['quantity'], 
        ];
    }

    $stmt->close();
    $conn->close();

    // Limpa qualquer saída inesperada
    ob_end_clean();

    // Verifica se encontrou itens no carrinho
    if (empty($cart_items)) {
        echo json_encode(['success' => true, 'items' => []]);
        exit;
    }

    // Retorna os itens do carrinho
    echo json_encode([
        'success' => true,
        'items' => $cart_items,
    ]);

} catch (Exception $e) {

    // Captura qualquer erro e limpar a saída
    ob_end_clean();

    // Log do erro
    error_log("Erro ao carregar itens do carrinho: " . $e->getMessage());

    // Retorna erro em formato JSON
    echo json_encode(['success' => false, 'message' => 'Erro ao carregar carrinho: ' . $e->getMessage()]);
}
?>
