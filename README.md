# ClubMap — site publicado

App de mapa de mesas para casas noturnas: planta da sala em tempo real, reservas, check-in e sincronização entre a receção e os tablets da sala.

> Este repositório contém apenas o **site publicado** (resultado compilado). O código-fonte vive noutro repositório (ver "Arquitetura").

## O que a app faz (produção)

**Mapa da sala** — planta do club desenhada em SVG: mesas (redondas, ovais, quadradas, retangulares, VIP, sofás em U), cabine de DJ, luzes de pista, portas, portões, paredes e zonas nomeadas. Cada mesa mostra cadeiras, lugares e o estado por cor: **livre** (verde), **reservada** (âmbar), **ocupada** (vermelho). No topo, contadores em tempo real de mesas e cadeiras por estado.

**Serviço (a noite a acontecer)** — tocar numa mesa abre a ficha: criar reserva (nome, telefone, nº de cadeiras a ocupar, hora prevista, nota/consumo mínimo), **check-in** quando o cliente chega, **walk-in** direto, guardar alterações, cancelar ou libertar a mesa. Lista de reservas da noite ordenada por hora. "Nova noite" limpa tudo para recomeçar.

**Editor de planta** (admin/gestor) — pressão longa num item prende a ferramenta (mover / rodar 15° / ampliar); modo "Vários" move grupos inteiros; criar, duplicar e eliminar mesas e zonas; ajustar dimensões do club. O staff não edita a planta.

**Contas e papéis** — login por email/senha (Firebase Auth). Três papéis: **admin** (regista casas, gera convites, espreita qualquer casa), **gestor** (edita a planta da sua casa) e **staff** (só serviço). Entrada de novos utilizadores por **código de convite** (G-XXXXXX gestor, S-XXXXXX staff). Regras de acesso impostas no servidor.

**Sincronização em tempo real** — o estado da casa vive no Firestore; qualquer alteração num aparelho (tablet da sala, telefone da receção) aparece nos outros ao segundo. Cache local para arrancar offline; as escritas têm debounce para poupar tráfego.

**Distribuição** — o mesmo código gera a **PWA** (este site, instalável no telemóvel/PC, com cache offline via service worker) e o **APK Android** para os tablets da sala.

## Arquitetura (3 peças)

```
Código-fonte (React Native + RN Web)     →  repo privado club-map-mobile
   ├─ gera o APK Android (tablets da sala)
   └─ gera o bundle web (vite) ──────────→  ESTE repo (site publicado)
                                                  │  git push site main
                                                  ▼
                                      clubmapmz.github.io (GitHub Pages)

Dados (contas, casas, plantas, reservas) →  Firebase/Firestore (nuvem)
```

- O site é uma PWA: `index.html` + `assets/index-*.js` (bundle React Native Web) + `sw.js` (service worker) + `manifest.json`.
- Os dados nunca estão neste repo — chegam do Firebase quando o utilizador entra.

## Estrutura deste repo

| Caminho | O que é |
|---|---|
| `index.html` | Entrada da app web (carrega o bundle) |
| `assets/index-*.js` | Bundle compilado (React Native Web via Vite) |
| `sw.js`, `manifest.json`, `icon-*.png` | PWA: cache offline, instalação, ícones |
| `demo/` | Demonstração comercial standalone (HTML puro, dados locais, expira sozinha — ver `DEMO_EXPIRA` no topo do ficheiro) |
| `.nojekyll` | Desativa o Jekyll no GitHub Pages |

## Remotes (IMPORTANTE)

| Remote | Repositório | Papel |
|---|---|---|
| `origin` | `CleveChambule/club-map-web` | Backup do site |
| `site` | `clubmapmz/clubmapmz.github.io` | **O que está no ar** |

**Publicar = `git push site main`.** O `git push` sozinho vai só para o backup e o site não muda.

Este repo é público: nunca commitar propostas, contratos, chaves ou documentos internos (o `.gitignore` já bloqueia os conhecidos).

## Atualizar o site (fluxo completo)

1. Editar o código no projeto fonte (`App.tsx` / `sync.ts`).
2. Compilar o web: `npx vite build` (sai em `web-dist/`).
3. Copiar `web-dist/index.html` e `web-dist/assets/` para este repo (ajustar o nome do bundle no `index.html` se mudou).
4. `git add` → `git commit` → `git push origin main` → `git push site main`.
5. Na app, o botão "Atualizar aplicação" limpa o cache do service worker nos aparelhos.

## Montar noutro PC (do zero)

1. Clonar o fonte: `git clone https://github.com/CleveChambule/club-map-mobile.git C:\ClubMap`
   — usar um **caminho curto** (ex.: `C:\ClubMap`): os builds Android falham em caminhos longos.
2. Clonar este repo: `git clone https://github.com/CleveChambule/club-map-web.git` e adicionar o remote do site:
   `git remote add site https://github.com/clubmapmz/clubmapmz.github.io.git`
3. No fonte: `npm install` (Node >= 22.11).
4. Web: `npx vite build` · Android: `npx react-native run-android` (ou `gradlew assembleRelease` em `android/`).
5. A assinatura do APK usa uma keystore que **não está em nenhum repo** — está guardada localmente com as instruções (procurar `CHAVE-ASSINATURA-LEIA-ME.txt` no PC antigo antes de migrar!).
6. Ficheiros de trabalho internos (estado do projeto, propostas) também não vêm nos repos — copiar manualmente do PC antigo (`ESTADO.md` e PDFs nesta pasta).

## Armadilha conhecida

Nunca usar `Get-Content`/`Set-Content` do PowerShell em ficheiros de código deste projeto — corrompe acentos e emojis (UTF-8). Editar sempre com editor normal.
