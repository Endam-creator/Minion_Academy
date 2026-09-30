# Minionese Academy

Site de fan pour apprendre le Minionese : 13 leçons, 8 mini-jeux, traducteur IA.
Non officiel, sans lien avec Illumination ni Universal Pictures.

## Mise en ligne

### 1. Le Worker (traducteur IA)

Prérequis : un compte Cloudflare avec la zone `endam-digital.com`, Node.js, une clé API Anthropic.

```bash
cd worker
npx wrangler login
npx wrangler secret put ANTHROPIC_API_KEY     # colle la clé, elle ne quitte jamais Cloudflare
npx wrangler deploy
```

Le Worker répond sur `https://minionese-api.endam-digital.com/translate`.
Si ta zone n'est pas sur Cloudflare : supprime le bloc `routes` de `wrangler.toml`, redéploie,
et utilise l'URL `https://minionese-api.<ton-compte>.workers.dev/translate`.

Pense aussi à fixer une limite de dépense mensuelle dans la console Anthropic.

### 2. Le site

1. Dans le dépôt : **Settings → Pages → Build and deployment → Deploy from a branch**, branche `main`, dossier `/docs`.
2. Ajoute un domaine personnalisé, par exemple `minionese.endam-digital.com` (enregistrement CNAME vers `<ton-user>.github.io`).
3. Vérifie que cette origine exacte figure dans `ALLOWED_ORIGINS` (`worker/wrangler.toml`), sinon ajoute-la et redéploie le Worker.
4. Si l'URL du Worker diffère, modifie la constante `TRANSLATE_API` dans `docs/index.html`.

### 3. Tester

```bash
curl -X POST https://minionese-api.endam-digital.com/translate \
  -H "Origin: https://minionese.endam-digital.com" -H "Content-Type: application/json" \
  -d '{"text":"Bonjour, je veux une banane !","direction":"fr2mi","lexique":"bonjour = bello ; banane = banana"}'
```
Réponse attendue : `{"translation":"Bello ! Me want banana !"}` (ou proche).
