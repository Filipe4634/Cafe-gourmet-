<?php

// Inicia a sessão
session_start();

// Conexão com o banco de dados
include 'db_connect.php'; 

// Configura cabeçalho para JSON
header('Content-Type: application/json');

// Verifica se o usuário está logado
if (!isset($_SESSION['user_id'])) {
    echo json_encode(['success' => false, 'message' => 'Usuário não autenticado']);
    exit;
}

// Recebe os dados enviados via POST
$data = json_decode(file_get_contents('php://input'), true);
$cart_item_id = $data['id'] ?? null;

// Verifica se o ID do item foi enviado
if (!$cart_item_id) {
    echo json_encode(['success' => false, 'message' => 'ID do item não fornecido']);
    exit;
}

// Remove o item do carrinho
$sql = "DELETE FROM cart WHERE id = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("i", $cart_item_id);

if ($stmt->execute()) {
    echo json_encode(['success' => true, 'message' => 'Item removido com sucesso']);
} else {
    echo json_encode(['success' => false, 'message' => 'Erro ao remover item do carrinho']);
}

$stmt->close();
$conn->close();
?>
