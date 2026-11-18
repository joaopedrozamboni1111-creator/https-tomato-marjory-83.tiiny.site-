<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Meu Jogo Espacial</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #111;
            color: #fff;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        h1 {
            margin: 0 0 10px 0;
            font-size: 24px;
            color: #00ffcc;
        }
        #placar {
            font-size: 20px;
            margin-bottom: 10px;
        }
        #containerJogo {
            position: relative;
            width: min(400px, 94vw);
            height: min(550px, 68vh);
        }
        canvas {
            width: 100%;
            height: 100%;
            touch-action: none;
            background-color: #050510;
            border: 3px solid #00ffcc;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 255, 204, 0.2);
            display: block;
        }
        #painelNome {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 85%;
            background-color: rgba(15, 20, 35, 0.95);
            padding: 15px;
            border-radius: 8px;
            border: 1px solid #00ffcc;
            z-index: 10;
        }
        #painelNome label {
            font-size: 13px;
            color: #00ffcc;
            margin-bottom: 8px;
        }
        #inputNome {
            padding: 8px;
            font-size: 15px;
            border-radius: 4px;
            border: 1px solid #555;
            background-color: #222;
            color: #fff;
            text-align: center;
            width: 80%;
            margin-bottom: 12px;
            outline: none;
        }
        #inputNome:focus {
            border-color: #00ffcc;
        }
        #btnJogar {
            padding: 10px 24px;
            font-size: 16px;
            font-weight: bold;
            background-color: #00ffcc;
            color: #050510;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.2s;
        }
        #btnJogar:hover {
            background-color: #00cca3;
        }
        #controlesMobile {
            display: none;
            grid-template-columns: repeat(3, 56px);
            grid-template-rows: repeat(2, 48px);
            gap: 6px;
            margin-top: 10px;
            touch-action: none;
            user-select: none;
        }
        #controlesMobile button {
            border: 1px solid #00ffcc;
            border-radius: 8px;
            background: #17202b;
            color: #00ffcc;
            font-size: 22px;
            font-weight: bold;
            touch-action: none;
        }
        #controlesMobile button:active { background: #00ffcc; color: #050510; }
        @media (max-width: 600px) {
            body { height: 100dvh; justify-content: flex-start; padding-top: 8px; box-sizing: border-box; }
            h1 { font-size: 20px; }
            #placar { font-size: 17px; margin-bottom: 6px; }
            #controlesMobile { display: grid; }
            #painelNome { bottom: 12px; padding: 10px; }
        }
    </style>
