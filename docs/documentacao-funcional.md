# Painel Administrativo · Rastro e Voz

Documentação do protótipo navegável do painel da equipe.
Versão de 15/09/2026. Autor: Marcos Caridade (UX/UI).

Protótipo: https://claude.ai/artifact/NRXcWaWNGeSRQoyn3L3XkQ

---

## 1. Objetivo

O painel serve para a equipe monitorar a rede e atender produtores, artesãos, instituições e compradores com eficiência.

Ele responde às quatro perguntas definidas pela cliente:

1. Estamos conseguindo trazer e ativar usuários?
2. A rastreabilidade está sendo realmente utilizada?
3. Os QR Codes estão gerando alcance?
4. O sistema está funcionando bem?

## 2. Requisitos da cliente e onde são atendidos

| Requisito | Onde aparece |
|---|---|
| Novos usuários por período | Hoje (cartão Ativação de usuários) e Indicadores, seção 1, em gráfico por perfil |
| Usuários ativos em 7, 30 e 40 dias | Indicadores, seção 1: três cartões, gráfico de evolução e tabela por perfil |
| Separação por perfil | Filtro global em Hoje e Indicadores, tabela por perfil e filtro em Usuários |
| Registros rastreáveis | Indicadores, seção 2, e tela Lotes |
| QR Codes gerados | Indicadores, seção 3, e coluna QR Code em Lotes |
| Produtores que efetivamente registraram | Indicadores, seção 2: número, percentual e listas de quem registrou e de quem não registrou |
| Filas offline presas | Barra superior, pendências de Hoje, Indicadores (seção 4) e Saúde do sistema |

## 3. Arquitetura de informação

**Rotina**
- Hoje
- Acessos (Solicitações, Convites, Pessoas e contas)
- Usuários (visão 360°)
- Lotes
- Atendimento

**Monitoramento**
- Indicadores
- Saúde do sistema

**Mais**
- Configurações (Operação, Técnico, Histórico)

Conta, senha, verificação em duas etapas, tema e Sair ficam no menu do usuário, no rodapé da barra lateral. A busca global com Ctrl ou Cmd + K fica na barra superior.

## 4. Telas

### 4.1 Hoje

- **Filtros globais:** perfil (seleção múltipla) e período (7, 30, 40, 90 dias e 12 meses).
- **Resumo da rede:** quatro cartões, cada um com título curto, big number, uma linha de explicação, selo de situação e variação em relação ao período anterior.

| Cartão | Big number | Explicação |
|---|---|---|
| Ativação de usuários | Novos usuários no período | Percentual dos aprovados que ativaram a conta |
| Rastreabilidade | % dos produtores ativos que registraram lote | Quantidade absoluta (ex.: 9 de 13) |
| QR e alcance | Consultas no período | Quantidade de QR Codes e de cidades |
| Sistema e alertas | Filas offline presas | Disponibilidade em 30 dias |

- **Situação dos cartões:** Bom, Atenção ou Crítico. A regra fica no ícone de informação.
- **O que precisa de você:** pendências ordenadas por impacto (filas presas, solicitações, chamados, lotes sem evidência, convites não abertos, contas sem lote), com ação no próprio item e opção de adiar.
- **Meta do MMP:** usuários ativos (meta 100) e vendas com QR validado (meta 5), com projeção.
- **Ritmo da equipe:** tempo até decidir solicitações, primeira resposta em chamados e lotes prontos para auditoria.
- **Acontecendo na rede:** feed de consultas de QR, lotes e ativações.

### 4.2 Indicadores

Há quatro seções, uma por pergunta, com atalhos fixos no topo. Cada seção abre com a pergunta e uma frase de resposta automática.

**Seção 1 · Ativação de usuários**
- Novos usuários, ativos em 7, 30 e 40 dias e taxa de ativação
- Novos usuários por período, empilhados por perfil
- Evolução dos ativos em 7, 30 e 40 dias
- Tabela por perfil
- Funil: pediram acesso, aprovados, ativaram, registraram lote, emitiram QR, QR consultado

**Seção 2 · Rastreabilidade**
- Registros rastreáveis, produtores que registraram, lotes por quem registra, prontos para auditoria e registros sem internet
- Lotes por período e cadeia
- Listas de quem registrou e de quem está ativo sem registro (com WhatsApp)
- Qualidade das evidências (GPS, foto, áudio, documento)
- Territórios com mais registros

**Seção 3 · QR e alcance**
- QR Codes gerados, consultas, consultas únicas, percentual de QRs consultados e consultas por QR
- Evolução de consultas e de QRs gerados
- Cidades de consulta, com percentual fora do Piauí
- QR Codes mais consultados
- Compradores cadastrados comparados ao público geral

**Seção 4 · Sistema e alertas**
- Disponibilidade, taxa de erro, tempo de resposta, sincronização offline e filas presas
- Lista das filas presas com atalho para resolver

O glossário de métricas fica no rodapé da tela. Há também um botão para exportar o relatório.

### 4.3 Acessos

- **Solicitações:** filas "Prontas para decidir", "Esperando e-mail" e "Decididas".
  - Aprovar funciona com atualização otimista e botão Desfazer por 8 segundos.
  - Recusar abre uma janela com motivos e mensagem editável.
  - Pedidos parecidos aparecem agrupados, com opção de mesclar.
  - Há seleção múltipla e atalhos J, K, A e R.
  - O painel lateral mostra documento e telefone mascarados, e revelar esses dados registra na auditoria.
