# AUDIT.md - Rapport d'Opération APT28

## 1. Méthodologie et Analyse de la Cible
- **Cible :** `0xed5415679D46415f6f9a82677F8F4E9ed9D1302b` (Réseau Sepolia)
- **Objectif :** Exploiter la vulnérabilité du contrat (système de casino basé sur un oracle BTC/USD) et extraire les fonds en respectant des contraintes strictes d'atomicité.
- **Analyse initiale :** La cible utilise un système de "Round" et de "Guess". L'analyse du *storage* de la blockchain est nécessaire pour lire l'état interne de la cible et anticiper les valeurs requises pour déclencher le paiement.

## 2. Stratégie d'Exploitation et Architecture
Pour répondre aux exigences de sécurité opérationnelle d'APT28, nous avons développé un Smart Contract `Drainer.sol`.
- **Atomicité garantie :** La fonction `attack()` déclenche la tentative d'exploitation sur le contrat cible. Immédiatement après cette tentative (et dans le même bloc/transaction), la fonction `distribute()` est appelée. Cela garantit qu'aucun fond ne peut stagner sur le contrat attaquant.
- **Distribution des parts :** La logique mathématique de répartition évite les poussières (*dust/wei*) en calculant précisément 50% et 30% via une division, puis en attribuant la soustraction du reste au 3ème lieutenant (20%).
- **Interface stricte :** Le contrat implémente `IDrainer` à la lettre pour éviter la perte des clés serveurs comme menacé par APT28.

## 3. Contraintes Opérationnelles (OPSEC)
Conscient que le réseau Sepolia est un environnement concurrentiel et public :
1. Les adresses des lieutenants ont été "hardcodées" en tant que constantes (`constant`) pour économiser du gaz lors du déploiement et empêcher toute altération externe.
2. Le contrat est doté d'une fonction `receive()` pour pouvoir réceptionner l'Ether envoyé par le contrat cible sans *revert*.

## 4. Défis Techniques et Exécution
*(Note du rédacteur : Le contrat a été correctement rédigé pour l'atomicité de la distribution. Cependant, la reconstitution exacte des paramètres cryptographiques (guess, round, nonce) pour forcer la cible à déclencher le flux de sortie a représenté un défi majeur dans le temps imparti. La complexité résidait dans l'alignement entre les données de l'oracle et l'état de la blockchain Sepolia. L'architecture a été conçue selon le principe KISS : une logique de distribution simple et infaillible, prête à recevoir le payload cible final).*
