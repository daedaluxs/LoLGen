<template>
  <div class="min-h-screen min-w-screen flex flex-col items-center">
    <!-- Controls Section -->
    <div class="w-full max-w-md px-4 mt-8 md:mt-20">
      <div class="flex flex-col items-center space-y-4">
        <input type="number" v-model="numberOfChamps" min="1" max="50"
          class="text-xl w-32 p-2 text-center bg-blue-600 text-gray-200 border-blue-400 border-2 rounded-md focus:outline-none select-none placeholder-gray-200"
          placeholder="# Champs">

        <div class="flex items-center space-x-2">
          <input type="checkbox" v-model="roleBasedGeneration" class="w-4 h-4">
          <label class="text-gray-200 text-lg">Of Each Role</label>
        </div>

        <button @click="generateChampions"
          class="text-gray-100 font-mono text-2xl md:text-3xl bg-blue-500 p-2 rounded-md select-none cursor-pointer hover:bg-blue-600 transition-colors">
          Random Champs
        </button>
      </div>
    </div>

    <!-- Teams Section -->
    <div class="w-full max-w-4xl px-4 mt-8 md:mt-12">
      <div class="flex flex-col md:flex-row justify-center space-y-4 md:space-y-0 md:space-x-4">
        <!-- Blue Team -->
        <div
          class="w-full md:w-[310px] h-[300px] md:h-[500px] bg-blue-600/12 overflow-auto border-blue-500 border-2 rounded-sm">
          <div class="grid grid-cols-4 sm:grid-cols-5 gap-1 p-1">
            <img v-for="champion in blueTeam" :key="champion"
              :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion"
              class="w-full aspect-square object-cover">
          </div>
        </div>

        <!-- Red Team -->
        <div
          class="w-full md:w-[310px] h-[300px] md:h-[500px] bg-red-600/12 overflow-auto border-red-500 border-2 rounded-sm">
          <div class="grid grid-cols-4 sm:grid-cols-5 gap-1 p-1">
            <img v-for="champion in redTeam" :key="champion"
              :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion"
              class="w-full aspect-square object-cover">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import championsData from '/src/assets/champs.json'

export default {
  name: 'ChampionRandomizer',

  data() {
    return {
      championsList: [],
      blueTeam: [],
      redTeam: [],
      numberOfChamps: 5,
      roleBasedGeneration: false,
      topChamps: [],
      jgChamps: [],
      midChamps: [],
      adcChamps: [],
      suppChamps: [],
      availabletop: [],
      availablejg: [],
      availablemid: [],
      availablesupp: [],
      availableadc: [],
    }
  },

  methods: {
    generateRandomNumber(min, max) {
      return Math.floor(Math.random() * (max - min)) + min
    },
    sortChampsByRole(data) {
      console.log('champlist', data)
      Object.entries(data).forEach(([champId, champion]) => {
        // For each role this champion can play
        // console.log(champion)
        champion.roles?.forEach(role => {
          switch (role) {
            case 'top':
              this.topChamps.push(champId);
              break;
            case 'jg':
              this.jgChamps.push(champId);
              break;
            case 'mid':
              this.midChamps.push(champId);
              break;
            case 'adc':
              this.adcChamps.push(champId);
              break;
            case 'supp':
              this.suppChamps.push(champId);
              break;
          }
        });
      });
    },

    generateChampions() {
      if (this.roleBasedGeneration) {
        this.generateChampionsRole();
      } else {
        this.generateChampionsFullRandom();
      }
    },

    generateChampionsFullRandom() {
      const availableChampions = [...this.championsList]
      this.blueTeam = []
      this.redTeam = []

      // Generate blue team champions
      for (let i = 0; i < this.numberOfChamps; i++) {
        const chosenIndex = this.generateRandomNumber(0, availableChampions.length)
        this.blueTeam.push(availableChampions[chosenIndex])
        availableChampions.splice(chosenIndex, 1)
      }

      // Generate red team champions
      for (let i = 0; i < this.numberOfChamps; i++) {
        const chosenIndex = this.generateRandomNumber(0, availableChampions.length)
        this.redTeam.push(availableChampions[chosenIndex])
        availableChampions.splice(chosenIndex, 1)
      }
    },
    removeFromRoleList(championId) {
      const roleArrays = [
        this.availabletop,
        this.availablejg,
        this.availablemid,
        this.availableadc,
        this.availablesupp
      ];
      roleArrays.forEach(array => {
        const index = array.indexOf(championId);
        if (index > -1) {
          array.splice(index, 1);
        }
      });
    },

    genFromList(available, team) {
      if (available.length >= 1) {
        const chosenIndex = this.generateRandomNumber(0, available.length)
        team.push(available[chosenIndex])
        this.removeFromRoleList(available[chosenIndex])
        available.splice(chosenIndex, 1)
      }
      else {
        this.genFromList(this.getCombinedLists(), team);
      }
    },

    getCombinedLists() {
      return [...new Set([
        ...this.availabletop,
        ...this.availablejg,
        ...this.availablemid,
        ...this.availableadc,
        ...this.availablesupp
      ])]
    },

    removeBannedChamps() {

    },

    generateChampionsRole() {
      this.availabletop = [...this.topChamps]
      this.availablejg = [...this.jgChamps]
      this.availablemid = [...this.midChamps]
      this.availableadc = [...this.adcChamps]
      this.availablesupp = [...this.suppChamps]
      this.blueTeam = []
      this.redTeam = []
      this.removeBannedChamps();

      // Generate blue team champions
      for (let r = 0; r < this.numberOfChamps; r++) {
        this.genFromList(this.availabletop, this.blueTeam);
        this.genFromList(this.availablejg, this.blueTeam);
        this.genFromList(this.availablemid, this.blueTeam);
        this.genFromList(this.availableadc, this.blueTeam);
        this.genFromList(this.availablesupp, this.blueTeam);

        this.genFromList(this.availabletop, this.redTeam);
        this.genFromList(this.availablejg, this.redTeam);
        this.genFromList(this.availablemid, this.redTeam);
        this.genFromList(this.availableadc, this.redTeam);
        this.genFromList(this.availablesupp, this.redTeam);
      }
    }
  },

  mounted() {
    this.championsList = Object.keys(championsData);
    this.sortChampsByRole(championsData);
  }
}
</script>