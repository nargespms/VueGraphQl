<template>
  <div id="app">
    <div>Data: {{ example1 }}</div>
    <button @click="getLanguage">Get Language</button>
    <hr />

    <button @click="getChampions">Get Champion</button>
    <div>
      chapions:
      <p v-for="champ in champions" :key="champ.index">
        {{ champ.name }}: {{ champ.attackDamage }}
      </p>
    </div>
    <hr />

    Name: <input v-model="name" />
    <div v-if="champion.hasOwnProperty('name')">Data: {{ champion }}</div>
    <div v-else>No Data Found.</div>
    <button @click="getChampionByName">Get Champion</button>

    <hr />

    Name: <input v-model="updatedChampName" /> Attack Damage:
    <input v-model.number="attack" />
    <div>
      Data:
      {{ updatedChampion }}
    </div>
    <button @click="updateAttackDamage">Update Champion</button>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: "app",
  data() {
    return {
      example1: "",
      champions: [],
      name: "",
      champion: {},
      updatedChampion: {},
      updatedChampName: "",
      attack: 10,
    };
  },
  methods: {
    async getLanguage() {
      try {
        const res = await axios.post("http://localhost:4000/graphql", {
          query: "{ language }",
        });
        this.example1 = res.data.data.language;
      } catch (e) {
        console.log("err", e);
      }
    },
    getChampions() {
      axios
        .post("http://localhost:4000/graphql", {
          query: `
        {
          getChampions {
            attackDamage,
            name
          }
        }
        `,
        })
        .then((res) => {
          this.champions = res.data.data.getChampions;
        });
    },

    getChampionByName() {
      axios
        .post("http://localhost:4000/graphql", {
          query: `
          query GetChampionByName($championName: String!) {
            getChampionByName(name: $championName) {
              name
              attackDamage
            }
          }`,
          variables: {
            championName: this.name,
          },
        })
        .then((res) => {
          if (res.data.data.getChampionByName !== null) {
            this.champion = res.data.data.getChampionByName;
          } else {
            this.champion = {};
          }
        });

      // dont know why it doesn't work this way

      // axios
      //   .post("http://localhost:4000/graphql", {
      //     query: `
      //     query GetChampionByName(${this.name}) {
      //       getChampionByName(${this.name}) {
      //         name
      //         attackDamage
      //       }
      //     }`,
      //   })
      //   .then((res) => {
      //     this.champion = res.data.data.getChampionByName;
      //   });
    },
    updateAttackDamage() {
      axios
        .post("http://localhost:4000/graphql", {
          query: `
          mutation UpdateAttackDamage($championName: String!,$attackDamage:Float) {
            updateAttackDamage(name: $championName,attackDamage:$attackDamage) {
              name
              attackDamage
            }
          }`,
          variables: {
            championName: this.updatedChampName,
            attackDamage: this.attack,
          },
        })
        .then((res) => {
          if (res.data.data.getChampionByName !== null) {
            this.updatedChampion = res.data.data.updateAttackDamage;
          } else {
            this.updatedChampion = {};
          }
        });
    },
  },
};
</script>
