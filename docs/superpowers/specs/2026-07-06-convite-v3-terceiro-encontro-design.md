# Convite v3 — "Bora pro terceiro encontro?" — Desenho

- **Data:** 2026-07-06
- **Repo:** convite (publicado em GitHub Pages: https://gustavodaitxosorio.github.io/convite/)
- **Arquivo alvo:** `terceiro-encontro.html` (novo, na raiz, ao lado de `index.html` e `segundo-encontro.html`)

## Contexto

Já existem duas páginas de convite (v1 = `index.html`, v2 = `segundo-encontro.html`),
ambas single-file, tema romântico + Shrek, em PT-BR. Os dois encontros já aconteceram.
O v3 é o convite para o **terceiro encontro**, com um layout melhor pensado para
**laptop/desktop** (as versões anteriores eram mobile-first e ficavam pequenas numa tela grande),
uma **música do Shrek** e **imagens** que o usuário vai fornecer.

## Objetivo

Uma página web autocontida, alegre e romântica, que convide para o terceiro encontro,
seja gostosa de abrir num laptop, toque *All Star* e termine gerando uma mensagem
que ela copia e manda de volta confirmando o encontro.

## Requisitos decididos (com o usuário)

- **Ocasião:** convite para o 3º encontro (mesma pegada dos anteriores, layout melhor).
- **Formato:** **card centralizado** (não split-stage). Conteúdo num card no meio,
  corações e fundo em volta. Desktop-first, mas responsivo (vira 1 coluna no celular).
- **Corações:** muitos, se movendo na **diagonal** (sobem indo pra direita), nas cores
  **rosa, vermelho e verde**. Ficam **atrás** dos botões (camada de fundo, sem capturar clique).
- **Fundo:** rosé romântico profundo, com um leve brilho verde (tie-in Shrek).
- **Música:** **All Star (Smash Mouth)**. Começa no **primeiro clique** da usuária
  (autoplay é bloqueado pelos navegadores antes de interação). Botão de mute 🔇/🔊.
- **Imagens:** Shrek memes + fundos decorativos (fornecidos pelo usuário).
- **Fluxo:** completo — Sim → escolher lugar → data/hora → mensagem pra copiar.
- **Botão "não":** brincadeira nova, encadeada — ao passar o mouse (ou tocar):
  (1) **treme** e vira "Sim" (texto muda, cor vira rosa); (2) pequena pausa pra registrar
  que agora existem **dois "Sim"**; (3) os dois **deslizam um pro outro e se fundem num
  "Sim" gigante** (`.mega`), com uma explosãozinha de corações. Aí (4) **ao clicar** no
  "Sim" gigante ele **cresce mais** (o "sim!" definitivo) e só então **libera/avança pra
  próxima tela** (o reveal). Impossível recusar. NÃO é o "foge do cursor" da v2.
  (Validado no mockup e aprovado pelo usuário.)
- **Pegadinha na lista de lugares:** manter uma opção falsa/engraçada (estilo o "McDonalds"
  da v2) que solta uma piada e "explode" ao clicar. Usuário vai fornecer a lista atualizada.

## Não-objetivos (YAGNI)

- Sem backend / sem envio automático (a confirmação é mensagem copiável, como na v2).
- Sem login, sem banco, sem analytics.
- Sem galeria de fotos pessoais (não haverá fotos do casal; só Shrek + decorativo).
- Sem múltiplas cenas em tela cheia (descartamos o formato "storybook").

## Arquitetura

Um único arquivo HTML com CSS e JS inline. Sem dependências externas de rede
(mesma abordagem das v1/v2, o que mantém o deploy trivial e a página funcionando offline
depois de carregada). Assets locais (mp3, imagens) referenciados por caminho relativo.

### Unidades (cada uma com responsabilidade única)

1. **Camada de corações** (`.hearts`, `z-index:1`, `pointer-events:none`)
   - JS gera ~50 corações com propriedades aleatórias (cor rosa/vermelho/verde, tamanho,
     velocidade, atraso, rotação) via CSS custom properties (`--dx`, `--s`, `--rot`, `--op`).
   - Um único keyframe `drift` (diagonal, de baixo → cima-direita). `animation-delay`
     negativo pra tela nascer cheia.
2. **Fundo romântico** — gradientes radiais quentes (rosa/vermelho) + um radial verde suave
   + linear rosé profundo. Vinheta por cima (`z-index:2`) pra dar clima.
3. **Áudio** — elemento `<audio>` com `all-star.mp3`; toca no primeiro clique de qualquer
   botão; botão de mute persistente. Trata falha de autoplay silenciosamente.
4. **Tela Pergunta** (`#question`) — efeito máquina de escrever monta a pergunta;
   botões **Sim** e **não**; gag do "não" → "Sim" → os dois "Sim" se fundem num só →
   clique no "Sim" gigante cresce e avança pro reveal
   (treme → morph → pausa → converge → `.mega` + burst → clique → `grow` → próxima tela).
   Primeiro clique dispara a música.
5. **Tela Reveal** (`#reveal`) — confete + explosão de corações, mensagem fofa, botão "próximo".
6. **Tela Planejamento** (`#form`) — "pra onde a gente vai?" com 3 opções de lugar
   (seleção única):
   - 🍨 La Gelateria + caminhada de leve
   - 🎬 Cineminha no Kinoplex Diamante
   - 🏐 Uma aula de vôlei de areia

   Campo **data (DD/MM)** com máscara. **Pegadinha da data:** se a data escolhida for
   **depois de hoje**, aparece a mensagem *"Certeza que não quer antes?? S2"* (compara a
   data com hoje zerando as horas; some quando a data volta a ser hoje/antes).
   Campos de **horário (HH:MM)** e "ideia" opcional seguem como na v2.
   (Opção-pegadinha estilo "McDonalds" da v2 fica opcional — usuário decide se entra.)
7. **Tela Sucesso** (`#sucesso`) — monta a mensagem final e oferece **copiar** (clipboard
   com fallback `execCommand`).

Reaproveitar as funções já provadas da v2 onde fizer sentido (máscara/validação de
data e hora `fmtHora`/`horaValida`/`dataValida`, troca de telas `mostrarTela`,
copiar `copiar`, explosão de corações), adaptando textos para o 3º encontro.

## Fluxo de dados / estado

Tudo client-side, em memória (variáveis JS): tela atual, se a música já iniciou,
lugar escolhido, data, hora, ideia. Ao finalizar, monta uma string e joga num
`<textarea>` readonly pra copiar. Nada é enviado nem persistido.

## Tratamento de erros

- **Autoplay bloqueado:** `audio.play()` pode rejeitar; capturar a Promise e ignorar
  (a música tenta de novo no próximo clique). Nunca deixar um dialog/erro aparecer.
- **Validação:** data precisa casar DD/MM válido; hora HH:MM 00–23:00–59; precisa ter
  lugar escolhido OU ideia preenchida. Mensagens de erro gentis e em PT-BR.
- **Clipboard:** `navigator.clipboard.writeText` com fallback pra seleção + `execCommand('copy')`.

## Responsivo / acessibilidade

- Card com `max-width` e `clamp()` na tipografia; vira 1 coluna e reduz no mobile.
- Gag do "não" funciona por `mouseenter` **e** `click` (pra tela de toque não ficar de fora).
- Contraste de texto sobre o fundo escuro cuidado; botões com área de toque confortável.

## Assets que o usuário fornece (bloqueiam só o código final, não o desenho)

- `all-star.mp3` (a música).
- Imagens do Shrek + fundos decorativos (definir nomes/posições quando chegarem).
- Lista atualizada de 3–5 lugares reais + a opção-pegadinha.

## Critérios de sucesso

- Abre bonito num laptop, com corações diagonais rosa/vermelho/verde atrás dos botões
  e fundo romântico.
- No primeiro clique a música toca; dá pra mutar.
- O "não" vira "Sim" ao passar o mouse/tocar.
- O fluxo completo funciona: Sim → reveal → escolher lugar/data/hora → mensagem copiável.
- Funciona também aberto no celular (degrada pra 1 coluna).
- Continua single-file e publicável no GitHub Pages ao lado da v1/v2.

## Itens em aberto (a resolver na implementação)

- Texto exato de cada tela (pergunta, reveal, sucesso) — rascunho na implementação,
  ajuste fino com o usuário.
- Nomes/posições das imagens do Shrek assim que forem enviadas.
- Se entra ou não a opção-pegadinha de lugar (estilo "McDonalds" da v2) — a lista atual
  tem 3 lugares reais.
- Arquivo `all-star.mp3` a ser enviado.
