# CLAUDE.md — Rotina com Propósito

## Visão geral do projeto

App de produtividade pessoal focado em intenção e propósito diário. É um **único arquivo HTML standalone** (sem backend, sem dependências externas além de Google Fonts), que roda direto no navegador e persiste dados via `localStorage`.

---

## Stack e arquitetura

- **Arquivo único:** `index.html` — todo CSS, HTML e JS dentro de um único arquivo
- **Zero dependências:** sem npm, sem frameworks, sem bundler
- **Persistência:** `localStorage` via objeto `state` serializado em JSON
- **Fontes:** Google Fonts — `Fraunces` (serif, display) + `DM Sans` (sans-serif, corpo)
- **Linguagem:** Português brasileiro (pt-BR) em toda a interface

---

## Design system

### Paleta de cores (CSS variables)
```css
--bg: #0d0d0f;         /* fundo principal */
--surface: #161618;    /* cards */
--surface2: #1e1e21;   /* cards secundários */
--border: #2a2a2e;     /* bordas */
--accent: #c8f060;     /* verde-lima (destaque principal) */
--accent2: #f0a060;    /* laranja (destaque secundário) */
--text: #f0ede8;       /* texto principal */
--muted: #7a7880;      /* texto secundário / labels */
--danger: #ff6b6b;     /* vermelho para ações destrutivas */
```

### Tipografia
- **Fraunces** (serif): títulos, números de destaque, frases em itálico, propósito
- **DM Sans** (sans-serif, weight 300): corpo de texto, labels, botões

### Padrões visuais
- Bordas arredondadas: `border-radius: 10–14px` em cards
- Labels em uppercase + `letter-spacing: 1.5–2px` + `font-size: 0.65–0.7rem`
- Accent com `border-left: 3px solid var(--accent)` em banners
- Animações com `cubic-bezier(0.34, 1.56, 0.64, 1)` (efeito bounce suave)
- Tema escuro em tudo — nunca usar fundo branco

---

## Estrutura de abas (views)

O app tem **4 views** controladas por tabs:

| Tab | ID da view | Função |
|-----|-----------|--------|
| Hoje | `view-hoje` | Tarefas do dia + progresso + propósito |
| Semana | `view-semana` | Planejar tarefas por dia da semana |
| Check-in | `view-checkin` | Registro matinal de humor, energia, sono, gratidão |
| Propósito | `view-proposito` | Definir e editar o propósito pessoal |

---

## Estado (localStorage)

```js
const state = {
  purpose: "",              // texto do propósito do usuário
  days: {                   // tarefas por dia { "2025-1-15": { tasks: [...] } }
    "YYYY-M-D": {
      tasks: [
        {
          name: "",         // nome da tarefa
          cat: "",          // categoria: "saude"|"trabalho"|"aprendizado"|"pessoal"|"familia"
          done: false,      // status
          reward: ""        // emoji de recompensa (opcional)
        }
      ]
    }
  },
  weekPlan: {               // planejamento semanal
    "YYYY-M-D": {
      tasks: [
        { name: "", time: "", cat: "", prio: "" }
      ]
    }
  },
  checkins: {               // check-ins matinais
    "YYYY-M-D": {
      mood: 1-5,            // 1=Mal, 5=Ótimo
      energy: 1-5,          // 1=Exausto, 5=Com tudo
      sleep: 7.5,           // horas de sono (float)
      gratitude: "",        // texto livre
      intention: "",        // palavra/frase do dia
      ts: "ISO string"
    }
  }
}
```

Função de persistência: `save()` → `localStorage.setItem('rotinaState', JSON.stringify(state))`
Função de carregamento: `load()` → `JSON.parse(localStorage.getItem('rotinaState'))`

---

## Categorias de tarefas

```js
const CATS = {
  saude:      { label: "Saúde",      color: "#c8f060" },  // accent
  trabalho:   { label: "Trabalho",   color: "#f0a060" },  // accent2
  aprendizado:{ label: "Aprendizado",color: "#a0c8ff" },  // azul
  pessoal:    { label: "Pessoal",    color: "#d0a0ff" },  // lilás
  familia:    { label: "Família",    color: "#ffb0c0" }   // rosa
}
```

