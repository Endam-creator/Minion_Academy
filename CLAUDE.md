# Minionese Academy

Site de fan, en français, pour apprendre le Minionese (la langue des Minions) : 13 leçons, 8 mini-jeux, un traducteur IA avec repli sur un dictionnaire local, des rangs et des badges.
Public : enfants et familles. Ton joyeux, zéro contenu inapproprié.

## Structure

```
index.html           Toute l'application (HTML + CSS + 3 blocs <script>), aucune dépendance, aucun build
worker/src/index.js  Proxy Cloudflare Worker : Workers AI gratuit par défaut (binding AI), Claude si ANTHROPIC_API_KEY (construit le prompt)
worker/wrangler.toml Config du Worker : origines autorisées, modèle, limite de débit, domaine
```

Hébergement : la racine du dépôt sur GitHub Pages (branche main, dossier « / (root) ») (sous-domaine endam-digital.com), `worker/` sur Cloudflare.

## Lancer en local

```bash
python3 -m http.server 8080                 # http://localhost:8080
cd worker && npx wrangler dev                    # proxy sur http://localhost:8787
```
Pour tester le traducteur IA en local, remplacer temporairement `TRANSLATE_API` par `http://localhost:8787/translate`
(`http://localhost:8080` est déjà dans `ALLOWED_ORIGINS`).

## Organisation de index.html

Les trois blocs `<script>` partagent des variables globales et doivent rester dans cet ordre.

