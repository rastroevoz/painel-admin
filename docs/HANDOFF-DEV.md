# Handoff para desenvolvimento · Painel Administrativo Rastro e Voz

Para: Quemia (DEV)
De: Marcos Caridade (UX/UI)
Data: 15/09/2026 · Versão 2 (cores corrigidas, arquivo funciona sem internet)

---

## 0. Como usar este pacote

| Item | O que é |
|---|---|
| `prototipo/painel-rastro-e-voz.html` | Protótipo navegável completo. Abra com duplo clique no Chrome ou Edge. Funciona sem internet (só a fonte Manrope precisa de conexão; sem ela aparece Segoe UI). |
| Link online | https://claude.ai/artifact/NRXcWaWNGeSRQoyn3L3XkQ |
| `docs/HANDOFF-DEV.md` | Este documento: rotas, componentes, dados, cálculos e critérios de aceite |
| `docs/documentacao-funcional.md` | Descrição funcional de cada tela |

O protótipo é a **referência visual e de comportamento**, e os dados dele são fictícios. O código é React 18 com `htm`, tudo em um arquivo, e serve para consultar a lógica: busque por `function metrics(` para ver os cálculos e por `function Hoje(`, `function Indicadores(` etc. para ver cada tela. Não é código para produção.

Dicas para navegar no protótipo:
- A URL guarda o estado. Por exemplo, `#indicadores?p=40&perfil=Produtor(a)` abre Indicadores com período de 40 dias e só produtores.
- `Ctrl/Cmd + K` abre a busca, e `?` mostra os atalhos.
- A última aba do navegador em modo responsivo (menos de 680 px) mostra a versão para celular.

---

## 1. Rotas

| Rota | Tela | Prioridade |
|---|---|---|
| `/admin` ou `#hoje` | Hoje | P0 |
| `#indicadores` (âncoras: `aquisicao`, `rastreabilidade`, `alcance`, `sistema`) | Indicadores | **P0 (requisito da cliente)** |
| `#saude` | Saúde do sistema | **P0 (requisito da cliente)** |
| `#acessos` (abas: solicitações, convites, pessoas) | Acessos | P1 |
| `#usuarios` e `#perfil?a={id}` | Usuários 360° e perfil | P1 |
| `#lotes` e `#lotes?a=lote:{id}` | Lotes | P1 |
| `#atendimento` e `#atendimento?a=t:{id}` | Atendimento | P2 |
| `#config` e `#config?a={subpagina}` | Configurações | P2 |

Filtros globais em query string: `p` (período em dias: 7, 30, 40, 90, 365) e `perfil` (lista separada por vírgula).

**Ordem sugerida de entrega:**
1. Registro de eventos (seção 4)
2. Endpoints de métricas
3. Hoje e Indicadores
4. Saúde do sistema com filas presas
5. Acessos
6. Usuários
7. Lotes
8. Atendimento
9. Configurações

Sem os eventos da seção 4, os indicadores de ativos e de filas presas continuam zerados, como acontece hoje no staging.

---

## 2. Design tokens

### Cores

**O tema padrão é sempre o claro.** O tema escuro só é ativado manualmente pelo menu do usuário e **não** acompanha o modo escuro do sistema operacional. A escolha fica em `localStorage` (`rv-tema`).

A barra lateral verde escura (`#1E4A2C`) é intencional: é a cor da marca. O restante da interface é claro.

