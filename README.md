# SiteFlow DDG

Sistema automatizado de prospecção fria + criação e entrega de sites one-page para PMEs em SP.

> **Documento de referência mestre**: [PLANO_SITEFLOW.md](./PLANO_SITEFLOW.md)
> **Status atual**: Fase 0 — Validação manual

## Visão geral

1. Raspa empresas em SP sem site no Google Maps (barbearia, salão, estética)
2. Aborda via WhatsApp Cloud API (Meta)
3. Qualifica via agente IA Claude (Vera)
4. Coleta briefing (Bruna)
5. Gera site one-page com template + Claude
6. Deploy Vercel + domínio próprio do cliente
7. Cobra via Asaas após aprovação

## Tickets

| Nicho | Ticket |
|---|---|
| Barbearia | R$700 |
| Salão / Estética facial | R$900 |

Inclui: site one-page + 3 ajustes + 1º ano de domínio.

Ajuste extra: R$50 por ajuste a partir do 4º.

## Stack

- Next.js 16 + Tailwind v4 + shadcn/ui (templates + painel)
- Supabase (schema `siteflow`)
- Chatwoot + n8n + Claude API (agentes)
- Meta Cloud API (WhatsApp prospecção)
- Vercel (1 projeto por cliente entregue)
- Asaas (cobrança)
- Registro.br (domínio em nome do cliente)

## Estrutura do repositório

```
siteflow/
├── PLANO_SITEFLOW.md       # documento mestre (LER PRIMEIRO)
├── README.md               # este arquivo
├── wizard/                 # Wizard HTML standalone (briefing visual)
│   ├── index.html
│   └── README.md
├── demos/                  # Demos visuais de referência
│   ├── barbearia/
│   │   ├── classico.html
│   │   ├── industrial.html
│   │   └── moderno.html
│   └── salao/              # (próximas leva)
└── (a ser criado Fase 1)
    ├── painel/             # Next.js painel interno
    ├── templates/          # templates Next.js dos sites
    ├── n8n/                # workflows JSON exportados
    ├── scripts/            # scripts standalone (raspagem Fase 0, etc)
    └── docs/               # contratos, scripts de agentes, copies
```

## Como ver os HTMLs

```bash
# abrir wizard
open wizard/index.html

# abrir demos de referência
open demos/barbearia/classico.html
open demos/barbearia/industrial.html
open demos/barbearia/moderno.html
```

## Fase 0 — Próximas ações

1. Setup Google Places API key
2. Script de raspagem: 100 barbearias em Pinheiros/Vila Mariana
3. Disparo manual de 100 mensagens via WhatsApp
4. Conduzir 3-5 conversas até "sim eu quero"
5. Fazer 2-3 sites manualmente (Claude Code + v0/Lovable)
6. Decisão go/no-go com dados reais

Detalhes em [PLANO_SITEFLOW.md § 8](./PLANO_SITEFLOW.md#8-roadmap-de-execução).
