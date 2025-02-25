<template>
  <div class="game-container">
    <h1 class="game-title">Nuxt Cookie Clicker</h1>
    <div class="cookie-section">
      <img
        src="~/assets/cookie.svg"
        alt="Cookie"
        @click="clickCookie"
        class="cookie-image"
      />
      <p class="cookie-count">Cookies: {{ cookies.toFixed(1) }}</p>
    </div>
    <div class="buildings-section">
      <h2>Buildings</h2>
      <p>
        Total Production: {{ totalProduction.toFixed(1) }} cookies per second
      </p>
      <div
        v-for="building in buildings"
        :key="building.name"
        class="building-card"
      >
        <h3>{{ building.name }} ({{ building.count }})</h3>
        <p>Cost: {{ getBuildingCost(building) }}</p>
        <p>Production: {{ getBuildingProduction(building) }}/s</p>
        <button
          @click="buyBuilding(building)"
          :disabled="cookies < getBuildingCost(building)"
        >
          Buy
        </button>
      </div>
    </div>
    <div class="research-section">
      <h2>Research</h2>
      <div v-if="currentResearch" class="research-progress">
        <div
          class="progress-bar"
          :style="{ width: researchProgress + '%' }"
        ></div>
        <p>
          Researching: {{ currentResearch.name }} ({{ researchTimer }}s
          remaining)
        </p>
      </div>
      <div
        v-for="tech in availableResearches"
        :key="tech.name"
        class="research-item"
      >
        <h3>{{ tech.name }}</h3>
        <p>Cost: {{ tech.cost }} cookies, Time: {{ tech.time }}s</p>
        <p>{{ tech.description }}</p>
        <button
          @click="startResearch(tech)"
          :disabled="cookies < tech.cost || currentResearch !== null"
        >
          Start Research
        </button>
      </div>
      <div class="completed-research">
        <h3>Completed Researches</h3>
        <p v-for="tech in completedResearches" :key="tech.name">
          {{ tech.name }}
        </p>
      </div>
    </div>
    <div class="prestige-section">
      <h2>Prestige</h2>
      <p>Prestige Points: {{ prestigePoints }}</p>
      <p>PP to gain on prestige: {{ calculatePrestigePoints() }}</p>
      <button @click="prestige">Prestige</button>
      <div class="prestige-tree">
        <h3>Prestige Tree</h3>
        <div
          v-for="upgrade in prestigeTree"
          :key="upgrade.id"
          class="prestige-upgrade"
        >
          <h3>{{ upgrade.name }}</h3>
          <p>Cost: {{ upgrade.cost }} PP</p>
          <p>{{ upgrade.description }}</p>
          <button
            @click="buyPrestigeUpgrade(upgrade)"
            :disabled="!canBuyPrestigeUpgrade(upgrade)"
          >
            Buy
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "~/assets/css/main.css";

