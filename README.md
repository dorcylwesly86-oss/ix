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
    <p>Escribe una operación matemática o una ecuación lineal y presiona <strong>Calcular</strong>. La página te mostrará el resultado y te explicará cómo se hace.</p>
    <label for="expression">Operación o ecuación:</label>
    <input id="expression" type="text" placeholder="Ejemplo: 12 + 8 o 2x + 3 = 7" />
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
      const cleaned = expr.replace(/\s+/g, '');
      if (!/^[0-9a-zA-Z()+\-*/=.,]+$/.test(cleaned)) {
        throw new Error('Solo se permiten números, letras, espacios y los operadores + - * / = ( ).');
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

    function parseLinearSide(side, variable) {
      const normalized = side.replace(/\s+/g, '').replace(/\*/g, '');
      const terms = normalized.match(/[+-]?[^+-]+/g) || [];
      let coeff = 0;
      let constant = 0;
      for (const term of terms) {
        if (!term) continue;
        const termValue = term.replace(/,/g, '.');
        const variableIndex = termValue.indexOf(variable);
        if (variableIndex >= 0) {
          let coefficient = termValue.slice(0, variableIndex);
          if (coefficient === '' || coefficient === '+') coefficient = '1';
          if (coefficient === '-') coefficient = '-1';
          const parsed = Number(coefficient);
          if (Number.isNaN(parsed)) {
            throw new Error(`Término algebraico no válido: ${term}`);
          }
          coeff += parsed;
        } else {
          const parsed = Number(termValue);
          if (Number.isNaN(parsed)) {
            throw new Error(`Constante no válida: ${term}`);
          }
          constant += parsed;
        }
      }
      return { coeff, constant };
    }

    function formatTerm(value, variable) {
      if (value === 0) return '0';
      const sign = value < 0 ? '-' : '';
      const absValue = Math.abs(value);
      const coeff = absValue === 1 ? '' : absValue;
      return `${sign}${coeff}${variable}`;
    }

    function solveLinearEquation(expr) {
      const cleaned = safeExpression(expr);
      const sides = cleaned.split('=');
      if (sides.length !== 2) {
        throw new Error('Escribe una ecuación con un solo signo =, por ejemplo 2x + 3 = 7.');
      }
      const variableMatch = cleaned.match(/[a-zA-Z]/);
      if (!variableMatch) {
        throw new Error('Escribe una ecuación con una variable, por ejemplo 2x + 3 = 7.');
      }
      const variable = variableMatch[0];
      const left = parseLinearSide(sides[0], variable);
      const right = parseLinearSide(sides[1], variable);
      const coeff = left.coeff - right.coeff;
      const constant = right.constant - left.constant;

      if (coeff === 0) {
        if (constant === 0) {
          return {
            value: 'Infinitas soluciones',
            steps: `Ecuación: ${sides[0]} = ${sides[1]}\nSe cancelan los términos con ${variable} y los constantes.\nResultado: la ecuación es verdadera para cualquier valor de ${variable}.`
          };
        }
        return {
          value: 'Sin solución',
          steps: `Ecuación: ${sides[0]} = ${sides[1]}\nSe cancelan los términos con ${variable}, pero queda una igualdad imposible.\nResultado: no hay solución.`
        };
      }

      const solution = constant / coeff;
      const leftFormatted = `${formatTerm(left.coeff, variable)} ${left.constant >= 0 ? '+' : '-'} ${Math.abs(left.constant)}`.replace('+ -', '- ');
      const rightFormatted = `${formatTerm(right.coeff, variable)} ${right.constant >= 0 ? '+' : '-'} ${Math.abs(right.constant)}`.replace('+ -', '- ');
      const steps = [
        `Ecuación: ${sides[0]} = ${sides[1]}`,
        `Simplificamos lado izquierdo: ${leftFormatted}`,
        `Simplificamos lado derecho: ${rightFormatted}`,
        `Pasamos los términos con ${variable} al lado izquierdo y las constantes al lado derecho.`,
        `Queda: ${formatTerm(coeff, variable)} = ${constant}`,
        `Dividimos entre ${coeff}: ${variable} = ${solution}`
      ];

      return {
        value: `${variable} = ${solution}`,
        steps: steps.join('\n')
      };
    }

    function explainExpression(expr) {
      const cleaned = safeExpression(expr);
      if (cleaned.includes('=')) {
        return solveLinearEquation(expr);
      }
      const simpleBinary = cleaned.match(/^([0-9.]+)([+\-*/])([0-9.]+)$/);
      if (simpleBinary) {
        return computeBinaryStep(simpleBinary[1], simpleBinary[2], simpleBinary[3]);
      }

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

      stepLines.push(`Se evalúa la expresión: ${current}`);
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
