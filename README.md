# Prospection Distributeurs

Outil de prospection et CRM léger pour une entreprise de distribution automatique (café, boissons, en-cas).

- **Session d'appels** : on choisit le temps disponible (15, 30, 45 ou 60 min). L'outil prépare la liste : d'abord les relances prévues, puis les meilleurs nouveaux prospects, avec téléphone, e-mail, dirigeant, script d'appel et e-mail prêt à envoyer.
- **Méthode conseillée** pour chaque prospect : appel, e-mail ou visite, selon le type d'entreprise et les coordonnées trouvées.
- **Relances automatiques** : pas de réponse, absent, rappeler plus tard, déjà équipé chez un concurrent… chaque résultat programme la suite tout seul.
- **Tournée** : les visites du jour et les commerces proches, dans l'ordre le plus court, avec l'itinéraire Google Maps.
- **CRM** : liste des prospects par statut, historique de chaque contact, ajout manuel (recommandations, demandes entrantes), export tableur.
- **Jamais de doublon** : chaque établissement est identifié par son SIRET.

Sources des données :
- **Registre officiel des entreprises** (API Recherche d'entreprises, gratuite, sans clé) : nom, adresse, activité, effectif, date d'installation, dirigeant.
- **Google Places** (facultatif) : téléphone, site web, horaires, note Google.
- **Site web de l'entreprise** : adresse e-mail (page d'accueil, contact ou mentions légales).

---

## Ce qu'il y a dans le dossier

| Fichier | Rôle |
|---|---|
| `index.html` | L'application (une seule page, hébergée sur GitHub Pages) |
| `config.example.js` | Modèle de configuration à copier en `config.js` |
| `supabase/schema.sql` | Création de la base de données |
| `supabase/functions/prospects/index.ts` | Fonction serveur : recherche d'entreprises et de coordonnées |
| `.github/workflows/keepalive.yml` | Empêche la mise en veille du projet Supabase gratuit |

Sans `config.js`, l'application s'ouvre en **mode démonstration** avec des entreprises fictives.

---

## Installation (environ 30 minutes)

### 1. Supabase (base de données et connexion)