| Token | Claro (padrão) | Escuro (opcional) | Uso |
|---|---|---|---|
| `--bg` | `#F3F6F1` | `#121417` | Fundo da página |
| `--card` | `#FFFFFF` | `#1B1E22` | Cartões, tabelas |
| `--card2` | `#F8FAF7` | `#22262B` | Hover, áreas secundárias |
| `--ink` | `#17301F` | `#ECEFF1` | Texto principal |
| `--muted` | `#56665A` | `#A3ABB3` | Texto secundário |
| `--line` | `#DCE3D8` | `#30353B` | Bordas |
| `--line2` | `#EAEFE7` | `#262A30` | Divisórias internas |
| `--green` | `#2C6B3F` | `#5DB37A` | Primária, situação Bom |
| `--green-s` | `#E3EEE3` | `#1F2E25` | Fundo de selo verde |
| `--amber` | `#8F5A0E` | `#E6B060` | Situação Atenção |
| `--amber-s` | `#F7EBD5` | `#352B18` | Fundo de selo âmbar |
| `--red` | `#A33D29` | `#EF8F7A` | Situação Crítico, erros |
| `--red-s` | `#F8E4DF` | `#3A221D` | Fundo de selo vermelho |
| `--blue` | `#2F5E7A` | `#86BAD8` | Informação |
| `--side` | `#1E4A2C` | `#16191C` | Barra lateral |

### Cores por perfil (gráficos e selos)

| Perfil | Cor |
|---|---|
| Produtor(a) | `#2C6B3F` |
| Artesã(o) | `#C08322` |
| Instituição | `#3A7497` |
| Comprador(a) | `#8561A0` |
| Outros | `#8A968D` |

### Cores por cadeia

| Cadeia | Cor |
|---|---|
| Carnaúba | `#2C6B3F` |
| Artesanato em palha | `#C08322` |
| Cachaça artesanal | `#3A7497` |
| Mel silvestre | `#A7662B` |

### Tipografia e espaçamento

- **Fonte:** Manrope (400, 500, 600, 700, 800), com fallback `"Segoe UI", system-ui, sans-serif`. Números com `font-feature-settings: "tnum"`.
- **Tamanhos:** página 14 px; `h1` 26 px/800; `h2` 17 px/700; `h3` 14 px/700; texto pequeno 12 px; big number dos cartões 40 px/800; valor de KPI 28 px/800.
- **Raios:** cartões 12 px; cartões de resumo 14 px; botões e campos 8 px; selos 20 px.
- **Espaçamento:** grade de 14 px entre cartões; padding de cartão 18 px; conteúdo com 24 a 28 px nas laterais.
- **Breakpoints:**
  - até 1180 px: grades de 4 viram 2
  - até 980 px: barra lateral só com ícones
  - até 680 px: barra inferior com Hoje, Acessos, Indicadores, Atender e Mais
- **Tema:** claro por padrão (`<html data-theme="light">`). Escuro só por escolha manual. Não usar `prefers-color-scheme`.

---

## 3. Componentes

| Componente | Descrição | Estados e regras |
|---|---|---|
| **CartaoResumo** (QCard) | Título curto, big number, uma linha de explicação, selo de situação, ícone de informação, variação | `ok` (Bom, barra verde), `warn` (Atenção, barra âmbar), `bad` (Crítico, barra vermelha). Clique leva à seção em Indicadores. O tooltip mostra a pergunta e a regra |
| **KPI** | Rótulo, valor, subtítulo opcional, variação, tooltip de definição | Pode ser clicável e levar à lista filtrada. Variação com seta e cor; `inv` inverte a cor quando subir é ruim |
| **Selo** (Pill) | Texto curto com cor | `g` verde, `a` âmbar, `r` vermelho, `b` azul, `n` neutro |
| **SeloPerfil** | Ponto colorido e nome do perfil | Cores da tabela de perfis |
| **SeloEspera** (SLA) | Tempo de espera | Neutro até a meta, âmbar entre meta e 2× a meta, vermelho acima de 2× a meta |
| **Evidencias** | 4 ícones: GPS, foto, áudio, documento | Preenchido quando anexado, esmaecido quando falta, com `aria-label` descrevendo os quatro |
| **Prontidao** | Selo do lote | 100%: "Pronto para auditoria" em verde; 50–75%: âmbar; abaixo de 50%: vermelho |
| **Stepper** | Pontos da jornada | 7 etapas (produção) ou 4 (comprador) |
| **FiltroPerfil** | Chips de seleção múltipla | "Todos os perfis" limpa a seleção; o estado vai para a URL |
| **FiltroPeriodo** | Select | 7, 30, 40, 90 dias, 12 meses |
| **PainelLateral** (Sheet) | Detalhe à direita, 480 px | Fecha com Esc e clique fora; foco no primeiro elemento |
| **Janela** (Modal) | Confirmações e formulários curtos | Fecha com Esc |
| **Aviso** (Toast) | Mensagem na base | 3,2 s; com botão Desfazer, 8 s |
| **DadoMascarado** | CPF, CNPJ, telefone | Botão Mostrar/Ocultar. **Mostrar registra evento na auditoria** |
| **JanelaWhatsApp** | Mensagem editável e botão enviar | Em produção abre `https://wa.me/55{telefone}?text={mensagem}` |
| **Graficos** | Barras empilhadas, linha, barras horizontais, funil | No protótipo são SVG próprio; em produção pode ser Recharts ou Chart.js |
| **Vazio** | Título, texto e ação | Obrigatório em toda lista |
| **Carregando** | Skeleton | Mesma forma do conteúdo |

