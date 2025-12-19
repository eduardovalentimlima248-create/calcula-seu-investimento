<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Investimentos Pro</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Poppins', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            color: #fff;
        }
        .container {
            max-width: 900px;
            margin: auto;
            background: #fff;
            color: #333;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }
        h1 {
            text-align: center;
            color: #667eea;
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        h2 {
            color: #764ba2;
            margin-top: 20px;
            font-size: 1.5em;
        }
        label {
            display: block;
            margin: 15px 0 5px;
            font-weight: bold;
            color: #667eea;
        }
        input, select {
            width: 100%;
            padding: 12px;
            margin-bottom: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            background: #f8f9fa;
        }
        button {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 14px 20px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            width: 100%;
            margin-bottom: 10px;
            font-weight: bold;
            transition: 0.3s;
        }
        button:hover {
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        .result {
            margin-top: 20px;
            padding: 20px;
            background: #e8f4f8;
            border-radius: 10px;
            font-size: 18px;
            font-weight: bold;
            color: #2c3e50;
        }
        .chart-container {
            margin-top: 20px;
            height: 300px;
            background: #f9f9f9;
            border-radius: 10px;
            padding: 10px;
        }
        table {
            width: 100%;
            margin-top: 20px;
            border-collapse: collapse;
            background: #f9f9f9;
            border-radius: 10px;
            overflow: hidden;
        }
        table, th, td {
            border: 1px solid #ddd;
        }
        th, td {
            padding: 12px;
            text-align: left;
        }
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .error {
            color: #e74c3c;
            font-weight: bold;
            margin-top: 10px;
        }
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
    </style>
</head>
<body>
    <div class="container">
        <h1>Calculadora de Investimentos Pro</h1>
        <h2>Insira seus dados e acompanhe seu crescimento</h2>
        <label for="valor">Valor Inicial (R$):</label>
        <input type="number" id="valor" placeholder="Ex: 1000">
        <label for="aporte">Aporte Mensal (R$):</label>
        <input type="number" id="aporte" placeholder="Ex: 100">
        <label for="taxa">Taxa de Juros (%):</label>
        <input type="number" id="taxa" placeholder="Ex: 1">
        <label for="periodo">Período:</label>
        <select id="periodo">
            <option value="mensal">Mensal</option>
            <option value="anual">Anual</option>
        </select>
        <label for="meses">Número de Meses:</label>
        <input type="number" id="meses" placeholder="Ex: 12">
        <label for="tipo">Tipo de Juros:</label>
        <select id="tipo">
            <option value="composto">Composto</option>
            <option value="simples">Simples</option>
        </select>
        <button onclick="calcular()">Calcular</button>
        <button onclick="limpar()">Limpar</button>
        <button onclick="exportar()">Exportar CSV</button>
        <div class="result" id="resultado"></div>
        <div class="chart-container">
            <canvas id="grafico"></canvas>
        </div>
        <table id="tabela"></table>
        <div id="erro" class="error"></div>
    </div>

    <script>
        let grafico = null;
        let dados = [];

        function calcular() {
            const valor = parseFloat(document.getElementById('valor').value);
            const aporte = parseFloat(document.getElementById('aporte').value) || 0;
            let taxa = parseFloat(document.getElementById('taxa').value);
            const periodo = document.getElementById('periodo').value;
            const meses = parseInt(document.getElementById('meses').value);
            const tipo = document.getElementById('tipo').value;
            const resultado = document.getElementById('resultado');
            const tabela = document.getElementById('tabela');
            const erro = document.getElementById('erro');

            erro.innerHTML = '';
            if (isNaN(valor) || isNaN(taxa) || isNaN(meses) || meses <= 0 || valor < 0 || taxa < 0) {
                erro.innerHTML = 'Por favor, preencha todos os campos corretamente.';
                tabela.innerHTML = '';
                if (grafico) grafico.destroy();
                return;
            }

            if (periodo === 'anual') taxa = (Math.pow(1 + taxa / 100, 1/12) - 1) * 100;

            let montante = valor;
            let valores = [];
            let tabelaHtml = '<tr><th>Mês</th><th>Montante (R$)</th><th>Crescimento (%)</th></tr>';
            let anterior = valor;

            for (let i = 1; i <= meses; i++) {
                if (tipo === 'composto') {
                    montante = (montante + aporte) * (1 + taxa / 100);
                } else {
                    montante += aporte + (montante * taxa / 100);
                }
                valores.push(montante);
                const crescimento = ((montante - anterior) / anterior * 100).toFixed(2);
                tabelaHtml += `<tr><td>${i}</td><td>R$ ${montante.toFixed(2)}</td><td>${crescimento}%</td></tr>`;
                anterior = montante;
            }

            resultado.innerHTML = `Valor final após ${meses} meses: R$ ${montante.toFixed(2)}`;
            tabela.innerHTML = tabelaHtml;
            dados = valores;

            if (grafico) grafico.destroy();
            const ctx = document.getElementById('grafico').getContext('2d');
            grafico = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: Array.from({ length: meses }, (_, i) => `Mês ${i+1}`),
                    datasets: [{
                        label: 'Montante (R$)',
                        data: valores,
                        borderColor: '#667eea',
                        backgroundColor: 'rgba(102, 126, 234, 0.1)',
                        fill: true
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        }
                    }
                }
            });
        }

        function limpar() {
            document.getElementById('valor').value = '';
            document.getElementById('aporte').value = '';
            document.getElementById('taxa').value = '';
            document.getElementById('meses').value = '';
            document.getElementById('tabela').innerHTML = '';
            document.getElementById('resultado').innerHTML = '';
            document.getElementById('erro').innerHTML = '';
            if (grafico) grafico.destroy();
        }

        function exportar() {
            const csv = dados.map((v, i) => `${i+1},${v.toFixed(2)}`).join('\n');
            const blob = new Blob(['Mês,Valor\n' + csv], { type: 'text/csv' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'investimentos.csv';
            a.click();
        }
    </script>
</body>
</html>
