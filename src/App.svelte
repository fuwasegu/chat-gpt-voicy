<script>
	let spacePressed = false;
	let userMessage = '';
	let aiMessage = '';
	let apiKey = '';
	let loading = false;

	let messageHistories = [];


	class GptMessage
	{
		constructor(role, content){
			this.role = role;
			this.content = content;
		}

		toObject() {
			return {
				role: this.role,
				content: this.content,
			};
		}
	}

	const createRecognition = () => {
		const SpeechRecognition = webkitSpeechRecognition || SpeechRecognition;
		const recognition = new SpeechRecognition();
		recognition.interimResults = true;
		recognition.continuous = true;
		recognition.lang = 'ja-JP';

		return recognition;
	};

	const postMessage = async (histories) => {
		if (apiKey == '') {
			alert('ChatGPT の API KEY を設定してください');
			return '';
		}
		const body = JSON.stringify({
			messages: histories,
			model: "gpt-3.5-turbo",
		});

		const res = await fetch("https://api.openai.com/v1/chat/completions", {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
				Authorization: `Bearer ${apiKey}`,
			},
			body,
		});

		const data = await res.json();

		const choice = 0;

		return data.choices[choice].message.content;
	};

	const readMessageWithVoice = (text) => {
		const speech = new SpeechSynthesisUtterance();
		speech.text = text;
		speech.lang = 'ja-JP';
		speech.rate = 1.2;
		speech.pitch = 1;
		speech.volume = 1;
		window.speechSynthesis.speak(speech);
	}

	const recognition = createRecognition();

	recognition.onresult = (event) => {
		userMessage = Object.keys(event.results).map((key) => event.results[key][0].transcript).join();
	};

	// （PC用）スペースキーが押されたときのイベントリスナー
	window.addEventListener('keydown', (event) => {
		if (event.code === 'Space' && !spacePressed) {
			spacePressed = true;
			// ここで処理を開始
			recognition.start();
		}
	});
	window.addEventListener('keyup', async (event) => {
		if (event.code === 'Space') {
			spacePressed = false;
			// ここで処理を停止
			recognition.stop();

			// ユーザーのメッセージを履歴に記録
			messageHistories.push(new GptMessage('user', userMessage));
			messageHistories = messageHistories;

			// ChatGPT に送る
			loading = true;
			aiMessage = await postMessage(messageHistories);
			loading = false;

			// ChatGPTのメッセージを履歴に記録
			messageHistories.push(new GptMessage('assistant', aiMessage));
			messageHistories = messageHistories;

			// メッセージを読ませる
			readMessageWithVoice(aiMessage);
		}
	});

	// （スマホ用）画面が押されたときのイベントリスナー
	// const main = document.getElementById('main');
	// main.addEventListener('touchstart', (event) => {
	// 	if (event.code === 'Space' && !spacePressed) {
	// 		spacePressed = true;
	// 		// ここで処理を開始
	// 		recognition.start();
	// 	}
	// })
	// main.addEventListener('touchend', async (event) => {
	// 	if (event.code === 'Space') {
	// 		spacePressed = false;
	// 		// ここで処理を停止
	// 		recognition.stop();

	// 		messageHistories.push(new GptMessage('user', userMessage));

	// 		loading = true;
	// 		aiMessage = await postMessage(messageHistories);
	// 		loading = false;

	// 		messageHistories.push(new GptMessage('assistant', aiMessage));
	// 		readMessageWithVoice(aiMessage);

	// 		console.log(messageHistories);
	// 	}
	// });
</script>

<main id="main">
	<h1>s AI 藤さん</h1>

	<div>
		<label for="api-key">
			<span style="margin-right: 16px;">API キー</span>
			<input bind:value={apiKey} id="api-key">
		</label>
	</div>

	<!-- <div class="speak-btn">
		<p>話す</p>
	</div> -->

	<div style="height: 100px;">
		<p style="color: #FF0000; font-size:24px; margin-bottom:0px">あなた</p>
		<small style="color:gray;">スペースキーを押しながら喋ってください</small>
		<p>{userMessage}</p>
	</div>

	<div style="margin-top: 48px; height: 100px;">
		<p style="color: #0000FF; font-size:24px;">わたし</p>
		{#if loading}
			<img src="loading.gif" alt="loading">
		{/if}
		{#if !loading}
			<p>{aiMessage}</p>
		{/if}
	</div>

	<hr />
	<h2>会話ログ</h2>

	{#each messageHistories as history, i}
		<p>
			<strong>{history.role}</strong>: {history.content}
		</p>
	{/each}
</main>

<style>
	main {
		text-align: center;
		justify-content: center;
		align-items: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		font-size: 4em;
		font-weight: 100;
	}

	.speak-btn {
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100px;
		height: 100px;
		background: #6fa1ff;
		color: #FFF;
		text-decoration: none;
		text-align: center;
		margin: auto;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
