<template>
  <div class="flex flex-wrap">
    <SidebarNav></SidebarNav>

    <div class="w-full lg:w-5/6 px-2 pt-2 bg-gray-200 relative">

      <div class="pl-2">
        <div class="flex flex-wrap justify-between items-center">
            <p class="text-2xl font-bold">COVID-19 {{ $t('overview') }}</p>
            <Share :url="shareUrl"></Share>
        </div>
        <p class="text-xs font-bold leading-tight my-2 text-gray-600">
          {{ $t('sources') }}: {{ $t('WHO') }}, CDC, ECDC, NHC of the PRC, JHU CSSE, DXY, QQ, {{ $t('international_media') }}
        </p>
      </div>

      <div class="flex flex-wrap">
        <div class="w-full lg:w-1/2 px-2">
          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <div class="flex flex-row lg:flex-row">
              <div class="px-2">
                <p class="text-sm font-bold text-red-600">{{ $t('total_confirmed') }}</p>
                <p class="text-xl font-bold text-red-600">{{ confirmed | formatNumber }}</p>
              </div>

              <div class="px-2">
                <p class="text-sm font-bold text-green-600">{{ $t('total_recovered') }}</p>
                <p class="text-xl font-bold text-green-600">{{ recovered | formatNumber }}</p>
              </div>

              <div class="px-2">
                <p class="text-sm font-bold text-gray-600">{{ $t('total_deaths') }}</p>
                <p class="text-xl font-bold text-gray-600">{{ deaths | formatNumber }}</p>
              </div>
            </div>
          </div>

          <div class="w-full rounded shadow-md bg-white p-3 mb-5 block lg:hidden">
            <HintClickCountry/>
            <div class="mt-3" style="max-height: 36.3rem; overflow: auto;">
              <table class="table-auto w-full">
                <thead class="text-xs leading-tight border-b-2">
                <tr>
                  <th class="border px-2 py-2">{{ $t('country') }}</th>
                  <th class="border px-1 py-2">{{ $t('total_confirmed') }}</th>
                  <th class="border px-1 py-2">{{ $t('total_recovered') }}</th>
                  <th class="border px-1 py-2">{{ $t('total_deaths') }}</th>
                </tr>
                </thead>
                <tbody class="font-bold">
                <tr v-for="loc in affectedCountryData" :key="loc.country">
                  <td class="bg-gray-200 text-xs border px-2 py-2 hover:bg-primary hover:text-white">
                    <template v-if="loc.countryCode === 'OT'">
                      <span>{{loc.country}}</span>
                      <a href="#notes-on-others">*</a>
                    </template>
                    <template v-else-if="loc.countryCode">
                      <nuxt-link :to="`/country/${loc.countryCode.toLowerCase()}`" style="display: block;">
                        <Flag :country-code="loc.countryCode"></Flag>
                        {{loc.country}}
                      </nuxt-link>
                    </template>
                  </td>
                  <td class="text-center border px-1 py-2">{{ loc.totalConfirmed | formatNumber }}</td>
                  <td class="text-center border px-1 py-2">{{ loc.totalRecovered | formatNumber }}</td>
                  <td class="text-center border px-1 py-2">{{ loc.totalDeaths | formatNumber }}</td>
                </tr>
                </tbody>
              </table>
              <div class="my-2 font-bold text-xs text-gray-600 leading-tight">
                * {{ $t('cruise_ship_notice') }}
                <a name="notes-on-others" class="anchor"></a>
              </div>
            </div>
          </div>

          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <OutbreakTrendChart :data="outbreakTrendData"></OutbreakTrendChart>
          </div>

          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <TopCountryWithDailyNewStats :data="topCountryWithDailyNewStatsData"/>
          </div>
        </div>

        <div class="w-full lg:w-1/2 px-2">
          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <AffectedCountry :data="affectedCountryData"></AffectedCountry>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SidebarNav from '~/components/Analytics/SidebarNav'
import OutbreakTrendChart from '~/components/Analytics/OutbreakTrend'
import TopCountryWithDailyNewStats from '~/components/Analytics/TopCountryWithDailyNewStats'
import AffectedCountry from '~/components/Analytics/AffectedCountry'
import Flag from '~/components/Flag';
import Share from '~/components/Share';
import HintClickCountry from '~/components/HintClickCountry';

export default {
  head() {
    const title = this.$t('covid_related_material');
    const description = this.$t('covid_related_material_description');

    return {
      title,
      meta: [
        { hid: 'title', name: 'title', content: title },
        { hid: 'description', name: 'description', content: description },
        { hid: 'og-title', property: 'og:title', content: title },
        { hid: 'og-description', property: 'og:title', content: description },
        { hid: 'twitter-title', property: 'twitter:title', content: title },
        { hid: 'twitter-description', property: 'twitter:title', content: description },
        { hid: 'og-image', property: 'og:image', content: process.env.API_BASE_URL + '/share/globalStatsToday' },
        { hid: 'twitter-image', property: 'twitter-image', content: process.env.API_BASE_URL + '/share/globalStatsToday' }
      ],
    };
  },

  components: {
    SidebarNav,
    OutbreakTrendChart,
    TopCountryWithDailyNewStats,
    AffectedCountry,
    Flag,
    Share,
    HintClickCountry
  },

  computed: {
    shareUrl() {
      return process.browser ? window.location.href : 'https://www.coronatracker.com/analytics';
    }
  },

  mounted () {
    this.loadStats()
    this.loadOutbreakTrend()
    this.loadTopCountryWithDailyNewStats()
    this.loadAffectedCountry()
  },

  data () {
    return {
      currentDate: new Date,
      confirmed: 0,
      deaths: 0,
      recovered: 0,
      outbreakTrendData: [],
      topCountryWithDailyNewStatsData: [],
      affectedCountryData: [],
    }
  },

  methods: {
    loadStats () {
      this.$api.stats.getGlobalStats()
        .then(data => {
          this.confirmed = data.totalConfirmed
          this.deaths = data.totalDeaths
          this.recovered = data.totalRecovered
        })
    },

    loadOutbreakTrend () {
      const limit = 100
      this.$api.analytics.fetchTopTrendingCases(limit)
        .then(data => {
          this.outbreakTrendData = data
        })
    },

    loadTopCountryWithDailyNewStats () {
      this.$api.analytics.fetchTopCountryWithDailyNewStatsSortByNewCases()
        .then(data => {
          this.topCountryWithDailyNewStatsData = data
        })
    },

    async loadAffectedCountry() {
      const items = await this.$api.stats.getTopNCountryStats()
      this.affectedCountryData = items.filter(a => a.countryCode && a.totalConfirmed)
    }

  }
}
</script>
