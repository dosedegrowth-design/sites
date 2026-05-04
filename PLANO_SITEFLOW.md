# PLANO SITEFLOW — Site as a Service automatizado DDG

> Documento de referência mestre. Atualizar a cada decisão importante.
> **Versão: v1.1 — 2026-04-27** (mudança: pagamento SÓ antes do go-live no domínio principal + Wizard de gosto visual substitui demos elaborados)
> Owner: Lucas Cassiano (DDG)

---

## 1. Objetivo

Construir um sistema **end-to-end** que:
1. **Raspa empresas em SP sem site** no Google Maps (nichos específicos) automaticamente
2. **Joga essa base num Supabase** + automação dispara mensagens em massa via WhatsApp Cloud API
3. **IA conduz a venda** (qualifica + apresenta valor) — cliente confirma interesse SEM pagar ainda
4. **IA Vera envia link único do Wizard** — cliente escolhe estilo visual (cor, fonte, vibe, blocos) num passo-a-passo simples
5. **Time humano DDG cria o site** usando Claude Opus 4.6/4.7 a partir do brief gerado pelo Wizard
6. **Site enviado pra aprovação** — cliente tem **3 revisões inclusas**, da 4ª em diante R$50/revisão
7. **Cliente aprova → SÓ AGORA paga** (antes do go-live no domínio principal dele)
8. **Pagamento confirmado** → sistema compra domínio + aponta DNS + SSL ativo → site live
9. **Painel central** controla tudo com visões por papel (admin / criador / financeiro)

**Premissa de venda fundamental**: cliente **só paga se gostar**. Risco-zero pra ele. DDG carrega o custo de produção até a aprovação. Diferencial competitivo gigante vs agências tradicionais que cobram 50% adiantado.

**Meta MVP**: 5-10 sites/dia entregues no piloto, escalando para 10-20/dia.

**Tese de negócio**: ticket R$700-900, alta margem, base de clientes vira lead list para upsell de outros serviços DDG (tráfego pago, Google Meu Negócio, redes).

---

## 2. Decisões Definidas (commitadas v1.0)

| Tema | Decisão |
|---|---|
| **Foco geográfico** | SP capital (expansão futura) |
| **Nichos piloto** | Barbearia (R$700) + Salão de Beleza/Estética facial (R$900) |
| **Modelo de venda** | IA Vera qualifica + envia link do Wizard (NÃO cobra ainda). Cliente preenche → time DDG cria → cliente aprova → **SÓ ENTÃO** Bruna gera cobrança Asaas. Pagamento pré-go-live no domínio principal. |
| **Modelo de entrega** | Time DDG usa Claude Opus 4.6/4.7 + brief do Wizard → gera site otimizado desktop+mobile |
| **Wizard de gosto** | Página única HTML (link unguessable) que coleta preferências visuais via passo-a-passo: paleta, fonte, vibe, blocos desejados, conteúdo essencial. Output = brief estruturado pro time DDG |
| **Demos visuais** | 3 demos completos (clássico/industrial/moderno) servem como **referência de polos estéticos** dentro do Wizard — cliente pode clicar 'quero algo assim' |
| **Escopo do site** | One-page com seções (Hero + Sobre + Serviços + Galeria + Depoimentos + Contato + Mapa) |
| **WhatsApp** | Meta Cloud API oficial (sistema interno DDG já lida com escalabilidade) |
| **Pagamento** | Cobrança SÓ após aprovação do cliente. IA Bruna gera link Asaas, cliente paga, e SÓ ENTÃO sistema migra do preview Vercel pro domínio principal do cliente. PIX/Boleto, vence em 24h. |
| **Contrato** | Documento curto (1 página) - escopo, prazo, pagamento, domínio em nome do cliente, garantia 30d |
| **Revisões** | 3 revisões inclusas. A partir da 4ª: R$50/revisão |
| **Manutenção** | NÃO há pacote mensal. Ajustes pós-entrega cobrados avulsos R$50. Cliente vira lead pra upsell DDG |
| **Domínio** | Incluído no preço, em nome do cliente. Tenta automação Registro.br API → fallback humano DDG |
| **Hospedagem** | Vercel (1 projeto por cliente, deploy isolado) |
| **Raspagem** | Google Places API oficial + filtro `website is null` (cron n8n diário) |
| **Orquestração** | n8n self-hosted (padrão DDG) |
| **Banco** | Supabase schema `siteflow` |
| **Conversa** | Chatwoot (account novo) + Meta Cloud API + agentes IA Claude |
| **Painel** | Next.js, **auth com usuários nominais e papéis** (admin/criador/financeiro), 1 painel único com visões por papel |
| **Tracking/Analytics** | NÃO incluído no produto base. GA/Tag Manager viram upsell pós-venda |

