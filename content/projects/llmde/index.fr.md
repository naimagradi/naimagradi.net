---
title: "llmde"
pitch: "Extraction de données assistée par IA pour les revues systématiques, chaque valeur reliée à la page et à la citation dont elle provient."
icon: wand-magic-sparkles
weight: 40
meta:
  - Thèse de doctorat, Université de Genève, 2026
  - Avec [Mathieu Scheltienne](https://mathieu.scheltienne.net), ingénierie du paquet
  - "Rôle : direction du projet, méthode d’extraction et conception des prompts"
  - Open source, licence MIT
skills: ["Conception de flux de travail LLM", "Synthèse des preuves", "Recherche reproductible"]
links:
  - label: Voir sur GitHub
    url: https://github.com/naimagradi/llmde
note: "Le modèle a atteint une précision de 97 % face à l’extraction manuelle."
---

**Un pipeline d’extraction par modèle de langage pour une revue systématique et une méta-analyse, construit pour que
chaque valeur porte sa page et sa citation et qu’une relectrice humaine garde le dernier mot.**

## L’outil

Extraire à la main les données de 39 essais est lent et varie d’une lectrice à l’autre. llmde confie la partie
mécanique à un modèle de langage sous des règles strictes : extraire exactement sans interpréter, renvoyer un null
explicite pour toute information absente, et citer la page et le passage derrière chaque valeur.

{{< figure src="images/extraction-architectures.png" alt="Trois architectures d’extraction de données par modèle de langage : appel unique tel que mis en œuvre, agentique, agentique avec passage humain" caption="Figure 2 de la thèse. Le panneau (a) est la conception mise en œuvre : un appel par article et par prompt, chaque valeur avec sa page et sa citation, confrontée à une extraction manuelle. Les panneaux (b) et (c) sont les variantes agentiques, la seconde avec un passage humain, proposées mais non construites." >}}

## La construction des prompts

Chaque prompt a été rédigé, testé sur deux ou trois articles volontairement variés en parallèle d’une lecture
humaine, puis révisé partout où les deux lectures divergeaient. Chaque écart a été diagnostiqué avant toute
modification, la plupart venant d’une définition à double lecture ou d’une variante de rapport sans règle prévue.

{{< figure src="images/calibration-loop.fr.svg" alt="Boucle de calibration des prompts : le modèle et un humain lisent le même article, les écarts sont diagnostiqués et le prompt révisé jusqu’à l’accord" caption="La boucle a tourné sur deux ou trois articles variés jusqu’à l’accord entre modèle et humain ; le prompt a ensuite été figé et appliqué à tout le corpus." >}}

## Les livrables

- **Extraction en double.** 39 essais pour la revue et 19 pour la méta-analyse, une fois à la main et une fois par
  le modèle (Claude Opus 4.6), réconciliées par discussion, avec une précision de 97 % du modèle face à
  l’extraction manuelle.
- **Paquet open source.** llmde sur GitHub, licence MIT : cinq prompts d’extraction, une instruction système, des
  connecteurs Claude et Gemini, une interface en ligne de commande qui reprend les exécutions interrompues.
- **Méthode documentée.** Chapitre 5 de la thèse : la conception à appel unique à côté de deux variantes
  agentiques, avec un plaidoyer pour conserver un passage humain.

## Mon rôle

J’ai dirigé le projet : j’ai conçu la méthode d’extraction et corédigé les prompts et les schémas, et
[Mathieu Scheltienne](https://mathieu.scheltienne.net) les a implémentés sous forme de paquet. J’ai assuré le volet
modèle de l’extraction en double et l’ai réconcilié avec le volet manuel.

## Compétences démontrées

- Des flux de travail LLM conçus avec la vérification intégrée : trace de preuve, nulls explicites, contrôles
  structurels, version de modèle figée.
- Une méthode de synthèse des preuves : extraction en double et calibration sur un échantillon pilote, comme le
  recommandent les guides de revue.
- La capacité à spécifier une méthode assez précisément pour qu’un ingénieur en fasse un outil réutilisable.
