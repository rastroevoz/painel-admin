# Manual técnico de manutenção · Painel Rastro e Voz

Para quem for pegar este projeto e mexer nele depois. Objetivo: dizer onde fica cada coisa, sem precisar ler o arquivo inteiro para descobrir.

---

## 1. O que é este projeto

- `prototipo/painel-rastro-e-voz.html` é um **protótipo navegável**, não o produto final. Um único arquivo HTML, sem build, sem servidor, sem backend de verdade.
- Usa **React 18** e a lib **htm** (permite escrever algo parecido com JSX em template string comum, sem precisar de Babel/webpack).
- Todos os dados (usuários, lotes, chamados, auditoria etc.) são **fictícios e gerados em memória** toda vez que a página carrega. Nada é salvo de verdade; ações como "aprovar", "criar chave" só disparam um toast de confirmação.
- Os outros dois documentos da pasta `docs/` complementam este arquivo:
  - `documentacao-funcional.md` — o que cada tela faz, do ponto de vista de negócio.
  - `HANDOFF-DEV.md` — rotas de página, endpoints de API sugeridos, regras de cálculo e critérios de aceite.

Se a tarefa for "ligar isso em um backend de verdade", comece pelo `HANDOFF-DEV.md` (seção 7, "Endpoints sugeridos") e use este manual para achar onde no HTML cada tela/ação vive.

---

## 2. Mapa de arquivos

| Caminho | Para que serve |
|---|---|
| `painel-admin/prototipo/painel-rastro-e-voz.html` | O protótipo. Abrir com duplo clique no Chrome/Edge. |
| `painel-admin/docs/documentacao-funcional.md` | Descrição de cada tela, em linguagem de negócio. |
| `painel-admin/docs/HANDOFF-DEV.md` | Rotas, endpoints de API, regras de cálculo, critérios de aceite. |
| `painel-admin/docs/MANUAL-TECNICO.md` | Este arquivo. |

---

## 3. Anatomia do arquivo HTML

O arquivo tem ~1500 linhas divididas em blocos bem diferentes. **90% do que se vai editar está dentro do bloco 4.**

| Linhas | Conteúdo | Mexer? |
|---|---|---|
| 1–8 | `<head>`, título, fonte Google Fonts (Manrope) | Raramente |
| 9–161 | CSS: variáveis de design (cores, espaçamento), layout, tema claro/escuro | Sim, para estilo |
| 162–461 | Bibliotecas de terceiros **minificadas**: React 18 + ReactDOM + Scheduler, e depois a lib `htm` | **Não mexer** |
| 464–1498 | `<script id="app-src">`: todo o código da aplicação, dentro de um único `(function(){ ... })()` | **Aqui mora tudo** |
| 1499–1500 | Fechamento de `</body></html>` | Não mexer |

Dica: os scripts das bibliotecas de terceiros (React etc.) foram colados minificados de propósito, para o arquivo funcionar sem internet e sem gerenciador de pacotes. Se precisar atualizar a versão do React, é mais seguro baixar um build novo e substituir o bloco inteiro do que editar linha a linha.

---

## 4. Como o código da aplicação (linhas 464–1498) está organizado

Quem escreveu já dividiu o arquivo com comentários de seção no formato `/* ================= Nome ================= */`. Use Ctrl+F por esse padrão para navegar rápido — é o mesmo truque sugerido no `HANDOFF-DEV.md` ("busque por `function Hoje(` etc.").

