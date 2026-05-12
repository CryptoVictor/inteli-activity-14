# Documentação Técnica: Componente `timeline.html`

Este documento descreve a estrutura, funcionamento e orientações de integração do componente de visualização de simulações passadas desenvolvido para o projeto de Otimização de Limites de Crédito — Banco PAN.

---

## 1. Visão Geral

O componente `timeline.html` é uma interface visual interativa que substitui listas estáticas de resultados por uma **timeline horizontal navegável**. Seu objetivo é facilitar a visualização e comparação de simulações passadas do modelo de otimização, oferecendo ao usuário uma experiência intuitiva e esteticamente alinhada à identidade do Banco PAN.

### 1.1. Propósito

| Aspecto | Descrição |
| :--- | :--- |
| **Onde é usado** | Seção "Todos os resultados" da interface principal |
| **O que exibe** | Cards de simulações passadas com métricas-chave |
| **Tecnologia** | p5.js (renderização em canvas) |
| **Interatividade** | Arrastar horizontalmente para navegar; hover eleva o card |

### 1.2. Estrutura Visual

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Card 1  │     │  Card 2  │     │  Card 3  │     │  Card 4  │
│          │     │          │     │          │     │          │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
─────●────────────────●────────────────●────────────────●──────  ← linha do tempo
```

---

## 2. Estrutura dos Dados

Cada entrada na timeline é representada por um objeto no array `simulacoes`, localizado no início do script.

### 2.1. Formato de um registro

```js
{
  titulo:   "Simulação #4",  // nome exibido no topo do card
  data:     "10/05/2026",   // data da simulação (formato DD/MM/AAAA)
  retorno:  "+12,4%",       // métrica principal (esquerda do card)
  risco:    "Baixo",        // métrica secundária (direita do card)
  positivo: true            // true = métrica em verde; false = vermelho
}
```

### 2.2. Array completo (exemplo atual)

```js
const simulacoes = [
  { titulo: "Simulação #4", data: "10/05/2026", retorno: "+12,4%", risco: "Baixo",  positivo: true  },
  { titulo: "Simulação #3", data: "05/04/2026", retorno: "+8,1%",  risco: "Médio",  positivo: true  },
  { titulo: "Simulação #2", data: "03/03/2026", retorno: "+5,7%",  risco: "Alto",   positivo: true  },
  { titulo: "Simulação #1", data: "07/01/2026", retorno: "-2,3%",  risco: "Baixo",  positivo: false },
];
```

> **Para adicionar uma nova simulação:** basta inserir um novo objeto no início do array. O canvas se redimensiona automaticamente e o scroll passa a funcionar quando o conteúdo ultrapassa a largura disponível.

---

## 3. Configuração Visual

As constantes abaixo controlam o layout do componente e podem ser ajustadas conforme necessidade.

| Constante | Valor padrão | Descrição |
| :--- | :---: | :--- |
| `CARD_W` | `160` | Largura de cada card em pixels |
| `CARD_H` | `108` | Altura de cada card em pixels |
| `GAP` | `48` | Espaço entre os cards em pixels |
| `MARGEM` | `32` | Recuo nas bordas esquerda e direita |
| `LINHA_Y` | `CARD_H + 36` | Posição vertical da linha do tempo |

### 3.1. Paleta de cores

| Constante | Valor | Uso |
| :--- | :--- | :--- |
| `AZUL` | `[0, 156, 222]` | Barra superior do card e dots da linha |
| `VERDE` | `[34, 197, 94]` | Métrica positiva |
| `VERMELHO` | `[239, 68, 68]` | Métrica negativa |
| `CINZA_BG` | `[247, 248, 250]` | Fundo do canvas |
| `CINZA_LINE` | `[208, 213, 221]` | Linha do tempo e divisores internos |

---

## 4. Interatividade

### 4.1. Navegação por arrasto

O usuário arrasta horizontalmente para navegar entre os cards. O deslocamento é **clamped** — ao chegar no primeiro ou no último card, o movimento para automaticamente.

```
offset mínimo → 0 (primeiro card visível à esquerda)
offset máximo → max(0, totalConteudo + MARGEM×2 - largura do canvas)
```

### 4.2. Efeito de hover

Ao posicionar o cursor sobre um card, ele se eleva suavemente (`elevacoes[i]` interpola via `lerp` até 8px) e a sombra aumenta de intensidade. O efeito reverte ao mover o cursor para fora.

---

## 5. Integração na Interface Principal

O componente é autossuficiente e pode ser incorporado na seção "Todos os resultados" de duas formas:

### 5.1. Via `<iframe>` (integração sem dependências)

```html
<iframe
  src="timeline.html"
  style="width: 100%; height: 180px; border: none;"
></iframe>
```

### 5.2. Via extração do script (integração direta)

Copiar o bloco `<script>` do `timeline.html` para o projeto principal e garantir que a dependência p5.js esteja carregada:

```html
<script src="https://cdn.jsdelivr.net/npm/p5@1.9.0/lib/p5.min.js"></script>
```

> O canvas é criado automaticamente pelo p5.js e inserido no `<body>`. Para controlá-lo em um container específico, utilize o modo de instância do p5.js (`new p5(sketch, containerElement)`).

---

## 6. Limitações e Próximos Passos

| Limitação atual | Melhoria sugerida |
| :--- | :--- |
| Dados estáticos no código | Consumir via API ou JSON externo |
| Rótulos fixos ("RETORNO", "RISCO") | Parametrizar por campo no objeto `simulacoes` |
| Canvas com largura fixa (`min(..., 780)`) | Usar `windowWidth` para responsividade total |
| Sem clique para detalhamento | Implementar modal ou redirecionamento ao clicar no card |