---

## 3. Arquitetura — Fluxo End-to-End (v1.1)

```
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 1 — RASPAGEM (cron n8n diário)                           │
│  Google Places API → filtro website=null → upsert siteflow.leads │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 2 — DISPARO MASSA (n8n)                                  │
│  Painel libera batch → Meta Cloud API template aprovado →        │
│  Chatwoot abre conversa → status `abordagem_enviada`             │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 3 — QUALIFICAÇÃO IA (Vera)                               │
│  Cliente responde → Vera qualifica + apresenta proposta:         │
│  "site profissional pronto em X dias, R$700, 3 revisões,         │
│  domínio incluso. SÓ PAGA SE APROVAR" → cliente aceita início    │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 4 — WIZARD DE GOSTO (link único Vercel)                  │
│  Vera envia link unguessable → cliente preenche passo-a-passo:   │
│  paleta · fonte · vibe · blocos · conteúdo essencial             │
│  → resultado salva no Supabase (siteflow.briefings)              │
│  → webhook notifica time DDG no painel                           │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 5 — CRIAÇÃO HUMANA + IA (time DDG + Claude Opus)         │
│  Painel mostra "novo site na fila" → criador abre Claude Opus    │
│  + brief Wizard → IA gera site → humano valida desktop+mobile    │
│  → push pro repo do cliente → Vercel preview gerado              │
│  → status `aguardando_aprovacao`                                 │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 6 — APROVAÇÃO + REVISÕES (Bruna)                         │
│  Bruna envia link preview pro cliente → coleta feedback          │
│  → criador aplica + redeploy → repete até aprovação              │
│  (3 inclusas, 4ª+ R$50 cobrado pela Bruna via Asaas)             │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 7 — PAGAMENTO (só aqui!)                                 │
│  Cliente aprovou → Bruna coleta dados (CPF/CNPJ, endereço) →     │
│  gera cobrança Asaas R$700/R$900 → vence 24h                     │
│  → cliente paga → webhook Asaas → status `pago`                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA 8 — DOMÍNIO + GO-LIVE                                    │
│  Sistema tenta compra Registro.br API no CPF do cliente          │
│  → aponta DNS Vercel → SSL auto → site sai do preview e vai      │
│  pro domínio principal do cliente                                │
│  → fallback humano DDG se API Registro.br falhar                 │
│  → Bruna envia mensagem de boas-vindas + manual                  │
│  → status `live`                                                 │
└──────────────────────────────────────────────────────────────────┘
```

### Fluxo simplificado
1. **Raspa → Dispara → Vera qualifica → Wizard preenche**
2. **Time cria → Cliente aprova (3 revisões)**
3. **Cliente paga → Sistema migra pro domínio dele → Live**

### Por que pagamento NO FINAL
- **Vende mais** ("só paga se gostar" remove fricção total)
- **Diferencial vs agências** (todas cobram adiantado)
- **Filtra mala automaticamente** — quem não tem grana pra pagar não chega até a aprovação porque some no caminho
- **Risco DDG é baixo**: custo de produção por site ~R$8 (APIs + ~30min humano com IA), valor recuperável se um cliente sumir
- **Pressão social** — cliente já viu o site dele pronto, é difícil não pagar

