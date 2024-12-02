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

// Consulta todos os pedidos
$sql = "SELECT * FROM orders";
$result = $conn->query($sql);

?>

<!DOCTYPE html>
<html lang="pt-br">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Painel de Administração</title>
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
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav ms-auto">
                        <li class="nav-item">
                            <a class="nav-link active" href="admin_dashboard.php">Pedidos</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="logout.php">Sair</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>

        <div class="container my-5">
            <div class="row">
                <div class="col-12 text-center">
                    <h1 class="display-4">Todos os Pedidos</h1>
                    <p class="lead">Gerencie os pedidos realizados por todos os usuários.</p>
                </div>
            </div>

            <?php if ($result->num_rows > 0): ?>
                <div class="table-responsive">
                    <table class="table table-striped table-hover">
                        <thead class="thead-dark">
                            <tr>
                                <th>#</th>
                                <th>Usuário</th>
                                <th>Data</th>
                                <th>Status</th>
                                <th>Ações</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php while ($row = $result->fetch_assoc()): ?>
                                <?php if (isset($row['order_id'])): ?>
                                    <tr>
                                        <td><?php echo $row['order_id']; ?></td>
                                        <td><?php echo $row['user_id']; ?></td>
                                        <td><?php echo $row['order_date']; ?></td>
                                        <td>
                                            <span class="badge <?php echo $row['payment_status'] == 'pendente' ? 'bg-warning' : 'bg-success'; ?>">
                                                <?php echo $row['payment_status']; ?>
                                            </span>
                                        </td>
                                        <td>
                                            <a href="view_order.php?id=<?php echo $row['order_id']; ?>" class="btn btn-info btn-sm">Ver</a>
                                        </td>
                                    </tr>
                                <?php else: ?>
                                    <tr>
                                        <td colspan="5" class="text-center">Pedido não encontrado ou excluído.</td>
                                    </tr>
                                <?php endif; ?>
                            <?php endwhile; ?>
                        </tbody>
                    </table>
                </div>
            <?php else: ?>
                <div class="alert alert-info" role="alert">
                    Nenhum pedido encontrado.
                </div>
            <?php endif; ?>
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
