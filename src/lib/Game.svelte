<script>
    import dictionnary from '../assets/dictionnary.json'

    const accentsTidy = function (s) {
        var r = s.toLowerCase();
        r = r.replace(new RegExp(/\s/g), "");
        r = r.replace(new RegExp(/[àáâãäå]/g), "a");
        r = r.replace(new RegExp(/æ/g), "ae");
        r = r.replace(new RegExp(/ç/g), "c");
        r = r.replace(new RegExp(/[èéêë]/g), "e");
        r = r.replace(new RegExp(/[ìíîï]/g), "i");
        r = r.replace(new RegExp(/ñ/g), "n");
        r = r.replace(new RegExp(/[òóôõö]/g), "o");
        r = r.replace(new RegExp(/œ/g), "oe");
        r = r.replace(new RegExp(/[ùúûü]/g), "u");
        r = r.replace(new RegExp(/[ýÿ]/g), "y");
        r = r.replace(new RegExp(/\W/g), "");
        return r;
    }

    function decoratePropositions(wordToFind, propositions) {
        let propositionsDisplayable = []
        propositions.forEach(proposition => {
            let propositionDecorated = proposition.map((letter, letterPosition) => {
                let color = 'transparent'
                if (wordToFind[letterPosition] === letter) {
                    color = 'red'
                } else if (wordToFind.filter(l => l === letter).length > 0) {
                    let letterInTheWord = wordToFind.filter(l => l === letter).length
                    let letterWellPositionned = wordToFind.filter((l, i) => {
                        return l === proposition[i] && l === letter
                    }).length

                    let letterBeforeNotWellPositionned = proposition.filter((l, i) => {
                        console.log(letter, letterPosition, l, i)
                        return l === letter && i < letterPosition && l !== wordToFind[i]
                    }).length

                    if (letterInTheWord - letterWellPositionned > letterBeforeNotWellPositionned) {
                        color = 'orange'
                    }

                    console.log({
                        letter: letter,
                        letterPosition: letterPosition,
                        letterWellPositionned: letterWellPositionned,
                        letterBeforeNotWellPositionned: letterBeforeNotWellPositionned
                    })
                }
                return {value: letter, color: color}
            })
            propositionsDisplayable = [...propositionsDisplayable, propositionDecorated]
        })

        return propositionsDisplayable
    }

    function recalculateInitialProposition(wordToFind, propositions) {
        let newProposition = []

        for (let i = 0; i < wordToFind.length; i++) {
            let letter = '.'
            propositions.forEach(proposition => {
                if (proposition[i] === wordToFind[i]) {
                    letter = wordToFind[i]
                }
            })

            newProposition = [...newProposition, letter]
        }

        return newProposition
    }

    function isWordInDictionnary(word) {
        return dictionnary.find(w => accentsTidy(w) === word) !== undefined
    }

    function randomWord(wordMaxSize) {
        let theWord = accentsTidy(dictionnary[Math.floor(dictionnary.length * Math.random())].toLowerCase())
        if (theWord.length > wordMaxSize) {
            return randomWord(wordMaxSize)
        }
        return theWord
    }

    let shake = false
    let wordToFind = Array.from(randomWord(6))
    console.log(wordToFind.join(''))
    let inputWord = ''

    let initialProposition = wordToFind.map(letter => '.')
    initialProposition[0] = wordToFind[0]

    let propositions = [initialProposition]
    let propositionsDisplayable = decoratePropositions(wordToFind, propositions)

    function submit(e) {
        e.preventDefault()
        if (inputWord.length !== wordToFind.length) {
            return
        }

        if (!isWordInDictionnary(inputWord)) {
            shake = true
            setTimeout(_ => shake = false, 600)
            return
        }

        propositions.pop()
        propositions = [...propositions, Array.from(inputWord)]

        if (wordToFind.join('') === accentsTidy(inputWord)) {
            alert('Gagné !')
        } else {
            propositions = [...propositions, recalculateInitialProposition(wordToFind, propositions)]
        }
        inputWord = ''

        propositionsDisplayable = decoratePropositions(wordToFind, propositions)
    }

    function keyUp(e) {
        e.preventDefault()

        propositionsDisplayable.pop()
        let newWord = inputWord
        if (inputWord === '') {
            newWord = recalculateInitialProposition(wordToFind, propositions).join('')
        }
        for (let i = newWord.length; i < wordToFind.length; i++) {
            newWord += '.'
        }
        propositionsDisplayable = [...propositionsDisplayable, Array.from(newWord).map(letter => {
            return {value: letter, color: 'transparent'}
        })]
    }

</script>

<form on:submit={submit}>
    <input class:shake="{shake}" type="text" maxlength={wordToFind.length} bind:value={inputWord} on:keyup={keyUp}/>
</form>

<ol>
    {#each propositionsDisplayable as proposition, i}
        <li>
            {#each proposition as letter, j}
                <span style="padding: 0 15px 0 15px; background-color: {letter.color}">{letter.value}</span>
            {/each}
        </li>
    {/each}
</ol>

<style>
    .shake {
        /* Start the shake animation and make the animation last for 0.5 seconds */
        animation: shake 0.5s;

        /* When the animation is finished, start again */
        animation-iteration-count: initial;
    }

    @keyframes shake {
        0% {
            transform: translate(1px, 1px) rotate(0deg);
        }
        10% {
            transform: translate(-1px, -2px) rotate(-1deg);
        }
        20% {
            transform: translate(-3px, 0px) rotate(1deg);
        }
        30% {
            transform: translate(3px, 2px) rotate(0deg);
        }
        40% {
            transform: translate(1px, -1px) rotate(1deg);
        }
        50% {
            transform: translate(-1px, 2px) rotate(-1deg);
        }
        60% {
            transform: translate(-3px, 1px) rotate(0deg);
        }
        70% {
            transform: translate(3px, 1px) rotate(-1deg);
        }
        80% {
            transform: translate(-1px, -1px) rotate(1deg);
        }
        90% {
            transform: translate(1px, 2px) rotate(0deg);
        }
        100% {
            transform: translate(1px, -2px) rotate(-1deg);
        }
    }
</style>