---

## 4. Papéis e Responsabilidades

### IA — Agente Vera (Vendas)
- Atende cliente que respondeu abordagem
- Qualifica (rapport + perfil + dor)
- Apresenta proposta + envia 3 demos visuais do nicho
- Trata objeções (preço, "tenho Instagram já", "vou pensar")
- Gera link de pagamento Asaas via tool
- Envia link, acompanha vencimento, lembra cliente
- Recebe webhook de pagamento confirmado → muda etapa → entrega pra Bruna

### IA — Agente Bruna (Pós-venda + Briefing)
- Assume após pagamento confirmado
- Coleta dados contratuais (CPF/CNPJ, endereço, email)
- Envia link de formulário web detalhado (briefing)
- Acompanha preenchimento, lembra se cliente esquecer
- Valida briefing (logo, fotos, textos completos)
- Move pra fila de criação no painel
- Após aprovação: envia mensagem boas-vindas + manual
- Cobra revisões extras (4ª+) via Asaas

### Humano — Criador DDG
- Recebe notificação no painel: "site pronto pra criar"
- Abre Claude Code/Opus 4.6/4.7
- Alimenta briefing + template escolhido pelo cliente
- IA gera o site
- Valida visual desktop + mobile
- Faz push pro repo do cliente
- Marca como "pronto pra revisão cliente" no painel

### Humano — Admin/Financeiro DDG
- Acompanha métricas no painel
- Resolve exceções (cliente reclamando, IA travou, pagamento problemático)
- Completa registro de domínio se API Registro.br falhar
- Toma decisões de copy/template/preço

---

## 5. Modelo de Dados (Supabase — schema `siteflow`)

