<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Conversor de Ganho</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      padding: 40px;
      background: #1e1e2f;
      color: #f0f0f0;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    h2 {
      color: #ffffff;
      margin-bottom: 10px;
    }
    p {
      margin-bottom: 20px;
    }
    textarea, pre {
      width: 100%;
      max-width: 500px;
      padding: 10px;
      margin-top: 10px;
      font-family: monospace;
      font-size: 14px;
      background: #2c2c3e;
      color: #ffffff;
      border: 1px solid #444;
      border-radius: 8px;
    }
    textarea {
      height: 150px;
    }
    button {
      padding: 12px 25px;
      margin-top: 20px;
      font-size: 16px;
      background-color: #4f46e5;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s ease;
    }
    button:hover {
      background-color: #4338ca;
    }
    .button-group {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      justify-content: center;
    }
    .tabela-nomes {
      margin-top: 20px;
      width: 200px;
      font-size: 12px;
      background: #2c2c3e;
      border-radius: 8px;
      padding: 10px;
    }
    table {
      margin-top: 20px;
      border-collapse: collapse;
      width: 100%;
      max-width: 600px;
      background: #2c2c3e;
      border-radius: 8px;
      overflow: hidden;
    }
    th, td {
      border: 1px solid #444;
      padding: 10px;
      text-align: left;
    }
    th {
      background-color: #3f3f5e;
      color: #fff;
    }
    td {
      color: #ccc;
    }
    .popup {
      display: none;
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: #2c2c3e;
      padding: 20px;
      border-radius: 8px;
      width: 400px;
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
      color: #fff;
      text-align: center;
    }
    .popup button {
      padding: 10px 15px;
      background-color: #4f46e5;
      border: none;
      border-radius: 8px;
      color: white;
      font-size: 14px;
      cursor: pointer;
    }
    .popup button:hover {
      background-color: #4338ca;
    }
    .popup h3 {
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <h2>Conversor de Ganho (Pro Tools)</h2>
  <p>Insira os valores lidos do Pro Tools (ex: <code>Kick -7</code>) e clique em "Calcular Correções". Veja abaixo os nomes padrão aceitos:</p>

  <textarea id="input" placeholder="Exemplo:\nKick -7\nVox -6\nEG -25"></textarea>
  <div class="button-group">
    <button onclick="calcular()">Calcular Correções</button>
  </div>

  <div class="tabela-nomes">
    <h3>Exemplos de Nomes</h3>
    <ul>
      <li>Kick</li>
      <li>Snare Top</li>
      <li>Snare Bot</li>
      <li>Tom 1</li>
      <li>Tom 2</li>
      <li>Tom 3</li>
      <li>Over</li>
      <li>Drum pad</li>
      <li>Bass</li>
      <li>Acoustic</li>
      <li>Vox (Vocals)</li>
      <li>Eletric Guitars (EG)</li>
      <li>Keys</li>
      <li>Piano</li>
      <li>Tracks</li>
    </ul>
  </div>

  <!-- Popup de Resultado -->
  <div id="popup" class="popup">
    <h3>Resultado:</h3>
    <pre id="output"></pre>
    <button onclick="copiarResultado()">Copiar Resultado</button>
  </div>

  <script>
    const padrao = {
      "Kick": -4,
      "Snare Top": -4,
      "Snare Bot": -4,
      "Tom 1": -4,
      "Tom 2": -4,
      "Tom 3": -4,
      "Over": -6,
      "Drum pad": -10,
      "Bass": -6,
      "Acoustic": -12,
      "Vox (Vocals)": -12,
      "Eletric Guitars (EG)": -12,
      "Keys": -10,
      "Piano": -10,
      "Tracks": -10
    };

    const apelidos = {
      "kick": "Kick", "kk": "Kick", "bumbo": "Kick",
      "snare top": "Snare Top", "sn top": "Snare Top", "caixa top": "Snare Top", "caixa": "Snare Top",
      "snare bot": "Snare Bot", "sn bot": "Snare Bot", "caixa bot": "Snare Bot",
      "tom1": "Tom 1", "tom 1": "Tom 1", "tom 01": "Tom 1",
      "tom2": "Tom 2", "tom 2": "Tom 2", "tom 02": "Tom 2",
      "tom3": "Tom 3", "tom 3": "Tom 3", "tom 03": "Tom 3",
      "over": "Over", "overhead": "Over", "oh": "Over",
      "drum pad": "Drum pad", "pad": "Drum pad", "percussão eletronica": "Drum pad",
      "bass": "Bass", "baixo": "Bass",
      "acoustic": "Acoustic", "violão": "Acoustic", "ac": "Acoustic",
      "vox": "Vox (Vocals)", "vocal": "Vox (Vocals)", "vocals": "Vox (Vocals)", "voz": "Vox (Vocals)",
      "vozes": "Vox (Vocals)", "lead": "Vox (Vocals)", "lead vocal": "Vox (Vocals)",
      "eg": "Eletric Guitars (EG)", "guitarra": "Eletric Guitars (EG)", "gtr": "Eletric Guitars (EG)",
      "eletric guitar": "Eletric Guitars (EG)", "guit elétrica": "Eletric Guitars (EG)",
      "keys": "Keys", "teclado": "Keys", "synth": "Keys", "sinte": "Keys",
      "piano": "Piano", "órgão": "Piano", "organ": "Piano",
      "tracks": "Tracks", "trilha": "Tracks", "playback": "Tracks", "pb": "Tracks"
    };

    function calcular() {
      const input = document.getElementById("input").value;
      const linhas = input.split("\n");
      const resultado = [];

      linhas.forEach(linha => {
        const match = linha.match(/(.+?)\s*(-?\d+\.?\d*)/);
        if (match) {
          let nomeOriginal = match[1].trim().toLowerCase();
          let valor = parseFloat(match[2]);

          const nomePadrao = apelidos[nomeOriginal] || apelidos[nomeOriginal.replace(/\s+/g, '')];
          const nome = nomePadrao || match[1].trim();

          const ref = padrao[nome];
          if (ref !== undefined) {
            const correcao = +(ref - valor).toFixed(1);
            const sinal = correcao > 0 ? "+" : "";
            resultado.push(`${nome} ${sinal}${correcao}dB`);
          } else {
            resultado.push(`${match[1].trim()}: Não encontrado no padrão.`);
          }
        }
      });

      // Exibe o pop-up com o resultado
      document.getElementById("output").textContent = resultado.join("\n");
      document.getElementById("popup").style.display = "block";
    }

    function copiarResultado() {
      const output = document.getElementById("output").textContent;
      navigator.clipboard.writeText(output).then(() => {
        alert("Resultado copiado para a área de transferência!");
        document.getElementById("popup").style.display = "none"; // Fecha o pop-up
      }, err => {
        alert("Erro ao copiar: " + err);
      });
    }
  </script>
</body>
