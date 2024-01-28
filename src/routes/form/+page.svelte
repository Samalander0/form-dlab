<script>
  const questions = [
    {
      id: "1A1",
      question: "What is your favorite sport?",
      options: ["1B1", "1B2", "1B3"],
      starting_question: 1
    },
    {
      id: "1B1",
      question: "Why do you like water sports?",
      when: "the response includes a water sport"
    },
    {
      id: "1B2",
      question: "What position do you play on your team?",
      when: "the response includes a team sport"
    },
    {
      id: "1B3",
      question: "How did you get into your favorite sport?",
      when: "none of the other questions fit"
    },
    {
      id: "2A1",
      question: "What sports do you currently participate in?",
      options: ["2B1", "NO_QUESTION"],
      starting_question: 2
    },
    {
      id: "2B1",
      question: "Why don't you currently participate in any sports?",
      when: "they say they don't participate in any sports"
    },
    {
      id: "NO_QUESTION",
      question: "NO_QUESTION"
    }
  ]

  let current_layer = 1
  let current_question = questions.find(question => question.starting_question == 1)
  let answer
  let loading = false
  let out_of_questions = false

  async function newQuestion() {
    if (!current_question.options) { // If the current question has no follow up questions, move on to a new layer
      newLayer()
      return;
    }

    loading = true;

    let query = `./api/question-picker/?original_question=${current_question.question}&question_answer=${answer}`

    if (current_question.options[0]) {
      const question_option = questions.find(question => question.id == current_question.options[0]);
      query = query + `&question_option_a=${question_option.question}&question_option_a_when=${question_option.when}`
    }
    if (current_question.options[1]) {
      const question_option = questions.find(question => question.id == current_question.options[1]);
      query = query + `&question_option_b=${question_option.question}&question_option_b_when=${question_option.when}`
    }
    if (current_question.options[2]) {
      const question_option = questions.find(question => question.id == current_question.options[2]);
      query = query + `&question_option_c=${question_option.question}&question_option_c_when=${question_option.when}`
    }

    query = encodeURI(query);

    console.log(query)

    const response = await fetch(query)
    const output = await response.json()
    console.log(output)

    if (output == "NO") {
      newLayer()
      return;
    }
    if (output == "A") {
      current_question = questions.find(question => question.id == current_question.options[0])
    }
    if (output == "B") {
      current_question = questions.find(question => question.id == current_question.options[1])
    }
    if (output == "C") {
      current_question = questions.find(question => question.id == current_question.options[2])
    }

    answer = "" // Clear Form Field
    loading = false
  }
  function newLayer() {
    console.log("Out of questions for this layer, moving on")

    current_layer++
    current_question = questions.find(question => question.starting_question == current_layer);

    if (!current_question) {
      out_of_questions = true
    }

    answer = ""
    loading = false
  }
</script>

<main>
  {#if out_of_questions}
    <div class="form-end">
      <h2>Thanks for filling out this form!</h2>
      <img src="https://media1.tenor.com/m/hEOM8E4epvgAAAAC/hahaha-thats-all-folks.gif" alt="That's all folks!"/>
    </div>
  {:else}
    <form on:submit={newQuestion}>
      <label for="question">{current_question?.question}</label>
      <input bind:value={answer} id="question" type="text" required/>
      <input type="submit" disabled={loading} value="Next ->"/>
    </form>
  {/if}
</main>

<style lang="scss">
  main {
    display: grid;
    place-items: center;
    height: 100vh;
    font-family: "Inter", sans-serif;
  }
  form {
    display: flex;
    flex-direction: column;
    width: 20rem;

    label {
      display: inline-block;
      font-weight: 600;
      margin-bottom: 0.5rem;
    }
    input:not([type="submit"]) {
      padding: 1em;
      margin-bottom: 1rem;
      border: 1px solid rgb(0 0 0 / 0.25);
      border-radius: 8px;
      font-family: inherit;
      background: #FAFAFA;

      &:focus {
        outline: none;
        border: 1px solid black;
      }
    }
    input[type="submit"] {
      background: black;
      border: 1px solid black;
      color: white;
      padding: 1em;
      border-radius: 8px;
      font-family: inherit;

      &:hover:not(:disabled) {
        background: transparent;
        color: black;
        cursor: pointer;
      }
      &:disabled {
        opacity: 0.5;
      }
    }
  }
  .form-end {
    text-align: center;
    img {
      border-radius: 8px;
    }
  }
</style>