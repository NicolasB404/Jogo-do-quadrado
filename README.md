<!DOCTYPE html>
<html>
<head>
  <title>Jogo do quadrado</title>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.1.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.1.0/firebase-database.js"></script>
  <script>
    const firebaseConfig = {
      apiKey: "SUA_API_KEY",
      authDomain: "SEU_DOMINIO.firebaseapp.com",
      databaseURL: "https://SEU_DOMINIO-default-rtdb.firebaseio.com",
      projectId: "SEU_PROJETO_ID",
      storageBucket: "SEU_BUCKET.appspot.com",
      messagingSenderId: "SEU_ID",
      appId: "SUA_APP_ID"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
  </script>

  <style>
    body { text-align: center; font-family: sans-serif; background-color: #1a1a1a; color: white; }
    #localjogo { width: 400px; height: 400px; border: 2px solid white; margin: auto; position: relative; background: #3d3d3d; }
    .square { width: 50px; height: 50px; position: absolute; cursor: pointer; }
    #quadrado { background: red; }
    #quadradofake { background: darkred; }
    #quadradogol { background: gold; display: none; }
    #ranking { margin-top: 20px; }
  </style>
</head>
<body>

  <h1>Jogo do quadrado</h1>
  <p>Clique no quadrado para obter pontos!</p>
  <p>Recorde: <span id="recorde">0</span></p>
  <div id="localjogo">
    <div id="quadrado" class="square"></div>
    <div id="quadradofake" class="square"></div>
    <div id="quadradogol" class="square"></div>
  </div>

  <button onclick="finalizar()">Finalizar partida</button>
  <div id="ranking">
    <h3>Ranking</h3>
    <ul id="listaRanking"></ul>
  </div>

  <script>
    let score = 0;
    const recordeEl = document.getElementById("recorde");
    const localjogo = document.getElementById("localjogo");
    const quadrado = document.getElementById("quadrado");
    const quadradofake = document.getElementById("quadradofake");
    const quadradogol = document.getElementById("quadradogol");

    function moverQuadrados() {
      const maxX = localjogo.clientWidth - 50;
      const maxY = localjogo.clientHeight - 50;
      [quadrado, quadradofake, quadradogol].forEach(q => {
        q.style.left = Math.random() * maxX + "px";
        q.style.top = Math.random() * maxY + "px";
      });

      quadradogol.style.display = Math.random() < 0.2 ? "block" : "none";
    }

    quadrado.onclick = () => {
      score++;
      recordeEl.textContent = score;
      moverQuadrados();
    };
    quadradofake.onclick = () => {
      score--;
      recordeEl.textContent = score;
      moverQuadrados();
    };
    quadradogol.onclick = () => {
      score += 5;
      recordeEl.textContent = score;
      quadradogol.style.display = "none";
      moverQuadrados();
    };

    function finalizar() {
      const nome = prompt("Qual seu nome?");
      if (!nome) return;
      salvarPontuacao(nome, score);
      mostrarRanking();
      score = 0;
      recordeEl.textContent = 0;
    }

    function salvarPontuacao(nome, pontos) {
      db.ref("ranking").push({ nome, pontos });
    }

    function mostrarRanking() {
      db.ref("ranking").orderByChild("pontos").limitToLast(10).once("value", snapshot => {
        const lista = document.getElementById("listaRanking");
        lista.innerHTML = "";
        const dados = Object.values(snapshot.val() || {}).sort((a, b) => b.pontos - a.pontos);
        dados.forEach(j => {
          const item = document.createElement("li");
          item.textContent = `${j.nome}: ${j.pontos} pontos`;
          lista.appendChild(item);
        });
      });
    }

    moverQuadrados();
  </script>
</body>
</html>
