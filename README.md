# Minionese Academy

Site de fan pour apprendre le Minionese : 13 leçons, 8 mini-jeux, traducteur IA.
Non officiel, sans lien avec Illumination ni Universal Pictures.

## Mise en ligne

### 1. Le Worker (traducteur IA, gratuit)

Le Worker utilise **Cloudflare Workers AI** : 10 000 Neurons offerts par jour, sans carte bancaire.
Au-delà, la requête échoue et le site bascule sur son dictionnaire local. Aucune facture possible sur le forfait gratuit.

Par le tableau de bord (sans terminal) :
1. dash.cloudflare.com → **Workers & Pages** → **Create** → modèle **Hello World**, nom `minionese-api` → **Deploy**. Note l'adresse `https://minionese-api.xxxx.workers.dev`.
2. **Edit code** : remplace tout par le contenu de `worker/src/index.js` → **Deploy**.
3. **Settings → Bindings → Add → Workers AI**, nom de variable `AI` → **Deploy**.
4. Mets l'adresse du Worker dans la constante `TRANSLATE_API` de `index.html` (suivie de `/translate`).

Option payante : ajouter un secret `ANTHROPIC_API_KEY` fait passer le Worker sur Claude (meilleure qualité).

### 2. Le site

1. Dans le dépôt : **Settings → Pages → Build and deployment → Deploy from a branch**, branche `main`, dossier `/ (root)`.
2. Ajoute un domaine personnalisé, par exemple `minionese.endam-digital.com` (enregistrement CNAME vers `<ton-user>.github.io`).
3. Vérifie que cette origine exacte figure dans `ALLOWED_ORIGINS` (`worker/wrangler.toml`), sinon ajoute-la et redéploie le Worker.
4. Si l'URL du Worker diffère, modifie la constante `TRANSLATE_API` dans `index.html`.

### 3. Tester

```bash
curl -X POST https://minionese-api.endam-digital.com/translate \
  -H "Origin: https://minionese.endam-digital.com" -H "Content-Type: application/json" \
  -d '{"text":"Bonjour, je veux une banane !","direction":"fr2mi","lexique":"bonjour = bello ; banane = banana"}'
```
Réponse attendue : `{"translation":"Bello ! Me want banana !"}` (ou proche).
