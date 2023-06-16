<script>
  import { afterUpdate, beforeUpdate, onMount } from 'svelte';

  export let question;
  export let propositions;
  export let onSelectedValue;
  export let nbQuestions;
  export let cpt = 0;

  function getSelectedIndex() {
    const selectedIndex = [];
    document.querySelectorAll('input[type=checkbox]').forEach((el) => {
      if (el.checked) {
        selectedIndex.push(el.value);
      }
    });
    return selectedIndex;
  }

  function handleCheckBox() {
    const selectedIndex = getSelectedIndex();
    onSelectedValue(selectedIndex);
  }

  export function compileFormuleMathematique() {
    var exerciceContainer = document.getElementById('questionTexte');
    exerciceContainer.innerHTML = question.texte;

    MathJax.typesetPromise([exerciceContainer]).catch((err) =>
      console.error(err)
    );
  }

  afterUpdate(() => {
    compileFormuleMathematique();
    // décocher les checkbox
    document.querySelectorAll('input[type=checkbox]').forEach((el) => {
      el.checked = false;
    });
    // pour chaque question qui s'affiche leur donné un index et les incrémente
    cpt++;
  });
</script>

<main>
  <div class="question">
    <h1>Question {cpt}/{nbQuestions}</h1>
    <h3>id : {question.index}</h3>
    <div class="enonce">
      <div id="questionTexte" />
      <br />
      <div class="propositions">
        {#each propositions as proposition, index}
          <div class="proposition">
            <input
              type="checkbox"
              id="proposition"
              name="proposition"
              value={index}
              on:click={handleCheckBox}
            />
            <label for={index}>{proposition}</label>
          </div>
        {/each}
        <br />
      </div>
    </div>
  </div>
</main>

<style>
  label {
    font-size: 1.5em;
  }
</style>
