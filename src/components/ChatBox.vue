<template>
    <div>
      <form @submit.prevent="sendQuestion">
        <input type="text" v-model="question" placeholder="Posez votre question ici">
        <button type="submit">Envoyer</button>
      </form>
      <div v-if="answer">
        <p>Réponse : {{ answer }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        question: '',
        answer: null
      };
    },
    methods: {
      async sendQuestion() {
        if (this.question.trim() === '') return; // Éviter d'envoyer des questions vides
  
        try {
          const response = await axios.post('http://localhost:3000/api/question', {
            question: this.question
          });
  
          this.answer = response.data.answer;
        } catch (error) {
          console.error("Erreur lors de l'envoi de la question : ", error);
          this.answer = "Nous n'avons pas pu obtenir de réponse. Veuillez réessayer.";
        }
      }
    }
  };
  </script>
  