---

## 4. Eventos que precisam ser registrados (bloqueante)

Sem estes eventos não dá para calcular usuários ativos, alcance de QR nem filas presas.

### 4.1 `eventos_uso`
Registrar quando o usuário:

| `tipo` | Quando |
|---|---|
| `login` | Login manual no app ou painel (não contar renovação automática de sessão) |
| `app_aberto` | App aberto em primeiro plano com sessão válida (no máximo 1 por hora por usuário) |
| `lote_criado` | Lote salvo |
| `lote_editado` | Lote alterado |
| `evidencia_anexada` | GPS, foto, áudio ou documento anexado (`meta.tipo`) |
| `qr_emitido` | QR Code emitido |
| `chamado_aberto` | Chamado criado |

### 4.2 `consultas_qr`
Registrar toda abertura da página pública do QR:
- lote
- data e hora
- cidade, estado e país aproximados (via geolocalização do IP, sem guardar o IP completo)
- hash do aparelho (cookie anônimo ou hash de user agent + IP truncado, com rotação diária)
- comprador, quando a pessoa estiver logada

### 4.3 `filas_sincronizacao`
O app envia a cada tentativa de sincronização:
- id do dispositivo, modelo e versão do app
- quantidade de itens pendentes e o tipo
- data do item pendente mais antigo
- resultado (`ok` ou `erro`) e código ou mensagem de erro

---

## 5. Modelo de dados

SQL de referência (MySQL 8 ou PostgreSQL, ajustar tipos se necessário).

