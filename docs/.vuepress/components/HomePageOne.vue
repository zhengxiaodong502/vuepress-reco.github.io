<template>
  <article class="home-page-one-wrapper">
    <section class="description">
      <h1>{{ $frontmatter.heroText || $title }}</h1>
      <p class="description">{{ $description }}</p>
      <div>
        <iframe
          src="https://ghbtns.com/github-btn.html?user=vuepress-reco&repo=vuepress-theme-reco&type=star&count=true&q=GitHub+Octocat+in:readme+user:defunkt"
          frameborder="0"
          scrolling="0"
          width="100px"
          height="20px"
        ></iframe>
        <img
          :src="`https://img.shields.io/badge/download-${downloads}-orange.svg?style=social&logo=npm`"
          alt=""
        >
        <!-- <iframe src="https://ghbtns.com/github-btn.html?user=vuepress-reco&repo=vuepress-theme-reco&type=fork&count=true" frameborder="0" scrolling="0" width="100px" height="20px"></iframe>
        <iframe src="https://ghbtns.com/github-btn.html?user=vuepress-reco&repo=vuepress-theme-reco&type=watch&count=true&v=2" frameborder="0" scrolling="0" width="100px" height="20px"></iframe>-->
      </div>
      <router-link class="btn-about" :to="$frontmatter.actionLink">{{ $frontmatter.actionText }}</router-link>
      <img class="banner" src="./images/blog.svg" alt="">
    </section>
    <section class="wish yesterday">
      <div class="wish-inner">
        <div class="img-wrapper">
          <img src="./images/yesterday.svg" alt="">
        </div>
        <div class="text-wrapper">
          <h1>{{ features[0].title }}</h1>
          <p class="description">{{ features[0].details }}</p>
        </div>
      </div>
    </section>
    <section class="wish today">
      <div class="wish-inner">
        <div class="text-wrapper">
          <h1>{{ features[1].title }}</h1>
          <p class="description">{{ features[1].details }}</p>
        </div>
        <div class="img-wrapper">
          <img src="./images/today.svg" alt="">
        </div>
      </div>
    </section>
    <section class="wish tomorrow">
      <div class="wish-inner">
        <div class="img-wrapper">
          <img src="./images/tomorrow.svg" alt="">
        </div>
        <div class="text-wrapper">
          <h1>{{ features[2].title }}</h1>
          <p class="description">{{ features[2].details }}</p>
        </div>
      </div>
    </section>
    <section class="md-content-wrapper">
      <Content/>
    </section>
  </article>
</template>

