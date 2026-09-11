# Budget

Application de suivi de comptes personnels en **un seul fichier HTML**. Elle s'ouvre par
double-clic, fonctionne hors-ligne, sans installation, sans serveur, sans compte et sans
dépendance externe : les graphiques, le parseur CSV et le générateur de fichiers Excel sont
écrits à la main dans le fichier.

**Aucune donnée ne quitte le navigateur.** Tout est stocké dans le `localStorage` de la machine.

## Utilisation

Ouvre `index.html` dans un navigateur récent. Au premier lancement, choisis de partir de zéro
ou de charger un jeu de démonstration pour explorer l'interface.

Trois relevés d'exemple sont fournis pour tester l'import :

- `exemple-releve-montant-signe.csv` — une colonne de montant, négatif pour les sorties ;
- `exemple-releve-debit-credit.csv` — colonnes débit et crédit séparées, valeurs entre guillemets ;
- `exemple-releve-epargne.csv` — relevé d'un Livret A, pour voir l'absorption des jambes de
  répartition (voir plus bas).

## Ce que l'application fait

### Comptes et transactions
- Plusieurs comptes, avec solde initial, couleur, archivage et **mots-clés** servant à
  reconnaître automatiquement les virements entre tes propres comptes.
- **Le type suit le signe** : un montant positif est un revenu, un montant négatif une dépense.
  Rien à choisir, et aucune ligne ne peut afficher un type qui contredit son montant. Une simple
  case « virement interne » marque en plus les mouvements entre tes propres comptes.
- Saisie et modification manuelles, notes et tags libres.
- Total personnalisable : tous les comptes ou seulement certains.
- **Sélection multiple** dans la liste : une case par ligne, une case « tout » à trois états qui
  ne porte que sur le résultat filtré, **Maj+clic** pour cocher une plage entière. Une barre
  d'actions apparaît en bas avec le nombre et le total de la sélection, et permet d'affecter une
  catégorie, corriger le sens des montants ou le marquage « virement interne », ajouter ou retirer
  des tags, marquer comme professionnel, ou supprimer. Chaque action de masse s'annule d'un `Ctrl+Z`.
- Un **virement interne peut recevoir une catégorie**, comme n'importe quelle autre transaction.
  Un virement vers une épargne reçoit d'ailleurs automatiquement la catégorie de nature « épargne »
  s'il n'en a pas. La somme du camembert par catégorie retombe toujours exactement sur le total des
  dépenses.
- **Les virements internes se comptent tout seuls**, sans réglage. Tout repose sur le
  « périmètre dépensable » : tes comptes sélectionnés qui ne sont pas des épargnes. Une jambe de
  virement compte si, et seulement si, son propre compte est dans ce périmètre et sa contrepartie
  en est dehors.

  | Mouvement | Effet sur le résultat |
  | --- | --- |
  | Compte courant → Livret A | la sortie compte, l'entrée non : de l'argent mis de côté |
  | Compte courant → compte joint suivi | neutre, rien ne compte des deux côtés |
  | Compte courant → compte non suivi | la sortie compte, l'argent a quitté le périmètre |
  | Livret A → compte courant | l'entrée compte : une reprise sur l'épargne est un revenu |

  Un mouvement neutre est **exclu** des deux côtés plutôt que compté deux fois : le résultat serait
  juste dans les deux cas, mais compter les deux jambes gonflerait les totaux bruts de revenus et
  de dépenses du même montant, faussant « quel pourcentage de mes revenus je dépense ».
  Sur un virement ventilé, chaque destination est jugée séparément.

### Import CSV
- Glisser-déposer, détection automatique du séparateur, de l'encodage (UTF-8 / ISO-8859-1)
  et du rôle de chaque colonne.
- Deux formats : montant signé unique, ou débit et crédit séparés.
- **Presets par compte** : le format est mémorisé et réappliqué automatiquement — sauf s'il ne
  sait pas lire le fichier déposé, auquel cas la détection automatique reprend la main.
- **Anti-doublon** par empreinte (compte + date + montant + libellé) : ré-importer le même
  relevé n'ajoute rien.
- Aperçu avant validation et **annulation d'un import entier** en un clic.

### Catégorisation
- Catégories de dépense et de revenu, ajoutables, renommables, supprimables, avec
  sous-catégories.