```sql
-- Usuários e organizações
CREATE TABLE usuarios (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  nome            VARCHAR(160) NOT NULL,
  perfil          ENUM('produtor','artesao','instituicao','comprador','outros') NOT NULL,
  organizacao_id  BIGINT NULL,
  cidade          VARCHAR(80),
  uf              CHAR(2),
  cadeia          VARCHAR(60),
  email           VARCHAR(160),
  telefone        VARCHAR(20),          -- guardar completo, exibir mascarado
  documento       VARCHAR(20),          -- guardar completo, exibir mascarado
  codigo_rastro   VARCHAR(14) UNIQUE,   -- RV-XXXX-XXXX
  papel           ENUM('administrador','operacional','membro') DEFAULT 'membro',
  pedido_em       DATETIME NOT NULL,
  aprovado_em     DATETIME NULL,
  ativado_em      DATETIME NULL,
  desativado_em   DATETIME NULL
);

CREATE TABLE solicitacoes (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  nome            VARCHAR(160) NOT NULL,
  organizacao     VARCHAR(160),
  perfil          ENUM('produtor','artesao','instituicao','comprador','outros') NOT NULL,
  email           VARCHAR(160),
  email_confirmado_em DATETIME NULL,
  documento       VARCHAR(20),
  telefone        VARCHAR(20),
  finalidade      TEXT,
  status          ENUM('pendente','aprovada','recusada','mesclada') DEFAULT 'pendente',
  motivo_recusa   VARCHAR(80) NULL,
  decidido_por    BIGINT NULL,
  decidido_em     DATETIME NULL,
  mesclada_em_id  BIGINT NULL,
  usuario_existente_id BIGINT NULL,
  criada_em       DATETIME NOT NULL
);

CREATE TABLE convites (
  codigo          VARCHAR(40) PRIMARY KEY,  -- PRODUTOR-XXXXXXXX etc.
  solicitacao_id  BIGINT NULL,
  usuario_id      BIGINT NULL,
  destino_email   VARCHAR(160),
  enviado_em      DATETIME NOT NULL,
  aberto_em       DATETIME NULL,
  ativado_em      DATETIME NULL,
  expira_em       DATETIME NOT NULL
);

-- Rastreabilidade
CREATE TABLE lotes (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  codigo          VARCHAR(60) NOT NULL,     -- CARNAUBA_<timestamp>_<hash>
  usuario_id      BIGINT NOT NULL,
  produto         VARCHAR(120),
  quantidade      DECIMAL(10,2),
  unidade         VARCHAR(10),
  cidade          VARCHAR(80),
  destino         VARCHAR(80),
  cadeia          VARCHAR(60),
  criado_offline  BOOLEAN DEFAULT FALSE,
  criado_em       DATETIME NOT NULL,        -- data do registro no campo
  sincronizado_em DATETIME NOT NULL,        -- data de chegada ao servidor
  excluido_em     DATETIME NULL,            -- lixeira de 24 h
  duplicado_de    BIGINT NULL
);

CREATE TABLE evidencias (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  lote_id         BIGINT NOT NULL,
  tipo            ENUM('gps','foto','audio','documento') NOT NULL,
  arquivo_url     VARCHAR(255),
  latitude        DECIMAL(9,6), longitude DECIMAL(9,6),
  criada_em       DATETIME NOT NULL
);

CREATE TABLE qr_codes (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  lote_id         BIGINT NOT NULL UNIQUE,
  emitido_em      DATETIME NOT NULL
);

CREATE TABLE consultas_qr (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  lote_id         BIGINT NOT NULL,
  consultado_em   DATETIME NOT NULL,
  cidade          VARCHAR(80), uf VARCHAR(40), pais VARCHAR(60),
  aparelho_hash   CHAR(64) NOT NULL,
  comprador_id    BIGINT NULL,
  INDEX (lote_id, consultado_em), INDEX (consultado_em)
);

-- Uso e sistema
CREATE TABLE eventos_uso (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  usuario_id      BIGINT NOT NULL,
  tipo            VARCHAR(40) NOT NULL,
  meta            JSON NULL,
  criado_em       DATETIME NOT NULL,
  INDEX (usuario_id, criado_em), INDEX (criado_em)
);

CREATE TABLE dispositivos (
  id              VARCHAR(40) PRIMARY KEY,
  usuario_id      BIGINT NOT NULL,
  modelo          VARCHAR(80),
  versao_app      VARCHAR(20),
  ultimo_contato_em DATETIME,
  pendentes       INT DEFAULT 0,
  pendentes_tipo  VARCHAR(80),
  pendente_desde  DATETIME NULL,
  resolvido_em    DATETIME NULL
);

CREATE TABLE tentativas_sincronizacao (
  id              BIGINT PRIMARY KEY AUTO_INCREMENT,
  dispositivo_id  VARCHAR(40) NOT NULL,
  resultado       ENUM('ok','erro') NOT NULL,
  erro_codigo     VARCHAR(40) NULL,
  erro_mensagem   VARCHAR(255) NULL,
  itens           INT,
  criada_em       DATETIME NOT NULL
);

-- Atendimento e auditoria
CREATE TABLE chamados (
  id BIGINT PRIMARY KEY AUTO_INCREMENT, usuario_id BIGINT NOT NULL,
  assunto VARCHAR(160), canal ENUM('app','whatsapp','email'),
  status ENUM('sem_resposta','em_andamento','aguardando_usuario','resolvido') DEFAULT 'sem_resposta',
  responsavel_id BIGINT NULL, nota_satisfacao TINYINT NULL,
  criado_em DATETIME NOT NULL, primeira_resposta_em DATETIME NULL, resolvido_em DATETIME NULL
);
CREATE TABLE mensagens (
  id BIGINT PRIMARY KEY AUTO_INCREMENT, chamado_id BIGINT NOT NULL,
  autor ENUM('usuario','equipe','nota_interna'), autor_id BIGINT,
  texto TEXT, audio_url VARCHAR(255), audio_duracao INT, criada_em DATETIME NOT NULL
);
CREATE TABLE notas_usuario (
  id BIGINT PRIMARY KEY AUTO_INCREMENT, usuario_id BIGINT NOT NULL,
  autor_id BIGINT NOT NULL, texto TEXT, criada_em DATETIME NOT NULL
);
CREATE TABLE incidentes (
  id BIGINT PRIMARY KEY AUTO_INCREMENT, titulo VARCHAR(160),
  inicio DATETIME, fim DATETIME NULL, status VARCHAR(20)
);
CREATE TABLE auditoria (
  id BIGINT PRIMARY KEY AUTO_INCREMENT, ator_id BIGINT NULL,  -- NULL = sistema
  acao VARCHAR(60), alvo_tipo VARCHAR(40), alvo_id BIGINT,
  descricao VARCHAR(255),  -- texto legível: "Marcos aprovou o acesso de Francisco"
  dados_tecnicos JSON, criado_em DATETIME NOT NULL
);
```

