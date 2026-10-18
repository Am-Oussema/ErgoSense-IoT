# ErgoSense --- TP0 : Étude préliminaire d'une solution IoT

**Projet :** ErgoSense --- système IoT de suivi postural avec retour
haptique\
**Niveau :** Licence 3 / dernière année de Bachelor en Internet of
Things\
**Équipe :** 3-4 étudiants\

------------------------------------------------------------------------

## Présentation de la solution

ErgoSense est un prototype qui vise à aider les personnes travaillant
longtemps assises devant un ordinateur à prendre conscience d'une
inclinaison prolongée du haut du corps. Un capteur inertiel MPU6050
mesure l'accélération et la vitesse angulaire. Un Microcontrôleur traite
localement les mesures et peut commander un moteur vibrant lorsqu'une
condition de posture définie persiste. Le dispositif envoie ensuite des
mesures résumées et des événements à une plateforme IoT pour visualiser
l'historique et les tendances.

**Limites :** le système estime l'orientation au niveau où le capteur
est fixé. Il ne diagnostique pas une maladie, ne mesure pas directement
la fatigue et ne prouve pas à lui seul qu'une posture est médicalement
mauvaise. Le placement du capteur, la calibration et les essais
utilisateurs influencent les résultats.

# Partie I --- Choix des dispositifs IoT

## 1. Quelle est la valeur ajoutée globale de la solution ?

La solution ErgoSense vise à améliorer la prise de conscience posturale
des étudiants et des personnes qui travaillent longtemps devant un
ordinateur. Elle détecte une inclinaison du haut du corps qui persiste
au-delà d'une durée définie et avertit l'utilisateur par une vibration
discrète. Les données collectées permettent de visualiser la durée des
épisodes détectés, la fréquence des alertes et l'évolution des mesures
au cours du temps sur un tableau de bord IoT. L'objectif est de fournir
un retour simple et mesurable, et non de poser un diagnostic médical.

**Valeur ajoutée attendue :** - Retour haptique local sans devoir
regarder un écran. - Historique des événements et visualisation des
tendances. - Possibilité d'ajuster le seuil et la durée après
calibration. - Fonctionnement de la détection et de la vibration même si
le Wi-Fi est indisponible.

## 2. Quelles sont les contraintes qui pourraient influencer le choix du dispositif ?

Le choix du dispositif doit prendre en compte la portabilité, le
confort, la consommation énergétique, le coût, la disponibilité des
composants, la fiabilité des mesures et la connectivité. Le capteur sera
fixé sur le haut du dos ou sur un support vestimentaire stable afin
d'estimer l'inclinaison du torse. Le traitement de détection et la
commande de vibration seront exécutés localement sur l'ESP32 pour ne pas
dépendre de la connexion Internet.

Le prototype utilisera : - **Microcontrôleur :** lecture du capteur, traitement
local et connexion Wi-Fi. - **Capteur :** mesures d'accélération sur
trois axes et de vitesse angulaire sur trois axes. - **Moteur vibrant et
étage de commande adapté :** retour haptique, à confirmer selon le
matériel disponible. - **Batterie et circuit de charge/protection
compatibles :** à choisir après vérification de la carte et des besoins
d'autonomie.

Un moteur ne doit pas être alimenté directement depuis une broche GPIO
du microcontrôleur ; un étage de commande adapté est nécessaire. La référence
exacte de la carte ESP32 et les caractéristiques du module MPU6050
doivent être vérifiées. Le MPU6050 ne mesure pas directement la posture
: celle-ci est estimée à partir des mesures et de la calibration.

## 3. À quelle fréquence le dispositif doit-il envoyer des données à la plateforme ?

Il faut distinguer la fréquence d'échantillonnage du capteur et la
fréquence de transmission réseau.

  -----------------------------------------------------------------------
  Opération               Valeur initiale         Justification
                          proposée                
  ----------------------- ----------------------- -----------------------
  Lecture du MPU6050      25 Hz                   Point de départ pour
                                                  suivre l'orientation et
                                                  les mouvements ; à
                                                  valider par des essais.

  Évaluation locale de la Environ toutes les 0,5  Permet de vérifier si
  posture                 s                       une condition persiste
                                                  plutôt que de réagir à
                                                  chaque mouvement bref.

  Envoi périodique à la   Toutes les 2 s au       Fournit une
  plateforme              départ                  visualisation presque
                                                  en temps réel sans
                                                  envoyer chaque
                                                  échantillon brut.

  Envoi d'événement       Lorsqu'une alerte       Signale rapidement les
                          commence ou se termine  changements utiles.
  -----------------------------------------------------------------------

Ces valeurs sont des paramètres initiaux, pas des résultats mesurés.
Elles seront ajustées selon la stabilité de la détection, la latence, la
qualité de la connexion et l'autonomie. La vibration, si le moteur est
disponible, sera déclenchée localement après une durée de persistance
réglable.

# Partie II --- Spécification de la topologie réseau des dispositifs IoT

