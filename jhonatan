<?php
session_start();

// Nome do arquivo onde as reuniões serão salvas
$file = "reunioes.txt";

$mensagem = "";

// Verificar se o usuário está logado
$isLogged = isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;

// Função para gerar um ID único
function gerarId() {
    return uniqid();
}

// Processar login
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['login'])) {
    $username = trim($_POST['username']);
    $password = trim($_POST['password']);

    // Credenciais fictícias (substitua por um sistema real se necessário)
    $validUsername = "admin";
    $validPassword = "12345";

    if ($username === $validUsername && $password === $validPassword) {
        $_SESSION['logged_in'] = true;
        header("Location: " . $_SERVER['PHP_SELF']);
        exit();
    } else {
        $mensagem = "Usuário ou senha inválidos.";
    }
}

// Processar logout
if (isset($_GET['logout'])) {
    session_destroy();
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

// Processar adição, edição ou exclusão de reuniões apenas se o usuário estiver logado
if ($isLogged && $_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['reuniao']) && isset($_POST['data'])) {
    $reuniao = trim($_POST['reuniao']);
    $data = trim($_POST['data']);
    $dataEdicao = date('d/m/Y H:i:s');
    $status = isset($_POST['status']) ? trim($_POST['status']) : "Pendente";

    if (!empty($reuniao) && !empty($data)) {
        $reunioes = file_exists($file) ? file($file, FILE_IGNORE_NEW_LINES) : [];

        if (isset($_POST['id'])) {
            // Editar reunião existente
            $id = $_POST['id'];
            foreach ($reunioes as $indice => $linha) {
                $dados = explode("|", $linha);
                if ($dados[0] === $id) {
                    $reunioes[$indice] = "$id|$reuniao|$data|$dataEdicao|$status";
                    break;
                }
            }
            $mensagem = "Reunião editada com sucesso!";
        } else {
            // Criar uma nova reunião com ID único
            $id = gerarId();
            $reunioes[] = "$id|$reuniao|$data|$dataEdicao|$status";
            $mensagem = "Reunião salva com sucesso!";
        }

        // Salvar as reuniões no arquivo
        file_put_contents($file, implode("\n", $reunioes) . "\n");
    } else {
        $mensagem = "Por favor, preencha todos os campos.";
    }
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

// Processar exclusão
if ($isLogged && isset($_POST['excluir'])) {
    $idExcluir = $_POST['excluir'];
    $reunioes = file_exists($file) ? file($file, FILE_IGNORE_NEW_LINES) : [];
    foreach ($reunioes as $indice => $linha) {
        $dados = explode("|", $linha);
        if ($dados[0] === $idExcluir) {
            unset($reunioes[$indice]);
            break;
        }
    }
    file_put_contents($file, implode("\n", $reunioes) . "\n");
    $mensagem = "Reunião excluída com sucesso!";
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

// Processar conclusão
if ($isLogged && isset($_POST['concluir'])) {
    $idConcluir = $_POST['concluir'];
    $reunioes = file_exists($file) ? file($file, FILE_IGNORE_NEW_LINES) : [];
    foreach ($reunioes as $indice => $linha) {
        $dados = explode("|", $linha);
        if ($dados[0] === $idConcluir) {
            $dados[4] = "Concluido";
            $reunioes[$indice] = implode("|", $dados);
            break;
        }
    }
    file_put_contents($file, implode("\n", $reunioes) . "\n");
    $mensagem = "Reunião marcada como concluída!";
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

// Carregar reuniões existentes
$reunioes = file_exists($file) ? file($file, FILE_IGNORE_NEW_LINES) : [];

// Separar reuniões agendadas e concluídas
$reunioesAgendadas = [];
$reunioesConcluidas = [];
foreach ($reunioes as $reuniao) {
    $dados = explode("|", $reuniao);
    $status = trim($dados[4]);
    if ($status === "Concluido") {
        $reunioesConcluidas[] = $reuniao;
    } else {
        $reunioesAgendadas[] = $reuniao;
    }
}

// Função para determinar a cor da reunião com base na data
function getCorReuniao($data) {
    $hoje = new DateTime();
    $dataReuniao = new DateTime($data);
    $diff = $hoje->diff($dataReuniao);

    if ($dataReuniao < $hoje) {
        return "text-danger"; // Vermelho: Data já passou
    } elseif ($diff->days <= 4 && $dataReuniao >= $hoje) {
        return "text-warning"; // Amarelo: Faltam 4 dias ou menos
    }
    return ""; // Sem cor
}

// Filtrar reuniões concluídas por mês
if (isset($_GET['mes']) && !empty($_GET['mes'])) {
    $mes = $_GET['mes'];
    $reunioesConcluidas = array_filter($reunioesConcluidas, function ($reuniao) use ($mes) {
        $dados = explode("|", $reuniao);
        $data = DateTime::createFromFormat('Y-m-d', trim($dados[2]));
        return $data && $data->format('m') === $mes;
    });
}
?>

<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cronograma de Atualização - Páginas de Contratação</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container py-5">
        <h1 class="mb-4 text-center">Cronograma de Atualização - Páginas de Contratação</h1>

        <?php if (!$isLogged) : ?>
            <!-- Formulário de login -->
            <div class="card mb-4">
                <div class="card-body">
                    <h2 class="mb-4 text-center">Login</h2>
                    <form method="POST" action="">
                        <div class="mb-3">
                            <label for="username" class="form-label">Usuário:</label>
                            <input type="text" id="username" name="username" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label for="password" class="form-label">Senha:</label>
                            <input type="password" id="password" name="password" class="form-control" required>
                        </div>
                        <button type="submit" name="login" class="btn btn-primary w-100">Entrar</button>
                    </form>
                </div>
            </div>
        <?php else : ?>
            <!-- Mostrar botão de logout -->
            <div class="text-end mb-4">
                <a href="?logout=true" class="btn btn-danger">Sair</a>
            </div>

            <!-- Formulário de criação de reunião -->
            <div class="card mb-4">
                <div class="card-body">
                    <form method="POST" action="">
                        <div class="mb-3">
                            <label for="reuniao" class="form-label">Nome da Reunião:</label>
                            <input type="text" id="reuniao" name="reuniao" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label for="data" class="form-label">Data:</label>
                            <input type="date" id="data" name="data" class="form-control" required>
                        </div>
                        <button type="submit" class="btn btn-primary w-100">Salvar Reunião</button>
                    </form>
                </div>
            </div>
        <?php endif; ?>

        <!-- Mostrar reuniões agendadas -->
        <h2 class="mb-3">Proximas atualizações</h2>
        <ul class="list-group">
            <?php foreach ($reunioesAgendadas as $reuniao) : ?>
                <?php
                list($id, $nome, $data, $dataEdicao, $status) = explode("|", $reuniao);
                $cor = getCorReuniao($data);
                ?>
                <li class="list-group-item d-flex justify-content-between align-items-center <?php echo $cor; ?>">
                    <div>
                        <strong><?php echo htmlspecialchars($nome); ?></strong><br>
                        <small>Data: <?php echo htmlspecialchars((new DateTime($data))->format('d/m/Y')); ?></small><br>
                        <small>Editado em: <?php echo htmlspecialchars($dataEdicao); ?></small><br>
                        <small>Status: <?php echo htmlspecialchars($status); ?></small>
                    </div>
                    <?php if ($isLogged) : ?>
                        <div>
                            <form method="POST" action="" class="d-inline">
                                <button type="submit" name="concluir" value="<?php echo $id; ?>" class="btn btn-success btn-sm">Concluir</button>
                            </form>
                            <form method="POST" action="" class="d-inline">
                                <input type="hidden" name="id" value="<?php echo $id; ?>">
                                <input type="text" name="reuniao" value="<?php echo htmlspecialchars($nome); ?>" class="form-control d-inline w-auto" required>
                                <input type="date" name="data" value="<?php echo htmlspecialchars($data); ?>" class="form-control d-inline w-auto" required>
                                <button type="submit" class="btn btn-warning btn-sm">Salvar</button>
                            </form>
                            <form method="POST" action="" class="d-inline">
                                <button type="submit" name="excluir" value="<?php echo $id; ?>" class="btn btn-danger btn-sm">Excluir</button>
                            </form>
                        </div>
                    <?php endif; ?>
                </li>
            <?php endforeach; ?>
        </ul>

        <!-- Mostrar reuniões concluídas -->
        <h2 class="mt-5 mb-3">Atualizações realizadas</h2>
        <form method="GET" action="" class="mb-3">
            <div class="input-group">
                <select name="mes" class="form-select">
                    <option value="">Todos os meses</option>
                    <?php for ($i = 1; $i <= 12; $i++) : ?>
                        <option value="<?php echo str_pad($i, 2, '0', STR_PAD_LEFT); ?>">
                            <?php echo DateTime::createFromFormat('!m', $i)->format('F'); ?>
                        </option>
                    <?php endfor; ?>
                </select>
                <button type="submit" class="btn btn-primary">Filtrar</button>
            </div>
        </form>
        <ul class="list-group">
            <?php foreach ($reunioesConcluidas as $reuniao) : ?>
                <?php
                list($id, $nome, $data, $dataEdicao, $status) = explode("|", $reuniao);
                ?>
                <li class="list-group-item">
                    <strong><?php echo htmlspecialchars($nome); ?></strong><br>
                    <small>Data: <?php echo htmlspecialchars((new DateTime($data))->format('d/m/Y')); ?></small><br>
                    <small>Editado em: <?php echo htmlspecialchars($dataEdicao); ?></small><br>
                    <small>Status: <?php echo htmlspecialchars($status); ?></small>
                </li>
            <?php endforeach; ?>
        </ul>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
