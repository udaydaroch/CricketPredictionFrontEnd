<template>
  <div id="app">
    <div class="card">
      <form @submit.prevent="predictOutcome">
        <div class="form-group">
          <label for="team1">Team 1:</label>
          <select v-model="match.team1">
            <option v-for="team in teams" :key="team" :value="team">{{ team }}</option>
          </select>
        </div>

        <div class="form-group">
          <label for="team2">Team 2:</label>
          <select v-model="match.team2">
            <option v-for="team in teams" :key="team" :value="team">{{ team }}</option>
          </select>
        </div>

        <div class="form-group">
          <label for="toss_winner">Toss Winner:</label>
          <select v-model="match.toss_winner">
            <option v-if="match.team1 && match.team2" :value="match.team1">{{ match.team1 }}</option>
            <option v-if="match.team1 && match.team2" :value="match.team2">{{ match.team2 }}</option>
          </select>
        </div>

        <div class="form-group">
          <label for="gender">Gender:</label>
          <select v-model="match.gender">
            <option value="male">Male</option>
            <option value="female">Female</option>
          </select>
        </div>

        <div class="form-group">
          <label for="venue">Venue:</label>
          <select v-model="match.venue">
            <option v-for="venue in venues" :key="venue" :value="venue">{{ venue }}</option>
          </select>
        </div>

        <button type="submit">Predict Outcome</button>
      </form>

      <div v-if="prediction" class="prediction-result">
        <p>Predicted Winner: {{ prediction.predicted_winner }}</p>
        <p>Probability: {{ prediction.predicted_probability.toFixed(2) }}%</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      match: {
        team1: '',
        team2: '',
        toss_winner: '',
        gender: '',
        venue: ''
      },
      teams: [],
      venues: [],
      prediction: null
    };
  },
  created() {
    this.fetchMatchData();
  },
  methods: {
    fetchMatchData() {
      axios.get('http://localhost:5000/matches')
          .then(response => {
            this.teams = response.data.teams;
            this.venues = response.data.venues;
          })
          .catch(error => {
            console.error('Error fetching match data:', error);
          });
    },
    predictOutcome() {
      // Ensure team1 and team2 are different
      if (this.match.team1 === this.match.team2) {
        alert('Team 1 and Team 2 must be different.');
        return;
      }

      axios.post('http://localhost:5000/predict', this.match)
          .then(response => {
            this.prediction = response.data;
          })
          .catch(error => {
            console.error('Error predicting outcome:', error);
          });
    }
  },
  watch: {
    'match.team1'(newTeam1) {
      // Adjust team2 options based on team1 selection
      if (newTeam1 === this.match.team2) {
        this.match.team2 = '';
      }
    },
    'match.team2'(newTeam2) {
      // Ensure toss_winner is either team1 or team2
      if (newTeam2 !== this.match.team1 && newTeam2 !== '') {
        this.match.toss_winner = newTeam2;
      } else {
        this.match.toss_winner = this.match.team1;
      }
    }
  }
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #435d75;
  height: 98vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #b1c0ce;
  margin: 0;
}

.card {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  min-width: 500px;
  max-height: 700px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.form-group {
  margin-bottom: 16px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

select {
  margin-bottom: 8px;
  padding: 8px;
  font-size: 16px;
  width: 100%;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 20px;
  width: 100%;
}

button:hover {
  background-color: #0056b3;
}

.prediction-result {
  margin-top: 20px;
}

p {
  font-size: 18px;
  margin: 0;
}
</style>