1. Créer un compte sur [supabase.com](https://supabase.com), puis **New project**. Région : Europe (Paris ou Francfort). Plan gratuit.
2. **SQL Editor > New query** : coller le contenu de `supabase/schema.sql`, puis **Run**. *(Mise à jour depuis une version précédente : relancer le même fichier, il ajoute ce qui manque sans rien effacer, puis redéployer la fonction et faire **Réglages > Recalculer les notes**.)*
3. **Authentication > Sign In / Providers** :
   - laisser **Email** activé ;
   - désactiver **Allow new users to sign up** (personne ne pourra créer de compte seul).
4. **Authentication > Users > Add user > Create new user** : créer un compte pour toi et un pour ton beau-père (e-mail + mot de passe), en cochant **Auto Confirm User**.
5. **Project Settings > API** (ou **API Keys**) : noter l'**URL du projet** et la clé **anon** (ou **publishable**).

### 2. La fonction serveur

Au choix :

- **Depuis le tableau de bord** : **Edge Functions > Deploy a new function > Via Editor**, nommer la fonction exactement `prospects`, coller le contenu de `supabase/functions/prospects/index.ts`, puis **Deploy**.
- **En ligne de commande** (depuis ce dossier) :
  ```bash
  npx supabase login
  npx supabase functions deploy prospects --project-ref <identifiant-du-projet>
  ```

Laisser l'option de vérification du jeton (JWT) activée : seuls les utilisateurs connectés peuvent appeler la fonction.

### 3. Google Places (facultatif, recommandé)

Sans cette clé, l'outil fonctionne, mais il faut chercher les numéros à la main (un bouton ouvre la recherche Google).

1. [console.cloud.google.com](https://console.cloud.google.com) : créer un projet, puis associer un compte de facturation (carte bancaire obligatoire).
2. **APIs & Services > Library** : activer **Places API (New)**.
3. **APIs & Services > Credentials > Create credentials > API key**. Dans les restrictions de la clé, choisir **Restrict key** et ne cocher que **Places API (New)**.
4. Sécurité budgétaire :
   - **Billing > Budgets & alerts** : créer un budget de 5 € avec alerte e-mail ;
   - **APIs & Services > Places API (New) > Quotas** : limiter les requêtes **Text Search** par jour (par exemple 50).
5. Dans Supabase : **Edge Functions > Secrets**, ajouter `GOOGLE_PLACES_API_KEY` avec la clé.
   Facultatif : `DAILY_GOOGLE_CAP` pour changer le plafond (30 fiches par jour par défaut).

Google offre un quota gratuit mensuel ; avec le plafond de 30 fiches par jour, l'usage reste normalement dans ce quota. Vérifie les tarifs à jour sur la page de tarification de Google Maps Platform.

### 4. GitHub Pages (l'adresse de l'application)

1. Créer un dépôt GitHub **public** (GitHub Pages est gratuit pour les dépôts publics). Le code ne contient aucune donnée de prospects : elles restent dans Supabase, protégées par mot de passe.
2. Copier `config.example.js` en `config.js` et remplir l'URL, la clé anon et le nom de l'entreprise.
3. Envoyer tous les fichiers dans le dépôt.
4. **Settings > Pages** : source **Deploy from a branch**, branche `main`, dossier `/ (root)`.
5. L'application est disponible à `https://<ton-compte>.github.io/<nom-du-depot>/`.
6. Dans Supabase, **Authentication > URL Configuration** : mettre cette adresse dans **Site URL**.

### 5. Garder le projet actif

Supabase met en pause un projet gratuit resté 7 jours sans activité. Pour l'éviter :
GitHub > **Settings > Secrets and variables > Actions**, créer `SUPABASE_URL` et `SUPABASE_ANON_KEY`. Le robot `keepalive.yml` contacte la base chaque matin.

### 6. Premier lancement

1. Ouvrir l'adresse, se connecter.
2. **Réglages** : prénom, entreprise, téléphone, et surtout **l'offre en une phrase** (elle est reprise dans les scripts et les e-mails). Vérifier le point de départ et le rayon.
3. **Réglages > Mettre à jour la base** : l'outil parcourt le registre des entreprises autour du point de départ (quelques minutes la première fois, avec une barre de progression). Sinon, la première session le fait automatiquement, juste assez pour démarrer.
   Ensuite, **Réglages > Vos clients actuels** : coller la liste des clients (un par ligne, avec la ville), puis **Analyser mes clients**.
4. Sur le téléphone de ton beau-père : ouvrir l'adresse, puis **Partager > Sur l'écran d'accueil** pour l'avoir comme une application.

---

## Comment l'outil choisit les prospects

La question posée pour chaque établissement : **« Est-ce qu'il y a du monde sur place, sans autre solution pour boire un café, et quelqu'un sur place qui peut dire oui ? »**

L'outil croise quatre sources gratuites :

| Source | Ce qu'elle apporte | Clé nécessaire |
|---|---|---|
| Registre officiel des entreprises (recherche-entreprises.api.gouv.fr) | Activité, effectif, date d'installation, taille du réseau, dirigeant | Non |
| OpenStreetMap | Cafés, boulangeries, restaurants, supérettes et distributeurs autour de chaque prospect | Non |
| BOAMP (annonces des marchés publics) | Appels d'offres « distributeurs automatiques » dans l'Hérault | Non |
| Google (facultatif) + site web de l'entreprise | Téléphone, e-mail, horaires, affluence, recrutement, cantine | Google : oui |

Chaque prospect reçoit une **note sur 100**, faite de 5 critères visibles dans sa fiche (« Détail de la note ») :

1. **Monde sur place (35 %)** : salariés + visiteurs du jour (patients, clients qui attendent, adhérents, stagiaires). Bonus pour les équipes en horaires décalés.
2. **Peu d'alternatives autour (15 %)** : aucun café ni boulangerie à moins de 400 m = idéal. Malus si un distributeur est déjà visible sur la carte ou si le site parle d'une cantine.
3. **Bon moment (15 %)** : installation de moins d'un an, recrutement en cours, déménagement.
4. **Décision accessible (15 %)** : entreprise locale (pas de centrale d'achat nationale), dirigeant connu, numéro disponible. Le public est plafonné (marchés publics obligatoires).
5. **Proche de vos clients (20 %)** : ressemble à vos clients actuels ou se trouve à côté de l'un d'eux. L'outil apprend aussi des résultats : une catégorie qui signe souvent remonte, une qui refuse toujours descend.

La fiche indique aussi **ce qu'on peut installer** (« Une machine à café », « Café + boissons fraîches et snacks »…), le **nombre estimé de personnes sur place** et **qui demander**.

Sont exclus : établissements fermés, entreprises non diffusibles, collèges et lycées (distributeurs interdits pour les élèves), et vos clients actuels.

**Vos clients actuels** : dans **Réglages > Vos clients actuels**, écrire un nom par ligne (avec la ville), puis **Analyser mes clients**. Ils ne seront jamais proposés, et leurs « jumeaux » remontent en tête.

**Appels d'offres publics** : vérifiés une fois par jour ; quand une collectivité ou un hôpital cherche un prestataire de distributeurs, une carte apparaît en haut de l'accueil avec le lien vers le dossier.

Les poids se règlent en haut de `supabase/functions/prospects/index.ts` (`CATS`, `NAF`).

## Méthode conseillée

- **Passer le voir** : commerces ouverts au public de moins de 10 salariés (garages, salles de sport…) où le patron est sur place, ou prospect sans téléphone ni e-mail.
- **Appeler** : sites avec une équipe, en demandant la personne qui s'occupe de la salle de pause ; ou directement le dirigeant d'une petite structure.
- **E-mail** : quand aucun numéro fiable n'existe.

Après 3 appels sans réponse, l'outil passe à la visite (ou à l'e-mail). Après 5 tentatives, le prospect est abandonné.

## Relances programmées

| Résultat | Ce qui se passe |
|---|---|
| Pas de réponse | Rappel 2 jours ouvrés plus tard, à une autre heure de la journée |
| Absent lors d'une visite | Repasser dans 7 jours (puis appel s'il y a un numéro) |
| Plaquette déposée | Appel 3 jours ouvrés plus tard |
| E-mail envoyé | Relance 4 jours ouvrés plus tard |
| Rappeler plus tard | À la date choisie |
| Déjà équipé | Clôturé, on passe au suivant |
| Pas intéressé | Clôturé, on passe au suivant |
| Mauvais numéro | Clôturé, on passe au suivant |
| RDV pris | Apparaît dans « Prochains rendez-vous » |
| Signé | Passe en client |

Les délais se changent dans **Réglages**.

## Points d'attention

- **Prospection par e-mail** : en B2B, la prospection est admise quand le message concerne l'activité professionnelle du destinataire et permet de refuser facilement la suite. Le modèle d'e-mail contient cette mention, et un prospect « Pas intéressé » n'est plus jamais reproposé. En cas de doute, consulter les recommandations de la CNIL.
- **Fiabilité des coordonnées** : la correspondance avec Google se fait par nom et position ; une fiche peut être fausse ou manquante. Le bouton « Mauvais numéro » clôt le prospect.
- **Coût Google** : plafonné par l'outil (30 fiches par jour) et par le quota que tu fixes dans la console Google.
