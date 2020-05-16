<template>
  <v-col cols="12" md="6" class="DataCard">
    <confirmed-cases-by-municipalities-table
      :title="$t('陽性患者数（市町村別）')"
      :title-id="'number-of-confirmed-cases-by-municipalities'"
      :chart-data="municipalitiesTable"
      :date="date"
      :info="info"
      :url="'https://opendata.pref.saitama.lg.jp/data/dataset/covid19-jokyo'"
      :url-label="$t('オープンデータへのリンク')"
    />
  </v-col>
</template>

<script>
import dayjs from 'dayjs'
import Data from '@/data/data.json'
import ConfirmedCasesByMunicipalitiesTable from '@/components/ConfirmedCasesByMunicipalitiesTable.vue'
import PopulationData from '@/data/population.json'

export default {
  components: {
    ConfirmedCasesByMunicipalitiesTable
  },

  data() {
    // 市町村ごとの陽性患者数
    const municipalitiesTable = {
      headers: [],
      datasets: []
    }

    // 市町村ごとの人口データ
    const populationMap = {}
    PopulationData.datasets.forEach(element => {
      populationMap[element.city] = element.population
    })

    // ヘッダーを設定
    municipalitiesTable.headers = [
      { text: this.$t('市町村'), value: 'label' },
      { text: this.$t('陽性患者数'), value: 'nbCases', align: 'end' },
      { text: this.$t('人口'), value: 'population' },
      {
        text: this.$t('1万人あたりの陽性患者'),
        value: 'casesPerTenThousandPopulation'
      }
    ]

    // データを追加
    for (const item of Data.patients.data) {
      const retrievedMunicipality = municipalitiesTable.datasets.filter(
        _ => _.label === this.$t(item.居住地)
      )
      if (retrievedMunicipality.length === 0) {
        const population = populationMap[item.居住地]
        municipalitiesTable.datasets.push({
          label: this.$t(item.居住地),
          nbCases: 1,
          population
        })
      } else {
        const idx = municipalitiesTable.datasets.indexOf(
          retrievedMunicipality[0]
        )
        municipalitiesTable.datasets[idx].nbCases++
      }
    }

    // データをソート
    municipalitiesTable.datasets.sort((a, b) => {
      return a.nbCases < b.nbCases ? 1 : -1
    })

    const date = dayjs(Data.patients.date).format('YYYY/MM/DD HH:mm')

    const info = {
      sText: this.$t('{date}の累計', {
        date: dayjs(Data.patients.date).format('M/DD')
      })
    }

    const data = {
      Data,
      date,
      municipalitiesTable,
      info
    }
    return data
  }
}
</script>