1. **Données et état** : `state` (bananes, leçons faites, jeux faits, mots bonus débloqués), `LEVELS` (3 niveaux, 13 leçons), `ALL_LESSONS`, `BASE_DICT` (~130 entrées FR → Minionese, mêmes règles que les leçons), `BONUS_DICT` (mots bonus inventés pour l'Académie, débloqués par les jeux et signalés comme inventés), `GAMES`, `RANKS`, `BADGES`.
2. **Navigation, leçons et jeux** : `go()`, `renderLessons()`, `finishLesson()`, puis un couple `startX / endX` par jeu.
3. **Traducteur, dictionnaire, progrès, sauvegarde** : `localTranslate()`, `aiTranslate()` (appelle le Worker), `translateText()` (repli automatique sur le local), `autoSave()` / `autoLoad()` (localStorage, clé `minionese-academy-v1`, appelés après chaque changement de progression), `exportSave()` / `importSave()` (code JSON en base64 pour changer d'appareil), `resetProgress()` (remise à zéro en deux clics).

Format d'une leçon : `{id, emoji, title, intro, vocab:[[français, minionese, origine], ...], grammar, unlocksGame, reward}`.

## Jeux (clé → débloqué par)

`quiz` L1 · `flash` L2 · `match` L3 · `memory` L5 · `mystery` L6 · `builder` (phrases à reconstruire) L8 · `tf` (vrai/faux) L9 · `rush` L11 (champ `needLesson` de `GAMES`).
Données des jeux : `FLASH`, `MATCH_ALL`, `MEMORY_ALL`, `MYSTERY` (lettres A-Z uniquement), `SENTENCES`, `RUSH_POOL` ; le quiz et le vrai/faux puisent dans le vocabulaire des leçons.
Les 8 jeux sont implémentés. Si on en ajoute un (ex. Duel), mettre à jour `GAMES`, le déblocage dans `LEVELS`, les badges et le texte « 8 mini-jeux » de la page.

## Atelier « Fabrique ton mot » (onglet Atelier)

- `craftWord()` applique à un mot les règles observées dans les sous-titres : suffixe « -a » (`withA` : help → helpa, stop → stoppa, stone → stone-a, happy → happa),
  article « le », « B » initial (`withB` : apple → bapple, thank → bank, good night → bood night, b → p comme pank you), répétition, « Tu le … ! ».
- Chaque carte cite l'exemple réel qui prouve la règle. Les mots produits sont présentés comme « fabriqués par toi », jamais comme officiels.
- Voix : `speakText()` (partagée avec le traducteur). Entrée utilisateur échappée (`escapeHtml`).

## Traducteur IA

- Le navigateur envoie `{text, direction: "fr2mi" | "mi2fr", lexique}` au Worker, qui renvoie `{translation}`.
- Le prompt est construit **dans le Worker**, jamais côté client. La clé API n'apparaît jamais dans `index.html`.
- Moteur : Cloudflare Workers AI (`@cf/meta/llama-3.1-8b-instruct`, 10 000 Neurons/jour gratuits, binding `AI`) ; Claude seulement si le secret `ANTHROPIC_API_KEY` existe.
- Garde-fous du Worker : origines autorisées, 500 caractères max, 10 requêtes/min/IP, `max_tokens` 400, consigne anti-injection et filtre « contenu pour enfants ».
- Toute erreur (réseau, 429, 5xx) → le moteur local prend le relais et un toast prévient l'utilisateur.

## Règles de contenu

- **Chaque mot a une source**, notée dans la colonne « origine » (audit complet le 16/11/2026) :
  - `✔ …` = entendu dans les films (sous-titres officiels), cité par Pierre Coffin (Variety, 2026) ou présent dans la liste de référence reprise par la presse
    (bello, poopaye, tank yu, bi-do, pwede na, para tu, sa la ka, tatata bala tu, underwear, luk at tu, tulaliloo ti amo, hana/dul/sae, bapple, gelato,
    buttom, chasy, kampai, bee do, muak, la boda, stupa, baboi/papoy, bananonina, po ka, bu ta na ma ka) ;
  - `emprunt à …` = vrai mot d'une autre langue (por favor, fiesta, piccolo…), présenté comme tel.
  - **Interdit** : les mots des wikis de fiction (« fanon ») et toute étymologie inventée. Retirés lors de l'audit : ma twat, bulaka, flavuk, popadom, melomo,
    pekang, naje, kam-nen, yakitori (« cours »), baduga, comarino, crona, trabaho, molla, papagaya, papagena, loka, sella, matoka, profiterol, xylopo, etc.
  - Source principale des ajouts du 16/11/2026 : les sous-titres anglais officiels de « Moi, moche et méchant 4 » (helpa, pusha, stoppa, looka, yummos, no problemo,
    all is goot, perfecto, tu le besta / stronga, porque meo, scusa, yippa, smoocha, ciao bellos, tiramisu, prosciutto, strudel, pomodoro…), des « Minions 2 »
    (bye-bye, pank you, come-ay, look-a the map-a, la casa, la stone-a, la telefono, mi amor, chica, Mini-Boss, trey ticket-as…) et du film de 2026.
    L'origine de chaque mot cite le film. La règle « -a » (helpa, pusha) et l'article « le » universel sont observés dans ces sous-titres (leçon 11).
  - Les mots bonus (`BONUS_DICT`) sont volontairement inventés et doivent rester signalés comme tels.
- Leçon 13 (« Des Minions et des monstres », 2026) : vérifiée le 16/11/2026. Tout le vocabulaire vient des sous-titres anglais officiels du film
  (bank you, bank you a tutti, mee-ta, soy, xiè xie, kanpai, para le movies, piñata / carbonara / lasagna, toot toot, « Mbappé de Platini »).
  Faits confirmés : Hollywood 1927, trio James / Henry / Ed (Ed parle en langue des signes), monstre Goomi (acronyme de « Gary Orkam Oliver Magma Ichabod the Deceiver »),
  « xiè xie » glissé à la fin (Variety). Retirés de la leçon 13 car absents de ce film : casos, pignata, et teléfono / mi amor (ces deux-là viennent en réalité des « Minions 2 » et sont enseignés leçons 4 et 10) ; « Gran Jefe » vient d'un doublage latino-américain antérieur (leçon 10).
- Propriété intellectuelle : aucune image, aucun logo ni extrait officiel (emojis et CSS uniquement). La mention « site de fan non officiel » en bas de page reste en place. Pas de monétisation sous la marque Minions.

## Pistes (par priorité)

1. Jeu « Duel » contre l'IA, via le même Worker (ajouter une route dédiée plutôt que d'ouvrir `/translate`).
2. PWA (manifest + icône) pour l'usage mobile. Pas d'app native.
