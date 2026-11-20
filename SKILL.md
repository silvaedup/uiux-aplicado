---
name: uiux-aplicado
description: Audita a interface de um app web existente com critérios de UI/UX (heurísticas de Nielsen com severidade, WCAG 2.2 AA, leis de UX, estados e microcopy, tipografia de números) e APLICA as correções no código, verificando com testes e no navegador. Use sempre que o usuário pedir para melhorar a usabilidade, a UX, a UI, a acessibilidade ou a "experiência" de um sistema, aplicar um guia ou boas práticas de design no código, revisar formulários, mensagens de erro, exclusões, toasts, foco ou área de toque, ou disser coisas como "deixa o app mais fácil de usar", "aplica isso no sistema", "tá confuso para o usuário", mesmo sem citar UX ou WCAG. Não use para criar um design do zero sem código existente nem para só explicar teoria de UX.
---

# UI/UX aplicado a um sistema existente

O objetivo é sair de "princípios" para mudanças concretas, pequenas e verificadas no código de alguém. O maior risco aqui é fazer uma lista bonita de recomendações genéricas e não mudar nada, ou o oposto: reescrever metade da interface. O caminho certo fica no meio: achar as poucas falhas que mais atrapalham quem usa, corrigir cada uma no padrão do próprio projeto e provar que funcionou.

## 1. Conhecer o sistema antes de julgar

- Leia o README, a estrutura de pastas, o ponto de entrada da interface (HTML/JS/CSS ou componentes) e os testes existentes. Descubra como o app é rodado e testado.
- Faça um inventário do que **já existe**: tema escuro, alto contraste, redução de animação, navegação por teclado, rótulos ARIA, formatação de números. Não "proponha" o que já está pronto; isso faz o relatório parecer genérico e desperdiça o tempo de quem lê.
- Dados reais são sagrados. Se o app tem banco ou arquivos de dados do usuário, teste com uma cópia ou pasta temporária (variável de ambiente, banco em memória, fixture). Nunca rode testes manuais contra os dados de verdade.

## 2. Auditar com critério

Percorra as telas e fluxos principais (cadastrar, editar, excluir, buscar, errar um campo, ficar sem rede) e anote problemas. Para cada um registre: onde está, qual princípio viola e a severidade de 0 a 4 (Nielsen: 4 = catástrofe, impede a tarefa; 3 = grave; 2 = menor; 1 = cosmético).

Use `references/checklist.md` como roteiro: ele tem as 10 heurísticas com o que procurar em código, os critérios novos do WCAG 2.2 AA, as leis de UX úteis e os padrões de correção com trechos prontos. Leia antes de auditar.

Os achados que mais aparecem em apps reais, e que costumam valer a pena primeiro:
- Excluir sem "Desfazer" (ou com confirmação em tudo, o que treina a pessoa a clicar "OK" sem ler).
- Erro de formulário mostrado num aviso que some em segundos, longe do campo.
- Mensagens vagas ("Algo deu errado", "Nada encontrado") sem dizer o que fazer.
- Alvos de toque pequenos (ícones "?" de 16px, checkboxes) e foco escondido atrás de barras fixas no celular.
- Números sem alinhamento tabular, hífen no lugar do sinal de menos, valores negativos ambíguos.

## 3. Priorizar

Ordene por severidade × frequência ÷ esforço. Corrija primeiro o que é severo e barato. Deixe registrado o que **não** vai mudar e por quê. Exemplo: manter confirmação para uma exclusão em massa irreversível é uma decisão correta, não uma falha; o "Desfazer" serve para ações pontuais.

Se a lista ficar grande, aplique as 5 a 8 mais importantes e entregue o resto como próximos passos. Mudanças pequenas e verificadas valem mais que muitas mudanças sem teste.

## 4. Aplicar no padrão do projeto

- Siga as convenções existentes: mesmo estilo de código, mesmos nomes, mesmo sistema de eventos, mesmas variáveis CSS. Uma correção de UX que introduz um framework novo não é uma correção.
- Prefira mecanismos reutilizáveis a remendos por tela: uma função `showFormError` usada por todos os formulários, um `toast` com ação usado por todas as exclusões.
- Texto da interface no idioma do app, curto, dizendo o que aconteceu e o que fazer ("Sem conexão com o servidor. Confira a internet e tente de novo.").
- Se o app tem service worker/PWA com cache versionado, suba a versão do cache, senão quem instalou continua vendo a interface antiga.
- Atualize o README ou a documentação de funções quando o comportamento visível mudar.

## 5. Verificar de verdade

- Rode a suíte de testes existente. Se mudou uma função pura (formatação, parsing), adicione um teste para ela.
- Abra o app no navegador (Playwright ou o navegador disponível) com dados temporários e exercite cada fluxo alterado: erro aparece no campo e recebe foco? Excluir esconde a linha e "Desfazer" traz de volta? Sem desfazer, a exclusão chega ao servidor?
- Meça o que é mensurável em vez de supor: tamanho real do alvo com `getBoundingClientRect`/`elementFromPoint`, contraste das cores novas, `document.activeElement` após cada ação.
- Pare o servidor de teste e apague os dados temporários ao terminar.

## 6. Relatar

Responda no idioma do usuário. Estrutura que funciona:

1. Uma frase do resultado (o que mudou, testes passaram, dados reais intocados).
2. Tabela: **o que mudou | princípio (heurística ou critério WCAG) | arquivo**.
3. Como foi verificado (testes + passos no navegador).
4. O que ficou de fora de propósito e por quê.
5. Próximo passo que só o usuário pode fazer, normalmente um teste de usabilidade com 5 pessoas fazendo tarefas reais.

Não mostre o diff inteiro; aponte arquivos e linhas (`arquivo:linha`) para quem quiser conferir.