```sql
-- Leads raspados
create table siteflow.leads (
  id uuid primary key default gen_random_uuid(),
  place_id text unique,
  nome_empresa text not null,
  telefone text,
  whatsapp_validado boolean default false,
  instagram text,
  endereco text,
  bairro text,
  cidade text default 'São Paulo',
  estado text default 'SP',
  cep text,
  categoria_gmb text,
  nicho text, -- 'barbearia' | 'salao_beleza' | 'estetica_facial'
  lat numeric,
  lng numeric,
  google_rating numeric,
  google_reviews_count int,
  tem_site boolean default false,
  fonte_raspagem text default 'google_places',
  status text default 'nao_abordado',
  -- nao_abordado | abordagem_enviada | sem_resposta | qualificando |
  -- proposta_enviada | demos_enviados | aguardando_pagto | pago |
  -- coletando_briefing | briefing_completo | em_construcao |
  -- aguardando_aprovacao | aprovado | live | perdido | cancelado
  abordado_em timestamptz,
  ultima_atividade_em timestamptz,
  criado_em timestamptz default now(),
  atualizado_em timestamptz default now()
);

-- Conversas (vinculação Chatwoot)
create table siteflow.conversas (
  id uuid primary key default gen_random_uuid(),
  lead_id uuid references siteflow.leads(id),
  chatwoot_conversation_id int unique,
  agente_atual text default 'vera', -- vera | bruna | humano
  ultima_mensagem_em timestamptz,
  total_mensagens int default 0,
  criado_em timestamptz default now()
);

-- Mensagens (cache local + auditoria)
create table siteflow.mensagens (
  id uuid primary key default gen_random_uuid(),
  conversa_id uuid references siteflow.conversas(id),
  chatwoot_message_id int,
  origem text, -- inbound | outbound
  agente text, -- vera | bruna | humano | cliente
  conteudo text,
  enviada_em timestamptz default now()
);

-- Clientes (dados contratuais coletados pós pagamento)
create table siteflow.clientes (
  id uuid primary key default gen_random_uuid(),
  lead_id uuid references siteflow.leads(id) unique,
  nome_completo text,
  cpf_cnpj text,
  email text,
  telefone text,
  endereco_completo text,
  cidade text,
  uf text,
  cep text,
  criado_em timestamptz default now()
);

-- Briefings (formulário web)
create table siteflow.briefings (
  id uuid primary key default gen_random_uuid(),
  cliente_id uuid references siteflow.clientes(id),
  template_escolhido_id uuid,
  nome_empresa text,
  nicho text,
  cores_pref jsonb, -- {primaria, secundaria, acento}
  logo_url text,
  fotos_urls text[],
  textos jsonb, -- {sobre, slogan, servicos[], depoimentos[]}
  contatos jsonb, -- {whatsapp, instagram, email_publico, endereco_publico}
  redes_sociais jsonb,
  dominio_desejado text,
  formulario_token text unique, -- pra link público do form
  preenchido_em timestamptz,
  validado_em timestamptz,
  criado_em timestamptz default now()
);

-- Demos crus (catálogo visual)
create table siteflow.demos (
  id uuid primary key default gen_random_uuid(),
  nicho text not null,
  nome text not null, -- 'Clássico', 'Industrial', 'Moderno', 'Acolhedor'
  slug text unique,
  preview_url text, -- link público do HTML
  cores jsonb,
  fontes jsonb,
  ativo boolean default true,
  criado_em timestamptz default now()
);

-- Sites entregues
create table siteflow.sites (
  id uuid primary key default gen_random_uuid(),
  briefing_id uuid references siteflow.briefings(id),
  demo_id uuid references siteflow.demos(id),
  criador_user_id uuid, -- quem criou (humano DDG)
  github_repo text,
  vercel_project_id text,
  vercel_preview_url text,
  vercel_prod_url text,
  dominio text,
  dominio_comprado boolean default false,
  dns_apontado boolean default false,
  ssl_ativo boolean default false,
  rounds_revisao int default 0,
  status text default 'em_construcao',
  -- em_construcao | em_revisao_cliente | aprovado | publicado
  iniciado_em timestamptz,
  aprovado_em timestamptz,
  publicado_em timestamptz,
  criado_em timestamptz default now()
);

-- Cobranças
create table siteflow.cobrancas (
  id uuid primary key default gen_random_uuid(),
  lead_id uuid references siteflow.leads(id),
  site_id uuid references siteflow.sites(id),
  tipo text, -- venda_inicial | revisao_extra | upsell
  asaas_charge_id text,
  valor numeric,
  status text default 'pendente', -- pendente | pago | vencido | estornado
  vencimento timestamptz,
  pago_em timestamptz,
  link_pagamento text,
  criado_em timestamptz default now()
);

-- Usuários do painel (DDG team)
create table siteflow.users (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  nome text,
  papel text, -- admin | criador | financeiro
  ativo boolean default true,
  criado_em timestamptz default now()
);

-- Eventos (auditoria)
create table siteflow.eventos (
  id uuid primary key default gen_random_uuid(),
  lead_id uuid references siteflow.leads(id),
  ator text, -- vera | bruna | humano:nome | sistema
  tipo text,
  payload jsonb,
  criado_em timestamptz default now()
);
```

---

## 6. Painel Interno (Next.js + Vercel)

### Auth
- Supabase Auth com email/senha + magic link
- Tabela `siteflow.users` define o papel
- Middleware Next.js redireciona conforme papel

### Visões por Papel

#### Admin (Lucas + sócios)
- Tudo: leads, kanban, sites, financeiro, config
- Métricas: conversão por etapa, ROAS WhatsApp, ticket médio, MRR upsell

#### Criador (time produção DDG)
- `/criacao` — fila de sites pra construir (status `briefing_completo`)
- Cada card: link briefing + demo escolhido + botão "iniciar criação"
- Ao clicar "iniciar": move pra `em_construcao`, atribui ao usuário
- Botão "marcar pronto pra revisão cliente": move pra `em_revisao_cliente` + dispara Bruna
- Lista de revisões pendentes (cliente pediu ajuste)
- Histórico próprio do criador