## 1. Connexion directe à la plateforme ou utilisation d'une passerelle ?

Pour le prototype, l'ESP32 se connectera directement au réseau Wi-Fi
fourni par un point d'accès, par exemple le partage de connexion d'un
téléphone. Le téléphone sert uniquement de point d'accès réseau ; il
n'effectue pas le traitement applicatif et n'est pas une passerelle IoT
dédiée. L'ESP32 enverra les données via Internet à la plateforme IoT
choisie.

**Topologie proposée :**

``` text
MPU6050
   │ I²C
   ▼
ESP32 ── Wi-Fi ──► Point d’accès du téléphone
                         │ Internet
                         ▼
                  Plateforme IoT
                  (ThingsBoard envisagé)
                         │
                         ▼
                 Tableau de bord web
```

Aucune passerelle séparée n'est nécessaire pour ce prototype, car
l'ESP32 possède une connectivité Wi-Fi. Cette approche réduit le nombre
de composants et le temps de développement. Elle dépend toutefois de la
disponibilité du partage de connexion, de l'accès Internet et de la
possibilité pour la plateforme d'accepter la connexion.

## 2. Quels protocoles de communication sont prévus ?

Les protocoles proposés sont :

-   **I²C entre le MPU6050 et l'ESP32 :** bus local court pour lire les
    mesures du capteur.
-   **Wi-Fi entre l'ESP32 et le point d'accès :** adapté à un prototype
    en intérieur avec un débit suffisant et sans module radio
    supplémentaire.
-   **MQTT entre l'ESP32 et la plateforme IoT :** protocole léger de
    messagerie publish/subscribe adapté à la télémétrie. Il sera utilisé
    si l'instance de plateforme sélectionnée permet une connexion MQTT
    depuis l'ESP32.
-   **HTTPS pour l'accès au tableau de bord web :** utilisé par le
    navigateur pour consulter l'interface de la plateforme.

MQTT est le choix privilégié pour la télémétrie. HTTP/HTTPS reste une
solution de remplacement si MQTT n'est pas pris en charge ou s'il se
révèle plus difficile à configurer dans l'environnement retenu. Le choix
final dépendra des paramètres d'accès, des identifiants et de la
compatibilité de l'instance ThingsBoard utilisée.

**Point de vigilance :** le partage de connexion mobile peut entraîner
une consommation de données et une latence variables. Les messages
seront donc limités aux mesures résumées et aux événements utiles. Les
identifiants de connexion ne doivent jamais être publiés dans le dépôt
GitHub.

# Partie III --- Stockage, mise à jour et traitement des données

## 1. Comment les données sont-elles stockées sur la plateforme et à quelle fréquence sont-elles mises à jour ?

La plateforme envisagée est **ThingsBoard**, sous réserve de confirmer
qu'une instance accessible et adaptée au projet peut être utilisée. Elle
recevra des données de télémétrie de l'ESP32 et les associera à des
horodatages. Les données pourront être visualisées dans un tableau de
bord et conservées selon les capacités et la politique de rétention de
l'instance choisie. Le mode de stockage exact dépendra de l'instance
ThingsBoard utilisée ; il ne faut pas supposer qu'une base de données ou
une durée de conservation spécifique est garantie avant de vérifier sa
configuration.

**Données proposées :** - angle ou indicateur d'inclinaison estimé ; -
état de posture estimé (par exemple `normal` ou `leaning`) ; - état
d'alerte ; - durée de l'épisode détecté ou compteur d'événements ; -
horodatage ou temps écoulé depuis le démarrage.

Les noms et unités exacts des champs seront fixés pendant
l'implémentation. Il est préférable de transmettre des mesures résumées
toutes les deux secondes au début, ainsi que des événements lorsqu'une
alerte commence ou se termine, plutôt que chaque échantillon brut du
capteur.

## 2. Quels traitements ou filtres sont appliqués avant le stockage ?

Le traitement proposé sur l'ESP32 comprend : 1. Lecture périodique des
données du MPU6050. 2. Calibration d'une orientation de référence
lorsque le capteur est placé correctement. 3. Estimation de
l'inclinaison à partir des mesures inertielles. 4. Lissage simple ou
filtrage des mesures si les premiers essais montrent du bruit. 5.
Application d'un seuil et d'une durée de persistance configurables afin
d'ignorer les mouvements brefs. 6. Envoi de télémétrie résumée et
d'événements vers la plateforme.

Le filtrage exact sera choisi après observation des mesures réelles. Une
mesure issue d'un accéléromètre peut être perturbée par les mouvements
dynamiques ; il faudra donc tester le dispositif en position assise,
pendant des mouvements ordinaires et lors de changements volontaires de
posture.

Si la connexion Wi-Fi est perdue, la détection et la vibration doivent
continuer localement. La transmission reprendra lorsque la connexion
sera rétablie. La mise en mémoire tampon locale des messages non envoyés
est une amélioration possible, mais elle ne sera incluse que si le temps
de développement le permet.

