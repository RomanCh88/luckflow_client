<template>
  <div class="home">
    <p @click="nextDate">next</p>
    <img alt="Vue logo" @click="random" src="../assets/logo.png" />
    <p>{{ stats.luck }}</p>
    <p>{{ stats.jackpots }}</p>
    <p>{{ stats.fiveNumbers }}</p>
    <p>{{ stats.fourNumbersBonus }}</p>
    <p>{{ stats.fourNumbers }}</p>
    <HelloWorld :msg="stats.jackpots" />
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
      birthday: "17051988",
      winners: [],
      stats: {},
      period: "",
      seed: "",
      numerology: "",
    };
  },

  methods: {
    nextDate() {
      this.birthday = moment(this.birthday, "DDMMYYYY")
        .add(1, "days")
        .format("DDMMYYYY");
      this.stats = {};
      this.winners = [];
      this.random();
    },
    random() {
      const { birthday } = this;
      const x = this.addDigitsRecursively(this.birthday);
      this.period = 18993;
      const birthdayTimestamp = moment(birthday, "DDMMYYYY").format("X");
      let today = moment().startOf("day").format("X");

      for (let i = 0; i < this.period; i++) {
        today = moment().add(i, "days").startOf("day").format("X");
        let seed = today - birthdayTimestamp;
        let lotteryBalls = [...Array(70).keys()];
        let handNumbers = [...Array(70).keys()];
        this.seed = seed;
        this.numerology = x;
        let randomLeft = seedrandom.alea(seed * x);
        let randomRight = seedrandom.alea(today);
        let cloneLeft = randomLeft();
        let cloneRight = randomRight();
        let randomBoth = this.moveInArray(
          (
            randomLeft().toString().substring(2).slice(0, 10) +
            randomRight().toString().substring(2).slice(0, 10)
          ).match(/.{1,2}/g)
        );

        let randomLotteryBonus = (
          ((cloneLeft.toString().substring(2).charAt(10) +
            cloneRight.toString().substring(2).charAt(10)) /
            99) *
          25
        ).toFixed(0);

        let randomHandBonus = (
          ((cloneLeft.toString().substring(2).charAt(11) +
            cloneRight.toString().substring(2).charAt(11)) /
            99) *
          25
        ).toFixed(0);

        let lottery = randomBoth.slice(0, 5);
        let hand = randomBoth.slice(5, 11);

        let bonus = randomLotteryBonus === randomHandBonus;

        lottery.forEach((number, index) => {
          let basketIndex = ((number / 99) * lotteryBalls.length).toFixed(0);
          lottery[index] = lotteryBalls[basketIndex];
          lotteryBalls.splice(basketIndex, 1);
        });

        hand.forEach((number, index) => {
          let basketIndex = ((number / 99) * handNumbers.length).toFixed(0);
          hand[index] = handNumbers[basketIndex];
          handNumbers.splice(basketIndex, 1);
        });

        let matches = this.getMatches(lottery, hand);
        if (matches.length > 1 || bonus) {
          let date = moment.unix(today).format();
          this.winners.push({
            date,
            lottery: [...lottery, parseInt(randomLotteryBonus)],
            hand: [...hand, parseInt(randomHandBonus)],
            matches,
            bonus,
          });
        }
      }

      this.analyzeWins();
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
    getMatches(lottery, hand) {
      return lottery.filter((ball) => hand.indexOf(ball) != -1);
    },
    analyzeWins() {
      const jackpots = this.winners.filter(
        (item) => item.bonus === true && item.matches.length === 5
      ).length;
      const fiveNumbers = this.winners.filter(
        (item) => item.bonus === false && item.matches.length === 5
      ).length;
      const fourNumbersBonus = this.winners.filter(
        (item) => item.bonus === true && item.matches.length === 4
      ).length;
      const fourNumbers = this.winners.filter(
        (item) => item.bonus === false && item.matches.length === 4
      ).length;
      const threeNumbersBonus = this.winners.filter(
        (item) => item.bonus === true && item.matches.length === 3
      ).length;
      const threeNumbers = this.winners.filter(
        (item) => item.bonus === false && item.matches.length === 3
      ).length;
      const twoNumbersBonus = this.winners.filter(
        (item) => item.bonus === true && item.matches.length === 2
      ).length;
      const numberBonus = this.winners.filter(
        (item) => item.bonus === true && item.matches.length === 1
      ).length;
      const bonus = this.winners.filter(
        (item) => item.bonus === true && item.matches.length < 1
      ).length;
      let luck = jackpots > 0 ? jackpots * (302575350 / this.period) : 1;
      luck *= fiveNumbers > 0 ? fiveNumbers * (12607306 / this.period) : 1;
      luck *=
        fourNumbersBonus > 0 ? fourNumbersBonus * (931000 / this.period) : 1;
      luck *= fourNumbers > 0 ? fourNumbers * (38792 / this.period) : 1;
      luck *=
        threeNumbersBonus > 0 ? threeNumbersBonus * (14547 / this.period) : 1;
      luck *= threeNumbers > 0 ? threeNumbers * (606 / this.period) : 1;
      luck *= twoNumbersBonus > 0 ? twoNumbersBonus * (693 / this.period) : 0;
      luck *= numberBonus > 0 ? numberBonus * (89 / this.period) : 0;
      luck *= bonus > 0 ? bonus * (37 / this.period) : 1;

      this.stats = {
        jackpots,
        fiveNumbers,
        fourNumbersBonus,
        fourNumbers,
        threeNumbersBonus,
        threeNumbers,
        twoNumbersBonus,
        numberBonus,
        bonus,
        luck: (luck * 100).toFixed(0) + "%",
      };
    },
  },
};
</script>
