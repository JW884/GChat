<template>
  <div>
    <h1>Cookie Clicker</h1>
    <div>
      <p>Cookies: {{ cookies.toFixed(1) }}</p>
      <button @click="clickCookie">Click Me!</button>
    </div>
    <div>
      <h2>Buildings</h2>
      <p>
        Total Production: {{ totalProduction.toFixed(1) }} cookies per second
      </p>
      <div v-for="building in buildings" :key="building.name">
        <p>
          {{ building.name }} ({{ building.count }}) - Cost:
          {{ getBuildingCost(building) }} - Production:
          {{ building.productionRate }}/s
        </p>
        <button
          @click="buyBuilding(building)"
          :disabled="cookies < getBuildingCost(building)"
        >
          Buy
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cookies: 0,
      cookiesPerClick: 1,
      buildings: [
        {
          name: "Cursor",
          baseCost: 15,
          costMultiplier: 1.15,
          productionRate: 0.1,
          count: 0,
        },
        {
          name: "Grandma",
          baseCost: 100,
          costMultiplier: 1.15,
          productionRate: 1,
          count: 0,
        },
        {
          name: "Farm",
          baseCost: 1100,
          costMultiplier: 1.15,
          productionRate: 8,
          count: 0,
        },
      ],
    };
  },
  computed: {
    totalProduction() {
      return this.buildings.reduce(
        (total, building) => total + building.productionRate * building.count,
        0
      );
    },
  },
  methods: {
    clickCookie() {
      this.cookies += this.cookiesPerClick;
    },
    getBuildingCost(building) {
      return Math.floor(
        building.baseCost * Math.pow(building.costMultiplier, building.count)
      );
    },
    buyBuilding(building) {
      const cost = this.getBuildingCost(building);
      if (this.cookies >= cost) {
        this.cookies -= cost;
        building.count += 1;
      }
    },
  },
  mounted() {
    setInterval(() => {
      this.cookies += this.totalProduction / 10;
    }, 100);
  },
};
</script>
