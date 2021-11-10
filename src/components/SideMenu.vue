<template>
  <div class="menu">
    <div class="toolbar">
      <div class="toolbar__header">
        <template v-if="!isUserOpenned">
          <h3>Информация</h3>
        </template>
        <template v-else>
          <div class="action">
            <div class="arrow" @click="closeProfile"></div>
          </div>
          <h3>Профиль</h3>
        </template>
      </div>
      <div class="toolbar__actions"></div>
    </div>
    <div class="content">
        <!-- v-show т.к. отрисовываем список подразделений и диаграмму 1 раз и нам не нужно их ререндерить -->
      <div v-show="!isUserOpenned" class="legend">
        <div class="legend__data">
          <div v-if="legend.length > 0" class="legend__items">
            <Draggable v-model="legend">
              <legend-item
                v-for="(item, index) in legend"
                :key="index"
                :color="item.color"
                :text="item.text"
                :counter="groupCounter[index]"
                class="legend__item"
              />
            </Draggable>
          </div>
          <span v-else class="legend--empty"> Список пуст </span>
        </div>
        <div class="legend__chart">
          <PieChart ref="chart" />
        </div>
      </div>
      <div v-if="isUserOpenned" class="profile">
        <div v-if="!person" class="profile__empty">Место пустое</div>

        <PersonCard v-else-if="isUserOpenned" :person="person" />
      </div>
    </div>
  </div>
</template>

<script>
import LegendItem from "./SideMenu/LegendItem.vue";
import PersonCard from "./SideMenu/PersonCard.vue";
import legend from "@/assets/data/legend.json";
import Draggable from "vuedraggable";
import { Doughnut as PieChart } from "vue-chartjs";
import tables from "@/assets/data/tables.json";

export default {
    props: {
        isUserOpenned: {
            type: Boolean,
            default: false,
        },
        person: {
            type: Object,
            default: null,
        },
    },
    components: {
        LegendItem,
        PersonCard,
        Draggable,
        PieChart,
    },
    data() {
        return {
            legend: [],
            tables: [],
            groupCounter: [],
        };
    },
    created() {
        this.loadLegend();
    },
    mounted() {
        this.countGroupEls();
        this.makeChart();
    },
    methods: {
         makeChart() {
            const legendChartData = {
                labels: this.legend.map((it) => it.text),
                datasets: [
                    {
                        label: "Легенда",
                        backgroundColor: this.legend.map(
                            (legendItem) => legendItem.color
                        ),
                        data: this.groupCounter
                    },
                ],
            };

            const options = {
                borderWidth: "10px",
                legend: {
                    display: false,
                },
            };

            this.$refs.chart.renderChart(legendChartData, options);
        },
        loadLegend() {
            this.legend = legend;
            this.tables = tables;
        },
        closeProfile() {
            this.$emit("update:isUserOpenned", false);
        },
        countGroupEls() {
         const countEls = new Map();
         this.tables.map((tableItem) => (tableItem.group_id))
            .sort((a,b) => (a - b)).forEach((el) => {
             if (countEls.has(el)) {
                 countEls.set(el, countEls.get(el) + 1)
             } else {
                 countEls.set(el, 1)
             }
         });
         this.groupCounter = Array.from(countEls).map((arr) => arr[1]);
        }
    }
};
</script>

<style scoped>
.menu {
  border-left: 1px solid #ccd8e4;
  padding: 24px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toolbar .toolbar__actions button {
  font-size: 0.76rem;
  text-transform: uppercase;
  letter-spacing: 0.08rem;
  padding: 2px 6px;
}

.toolbar__header {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
}

.toolbar__header .action {
  cursor: pointer;
  margin-right: 14px;
  width: 20px;
  height: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.toolbar__header .action .arrow {
  width: 10px;
  height: 10px;
  border-top: 2px solid blue;
  border-right: 2px solid blue;
  transform: rotate(-135deg);
}

h3 {
  margin: 0;
}

.content {
  flex: 1;
  overflow-y: auto;
}

.content .legend {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}

.content .legend .legend__data {
  display: flex;
  height: 100%;
}

.content .legend .legend__items {
  flex: 1;
  width: 100%;
}

.content .legend .legend__items .legend__item:not(:first-child) {
  margin-top: 16px;
}

.content .legend .legend__items .legend__item {
  cursor: pointer;
}

.content .legend .legend__items .legend__item.sortable-chosen {
  opacity: 25%;
}

.content .legend .legend--empty {
  align-self: center;
  width: 100%;
  text-align: center;
}

.profile {
  padding-top: 20px;
}
</style>
