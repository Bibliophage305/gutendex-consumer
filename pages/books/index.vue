<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>
              Filters
              <v-spacer />
              <v-menu left :close-on-content-click="closeOnContentClick">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn color="primary" dark v-bind="attrs" v-on="on">
                    <v-icon left> mdi-plus </v-icon>
                    Add Filter
                  </v-btn>
                </template>

                <v-list>
                  <v-list-item v-if="!filters.length">
                    <v-list-item-title> No more filters </v-list-item-title>
                  </v-list-item>
                  <v-list-item
                    v-for="filter in inactiveFilters"
                    :key="filter.name"
                  >
                    <v-list-item-title>{{ filter.title }}</v-list-item-title>
                    <v-list-item-action>
                      <v-btn icon @click="activeFilters.push(filter.name)">
                        <v-icon> mdi-plus </v-icon>
                      </v-btn>
                    </v-list-item-action>
                  </v-list-item>
                </v-list>
              </v-menu>
            </v-card-title>
            <v-card-text>
              <v-list-item
                v-for="filter in filters.filter((filter) =>
                  activeFilters.includes(filter.name)
                )"
                :key="filter.name"
              >
                <v-select
                  v-if="filter.type === 'select' && filter.searchable === false"
                  :items="filter.options"
                  v-model="filter.value"
                  :multiple="filter.multiple"
                  item-text="text"
                  item-value="value"
                  :label="filter.title"
                  :chips="filter.multiple"
                  deletable-chips
                />
                <v-autocomplete
                  v-else-if="
                    filter.type === 'select' && filter.searchable === true
                  "
                  :items="filter.options"
                  v-model="filter.value"
                  :multiple="filter.multiple"
                  item-text="text"
                  item-value="value"
                  :label="filter.title"
                  :chips="filter.multiple"
                  deletable-chips
                />
                <v-text-field
                  v-else
                  v-model="filter.value"
                  :label="filter.title"
                  :type="filter.type"
                />
                <v-list-item-action>
                  <v-btn
                    icon
                    @click="
                      activeFilters = activeFilters.filter(
                        (n) => n !== filter.name
                      )
                    "
                  >
                    <v-icon> mdi-close </v-icon>
                  </v-btn>
                </v-list-item-action>
              </v-list-item>
            </v-card-text>
            <v-card-actions>
              <span v-if="!$fetchState.pending && $fetchState.error === null"
                >{{ bookTotal }} books found</span
              >
              <v-spacer />
              <v-btn
                color="primary"
                :disabled="$fetchState.pending"
                @click="$fetch"
              >
                Search
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-row v-if="$fetchState.pending">
        <v-col>
          <v-card loading>
            <v-card-title> Fetching books </v-card-title>
            <v-card-text> Getting them from the api </v-card-text>
          </v-card>
        </v-col>
      </v-row>
      <v-row v-else-if="$fetchState.error">
        <v-col>
          <v-card>
            <v-card-title> There was a problem </v-card-title>
            <v-card-text> It broke </v-card-text>
          </v-card>
        </v-col>
      </v-row>
      <template v-else>
        <v-row v-for="book in books" :key="book.id">
          <v-col>
            <book-card :book="book" />
          </v-col>
        </v-row>
        <v-row v-if="nextUrl != null">
          <v-col class="text-center">
            <v-progress-circular
              indeterminate
              color="primary"
              v-intersect="intersect"
            />
          </v-col>
        </v-row>
      </template>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'BooksPage',

  data() {
    return {
      books: [],
      nextUrl: null,
      bookTotal: 0,
      allowLoading: true,
      filterListVisible: false,
      filters: [
        {
          name: 'sort',
          title: 'Sorting',
          type: 'select',
          searchable: false,
          multiple: false,
          options: [
            { text: 'Popular', value: '' },
            { text: 'Ascending ID', value: 'ascending' },
            { text: 'Descending ID', value: 'descending' },
          ],
          value: '',
        },
        {
          name: 'search',
          title: 'Search by text',
          type: 'text',
          value: '',
        },
        {
          name: 'topic',
          title: 'Topic',
          type: 'text',
          value: '',
        },
        {
          name: 'author_year_start',
          title: 'Author year start',
          type: 'number',
          value: '',
        },
        {
          name: 'author_year_end',
          title: 'Author year end',
          type: 'number',
          value: '',
        },
        {
          name: 'copyright',
          title: 'Copyright',
          type: 'select',
          searchable: false,
          multiple: true,
          options: [
            { text: 'In copyright', value: 'true' },
            { text: 'Out of copyright', value: 'false' },
            { text: 'No information available', value: 'null' },
          ],
          value: [],
        },
        {
          name: 'languages',
          title: 'Languages',
          type: 'select',
          searchable: true,
          multiple: true,
          options: [
            { text: 'Abkhazian', value: 'ab' },
            { text: 'Afar', value: 'aa' },
            { text: 'Afrikaans', value: 'af' },
            { text: 'Albanian', value: 'sq' },
            { text: 'Amharic', value: 'am' },
            { text: 'Arabic', value: 'ar' },
            { text: 'Armenian', value: 'hy' },
            { text: 'Assamese', value: 'as' },
            { text: 'Aymara', value: 'ay' },
            { text: 'Azerbaijani', value: 'az' },
            { text: 'Bashkir', value: 'ba' },
            { text: 'Basque', value: 'eu' },
            { text: 'Bengali, Bangla', value: 'bn' },
            { text: 'Bhutani', value: 'dz' },
            { text: 'Bihari', value: 'bh' },
            { text: 'Bislama', value: 'bi' },
            { text: 'Breton', value: 'br' },
            { text: 'Bulgarian', value: 'bg' },
            { text: 'Burmese', value: 'my' },
            { text: 'Byelorussian', value: 'be' },
            { text: 'Cambodian', value: 'km' },
            { text: 'Catalan', value: 'ca' },
            { text: 'Chinese', value: 'zh' },
            { text: 'Corsican', value: 'co' },
            { text: 'Croatian', value: 'hr' },
            { text: 'Czech', value: 'cs' },
            { text: 'Danish', value: 'da' },
            { text: 'Dutch', value: 'nl' },
            { text: 'English', value: 'en' },
            { text: 'Esperanto', value: 'eo' },
            { text: 'Estonian', value: 'et' },
            { text: 'Faeroese', value: 'fo' },
            { text: 'Fiji', value: 'fj' },
            { text: 'Finnish', value: 'fi' },
            { text: 'French', value: 'fr' },
            { text: 'Frisian', value: 'fy' },
            { text: 'Gaelic (Scots Gaelic)', value: 'gd' },
            { text: 'Galician', value: 'gl' },
            { text: 'Georgian', value: 'ka' },
            { text: 'German', value: 'de' },
            { text: 'Greek', value: 'el' },
            { text: 'Greenlandic', value: 'kl' },
            { text: 'Guarani', value: 'gn' },
            { text: 'Gujarati', value: 'gu' },
            { text: 'Hausa', value: 'ha' },
            { text: 'Hebrew', value: 'iw' },
            { text: 'Hindi', value: 'hi' },
            { text: 'Hungarian', value: 'hu' },
            { text: 'Icelandic', value: 'is' },
            { text: 'Indonesian', value: 'in' },
            { text: 'Interlingua', value: 'ia' },
            { text: 'Interlingue', value: 'ie' },
            { text: 'Inupiak', value: 'ik' },
            { text: 'Irish', value: 'ga' },
            { text: 'Italian', value: 'it' },
            { text: 'Japanese', value: 'ja' },
            { text: 'Javanese', value: 'jw' },
            { text: 'Kannada', value: 'kn' },
            { text: 'Kashmiri', value: 'ks' },
            { text: 'Kazakh', value: 'kk' },
            { text: 'Kinyarwanda', value: 'rw' },
            { text: 'Kirghiz', value: 'ky' },
            { text: 'Kirundi', value: 'rn' },
            { text: 'Korean', value: 'ko' },
            { text: 'Kurdish', value: 'ku' },
            { text: 'Laothian', value: 'lo' },
            { text: 'Latin', value: 'la' },
            { text: 'Latvian, Lettish', value: 'lv' },
            { text: 'Lingala', value: 'ln' },
            { text: 'Lithuanian', value: 'lt' },
            { text: 'Macedonian', value: 'mk' },
            { text: 'Malagasy', value: 'mg' },
            { text: 'Malay', value: 'ms' },
            { text: 'Malayalam', value: 'ml' },
            { text: 'Maltese', value: 'mt' },
            { text: 'Maori', value: 'mi' },
            { text: 'Marathi', value: 'mr' },
            { text: 'Moldavian', value: 'mo' },
            { text: 'Mongolian', value: 'mn' },
            { text: 'Nauru', value: 'na' },
            { text: 'Nepali', value: 'ne' },
            { text: 'Norwegian', value: 'no' },
            { text: 'Occitan', value: 'oc' },
            { text: 'Oriya', value: 'or' },
            { text: 'Oromo, Afan', value: 'om' },
            { text: 'Pashto, Pushto', value: 'ps' },
            { text: 'Persian', value: 'fa' },
            { text: 'Polish', value: 'pl' },
            { text: 'Portuguese', value: 'pt' },
            { text: 'Punjabi', value: 'pa' },
            { text: 'Quechua', value: 'qu' },
            { text: 'Rhaeto-Romance', value: 'rm' },
            { text: 'Romanian', value: 'ro' },
            { text: 'Russian', value: 'ru' },
            { text: 'Samoan', value: 'sm' },
            { text: 'Sangro', value: 'sg' },
            { text: 'Sanskrit', value: 'sa' },
            { text: 'Serbian', value: 'sr' },
            { text: 'Serbo-Croatian', value: 'sh' },
            { text: 'Sesotho', value: 'st' },
            { text: 'Setswana', value: 'tn' },
            { text: 'Shona', value: 'sn' },
            { text: 'Sindhi', value: 'sd' },
            { text: 'Singhalese', value: 'si' },
            { text: 'Siswati', value: 'ss' },
            { text: 'Slovak', value: 'sk' },
            { text: 'Slovenian', value: 'sl' },
            { text: 'Somali', value: 'so' },
            { text: 'Spanish', value: 'es' },
            { text: 'Sudanese', value: 'su' },
            { text: 'Swahili', value: 'sw' },
            { text: 'Swedish', value: 'sv' },
            { text: 'Tagalog', value: 'tl' },
            { text: 'Tajik', value: 'tg' },
            { text: 'Tamil', value: 'ta' },
            { text: 'Tatar', value: 'tt' },
            { text: 'Tegulu', value: 'te' },
            { text: 'Thai', value: 'th' },
            { text: 'Tibetan', value: 'bo' },
            { text: 'Tigrinya', value: 'ti' },
            { text: 'Tonga', value: 'to' },
            { text: 'Tsonga', value: 'ts' },
            { text: 'Turkish', value: 'tr' },
            { text: 'Turkmen', value: 'tk' },
            { text: 'Twi', value: 'tw' },
            { text: 'Ukrainian', value: 'uk' },
            { text: 'Urdu', value: 'ur' },
            { text: 'Uzbek', value: 'uz' },
            { text: 'Vietnamese', value: 'vi' },
            { text: 'Volapuk', value: 'vo' },
            { text: 'Welsh', value: 'cy' },
            { text: 'Wolof', value: 'wo' },
            { text: 'Xhosa', value: 'xh' },
            { text: 'Yiddish', value: 'ji' },
            { text: 'Yoruba', value: 'yo' },
            { text: 'Zulu', value: 'zu' },
          ],
          value: [],
        },
      ],
      activeFilters: [],
    }
  },

  async fetch() {
    let url = new URL('https://gutendex.com/books')
    for (let filter of this.filters) {
      if (this.activeFilters.includes(filter.name) && filter.value) {
        url.searchParams.append(filter.name, filter.value)
      }
    }
    const data = await this.getBooks(url)
    this.books = data.results
    this.nextUrl = data.next
    this.bookTotal = data.count
  },

  computed: {
    inactiveFilters() {
      return this.filters.filter(
        (filter) => !this.activeFilters.includes(filter.name)
      )
    },

    filterSentence() {
      if (this.activeFilters.length === 0) {
        return 'No filters active'
      }
      return (
        'Active filters: ' +
        this.filters
          .filter((filter) => this.activeFilters.includes(filter.name))
          .map((filter) => filter.title)
          .join(', ')
      )
    },
  },

  methods: {
    async getBooks(url) {
      this.allowLoading = false
      const response = await fetch(url)
      const data = await response.json()
      console.log(data)
      this.allowLoading = true
      return data
    },

    async intersect(entries) {
      if (
        this.allowLoading &&
        entries[0].isIntersecting &&
        this.nextUrl != null
      ) {
        const data = await this.getBooks(this.nextUrl)
        this.books = this.books.concat(data.results)
        this.nextUrl = data.next
      }
    },
  },
}
</script>
