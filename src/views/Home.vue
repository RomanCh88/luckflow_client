<template>
  <div class="home">
    <img alt="Vue logo" @click="random" src="../assets/logo.png" />
    <HelloWorld :msg="msg" />
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import seedrandom from "seedrandom";
import moment from "moment";

export default {
  name: "Home",
  components: {
    HelloWorld,
  },
  data() {
    return {
      msg: "",
      birthday: "18051988",

      winners: [],
    };
  },

  methods: {
    random() {
      const { birthday } = this;
      const x = this.addDigitsRecursively(this.birthday);
      const birthdayTimestamp = moment(birthday, "DDMMYYYY").format("X");
      let today = moment().startOf("day").format("X");

      for (let i = 0; i < 30; i++) {
        today = moment().add(i, "days").startOf("day").format("X");
        let seed = today - birthdayTimestamp;
        let randomLeft = seedrandom.xor4096(seed);
        let randomRight = seedrandom.xor4096(seed * x);
        let randomBoth = this.moveInArray(
          (
            randomLeft().toString().substring(2).slice(0, 12) +
            randomRight().toString().substring(2).slice(0, 12)
          )
            .replaceAll(".", "")
            .match(/.{1,2}/g)
        );
        let matches = this.getMatches([...randomBoth]);
        if (matches.length > 0) {
          let date = moment.unix(today).format();
          let lottery = randomBoth.slice(0, 6);
          let hand = randomBoth.slice(6, 12);
          let luck = (matches.length / 6).toFixed(2) * 100 + "%";
          this.winners.push({
            date,
            lottery,
            hand,
            matches,
            luck,
          });
        }
      }
    },
    addDigitsRecursively(number) {
      let final;
      while ((final = this.addDigits(number)) > 9) number = final.toString();
      return final;
    },
    addDigits(str) {
      return str
        .split("")
        .map(function (n) {
          return parseInt(n);
        })
        .reduce(function (a, b) {
          return a + b;
        });
    },
    moveInArray(arr) {
      var items = arr.splice(3, 3);

      return arr.concat(items);
    },
    getMatches(arr) {
      const hand = arr.splice(5, 6);
      return arr.filter((ball) => hand.indexOf(ball) != -1);
    },
  },
};
</script>
