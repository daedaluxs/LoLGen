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
    <div class="w-full max-w-4xl px-4 mt-[16px] md:mt-[30px]">
      <div class="flex flex-col md:flex-row justify-center space-y-4 md:space-y-0 md:space-x-4">
        <!-- Blue Team -->
        <div
          class="w-full scrollbar-hide md:w-[420px] h-[350px] md:h-[590px] bg-blue-600/12 overflow-auto border-blue-500 border-2 rounded-sm">
          <div class="grid grid-cols-5 gap-1 p-1">
            <img v-for="champion in blueTeam" :key="champion"
              :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion"
              class="rounded-md w-full aspect-square object-cover">
          </div>
        </div>

        <!-- Red Team -->
        <div
          class="w-full scrollbar-hide md:w-[420px] h-[350px] md:h-[590px] bg-red-600/12 overflow-auto border-red-500 border-2 rounded-sm">
          <div class="grid grid-cols-5 gap-1 p-1">
            <img v-for="champion in redTeam" :key="champion"
              :src="`https://cdn.communitydragon.org/15.4.0/champion/${champion}/square`" :alt="champion"
              class="rounded-md w-full aspect-square object-cover">
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
      champData: {},
      championsList: [],
      blueTeam: [],
      redTeam: [],
      numberOfChamps: 2,
      roleBasedGeneration: true,
      topChamps: [],
      jgChamps: [],
      midChamps: [],
      adcChamps: [],
      suppChamps: [],
      allavail: {
        '1': [],
        '2': [],
        '3': [],
        '4': [],
        '5': [],
      },

    }
  },

  methods: {
    generateRandomNumber(min, max) {
      return Math.round(((Math.random() * (max - min)) + min) * 10) / 10;
    },
    sortChampsByRole(data) {
      // console.log('champlist', data)
      Object.entries(data).forEach(([champId, champion]) => {
        // For each role this champion can play
        // console.log(champion)
        Object.keys(champion.roles)?.forEach(role => {
          let item = {
            "id": champId,
            "weight": champion.roles[role]
          }
          switch (role) {
            case 'top':
              this.topChamps.push(item);
              break;
            case 'jg':
              this.jgChamps.push(item);
              break;
            case 'mid':
              this.midChamps.push(item);
              break;
            case 'adc':
              this.adcChamps.push(item);
              break;
            case 'supp':
              this.suppChamps.push(item);
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
        const chosenIndex = Math.round(this.generateRandomNumber(0, availableChampions.length));
        this.blueTeam.push(availableChampions[chosenIndex])
        availableChampions.splice(chosenIndex, 1)
      }

      // Generate red team champions
      for (let i = 0; i < this.numberOfChamps; i++) {
        const chosenIndex = Math.round(this.generateRandomNumber(0, availableChampions.length));
        this.redTeam.push(availableChampions[chosenIndex])
        availableChampions.splice(chosenIndex, 1)
      }
    },
    removeFromRoleList(championName) {
      const roleArrays = [
        this.allavail[1],
        this.allavail[2],
        this.allavail[3],
        this.allavail[4],
        this.allavail[5]
      ];
      roleArrays.forEach(array => {
        delete array[championName]
      });
    },

    getWeightById(targetId, role) {
      const found = this.champData[targetId];
      // const found = Object.values(this.championsList).find(item => item.id === targetId);
      return found.roles[role];
    },

    genFromList(rolenum, team) {
      if (Object.keys(this.allavail[rolenum]).length >= 1) {
        const chosenIndex = this.generateRandomNumber(0, this.getMaxValue(this.allavail[rolenum]))
        const champname = this.findChampionByValue(chosenIndex, this.allavail[rolenum])
        team.push(champname)
        this.removeFromRoleList(champname)
        //regenerate all roles.(because champs can have multiple roles)
        for (let role = 1; role <= 5; role++) {
          this.allavail[role] = this.generateRangesruntime(this.allavail[role], role);
        }
      }
      else {
        //maybe just make this nothing..
        let templist = this.getCombinedLists();
        let tempranges = this.generateRangesruntime();
        this.genFromList(tempranges, team);
      }
    },

    getMaxValue(ranges) {
      return Math.max(...Object.values(ranges).map(range => range.max));
    },

    getCombinedLists() {
      const allChamps = {
        ...this.allavail[1],
        ...this.allavail[2],
        ...this.allavail[3],
        ...this.allavail[4],
        ...this.allavail[5]
      };

      // Create an object to store unique ranges
      const uniqueRanges = {};

      // Process each champion's range, keeping only one entry per champion
      Object.entries(allChamps).forEach(([champion, range]) => {
        uniqueRanges[champion] = range;
      });

      return uniqueRanges;
    },

    removeBannedChamps() {

    },

    generateRangesruntime(data, rolenum) {
      const lookup = {
        1: 'top',
        2: 'jg',
        3: 'mid',
        4: 'adc',
        5: 'supp'
      }
      const ranges = {};
      let currentStart = 0;
      const round = (num) => Math.round(num * 10) / 10;

      // Get entries from the object
      const entries = Object.entries(data);

      for (let i = 0; i < entries.length; i++) {

        const [key, champion] = entries[i];  // each entry is [key, value]
        const weight = this.getWeightById(key, lookup[rolenum]);
        // const weight = round(champion.weight); //THIS NEEDS TO GET THE CHAMP WEIGHT BY IDDDDDD
        ranges[key] = {
          min: round(currentStart),
          max: round(currentStart + weight)
        };

        currentStart = round(ranges[key].max + 0.1);

      }
      return ranges;
    },

    generateRanges(data) {
      const ranges = {};
      let currentStart = 0;

      // Helper function to round to nearest 0.1
      const round = (num) => Math.round(num * 10) / 10;

      data.sort((a, b) => a.weight - b.weight);

      for (let i = 0; i < data.length; i++) {
        const key = data[i].id;
        const weight = round(data[i].weight);

        ranges[key] = {
          min: round(currentStart),
          max: round(currentStart + weight)
        };

        currentStart = round(ranges[key].max + 0.1);
      }

      return ranges;
    },

    findChampionByValue(value, ranges) {
      return Object.entries(ranges).find(([champion, range]) =>
        value >= range.min && value <= range.max
      )?.[0] || null;
    },

    generateChampionsRole() {
      this.allavail[1] = this.generateRanges(this.topChamps);
      this.allavail[2] = this.generateRanges(this.jgChamps);
      this.allavail[3] = this.generateRanges(this.midChamps);
      this.allavail[4] = this.generateRanges(this.adcChamps);
      this.allavail[5] = this.generateRanges(this.suppChamps);
      this.blueTeam = []
      this.redTeam = []
      this.removeBannedChamps();

      // Generate blue team champions
      for (let r = 0; r < this.numberOfChamps; r++) {
        this.genFromList(1, this.blueTeam);
        this.genFromList(2, this.blueTeam);
        this.genFromList(3, this.blueTeam);
        this.genFromList(4, this.blueTeam);
        this.genFromList(5, this.blueTeam);

        this.genFromList(1, this.redTeam);
        this.genFromList(2, this.redTeam);
        this.genFromList(3, this.redTeam);
        this.genFromList(4, this.redTeam);
        this.genFromList(5, this.redTeam);
      }
    }
  },

  mounted() {
    this.champData = championsData;
    this.championsList = Object.keys(championsData);
    this.sortChampsByRole(championsData);

  }
}
</script>