#### Financeiro (admin DDG)
- `/financeiro` — cobranças Asaas, recebimentos, vencidos, estornos
- Métricas: receita diária/mensal, ticket médio, % conversão pagamento
- Botão pra reemitir cobrança vencida

### Rotas Principais

| Rota | Papel | Função |
|---|---|---|
| `/leads` | admin | Lista raspagem com filtros, botão "disparar abordagem" individual ou batch |
| `/kanban` | admin | Board com 12 colunas (etapas), drag-and-drop manual |
| `/lead/[id]` | admin | Detalhes: dados raspados + conversa Chatwoot embedada + briefing + site |
| `/criacao` | criador, admin | Fila de sites pra criar |
| `/site/[id]` | criador, admin | Detalhes do site em construção |
| `/financeiro` | financeiro, admin | Cobranças e receita |
| `/config` | admin | Nichos, demos, templates de mensagem, papéis IA |
| `/demos` | público (link unguessable) | Galeria de demos pra cliente ver no WhatsApp |

---

## 7. Demos Crus (estratégia visual)

**Mudança v1.0**: antes os "templates" iam ser construídos só na hora de gerar o site. Agora a gente cria **demos crus visuais** que cliente vê **antes** de comprar.

### Por quê
- Cliente vê o estilo antes de fechar (reduz "não era o que eu esperava")
- IA Vera consegue mostrar valor concreto na venda
- Time DDG tem direção visual clara antes de criar
- Cliente já chega no briefing sabendo o estilo escolhido

### Formato
- HTML estático standalone (1 arquivo .html por demo)
- Tailwind via CDN
- Conteúdo placeholder plausível (ex: "Barbearia do Zé - Pinheiros")
- Imagens stock Unsplash
- Mobile-responsive
- Pasta `siteflow/demos/`
- Hospedagem: GitHub Pages quando precisar mostrar publicamente

### Catálogo Inicial (6 demos)

#### Barbearia (3 vibes)
1. **Clássico** — preto + dourado, serif editorial, fotos em preto-e-branco, tom "tradição"
2. **Industrial** — escuro, concreto, mono+sans condensada, brutalist leve, tom "oficina urbana"
3. **Moderno** — branco + verde-petróleo, sans-serif clean, muito espaço em branco, tom "premium contemporâneo"

#### Salão de Beleza / Estética Facial (3 vibes)
1. **Clássico** — preto + dourado, elegante feminino, serifs delicadas, tom "luxo"
2. **Acolhedor** — rosa pastel + beige, serifs delicadas, fotos lifestyle, tom "convidativo e quentinho"
3. **Moderno** — branco + verde-petróleo, minimalista, foco em tipografia, tom "spa contemporâneo"

### Como funciona na venda
1. Cliente diz "tenho interesse"
2. Vera envia link da galeria filtrada por nicho (ex: `https://demos.siteflow.dosedegrowth.pro/barbearia`)
3. Cliente vê os 3 estilos, escolhe um
4. Vera registra escolha + parte pra fechar pagamento
5. Pós-pagamento, Bruna usa o demo escolhido como base do briefing

---

## 8. Stack Definitiva

