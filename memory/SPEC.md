# Bloco do Beco — Mapa de Talentos do Jardim Ibirapuera (SPEC)

## What it does
Community site for the collective **Bloco do Beco** to map and measure local talents (musicians, seamstresses,
artisans, visual artists, dance/theatre, gastronomy) in Jardim Ibirapuera, São Paulo, and connect residents to the
activities/events they offer. UI in pt-BR.

## Pages (frontend/src/pages)
- `/` Home: hero + public summary counters + map preview, category chips, AI assistant "Encontre um talento" (Claude), latest talents, activities teaser
- `/talentos` catalog: search + CASCADING filters Categoria → Especialidade → Tempo de atuação → Faixa etária (options/counts depend on previous)
- `/talento/:id` profile (approved = public; pending/rejected visible only to owner + admin, with status banner)
- `/atividades` activities & events: DATE filter (Qualquer data/Hoje/Próximos 7 dias/Este mês/Todas as próximas/período custom) then cascade Categoria → Tipo → Valor → Talento. Date filter only keeps activities with a `date`.
- `/mapa` Leaflet/OSM map with cascade Categoria → Especialidade → Oferece → Tempo de atuação
- `/cadastrar` requires Emergent Google sign-in; form is prefilled with the Google name + picture; submitted profiles get status `pending`. All "Cadastrar talento" CTAs (RegisterCta) send anonymous visitors straight to Google sign-in.
- `/meus-perfis` artist's own profiles with status; edit (rejected → back to pending) / delete
- `/admin` (password or Google admin email): tabs Aprovações (approve/reject), Talentos (status filter, edit, delete, CSV), Painel de dados (cascade Categoria → Gênero → Faixa etária → Tempo de atuação + charts + AI report). Public dashboard page was removed.

## Data model
`talents` (string uuid `id`): name, category, specialty, bio, gender, age_range, experience, address, lat, lng, phone,
instagram, photo, activities[{id,title,type,description,date(YYYY-MM-DD|null),schedule,price}],
status(pending|approved|rejected), owner_id (users.user_id, null for seed), owner_email (admin-only in responses), created_at.
`users` {user_id,email,name,picture} · `user_sessions` {user_id,session_token,expires_at(TTL)} · `ai_logs`.

## API (/api)
Public: GET /talents · GET /talents/{id} (404 unless approved/owner/admin) · GET /activities · GET /summary · GET /today (America/Sao_Paulo)
Artist (Google session): POST /talents · GET /me/talents · PUT/DELETE /talents/{id} (owner)
Admin: GET /admin/talents?status · PATCH /talents/{id}/status {status} · PUT/DELETE any · GET /stats · POST /ai/report
Auth: POST /auth/session {session_id} · GET /auth/me → {user|null, is_admin} (always 200) · POST /auth/login {password} · POST /auth/logout
AI (SSE): POST /ai/recommend · POST /ai/bio · POST /ai/report (admin)

## Authorization
lib/auth.py `authorize(principal, action, talent)` — RBAC (admin) + ownership. Status/owner never client-writable.

## AI
Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`) via emergentintegrations + EMERGENT_LLM_KEY.

## Seed
`cd /app/backend && python seed.py` — 10 approved example talents (idempotent, ids preserved), activities with dates relative to today
(e.g. "Ateliê aberto de pintura" today, "Oficina de Upcycling" +2d, "Roda de Samba do Beco" +4d, "Espetáculo Beco em Cena" +12d).
Also 2 real user-created profiles (Rafael, Paulo Rogerio da Silva) migrated to approved.
