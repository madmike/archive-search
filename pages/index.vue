<template>
  <div class="container">
    <div>
      <h2 class="subtitle">
        Универсальный поисковик по архивам открытых данных
      </h2>

      <b-row>
        <b-input-group class="mt-5">
          <b-form-input v-model="q" size="lg" placeholder="Введите свой поисковый запрос" />
          <b-input-group-append>
            <b-button size="lg" variant="outline-success" @click="search">
              Найти
            </b-button>
          </b-input-group-append>
        </b-input-group>
      </b-row>

      <b-row v-if="resultsReady" class="mt-5">
        <b-card id="results" class="w-100" no-body>
          <b-tabs content-class="mt-3" card>
            <b-tab v-for="(archiveTitle, archiveKey) in archives" :key="archiveKey">
              <template slot="title">
                {{ archiveTitle }} <b-badge variant="light">
                  {{ count[archiveKey] }} <span class="sr-only">публикаций</span>
                </b-badge>
              </template>
              <template v-if="count[archiveKey] > 0">
                <p>
                  <b-row><b-col><p>Всего нашлось <b>{{ count[archiveKey] }}</b> публикаций:</p></b-col></b-row>
                  <b-row v-for="(result, idx) in results[archiveKey]" :key="result.url">
                    <b-col cols="1">
                      <h5 class="pt-3">
                        {{ idx + 1 }}.
                      </h5>
                    </b-col>
                    <b-col cols="11" class="result">
                      <b-media>
                        <h5 class="pt-3">
                          <b-link :href="result.url" target="_blank" class="stretched-link">
                            {{ result.title }}
                          </b-link>
                        </h5>
                        <p style="max-height: 150px; overflow-x: hidden;" v-html="result.description" />
                      </b-media>
                    </b-col>
                  </b-row>
                </p>
              </template>
              <template v-else>
                По вашему запросу ничего не найдено...
              </template>
            </b-tab>
          </b-tabs>
        </b-card>
      </b-row>
    </div>
  </div>
</template>

<script>
const axios = require('axios')

export default {
  data() {
    return {
      q: '',
      resultsReady: false,
      archives: {
        noosphere: 'Ноосфера',
        aire: 'Open AIRE',
        europeana: 'Europeana PRO',
        internet_archive: 'Internet Archive',
        wikidata: 'Wikidata'
      },
      results: {},
      count: {}
    }
  },
  methods: {
    async search() {
      try {
        this.$nuxt.$loading.start()
        this.resultsReady = false

        const resNoosphere = await axios.get('https://noosphere.ru/pubs.json?query=' + this.q)
        const resAire = await axios.get('http://api.openaire.eu/search/publications?format=json&size=15&title=' + this.q)
        const resEuropeana = await axios.get('https://www.europeana.eu/api/v2/search.json?query=' + this.q + '&wskey=TtdkGCiKH')
        const resInternetArchive = await axios.get('http://openlibrary.org/search.json?q=' + this.q)
        const resWikiData = await axios.get('https://api.codetabs.com/v1/proxy?quest=' + encodeURIComponent('https://www.wikidata.org/w/api.php?action=wbsearchentities&language=en&format=json&search=' + encodeURIComponent(this.q)))

        console.log(resNoosphere)
        console.log(resAire)
        console.log(resEuropeana)
        console.log(resInternetArchive)
        console.log(resWikiData)

        this.$nuxt.$loading.finish()
        this.resultsReady = true

        this.count.noosphere = resNoosphere.data.numFound
        this.count.aire = resAire.data.response.header.total.$
        this.count.europeana = resEuropeana.data.totalResults
        this.count.internet_archive = resInternetArchive.data.numFound
        this.count.wikidata = resWikiData.data.search.length

        this.results.noosphere = resNoosphere.data.items

        this.results.aire = (resAire.data.response.results ? resAire.data.response.results.result.map((it) => {
          return {
            url: 'https://explore.openaire.eu/search/publication?articleId=' + it.header['dri:objIdentifier'].$,
            title: it.metadata['oaf:entity']['oaf:result'].title.$,
            description: (it.metadata['oaf:entity']['oaf:result'].description || {}).$
          }
        }) : [])

        this.results.europeana = resEuropeana.data.items.map((it) => {
          return {
            url: it.guid,
            title: it.title.join(', '),
            description: (it.dcDescription || []).join('<br />')
          }
        })

        this.results.internet_archive = resInternetArchive.data.docs.map((it) => {
          return {
            url: 'https://openlibrary.org' + it.key,
            title: it.title,
            description: (it.subject || []).join(', ')
          }
        })

        this.results.wikidata = resWikiData.data.search.map((it) => {
          return {
            url: it.url,
            title: it.label,
            description: it.description
          }
        })
      } catch (e) {
        this.error = e.response.data.message
      }
    }
  }
}
</script>

<style lang="scss">
  #results .result:hover { background-color: #EBE2E7; }
</style>
