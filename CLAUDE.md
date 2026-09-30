# Minionese Academy

Site de fan, en français, pour apprendre le Minionese (la langue des Minions) : 13 leçons, 8 mini-jeux, un traducteur IA avec repli sur un dictionnaire local, des rangs et des badges.
Public : enfants et familles. Ton joyeux, zéro contenu inapproprié.

## Structure

```
docs/index.html      Toute l'application (HTML + CSS + 3 blocs <script>), aucune dépendance, aucun build
worker/src/index.js  Proxy Cloudflare Worker vers l'API Claude (garde la clé, construit le prompt)
worker/wrangler.toml Config du Worker : origines autorisées, modèle, limite de débit, domaine
```

Hébergement : `docs/` sur GitHub Pages (dossier /docs de la branche main) (sous-domaine endam-digital.com), `worker/` sur Cloudflare.

## Lancer en local

```bash
cd docs && python3 -m http.server 8080          # http://localhost:8080
cd worker && npx wrangler dev                    # proxy sur http://localhost:8787
```
Pour tester le traducteur IA en local, remplacer temporairement `TRANSLATE_API` par `http://localhost:8787/translate`
(`http://localhost:8080` est déjà dans `ALLOWED_ORIGINS`).

## Organisation de index.html

Les trois blocs `<script>` partagent des variables globales et doivent rester dans cet ordre.

1. **Données et état** : `state` (bananes, leçons faites, jeux faits, mots bonus débloqués), `LEVELS` (3 niveaux, 13 leçons), `ALL_LESSONS`, `BASE_DICT` (~140 entrées FR → Minionese), `BONUS_DICT` (mots débloqués par les jeux), `GAMES`, `RANKS`, `BADGES`.
2. **Navigation, leçons et jeux** : `go()`, `renderLessons()`, `finishLesson()`, puis un couple `startX / endX` par jeu.
3. **Traducteur, dictionnaire, progrès, sauvegarde** : `localTranslate()`, `aiTranslate()` (appelle le Worker), `translateText()` (repli automatique sur le local), `autoSave()` / `autoLoad()` (localStorage, clé `minionese-academy-v1`, appelés après chaque changement de progression), `exportSave()` / `importSave()` (code JSON en base64 pour changer d'appareil), `resetProgress()` (remise à zéro en deux clics).

Format d'une leçon : `{id, emoji, title, intro, vocab:[[français, minionese, origine], ...], grammar, unlocksGame, reward}`.

## Jeux (clé → débloqué par)

`quiz` L1 · `flash` L2 · `match` L3 · `memory` L4 · `mystery` L5 · `builder` (phrases à reconstruire) L6 · `tf` (vrai/faux) L7 · `rush` L8.
Les 8 jeux sont implémentés. Si on en ajoute un (ex. Duel), mettre à jour `GAMES`, le déblocage dans `LEVELS`, les badges et le texte « 8 mini-jeux » de la page.

## Traducteur IA

- Le navigateur envoie `{text, direction: "fr2mi" | "mi2fr", lexique}` au Worker, qui renvoie `{translation}`.
- Le prompt est construit **dans le Worker**, jamais côté client. La clé API n'apparaît jamais dans `docs/`.
- Garde-fous du Worker : origines autorisées, 500 caractères max, 10 requêtes/min/IP, `max_tokens` 400, consigne anti-injection et filtre « contenu pour enfants ».
- Toute erreur (réseau, 429, 5xx) → le moteur local prend le relais et un toast prévient l'utilisateur.

## Règles de contenu

- **Chaque mot ajouté a une source** (film, interview de Pierre Coffin, page Fandom), notée dans la colonne « origine ». On n'invente pas de vocabulaire présenté comme officiel.
- Leçon 13 (« Des Minions et des monstres », 2026) : sont confirmés le cadre (Hollywood des années 1920), le trio James / Henry / Ed, et les mots « Mbappé », « casos », « toot toot ».
  **À revérifier** : « Goomi », « shyeh-shyeh », « Gran Jefe » et les autres mots VF de la leçon.
- Propriété intellectuelle : aucune image, aucun logo ni extrait officiel (emojis et CSS uniquement). La mention « site de fan non officiel » en bas de page reste en place. Pas de monétisation sous la marque Minions.

## Pistes (par priorité)

1. Revérifier le vocabulaire de la leçon 13 marqué ci-dessus.
2. Jeu « Duel » contre l'IA, via le même Worker (ajouter une route dédiée plutôt que d'ouvrir `/translate`).
3. PWA (manifest + icône) pour l'usage mobile. Pas d'app native.
