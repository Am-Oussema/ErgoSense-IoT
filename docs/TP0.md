# ErgoSense --- TP0 : Étude préliminaire d'une solution IoT

**Projet :** ErgoSense --- système IoT de suivi postural avec retour
haptique\
**Niveau :** Licence 3 / dernière année de Bachelor en Internet of
Things\
**Équipe :** 2 étudiants\
**Périmètre de cette version :** TP0, Partie I --- Choix des dispositifs
IoT\
**État :** proposition initiale à valider par des essais pratiques

------------------------------------------------------------------------

## 1. Présentation courte de la solution

ErgoSense est un prototype portable qui vise à aider les personnes
travaillant longtemps assises devant un ordinateur à prendre conscience
d'une posture penchée maintenue. Un capteur inertiel mesure
l'orientation et les mouvements du haut du corps. Un ESP32 traite
localement ces mesures et peut déclencher une vibration lorsqu'une
posture définie comme défavorable persiste au-delà d'une durée réglable.
Le dispositif envoie ensuite des résumés et des événements à une
plateforme IoT afin de visualiser l'historique et les tendances.

**Limite importante :** le prototype estime l'orientation du segment du
corps où le capteur est fixé. Il ne diagnostique pas une maladie, ne
mesure pas directement la fatigue et ne garantit pas qu'une posture soit
médicalement mauvaise. La position du capteur, la calibration et les
essais avec plusieurs utilisateurs influencent les résultats.

## 2. TP0 --- Partie I : Choix des dispositifs IoT

### Question 1 --- Quelle est la valeur ajoutée globale de la solution ?

**Réponse proposée :**

La solution ErgoSense vise à améliorer la prise de conscience posturale
des étudiants et des personnes qui travaillent longtemps devant un
ordinateur. Elle détecte une inclinaison du haut du corps qui persiste
au-delà d'une durée définie et avertit l'utilisateur par une vibration
discrète. Les données collectées permettent de visualiser la durée des
épisodes détectés, la fréquence des alertes et l'évolution des mesures
au cours du temps sur un tableau de bord IoT. L'objectif est de fournir
un retour simple et mesurable, et non de poser un diagnostic médical.

**Valeur ajoutée attendue :** - retour haptique local sans devoir
regarder un écran ; - historique des événements et visualisation des
tendances ; - possibilité de calibrer le seuil et la durée selon
l'utilisateur ; - fonctionnement de l'alerte locale même si le Wi-Fi ou
la plateforme est indisponible.

### Question 2 --- Quelles sont les contraintes qui pourraient influencer le choix du dispositif ?

**Réponse proposée :**

Le choix du dispositif doit prendre en compte la portabilité, le
confort, la consommation énergétique, le coût, la disponibilité des
composants, la fiabilité des mesures et la connectivité. Le capteur doit
être fixé de manière stable, idéalement sur le haut du dos ou sur un
support vestimentaire adapté, car la position du capteur détermine ce
que l'on peut estimer. Le dispositif doit aussi fonctionner sans
dépendre en permanence d'Internet : la détection et la vibration doivent
être exécutées localement par l'ESP32. La transmission Wi-Fi doit être
limitée à des résumés et événements utiles afin de réduire le trafic et
la consommation. Enfin, les mesures doivent être calibrées et testées,
car les accélérations pendant les mouvements peuvent perturber
l'estimation de l'inclinaison.

**Dispositifs retenus pour le prototype :**

  ---------------------------------------------------------------------------
  Élément                 Rôle                    Justification
  ----------------------- ----------------------- ---------------------------
  ESP32 (carte de         Lecture du capteur,     Offre des entrées/sorties
  développement déjà      traitement local,       adaptées au prototype et
  disponible)             connexion Wi-Fi         une connectivité Wi-Fi
                                                  intégrée sur les variantes
                                                  courantes d'ESP32. Vérifier
                                                  le modèle exact de la
                                                  carte.

  MPU6050 (module déjà    Mesure de               Suffisant pour un prototype
  disponible)             l'accélération sur 3    qui estime
                          axes et de la vitesse   l'orientation/inclinaison
                          angulaire sur 3 axes    et détecte les mouvements.
                                                  Il ne mesure pas
                                                  directement la posture et
                                                  ne possède pas de
                                                  magnétomètre.

  Moteur vibrant +        Retour haptique lors    Permet une notification
  transistor/MOSFET       d'une alerte            discrète. Ne pas alimenter
  adapté                                          directement un moteur
                                                  depuis une broche GPIO :
                                                  utiliser un étage de
                                                  commande approprié et une
                                                  alimentation adaptée.

  Batterie et module de   Alimentation portable   À sélectionner selon la
  charge/protection                               carte, le moteur et
  compatibles                                     l'autonomie visée.
                                                  Respecter les
                                                  spécifications de la
                                                  batterie et du module de
                                                  charge.
  ---------------------------------------------------------------------------

