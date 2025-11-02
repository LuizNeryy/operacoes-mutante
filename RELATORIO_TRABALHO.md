# Relatório de Teste de Mutação - Análise de Eficácia de Testes

**Disciplina:** Testes de Software  
**Trabalho:** Análise de Eficácia de Testes com Teste de Mutação  
**Nome:** Luiz Nery 
**Data:** 02 de novembro de 2025

---

## 1. Análise Inicial

### 1.1 Cobertura de Código Inicial

A análise inicial da suíte de testes revelou os seguintes percentuais de cobertura:

| Métrica | Percentual |
|---------|-----------|
| **Statements (Declarações)** | 85.41% |
| **Branches (Ramificações)** | 58.82% |
| **Functions (Funções)** | **100%** ✅ |
| **Lines (Linhas)** | 98.64% |

**Número de Testes:** 50 testes passando

### 1.2 Pontuação de Mutação Inicial

Após a primeira execução do StrykerJS, obtivemos:

| Métrica | Valor |
|---------|-------|
| **Mutation Score** | **73.71%** ⚠️ |
| **Mutantes Criados** | 213 |
| **Mutantes Mortos** | 154 |
| **Mutantes Sobreviventes** | **56** |
| **Timeouts** | 3 |

### 1.3 Discrepância entre Cobertura e Eficácia

**Observação Crítica:** Apesar de termos **100% de cobertura de funções** e **98.64% de cobertura de linhas**, a pontuação de mutação foi de apenas **73.71%**. 

**Por que isso acontece?**

Esta discrepância demonstra perfeitamente o conceito central do trabalho: **alta cobertura de código NÃO garante testes eficazes**. 

- **Cobertura de código** mede apenas se o código foi executado
- **Teste de mutação** mede se os testes realmente verificam o comportamento correto

Exemplo prático encontrado:
```javascript
// Teste original (fraco)
test('deve verificar se um número é maior que outro', () => { 
  expect(isMaiorQue(10, 5)).toBe(true); 
});

// Problema: Não testa o caso a === b
// Mutante sobrevivente: a > b → a >= b
```

---

## 2. Análise de Mutantes Críticos

### 2.1 Mutante Crítico #1: Comparadores de Limites (Boundary Conditions)

**Localização:** Funções `isMaiorQue()` e `isMenorQue()` (linhas 104-105)

**Mutações que Sobreviveram:**
```javascript
// Código original
function isMaiorQue(a, b) { return a > b; }
function isMenorQue(a, b) { return a < b; }

// Mutantes sobreviventes
function isMaiorQue(a, b) { return a >= b; }  // Sobreviveu!
function isMenorQue(a, b) { return a <= b; }  // Sobreviveu!
```

**Por que o teste original falhou em detectar?**

Os testes originais apenas verificavam casos onde `a != b`:
- `isMaiorQue(10, 5)` → retorna `true` tanto para `>` quanto para `>=`
- Faltava testar o caso crítico: `isMaiorQue(5, 5)` que deveria retornar `false`

**Screenshot do Relatório:**
![Mutante Comparadores](screenshots/mutante_comparadores.png)

---

### 2.2 Mutante Crítico #2: Função isPrimo - Casos Limites

**Localização:** Função `isPrimo()` (linhas 73-75)

**Mutações que Sobreviveram:**
```javascript
// Código original
function isPrimo(n) {
  if (n <= 1) return false;
  for (let i = 2; i < n; i++) {
    if (n % i === 0) return false;
  }
  return true;
}

// Mutantes sobreviventes
if (n < 1) return false;        // <= virou <
if (n <= 1) return true;        // false virou true  
for (let i = 2; i >= n; i++)    // < virou >=
if (n * i === 0) return false;  // % virou *
```

**Por que o teste original falhou em detectar?**

O teste original apenas verificava um caso de número primo (7):
```javascript
test('deve verificar que um número é primo', () => { 
  expect(isPrimo(7)).toBe(true); 
});
```

**Casos NÃO testados:**
- Números ≤ 1 (0, 1, -5)
- Número 2 (menor primo)
- Números pares > 2 (4, 6, 8)
- Números compostos ímpares (9, 15)

**Screenshot do Relatório:**
![Mutante isPrimo](screenshots/mutante_isprimo.png)

---

### 2.3 Mutante Crítico #3: Conversão de Temperatura

**Localização:** Funções `celsiusParaFahrenheit()` e `fahrenheitParaCelsius()` (linhas 93-94)

**Mutações que Sobreviveram:**
```javascript
// Código original
function celsiusParaFahrenheit(celsius) { return (celsius * 9/5) + 32; }

// Mutantes sobreviventes
return (celsius / 9/5) + 32;    // * virou /
return (celsius * 9 * 5) + 32;  // /5 virou *5
```

**Por que o teste original falhou em detectar?**