| Camada | Ferramenta | Status |
|---|---|---|
| Raspagem | Google Places API + n8n cron | Configurar Fase 1 |
| Validação WhatsApp | Sistema interno DDG | Reutilizar |
| Banco | Supabase schema `siteflow` | Criar Fase 1 |
| Orquestração | n8n self-hosted | Já tem |
| Conversa | Chatwoot (account novo dedicado) | Criar account Fase 1 |
| Agentes IA Vera/Bruna | Claude API (Sonnet 4.6/4.7) | Configurar Fase 1 |
| Demos crus | HTML estático + Tailwind CDN | **Em criação agora** |
| Templates definitivos | Next.js 16 + Tailwind v4 + shadcn/ui | Pós-validação Fase 0 |
| Geração assistida | Claude Opus 4.6/4.7 (uso humano DDG) | Já tem |
| Hospedagem cliente | Vercel API (1 projeto/cliente) | Configurar Fase 1 |
| Pagamento | Asaas | Já tem |
| Domínio | Registro.br API + fallback humano | Fase 2 (manual no MVP) |
| Painel | Next.js + Vercel + Supabase Auth | Construir Fase 1 |

---

## 9. Math de Negócio

### Custo por site entregue
| Item | Custo |
|---|---|
| Google Places API | ~R$0,02 |
| Meta Cloud API (1 abordagem + ~30 msgs) | ~R$0,50 |
| Claude API (Vera + Bruna conversa) | ~R$1,50 |
| Claude Opus 4.7 (criação assistida — uso humano) | ~R$5 |
| Vercel deploy isolado | R$0 (Pro absorve) |
| Asaas taxa transação PIX | ~R$0,49 |
| Domínio Registro.br | R$40 |
| **Total/site entregue** | **~R$48** |

### Custos fixos mensais
| Item | Custo |
|---|---|
| Supabase Pro | R$130 |
| Vercel Pro (multi-projeto) | R$110 |
| Apify (enriquecimento Instagram) | ~R$250 |
| Chatwoot/n8n self-hosted | já existe |
| Salário criador DDG (1 pessoa, 220 sites/mês) | a definir |
| **Total fixo (sem salário)** | **~R$490/mês** |

### Receita 220 sites/mês × R$800 médio
- **Receita bruta**: ~R$176.000/mês
- Custos variáveis (220 × R$48): R$10.560
- Custos fixos (sem salário): R$490
- WhatsApp prospecção (15k msgs marketing): ~R$4.500
- **Margem bruta antes de salário criador**: ~R$160.000/mês
- Salário criador full-time DDG (220 sites/mês = ~10/dia, possível com IA assistida): R$5-10k
- **Margem líquida estimada**: ~R$150.000/mês (~85%)

⚠️ Math depende de validar conversão real na Fase 0 (esperado 0,3-2% de cold WhatsApp B2B).

---

## 10. Roadmap de Execução

### Fase 0 — Demos visuais + Validação manual (em andamento agora)
**Objetivo**: ter os 6 demos prontos + validar conversão real

- [x] Decidir arquitetura v1.0 (este documento)
- [ ] Criar 6 demos HTML crus (3 barbearia + 3 salão)
- [ ] Subir demos no GitHub Pages
- [ ] Raspar 100 leads (50 barbearia Pinheiros/VM + 50 salão mesma região)
- [ ] Disparar 100 mensagens manuais via WhatsApp
- [ ] Conduzir conversas até 3-5 "sim eu quero" (manual)
- [ ] Fazer 2-3 sites manualmente (Claude Opus + Claude Code) usando os demos
- [ ] Decisão go/no-go com dados reais

### Fase 1 — MVP Automação (pós-validação)
**Objetivo**: 5 sites/dia assistido

- [ ] Setup Meta Cloud API + 3 templates marketing aprovados
- [ ] Schema `siteflow` no Supabase + RLS + dados iniciais (demos)
- [ ] Painel Next.js base (auth + leads + kanban + criacao + financeiro)
- [ ] n8n: cron raspagem + webhook Chatwoot + roteador Vera/Bruna
- [ ] Agente Vera (qualificação + venda + cobrança via Asaas tool)
- [ ] Agente Bruna (briefing + revisão + cobrança extras)
- [ ] Formulário web de briefing (rota pública com token)
- [ ] Integração Asaas (cobrança + webhook pagamento)
- [ ] Integração Vercel API (criar projeto por cliente)