| Linha | Seção | O que tem lá |
|---|---|---|
| 469 | Utilidades | Funções puras auxiliares: `rnd` (gerador aleatório com semente fixa), `rint`, `pick`, `fD`/`fDT` (formatar data), `ago` ("há 3 dias"), `nf`/`nf1` (formatar número), `first` (primeiro nome) etc. |
| 492 | Dados de exemplo | **Todos os dados fictícios**: `users`, `LOTES`, `CADEIAS`, `PROD` (catálogo), `REQS0` (solicitações), `INV0` (convites), `TICK0` (chamados), `DEVICES0` (dispositivos/filas offline), `AUDIT` (auditoria), `NOTES0`, `CONS` (consultas de QR) etc. |
| 659 | Métricas | `function metrics(dias, perfis, deleted)` — calcula os KPIs e séries usados em Hoje/Indicadores a partir dos dados de exemplo. |
| 731 | Ícones | Dicionário `IC` com os paths SVG de cada ícone + componente `<Icon>`. |
| 735 | Contexto e componentes base | `Ctx` (React Context) e o hook `useA()`; componentes genéricos: `Btn`, `Pill`, `Kpi`, `Modal`, `Sheet`, `Empty`, `DL`, `Masked`, `Stepper` etc. |
| 767 | Gráficos | `Bars`, `Line`, `HBars`, `Funnel`, `MiniMap` — gráficos desenhados à mão em SVG, sem biblioteca externa. |
| 791 | Filtros globais | `PeriodSel`, `PerfilSel`, `FilterBar` (filtro de período e perfil usado em várias telas). |
| 797 | Tela: Hoje | `function Hoje()` |
| 874 | Tela: Indicadores | `function Indicadores({focus})` |
| 984 | Tela: Acessos | `function Acessos({tab0})`, `RecusaModal`, `RowMenu` |
| 1103 | Tela: Usuários | `function Usuarios({arg})`, `function Perfil({id})` (perfil 360°) |
| 1190 | Tela: Lotes | `function Lotes({arg})` |
| 1257 | Tela: Atendimento | `function Atendimento({arg})` |
| 1335 | Tela: Saúde | `function Saude()` |
| 1380 | Tela: Configurações | `function Config({arg})` — as 6 subpáginas do menu (ver seção 6 abaixo) |
| 1398 | Paleta, notificações e shell | `Palette` (busca `Ctrl/Cmd+K`), `App()` (componente raiz: roteamento, menu lateral, cabeçalho, modais globais), `WhatsModal` |

---

## 5. Roteamento

Não tem React Router. É tudo feito à mão via **hash da URL**, dentro de `App()` (linha 1410).

- Formato: `#tela?p=periodo&perfil=lista,separada,por,virgula&a=argumento`
  Exemplo real: `#config?p=30&a=catalogo`
- `parse()` (linha 1411) lê a URL uma vez, ao carregar a página.
- `go(view, arg)` (linha 1428) troca de tela: atualiza `view`/`arg` no estado e navega.
- Um `useEffect` (linha 1425) escreve de volta na URL sempre que `view`, `period`, `perfis` ou `arg` mudam, via `history.replaceState`.
- O `arg` é o "sub-parâmetro" de cada tela (ex.: em Config, `arg="catalogo"` escolhe qual subpágina desenhar; em Lotes, `arg="lote:12"` abre o painel de um lote específico).
- Qual componente desenhar para cada `view` está no bloco de `if/else` em **linhas 1442–1451**.

**Para adicionar uma tela nova:**
1. Escrever o componente (seguindo o padrão das telas existentes).
2. Adicionar um item no array `nav` (linha 1437) para aparecer no menu lateral.
3. Adicionar um branch `else if(view==="minhatela")page=html\`<${MinhaTela} .../>\`` perto da linha 1450.

---

## 6. Tela de Configurações (`#config`) em detalhe

Fica toda na `function Config({arg})`, linha 1420.

- Array `G` (logo no início da função) — define os grupos do menu ("Operação", "Técnico", "Histórico") e as 6 subpáginas: `catalogo`, `formularios`, `creditos`, `auditoria`, `integracoes`, `arquivo`. É esse array que gera os cartões da tela `#config` sem argumento.
- Uma cadeia de `if/else if` — uma por subpágina — decide o conteúdo (`body`) a partir de `arg`.
- **As 6 subpáginas já chamam a API real** (atrás do interruptor `API_ENABLED` — ver seção 9). Cada uma tem seu próprio `useState`/`useEffect` de carregamento e sua própria função de ação (`revisar`, `cadastrarUnidade`, `salvarFormulario`, `buscarAlvo`/`concederCreditos`, `criarCampanha`/`alternarCampanha`, `enviarVoucher`, `criarChave`/`revogarChave`), todas antes do `if(!arg)return...`.
- **Para adicionar uma 7ª subpágina:** acrescentar a entrada em `G`, um novo `else if(arg==="chave"){...}` nessa cadeia e, se for chamar API, seguir o mesmo padrão das outras (estado + `useEffect` com `apiGet` + fallback para dado fictício). Dado fictício novo entra na seção "Dados de exemplo" (linha 492).
- O contrato completo de rotas está documentado em `HANDOFF-DEV.md` (seção 7) e no PDF/DOCX `edpoints_rastro_e_voz.*` na raiz do projeto.