---

## 6. Cálculo dos indicadores

Parâmetros de todos os endpoints: `dias` (7, 30, 40, 90, 365) e `perfis` (opcional). "Período anterior" é a janela de mesmo tamanho imediatamente antes.

### 6.1 Ativação de usuários

```sql
-- Novos usuários no período
SELECT COUNT(*) FROM usuarios
WHERE ativado_em > NOW() - INTERVAL :dias DAY AND perfil IN (:perfis);

-- Ativos em N dias (N = 7, 30, 40)
SELECT COUNT(DISTINCT e.usuario_id) FROM eventos_uso e
JOIN usuarios u ON u.id = e.usuario_id
WHERE e.criado_em > NOW() - INTERVAL :n DAY AND u.perfil IN (:perfis);

-- Taxa de ativação (aprovados no período, janela mínima de 30 dias)
SELECT ROUND(100 * SUM(ativado_em IS NOT NULL) / COUNT(*)) FROM usuarios
WHERE aprovado_em > NOW() - INTERVAL GREATEST(:dias, 30) DAY AND perfil IN (:perfis);
```

- **Novos usuários por período:** agrupar `ativado_em` por dia (7 dias), por semana (30 a 90 dias) ou por mês (12 meses), separando por perfil.
- **Evolução dos ativos:** para cada fim de intervalo, contar usuários distintos com evento nos N dias anteriores àquela data.
- **Funil:** considera quem pediu acesso nos últimos `max(dias, 90)` dias, exceto compradores. Etapas: aprovado, ativado, tem lote, tem QR, tem consulta.

### 6.2 Rastreabilidade

```sql
-- Registros rastreáveis no período
SELECT COUNT(*) FROM lotes l JOIN usuarios u ON u.id = l.usuario_id
WHERE l.excluido_em IS NULL AND l.criado_em > NOW() - INTERVAL :dias DAY AND u.perfil IN (:perfis);

-- Produtores que registraram / produtores ativos
WITH ativos AS (
  SELECT DISTINCT e.usuario_id FROM eventos_uso e JOIN usuarios u ON u.id = e.usuario_id
  WHERE e.criado_em > NOW() - INTERVAL :dias DAY AND u.perfil <> 'comprador' AND u.perfil IN (:perfis)
),
registraram AS (
  SELECT DISTINCT usuario_id FROM lotes
  WHERE excluido_em IS NULL AND criado_em > NOW() - INTERVAL :dias DAY
)
SELECT (SELECT COUNT(*) FROM ativos a JOIN registraram r USING (usuario_id)) AS registraram,
       (SELECT COUNT(*) FROM ativos) AS ativos;
```

