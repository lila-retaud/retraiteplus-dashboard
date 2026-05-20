# RetraitePlus — Regles d'alertes PPC (dashboard operationnel)

**Client** : RetraitePlus (6 comptes Google Ads)
**Mise a jour** : 2026-05-20
**Auteur** : Lila

Ce document liste toutes les regles d'alertes implementees dans le dashboard `dashboard-anomalies.html` et le script `generate_dashboard.py`. Chaque regle est calibree sur les donnees reelles du compte (audits avril-mai 2026).

Objectif : que n'importe quel membre de l'equipe sache immediatement quoi regarder et pourquoi.

---

## Vue d'ensemble des categories

| Cat | Nom | Fenetre | Comptes concernes |
|---|---|---|---|
| A | CPL/CPA anormalement eleve | 7j glissants | BE, IT, CA1, CA2, CH |
| B | Campagne active - spend eleve et 0 conversion | 7j glissants | Tous |
| C | Campagne active qui ne diffuse pas | Consecutif | Tous |
| D | Campagne limitee par le budget (IS Lost Budget) | 7j glissants | Tous |
| E | Search terms suspects | 7j glissants | Tous |
| F | CPC anormalement eleve | 7j glissants | BE, IT, CA1, CA2 |
| G | CTR anormalement bas | 7j glissants | Tous |
| H | Alertes structurelles (Smart Bidding, CVR, concentration) | Variable | Tous |

**Note FR (Wikiehpad) et CH (Heimfinder)** : comptes lances en avril 2026. Pendant la phase de collecte de donnees (60 premiers jours), les CAT A et F ne s'appliquent pas (pas de CPL baseline fiable). Les CAT B, C, D, E, G, H restent actives.

---

## CAT A — CPL/CPA anormalement eleve

### Logique
Compare le CPL moyen des 7 derniers jours au CPL baseline de reference (moyenne ponderee fev-mars-avr 2026 (poids : avr 50%, mars 30%, fev 20%)).

- **WARNING** : CPL 7j > baseline x 1.5
- **CRITICAL** : CPL 7j > baseline x 2.0

### Seuils par compte

| Compte | Devise | CPL baseline | WARNING | CRITICAL |
|---|---|---|---|---|
| BE MR Repos | EUR | 20.94 € | 31.41 € | 41.88 € |
| IT Casa Di Riposo | EUR | 3.40 € | 5.10 € | 6.80 € |
| CA Choisir RPA | CAD | 10.58 | 15.87 | 21.16 |
| CA Places Senior | CAD | 9.60 | 14.40 | 19.20 |
| CH Heimfinder | EUR | 100 € | 200 € | 300 € |
| FR Wikiehpad | EUR | N/A | — | — |

### Pourquoi ces seuils ?

- x1.5 : ecart suffisamment significatif pour meriter attention sans generer de faux positifs
- x2.0 : doublement du CPL = signal fort de dysfonctionnement (ex : BX MR Ixelles a 119€ de CPL pour une cible de 38€ en mai 2026 = x3.1)
- Le CPL CH a 100€ est une valeur observee en phase initiale, non un CPL cible valide - a recalibrer apres 60 jours de donnees

### Quoi verifier en cas d'alerte

1. Smart Bidding en phase d'apprentissage ? (si oui : attendre 7 jours supplementaires)
2. Modification recente du tCPA ou du budget ?
3. Degradation du CVR ou du trafic hors-cible ?
4. Landing page down ou formulaire en erreur ?

---

## CAT B — Campagne active avec spend eleve et 0 conversion

### Logique

Campagne en statut ENABLED avec spend > seuil sur 7 jours et 0 conversion enregistree.

- **WARNING** : spend > seuil bas ET 0 conv
- **CRITICAL** : spend > seuil haut ET 0 conv

### Seuils par compte

| Compte | Devise | WARNING | CRITICAL |
|---|---|---|---|
| BE MR Repos | EUR | 50 € | 100 € |
| IT Casa Di Riposo | EUR | 20 € | 40 € |
| CA Choisir RPA | CAD | 40 | 80 |
| CA Places Senior | CAD | 50 | 100 |
| FR Wikiehpad | EUR | 30 € | 70 € |
| CH Heimfinder | EUR | 100 € | 200 € |

### Pourquoi ces seuils ?

