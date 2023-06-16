<script>
  import Question from './Question.svelte';
  import { onMount } from 'svelte';
  import jQuery from 'jquery';

  // catalogue.json
  let donnees;
  // numéros des questions à afficher
  let listeQuestions = [];
  // liste contenant l'index et le texte de chaque question
  let questions = [];
  // index de la question en cours
  let questionIndex = 0;
  // liste des propositions de toutes les questions
  let propositions = [];
  // liste des corrections de toutes les questions
  let corrections = [];
  // indice de la proposition sélectionnée par l'utilisateur pour chaque question
  let selectedValue = null;
  // score +1 sinon enlever autant de points que le pourcentage de pénalité
  let scoreSur20 = 0;
  let noteSur100 = 0;
  let nbQuestionReussi = 0;
  let nbQuestionEchoue = 0;
  let nbQuestionSaute = 0;

  // tableau contenant les valeurs sélectionnées par question
  let tabValeurSelectionner = [];

  const urlDeLaPage = new URLSearchParams(window.location.search);
  const listeQuestionsString = urlDeLaPage.get('c');
  const rand = urlDeLaPage.get('r');
  const penalite = urlDeLaPage.get('p');
  let idQuizHache = urlDeLaPage.get('i');
  let nickName = urlDeLaPage.get('n');

  const userID = getUserId();
  let resultatAEnvoyerAuPHP = [];

  let nbQuestions;

  let timer; // Référence de l'intervalle du timer
  let temps = 0; // Temps écoulé en secondes

  onMount(() => {
    fetchData();
    getListeQuestions();
  });

  function generateUniqueID() {
    // Générer un ID unique, par exemple en utilisant un algorithme de hachage ou une bibliothèque externe
    // Dans cet exemple, nous utilisons simplement un timestamp pour des fins de démonstration
    return Date.now().toString();
  }

  function startTimer() {
    timer = setInterval(() => {
      temps++;
    }, 1000);
  }
  function stopTimer() {
    clearInterval(timer); // Arrêter l'intervalle du timer
  }

  function handleSelectedValue(value) {
    selectedValue = value;
  }

  function shuffleQuestions() {
    // Utilisation de l'algorithme de Fisher-Yates pour mélanger les questions
    for (let i = questions.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [questions[i], questions[j]] = [questions[j], questions[i]];
      [propositions[i], propositions[j]] = [propositions[j], propositions[i]];
      [corrections[i], corrections[j]] = [corrections[j], corrections[i]];
    }
  }

  async function fetchData() {
    try {
      const response = await fetch(
        'https://raw.githubusercontent.com/Phil1010/quiz-exo7-stage2023/main/catalogue.json'
      );
      donnees = await response.json();
      fetchQuestions();
    } catch (error) {
      console.error('Erreur lors de la récupération des données :', error);
    }
    if (listeQuestionsString != null) getNbQuestions();
  }

  function fetchQuestions() {
    if (donnees) {
      donnees.forEach((element, index) => {
        if (listeQuestions.includes(index)) {
          const newQuestion = { index: index, texte: element.texte };
          questions = [...questions, newQuestion];
          propositions = [
            ...propositions,
            element.reponses.map((reponse) => reponse.texte),
          ];
          corrections = [
            ...corrections,
            element.reponses.map((reponse) => reponse.correct),
          ];
        }
      });

      if (rand == '1') {
        shuffleQuestions();
      }
    }
  }

  function demarrer() {
    document.querySelector('.accueil').style.display = 'none';
    document.querySelector('#btnNext').style.display = '';
    document.querySelector('.question').style.display = '';
    startTimer();
    if (nickName == null) {
      nickName = document.querySelector('#pseudo').value;
    }
  }

  function nextQuestion() {
    questionIndex++;
    if (selectedValue == corrections[questionIndex - 1].indexOf(true)) {
      scoreSur20++;
      nbQuestionReussi++;
      resultatAEnvoyerAuPHP.push(1);
    } else if (selectedValue == null || selectedValue.length > 1) {
      nbQuestionSaute++;
      resultatAEnvoyerAuPHP.push(0);
    } else {
      const scorePenalite = scoreSur20 - p / 100; // Applique la pénalité en soustrayant le pourcentage p du score sur 20
      scoreSur20 = scorePenalite;
      nbQuestionEchoue++;
      resultatAEnvoyerAuPHP.push(-1);
    }
    if (questionIndex == questions.length) {
      if (scoreSur20 < 0) {
        scoreSur20 = 0;
      }
      scoreSur20 = (scoreSur20 * 20) / questions.length;
      finQuiz();
    }
    tabValeurSelectionner.push(propositions[questionIndex - 1][selectedValue]);
    selectedValue = null;
  }

  function finQuiz() {
    document.querySelector('.score').style.display = '';
    document.querySelector('#btnNext').style.display = 'none';
    document.querySelector('#btnRejouer').style.display = '';
    document.querySelector('.timer').style.display = '';
    document.querySelector('.detailsQuestions').style.display = '';
    document.querySelector('#voirCorrection').style.display = '';
    stopTimer();
    getNoteSur100();
    resultatAEnvoyerAuPHP = resultatAEnvoyerAuPHP.map((resultat) => {
      if (resultat == -1) {
        return 2;
      }
      if (resultat == 0) {
        return 0;
      }
      if (resultat == 1) {
        return 1;
      }
    });
    resultatAEnvoyerAuPHP = resultatAEnvoyerAuPHP.join('');
    envoyerDonneesQuizFini();
    //envoyerStats();
  }

  function envoyerDonneesQuizFini() {
    if (idQuizHache == null) idQuizHache = '';
    if (nickName == null) nickName = '';

    jQuery.ajax({
      url: 'https://damienmegy.xyz/php/quizapp-exo7/quiz_fini.php',
      method: 'POST',
      data: {
        quizID: JSON.stringify(idQuizHache),
        quizContent: JSON.stringify(listeQuestionsString),
        userID: JSON.stringify(userID),
        userName: JSON.stringify(nickName),
        results: JSON.stringify(resultatAEnvoyerAuPHP),
        elapsedTime: JSON.stringify(temps),
        note: JSON.stringify(noteSur100.toString()),
        penalite: JSON.stringify(penalite),
      },

      success: function (response) {
        console.log('Réponse du server : ');
        console.log(response);
      },
      error: function (xhr, status, error) {
        console.error(error);
      },
    });
  }

  function envoyerStats() {
    var quizID = idQuizHache;
    var xhr = new XMLHttpRequest();
    xhr.open(
      'POST',
      'http://localhost/quizapp-exo7/backend/quiz_stats.php',
      true
    );
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

    xhr.onreadystatechange = function () {
      if (xhr.readyState === XMLHttpRequest.DONE) {
        if (xhr.status === 200) {
          var responseData = JSON.parse(xhr.responseText);
          console.log('Réponse du serveur :', responseData);
          // Traitement des données retournées
        } else {
          console.error(
            'Erreur lors de la requête :',
            xhr.status,
            xhr.statusText
          );
          // Gestion des erreurs
        }
      }
    };

    var data = 'quizID=' + encodeURIComponent(quizID);
    xhr.send(data);
  }

  function rejouer() {
    if (nickName != '') {
      const url = new URL(window.location.href);
      url.searchParams.set('n', nickName); // Ajouter le nouveau pseudo comme paramètre 'n' dans l'URL
      window.location.href = url.toString(); // Recharger la page avec les nouveaux paramètres
    } else {
      window.location.reload();
    }
  }

  function cliqueCorrection() {
    affichageCorrection();
    document.querySelector('#voirCorrection').style.display = 'none';
    document.querySelector('#cacherCorrection').style.display = '';
  }

  function affichageCorrection() {
    var exerciceContainer = document.getElementById('ligneEnonceCompile');
    var table = document.createElement('table'); // Création du tableau
    table.style.borderCollapse = 'collapse'; // Ajout de la propriété de fusion des bordures

    // Création de la première ligne avec les titres
    var headerRow = table.insertRow();
    var questionHeaderCell = headerRow.insertCell();
    questionHeaderCell.innerHTML = 'Questions';
    questionHeaderCell.style.fontWeight = 'bold';
    questionHeaderCell.style.border = '1px solid white'; // Ajout de la bordure à la cellule du titre

    var correctionHeaderCell = headerRow.insertCell();
    correctionHeaderCell.innerHTML = 'Corrections';
    correctionHeaderCell.style.fontWeight = 'bold';
    correctionHeaderCell.style.border = '1px solid white'; // Ajout de la bordure à la cellule du titre

    var correctionHeaderCell = headerRow.insertCell();
    correctionHeaderCell.innerHTML = 'Vos réponses';
    correctionHeaderCell.style.fontWeight = 'bold';
    correctionHeaderCell.style.border = '1px solid white';

    questions.forEach((element) => {
      var row = table.insertRow(); // Insertion d'une nouvelle ligne dans le tableau

      var questionCell = row.insertCell(); // Insertion de la cellule pour la question
      questionCell.innerHTML =
        'Q.' +
        [questions.indexOf(element) + 1] +
        '/' +
        nbQuestions +
        ' Id.' +
        element.index +
        ' : ' +
        element.texte;
      questionCell.style.border = '1px solid white'; // Ajout de la bordure à la cellule

      var correctionCell = row.insertCell(); // Insertion de la cellule pour la correction
      correctionCell.innerHTML =
        propositions[questions.indexOf(element)][
          corrections[questions.indexOf(element)].indexOf(true)
        ];
      correctionCell.style.border = '1px solid white'; // Ajout de la bordure à la cellule

      var reponseCell = row.insertCell(); // Insertion de la cellule pour la reponse donné par l'utilisateur
      reponseCell.innerHTML = tabValeurSelectionner[questions.indexOf(element)];
      reponseCell.style.border = '1px solid white'; // Ajout de la bordure à la cellule
      if (reponseCell.innerHTML == 'undefined') {
        reponseCell.innerHTML = 'Question sauté / Réponse invalide';
      }
    });
    exerciceContainer.appendChild(table); // Ajout du tableau au conteneur

    MathJax.typesetPromise([exerciceContainer]).catch((err) =>
      console.error(err)
    );
  }

  let p = 0; // Pourcentage de pénalité initial

  if (penalite) {
    const parsedP = parseInt(penalite);
    if (!isNaN(parsedP) && parsedP >= 0 && parsedP <= 100) {
      p = parsedP;
    } else {
      console.error(
        'Erreur lors de la récupération du pourcentage de pénalité :',
        penalite
      );
    }
  }

  function getNbQuestions() {
    nbQuestions = listeQuestionsString.split(' ').length;
  }

  function getUserId() {
    let userID = localStorage.getItem('userID');

    if (!userID) {
      // Générer un nouvel ID utilisateur unique
      userID = generateUniqueID();

      // Stocker l'ID utilisateur dans le localStorage
      localStorage.setItem('userID', userID);
    }

    return userID;
  }

  function getListeQuestions() {
    if (listeQuestionsString == null) {
      document.querySelector('.accueil').innerHTML =
        "<h1>Bienvenue !</h1> <br> <h3>Pour commencer, veuillez saisir une liste de question dans les paramètres de l'URL. <br> Voici par exemple un quiz contenant les question : 1,2,3,4 et 5 avec l'ordre d'affichage des questions randomnisées et sans pénalité : <br> <a href='https://phil1010.github.io/quizapp-exo7/?c=1+2+3+4+5&r=1&p=0'>https://phil1010.github.io/quizapp-exo7/?c=1+2+3+4+5&r=1&p=0</a> </h3> ";
      document.querySelector('#btnNext').style.display = 'none';
      return;
    }
    listeQuestions.push(...listeQuestionsString.split(' '));
    listeQuestions = listeQuestions.map((question) => parseInt(question));
  }

  function getNoteSur100() {
    noteSur100 = (scoreSur20 * 100) / 20;
  }