**Choix de conception :** le capteur sera porté sur le haut du dos ou
sur un support vestimentaire stable, et non simplement au poignet, pour
estimer l'inclinaison du torse. Une calibration de référence sera
effectuée lorsque l'utilisateur adopte la position de départ choisie. Le
seuil d'inclinaison et la durée de persistance seront ajustables après
les premiers essais ; ils ne doivent pas être présentés comme des seuils
médicaux universels.

### Question 3 --- À quelle fréquence le dispositif doit-il envoyer des données à la plateforme ?

Il faut distinguer la fréquence de lecture du capteur de la fréquence
d'envoi des données vers la plateforme.

**Paramètres initiaux proposés pour le prototype :**

  -----------------------------------------------------------------------
  Opération               Fréquence initiale      Pourquoi
                          proposée                
  ----------------------- ----------------------- -----------------------
  Lecture du MPU6050      25 Hz (25               Point de départ
                          échantillons/s)         raisonnable pour suivre
                                                  l'orientation et les
                                                  mouvements sans traiter
                                                  inutilement un volume
                                                  élevé de données. À
                                                  confirmer par des
                                                  essais.

  Évaluation locale de la Toutes les 0,5 s        Suffisant pour détecter
  posture                 environ (2 Hz)          une inclinaison
                                                  maintenue, avec une
                                                  logique de persistance
                                                  pour éviter les alertes
                                                  sur les mouvements
                                                  brefs.

  Envoi périodique à la   Toutes les 2 s au       Fournit une
  plateforme              départ                  visualisation presque
                                                  en temps réel sans
                                                  transmettre chaque
                                                  échantillon brut. Cette
                                                  fréquence pourra être
                                                  réduite ou augmentée
                                                  après les tests.

  Envoi d'un événement    À l'événement, par      Permet de signaler les
  important               exemple lorsqu'une      changements utiles sans
                          alerte commence ou se   attendre le prochain
                          termine                 envoi périodique.

  Vibration               Après persistance       Évite de vibrer pour
                          au-delà d'une durée     chaque mouvement bref ;
                          réglable                la durée doit être
                                                  choisie lors des essais
                                                  utilisateur.
  -----------------------------------------------------------------------

**Réponse rédigée pour le TP :**

Le MPU6050 sera lu initialement à 25 Hz. L'ESP32 évaluera localement
l'inclinaison environ toutes les 0,5 seconde et déclenchera une
vibration seulement si la condition définie persiste pendant une durée
réglable. Pour limiter le trafic réseau et la consommation, la
plateforme recevra des données résumées toutes les 2 secondes au début
du prototype, ainsi que des messages événementiels lors d'une alerte.
Ces fréquences sont des valeurs de départ à valider expérimentalement ;
elles seront ajustées selon la stabilité de la détection, la latence, la
fiabilité de la connexion et l'autonomie mesurée. La détection et la
vibration resteront locales afin de continuer à fonctionner en cas de
perte du Wi-Fi.

------------------------------------------------------------------------

## 3. Décisions prises et points encore ouverts

### Décisions initiales

-   Carte disponible : ESP32 (vérifier la référence exacte).
-   Capteur disponible : module MPU6050.
-   Position envisagée : haut du dos / support vestimentaire stable.
-   Détection et vibration : traitement local sur l'ESP32.
-   Fréquence de lecture initiale : 25 Hz.
-   Envoi initial vers la plateforme : toutes les 2 secondes, plus
    événements.
-   Plateforme IoT : pas encore choisie.

### À vérifier avant de finaliser le matériel

-   Référence exacte de la carte ESP32 et tension logique.
-   Broches I²C disponibles et câblage réel du module MPU6050.
-   Présence et caractéristiques du moteur vibrant.
-   Type de batterie, circuit de protection/charge et autonomie
    souhaitée.
-   Stabilité des mesures lorsque le capteur est porté.
-   Seuil et durée de persistance à choisir à partir de données de test,
    pas à partir d'une valeur médicale supposée.

## 4. Références techniques à consulter pendant l'implémentation

-   Documentation Espressif ESP32 :
    https://docs.espressif.com/projects/esp-idf/en/stable/esp32/
-   Fiche technique InvenSense/TDK MPU-6050 (accéléromètre et gyroscope
    6 axes) : rechercher la fiche technique officielle «
    MPU-6000/MPU-6050 Product Specification » sur le site du fabricant.
-   Documentation Arduino-ESP32 si l'implémentation utilise Arduino IDE
    : https://docs.espressif.com/projects/arduino-esp32/en/latest/

> Avant de remettre le rapport, vérifier les spécifications contre la
> référence exacte du matériel utilisé et noter les résultats réels des
> essais. Les fréquences de ce document sont des paramètres initiaux
> proposés, pas des mesures validées.

## 5. Prochaine étape

Après validation de cette Partie I, rédiger la Partie II du TP0 :
topologie réseau, choix entre connexion directe et passerelle, et
justification du protocole de communication. La plateforme et le
protocole (par exemple MQTT ou HTTP) seront sélectionnés en fonction des
exigences du TP, de la facilité de mise en œuvre et du temps disponible.
