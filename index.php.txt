<?php
// Autenticação
include 'session_check.php'; 

// Conexão com o banco de dados
include 'db_connect.php'; 

// Buscar os produtos no banco de dados
$sql = "SELECT * FROM products";
$result = $conn->query($sql);

// Fecha a conexão com o banco de dados
$conn->close(); 

?>

<!DOCTYPE html>
<html lang="pt-br">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Produtos</title>
        <!-- CSS do Bootstrap e ícones -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
        <link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-icons/1.10.3/font/bootstrap-icons.min.css" rel="stylesheet">
        <link rel="stylesheet" href="style.css">
    </head>

    <body>
        <!-- Seção de Cabeçalho -->
        <header class="bg-dark text-white p-3">
            <div class="container d-flex justify-content-between align-items-center">
                <!-- Contêiner de Logo -->
                <div class="logo-container">
                    <img src="https://i.ibb.co/VDhBwwW/rb-54432.png" alt="Logo da Loja" class="logo-image">
                    <span class="logo-text">Coffee Shop</span>
                </div>
                <!-- Botões de Navegação -->
                <nav class="d-flex align-items-center">
                    <button id="ordersButton" class="btn btn-outline-light me-2">
                        <i class="bi bi-list-check"></i> Meus Pedidos
                    </button>
                    <button id="cartButton" class="btn btn-outline-light position-relative ms-2">
                        <i class="bi bi-cart"></i> Carrinho
                        <span id="cartQuantityBadge" class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger">0</span>
                    </button>
                    <button id="logoutButton" class="btn btn-outline-light ms-2">
                        <i class="bi bi-box-arrow-right"></i> Sair
                    </button>
                </nav>
            </div>
        </header>

        <!-- Seção Hero -->
        <section class="hero bg-light py-5 text-center">
            <div class="container">
                <h1>Bem-vindo à nossa loja</h1>
                <p>Descubra nossa linha exclusiva de café gourmet.</p>
            </div>
        </section>

        <!-- Seção de Produtos -->
        <section class="section container py-5">
            <h2 class="mb-4 text-center">Nossos produtos</h2>
            <div class="d-flex flex-wrap justify-content-center">
                <?php
                if ($result->num_rows > 0) {
                    // Exibir os produtos encontrados
                    while($row = $result->fetch_assoc()) {
                        echo '
                        <div class="product-card card mx-2 mb-3">
                            <img src="' . $row['image_url'] . '" class="card-img-top" alt="' . $row['name'] . '">
                            <div class="card-body text-center">
                                <h5 class="card-title">' . $row['name'] . '</h5>
                                <p class="card-text">R$' . number_format($row['price'], 2, ',', '.') . '</p>
                                <button class="btn btn-primary" onclick="addToCart(' . $row['id'] . ')">Comprar</button>
                            </div>
                        </div>';
                    }
                } else {
                    // Caso não haja produtos
                    echo "<p>Nenhum produto encontrado</p>";
                }
                ?>
            </div>
        </section>

        <!-- Seção de Rodapé -->
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

        <!-- Modal do Carrinho -->
        <div id="cartModal" class="modal">
            <div class="modal-content">
                <span class="close-button" id="closeCartButton">&times;</span>
                <h2>Seu Carrinho</h2>
                <div id="cartItemsContainer">
                    <!-- Os itens serão adicionados dinamicamente aqui -->
                </div>
                <div class="cart-total mt-3">
                    <p>Total: <span id="cartTotal">R$0,00</span></p>
                </div>
                <button id="finalizeOrderButton" class="btn btn-success w-100 mt-3">Finalizar Pedido</button>
            </div>
        </div>

        <!-- JS do Bootstrap -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
        <script src="script.js"></script>
        
    </body>

</html>

