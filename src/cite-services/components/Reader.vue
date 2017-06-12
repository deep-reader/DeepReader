<template>
  <div class="root">
    <header>
      <div class="reader-nav">
        <a href="/#/cts/">CTS API (Perseids)</a>
        &bull;
        <a href="/#/cite-services/">cite-services</a>
        &bull;
        <a href="/#/morphgnt/">MorphGNT API</a>
      </div>
      <h1><b>LORE</b>: Learner’s Online Reading Environment (cite-services)</h1>
    </header>
    <div class="grid-wrapper">
      <div class="left">
        <CtsUrn></CtsUrn>
      </div>

      <div class="main">
        <template v-if="passage">

          <pagination :prev="passageLink(query, passage.prev)" :next="passageLink(query, passage.next)" :title="passage.title"></pagination>

          <div id="text" :class="'textSize-' + this.textSize">
            <div v-for="node in passage.text.Nodes">
              {{ node.text }}
            </div>
          </div>

          <pagination :prev="passageLink(query, passage.prev)" :next="passageLink(query, passage.next)" :title="passage.title"></pagination>

        </template>
        <div v-else>
          Nothing here yet.
        </div>
      </div>
      <div class="right">
        <text-formatting></text-formatting>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import fetch from 'universal-fetch';

import Pagination from '@/components/Pagination';
import TextFormatting from '@/components/TextFormatting';
import CtsUrn from '@/cite-services/components/CtsUrn';

export default {
  name: 'reader',
  created() {
    this.fetchData();
  },
  mounted() {
    window.addEventListener('keyup', this.handleKeyUp);
  },
  beforeDestroy() {
    window.removeEventListener('keyup', this.handleKeyUp);
  },
  data() {
    return {
      query: null,
    };
  },
  computed: mapGetters(['user', 'passage', 'textSize']),
  watch: {
    $route: 'fetchData',
  },
  methods: {
    fetchData() {
      this.asyncData({ query: this.$route.query }).then(({ query, passage }) => {
        this.query = query;
        this.$store.commit('setReader', { passage });
      });
    },
    async asyncData({ query }) {
      const urn = query.urn;
      const url = `http://localhost:8080/million/texts/${urn}`;
      const response = await fetch(url);
      const text = await response.json();
      const passage = {
        text,
        next: text.Nodes[0].next,
        prev: text.Nodes[0].previous,
      };
      return { query, passage };
    },
    passageLink(query, urn) {
      if (urn) {
        return {
          query: Object.assign({}, query, { urn }),
        };
      }
      return null;
    },
    handleKeyUp(e) {
      if (e.key === 'ArrowLeft') {
        if (this.passage.prev) {
          this.$router.push(this.passageLink(this.query, this.passage.prev));
        }
      } else if (e.key === 'ArrowRight') {
        if (this.passage.next) {
          this.$router.push(this.passageLink(this.query, this.passage.next));
        }
      }
    },
    handleWordSelect(word) {
      this.$emit('word-select', word);
      this.$emit('word-select2', word);
    },
  },
  components: {
    Pagination,
    TextFormatting,
    CtsUrn,
  },
};
</script>

<style lang="scss">

  /* variables */

  $main-font-family: "Skolar";
  $widget-font-family: "PT Sans", $main-font-family;

  /* hover opacity */

  .widget, .root > header, .page-nav-1 {
    opacity: 0;
    transition: opacity 0.5s ease-in-out;
  }

  body:hover {
    .widget, .root > header, .page-nav-1 {
      opacity: 1;
    }
  }

  /* grid */

  .grid-wrapper {
    display: grid;
    grid-template-columns: 2fr 6fr 4fr;
    grid-column-gap: 10px;
    margin: 10px;
    > * {
      min-width: 200px;
    }
  }

  /* header */

  .root > header {
    background: #F7F7F7;
    padding: 10px 20px;
    > h1 {
      font-size: 24pt;
      font-family: $main-font-family;
      margin: 0;
      font-weight: normal;
      color: #444;
    }
    .reader-nav {
      float: right;
      font-family: $widget-font-family;
      color: #999;
      a {
        text-decoration: none;
        color: inherit;
        cursor: pointer;
        &:hover {
          color: #000;
        }
      }
    }
  }

  /* main column */

  .main {
    font-family: $main-font-family;
    height: 500px;
    margin: 20px 50px;
    > p:first-of-type {
      margin-top: 0;
    }
  }

  /* text */

  #text {
    clear: both;
    word-spacing: 0.3em;
    color: #333;
    &.textSize-small {
      font-size: 14pt;
    }
    &.textSize-normal {
      font-size: 16pt;
    }
    &.textSize-large {
      font-size: 20pt;
    }
  }

  /* widgets */

  .widget {
    margin-bottom: 10px;
    font-family: $widget-font-family;
    color: #666;
    transition: color 0.2s ease-in-out, opacity 0.5s ease-in-out;
    background: #F7F7F7;
    font-size: 9pt;
    > header {
      cursor: pointer;
      background: #EEE;
      transition: background 0.2s ease-in-out, opacity 0.5s ease-in-out;
      padding: 5px 10px;
      font-weight: bold;
      .summary {
        float: right;
        font-weight: normal;
      }
    }
    > section {
      padding: 10px 10px;
      ul {
        list-style-type: none;
        margin: 0;
        padding: 0;
      }
      .textSize-small {
        font-family: $main-font-family;
        font-size: 14pt;
      }
      .textSize-normal {
        font-family: $main-font-family;
        font-size: 16pt;
      }
      .textSize-large {
        font-family: $main-font-family;
        font-size: 20pt;
      }
    }
    &:hover {
      color: #000;
      > header {
        background: #DDD;
      }
    }
  }

  /* pagination */

  div.page-nav-1 {

    display: flex;
    justify-content: space-between;

    overflow: auto;

    font-family: $widget-font-family;

    > div {
      &.prev {
        width: 30px;  // fix width so takes up space even without link
        float: left;
        text-align: left;
      }
      &.next {
        width: 30px;  // fix width so takes up space even without link
        float: right;
        text-align: right;
      }
      line-height: 20pt;
    }

    .title {
      text-align: center;
    }

    a {
      font-weight: bold;
      font-size: 20pt;
      text-decoration: none;
      color: #999;
      cursor: pointer;
    }

    a:hover {
      color: #000;
    }
  }
</style>