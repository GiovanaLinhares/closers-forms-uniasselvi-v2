# Portal de Registro de Matrículas — Uniasselvi

## O que é este projeto

Sistema web de registro de matrículas para consultores de vendas da Uniasselvi (internos e externos HAG). Aplicação 100% estática (HTML/JS), usando **Supabase** como backend (banco PostgreSQL) e **GitHub Pages** como hospedagem.

HAG é uma empresa de call center terceirizada que vende os mesmos produtos da Uniasselvi — operação externa, usada principalmente para escalada.

---

## Arquivos do projeto

| Arquivo | Função |
|---|---|
| `index.html` | Formulário principal de registro de matrículas |
| `dashboard.html` | Painel analítico com KPIs, gráficos, filtros, edição inline e auditoria |
| `Logo_Uniasselvi.png` | Logo usada no header |
| `Base_Colaboradores_Uniasselvi.xlsx` | Base oficial de colaboradores (151 pessoas — EFETIVO + HAG) |
| `Base_Polo_Uniasselvi.xlsx` | Base oficial de polos |
| `CLAUDE.md` | Documentação do projeto |

---

## Backend — Supabase

**Projeto Supabase:** `closers-uniasselvi`
**URL:** `https://wfequjtoutpqvfyjdrvo.supabase.co`
**Anon Key:** `sb_publishable_NYXnlMO9SZc4Sg3zxPCZtw_Uto0gecQ`

### Tabela `matriculas` — colunas existentes

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | uuid | Chave primária (gerada pelo Supabase) |
| `origem_matricula` | text | Contact Center / Leo App/Web |
| `consultor` | text | Nome do(a) consultor(a) |
| `canal_consultor` | text | Interno / HAG — derivado da base de colaboradores |
| `candidato` | text | Número do contrato (apenas dígitos) |
| `data_atendimento` | date | Data do atendimento |
| `ticket` | text | Número do ticket |
| `canal_atendimento` | text | Interno / Hag |
| `modalidade` | text | Graduação EAD / Ed. Continuada — subtipo |
| `gestor` | text | Nome do(a) supervisor(a) |
| `polo` | text | Polo selecionado |
| `status` | text | Ver opções abaixo |
| `data_pagamento` | date | Preenchida quando status indica pagamento |
| `observacoes` | text | Campo livre (opcional) |
| `status_auditoria` | text | Auditoria Feita - ok / Erro / null (pendente) |
| `obs_auditoria` | text | Motivo do erro (preenchido quando status_auditoria = Erro) |
| `registrado_em` | text | Timestamp ISO gerado no front-end |

> **Coluna `id` é UUID** — nunca usar `Number()` para converter. Usar o valor string diretamente.

### Valores possíveis para `status`

| Valor | Quem define | Momento |
|---|---|---|
| `Aguardando Pagamento` | Consultor (formulário) | Registro inicial |
| `Pago na data do vencimento` | Consultor ou Analista | Pagamento no dia 10 |
| `Boleto Pago Antecipado` | Consultor (formulário) | Pagamento antes do dia 10 |
| `Pagamento – Atrasado` | Analista (dashboard) | Pagamento após o dia 10 |

> **Regra de negócio:** Todo boleto vence no dia 10 de cada mês. Pago antes = Antecipado. Pago após = Atrasado.

### Tabela `colaboradores`

| Coluna | Tipo |
|---|---|
| `nome` | text |
| `email` | text |
| `matricula` | text |
| `gestor` | text |
| `data_admissao` | date |
| `canal` | text |

> **Regra para classificar EFETIVO vs HAG:** usar o domínio do e-mail (`DS_EMAIL` na planilha).
> Domínios `@uniasselvi`, `@unicesumar` e `@vitru` → `EFETIVO` (Interno).
> Qualquer outro domínio → `HAG`.
> A coluna `DS_CONTRATO` da planilha pode estar em branco para novos colaboradores — **não usar como critério**.

### Políticas RLS configuradas (tabela `matriculas`)

```sql
CREATE POLICY "allow_select_matriculas" ON public.matriculas
FOR SELECT TO public USING (true);

CREATE POLICY "allow_insert_matriculas" ON public.matriculas
FOR INSERT TO public WITH CHECK (true);

CREATE POLICY "allow_update_matriculas" ON public.matriculas
FOR UPDATE TO public USING (true) WITH CHECK (true);

GRANT UPDATE ON public.matriculas TO anon;
```

---

## GitHub Pages

**Conta:** `vitrucontactSales`
**Repositório:** `vitrucontactSales/closers-forms-uniasselvi`
**URL pública:** `https://vitrucontactsales.github.io/closers-forms-uniasselvi/`
**Arquivos publicados:** `index.html`, `dashboard.html`, `Logo_Uniasselvi.png`

