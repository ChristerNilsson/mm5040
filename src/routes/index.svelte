<script context="module">
	export const ssr = false;
</script>

<script>
	import _ from 'lodash'
	import {permutations} from 'lodash.permutations'
	const SYMBOLS = '0123456789abcdef'
	let M = 4
	let N = 10
	let command = ""
	let facit = ""
	let guess = ""
	let cands = []
	let errors = []
	let headers = []
	let history = []

	const assert = console.assert
	const log = console.log
	const pack = (digits) => digits.join("")

	const init = () => _(SYMBOLS.substring(0,N)).permutations(M).map(v => _.join(v, '')).value()

	const newGame = (cmds) => {
		history = []
		if (cmds.length > 1) M = parseInt(cmds[1])
		if (cmds.length > 2) N = parseInt(cmds[2])
		if (N > 16) {
			lines = ["N must be less or equal to 16"]
			return
		}
		if (M > 6) {
			lines = ["M must be less or equal to 6"]
		  return
		}
		facit = SYMBOLS.substring(0,N)
		facit = _.shuffle(facit)
		facit = pack(facit.slice(0,M))
		guess = ""
		cands = init()

		headers = []
		headers.push(`Select ${M} unique digits from ${SYMBOLS.substring(0,N)}!`)
		headers.push(`Candidates: ${cands.length}`)
		headers = headers
		command = ""
	}

	const makeAnswer = (f,g) => { // facit,guess
		const m = f.length
		const res = []
		for (const i in _.range(m))
			for (const j in _.range(m))
				if (f[i] == g[j]) 
					res.push(Math.abs(i-j))
		res.sort()
		return pack(res)
	}
	assert(makeAnswer("1234","5678") == "")
	assert(makeAnswer("1234","1678") == "0")
	assert(makeAnswer("1234","1278") == "00")
	assert(makeAnswer("1234","1235") == "000")
	assert(makeAnswer("1234","1234") == "0000")
	assert(makeAnswer("1234","3241") == "0123")
	assert(makeAnswer("1234","4321") == "1133")
	assert(makeAnswer("1234","3412") == "2222")
	assert(makeAnswer("1234","4561") == "33")

	const reduce = (cands,guess) => {
		const res = []
		const answer1 = makeAnswer(facit,guess)
		for (const cand in cands) {
			const answer2 = makeAnswer(cand, guess)
			if (answer1 == answer2)
				res.push(cand)
		}
		return res
	}

	const handleGuess = (guess) => {
		if (guess.length != M || _.uniq(guess).length != M) {
			if (guess.length > 0) {
				errors = [`Select ${M} unique digits from ${SYMBOLS.substring(0,N)}!`]
			} else {
				errors = []
			}
			return 
		}

		const answer = makeAnswer(facit,guess)
		cands = reduce(cands,guess)
		history.push(`${guess} [${answer}] (${cands.length})`)
		if (answer == '0000') history.push(`Solved in ${history.length} guesses!`)
		history = history
		command=''
	}

	const handler = (event) => {
		log(event)
		errors = []
		const cmds = command.split(' ')
		if (cmds.length == 0) {
			errors = []
		} else if (cmds[0]=='n') {
			newGame(cmds)
		} else {
			handleGuess(cmds[0])
		}
	}

	const help = () => {
		headers = []
		headers.push('n = new Game')
		headers.push('n 4 = new Game selecting 4 digits')
		headers.push('n 4 10 = new Game selecting 4 digits from 0123456789')
		headers.push('Hit enter to continue')
	}

	help()

</script>

<input bind:value={command} on:change={handler}>
{#each errors as error}
	<div style="color:red">{error}</div>
{/each}
{#each headers as header}
	<div>{header}</div>
{/each}
{#each history as line}
	<div>{line}</div>
{/each}