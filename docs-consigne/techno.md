Voici l'ordre logique d'utilisation dans un pipeline de développement (du code jusqu'à la surveillance du serveur en production) avec leurs descriptions :

1. GitHub Workflow (Outil d'automatisation)
   - À quoi ça sert : C'est le moteur d'exécution intégré à GitHub. Il permet de déclencher automatiquement des suites d'actions lorsqu'un événement survient sur le code (comme un ajout ou une modification).
   - Exemple d'utilisation : Lancer automatiquement des tests de sécurité et vérifier que le code compile bien à chaque fois qu'un développeur propose une nouvelle fonctionnalité.

2. CI/CD (Concept : Intégration Continue / Déploiement Continu)
   - À quoi ça sert : C'est la méthode de travail globale (souvent implémentée grâce à des outils comme GitHub Workflow). Elle vise à automatiser tout le cheminement du code, de sa création par le développeur jusqu'à sa mise à disposition pour les utilisateurs.
   - Exemple d'utilisation : La philosophie d'une entreprise qui décide que chaque validation de code passera par des vérifications robotisées (CI) puis sera envoyée automatiquement sur les serveurs des clients (CD) sans intervention humaine.

3. Ansible (Outil de configuration et d'automatisation)
   - À quoi ça sert : C'est un outil qui permet de préparer et configurer des serveurs à distance. Au lieu de taper manuellement des commandes sur chaque machine, on décrit l'état final désiré, et Ansible s'occupe de l'appliquer partout.
   - Exemple d'utilisation : Installer la bonne version d'un serveur web, créer des utilisateurs spécifiques et appliquer des règles de sécurité sur 50 serveurs différents en même temps.

4. Prometheus (Outil de surveillance physique et collecte de métriques)
   - À quoi ça sert : C'est un système qui va régulièrement interroger diverses machines et applications pour récolter et stocker des données chiffrées sur leur état de santé et leurs performances (les "métriques").
   - Exemple d'utilisation : Un thermomètre numérique invisible qui enregistre toutes les 5 secondes la charge du processeur de vos serveurs pour voir si la machine sature aux heures de pointe.

5. Grafana (Outil de visualisation et de tableaux de bord)
   - À quoi ça sert : Il récupère les bases de données chiffrées illisibles (issues de bases comme Prometheus) et les transforme en graphiques, courbes et compteurs visuels faciles à interpréter.
   - Exemple d'utilisation : Un grand écran dans le bureau de l'équipe technique affichant une courbe dynamique du nombre d'erreurs sur le site web, qui devient rouge et clignote si un serveur tombe en panne.
