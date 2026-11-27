# Étude : une app de tâches façon Todoist, avec Obsidian comme base

> **Objectif** : une app qui marche comme Todoist (projets, sections, sous-tâches, échéances, récurrence, étiquettes, priorités, filtres, vues Aujourd'hui / À venir) et qui utilise le vault Obsidian comme **seule source de vérité**. Elle récupère toutes les tâches (`- [ ]`) où qu'elles soient dans le vault et fonctionne sur **iOS et PC**.
>
> Date de l'étude : 26/11/2026

---

## 1. En bref

| Question | Réponse courte |
|---|---|
| C'est faisable ? | **Oui.** Le Markdown d'Obsidian se prête bien au modèle Todoist : une note = un projet, un titre `##` = une section, une case indentée = une sous-tâche, un `#tag` = une étiquette. |
| Ça existe déjà ? | **En partie.** Plusieurs plugins inspirés de Todoist existent (belki, Graphite, Flowstitch, Tasks Plus, Task Genius), ainsi qu'une app iOS native qui lit le vault (TaskForge). **À tester avant de coder** (voir §3). |
| Meilleure architecture ? | **Un plugin Obsidian d'abord** (une seule base de code pour PC + iPhone, aucun souci de synchro), **puis une app iOS compagnon** seulement si tu as besoin de notifications et de widgets fiables. |
| Le vrai point dur ? | **Les notifications sur iOS.** Un plugin Obsidian ne peut pas te notifier quand l'app est fermée. Pour ça, il faut une app native (ou un serveur et une PWA). |
| Sans Obsidian Sync ? | **Oui.** iCloud Drive si ton ordinateur est un Mac. Si c'est un PC Windows, Self-hosted LiveSync sur un serveur chez toi, car iCloud est déconseillé avec Windows (voir §5). |
| Décision qui conditionne tout | **Mac ou Windows** (ça décide de la synchro, donc de la façon d'avoir des notifications sur iPhone), et **le format** de tes tâches actuelles. |

---

## 2. Le besoin, fonction par fonction

| Fonction Todoist | Équivalent dans le vault | Difficulté |
|---|---|---|
| Inbox | Une note `Inbox.md` (cible par défaut de l'ajout rapide) et une vue « tâches sans projet » | Facile |
| Projets | Une note avec `type: projet` en frontmatter, ou un dossier, ou un tag `#projet/xxx` | Facile |
| Sections | Les titres `##` dans la note projet | Facile |
| Sous-tâches | Les cases à cocher indentées | Facile |
| Description et commentaires | Les lignes indentées (non cochables) sous la tâche | Moyen |
| Date d'échéance | `📅 2026-11-30` (format du plugin Tasks) | Facile |
| Heure | Pas gérée nativement par Tasks. Convention proposée : heure en début de texte, `- [ ] 14:30 Appel` (comme Day Planner) | Moyen |
| Date planifiée / de début | `⏳` (planifiée) et `🛫` (début) | Facile |
| Récurrence | `🔁 every week` ; à la complétion, on crée l'occurrence suivante | Moyen |
| Priorités P1 à P4 | `🔺` = P1, `⏫` = P2, `🔼` = P3, rien = P4 | Facile |
| Étiquettes (`@label`) | Tags Obsidian `#label` (imbriqués possibles : `#ctx/maison`) | Facile |
| Ajout rapide en langage naturel (« demain 14h p1 #Courses @urgent ») | Parseur de dates FR (chrono-node) et syntaxe Todoist traduite à l'écriture | Moyen |
| Vues Aujourd'hui / À venir / En retard | Requêtes sur l'index | Facile |
| Filtres (`today & p1`, `#Travail \| overdue`) | Mini-langage de requêtes, filtres enregistrés dans un fichier du vault | Moyen |
| Vue tableau (kanban par section) | Glisser une carte = déplacer la ligne sous un autre `##` | Moyen |
| Vue calendrier | Sur les dates `📅` / `⏳` | Moyen |
| Glisser-déposer pour réordonner | Dans un projet, l'ordre = l'ordre des lignes (naturel). Dans « Aujourd'hui », l'ordre manuel est stocké à part | Moyen |
| Durée | Champ personnalisé, par exemple `[durée:: 30m]` | Facile |
| Rappels et notifications | Plugin : seulement quand Obsidian est ouvert. App native : notifications locales fiables | **Dur sur iOS** |
| Widgets iOS, Siri, feuille de partage | App native uniquement | Dur |
| Statistiques (karma, tâches faites par jour) | À partir des dates `✅` | Facile |
| Partage et collaboration | Hors périmètre (vault personnel) | Non prévu |

---

## 3. Ce qui existe déjà (à tester avant de coder)

D'après leurs fiches publiques (non testés en détail) :

| Outil | Type | Ce qu'il apporte | Limites probables |
|---|---|---|---|
| **belki** | Plugin | « Todoist-like » assumé : Inbox, Aujourd'hui, À venir, Projets, Filtres et étiquettes, priorités P1 à P4, couleurs | Jeune, environ 7 000 téléchargements |
| **Graphite** | Plugin | Inbox, Aujourd'hui (avec En retard), À venir, Projets, Filtres et étiquettes, Activité, Terminées, Recherche | À vérifier |
| **Flowstitch** | Plugin | Aujourd'hui, À venir, Projets et sections, Inbox, historique des terminées | Orienté synchro avec Todoist |
| **Tasks Plus** | Plugin | Vues fixes : Inbox, Aujourd'hui, À venir, N'importe quand, Un jour, Journal | Modèle plutôt « Things » |
| **Task Genius** | Plugin | Inbox, Prévisions, Liste, Kanban, Calendrier, Gantt ; index du vault en arrière-plan | Riche, donc plus complexe |
| **Tasks** | Plugin (la référence) | Le **format** de facto (`📅 ⏫ 🔁 🆔 ⛔`), avec des requêtes dans les notes | Pas d'interface « app » |
| **TaskNotes** | Plugin (+ app iPhone en bêta) | Une note par tâche, vues basées sur Obsidian Bases | Ne ramasse pas les cases à cocher éparpillées |
| **TaskForge** | App iOS/Mac native | Lit et écrit le format Tasks dans le vault, notifications, widgets, ajout rapide, langage naturel | Payant (environ 2 $/mois ou 30 $ à vie), pas de version Windows |
| **VaultMate** (ex-Obsi) | App mobile | Gestion des tâches Obsidian sur mobile | À vérifier |

**Combinaison à tester en une semaine** : *belki* ou *Graphite* (PC et iPhone, dans Obsidian) avec *TaskForge* (notifications et widgets sur iPhone). Les deux utilisent le format Tasks, donc ils sont compatibles entre eux.

Si ça couvre 90 % de ton besoin, le plus rentable est de s'arrêter là. Si tu veux **exactement** l'ergonomie Todoist, maîtriser le format ou simplement construire le projet (vibe coding), la suite de l'étude décrit comment faire.

---

## 4. Les architectures possibles

### A. Plugin Obsidian (TypeScript) : recommandé pour démarrer

L'app vit **dans** Obsidian, sous forme d'onglet ou de panneau, avec une interface Todoist.

- **Pour** : accès direct au vault ; l'index des tâches est déjà fourni par Obsidian (`metadataCache` → `listItems`) ; **une seule base de code pour Windows, Mac, iPhone et iPad** ; la synchro reste celle que tu utilises déjà ; pas d'App Store ni de compte développeur Apple.
- **Contre** : **pas de notifications quand Obsidian est fermé** (iOS suspend l'app) ; pas de widget ni de raccourci Siri natif ; démarrage plus lent sur iPhone ; interface limitée au cadre d'Obsidian.

### B. App autonome qui lit le dossier du vault

- **PC** : Tauri (Rust et interface web, léger) ou Electron.
- **iOS** : SwiftUI, ou Capacitor pour réutiliser le code TypeScript. L'utilisateur choisit le dossier du vault une fois dans le sélecteur Fichiers, et l'app garde l'accès via un *bookmark* (mécanisme officiel d'Apple). C'est ce que fait TaskForge.
- **Pour** : notifications locales fiables, widgets, ajout rapide via la feuille de partage ou Siri, démarrage instantané.
- **Contre** :
  - **Si tu utilises Obsidian Sync ou un plugin de synchro (LiveSync, Remotely Save)** : seule l'app Obsidian synchronise. Ce que l'app compagnon modifie sur l'iPhone ne part vers le PC qu'à la prochaine ouverture d'Obsidian. Avec un vault dans **iCloud Drive**, la synchro se fait en arrière-plan.
  - Risque de conflit si Obsidian et l'app modifient le même fichier en même temps.
  - Compte Apple Developer (environ 99 €/an) pour l'installer durablement (TestFlight). Sans compte, l'app expire tous les 7 jours.
  - Deux interfaces à maintenir (PC et iOS).

### C. Hybride : un noyau commun, deux interfaces (cible à terme)

Une bibliothèque **`task-core`** en TypeScript contient toutes les règles : parseur, sérialiseur, requêtes, récurrence, dates en langage naturel. Deux interfaces l'utilisent : **le plugin Obsidian** (PC et iPhone) et **une app compagnon iOS légère** dédiée aux notifications, aux widgets et à l'ajout rapide.

```
        ┌──────────────────── Vault Obsidian (.md) ────────────────────┐
        │  notes projets · notes quotidiennes · Inbox.md · partout      │
        └──────────────▲──────────────────────────────▲─────────────────┘
                       │ lecture / écriture ligne      │ lecture / écriture ligne
            ┌──────────┴──────────┐         ┌──────────┴───────────┐
            │  Plugin Obsidian    │         │  App compagnon iOS   │
            │  (Windows + iPhone) │         │  notifs · widgets ·  │
            │  interface Todoist  │         │  ajout rapide        │
            └──────────┬──────────┘         └──────────┬───────────┘
                       └────────── task-core (TS) ─────┘
          parseur · sérialiseur · requêtes/filtres · récurrence · dates FR
```

### D. Serveur et PWA (variante « tout web »)

Un petit serveur (NAS, Raspberry Pi ou VPS) garde une copie du vault **en fichiers lisibles** :

- **Sans Obsidian Sync** : `livesync-bridge` recopie le vault Self-hosted LiveSync dans un dossier, dans les deux sens (voir §5).
- Avec Obsidian Sync : Obsidian Headless, le client Sync en ligne de commande (en bêta ouverte depuis mars 2026).

Le serveur fait tourner `task-core`, envoie les rappels et peut servir une **PWA** façon Todoist, installée sur l'écran d'accueil de l'iPhone et sur le PC. Pour les rappels, deux voies : **ntfy** (app iOS gratuite, serveur auto-hébergeable, c'est le plus simple) ou Web Push via la PWA (iOS 16.4+).

- **Pour** : notifications fiables sur iPhone sans app native ; ajout rapide même hors d'Obsidian ; une seule interface web partout.
- **Contre** : un serveur à garder allumé et à sécuriser (Tailscale évite d'exposer un port sur Internet) ; mode hors ligne de la PWA plus difficile.

### Recommandation

**Le choix dépend de ta synchro (§5) :**

- **Mac + iCloud** : A, puis C (app iOS compagnon native). C'est le chemin le plus simple.
- **Windows + serveur maison (LiveSync)** : A, puis D (service de rappels sur le même serveur). L'app iOS native perd son intérêt : avec LiveSync, les fichiers de l'iPhone ne changent que lorsque Obsidian est ouvert.

---

## 5. Synchroniser le vault sans Obsidian Sync

But : arrêter Obsidian Sync, qui coûte environ 4 $/mois en annuel (5 $ en mensuel).

**Le critère décisif est ton ordinateur : Mac ou PC Windows ?** L'aide officielle d'Obsidian déconseille iCloud dès qu'une machine Windows est dans la boucle. Les utilisateurs signalent des doublons (« note (1).md »), des fichiers bloqués « en attente », voire corrompus. Une note dans un forum dit que la version 1.11 aurait corrigé un problème, mais rien d'officiel ne le confirme.

### Options comparées

| Option | Coût | iPhone | Windows | Temps réel | Fichiers lisibles sur un serveur | Effet sur l'app de tâches |
|---|---|---|---|---|---|---|
| **iCloud Drive** | 0 € (5 Go gratuits) ou iCloud+ déjà payé | Natif | **Déconseillé** | Presque | Pas de serveur | **Idéal pour une app iOS compagnon** : iOS synchronise en arrière-plan |
| **Self-hosted LiveSync + CouchDB chez toi** (NAS, Raspberry Pi, mini-PC) | Électricité seulement | Plugin | Plugin | **Oui** | Oui, via `livesync-bridge` | Plugin OK ; rappels iPhone via un service sur ce serveur (option D) |
| **Self-hosted LiveSync + stockage objet** (Cloudflare R2 gratuit, MinIO) | 0 € dans le quota gratuit R2 | Plugin | Plugin | Non (périodique) | Non | Plugin OK ; pas de serveur pour les rappels |
| Self-hosted LiveSync + CouchDB sur un VPS | Environ 4 à 6 €/mois | Plugin | Plugin | Oui | Oui | Comme chez toi, mais **même prix qu'Obsidian Sync**, donc aucune économie |
| Remotely Save + WebDAV (Nextcloud, Synology) | 0 € si le serveur existe déjà | Plugin | Plugin | Non (périodique) | Oui | Possible, mais conflits mal gérés en version gratuite (la gestion avancée est payante) |
| Git (plugin Obsidian Git) | 0 € | Lent | Bon | Non | Oui | Plutôt pour la **sauvegarde** que pour la synchro |
| Syncthing | 0 € | Fragile (iOS coupe les apps en arrière-plan) | Bon | Oui entre ordinateurs | Oui | Peu fiable sur iPhone |

### Recommandation

- **Si ton ordinateur est un Mac : iCloud Drive partout.** C'est gratuit, sans serveur, et c'est le meilleur terrain pour une app iOS compagnon (rappels, widgets).
- **Si c'est un PC Windows : Self-hosted LiveSync avec CouchDB sur une machine chez toi** (NAS, Raspberry Pi, mini-PC). Tu y accèdes depuis l'extérieur avec **Tailscale** (VPN gratuit, sans ouvrir de port). La synchro est en temps réel, chiffrée de bout en bout si tu actives l'option, et identique sur Windows et sur iPhone. Le même serveur pourra plus tard faire tourner le service de rappels de l'app de tâches (option D).
  - **Pas de machine allumée en permanence ?** LiveSync + Cloudflare R2 (gratuit, aucun serveur à gérer). Mais tu n'auras des rappels que lorsque Obsidian est ouvert.
- **Un VPS loué n'apporte rien** pour la seule synchro : il coûte autant qu'Obsidian Sync. Il ne se justifie que s'il héberge aussi l'app de tâches (option D).

### Migrer depuis Obsidian Sync sans rien perdre

1. **Sauvegarde** : zippe le dossier du vault sur l'ordinateur.
2. Vérifie que Sync est à jour sur tous les appareils, puis **désactive Sync partout** (Réglages → Plugins principaux → Synchronisation). Ne fais jamais tourner deux synchros sur le même vault.
3. Installe la nouvelle synchro **sur l'ordinateur d'abord**, qui sert de source de référence.
   - iCloud sur Mac : déplace le vault dans `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/` et active « Toujours conserver le téléchargement » sur ce dossier.
   - LiveSync : installe CouchDB avec Docker sur le serveur, puis le plugin sur l'ordinateur, et envoie le vault vers le serveur.
4. Sur l'iPhone, **crée un vault neuf et vide** (dans iCloud, ou local et rempli par LiveSync via l'« URI de configuration » générée sur l'ordinateur). Ne le fusionne pas avec l'ancienne copie, pour éviter les doublons. Supprime l'ancien vault de l'iPhone une fois que tout est vérifié.
5. **Remplace l'historique des versions de Sync** : plugin Obsidian Git sur l'ordinateur (commit automatique toutes les 30 min vers un dépôt privé), ou sauvegardes du serveur. Garde aussi le plugin principal « Récupération de fichiers ».
6. **Résilie Obsidian Sync** après une à deux semaines sans problème.

---

## 6. Le modèle de données (la décision la plus importante)

Ton besoin est de « récupérer les tâches partout ». Le format de base doit donc être **la case à cocher en ligne, au format Tasks (emoji)**, et non « une note par tâche ». C'est le format le plus répandu, et belki, Task Genius, TaskForge et le plugin Tasks le comprennent déjà : aucun verrouillage.

### Exemple de note projet

```markdown
---
type: projet
couleur: bleu
---
# Rénovation cuisine

## À commander
- [ ] Choisir le plan de travail #achats ⏫ 📅 2026-12-05
  Prendre du stratifié si le budget dépasse 800 €.
  - [ ] Demander 3 devis
  - [ ] Comparer les délais
- [ ] 09:00 Appeler le plombier #appels 🔺 📅 2026-11-28

## Récurrent
- [ ] Vider le lave-vaisselle #maison 🔁 every day ⏳ 2026-11-27
- [x] Sortir les poubelles #maison 🔁 every week ✅ 2026-11-25
```

### Règles proposées (configurables)

1. **Projet d'une tâche**, par ordre de priorité : tag `#projet/xxx` sur la tâche, puis champ `projet:` dans le frontmatter de la note, puis note `type: projet`, puis dossier configuré comme « dossier de projets ». À défaut, la tâche est **sans projet** et apparaît dans la vue *Inbox*, mais **reste dans son fichier d'origine**, par exemple ta note quotidienne.
2. **« Déplacer vers un projet »** : soit on coupe la ligne et on la colle sous le bon `##` de la note projet, soit on ajoute un tag `#projet/xxx` sans déplacer la ligne. Les deux modes sont proposés, au choix dans les réglages.
3. **Identité stable** : un ID `🆔 a1b2c3` n'est ajouté **que lorsqu'il est utile** (rappel, dépendance, ordre manuel), pour ne pas encombrer les notes. Sinon, la tâche est identifiée par fichier, ligne et empreinte du texte.
4. **Écriture chirurgicale** : on ne modifie **que la ligne concernée** (dans un plugin, `vault.process()`), jamais le fichier entier, pour respecter la mise en forme de l'utilisateur.
5. **Ajout rapide** : on garde la syntaxe Todoist (`#Projet`, `@étiquette`, `p1`, « demain 14h », « tous les lundis ») et on la traduit au format Markdown à l'écriture : `@urgent` devient `#urgent`, `p1` devient `🔺`, « demain » devient `📅 2026-11-27`.
6. **Récurrence** : quand on coche une tâche récurrente, elle reçoit `✅ date` et l'occurrence suivante est insérée au-dessus, comme le fait le plugin Tasks.
7. **Filtres enregistrés** : dans un fichier du vault (`.obsidian/plugins/<plugin>/data.json` ou `Filtres.md`), pour qu'ils soient synchronisés eux aussi.

---

## 7. Points durs et risques

| Risque | Impact | Parade |
|---|---|---|
| Pas d'ID natif dans le Markdown | Une tâche « perdue » après une modification à la main | Fichier + ligne + empreinte, puis `🆔` ajouté au besoin ; réconciliation à chaque réindexation |
| Conflits de synchro (iCloud crée des copies « en conflit ») | Doublons ou perte d'une modification | Écritures ligne par ligne, relecture juste avant d'écrire, détection des fichiers en conflit |
| Formats hétérogènes (cases nues, format Dataview `[due:: …]`, emoji) | Métadonnées invisibles | Lire les deux formats, écrire en emoji, proposer une normalisation optionnelle |
| Gros vault (des milliers de notes) | Lenteur | Index incrémental : le cache d'Obsidian dans le plugin, SQLite avec date de modification dans une app native |
| Notifications sur iOS | Promesse « Todoist » non tenue | App compagnon native (C) ou PWA et serveur (D) |
| Heures et fuseaux horaires | Tâche affichée au mauvais jour | Dates locales, heure en convention séparée |
| Le format Tasks ne lit les champs **qu'en fin de ligne** | Champs ignorés si du texte suit | Le sérialiseur écrit toujours les tags et le texte **avant** les emojis |
| Revue App Store et compte Apple | Coût et délais | TestFlight pour un usage perso ; publication publique optionnelle |

---

## 8. Stack technique conseillée

- **Langage** : TypeScript partout (le noyau est partagé).
- **`task-core`** : parseur et sérialiseur de lignes de tâches (tests unitaires lourds), moteur de filtres, `rrule` (récurrence), `chrono-node` avec la locale `fr` (dates en langage naturel). Tests avec Vitest.
- **Plugin Obsidian** : API officielle d'Obsidian, interface en **Svelte** ou React, build avec esbuild (gabarit officiel `obsidian-sample-plugin`). Test sur iPhone via la synchro de ton vault, ou avec BRAT pour installer les versions bêta.
- **App iOS compagnon** (phase 3) : Capacitor pour réutiliser `task-core` et l'interface, avec **extension Swift** pour les widgets (WidgetKit) et les notifications locales (`UNUserNotificationCenter`). Alternative : tout en SwiftUI, en exécutant `task-core` via JavaScriptCore.
- **Desktop autonome** : *inutile au début*, car le plugin couvre Windows. Tauri plus tard si tu veux une app hors d'Obsidian.

---

## 9. Feuille de route et estimation

Les estimations sont indicatives, en jours de développement effectifs pour une personne aidée par l'IA.

| Phase | Contenu | Durée |
|---|---|---|
| **0. Quitter Obsidian Sync** | Nouvelle synchro (iCloud ou LiveSync), migration, sauvegarde Git (§5) | 1 soirée, puis 1 à 2 semaines d'observation |
| **0 bis. Test de l'existant** | belki ou Graphite avec TaskForge, sur ton vault réel | 1 semaine d'usage |
| **1. MVP plugin** | `task-core` (parseur, sérialiseur, index), vues Inbox / Aujourd'hui / À venir / Projets, ajout rapide en langage naturel, cocher, récurrence, priorités, étiquettes | 10 à 15 j |
| **2. V1 plugin** | Filtres et filtres enregistrés, sections et sous-tâches éditables, kanban, calendrier, glisser-déposer, raccourcis clavier, réglages des règles de projet, stats | 10 à 15 j |
| **3a. Compagnon iOS** (si Mac + iCloud) | Accès au dossier du vault, lecture et écriture via `task-core`, notifications locales, widget Aujourd'hui, ajout rapide (partage, Siri) | 20 à 30 j |
| **3b. Service serveur** (si Windows + LiveSync) | `livesync-bridge` + `task-core` en tâche de fond, rappels via ntfy, puis éventuellement une PWA d'ajout rapide | 5 à 10 j (rappels seuls), 15 à 25 j avec la PWA |
| **Bonus** | **Migration de tes tâches Todoist actuelles** vers le vault au bon format. Ton compte Todoist est connecté à cette session, je peux le faire directement. | 1 j |

---

## 10. Questions pour décider

Réponse déjà reçue : synchro actuelle = Obsidian Sync, à remplacer par iCloud ou un serveur perso.

1. **Ton ordinateur est-il un Mac ou un PC Windows ?** C'est ce qui tranche entre iCloud et un serveur maison.
2. **As-tu une machine allumée en permanence chez toi** (NAS, Raspberry Pi, vieux PC, box avec Docker) ?
3. **Où et comment sont écrites tes tâches aujourd'hui** : cases nues, format Tasks avec emojis, format Dataview, notes quotidiennes ?
4. **Les rappels et widgets sur iPhone sont-ils indispensables**, ou une app ouverte à la demande te suffit-elle ?
5. Usage **uniquement personnel** (sans partage de projets) ?
6. Veux-tu **migrer tes tâches Todoist** existantes dans le vault ?

---

## Sources

- belki : https://community.obsidian.md/plugins/belki
- Graphite : https://community.obsidian.md/plugins/graphite
- Flowstitch : https://community.obsidian.md/plugins/flowstitch
- Tasks Plus : https://community.obsidian.md/plugins/tasks-plus
- Task Genius : https://github.com/taskgenius/taskgenius-plugin, https://taskgenius.md/docs/task-view
- TaskNotes : https://community.obsidian.md/plugins/tasknotes
- TaskForge : https://apps.apple.com/app/id6744716215, https://forum.obsidian.md/t/ios-macos-taskforge-the-native-task-manager-for-obsidian-users/99347
- VaultMate (ex-Obsi) : https://forum.obsidian.md/t/vaultmate-ex-obsi-app-to-manage-obsidian-tasks-on-mobile/97625
- Format emoji du plugin Tasks : https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format
- Dépendances (🆔 / ⛔) du plugin Tasks : https://publish.obsidian.md/tasks/Getting+Started/Task+Dependencies
- Obsidian Headless (Sync en ligne de commande) : https://obsidian.md/help/sync/headless
- Tarifs Obsidian Sync : https://obsidian.md/sync
- iCloud et Windows, retours d'utilisateurs : https://forum.obsidian.md/t/icloud-on-windows-creates-duplicate-files/48358, https://dev.to/eristoddle/obsidian-icloud-sync-in-2026-including-the-windows-problem-4503
- Self-hosted LiveSync (CouchDB, stockage objet, pair à pair) : https://github.com/vrtmrz/obsidian-livesync, https://community.obsidian.md/plugins/obsidian-livesync
- livesync-bridge (vault LiveSync vers dossier de fichiers) : https://github.com/vrtmrz/livesync-bridge
- Remotely Save (WebDAV, S3) : https://community.obsidian.md/plugins/remotely-save
- Obsidian CLI : https://obsidian.md/cli
- Apple, accès aux dossiers depuis le sélecteur de documents : https://developer.apple.com/documentation/uikit/providing-access-to-directories
