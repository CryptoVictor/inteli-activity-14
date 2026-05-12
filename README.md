# Timeline de Simulações — Banco PAN

Este documento descreve o desenvolvimento do componente visual de histórico de simulações, criado como parte da interface da solução de Otimização de Limites de Crédito do Banco PAN / BTG Pactual.

---

## 1. Introdução à Proposta

O projeto de Otimização de Limites de Crédito do Banco PAN propõe um modelo de **Programação Linear** que maximiza a receita de intercâmbio da carteira de crédito sob restrições de inadimplência e capacidade de pagamento. A solução é desenvolvida em Python e se integra ao motor de decisão do banco para definir limites pré-aprovados de forma individualizada por cluster de cliente.

### 1.1. Contexto do Componente

Dentro da interface de uso do modelo, os analistas precisam visualizar e comparar simulações já executadas. A abordagem original — uma lista estática de resultados — não oferecia contraste visual nem facilitava a comparação rápida entre cenários.

O componente `timeline.html` foi desenvolvido para substituir essa lista por uma **timeline horizontal interativa**, posicionada na seção "Todos os resultados" da interface principal.

| Aspecto | Descrição |
| :--- | :--- |
| **Onde é exibido** | Seção "Todos os resultados" da interface do modelo |
| **Objetivo** | Facilitar a comparação visual de simulações passadas |
| **Tecnologia** | p5.js — renderização via canvas HTML5 |
| **Público** | Times de Data Science, Estratégia e Risco do Banco PAN |

### 1.2. Estrutura Visual

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Card 1  │     │  Card 2  │     │  Card 3  │     │  Card 4  │
│ título   │     │ título   │     │ título   │     │ título   │
│ data     │     │ data     │     │ data     │     │ data     │
│──────────│     │──────────│     │──────────│     │──────────│
│ retorno  │     │ retorno  │     │ retorno  │     │ retorno  │
│ risco    │     │ risco    │     │ risco    │     │ risco    │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
─────●────────────────●────────────────●────────────────●──────
```

---

## 2. Rascunho Inicial

O desenvolvimento seguiu uma progressão iterativa, partindo de formas geométricas simples até o componente final.

### 2.1. Evolução do Componente

| Versão | Descrição | Estado |
| :---: | :--- | :---: |
| v0 | Quadrado único estático no canvas | Descartado |
| v1 | Múltiplos quadrados coloridos com data em texto | Base |
| v2 | Carrossel automático com loop infinito | Substituído |
| v3 | Navegação manual por arrasto, sem loop | Atual |
| v4 | Cards com gráfico de barras interno | Explorado |
| **v5** | **Cards profissionais com métricas, sombra e hover** | **Vigente** |

### 2.2. Decisões de Projeto

* **p5.js em vez de HTML/CSS puro:** permite animações suaves (elevação por hover, lerp) sem dependências de framework, facilitando a futura integração via `<iframe>` ou modo de instância.

* **Canvas dimensionado pelo conteúdo:** a largura do canvas é calculada a partir do número de simulações, garantindo que nenhuma informação fique truncada e que o scroll só apareça quando necessário.

* **Arrasto com clamping:** o deslocamento (`offset`) é restrito entre `0` e `maxOffset()`, impedindo que o usuário navegue além do primeiro ou do último card.

* **Elevação por hover em vez de escala:** o efeito de levantar o card (`elevacoes[i]` via `lerp`) é mais sutil e alinhado ao padrão visual corporativo do que o crescimento proporcional testado nas versões anteriores.

---

## 3. Registro do Resultado Obtido

### 3.1. Componente Final

O componente entregue exibe cards individuais por simulação, com as seguintes informações:

| Campo | Localização no card | Tipo |
| :--- | :--- | :--- |
| `titulo` | Topo do card, negrito | Texto |
| `data` | Abaixo do título, cinza | Texto (DD/MM/AAAA) |
| `retorno` | Métrica principal, colorida | Numérico (%) |
| `risco` | Métrica secundária | Texto (Baixo / Médio / Alto) |
| `positivo` | Controla cor do retorno | Booleano |

### 3.2. Comportamentos Implementados

* **Hover:** card eleva 8px suavemente com aumento de sombra
* **Arrasto:** navegação horizontal com limite nos extremos
* **Cursor:** `grab` em repouso, `grabbing` ao arrastar
* **Dot da linha:** aumenta de tamanho ao hover no card correspondente

### 3.3. Como Adicionar uma Nova Simulação

Inserir um novo objeto no array `simulacoes` em `timeline.html`:

```js
const simulacoes = [
  { titulo: "Simulação #5", data: "20/05/2026", retorno: "+14,1%", risco: "Baixo", positivo: true },
  // ... entradas anteriores
];
```

O canvas e o scroll se ajustam automaticamente.

### 3.4. Integração na Interface Principal

```html
<!-- opção mais simples: iframe -->
<iframe src="timeline.html" style="width: 100%; height: 180px; border: none;"></iframe>
```

Para integração direta ao projeto React/JS, utilizar o modo de instância do p5.js:

```js
import p5 from 'p5';
new p5(sketch, document.getElementById('timeline-container'));
```

---

## 4. Referências

- p5.js — [https://p5js.org](https://p5js.org)
- INTELI. **TAPI — Otimização combinatória e pesquisa operacional: Banco PAN**. São Paulo: Inteli, 2026.