### Fase 2 — Escala Total (pós-MVP rodando)
**Objetivo**: 10-20 sites/dia automatizado

- [ ] Compra de domínio automatizada via Registro.br API
- [ ] Apontamento DNS automático
- [ ] A/B test de copy de abordagem (3 variações Meta)
- [ ] Métricas avançadas (conversão por etapa, ROAS, CAC)
- [ ] Otimização do prompt da Vera com base em dados reais

### Fase 3 — Expansão (futuro)
- Self-service web pra cliente preencher briefing direto (sem WhatsApp)
- Mais nichos (clínica odontológica, oficina, restaurante)
- Expansão geográfica (RJ, BH, Curitiba)
- Upsell automatizado: pacote de tráfego, GMB, redes sociais
- Portal cliente (acompanha status do site dele)

---

## 11. Agentes IA — Comportamento

### Vera — Vendas (ponta-a-ponta)
**Trigger**: cliente responde abordagem Meta
**Objetivo**: levar até pagamento confirmado
**Comportamento**:
1. Apresentação curta (1x): "Oi [nome], aqui é a Vera da Dose de Growth. Vi que você tem [empresa] e ainda não tem site profissional, posso te mostrar uma proposta?"
2. Qualifica (rapport + dor): "Há quanto tempo está aberto? Como divulga hoje? Já pensou em ter site?"
3. Apresenta valor: "Site profissional pronto em poucos dias por R$700 [ou R$900], domínio incluso, 3 revisões"
4. Envia link da galeria de demos do nicho: `https://demos.siteflow.dosedegrowth.pro/barbearia`
5. Trata objeções (preço, "Instagram já basta", "vou pensar")
6. Cliente escolhe demo → Vera registra
7. Vera dispara cobrança via tool Asaas → manda link
8. Acompanha pagamento, lembra cliente se passar do prazo
9. Webhook Asaas confirma → Vera passa pra Bruna

**Tools Vera**:
- `enviar_demos_galeria(nicho)` — envia link galeria filtrada
- `gerar_cobranca_asaas(valor, descricao, vencimento)` — cria charge
- `registrar_demo_escolhido(demo_id)` — salva escolha
- `consultar_status_pagamento(charge_id)` — verifica
- `mover_para_bruna(lead_id)` — handoff

### Bruna — Pós-venda + Briefing
**Trigger**: pagamento confirmado (handoff da Vera)
**Objetivo**: coletar briefing completo + acompanhar revisões
**Comportamento**:
1. Saudação: "Pagamento confirmado, [nome]! Agora vou coletar as informações pro time criar seu site"
2. Coleta dados contratuais via perguntas (CPF/CNPJ, endereço completo, email)
3. Envia link de formulário web detalhado: `https://siteflow.dosedegrowth.pro/briefing/[token]`
4. Acompanha preenchimento (lembra se passar 24h sem mexer)
5. Valida briefing (logo enviado? fotos enviadas? textos completos?)
6. Move pra fila de criação no painel
7. Quando criador DDG marca "pronto pra revisão": Bruna envia link preview
8. Coleta feedback do cliente, passa pro criador
9. Após aprovação: envia mensagem boas-vindas + manual + status `live`
10. Se cliente pedir 4ª+ revisão: Bruna avisa cobrança R$50 + gera Asaas

**Tools Bruna**:
- `gerar_link_formulario(cliente_id)` — cria token + URL
- `verificar_briefing_completo(briefing_id)` — valida
- `mover_para_criacao(briefing_id)` — handoff pro criador
- `enviar_preview(site_id)` — envia URL preview pro cliente
- `cobrar_revisao_extra(site_id, motivo)` — gera Asaas R$50
- `marcar_aprovado(site_id)` — finaliza

---

## 12. Riscos e Mitigações