export default {
  data() {
    return {
      cookies: 0,
      totalCookiesEarned: 0,
      cookiesPerClick: 1,
      prestigePoints: 0,
      buildingProductionMultiplier: 1, // Explicitly initialized
      buildingCostMultiplier: 1, // Explicitly initialized
      buildings: [
        {
          name: "Cursor",
          baseCost: 15,
          costMultiplier: 1.15,
          baseProductionRate: 0.1,
          count: 0,
        },
        {
          name: "Grandma",
          baseCost: 100,
          costMultiplier: 1.15,
          baseProductionRate: 1,
          count: 0,
        },
        {
          name: "Farm",
          baseCost: 1100,
          costMultiplier: 1.15,
          baseProductionRate: 8,
          count: 0,
        },
        {
          name: "Mine",
          baseCost: 12000,
          costMultiplier: 1.15,
          baseProductionRate: 47,
          count: 0,
        },
        {
          name: "Factory",
          baseCost: 130000,
          costMultiplier: 1.15,
          baseProductionRate: 260,
          count: 0,
        },
        {
          name: "Bank",
          baseCost: 1.4e6,
          costMultiplier: 1.15,
          baseProductionRate: 1400,
          count: 0,
        },
        {
          name: "Temple",
          baseCost: 2e7,
          costMultiplier: 1.15,
          baseProductionRate: 7800,
          count: 0,
        },
      ],
      prestigeTree: [
        {
          id: "prod1",
          name: "Production Boost I",
          description: "Increases building production by 10%",
          cost: 1,
          effect: "applyProductionBoost",
          prerequisites: [],
          purchased: false,
        },
        {
          id: "prod2",
          name: "Production Boost II",
          description: "Increases building production by another 10%",
          cost: 2,
          effect: "applyProductionBoost",
          prerequisites: ["prod1"],
          purchased: false,
        },
        {
          id: "cost1",
          name: "Cost Reduction I",
          description: "Reduces building costs by 5%",
          cost: 1,
          effect: "applyCostReduction",
          prerequisites: [],
          purchased: false,
        },
      ],
      researches: [
        {
          name: "Faster Clicks",
          description: "Increases cookies per click by 1",
          cost: 1000,
          time: 60,
          effect: "applyFasterClicks",
          researched: false,
        },
        {
          name: "Efficient Baking",
          description: "Increases Grandma production by 20%",
          cost: 5000,
          time: 120,
          effect: "applyEfficientBaking",
          researched: false,
        },
      ],
      currentResearch: null,
      researchTimer: 0,
      researchInterval: null,
    };
  },
  computed: {
    totalProduction() {
      return (
        this.buildings.reduce(
          (total, building) =>
            total + this.getBuildingProduction(building) * building.count,
          0
        ) || 0
      ); // Fallback to 0 if undefined
    },
    availableResearches() {
      return this.researches.filter((tech) => !tech.researched);
    },
    completedResearches() {
      return this.researches.filter((tech) => tech.researched);
    },
    researchProgress() {
      if (!this.currentResearch || this.researchTimer === 0) return 0;
      return (
        ((this.currentResearch.time - this.researchTimer) /
          this.currentResearch.time) *
        100
      );
    },
  },
  methods: {
    clickCookie() {
      const earned = this.cookiesPerClick;
      this.cookies += earned;
      this.totalCookiesEarned += earned;

      const cookieImage = document.querySelector(".cookie-image");
      if (cookieImage) {
        const computedStyle = window.getComputedStyle(cookieImage);
        const transform = computedStyle.getPropertyValue("transform");
        let angle = 0;
        if (transform && transform !== "none") {
          const matrix = transform.match(/matrix\((.+)\)/);
          if (matrix) {
            const values = matrix[1].split(", ");
            const cos = parseFloat(values[0]);
            const sin = parseFloat(values[1]);
            angle = Math.round(Math.atan2(sin, cos) * (180 / Math.PI));
          }
        }

        cookieImage.classList.remove("bounce");
        void cookieImage.offsetWidth; // Force reflow
        cookieImage.style.setProperty("--spin-angle", `${angle}deg`);
        cookieImage.classList.add("bounce");

        setTimeout(() => {
          cookieImage.classList.remove("bounce");
        }, 400);
      }
    },
    getBuildingCost(building) {
      return Math.floor(
        building.baseCost *
          Math.pow(building.costMultiplier, building.count) *
          this.buildingCostMultiplier
      );
    },
    getBuildingProduction(building) {
      return building.baseProductionRate * this.buildingProductionMultiplier;
    },
    buyBuilding(building) {
      const cost = this.getBuildingCost(building);
      if (this.cookies >= cost) {
        this.cookies -= cost;
        building.count += 1;
      }
    },
    calculatePrestigePoints() {
      return Math.floor(Math.pow(this.totalCookiesEarned / 1e12, 0.5));
    },
    prestige() {
      const ppGained = this.calculatePrestigePoints();
      if (ppGained > 0) {
        this.prestigePoints += ppGained;
        this.cookies = 0;
        this.totalCookiesEarned = 0;
        this.buildings.forEach((building) => (building.count = 0));
      }
    },
    canBuyPrestigeUpgrade(upgrade) {
      const hasPrerequisites = upgrade.prerequisites.every((prereqId) => {
        const prereq = this.prestigeTree.find((u) => u.id === prereqId);
        return prereq && prereq.purchased;
      });
      return (
        this.prestigePoints >= upgrade.cost &&
        !upgrade.purchased &&
        hasPrerequisites
      );
    },
    buyPrestigeUpgrade(upgrade) {
      if (this.canBuyPrestigeUpgrade(upgrade)) {
        this.prestigePoints -= upgrade.cost;
        this[upgrade.effect]();
        upgrade.purchased = true;
      }
    },
    applyProductionBoost() {
      this.buildingProductionMultiplier *= 1.1;
    },
    applyCostReduction() {
      this.buildingCostMultiplier *= 0.95;
    },
    startResearch(tech) {
      if (
        this.cookies >= tech.cost &&
        !tech.researched &&
        this.currentResearch === null
      ) {
        this.cookies -= tech.cost;
        this.currentResearch = tech;
        this.researchTimer = tech.time;
        this.researchInterval = setInterval(() => {
          if (this.researchTimer > 0) {
            this.researchTimer -= 1;
          } else {
            this.completeResearch();
          }
        }, 1000);
      }
    },
    completeResearch() {
      if (this.currentResearch) {
        this[this.currentResearch.effect]();
        this.currentResearch.researched = true;
        this.currentResearch = null;
        clearInterval(this.researchInterval);
      }
    },
    applyFasterClicks() {
      this.cookiesPerClick += 1;
    },
    applyEfficientBaking() {
      const grandma = this.buildings.find((b) => b.name === "Grandma");
      if (grandma) {
        grandma.baseProductionRate *= 1.2;
      }
    },
  },
  mounted() {
    setInterval(() => {
      this.cookies += this.totalProduction / 10;
      this.totalCookiesEarned += this.totalProduction / 10;
    }, 100);
  },
};
</script>
