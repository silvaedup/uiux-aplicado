# Checklist de auditoria e padrões de correção

Sumário: 1. Heurísticas de Nielsen · 2. WCAG 2.2 AA · 3. Leis de UX · 4. Estados e microcopy · 5. Números e dinheiro · 6. Padrões de correção (código)

## 1. Heurísticas de Nielsen: o que procurar no código

| # | Heurística | Sinais no código |
|---|---|---|
| 1 | Visibilidade do estado | Ações async sem indicador de carregamento; botão de salvar que não desabilita; sem feedback após salvar |
| 2 | Linguagem do usuário | Mensagens com termos técnicos (HTTP 500, null, "payload"); rótulos internos expostos |
| 3 | Controle e liberdade | `confirm()` antes de cada exclusão; nenhum "Desfazer"; diálogos sem Esc; sem voltar |
| 4 | Consistência | Botões primários com estilos diferentes; mesma ação com nomes diferentes em telas diferentes |
| 5 | Prevenção de erros | Inputs sem `inputmode`/`type` adequado; aceitar formatos ambíguos sem normalizar; ação destrutiva ao lado da principal |
| 6 | Reconhecer em vez de lembrar | Campos sem rótulo visível (só placeholder); filtros que somem; exigir código decorado |
| 7 | Flexibilidade | Sem atalhos de teclado para ações frequentes; sem valores padrão sensatos |
| 8 | Estética minimalista | Informação repetida; muitos botões com o mesmo peso visual |
| 9 | Ajudar a reconhecer e corrigir erros | Erro só em toast/alert; erro sem dizer qual campo nem como corrigir |
| 10 | Ajuda e documentação | Termos do domínio sem explicação contextual (tooltip "?") |

## 2. WCAG 2.2 AA: critérios que mais falham

- **1.4.3 Contraste** 4,5:1 texto normal, 3:1 texto grande e componentes (1.4.11).
- **2.2.1 Tempo ajustável**: avisos que somem devem durar o suficiente e pausar ao passar o mouse/focar.
- **2.4.3 Ordem do foco** e **2.4.7 Foco visível**.
- **2.4.11 Foco não encoberto (novo)**: barras fixas (rodapé de ações, cabeçalho) não podem esconder o elemento focado. Correção: `scroll-padding-top/bottom` no `html`.
- **2.5.7 Movimento de arrastar (novo)**: toda ação de arrastar precisa de alternativa por clique.
- **2.5.8 Tamanho do alvo (novo)**: no mínimo 24×24 CSS px, ou espaçamento equivalente. Ícones pequenos ganham área com pseudo-elemento.
- **3.2.6 Ajuda consistente (novo)**: ajuda no mesmo lugar em todas as telas.
- **3.3.1 Identificação de erro / 3.3.3 Sugestão**: erro em texto, ligado ao campo, com exemplo do formato certo.
- **3.3.7 Entrada redundante (novo)**: não pedir de novo o que a pessoa já informou no mesmo fluxo.
- **3.3.8 Autenticação acessível (novo)**: permitir colar senha e gerenciador de senhas; sem quebra-cabeça cognitivo.
- **4.1.3 Mensagens de status**: toasts e resultados de busca com `role="status"`/`aria-live`.

## 3. Leis de UX úteis na prática

- **Fitts**: ações frequentes grandes e perto do polegar; destrutivas longe das principais.
- **Hick**: menos opções por tela; menus agrupados.
- **Jakob**: siga convenções que as pessoas já conhecem de outros apps.
- **Tesler**: a complexidade inevitável fica no sistema (ex.: aceitar "1.500" e "1500,00").
- **Doherty**: resposta em < 400 ms ou indicador de progresso; atualização otimista quando seguro.

## 4. Estados e microcopy

Toda lista/tela tem estados: vazio (primeira vez), vazio (filtro sem resultado), carregando, erro, parcial, sucesso. Verifique se cada um tem texto e próxima ação.

Fórmula da mensagem de erro: **o que aconteceu + o que fazer**, sem culpar a pessoa.
- Ruim: "Valor inválido." → Bom: "Informe um valor maior que zero. Ex.: 45,90"
- Ruim: "Erro de rede." → Bom: "Sem conexão com o servidor. Confira a internet e tente de novo."
- Ruim: "Nada encontrado." → Bom: "Nenhum resultado com esses filtros. Tente outra palavra ou limpe os filtros."

## 5. Números e dinheiro

- `font-variant-numeric: tabular-nums` em colunas de valores.
- Sinal de menos tipográfico (U+2212 "−") em vez do hífen; se o app lê de volta o que formata, garanta que o parser aceita "−".
- Valores alinhados à direita; moeda formatada com `Intl.NumberFormat` do locale.
- Não depender só de cor para positivo/negativo (sinal ou ícone também).

## 6. Padrões de correção (adapte ao estilo do projeto)

### Erro de formulário inline e acessível
```js
function showFormError(form, err) {
  clearFormError(form);
  const p = document.createElement("p");
  p.className = "form-err"; p.id = form.id + "-err"; p.setAttribute("role", "alert");
  p.textContent = err.message;
  (form.querySelector(".actions") || form.querySelector("[type=submit]")).before(p);
  const field = err.field && form.elements[err.field];
  if (field) {
    field.setAttribute("aria-invalid", "true");
    field.setAttribute("aria-errormessage", p.id);
    field.focus();
    field.addEventListener("input", () => clearFormError(form), { once: true });
  }
}
```
A validação lança um erro com `field` (nome do input) para o handler saber onde marcar.

### Exclusão com "Desfazer" (exclusão adiada)
1. Esconda a linha imediatamente (`hidden`), feche o diálogo.
2. Mostre um toast "Excluído. [Desfazer]" por ~6 s; mova o foco para "Desfazer" se a ação veio de um menu.
3. Desfazer: mostre a linha de novo, nada vai ao servidor.
4. Ao fechar o toast sem desfazer: envie o DELETE.
5. `pagehide`: envie o DELETE pendente com `fetch(url, {method:"DELETE", keepalive:true})`.
6. Antes de logout ou navegação que recarrega dados, conclua a exclusão pendente.
7. Depois de re-renderizar a lista, esconda de novo as linhas pendentes.
Mantenha confirmação explícita só para exclusões em massa ou em cascata.

### Toast acessível
- Container com `role="status"` e `aria-live="polite"`; erros podem usar `role="alert"`.
- Duração: ~3,5 s informativo, ~6 s com ação, ~7 s erro. Pausa em `mouseenter`/`focusin`, retoma em `mouseleave`/`focusout`.
- Ao fechar com foco dentro dele, devolva o foco a um lugar lógico (título da tela ou o elemento que originou).
- Botão de ação com no mínimo 24×24 px (ideal 44 px de altura no celular).

### Área de toque sem mudar o visual
```css
.icon-btn { position: relative; }
.icon-btn::after { content: ""; position: absolute; inset: -6px; }
```

### Foco não encoberto por barra fixa
```css
html { scroll-padding-bottom: calc(160px + env(safe-area-inset-bottom)); }
@media (min-width: 980px) { html { scroll-padding-bottom: 24px; } }
```

### Campo inválido visível sem depender só de cor
```css
[aria-invalid="true"] { border-color: var(--danger); box-shadow: 0 0 0 1px var(--danger); }
.form-err::before { content: "⚠ "; }
```