- **Prontos para auditoria:** lote com os 4 tipos de evidência.
- **Percentual de prontidão:** quantidade de tipos distintos ÷ 4.

### 6.3 QR e alcance

```sql
-- Consultas e consultas únicas
SELECT COUNT(*) AS consultas, COUNT(DISTINCT aparelho_hash) AS unicas
FROM consultas_qr c JOIN lotes l ON l.id = c.lote_id JOIN usuarios u ON u.id = l.usuario_id
WHERE c.consultado_em > NOW() - INTERVAL :dias DAY AND u.perfil IN (:perfis);

-- % de QR Codes já consultados (histórico total)
SELECT ROUND(100 * COUNT(DISTINCT c.lote_id) / COUNT(DISTINCT q.lote_id))
FROM qr_codes q LEFT JOIN consultas_qr c ON c.lote_id = q.lote_id;

-- Cidades de consulta
SELECT cidade, uf, COUNT(*) total FROM consultas_qr
WHERE consultado_em > NOW() - INTERVAL :dias DAY GROUP BY cidade, uf ORDER BY total DESC LIMIT 10;
```

### 6.4 Sistema e alertas

```sql
-- Filas offline presas
SELECT d.*, u.nome, u.perfil FROM dispositivos d JOIN usuarios u ON u.id = d.usuario_id
WHERE d.pendentes > 0 AND d.resolvido_em IS NULL
  AND (
    (d.pendente_desde < NOW() - INTERVAL 24 HOUR AND d.ultimo_contato_em > d.pendente_desde)
    OR (SELECT COUNT(*) FROM tentativas_sincronizacao t
        WHERE t.dispositivo_id = d.id AND t.resultado = 'erro'
          AND t.criada_em > NOW() - INTERVAL 24 HOUR) >= 3
  );

-- Sucesso de sincronização em 24 h
SELECT ROUND(100 * SUM(resultado='ok') / COUNT(*)) FROM tentativas_sincronizacao
WHERE criada_em > NOW() - INTERVAL 24 HOUR;
```

Disponibilidade, taxa de erro e tempo de resposta vêm do monitoramento do servidor, por exemplo UptimeRobot ou os logs de acesso das rotas `/api/*.php`.

### 6.5 Regras de situação dos cartões

| Cartão | Bom | Atenção | Crítico |
|---|---|---|---|
| Ativação de usuários | novos ≥ anterior **e** ativação ≥ 60% | demais casos | novos = 0 |
| Rastreabilidade | registraram ≥ 50% dos ativos | 25% a 49% | < 25% |
| QR e alcance | consultas ≥ anterior **e** QRs consultados ≥ 50% | demais casos | consultas = 0 |
| Sistema e alertas | disponibilidade 30 d ≥ 99% **e** 0 filas presas | 1 a 4 filas presas | disponibilidade < 99% **ou** ≥ 5 filas presas |

Deixar os limites configuráveis.

---

## 7. Endpoints sugeridos

Todos exigem sessão de administrador. Parâmetros comuns: `dias` e `perfis`.

