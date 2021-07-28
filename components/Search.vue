<template>
  <div>
    <ais-instant-search
        index-name="dev_pawkat"
        :search-client="searchClient"
        :search-function="searchFunction"
        :routing="routing"
    >
      <ais-index index-name="dev_pawkat">
        <ais-search-box autofocus>
          <input
              slot-scope="{ currentRefinement, refine }"
              :value="currentRefinement"
              @input="refine($event.currentTarget.value)"
              placeholder="Enter your search keyword"
              class="search-input"
          />
        </ais-search-box>

        <ais-hits-per-page
            :items="histPerPageItems"
        >
          <template slot-scope="{ items, refine, createURL }">
            <v-select
                :items="items"
                label="Items per page"
                v-model="histPerPage"
                @change="changeHitsPerPage(refine)"
                item-text="label"
                item-value="value"
            >

            </v-select>
          </template>
        </ais-hits-per-page>

        <div class="search-controls form-inline">
          <ais-powered-by/>

          <ais-stats class="text-caption"/>
        </div>

        <ais-state-results>
          <p
              slot-scope="{ state: { query }, results: { hits } }"
              v-show="!hits.length"
          >
            No results found for the query: <q>{{ query }}</q>
          </p>
          <div class="v-data-table theme--dark" slot-scope="{ state: { query }, results: { hits } }">
            <div class="v-data-table__wrapper">
              <table>
                <thead>
                <tr>
                  <th class="text-left">
                    Name
                  </th>
                  <th class="text-left">
                    Price
                  </th>
                  <th class="text-center table-image-cell">
                    Image
                  </th>
                </tr>
                </thead>
                <tbody>
                <v-dialog
                    transition="dialog-bottom-transition"
                    max-width="600"
                    v-for="item in hits"
                    :key="item.objectID"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <tr
                        v-bind="attrs"
                        v-on="on"
                    >
                      <td class="py-2">
                        {{ item.name }}
                      </td>
                      <td class="py-2">{{ item.price }}</td>
                      <td class="py-2 table-image-cell">
                        <div class="item-image" v-if="item.image">
                          <img :src="item.image" :alt="item.name"/>
                        </div>
                      </td>
                    </tr>
                  </template>
                  <template v-slot:default="dialog">
                    <v-card>
                      <v-toolbar
                          color="primary"
                          dark
                      >{{ item.name }}
                      </v-toolbar>
                      <v-card-text>
                        <div class="pt-6">
                          <div class="item-image mb-4" v-if="item.image">
                            <img :src="item.image" :alt="item.name"/>
                          </div>
                          <p>Brand: {{ item.brand }}</p>
                          <p>Price: ${{ item.price }}</p>
                          <p>Categories: {{ item.categories.join(', ') }}</p>
                        </div>
                      </v-card-text>
                      <v-card-actions class="justify-end">
                        <v-btn
                            text
                            @click="dialog.value = false"
                        >Close
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </template>
                </v-dialog>
                </tbody>
              </table>
            </div>
          </div>

        </ais-state-results>


        <ais-pagination :padding="2">
          <ul slot-scope="{
                  currentRefinement,
                  nbPages,
                  pages,
                  isFirstPage,
                  isLastPage,
                  refine,
                  createURL
                }"
              class="v-pagination theme--dark"
          >
            <li v-if="!isFirstPage">
              <a type="button" aria-label="Next page" class="v-pagination__navigation"
                 :href="createURL(0)" @click.prevent="refine(0)">
                <i aria-hidden="true" class="v-icon notranslate mdi mdi-chevron-double-left theme--dark"></i>
              </a>
            </li>
            <li v-if="!isFirstPage" class="prev">
              <a type="button" aria-label="Previous page"
                 class="v-pagination__navigation"
                 :href="createURL(currentRefinement - 1)"
                 @click.prevent="refine(currentRefinement - 1)">
                <i aria-hidden="true" class="v-icon notranslate mdi mdi-menu-left theme--dark"></i>
              </a>
            </li>
            <li v-for="page in pages" :key="page">
              <a type="button" aria-label="Goto Page 2" class="v-pagination__item"
                 :href="createURL(page)"
                 :class="page===currentRefinement ? 'v-pagination__item--active primary' : ''"
                 @click.prevent="refine(page)">{{ page + 1 }}</a>
            </li>
            <li v-if="!isLastPage" class="next">
              <a type="button" aria-label="Next page" class="v-pagination__navigation"
                 :href="createURL(currentRefinement + 1)"
                 @click.prevent="refine(currentRefinement + 1)">
                <i aria-hidden="true" class="v-icon notranslate mdi mdi-menu-right theme--dark"></i>
              </a>
            </li>
            <li v-if="!isLastPage">
              <a type="button" aria-label="Next page" class="v-pagination__navigation"
                 :href="createURL(nbPages)" @click.prevent="refine(nbPages)">
                <i aria-hidden="true" class="v-icon notranslate mdi mdi-chevron-double-right theme--dark"></i>
              </a>
            </li>
          </ul>
        </ais-pagination>

      </ais-index>
    </ais-instant-search>
  </div>

</template>
<script>
import algoliasearch from 'algoliasearch';
import {history} from 'instantsearch.js/es/lib/routers';
import {simple} from 'instantsearch.js/es/lib/stateMappings';

export default {
  data() {
    return {
      searchClient: algoliasearch(
          'P1DNYP103U',
          '603319c5d40d1b1c5a3ca4cb2447a961'
      ),
      routing: {
        router: history(),
        stateMapping: simple(),
      },
      searchFunction(helper) {
        const page = helper.getPage(); // Retrieve the current page
        // if (this.query) {
        helper.setQuery() // this call resets the page
            .setPage(page) // we re-apply the previous page
            .search();
        // }
      },
      histPerPage: {label: '10 hits per page', value: 10, default: true},
      histPerPageItems: [
        {label: '10 hits per page', value: 10, default: true},
        {label: '20 hits per page', value: 20},
      ],
    }
  },
  methods: {
    changeHitsPerPage(refine) {
      refine(this.histPerPage);
    },
  },
  watch: {
    query: function (newQuery, oldQuery) {
      // console.log(newQuery, oldQuery);
      if (newQuery) {
        this.$router.push({query: {query: newQuery}})
      } else {
        this.$router.push({query: {}})
      }
    },
  }
}
</script>
<style lang="scss">


.ais {
  &-InstantSearch {
    padding: 40px 0;
  }

  &-SearchBox {
    &:not(:last-child) {
      margin-bottom: 20px;
    }
  }

  &-Pagination {
    &:not(:first-child) {
      margin-top: 40px;
    }
  }
}

.search-input {
  display: block;
  border-bottom: 1px solid $white;
  width: 100%;
  color: $white;
  outline: none;
  box-shadow: none;
}


.search-controls {
  text-align: right;

  &:not(:last-child) {
    margin-bottom: 40px;
  }
}

.v-pagination {
  & > li {
    &.prev, &.next {
      @include md-max {
        display: none;
      }
    }
  }

  a {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.item-image {
  display: flex;
  align-items: center;
  justify-content: center;
}

.table-image-cell {
  @include md-max {
    display: none;
  }
}
</style>