| Risco | Probabilidade | Mitigação |
|---|---|---|
| Conversão WhatsApp B2B baixa (0,3-0,8%) | Alta | Fase 0 valida com 100 disparos manuais antes de escalar |
| IA Vera fechando venda sem qualidade | Alta | Demos visuais reduzem expectativa errada; prompt agressivo + tool de "sair pra humano" se complexo |
| Meta reprovar templates de venda fria | Média | 3 variações + tom "oportunidade local" + sistema interno DDG já testado |
| Cliente recusa CPF/CNPJ pro domínio | Média | Plano B: subdomínio `ddgsites.com.br/cliente` no MVP |
| 220 sites/mês sobrecarrega criador DDG | Alta | IA Opus 4.7 acelera muito; medir tempo médio na Fase 0; contratar mais conforme volume |
| Cliente reclama da qualidade do site IA-gerado | Alta | Demos crus reduzem expectativa errada; criador humano valida antes de mandar |
| API Registro.br exigir aceite manual | Alta | Fallback humano DDG completa registro |
| Cliente não preenche o formulário web | Alta | Bruna lembra via WhatsApp; pode preencher pelo cliente se necessário |
| Fluxo handoff Vera→Bruna falha | Média | Painel mostra todos os handoffs + admin pode forçar manualmente |

---

## 13. Próximas Ações Imediatas (Fase 0)

1. **Criar 6 demos HTML** ✅ em andamento
2. **Galeria HTML index dos demos**
3. **Setup Google Places API key** (projeto GCP `dose-de-growth`)
4. **Script de raspagem standalone** (Node.js) — 100 leads em SP capital (50 barbearia + 50 salão)
5. **Disparo manual** de 100 mensagens com link da galeria de demos
6. **Validação manual** das conversas até pagamento + criação de 2-3 sites com Claude Opus
7. **Decisão go/no-go** com dados reais antes de investir em automação

---

## 14. Glossário

- **Lead**: empresa raspada que ainda não foi abordada
- **Abordagem**: primeira mensagem WhatsApp via template Meta
- **Demo**: HTML estático com estilo visual cru (3 por nicho)
- **Vera**: agente IA Claude responsável por venda ponta-a-ponta
- **Bruna**: agente IA Claude responsável por pós-venda + briefing
- **Criador**: humano DDG que constrói o site usando Claude Opus + briefing
- **Briefing**: formulário web detalhado preenchido pelo cliente após pagamento
- **Preview**: site no domínio temporário Vercel (`xxx.vercel.app`)
- **Go-live**: site no domínio próprio do cliente com SSL ativo
- **Round de revisão**: cada vez que cliente pede mudança no preview (3 inclusas, 4ª+ R$50)
- **Upsell**: venda futura de outros serviços DDG (tráfego, GMB, redes) usando a base de clientes SiteFlow

---

## 15. Changelog

### v1.0 — 2026-04-27
- **Mudança crítica**: IA agora conduz venda ponta-a-ponta (Vera) + briefing (Bruna). Tudo IA até o briefing completo.
- **Time humano DDG**: só entra na **criação do site** usando Claude Opus 4.6/4.7 + briefing + demo escolhido.
- **Demos crus**: nova estratégia. 6 demos HTML (3 barbearia + 3 salão) que cliente vê **antes** de comprar.
- **Pagamento antes do briefing**: Vera fecha venda → Asaas confirma → só então Bruna coleta dados. Reduz drasticamente trabalho perdido.
- **Briefing via formulário web**: Bruna manda link, cliente preenche estruturado.
- **Painel com papéis**: admin / criador / financeiro. Auth nominal (não senha compartilhada).
- **Domínio**: tenta automação → fallback humano DDG.

### v0.1 — 2026-04-27
- Versão inicial
- Tudo IA na conversa, time DDG não entrava no fluxo
- Briefing via WhatsApp (Bruna coletava tudo conversando)
- Sem demos crus (templates só apareciam pro cliente após "sim")

---

_Documento vivo. Toda decisão importante deve ser registrada aqui antes de ser implementada._