- **Convites:** andamento Enviado → Aberto → Ativado, com ação conforme o estado (lembrar, copiar link, gerar novo).
- **Pessoas e contas:** papel com confirmação, Código Rastro, lotes, último uso e menu de ações.

### 4.4 Usuários (360°)

- Filtro por perfil com contagem.
- Filtros de jornada: aprovado sem ativar, ativou sem lote, lote sem evidências, QR sem consulta e sem uso há 7, 30 ou 40 dias.
- Coluna "Próximo passo" com a ação sugerida.
- Perfil completo com:
  - jornada visual (7 etapas para produção, 4 para compradores)
  - linha do tempo
  - notas da equipe
  - chamados
  - celulares e situação da sincronização

### 4.5 Lotes

- Filtros: todos, prontos, faltam evidências, sem QR, duplicados, cidade, cadeia, com ou sem internet e período. Os filtros ativos aparecem como chips removíveis.
- Tabela sem rolagem horizontal, com ícones de evidência, selo de prontidão e consultas do QR.
- Painel lateral com:
  - prévia das evidências
  - alcance do QR por cidade
  - ação "Pedir evidências ao produtor" por WhatsApp
- Lixeira de 24 horas com opção de restaurar.

### 4.6 Atendimento

- Indicadores de chamados abertos, primeira resposta, tempo até resolver e satisfação.
- Três colunas: lista de chamados, conversa e contexto do usuário.
- Na conversa:
  - respostas rápidas
  - nota interna
  - sugestão de resposta marcada para revisão
  - escolha do canal
  - situação e responsável
- O contexto do usuário mostra a fila offline presa, quando houver.

### 4.7 Saúde do sistema

- Situação em uma frase e indicadores principais.
- Tabela de filas offline presas com:
  - motivo em linguagem simples
  - ação "Avisar por WhatsApp"
  - chamado técnico
  - opção de marcar como resolvido
  - histórico de tentativas
- Gráfico de 30 dias, incidentes, experiência de cadastro e detalhes técnicos recolhidos.

### 4.8 Configurações

- **Operação:** Catálogo, Formulários de cadastro, Créditos e vouchers
- **Técnico:** Auditoria de ações (em linguagem legível, com dados técnicos recolhidos), Integrações e links de passagem
- **Histórico:** Arquivo do marketplace (somente leitura)

## 5. Definições das métricas

| Métrica | Definição |
|---|---|
| Usuário novo | Conta ativada dentro do período |
| Usuário ativo em N dias | Fez ao menos uma ação relevante na janela: entrar no app, registrar ou editar lote, anexar evidência, emitir QR ou abrir chamado. Atualização automática não conta |
| Perfil | Produtor(a), Artesã(o), Instituição, Comprador(a), Outros |
| Produtor que registrou | Usuário de produção com pelo menos um lote no período |
| Registro rastreável | Lote salvo e sincronizado com o servidor |
| QR gerado | QR Code emitido para um lote |
| Consulta de QR | Abertura da página pública do QR. A consulta única conta uma vez por aparelho |
| Fila offline presa | Celular com registros pendentes há mais de 24 h, mesmo tendo se conectado, ou com falhas repetidas |

## 6. Regras de situação dos cartões

| Cartão | Bom | Atenção | Crítico |
|---|---|---|---|
| Ativação de usuários | Mais novos que no período anterior e ativação acima de 60% | Demais casos | Nenhuma ativação no período |
| Rastreabilidade | 50% ou mais registraram | Entre 25% e 50% | Abaixo de 25% |
| QR e alcance | Consultas crescendo e mais de 50% dos QRs já consultados | Demais casos | Nenhuma consulta |
| Sistema e alertas | Disponibilidade acima de 99% e nenhuma fila presa | Há filas presas | Disponibilidade abaixo de 99% ou 5 ou mais filas presas |

## 7. Diretrizes de UX e UI aplicadas

- Um big number por cartão, com explicação de uma linha e a pergunta completa em tooltip.
- A situação é indicada por selo e barra lateral, nunca só por cor.
- A ação fica junto da informação, e ações reversíveis têm Desfazer em vez de confirmação.
- Dados pessoais aparecem mascarados, e revelar registra na auditoria (LGPD).
- Linguagem simples em português do Brasil, sem termos técnicos fora das áreas técnicas.
- Tema claro e escuro, versão para tablet e celular, navegação por teclado e respeito à preferência de movimento reduzido.
- Estados de carregamento, vazio e confirmação em todas as telas.

## 8. Pendências para validar com a cliente

1. Confirmar se a janela de 40 dias está correta ou se o esperado era 90 dias.
2. Validar a definição de usuário ativo.
3. Verificar com a Quemia se já existe registro de eventos de uso, de consultas de QR com localização e do estado das filas de sincronização. Os números do protótipo são dados de exemplo.

## 9. Dados necessários para a versão real

- Usuários com perfil
- Solicitações e convites
- Lotes e evidências
- QR Codes
- Consultas de QR (data, cidade, estado, país, aparelho, comprador)
- Eventos de uso (para ativos em 7, 30 e 40 dias)
- Dispositivos e filas de sincronização
- Chamados e mensagens
- Notas
- Incidentes e auditoria