</script>

<main>
  <div class="accueil">
    {#if listeQuestionsString != null}
      <h1>Bienvenue !</h1>
      {#if nickName == null}
        Entrez votre pseudo (facultatif)
        <br />
        <br />
        <input
          id="pseudo"
          type="text"
          style="border-radius: 5%;"
          maxlength="15"
        />
        <br />
      {:else if nickName != ''}
        <h3>Bonjour {nickName}</h3>
      {/if}
      <h3>
        Le quiz contient {nbQuestions} questions. <br />
        Si vous etes prêt(e) appuyez sur le bouton commencer !
      </h3>
      <br />

      <button id="btnStart" on:click={demarrer}> Commencer </button>
    {/if}
  </div>

  <div class="question" style="display: none;">
    {#if questions[questionIndex]}
      <Question
        question={questions[questionIndex]}
        propositions={propositions[questionIndex]}
        onSelectedValue={handleSelectedValue}
        nbQuestions={questions.length}
      />
    {/if}
  </div>
  <button id="btnNext" on:click={nextQuestion} style="display: none;">
    Next
  </button>
  <div class="score" style="display: none;">Score : {scoreSur20} / 20</div>
  <div class="timer" style="display: none;">
    Temps: {temps}s
  </div>
  <div class="detailsQuestions" style="display: none;">
    Nombre de bonne réponse : {nbQuestionReussi} <br /> Nombre de mauvaise
    réponse : {nbQuestionEchoue} <br /> Nombre de question sauté : {nbQuestionSaute}
  </div>
  <br />
  <div id="ligneEnonceCompile" style="margin-bottom: 5rem;" />

  <br />
  <button id="voirCorrection" on:click={cliqueCorrection} style="display: none;"
    >Voir le corrigé</button
  >
  <br />
  <button
    id="cacherCorrection"
    on:click={() => {
      document.querySelector('#voirCorrection').style.display = '';
      document.querySelector('#cacherCorrection').style.display = 'none';
      document.querySelector('#ligneEnonceCompile').innerHTML = '';
    }}
    style="display: none;">Caché le corrigé</button
  >
  <br />

  <br />
  <button id="btnRejouer" on:click={rejouer} style="display: none;">
    Rejouer
  </button>

  <div />
</main>

<style>
  /* stacker les boutons en bas centrer */
  button {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 10rem;
    height: 50px;
    border: none;
    color: white;
    text-align: center;
    /* centrer */
    margin-left: auto;
    margin-right: auto;
    left: 0;
    right: 0;
    font-size: 16px;
    margin-bottom: 2rem;
  }
  /* bouton voir correction au dessus du bouton rejouer */
  #voirCorrection {
    position: fixed;
    bottom: 50px;
    left: 0;
    width: 10rem;
    height: 50px;
    border: none;
    color: white;
    text-align: center;
    /* centrer */
    margin-left: auto;
    margin-right: auto;
    left: 0;
    right: 0;
    font-size: 16px;
    margin-bottom: 4rem;
  }
  #cacherCorrection {
    position: fixed;
    bottom: 50px;
    left: 0;
    width: 10rem;
    height: 50px;
    border: none;
    color: white;
    text-align: center;
    margin-left: auto;
    margin-right: auto;
    left: 0;
    right: 0;
    font-size: 16px;
    margin-bottom: 4rem;
  }

  /* augmenter la taille de l'input du pseudo */
  input {
    width: 15rem;
    height: 2rem;
  }
</style>