O teste apenas verificava um ponto: `0°C = 32°F`:
```javascript
test('deve converter Celsius para Fahrenheit', () => { 
  expect(celsiusParaFahrenheit(0)).toBe(32); 
});
```

**Problema:** Com celsius = 0, a multiplicação/divisão não faz diferença! Faltavam testes com valores não-zero que validassem a fórmula completa.

**Screenshot do Relatório:**
![Mutante Temperatura](screenshots/mutante_temperatura.png)

---

## 3. Soluções Implementadas

### 3.1 Solução para Comparadores de Limites

**Novos testes adicionados:**
```javascript
test('44b. deve retornar false quando números são iguais (isMaiorQue)', () => { 
  expect(isMaiorQue(5, 5)).toBe(false); 
});

test('44c. deve retornar false quando primeiro número é menor', () => { 
  expect(isMaiorQue(3, 8)).toBe(false); 
});

test('45b. deve retornar false quando números são iguais (isMenorQue)', () => { 
  expect(isMenorQue(5, 5)).toBe(false); 
});

test('45c. deve retornar false quando primeiro número é maior', () => { 
  expect(isMenorQue(10, 3)).toBe(false); 
});

test('46b. deve retornar false quando números são diferentes', () => { 
  expect(isEqual(7, 8)).toBe(false); 
});
```

**Por que esses testes são eficazes?**
- Testam o caso de **igualdade** (boundary condition crítica)
- Testam ambos os lados da condição (maior E menor)
- Garantem que `>` não pode ser substituído por `>=` sem falhar

**Mutantes mortos:** 6 mutantes eliminados

---

### 3.2 Solução para isPrimo

**Novos testes adicionados:**
```javascript
test('33b. deve retornar false para números menores ou iguais a 1', () => { 
  expect(isPrimo(0)).toBe(false);
  expect(isPrimo(1)).toBe(false);
  expect(isPrimo(-5)).toBe(false);
});

test('33c. deve retornar true para o número 2 (menor primo)', () => { 
  expect(isPrimo(2)).toBe(true); 
});

test('33d. deve retornar false para números pares maiores que 2', () => { 
  expect(isPrimo(4)).toBe(false);
  expect(isPrimo(6)).toBe(false);
  expect(isPrimo(8)).toBe(false);
});

test('33e. deve retornar false para números compostos ímpares', () => { 
  expect(isPrimo(9)).toBe(false);
  expect(isPrimo(15)).toBe(false);
});
```

**Por que esses testes são eficazes?**
- Cobrem **todos os casos limites**: 0, 1, negativos, 2
- Testam números **pares** (que devem falhar rápido)
- Testam números **compostos ímpares** (que exercitam o loop)
- Garantem que as condições `<=`, `<`, `%` não podem ser alteradas

**Mutantes mortos:** 8 mutantes eliminados

---

### 3.3 Solução para Conversão de Temperatura

**Novos testes adicionados:**
```javascript
test('38b. deve converter 100°C para 212°F', () => { 
  expect(celsiusParaFahrenheit(100)).toBe(212); 
});

test('38c. deve converter -40°C para -40°F', () => { 
  expect(celsiusParaFahrenheit(-40)).toBe(-40); 
});

test('38d. deve converter 37°C para 98.6°F (temperatura corporal)', () => { 
  expect(celsiusParaFahrenheit(37)).toBeCloseTo(98.6, 1); 
});

test('39b. deve converter 212°F para 100°C', () => { 
  expect(fahrenheitParaCelsius(212)).toBe(100); 
});

test('39c. deve converter -40°F para -40°C', () => { 
  expect(fahrenheitParaCelsius(-40)).toBe(-40); 
});

test('39d. deve converter 98.6°F para 37°C (temperatura corporal)', () => { 
  expect(fahrenheitParaCelsius(98.6)).toBeCloseTo(37, 1); 
});
```

**Por que esses testes são eficazes?**
- Testam **múltiplos valores** não-zero
- Incluem **casos especiais** (-40°C = -40°F, ponto de igualdade)
- Validam **valores conhecidos** (ponto de ebulição, temperatura corporal)
- Garantem que a **fórmula matemática completa** está correta

**Mutantes mortos:** 4 mutantes eliminados

---

### 3.4 Outras Melhorias Significativas

#### Testes para Exceções com Mensagens Específicas
```javascript
// Antes (fraco)
expect(() => divisao(5, 0)).toThrow();

// Depois (forte)
expect(() => divisao(5, 0)).toThrow('Divisão por zero não é permitida.');
```

#### Testes para Arrays Vazios
```javascript
test('9b. deve retornar 0 para array vazio', () => { 
  expect(mediaArray([])).toBe(0); 
});

test('11b. deve lançar erro para array vazio (maximoArray)', () => {
  expect(() => maximoArray([])).toThrow('Array vazio не possui valor máximo.');
});
```

