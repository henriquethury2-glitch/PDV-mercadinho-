# PDV Mercadinho

Sistema completo de Ponto de Venda para comércio de pequeno porte.

**Stack:** React 18 + TypeScript + Vite + Supabase + Tailwind CSS

---

## Estrutura do projeto

```
pdv/
├── src/
│   ├── components/
│   │   ├── auth/       AuthGuard
│   │   └── layout/     Layout, Sidebar, Topbar
│   ├── lib/
│   │   ├── supabase.ts cliente tipado
│   │   └── format.ts   formatadores (brl, dates, masks)
│   ├── pages/          uma página por módulo
│   ├── store/
│   │   ├── authStore   Zustand + persistência
│   │   └── cartStore   carrinho PDV
│   └── types/
│       └── database.ts todos os tipos + Database
├── supabase/
│   ├── migrations/
│   │   ├── 001_schema.sql   tabelas, RLS, funções SQL
│   │   └── 002_storage.sql  bucket de fotos
│   └── functions/
│       ├── gerar-pix/       Edge Function PIX (Efí)
│       ├── webhook-pix/     Webhook de confirmação PIX
│       └── emitir-nfce/     Edge Function NFC-e (Focus NFe)
└── .env.example
```

---

## Setup — passo a passo

### 1. Criar projeto no Supabase

1. Acesse [supabase.com](https://supabase.com) → New Project
2. Guarde: **Project URL** e **anon key** (Settings → API)

### 2. Executar migrations

No **SQL Editor** do Supabase, execute em ordem:

```
supabase/migrations/001_schema.sql
supabase/migrations/002_storage.sql
```

> Ou via CLI: `supabase db push`

### 3. Configurar variáveis de ambiente

```bash
cp .env.example .env
```

Edite `.env`:

```env
VITE_SUPABASE_URL=https://xxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...

# PIX — preencha quando tiver provedor definido
VITE_PIX_CHAVE=sua@chave.pix

# NFC-e — preencha com dados reais
VITE_NFCE_TOKEN=token_focus_nfe
VITE_NFCE_BASE_URL=https://homologacao.focusnfe.com.br
VITE_EMPRESA_CNPJ=00000000000000
VITE_EMPRESA_UF=AM
```

### 4. Instalar e rodar

```bash
npm install
npm run dev
```

Acesse: `http://localhost:5173`

### 5. Criar o primeiro usuário admin

No SQL Editor do Supabase:

```sql
-- Crie o usuário via Auth > Users > Invite user ou:
-- Depois de criar via UI, promova para admin:
update profiles set role = 'admin' where email = 'seu@email.com';
```

Ou na tela de login, clique em "Cadastro de funcionário" (Settings) após o primeiro login.

---

## Edge Functions (opcional)

### PIX real (Efí/Gerencianet)

```bash
supabase functions deploy gerar-pix
supabase functions deploy webhook-pix

# Secrets da função
supabase secrets set PIX_CLIENT_ID=xxx
supabase secrets set PIX_CLIENT_SECRET=xxx
supabase secrets set PIX_BASE_URL=https://pix.api.efipay.com.br
supabase secrets set PIX_CHAVE=sua@chave.pix
```

Configure o webhook do seu provedor para:
`https://<project>.supabase.co/functions/v1/webhook-pix`

### NFC-e real (Focus NFe)

```bash
supabase functions deploy emitir-nfce

supabase secrets set NFCE_TOKEN=token_focus
supabase secrets set NFCE_BASE_URL=https://homologacao.focusnfe.com.br
supabase secrets set EMPRESA_CNPJ=00000000000000
supabase secrets set EMPRESA_UF=AM
```

> ⚠️ Emissão real de NFC-e requer: certificado digital A1, credenciamento na SEFAZ do estado, regime tributário configurado no Focus NFe e dados fiscais corretos em cada produto (NCM, CSOSN, CFOP).

---

## Módulos entregues

| Módulo | Status |
|---|---|
| Login + recuperação de senha | ✅ Supabase Auth |
| Dashboard com gráficos | ✅ dados reais |
| Caixa PDV — busca, carrinho, checkout | ✅ completo |
| Formas de pagamento | ✅ Dinheiro/Crédito/Débito + PIX adapter |
| Cadastro de produtos com foto | ✅ Supabase Storage |
| Campos fiscais por produto | ✅ NCM, CSOSN, CFOP, origem |
| Controle de estoque | ✅ movimentações transacionais |
| Financeiro — contas a pagar/receber | ✅ |
| Relatórios + exportação CSV | ✅ |
| Clientes e fornecedores | ✅ |
| Gerenciamento de usuários (admin) | ✅ |
| RLS por perfil (admin/caixa) | ✅ |
| PIX — geração QR + polling confirmação | ✅ Edge Function (precisa de credenciais) |
| NFC-e — emissão via Focus NFe | ✅ Edge Function (precisa de cert. digital) |
| Funções SQL transacionais | ✅ finalizar_venda, cancelar_venda |
| Audit log de operações | ✅ |

---

## Permissões por perfil

| Ação | Admin | Caixa |
|---|---|---|
| Dashboard | ✅ | ✅ |
| Caixa PDV | ✅ | ✅ |
| Cadastrar/editar produtos | ✅ | ❌ |
| Movimentação de estoque | ✅ | ✅ |
| Financeiro | ✅ | leitura |
| Relatórios | ✅ | ❌ |
| Configurações / usuários | ✅ | ❌ |
| Cancelar venda | ✅ | próprias |

---

## Próximas etapas sugeridas

1. **PIX**: definir provedor → configurar credenciais → testar em homologação
2. **NFC-e**: obter certificado digital → credenciar no Focus NFe homologação → testar emissão
3. **Leitor de código de barras**: integrar biblioteca `@zxing/browser` para câmera ou HID
4. **Impressão de comprovante**: `window.print()` já está presente no caixa — criar CSS `@media print`
5. **Fechamento de caixa**: página de fechamento com totais por forma de pagamento
