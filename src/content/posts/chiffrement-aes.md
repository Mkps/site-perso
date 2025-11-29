---
title: "Chiffrement AES"
published: 2025-11-28
draft: false
tags: ['cryptographie', 'cybersecurite']
---

## Qu'est-ce que le chiffrement AES

Le chiffrement AES, pour *Advanced Encryption Standard* (fr:*Norme de Chiffrement Avancé*), est un algorithme de chiffrement symétrique largement répandu et conçu pour être efficace et rapide tout en offrant un haut niveau de sécurité.
Un chiffrement est dit symétrique quand la même clé est utilisé pour chiffré et déchiffré les données.

L'AES est basé sur le chiffrement de blocs de 16 octets par une clé de 16, 24 ou 32 octets(128, 192 ou 256 bits), cette norme est un sous-ensemble de l'algorithme de **Rijndael**, ce dernier pouvant utiliser des blocs et clés de taille multiple de 32 entre 128 et 256 bits. Le chiffrement AES dispose de plusieurs modes:
  - ECB mode: Electronic Code Book mode
  - CBC mode: Cipher Block Chaining mode
  - CFB mode: Cipher FeedBack mode
  - OFB mode: Output FeedBack mode
  - CTR mode: Counter mode

---
### Le mode ECB - Electronic CodeBook

Dans ce mode, chaque bloc de 16 octets est chiffré indépendamment à l'aide de la même clé. C'est le mode le plus simple mais aussi le moins sécurisée.

**Fonctionnement :**
1. Le message est divisé en blocs de 16 octets.
2. Chaque bloc est chiffré séparément en utilisant AES.
3. Les blocs chiffrés sont concaténés pour former le message chiffré final.

**Illustration :**
```plaintext
Message clair : [Bloc 1] [Bloc 2] [Bloc 3]
Chiffrement AES : [Chiffre 1] [Chiffre 2] [Chiffre 3]
```

Chaque bloc produit un même résultat si le contenu d'origine est identique, car aucun lien ou dépendance n'existe entre les blocs.

#### Quelles est sa faiblesse?
Le chiffrement AES-ECB offre une faible résistance à l'analyse cryptographique dû à la possibilité d'exploiter les motifs répétés dans les blocs. Exemple connu le pingouin Linux passé au AES-ECB peut toujours être deviné.

---
### Le mode CBC – *Cipher Block Chaining*

Le mode CBC introduit une dépendance entre les blocs pour renforcer la sécurité.

**Fonctionnement :**
1. Le premier bloc est combiné (par XOR) avec un **vecteur d’initialisation** (IV), puis chiffré.
2. Chaque bloc suivant est XORé avec le **bloc chiffré précédent** avant chiffrement.

**Avantages :**
* Moins vulnérable que ECB : les blocs identiques n’ont pas le même chiffrement. Ce qui permet une meilleure diffusion.
* Nécessite un VI aléatoire pour chaque message.

**Inconvénients :**
* Le chiffrement/déchiffrement ne peut pas être parallélisé.
* Sensible à une modification du message en cas de perte ou de corruption.

---

### Le mode CFB – *Cipher Feedback*

Le mode CFB transforme AES en **chiffrement par flot**, utile pour des données de taille non multiple de 16 octets.

**Fonctionnement :**
1. Le chiffrement du IV est utilisé comme un flux pseudo-aléatoire.
2. Ce flux est XORé avec les données en clair pour produire le texte chiffré.
3. Le texte chiffré est ensuite injecté dans le processus suivant.

**Avantages :**
* Permet de chiffrer des petits morceaux de données (bit, octet).
* Supporte le **chiffrement en continu**.

**Inconvénients :**
* Ne permet pas le déchiffrement parallèle.
* Erreurs de transmission peuvent se propager.

---

### Le mode OFB – *Output Feedback*

Le mode OFB est similaire à CFB, mais le texte chiffré n’est pas réinjecté dans le système.

**Fonctionnement :**
1. Le IV est chiffré pour produire un flux de bits.
2. Ce flux est XORé avec les données en clair pour produire le texte chiffré.
3. La sortie de l’AES est réutilisée comme entrée pour le tour suivant.

**Avantages :**
* Rend AES entièrement en mode **flot**, sans propagation d'erreur.
* Chiffrement et déchiffrement sont **symétriques**.

**Inconvénients :**
* Vulnérable aux attaques par relecture si le IV est réutilisé.
* Plus lent que CTR en général.

---

### Le mode CTR – *Counter*

Le mode CTR transforme AES en un **chiffrement par flot** très performant, adapté au chiffrement parallèle.

**Fonctionnement :**
1. Un compteur aléatoire est choisi puis est incrémenté à chaque bloc.
2. Ce compteur est chiffré, et le résultat est XORé avec le bloc de texte en clair.

**Avantages :**
* Le chiffrement/déchiffrement est **parallélisable**.
* Résistant aux erreurs de transmission.
* Très rapide et couramment utilisé (ex: TLS, GCM).

**Inconvénients :**
* Le compteur (ou le nonce) ne doit **jamais être réutilisé** avec la même clé.

---

### Le mode GCM – *Galois/Counter Mode*

Le mode GCM combine le mode **CTR** avec un **code d’authentification** (MAC) basé sur les opérations de type **Galois Field**. Il permet le chiffrement **authentifié**, aussi appelé **AEAD** (*Authenticated Encryption with Associated Data*).

**Fonctionnement :**
1. Le chiffrement se fait en mode **CTR**.
2. Un **tag d’authentification** est généré via une fonction de hachage universelle sur les données chiffrées et des données associées (AAD).
3. À la réception, ce tag est vérifié pour s’assurer que les données n’ont pas été modifiées.

**Avantages :**
* Confidentiel **et** authentifié (confidentialité + intégrité).
* **Très rapide**, bien optimisé matériellement.
* Utilisé dans **TLS v1.2 et v1.3**, **IPsec**, etc.

**Inconvénients :**

* Le **nonce** (vecteur unique) ne doit **jamais être réutilisé** avec la même clé.

---

### Le mode CCM – *Counter with CBC-MAC*

Le mode CCM est un autre mode AEAD (chiffrement authentifié) combinant :
* **CTR** pour le chiffrement,
* **CBC-MAC** pour l’authentification.

**Fonctionnement :**
1. Le message est authentifié via un CBC-MAC.
2. Puis il est chiffré en CTR.

**Avantages :**
* Fournit confidentialité + intégrité.
* Utilisé dans **IEEE 802.15.4**, **Wi-Fi WPA2**, **Bluetooth LE**.

**Inconvénients :**
* Moins performant que GCM.
* Plus difficile à implémenter efficacement.

---

## Résumé des modes d’AES

| Mode    | Type             | Authentifié | Parallélisable | Cas d’usage typique                   |
| ------- | ---------------- | ----------- | -------------- | ------------------------------------- |
| **ECB** | Bloc             | ❌           | ✅              | Démonstration, jamais en production   |
| **CBC** | Bloc (chaîné)    | ❌           | ❌              | Fichiers anciens, obsolète en TLS     |
| **CFB** | Flot (pseudo)    | ❌           | ❌              | Chiffrement en continu                |
| **OFB** | Flot (pseudo)    | ❌           | ❌              | Environnements bruités, données audio |
| **CTR** | Flot (compteur)  | ❌           | ✅              | Performances, volume élevé de données |
| **GCM** | AEAD (CTR + MAC) | ✅           | ✅              | TLS, SSH, IPsec, HTTPS moderne        |
| **CCM** | AEAD (CTR + CBC) | ✅           | ❌              | Wi-Fi, Zigbee, Bluetooth LE           |
