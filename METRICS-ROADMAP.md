# Metrics-roadmap: nästa steg för analysdashboarden

## Nuläge
`analyzer.html` arbetar mot en CSV-export från användarregistreringen med fälten:
`id, analytics_id, created_at, age_ranges, subjects, role, should_import`

## Datakällor att utforska

### 1. GA4 / BigQuery (Data & Analys-teamet har redan detta)
En export (`bigquery.csv`) visar att det finns sessionsnivådata med:
- Trafikursprung: `session_source`, `session_medium`, `session_campaign`, `channel_group`
- Speldata: `player_play_count` per session
- Aktivering: `first_login_datetime`
- Konvertering: `converted` (exakt definition okänd — kolla med D&A-teamet)
- Profil inbakad per session: `profile_role`, `profile_age_ranges`, `profile_subjects`

**Nästa steg:** Ta kontakt med Data & Analys för att förstå:
- Vad `converted` betyder
- Om `content_id`/`program_id` finns (vilket program spelades?)
- Hur man får ut historik (>1 dag) på ett strukturerat sätt

### 2. Programmetadata (viktigast på sikt)
Ni har metadata på alla program inkl. målgrupp (utbildningsnivå) och ämnen.
Det intressanta: matchar det lärarna faktiskt tittar på med vad de deklarerat i sin profil?

**Analyser som vore möjliga:**
- Profilmatch-rate: % av visade program som matchar användarens nivå/ämne
- Konsumtion per profilsegment: vad tittar SO-lärare på högstadiet faktiskt på?
- Korsintresse: i vilken utsträckning går lärare utanför sin deklarerade profil?
- Innehållsräckvidd: vilka program når breda vs. smala målgrupper?

**Datakrav:** En tabell med `(user_id, program_id, viewed_at)` + programmetadata `(program_id, target_levels[], target_subjects[])`.

### 3. Keycloak (inloggningshistorik)
- Senaste inloggning → churn-proxy ("ej aktiv på 90 dagar")
- Antal inloggningstillfällen → engagemangsgrad
- Aktiveringsgrad: % av registrerade som loggat in minst en gång

### 4. Registreringskälla (saknas idag)
UTM-parametrar sparas inte vid registrering. Skulle ge svar på vilka kampanjer som driver kvalitativa registreringar (dvs. de som faktiskt aktiveras).

---

## Analyser att bygga när data finns

| Analys | Datakälla | Prioritet |
|--------|-----------|-----------|
| Tid till komplett profil (med roll) | Befintlig export | Hög — kan göras nu |
| Aktiveringsgrad (loggat in efter reg.) | Keycloak | Hög |
| Kohortretention (aktiva per kohort) | Keycloak | Medel |
| Profilmatch mot konsumerade program | GA4 + programmetadata | Hög (kräver D&A) |
| Trafikursprung per profilkvalitet | GA4 | Medel |
| Churn-identifiering | Keycloak | Medel |

---

## Tid till komplett profil — kan byggas nu

Befintlig `users-export.csv` innehåller `created_at`. Om vi definierar "komplett" som `role != ''` kan vi redan nu visa:
- Fördelning: hur många dagar efter registrering fick användare en roll?
- Kohortanalys: faller fler ur i nyare kohorter?

Detta kan läggas till i befintliga **Utan roll**-fliken.