| Método e rota | Retorno |
|---|---|
| `GET /api/admin/resumo` | Os 4 cartões: `titulo, valor, explicacao, situacao, variacao` |
| `GET /api/admin/pendencias` | Itens da caixa "O que precisa de você", ordenados por impacto |
| `GET /api/admin/indicadores/ativacao` | KPIs, série por perfil, série de ativos 7/30/40, tabela por perfil, funil |
| `GET /api/admin/indicadores/rastreabilidade` | KPIs, série por cadeia, listas, evidências, territórios |
| `GET /api/admin/indicadores/alcance` | KPIs, série, cidades, top QR, compradores |
| `GET /api/admin/indicadores/sistema` | KPIs e filas presas |
| `GET /api/admin/saude` | Situação, KPIs, filas presas, série 30 dias, incidentes, rotas lentas |
| `POST /api/admin/filas/{id}/resolver` | Marca fila como resolvida |
| `GET /api/admin/solicitacoes?status=` | Lista com grupos de similares |
| `POST /api/admin/solicitacoes/aprovar` | `{ids: []}`, cria convites e envia e-mail |
| `POST /api/admin/solicitacoes/desfazer` | Reverte aprovação ou recusa em até 10 s |
| `POST /api/admin/solicitacoes/recusar` | `{ids: [], motivo, mensagem}` |
| `POST /api/admin/solicitacoes/{id}/mesclar` | Mescla similares |
| `GET /api/admin/convites` e `POST /api/admin/convites/{codigo}/lembrar` e `/renovar` | Convites |
| `GET /api/admin/usuarios?perfil=&etapa=` | Lista com etapa da jornada e próximo passo |
| `GET /api/admin/usuarios/{id}` | Perfil 360°: jornada, linha do tempo, notas, chamados, dispositivos |
| `POST /api/admin/usuarios/{id}/notas` e `PATCH /api/admin/usuarios/{id}/papel` | Notas e papel |
| `POST /api/admin/auditoria/visualizacao` | Registra quando alguém revela dado mascarado |
| `GET /api/admin/lotes?filtros` e `GET /api/admin/lotes/{id}` | Lotes |
| `DELETE /api/admin/lotes/{id}` e `POST /api/admin/lotes/{id}/restaurar` | Lixeira 24 h |
| `GET /api/admin/chamados` e `POST /api/admin/chamados/{id}/mensagens` e `PATCH /api/admin/chamados/{id}` | Atendimento |
| `GET /api/admin/catalogo` | Cadeias com seus produtos e unidades |
| `POST /api/admin/catalogo/{cadeia}/produtos` | Cria produto `{nome, unidade}` na cadeia |
| `PATCH /api/admin/catalogo/produtos/{id}` e `DELETE /api/admin/catalogo/produtos/{id}` | Edita ou remove produto do catálogo |
| `GET /api/admin/formularios?cadeia=` | Etapas e campos do formulário de cadastro da cadeia, com o que está ativo |
| `PATCH /api/admin/formularios?cadeia=` | `{campos: []}`, define quais campos ficam ativos nos próximos cadastros |
| `GET /api/admin/creditos` | Contas com saldo de QR Codes ou acesso patrocinado, e uso no período |
| `POST /api/admin/creditos/{usuarioId}/imprimir` | Gera voucher para impressão |
| `POST /api/admin/creditos/{usuarioId}/enviar` | Envia voucher por e-mail |
| `GET /api/admin/auditoria?quem=` | Lista de ações (`quem, acao, alvo, quando`) com dados técnicos recolhidos |
| `GET /api/admin/integracoes` | Chaves e links de passagem ativos |
| `POST /api/admin/integracoes/chaves` | Cria chave de integração; valor completo só vem nessa resposta |
| `POST /api/admin/integracoes/links` | Cria link de passagem temporário para sistema parceiro |
| `DELETE /api/admin/integracoes/chaves/{id}` e `DELETE /api/admin/integracoes/links/{id}` | Revoga chave ou link |
| `GET /api/admin/arquivo` | Vendas e cobranças do fluxo de marketplace retirado em 27/07/2026, somente leitura |

---

## 8. Etapa da jornada (regra)

**Produção** (produtor, artesão, instituição, outros). Aplicar a primeira condição verdadeira, de cima para baixo:

| Etapa | Condição |
|---|---|
| 7. QR consultado | algum lote com consulta |
| 6. Emitiu QR Code | algum lote com QR |
| 5. Anexou evidências | algum lote com 2 ou mais tipos de evidência |
| 4. Registrou lote | tem lote |
| 3. Ativou a conta | `ativado_em` preenchido |
| 2. Aprovado | `aprovado_em` preenchido |
| 1. Pediu acesso | demais |