# Partie IV --- Utilisation des données pour générer une meilleure valeur

## 1. Comment les données collectées peuvent-elles créer de la valeur ?

Les données d'ErgoSense peuvent apporter une valeur pratique à
l'utilisateur et fournir des indicateurs pour évaluer le prototype.

-   **Retour immédiat :** la vibration rappelle à l'utilisateur qu'une
    condition de posture configurée persiste.
-   **Suivi des habitudes :** le tableau de bord peut afficher le nombre
    d'épisodes détectés, leur durée cumulée et leur répartition dans le
    temps.
-   **Évaluation des progrès :** l'utilisateur peut comparer les
    statistiques de différentes sessions, sans interpréter ces données
    comme un diagnostic médical.
-   **Personnalisation :** les seuils et la durée de persistance peuvent
    être ajustés selon la calibration et les résultats d'essais.
-   **Évaluation technique :** les événements et les mesures permettent
    à l'équipe de comparer le comportement du système dans différentes
    situations, d'identifier les fausses alertes et d'améliorer la
    détection.
-   **Potentiel d'évolution :** une version future pourrait produire des
    rapports de tendances ou être utilisée dans des programmes de
    sensibilisation ergonomique, avec consentement des utilisateurs et
    protection des données.

La valeur principale du prototype est d'associer un retour haptique
local à une visualisation de données dans une plateforme IoT. Le tableau
de bord permet de mieux comprendre les événements détectés et d'évaluer
le fonctionnement du système. Il ne faut pas prétendre que le dispositif
prévient ou traite une maladie sans études appropriées.

## Indicateurs possibles pour démontrer le projet

Pour évaluer le prototype, l'équipe peut mesurer : - la proportion
d'épisodes volontairement provoqués qui sont détectés ; - le nombre de
fausses alertes pendant une session de test ; - le délai entre le début
d'une condition persistante et la vibration ; - la régularité de la
réception des données sur le tableau de bord ; - le comportement du
système lorsque le Wi-Fi est coupé puis rétabli.

Les résultats devront être mesurés pendant les essais et rapportés
honnêtement ; aucune performance n'est considérée comme validée dans ce
document.

------------------------------------------------------------------------

# Synthèse des choix de conception

  -----------------------------------------------------------------------
  Élément                 Choix provisoire        État
  ----------------------- ----------------------- -----------------------
  Microcontrôleur         ESP32                   Matériel déclaré
                                                  disponible ; référence
                                                  exacte à vérifier

  Capteur                 MPU6050                 Matériel déclaré
                                                  disponible ;
                                                  communication à tester

  Position du capteur     Haut du dos / support   Choix de conception
                          vestimentaire stable    

  Retour haptique         Moteur vibrant avec     Disponibilité non
                          commande adaptée        vérifiée

  Réseau local            Wi-Fi via partage de    Environnement prévu ; à
                          connexion du téléphone  tester

  Protocole de télémétrie MQTT                    Proposé, sous réserve
                                                  de compatibilité de la
                                                  plateforme

  Plateforme              ThingsBoard             Candidate, accès et
                                                  configuration à
                                                  confirmer

  Lecture du capteur      25 Hz                   Valeur initiale
                                                  proposée

  Évaluation locale       Environ 2 fois par      Valeur initiale
                          seconde                 proposée

  Transmission périodique Toutes les 2 s +        Valeur initiale
                          événements              proposée

  Stockage                Télémétrie horodatée    À confirmer selon
                          sur la plateforme       l'instance

  Traitement              Calibration,            À implémenter et tester
                          estimation, filtrage    
                          éventuel et persistance 
  -----------------------------------------------------------------------

## Références techniques à consulter

-   Documentation Espressif ESP32 :
    https://docs.espressif.com/projects/esp-idf/en/stable/esp32/
-   Documentation Arduino-ESP32 si l'implémentation utilise Arduino IDE
    : https://docs.espressif.com/projects/arduino-esp32/en/latest/
-   Documentation ThingsBoard : https://thingsboard.io/docs/
-   Spécification officielle du MPU-6050 : consulter la fiche technique
    « MPU-6000/MPU-6050 Product Specification » du fabricant.

## Prochaines étapes pratiques

1.  Identifier la référence exacte de la carte ESP32.
2.  Tester la communication I²C entre l'ESP32 et le MPU6050.
3.  Vérifier si un moteur vibrant et un circuit de commande sont
    disponibles.
4.  Créer et tester l'accès à l'instance ThingsBoard choisie.
5.  Valider la connexion MQTT avec un petit message de test.
6.  Mesurer les fréquences, la stabilité de la détection et le
    comportement lors d'une perte de réseau.

**Important :** ce document est une proposition de conception. Les
fréquences, la topologie, la plateforme et les performances doivent être
confirmées pendant l'implémentation. Les décisions finales seront mises
à jour dans le dépôt après les tests.
