<script>
  import Question from './Question.svelte';
  import { onMount } from 'svelte';

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
  // score +1 si bonne réponse 0 sinon
  let score = 0;

  const listeQuestionsDansURL = new URLSearchParams(window.location.search);
  const listeQuestionsString = listeQuestionsDansURL.get('listeQuestions');
  const rand = listeQuestionsDansURL.get('rand');

  function getListeQuestions() {
    if (listeQuestionsString == null) {
      document.querySelector('.question').textContent = 'Quiz non disponible.';
      document.querySelector('#btnNext').style.display = 'none';
      return;
    }
    listeQuestions.push(...listeQuestionsString.split(','));
    listeQuestions = listeQuestions.map((question) => parseInt(question));
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
  }

  onMount(() => {
    fetchData();
    getListeQuestions();
  });

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

      if (rand == 'true') {
        shuffleQuestions();
      }
    }
  }

  function nextQuestion() {
    questionIndex++;
    if (selectedValue == corrections[questionIndex - 1].indexOf(true)) {
      score = score + 1;
    } else {
      score = score;
    }
    if (questionIndex == questions.length) {
      finQuiz();
    }
  }

  function finQuiz() {
    document.querySelector('.score').style.display = '';
    document.querySelector('#btnNext').style.display = 'none';
  }
</script>

<main>
  <div class="question">
    {#if questions[questionIndex]}
      <Question
        question={questions[questionIndex]}
        propositions={propositions[questionIndex]}
        onSelectedValue={handleSelectedValue}
      />
    {/if}
  </div>
  <button id="btnNext" on:click={nextQuestion}> Next </button>
  <div class="score" style="display: none;">Score : {score}</div>
</main>

<style>
  /* Ajoutez ici vos styles CSS personnalisés */
</style>