<script>
export default {
  data () {
    return {
      downloads: 0
    }
  },

  computed: {
    features () {
      return this.$frontmatter.features
    }
  },

  created () {
    const date = new Date()
    const year = date.getFullYear()
    const mounth = date.getMonth() + 1
    const day = date.getDate()
    this.npmPackageDownloads('vuepress-theme-reco', `2018-09-12:${year}-${mounth}-${day}`).then(res => {
      this.downloads = this.toThousandslsFilter(res)
    })
  },

  methods: {
    toThousandslsFilter (num) {
      const numStr = String(num)
      if (numStr === '' || numStr == undefined) {
        return ''
      }
      const IntPart = /\./g.test(numStr) ? numStr.slice(0, numStr.indexOf('.')) : numStr
      const FloatPart = /\./g.test(numStr) ? numStr.substring(numStr.indexOf('.')) : ''

      const orderPrice2 = (+IntPart || 0).toString().replace(/^-?\d+/g, m => m.replace(/(?=(?!\b)(\d{3})+$)/g, ',')) + FloatPart
      return orderPrice2
    },

    npmPackageDownloads (packages, dateRange) {
      packages = this._handlePackages(packages)
      dateRange = this._handleDateRange(dateRange)
      return this._getDownloadsOfDateRange(packages, dateRange)
    },

    async _getDownloadsOfDateRange (packages, dateRange) {
      let downloads = 0
      if (Array.isArray(dateRange)) {
        let fetchPromise = []
        dateRange.map(item => {
          fetchPromise.push(this._fetch(packages, item))
        })
        const result = await Promise.all(fetchPromise)
        downloads = result.reduce((all, next) => {
          return all + next.downloads
        }, 0)
        return downloads
      }
      const result = await this._fetch(packages, dateRange)
      downloads = result.downloads
      return downloads
    },

    _fetch (packages, dateRange) {
      const BASE_URI = 'https://api.npmjs.org/downloads/point/'
      return new Promise ((resolve, reject) => {
        fetch(`${BASE_URI}${dateRange}/${packages}`).then(this._parseJSON).then(res => {
          resolve(res)
        }).catch(err => {
          reject(err)
        })
      })
    },

    _parseJSON (response) {
      return response.json()
    },

    _handleDateRange (dateRange) {
      const index = dateRange.indexOf(':')
      if (index> -1) {
        const dr = dateRange.split(':')
        const newDateRange = dr
        const YEAR = 365 * 24 * 60 * 60 * 1000
        const DATE_RANGE = new Date(dr[1]).getTime() - new Date(dr[0]).getTime()
        const year = parseInt(DATE_RANGE / YEAR)
        if (year > 0) {
          for (let i = 0; i < year; i++) {
            const date = this._getDate(newDateRange[i])
            newDateRange.splice(i + 1, 0, date)
          }
          for (let i = 0, length = newDateRange.length; i < length - 1; i++) {
            newDateRange[i] = `${newDateRange[i]}:${newDateRange[i + 1]}`
          }
          newDateRange.length = year + 1
          return newDateRange
        }
        return dateRange
      }
      return dateRange
    },

    _getDate (date) {
      const dateArr = date.split('-')
      dateArr[0] = Number(dateArr[0]) + 1
      return dateArr.join('-')
    },

    _handlePackages (packages) {
      if (Array.isArray(packages)) {
        return `-,${packages.join(',')}`
      }
      return packages
    }
  }
}
</script>


<style lang="stylus" scoped>
.home-page-one-wrapper {
  padding-top: 3.4rem;

  section {
    &.description {
      box-sizing: border-box;
      margin: 0 auto 6rem;
      max-width: 46rem;
      width: 100%;
      text-align: center;

      h1 {
        margin-top: 8rem;
      }

      p {
        margin-bottom: 2rem;
      }

      .btn-about {
        margin: 2rem 0;
        display: inline-block;
        padding: 1rem 3rem;
        border-radius: 0.25rem;
        background: $accentColor;
        color: #fff;
      }

      .banner {
        display: block;
        width: 100%;
      }
    }

    &.wish {
      overflow: hidden;

      &.yesterday, &.tomorrow {
        background: var(--code-color);
      }

      .wish-inner {
        box-sizing: border-box;
        margin: 0 auto;
        padding: 2rem;
        max-width: 56rem;
        width: 100%;
        min-height: 25rem;
        display: flex;
        align-items: center;

        > div {
          flex: auto;

          &.img-wrapper {
            max-width: 22rem;

            img {
              display: block;
              width: 100%;
            }
          }

          &.text-wrapper {
            box-sizing: border-box;
            padding: 0 2rem;
          }
        }
      }
    }
  }
}

@media (max-width: $MQMobile) {
  .home-page-one-wrapper {
    section {
      padding: 0 2rem;

      &.description {
        h1 {
          margin-top: 5rem;
        }
      }

      &.md-content-wrapper {
        padding: 0;
      }

      &.wish {
        .wish-inner {
          display: block;
          padding: 2rem 0;

          .img-wrapper {
            margin: 0 auto;
          }
        }
      }
    }
  }
}

@media (max-width: $MQMobileNarrow) {
  .home-page-one-wrapper {
    section {
      padding: 0 2rem;

      &.description {
        h1 {
          margin-top: 5rem;
        }
      }

      &.md-content-wrapper {
        padding: 0;
      }

      &.wish {
        .wish-inner {
          display: block;
          padding: 2rem 0;

          .img-wrapper {
            margin: 0 auto;
          }
        }
      }
    }
  }
}
</style>