---

## 7. Estado e dados compartilhados entre telas

- `Ctx` (linha 736) é um `React.createContext`. `useA()` (linha 737) é o atalho `useContext(Ctx)`.
- Todo componente de tela chama `const A = useA()` para ler/alterar estado global: período (`A.period`), perfis filtrados (`A.perfis`), métricas prontas (`A.M`), listas (`A.reqs`, `A.invites`, `A.tickets`, `A.devices`, `A.notes`), navegação (`A.go`, `A.openUser`), feedback (`A.toast`, `A.confirm`), tema (`A.cycleTheme`) etc.
- O objeto `A` é montado dentro de `App()` (linha 1434) e passado via `<${Ctx.Provider} value=${A}>`.
- Todo o estado vive em `useState` dentro de `App()`. **Não existe persistência real** — um F5 na página reseta tudo para os dados de exemplo originais (exceto tema, que fica salvo em `localStorage["rv-tema"]`).

---

## 8. Dados fictícios e a semente aleatória

- Linha 470: `let _s=915262;` — semente fixa do gerador `rnd()` (linha 471, um PRNG simples tipo mulberry32).
- Por causa dessa semente fixa, **os dados fictícios são sempre os mesmos** a cada carregamento (mesmos nomes, mesmos lotes, mesmas datas). Isso é proposital, para quem for validar a tela sempre ver o mesmo cenário.
- Se precisar de mais variedade de exemplos, mude a semente (`_s`) ou os arrays-fonte (`FN`, `LN`, `CPI`, `CADEIAS` etc., a partir da linha 495). Trocar `rnd()` por `Math.random()` funciona, mas perde a reprodutibilidade — os dados mudam a cada F5.

---

## 9. Onde entram APIs de verdade

A tela de **Configurações** (as 6 subpáginas: catálogo, formulários, créditos, auditoria, integrações e arquivo) já está com as chamadas de API escritas, seguindo o contrato oficial (`edpoints_rastro_e_voz.pdf`/`.docx`, na raiz do projeto, fora de `painel-admin/`). O resto do protótipo (Hoje, Indicadores, Acessos, Usuários, Lotes, Atendimento, Saúde) ainda não tem chamada nenhuma — só os dados fictícios da seção "Dados de exemplo".

### Como ligar a API que já está escrita (Configurações)

Toda chamada passa por um helper único, na seção `/* ================= API administrativa ================= */`, logo no início do `<script id="app-src">` (por volta da linha 496):

```js
const API_ENABLED=false;   // linha 500 — trocar para true quando a API estiver disponível
const API_BASE="";         // linha 501 — apontar para a URL da API, ex.: "https://app.rastroevoz.com.br"
```

Passo a passo para ativar:

1. Abrir `prototipo/painel-rastro-e-voz.html` e localizar `const API_ENABLED=false;` (Ctrl+F).
2. Trocar para `const API_ENABLED=true;`.
3. Preencher `API_BASE` com a raiz da API (sem `/api` no final — cada chamada já monta o caminho completo, ex. `/api/admin_catalog_suggestions.php?...`). Se a API estiver no mesmo domínio que o painel, pode deixar `API_BASE=""`.
4. Se a API exigir autenticação por cookie de sessão, nada mais precisa mudar: `apiCall` (linha 502) já manda `credentials:"include"` em toda chamada. Se for por `Authorization: Bearer <token>`, é preciso guardar o token (ex. depois do login) e incluir o header dentro de `apiCall`, no objeto passado a `fetch`.
5. Testar cada subpágina de Configurações (`#config?a=catalogo`, `#config?a=formularios`, `#config?a=creditos`, `#config?a=auditoria`, `#config?a=integracoes`, `#config?a=arquivo`) e confirmar que o aviso "API ainda não conectada" some — isso indica que a chamada real teve `success:true` na resposta.

