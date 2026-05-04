# SiteFlow — Wizard de Briefing Visual

Wizard standalone (HTML único + Tailwind via CDN) que coleta o briefing visual do cliente após a IA Vera enviar o link no WhatsApp.

## Como funciona

1. IA Vera qualifica e envia link único pelo WhatsApp: `https://wizard.siteflow.com.br/?token=abc123`
2. Cliente preenche 7 perguntas em ~2 minutos
3. Output (JSON estruturado) é enviado pra Supabase / webhook n8n
4. Time DDG recebe notificação no painel e cria o site com Claude Opus

## Estrutura

```
wizard/
├── index.html    # Wizard único (Tailwind CDN, sem build)
└── README.md     # este arquivo
```

## Output esperado (payload do `finalizar()`)

```json
{
  "nicho": "barbearia",
  "vibe": "industrial",
  "paleta": "escuro-laranja",
  "fonte": "sans-bold",
  "secoes": ["hero", "servicos", "galeria", "contato"],
  "logo": "sim",
  "fotos": "muitas",
  "nomeEmpresa": "Barbearia do Zé",
  "cidadeBairro": "Pinheiros, São Paulo",
  "slogan": "Tradição que atravessa gerações",
  "whatsappNegocio": "(11) 9 8765-4321",
  "instagram": "barbeariadoze",
  "endereco": "Rua dos Pinheiros, 1234, Pinheiros, SP",
  "dominio": "barbeariadoze",
  "referencias": "Gosto do estilo da Norte Barber...",
  "submitted_at": "2026-04-27T14:30:00.000Z",
  "token": "abc123"
}
```

## Testar localmente

```bash
# Opção 1: abrir direto no browser
open index.html

# Opção 2: serve via http (recomendado pra testar links cross-origin)
npx serve .
# acessa http://localhost:3000
```

## Deploy

### GitHub Pages (MVP)
```bash
# Cria repo público
gh repo create dosedegrowth-design/siteflow-wizard --public --source=. --push
# Ativa GH Pages no settings → branch main → /
# URL: https://dosedegrowth-design.github.io/siteflow-wizard/
```

### Vercel (recomendado pra prod)
```bash
vercel
# Domínio custom: wizard.siteflow.dosedegrowth.pro ou wizard.dosedegrowth.com.br
```

## TODO de integração (Fase 1)

- [ ] Substituir `console.log` no `finalizar()` por POST pra webhook n8n / Edge Function Supabase
- [ ] Salvar progresso parcial no localStorage (cliente fecha aba e volta)
- [ ] Validar token na URL (rejeitar links inválidos/expirados)
- [ ] Adicionar campo "categoria customizada" quando nicho = outro
- [ ] Upload de logo direto no wizard (Supabase Storage)
- [ ] Upload múltiplo de fotos (drag-and-drop)
- [ ] Verificação automática de disponibilidade do domínio (Registro.br API)
- [ ] Resumo final em PDF auto-gerado pra cliente baixar
- [ ] Tracking analytics (qual etapa cliente abandona mais)

## Decisões de design

- **Premium feel**: tipografia Fraunces (serif variável) + Inter sans + JetBrains Mono pra metadata
- **Minimalismo intencional**: papel cor `#fdfcf8`, escuro `#0a0a0a`, accent `#ff5c1f` (laranja DDG)
- **Cards com elevação**: hover suave + border collapse pra estado selecionado (preto sólido)
- **Microinterações**: progress bar animada, fade-up entre etapas, accent dot pulsando
- **Mobile-first**: validado em 360px+, header sticky, nav buttons fixos no bottom
- **Sem framework**: HTML puro + Tailwind CDN. Zero build, deploy em qualquer lugar