#### Testes para Função clamp (Limites)
```javascript
test('36b. deve retornar mínimo quando valor é menor que min', () => { 
  expect(clamp(-5, 0, 10)).toBe(0); 
});

test('36e. deve retornar max quando valor é igual a max', () => { 
  expect(clamp(10, 0, 10)).toBe(10); 
});
```

---

## 4. Resultados Finais

### 4.1 Evolução da Pontuação de Mutação

| Fase | Mutation Score | Mutantes Mortos | Mutantes Sobreviventes | Número de Testes |
|------|----------------|-----------------|------------------------|------------------|
| **Inicial** | 73.71% | 154 | 56 | 50 |
| **1ª Melhoria** | 94.84% | 199 | 11 | 92 |
| **Final** | **96.71%** ✅ | **203** | **7** | **99** |

### 4.2 Melhoria Alcançada

- **Aumento da Pontuação:** +23.00 pontos percentuais
- **Redução de Mutantes Sobreviventes:** 87.5% (de 56 para 7)
- **Aumento de Testes:** +98% (de 50 para 99 testes)

### 4.3 Cobertura Final

| Métrica | Percentual |
|---------|-----------|
| **Statements** | 85.41% |
| **Branches** | 58.82% |
| **Functions** | 100% |
| **Lines** | 98.64% |

**Observação:** A cobertura permaneceu praticamente a mesma, mas a **eficácia** dos testes aumentou dramaticamente!

### 4.4 Mutantes Remanescentes

Os 7 mutantes sobreviventes finais são extremamente específicos:

1. **Fatorial - Condições booleanas internas** (4 mutantes na linha 19)
   - Mutações em expressões `||` e subcondições do `if (n === 0 || n === 1)`
   - Difíceis de eliminar sem refatorar a lógica da função
   
2. **produtoArray - Array vazio** (1 mutante na linha 84)
   - `if (numeros.length === 0)` → `if (false)`
   
3. **clamp - Operadores de comparação limítrofes** (2 mutantes nas linhas 88-89)
   - `<` vs `<=` e `>` vs `>=` quando valor está exatamente no limite

Estes mutantes representam menos de **3.3%** do total e são casos extremos de otimização.

---

## 5. Conclusão

### 5.1 Principais Aprendizados

Este trabalho demonstrou de forma prática e incontestável que:

1. **Cobertura de código ≠ Qualidade de testes**
   - Atingimos 98.64% de cobertura de linhas, mas apenas 73.71% de eficácia inicial
   - Muitos testes apenas "executavam" o código sem validar corretamente o comportamento

2. **Teste de mutação revela fraquezas ocultas**
   - 56 mutantes sobreviventes iniciais apontaram exatamente onde os testes eram fracos
   - Casos de borda (boundary conditions) eram sistematicamente ignorados

3. **Testes eficazes requerem pensamento crítico**
   - Não basta testar "um caso que funciona"
   - É necessário testar: valores limites, casos extremos, comportamentos negativos

4. **Asserções específicas são fundamentais**
   - Testar apenas `toThrow()` é insuficiente
   - É preciso validar a mensagem específica: `toThrow('mensagem exata')`

### 5.2 Impacto Prático

A melhoria de **73.71% → 96.71%** na pontuação de mutação significa que:

- **Antes:** Apenas 3 em cada 4 bugs introduzidos seriam detectados
- **Depois:** Praticamente todos os bugs (97 em cada 100) seriam detectados

Esta é a diferença entre uma suíte de testes **aparentemente boa** e uma suíte **realmente confiável**.

### 5.3 Reflexão Final

O teste de mutação é uma ferramenta **indispensável** para avaliar a qualidade real de uma suíte de testes. Enquanto métricas tradicionais de cobertura podem dar uma falsa sensação de segurança, o teste de mutação:

- **Desafia ativamente** a eficácia dos testes
- **Identifica precisamente** onde melhorias são necessárias
- **Garante** que os testes realmente protegem contra regressões

Para projetos críticos onde bugs podem ter alto custo, investir em teste de mutação não é opcional - **é essencial**.

---

## 6. Anexos

### 6.1 Comandos Executados

```bash
# Instalação das dependências
npm install
npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner

# Análise inicial
npm test
npm run coverage

# Teste de mutação
npx stryker run

# Após melhorias
npm test
npx stryker run
```

### 6.2 Arquivos Modificados

- `test/operacoes.test.js` - Suíte de testes aprimorada (50 → 99 testes)

### 6.3 Relatórios Gerados

- Relatório HTML do Stryker: `reports/mutation/mutation.html`
- Cobertura inicial: 98.64% de linhas, 73.71% de mutation score
- Cobertura final: 98.64% de linhas, **96.71% de mutation score**

---

**Repositório GitHub:** https://github.com/LuizNeryy/operacoes-mutante

**Data de Conclusão:** 02 de novembro de 2025
