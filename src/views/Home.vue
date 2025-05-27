<template>
    <div id="main-container">
        <section>
            <article class="genres" v-if="!categorySelected">
                <h2>Select a theme</h2>
                <ul id="genres-list">
                    <li v-for="category in categories" :key="category.id">
                        <button class="genres-title" v-on:click="categoryIsSelected(category.id, category.name)">{{ category.name }}</button>
                    </li>
                </ul>
            </article>
            <article class="levels" v-else-if="!levelSelected">
                <h2>Select a level</h2>
                <button class="btn-ariane" @click="resetQuiz()">Back to theme selection</button>
                <ul id="levels-list">
                    <li v-for="level in levels">
                        <button class="levels-title"  @click="levelIsSelected(level)">{{ level }}</button>
                    </li>
                </ul>
            </article>
            <article class="questions" v-else>
                <div v-if="questionInfo">
                    <div id="score"><span>{{ points }}</span></div>
                    <div id="points-won"><span>{{ pointsWon }}</span></div>
                    <h3 class="question-label" v-html="questionInfo.question"></h3>
                    <span class="question-number">{{ questionNumber }} / 10</span>
                    <div class="question-container">
                        <ul class="question-answer-multiple" v-if="questionInfo.type == 'multiple'">
                            <li v-for="answer in this.questionInfo.answsersList">
                                <label>
                                    <input
                                        type="radio"
                                        :value="answer"
                                        v-model="selectedAnswer"
                                        name="userAnswer"
                                    />
                                    <span class="answer-title" v-html="answer"></span>
                                </label>
                            </li>
                        </ul>
                        <ul class="question-answer-boolean" v-else>
                            <li>
                                <label>
                                    <input
                                        type="radio"
                                        value="True"
                                        v-model="selectedAnswer"
                                        name="userAnswer"
                                    />
                                    <span class="answer-title">True</span>
                                </label>
                            </li>
                            <li>
                                <label>
                                    <input
                                        type="radio"
                                        value="False"
                                        v-model="selectedAnswer"
                                        name="userAnswer"
                                    />
                                    <span class="answer-title">False</span>
                                </label>
                            </li>
                        </ul>
                        <button v-if="selectedAnswer" id="question-next" @click="checkAnswser()">{{ buttonValidation }} <span id="countdown">{{ countdown }}</span></button>
                    </div>
                </div>
                <div v-else>
                    <div class="quiz-end" v-if="questionListIsReset === false">
                        <h3>End of quiz !</h3>
                        <p v-if="points == 30">
                            Congratulations, it's a clean sweep with <span class="point">{{ points }}</span> points.
                        </p>
                        <p v-else-if="points >= 10">
                            Well done for getting  <span class="point">{{ points }}</span> points.
                        </p>
                        <p v-else-if="points > 0">
                            Not too bad, you got <span class="point">{{ points }}</span> points.
                        </p>
                        <p v-else-if="points <= 0">
                            The good thing is that you can only make progress with <span class="point">{{ points }}</span> points.
                        </p>
                        <button class="quiz-reset"  @click="resetQuiz()">Replay</button>
                    </div>
                </div>
            </article>
        </section>
    </div>
  </template>
  

  <script type="module">
    const apiUrl = "https://opentdb.com/api.php?"
    const apiCategories = 'https://opentdb.com/api_category.php';
    const apiToken ="https://opentdb.com/api_token.php?command=request";

    export default {
        name: 'Accueil',

        data(){
            return {
                points : 0,
                pointsWon : "",
                categories : null,
                categorySelected : null,
                levels: [
                    'easy',
                    'medium',
                    'hard'
                ],
                levelSelected : null,
                token : sessionStorage.getItem("token"),
                questionsList : null,
                questionInfo : null,
                questionListIsReset : false,
                questionNumber : 0,
                selectedAnswer : "",
                questionLoading : false,
                countdown: null,
                buttonValidation : "Confirm answer",
            }
        },
        mounted() {
            // on se met en haut de la page
            window.scrollTo(0, 0);
            // Récupération des genres au lancement de l'app 
            fetch(apiCategories)
                .then(response => response.json())
                .then((response) => { 
                    this.categories = response.trivia_categories;
                })
                .catch(err => console.error(err));

            
            
        },
        watch: {
            categorySelected: {
                handler: 'quizParam',
                immediate: true,
            },
            levelSelected: {
                handler: 'quizParam',
                immediate: true,
            },
        },
        methods: {
            categoryIsSelected(categoryId,categoryName){
                this.categorySelected = {
                    'id' : categoryId,
                    'name' :categoryName
                }
            },
            levelIsSelected(levelName){
                this.levelSelected = levelName;
            },
            quizParam() {
                if (this.categorySelected !== null && this.levelSelected !== null) {
                    // On verifie qu'on a bien une catégorie et un niveau de sélectionné, si c'est le cas on lance le quizz
                    this.getQuizQuestions(this.categorySelected, this.levelSelected);
                }
            },
            async getQuizQuestions(categoryId, levelName){
                // On récupère les questions en fonction de la catégorie et du niveau choisi
                try {
                    var apiUrlQuizz = `${apiUrl}amount=10&category=${categoryId.id}&difficulty=${levelName}${this.token !== null ? '&token=' + this.token : '' }`;
                    
                    const response = await fetch(apiUrlQuizz);
                    const data = await response.json();
                    
                    this.questionListIsReset = false,

                    this.questionsList = data.results;
                    this.currentQuestion();

                    await this.getToken(); 

                } catch (error){
                    console.error(error);
                }
            },
            async getToken() {
                // On récupère le token afin de ne pas faire afficher les mêmes questions à chaque fois
                try {
                    const response = await fetch(apiToken);
                    const data = await response.json();

                    this.token = data.token;
                    sessionStorage.setItem("token", `${this.token}`);
                } catch (error) {
                    console.error(error);
                }
            },
            nextQuestion() {
                // On retire la première entrée du tableau
                this.questionsList.shift();
                this.userAnswer = null;
                this.pointsWon = "";
                
                // One nettoye les inputs

                var answers = document.querySelectorAll(`.question-container input`);
                answers.forEach(answer => {
                    answer.disabled = false;
                    answer.classList.remove("bad-answer", "good-answer");
                });
                var answer = document.querySelector('input:checked');
                answer.checked = false;

                this.selectedAnswer = '';
                this.buttonValidation = "Confirm answer";

                var showPointsWon = document.getElementById('points-won');
                showPointsWon.classList.remove("bad-answer", "good-answer");
                // On met à jour la question
                this.currentQuestion();

                
            },
            currentQuestion(){
                // On met à jour la question et ses réponses en prenant la première entrée du tableau
            
                this.questionInfo = this.questionsList[0];
                if(this.questionsList[0]){
                    var badAnswers = this.questionsList[0].incorrect_answers;
                    var goodAnswers = this.questionsList[0].correct_answer;
                    badAnswers = Object.values(badAnswers);
                    badAnswers.push(goodAnswers);
                    this.answersQuestion = badAnswers;
                    this.shuffleArray(this.answersQuestion);
                    this.questionInfo.answsersList = this.answersQuestion;
                    this.questionNumber += 1;
                }
            },
            shuffleArray(array) {
                // on fait un mélange des réponses pour éviter que la bonne question soit toujours placé au même endroit
                for (let i = array.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [array[i], array[j]] = [array[j], array[i]];
                }
                return array;
            },
            checkAnswser(){
                // On empêche l'utilisateur de sélectionner une autre réponse
                var answer = document.querySelector(`.question-container input[value="${this.selectedAnswer}"]`);
                var otherAnswers = document.querySelectorAll(`.question-container input:not([value="${this.selectedAnswer}"])`);
                var showPointsWon = document.getElementById('points-won');

                otherAnswers.forEach(otherAnswer => {
                    otherAnswer.disabled = true;
                });

                // On vérifie si la réponse sélectionnée est la bonne
                if(this.questionsList[0].correct_answer === this.selectedAnswer){
                    setTimeout(() => {
                        this.points += 2;
                    }, 1000);
    
                    this.pointsWon = "+2";
                    answer.classList.add("good-answer");
                    showPointsWon.classList.add("good-answer");
                }else{
                    setTimeout(() => {
                        this.points -= 1;
                    }, 1000);

                    var goodAnswerWas = document.querySelector(`.question-container input[value="${this.questionsList[0].correct_answer}"]`);

                    goodAnswerWas.classList.add("good-answer");

                    this.pointsWon = "-1"
                    answer.classList.add("bad-answer");
                    showPointsWon.classList.add("bad-answer");
                }

                // On enclanche le découmpte
                this.startCountdown();

                if(this.questionNumber == 10){
                    this.buttonValidation = "End of quiz in";
                }else{
                    this.buttonValidation = "Next question in";
                }
                
                var goNextQuestion = setTimeout(this.nextQuestion, 3500);
            },
            startCountdown(){
                // on fait un décompte avant la prochaine question
                this.countdown = 3;

                var timer = setInterval(() => {
                    if (this.countdown > 0) {
                        this.countdown--
                    } else {
                        clearInterval(timer);
                        this.countdown = null;
                    }
                },999);
               
            },
            resetQuiz() {
                // On reset le quizz
                this.questionInfo = null;
                this.questionsList = [];
                this.categorySelected = null;
                this.levelSelected = null;
                this.points = 0;
                this.questionListIsReset = true;
                this.questionNumber = 0;
            }
        },
    }
</script>