**Comprador:**

| Etapa | Condição |
|---|---|
| 4. Compra registrada | venda com QR validado |
| 3. Consulta com frequência | 3 ou mais consultas |
| 2. Consultou o primeiro QR | 1 ou mais consultas |
| 1. Cadastrado | demais |

**Próximo passo sugerido em Usuários**, em ordem de prioridade:

| Situação | Ação sugerida |
|---|---|
| Sem aprovação | Analisar pedido |
| Aprovado sem ativar | Lembrar convite |
| Fila presa | Resolver fila presa |
| Ativo sem lote | Chamar no WhatsApp |
| Lote com menos de 50% de evidências | Pedir evidências |
| Sem uso há 30 dias | Retomar contato |
| Demais casos | Ver perfil |

---

## 9. Segurança e LGPD

- Documento e telefone sempre mascarados na interface (`***.482.117-**`, `(86) 9****-4410`). A API só devolve o dado completo em endpoint próprio, que registra na `auditoria`.
- Consultas de QR não guardam IP completo, apenas cidade aproximada e hash do aparelho com rotação.
- Excluir lote é exclusão lógica: `excluido_em` com remoção definitiva após 24 h por rotina agendada.
- Toda ação do painel (aprovar, recusar, mesclar, mudar papel, excluir, restaurar, revelar dado) grava na `auditoria` com descrição legível.
- Nenhum JSON cru, caminho de arquivo ou termo técnico em inglês fora de "Ver dados técnicos".

---

## 10. Critérios de aceite

**Requisitos da cliente**
- [ ] Hoje mostra os 4 cartões (Ativação de usuários, Rastreabilidade, QR e alcance, Sistema e alertas) com big number, explicação de 1 linha, selo de situação e variação, visíveis sem rolar em 1280 px.
- [ ] Novos usuários por período em gráfico, separado por perfil.
- [ ] Ativos em 7, 30 e 40 dias calculados a partir de `eventos_uso`.
- [ ] Filtro de perfil aplicado a todos os indicadores e guardado na URL.
- [ ] Registros rastreáveis e QR Codes gerados com variação.
- [ ] Produtores que registraram: número, percentual e as duas listas.
- [ ] Consultas, consultas únicas, cidades e QRs mais consultados.
- [ ] Filas offline presas na barra superior, em Hoje, em Indicadores e em Saúde do sistema, com ação de avisar por WhatsApp.
- [ ] Toda métrica com tooltip de definição e glossário no rodapé de Indicadores.
- [ ] Quando não houver dado medido, mostrar "Medição ainda não disponível" em vez de 0%.

**Operação**
- [ ] Aprovar e recusar em 1 clique, com Desfazer por 8 s.
- [ ] Pedidos parecidos agrupados, com opção de mesclar.
- [ ] Lotes sem rolagem horizontal em 1280 px.
- [ ] "Pedir evidências" abre mensagem de WhatsApp pronta e editável.
- [ ] Lixeira de 24 h funcionando.

**Qualidade**
- [ ] Contraste WCAG 2.2 AA nos dois temas.
- [ ] Toda ação acessível por teclado, com foco visível e Esc fechando painéis.
- [ ] Estados de carregamento, vazio e erro em todas as listas.
- [ ] Layout funcional em 1280 px, 768 px e 375 px.
- [ ] Datas no formato `dd/mm/aaaa` e números no padrão brasileiro.

---

## 11. Pendências de negócio

1. Confirmar com a cliente se a janela é de **40 dias** ou 90 dias. O código deve aceitar o valor como parâmetro.
2. Validar a definição de **usuário ativo** (seção 4.1).
3. Definir a origem da **disponibilidade e taxa de erro** (ferramenta de monitoramento).

Dúvidas de interação ou visual: falar com Marcos.
