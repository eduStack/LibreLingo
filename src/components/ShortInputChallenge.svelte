<script>
    import { slide } from "svelte/transition"
    import { onMount } from "svelte"
    import levenshtein from "js-levenshtein"
    import ChallengePanel from "./ChallengePanel"

    export let challenge
    export let registerResult
    export let resolveChallenge
    let answer = null
    let submitted = false
    let correct = null

    $: submitChallenge = () => {
        if (!answer) return
        if (submitted) return

        challenge.formInTargetLanguage.forEach(form => {
            correct =
                correct ||
                levenshtein(
                    answer
                        .toLowerCase()
                        .replace(/^\s+|\s+$/g, "")
                        .replace(/\s+/g, " "),
                    form.toLowerCase()
                ) <= 1
        })

        registerResult(correct)
        submitted = true
    }

    $: finishChallenge = () => {
        answer = null
        submitted = false
        resolveChallenge()
    }

    const focusMe = el => {
        setTimeout(() => {
            el.focus()
        }, 1)
    }
</script>

<form on:submit|preventDefault="{submitChallenge}">
    <p>
        Type
        <b>{challenge.meaningInSourceLanguage}</b>
        in Spanish!
    </p>
    <input
        tabindex="0"
        data-test="answer"
        type="text"
        class="input"
        placeholder="Type your answer…"
        disabled="{submitted}"
        use:focusMe
        bind:value="{answer}" />

    {#if answer && !submitted}
        <ChallengePanel message="" buttonText="Submit" submit />
    {/if}

    {#if submitted}
        {#if !correct}
            <ChallengePanel
                message="Incorrect solution!"
                messageDetail="{`Correct answer: ${challenge.formInTargetLanguage[0]}`}"
                buttonText="Continue"
                incorrect
                buttonAction="{finishChallenge}" />
        {/if}
        {#if correct}
            <ChallengePanel
                message="Correct solution!"
                buttonText="Continue"
                correct
                buttonAction="{finishChallenge}" />
        {/if}
    {/if}

</form>