- **Ordre personnalisé** : glisse la poignée ⠿ ou utilise les flèches ▲▼ pour ranger tes
  catégories comme tu veux, dépenses et revenus séparément. Deux tris rapides — alphabétique et
  par montant dépensé sur la période — servent de point de départ, et restent modifiables à la
  main. Cet ordre s'applique **partout** : menus de saisie, file « À valider », filtres, règles,
  classement en masse, tableau des budgets et export. Les statistiques, elles, restent classées
  par montant décroissant : c'est un classement, pas une liste.
- Deux axes supplémentaires par catégorie de dépense : **fixe / variable** et
  **essentiel / loisir / épargne**.
- Moteur de **règles** éditables (contient, commence par, égal, expression régulière, plage de
  montant, compte, sens), évaluées par priorité.
- **Apprentissage explicite** : recatégoriser une transaction propose la création d'une règle,
  applicable à l'historique. Rien ne se crée en douce.
- File **« À valider »** : y atterrissent les lignes qu'aucune règle ne sait classer et les
  virements vers des particuliers non reconnus.
- **Moteur de régularité** : même libellé, montant à ±10 %, même jour du mois à ±2 jours, vu au
  moins 2 fois. Un mouvement ainsi reconnu ne déclenche plus de question. Les seuils sont
  réglables. Le même moteur détecte les abonnements et dépenses récurrentes.

### Répartition de l'épargne
Un seul virement part du compte courant, et il alimente plusieurs épargnes : Livret A, PER,
pockets. L'application sait ventiler ce montant unique.

- **Ventilation** d'un virement sortant entre plusieurs comptes, en **euros ou en pourcentage**,
  au choix ligne par ligne. Le reste à répartir s'affiche en direct, et l'enregistrement est refusé
  tant que le compte ne tombe pas juste. Le résidu de centimes est reporté sur la dernière ligne :
  une répartition en tiers ne fait pas dériver les soldes.
- Pour chaque destination, l'application **réutilise la ligne du relevé** si elle existe déjà,
  sinon **elle la crée** — les soldes de tes épargnes sont donc justes même si tu ne télécharges
  jamais leurs relevés.
- **Absorption** : si tu importes le relevé de l'épargne plus tard, la vraie ligne prend la place
  de celle qui avait été créée. Une seule transaction subsiste, le solde ne double pas, et le lien
  vers le virement d'origine est conservé.
- **Modèles de répartition** nommés et réutilisables. Un modèle reconnaît le virement à son libellé
  et s'applique au mois suivant, puis la répartition passe par la file « À valider » pour
  confirmation. Une fois confirmée et le mouvement reconnu comme habituel, elle s'applique
  silencieusement.
- Modifier une répartition ne détruit jamais une ligne venue d'un vrai relevé : seules les lignes
  créées par l'application sont ajustées ou retirées.
- **Ventiler un virement ne change pas ton résultat du mois** : la sortie et les entrées s'annulent,
  comme pour n'importe quel mouvement interne.

### Vue Épargne
- Total épargné, versé sur la période, **effort d'épargne** (versements rapportés aux revenus),
  avancement global des objectifs.
- Répartition par compte et par **groupe** (Précaution, Retraite, Projets…).
- Tableau par épargne : solde, part du total, versements de la période, objectif et versement
  mensuel nécessaire pour tenir l'échéance.
- Évolution du total épargné, historique des répartitions, liste des modèles.
- Les **objectifs adossés à un compte** se remplissent tout seuls depuis son solde : rien à
  ressaisir chaque mois.

### Analyse
- Pourcentages de revenu, de dépense, de reste à dépenser et de dépassement.
- Répartition fixe / variable et essentiel / loisir / épargne.
- Budgets par catégorie, avec projection de fin de mois et objectifs d'épargne.
- Comparatifs mois précédent et moyenne des mois connus, principaux bénéficiaires,
  évolution du solde, carte de chaleur annuelle par catégorie.
- Filtres par mois, année ou intervalle libre ; recherche avancée enregistrable.

### Onglet Pro — entreprise individuelle
Calcule le **chiffre d'affaires nécessaire pour atteindre un revenu net donné**.

- Régime **micro-entreprise** ou **réel** (BIC réel / BNC déclaration contrôlée).
- Quatre catégories d'activité — BIC vente, BIC prestations, BNC libéral non réglementé (SSI),
  BNC libéral réglementé (CIPAV) — et **activité mixte** avec répartition du CA.
