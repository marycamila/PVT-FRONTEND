<template>
  <v-row >
    <v-col cols="6"  class="text-left ma-0 pb-0">
      <v-toolbar-title>APORTES ACTIVO
        <v-tooltip top>
          <template v-slot:activator="{ on }">
            <v-btn top
              small
              dark
              icon
              color="success"
              class="my-2 mr-4"
              v-on="on"
              @click="showDetailActive()"
            >
              <v-icon> mdi-arrow-right-bold </v-icon>
            </v-btn>
          </template>
          <div>
            <span>Detalle de Aportes</span>
          </div>
        </v-tooltip>
      </v-toolbar-title>
    </v-col>
    <v-col cols="6" class="text-right ma-0 pb-0"  v-if="show">
      <v-tooltip top class="my-0">
        <template v-slot:activator="{ on }">
          <v-btn
            x-small
            color="info"
            :loading="loading_print_active"
            v-on="on"
            class="ma-2 pa-3"
            outlined
            elevation="2"
            @click="printContributionsActive()"
            >
            <v-icon> mdi-download</v-icon>Certificación
          </v-btn>
        </template>
        <div>
          <span>Certificación de Aportes</span>
        </div>
      </v-tooltip>
    </v-col>
    <v-col cols="12"  v-if="show">
      <v-data-table
        :headers="headers"
        :items="contributions"
        :loading="loading"
        single-expand
        item-key="year"
        :itemsPerPage="contributions.length"
        hide-default-footer
      >
        <template v-slot:item="props">
          <tr :class="props.isExpanded ? 'secondary white--text': ''">
            <td class="text-left" @click.stop="expand(props)">{{ props.item.year }}</td>
            <td class="text-right" @click.stop="expand(props)" v-for="(key,index) in props.item.contributions" :key="index" >{{Object.entries(key.detail).length !==0 ? key.detail.total:'0,00' }}</td>
          </tr>
        </template>
        <template v-slot:expanded-item="{item}">
          <tr>
            <td :colspan="13" class="px-0">
              <v-data-table 
                :items="item.contributions"
                :hide-default-footer="true"
                :itemsPerPage="12"
                class="tertiary">
                <template v-slot:body="{ items }">
                  <tbody>
                    <tr v-for="header in headersDetail" :key="header.id">
                      <td class="font-weight-bold">
                        {{ header.text }}
                      </td>
                      <td class="text-right" v-for="item in items" :key="item.id">
                        {{ Object.entries(item.detail).length !==0 ? item.detail[header.value]: '0,00'}}
                      </td>
                    </tr>
                  </tbody>
                </template>
              </v-data-table>
            </td>
          </tr>
        </template>
      </v-data-table>
    </v-col>
    <v-col cols="12"  v-else>
      NO SE ENCONTRARON REGISTROS.
    </v-col>
  </v-row>
</template>
<script>
export default {
  name: "ListContributionActive",
  props: {
    affiliate: {
      type: Object,
      require: true,
    },
    show_detail: {
      type: Object,
      require: true,
    },
    state: {
      type: Object,
      require: true,
    },
  },
  components: {},
  data: () => ({
    itemsPerPage: 0,
    loading: true,
    search: "",
    active: true,
    show: false,
    loading_print_active: false,
    contributions: [],
    headers: [
      { text: "Año",value: "year", class: ["table", "white--text", "text-left","text-uppercase"], width: "10%", sortable: true,},
      { text: "Enero",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Febrero",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Marzo",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Abril",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Mayo",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Junio",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Julio",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Agosto",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%",sortable: false,},
      { text: "Septiembre",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%",sortable: false,},
      { text: "Octubre",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%",sortable: false,},
      { text: "Noviembre",value: "", class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
      { text: "Diciembre",value: "",class: ["table", "white--text", "text-right","text-uppercase"], width: "7%", sortable: false,},
    ],
    headersDetail: [
      { text: "Cotizable", value: "quotable", class: ["table", "white--text", "text-left"], width: "10%", sortable: true,},
      { text: "Fondo de Retiro", value: "retirement_fund", class: ["table", "white--text", "text-left"], width: "10%", sortable: true,},
      { text: "Cuota Mortuoria ", value: "mortuary_quota", class: ["table", "white--text", "text-left"], width: "10%", sortable: true,},
      { text: "Aporte", value: "total", class: ["table", "white--text", "text-left"], width: "16%", sortable: true,},
      { text: "Reintegro", value: "reimbursement_total", class: ["table", "white--text", "text-left"], width: "10%", sortable: true,},
    ],
  }),
  mounted() {
    this.getActiveAffiliateContribution(this.$route.params.id);
  },
  methods: {
    expand(props) {
      props.expand(!props.isExpanded && this.active);
    },
    async getActiveAffiliateContribution(id) {
      try {
        this.loading = true;
        let res = await this.$axios.post(
          `/contribution/active_affiliate_contribution`,
          {
            affiliate_id: id,
          }
        );
        this.show = res.hasContributions;
        this.contributions = res.payload.all_contributions;
        console.log(this.contributions);
      } catch (e) {
        console.log(e);
      } finally {
        this.loading = false;
      }
    },
    async printContributionsActive() {
      this.loading_print_active = true;
      try {
        let res = await this.$axios.get(
          `/contribution/print_contributions_active/${this.$route.params.id}`,
          undefined,
          { responseType: "blob" }
        );
        const url = window.URL.createObjectURL(new Blob([res]));
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", "Aporte_Activo.pdf");
        document.body.appendChild(link);
        link.click();
      } catch (e) {
        console.log(e);
        this.loading_print_active = false;
      } finally {
        this.loading_print_active = false;
      }
    },
    showDetailActive() {
      this.show_detail.passive = false;
      this.state.active = true;
      this.show_detail.active = true;
    },
  },
};
</script>
<style scoped>
.filter-text {
  font-size: 12px;
  height: 2px;
  margin: 0 0 40px 0;
  padding: 0;
  width: 100%;
}
</style>