Enquanto `API_ENABLED` for `false` (ou uma chamada falhar/der erro), cada subpágina cai sozinha para os dados de exemplo e mostra esse aviso — o protótipo continua navegável mesmo sem API. Essa lógica de fallback fica dentro de `apiCall` (linhas 502–509) e é usada por `apiGet`/`apiPost` (linhas 510–511), chamados de dentro de `function Config({arg})` (linha 1420).

### Rotas já plugadas por subpágina

| Subpágina | Rotas usadas |
|---|---|
| Catálogo (`a=catalogo`) | `GET/POST admin_catalog_suggestions.php` (sugestões), `GET/POST measurement_units.php` (unidades) |
| Formulários (`a=formularios`) | `GET admin_chain_subtypes.php?action=list`, `POST ...?action=update_form_config` |
| Créditos (`a=creditos`) | `GET/POST qr_credit_promotions.php` (campanhas, busca de usuário, concessão de créditos), `POST mvp_admin.php?action=email_voucher` |
| Auditoria (`a=auditoria`) | `GET mvp_admin.php?action=audit_log` |
| Integrações (`a=integracoes`) | `GET/POST admin_partner_keys.php` (listar, criar, revogar chave) |
| Arquivo (`a=arquivo`) | `GET mvp_admin.php?action=overview` |

O botão "Criar link de passagem" (em Integrações) continua só local (`A.toast`) porque o contrato da API não tem rota para isso ainda — quando existir, seguir o mesmo padrão de `apiPost`.

### Para ligar as demais telas (Hoje, Indicadores, Acessos, Usuários, Lotes, Atendimento, Saúde)

Essas telas ainda não têm nenhuma chamada de API escrita. Para começar:

1. Consultar a tabela "Endpoints sugeridos" em `HANDOFF-DEV.md` (seção 7) — mapeia rota sugerida × retorno esperado para cada uma.
2. Reaproveitar o mesmo helper (`apiGet`/`apiPost`, linhas 510–511) em vez de criar um novo mecanismo de `fetch`.
3. Trocar os arrays fixos da seção "Dados de exemplo" (linha 492) por estado carregado via `apiGet` em `useEffect`, seguindo o padrão usado em `Config` (linha 1420): estado próprio + estado "offline" + fallback pro dado fictício quando `res.ok` for `false`.
4. Trocar as ações que hoje só chamam `A.toast("mensagem")` por `apiPost` para o endpoint correspondente, seguido de atualização do estado local.
5. Manter a UI como referência de comportamento — o handoff é explícito que o protótipo é "referência visual e de comportamento", não código para produção (linha 18 do `HANDOFF-DEV.md`).

---

## 10. Tema claro/escuro

- Variáveis de cor em `:root { ... }` (linha 10) e a versão escura em `:root[data-theme="dark"] { ... }` (linha 11). Todo o resto do CSS usa `var(--nome)`.
- `cycleTheme()` (linha 1430, dentro de `App()`) alterna entre `"light"` e `"dark"`, salva em `localStorage["rv-tema"]` e aplica via `document.documentElement.setAttribute("data-theme", theme)` (linha 1424).
- **Para mudar uma cor:** editar a variável correspondente nas duas linhas (10 e 11), não em cada componente.

---

## 11. Ícones

- Dicionário `IC` (linha 732): cada chave é o nome do ícone, o valor é uma string com os comandos SVG separados por `|`.
  - Sem prefixo → vira um `<path d="...">`.
  - Prefixo `c:cx,cy,r` → vira um `<circle>`.
  - Prefixo `r:x,y,w,h,rx` → vira um `<rect>`.