- ACRE, TVA (franchise ou assujetti, prix fixés HT ou TTC), CFE avec ses exonérations,
  charges professionnelles, foyer fiscal, versement libératoire.
- Régime réel calculé **branche par branche** (maladie, indemnités journalières, retraite de
  base et complémentaire, invalidité-décès, allocations familiales, CSG-CRDS, cotisations
  minimales), ou par taux global en mode simplifié.
- Le calcul inverse net → CA se fait par **balayage puis dichotomie**, et non par formule :
  la cascade est pleine de paliers (tranches d'impôt, cotisations minimales, seuils micro et
  TVA, exonération de CFE sous 5 000 € de CA) qu'une formule fermée traiterait mal.
- Comparateur **micro vs réel** avec le point de bascule : à partir de quel niveau de charges
  le réel devient plus avantageux.
- Scénarios nommés, comparables et exportables.

> **Les taux préchargés sont des points de départ, pas une vérité fiscale.** La législation
> évolue chaque année et plusieurs paramètres ont changé récemment (taux de cotisations BNC
> relevés par paliers, seuils de franchise en base de TVA remaniés). Chaque jeu d'année est
> marqué « à vérifier » tant que tu ne l'as pas confirmé, et l'écran **Pro → Vérifier mes taux**
> liste chaque valeur avec les sources officielles à consulter. Ce simulateur est indicatif et
> ne remplace pas un expert-comptable.

### Personnalisation
- Couleur d'accent, fond, texte, police, densité, arrondi des angles, mode sombre.
- Position du menu : à gauche, à droite ou en haut.
- Tableau de bord en blocs **déplaçables** (poignée ⠿), redimensionnables (3, 4, 6, 8 ou
  12 colonnes sur 12) et masquables.
- Colonnes du tableau des transactions configurables ; thème exportable et importable.

### Sauvegarde et exports
- **Sauvegarde complète `.json`** — c'est le fichier à conserver.
- **Classeur `.xlsx`** multi-onglets (Transactions, Résumé mensuel, Catégories, Comptes, Épargne
  et, si le profil existe, Objectif pro), généré sans librairie externe.
- **CSV** à séparateur point-virgule avec BOM, prêt pour Excel français.
- Impression et PDF via le navigateur.
- Instantanés automatiques avant chaque import (5 derniers conservés) et annulation `Ctrl+Z`.
- **Verrouillage optionnel par mot de passe** (AES-GCM, clé dérivée par PBKDF2), désactivé
  par défaut. Le mot de passe n'est stocké nulle part : perdu, les données sont illisibles.
  Exporte une sauvegarde avant de l'activer.

## Raccourcis clavier

| Touche | Action |
| --- | --- |
| `N` | Nouvelle transaction |
| `/` | Recherche |
| `Ctrl+Z` | Annuler |
| `Échap` | Fermer une fenêtre, ou vider la sélection |
| `Ctrl+A` | Tout sélectionner (vue Transactions) |
| `Maj+clic` | Cocher une plage de lignes |
| `1` … `9` | Naviguer entre les onglets |

## Comment marquer une épargne

Onglet **Comptes** → modifier le compte → cocher **« c'est une épargne »**. Donne-lui un groupe
(Précaution, Retraite, Projets…) et, si tu veux, un objectif. Il apparaît alors dans l'onglet
**Épargne** et devient une destination proposée dans l'éditeur de répartition.

## Sauvegarde : à lire

Les données vivent dans le `localStorage` du navigateur. Vider les données du site, changer de
navigateur ou naviguer en mode privé les fait disparaître. **Exporte régulièrement une
sauvegarde `.json`** et conserve-la ailleurs. L'application rappelle la date de la dernière
sauvegarde dans les réglages.

## Détails techniques

Fichier unique, sans build ni dépendance. Persistance `localStorage`, chiffrement optionnel via
WebCrypto. Graphiques en SVG généré (anneau, barres, courbe, cascade, carte de chaleur, jauges).
Générateur `.xlsx` maison : archive ZIP en méthode « stored » avec CRC32 et parties OOXML
minimales. Parseur CSV maison gérant les guillemets, les séparateurs `;` `,` tabulation `|` et
les retours à la ligne échappés.
