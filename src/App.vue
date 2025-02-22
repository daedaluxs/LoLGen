<template>
  <div class="min-h-screen min-w-screen">
    <div></div>
    <div class="absolute mt-[20vh] ml-[44vw]">
      <input type="number" v-model="numberOfChamps" min="1" max="50"
        class="text-semibold mb-[12px] ml-[55px] text-xl focus:outline-none select-none bg-blue-600 text-gray-200 border-blue-400 border-2 rounded-md w-[100px] placeholder-semibold placeholder-gray-200"
        placeholder="# Champs">
      <div class="flex items-center pl-[40px] mb-[10px]">
        <input type="checkbox" v-model="roleBasedGeneration" class="mr-2">
        <label class="text-gray-200 text-lg">Of Each Role</label>
      </div>
      <div @click="generateChampions"
        class="text-gray-100 font-mono text-3xl bg-blue-500 p-2 rounded-md select-none cursor-pointer">
        Random Champs
      </div>
    </div>
    <div class="absolute mt-[36vh] ml-[33vw]">
      <div class="flex pt-[20px]">
        <div
          class="w-[310px] h-[500px] bg-blue-600/12 overflow-auto grid grid-cols-5 border-blue-500 border-2 rounded-sm mr-[10px]">
          <img v-for="champion in blueTeam" :key="champion"
            :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion" width="60"
            height="60">
        </div>
        <div
          class="w-[310px] bg-red-600/12 overflow-auto grid grid-cols-5 h-[500px] border-red-500 border-2 rounded-sm ml-[10px]">
          <img v-for="champion in redTeam" :key="champion"
            :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion" width="60"
            height="60">
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

    removeBannedChamps(){

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