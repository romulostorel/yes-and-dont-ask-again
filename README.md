# tigrinho.dev

Um simulador do Claude Code no browser onde a única mecânica é o prompt de permissão.

Aparece tool call atrás de tool call com comandos cada vez mais cabeludos, e você fica
apertando `1. Yes`. Os comandos escalam de `npm install` até coisas que ninguém deveria
autorizar, e o site mede quanto tempo cada um ficou na sua tela antes de você aceitar.
No fim ele te mostra o extrato.

**Nada é executado.** Todo comando é uma string e todo output é ficção escrita à mão.
A página não faz nenhuma requisição de rede — não tem backend, não tem analytics,
não tem nada pra vazar.

## Rodar

Abre o `index.html` com dois cliques. É isso.

Tudo está inline num arquivo só, inclusive a fonte, justamente pra isso funcionar sem
servidor. Se você quebrar em ES modules depois, `file://` passa a bloquear por CORS e
você vai precisar de `python3 -m http.server` só pra ver a página.

## Publicar no GitHub Pages

```sh
gh repo create tigrinho-dev --public --source=. --push
gh api -X POST repos/:owner/tigrinho-dev/pages -f build_type=legacy \
  -F 'source[branch]=main' -F 'source[path]=/'
```

Ou pela interface: Settings → Pages → Deploy from a branch → `main` / `root`.

Sai no ar em `https://<user>.github.io/tigrinho-dev/` em um ou dois minutos.

O `.nojekyll` está aí pra o Pages servir o arquivo cru em vez de passar pelo Jekyll.

### Domínio próprio

Se você registrar o `tigrinho.dev`, cria um arquivo `CNAME` com o domínio dentro e
aponta o DNS pros IPs do Pages (`185.199.108.153` … `.111.153`). Aí dá pra tirar o
`/tigrinho-dev` da URL.

## Editar

Tudo mora em `index.html`. As partes que valem mexer:

| o que | onde |
|---|---|
| a escada de comandos, 5 tiers | `const TIERS` |
| os finais que matam a sessão | `const KILLS` |
| tools que não pedem permissão (`Read`, `Grep`) | `const FREEBIES` |
| as falas do Claude entre os comandos | `const PROSE` |
| as reações de "acho que fiz merda" | `const OOPS` |
| o colapso do registro, uma vez por sessão | `meltdown()` |
| verbos do spinner (`Percolating…`) | `VERBS_OK` / `VERBS_OFF` |
| quando cada tier começa | `tierFor()` |
| quantos aceites até a morte | `killAt()` |
| os ranks e seus textos | `rankFor()` |
| a tela de morte | `buildSlate()` |
| o texto que o botão copia | `shareText()` |

### Adicionar um comando

Empurra um objeto no tier certo de `TIERS`:

```js
{c:"kubectl scale deploy/api --replicas=0",   // o comando
 w:"Reduce idle compute cost",                 // a justificativa inocente
 o:"deployment.apps/api scaled"}               // o output falso
```

Campos opcionais: `kind` (`"Edit"` / `"Write"` / `"MCP"`), `diff` (linhas
`["add"|"del"|"ctx", texto]`), `after` (um segundo tool call que revela a merda que o
primeiro fez), `ok` / `bad` (cor do `●`), `note` (o que mostrar quando não tem output),
`oops` (a ficha caindo depois que já foi).

O `oops` sempre dispara; sem ele, sorteia do `OOPS` do tier — e sorteia pouco, porque
um Claude que entra em pânico toda vez para de ter graça. A piada é ele **não** notar.
Mantém as falas no registro do Claude Code de verdade: o understatement é o que faz rir,
não o palavrão. `"I should mention those were the only backups."` bate mais forte que
qualquer grito.

O campo `w` é onde a piada vive. Quanto mais razoável a justificativa, pior o comando.
E o `tell` é o que o extrato mostra no fim: a cláusula que você realmente autorizou,
puxada de dentro da parede onde estava enterrada.

### Adicionar um final

`KILLS` é uma lista, e a sessão sorteia um no carregamento. Cada final precisa de
`setup` (a fala que prepara), `c` / `w` / `tell` / `o` como qualquer comando, `dying`
(a última linha antes da tela apagar), `lede` e `share` (os textos da lápide), e um
`refuse` — o que acontece se você recusar no último segundo. O `refuse` tem duas falas,
um `mid` (um comando intermediário que o Claude roda pra "atender" sua objeção) e o
`final`, que ele executa de qualquer forma.

## Como se joga

- `1` aceita. `2` liga o `▸▸ accept edits on` e os prompts passam a se auto-aceitar
  sozinhos — você chega ao fim mais rápido. `shift+tab` retoma o controle.
- `3` recusa, e recusar realmente te protege: é assim que se chega no rank
  `CODE REVIEWER`.
- Recusar 6 seguidos tem um final próprio.

## Fonte

Geist Mono, embedada em base64 ([OFL 1.1](https://github.com/vercel/geist-font)).

Ela não tem `⎿` nem `⏵`, os dois glifos que o Claude Code de verdade usa, então eles
foram trocados por `╰` e `▸` — que a fonte tem e que ficam na grade monoespaçada.
Pelo mesmo motivo a régua e o cursor da tela de morte são CSS, não caracteres de bloco.
