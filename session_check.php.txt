<?php
// Inicia a sessão
session_start();

// Páginas permitidas sem login
$unprotected_pages = ['login.php', 'register.php'];

// Verifica se o usuário nõa está logado
if (!isset($_SESSION['user_id'])) {

    // Verifica se a página atual não está na lista de páginas liberadas
    $current_page = basename($_SERVER['PHP_SELF']);
    if (!in_array($current_page, $unprotected_pages)) {
        header("Location: login.php");
        exit;
    }
}
?>
