## Document de conception de l'application

# Introduction

L'application est inspiré de https://dmegy.perso.math.cnrs.fr/quiz/. Cette application permet aux utilisateurs de répondre à une série de questions mathématiques et d'obtenir un score en fonction de leurs réponses. L'application prend en charge diverses fonctionnalités telles que l'ordre aléatoire des questions, une pénalité pour les réponses incorrectes et l'affichage des corrections des questions.

# Objectif

L'objectif de cette application est de permettre aux étudiants de rejoindre une session de quiz créer à partir de : "page de création de quiz".
Pour rejoindre une session de quiz l'utilisateur doit fournir une URL valide par exemple : https://phil1010.github.io/quizapp-exo7/?listeQuestions=1+2+3+4+5&rand=true&p=0.
Une fois le quiz terminer l'utilisateur aura un feedback de sa session.
L'application doit être compatible en format mobile et desktop.

# Architecture

Aperçu de l'architecture :
L'application de Quiz est construite à l'aide du framework Svelte, un framework JavaScript moderne pour la création d'interfaces utilisateur. L'application se compose de deux composants principaux : App.svelte et Question.svelte.

App.svelte : Ce composant gère la fonctionnalité globale de l'application. Il récupère les données du quiz à partir d'un fichier JSON externe (catalogue.json), initialise les variables nécessaires, gère les entrées de l'utilisateur, le chronomètre, calcule le score et contrôle le déroulement du quiz.

Question.svelte : Ce composant représente une question individuelle dans le quiz. Il reçoit les données de la question et les options de réponse en tant que propriétés, permet à l'utilisateur de sélectionner une réponse et déclenche la fonction onSelectedValue lorsque une réponse est sélectionnée.

# Interface utilisateur

L'application de Quiz dispose d'une interface utilisateur simple et intuitive. Elle comprend les éléments suivants :

Écran de démarrage : Affiche un message de bienvenue, le nombre de questions dans le quiz et un champ de saisie pour que l'utilisateur puisse entrer son pseudo (facultatif). Un bouton "Démarrer" permet à l'utilisateur de commencer le quiz.

Écran de question : Présente chaque question une par une, avec plusieurs options de réponse sous forme de cases à cocher. L'utilisateur peut sélectionner une option pour chaque question.

Boutons de navigation : Un bouton "Suivant" apparaît après chaque question. Il permet à l'utilisateur de passer à la question suivante ou de soumettre ses réponses à la fin du quiz.

Affichage du score : Après la fin du quiz, le score de l'utilisateur sur 20 est affiché, ainsi que le temps nécessaire pour terminer le quiz.

Affichage des détails : Des détails supplémentaires sont affichés, y compris le nombre de questions correctes, incorrectes et sautées.

Affichage des corrections : Une option permettant de voir les corrections de chaque question est disponible. En cliquant sur le bouton "Voir le corrigé", un tableau affiche les questions, les réponses correctes et les réponses de l'utilisateur.

Bouton de rejouer : Après la fin du quiz, un bouton "Rejouer" permet à l'utilisateur de commencer un nouveau quiz.