Calibres sur les anomalies detectees en audit :
- BE : BX - MR Quartiers Mineurs avait 122€/0 conv sur 15 jours en mai 2026 = CRITICAL manifeste
- BE : BX - MR Watermael-Boitsfort = 41€/0 conv = WARNING
- IT : CPC tres bas (~0.53€) donc 20€ = ~38 clics sans aucune conversion = signal anormal
- CH : CPC eleve par nature, 100€ represent environ 10-15 clics seulement - seuil haut justifie

### Exceptions normales (ne pas alerter)
- Campagne nouvellement creee (< 7 jours) avec peu d'historique
- Campagne avec strategie d'encheres en apprentissage
- Campagne dont le suivi de conversion n'est pas configure (a verifier avant toute chose)

---

## CAT C — Campagne active qui ne diffuse pas

### Logique

Campagne statut = ENABLED + 0 impression sur N jours consecutifs.

- **WARNING** : 0 impression pendant 3 jours consecutifs
- **CRITICAL** : 0 impression pendant 5 jours consecutifs

### Pourquoi ce seuil ?

3 jours = suffisant pour distinguer une anomalie d'une fluctuation week-end ou d'une pause momentanee.
5 jours = certainement un probleme de configuration a corriger.

### Causes frequentes a investiguer (affichees dans l'alerte)

1. Budget journalier epuise sur toute la journee (verifier IS Lost Budget)
2. Annonces en attente de validation Google
3. Tous les mots-cles en pause ou en conflit
4. Score de qualite des annonces trop bas
5. Cible geographique trop restrictive
6. Erreur de tracking qui a bloque la campagne

---

## CAT D — Campagne limitee par le budget (IS Lost Budget)

### Logique

Impression Share perdu cause du budget trop faible.

- **WARNING** : IS Lost Budget > 20% sur 7j
- **CRITICAL** : IS Lost Budget > 40% sur 7j

Distinct de l'IS Lost Rank (pertes sur le rang d'encheres) :
- IS Lost Budget -> action : augmenter le budget
- IS Lost Rank -> action : augmenter l'encheres/tCPA

### Seuils FR et CH

Pendant la phase de collecte, les seuils sont un peu plus permissifs (30% warning / 50% critical) car l'algorithme Smart Bidding ajuste naturellement les encheres au depart.

### Signal connu : CH Heimfinder

Lors de l'audit du 4-10 mai 2026, la campagne MOON - Generique affichait 77% d'IS Lost Rank - signal critique de sous-enchere, pas de manque de budget. Le dashboard affiche les deux metriques separement pour distinguer les causes.

---

## CAT E — Search terms suspects

### Logique

Search term visible (non anonymise par Google) avec spend > seuil ET 0 conversion sur 7 jours.

Affiche dans un tableau dedie par compte : terme | campagne | clics | cout | conv | suggestion d'action.

### Seuils par compte

| Compte | Seuil spend | Devise |
|---|---|---|
| BE MR Repos | 30 € | EUR |
| IT Casa Di Riposo | 10 € | EUR |
| CA Choisir RPA | 30 | CAD |
| CA Places Senior | 30 | CAD |
| FR Wikiehpad | 15 € | EUR |
| CH Heimfinder | 50 € | EUR |

### Pourquoi cette section ?

En avril 2026 sur le compte BE, 1 697 search terms avaient un spend > 0 et 0 conversion = 3 915 € de gaspillage visible (70.2% du spend sur les search terms visibles). La section search terms du dashboard permet de cibler les pires offenseurs chaque semaine sans avoir a exporter manuellement depuis Google Ads.

### Action suggessee automatiquement