---

## Identidade Visual — Uniasselvi

Tema **dark**. Cores principais:

| Cor | Hex | Uso |
|---|---|---|
| Amarelo Uniasselvi | `#FFD400` | Destaques, CTAs, bordas de foco |
| Preto | `#000000` | Header, card-head |
| Fundo escuro | `#0F0F0F` | Background da página |
| Card | `#1A1A1A` | Cards e modais |
| Texto principal | `#F0F0F0` | — |
| Texto secundário | `#999999` | Labels, hints |

---

## Estrutura da equipe

A base oficial é o arquivo `Base_Colaboradores_Uniasselvi.xlsx` (151 colaboradores). Cargos presentes:
- **Coordenador de Vendas** — ex.: Giovana Lauer Linhares
- **Supervisor de Vendas** — ex.: Beatriz Ferrari Jordão, Sandoval Eduardo Nazareth Souza, Paulo Henrique Ferreira, Paulo Parra Misak (HAG)
- **Analista de Vendas / Qualidade** — auditam e alteram status no dashboard
- **Assistente de Vendas** — preenchem o formulário (maioria dos 151)

### Gestores no formulário (campo Gestor)
- Beatriz Ferrari Jordão
- Sandoval Eduardo Nazareth Souza

> Se precisar adicionar gestores, editar o `<select id="gestor">` no `index.html`.

---

## Campos do formulário (`index.html`)

| # | Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|---|
| 1 | Origem da Matrícula | select | Sim | Contact Center / Leo App/Web |
| 2 | Nome do(a) Consultor(a) | autocomplete | Sim | 151 nomes da base oficial — busca por digitação. Lista hardcoded em `LISTA_CONSULTORES` no JS. Exibe badge "Atendimento Interno" ou "Atendimento HAG" ao selecionar |
| 3 | Número do Contrato | text | Sim | Apenas dígitos — qualquer sequência numérica |
| 4 | Data do Atendimento | date | Sim | — |
| 5 | Ticket | text | Sim | Campo livre (ex.: TKT-12345) |
| 6 | Modalidade de Curso | select | Sim | Graduação EAD / Ed. Continuada |
| 6b | Tipo de Ed. Continuada | select | Condicional | Aparece só quando Ed. Continuada selecionada: Pós Graduação / Profissionalizante / Técnico |
| 7 | Gestor(a) | select | Sim | Apenas supervisores |
| 8 | Polo | autocomplete | Sim | Centenas de polos — busca por digitação |
| 9 | Status | select | Sim | Aguardando Pagamento / Pago na data do vencimento / Boleto Pago Antecipado |
| 9b | Data do Pagamento | date | Condicional | Aparece quando Status indica pagamento. Label muda conforme status |
| 10 | Observações | textarea | Não | Campo livre |

> **Canal de Atendimento foi removido do formulário.** O valor é derivado automaticamente do consultor selecionado: consultor EFETIVO → `canal_atendimento = 'Interno'`; consultor HAG → `canal_atendimento = 'Hag'`. A coluna `canal_atendimento` continua sendo salva no banco normalmente.

---

## Senha do sistema

| Acesso | Senha |
|---|---|
| Dashboard | `Uni@sselvi@2026` |
| Cadastrar Colaborador | `Uni@sselvi@2026` |

Proteção via `sessionStorage` — chave `dash_auth = '1'`.

---

## Chave do localStorage

```javascript
const DADOS_KEY = 'uniasselvi_closers_v1';
```

Backup local de cada registro salvo. A fonte autoritativa é o Supabase.

---

## Dashboard (`dashboard.html`)

### KPIs
- **Atuação de Boleto** (total de matrículas)
- Pagas (conta todos os status de pagamento efetivo)
- Aguardando Pagamento
- Taxa de Conversão (% Pagas / Total)

### Barra de atualização
Exibida no topo do conteúdo (abaixo do header), alinhada à direita:
- Timestamp "Última atualização: dd/mm/aaaa hh:mm" — atualizado após cada carga
- Botão **↻ Atualizar** — recarrega todos os dados do Supabase sem recarregar a página

### Filtros
Gestor · Consultor · Status · Canal · Modalidade · **Origem** · Data De/Até

### Gráficos (Chart.js 4 + chartjs-plugin-datalabels@2)
Dispostos em uma única linha, 4 colunas (grid `2fr 1fr 1fr 1fr`):
1. Matrículas por Consultor (barras horizontais, top 15)
2. Interno vs HAG (rosca)
3. Distribuição por Status (rosca — 4 categorias: Antecipado, Aguardando, Atrasado, Pago)
4. Origem da Matrícula (rosca — Contact Center / Leo App/Web)

