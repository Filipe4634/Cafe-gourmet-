<?php
// Inicia a sessão
session_start();

// Conexão com o banco de dados
include 'db_connect.php';

// Verifica se o usuário está logado
if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'Usuário não autenticado']);
    exit;
}

// Recebe os dados enviados via POST
$data = json_decode(file_get_contents('php://input'), true);
$product_id = $data['product_id'] ?? null;

// Verifica se o ID do produto foi enviado
if (!$product_id) {
    echo json_encode(['success' => false, 'message' => 'ID do produto não fornecido']);
    exit;
}

// Captura o ID do usuário logado
$user_id = $_SESSION['user_id'];

// Verifica se o produto já está no carrinho
$sql_check = "SELECT id, quantity FROM cart WHERE user_id = ? AND product_id = ?";
$stmt_check = $conn->prepare($sql_check);
$stmt_check->bind_param("ii", $user_id, $product_id);
$stmt_check->execute();
$result_check = $stmt_check->get_result();

if ($result_check->num_rows > 0) {

    // Produto já está no carrinho, atualizar a quantidade
    $cart_item = $result_check->fetch_assoc();
    $new_quantity = $cart_item['quantity'] + 1;

    $sql_update = "UPDATE cart SET quantity = ? WHERE id = ?";
    $stmt_update = $conn->prepare($sql_update);
    $stmt_update->bind_param("ii", $new_quantity, $cart_item['id']);
    $stmt_update->execute();
    $stmt_update->close();

    echo json_encode(['success' => true, 'message' => 'Quantidade atualizada no carrinho']);
} else {
    
    // Produto não está no carrinho, inserir um novo item
    $sql_insert = "INSERT INTO cart (user_id, product_id, quantity, added_at) VALUES (?, ?, 1, NOW())";
    $stmt_insert = $conn->prepare($sql_insert);
    $stmt_insert->bind_param("ii", $user_id, $product_id);

    if ($stmt_insert->execute()) {
        echo json_encode([
            'success' => true,
            'message' => 'Produto adicionado ao carrinho',
            'item' => [
                'id' => $stmt_insert->insert_id,
                'product_id' => $product_id,
                'quantity' => 1,
            ],
        ]);
    } else {
        echo json_encode(['success' => false, 'message' => 'Erro ao adicionar produto ao carrinho']);
    }

    $stmt_insert->close();
}

$stmt_check->close();
$conn->close();
?>
