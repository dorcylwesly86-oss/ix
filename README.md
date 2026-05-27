<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Página de Matemáticas</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f7f9fc;
      color: #20232a;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .container {
      max-width: 640px;
      width: 100%;
      margin: 24px;
      background: #ffffff;
      border-radius: 16px;
      box-shadow: 0 18px 40px rgba(0, 0, 0, 0.08);
      padding: 32px;
    }
    h1 {
      margin-top: 0;
      color: #1d3557;
    }
    label,
    p {
      margin: 0 0 16px;
      line-height: 1.6;
    }
    input {
      width: 100%;
      padding: 14px 16px;
      border: 1px solid #d3d9e0;
      border-radius: 10px;
      font-size: 1rem;
      box-sizing: border-box;
    }
    button {
      margin-top: 16px;
      padding: 14px 22px;
      background: #2a9d8f;
      color: white;
      border: none;
      border-radius: 10px;
      font-size: 1rem;
      cursor: pointer;
    }
    button:hover {
      background: #21867b;
    }
    .output {
      margin-top: 24px;
      padding: 20px;
      background: #eef6f4;
      border-radius: 12px;
      border: 1px solid #d7e5e2;
    }
    .output h2 {
      margin-top: 0;
      font-size: 1.2rem;
    }
    pre {
      white-space: pre-wrap;
      word-break: break-word;
      margin: 0;
      font-family: 'Courier New', Courier, monospace;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Página de Matemáticas</h1>
    <p>Escribe una operación matemática y presiona <strong>Calcular</strong>. La página te mostrará el resultado y te dirá cómo se hace.</p>
    <label for="expression">Operación:</label>
    <input id="expression" type="text" placeholder="Ejemplo: 12 + 8" />
    <button id="calculate">Calcular</button>

    <div class="output" id="output" hidden>
      <h2>Resultado</h2>
      <pre id="result"></pre>
      <h2>Cómo se hace</h2>
      <pre id="steps"></pre>
    </div>
  </div>

  <script>
    const expressionInput = document.getElementById('expression');
    const calculateButton = document.getElementById('calculate');
    const outputBox = document.getElementById('output');
    const resultBox = document.getElementById('result');
    const stepsBox = document.getElementById('steps');

    function safeExpression(expr) {
      const cleaned = expr.replace(/ /g, '');
      if (!/^[0-9()+\-*/.]+$/.test(cleaned)) {
        throw new Error('Solo se permiten números, espacios y los operadores + - * / ( ).');
      }
      if (/\*\*|\/\/|\+\+|\-\-|\+\-|\-\+/.test(cleaned)) {
        return cleaned.replace(/\+\+/g, '+').replace(/\-\-/g, '+').replace(/\+\-/g, '-').replace(/\-\+/g, '-');
      }
      return cleaned;
    }

    function computeBinaryStep(a, op, b) {
      const n1 = Number(a);
      const n2 = Number(b);
      if (isNaN(n1) || isNaN(n2)) {
        throw new Error('No se pudo leer los números de la operación.');
      }
      switch (op) {
        case '+':
          return {
            value: n1 + n2,
            steps: `Suma: ${n1} + ${n2} = ${n1 + n2}`
          };
        case '-':
          return {
            value: n1 - n2,
            steps: `Resta: ${n1} - ${n2} = ${n1 - n2}`
          };
        case '*':
          return {
            value: n1 * n2,
            steps: `Multiplicación: ${n1} × ${n2} = ${n1 * n2}`
          };
        case '/':
          if (n2 === 0) {
            throw new Error('No se puede dividir entre cero.');
          }
          return {
            value: n1 / n2,
            steps: `División: ${n1} ÷ ${n2} = ${n1 / n2}`
          };
        default:
          throw new Error('Operador no soportado.');
      }
    }

    function explainExpression(expr) {
      const cleaned = safeExpression(expr);
      const simpleBinary = cleaned.match(/^([0-9.]+)([+\-*/])([0-9.]+)$/);
      if (simpleBinary) {
        return computeBinaryStep(simpleBinary[1], simpleBinary[2], simpleBinary[3]);
      }

      const nested = [];
      let current = cleaned;
      const stepLines = [];

      while (true) {
        const parenMatch = current.match(/\([^()]+\)/);
        if (!parenMatch) break;
        const inner = parenMatch[0].slice(1, -1);
        const innerResult = explainExpression(inner);
        current = current.replace(parenMatch[0], String(innerResult.value));
        stepLines.push(`Evalúa ${parenMatch[0]} = ${innerResult.value}`);
      }

      let value;
      try {
        value = Function(`return ${current}`)();
      } catch (err) {
        throw new Error('Operación inválida o demasiado compleja. Usa expresiones simples.');
      }
      if (typeof value !== 'number' || !isFinite(value)) {
        throw new Error('El resultado no es un número válido.');
      }

      if (stepLines.length === 0) {
        stepLines.push(`Se evalúa la expresión: ${current}`);
      }
      stepLines.push(`Resultado final: ${value}`);
      return {
        value,
        steps: stepLines.join('\n')
      };
    }

    calculateButton.addEventListener('click', () => {
      const expr = expressionInput.value.trim();
      if (!expr) {
        alert('Escribe una operación primero.');
        return;
      }
      try {
        const answer = explainExpression(expr);
        resultBox.textContent = answer.value;
        stepsBox.textContent = answer.steps;
        outputBox.hidden = false;
      } catch (error) {
        resultBox.textContent = 'Error';
        stepsBox.textContent = error.message;
        outputBox.hidden = false;
      }
    });

    expressionInput.addEventListener('keypress', event => {
      if (event.key === 'Enter') {
        calculateButton.click();
      }
    });
  </script>
</body>
</html>