</head>
<body>

    <h1>🚀 Meu Jogo Espacial</h1>
    <div id="placar">Pontos: 0</div>

    <div id="containerJogo">
        <canvas id="telaJogo" width="400" height="550"></canvas>

        <div id="painelNome">
            <label for="inputNome">Digite seu nome (opcional):</label>
            <input type="text" id="inputNome" maxlength="8" placeholder="Seu nome aqui">
            <button type="button" id="btnJogar" onclick="iniciarPartida()">COMEÇAR JOGO</button>
        </div>
    </div>
    <div id="controlesMobile" aria-label="Controles do jogo">
        <span></span><button data-direcao="ArrowUp" aria-label="Cima">▲</button><span></span>
        <button data-direcao="ArrowLeft" aria-label="Esquerda">◀</button>
        <button data-direcao="ArrowDown" aria-label="Baixo">▼</button>
        <button data-direcao="ArrowRight" aria-label="Direita">▶</button>
    </div>

    <script>
        const canvas = document.getElementById("telaJogo");
        const ctx = canvas.getContext("2d");
        const placarElemento = document.getElementById("placar");
        const painelNome = document.getElementById("painelNome");
        const inputNome = document.getElementById("inputNome");

        // CONFIGURAÇÕES FÁCEIS DE ALTERAR
        const CONFIG = {
            corJogador: "#ffffff",
            corTextoJogador: "#050510",
            velocidadeJogador: 6,
            corObstaculo: "#ff4444",
            velocidadeObstaculoInicial: 4,
            frequenciaObstaculos: 35
        };

        // Estados do jogo: "INICIO", "JOGANDO", "GAMEOVER"
        let estadoJogo = "INICIO";
        let pontos = 0;
        let frames = 0;
        let nomeJogador = "";
        let direcaoAtual = "CIMA";

        // Carregar ranking com segurança
        let ranking = [];
        try {
            ranking = JSON.parse(localStorage.getItem("rankingJogoEspacial")) || [];
        } catch (e) {
            ranking = [];
        }

        // Jogador (Quadrado Branco)
        const jogador = {
            x: canvas.width / 2 - 25,
            y: canvas.height - 80,
            largura: 50,
            altura: 50
        };

        let obstaculos = [];
        let teclas = {};

        // Função para iniciar o jogo
        function iniciarPartida() {
            nomeJogador = inputNome.value.trim();
            inputNome.blur(); // Tira o cursor da caixa de texto
            painelNome.style.display = "none"; // Esconde o painel
            teclas = {}; // Limpa teclas presas
            estadoJogo = "JOGANDO";
        }

        // Reiniciar o jogo (Mantém o nome ou permite trocar caso o painel volte)
        function reiniciarJogo() {
            pontos = 0;
            frames = 0;
            placarElemento.innerText = "Pontos: 0";
            jogador.x = canvas.width / 2 - 25;
            jogador.y = canvas.height - 80;
            obstaculos = [];
            teclas = {};
            estadoJogo = "JOGANDO";
        }

        // Função chamada no Game Over para abrir a opção de trocar nome novamente
        function abrirTrocaNome() {
            painelNome.style.display = "flex";
            inputNome.focus();
            estadoJogo = "INICIO";
        }

        // Apertar ENTER dentro da caixa de texto também inicia
        inputNome.addEventListener("keyup", function(e) {
            if (e.key === "Enter") {
                iniciarPartida();
            }
        });

        // Controles do teclado
        window.addEventListener("keydown", function(e) {
            // Se estiver digitando o nome, ignora atalhos do jogo
            if (document.activeElement === inputNome) {
                return;
            }

            teclas[e.key] = true;

            if (e.code === "Space") {
                e.preventDefault(); // Evita rolar a página
                if (estadoJogo === "INICIO") {
                    iniciarPartida();
                } else if (estadoJogo === "GAMEOVER") {
                    reiniciarJogo();
                }
            }

            // Atalho opcional na tela de Game Over para mudar de nome
            if ((e.key === "c" || e.key === "C") && estadoJogo === "GAMEOVER") {
                abrirTrocaNome();
            }
        });

        window.addEventListener("keyup", function(e) {
            teclas[e.key] = false;
        });

        // Controles por toque: funcionam em celulares e tablets
        document.querySelectorAll("#controlesMobile button").forEach(botao => {
            const direcao = botao.dataset.direcao;
            const pressionar = e => {
                e.preventDefault();
                if (estadoJogo === "INICIO") iniciarPartida();
                if (estadoJogo === "GAMEOVER") reiniciarJogo();
                teclas[direcao] = true;
            };
            const soltar = e => {
                e.preventDefault();
                teclas[direcao] = false;
            };
            botao.addEventListener("pointerdown", pressionar);
            botao.addEventListener("pointerup", soltar);
            botao.addEventListener("pointercancel", soltar);
            botao.addEventListener("pointerleave", soltar);
        });

        function salvarRecorde(nome, pontuacao) {
            if (pontuacao <= 0) return;
            const nomeExibicao = nome !== "" ? nome : "Anônimo";
            ranking.push({ nome: nomeExibicao, points: pontuacao }); // Mapeia para compatibilidade
            ranking.sort((a, b) => b.points - a.points);
            ranking = ranking.slice(0, 3);
            try {
                localStorage.setItem("rankingJogoEspacial", JSON.stringify(ranking));
            } catch (e) {}
        }

        function criarObstaculo() {
            const tamanho = Math.random() * 30 + 20;
            const posX = Math.random() * (canvas.width - tamanho);
            obstaculos.push({
                x: posX,
                y: -tamanho,
                largura: tamanho,
                altura: tamanho,
                velocidade: CONFIG.velocidadeObstaculoInicial + (pontos * 0.05)
            });
        }

        function atualizar() {
            if (estadoJogo !== "JOGANDO") return;

            frames++;

            // Movimentação nas 4 direções
            if ((teclas["ArrowLeft"] || teclas["a"] || teclas["A"]) && jogador.x > 0) {
                jogador.x -= CONFIG.velocidadeJogador;
                direcaoAtual = "ESQUERDA";
            }
            if ((teclas["ArrowRight"] || teclas["d"] || teclas["D"]) && jogador.x + jogador.largura < canvas.width) {
                jogador.x += CONFIG.velocidadeJogador;
                direcaoAtual = "DIREITA";
            }
            if ((teclas["ArrowUp"] || teclas["w"] || teclas["W"]) && jogador.y > 0) {
                jogador.y -= CONFIG.velocidadeJogador;
                direcaoAtual = "CIMA";
            }
            if ((teclas["ArrowDown"] || teclas["s"] || teclas["S"]) && jogador.y + jogador.altura < canvas.height) {
                jogador.y += CONFIG.velocidadeJogador;
                direcaoAtual = "BAIXO";
            }

            // Gerar novos obstáculos
            if (frames % CONFIG.frequenciaObstaculos === 0) {
                criarObstaculo();
            }

            // Mover obstáculos e verificar colisão
            for (let i = obstaculos.length - 1; i >= 0; i--) {
                let obs = obstaculos[i];
                obs.y += obs.velocidade;

                // Passou da tela (Ponto marcado)
                if (obs.y > canvas.height) {
                    obstaculos.splice(i, 1);
                    pontos++;
                    placarElemento.innerText = "Pontos: " + pontos;
                    continue;
                }

                // Checagem de colisão por caixa (AABB)
                if (
                    jogador.x < obs.x + obs.largura &&
                    jogador.x + jogador.largura > obs.x &&
                    jogador.y < obs.y + obs.altura &&
                    jogador.y + jogador.altura > obs.y
                ) {
                    estadoJogo = "GAMEOVER";
                    salvarRecorde(nomeJogador, pontos);
                }
            }
        }

        function desenhar() {
            // Limpa a tela com tom escuro espacial
            ctx.fillStyle = "#050510";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            if (estadoJogo === "INICIO") {
                // Fundo de espera animado de fundo
                ctx.fillStyle = "rgba(0, 255, 204, 0.1)";
                ctx.font = "bold 20px Arial";
                ctx.textAlign = "center";
                ctx.fillText("BEM-VINDO AO ESPAÇO", canvas.width / 2, canvas.height / 2 - 40);
                return;
            }

            // Desenhar jogador como um triângulo apontando para cima
            const centroX = jogador.x + jogador.largura / 2;
            const topoY = jogador.y;
            ctx.fillStyle = CONFIG.corJogador;
            ctx.beginPath();
            ctx.moveTo(centroX, topoY);
            ctx.lineTo(jogador.x + jogador.largura, jogador.y + jogador.altura);
            ctx.lineTo(jogador.x, jogador.y + jogador.altura);
            ctx.closePath();
            ctx.fill();

            // Nome do jogador acima do triângulo
            ctx.fillStyle = "#00ffcc";
            ctx.font = "bold 11px Arial";
            ctx.textAlign = "center";
            ctx.textBaseline = "bottom";
            const textoExibir = nomeJogador !== "" ? nomeJogador : "Nemo";
            ctx.fillText(textoExibir, centroX, jogador.y - 4);

            // Desenhar Obstáculos
            ctx.fillStyle = CONFIG.corObstaculo;
            for (let obs of obstaculos) {
                ctx.fillRect(obs.x, obs.y, obs.largura, obs.altura);
            }

            // Tela de Game Over
            if (estadoJogo === "GAMEOVER") {
                ctx.fillStyle = "rgba(0, 0, 0, 0.85)";
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                ctx.fillStyle = "#ff4444";
                ctx.font = "bold 30px Arial";
                ctx.textAlign = "center";
                ctx.fillText("GAME OVER", canvas.width / 2, 120);

                ctx.fillStyle = "#fff";
                ctx.font = "18px Arial";
                ctx.fillText("Você fez " + pontos + " pontos!", canvas.width / 2, 170);

                // Exibir Top 3 Recordes Internos
                ctx.fillStyle = "#00ffcc";
                ctx.font = "bold 16px Arial";
                ctx.fillText("🏆 RECORDES:", canvas.width / 2, 230);

                ctx.fillStyle = "#ddd";
                ctx.font = "15px Arial";
                if (ranking.length === 0) {
                    ctx.fillText("Nenhum recorde ainda!", canvas.width / 2, 260);
                } else {
                    ranking.forEach((pos, idx) => {
                        ctx.fillText(`${idx + 1}. ${pos.nome} - ${pos.points || pos.pontos} pts`, canvas.width / 2, 260 + (idx * 25));
                    });
                }

                // Instruções de reinício
                ctx.fillStyle = "#00ffcc";
                ctx.font = "bold 14px Arial";
                ctx.fillText("Pressione [ESPAÇO] para Jogar de Novo", canvas.width / 2, 420);
                
                ctx.fillStyle = "#aaa";
                ctx.font = "13px Arial";
                ctx.fillText("ou pressione [C] para Trocar de Nome", canvas.width / 2, 450);
            }
        }

        // Loop principal do jogo rodando a 60 FPS estáveis
        function loop() {
            atualizar();
            desenhar();
            requestAnimationFrame(loop);
        }

        // Inicia o processo gráfico em segundo plano
        loop();
    </script>
</body>
</html>