- Componente `<Icon n="nome" s={tamanho}/>` (linha 733) monta o SVG a partir do dicionário.
- **Para adicionar um ícone novo:** conseguir o path SVG (24×24, estilo [Feather Icons](https://feathericons.com) é o que já está em uso) e acrescentar uma entrada em `IC`.

---

## 12. Convenções de código (para editar de forma consistente)

- **Sem build step de propósito.** Não dá para usar JSX de verdade, TypeScript, imports de módulos npm etc. — tudo tem que rodar direto no navegador abrindo o arquivo local. O que parece JSX (`html\`<div>...\`\`}) é a lib `htm`, que interpreta a string em tempo de execução.
- Estilo compacto: uma função por linha na maioria das vezes, sempre com ponto e vírgula, pouca quebra de linha dentro de componentes pequenos. Isso é de propósito para manter tudo em um arquivo só sem ficar quilométrico — **manter o mesmo estilo ao editar**, em vez de reformatar para múltiplas linhas.
- Nomes de dados de domínio em português (`lotes`, `cadeia`, `perfis`, `TEAM`); utilitários curtos e genéricos em inglês (`nf`, `fD`, `ago`, `pick`).
- Navegar pelo arquivo por nome de função com Ctrl+F, não por número de linha decorado — os números mudam a cada edição. As tabelas deste manual são um ponto de partida, não um substituto pra busca.

---

## 13. Como rodar e testar

- Abrir `prototipo/painel-rastro-e-voz.html` direto no Chrome ou Edge (duplo clique). Não precisa de servidor, `npm install` nem build.
- Internet só é usada para carregar a fonte Manrope; sem conexão, cai para Segoe UI (a página avisa isso no comentário do handoff).
- Atalhos úteis pra navegar testando: `Ctrl/Cmd+K` abre busca/paleta de comandos, `?` mostra atalhos de teclado.
- Depois de editar, dar refresh na página já reflete a mudança — não tem cache de build para limpar.

---

## 14. Checklist de tarefas comuns

| Tarefa | Onde mexer |
|---|---|
| Adicionar subpágina em Configurações | Array `G` (linha 1383) + novo `else if` em `Config` (linhas 1389–1394) |
| Adicionar tela nova no menu | Array `nav` em `App()` (linha 1437) + branch no `if/else` de páginas (linhas 1442–1451) |
| Mudar uma cor do tema | Variável em `:root` (linha 10) e `:root[data-theme="dark"]` (linha 11) |
| Adicionar um ícone | Entrada no dicionário `IC` (linha 732) |
| Mudar/adicionar dado fictício | Seção "Dados de exemplo" (a partir da linha 492) |
| Mudar uma regra de cálculo de KPI | `function metrics(...)` (linha 667) |
| Ligar a API das 6 subpáginas de Configurações | Trocar `API_ENABLED` para `true` e preencher `API_BASE` (linhas 500–501) — ver seção 9 |
| Ligar uma ação em API real numa tela que ainda não tem | Ver seção 9 (parte "Para ligar as demais telas") + tabela de endpoints no `HANDOFF-DEV.md` |
| Documentar uma rota de API nova | `HANDOFF-DEV.md`, seção "7. Endpoints sugeridos" |

---

## 15. O que não fazer

- **Não editar** o bloco de bibliotecas minificadas (linhas 162–461) — é React/ReactDOM/Scheduler/htm de terceiros. Se precisar atualizar a versão, substituir o bloco inteiro por um build novo, não editar linha a linha.
- **Não tratar este arquivo como código de produção.** É um protótipo de referência visual e de comportamento — o próprio handoff é explícito sobre isso.
- **Não remover a semente fixa do `rnd()`** sem necessidade — isso muda todos os dados de exemplo mostrados nas telas e quebra a reprodutibilidade usada para validar comportamento.
- **Não dividir o arquivo em vários arquivos/módulos** sem repensar a arquitetura — hoje tudo depende de estar no mesmo escopo (um único IIFE), então recortar um pedaço para outro arquivo exige resolver imports/exports, o que não é trivial com `htm` sem bundler.
