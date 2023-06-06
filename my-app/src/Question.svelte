<script>
  import { afterUpdate, onMount } from 'svelte';

  export let question;
  export let propositions;
  export let onSelectedValue;

  function handleRadioButtonClick(event) {
    const selectedIndex = event.target.value;
    onSelectedValue(selectedIndex);
  }

  function compileFormuleMathematique() {
    var exerciceContainer = document.getElementById('questionTexte');
    exerciceContainer.innerHTML = question.texte;

    MathJax.typesetPromise([exerciceContainer]).catch((err) =>
      console.error(err)
    );
  }
  onMount(() => {
    compileFormuleMathematique();
  });

  afterUpdate(() => {
    compileFormuleMathematique();
  });
</script>

<main>
  <div class="question">
    <h1>Question {question.index}</h1>
    <div class="enonce">
      <div id="questionTexte" />
      <br />
      <div class="propositions">
        {#each propositions as proposition, index}
          <div class="proposition">
            <input
              type="radio"
              id="proposition"
              name="proposition"
              value={index}
              on:click={handleRadioButtonClick}
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
</style>