---

## Funções principais

- `renderTasks()` — renderiza lista de tarefas do dia
- `toggleTask(i)` — marca/desmarca tarefa
- `addTask()` — adiciona nova tarefa ao dia de hoje
- `deleteTask(i)` — remove tarefa
- `checkReward()` — exibe badge de recompensa ao completar todas as tarefas
- `renderWeekView()` — renderiza planejamento semanal
- `applyWeekPlanToToday()` — mescla tarefas planejadas da semana para o dia atual (chamado no init)
- `renderCheckin()` / `saveCheckin()` — check-in matinal
- `renderPurpose()` / `savePurpose()` — gerencia propósito pessoal
- `getToday()` — retorna key do dia atual (`"YYYY-M-D"`)
- `getTodayTasks()` — retorna ou cria objeto do dia atual em `state.days`

---

## Regras de desenvolvimento

1. **Manter arquivo único** — não separar em múltiplos arquivos
2. **Não introduzir dependências** — sem npm, sem frameworks externos
3. **Sempre usar as CSS variables** definidas no `:root` — nunca hardcodar cores
4. **Mobile first** — o app é usado principalmente no celular
5. **Português em tudo** — labels, mensagens, placeholders, comentários no código
6. **localStorage como única persistência** — não adicionar backend sem solicitação explícita
7. **Preservar o estilo visual** — dark theme, tipografia Fraunces+DM Sans, paleta atual

---

## 🚧 Próxima implementação — Tela de abertura matinal (Morning Gateway)

**Status:** planejada, ainda não implementada. Esta é a próxima feature a ser desenvolvida.

### Objetivo
Quando o usuário abre o app de manhã sem tarefas definidas para o dia, em vez de cair na tela vazia do "Hoje", aparece uma **tela rápida de início de dia** que transforma a abertura do app num ritual. O ponto principal é **reduzir o atrito da tela em branco** — sem tarefas, o cérebro não sente urgência. Com o gateway, o usuário "começa o dia" pelo app, não só registra tarefas nele.

### Condição de exibição
Mostrar o gateway quando:
- É a primeira abertura do dia (ou não há check-in feito ainda hoje), **E**
- O dia de hoje não tem tarefas definidas (ou tem 0 tarefas)

### Fluxo da tela (em ordem)

**1. Saudação contextual**
```
"Bom dia, Lucas. Quinta-feira, 24 de abril."
"Qual é o seu foco hoje?"
```
- Nome hardcodado ou extraído de alguma preferência do usuário
- Data formatada por extenso em pt-BR

**2. Sugestões inteligentes**
- Mostra tarefas de ontem que não foram concluídas (já existe lógica no código)
- Mostra tarefas padrão configuradas (se houver)
- O usuário toca nas que quer trazer para o dia de hoje
- Visual: chips/cards tocáveis, estilo seleção toggle

**3. Campo rápido para adicionar algo novo**
- Input simples com placeholder: `+ o que mais você precisa fazer hoje?`
- Permite adicionar tarefas extras além das sugestões

**4. Botão de confirmar**
- Texto: `"Começar o dia →"`
- Ao tocar: salva as tarefas selecionadas/adicionadas no dia atual e vai para a view "Hoje" normal

### Notas de implementação
- Verificar condição de exibição dentro da função `init()` / bloco de inicialização no final do arquivo
- Pode ser implementado como um overlay/modal sobre a view "Hoje" ou como uma view extra temporária
- Manter o mesmo design system (dark theme, Fraunces + DM Sans, paleta de cores)
- Após "Começar o dia", nunca mais mostrar o gateway naquele dia (persistir flag no `state` ou checar se já tem tarefas)

---

## Backlog futuro

- [ ] Histórico de conclusões por semana/mês
- [ ] Notificações/lembretes via Service Worker
- [ ] Export de dados (JSON ou CSV)
- [ ] Modo de foco (Pomodoro integrado)
- [ ] Estatísticas de humor e energia ao longo do tempo