Si le terme declenche l'alerte, le dashboard suggere :
- Exclure en negatif exact si clairement hors-cible (ex : emploi, recrutement, offre d'emploi, tarif, prix, stage)
- Investiguer si ambigu (ex : "maison de retraite publique" - peut etre hors-portee du service)
- Conserver si c'est une recherche pertinente qui ne convertit pas encore (petit volume, patience)

---

## CAT F — CPC anormalement eleve

### Logique

Hausse du CPC moyen du compte par rapport a la baseline.

- **WARNING** : CPC 7j > baseline x 1.75
- **CRITICAL** : CPC 7j > baseline x 2.5

### Seuils par compte

| Compte | Devise | Baseline | WARNING | CRITICAL |
|---|---|---|---|---|
| BE MR Repos | EUR | 1.62 € | 2.84 € | 4.05 € |
| IT Casa Di Riposo | EUR | 0.524 € | 0.917 € | 1.310 € |
| CA Choisir RPA | CAD | 0.975 | 1.706 | 2.438 |
| CA Places Senior | CAD | 1.229 | 2.151 | 3.073 |
| FR Wikiehpad | EUR | N/A phase collecte | — | — |
| CH Heimfinder | EUR | N/A marche premium | pas d'alerte | pas d'alerte |

**CH : pas d'alerte CPC.** Le marche suisse a des CPC structurellement tres eleves et variables - une alerte CPC serait un bruit constant sans valeur diagnostique.

### Causes frequentes d'un CPC en hausse

1. Concurrence plus agressive sur les termes principaux (ex : concurrent qui augmente son budget)
2. Hausse des CPA cibles dans Smart Bidding (l'algo monte les encheres)
3. Changement de match type (du exact vers du large)
4. Nouveau KW a fort CPC qui capte du volume

---

## CAT G — CTR anormalement bas

### Logique

CTR moyen d'une campagne sur 7j en-dessous des seuils. Calcule uniquement pour les campagnes avec > seuil d'impressions minimum (evite les faux positifs sur petits volumes).

- **WARNING** : CTR < seuil warning
- **CRITICAL** : CTR < seuil critical

### Seuils par compte

| Compte | Baseline | WARNING | CRITICAL | Impr. minimum |
|---|---|---|---|---|
| BE MR Repos | 8.43% | < 4% | < 2% | 500 |
| IT Casa Di Riposo | ~14% | < 7% | < 4% | 500 |
| CA Choisir RPA | ~11% | < 5% | < 3% | 500 |
| CA Places Senior | ~10.9% | < 5% | < 3% | 500 |
| FR Wikiehpad | N/A | < 3% | < 1.5% | 200 |
| CH Heimfinder | N/A | < 4% | < 2% | 200 |

### Signal indicatif

Un CTR tres bas peut indiquer :
- Annonces mal alignees avec l'intention de recherche
- Extensions d'annonces manquantes (sitelinks, callouts)
- Qualite des annonces degradee
- Impression sur des termes tres generiques (broad match hors-cible)
- Positionnement trop bas (manque d'encheres)

---

## CAT H — Alertes structurelles

### H1 — Campagne en apprentissage Smart Bidding depuis trop longtemps

**Condition** : campagne avec strategie tCPA/tROAS en statut "Apprentissage" depuis > 14 jours (21 jours pour FR et CH en phase collecte).

**Niveau** : WARNING

**Cause** : l'algo Smart Bidding a besoin d'au moins 50 conversions/mois pour sortir de l'apprentissage. En dessous, il reste bloque. Action : verifier le volume de conversions, envisager de baisser le tCPA temporairement ou passer en Max Conversions.

---

### H2 — Concentration budgetaire excessive

**Condition** : une seule campagne absorbe > 65% du budget total du compte sur 7j.

**Niveau** : WARNING (INFO pour CA1 et CA2 qui ont une structure naturellement concentree)

**Exemple connu** : BX - MR Bruxelles = 58.95% du budget BE en avril 2026. Si cette campagne deraille, le compte entier en souffre.

**Action** : verifier si la concentration est voulue (campagne principale performante) ou si c'est une derive (budget d'une campagne sous-performante qui monte).

---

### H3 — CVR en chute

**Condition** : CVR du compte sur les 7 derniers jours baisse de plus de X% par rapport aux 14 jours precedents.

- **WARNING** : baisse > 30%
- **CRITICAL** : baisse > 50%

**Exemple connu** : en mai 2026 sur BE, le CVR global est reste stable (7.52% -> 7.90%) malgre les modifications de Vague 1. Mais BX - MR Quartiers Mineurs est passe de 6.1% a 0% = chute complete.

**Causes possibles** :
- Landing page down ou formulaire en erreur
- Changement de trafic (termes moins qualifies)
- Modification structurelle du compte (pause de campagnes performantes)
- Probleme de tracking

---

### H4 — RSA efficacite Faible dominante

**Condition** : > 50% des RSA actifs d'une campagne sont en efficacite "Faible" Google Ads.

**Niveau** : INFO (a traiter dans les 30 jours, pas en urgence)

**Contexte** : sur BE en avril 2026, 39.8% des RSA etaient en efficacite Faible et seulement 5.4% en Excellente. Les RSA en Faible sont un drain de performance chronique.

**Action** : diversifier les headlines, inclure le KW principal, varier les angles (benefice, question, CTA, urgence), ajouter plus de headlines et descriptions uniques.

---

## CAT I — Tracking fire (conversion silencieuse)

### Logique

CVR tombe a 0% sur N jours consecutifs alors que les clics et le spend continuent normalement.
Distinct de CAT B (spend eleve + 0 conv) : ici le compte tourne normalement en apparence — budget depense, clics recus — mais aucune conversion enregistree. Signal : formulaire casse, landing page down, tag de conversion supprime ou modifie.

**Condition** : 0 conversion sur N jours consecutifs ET clics > 50% de la moyenne 7j (le compte tourne) ET baseline conv/jour depasse le seuil minimum du compte.

- **CRITIQUE** : 2 jours consecutifs (3 jours pour comptes en phase collecte)

### Seuils par compte

| Compte | Min conv/j baseline | Jours declencheur |
|---|---|---|
| BE MR Repos | 5 | 2 |
| IT Casa Di Riposo | 10 | 2 |
| CA Choisir RPA | 3 | 2 |
| CA Places Senior | 10 | 2 |
| FR Wikiehpad | 2 | 3 |
| CH Heimfinder | 1 | 3 |

### Pourquoi ce niveau de priorite ?

En Data-Driven Attribution (le modele utilise sur tous les comptes RetraitePlus), les conversions sont fractionnelles et distribuees dans le temps. Un tracking fire peut ressembler a une simple baisse de performance pendant 24h. Sur BE a ~370€/j, 2 jours de spend avec 0 conv = 740€ de budget brule sans aucun lead genere.

### Verification immediate

1. Tester le formulaire sur la landing page manuellement
2. Verifier les evenements de conversion dans GA4 (rapport temps reel)
3. Verifier le tag Google Ads dans Tag Assistant
4. Regarder si une modification de LP ou GTM a eu lieu dans les 48h precedentes

---

## CAT J — Spike de spend anormal

### Logique

Le spend d'une journee depasse un multiple du spend quotidien moyen des 7 derniers jours.

- **WARNING** : spend 1j > moyenne 7j x 1.5
- **CRITIQUE** : spend 1j > moyenne 7j x 2.0

Valeurs identiques pour tous les comptes.

### Pourquoi surveiller ?

Smart Bidding (tCPA, Max Conversions) peut injecter massivement du budget sur une journee lors d'une phase d'exploration, apres une modification de tCPA, ou en reponse a une variation saisonniere qu'il juge favorable. Sur CA2 (~650 CAD/j budget moyen), un spike a x2.0 = 1 300 CAD depenses en une journee.

Causes frequentes :
- Modification recente du tCPA ou du budget (+20% ou plus)
- Smart Bidding en sortie d'apprentissage qui "rattrape" le volume manque
- Budget shared mal configure
- Pic de recherche lie a un evenement externe (crise sanitaire, couverture media du sujet)

### Exceptions normales

- Lundi apres un week-end faible (rebond naturel)
- Premier jour du mois (budget remis a zero)
- Apres une pause volontaire de campagne (volume rattrape)

Si le spike coincide avec l'une de ces situations, verifier quand meme que le CPL reste dans les seuils.

---

## CAT K — Ecart tCPA cible vs CPL reel

### Logique

Si le tCPA configure dans Google Ads est X et que le CPL reel sur 7j est Y, l'algorithme Smart Bidding va reduire les impressions pour "proteger" la cible. Le volume s'effondre silencieusement sans que les alertes CPL (CAT A) ne se declenchent, puisque le CPL est "correct" vu de l'algo.

- **WARNING** : CPL reel 7j > tCPA configure x 1.3
- **CRITIQUE** : CPL reel 7j > tCPA configure x 1.6

### Seuils par compte

| Compte | Multiplicateur WARNING | Multiplicateur CRITIQUE |
|---|---|---|
| BE, IT, CA1, CA2 | x 1.3 | x 1.6 |
| FR, CH (collecting) | x 1.5 | x 2.0 |

Les comptes en phase collecte ont des seuils plus permissifs car le tCPA n'est pas encore fixe de facon definitive.

### Exemple concret

Sur BE en mai 2026 : BX-MR Ixelles avait un tCPA configure a 38€. L'algorithme n'arrivait pas a convertir a ce niveau (CPL reel 119€ = x3.1). Resultat : le compte a throttle les impressions. L'alerte CAT A s'est declenchee (CPL eleve) mais CAT K aurait permis de diagnostiquer la cause specifique (tCPA trop serre) plutot que de simplement constater le symptome.

### Donnee requise

Le tCPA configure est lisible via l'API Google Ads (champ `campaign.target_cpa.target_cpa_micros`). Non disponible dans Supermetrics — necessite l'API native.

---

## CAT L — Annonce refusee sans backup

### Logique

Un groupe d'annonces se retrouve avec 0 RSA en statut "Eligible" (toutes ses RSA sont refusees, en attente de validation, ou pausees). Le groupe peut continuer a diffuser sur des ETAs historiques (obsoletes) sans que CAT C (non-diffusion) se declenche, mais avec un CTR et un CVR degrades.

**Condition** : groupe d'annonces ENABLED avec 0 RSA eligible depuis 1 jour -> CRITIQUE immediat.

**Niveau** : CRITIQUE (pas de gradation : une annonce refusee dans un groupe sans backup = probleme immediat)

### Pourquoi c'est important dans ce contexte

Sur BE en avril 2026 : 75.9% des annonces sont encore des ETAs (format obsolete). Si les RSA actives d'un groupe sont refusees, le groupe retombe sur ces ETAs non optimisables. Les ETAs ne peuvent plus etre modifiees depuis juin 2022 donc aucun correctif n'est possible sans creer une nouvelle RSA.

### Causes frequentes de refus

- Headline ou description qui contient une promesse non verifiable ("Meilleure maison de retraite")
- URL finale differente du domaine affiche (path URL)
- Contenu considere comme medical ou financier sensible (frequent dans le secteur senior)
- Repetition excessive du meme terme dans les headlines

### Donnee requise

Statut des annonces lisible via l'API Google Ads (rapport `ad_group_ad`, champ `policy_summary.approval_status`). Non disponible dans Supermetrics.

---

## Tableau de synthese complet

| Cat | Nom | Fenetre | Sev. max | API requise | Comptes |
|---|---|---|---|---|---|
| A | CPL anormalement eleve | 7j glissants | CRITIQUE | Non | BE, IT, CA1, CA2, CH |
| B | Spend eleve et 0 conversion | 7j glissants | CRITIQUE | Non | Tous |
| C | Campagne sans diffusion | Consecutif | CRITIQUE | Non | Tous |
| D | IS Lost Budget / Rank | 7j glissants | CRITIQUE | Non | Tous |
| E | Search terms suspects | 7j glissants | WARNING | Non (Supermetrics limite) | Tous |
| F | CPC anormalement eleve | 7j glissants | CRITIQUE | Non | BE, IT, CA1, CA2 |
| G | CTR anormalement bas | 7j glissants | CRITIQUE | Non | Tous |
| H | Smart Bidding, CVR, concentration, RSA | Variable | WARNING | Partiel | Tous |
| I | Tracking fire | Consecutif | CRITIQUE | Non | Tous |
| J | Spike de spend | Quotidien | CRITIQUE | Non | Tous |
| K | Ecart tCPA cible / CPL reel | 7j glissants | CRITIQUE | Oui (tCPA) | BE, IT, CA1, CA2 |
| L | Annonce refusee sans backup | Quotidien | CRITIQUE | Oui | Tous |

---

## Lecture du dashboard

### Niveaux de severite

| Couleur | Niveau | Signification |
|---|---|---|
| Rouge | CRITIQUE | Action requise dans la journee |
| Orange | WARNING | A verifier dans les 48h |
| Bleu | INFO | A traiter dans les 7-30 jours |
| Vert | OK | Tout est dans les seuils |

### Frequence de generation recommandee

- **Dashboard HTML** : generation quotidienne (script Python + cron)
- **Email alerte** : draft quotidien genere automatiquement, a envoyer manuellement apres verification

### Mise a jour des baselines

Les seuils de CPL baseline doivent etre mis a jour :
- Tous les 90 jours sur les comptes stables (BE, IT, CA1, CA2)
- Apres chaque modification majeure de compte (changement de tCPA > 20%, refonte de structure)
- FR et CH : premiere baseline a calculer apres 60 jours de donnees (fin juin 2026)

---

## Informations manquantes (a completer)

| Information | Comptes | Statut |
|---|---|---|
| Customer ID Google Ads | BE MR Repos, IT Casa Di Riposo | A renseigner |
| CPL cibles officiels client (Samuel B.) | Tous | Non fournis a ce stade |
| Frequence cron souhaitee | - | A definir |
| Acces API Google Ads | Tous | En attente approbation Basic access |