> Os 3 gráficos de rosca exibem rótulos fixos nas fatias (valor + %) via `chartjs-plugin-datalabels`. O gráfico de barras tem `datalabels: { display: false }`.

### Tabela
Colunas (14 no total): # · Consultor(a) · **Origem (editável)** · Candidato · Data Atend. · Ticket · Canal · Polo · Modalidade · Gestor(a) · **Status (editável)** · **Data Pagamento (editável)** · **Auditoria (editável)** · Registrado Em

> Coluna **Tipo** foi removida (era redundante com Canal). A informação de canal (Interno/HAG) fica apenas na coluna Canal.

### Origem editável pelo analista
Select inline com opções: `—` / `Contact Center` / `Leo App/Web`
Salva em `origem_matricula` no Supabase. Atualiza KPIs e gráficos imediatamente.
Função: `atualizarOrigem(sel)` — mesmo padrão de `atualizarStatus`.

### Status editável pelo analista
Opções disponíveis no dashboard:
- Aguardando Pagamento
- Pago na data do vencimento
- Boleto Pago Antecipado
- Pagamento – Atrasado *(exclusivo do dashboard)*

Campo de data aparece para qualquer status de pagamento efetivo. Limpa ao voltar para "Aguardando Pagamento".

### Auditoria inline
- Select por linha: `— Pendente —` / `Auditoria Feita - ok` / `Erro`
- Ao selecionar **Erro**: textarea de observação aparece abaixo do select (salva `obs_auditoria` no Supabase via `onblur`)
- Ao mudar de Erro para outra opção: textarea some e `obs_auditoria` é limpa no banco

### Exportar CSV
Botão no header exporta todos os registros filtrados com todos os campos.

---

## Regras de negócio implementadas

- **Origem da Matrícula** obrigatória — Contact Center ou Leo App/Web
- Número do contrato aceita **qualquer sequência numérica** — letras e símbolos são bloqueados
- Verificação de duplicata: `onblur` (localStorage) + antes de salvar (Supabase)
- Status com pagamento exige Data do Pagamento — label muda conforme o status selecionado
- Polo e Consultor devem ser selecionados da lista de autocomplete (digitação livre não é aceita)
- **Regra do dia 10:** boleto vence dia 10. Antes = Antecipado. Após = Atrasado
- Acesso ao dashboard e cadastro de colaboradores protegidos por senha

### Calendário de 2ª mensalidade por edital

**Edital 1:** entrada nov–jan → fev · entrada fev → mar · mar → abr · abr → mai · mai → jun

**Edital 2:** entrada jun–jul → ago · ago → set · set → out · out → nov · nov → dez

> A partir de 26/02, o Edital 2 finaliza em outubro com segundo pagamento final em 10/11.

---

## Status do projeto

- [x] Supabase configurado — tabelas `matriculas` e `colaboradores` criadas
- [x] Colunas `origem_matricula`, `canal_consultor`, `status_auditoria`, `obs_auditoria` adicionadas à tabela `matriculas`
- [x] Políticas RLS configuradas (SELECT, INSERT, UPDATE)
- [x] Repositório GitHub criado — `vitrucontactSales/closers-forms-uniasselvi`
- [x] Formulário `index.html` — todos os campos implementados e validados
- [x] Campo "Origem da Matrícula" implementado
- [x] Campo "Consultor(a)" com autocomplete — 151 colaboradores da base oficial
- [x] Status do formulário: Aguardando / Pago na data / Boleto Pago Antecipado
- [x] Dashboard `dashboard.html` — KPIs, gráficos, filtros, edição inline de status e data
- [x] Status do analista no dashboard: + Pagamento – Atrasado
- [x] Coluna de Auditoria no dashboard (ok / Erro + observação)
- [x] Lista de polos inserida no formulário
- [x] Gráfico "Interno vs HAG" no dashboard
- [x] CSV de exportação atualizado com todos os campos (Origem, Tipo Consultor, Auditoria, Obs. Auditoria)
- [x] Campo "Canal de Atendimento" removido do formulário — derivado automaticamente do consultor selecionado
- [x] Coluna "Tipo" removida da tabela do dashboard (redundante com Canal)
- [x] Coluna Origem da Matrícula na tabela do dashboard (editável inline) + filtro
- [x] Gráfico de rosca "Origem da Matrícula" adicionado (4º gráfico)
- [x] Rótulos fixos nos gráficos de rosca (valor + %) via chartjs-plugin-datalabels
- [x] KPI "Total de Matrículas" renomeado para "Atuação de Boleto"
- [x] Barra de última atualização + botão Atualizar no topo do dashboard
- [ ] Email de notificação ao cadastrar colaborador — Supabase Edge Function + Resend
- [ ] Publicar no GitHub Pages
