# NEXOR SUÍTES — demo comercial (Pousada Mar Azul)

Demo comercial PWA. Sem backend: dados fictícios locais no navegador (Etapas 1 a 5).

```bash
npm install
npm run dev      # http://localhost:5173
npm test         # testes da camada de dados (tsx)
npm run build    # gera dist/ (deploy na Vercel; vercel.json já trata as rotas)
```

Rotas da gestão: `/`, `/solicitacoes`, `/governanca`, `/manutencao`, `/quartos`, `/reservas`, `/hospede`, `/qr-codes`.
Rota pública do hóspede (tela cheia, sem menu da gestão): `/guest/mar-azul/room/204` (troque 204 por qualquer número de quarto).

## Dados (Etapa 2)
- `src/types.ts` — entidades (Room, Guest, Reservation, GuestRequest, Task).
- `src/lib/dates.ts` — utilitários de data (hoje do demo = 10/10/2026).
- `src/data/seed.ts` — dados iniciais da Pousada Mar Azul (28 quartos, 34 reservas, 9 solicitações, 3 tarefas).
- `src/data/selectors.ts` — KPIs, timeline, alertas e verificação de integridade, tudo calculado.
- `src/data/store.ts` — store única + persistência em `localStorage` (`nexor-suites:demo`) + ações (check-in, check-out, estado do quarto, solicitações, **restaurar demonstração**).

`scripts/sandbox-*.mjs` e `scripts/test-data.ts` servem para validar sem internet no sandbox; no seu computador use os comandos acima.

## Etapa 3 — hóspede, rota pública e QR reais
- `src/views/Guest.tsx` — experiência do hóspede (`/guest/<slug>/room/<quarto>`): início com saudação e estadia, 10 acessos, "Solicitar algo" funcional. Seções navegam por `#hash` (ex.: `#wifi`); o botão voltar do navegador funciona.
- Pedidos do hóspede: `submitGuestRequest()` em `src/data/store.ts` cria uma solicitação (status `nova`, exibida como **Pendente**), que aparece na gestão (Dashboard, `/hospede`, detalhe do quarto), entra nos KPIs e persiste no `localStorage`. **Restaurar demonstração** remove esses pedidos.
- `src/lib/qr.ts` — gerador de QR próprio (sem dependências; modo byte, correção M, versões 1–10). `src/components/QrImage.tsx` desenha o QR e baixa PNG.
- `src/lib/urls.ts` — monta a URL do QR com `window.location.origin` (localhost no desenvolvimento, domínio da Vercel em produção). Nenhuma URL fixa.
- Tela **QR Codes**: um QR real por quarto (28), com número, categoria, status, rota, Visualizar e Baixar.

Testes: `npm test` (dados + pedidos + URLs). Testes de navegador do sandbox: `scripts/sandbox-e2e.mjs` (Etapas 1–2) e `scripts/sandbox-e2e-etapa3.mjs` (Etapa 3, inclui leitura do QR com leitor real).

## Etapa 4 — operação (solicitações, governança, manutenção, check-in/out)
- **Solicitações** (`/solicitacoes`): pedidos dos hóspedes com status Pendente, Em andamento e Concluído. A mudança aparece no Dashboard e na página do hóspede. Pedido de manutenção pode virar ordem de manutenção.
- **Governança** (`/governanca`): Checkout → Limpeza pendente → Em limpeza → Limpo → (Inspeção, opcional) → Livre. Liberar o quarto só é permitido a partir de Limpo ou Inspeção.
- **Manutenção** (`/manutencao`): Aberto → Em atendimento → Resolvido. Quarto livre com ordem aberta fica em Manutenção; ao resolver, volta a Livre.
- **Check-in / check-out** (Reservas e detalhe do quarto): Reserva → Hospedado → Quarto Ocupado; Hospedado → Check-out → Quarto em Limpeza.
- **Dashboard**: contadores de solicitações, limpeza, manutenção, check-ins e check-outs e atividades recentes, todos calculados a partir da store.
- Atividades ficam em `state.activity` (store). Dados salvos pela Etapa 3 são migrados sem perder pedidos.
- Regras de transição e contadores: `src/data/store.ts` (`setHousekeeping`, `setMaintenance`, `openMaintenance`, `requestToMaintenance`) e `src/data/selectors.ts` (`operations`, `recentActivity`, `guestOfRoom`).

Testes: `npm test` (dados, incluindo a Etapa 4). Navegador do sandbox: `scripts/sandbox-e2e-etapa4.mjs` (fluxos, persistência, reset, hóspede, mobile).

## Etapa 5 — PWA, acabamento e Vercel
- **PWA**: `public/manifest.webmanifest` (nome e nome curto "Nexor Suítes", `display: standalone`, cores do tema, ícones oficiais 192/512 e maskable 512 em `public/brand/`). `public/sw.js` é o service worker; `src/lib/pwa.ts` registra-o (fora do `vite dev`).
- **Cache do service worker**: navegação com fallback para o shell em cache; assets do mesmo domínio com cache imediato. A store continua no localStorage, então a interface abre offline depois de uma primeira visita.
- **Instalação**: botão discreto "Instalar app" (sidebar e cabeçalho do Dashboard), visível apenas quando o navegador oferece instalação. Não há popup.
- **Demo**: indicador "Demo" na barra lateral. "Restaurar demonstração" volta ao estado inicial.
- **Manutenção**: ordens criadas a partir de solicitações mostram "Origem: Solicitação do hóspede" com hóspede e quarto.
- **Vercel**: `vercel.json` declara framework Vite, `npm run build`, saída `dist`, rewrites SPA (rotas diretas e `/guest/<slug>/room/<n>`) e cabeçalhos para `sw.js` (sem cache) e o manifest.
- `npm run typecheck` executa `tsc --noEmit` isoladamente.

Testes: `npm test` (dados) e `scripts/sandbox-e2e-etapa5.mjs` (PWA, offline, rotas diretas, fluxos, mobile). As etapas 3 e 4 têm seus próprios scripts e continuam passando.
